Yes. A complete AWS EFS project is one of the best ways to understand shared storage in AWS.

# Project: Shared Web Content Using Amazon EFS Across Multiple EC2 Instances

## Architecture

![Image](https://images.openai.com/static-rsc-4/SinBr_RRNiVOKnG6R5oExCoAMDqkLfUPCMzCB7Nq4a22Ct_z3mDwq0F-mU2O63iXZQ0BEY6bAAB9WJjT_QYOZnUTh8TaykFcfFcU1_DAxIuRsCADkDRqTfBazLljioMlDDFBviwMA1MWzdYZCOIac-QnhCbjhdJ5HhCprjjMfbCBeGiJDCiW-Vhl5JvZpZZz?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/ZyfzREjDdL7Lru_nj9Of6ScsaW0C7Pw4ImQjadFNu6bUrOSUfdKlqfqcn6rqbC7jVeYA4eZY2J1hSsyb2YkTl7UoNL-XEN5KEL3pC49LSf7fX80jC6leJM1Lr7MgJh4FZF_LAW8ZZbaFdifsDbXghurwugZFd1N64OeNxFkpBmZ8Yu0nlRCHDu4y6ZMGgNox?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/bHuth8R6VmZuCF0lfzP9DlZ3qkW7moYoMJp4rJhycjElr9LLUO9U0Q86oseVO1YT6l_M3xwX_LYbsiJjDwBr1Ah1uqozy6hisIAz-ywn0C1KGq-hERF1lJ5cNh9W7SQjp8-_RAcdDGFSIAqmwoU4GMRCCaRSJ2Yde8WuxDUayPmcTNq58vHPgnlsQUDF5JLr?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/3BSknMTmEsXWbRMMGnaLpaEj18lK7EXgysxU4DD_-6dorCbrbLN9afHmJ1Z_1fKw-fPtE7LC3dnNITx1BlE_fbXOQ9gLfOd2oi9hhLwtbatT9RFa9eyokmmAqE-o7y_OGRnfKp3-AXZGv5kJRnbuzidF3F6P1iySm-IQYld3sTAzhbn1O3XbmPNfxw0_nCo7?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/5A2vkl14We0XEtWGbjgK6lKFHKimnpBd2bq-lV_bwv98z47ZSPCNjZmwKJM1R2V9zXkxKxFRs-R2MB9v75sjxnV1DhU2U_xxmDaT7bo3pE2VPieAXQLNwg7tNNADnfAYnqXWXb_tnbxqmbn-JJTyAoBAyMNlvVN_JILTU8vgD9IewwEA9x6PZrobxAFdQLsx?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/mnfY4zHAtLx-c33b3l5k-5-I3k2qJfxHp3yLpMD6juiTJ3bSYC9vlnxdMoBei-cxOHe4VClvMkPiijMiBDd5-tJ9kb6o1wI3aGOb5OmQTxwWtuDbogzrW7ch3qgHpjnFREjbmgnejUqdhvZRqIOULxkhxshs3H1igyc8pLX3pVh74ZHg92jnR0ww-XTTHLFt?purpose=fullsize)

### Goal

Create a shared file system that can be accessed simultaneously by multiple EC2 instances.

**Services Used**

* Amazon Web Services EFS
* Amazon Web Services EC2
* Security Groups
* VPC
* Mount Targets

### Real-World Use Case

Suppose you have 2 web servers behind a load balancer. If users upload images to one server, those images should be visible on the other server. EFS solves this problem.

---

# Step 1: Create an EFS File System

1. Open AWS Console.
2. Navigate to:
   **Services → EFS**
3. Click **Create file system**.
4. Select your VPC.
5. Keep default settings.
6. Click **Create**.

Note the:

* File System ID (example: fs-123456789)

---

# Step 2: Verify Mount Targets

Go to:

**EFS → Your File System → Network**

You should see mount targets created in your Availability Zones.

Example:

| AZ         | Mount Target |
| ---------- | ------------ |
| us-east-1a | Available    |
| us-east-1b | Available    |

---

# Step 3: Create Security Group for EFS

Create SG named:

`efs-sg`

Add inbound rule:

| Type | Port | Source             |
| ---- | ---- | ------------------ |
| NFS  | 2049 | EC2 Security Group |

NFS uses port 2049.

---

# Step 4: Launch Two EC2 Instances

Create:

* WebServer1
* WebServer2

Use:

* Amazon Linux 2023
* t2.micro or t3.micro
* Same VPC as EFS

Security Group:

Allow:

* SSH (22)
* HTTP (80)

---

# Step 5: Install EFS Utilities

SSH into EC2.

Update packages:

```bash
sudo dnf update -y
```

Install EFS utilities:

```bash
sudo dnf install -y amazon-efs-utils
```

Verify:

```bash
rpm -qa | grep efs
```

---

# Step 6: Mount EFS

Create mount directory:

```bash
sudo mkdir /efs
```

Mount EFS:

```bash
sudo mount -t efs fs-xxxxxxxx:/ /efs
```

Replace:

```bash
fs-xxxxxxxx
```

with your File System ID.

Verify:

```bash
df -h
```

You should see EFS mounted.

---

# Step 7: Make Mount Persistent

Edit:

```bash
sudo vi /etc/fstab
```

Add:

```bash
fs-xxxxxxxx:/ /efs efs defaults,_netdev 0 0
```

Test:

```bash
sudo mount -a
```

No errors means success.

---

# Step 8: Test Shared Storage

On EC2-1:

```bash
echo "Hello from Server 1" > /efs/test.txt
```

Check:

```bash
cat /efs/test.txt
```

---

On EC2-2:

Mount the same EFS.

Run:

```bash
cat /efs/test.txt
```

Expected:

```text
Hello from Server 1
```

This proves both servers are using the same shared storage.

---

# Step 9: Install Apache

On both servers:

```bash
sudo dnf install httpd -y

sudo systemctl enable httpd
sudo systemctl start httpd
```

Mount EFS to web root:

```bash
sudo mount -t efs fs-xxxxxxxx:/ /var/www/html
```

Create webpage:

```bash
echo "AWS EFS Shared Website" | sudo tee /var/www/html/index.html
```

---

# Step 10: Validation

Open:

```text
http://<EC2-1-Public-IP>
```

and

```text
http://<EC2-2-Public-IP>
```

Both should display:

```text
AWS EFS Shared Website
```

Because both instances are reading the same EFS storage.

---

# Bonus Tasks (Interview Level)

### Task 1: Auto Mount EFS During Boot

Use User Data:

```bash
#!/bin/bash
dnf install -y amazon-efs-utils
mkdir /efs
mount -t efs fs-xxxxxxxx:/ /efs
```

### Task 2: Use Auto Scaling Group

* Create Launch Template
* Auto Scaling Group
* Mount EFS automatically
* Verify all instances share files

### Task 3: Backup EFS

Use:

* Amazon Web Services Backup
* EFS Backup Policy

### Task 4: Encrypt EFS

Create EFS with:

* Encryption at Rest
* Encryption in Transit

---

# DevOps Resume Project Description

**Project:** Shared Web Hosting Using Amazon EFS

* Created Amazon EFS and mounted it across multiple EC2 instances.
* Configured NFS security rules and mount targets.
* Implemented persistent storage using `/etc/fstab`.
* Hosted a shared Apache website across multiple servers.
* Validated data consistency and high availability across Availability Zones.
* Automated mounting using User Data scripts.

This project is commonly asked in AWS SysOps, Cloud Support, and DevOps interviews because it demonstrates storage, networking, Linux, and EC2 concepts together.
