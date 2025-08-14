# LLM Prompt: Generate AWS CloudFormation Template (YAML) for Infrastructure Deployment

You are an expert AWS Infrastructure as Code (IaC) generator.  
Generate a **production-grade AWS CloudFormation template** in **YAML format** based on the following detailed specifications.  
Ensure that the template is **fully functional, logically structured, and deployable without manual changes**.

---

## Deployment Region
- All resources **must** be deployed in:  
  - **Region:** `us-east-1`

---

## Networking Requirements
1. **VPC Configuration**
   - Create a new VPC with CIDR block: `10.0.0.0/16`
   - Enable DNS Hostnames and DNS Support
   - Tag all VPC-related resources with: `Project: Migration`

2. **Subnets**
   - **Public Subnets**
     - At least 2 public subnets in different Availability Zones (AZs).
     - Must be able to host internet-facing resources (e.g., ELB, NAT Gateways).
   - **Private Subnets**
     - At least 2 private subnets in different AZs.
     - EC2 instances and RDS instances will reside here.

3. **Internet Connectivity**
   - **Internet Gateway (IGW)** attached to the VPC for public subnets.
   - **NAT Gateways** in public subnets for outbound internet access from private subnets.

4. **Route Tables**
   - Public route table for public subnets with default route to IGW.
   - Private route table for private subnets with default route to NAT Gateway(s).

5. **VPC Peering**
   - Establish VPC Peering connection with an **existing VPC**.  
   - Existing VPC ID will be provided as a parameter (`ExistingVPCId`).
   - Configure route tables to allow bidirectional communication between VPCs.

---

## Compute Requirements
1. **EC2 Instances**
   - Launch EC2 instances in **private subnets**.
   - Enable **Auto Scaling Group (ASG)** to ensure high availability and scalability.
     - Minimum: 2 instances
     - Maximum: 6 instances
     - Desired Capacity: 3 instances
   - Use the latest Amazon Linux 2 AMI.
   - Associate EC2 instances with:
     - Security Groups allowing only required ports.
     - IAM Role with least-privilege access.
   - **Storage:**
     - All EBS volumes **must** be encrypted with AWS KMS (create a KMS CMK if not available).
   - **Monitoring:**
     - Enable CloudWatch detailed monitoring.
     - Send logs to CloudWatch Logs.

2. **Elastic Load Balancer (ELB)**
   - Deploy an Application Load Balancer (ALB) in public subnets.
   - Distribute inbound traffic to EC2 instances in private subnets.
   - Enable health checks.

---

## Storage Requirements
1. **Amazon S3**
   - Create an S3 bucket for application static assets.
   - Enable **bucket versioning**.
   - Enable **default encryption (SSE-S3 or SSE-KMS)**.
   - **Lifecycle Policy**:
     - Transition objects to **Glacier** after **30 days**.
   - Block all public access (use CloudFront for public content delivery).

---

## Database Requirements
1. **Amazon RDS**
   - Use Amazon RDS (MySQL or PostgreSQL — parameterized).
   - Deploy into private subnets.
   - Multi-AZ enabled for high availability.
   - Enable automated backups (7-day retention).
   - Enable storage encryption using AWS KMS.
   - Restrict access using Security Groups and IAM.

---

## Security Requirements
1. **IAM Roles and Policies**
   - Create IAM roles for:
     - EC2 instances (allow only required S3, CloudWatch, and KMS actions).
     - RDS monitoring.
     - CloudFormation execution.
   - Implement **least-privilege access**.
   - All roles and policies must be explicitly defined in the template.

2. **AWS KMS**
   - Create a Customer Managed Key (CMK) for:
     - EBS volume encryption.
     - RDS encryption.
     - S3 bucket encryption.
   - Key policy should allow access only to necessary IAM roles.

---

## Content Delivery
1. **Amazon CloudFront**
   - Create a CloudFront distribution for the S3 bucket.
   - Enable HTTPS using ACM-managed certificate (in `us-east-1`).
   - Configure caching and default root object.

---

## Monitoring & Logging
1. **AWS CloudTrail**
   - Enable CloudTrail in all regions.
   - Store logs in an encrypted S3 bucket.
   - Enable log file integrity validation.

2. **Amazon CloudWatch**
   - Create CloudWatch log groups for:
     - EC2 instances
     - RDS
     - ELB access logs
   - Ensure log retention policy is set (e.g., 90 days).

---

## DNS Management
1. **Amazon Route 53**
   - Create a hosted zone for the application domain (parameterized).
   - Configure **Failover Routing Policy**:
     - Primary: CloudFront distribution
     - Secondary: Static S3 website or alternate resource
   - Health checks for failover.

---

## Tagging Standards
- **All resources** must have the following tag:  
  - `Key: Project`  
  - `Value: Migration`

---

## Output Requirements
The generated CloudFormation YAML should:
1. Use **parameters** for configurable values (e.g., VPC CIDR, instance types, domain name, existing VPC ID).
2. Include **Outputs** for:
   - VPC ID
   - Subnet IDs
   - Security Group IDs
   - Load Balancer DNS
   - RDS endpoint
   - CloudFront distribution URL
3. Follow AWS best practices for security, availability, and cost optimization.

---

**Deliverable:**  
A **valid, production-ready CloudFormation YAML file** that can be deployed in `us-east-1` without manual edits, meeting all above requirements.
