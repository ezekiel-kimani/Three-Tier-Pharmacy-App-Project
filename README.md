# AWS Three-Tier Pharmacy Application

## Project Overview

This project involved deploying a pharmacy web application on Amazon Web Services (AWS) using a three-tier architecture. The work focused on configuring the cloud infrastructure, connecting the application to its database, distributing traffic across application servers, and troubleshooting session management across multiple instances.

The architecture uses an Amazon VPC with public and private subnets across two Availability Zones. An Application Load Balancer distributes incoming traffic to Amazon EC2 application servers managed by an Auto Scaling Group. Amazon RDS for PostgreSQL stores application data, while Amazon ElastiCache for Valkey provides shared session storage. Amazon S3 stores static media, including medicine images, and Amazon CloudFront supports content delivery and caching.

The project provided practical experience in AWS networking, compute, database services, load balancing, scaling, access control, content delivery, and application troubleshooting.

**Live application:** [Open the Pharmacy Application](http://pharmacy-alb-1414575796.us-east-1.elb.amazonaws.com:8080/)

**AWS Region:** US East (N. Virginia) — `us-east-1`

## Architecture

The architecture separates the application's main components and places the application servers in private subnets across two Availability Zones. The Application Load Balancer provides the entry point for application traffic, while the database and shared session store support application operations.


### Request and Data Flow

1. Users access the pharmacy application through the internet and the Application Load Balancer.
2. The load balancer forwards requests to healthy registered EC2 instances through its target group.
3. The EC2 application servers process requests and communicate with Amazon RDS for PostgreSQL to retrieve or store application data.
4. Amazon ElastiCache for Valkey provides shared session storage, allowing application instances to access session information consistently.
5. Amazon S3 stores static media, including medicine images. Amazon CloudFront delivers eligible content and serves cached copies when available.
6. A bastion host provides an access point for authorised infrastructure administration.

## AWS Services and Resources

| AWS service or resource | Purpose |
|---|---|
| Amazon VPC | Provides the network environment for the application. |
| Public and private subnets | Separate internet-facing resources from internal application and database resources. |
| Internet Gateway | Enables internet connectivity for resources with appropriate routes and security rules. |
| NAT Gateway | Allows resources in private subnets to initiate permitted outbound internet connections. |
| Route tables | Control how network traffic is routed. |
| Application Load Balancer | Receives incoming application traffic and distributes requests across registered application servers. |
| Target group | Registers application instances and supports health checks and traffic routing. |
| Amazon EC2 | Runs the pharmacy application on application servers in private subnets. The instances process requests and communicate with the database and shared session store. |
| EC2 Launch Template | Defines the instance configuration used to launch consistently configured EC2 instances. |
| EC2 Auto Scaling Group | Manages the configured application instance capacity across two Availability Zones. |
| Amazon RDS for PostgreSQL | Stores the application's relational data. |
| Amazon ElastiCache for Valkey | Provides shared session storage across application instances. |
| AWS IAM and instance profile | Provide permissions for authorised AWS operations and make the configured instance role available to EC2 instances. |
| AWS Systems Manager | Supports managed instance administration through the configured integration. |
| Bastion host | Provides an access point for infrastructure administration. |
| Security groups | Control permitted network traffic to and from supported resources. |
| Amazon S3 | Stores static media, including medicine images. |
| Amazon CloudFront | Delivers eligible content through edge locations and caches content according to the configured cache behaviour. |

## Implementation

### 1. Network Configuration

An Amazon VPC was configured with public and private subnets across two Availability Zones. The network includes an Internet Gateway, a NAT Gateway, and route tables to control connectivity. The Application Load Balancer is internet-facing, while the application instances run in private subnets. Security groups control network traffic between the relevant components.

### 2. Application Servers, Load Balancing and Auto Scaling

The Application Load Balancer, `pharmacy-alb`, receives application traffic on HTTP port `8080` and forwards requests through the `pharmacy-app-tg` target group. The `pharmacy-app-asg` Auto Scaling Group manages the application instances.

The `pharmacy-app-launch-template` defines the EC2 configuration used when instances are launched, helping maintain consistent configurations across instances.

The configured Auto Scaling Group has a minimum capacity of two instances, a desired capacity of three, and a maximum capacity of three. The application tier spans two Availability Zones.

This arrangement supports load distribution and the replacement or scaling of instances according to the Auto Scaling configuration and applicable health checks.

### 3. Database Configuration

Amazon RDS for PostgreSQL provides relational database storage for the application. The database is placed in the private network, with security group rules controlling application connectivity. The database storage is encrypted using AWS Key Management Service (AWS KMS).

The deployed database uses a Single-AZ configuration. Although the application tier spans two Availability Zones, the database does not have a Multi-AZ standby deployment.

### 4. Shared Session Management

An important troubleshooting task involved maintaining application sessions when requests were handled by different EC2 instances. When an application relies on instance-local session storage, a subsequent request routed to another instance may not have access to the original session. This can affect the user's experience.

Amazon ElastiCache for Valkey was configured as shared session storage to address this issue. The application instances could then access shared session information instead of relying solely on local session files.

### 5. Static Media and Content Delivery

Amazon S3 stores static media, including the medicine images used by the application. Amazon CloudFront provides the content delivery layer shown in the architecture.

CloudFront caches eligible content at edge locations according to its cache configuration. When a valid cached object is available, CloudFront can serve it without retrieving the object from the origin again. This reduces repeated origin requests and can improve content delivery speed.

CloudFront caching does not automatically cache PostgreSQL queries or replace the application's database.

### 6. Access and Administration

AWS IAM provides permissions for authorised AWS operations. The configured instance profile makes the EC2 instance role available to the instances, while AWS Systems Manager supports managed instance administration through the project's SSM integration.

A bastion host provides an access point for infrastructure administration. Security groups restrict network traffic according to the configured access requirements.

Within the application, customer and administrator functions use a shared login entry point, with role-based access controls enforced by the application. Separate login pages were not implemented.

## Testing and Verification

The application was accessed through the Application Load Balancer, and the customer workflow was tested. The tested application functions included:

- Viewing the medicine catalogue.
- Adding medicines to the cart.
- Reviewing checkout information.
- Accessing the My Orders section.
- Viewing existing orders and their statuses.
- Confirming the application's database connection.

The shared-session configuration was used to address the session issue encountered when requests were handled by different application instances.

These checks verified the tested application workflows. They do not establish that every failure scenario, security control, or performance limit was tested.

## AWS Well-Architected Considerations

The architecture incorporates principles from four AWS Well-Architected Framework pillars. This section describes their relevance to the implementation; it does not represent a formal Well-Architected review.

**Operational Excellence**

The EC2 launch template and Auto Scaling Group provide a consistent approach to launching and managing application instances. The project also involved troubleshooting application sessions and documenting the resulting configuration.

**Security**

Private subnets, security groups, IAM permissions, and encrypted RDS storage provide layers of network and access control. The application currently uses HTTP on port `8080`; configuring HTTPS with a suitable TLS certificate remains an improvement.

**Reliability**

The application tier spans two Availability Zones and uses an Application Load Balancer and Auto Scaling. Valkey provides shared session storage across application instances. The Single-AZ RDS configuration remains a limitation to database availability.

**Performance Efficiency**

The Application Load Balancer distributes incoming requests across registered application instances. CloudFront caches eligible static media, helping reduce repeated origin requests and improve delivery speed when cached content is available.

## Limitations and Future Improvements

The following limitations are relevant to the deployed architecture:

- **HTTPS:** The application endpoint uses HTTP on port `8080`. HTTPS with a custom domain and an appropriate TLS certificate remains an improvement.
- **Database availability:** RDS uses Single-AZ deployment. A Multi-AZ configuration could improve database availability.
- **Resilience testing:** The documented testing does not establish that full load testing, disaster-recovery exercises, or all failure scenarios were completed.
- **Monitoring and operations:** Further work could include reviewing monitoring coverage, alarms, and recovery procedures.
- **Cost management:** Instance sizing, database requirements, and resource utilisation should be reviewed against actual workload needs.

These are potential improvements rather than features claimed to have been implemented.

## Key Learnings

This project provided practical experience in:

- Configuring an Amazon VPC with public and private subnets.
- Connecting an Application Load Balancer to an Auto Scaling Group.
- Using launch templates to maintain consistent EC2 configurations.
- Connecting application servers to Amazon RDS for PostgreSQL.
- Troubleshooting cross-instance sessions using Valkey as shared session storage.
- Applying IAM permissions and security groups to AWS resources.
- Using Amazon S3 and CloudFront for static media storage and delivery.
- Testing application workflows and documenting verified results and limitations.

## Conclusion

The AWS Three-Tier Pharmacy Application brought together networking, application servers, database storage, shared session management, access control, and content delivery in a working deployment.

A key lesson from the project was that distributing an application across multiple instances requires more than a load balancer and Auto Scaling. Session management, database connectivity, network permissions, and the behaviour of individual components must also be considered.

The project strengthened my practical understanding of how AWS services work together to support a web application and how to troubleshoot issues during deployment.
