# 💡 Design Decisions

This document explains the architectural choices made in this project and the reasoning behind each decision.

---

## 1. Why 2 Availability Zones?

Deploying across 2 AZs eliminates single points of failure. If one AZ experiences an outage, the other continues serving traffic without interruption. This is the foundation of high availability in AWS.

---

## 2. Why one ALB instead of one per AZ?

An Application Load Balancer is a **Regional service** — a single ALB automatically places nodes in each selected AZ. There is no need to create one per AZ. Using one ALB simplifies management and reduces cost while still achieving cross-AZ load distribution.

---

## 3. Why are EC2 instances in private subnets?

Placing EC2 instances in private subnets means they are not directly reachable from the internet. All inbound traffic must pass through the ALB, which acts as the single controlled entry point. This follows the **principle of least privilege** and reduces the attack surface.

---

## 4. Why NAT Gateway per AZ instead of one shared?

Using a single NAT Gateway creates a cross-AZ dependency. If the NAT Gateway's AZ goes down, instances in the other AZ lose outbound internet access. Having one NAT Gateway per AZ ensures **full AZ independence**.

---

## 5. Why RDS Multi-AZ instead of a single instance?

RDS Multi-AZ provides:
- **Synchronous replication** to a standby instance in another AZ
- **Automatic failover** (typically under 2 minutes) if the primary fails
- No manual intervention required during a database failure

This is essential for production-grade availability.

---

## 6. Why is RDS not publicly accessible?

The database contains sensitive application data. Exposing it to the internet increases the risk of unauthorized access. Only EC2 instances within the VPC can connect to RDS, enforced by Security Group rules.

---

## 7. Why use Security Groups instead of NACLs only?

Security Groups are stateful and operate at the instance level, making them easier to manage for application-tier access control. They allow fine-grained rules based on source Security Group (e.g., allow traffic from ALB-SG only), which NACLs cannot do.

---

## 8. Why CloudWatch + SNS instead of just CloudWatch?

CloudWatch collects metrics and triggers alarms, but on its own it does not notify anyone. SNS adds a **notification layer** — it can send emails or SMS messages when an alarm fires, enabling the team to respond quickly to issues.

---

## 9. Why IAM Roles instead of access keys on EC2?

Hardcoding AWS credentials on EC2 instances is a serious security risk. IAM Roles provide **temporary, automatically rotated credentials** that are attached to the instance. This follows AWS security best practices and the Well-Architected Framework.

---

## 10. Why t2.micro for EC2 and db.t3.micro for RDS?

This project is a learning and demonstration exercise. These instance types fall within the **AWS Free Tier**, minimizing cost while still demonstrating the architecture concepts effectively.
