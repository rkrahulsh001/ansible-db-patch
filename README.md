# Database Patch Management Automation using Ansible & Jenkins

Production-ready database patch management framework for **MySQL**, **PostgreSQL**, and **MongoDB** using **Ansible** and **Jenkins**.

The framework automates the complete patch lifecycle including **Pre-check**, **Backup**, **Upgrade/Downgrade**, **Validation**, **Rollback**, and **Execution Summary** across Development, Staging, and Production environments.

---

# Features

- Supports MySQL, PostgreSQL, and MongoDB
- Single Ansible playbook for all supported databases
- Jenkins parameterized pipeline
- Supports both Upgrade and Downgrade
- Automatic database backup before patching
- Backup verification before proceeding
- Health check validation after patching
- Running version verification
- Package version verification
- Automatic rollback on failure
- Detailed execution summary
- Environment independent (Dev / Staging / Production)

---

# Architecture

```
                    Jenkins Job

      Parameters:
      • ANY_HOSTS
      • DB_TYPE
      • VERSION

                    │
                    ▼

        Ansible Playbook (db-patch.yml)

                    │
                    ▼

┌─────────────────────────────────────────────┐
│                 PRE-CHECK                   │
├─────────────────────────────────────────────┤
│ • Service Status                            │
│ • Disk Space                                │
│ • Connectivity Check                        │
│ • Active Connections                        │
│ • Package Version                           │
│ • Running Version                           │
└─────────────────────────────────────────────┘
                    │
                    ▼

┌─────────────────────────────────────────────┐
│                  BACKUP                     │
├─────────────────────────────────────────────┤
│ • mysqldump                                 │
│ • pg_dumpall                                │
│ • mongodump                                 │
│ • Backup Verification                       │
└─────────────────────────────────────────────┘
                    │
                    ▼

┌─────────────────────────────────────────────┐
│             PATCH EXECUTION                 │
├─────────────────────────────────────────────┤
│ • Upgrade (latest)                          │
│ • Downgrade (specific version)              │
│ • Package Installation                      │
└─────────────────────────────────────────────┘
                    │
                    ▼

┌─────────────────────────────────────────────┐
│                 VALIDATION                  │
├─────────────────────────────────────────────┤
│ • Restart Service                           │
│ • Database Health Check                     │
│ • Running Version Verification              │
└─────────────────────────────────────────────┘
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
    Validation OK      Validation Failed
          │                   │
          │                   ▼
          │            Restore Backup
          │                   │
          │            Restart Service
          │                   │
          │            Health Check
          │                   │
          └─────────┬─────────┘
                    ▼

┌─────────────────────────────────────────────┐
│             EXECUTION SUMMARY               │
├─────────────────────────────────────────────┤
│ • Host                                      │
│ • Database                                  │
│ • Requested Version                         │
│ • Before Package Version                    │
│ • Before Running Version                    │
│ • After Package Version                     │
│ • After Running Version                     │
│ • Backup Path                               │
│ • Backup Size                               │
│ • Patch Status                              │
│ • Rollback Status                           │
└─────────────────────────────────────────────┘
```

---

# Workflow

```
Pre-check

↓

Backup

↓

Patch (Upgrade / Downgrade)

↓

Restart Service

↓

Health Validation

↓

Rollback (If Required)

↓

Execution Summary
```

---

# Supported Databases

| Database | Status |
|-----------|--------|
| MySQL | ✅ Supported |
| PostgreSQL | ✅ Supported |
| MongoDB | ✅ Supported |
| Microsoft SQL Server | 🚧 Planned |
| Oracle Database | 🚧 Planned |

---

# Repository Structure

```
ansible-db-patch/

├── db-patch.yml
├── Jenkinsfile
├── README.md
└── inventory/
    └── hosts
```

---

# Inventory

```
[dev_db]
172.17.0.7

[staging_db]
172.17.0.9

[prod_db]
172.17.0.11

[all:vars]

ansible_user=ansible
ansible_ssh_private_key_file=/var/lib/jenkins/.ssh/ansible_key
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

---

# Jenkins Parameters

| Parameter | Description | Example |
|------------|-------------|---------|
| ANY_HOSTS | Target Inventory Group | dev_db |
| DB_TYPE | Database Type | mysql |
| VERSION | Version to install | latest / 8.0.46-0ubuntu0.22.04.3 |

---

# Example Jenkins Execution

```
ANY_HOSTS = dev_db

