Absolutely — we can do the **AWS RDS MySQL real-time practice project using PowerShell** instead of Linux shell.

I’ll convert the **RDS MySQL + EC2 + sample DB project** into a **PowerShell-based hands-on lab**. I’ll assume you want to use **a Windows EC2 instance** and run commands in **PowerShell** on that machine.

---

# Project: AWS RDS MySQL Practice Using PowerShell

## Architecture

* **AWS RDS MySQL** → managed database
* **Windows EC2** → client machine where you’ll use PowerShell
* **MySQL client** → installed on EC2 to connect to RDS

---

# Part 1 — Create the RDS MySQL instance

## Step 1: Create RDS in AWS Console

In **AWS Console → RDS → Create database**

Use these settings:

* **Engine type:** MySQL
* **Template:** Free tier / Dev-Test
* **DB instance identifier:** `employee-db`
* **Master username:** `admin`
* **Master password:** set your password
* **DB instance class:** `db.t3.micro` or free-tier eligible option
* **Storage:** 20 GB
* **Connectivity:**

  * Choose the same **VPC** where your EC2 will be created
  * **Public access:** No
* **Initial database name:** optional, can leave blank

Create the DB and wait until status becomes **Available**.

---

# Part 2 — Launch a Windows EC2 instance

## Step 2: Create Windows EC2

Launch a **Windows Server EC2 instance** in the **same VPC** as the RDS instance.

Make sure:

* it has a security group allowing **RDP (3389)** from your IP
* you can log in with Remote Desktop

Once it’s running:

* connect to the Windows EC2 using **RDP**
* open **PowerShell as Administrator**

---

# Part 3 — Configure Security Groups

## Step 3: Allow EC2 to connect to RDS on port 3306

Go to the **RDS security group** and add an inbound rule:

* **Type:** MySQL/Aurora
* **Port:** `3306`
* **Source:** EC2 security group

That’s the most important network step.

---

# Part 4 — Install MySQL client on Windows EC2 using PowerShell

You need a MySQL client so PowerShell can connect to RDS.

## Option A: Install MySQL using Chocolatey

First check if Chocolatey exists:

```powershell
choco --version
```

If Chocolatey is not installed, install it:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

Now install MySQL client:

```powershell
choco install mysql -y
```

After installation, close and reopen PowerShell, then test:

```powershell
mysql --version
```

---

# Part 5 — Connect to RDS from PowerShell

## Step 4: Get the RDS endpoint

From **AWS Console → RDS → your DB → Connectivity & security**, copy:

* **Endpoint**
* **Port** (normally `3306`)

Example endpoint:

```text
employee-db.abc123xyz.ap-south-1.rds.amazonaws.com
```

---

## Step 5: Test network connectivity from PowerShell

Before logging in to MySQL, test whether port 3306 is reachable:

```powershell
Test-NetConnection -ComputerName "employee-db.abc123xyz.ap-south-1.rds.amazonaws.com" -Port 3306
```

### Expected result

You want to see:

```powershell
TcpTestSucceeded : True
```

If it shows `False`, the problem is usually:

* wrong RDS security group rule
* EC2 and RDS not in the same VPC / route issue
* wrong endpoint

---

## Step 6: Connect to MySQL from PowerShell

Use this command:

```powershell
mysql -h employee-db.abc123xyz.ap-south-1.rds.amazonaws.com -P 3306 -u admin -p
```

It will ask for the password.

If login works, you’ll get the MySQL prompt.

---

# Part 6 — Create database and table

Once connected to MySQL, run the following SQL.

```sql
CREATE DATABASE employee_db;
USE employee_db;

CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    role_name VARCHAR(100),
    salary INT
);

INSERT INTO employees (name, role_name, salary)
VALUES
('Farhan', 'Cloud Engineer', 50000),
('Rahul', 'DevOps Engineer', 60000),
('Priya', 'Database Admin', 55000);

SELECT * FROM employees;
```

---

# Part 7 — Run SQL from a `.sql` file using PowerShell

Instead of typing SQL manually, you can create a SQL file and run it from PowerShell.

## Step 7: Create `setup.sql`

In PowerShell:

```powershell
@"
CREATE DATABASE employee_db;
USE employee_db;

CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    role_name VARCHAR(100),
    salary INT
);

INSERT INTO employees (name, role_name, salary)
VALUES
('Farhan', 'Cloud Engineer', 50000),
('Rahul', 'DevOps Engineer', 60000),
('Priya', 'Database Admin', 55000);

SELECT * FROM employees;
"@ | Out-File -FilePath C:\Temp\setup.sql -Encoding ascii
```

Now execute it against RDS:

```powershell
mysql -h employee-db.abc123xyz.ap-south-1.rds.amazonaws.com -P 3306 -u admin -p < C:\Temp\setup.sql
```

---

# Part 8 — Create a PowerShell script to query RDS

Now let’s make this feel like a real project by querying RDS from a PowerShell script.

## Step 8: Create `query-employees.ps1`

```powershell
$RdsHost = "employee-db.abc123xyz.ap-south-1.rds.amazonaws.com"
$RdsPort = 3306
$RdsUser = "admin"
$Database = "employee_db"

Write-Host "Enter RDS password:" -ForegroundColor Yellow
$SecurePass = Read-Host -AsSecureString
$BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($SecurePass)
$PlainPassword = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)

$query = "SELECT * FROM employees;"

mysql -h $RdsHost -P $RdsPort -u $RdsUser -p$PlainPassword -D $Database -e $query

[System.Runtime.InteropServices.Marshal]::ZeroFreeBSTR($BSTR)
```

Run it like this:

