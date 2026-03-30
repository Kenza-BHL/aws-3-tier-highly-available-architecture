[implementation-steps.md](https://github.com/user-attachments/files/26361435/implementation-steps.md)
# 📘 Implementation Steps

Step-by-step guide to deploy the 3-tier highly available architecture on AWS using the AWS Management Console.

---

## Prerequisites

- AWS Account with appropriate permissions
- Basic understanding of AWS VPC, EC2, and RDS concepts
- A simple web application (e.g., Apache/Nginx HTML page for testing)

---

## Step 1 — Create the VPC

1. Go to **VPC Console → Your VPCs → Create VPC**
2. Set the following:
   - Name: `project-vpc`
   - IPv4 CIDR: `10.0.0.0/16`
3. Click **Create VPC**

---

## Step 2 — Create Subnets

Create 6 subnets across 2 Availability Zones:

| Subnet Name | AZ | CIDR | Type |
|---|---|---|---|
| `public-subnet-az1` | AZ1 | `10.0.0.0/24` | Public |
| `public-subnet-az2` | AZ2 | `10.0.1.0/24` | Public |
| `private-app-subnet-az1` | AZ1 | `10.0.2.0/24` | Private |
| `private-app-subnet-az2` | AZ2 | `10.0.3.0/24` | Private |
| `private-db-subnet-az1` | AZ1 | `10.0.4.0/24` | Private |
| `private-db-subnet-az2` | AZ2 | `10.0.5.0/24` | Private |

For each subnet:
1. Go to **Subnets → Create Subnet**
2. Select the VPC created in Step 1
3. Enter the name, AZ, and CIDR as shown above

---

## Step 3 — Create and Attach Internet Gateway

1. Go to **Internet Gateways → Create Internet Gateway**
   - Name: `project-igw`
2. After creation, click **Actions → Attach to VPC**
3. Select `project-vpc`

---

## Step 4 — Create NAT Gateways

Create one NAT Gateway per public subnet:

**NAT Gateway 1 (AZ1):**
1. Go to **NAT Gateways → Create NAT Gateway**
2. Name: `nat-gw-az1`
3. Subnet: `public-subnet-az1`
4. Connectivity: **Public**
5. Click **Allocate Elastic IP**, then **Create NAT Gateway**

**NAT Gateway 2 (AZ2):** Repeat with `public-subnet-az2`, name `nat-gw-az2`

> ⚠️ NAT Gateways take a few minutes to become available.

---

## Step 5 — Configure Route Tables

**Public Route Table:**
1. Go to **Route Tables → Create Route Table**
   - Name: `public-rt`, VPC: `project-vpc`
2. Edit Routes → Add route:
   - Destination: `0.0.0.0/0` → Target: `project-igw`
3. Subnet Associations → Associate both public subnets

**Private Route Table AZ1:**
1. Create Route Table: `private-rt-az1`
2. Add route: `0.0.0.0/0` → Target: `nat-gw-az1`
3. Associate: `private-app-subnet-az1` and `private-db-subnet-az1`

**Private Route Table AZ2:**
1. Create Route Table: `private-rt-az2`
2. Add route: `0.0.0.0/0` → Target: `nat-gw-az2`
3. Associate: `private-app-subnet-az2` and `private-db-subnet-az2`

---

## Step 6 — Create Security Groups

**ALB Security Group** (`alb-sg`):
| Rule | Type | Port | Source |
|---|---|---|---|
| Inbound | HTTP | 80 | `0.0.0.0/0` |
| Inbound | HTTPS | 443 | `0.0.0.0/0` |

**EC2 Security Group** (`ec2-sg`):
| Rule | Type | Port | Source |
|---|---|---|---|
| Inbound | HTTP | 80 | `alb-sg` |
| Inbound | HTTPS | 443 | `alb-sg` |

**RDS Security Group** (`rds-sg`):
| Rule | Type | Port | Source |
|---|---|---|---|
| Inbound | MySQL/Aurora | 3306 | `ec2-sg` |

---

## Step 7 — Create IAM Role for EC2

1. Go to **IAM → Roles → Create Role**
2. Trusted entity: **EC2**
3. Attach policy: `AmazonSSMManagedInstanceCore` (for Session Manager access)
4. Name: `ec2-ssm-role`

---

## Step 8 — Create Launch Template

1. Go to **EC2 → Launch Templates → Create Launch Template**
2. Configure:
   - Name: `web-server-template`
   - AMI: Amazon Linux 2023
   - Instance type: `t2.micro` (Free Tier)
   - IAM Instance Profile: `ec2-ssm-role`
   - Security Group: `ec2-sg`
3. User Data (installs Apache):
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from $(hostname -f)</h1>" > /var/www/html/index.html
```

---

## Step 9 — Create Target Group

1. Go to **EC2 → Target Groups → Create Target Group**
2. Configure:
   - Target type: **Instances**
   - Name: `web-servers-tg`
   - Protocol: HTTP, Port: 80
   - VPC: `project-vpc`
   - Health check path: `/`

---

## Step 10 — Create Application Load Balancer

1. Go to **EC2 → Load Balancers → Create Load Balancer → Application Load Balancer**
2. Configure:
   - Name: `project-alb`
   - Scheme: **Internet-facing**
   - VPC: `project-vpc`
   - Subnets: select **both public subnets** (AZ1 and AZ2)
   - Security Group: `alb-sg`
3. Listener: HTTP:80 → Forward to `web-servers-tg`
4. Click **Create Load Balancer**

---

## Step 11 — Create Auto Scaling Group

1. Go to **EC2 → Auto Scaling Groups → Create Auto Scaling Group**
2. Configure:
   - Name: `web-asg`
   - Launch Template: `web-server-template`
3. Network:
   - VPC: `project-vpc`
   - Subnets: `private-app-subnet-az1`, `private-app-subnet-az2`
4. Load Balancing: Attach to `web-servers-tg`
5. Health checks: Enable ELB health checks
6. Group size:
   - Desired: 2
   - Minimum: 2
   - Maximum: 4
7. Scaling Policy: **Target Tracking**
   - Metric: Average CPU Utilization
   - Target: 70%

---

## Step 12 — Create RDS Subnet Group

1. Go to **RDS → Subnet Groups → Create DB Subnet Group**
2. Configure:
   - Name: `rds-subnet-group`
   - VPC: `project-vpc`
   - Subnets: `private-db-subnet-az1`, `private-db-subnet-az2`

---

## Step 13 — Create RDS Instance (Multi-AZ)

1. Go to **RDS → Databases → Create Database**
2. Configure:
   - Engine: MySQL (or PostgreSQL)
   - Template: **Free Tier** or **Production**
   - Deployment: **Multi-AZ DB Instance** ✅
   - DB Instance ID: `project-db`
   - Master username & password: (save these securely)
   - Instance class: `db.t3.micro`
   - Storage: 20 GB
3. Connectivity:
   - VPC: `project-vpc`
   - Subnet Group: `rds-subnet-group`
   - Public access: **No**
   - Security Group: `rds-sg`
4. Click **Create Database**

---

## Step 14 — Set Up CloudWatch Alarm + SNS

**Create SNS Topic:**
1. Go to **SNS → Topics → Create Topic**
   - Type: Standard
   - Name: `infra-alerts`
2. Create Subscription: Email → enter your email address
3. Confirm the subscription from your inbox

**Create CloudWatch Alarm:**
1. Go to **CloudWatch → Alarms → Create Alarm**
2. Select metric: **EC2 → By Auto Scaling Group → CPUUtilization**
3. Condition: Greater than **80%** for 2 consecutive periods
4. Action: Send notification to `infra-alerts` SNS topic
5. Name: `high-cpu-alarm`

---

## Step 15 — Test the Architecture

1. Copy the ALB DNS name from the Load Balancers console
2. Open it in a browser — you should see the Apache welcome page
3. Refresh multiple times to see traffic routing between instances
4. Check the Auto Scaling Group to verify both instances are healthy

---

## ✅ Architecture Checklist

- [ ] VPC created with correct CIDR
- [ ] 6 subnets across 2 AZs
- [ ] Internet Gateway attached
- [ ] NAT Gateways in both public subnets
- [ ] Route tables configured correctly
- [ ] Security Groups with least-privilege rules
- [ ] ALB deployed in public subnets
- [ ] EC2 instances in private subnets
- [ ] Auto Scaling Group with scaling policy
- [ ] RDS Multi-AZ deployed in private subnets
- [ ] CloudWatch alarm connected to SNS