DB_TYPE = mysql

VERSION = 8.0.46-0ubuntu0.22.04.3
```

### What happens?

1. Jenkins starts the build.
2. Ansible connects to the `dev_db` inventory group.
3. Performs all pre-check validations.
4. Takes a complete database backup.
5. Installs the requested package version.
6. Restarts the database service.
7. Executes health validation.
8. Performs rollback automatically if validation fails.
9. Generates an execution summary.

---

# Database Operations

## MySQL

### Backup

```
mysqldump --all-databases
```

### Health Check

```
SELECT 1;
```

### Running Version

```
SELECT VERSION();
```

### Restart

```
service mysql restart
```

---

## PostgreSQL

### Backup

```
pg_dumpall
```

### Health Check

```
SELECT 1;
```

### Running Version

```
SELECT version();
```

### Restart

```
service postgresql restart
```

---

## MongoDB

### Backup

```
mongodump
```

### Health Check

```
db.runCommand({ ping:1 })
```

### Running Version

```
db.version()
```

### Restart

```
mongo_restart.sh
```

---

# Version Management

## Upgrade

Upgrade to the latest available package.

```
VERSION=latest
```

Example:

```
mysql-server → latest
```

---

## Upgrade to Specific Version

```
VERSION=8.0.46-0ubuntu0.22.04.3
```

---

## Downgrade

Install a previous package version.

```
VERSION=8.0.28-0ubuntu4
```

The playbook uses:

```
allow_downgrade: yes
```

---

# Playbook Execution Flow

```
Gather Facts

↓

Pre-check

↓

Backup

↓

Package Installation

↓

Restart Service

↓

Validation

↓

Success

↓

Execution Summary
```

---

# Pre-check Validation

The playbook validates:

- Service availability
- Disk space
- Database connectivity
- Active database connections
- Installed package version
- Running database version

---

# Backup Validation

Before patching:

- Backup is created.
- Backup file existence is verified.
- Backup size is validated.
- Patching stops automatically if backup validation fails.

---

# Health Validation

After patching:

- Restart database service
- Verify database connectivity
- Verify running version
- Confirm service health

---

# Automatic Rollback

Rollback is automatically triggered if:

- Package installation fails
- Database restart fails
- Health validation fails

Rollback process:

```
Restore Backup

↓

Restart Database

↓

Health Validation

↓

Execution Summary
```

---

# Example Execution Summary

```
==================================================

Host              : 172.17.0.7

Database Type     : mysql

Requested Version : 8.0.46-0ubuntu0.22.04.3

Before Package    : 8.0.28

Before Running    : 8.0.28

After Package     : 8.0.46

After Running     : 8.0.46

Backup Path       : /tmp/db_backup_mysql.dump

Backup Size       : 1257 KB

Patch Status      : SUCCESS

Rollback          : NOT-NEEDED

==================================================
```

---

# Safety Features

- Automatic database backup
- Backup verification
- Package version verification
- Running version verification
- Health check validation
- Automatic rollback
- Restart validation
- Connection validation
- Execution summary
- Single playbook for multiple databases
- Environment-specific execution

---

# Benefits

- Single playbook for all supported databases
- Production-ready patch workflow
- Supports Upgrade and Downgrade
- Parameterized Jenkins pipeline
- Consistent execution across Dev, Staging, and Production
- Easy to extend for additional databases
- Reusable automation framework
- Reduced manual effort
- Improved operational reliability

---

# Future Enhancements

- SQL Server Support
- Oracle Database Support
- CVE-based Security Patching
- Grafana Patch Compliance Dashboard
- Slack Notifications
- Email Notifications
- Maintenance Window Scheduling
- AWS Systems Manager Integration
- Terraform Integration
- Centralized Backup Storage (S3/NFS)
- Change Management Integration
- Patch Compliance Reports

---

# Author

**Rahul Sharma**


