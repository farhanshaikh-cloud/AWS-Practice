Here is a more professional and detailed version of your AWS Load Balancer lab that you can use for practice, documentation, or interview preparation.

# AWS Application Load Balancer (ALB) – Traffic Distribution Lab

## Objective

Deploy two Amazon Linux EC2 instances, configure a web server on each instance, create a Target Group and Application Load Balancer, and verify traffic distribution between the instances.

---

# Step 1: Launch Two EC2 Instances

1. Login to AWS Console.
2. Navigate to **EC2 Dashboard**.
3. Click **Launch Instance**.

### Instance Configuration

| Setting               | Value                    |
| --------------------- | ------------------------ |
| Name                  | WebServer-1              |
| AMI                   | Amazon Linux 2023        |
| Instance Type         | t3.micro                 |
| Key Pair              | Select existing key pair |
| Network               | Default VPC              |
| Auto Assign Public IP | Enable                   |

### Security Group

Create a new Security Group with:

| Type  | Port | Source   |
| ----- | ---- | -------- |
| SSH   | 22   | My IP    |
| HTTP  | 80   | Anywhere |
| HTTPS | 443  | Anywhere |

---

# Step 2: Configure User Data

Paste the following script in **Advanced Details → User Data**

```bash
#!/bin/bash

yum update -y
yum install httpd -y

systemctl start httpd
systemctl enable httpd

cd /var/www/html

echo "<h1>Welcome from $(hostname)</h1>" > index.html
```

Launch **2 instances** using the same configuration.

---

# Step 3: Verify Web Server Installation

Wait 2–3 minutes for User Data execution.

Navigate to:

**EC2 → Instances**

Note the Public IPv4 addresses.

Example:

```text
WebServer-1 : http://98.89.32.91
WebServer-2 : http://34.227.227.17
```

Open both URLs in a browser.

Expected output:

```html
Welcome from ip-172-31-x-x
```

Each instance should display a different hostname.

---

# Step 4: Create Target Group

Navigate to:

**EC2 → Target Groups → Create Target Group**

### Configuration

| Setting     | Value       |
| ----------- | ----------- |
| Target Type | Instances   |
| Protocol    | HTTP        |
| Port        | 80          |
| VPC         | Default VPC |

### Health Check

| Setting  | Value       |
| -------- | ----------- |
| Protocol | HTTP        |
| Path     | /index.html |

Click **Next**.

---

# Step 5: Register EC2 Instances

Select:

* WebServer-1
* WebServer-2

Click:

```text
Include as pending below
```

Then click:

```text
Create Target Group
```

---

# Step 6: Verify Target Health

Open:

**Target Groups → Targets**

Wait until both instances show:

```text
Healthy
```

Example:

```text
WebServer-1   Healthy
WebServer-2   Healthy
```

If targets are unhealthy:

* Verify Apache is running.
* Verify Security Group allows HTTP traffic.
* Verify health check path is correct.

---

# Step 7: Create Application Load Balancer

Navigate to:

**EC2 → Load Balancers → Create Load Balancer**

Select:

**Application Load Balancer**

---

# Step 8: Configure Load Balancer

### Basic Configuration

| Setting         | Value           |
| --------------- | --------------- |
| Name            | MyLoadBalancer  |
| Scheme          | Internet-facing |
| IP Address Type | IPv4            |

### Listener

| Protocol | Port |
| -------- | ---- |
| HTTP     | 80   |

---

# Step 9: Select Network Mapping

Choose:

* Default VPC

Select all available Availability Zones.

Example:

```text
us-east-1a
us-east-1b
us-east-1c
```

AWS will create a Load Balancer node in each selected AZ.

---

# Step 10: Configure Load Balancer Security Group

Create or select a Security Group with:

| Type  | Port | Source   |
| ----- | ---- | -------- |
| HTTP  | 80   | Anywhere |
| HTTPS | 443  | Anywhere |

---

# Step 11: Configure Listener Routing

For:

```text
Default Action
```

Select the Target Group created earlier.

Example:

```text
TG-WebServers
```

Click:

```text
Create Load Balancer
```

---

# Step 12: Obtain Load Balancer DNS Name

Navigate to:

**EC2 → Load Balancers**

Select your ALB.

Copy the DNS Name:

```text
http://myloadbalancer-1193861558.us-east-1.elb.amazonaws.com
```

---

# Step 13: Test Traffic Distribution

Open the Load Balancer URL multiple times:

```text
http://myloadbalancer-1193861558.us-east-1.elb.amazonaws.com
```

Refresh repeatedly.

You should see responses from:

```text
Welcome from ip-172-31-x-x
```

and

```text
Welcome from ip-172-31-y-y
```

showing that traffic is being distributed across both instances.

---

# Step 14: Simulate High Availability

1. Go to Target Group.
2. Select one instance.
3. Click:

```text
Deregister
```

OR

```text
Stop one EC2 instance
```

Refresh the Load Balancer URL.

Expected Result:

```text
Application remains accessible.
```

Traffic automatically routes to the healthy instance.

---

# Step 15: Cleanup Resources

To avoid AWS charges:

### Delete Load Balancer

```text
EC2 → Load Balancers → Delete
```

### Delete Target Group

```text
EC2 → Target Groups → Delete
```

### Terminate EC2 Instances

```text
EC2 → Instances → Terminate
```

### Verify Cleanup

Ensure no resources remain:

* Load Balancers
* Target Groups
* EC2 Instances
* Elastic IPs (if any)

---

## Learning Outcomes

After completing this lab, you will understand:

* EC2 provisioning
* User Data automation
* Apache web server deployment
* Target Groups
* Health Checks
* Application Load Balancers (ALB)
* Traffic Distribution
* High Availability
* AWS Resource Cleanup

This is one of the most common AWS DevOps interview hands-on scenarios and is excellent practice for Solutions Architect and DevOps Engineer roles. 🚀