```powershell
powershell -ExecutionPolicy Bypass -File .\query-employees.ps1
```

This will fetch the records from the RDS table.

---

# Part 9 — Add insert operation using PowerShell

Create a script called `insert-employee.ps1`.

```powershell
$RdsHost = "employee-db.abc123xyz.ap-south-1.rds.amazonaws.com"
$RdsPort = 3306
$RdsUser = "admin"
$Database = "employee_db"

$name = Read-Host "Enter employee name"
$role = Read-Host "Enter employee role"
$salary = Read-Host "Enter employee salary"

Write-Host "Enter RDS password:" -ForegroundColor Yellow
$SecurePass = Read-Host -AsSecureString
$BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($SecurePass)
$PlainPassword = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)

$query = "INSERT INTO employees (name, role_name, salary) VALUES ('$name', '$role', $salary);"

mysql -h $RdsHost -P $RdsPort -u $RdsUser -p$PlainPassword -D $Database -e $query

Write-Host "Employee inserted successfully."

[System.Runtime.InteropServices.Marshal]::ZeroFreeBSTR($BSTR)
```

Then run:

```powershell
powershell -ExecutionPolicy Bypass -File .\insert-employee.ps1
```

---

# Part 10 — Full PowerShell project structure

You can keep the project like this on the EC2 Windows machine:

```text
C:\RDS-MySQL-Project\
│
├── setup.sql
├── query-employees.ps1
├── insert-employee.ps1
├── update-salary.ps1
└── delete-employee.ps1
```

---

# Part 11 — Update salary script

## `update-salary.ps1`

```powershell
$RdsHost = "employee-db.abc123xyz.ap-south-1.rds.amazonaws.com"
$RdsPort = 3306
$RdsUser = "admin"
$Database = "employee_db"

$id = Read-Host "Enter employee ID"
$newSalary = Read-Host "Enter new salary"

Write-Host "Enter RDS password:" -ForegroundColor Yellow
$SecurePass = Read-Host -AsSecureString
$BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($SecurePass)
$PlainPassword = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)

$query = "UPDATE employees SET salary = $newSalary WHERE id = $id;"

mysql -h $RdsHost -P $RdsPort -u $RdsUser -p$PlainPassword -D $Database -e $query

Write-Host "Salary updated successfully."

[System.Runtime.InteropServices.Marshal]::ZeroFreeBSTR($BSTR)
```

---

# Part 12 — Delete employee script

## `delete-employee.ps1`

```powershell
$RdsHost = "employee-db.abc123xyz.ap-south-1.rds.amazonaws.com"
$RdsPort = 3306
$RdsUser = "admin"
$Database = "employee_db"

$id = Read-Host "Enter employee ID to delete"

Write-Host "Enter RDS password:" -ForegroundColor Yellow
$SecurePass = Read-Host -AsSecureString
$BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($SecurePass)
$PlainPassword = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)

$query = "DELETE FROM employees WHERE id = $id;"

mysql -h $RdsHost -P $RdsPort -u $RdsUser -p$PlainPassword -D $Database -e $query

Write-Host "Employee deleted successfully."

[System.Runtime.InteropServices.Marshal]::ZeroFreeBSTR($BSTR)
```

---

# Part 13 — Backup the database from PowerShell

You can also practice backup/export.

## Export database

```powershell
mysqldump -h employee-db.abc123xyz.ap-south-1.rds.amazonaws.com -P 3306 -u admin -p employee_db > C:\RDS-MySQL-Project\employee_db_backup.sql
```

## Import database

```powershell
mysql -h employee-db.abc123xyz.ap-south-1.rds.amazonaws.com -P 3306 -u admin -p employee_db < C:\RDS-MySQL-Project\employee_db_backup.sql
```

---

# Part 14 — Real practice tasks for this project

## Task 1: Connectivity test

* create RDS
* create Windows EC2
* install MySQL client in PowerShell
* run:

  ```powershell
  Test-NetConnection -ComputerName "<RDS-ENDPOINT>" -Port 3306
  ```

## Task 2: DB creation

* connect to RDS
* create `employee_db`
* create `employees` table
* insert 3 rows

## Task 3: PowerShell automation

Create PowerShell scripts for:

* list employees
* insert employee
* update employee salary
* delete employee

## Task 4: Backup and restore

* export DB with `mysqldump`
* create RDS snapshot from console
* restore it into a new instance

---

# Recommended final project flow

Follow this order:

## Stage 1

* Create RDS MySQL
* Create Windows EC2
* configure security groups
* test port 3306 with `Test-NetConnection`

## Stage 2

* install MySQL client using PowerShell
* connect to RDS
* create DB and table

## Stage 3

* create `.ps1` scripts for CRUD operations
* test insert/update/delete

## Stage 4

* export DB backup
* restore from backup
* create RDS snapshot

---

# My recommendation for your assignment/project

Build this exact project:

## **Project name: AWS RDS MySQL Employee Management Using PowerShell**

### Features

* RDS MySQL database on AWS
* Windows EC2 client machine
* PowerShell scripts to:

  * connect to RDS
  * create schema
  * insert employee
  * list employees
  * update salary
  * delete employee
  * export backup

---

If you want, I can do the next step and give you a **complete ready-to-run project folder** with:

* `01-install-mysql-client.ps1`
* `02-test-rds-connection.ps1`
* `03-create-db.ps1`
* `04-insert-employee.ps1`
* `05-list-employees.ps1`
* `06-update-employee.ps1`
* `07-delete-employee.ps1`
* `08-backup-db.ps1`

all written properly in PowerShell, in the correct order, with your folder structure and commands.

If you want that, just say **“give me the full PowerShell project files”** and I’ll generate the entire project for you.
