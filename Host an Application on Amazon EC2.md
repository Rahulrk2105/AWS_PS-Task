# 🚀 Lab 01 – Host an Application on Amazon EC2

![AWS](https://img.shields.io/badge/AWS-EC2-orange)
![Amazon Linux](https://img.shields.io/badge/OS-Amazon%20Linux%202023-green)
![Apache](https://img.shields.io/badge/Web%20Server-Apache-red)
![Status](https://img.shields.io/badge/Status-Completed-success)

---

# Table of Contents

- Overview
- Objective
- Solution Architecture
- Deployment Workflow
- AWS Resources Used
- Lab Environment
- Prerequisites
- Implementation
  - Create VPC
  - Create Public Subnet
  - Create Internet Gateway
  - Create Route Table
- Next Steps

---

# Overview

This lab demonstrates the complete deployment of a static web application on an Amazon EC2 instance using a custom Amazon VPC. The infrastructure was manually configured to understand AWS networking fundamentals, EC2 provisioning, and automated application deployment using EC2 User Data.

The web application is hosted using the Apache HTTP Server on Amazon Linux 2023 and is accessible through the instance's public IPv4 address.

---

# Objective

The objective of this lab is to:

- Build a custom VPC
- Configure public networking
- Launch an EC2 instance
- Deploy Apache HTTP Server
- Automatically deploy a static HTML application using EC2 User Data
- Validate application accessibility using the EC2 public IP

---

# Solution Architecture

```text
                                Internet
                                    │
                                    ▼
                           Internet Gateway
                              (app-Igw)
                                    │
                                    ▼
                          Public Route Table
                              (public-rt)
                                    │
                                    ▼
                            Public Subnet
                              10.0.0.0/24
                                    │
                                    ▼
                           Security Group
                 ┌────────────────────────────────┐
                 │ SSH (22)  → My Public IP       │
                 │ HTTP (80) → 0.0.0.0/0          │
                 └────────────────────────────────┘
                                    │
                                    ▼
                      Amazon EC2 Instance (App-Ec2)
                                    │
                          Amazon Linux 2023
                                    │
                          EC2 User Data Script
                                    │
                                    ▼
                        Apache HTTP Server (httpd)
                                    │
                                    ▼
                         Static HTML Application
                                    │
                                    ▼
                           Public IPv4 Address
                                    │
                                    ▼
                                End Users
```

---

# Deployment Workflow

```text
Start
   │
   ▼
Login to AWS Console
   │
   ▼
Select AWS Region (us-east-1)
   │
   ▼
Create Custom VPC (App-VPC)
   │
   ▼
Create Public Subnet
   │
   ▼
Create Internet Gateway
   │
   ▼
Attach Internet Gateway to VPC
   │
   ▼
Create Public Route Table
   │
   ▼
Add Route
0.0.0.0/0 → Internet Gateway
   │
   ▼
Associate Route Table
with Public Subnet
   │
   ▼
Create Security Group
   │
   ├── SSH (22) → My Public IP
   └── HTTP (80) → Internet
   │
   ▼
Launch EC2 Instance
   │
   ▼
Select Amazon Linux 2023
   │
   ▼
Select Instance Type (t3.micro)
   │
   ▼
Configure gp3 Storage
   │
   ▼
No IAM Role Attached
   │
   ▼
Paste EC2 User Data
   │
   ▼
Launch Instance
   │
   ▼
Apache Installed Automatically
   │
   ▼
Static HTML Application Deployed
   │
   ▼
Access Application using Public IPv4
   │
   ▼
Deployment Completed
```

---

# AWS Resources Used

| Resource | Configuration |
|----------|---------------|
| AWS Region | us-east-1 (N. Virginia) |
| VPC | App-VPC |
| VPC CIDR | 10.0.0.0/16 |
| Public Subnet | Public-Subnet |
| Subnet CIDR | 10.0.0.0/24 |
| Internet Gateway | app-Igw |
| Route Table | public-rt |
| EC2 Instance | App-Ec2 |
| Operating System | Amazon Linux 2023 |
| AMI Architecture | x86_64 |
| Instance Type | t3.micro |
| Root Volume | gp3 |
| Key Pair | App-key.pem |
| IAM Role | None |
| Web Server | Apache HTTP Server |
| Deployment Method | EC2 User Data |

---

# Lab Environment

| Component | Value |
|-----------|-------|
| Platform | AWS Management Console |
| Operating System | Amazon Linux 2023 |
| Package Manager | DNF |
| Web Server | Apache HTTP Server |
| Application Type | Static HTML |
| Connectivity | Public Internet |

---

# Prerequisites

Before starting this lab, ensure the following prerequisites are available:

- AWS Account
- AWS Management Console Access
- EC2 Key Pair (.pem)
- Basic Linux knowledge
- Internet connection

---

# Implementation

## Step 1 – Create a Custom VPC

### AWS Console Navigation

```
AWS Console
    ↓
VPC
    ↓
Your VPCs
    ↓
Create VPC
```

### Configuration

| Setting | Value |
|---------|-------|
| Name | App-VPC |
| IPv4 CIDR | 10.0.0.0/16 |
| Tenancy | Default |

### Procedure

1. Open the AWS Management Console.
2. Navigate to **VPC**.
3. Select **Create VPC**.
4. Choose **VPC Only**.
5. Enter **App-VPC** as the name.
6. Configure the IPv4 CIDR block as **10.0.0.0/16**.
7. Leave the remaining settings as default.
8. Click **Create VPC**.

### Validation

- VPC Status = Available
- CIDR = 10.0.0.0/16

---

## Step 2 – Create a Public Subnet

### AWS Console Navigation

```
VPC
 ↓
Subnets
 ↓
Create Subnet
```

### Configuration

| Setting | Value |
|---------|-------|
| Name | Public-Subnet |
| VPC | App-VPC |
| CIDR | 10.0.0.0/24 |
| Auto Assign Public IPv4 | Enabled |

### Procedure

1. Open **Subnets**.
2. Select **Create Subnet**.
3. Select **App-VPC**.
4. Enter **Public-Subnet**.
5. Configure **10.0.0.0/24**.
6. Enable **Auto Assign Public IPv4**.
7. Click **Create Subnet**.

### Validation

- Subnet Status = Available
- Public IPv4 Assignment = Enabled

---

## Step 3 – Create an Internet Gateway

### AWS Console Navigation

```
VPC
 ↓
Internet Gateways
 ↓
Create Internet Gateway
```

### Configuration

| Setting | Value |
|---------|-------|
| Name | app-Igw |

### Procedure

1. Create a new Internet Gateway.
2. Name it **app-Igw**.
3. Click **Create**.
4. Select the gateway.
5. Click **Attach to VPC**.
6. Choose **App-VPC**.
7. Click **Attach**.

### Validation

- State = Attached

---

## Step 4 – Create a Public Route Table

### AWS Console Navigation

```
VPC
 ↓
Route Tables
 ↓
Create Route Table
```

### Configuration

| Setting | Value |
|---------|-------|
| Name | public-rt |
| VPC | App-VPC |

### Procedure

1. Create a new Route Table.
2. Name it **public-rt**.
3. Associate it with **App-VPC**.
4. Open **Routes**.
5. Add the following route:

| Destination | Target |
|-------------|--------|
| 0.0.0.0/0 | app-Igw |

6. Save the route.
7. Open **Subnet Associations**.
8. Associate **Public-Subnet**.

### Validation

- Default Route Configured
- Internet Gateway Attached
- Public Subnet Associated

---


# Step 5 – Create a Security Group

### AWS Console Navigation

```text
AWS Console
    ↓
EC2
    ↓
Security Groups
    ↓
Create Security Group
```

## Configuration

| Setting | Value |
|---------|-------|
| Security Group Name | app-sg |
| Description | Security Group for EC2 Application |
| VPC | App-VPC |

### Inbound Rules

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| SSH | TCP | 22 | My IP |
| HTTP | TCP | 80 | 0.0.0.0/0 |

### Outbound Rules

Leave the default outbound rule.

| Type | Destination |
|------|-------------|
| All Traffic | 0.0.0.0/0 |

### Validation

- Security Group created successfully
- SSH (22) allowed only from My IP
- HTTP (80) allowed from the Internet

---

# Step 6 – Create a Key Pair

### AWS Console Navigation

```text
AWS Console
    ↓
EC2
    ↓
Key Pairs
    ↓
Create Key Pair
```

## Configuration

| Setting | Value |
|---------|-------|
| Key Pair Name | App-key |
| Type | RSA |
| Format | .pem |

### Procedure

1. Open **Key Pairs**
2. Click **Create Key Pair**
3. Enter **App-key**
4. Select **RSA**
5. Select **.pem**
6. Click **Create**
7. Store the downloaded file securely.

---

# Step 7 – Launch EC2 Instance

### AWS Console Navigation

```text
AWS Console
    ↓
EC2
    ↓
Instances
    ↓
Launch Instance
```

---

## Instance Details

| Property | Value |
|----------|-------|
| Name | App-Ec2 |
| Region | us-east-1 |
| AMI | Amazon Linux 2023 |
| Architecture | x86_64 |
| Instance Type | t3.micro |

---

## Configure AMI

Select

```
Amazon Linux 2023
```

---

## Configure Instance Type

Select

```
t3.micro
```

---

## Configure Key Pair

Select

```
App-key
```

---

## Configure Networking

| Property | Value |
|----------|-------|
| VPC | App-VPC |
| Subnet | Public-Subnet |
| Auto Assign Public IP | Enabled |
| Firewall | Select Existing Security Group |
| Security Group | app-sg |

---

## Configure Storage

| Property | Value |
|----------|-------|
| Volume Type | gp3 |
| Root Volume | Default Size |
| Delete on Termination | Enabled |

---

## Configure IAM Role

```
No IAM Role Attached
```

---

# Step 8 – Configure EC2 User Data

Expand

```
Advanced Details
```

Locate

```
User Data
```

Paste the following script.

```bash
#!/bin/bash

dnf update -y

dnf install -y httpd

systemctl enable httpd
systemctl start httpd

cat <<EOF > /var/www/html/index.html
<!DOCTYPE html>
<html>
<head>
<title>AWS EC2 Application</title>
<style>
body{
font-family:Arial;
background:#f4f7f9;
text-align:center;
margin-top:100px;
}

.container{
background:white;
width:60%;
margin:auto;
padding:30px;
border-radius:10px;
box-shadow:0px 0px 10px gray;
}

h1{
color:#ff9900;
}

footer{
margin-top:20px;
color:gray;
}
</style>
</head>

<body>

<div class="container">

<h1>Application Hosted Successfully!</h1>

<h2>Amazon EC2 - Amazon Linux 2023</h2>

<p>
This application was deployed automatically using
<b>EC2 User Data</b>.
</p>

<p>
Apache Web Server is running successfully.
</p>

<footer>

AWS Professional Services Lab - Task 1

</footer>

</div>

</body>
</html>
EOF
```

---

# Step 9 – Review Configuration

Verify the following before launching.

| Component | Value |
|-----------|-------|
| Region | us-east-1 |
| VPC | App-VPC |
| Public Subnet | Public-Subnet |
| Internet Gateway | Attached |
| Route Table | public-rt |
| Security Group | app-sg |
| EC2 Name | App-Ec2 |
| Operating System | Amazon Linux 2023 |
| Instance Type | t3.micro |
| Storage | gp3 |
| IAM Role | None |
| Key Pair | App-key |
| User Data | Configured |

---

# Step 10 – Launch Instance

Click

```
Launch Instance
```

Wait until the instance transitions through the following states.

```text
Pending

↓

Initializing

↓

Running

↓

2/2 Status Checks Passed
```

---

# Expected Outcome

The EC2 instance is successfully created with the following configuration.

| Property | Status |
|----------|--------|
| Instance State | Running |
| Public IPv4 | Assigned |
| Public DNS | Available |
| Apache Installed | Yes |
| Apache Started | Yes |
| HTML Application | Deployed Automatically |

---

---

# Step 11 – Verify EC2 Instance

### AWS Console Navigation

```text
AWS Console
    ↓
EC2
    ↓
Instances
```

Select the **App-Ec2** instance and verify the following details.

## Instance Details

| Property | Expected Value |
|-----------|----------------|
| Instance State | Running |
| Status Checks | 2/2 Passed |
| Public IPv4 Address | Assigned |
| Public DNS | Available |
| Platform | Amazon Linux 2023 |
| Instance Type | t3.micro |

### Validation

- EC2 instance is in the **Running** state.
- All status checks have passed.
- Public IPv4 address is assigned.
- Public DNS name is available.

---

# Step 12 – Connect to the EC2 Instance

### Prerequisites

- Downloaded Key Pair (`App-key.pem`)
- Public IPv4 Address of the EC2 instance

### Open Terminal

Navigate to the directory containing the key pair.

Example:

```bash
cd ~/Downloads
```

### Connect Using SSH

```bash
ssh -i "App-key.pem" ec2-user@<Public-IP>
```

Example

```bash
ssh -i "App-key.pem" ec2-user@ec2-xx-xx-xx-xx.compute-1.amazonaws.com
```

### Expected Output

```text
Last login: Tue Aug ...

[ec2-user@ip-10-0-0-154 ~]$
```

The prompt confirms a successful SSH connection.

---

# Step 13 – Verify the Operating System

Run the following command.

```bash
cat /etc/os-release
```

Expected Output

```text
Amazon Linux 2023
```

This confirms the EC2 instance is running the selected operating system.

---

# Step 14 – Verify Apache Installation

Check the Apache service status.

```bash
sudo systemctl status httpd
```

Expected Output

```text
Active: active (running)
```

Verify Apache starts automatically.

```bash
sudo systemctl is-enabled httpd
```

Expected Output

```text
enabled
```

---

# Step 15 – Verify Web Application

Retrieve the application directly from the local web server.

```bash
curl http://localhost
```

Expected Result

The HTML page created through **EC2 User Data** is displayed.

---

# Step 16 – Test the Application

Open a web browser.

Navigate to

```text
http://<Public-IP>
```

Example

```text
http://54.xxx.xxx.xxx
```

Expected Result

The browser displays the following page.

```text
Application Hosted Successfully!

Amazon EC2 - Amazon Linux 2023

This application was deployed automatically using EC2 User Data.

Apache Web Server is running successfully.
```

This confirms that:

- Apache is serving web content.
- HTTP traffic is allowed through the Security Group.
- The User Data script executed successfully.

---

# Deployment Validation

| Validation Item | Status |
|-----------------|--------|
| VPC Created | ✅ |
| Public Subnet Created | ✅ |
| Internet Gateway Attached | ✅ |
| Route Table Configured | ✅ |
| Public Route Configured | ✅ |
| Security Group Created | ✅ |
| SSH Access Verified | ✅ |
| HTTP Access Verified | ✅ |
| EC2 Running | ✅ |
| Amazon Linux 2023 Installed | ✅ |
| Apache Installed | ✅ |
| Apache Running | ✅ |
| Static Application Deployed | ✅ |
| Public IPv4 Assigned | ✅ |
| Browser Access Verified | ✅ |

---

# Resource Summary

| Resource | Name |
|----------|------|
| Region | us-east-1 |
| VPC | App-VPC |
| Public Subnet | Public-Subnet |
| Internet Gateway | app-Igw |
| Route Table | public-rt |
| EC2 Instance | App-Ec2 |
| Operating System | Amazon Linux 2023 |
| Instance Type | t3.micro |
| Root Volume | gp3 |
| Key Pair | App-key |
| Security Group | app-sg |
| IAM Role | None |
| Web Server | Apache HTTP Server |


---

# Cleanup

To avoid unnecessary AWS charges, delete the resources after completing the lab.

Delete the resources in the following order.

1. EC2 Instance
2. Security Group
3. Route Table
4. Internet Gateway
5. Public Subnet
6. VPC

---

# Learning Outcomes


- Create a custom Amazon VPC.
- Configure a public subnet.
- Attach an Internet Gateway.
- Configure a Route Table.
- Configure a Security Group.
- Launch an Amazon EC2 instance.
- Select an Amazon Machine Image (AMI).
- Configure an EC2 instance type.
- Configure an EBS root volume.
- Deploy an application using EC2 User Data.
- Install and configure the Apache HTTP Server.
- Connect to an EC2 instance using SSH.
- Verify Linux services.
- Host a static web application on Amazon EC2.

---

