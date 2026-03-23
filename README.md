# aws-3-tier-highly-available-architecture
Highly available and scalable AWS 3-tier architecture using ALB, Auto Scaling, and RDS Multi-AZ following cloud best practices.

# AWS 3-Tier Highly Available Architecture

## 📌 Project Overview

This project is part of my AWS cloud learning journey and hands-on practice.
It demonstrates a highly available, secure, and scalable **3-tier architecture** built on AWS following best practices.

---

## 🎯 Project Objective

The goal of this project is to:

* Design a highly available architecture using Multi-AZ
* Implement secure networking using public and private subnets
* Apply scalability using Auto Scaling
* Separate infrastructure into three layers (Presentation, Application, Database)
* Follow AWS best practices for cloud architecture

---

## 🏗️ Architecture Overview

The system is designed using a **3-tier architecture**:

### 1️⃣ Presentation Layer

* Application Load Balancer (ALB)
* Deployed across multiple public subnets
* Distributes incoming traffic

---

### 2️⃣ Application Layer

* Amazon EC2 instances
* Auto Scaling Group (ASG)
* Deployed in private subnets

---

### 3️⃣ Database Layer

* Amazon RDS (Multi-AZ)

  * Primary instance (AZ1)
  * Standby instance (AZ2)
* Deployed in private subnets

---

## 🌐 Network Design

* VPC: `10.0.0.0/16`
* 2 Availability Zones
* Public Subnets:

  * ALB
  * NAT Gateway
* Private Subnets:

  * EC2 (Application Layer)
  * RDS (Database Layer)

---

## 🔐 Security

* Only ALB is publicly accessible
* EC2 instances are in private subnets
* RDS is not publicly accessible
* Security Groups control traffic between layers

---

## 🌍 Internet Access

* Internet Gateway attached to VPC
* NAT Gateway in each public subnet
* Private resources access the internet via NAT

---

## ⚖️ Load Balancing

* Application Load Balancer (ALB)
* Listener (HTTP/HTTPS)
* Target Group with health checks

---

## 📈 Scalability

* Auto Scaling Group
* Target Tracking Policy (CPU-based scaling)

---

## 📊 Monitoring

* Amazon CloudWatch (basic monitoring)

---

## 🔄 Traffic Flow

1. Client sends request
2. Request reaches ALB
3. ALB forwards request to Target Group
4. EC2 instances process the request
5. EC2 communicates with RDS
6. Response returns to the client via ALB

---

## 🖼️ Architecture Diagram

![Architecture Diagram](./AWS (2025) horizontal framework.png)

---

## 🎥 Project Demo

Watch the full project demonstration here:
https://

---

## 🧪 Implementation

This project was implemented using:

* AWS Management Console
* Manual configuration of all services

---

## 💡 Key Features

* High Availability (Multi-AZ)
* Secure architecture (private subnets)
* Scalable system (Auto Scaling)
* Fault tolerance
* Cost-aware design

---

## 💼 Skills Demonstrated

* AWS VPC & Networking
* Application Load Balancer (ALB)
* Auto Scaling
* RDS Multi-AZ
* Cloud Security Best Practices

---

## 📌 Author

Kenza Behlouli
