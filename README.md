# AWS Three-Tier Web Application Architecture

A hands-on AWS project demonstrating a classic three-tier web application design using a custom VPC, public and private subnets, an Application Load Balancer, EC2 application instances, Amazon RDS for MySQL, Auto Scaling, IAM and CloudWatch/SNS.

## Project overview

The architecture separates the web entry point, application workload and database into distinct network tiers:

- **Presentation tier:** internet-facing Application Load Balancer in public subnets.
- **Application tier:** EC2-based web application in private subnets. The walkthrough installs Apache/PHP and deploys WordPress.
- **Data tier:** Amazon RDS MySQL in dedicated private database subnets.

The application is reached through the load balancer rather than by exposing the EC2 application instance directly.

## Architecture and network design

| Component | Configuration in this walkthrough |
|---|---|
| AWS VPC | `10.0.0.0/16` |
| Public subnet 1 | `10.0.1.0/24` |
| Public subnet 2 | `10.0.2.0/24` |
| Private application subnet 1 | `10.0.3.0/24` |
| Private application subnet 2 | `10.0.4.0/24` |
| Private database subnet 1 | `10.0.5.0/24` |
| Private database subnet 2 | `10.0.6.0/24` |
| Internet access | Internet Gateway for public subnets |
| Private outbound access | NAT Gateway and private route |
| Load balancing | Application Load Balancer and target group |
| Application compute | EC2, AMI and Auto Scaling Group |
| Database | Amazon RDS for MySQL |
| Operations | IAM instance role, CloudWatch and SNS |

### Traffic flow

```mermaid
flowchart TB
    U[Internet user] --> ALB[Application Load Balancer\nPublic subnets]
    ALB --> APP[EC2 application tier\nPrivate subnets]
    APP --> DB[Amazon RDS MySQL\nPrivate DB subnets]
    APP --> CW[CloudWatch]
    CW --> SNS[SNS notifications]
    PUB[Public subnet route] --> IGW[Internet Gateway]
    PRIV[Private subnet route] --> NAT[NAT Gateway]
    NAT --> IGW
```

### Security group intent

The security groups are intended to enforce tier-to-tier access:

| Security group | Inbound access |
|---|---|
| Load balancer | HTTP/HTTPS from approved client ranges |
| Application | Application HTTP traffic from the load balancer security group |
| Database | MySQL traffic from the application security group only |

For a real deployment, restrict public ingress to the required client ranges, use HTTPS with an appropriate certificate, and avoid opening SSH/RDP to the entire internet. Prefer Systems Manager Session Manager for administrative access where possible.

## Implementation walkthrough

1. **Create the VPC and subnets.** Create the VPC and separate public, private application and private database subnets across Availability Zones.
2. **Configure routing.** Attach an Internet Gateway, create public and private route tables, and route private-subnet outbound traffic through the NAT Gateway.
3. **Create security groups.** Allow client traffic to the load balancer, load-balancer-to-application traffic, and application-to-database traffic. Keep the database private.
4. **Create an IAM instance role.** The original walkthrough uses the `AmazonSSMManagedInstanceCore` policy for Systems Manager access.
5. **Launch the application instance.** Place the EC2 instance in a private application subnet and attach the application security group and IAM role.
6. **Install the web stack.** Install Apache, PHP and the required PHP/MySQL integration packages. Deploy WordPress application files to the web root.
7. **Create the database.** Create an RDS subnet group using the private database subnets, provision the MySQL database, and create the application database and credentials.
8. **Configure the application.** Add the RDS endpoint and database connection values to the WordPress configuration. Never commit real credentials to Git.
9. **Create an AMI and target group.** Capture the configured application instance as an AMI and register instances with a target group.
10. **Create the Application Load Balancer.** Place the ALB in the public subnets and configure its listener to forward to the application target group.
11. **Configure Auto Scaling.** Use the application AMI and target group to define the Auto Scaling Group and its capacity settings.
12. **Add monitoring and notifications.** Configure CloudWatch monitoring and SNS notifications for the selected events/alarms.

## Validation checklist

Use these checks when recreating the environment:

- [ ] VPC, subnet CIDRs and route-table associations match the design.
- [ ] Public load balancer is reachable on its configured listener.
- [ ] Application instances remain in private subnets.
- [ ] Target group reports healthy application targets.
- [ ] Application can connect to the private RDS database.
- [ ] Database is not directly reachable from the public internet.
- [ ] WordPress page loads through the ALB DNS name.
- [ ] Auto Scaling Group reports the intended desired capacity.
- [ ] CloudWatch alarms and SNS notifications are tested.

## Important design notes

- This repository documents a **console-based implementation walkthrough** with screenshots. It is not an automated Terraform deployment.
- The original design uses a single NAT Gateway. That is a cost-conscious learning setup, but it can introduce an Availability Zone dependency. A production design should assess per-AZ NAT Gateways, resilience, cost and egress controls.
- Configure TLS at the load balancer for real workloads.
- Use Secrets Manager or Systems Manager Parameter Store for database credentials rather than storing secrets in source code or AMIs.
- Review EC2/RDS sizing, backups, encryption, patching, logging and least-privilege IAM before using a similar design for production.
- The resource configuration and screenshots are an educational reference, not a claim of a currently running AWS environment.

## Technology stack

Amazon VPC · EC2 · Application Load Balancer · Auto Scaling · Amazon RDS (MySQL) · IAM · Systems Manager · CloudWatch · SNS · WordPress · Apache · PHP

