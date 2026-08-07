# 🚀 Lab 04 – Host a Database on Amazon EC2

![AWS](https://img.shields.io/badge/AWS-EC2-orange)
![Amazon Linux](https://img.shields.io/badge/Amazon%20Linux-2023-red)
![Database](https://img.shields.io/badge/Database-MariaDB-blue)
![SQL](https://img.shields.io/badge/SQL-Enabled-green)
![Linux](https://img.shields.io/badge/Linux-Shell-black)
![Backup](https://img.shields.io/badge/Backup-Script-success)

---
In this lab, I built a dedicated MariaDB database server on Amazon EC2 by installing and configuring the database engine, securing the server, creating databases and users, and implementing an automated backup solution using shell scripting.

---

# 🏗️ Architecture

```
                    SSH (22)
                        │
                        ▼
           Amazon EC2 (Database Server)
             Amazon Linux 2023
                        │
                MariaDB 11.4 Server
                        │
        ┌───────────────┴───────────────┐
        │                               │
        ▼                               ▼
   companydb Database             dbadmin User
        │
        ▼
   employees Table
        │
        ▼
   Sample Employee Records
        │
        ▼
      Backup Script
        │
        ▼
   SQL Backup (.sql)
```

---

---

# 🎯 Objectives

- Launch a dedicated Amazon EC2 instance
- Configure server storage
- Install MariaDB Database Engine
- Configure MariaDB service
- Secure the database installation
- Create a sample database
- Create a dedicated database user
- Insert sample data
- Configure a database backup script
- Validate the backup process

---

# 🛠️ Prerequisites

- AWS Account
- Amazon EC2
- Amazon Linux 2023
- SSH Client
- Basic Linux Knowledge
- Basic SQL Knowledge

---

# 🖥️ Environment

| Resource | Value |
|----------|-------|
| Cloud Provider | AWS |
| Service | Amazon EC2 |
| Operating System | Amazon Linux 2023 |
| Database Engine | MariaDB 11.4 |
| Instance Type | t3.micro |
| Storage | 8 GB gp3 |
| Backup Directory | /opt/db-backups |

---

# 📷 Infrastructure

![EC2 Instance](screenshots/01-ec2-instance.png)

A dedicated EC2 instance named **Database-server** was launched to host the database independently from previous labs.

---

# ⚙️ Implementation

## Step 1 – Launch Amazon EC2

Created a new EC2 instance with:

- Amazon Linux 2023
- t3.micro
- Default VPC
- 8 GB gp3 Storage
- SSH Security Group

---

## Step 2 – Verify Operating System

Verified the operating system and storage configuration.

```bash
hostname

cat /etc/os-release

df -h

lsblk
```

Verified:

- Amazon Linux 2023
- Root Volume
- Mounted Filesystem

---

## Step 3 – Install MariaDB

Updated the server packages.

```bash
sudo dnf update -y
```

Installed MariaDB 11.4.

```bash
sudo dnf install mariadb114-server -y
```

Verified installation.

```bash
mariadb --version
```

---

## Step 4 – Configure MariaDB Service

Started MariaDB.

```bash
sudo systemctl start mariadb
```

Enabled MariaDB to start automatically.

```bash
sudo systemctl enable mariadb
```

Verified service status.

```bash
sudo systemctl status mariadb
```

Verified database listener.

```bash
sudo ss -tulpn | grep 3306
```

---

# 📷 MariaDB Service

![MariaDB](screenshots/02-mariadb-service.png)

MariaDB service was successfully started and configured to launch automatically during system boot.

---

## Step 5 – Secure MariaDB

Executed:

```bash
sudo mariadb-secure-installation
```

Completed the following:

- Root Password Configured
- Anonymous Users Removed
- Remote Root Login Disabled
- Test Database Removed
- Privilege Tables Reloaded

---

## Step 6 – Create Database

Connected to MariaDB.

```bash
sudo mariadb -u root -p
```

Created database.

```sql
CREATE DATABASE companydb;
```

Verified.

```sql
SHOW DATABASES;
```

---

## Step 7 – Create Database User

Created a dedicated database user.

```sql
CREATE USER 'dbadmin'@'localhost' IDENTIFIED BY '********';
```

Granted privileges.

```sql
GRANT ALL PRIVILEGES ON companydb.* TO 'dbadmin'@'localhost';
```

Reloaded privilege tables.

```sql
FLUSH PRIVILEGES;
```

Verified user creation.

```sql
SELECT User, Host FROM mysql.user;
```

---

# 📷 Database Configuration

![Database](screenshots/03-database-configuration.png)

Successfully created:

- companydb
- dbadmin User
- Database Privileges

---

## Step 8 – Create Table

Connected using the newly created database user.

```bash
mariadb -u dbadmin -p
```

Selected database.

```sql
USE companydb;
```

Created table.

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    department VARCHAR(100)
);
```

---

## Step 9 – Insert Sample Records

Inserted sample employee records.

```sql
INSERT INTO employees (name, department)
VALUES
('Rahul','Cloud'),
('John','DevOps');
```

Verified data.

```sql
SELECT * FROM employees;
```

---

## Step 10 – Configure Backup Script

Created backup directory.

```bash
sudo mkdir -p /opt/db-backups
```

Created backup script.

```bash
sudo nano /opt/db-backups/backup.sh
```

Updated permissions.

```bash
sudo chmod +x /opt/db-backups/backup.sh
```

Executed backup.

```bash
sudo /opt/db-backups/backup.sh
```

Verified backup.

```bash
ls -lh /opt/db-backups
```

Validated backup contents.

```bash
head -20 /opt/db-backups/companydb-*.sql
```

---

# 📷 Backup Validation

![Backup](screenshots/04-backup-validation.png)

Successfully generated a SQL backup containing:

- Database Structure
- Table Definition
- Data Records

---

---

# ✅ Verification

The following components were successfully validated:

- EC2 Instance Running
- MariaDB Service Active
- Port 3306 Listening
- Database Created
- Database User Created
- Table Created
- Sample Records Inserted
- Backup Script Executed
- SQL Backup Generated

---

# 🔍 Troubleshooting

| Issue | Resolution |
|--------|------------|
| Port 3306 not accessible | Verified MariaDB service and Security Group configuration |
| Unable to create duplicate user | Verified the user already existed before creation |
| mysqldump deprecation warning | Updated the script to use `mariadb-dump` |

---

# 📚 Key Learnings

- Deploying a dedicated database server on Amazon EC2
- Installing and managing MariaDB using `systemd`
- Securing a database installation
- Creating databases and users
- Managing database privileges
- Performing SQL operations
- Creating logical database backups using `mariadb-dump`
- Validating backup files

---

# 🎯 Conclusion

Successfully deployed and configured a dedicated MariaDB database server on Amazon EC2 using Amazon Linux 2023.

The implementation included database installation, service configuration, security hardening, database administration, and backup automation.

This lab demonstrates fundamental database administration skills on AWS and serves as the foundation for future labs involving storage expansion, workload protection, and disaster recovery.
