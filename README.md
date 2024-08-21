# AWS 3-Tier Architecture with Blue-Green Deployment Strategy 

This project demonstrates how to set up a 3-tier architecture on AWS, which includes a VPC with subnets, a load balancer, auto-scaling, a MySQL database, and a blue-green deployment strategy for continuous delivery.

## Architecture Overview

- **Tier 1: Web Layer (Public Subnets)** - Contains the load balancer and web servers.
- **Tier 2: Application Layer (Private Subnets)** - Contains the application servers.
- **Tier 3: Database Layer (Database Subnets)** - Contains the MySQL database.

## Prerequisites

- AWS Account with Free Tier eligibility
- Basic understanding of AWS services (VPC, EC2, RDS, ALB, etc.)

## Steps to Implement

### 1. Create a VPC

- **VPC Name:** `3-Tier-VPC`
- **CIDR Block:** `10.0.0.0/16`
- **Subnets:**
  - Public Subnet 1: `10.0.1.0/24`
  - Public Subnet 2: `10.0.2.0/24`
  - Private Subnet 1: `10.0.3.0/24`
  - Private Subnet 2: `10.0.4.0/24`
  - Database Subnet 1: `10.0.5.0/24`
  - Database Subnet 2: `10.0.6.0/24`

### 2. Create an Internet Gateway

- **Name:** `3-Tier-IGW`
- Attach it to the VPC.

### 3. Create a NAT Gateway

- **Name:** `3-Tier-NATGW`
- **Subnet:** Public Subnet 1
- **Elastic IP:** Allocate and associate a new Elastic IP.

### 4. Configure Route Tables

- **Public Route Table:**
  - Associate with Public Subnets.
  - Add a route for `0.0.0.0/0` pointing to the Internet Gateway.
- **Private Route Table:**
  - Associate with Private Subnets.
  - Add a route for `0.0.0.0/0` pointing to the NAT Gateway.
- **Database Route Table:**
  - Associate with Database Subnets.
  - No route for `0.0.0.0/0` (to ensure the database subnets remain private).

### 5. Create Security Groups

- **Web Security Group:**
  - Allow inbound HTTP (80) and HTTPS (443) from anywhere.
  - Allow inbound SSH (22) from your IP address.
- **App Security Group:**
  - Allow inbound traffic from the Web Security Group on the application port (e.g., 8080).
- **Database Security Group:**
  - Allow inbound MySQL (3306) from the App Security Group.

### 6. Launch EC2 Instances for Web and Application Layers

- **Launch Template:**
  - **Name:** `3-Tier-Launch-Template`
  - **Instance Type:** `t2.micro` (Free Tier)
  - **AMI:** Choose a Free Tier eligible Linux AMI (e.g., Amazon Linux 2)
  - **Key Pair:** Select or create a new key pair for SSH access.
  - **Network:** Select the VPC and the corresponding subnets for Web and Application layers.
  - **Security Group:** Use the Web Security Group for web servers and App Security Group for application servers.
- **Auto Scaling Group:**
  - Create an Auto Scaling group for each layer (Web and Application).
  - Set the desired capacity, minimum, and maximum instances according to your needs.
  - **Scaling Policy:** Configure based on CPU utilization.

### 7. Configure a Load Balancer (ALB)

- **Name:** `3-Tier-ALB`
- **Type:** Application Load Balancer
- **Network:** Select the VPC and associate with Public Subnets.
- **Security Group:** Use the Web Security Group.
- **Listeners:** Add listeners for HTTP (80) and HTTPS (443).
- **Target Groups:** Create target groups for both the web and application layers.
  - **Target Type:** Instance
  - **Health Check Path:** `/health` or similar endpoint.

### 8. Set Up MySQL Database (RDS)

- **Instance Type:** `db.t2.micro` (Free Tier)
- **Engine:** MySQL
- **Username:** `admin`
- **Password:** `<YourSecurePassword>`
- **DB Instance Identifier:** `3-tier-mysql-db`
- **DB Subnet Group:** Create or use the default DB subnet group and associate it with the database subnets.
- **Public Access:** No (Keep the database private)
- **Security Group:** Use the Database Security Group.

### 9. Blue-Green Deployment Setup

- **Create Separate Environments:**
  - **Blue Environment:** Existing version of the application running on the first set of instances.
  - **Green Environment:** New version of the application deployed to a new set of instances.
- **Create Additional Launch Template for Green Environment:** Duplicate the launch template with modifications if needed.
- **Target Group Switching:**
  - Attach the Green Environment’s instances to the target group.
  - Perform health checks and testing.
  - Redirect traffic to the Green Environment by modifying the target groups attached to the load balancer.
- **Rollback Plan:** If the Green Environment fails, redirect traffic back to the Blue Environment.

### 10. Final Testing and Monitoring

- **Test the Application:** Ensure all tiers are functioning correctly.
- **Monitoring:** Set up CloudWatch alarms for monitoring instance health, database performance, and auto-scaling actions.
- **Logging:** Enable logging for the load balancer and store logs in S3.

### 11. Cleanup (Optional)

- **Terminate Instances:** If you're done testing, terminate the EC2 instances.
- **Delete Resources:** Clean up other AWS resources to avoid unnecessary charges.

---

This guide provides a comprehensive approach to setting up a 3-tier architecture on AWS with a blue-green deployment strategy. Ensure that you follow each step closely to create a robust and scalable application infrastructure.