## Screenshots

The screenshots below document the original console walkthrough. Expand a section to view the evidence.

<details>
<summary>01. Architecture overview</summary>

![Architecture overview](https://github.com/user-attachments/assets/26d6fa14-3718-40ac-98ea-382d34890494)
</details>

<details>
<summary>02. VPC and subnet configuration</summary>

![VPC and subnet configuration](https://github.com/user-attachments/assets/567aec20-c414-4af7-a87c-f98cdf7a5d33)
</details>

<details>
<summary>03. Public and private subnet layout</summary>

![Public and private subnet layout](https://github.com/user-attachments/assets/da710ff4-f774-462c-89f3-fb04e783f906)
</details>

<details>
<summary>04. Route table configuration</summary>

![Route table configuration](https://github.com/user-attachments/assets/932f9b68-2cd0-486c-a263-568b1b2a3480)
</details>

<details>
<summary>05. Internet Gateway</summary>

![Internet Gateway](https://github.com/user-attachments/assets/404779f1-90fd-4909-9445-517653176924)
</details>

<details>
<summary>06. NAT Gateway</summary>

![NAT Gateway](https://github.com/user-attachments/assets/0b6036c4-e69f-4b1f-a465-15d2e4faf565)
</details>

<details>
<summary>07. Public route to Internet Gateway</summary>

![Public route to Internet Gateway](https://github.com/user-attachments/assets/55ece915-5840-4c27-95c6-8c0eb4ccf4de)
</details>

<details>
<summary>08. Private route through NAT Gateway</summary>

![Private route through NAT Gateway](https://github.com/user-attachments/assets/7d2f1540-2918-4ca8-9287-ea27e3dfd111)
</details>

<details>
<summary>09. Security group rules</summary>

![Security group rules](https://github.com/user-attachments/assets/d9a46148-c34b-4d25-9c6d-44cdca0a1075)
</details>

<details>
<summary>10. EC2 Systems Manager IAM role</summary>

![EC2 Systems Manager IAM role](https://github.com/user-attachments/assets/628753b4-f603-4c52-8ecb-991c9172f01e)
</details>

<details>
<summary>11. Application EC2 instance</summary>

![Application EC2 instance](https://github.com/user-attachments/assets/85edd418-c3a3-4036-a5e0-11f20080e654)
</details>

<details>
<summary>12. RDS subnet group – primary view</summary>

![RDS subnet group – primary view](https://github.com/user-attachments/assets/54d80a1a-1b21-422d-9a0d-53c02937021a)
</details>

<details>
<summary>13. RDS subnet group – subnet selection</summary>

![RDS subnet group – subnet selection](https://github.com/user-attachments/assets/f261cf9d-81c2-4713-a7e5-016a356bb756)
</details>

<details>
<summary>14. Database connection and setup</summary>

![Database connection and setup](https://github.com/user-attachments/assets/eeb44cd6-42bc-498c-9d39-52463050022e)
</details>

<details>
<summary>15. WordPress application files</summary>

![WordPress application files](https://github.com/user-attachments/assets/f342acca-22bc-4414-ac1e-4600316d58f8)
</details>

<details>
<summary>16. WordPress database configuration</summary>

![WordPress database configuration](https://github.com/user-attachments/assets/3e5d3897-8f90-41dc-be7c-af626ee1853b)
</details>

<details>
<summary>17. Application server AMI</summary>

![Application server AMI](https://github.com/user-attachments/assets/ae6f8dc1-8e9f-41e6-8ec9-dad58d4aefc3)
</details>

<details>
<summary>18. Target group</summary>

![Target group](https://github.com/user-attachments/assets/8a702df1-ffd2-46e2-ab24-dbe7126fbb9e)
</details>

<details>
<summary>19. Application Load Balancer configuration</summary>

![Application Load Balancer configuration](https://github.com/user-attachments/assets/72a228b7-4db0-4718-be75-2023ab4f5f48)
</details>

<details>
<summary>20. Application response through load balancer</summary>

![Application response through load balancer](https://github.com/user-attachments/assets/3021d52d-105a-4908-9b2c-2df66a52adcc)
</details>

<details>
<summary>21. WordPress deployment verification</summary>

![WordPress deployment verification](https://github.com/user-attachments/assets/babeddeb-6bb0-4d93-9ff0-0881667a5b2a)
</details>

<details>
<summary>22. Application availability verification</summary>

![Application availability verification](https://github.com/user-attachments/assets/91c5d519-d3ea-44d8-87f9-e31146c7e694)
</details>

<details>
<summary>23. Auto Scaling Group configuration</summary>

![Auto Scaling Group configuration](https://github.com/user-attachments/assets/e4ede581-bf13-4ae2-9fc7-6e0368983a5e)
</details>

<details>
<summary>24. Auto Scaling Group capacity</summary>

![Auto Scaling Group capacity](https://github.com/user-attachments/assets/2a431477-722b-4266-975c-c5e6dc3b95a6)
</details>

<details>
<summary>25. SNS and CloudWatch configuration</summary>

![SNS and CloudWatch configuration](https://github.com/user-attachments/assets/bddd3c7f-67de-42c0-a48e-83dc6f79ce4c)
</details>

<details>
<summary>26. Monitoring and notification configuration</summary>

![Monitoring and notification configuration](https://github.com/user-attachments/assets/f7db7c4f-420c-487b-a602-9eec424cf362)
</details>

## Author

[JOSH801251](https://github.com/JOSH801251)
