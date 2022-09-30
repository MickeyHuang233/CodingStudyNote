#📆2022年 
狀態:: #🔲TODO 
完成日期:: 2022-12-31
標籤:: #💻編程/☁雲端/AWS
子筆記:: 
教程:: [AWS官方考試資源](https://explore.skillbuilder.aws/learn/course/134/play/31418/aws-cloud-practitioner-essentials-all-modules)
備註:: 

# AWS Cloud Practitioner
## ☁Introduction to AWS
- pay-as-you-go: you only pay for what you use, don't pre-pay for anything.
- Deployment models for cloud computing
	- on-premises: a private cloud deployment.
	- hybrid-cloud: cloud-based resources are connected to on-premises infrastructure.
	- all-in cloud: all parts of the application in the cloud.

## ☁Compute in the Cloud
- [All AWS Server Introduction](https://aws.amazon.com/tw/products/compute/)
- [AWS Compute Blog](https://aws.amazon.com/blogs/compute/)
- [Hands-On Tutorials: Compute](https://aws.amazon.com/getting-started/hands-on/)

### Amazon EC2
- [Amazon EC2(Elastic Compute Cloud)](https://aws.amazon.com/tw/ec2/): a virtual server. highly flexible, cost effective, and quick
- [Amazon EC2 instance types](https://aws.amazon.com/ec2/instance-types/) are optimized for different tasks.
	- General purpose instances
	- Compute optimized instances
	- Memory optimized instances
	- Accelerated computing instances, such as graphics applications, game streaming, and application streaming.
	- Storage optimized instances, include distributed file systems, data warehousing applications, and high-frequency online transaction processing (OLTP) systems.
- pricing
	- [On-Demand](https://aws.amazon.com/tw/ec2/pricing/on-demand/): short-term, irregular workloads that cannot be interrupted. No upfront costs or minimum contracts apply.
	- [Savings Plan](https://aws.amazon.com/tw/savingsplans/): A commitment to a consistent amount of usage measured in dollars per hour for a one or three-year term.
	- [Reserved Instances](https://aws.amazon.com/tw/ec2/pricing/reserved-instances/): These are suited for steady-state workloads or ones with predictable usage. You qualify for a discount once you commit to a one or three-year term and can pay for them with three payment options:
		- all upfront
		- partial upfront
		- no upfront
	- [Spot Instances](https://aws.amazon.com/tw/ec2/spot/pricing/): They allow you to request spare Amazon EC2 computing capacity for up to 90% off of the On-Demand price. AWS can reclaim the instance at any time they need it, giving you a two-minute warning to finish up work and save state.
	- [Dedicated Hosts](https://aws.amazon.com/tw/ec2/dedicated-hosts/): Physical hosts dedicated for your use for EC2
- Scaling EC2: scalability and elasticity
	![[Cloud Practitioner_01_auto scaling group.png]]

### Amazon ELB
[Amazon ELB(Elastic Load Balancing)](https://aws.amazon.com/tw/elasticloadbalancing/): A load balancer is an application that takes in requests and routes them to the instances to be processed.
![[Cloud Practitioner_02_Elastic Load Balancing.png]]

### Messaging and queuing
- [Amazon SQS(Simple Queue Service)](https://aws.amazon.com/tw/sqs/)
	![[Cloud Practitioner_03_Simple Queue Service.png]]
	- send, store, messages between software components
	- at any volume
	- The data contained within a message is called a payload
- [Amazon SNS(Simple Notification Service)](https://aws.amazon.com/tw/sns/)
	- pub/sub model(publish/subscribe)
	- you can create something called an SNS topic which is just a channel for messages to be delivered.

### Additional compute services
- [Amazon Elastic Container Service (Amazon ECS)](https://aws.amazon.com/ecs/)<--[Docker](https://www.docker.com/)
- [Amazon Elastic Kubernetes Service (Amazon EKS)](https://aws.amazon.com/eks/)<--[Kubernetes](https://kubernetes.io/)
- [Serverless](https://aws.amazon.com/getting-started/deep-dive-serverless/): Your code runs on servers, but you do not need to provision or manage these servers.
	- [AWS Lambda](https://aws.amazon.com/lambda) is a service that lets you run code without needing to provision or manage servers.
		![[Cloud Practitioner_04_AWS Lambda.png]]
	- [AWS Fargate](https://aws.amazon.com/fargate/)  is a serverless compute platform for ECS or EKS.

## ☁Global Infrastructure and Reliability
- [AWS Networking and Content Delivery Blog](https://aws.amazon.com/blogs/networking-and-content-delivery/)

### Global Infrastructure
- [Region](https://aws.amazon.com/tw/about-aws/global-infrastructure/regions_az/): A Region is a geographical area that contains AWS resources.
	- key factors to choose a Region
		- Compliance with data governance and legal requirements
		- Proximity to your customers
		- Available services within a Region: Sometimes, the closest Region might not have all the features that you want to offer to customers.
		- Pricing
- Availability Zone: An Availability Zone is a single data center or a group of data centers within a Region.
	- you run across at least two Availability Zones in a Region.
	![[Cloud Practitioner_05_Availability Zone.png]]
-  [Amazon CloudFront](https://aws.amazon.com/tw/cloudfront/) is a service that helps deliver data, video, applications, and APIs to customers around the world with low latency and high transfer speeds.
	- An **edge location** is a site that Amazon CloudFront uses to store cached copies of your content closer to your customers for faster delivery.
	![[Cloud Practitioner_06_Amazon CloudFront.png]]
- [Amazon Route 53](https://aws.amazon.com/tw/route53/) helping direct customers to the correct web locations with reliably low latency.
- [AWS Outposts](https://aws.amazon.com/tw/outposts/rack/features/) will basically install a fully operational mini Region, right inside your own data center.

### Ways to interact with AWS services
- AWS Management Console
	- test environments
	- viewing AWS bills
	- viewing monitoring
	- working with other non technical resources
- AWS Command Line Interface (AWS CLI)
- Software Development Kits (SDKs): The SDKs allow you to interact with AWS resources through various programming languages.
- [AWS Elastic Beanstalk](https://aws.amazon.com/tw/elasticbeanstalk/) : you provide code and configuration settings, and Elastic Beanstalk deploys the resources necessary to perform the following tasks:
	- Adjust capacity
	- Load balancing
	- Automatic scaling
	- Application health monitoring
- [AWS CloudFormation](https://aws.amazon.com/tw/cloudformation/): You can build an environment by writing lines of code instead of using the AWS Management Console to individually provision resources.

## ☁Networking
- [AWS Networking & Content Delivery Blog](https://aws.amazon.com/blogs/networking-and-content-delivery/)
- [How Amazon VPC works](https://docs.aws.amazon.com/vpc/latest/userguide/how-it-works.html)

### Connectivity to AWS
- [Amazon Virtual Private Cloud (VPCs)](https://aws.amazon.com/tw/vpc/): A VPC lets you provision a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define.
	- Virtual Private Cloud is essentially your own private network in AWS.
	- define your private IP range for your AWS resources
	- place things like EC2 instances and ELBs inside of your VPC
- A subnet is a section of a VPC that can contain resources such as Amazon EC2 instances.
	- Public subnets contain resources that need to be accessible by the public.
	- Private subnets contain resources that should be accessible only through your private network
- connect VPC gateway
	- Internet gateway(IGW): An internet gateway is a connection between a VPC and the internet.
		![[Cloud Practitioner_07_Internet gateway.png]]
	- Virtual private gateway: Virtual private gateway establish a virtual private network (VPN) connection between your VPC and a private network, such as an on-premises data center or internal corporate network.
		![[Cloud Practitioner_08_Virtual private gateway.png]]
	- [AWS Direct Connect](https://aws.amazon.com/directconnect/) enables you to establish a dedicated private connection between your data center and a VPC.
		![[Cloud Practitioner_09_AWS Direct Connect.png]]

### Network ACLs and security group
![[Cloud Practitioner_10_Network ACLs and security group.png]]
- [Network access control lists (Network ACLs)](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html) is a virtual firewall that controls inbound and outbound traffic at the ==subnet level.==
	- check to see if the sender is on the approved list when package access subnet.
	- Network ACLs perform ==stateless== packet filtering. They remember nothing and check packets that cross the subnet border each way: inbound and outbound.
	- By default, your account’s default network ACL ==allows== all inbound and outbound traffic.
- [security group](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html) is a virtual firewall that controls inbound and outbound traffic for ==an Amazon EC2 instance==.
	- By default, a security group ==denies== all inbound traffic and allows all outbound traffic.
	- Security group has some kind of a memory when it comes to who to allow in or out.

### Global networking
- [Amazon Route 53](https://aws.amazon.com/tw/route53/) is AWS's domain name service(DNS)
	- Route 53 can direct traffic to different endpoints using several different routing policies, such as latency-based routing, geolocation DNS, geoproximity, and weighted round robin.
- Content Delivery Network(CDN) is a network that helps to deliver edge content to users based on their geographic location.

## ☁Storage and Databases
- [AWS Storage Blog](https://aws.amazon.com/blogs/storage/)
- [AWS Database Blog](https://aws.amazon.com/blogs/database/)
- [Hands-On Tutorials: Storage](https://aws.amazon.com/getting-started/hands-on/)

### Amazon EBS
- **Block storage** breaks those files down to small component parts or blocks.

- [Instance Store](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html) provides ==temporary== ==block-level== storage for an Amazon EC2 instance.
	![[Cloud Practitioner_11_Instance Store.png]]

- [Amazon Elastic Block Store (Amazon EBS)](https://aws.amazon.com/ebs) is a service that provides ==block-level== storage volumes that you can use with Amazon EC2 instances.
	- If you stop or terminate an Amazon EC2 instance, all the data on the attached EBS volume remains available.
	- EBS volume stores data in ==a single Availability Zone==.

|               | 通用SSD(gp2)                         | 雲配置的IOPS SSD(io1)            | 吞吐量優化型HDD(st1)       | Cold HDD(sc1)      |
| ------------- | ------------------------------------ | -------------------------------- | -------------------------- | ------------------ |
| 使用案例      | 啟動卷、低延遲交互式應用、開發、測試 | I/O密集型NoSQL數據庫和關系數據庫 | 大數據、數據倉庫、日誌處理 | 大量不常訪問的數據 |
| 卷大小        | 1GiB ~ 16TiB                         | 4GiB ~ 16TiB                     | 500GiB ~ 16TiB             | 500GiB ~ 16TiB     |
| 最大IOPS/卷   | 10000                                | 32000                            | 500                        | 250                |
| 最大吞吐量/卷 | 160MiB/s                             | 500MiB/s                         | 500MiB/s                   | 250MiB/s           |
| 主要性能屬性  | IOPS                                 | IOPS                             | MiB/s                      | MiB/s              |

- [EBS snapshot](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSSnapshots.html) is an incremental backup.
	![[Cloud Practitioner_12_EBS snapshot.png]]

### Amazon S3
- **Object storage** treats any file as a complete, discreet object.
- [Amazon Simple Storage Service (Amazon S3)](https://aws.amazon.com/s3/) is a service that provides ==object-level== storage. Amazon S3 stores data as objects in buckets.

- S3 features
	- You can upload any type of file to Amazon S3, such as images, videos, text files, and so on.
	- Amazon S3 offers unlimited storage space.
	- The maximum file size for an object in Amazon S3 is 5 TB.
	- write once/read many
	- You can even create multiple buckets and store them across different classes or tiers of data.
	- You can then create permissions to limit who can see or even access objects.
	- 99.999999999% availability data

- S3 storage classe category
	- S3 Standard
		- Designed for frequently accessed data
		- Stores data in a minimum of three Availability Zones
		- S3 Standard has a higher cost than other storage classes intended for infrequently accessed data and archival storage.
	- S3 Standard-Infrequent Access (S3 Standard-IA)
		- Ideal for infrequently accessed data
		- the same level of availability as S3 Standard
		- S3 Standard-IA store data in a minimum of three Availability Zones.
		- Similar to S3 Standard but has a lower storage price and higher retrieval price
	- S3 One Zone-Infrequent Access (S3 One Zone-IA)
		- Stores data in a single Availability Zone
		- Has a lower storage price than S3 Standard-IA
	- S3 Intelligent-Tiering
		- Ideal for data with unknown or changing access patterns
		- Requires a small monthly monitoring and automation fee per object
		- If you haven’t accessed an object for 30 consecutive days, Amazon S3 automatically moves it to the infrequent access tier, S3 Standard-IA.
		- If you access an object in the infrequent access tier, Amazon S3 automatically moves it to the frequent access tier, S3 Standard.
	- S3 Glacier
		- Low-cost storage designed for data archiving
		- Able to retrieve objects within a few minutes to hours
	- S3 Glacier Deep Archive
		- Lowest-cost object storage class ideal for archiving
		- Able to retrieve objects within 12 hours

### Amazon EFS
- [Amazon Elastic File System (Amazon EFS)](https://aws.amazon.com/efs/) is a scalable file system used with AWS Cloud services and on-premises resources.
	- EFS can have multiple instances reading and writing from it at the same time.
	- EFS enables you to access data concurrently from ==all the Availability Zones in the Region== where a file system is located.
	- EFS is a true file system for Linux.
	- EFS is also a regional resource.
	- Automatically scales

### Amazon RDS
- [Amazon Relational Database Service (Amazon RDS)](https://aws.amazon.com/rds/) is a service that enables you to run relational databases in the AWS Cloud.
	- Amazon RDS is a managed service that automates tasks such as hardware provisioning, database setup, patching, and backups.
	- Many Amazon RDS database engines offer encryption at rest and encryption in transit.
	- Supported database engines include:
		- Amazon Aurora
		- PostgreSQL
		- MySQL
		- MariaDB
		- Oracle Database
		- Microsoft SQL Server
- [Amazon Aurora](https://aws.amazon.com/rds/aurora/) is an enterprise-class relational database.
	- It comes in two forms, MySQL and PostgreSQL.
	- It is up to five times faster than standard MySQL databases and up to three times faster than standard PostgreSQL databases.
	- It's priced is 1/10th the cost of commercial grade databases.
	- you have six copies at any given time.
	- continuously backs up your data to Amazon S3
	- point in time recovery

### Amazon DynamoDB
- [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) is a key-value database service.
	- It's fully managed, and it's highly scalable.
	- DynamoDB is a non-relational database.
	- It has millisecond response time.

### Amazon Redshift
[Amazon Redshift](https://aws.amazon.com/redshift) is a data warehousing service that you can use for big data analytics.

### AWS DMS
[AWS Database Migration Service (AWS DMS)](https://aws.amazon.com/dms/) enables you to migrate relational databases, nonrelational databases, and other types of data stores.

![[Cloud Practitioner_13_AWS DMS.jpg]]

- the source and target databases don't have to be of the same type.
- use cases for AWS DMS
	- migrate relational databases, nonrelational databases, and other types of data stores.
	- Development and test database migrations
	- Database consolidation
	- Continuous replication

### Additional database services
- [Amazon DocumentDB](https://aws.amazon.com/documentdb) is a document database service that supports MongoDB workloads. (MongoDB is a document database program.)
- [Amazon Neptune](https://aws.amazon.com/neptune) is a graph database service.
- [Amazon Quantum Ledger Database (Amazon QLDB)](https://aws.amazon.com/qldb) is a ledger database service. You can use Amazon QLDB to review a complete history of all the changes that have been made to your application data.
- [Amazon Managed Blockchain](https://aws.amazon.com/managed-blockchain) is a service that you can use to create and manage blockchain networks with open-source frameworks.
- cache solution
	- [Amazon ElastiCache](https://aws.amazon.com/elasticache) is a service that adds caching layers on top of your databases to help improve the read times of common requests.
	- [Amazon DynamoDB Accelerator (DAX)](https://aws.amazon.com/dynamodb/dax/) is an in-memory cache for DynamoDB. It helps improve response times from single-digit milliseconds to microseconds.

## ☁Security
- [Whitepaper: Introduction to AWS Security](https://docs.aws.amazon.com/whitepapers/latest/introduction-aws-security/welcome.html)
- [AWS Security Blog](https://aws.amazon.com/blogs/security/)

### Shared responsibility model
![[Cloud Practitioner_14_Shared responsibility model.png]]
[Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/) divides into customer responsibilities (commonly referred to as “security ==in== the cloud”) and AWS responsibilities (commonly referred to as “security ==of== the cloud”).

### AWS IAM
[AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/) enables you to manage access to AWS services and resources securely.

- By default, when you create a new IAM user in AWS, it has no permissions associated with it.
- An IAM policy is a JSON document that describes what API calls a user can or cannot make.
- **IAM user**
- **IAM group**
- **IAM policy** is a document that allows or denies permissions to AWS services and resources.
```json
{  
	"Version": "2012-10-17",  
	"Statement": [  
		{  
			"Sid": "MyListBucketStatement",   
			"Effect": "Allow",  
			"Action": "s3:ListBucket",  
			"Resource": [
				"arn:aws:s3:::com.test.demo.file",
				"arn:aws:s3:::com.test.demo.images",  
				"arn:aws:s3:::com.test.demo.notes" 
			]  
		}  
	]  
}
```
- **IAM roles** is an identity that you can assume to gain ==temporary== access to permissions. When someone assumes an IAM role, they abandon all previous permissions that they had under a previous role and assume the permissions of the new role.
- In IAM, [multi-factor authentication (MFA)](https://aws.amazon.com/tw/iam/features/mfa/) provides an extra layer of security for your AWS account.

### AWS Organizations
[AWS Organizations](https://aws.amazon.com/organizations) is as a central location to manage multiple AWS accounts.

- Centralized management of all your AWS accounts.
- You can use the primary account of your organization to consolidate and pay for all member accounts.
- You can group accounts into organizational units, or OUs.
- You can centrally control permissions for the accounts in your organization by using [Service Control Policies (SCPs)](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html).
- An SCP affects all IAM users, groups, and roles within an account, including the AWS account root user.
![[Cloud Practitioner_15_organizational units.png]]

### Compliance
- [AWS Artifact](https://aws.amazon.com/artifact) is a service that provides on-demand access to AWS security and compliance reports and select online agreements. AWS Artifact consists of two main sections: 
	- AWS Artifact Agreements
	- AWS Artifact Reports.
- [Customer Compliance Center](https://aws.amazon.com/compliance/customer-center/) contains resources to help you learn more about AWS compliance.

### Denial-of-service attacks
- [AWS Shield](https://aws.amazon.com/shield) is a service that protects applications against DDoS attacks.
	- AWS Shield Standard protects your AWS resources from the most common, frequently occurring types of DDoS attacks.
	- AWS Shield Advanced is a paid service that provides detailed attack diagnostics and the ability to detect and mitigate sophisticated DDoS attacks.

### Additional security services
- [AWS Key Management Service (AWS KMS)](https://aws.amazon.com/kms) enables you to perform encryption operations through the use of cryptographic keys.
- [AWS Web Application Firewall(WAF)](https://aws.amazon.com/waf) lets you monitor network requests that come into your web applications. AWS WAF works together with Amazon CloudFront and an Application Load Balancer.
- [Amazon Inspector](https://aws.amazon.com/inspector/) helps to improve the security and compliance of applications by running automated security assessments.
- [Amazon GuardDuty](https://aws.amazon.com/guardduty) is a service that provides intelligent threat detection for your AWS infrastructure and resources.

## ☁Monitoring and Analytics
- [AWS Management & Governance Blog](https://aws.amazon.com/blogs/mt/)

### Amazon CloudWatch
- [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) is a web service that enables you to monitor and manage various metrics and configure alarm actions based on data from those metrics.
	- [metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html)
	- [alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)
	- [dashboard](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Dashboards.html)

### AWS CloudTrail
- [AWS CloudTrail](https://aws.amazon.com/cloudtrail/) records API calls for your account. The recorded information includes the identity of the API caller, the time of the API call, the source IP address of the API caller, and more.
- [CloudTrail Insights](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-insights-events-with-cloudtrail.html) allows CloudTrail to automatically detect unusual API activities in your AWS account.

### AWS Trusted Advisor
- [AWS Trusted Advisor](https://aws.amazon.com/premiumsupport/technology/trusted-advisor/) is a web service that inspects your AWS environment and provides real-time recommendations in accordance with AWS best practices.
	![[Cloud Practitioner_16_AWS Trusted Advisor.png]]
	- cost optimization
	- performance
	- security
	- fault tolerance
	- service limits

## ☁Pricing and Support
- [AWS Free Tier](https://aws.amazon.com/free/) enables you to begin using certain services without having to worry about incurring costs for the specified period.
	- Always Free
	- 12 Months Free
	- Trials
- How AWS pricing works
	- Pay for what you use.
	- Pay less when you reserve. Some services offer reservation options that provide a significant discount compared to On-Demand Instance pricing.
	- Pay less with volume-based discounts when you use more.
- [AWS Pricing Calculator](https://calculator.aws/#/) lets you explore AWS services and create an estimate for the cost of your use cases on AWS.
- [consolidated billing](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/consolidated-billing.html) feature of AWS Organizations enables you to receive a single bill for all AWS accounts in your organization. The default maximum number of accounts allowed for an organization is 4, but you can contact AWS Support to increase your quota, if needed.
	- Ability to share bulk discount pricing, Savings Plans, and Reserved Instances across the accounts in your organization.
- In [AWS Budgets](https://aws.amazon.com/aws-cost-management/aws-budgets), you can create budgets to plan your service usage, service costs, and instance reservations.
	- The information in AWS Budgets updates three times a day.
- [AWS Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/) is a tool that enables you to visualize, understand, and manage your AWS costs and usage over time.
- AWS offers four different [Support plans](https://aws.amazon.com/premiumsupport/plans/) to help you troubleshoot issues, lower costs, and efficiently use AWS services.
	- Basic: Basic Support is free for all AWS customers. It includes access to whitepapers, documentation, and support communities.
		- you have access to a limited selection of AWS Trusted Advisor checks.
		- AWS Personal Health Dashboard provides alerts and remediation guidance when AWS is experiencing events that may affect you.
	- Developer
		- Best practice guidance
		- Client-side diagnostic tools
		- Building-block architecture support, which consists of guidance for how to use AWS offerings, features, and services together
	- Business
		- Use-case guidance to identify AWS offerings, features, and services that can best support your specific needs
		- All AWS Trusted Advisor checks
		- Limited support for third-party software, such as common operating systems and application stack components
	- Enterprise
		- Application architecture guidance
		- Infrastructure event management
		- A Technical Account Manager
		- Access to a Technical Account Manager (TAM)
- [AWS Marketplace](https://aws.amazon.com/marketplace) is a curated digital catalog that streamlines your steps to find, deploy and manage third party software running in your AWS architecture.

## ☁Migration and Innovation
- [AWS Cloud Enterprise Strategy Blog](https://aws.amazon.com/blogs/enterprise-strategy/)
- [Modernizing with AWS Blog](https://aws.amazon.com/blogs/modernizing-with-aws/)

### AWS CAF
- [AWS Cloud Adoption Framework (AWS CAF)](https://d1.awsstatic.com/whitepapers/aws_cloud_adoption_framework.pdf) organizes guidance into six areas of focus, called Perspectives.
	- [Whitepaper: An Overview of the AWS Cloud Adoption Framework](https://d1.awsstatic.com/whitepapers/aws_cloud_adoption_framework.pdf)
	- focus on business capabilities
		- Business: helps you to move from a model that separates business and IT strategies into a business model that integrates IT strategy.
		- People
		- Governance
	- focus on technical capabilities
		- Platform: The Platform Perspective of the AWS Cloud Adoption Framework also includes principles for implementing new solutions and migrating on-premises workloads to the cloud.
		- Security: ensures that the organization meets security objectives for visibility, auditability, control, and agility.
		- Operations: helps you to enable, run, use, operate, and recover IT workloads to the level agreed upon with your business stakeholders.

### 6 R's Migration strategies
[migration strategies](https://aws.amazon.com/blogs/enterprise-strategy/6-strategies-for-migrating-applications-to-the-cloud/) that you can implement are:
- Rehosting: involves moving applications without changes. 
- Replatforming: involves making a few cloud optimizations to realize a tangible benefit. Optimization is achieved without changing the core architecture of the application.
- Refactoring/re-architecting: involves reimagining how an application is architected and developed by using cloud-native features.
- Repurchasing: involves moving from a traditional license to a software-as-a-service model.
- Retaining: keeping applications that are critical for the business in the source environment.
- Retiring: the process of removing applications that are no longer needed.

### AWS Snow Family
- The [AWS Snow Family](https://aws.amazon.com/snow) is a collection of physical devices that help to physically transport up to exabytes of data into and out of AWS.
- [AWS Snowcone](https://aws.amazon.com/snowcone) is a small, rugged, and secure edge computing and data transfer device. It features 2 CPUs, 4 GB of memory, and 8 TB of usable storage.
- [AWS Snowball](https://aws.amazon.com/snowball/)
	- **Snowball Edge Storage Optimized** devices are well suited for large-scale data migrations and recurring transfer workflows, in addition to local computing with higher capacity needs.
		-  Storage: 80 TB of hard disk drive (HDD) capacity for block volumes and Amazon S3 compatible object storage, and 1 TB of SATA solid state drive (SSD) for block volumes. 
		- Compute: 40 vCPUs, and 80 GiB of memory to support Amazon EC2 sbe1 instances (equivalent to C5).
	- **Snowball Edge Compute Optimized** provides powerful computing resources for use cases such as machine learning, full motion video analysis, analytics, and local computing stacks.
		- Storage: 42-TB usable HDD capacity for Amazon S3 compatible object storage or Amazon EBS compatible block volumes and 7.68 TB of usable NVMe SSD capacity for Amazon EBS compatible block volumes. 
		- Compute: 52 vCPUs, 208 GiB of memory, and an optional NVIDIA Tesla V100 GPU. Devices run Amazon EC2 sbe-c and sbe-g instances, which are equivalent to C5, M5a, G3, and P3 instances.
	- [AWS Snowmobile](https://aws.amazon.com/snowmobile) is an exabyte-scale data transfer service used to move large amounts of data to AWS. You can transfer up to 100 petabytes of data per Snowmobile

### Innovation with AWS
- [Amazon SageMaker](https://aws.amazon.com/tw/sagemaker/): Quickly build, train, and deploy machine learning models at scale.
- [Amazon Augmented AI(Amazon A2I)](https://aws.amazon.com/tw/augmented-ai/): provide a machine learning platform that any business can build upon without needing PhD level expertise in-house.
- [AWS DeepRacer](https://aws.amazon.com/tw/deepracer/): One of the newest branches of machine learning algorithms, all while having fun in a racing environment. 
- [Amazon Textract](https://aws.amazon.com/tw/textract/): Extracting text and data from documents.
- [Amazon Lex](https://aws.amazon.com/tw/lex/): Build voice and text chatbots

## ☁The Cloud Journey
- [AWS Architecture Blog](https://aws.amazon.com/blogs/architecture)
- [AWS Well-Architected Framework](https://d1.awsstatic.com/whitepapers/architecture/AWS_Well-Architected_Framework.pdf) helps you understand how to design and operate reliable, secure, efficient, and cost-effective systems in the AWS Cloud. It  is based on five pillars: 
	- Operational excellence
	- Security
	- Reliability
	- Performance efficiency
	- Cost optimization
- Benefits of the AWS Cloud
	- Trade upfront expense for variable expense.
	- Benefit from massive economies of scale.
	- Stop guessing capacity.
	- Increase speed and agility.
	- Stop spending money running and maintaining data centers.
	- Go global in minutes.


