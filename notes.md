# AWS CLI

## Configure CLI

Configure AWS cli with access key and access key secret
```
aws configure
```

Show configuration
```
aws configure list
```

## Pagination

By default CLI uses a page size of 1000

* If you see errors when running list commands on large number of resources, default page size might be too high
* Likely to see timed out error

Use --page-size and/or --max-items option

E.g.

--page-size still retrieves all objects

```
aws s3api list-objects --bucket bucket-name --page-size 100
```
--max-items specifies max items to return
```
aws s3api list-objects --bucket bucket-name --max-items 100
```
## Region

Use **--region** parameter to specify a region

# IAM

* Centralized control of AWS account
* Shared access to your AWS account
* Granular permissions
* Identity federation
  
* Multi-Factor Authentication
* Temporary Access for users, devices and services
* Password policies
* Integration with many AWS services
* Compliance - PCI DCSS

## Core Concepts

* Users - End users
* Groups - Collection of users
* Roles - Roles can be assigned to users, applications and services
* IAM Policy - Document that defines one or more permissions. Can be attached to users, groups and roles

## IAM Policy Simulator

* Test the effects of IAM policies before commiting them to production
* Test policies already attached

## Access Key and Access Key Secret

Used for programmatic access

## AWS Limits

* API Rate Limits
  * For Intermittent Errors implement Exponential Backoff
  * For Consistent Errors request API Throttling limit increase
* Service Quotas (Service Limits)
  * Request service limit increase by opening ticket
  * Service Quotas API to request quota increase

### Exponential Backoff

* If you get ThrottlingException
* Already included in the AWS SDK
* Must implement yourself if not using AWS SDK
  * Must only implement if receiving 5xx server errors and throttling
  * Do not implement on 4xx client errors
  
## AWS Credentials Provider Chain

* CLI will look for credentials in this order
  * -- region, --output and --profile
  * ENV variables - AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY and AWS_SESSION_TOKEN
  * CLI credentials file - aws configure ~\.aws\credentials
  * CLI configuration file - aws configure ~\.aws\config
  * Container credentials - ECS
  * Instance profile credentials - EC2

## AWS Signature v4 Signing

* When you call the AWS HTTP API, you sign the request so that AWS can identify you, using your AWS credentials (access & secret key)
* When using the SDK or CLI, the HTTP requests are signed for you
* You should sign using SigV4

SigV4 options:
* HTTP Header (Authorization header)
* Query String

## STS - Security Token Service

* Allows to grant limited and temporary access to AWS resources (up to 1 hour)
* AssumeRole: Assume roles within your account or cross account
* AssumeRoleWithSAML: return credentials for users logged in with SAML
* AssumeRoleWithWebIdentity:
  * Returns creds for users logged in with IdP
  * AWS recommends against using this, use Identity Pools instead
* GetSessionToken: for MFA, from a user or root user
* GetFederationToken: botain temporary creds for a federated user
* GetCallerIdentity: return details about IAM user or role used in the API call
* DecodeAuthorizationMessage: decode error message when an API is denied

### Using STS to assume a role

* Define an IAM role within your account or cross account
* Define which principals can access this IAM role
* Use AWS STS to retrieve credentials and impersonate the IAM role you have access to (AssumeRole API)
* Temporary credentials can be valid between 15min to 1 hour

### STS with MFA

* Use GetSessionToken from STS
* Appropriate IAM policy using IAM conditions
* aws:MultiFactorAuthPresent:true
* GetSessionToken returns:
  * Access ID
  * Secret Key
  * Session Token
  * Expiration Date

### Authorization Model Evaluation of Policies

1. IF explicit DENY; end decision and DENY
2. Else if explicit ALLOW; end decision with ALLOW
3. Else DENY

### IAM Policies & S3 Bucket Policies

* IAM policies are attached to users, roles and groups
* S3 bucket policies are attached to buckets
* When evaluating if an IAM Principal can perform an operation on a bucket, the union of its assigned IAM policies and S3 Bucket Policies will be evaluated

### Dynamic Policies

* Leverage special policy variable ${aws:username}

### Inline vs Managed Policies

* AWS Managed Policy
  * Maintained by AWS
  * Good for power users and administrators
  * Updated in case of new services
* Customer Managed Policy
  * Best Practice, re-usable, can be applied to many principals
  * Version Controller + rollback, change management
* Inline Policy
  * Strict one-to-one relationship between policy and principal
  * Policy is deleted if you delete the IAM principal

### User Permissions to Pass a Role to an AWS Service

* To configure many AWS services, you must pass an IAM role to the service
* The service will later assume the role and perform actions
* You need the IAM permission, iam:PassRole
* iam:GetRole, to view the role being passed
* Roles can only be passed to what their trust allows
* A trust policy for the role that allows the service to assume the role
  * sts:AssumeRole

### AWS Directory Services

AD Domain Services:
* Centralized security managed, create accounts, assing permissions
* Objects are orginazed in trees
* A group of trees is a forest

AWS Directory Services:
* AWS Managed Microsoft AD
  * Create your own AD in AWS, manage users locally, supports MFA
  * Establish trust connections with your on prem AD
* AD Connector:
  * Directory Gateway (Proxy) to redirect to on prem AD, supports MFA
  * Users are managed on the on prem AD
* Simple AD
  * AD compatible managed directory on AWS
  * Cannot be joined with on prem AD

# EC2

* Secure, resizable compute capacity in the cloud
* Pay-as-you-go model

## Pricing options

* On Demand
  * Pay by hour or by second depending on instance type
* Reserved
  * Reserved capacity for one or three years.
  * Up to 72% discount on hourly charge
  * Regional
* Spot
  * Purchase unused capacity at up to 90% discount
  * Prices fluctuate with supply and demand
  * Instances get terminated if pricing exceeds maximum set by user
* Dedicated hosts
  * Physical EC2 server dedicated for your use
  * Most expensive
  * Good for compliance

### On-Demand instances

* Flexible and low cost without any up front payment or long term commitment
* Short term, spiky or unpredictable workloads that cannot be interrupted
* Testing and development

### Reserved instances

* Apps with predictable usage
* Specific capacity requirements
* Pay up front

Types of reserved instances:
* Standard
  * Up to 72% discount
* Convertible
  * Up to 54% discount
  * Option to change instance type to greater or equal value
* Scheduled 
  * Launch within time window that you specify
  * Match capacity reservation to a predictable recurring schedule
  
### Spot instances

* Set maximum price that you're willing to pay
* Instance get terminated or hibernated if price exceeds set maximum
* Apps that are only feasible at very low compute prices
* Urgent need for large amounts of additional computing capacity

### Dedicated hosts

* Compliance
* Software licenses
* On-Demand (hourly) or Reserved (70% discount)

### Savings Plans

* Save up to 72%
* Commit to one or three years in $/hour
* Also includes serverless technologies like Lambda or Fargate

### Pricing Calculator

Explore AWS services anr pricing using the pricing calculator

## EC2 Instance Metadata

* URL is http://169.254.169.254/latest/meta-data
* You can retrieve IAM role but not IAM Policy

## Instance Types

* Hardware
* Capabilities
* Application requirements

Types:
* General purpose
  * Balance of compute, memory, storage and network
* Compute optimized
  * Higher cpu
* Memory Optimized
  * Higher memory
* Accelerated computing
  * Special hardware accelerators
  * GPUs
  * FPGAs
* Storage optimized
  * High storage access

## EBS Volumes

Elastic Block Store

* Storage volumes that you can attach to EC2 instances
* When launching instance, at least 1 EBS volume is attached which contains the OS
  
Features:
* Production and mission critical workloads
* HA
  * Automatically replicated automatically within single AZ
* Scalable
  * Dynamically increase capacity and change volume type

### Regions and AZs

* EBS volume is tied to specific AZ
* To make a volume available outside of the Availability Zone, you can create a snapshot and restore that snapshot to a new volume anywhere in that Region.
* You can copy snapshots to other Regions and then restore them to new volumes there, making it easier to leverage multiple AWS Regions for geographical expansion, data center migration, and disaster recovery.

### Snapshots

Point in time copy of existing EBS volume.

### Encryption

* Uses KMS
* EBS volumes can only be encrypted at creation time. 
* To encrypt an existing EBS volume, create a snapshot then create a new encrypted EBS volume from the snapshot.
* You cannot create an unencrypted volume from an encrypted snapshot.
* If account is enabled for encryption by default, you cannot create an unecrypted volume.
  
### EBS types

#### General Purpose SSD (gp2)
* 3 IOPS per GiB
* Max 16000 IOPS per volume
* gp2 volumes < 1 TB can burst up to 3000 IOPS
* Good for boot volumes or dev and test

#### General Purpose SSD (gp3)
* Baseline of 3000 IOPS for any volume size
* Max 16000 IOPS per volume
* 20% cheaper than gp2

#### Provisioned IOPS SSD (io1)
* Up to 64000 IOPS per volume 
* 50 IOPS per GiB
* Most performant and expensive option
* For IO intensive applications

#### Provisioned IOPS SSD (io2)
* Higher durabality and more IOPS
* io2 is the same price as io1
* Up to 64000 IOPS per volume 
* 500 IOPS per GiB
* 99.999% durability instead of up to 99.9%

#### Provisioned IOPS SSD io2 Block Express
* SAN network in the cloud
* Highest performance, sub-millisecond latency
* EBS Block Express architecture
* 4x throughput, IOPS and capacity of regular io2 volumes
* Up to 64TB, 256000 IOPS per volume
* 99.999% durability

#### Througput optimized HDD (st1)
* Low cost HDD
* 40MB/s per TB
* Bursts up to 250 MB/s per TB
* Max throughput of 500 MB/s per volume
* Frequently-accessed, throughput intensive workloads
* Cannot be a boot volume

#### Cold HDD (sc1)
* Lowest cost option
* 12 MB/s per TB
* Burst up to 80MB/s
* Max throughput of 250 MB/s
* Cannot be a boot volume
  
### IOPS vs Throughput

IOPS:
* Measures number of read and write operations per seconds (IOPS)
* Important metric for quick transactions, low latency apps
* Ability to read and write quickly

Throughput:
* Measures number of bits read or written per second (MB/s)
* Important metric for large dataset, large IO sizes, complex queries
* Ability to deal with large datasets

## EC2 CLI


```
```

## EFS

* Serverless, fully elastic file storage to share files without provisioning or managing storage capacity and performance

### Storage Classes

* EFS Standard
  * Frequently accessed data requiring the highest durability and availability.
* EFS Standard Infrequent Access (IA)
  * Long lived, infrequently accessed data requiring the highest durability and availability.
* EFS One Zone
  * Frequently accessed data that doesn’t require highest levels of durability and availability.
* EFS One Zone Infrequent Access IA
  * Long lived, infrequently accessed data that doesn’t require highest levels of durability and availability.
  
## Elastic Load Balancer

A load balancer distributes network traffic across a group of servers.

Types:
* Application Load Balancer
  * HTTP and HTTPS
  * Websocket
* Network Load Balancer
  * TCP and high performance
  * TLS
  * UDP
* Classic Load Balancer
  * HTTP/HTTPS and TCP
* Gateway Load Balancer
  * Virtual Applicances
  * Virtual Firewalls
  * IDS/IPS systems
  * IP Protocol

### Types
#### Application Load Balancer

* Used for load balancing HTTP/HTTPS
* Operate at layer 7 (application layer)
* Supports advanced request routing based on the HTTP header

Routing tables to different target groups:
* Based on path in url
* Based on hostname in url
* Based on Query String or Headers
* Port mapping feature to redirect to dynamic port in ECS
* Great fit for microservices and container based apps
* Based on HTTP method

Target groups:
* EC2 instances - HTTP
* ECS tasks - HTTP
* Lambda functions - HTTP request is translated into JSON  event
* IP addresses - must be private IPs

Other features:
* ALB can route to multiple target groups
* Health checks are at target group level
* Fixed hostname
* True IP of client is inserted in the header X-Forwarded-For
* Also possible to get port (X-Forwarded-Port) and protocol (X-Forwarded-Proto)

#### Network Load Balancer

* High performance (handles millions of request per seconds)
* Load balances TCP traffic
* Operates at layer 4 (network layer)
* Capable of handling millions of requests per second while mainting ultra-low latencies (~100ms vs 400ms ALB)
* Most expensive option
* Has one static IP per AZ and supports assigning Elastic IPs
* TCP/UDP

Target groups:
* EC2 instances
* IP addresses (must be private IPs)
* Application Load Balancer
* Health Checks support TCP, HTTP, HTTPS
* No security group

#### Classic Load Balancer

* Supports layer 7 features, such as X-Forwarded-For and sticky sessions
* Supports layer 4 load balancing

### Gateway Load Balancer

* Virtual Applicances
* Virtual Firewalls
* IDS/IPS systems
* IP Protocol
* Operates at Layer 3 - IP packets

Functions:
* Transparent Network Gateway - Single entry/exit for all traffic
* Load balancer
* GENEVE protocol on port 6801

Target Groups:
* EC2 instances
* IP addresses

### X-Forwarded-For header

Identifies originating IP address of client connecting through a load balancer

### Sticky Sessions

Requests from a specific client are forwarded to the same target

* Uses a cookie with expiration date you control

LBs:
* Classic LB
* Application LB
* NLB

Cookies:
* Application-based cookie
  * Custom cookie
    * Generated by target
    * Can include any custom attributes
    * Must be specified individually for each target group
    * Name must not be AWSALB, AWSALBAPP or AWSALBTG
  * Application cookie
    * Generated by LB
    * Name must not be AWSALBAPP
* Duration based cookie
  * Generated by LB
  * Name is AWSALB for ALB, AWSELB for CLB

### Cross-Zone Load Balancing 

With cross-zone:
* Each load balancer instance distributes evenly across all registered instance in an AZ

Without cross-zone:
* Requests are distribiuted in the instances of the node of the Elastic Load Balancer

Enabled:
* ALB
  * enabled by default
  * Disabled at Target Group level
  * No charges for inter AZ data
* NLB & GLB
  * Disabled by default
  * You pay charges for inter AZ data
* CLB
  * Disabled by default
  * No charges for inter AZ data

### SSL/TLS

* SSL certificate allows traffic between clients and LB to be encrypted in transit

* LB uses x.509 certificates
* You can manage certs via ACM
* You can upload your own certficiates

HTTPS listener:
* Specify default certificate
* Add optional list of certs to support multiple domains
* Clients can use SNI to specify hostname they react
* Ability to specify a security policy to support older versions of TLS/SSL

Classic Load Balancer:
* Supports only one SSL certificate
* Must use multiple CLB for multiple hostname with multiple SSL certs
  
Application Load Balancer:
* Supports multiple listeners with multiple SSL certificates
* Uses Server Name Indication 
  
Network Load Balancer:
* Supports multiple listeners with multiple SSL certificates
* Uses Server Name Indication 

#### Server Name Indication (SNI)

* Solves problem of loading multipole SSL certificates onto one web server
* New protocol 
* Requires client to indiciate the hostname of the target server in the initial SSL handshake
* The server will then find the correct cert, or return the default one
* Only works for ALB & NLB, Cloudfront

### Connection draining

Feature naming:
* Connection draining - CLB
* Deregistration Delay - ALB & NLB

* Time to complete in-flight requests while the instance is deregistering or unhealthy
* Stops sending new requests to the EC2 instance which is deregistering
* Between 1 to 3600 seconds
* Can be disabled
* Set to a low value if you requests are short

### Auto Scaling Group

* Scale out (add EC2 instances) to match increased load
* Scale in (remove EC2 instances) to match decreased load
* Ensure we have min and max number of EC2 instances running
* Automatically register new instances to a LB
* Re-create EC2 instances to match scaling policy

#### Auto Scaling Group attributes

* Launch Template
  * AMI + Instance tyoe
  * EC2 user data
  * EBS volumes
  * Security groups
  * SSH Key pair
  * IAM roles
  * Network & subnet info
  * LB info
* Capacity
  * Minimum capacity
  * Desired capacity
  * Maximum capacity
  
#### Cloudwatch Alarms

* It's possible to scale an ASG based on CloudWatch Alarms
* An alarm monitors a metric
* Metrics, such as Average CPU, are computed for the overall ASG instances
* Scale-out and scale-in policies
  
#### Dynamic Scaling Policies

* Target Tracking Scaling
  * Simple
  * E.g. Avg ASG CPU ~= 40%
* Simple / Step Scaling
  * Take action when reaching threshold
* Scheduled Actions
  * Anticipate scaling based on known usage patterns
* Predictive Scaling
  * Continually forecast load and schedule scaling ahead
  
Good metrics to scale on:
* CPUUtilization: Average CPU usage across instances
* RequestCountPerTarget
* Average Network In / Out: if application is network bound
* Any custom metric

Scaling cooldown:
* After scaling activity happens, you're in cooldown period (default 300s)
* During cooldown period, ASG will not launch or terminate additional instance
* Advice: use a ready-to-use AMI to reduce config time in orde to be serving request faster and reduce cooldown period

#### Auto Scaling - Instance Refresh

* Update launch template and then re-create all EC2 instances
* New Launch TEmplate -> StartInstanceRefresh
  * Based on min health percentage
  * Specify warm-up time


### Common Errors

* 504 - Gateway Timeout
  * Target failed to respond
  * LB could not establish connection to the target

### Access Logs

Elastic Load Balancing provides access logs that capture detailed information about requests sent to your load balancer. 

Each log contains information such as the time the request was received, the client's IP address, latencies, request paths, and server responses. You can use these access logs to analyze traffic patterns and troubleshoot issues.

### Request Tracing

When the load balancer receives a request from a client, it adds or updates the **X-Amzn-Trace-Id** header before sending the request to the target. Any services or applications between the load balancer and the target can also add or update this header.

You can use request tracing to track HTTP requests from clients to targets or other services. If you enable access logs, the contents of the X-Amzn-Trace-Id header are logged

### Health Checks

* Crucial for load balancers
* Enable the LB to know if instance is available to reply to requests
* Health check is done on a port and route
* If the response is not 200 the target will be considered unhealth and not used

## RDS

Relational Database Services

Features:
* Generally used for OLTP (transactional processing)
* Multi-AZ
* Failover capability
* Automated backups

Supports:
* MySql
* MS Sql Server
* PostgreSQL
* Oracle 
* MariaDB
* AWS Aurora (MySql/PostgreSQL compatible)
  
### OLTP vs OLAP

OLTP:
* Processes data from transactions in real-time (payments...)
* OLTP is about data processing and completing large number of small transactions in real-time

OLAP:
* Processes complex queries to analyze large sets of historical data
* RDS is not suited for analyzing large amounts of data

### Deployment options

#### Production

* Multi-AZ DB Cluster
  * 1 primary DB instance and 2 read-only standby instances
  * Each instance in a different AZ
  * Provides HA, redundancy and increased read capacity
* Multi-AZ DB Instance
  * 1 primary and 1 read-only standby instance 
  * Each instance in a different AZ
  * Provides HA and redundancy but no increased read capacity
* Single DB Instance

### Storage autoscaling

* Enables storage to increase when passing threshold
* Enabled by default

### Database authentication

* Password authentication
* Password and IAM database authentication
* Password and Kerberos authentication

### Backups

#### Database snapshots
* Manual, ad-hoc, user initiated
* Provides snapshot of storage volume attached to DB
* No retention period

#### Automated backups
* Enabled by default
* Only for InnoDB
* Takes full daily snapshot and transaction logs
* Transactions logs are used for replays between now and latest snapshot
* Point-In-Time recovery down to second within retention period of 1-35 days
* Stored in S3

#### Restoring Database

Restored version will always be a new RDS instance with a new DNS endpoint

### Security

* IAM auth
* Security groups
* No SSH except on RDS custom
* Audit Logs can be enabled and sent to CloudWatch Logs

#### Encryption at Rest

* Enabled by default at creation time
* Uses KMS (AES-256)
* Includes all underlying storage, automated backups, snapshots, logs and read replicas
* Cannot be enabled on an unencrypted RDS instance
  * To encrypt an unencrypted DB, take an unencrypted snapshot, create an encrypted snapshot and restore to a new encrypted DB
* Master needs to be encrypted for replicas to be encrypted

#### Encryption in Transit
 
* TLS by default, use AWS TLS root certificates client-side

### Deletion protection

* Disabled by default

### Multi-AZ and Read Replicas

#### Multi-AZ

Exact standby copy of database in another AZ

* Provides resilience, not scaling
* Supported by all RDS types
* RDS will automatically failover to standby without intervention
* For DR, not for increasing performance
* SYNC replication
* One DNS name - automatic failover
* Read replicas can be setup as Multi-AZ for DR

From single to multi AZ
* Snapshot is taken
* DB restored from snapshot
* Sync established between the DBs

#### Read Replica

Read-only copy of your primary database

* Provides read performance
* Can be located in the same or another AZ as primary DB
* Can be cross-region
* Each read replica has its own DNS name
* Can be promoted to be their own DB
  * Breaks replication
* Requires automatic backup
* Up to 15 replicas are supported
* Replication is ASYNC, reads are eventually consistent
* Only SELECT statements
* Network cost in same region is free
* Cross region incurs replication fee

### RDS Proxy Pooling

RDS Proxy pools and shares DB connections

* Serverless and scales automatically through pooling and sharing DB connections
* Preserves application connections (during failover)
* Detects failover and routes requests
* Deployable over Multi-AZ for protection
* Up to 66% faster failover times
* Enforce IAM authentication for DB and securely store credentials in AWS Secrets Manager
* Never publicly accessible (must be accessed from VPC)

## Aurora

* Proprietary tech from AWS
* Postgres and MySql are both supported as Aurora DB
* Cloud Optimized
* Aurora storage automatically grows in increments
* Can have to up to 15 replicas
* Failover in Aurora is instantaneous. HA native
* Cost is 20% more than RDS
  
### High Availabilty and Read Scaling

* 6 copies across 3 AZ
  * 4 of 6 for writes
  * 3 of 6 for reads
  * self healing with peer to peer replication
  * Shared Storage is striped across 100s of volumes
  * Storage is auto expanding
* One instance takes writes (master)
* Automated failover for master in less than 30s
* Master + up to 15 read replicas serve reads
* Support for Cross Region Replication

Writer Endpoint:
* Pointing to the master

Reader Endpoint:
* Connection load balancing

## ElastiCache

Stores frequently accessed data in in-memory cache

* In-Memory Cache (Key Value)
* Improves database performance
* Great for read-heavy database workloads
* Needs additional coding effort

Types:
* Memcached
  * Basic object caching
  * Scales horizontally
  * No persistance, Multi-AZ or failover
  * NO HA
  * Multi-node for partitioning of data (sharding)
* Redis
  * Enterprise features like persistance, replication, Multi-AZ and failover
  * Supports sorting, ranking and complex data types like lists and hash
  * Supports Sets and Sorted Sets
  * Read replicas to scale reads and HA
  * Backup and restore features
  
ElastiCache is a good choice if DB is read-heavy and not prone to frequent changing

### ElastiCache for Redis

| |Single Node	|Cluster Mode Disabled	|Cluster Mode Enabled|
|--|--|--|--|
|Replication?	 |No	|Yes (up to 5 replicas per node)|	Yes(up to 5 replicas per node)|
|Data Partitioning?|	Yes	|No (single shard)	|Yes (up to 90 shards)|
|Scaling	|Change node type (vertical scaling)	|Change node type(vertical scaling)	| Add/remove shards and |rebalance (horizontal scaling)|
|Multi-AZ?	|No|	Optional with at least 1 replica|	Required|

While using Redis with cluster mode enabled, there are some limitations:
* You cannot manually promote any of the replica nodes to primary.
* Multi-AZ is required.
* You can only change the structure of a cluster, the node type, and the number of nodes by restoring from a backup.

### Caching Strategies

* Data may be out of date, eventually consistent
* For slowly changing data, few keys are frequently needed
* Good for key-value data or aggregated results

#### Lazy Loading / Cache-Aside / Lazy Population

* Data is retrieved and loaded into cache when cache miss

Pros: 
* Only requested data is cached
* Node failures are not fatal (only longer warm up time)
  
Cons:
* Cache miss results in 3 round trips
* Stale data

#### Write-Through

* Add or Update cache when database is updated

Pros:
* Data is never stale
* Reads are quick
* Write penalty vs Read penalty (each write requires 2 calls)

Cons:
* Missing data until DB is updated
* Mitigation is to implement Lazy Loading as well
* Cache churn - a lot of data will never be read

### Cache Evictions and TTL

Cache eviction can occur in three ways:
* Delete item explicitly
* Item is evicted becayse memory is full and not recently used (LRU)
* Set a TTL

## MemoryDB for Redis

In-memory database

* Massively scalable (> 100 TB)
* Highly available
  * Multi-AZ
  * Transaction log for recovery and durability
* Can be used as primary database
* Ultra-fast performance 
  * Supports > 160 million requests per second
  * Microsecond read and single digit millisecond write latency  

## Systems Manager Parameter Store

Secrets and configuration data parameter store

* Store Confidential Information
* Store values as Plain Text or Encrypted
* Integration with EC2, CloudFormation, Lambda, CodeX

Standard:
* Up to 10000 parameters
* Parameter value up to 4 KB

Advanced:
* \> 10000 parameters
* Up to 8KB parameter size

SecureString data type:
* Encrypted with KMS

## Secrets Manager

* Centrally manage Secrets used to accesss resources inside and outside AWS
* Rotate secrets without code deployment
* Secure secrets with fine-grained control

Secret types:
* RDS Database
* Redshift Cluster
* DocumentDB
* API Keys

### Customer Master Key (CMK)

* Logical representation of Master Key
* CMKs can only encrypt up to 4KB of data 

### Automatic Rotation

* Every 30, 60, 90 or custom # of days
* Create new or use existing Lambda for secret rotation

### Regional Replication

* Secrets can be replicated to another region

## EC2 Image Builder

Automates the process of creating and managing images

* Create EC2 Images
* Simple to use
* Validate your image

Base OS -> Define Software to Install -> Test -> Distribute

### Image Pipeline

Defines configuration and end-to-end process of building images.

Includes:
* Image Recipe
* Distribution
* Test settings

Image Recipe:
* Source Image - Base OS
* Build components - Software components to include in the image

AMIs are region bound

### Using AMIs in different regions

AMIs can be copied to another region

* Encrypted AMI to unencrypted AMI is not supported
* By default, encryption options are retained

# VPC

## VPC

* Private network to deploy your resources
* regional
* Can span multiple AZs

## Subnets

Allow to parition network using CIDR, e.g. 10.0.0.0/24

### Public subnet
* Accessible through the internet
* Has route to Internet Gateway

### Private subnet
* Not accessible through the internet

### Route Tables

* To define access to internet and between subnets

## IGW

* To connect subnets to the internet

## Nat

* To access internet from within private subnet
* Private subnet with route to NAT

### Nat Gateway

* AWS managed

### Nat instance

* Self managed

## NACL

Network Access Control List

* Firewall that controls traffic from and to subnet
* Can have ALLOW and DENY rules
* Are attached at Subnet level
* Rules only include IP addresses
* Default NACL includes all in and out traffic
* Stateless, return traffic has to be allowed
  
## Security Groups

* Firewall that controls traffic to and from ENIs/EC2 instances
* Rules include IP addresses and other security groups
* Stateful, return traffic is automatically allowed

## VPC Flow Logs

* Captures info about IP traffic going through your interfaces
* VPC Flow Logs
* Subnet Flow Logs
* ENI Flow Logs
* ELB, ElastiCache, RDS
* Can go to S3, Cloudwatch, Kinesis Firehose

## VPC Peering

* Connect two VPCs privately using AWS network
* Must not have overlapping CIDR
* Not transitive, it must be established for each VPC

## Endpoints

* Allows to connect to AWS services using private network instead of through public internet
* Enhanced security and lower latency
* Only used from within VPC

Gateway Endpoints:
* S3
* DynamoDB

Interface Endpoints:
* Everything Else
  
## Site-To-Site VPN

* To connect on-prem network to VPC
* Automatically encrypted
* Goes over public internet

## Direct Connect (DX)

* To connect on-prem network to VPC
* Reduces network cost, increases bandwith throughput and provides a more consistent network experience

# Route 53

AWS DNS service

Allows to map domain names to IP addresses, EC2 instances, ALBs, S3 buckets...

Supported record types:
* A record type - maps hostname to IPv4
* AAAA record type - maps hostname to IPv6
* CAA record type
* CNAME record type - maps hostname to another hostname
* DS record type
* MX record type
* NAPTR record type
* NS record type - Name server of the zone
* PTR record type
* SOA record type
* SPF record type
* SRV record type
* TXT record type

## Hosted zone

* Container for DNS records that define how to route traffic to a domain and it's subdomains

* Public hosted zone - contains records that specify how to route traffic on the Internet
* Private hosted zone - contains records that specify how you route traffic within one or more VPCs

## Records TTL

* Records are cached for duration specified by the TTL
* High TTL
  * Less traffic
  * Possibly outdated records
* Low TTL
  * More traffic
  * No outdated records
  * Easy to change records
* Mandatory for every record type except ALIAS

## CNAME vs ALIAS

* AWS Resources expose an AWS hostname

CNAME:
* Points a hostname to any other hostname
* Only for non root domain

ALIAS:
* Points a hostname to an AWS Resource
* Works for ROOT and NON ROOT domain
* Free of charge
* Native heath check
* Automatically recognizes changes in the resources IP addresses
* Always of type A/AAAA
* Cannot set TTL
* Targets:
  * ELBs
  * CloudFront distributions
  * API Gateway
  * Elastic Beanstalk
  * S3 Website
  * VPC Interface Endpoint
  * Route 53 record in same hosted zone
* Cannot set ALIAS for EC2 instance

## Routing Policies

* Define how Route 53 responds to DNS queries

Policies:
* Simple
* Weighted
* Failover
* Latency Based
* Geolocation
* Multi-Value Answer
* Geoproximity
* IP Based

### Simple

* Route traffic to a single resource
* Can specify multiple values in the same record
* If multiple values are returned, a random one is chosen by the client
* When ALIAS enabled, specify only one AWS resource
* Cannot be associated with health check

### Weighted

* Control % of requests that go to a specific resource
* DNS records must have the same name and type
* Can be associated with Health Checks
* For load balancing between regions, testing new versions
* Assign a weight of 0 to stop sending traffic to a resource
* If all resources have a weight of 0 then all records will be returned equally

### Latency Based

* Redirect to resource that has the least latency
* Latency is based on traffic between users and AWS regions
* Can be associated with Health Checks (has failover capability)

### Failover 

* Associate primary record with a Health Check (mandatory)
* Route 53 responds with primary record if healthy else it returns secondary record

### Geolocation

* Based on user location
* Most precise location is selected
* Should create a Default record
* Can be associated with Health Checks
* For website localization, restrict content distribution, load balancing,...

### Geoproximity 

* Route traffic to your resources based on the geographic location of users and resources
* Ability to shift more traffic to resources based on the defined bias
* To change size of geographic region, specify bias values
  * To expand (1 to 99) - more traffic
  * To shrink (-1 to -99) - less traffic
* Resources can be:
  * AWS resources (specify AWS region)
  * Non-AWS resources (specify Latitude and Longitude)
* You must use Route 53 Traffic Flow (advanced) to use this feature

### Traffic Flow

* Visual configuration tool to simplify large and complex configurations

### IP Based

* Based on clients IP addresses
* Provide a list of CIDRs for your clients and corresponding endpoints/locations
* Optimize perfomance, reduce network costs

### Multi-Value Answer

* Route traffic to multiple resources
* Returns multiple values/resources
* Can be associated with Health Checks
* Up to 8 healthy records are returned for each quer
* Not a substiture for having an ELB
  
## Health Checks

* HTTP Health Checks are only for public resources
* Automated DNS Failover
  * Monitor an endpoint
  * Monitor other health checks (calculated health checks)
  * Monitor CloudWacth alarms
* Integrated with CloudWatch Metrics

### Monitor an Endpoint

* HTTP, HTTPS, TCP
* If > 18 % report healthy then endpoint is considered healthy
* Pass only when endpoint responds with 2** or 3** status codes
* Can be setup to pass / fail based on the text in first 5120 bytes of response
* Configure router/firewall to allow incoming requests from Route 53 health checkers
* +- 15 global health checkers will check endpoint health

### Calculated Health Checks

* Combine result of multiple health checks into a single health check
* You can use OR, AND or NOT
* Can monitor 256 Child Health Chcks
* Specify how many health checks are needed to pass

### Private Hosted Zone

* Health Checkers are outside VPC
* Create a CloudWatch Metric and associate CloudWatch Alarm then associate Health Check
     
# Beanstalk

Developer problems on AWS:
* Managing infrastructure
* Deploying Code
* Configuring all the database, load balancers,...
* Scaling concerns
* Most web apps have the same architecture
* Consistency across apps

Beanstalk:
* Developer centric view of deploying apps on AWS
* Uses different components: EC2, ASG, ELB, RDS,...
* Managed service
  * Handles capacity provision, load balancing, health monitoring, instance config
  * App code is only responsibility of the developer
* Full control over config
* Pay only for underlying resources

Components:
* Application:
  * Version: iteration of the code
  * Environment
    * Collection of AWS resources running an app version
    * Tiers: 
      * Web Server Environment Tier (public with ELB and ASG)
      * Worker Environment Tier (private with SQS and ASG)
    * Create multiple environments (dev, prod)

Deployment Modes:
* Single instance (dev)
* HA with load balancer (prod)

## Deployment Options

* All at once
* Rolling 
* Rolling with additional batches instances
* Immutable
* Traffic splitting
* Blue/Green

### Warning

Some policies replace all instances during the deployment or update. This causes all accumulated Amazon EC2 burst balances to be lost. 

It happens in the following cases:
* Managed platform updates with instance replacement enabled
* Immutable updates
* Deployments with immutable updates or traffic splitting enabled

### All at once

* All instances are updated at once
* Fastest
* Downtime
* Great for quick iterations in dev
* No additional cost
* Manual redeploy

### Rolling

* Application will be running below capacity
* Can set bucket size
* Deployment will be executed by bucket size
* Application is running both versions simultaneously
* No additional cost
* Can be long deployment
* No downtime
* Manual redeploy
* Batch size can be % or fixed size

### Rolling with additional batches

* Application will be running at capacity
* Can set bucket size
* Deployment will be executed by bucket size
* Application is running both versions simultaneously
* Can be long deployment
* Additional batches are added during deployment to ensure capacity
* Additional batches are removed after deployment
* No downtime
* Good for prod
* Small additional cost
* Manual redeploy
* Batch size can be % or fixed size

### Immutable

* Zero downtime
* New code is deployed to new instances on a temporary ASG
* High cost, double capacity
* Longest deployment
* Quick rollback (terminate new ASG)
* Great for prod
* Terminate new instances
  
### Traffic Splitting

* Canary Testing
* New version is deployed to a temporary ASG with the same capacity
* Small % of traffic is sent to temporary ASG for a configurable amount of time
* Deployment health is monitored
* If there is a failure an automated rollback is triggered (very quick)
* Zero downtime
* New instances are migrated from temporary ASG to original ASG
* Rollback: Reroute traffic and terminate new instances

### Blue/Green

* Not a direct feature of Beanstalk
* Zero downtime and release facility
* Create a new stage environment and deploy v2 there
* The new environment (green) can be validated independently and rolled back
* Route 53 can be setup using weighted policy to redirect traffic
* Swap URLs using Beanstalk 
* Rollback: Swap URLs

## Lifecycle policy

* At most 1000 app versions can be stored
* Lifecycle policies allow to phase out old app version
  * Based on time
  * Based on space
* Currently used versions won't be deleted
* Option not to delete the source bundle in S3 to prevent data loss

## Beanstalk Extensions

* Zip file containing our code must be deployed to Beanstalk
* All parameters in UI can be configured with code using files
* Requirements:
  * .ebextensions/ dir in the root of the source code
  * YAML/JSON
  * .config extensions (e.g. logging.config)
  * Able to modify some default settings using options_settings
  * Ability to add resources such as RDS, ElastiCache, DynamoDB
* Resources managed by .ebextensions get deleted if environment goes away

### Beanstalk & CloudFormation

* Beanstalk relies on CloudFormation
* You can define CloudFormation resources in your .ebextensions to provision anything you want

## Beanstalk Cloning

* Clone an environment with the exact same configuration
* Useful for deploying a test version
* All resources and configuration are preserved
* RDS database type (data is not preserverd)
* Env variables
* After cloning, you can change settings

## Beanstalk Migration

Load Balancer:
* After creating Beanstalk environment, you cannot change the type
* To migrate
  * Create new environment with same config except LB (no cloning)
  * Deploy app to new environment
  * Shift traffic (CNAME swap)

RDS:
* Can be provisioned with Beanstalk (good for dev/test)
* Not good for prod as it is tied to environment lifecycle
* To decouple:
  * Create snapshot of RDS DB
  * Protect RDS DB from deletion 
  * Create new Beanstalk environment, without RDS, point app to existing RDS DB
  * Perform a CNAME swap
  * Terminate old environment
  * Delete CloudFormation stack

# S3

Simple Storage Service

* Provides secure, durable, scalable object storage
* Simple to use
  
Object based storage:
* Manages data as objects rather than in file systems or data blocks
* Cannot be used to run OS or DB

Basics:
* Unlimited storage
* Objects up to 5TB in size
* Files are stored in buckets
* Universal namespace shared by ALL aws accounts
* When uploading file to S3, you will receive an HTTP 200 code
* Max file size with a single PUT is 5GB

Key-Value store:
* Key: name of the object
* Value: data itself
* Version ID: import for storing multple version of the same object
* MEtdata: data about data you're storing, e.g. content-type

Highly available and highly durable:
* Availability: 99.95% - 99.99%
* Durability: 99.99999999999%

Tiered storage:
* Range of storage classes

Lifecycle management:
* Define rules to automatically transition objects

Versioning:
* All version of an object are stored

Security:
* Server-side encryption
* ACLs
* Bucket policies

## Storage Classes

* S3 Standard
* S3 Standard IA
* S3 One-Zone IA
* S3 Glacier
* S3 Glacier Deep Archive

### S3 Standard

* Highly available and highly durable
* Data stored across >= 3 AZs
* 99.99%
* 99.99999999999%
* Designed for frequent access
* General purpose

### S3 Standard Infrequent Access (S3-IA)

* For infrequently accessed data
* Rapid access
* Pay to access data
* Long term storage, backups...
* Minimum storage duration: 30 days
* 99.99%
* 99.99999999999%

### S3 One-Zone IA

* Same as S3 IA but data is stored redundantly within single AZ
* 20% less cost S3 IA
* For non-critical data
* 99.95% available

### S3 Glacier

* Cheap storage
* For data that is very infrequently accessed
* For data archiving
* Retrieval times from 1m to 12h
* Min 90 days storage duration
* Sub-classes:
  * Flexible retrieval
  * Instant retrieval

### S3 Deep Glacier

* Default retrieval time: 12h
* 180 days minimun storage duration
* For rarely accessed data

## S3 Intelligent Tiering

2 tiers:
* Frequent access
* Infrequent access

Automatically moves data to most cost-effective tier for a small fee

## Securing Buckets

* All new created buckets are private by default
* Only bucket owner has access
* No public access by default

### Bucket policies

* Applied at bucket level
* Permissions apply to all objects in bucket
* Usefull for groups of files
* Written in JSON

#### Read Access

```
{

“Effect”: “Allow”,

“Principal”: “*”,

“Action”: “s3:GetObject”,

“Resource”: “arn:aws:s3:::YOURPUBLICBUCKET/*”

}
```

#### Full Access

```
{

“Effect”: “Allow”,

“Principal”: “*”,

“Action”: “s3:*,

“Resource”: “arn:aws:s3:::YOURPUBLICBUCKET/*”

}
```

#### Encryption in-transit

```
{

“Action”: “s3:*”,

“Effect”: “Deny”,

“Principal”: “*”,

“Resource”: “arn:aws:s3:::YOURBUCKETNAME/*”,

“Condition”: {

“Bool”: { “aws:SecureTransport”: false }

}
}
```
#### Server-Side Encryption

### Bucket ACLs

* Applied at object level
* Grant access to specific objects
* Fine grained control

### Access Logs

* Not enabled by default
* Logs all requests made to the S3 bucket

## S3 Encyrption

* Security best practice
* Data needs to be protected

Options:
* Encryption in transit 
  * SSL/TLS
  * HTTPS
* Encryption at rest - Server Side (SSE)
  * SSE-S3 
    * AES 256
    * S3 managed keys
    * Enabled by default
  * SSE-KMS
    * KMS managed keys
  * SSE-C
    * Customer managed keys  
* Encryption at rest - Client Side

### SSE-S3

* Encryption using keys handled, managed and owned by AWS
* Object is encrypted server-side
* Encryption type is AES-256
* Must set header "x-amz-server-side-encryption":"AES256"
* Enabled by default for new buckets & new objects

### SSE-KMS

* Encryption using keys handled and managed by AWS KMS
* KMS advantages: user control + audit key usage using CloudTrail
* Object is encrypted server-side
* Must set header "x-amz-server-side-encryption":"aws:kms"

Limitiations:
* Might be impacted by KMS limits
* When you upload, it calls GenerateDataKey KMS API
* When you download, it calls Decrypt KMS API

### SSE-C

* Encryption using keys handled and managed by the customer
* Encryption key is not store
* HTTPS must be used
* Encryption key must be provided in HTTP headers

### Client-Side Encryptio

* Use client libraries like AWS Client-Side Encryption Library
* Clients must encrypt and decrypt themselves when uploading and downloading from S3
* Customer manages keys and encryption cycle

### Encryption in transit (SSL/TLS)

* Two endpoints exposed:
  * HTTP
  * HTTPS
* Force Encryption in Transit 
  * Policy "aws:SecureTransport":"false"
  
## CORS

* Cross-Origin Resource Sharing is enabled on buckets
* Needs to be configured to allow resources in one bucket to access resources in another bucket

## MFA Delete

* Required to permanently delete an object version
* Required to suspend versioning on the bucket
* Versioning must be enabled
* Only bucket owner (root account) can enable/disable MFA delete

## Lifecycle Rules

* You can transition objects between storage classes
* Moving objects can be automated using lifecycle rules

Transition Actions
* Configure objects to transition to another class

Expiration actions
* Configure objects to expire (be deleted) after a specified time

Rules:
* Can be created for a certain prefix
* Can be created for certain object tags

### S3 Storage Class Analysis

* Recommandations for Standard and Standard IA
* Report is updated daily
* 24 to 48 hours to start seeing data analysis

## Event Notifications

* s3:ObjectCreated, s3:ObjectRemoved, s3:ObjectRestore, s3:Replication,...
* Object name filtering
* Can create as many events as desired
* Typically delivered in seconds but can take minutes
* Policy needs to be attached to target (Resource Access Policy)
* All events end up in EventBridge

## S3 Performance

* Automatically scales to high request rates, latency 100-200ms
* Achieve at least 3500 PUT/COPY/POST/DELETE or 5500 GET/HEAD per second per prefix
* No limits to number of prefixes in a bucket
* Spread reads across all four prefixes evenly, you can 22000 requests for HEAD/GET
* Multi-Part upload
  * Recommend for files > 100 MB
  * Mandatory for files > 5GB
  * Can help parallelize uploads
* S3 Transfer Acceleration
  * Compatible with multi-part upload

S3 Byte-Range fetches:
* Parallelize GETs by requesting specific byte ranges
* Better resilience in case of failures
* Can be used to speed up downloads
* Can be used to retrieve only part of object

## S3 SELECT & Glacier Select

* Retrieve less data using SQL by performing server-side filtering
* Can filter by rows & columns (simple SQL statements)
* Less network transfer, less CPU cost client-side

## S3 Object Tags & Metadata

* User-Defined Object Metadata
  * When uploading object you can specify metadata
  * Name-value pairs
  * Must begin with x-amz-meta-*
  * Keys are stored in lowercase
  * Can be retrieved while retrieving the object
* Object tags
  * Key-value pairs
  * Can be used for fine-grained permissions
  * Useful for analytics purposes

## S3 Access Logs

* All requests made to s3 will be logged into another S3 bucket
* Target bucket must be in the same region
* Never set your logging bucket to be the monitored bucket (logging loop)

## Pre-Signed URLS

* Generate pre-signed URLS using Console, CLI or SDK
* URL expiration
  * Console - 1 min to 720 min
  * CLI - default 3600s, up to 604800s
* Users given a pre-signed URL inherit permissions of the user that generated the URL for GET/PUT

## Consistency Model

Amazon S3 provides strong read-after-write consistency for PUT and DELETE requests of objects in your Amazon S3 bucket in all AWS Regions. 
This behavior applies to both writes to new objects as well as PUT requests that overwrite existing objects and DELETE requests.

Here are examples of this behavior:
* A process writes a new object to Amazon S3 and immediately lists keys within its bucket. The new object appears in the list.
* A process replaces an existing object and immediately tries to read it. Amazon S3 returns the new data.
* A process deletes an existing object and immediately tries to read it. Amazon S3 does not return any data because the object has been deleted.
* A process deletes an existing object and immediately lists keys within its bucket. The object does not appear in the listing.

Bucket configurations have an eventual consistency model. Specifically, this means that:
* If you delete a bucket and immediately list all buckets, the deleted bucket might still appear in the list.
* If you enable versioning on a bucket for the first time, it might take a short amount of time for the change to be fully propagated. We recommend that you wait for 15 minutes after enabling versioning before issuing write operations (PUT or DELETE requests) on objects in the bucket.


## Access Points

* Access Point Policy to grant R/W permissions to a certain prefix
* Simplify security management
* Each access point has:
  * Proper DNS name
  * Access Point Policy

VPC Origin:
* Can be defined to be only privately accessible
* Must create a VPC endpoint (Gateway or Interface Endpoint)
* Endpoint policy must allow access to target bucket and Access Point

## Object Lambda

* Use Lambda Functions to change the object before it is retrieved
* Only one S3 bucket is needed, on top of which an S3 Access Point S3 Object Lambda Access Point is created

## S3 API
  
## S3 CLI

List buckets
```
aws s3 ls
```

List bucket contents
```
aws s3 ls s3://bucket-name
```

Make bucket
```
aws s3 mb s3://bucket-name
```

Upload file
```
aws s3 cp source-name s3://bucket-name
```

# CloudFront

AWS CDN

* A system of distributed servers to deliver web content
* Low latency and high data transfer speeds

## Edge Locations

* Users access content from Edge Locations
* Spread around the world
* Keep a cache of copies of your objects
* 216 edge locations
* DDoS protection
* Integration with Shield and WAF

## Origins

An origin is the location where content is stored, and from which CloudFront gets content to serve to viewers. To specify an origin:

* Use S3OriginConfig to specify an Amazon S3 bucket that is not configured with static website hosting.
  
* Use CustomOriginConfig to specify all other kinds of origins, including:
  * An Amazon S3 bucket that is configured with static website hosting
  * An Elastic Load Balancing load balancer
  * An AWS Elemental MediaPackage endpoint
  * An AWS Elemental MediaStore container
  * Any other HTTP server, running on an Amazon EC2 instance or any other kind of host
  * Route 53 address

## Distribution

* Name given to origin
* Configuration settings for content you wish to distribute

## Caching & Caching Policies

* Each object in the cache is identified using the Cache Key
* Maximize Cache Hit ratio to minimize requests to the origin
* CreateInvalidation API

Cache Key:
* Unique identifier for each object in cache
* By default, consists of hostname + resource portion of the URL
* Other elements can be added (HTTP headers, cookies, query strings) using Cache Policies

Cache Policies:
* Cache based on:
  * HTTP headers
  * Cookies
  * Query Strings
* Control TTL (0s to 1y)
  * Can be set by origin using Cache-Control and Expires headers
* Create custom policies or use Predefined Managed Policies
* All HTTP headers, cookies and query strings in the Cache Key are included in origin requests

HTTP Headers:
* None
  * Dont include any headers in cache key
  * Headers are not forwarded
  * Best cache performance
* Whitelist
  * Only specified headers are included in the Cache Key
  * Forwarded to origin

Query Strings:
* None
  * Query strings are not included
  * Not forwarded
* Whitelist
  * Specify which query strings are included
* Include All-Except
* All

Origin Request Policy:
* Specify values you want to include in origin request without including them in the Cache Key
* You can include:
  * HTTP Headers
  * Cookies
  * Query Strings

## Cache Invalidations

* Force entire or partial cache refresh

## Cache Behaviours

* Configure different settings for a given URL path pattern
* Route to different kind of origins/origin groups based on content type or path pattern
* When adding additional Cache Behaviours, the Default Cache Behaviour is always the last to be processed and is always /*
* Signed Cookies
* Route Dynamic vs Static content

## ALB or EC2 as an origin

* EC2 instances must be public
* ALB must be public
  * EC2 instances can be private

## Geo Restriction

* Restrict who can access your distribution based on country of request
  * AllowList
  * BlockList
* Uses 3rd party IP address list to determine country

## Origin Failover

Setup:
* To set up origin failover, you must have a distribution with at least two origins. 
* Next, you create an origin group for your distribution that includes two origins, setting one as the primary. 
* Finally, you create or update a cache behavior to use the origin group.

After you configure origin failover for a cache behavior, CloudFront does the following for viewer requests:

* When there’s a cache hit, CloudFront returns the requested object.
* When there’s a cache miss, CloudFront routes the request to the primary origin in the origin group.
* When the primary origin returns a status code that is not configured for failover, such as an HTTP 2xx or 3xx status code, CloudFront serves the requested object to the viewer.
* When any of the following occur:
  * The primary origin returns an HTTP status code that you’ve configured for failover
  * CloudFront fails to connect to the primary origin
  * The response from the primary origin takes too long (times out)
  * Then CloudFront routes the request to the secondary origin in the origin group.

Note:
* CloudFront routes all incoming requests to the primary origin, even when a previous request failed over to the secondary origin. CloudFront only sends requests to the secondary origin after a request to the primary origin fails.
* CloudFront fails over to the secondary origin only when the HTTP method of the viewer request is GET, HEAD, or OPTIONS. CloudFront does not fail over when the viewer sends a different HTTP method (for example POST, PUT, and so on).

## TTL (Time To Live)

* Objects are cached for a period of time which is defined by the TTL
* Cache can be invalidated (incurs fee)

## S3 Transfer Acceleration

* Enables fast transfer of date file over long distances through Edge Locations

## SSL/TLS - HTTPS

* Certificates need to be created in us-east-1 (N. Virginia)

## Viewer protocol policy

Options:
* HTTP and HTTPS
* Redirect HTTP to HTTPS
* HTTPS only

## Allowed HTTP methods

Options:
* GET, HEAD
* GET, HEAD, OPTIONS
* GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE

## Signed URLs & Signed Cookies

* Distribute paid shared content
* Attach a policy which includes:
  * URL expiration
  * IP ranges to access data from
  * Trusted signers
* Signed URL - access to individual files (one signed url per file)
* Signed Cookies - access to multiple files
* Use AWS SDK to generate Signed URL/Cookie

CloudFront Signed URL vs S3 Pre-Signed URL
* CloudFront Signed URL:
  * Allow access to a path no matter the origin
  * Account wide key-pair, only root can manage it
  * Can filter by IP, path, date, expiration
  * Can leverage caching features
* S3 Pre-Signed URL
  * Issue a request as the person who pre-signed the URL
  * Uses the IAM key of the signing IAM principal
  * Limited lifetime

### Signed URL Process

Two types of signers
* Trusted key group (recommended)
  * Can leverage API to create and rotate keys (and IAM and API security)
* AWS account that contains CloudFront Key Pair
  * Need to manage keys using root account and console
  * Not recommended
  * Max 2 key pairs
* Create one or more trusted key groups per distribution
* Generate own public/private key
  * Private key is used by application to sign URLs
  * Public key is used by CloudFront to verify URLs

## Multiple Origins

* To route to different kind of origins based on content type
* Based on path pattern

## Origin Groups

* To increase HA and do failover
* Origin Group: one primary and one secondary
* If primary fails the second is used
* Primary status is determined per request

## Field Level Encryption

* Protect user sensitive info through application stack
* Adds additional layer of security along with HTTPS
* Sensitive info encrypted at the edge close to the user
* Asymmetric encryption
* Usage:
  * Specify set of fields in POST requests that you want to be encrypted
  * Specify the public key to encrypt them

## Real Time Logs

* Get real-time requests received by CloudFront sent to Kinesis Data Streams
* Monitor, analyze and take actions based on content delivery performance
* Allows you to choose:
  * Sampling Rate
  * Specific fields and specific Cache Behaviours (path patterns)

## Function Associations

* Viewer request
* Viewer response
* Origin request
* Origin response

Viewer request -> Origin request -> Origin response -> Viewer response

## Price class

* All edge locaiton
* NA and Europe
* NA, EMEA

## AWS WAF

* Protects against web attacks 

## AWS Shield

* Protects against DDOS attacks

## Supported HTTP versions

* HTTP/2
* HTTP/3

## Geographic restrictions

* Allow or deny countries

## Origin Access Control

* Force all requests to go through Cloudfront instead of directly to your origin
* Replaces Origin Access Identity

E.g. Bucket policy
```
{
    "Version": "2012-10-17",
    "Statement": {
        "Sid": "AllowCloudFrontServicePrincipalReadOnly",
        "Effect": "Allow",
        "Principal": {
            "Service": "cloudfront.amazonaws.com"
        },
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::<S3 bucket name>/*",
        "Condition": {
            "StringEquals": {
                "AWS:SourceArn": "arn:aws:cloudfront::<AWS account ID>:distribution/<CloudFront distribution ID>"
            }
        }
    }
}
```

# ECS, ECR and Fargate

## Docker

* Software development platform to deploy applications
* Apps are packaged in containers that can be run on any OS
* Apps run the same
  * Any machine
  * No compatibility issues
  * Predictable behaviour
  * Less work
  * Easier to maintain and deploy
* For microservices architecture, lift-and-shift,...

## ECS Launch Types

ECS: Elastic Container Service
* Launch Docker containers on AWS: Launch ECS Tasks on ECS Clusters
* EC2 launch type: you must provision & maintain the infrastructe (EC2 instances)
* Each EC2 instance must run ECS Agent to register in the ECS cluster
* AWS takes care of starting / stopping the containers

Fargate:
* You do not provision the infrastructure
* Serverless
* Only create task definitions
* AWS runs ECS tasks based on CPU / RAM you need

## ECS IAM Roles

EC2 Instance Profile (EC2 Launch Type only):
* USed by ECS Agent
* Make API Calls to ECS Service
* Send container logs to CloudWatch Logs
* Pull Docker image from ECR
* REference sensitive data in Secrets Manager or Parameter Store

ECS Task ROle:
* Allows each task to have a specific role
* Use different roles for each different ECS service
* Task Role is defined in the task definition

## Load Balancer Integrations

* ALB supported for most use cases
* NLB recommended for high througput/high performance use cases or to pair with AWS Private Link
* CLB supported but not recommended
  * No advanced features
  * No Fargate

## Data Volumes (EFS)

* Mount EFS file systems onto ECS tasks
* Compatible with EC2 and Fargate launch types
* Tasks running in any AZ will share the same data in the EFS file system
* Fargate + EFS = serverless
* Persistent multi-AZ shared storage
* S3 cannot be mounted 

## ECS Anywhere

* For on prem container hosts

## ECS Cluster

* Runs ECS tasks

Capacity provider:
* Fargate 
* Fargate Spot
* EC2: ASG

## ECS Task Definition

* Containerized services are defined in Task Definitions
* Task definitions are metadata in JSON form to tell ECS how to run a Docker container
* Contains crucial info:
  * Image name
  * Port binding
  * Memory and CPU
  * Env vars
  * Networking info
  * IAM role
  * Logging config
* Can define up to 10 containers per Task Definition

### EC2 Load balancing
* Dynamic Host Port Mapping if you define only the container port in the task definition
  * Integrates with ALB
  * Must allow on the EC2 SG any port from the ALB SG

### Fargate Load balancing
* Each task has a unique private IP
* Only define the container port

### IAM Role
* Only one Role per Task Definition

### /etc/ecs/ecs.config

### Environment Variables

* Hardcoded
* SSM Parameter Store
  * Fetched at runtime
* Secrets Manager
  * Fetched at runtime
* Env files (bulk) - S3

### Data Volumes (Bind Mounts)

* Share data between multiple containers in the same Task Definition
* Works for both EC2 and Fargate launch types
* EC2 tasks - using EC2 instance storage
  * Data tied to the lifecycle of EC2 instance
* Fargate Tasks - ephemeral storage
  * 20 GB - 200GB
  * Data tied to containers using them

### Sidecar containers

* Used for e.g. logging, metrics,...

### ECS Task Placement

* WHen a task of type EC2 is launched, ECS must determine where to place it, with constraints of CPU, memory and available port
* Wen a service scales in, ECS needs to determine which task to terminate
* Only for EC2 launch types
* Best effort

Task Placement Process:
* Identify instances that satisfy CPU, memory and port requirements
* Identify instances that satisfy task placement constraints
* Satisfy task placement strategies
* Select instances for task placement
  
Task placement strategies:
* Binpack:
  * Place tasks based on least available CPU or memory
  * Minimizes number of instances in use
  * Cost savings
* Random:
  * Place tasks randomly
* Spread:
  * Spread tasks evenly based on specified value
    * instanceId, attribute:ecs.availability-zone
* Can be mixed and matched

Task placement constraints:
* distinctInstance
  * Place each task on a different container instance
* memberOf:
  * Place task on instances that satisfy expression
    * Use Qluster Query Language

### Example Task Definition

```
{
   "containerDefinitions": [ 
      { 
         "command": [
            "/bin/sh -c \"echo '<html> <head> <title>Amazon ECS Sample App</title> <style>body {margin-top: 40px; background-color: #333;} </style> </head><body> <div style=color:white;text-align:center> <h1>Amazon ECS Sample App</h1> <h2>Congratulations!</h2> <p>Your application is now running on a container in Amazon ECS.</p> </div></body></html>' >  /usr/local/apache2/htdocs/index.html && httpd-foreground\""
         ],
         "entryPoint": [
            "sh",
            "-c"
         ],
         "essential": true,
         "image": "httpd:2.4",
         "logConfiguration": { 
            "logDriver": "awslogs",
            "options": { 
               "awslogs-group" : "/ecs/fargate-task-definition",
               "awslogs-region": "us-east-1",
               "awslogs-stream-prefix": "ecs"
            }
         },
         "name": "sample-fargate-app",
         "portMappings": [ 
            { 
               "containerPort": 80,
               "hostPort": 80,
               "protocol": "tcp"
            }
         ]
      }
   ],
   "cpu": "256",
   "executionRoleArn": "arn:aws:iam::012345678910:role/ecsTaskExecutionRole",
   "family": "fargate-task-definition",
   "memory": "512",
   "networkMode": "awsvpc",
   "runtimePlatform": {
        "operatingSystemFamily": "LINUX"
    },
   "requiresCompatibilities": [ 
       "FARGATE" 
    ]
}
```
## Auto Scaling

* Automatically scale desired number of ECS tasks
* ECS Auto Scaling uses AWS application Auto Scaling
  * ECS Average CPU Utilization
  * Average Memory Utilization
  * Request Count Per Target (ALB)
* Target Tracking - scale based on target value for a specific CloudWatch metric
* Step Scaling - scale based on a specified CloudWatch Alarm
* Scheduled Scaling - scale based on schedule
* ECS Auto Scaling is not EC2 auto scaling 
* Fargate Auto Scaling is much easier to setup

### Auto Scaling EC2 instances

* Accomodate ECS Service Scaling by adding underlying EC2 instances
* Auto Scaling Group Scaling
  * Scale your ASG based on CPU utilization
  * Add EC2 instances over time
* ECS Cluster Capacity Provider
  * Used to automatically provision and scale infrastructure for your ECS tasks
  * Capacity Provider paired with an ASG
  * Add EC2 Instances when you're missing capacity (CPU, RAM)

## ECS Rolling Updates

* When updating from v1 to v2, we can control how many tasks can be started and stopped and in which order

## ECR

Elastic Container Registry 

* Store and manage Docker images on AWS
* Public and private repositories (Gallery)
* Fully integrated with ECS
* Access is controlled through IAM
* Supports image vulnerability scanning, versionning, image tags, image lifecycle,...

## CloudWatch (awslogs driver)

Fargate:
* Add the required logConfiguration parameters to your task definition to turn on the awslogs log driver

EC2:
* To turn on awslogs driver, your Amazon ECS container instances require at least version 1.9.0 of the container agent

### CLI

aws ecr get-login-password --region region | docker login --username AWS --password-sttdin aws-acount-id.dkr.ecr.region.amazonaws.com

docker push aws_account_id.dkr.ecr.region.amazonaws.com/demo:latest

docker pull aws_account_id.dkr.ecr.region.amazonaws.com/demo:latest

## AWS Copilot

* CLI tool to build, release and operate production readu containerized apps
* Run your apps on AppRunner, ECS and FarGate
* Helps building apps instead of setting up infrastructure
* Provisions required infrastructure for containerized apps
* Automated deployments
* Deploy to multiple environments
* Troubleshooting, logs, health status

## AWS EKS

Elastic Kubernetes Service

* Launch managed Kubernetes clusters on AWS
* Open-source system for automatic deployment, scaling and management of containerized applications
* EKS supports EC2 if you want to deploy worker nodes or Fargate to deploy serverless containers

Node types:
* Managed node groups
  * Creates and manages EC2 instances
  * Nodes are part of an ASG managed by EKS
  * Supports on-demand or Spot instances
* Self-managed nodes
  * Nodes created by you and registered to the EKS cluster
  * Prebuilt AMI - Amazon EKS optimized AMI
  * Supports on-demand or Spot instances
* Fargate
  * No maintenance required
  * No nodes managed
  * Support for:
    * EBS
    * EFS (works with Fargate)
    * Amazon FSx for Lustre
    * FSx for NetApp ONTAP

Data volumes
* Specify StorageClass manifest on your EKS cluster
* Leverages Container Storage Interface compliant driver
  
# Athena

Enables you to run SQL queries on data stored in S3

* Serverless
* Pay per query and TB scanned
* Easy to use
* Integrated with S3

# Step Functions

* Model your workflows as state machines
* One state machine per workflow
* Written in JSON
* Visualization of the workflow and execution of the workflow, aswell as history
* Start workflow with SDK call, API Gateway, EventBridge

## Task States

* Do some work in your state machine
* Invoke one AWS Service:
  * Invoke Lambda functions
  * Run AWS Batch job
  * ECS task
  * ...
* Run one Activity
  * Activities poll the Step functions for work
  * Activities send result back to Step Function

States:
* Choice State: test for a condition to send to a branch (or default branch)
* Fail or Succeed State: stop execution with failure or success
* Pass State: simply pass its input to its output or inject some fixed data, without performing work
* Wait State: provide a delay for a certain amount of time or until a specified time/data
* Map State: Dynamically iterate steps
* Parallel State: Begin parallel branches of execution

## Error Handling

* Any state can encounter runtime errors for various reasons
  * State machine definition issues
  * Task failures
  * Transient issues
* Use Retry (to retry failed state) and Catch (to transition to failure path) in the State Machine to handle the errors instead of inside the application code
* Predefined error codes:
  * States.ALL: matches any error name
  * States.Timeout: task ran longer than TimeoutSeconds or no heartbeat received
  * States.TaskFailed: execution failure
  * States.Permissions: correct permissions are missing
* State may report its own errors

Retry (Task or Parallel State):
* Specifies retry mechanisms
* Evaluated from top to bottom
* ErrorEquals: match a specified kind of error
* IntervalSeconds: initial delay before retrying
* BackoffRate: multiple the delay after each retry
* MaxAttempts: defaults to 3, set 0 for never retry
* When max attempts are reached, the Catch kicks in

Catch (Task or Parallel State):
* Evaluated from top to bottom
* ErrorEquals: match a specified kind of error
* Next: State to send to
* ResultPath - A path that determines what input is sent to the state specified in the Next field

## Wait for Task Token

* Allows you to pause Step Functions during a Task until a Task Token is returned
* Task might wait for other AWS services, huma approval, 3rd party integration...
* Append .waitForTaskToken to the Resource field to wait for the Task Token to be returned
* Task will pause until it receives that TaskToken back with a SendTaskSuccess or SendTaskFailure API Call
* Push mechanism

## Activity Task

* Enables you to have the Task work performed by an Activity Worker
* Activity Worker apps can be running on EC2, Lambda...
* Activity WOrker poll for a TAsk using GetActivityTask API
* After Activity Worker completers its work, its sends a response of its success/failure using SendTaskSuccess or SendTaskFailure
* Poll mechanism
* Configure how long a task can wait using TimeoutSeconds
* Send heartbeat from Activity Worker using SendTaskHeartBeat within time set with HeartBeatSeconds
* Activity Task can wait up to 1 year

## Standard vs Express workflows

Standard:
* Max duration: 1 year
* Execution model: Exactly once execution
* Execution rate: Over 2000/s
* Execution history: Up to 90 days or using CloudWatch Logs
* Pricing: # of state transitions
* Use cases: non-idempotent actions
Express:
* Max duration: 5minutes
* Execution rate: > 100000/s
* Execution history: CloudWatch Logs
* Pricing: # of executions, duration and memoru consumption
* Use cases: IoT data ingestion, streaming data, mobile app backends
* Synchronous - At most once
  * Wait for Workflow to complete
  * When in need an immediate response
  * Can be invoked from API Gateway or Lambda function
  * Will not be restarted when fails
* Asynchronous - At least once
  * Doesn't wait for results to complete
  * Use case: don't need an immediate response
  * Must manage idempotence

# AppSync

* Managed service that uses GraphQL
* GraphQL makes it easy for applications to get exactly the data they need
* This includes combining data from one or more source
  * NoSQL data stores, relational databases, HTTP APIs
  * Integrates with DynamoDB, Aurora, Opensearch...
  * Custom sources with AWS Lambda
* Retrieve data in real-time with Websocket or MQTT on WebSOcker
* For mobile apps: local data access & data synchronization
* Starts with uploading one GraphQL schema

Authorization:
* API_KEY
* AWS_IAM: IAM access
* OPENID_CONNECT: OpenId Connect provider / JWT Token
* AMAZON_COGNITO_USER_POOLS
* For custom domain & HTTPS, use CloudFront in front of AppSync
  
# Amplify

* Create mobile and web applications
* Amplify Studio
* Amplify CLI
* Amplify Libraries
* Amplify Hosting
* Set of tools to get started with creating mobile and web apps
* Must-have features such as data storage, authentication, storage and ML, powered by AWS services
* Front end frameworks 
* Incorporates AWS best practices

Authentication:
* amplify add auth
* Leverages Amazon Cognito
* User registration, authentication, account recovery...
* Supports MFA, social sign-in...
* Prebuilt UI components
* Fine-grained authorization

Datastore:
* amplify add api
* Leverages AppSync and DynamoDB
* Work with local data and have automatic synchronization to the cloud without complex code
* Offline and real-time capabilities
* Model data with Amplify Studio

Hosting:
* amplify add hosting
* Build and host webapps
* CICID
* Pull Request Review
* Custom Domains
* Monitoring
* Redirect and Custom Headers
* Password Protection

End-to-End testing:
* Run end-to-end tests in the test phase in Amplify
* Catch regressions before pushing code to production
* use the test step to run any test commands at build time (amplify.yml)
* Integrated with Cypress testing framework
  * Allows you te generate UI report for your tests



# CloudFormation

Manage Infrastructure-As-Code

* CloudFormation is a declarative way of defining your AWS Infrastructure, for any resources

Benefits:
* Infrastructure as Code
  * No resources are manually created
  * Code can be version controlled
  * Changes to infrastructure are reviewed through code
* Cost
  * Each resource within the stack is tagged with an identifier
  * Cost estimation
  * Savings strategy
* Productivity
  * Ability to destroy and recreate infrastructure
  * Automated generation of diagrams
  * Declarative programming
* Separation of concern
  * Re-use stacks
* Leverage existing templates

How does it work:
* Templates have to be uploaded to S3 and then referenced in template
* To update a template, reupload a new version
* Stacks are identified by a name
* Deleting a stack deletes every single artifact   

Deploying templates:
* Manual way
  * Editing templates in CloudFoormation designer
  * Use console to input params,...
* Automated way
  * Editing templates in YAML
  * Use AWS CLI
  * Recommended

Building Blocks:
* Template components
  * Resources (mandatory)
  * Parameters
  * Mappings
  * Outputs
  * Conditionals
  * Metadata
* Template helpers
  * References
  * Functions

## Resources

* Core of CloudFormation templates
* Represents the different AWS Components that will be created and configured
* Resources are declared and can reference each other
* AWS figures out creation, updates and deletes of resources
* Over 224 types of resources
* Identifiers are of the form
  * AWS::aws-product-name::data-type-name 
  * e.g. AWS::EC2::Instance
  
```
  Type: AWS::EC2::Instance
  Properties: 
    AdditionalInfo: String
    Affinity: String
    AvailabilityZone: String
    BlockDeviceMappings: 
      - BlockDeviceMapping
    CpuOptions: 
      CpuOptions
    CreditSpecification: 
      CreditSpecification
    DisableApiTermination: Boolean
    EbsOptimized: Boolean
    ElasticGpuSpecifications: 
      - ElasticGpuSpecification
    ElasticInferenceAccelerators: 
      - ElasticInferenceAccelerator
    EnclaveOptions: 
      EnclaveOptions
    HibernationOptions: 
      HibernationOptions
    HostId: String
    HostResourceGroupArn: String
    IamInstanceProfile: String
    ImageId: String
    InstanceInitiatedShutdownBehavior: String
    InstanceType: String
    Ipv6AddressCount: Integer
    Ipv6Addresses: 
      - InstanceIpv6Address
    KernelId: String
    KeyName: String
    LaunchTemplate: 
      LaunchTemplateSpecification
    LicenseSpecifications: 
      - LicenseSpecification
    Monitoring: Boolean
    NetworkInterfaces: 
      - NetworkInterface
    PlacementGroupName: String
    PrivateDnsNameOptions: 
      PrivateDnsNameOptions
    PrivateIpAddress: String
    PropagateTagsToVolumeOnCreation: Boolean
    RamdiskId: String
    SecurityGroupIds: 
      - String
    SecurityGroups: 
      - String
    SourceDestCheck: Boolean
    SsmAssociations: 
      - SsmAssociation
    SubnetId: String
    Tags: 
      - Tag
    Tenancy: String
    UserData: String
    Volumes: 
      - Volume
```

## Parameters

* A way to provide inputs to templates
  * Reuse templates
  * Some inputs cannot be determined ahead of time
* Powerful, controlled and can prevent errors thanks to types

Parameter settings:
* Type:
  * String
  * Number
  * CommaDelimitedList
  * List<Type>
  * AWS Parameter
* Description
* Constraints
* ConstraintDescription (String)
* Min/MaxLength
* Min/MaxValue
* Defaults
* AllowedValues (array)
* AllowedPattern (regex)
* NoEcho (Boolean)

Reference parameters:
* Fn::Ref can be leveraged to reference parameters 
* Shorthand in YAML is !Ref  

### Pseudo Parameters

Can be used at any time and are enabled by default

* AWS::AccountId
* AWS::NotificationARNs
* AWS::NoValue
* AWS::Region
* AWS::StackId
* AWS::StackName

## Mappings

* Fixed variables
* Handy to differentiate between different environments (dev vs prod, regions, AMI types)
* Hardcoded in template
* When you know in advance all the values that can be used and that can be deduced, such as Region, AZ,...
* Fn::FindInMap
  * Return named value from a specific key
  * !FindInMap [ MapName, TopLevelKey, SecondLevelKey ]

```
Mappings: 
  RegionMap: 
    us-east-1: 
      "HVM64": "ami-0ff8a91507f77f867"
    us-west-1: 
      "HVM64": "ami-0bdb828fd58c52235"
    eu-west-1: 
      "HVM64": "ami-047bb4163c506cd98"
    ap-southeast-1: 
      "HVM64": "ami-08569b978cc4dfa10"
    ap-northeast-1: 
      "HVM64": "ami-06cd52961ce9f0d85"
```

## Outputs

* Outputs declares optional output values that can be imported into other stacks (if exported)
* Can be viewed in Console or using CLI
* Good for reusing stacks
* You cannot delete a stack if outputs are referenced by another stack
* Export block is required for exporting values
* Fn::ImportValue to use the value in another stack
* Exported Output values must be unique in Region

```
Outputs:
  StackVPC:
    Description: The ID of the VPC
    Value: !Ref MyVPC
    Export:
      Name: !Sub "${AWS::StackName}-VPCID"
```

## Conditions

* Used to control the creation of resources or outputs based on conditions
* Common ones:
  * Region
  * Environment
  * Any parameter value
* Each condition can reference another condition, parameter value or mapping  

```
Parameters:
  EnvType:
    Type: String
    AllowedValues:
      - prod
      - test
  BucketName:
    Default: ''
    Type: String
Conditions:
  IsProduction: !Equals 
    - !Ref EnvType
    - prod
  CreateBucket: !Not 
    - !Equals 
      - !Ref BucketName
      - ''
  CreateBucketPolicy: !And 
    - !Condition IsProduction
    - !Condition CreateBucket
Resources:
  Bucket:
    Type: 'AWS::S3::Bucket'
    Condition: CreateBucket
  Policy:
    Type: 'AWS::S3::BucketPolicy'
    Condition: CreateBucketPolicy
    Properties:
      Bucket: !Ref Bucket
      PolicyDocument: ...
```

## Intrinsic Functions

* Ref
  * Parameters: returns value of parameter
  * Resources: returns physical ID of underlying resource
* GetAtt
  * Attributes are attached to any resource you create
  * E.g. !GetAtt EC2Instance.AvailabilityZone
* FindInMap  
* ImportValue
* Join
  * !Join [ delimiter, [ comma-delimited list of values ] ]
  * Join values with a delimiter
* Sub
  * Used to substitute values within strings
  * E.g. !Sub "${AWS::StackName}-VPCID"
* Condition Functions
  * Equals
  * And
  * If
  * Not
  * Or

## Rollbacks

* Stack Creation fails:
  * Default: everything is rolled back 
  * Option to disable rollback and troubleshoot (preserve successfully provisioned resources)
* Stack Update fails
  * Default: everything is rolled back to last known working state
  * Option to disable rollback and troubleshoot (preserve successfully provisioned resources)

## Stack Notifications

* Send Stack events to SNS Topic
* Enable SNS Integration using Stack Options

## ChangeSets

* ChangeSets provides a preview of what will be updated and changed
* Will not tell you if an update will be successful

## Nested Stacks

* Stacks that are part of another stack
* Isolate repeated patterns or common components and call them from other stacks
* To update a nested stack, always update the parent (root stack)

### Cross vs Nested Stack

Cross Stacks:
* Helpful when stacks have different lifecycles
* Use Output Exports and Fn::ImportValue
Nested stack:
* When components must be re-used
* Nested stack is only important to parent stack

## StackSets

* Create, update or delete stacks across multiple accounts and regions
* Administrator account to create StackSets
* Trusted accounts to create, update and delete stack instances from StackSets
* When you update a StackSet, all associated stacks are updated across all accounts and regions

## Drift

* No protection from manual configuration changes
* CloudFormation Drift allows detecting changes against the stack template

## Stack Policies

* During an update, by default all update actions are allowed on all resources
* Stack Policies are JSON documents which define what update actions are allowed on specific resources during a Stack Update
* Protect resources from unintentional updates
* All resources in the Stack are protected by default
* Specify an explicit ALLOW for the resources you want to update

```
{
  "Statement" : [
    {
      "Effect" : "Allow",
      "Action" : "Update:*",
      "Principal": "*",
      "Resource" : "*"
    },
    {
      "Effect" : "Deny",
      "Action" : "Update:*",
      "Principal": "*",
      "Resource" : "LogicalResourceId/ProductionDatabase"
    }
  ]
}
```

# Serverless Application Model (SAM)

* Framework for developing and deploying serverless applications
* All the configuration is YAML code
* Generate complex CloudFormation from simple SAM YAML file
* Supports anything from CloudFormation
* Only two commands to deploy to AWS
* SAM can use CodeDeploy to deploy Lambda functions
* SAM can help you to run Lambda, API Gateway, DynamoDB locally

## Recipe

* Transform Headers indicates it's SAM template:
  * Transform: 'AWS::Serverless-2016-10-31'
* Write Code
  * AWS::Serverless::Function - Lambda function
  * AWS::Serverless::Api - API Gateway
  * AWS::Serverless::SimpleTable - DynamoDB table
* Package & Deploy:
  * aws cloudformation package / sam package
  * aws cloudformation deploy / sam deploy

## Deploying SAM app

1. Create S3 bucket
2. aws cloudformation package --s3-bucket --template-file --output-template-file / sam package --s3-bucket --template-file --output-template-file
3. aws cloudformation deploy --template-file --stack-name / sam deploy --template-file --stack-name

## Policy Templates

* List of templates to apply permissions to your Lambda Functions
* Important examples:
  * S3ReadPolicy: read only permissions to read objects in S3
  * SQSPollerPolicy: allows to poll an SQS queue
  * DynamoDBCrudPolicy: allwos CRUD operations on DynamodDB

## SAM and CodeDeploy

* SAM natively uses CodeDeploy to update Lambda functions
* Traffic Shifting feature
* Pre and Post traffic hooks features to validate deployment (before traffic shift starts and after it ends)
* Easy & Automated rollback using CloudWatch Alarms
* AutoPublishAlias
  * Detects when new code is being deployed
  * Creates and publishes an updated version of that function with the latest code
  * Points the alias to the updated version
  * of the Lambda function
* DeploymentPreference
  * Canary, Linear, AllAtOnce
  * Alarms
    * Alarms that can trigger a rollback
  * Hooks
    * Pre and post traffic shifting Lambda functions to test your deployment
* sam build

## CLI Debugging

* Locally build, test and debug your serverless applications that are defined using AWS SAM templates
* Provides a lambda-like execution environment
* SAM CLI + AWS Toolkits => step-through and debug your code
* AWS toolkits: IDE plugins  

## Local Capabilites

* Locally start AWS Lambda
  * sam local start-lambda
  * Starts a local endpoint that emulates AWS Lambda
  * Can run automated tests against this local endpoint
* Locally Invoke Lambda function
  * sam local invoke
  * Invoke Lambda function with payload once and quit after invocation completes
  * Helpful for generating test cases
  * If the function makes API calls to AWS, make sure to select the correct --profile option
* Locally start API Gateway Endpoint
  * sam local start-api
  * Starts a local HTTP server that hosts all your functions
  * Changes to functions are automatically reloaded
* Generate AWS Events for Lambda functions
  * sam local generate-event

## Serverless Application Repository (SAR)

* Managed repository for serverless applications
* Apps are packages using SAM
* Build and publish apps that can be reused by organizations
  * Can share publicly
  * Can share with specific AWS accounts
* Settings and behaviour can be customized using Environment Variables

# AWS Cloud Development Kit (CDK)

* Define cloud infrastructure using a familiar language (Java, Python, .NET)
* Contains high level components called constructs
* Code is compiled into a CloudFormation template
* Deploy infrastructure and application runtime code together

CDK vs SAM:
* SAM:
  * Serverless focused
  * Declarative in JSON or YAML
  * Leverages CloudFormation
* CDK
  * All AWS services
  * Define cloud infrastructure using a familiar language (Java, Python, .NET)
  * Leverages CloudFormation

CDK + SAM:
* Locally test CDK apps
* cdk synth (generates Cloudformation template)
* Use template to run SAM commands

## Constructs

* Components that encapsulate everything CDK needs to create final CloudFormation Stack
* Can represent single AWS resource or multiple related resources
* Construct Library
  * Included in AWS CDK
  * 3 different levels of Constructs available (L1, L2, L3)
* Construct Hub
  * Additional Constructs from AWS, 3rd parties and open-source community

### Layer 1

* CFN resources
* Represent available CloudFormation resources
* Names start with Cfn
* Must explicitly configure all resource properties

### Layer 2

* AWS resources but higher level (intent-based API)
* Similar functionality as LI but with convenient defaults and boilerplate
* Provide methods that make it simpler to work with the resource

### Layer 3

* Called Patterns
* Represents multiple related resource
* Helps completing common tasks

## Commands

* npm install -g aws-cdk-lib
* cdk init app - create new CDK project
* cdk synth - synthesizes and prints CloudFormation template
* cdk bootstrap - deploys CDK Toolkit staging Stack
* cdk deploy - deploy stacks
* cdk diff - diff local CDK and deployed Stack
* cdk destroy - Destroy stacks

## Bootstrapping

* Process of provisioning resources for CDK before you can deploy CDK apps into an AWS environment
* AWS environment = account + region
* CloudFormation Stack called CDKToolkit is created and contains:
  * S3 bucket
  * IAM roles
* cdk bootstrap aws://<aws_account>/<aws_region>
* Without bootstrapping, you'll receive the following error, "Policy contains a statement with one or more invalid principals"

## Unit Testing

* To test CDK apps, use CDK Assertions Module combined with popular test frameworks
* Verify we have specific resources, rules, conditions, parameters
* Two types of tests:
  * Fine-grained Assertions (common)
    * Test specific aspects of the template
  * Snapshot tests
    * Test synthesized template against a previously stored baseline template
* To import a template:
  * Template.fromStack(MyStack): stack built in CDK
  * Template.fromString(mystring): stack built outside of CDK

# Lambda

* Virtual functions - no servers to manage
* Limited by time - short executions (15m max)
* Run on demand
* Scaling is automated

Benefits:
* Easy pricing
  * Pay per request and compute time
  * Generous free tier - 1000000 requests and 400000 GBs of compute time
* Integrated with AWS
* Many programming languages
* Easy monitoring with CloudWatch
* Easy to get more resources
* Increasing RAM will also increase CPU
* Custom Runtime API

Lambda Container Image
* Must implement Lambda Runtime API
* ECS / Fargate is still preferred for running arbitrary Docker images

Main integrations:
* API Gateway
* Kinesis
* DynamoDB
* S3
* CloudFront
* CloudWatch Events / EventBridge
* CloudWatch Logs
* SNS
* SQS
* Cognito

## Synchronous Invocations

* Synchronous:
  * Function is invoked through CLI, SDK, API Gateway, ALB, CloudFront, S3 Batch, Cognito, Steps Functions,...
  * Results is returned right away
  * Error handling must happen client side (retries, exponential backoff)

## Integration with ALB

* To expose a Lambda function as an HTTP(S) endpoint
  * ALB 
  * API Gateway

ALB:
* Lambda function must be registered in a Target Group
* Request gets transformed from HTTP to JSON 
* Response gets transformed from JSON to HTTP
* Response must be in correct format for the ALB
  
Request
```
{
    "requestContext": {
        "elb": {
            "targetGroupArn": "arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/lambda-279XGJDqGZ5rsrHC2Fjr/49e9d65c45c6791a"
        }
    },
    "httpMethod": "GET",
    "path": "/lambda",
    "queryStringParameters": {
        "query": "1234ABCD"
    },
    "headers": {
        "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8",
        "accept-encoding": "gzip",
        "accept-language": "en-US,en;q=0.9",
        "connection": "keep-alive",
        "host": "lambda-alb-123578498.us-east-1.elb.amazonaws.com",
        "upgrade-insecure-requests": "1",
        "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36",
        "x-amzn-trace-id": "Root=1-5c536348-3d683b8b04734faae651f476",
        "x-forwarded-for": "72.12.164.125",
        "x-forwarded-port": "80",
        "x-forwarded-proto": "http",
        "x-imforwards": "20"
    },
    "body": "",
    "isBase64Encoded": false
}
```

Response
```
{
    "statusCode": 200,
    "statusDescription": "200 OK",
    "isBase64Encoded": False,
    "headers": {
        "Content-Type": "text/html"
    },
    "body": "<h1>Hello from Lambda!</h1>"
}
```

ALB Multi-Header Values
* ALB supports multi header values
* When enabled HTTP headers and query string parameters that are sent with multiple values are converted to arrays

## Asynchronous Invocations

* S3, SNS, CloudWatch Events
  * Events are placed in an Event Queue
  * Lambda attempts to retry on errors
    * 3 retries
    * 1m wait after 1st, 2m wait after 2nd
  * Make sure processing is idempotent
  * If function is retried, you'll see duplicate log entries
  * You can define a DLQ (dead-letter queue), SNS or SQS, for failed processing (permissions are required)
  * Async invocations allow you to speed up processing if you do not need to wait for the result 
  * StatusCode 202 is returned
  * Can put an event or message on SNS, SQS or EventBridge
  * Lambda Destinations:
    * For each execution status you can choose one of four destinations:
      * SQS
      * SNS
      * Lambda 
      * EventBridge

## CloudWatch Events / EventBridge

* Event pattern
* Schedule

## S3 Event Notifications

* s3:OBjectCreated, s3:ObjectRemoved, s3:ObjectRestore, s3:Replication,...
* Object name filetering
* If two writes are made to a single non-versioned object at the same time, it is possible that only a single event notification will be sent
* If you want to ensure that an event notification is sent for every successful write, you can enable versioning on your bucket
* Need to setup correct permissions

## Event Source Mapping

* Kinesis Data Streams
* SQS & SQS FIFO queue
* DynamoDB Streams

Common denominator:
* Records need to be polled from the source
* Invoked synchronously

### Kinesis & DynamoDB

Event source mapping (Kinesis & DynamoDB):
* Creates an iterator for each shard, processes items in orde
* Start with new items, from beginning or timestamp
* Processed items aren't removed from the stream
* Low traffic: use batch window to accumulate records before processing
* Multiple batches can be processed in parallel
  * Up to 10 batches per shard
  * In-order processing is still guaranteed for each partition key

Error handling:
* By default, if function returns error, entire batch is reprocessed until function succeeds or items in the batch expire
* To ensure in-order processing, processing for the affected shard is paused until error is resolved
* You can configure event source mapping to:
  * Discard old events
  * restrict number of retries
  * Split batch on error
* Discarded events can go to a Destination

### SQS & SQS FIFO

* Event Source Mapping will poll SQS (Long Polling)
* Specify batch size (1-10 messages)
* Recommended: Set the queue visibility timeout to 6x timeout of Lambda function
* To use a DLQ
  * Set-up on the SQS queue (DLQ for Lambda is only for async invocations)

### Queues & Lambda:

  * Lambda also supports in-order processing for FIFO
    * Scaling up to the number of active message groups
  * For standard queue, items aren't always processed in orde
  * Lambda scales up to process a standard queue as quickly as possible
  * When an error occurs, batches are returned to the queue as individual items and might be processed in a different grouping than the original batch
  * Occasionally, event source mapping might receive the same item from the queue twice, even if no function error occurred
  * Lambda deletes items from the queue after successful processing
  * Source queue can be configured to send items to a DLQ if they can't be processed

### Event Mapper Scaling

* Kinesis Data Streams & DynamoDB Streams
  * One Lambda invocation per stream shard
  * If using parallelization, up to 10 batches processed per shard simultaneously
* SQS Standard:
  * Lambda adds 60 more instances per minute to scale up
  * Up to 1000 batches of messages process simultaneously
* SQS FIFO:
  * Messages with the same GroupID will be processed in orde
  * Lambda function scales to the number of active message groups

## Event & Context Objects

Event Object:
* JSON document containing data for the function process
* Contains info from the invoking service
* Lambda runtime converts the event to an object
* Input arguments, invoking service arguments

Context Object:
* Provides methods and properties that provide information about the invocation, function and runtime environment
* Passed to your function by Lambda at runtime
* aws_request_id, function_name, memory_limit_in_mb

## Destinations

Asynchronous invocations:
* Can define destinations for successful and failed events
  * SQS
  * SNS
  * Lambda
  * EventBridge
* AWS recommends using Destinations over DLQ (both can be used at the same time)
* Event Source Mapping: for discarded event batches
  * SQS
  * SNS
* You can send events to a DLQ directly from SQS

## Execution Role

* Grants the function permissions to AWS services / resources
* When using an Event Source Mapping to invoke a function, Lambda uses execution role to read event data
* Best practice: one role per function
* Use resource-based policies to give other accounts and AWS services to use your Lambda resources
* An IAM principal cann access Lambda
  * If IAM policy attached to principal authorizes is (user access)
  * Resource based polices authorizes it (service access)

## Environment Variables

* Key-value pair in String form
* Adjust function behavior without updating code
* Env vars are available to your code
* Lambda adds its own system vars as well
* Helpful to store secrets (KMS)
* Secrets can be encrypted by Lambda or your own CMK

## Logging & Monitoring

* CloudWatch Logs
  * Execution logs are stored in CloudWatch Logs
  * Execution role needs correct IAM policy to write to CloudWatch
* CloudWatch Metrics
  * Metrics are displayed in CloudWatch Metrics
  * Invocation, Durations, Concurrent Executions
  * Error count, Success Rates, Throttles
  * Async Delivery Failures
  * Iterator Age (Kinesis & DynamoDB Streams)
* X-Ray Tracing
  * Enable in Lambda config (Active Tracing)
  * Runs X-Ray daemon
  * Use X-Ray SDK
  * Ensure function has correct IAM Permissions
  * Env vars:
    * _X_AMZN_TRACE_ID: tracing header
    * AWS_XRAY_CONTEXT_MISSING: default LOG°ERROR
    * AWS_XRAY_DAEMON_ADDRESS: X-Ray Daemon IP_ADDRESS:PORT

## Lambda@Edge and CloudFront functions

* Many modern apps execute some logic at the edge
* Edge functions:
  * Code that you write and attach to CloudFront distributions
  * Runs close to your users to minimize latency
* CloudFront provides two type:
  * CloudFront Functions
  * Lambda@Edge
* Use cases:
  * Security and privacy
  * Dynamic Web Apps at the Edge
  * SEO
  * Routing
  * ...

Client -> Viewer request -> CloudFront -> Origin Request -> Origin -> Origin Response -> CloudFront -> Viewer Response -> Client

### CloudFront Functions

* Lightweight functions written in JS
* High scale, latency sensitive CDN customizations
* Sub-ms startup times, millions of requests/second
* Used to change Viewer requests and responses
* Native feature of CloudFront 
* Max execution time < 1ms
* Max memory 2MB
* Total package size 10KB
* No access network, file system access
* No access to request body
* Free tier, 1/6 price of edge

### Lambda@Edge

* Written in NodeJS or Python
* Scales up to 1000s of requests/seconds
* Used to change Viewer and Origin requests and responses
* Author function in one Region then CloudFront replicates it to its locations
* Max execution time 5 - 10s
* Max memory 128MB up to 10GB
* Total package size 1-5MB
* access to network and file system
* access to request body
* no free tier, charged per request & duration

### Use Cases

CloudFront Functions:
* Cache Key normalization
* Header manipulation
* URL rewrites ore redirects
* Request authentication & authorization

Lambda@Edge:
* Longer execution time 
* Adjustable CPU or Memory
* 3rd party libraries
* Network access to use external services
* File system access
* Access to the body of HTTP requests

## Networking

* By default, Functions are launched outside your VPC and cannot access resources in your VPC
* In VPC
  * Define VPC ID, subnets and Security Groups
  * Lambda will create an ENI in your subnets
  * AWSLambdaVPCAccessExecutionRole
* Internet Access
* Deploying a Lambda function in a public subnet does not give it internet access or public IP
* Deploying a Lambda function in a private subnet gives it internet access if you have a NAT Gateway / Instance
* You can use VPC endpoints to privately access AWS services without a NAT

## Performance

RAM:
* From 128MB to 10GB in 1MB increments
* The more RAM you add, the more vCPU credits you get
* At 1792MB, a function has the equivalent of one full vCPU
  * After 1792MB you get more than one vCPU and need to use multithreading in your code to benefit from it
* If your app is CPU bound, increase RAM

Timeout:
* Default 3s, max is 900s (15min)

Execution context:
* Execution context is a temp runtime that initializes external dependencies
* Great for database connections, HTTP clients, SDK clients
* Maintained for some time in anticipation of anteceding invocations
* Can be reused to save time
* Includes /tmp directory
* Initialize objects outside the handler

/tmp space:
* If function needs to download a big file
* If function needs disk space to perform operations
* Max size is 10GB
* Ephemeral
* Reused for multiple invocations
* For permanent persistent, use S3
* To encrypt content on /tmp, you must generate KMS data keys 

## Layers

* Custom Runtimes
  * ex. C++, Rust
* Externalize dependencies to re-use them

## Mounting File Systems Mounting

* Lambda functions can access EFS file systems if they are running in a VPC
* Configure Lambda to mount EFS file systems to local directory during initalization
* Must leverage EFS Access Points
* Limitations:
  * EFS Connection limits (One function instance = one connection)

## Concurerency and Throttling

* Concurrency limit: up to 1000 executions
* Reserved concurrency can set at function level (=limit)
* Each invocation over the concurrency limit will trigger a Throttle
* Throttle behavior
  * If sync invoke -> ThrottleError - 429
  * Async invoke -> retry automatically and then DLQ
* IF higher limit is needed, open a support ticket
* Concurrency limit applies to all functions in your account

Concurrency and async invocations:
* If function doesn't have enough concurrency available to process all events, additional requests are throttled
* For throttling errors (429) and system errors (5xx), Lambda returns the event to the queue and attemtps to run the function again for up to 6 hours
* Retry interval increases exponentially from 1s to max of 5m

## Cold Starts & Provisioned Concurrency

* Cold Start:
  * New instance: code is loaded and code outside the handler runs (init)
  * If init is large this can take some time
  * First request server by new instances has higher latency than the rest
* Provisioned Concurrency
  * Concurrency is allocated before function is invoked
  * Cold Start never happens
  * Application Auto Scaling can manage concurrency (schedule or target utilization)  

## External Depedencies

* Packages need to be installed alongside your code and be zipped together
* Upload zip straight to Lambda if less than 50MB else to S3
* Native libraries work: they need to be compiled on Amazon Linux
* AWS SDK is included by default
* X-Ray SDK needs to be installed
  
## CloudFormation

* Inline
  * Code is defined inline in the CloudFormation template
  * For simple functions
  * Code.ZipFile property
  * External depedencies cannot be included 
* S3
  * Lambda zip must be stored in S3
  * S3 location must be refered in template
    * S3Bucket
    * S3Key
    * S3ObjectVersion
* S3 multiple accounts

## Container Images

* Deploy Lambda functions as container images of up to 10GB from ECR
* Pack complex and/or large dependencies in a container
* Base image must implement Lambda Runtime API
* Test containers locally using the Lambda Runtime Interface Emulator
* Unified workflow to build apps

Best practices:
* Strategies for optimizing container images:
  * Use AWS provided base images
  * Multi-Stage builds
  * Build from Stable to Frequently changing
    * Make your most frequently occurring changes as late as possible in Dockerfile
  * Singe Repo for functions with large Layers
* Use them to upload large Lambda Functions (up to 10GB)

## Versions 

* When you work on a Lambda function, the version is $LATEST
* When published, a version is created
* Versions are immutable
* Version have increasing version numbers
* Versions have their own ARN
* Version = code + config 
* Each version can accessed individually
* Unqualified ARN is the function ARN without version suffix
* ARN with version arn:aws:lambda:us-west-2:123456789012:function:my-function:1
* ARN with alias us-west-2:123456789012:function:my-function:TEST

## Aliases

* Pointers to Versions
* Aliases are mutable
* Aliases enable Canary deployments by assigning weights to versions ( Weighted Alias)
* Enable stable configurations of event triggers / destinations
* Aliases have their own ARNs
* Aliases cannot reference aliases

## CodeDeploy

* CodeDeploy can help automate traffic shfit for Lambda aliases
* Feature is integrated in SAM
* Strategies:
  * Linear: grow traffic every N minutes until 100%
    * Linear10PercentEvery3Minutes
    * Linear10PercentEvery10Minutes
  * Canary: try X % then 100%
    * Canary10Percent5Minutes
    * Canary10Percent30Minutes
  * AllAtOnce: immediate
* Can create Pre & Post Traffic hooks to check the health of the Lambda function
* AppSpec.yml:
  * Name: required
  * Alias: required
  * CurrentVersion: required
  * TargetVersion: required

## Function URL

* Dedicated HTTP(S) endpoint for your Lambda function
* A unique URL endpoint is generated for you
  * https://<url_id>.lambda-url.<region>.on.aws
* Access only through public internet
* Supports Resource based policies and CORS config
* Can be applied to any alias or $LATEST version
* Throttle function by using Reserved Concurrency

Security:
* Resource based policy:
  * Authorize other accounts
  * CIDRs
  * IAM principals
* Cross-Origin Resource Sharing
  * If calling URL from another domain
* AuthType NONE:
  * Allow public access
  * Resource based policy is always in effect
* AuthType AWS_IAM
  * IAM is used to authenticate and authorize request
  * Both Principal identiy based policy and resource based policy are evaluated
  * Principal must have lambda:InvokeFunctionUrl permission
  * Same account - Identity based policy or resource based policy as ALLOW
  * Cross account - Identity based policy and resource based policy as ALLOW

## CodeGuru

* Gain insights into performance using CodeGuru Profiler
* CodeGuru creates Profiler Group
* Supported for Java and Python runtimes

## Limits

Per region:
* Execution:
  * 128MB - 10GB (1MB increments)
  * Max execution time: 900s
  * Env vars (4KB)
  * Disk capacity in /tmp: 512MB to 10GB
  * Concurrency: 1000 executions
* Deployment:
  * Function deployment size (zip): 50MB
  * Size of uncompressed deployment (code + dependencies): 250MB
  * Size of env vars 

## Best practices

* Heavy-duty work outside of your handler
* Use env vars
* Minimize deployment package to runtime necessities
* Avoid using recursive code

# API Gateway:

Build, Deploy and Manage APIs

* Lambda + API Gateway: No infrastructure to manage
* Support for Websockets
* Handle API versioning
* Handle different environments
* Handle security (Authentication & Authorization)
* Create API keys, handle request throttling
* Swagger / Open API import
* Transform and validate requests and responses
* Cenerate SDK and API specifications
* Cache API responses
* 29s is max timeout

Integrations:
* Lambda
  * Invoke Function
  * Easy
* HTTP
  * Expose HTTP endpoints 
  * Use case:
    * Rate limiting
    * Caching 
    * User authentication
* AWS Services

Endpoint types:
* Edge-Optimized (default)
  * For global clients
  * Requests are routed through CloudFront Edge locations
  * API Gateway still lives in only one region
* Regional
  * For clients within the same region
  * Cloud manually combine with CloudFront
* Private
  * Can only be accessed from within a VPC using a, interface VPC endpoint (ENI)

Security
* User authentication through:
  * IAM Roles
  * Cognito
  * Custom Authorizer (Lambda Authorizer)
* Custom Domane Name HTTPS through ACM
  * If using Edge-Optimized, certificate must be in us-east-1
  * If using regional endpoint, certificate must be in the same region as the API Gateway
  * Must setup CNAME or ALIAS record

## Deployment Stages

* Changes become effective only after a deployment
* Deployment are made to Stages (as many you want)
* Each stage has its own configuration parameters
* Stages can be rolled back as a history of deployments is kept

### Stage Variables

* Like env vars but for API Gateway
* Use them for frequently changing config values
* Can be used in:
  * Lambda function ARNs
  * HTTP Endpoint
  * Parameter mapping templates
* Stage variables are passed to the context object in Lambda
* Format: ${stageVariables.variableName}

Stage Variables & Lambda Aliases
* We can create Stage Variable to reference a certain Lambda Alias
* ${stageVariables.lambdaAlias}

## Canary Deployments

* Enable canary deployments for any stage
* Choose % of traffic the canary channel receives

## Integration Types

* MOCK
  * To return a response without sending the request to a backend
* HTTP / Lambda
  * Integration request en Integration response must be congigured
  * Setup data mapping using mapping templates for the request and response
* AWS_PROXY (Lambda proxy)
  * Incoming request is input to Lambda
  * No mapping template, headers, query string parameters... are passed to the functions
* HTTP_PROXY
  * No mapping template
  * Request is passed directly
  * Response is forwarded directly
  * Possibility to add HTTP headers

## Mapping Templates

* Can be used to modify request or response
* Rename or modify query strings parameters
* Modify body content
* Add headers
* Uses Velocity Template Language: for loop, if else,...
* Filter output results
* Content-Type can be set to application/json or application/xml
* Use case:
  * JSON to XML with SOAP

## Open API Spec

* Common way of defining REST APIs using API definition as code
* Import existing OpenAPI 3.0 spec to API Gateway
  * Method
  * Method Request
  * Integration Request
  * Method REsponse
  * + AWS extensions for API gateway and setup every single option
* Can export current API as OpenAPI spec
* Can be written in YAML or JSON
* Using OpenAPI we can generate SDK for our applications

### Request Validation

* API Gateway can be congfigured to perform basic validation of an API request before proceeding with the integration request
* When validation fails, a 400 error is returned immediatly
* Reduces unnecessary calls to the backend
* Checks:
  * Required request parameters in the URI, query string and headers
  * Applicable request payload adheres to the configured JSON Schema request model of the method

## Caching

* Reduces number of calls made to the backend
* Default TTL (300s) - min 0s, max 3600s
* Defined per stage
* Possible to override cache settings per method
* Cache encryption option
* Capacity between 0.5GB and 237GB
* Expensive
  
Invalidation:
* Flush entire cache immediatly
* Clients can invalidate the cache with header Cache-Control: max-age=0 (proper IAM permissions required)
* If no InvalidateCache policy is imposed, any client can invalidate the cache

## Usage Plans & API Keys

Usage PLan:
* Who can access one or more API stages & methods
* How much and how fast they can access them
* Uses API keys to identify clients and meter access
* Configure throttling limits and quota limits that are enforced on individual clients
API Keys:
* Can use with usage plans to control access
* Throttling Limits are applied at API Key level
* Quota limits is the overal number of max requests

Correct Order for API Keys:
* Create one or more APIs, configure methods that require an API Key and deploy the APIs to stages
* Generate or import API keys to distribute to your clients
* Create usage plan with desired throttle and quota limits
* Associate API stages and API keys with the usage plan
* Callers of the API must supply the assigned API key in the X-Api-Key HTTP header

## Monitoring, Logging and Tracing

* CloudWatch Logs
  * Log contains info about request/response body
  * Enable at stage level
  * Can override setting on a per API basis
  * Might contain sensitive info
  * API Gateway calls AWS Security Token Service in order to assume the IAM role, so make sure that AWS STS is enabled for the Region.
* X-Ray
  * Enable tracing to get extra information about request in API gateway
* CloudWatch Metrics:
  * CacheHitCount & CacheHitMiss: efficiency of the cache
  * Count: number of requests in a given period
  * IntegrationLatency: Time between API Gateway request relay and backend response
  * Latency: Time between request received from client and response returned to client
* 4xx (client-side error) and 5xx (server-side error)
* Account limit
  * API Gateway throttles requests at 10000 rps across all APIs
  * In case of throttling => 429 Too Many Requests (retriable error)
  * Can set Stage limit & Method Limits
  * Usage Plans

Errors:
* 4xx:
  * 400 - bad request
  * 403 - access denied, WAF filtered
  * 429 - quota exceeded, Throttle
* 5xx:
  * 502 - bad gateway exception - usually an incompatible output returned from a Lambda proxy integration and occasionally for out-of-order invocations due to heavy loads
* 503 - service unavailable
* 504 - integration failure - Endpoint request timeout exception (29s timeout)

## CORS

* Must be enabled when you receive API calls from another domain
* OPTIONS preflight request must contain following headers: 
  * Access-Control-Allow-Methods
  * Access-Control-Allow-Headers
  * Access-Control-Allow-Origin
* Can be enabled through the console

## Authorization & Authentication

IAM Permissions:
* Create IAM policy and attach to User / Role
* Authentication = IAM, Authorization = IAM Policy
* Good to provide access within AWS
* Leverages Sig v4 capability where IAM credentials are stored in the HTTP headers

Resource Policies:
* Set JSON policy to define who and what can access API Gateway
* Cross Account Access
* Specific source IP address
* VPC Endpoint

Cognito User Pools
* Cognito fully manages user lifecycle, token expires automatically
* API Gateway verifies automatically through Cognito
* No custom implementation required
* Authentication = Cognito User Pools, Authorization = API Gateway Methods
* Can be backed by Facebook, Google,...

Lambda Authorizer:
* Most flexible
* Token based authorizer (Bearer Token)
* Request parameter based Lambda authorizer (headers, query string, stage var)
* Lambda must return an IAM policy for the user, result policy is cached
* Authentication = External, Authorization = Lambda function

## HTTP API vs REST API

HTTP API:
* Low latency
* Lambda Proxy
* HTTP Proxy
* Private integration
* No data mapping
* Supports OIDC and OAuth2.
* CORS
* No usage plans and API Keys
* No resource policies
* Low cost

## Websocket API

* Duplex interactive communication between client and server
* Enables stateful apps
* Often used in real-time apps such as chat apps, games, financial trading,...
* Lambda functions:
  * onConnect
  * sendMessage
  * onDisconnect

Connecting to the API:
* WebSocket URL
  * WSS://unique_id.execute-api.region.amazonaws.com/stage-name
* connectionId persists as long as connection is held
* Connection URL callback:
  * WSS://unique_id.execute-api.region.amazonaws.com/stage-name/@connections/connectionId
* Connection URL Operations:
  * POST - send message
  * GET - get latest connection status
  * DELETE - disconnect client
* Routing:
  * Incoming JSON messages are routed to different backend
  * If no routes => sent to $default
  * You request a route selection expression to select the field of the JSON to route from
    * e.g. $request.body.action
  * Result is evaluated against the route keys available in API Gateway

# DynamoDB

Serverless NoSql Database

* Non-relational and distributed
* Do no or have limited support for joins
* All the data needed for a queru is present in one row
* No aggregations
* Scale horizontally

DynamoDB:
* Fully managed, highly available with replication across multple AZ
* Scales to massive workloads
* Distributed
* Millions of requests per sconds, trillions of rows, 100s of TB of storage
* Fast and consistent in performance
* Integrated with IAM
* Event driven programming with DynamoDB streams
* Low cost and autoscaling capabilities
* Standard and Infrequent Access Table Class

Basics:
* Made of tables
* Each table has a primary key (decided at creation time)
* Each table has infinite number of rows
* Each item has attributes (can be null or added over time)
* Item max size is 400KB
* Data types:
  * Scalar types - String, Number, Binary, Boolean, Null
  * Document Types - List, Map
  * Set Types - String Set, Number Set, Binary Set
  
Partition Key options:
* Partition Key - Hash
  * Must be unique for each item
  * Must be diverse so that data is distributed (High cardinality)
* Partition Key + Sort Key (Hash + Range)
  * Combination must be unique
  * Data is grouped by partition key

## Read/Write capacity modes

* Provisioned Mode
  * Specify number of reads/writes per second
  * Plan capacity beforehand
  * Pay for provisioned read & write capacity units
* On-Demand mode
  * Read/writes automatically scale up/down
  * No capacity planning needed
  * Pay for what you use
  * More expensive

### Provisioned R/W

* Table mush have provisioned read and write capacity units
* Read Capacity Units
* Write Capacity Units
* Throughput can be exceeded termporarily, you'll get ProvisionedThroughputExceededException
* Exponential backoff

### Write Capacity Units (WCU)

* 1 WCU represents 1 write per second for an  item up to 1KB in sie
* If item is larger, more WCUs are used

### Strongly Consistent Read vs Eventually Consistent Read

Eventually Consistent Read (default):
* Possible to get stale data because of replication

Strongly Consistent Read:
* No more stale data
* Set ConistentRead to True in API calls
* Consumes twice the RCU

### Read Capacity Units (WCU)

* 1 RCU represents 1 Strongly Consistent Read per second or 2 eventually Consistent Reads per second for an item up to 4KB
* If item is larger, more RCUs are used

### Partitions Internal

* Data is stored in partitions
* Partition Keys go through a hashing algorithm to know to which partition they go to

### Throttling

* When provisioned RCU or WCU are exceeded we get ProvisionedThrougputExceededExceptions
* Reasons:
  * Hot keys
  * Hot partitions
  * Very large items
* Solutions:
  * Exponential backoff (included in SDK)
  * Distribute partition keys
  * If RCU issue, we can use DynamoDB Accelerator (DAX)

### On-Demand

* Unlimited WRU & RCU
* No throttling
* Pay for what you use
* More expensive

## Writing Data

* PutItem
  * Creates or fully replaces an item
  * Consumes WWCU
* UpdateItem
  * Edits an existing item's attributes an item if it does not exists
  * Can be used to implement Atomic Counter
* Conditional Writes
  * Access a write/update/delete only if conditions are met

## Reading Data

* GetItem
  * Read based on primary key
  * Eventually Consistent
  * Strongly Consistent
  * ProjectExpression can be specified to retrieve only certain attributes
* Query returns items based on
  * KeyConditionExpression
    * On Partition and Sort Key
  * FilterExpression
    * Additional filtering after the query operation
    * Use only with non-key attributes
* Able to do pagination on the results
* Can query table, LSI or GSI
* Scan
  * Scan entire table and then filter out data (client side)
  * Returns up to 1MB of data - use pagination
  * Consumes a lot of RCU
  * Limit impact using Limit
  * Use Paralles Scan to improve performance
  * Can use ProjectionExpression and FilterExpression
* Delete data
  * DeleteItem
    * Delete individual item
    * Ability to conditional delete
  * DeleteTable
* Batch Operations
  * Save latency
  * Operations are done in parallel
  * Part of a batch can fail, retry failed items
  * BatchWriteItem
    * Up to 25 PutItem and/or DeleteItem in one call
    * Up to 16MB of data written, up to 400KB of data per item
    * Can't update items 
    * UnprocessedItems (exponential backoff or add WCU)
  * BatchGetItem
    * Return items from one or more tables
    * Up to 100 items, up to 16MB of data
    * UnprocessedKeys for failed read operations (exponential backoff or add RCU)
* PartiQl
  * SQL compatible query language for DynamoDB

## Conditional Writes

* For PutItem, UpdateItem, DeleteItem and BatchWriteItem
* You can specify a Condition expression to determine which items should be modified:
  * attribute_exists
  * attribute_not_exists
  * attribute_type
  * contains
  * begins_with
  * size  

## Indexes - LSI & GSI

### Local Secondary Index (LSI)

* Alternative Sort Key 
* Same Partition Key
* Sort Key consists of one scalar attribute (String, Number or Binary)
* Up to 5 LSI per table
* Must be defined at table creation time
* Attribute Projections - can contain some or all the attributes of the basetable (KEYS_ONLY, INCLUDE,ALL)

### Global Secondary Index (GSI)

* Alternative Primary Key (HASH or HASH+RANGE) from the base table
* Speed up queries on non key attributes
* Index Key consists of scalar attributes (String, Number or Binary)
* Attribute Projections some or all the attributes of the base table (KEYS_ONLY, INCLUDE, ALL)
* Must provision RCUs & WCUs for the index
* Can be added or modified after table creation

### Indexes and Throttling

* GSI
  * If writes are throttled on GSI then main table will be throttled
  * Uses own RCU & WCU
* LSI
  * No special throttling considerations
  * Uses RCU & WCU of main table

## PartiQL

* Use SQL-like syntax to manipulate DynamoDB tables
* Supports some statements:
  * Insert
  * Delete 
  * Update 
  * Select

## Optimistic Locking

* Each item has an attribute that acts as a version number
* Update or delete only when version number matches
* Uses Conditional Write

## DynamoDB Accelerator (DAX)

* Fully managed, HA, seamless in-memory cache
* Microseconds latency for cached reads & queries
* No additional application logic required
* Solves Hot Key (too many reads)
* 5min TTL (default)
* Up to 10 nodes in the cluster
* Multi-AZ (3 nodes min recommended for production)
* Secure 

### DAX vs ElastiCache

DAX:
* Individual objects cache
* Query & Scan cache
ElastiCache:
* Store aggregations result

## Streams

* Ordered stream of item-level modifications (CRUD) in a table
* Streams records can be:
  * Sent to Kineses Data Streams
  * Read by AWS Lambda
  * Read by KCL apps
* Data retention for up to 24 hours
* Use cases:
  * React to changes in real-time
  * Analytics
  * Insert into derivative tables
  * Cross-region replication

Ability to choose the info that will be written to the stream:
  * KEYS_ONLY
  * NEW_IMAGE
  * OLD_IMAGE
  * NEW_AND_OLD_IMAGE
  * DynamoDB Streams are made of shards
  * Records are not retroactively populated in the streams

AWS Lambda:
* Define Event Source Mapping
* Ensure appropriate permissions
* Lambda is invoked synchronously

## TTL

* Automatically deletes items after an expiry timestamp
* Doesn't consume WCUs
* TTL must be a Number data type with Unix Epoch timestamp value
* Expired items deleted within 48h of expiration
* Undeleted items still appear in queries and scans
* Delete operations are written to Streams

## CLI

* --projection-expresison: one or more attributes to retrieve
* --filter-expression: filter items before being returned
* General AWS CLI pagination
  * --page-size
  * --max-items: returns NextToken
  * --starting-token: specify NextToken

## Transactions

* Coordinated, all or nothing operations (CRUD) to multiple items across one or more tables
* Provides ACID
* Read modes:
  * Eventual Consistency, Strong Consistency, Transactional
  * Standard, Transactional 
* Consumes 2x WCU and RCU
  * Prepare & Commit
* API
  * TransactGetItems (one or more of below): 
    * GetItem
  * TransactWriteItems (one or more of below): 
    * PutItem
    * UpdateItem 
    * DeleteItem
* Use cases:
  * Financial transactions
  * Managing orders
  * Multiplayer games

## Session State

* Can be used as Session State Cache
* vs ElastiCache:
  * ElastiCache is in-memory
  * DynamoDB is serverless
  * Both are key/value stores
* vs EFS:
  * Must be attached to EC2 instances as a network drive
* vs EBS & Instance Store
  * Only for local cache
* vs S3
  * High latency 
  * Not meant for small objects

## Partitioning Strategies

Write Sharding:
* Better distribution of items evenly acroiss partitions
* Add a suffix to Partition Key value
* Two methods for creating suffix:
  * Random suffix
  * Calculate using hashing algorythm

## Write Types

* Concurrent Writes
  * Multiple writes at the same time
  * Both will succeed
  * One of them will be overwritten
* Conditional Writes
  * Write based on condition (Version Number)
  * Only one write will succeed
* Atomic Writes
  * Both will succeed
  * Will be performed one at a time
* Batch writes

## Large Objects Pattern

* Only 400KB of data per item
* S3 bucket contains large object
* Object key is stored in DynamoDB

## Operations

* Table Cleanup
  * Scan + DeleteItem
    * Very slow
    * Expensive, consumes RCU & WCU
  * Drop Table + Recreate Table
    * Fast, efficient and cheap
* DynamoDB Table
  * Using Data Pipeline
  * Backup and restore into a new table
    * Takes some time
  * Scan + PutItem or BatchWriteItem
    * Write your own code
    * Expensive, consumes RCU & WCU

## Security

* VPC Endpoints available to access DynamoDB without using the internet
* Fully controlled by IAM
* Encryption at rest using KMS and in transit using SSL/TLS

## Backup and Restore

* Point in time recovery
* No performance impact

## Global Tables

* Multi-region, multi-active fully replicated, high performance

## DynamoDB Local

* For local development and testing

## AWS DMS

* Database Migration Services can be used to migrate to DynamoDB

## Fine-grained access

* Use Identity Provider (e.g. Cognito, Google,...)
* Assign IAM Role with Condition to limit access
* dynamodb:LeadingKeys (limit row level access for users on the Primary Key)
* Attributes (attribute level access)
* Users will only be only to modify their own data

# CodeCommit

* Version Control is the ability to understand various changes that happened to the code over time
* All these are enabled using a version control such as Git
* A Git repository can be synchronized on your computer, but it is usually upload on a central online repository
* Benefits are:
  * Collaborate with other developers
  * Make sure the code is backed-up 
  * Make sure it's fully viewable and auditable
* Private Git Repositories
* No size limit 
* Fully managed, highly available
* Code only in AWS Cloud account => increased security

Security:
* Interactions using Git
* Authentication
  * SSH Keys
  * HTTPS
* Authorization
  * IAM policies to manage users/roles permissions
* Encryption
  * Repos are automatically encrypted at rest using KMS
  * Encrypted in transit
* Cross account access
  * Use an IAM role and use sts:AssumeRole

# CodePipeline

* Visual Workflow to orchestrate your CICD
* Source - CodeCommit, ECR, S3, BItbucket, GitHub
* Build - CodeBuild, Jenkins, CloudBees, TeamCity
* Test - CodeBuild, AWS Device Farm, 3rd party tools
* Deploy - CodeDeploy, Elastic Beanstalk, CloudFormation, ECS, S3
* Invoke - Lambda, Step Functions
* Consists of stages:
  * Each stage can have sequential actions and/or parallel actions
  * Manual approval can be defined at any stage

Artifacts:
* Each pipeline stage can create artificats
* Artificats are stored in an S3 bucket and passed on to the next stage

Troubleshooting:
* For CodePipeline Pipeline/Action/Stage Exectution State Changes
* Use CloudWatch Events:  
  * You can create events for failed pipelines
  * You can create events for cancelled stages
* If CodePipeline fails a stage, your piopeline stops and you can get information in the console
* If pipeline can't perform an action, make sure the IAM service role attached has the needed permissions
* AWS CloudTrail can be used to audit AWS API calls

## Events vs Webhooks vs Polling

Events:
* Preferred way in AWS
* Can be triggered by external services like Github

Webhooks:
* Older way using HTTP and scripts

Polling:
* Older way

## Artifacts Action Type Constraints

Owner:
* AWS
* 3rd party - Github
* Custom - Jenkins
Action type:
* Source
* Build
* Test
* Approval - Manual
* Invoke
* Deploy

## Manual Approval Stage

* Owner is AWS
* Action is Manual
* User must have codepipeline:GetPipeline and codepipeline:PutApprovalResult permissions

## CloudFormation Integration

* CloudFormation is used to deploy complex infrastructure using a API
  * CREATE_UPDATE - create or update an existing stack
* Can be used to create and deploy test and prod infrastructure

# CodeBuild

* Source
* Build instructions:
  * buildspec.yml
  * Or insert manually
* Output logs can be stored in S3 & CloudWatch Logs
* CloudWatch Metrics to monitor build statistics
* Use EventBridge to detect failed build and trigger notifications
* CloudWatch Alarms to notify if you need thresholds for failures
* Build Projects can be defined within CodePipeline or CodeBuild
* Can be use with Docker to extend any environment
  
## buildspec.yml

* Must be at the root of the source code
* env
  * variables: plaintext
  * parameter-store
  * secrets-manager
* phases:
  * install
  * pre_build
  * build
  * post_build
* artifacts: what to upload to S3
* cache: files to cache to S3

```
version: 0.2

env:
  variables:
    JAVA_HOME: "/usr/lib/jvm/java-8-openjdk-amd64"
  parameter-store:
    LOGIN_PASSWORD: /CodeBuild/dockerLoginPassword

phases:
  install:
    commands:
      - echo Entered the install phase...
      - apt-get update -y
      - apt-get install -y maven
    finally:
      - echo This always runs even if the update or install command fails 
  pre_build:
    commands:
      - echo Entered the pre_build phase...
      - docker login -u User -p $LOGIN_PASSWORD
    finally:
      - echo This always runs even if the login command fails 
  build:
    commands:
      - echo Entered the build phase...
      - echo Build started on `date`
      - mvn install
    finally:
      - echo This always runs even if the install command fails
  post_build:
    commands:
      - echo Entered the post_build phase...
      - echo Build completed on `date`

reports:
  arn:aws:codebuild:your-region:your-aws-account-id:report-group/report-group-name-1:
    files:
      - "**/*"
    base-directory: 'target/tests/reports'
    discard-paths: no
  reportGroupCucumberJson:
    files:
      - 'cucumber/target/cucumber-tests.xml'
    discard-paths: yes
    file-format: CUCUMBERJSON # default is JUNITXML
artifacts:
  files:
    - target/messageUtil-1.0.jar
  discard-paths: yes
  secondary-artifacts:
    artifact1:
      files:
        - target/artifact-1.0.jar
      discard-paths: yes
    artifact2:
      files:
        - target/artifact-2.0.jar
      discard-paths: yes
cache:
  paths:
    - '/root/.m2/**/*'
```

## Local Build

* When in need of deep troubleshooting
* CodeBuild can be run locally with Docker
* Leverage CodeBuild Agent

## Inside VPC

* CodeBuild containers are launched outside your VPC
* VPC config can be specified:
  * VPC ID
  * Subnet ID
  * Security Group ID

## Security

* To access resources in your VPC, specify VPC config for your CodeBuild
* Secrets in CodeBuild
  * Don't store them as plaintext in env vars
  * Env vars can reference Parameter Store
  * Env vars can reference Secrets Manager

# CodeDeploy

* Deployment service that automates application deployment
* Deploy new app versions to EC2 instances, on prem services, Lambda functions, ECS services
* Automated Rollback capability in case of failed deployments, or trigger CloudWatch Alarm
* Gradual deployment control
* Deployment is defined in the appspec.yml file

## EC2/on prem 

* Can deploy to EC2 instances & on-prem servers
* Perform in-place deployments or blue/green deployments
* Must run the CodeDeploy Agent on the target instances
* Deployment speeds:
  * AllAtOnce: most downtime
  * HalfAtATime: reduced capacity by 50%
  * OneAtATime: slowest, lowest availability impact
  * Custom

In-Place deployment:
* New version is deployed on running instances
* Recommend for first deployment

Blue-Green deployment:
* Blue is current version
* Green is new version
* Traffic is shifted from blue to green after successful deployment
* Recommended for deployments in existing environments

CodeDeploy Agent:
* Must be running on the instances
* Can be installed and updated automatically using SSM
* Instances must have sufficient permissions to access S3 to get deployment bundles

## Lambda

* CodeDeploy can help you automate traffic shifting for Lambda aliases
* Integrated in SAM framework
* Linear: grow traffic every N minutes until 100%
  * LambdaLinear10PercentEvery3Minutes
  * LambdaLinear10PercentEvery10Minutes
* Canary: try X percent 100%
  * LambdaCanary10Percent5Minutes
  * LambdaCanary10Percent30Minutes
* AllAtOnce: immediate

## ECS

* CodeDeploy can help you automate the deployment of a new ECS Task Definition
* Only Blue/Green deployments
* Linear: grow traffic every N minutes until 100%
  * ECSLinear10PercentEvery3Minutes
  * ECSLinear10PercentEvery10Minutes
* Canary: try X percent 100%
  * ECSCanary10Percent5Minutes
  * ECSCanary10Percent30Minutes
* AllAtOnce: immediate

## appspec.yml

```
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html/WordPress
hooks:
  BeforeInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
      runas: root
  AfterInstall:
    - location: scripts/change_permissions.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/start_server.sh
    - location: scripts/create_test_db.sh
      timeout: 300
      runas: root
  ApplicationStop:
    - location: scripts/stop_server.sh
      timeout: 300
      runas: root    
```

## Deployment to EC2

* Define the deployment using the appspec.yml + Deployment Strategy
* Will do in-place update to your fleet of EC2 instances
* Can use hooks to verify the deployment after each deployment phase

## Deployment to an ASG

* In-place Deployment
  * Updates existing EC2 instances
  * Newly created EC2 instances by an ASG will also get automated deployments
* Blue/Green Deployment
  * A new ASG is created
  * Choose how long to keep old instances
  * Must be using ELB

## Redeploy & Rollbacks

* Rollback = redeploy a previously deployed revision of you application
* Deployment can be rolled back:
  * Automatically - rollback when a deployment fails or rollback when a CloudWatch Alarm thresholds are met
  * Manually 
* Disable Rollbacks - do not perform rollbacks for this deployment
* If a roll back happens, CodeDeploy redeploys the last known good revision as a new deployment (not a restored version)

## Troubleshooting

* Deployment Error: 'InvalidSignatureException - Signature expired'
  * CodeDeploy requires accurate time references
  * If date and time on your EC2 instances are not set correctyly, they might not match the signature date of your deployment request
  * Check log files to understand deployment issues
    * /opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log

# CodeStar

* Integrated solution that groups: CodeCommit, CodeBuild, CodeDeploy, CloudFormation, CodePipeline,...
* Quickly create CICD ready projects for EC2, Lambda, Elastic Beanstalk
  
# CodeArtifact

* Software packages depend on each other to be built and new ones are created
* Storing and retrieving these dependencies is called artificat management
* Traditionally you need to setup your own artificat management system
* CodeArtifact is a secure, scalable and cost-effictive artificact management for software development
* Works with common dependency management tools such as Maven, Gradle, npm, yarn, twine, pip and NuGet
* Developers and CodeBuild can then retrieve dependencies straight from CodeArtifact 
* EventBridge Integration
  * Event is created when a package version is created, modifified or deleted
* Resource policy can be used to authorize another account to access CodeArtifact
* A given principal can either read all the packages in a repo or none of them

## Upstream Repositories

* CodeArtifact repository can have other CodeArtifact repositories as Upstream Repositories
* Allows a package manager client to access the packages that are contained in more than one repository using a single repository endpoint
* Up to 10 Upstream Repositories
* Only one external connection

## External Connection

* External Connection is a connection between a CodeArtifact Repository and an external/public repository
* Allos you to fetch packages that are not already present in your Repository
* Create many repositories for many external connections

## Retention

* If a requested package version is found in an Upstream Repository, a reference to it is retained and is always available from the Downstream Repository
* The retained package version is not affected by changes to the Upstream Repository
* Intermediate repositories do not keep the package

## Domains

* Deduplicated Storage - Asset only needs to be stored once in a domain, even if its available in many repositories
* Fast Copying - only metadata record are updated when you pull packages from an Upstream CodeArtifcat Respository into a Downstream
* Easy Sharing Across Repositories and Teams - all the assets and metadata in a domain are encrypted with a single AWS KMS key
* Apply Policy Across Multiple Repositories - domain administrator can apply policy across the domain

# CodeGuru

* ML-powered service for automated code reviews and application performance recommendations
* Two functionalities
  * CodeGuru Review: automated code reviews for static code analysis
  * CodeGuru Profiler: recommendations about app performance during runtime

## CodeGuru Reviewer

* Identify critical issues, security vulnerabilities and hard-to-find bugs
* Uses ML

## CodeGuru Profiler

* Helps understand runtime behaviour of your application
* Features:
  * Identify and remove code inefficiencies
  * Improve application performance
  * Decrease compute costs
  * Provides heap summary (identify which objects using up memory)
  * Anomaly Detection
* Support applications running on AWS or on-prem
* Minimal overhead on application

### Agent Configuration

* MaxStackDepth - max depth of stacks in the code that is represent in the profile
* MemoryUsageLimitPercent - memory percentage used by the profiler
* MinimumTimeForReportingInMilliseconds - minimum time between sending reports (milliseconds)
* ReportingIntervalInMilliseconds - reporting interval used to report profiles (milliseconds)
* SamplingIntervalInMilliseconds - sampling interval that is used to profile samples (milliseconds)

# Cloud9

* Cloud-Based Integrated Development Environment (IDE)
* Code editor, debugger, terminal in a browser
* Prepackaged with essentials tools for popular programming languages
* Share your dev environment
* Fully intergrated with AWS SAM & Lambda

# SNS

Simple Notification Service

* Pub/sub
* Event producer only sends messages to one SNS topic
* As many event receivers (subssciptions) as we want to listen to the SNS topic notifications
* Each subscriber to the topic will get all the messages (note: new feature to filter messages)
* Up to 12.500.000 subscriptions per topic
* 100.000 topic limit
* Integrates with many AWS services

## Publish

Topic Publish (using SDK):
* Create a topic
* Create subscription(s)
* Publish to the topic

Direct Publish (for mobile apps SDK):
* Create a platform application
* Create a platform endpoint
* Publish to the platform endpoint
* Works with Google GCM, Apple APNS, Amazon ADM

## Security

* Encryption:
  * In flight using HTTPS API
  * At rest using KMS
  * Client side encryption
* Access Controls: 
  * IAM policies to regulate access to the SNS API
* Access Policies:
  * Cross account access
  * Other services

## Fan-out pattern

* To send a message to multiple queues
* Push once in SNS, receive in all SQS queues that are subscribers
* Fully decoupled, no data loss
* SQS allows for data persistence, delayed processing and retries of work
* Ability to add more SQS subscribers over time
* Make sure your SQS queue access policy allows for SNS to write
* Cross region delivery through SQS queues in other regions

Use cases:
* For the same combination of: event type and prefix you can only have one S3 event rule
* If you want to send the same s3 event to many SQS queues, use fan-out
* SNS can send to Kinesis

## FIFO Topic

* First In First Out (ordering of messages in the topic)
* Similar features as SQS FIFO
* Can only have SQS FIFO queues as subscribers
* Limited throughput
* In case you need fan out + ordering + deduplication

## Message Filtering

* JSON policy used to filter messages sent to SNS topic subscriptions
* If a subscription doesn't have a filter policy, it receives every message

# SQS

Simple Queue Service

* Oldest offering
* Fully manages service, used to decouple applications
* Unlimited throughput
* Unlimited number of messages in queue
* Default retention of messages: 4 days, maximum of 14 days
* Low latency (< 10 ms on publish and receive)
* Limitation of 256KB per message sent
* At least once delivery (can have duplicate messages)
* Best effort ordering (can have out of order messages)

## Producing Messages

* Produced to SQS using the SDK (SendMessage API)
* A message is persisted in SQS until a consumer deletes it
* Default retention of messages: 4 days, maximum of 14 days
* SQS Standard: Unlimited throughput

## Consuming Messages

* Consumers:
  * Running on EC2 instances
  * On prem servers
  * Lambda
  * ...
* Poll SQS for messages (can receive up to 10 messages at a time) (ReceiveMessage API)
* Process the messages
* Delete the Messages (DeleteMessage API)
* Consumers receive and process messages in parallel
* At least once delivery
* Best-effort message ordering
* Consumers delete messages after processing them
* Scale consumers horizontally

## SQS with Auto Scaling Group (ASG)

* ApproximateNumberOfMessages (CloudWatch Metric)
* CloudWatch Alarm -> Scaling Policy -> ASG

## Security

* Encryption
  * In flight using HTTPS API
  * At rest using KMS
  * Client side if client wants to perform encryption itself
* Access Control: IAM policies to regulate access to the SQS APi
* SQS Access Policies (similar to S3 bucket policies)
  * Useful for cross-account access
  * Useful for allowing other services to write to an SQS queue

## Access Policy

* Cross Account Access
* Publish S3 Event Notifications to SQS Queue

## Message Visibility Timeout

* After a message is polled, it becomes invisible to other consumers
* By default, message visibility timeout is 30s
* After visibility timeout, message is again visible in SQS
* If a message is not processed and deleted within the visibility timeout, the message will be processeed again
* A consumer can call the ChangeMessageVisibility API to get more time
* If visibility time is high and the consumer crashes, reprocessing will take time
* If visibility is too low, duplicates can occur

## Dead Letter Queues

* If a consumer fails to process a message within the visibility timeout, the message is returned to the queue
* We can set a threshold of how many times a message can be returned to the queue
* After the MaximumReceives threshold is exceeded, the message is sent to the DLQ
* DLQ of a FIFO queue must also be a FIFO queue
* DLQ of a Standard Queue must also be a Standard Queue
* Make sure to process the message before retention time expires

Redrive to source:
* Feature to help consume messages in the DLQ
* When our code is fixed, we can redrive the messages from the DLQ back to the source queue or another queue

## Delay Queues

* Delay a message (consumers don't see it immediately) up to 15 minutes
* Default is 0 seconds (message is available right away)
* Can set a default at queue level
* Can override the default on send using the DelaySeconds parameter

## Long Polling

* When a consumer requests messages from the queue, it can optionnaly "wait" for messages to arrive there if there none in the queue
* LongPolling decreases the number of API calls made to SQS while increasing the efficiency and latency of your application
* Wait time can be between 1 to 20s
* Long Polling is preferable to Short Polling
* Long Polling can be enabled at the queue level or at the API level using ReceiveMessageWaitTimeSeconds

## SQS Extended Client

* Java Library
* Uses S3 Bucket for large data

## API

* CreateQueue (MessageRetentionPeriod), DeleteQueue
* PurgeQueue: delete all the messages in queue
* SendMessage (DelaySeconds), ReceiveMessage, DeleteMessage
* MaxNumberOfMessage: default 1, max 10 (for ReceiveMessage API)
* ReceiveMessageWaitTimeSeconds: Long Polling
* ChangeMessageVisibility: message visibility
* Batch APIs for SendMessage, DeleteMessage, ChangeMessageVisibility helpds decrease costs

## FIFO Queue

* FIFO - First In First Out (ordering of messages in the queue)
* Limited throughput: 300 msg/s without batching, 3000 msg/s with batching
* Exactly-once send capability (removes duplicates)
* Messages are processed in order by the consumer

Uses:
* MessageGroupId
* MessageDeduplicationId

### Deduplication

* De-duplication interval is 5 minutes
* Two de-deduplication methods:
  * Content-based: will do SHA-256 hash of the message body
  * Explicitly provide a MessageDeduplicationID
* If you use the same value of MessageGroupID, you can only have one consumer and all the messages will be in orde
* To get ordering at the level of a subset of messages, specify different values for MessageGroupId
* Ordering across groups is not guaranteed

# Kinesis

* Makes it easy to collect, process and analyze streaming data in real-time
* Kinesis Data Streams: capture, process and store data streams
* Kinesis Data Firehose: load data streams into AWS data stores
* Kinesis Data Analytics: analyze data streams with SQL or Apache Flink

## Data Streams

* A Stream consists of Shards
* Producers send data (Records) into Streams 
* Records consist of:
  * Partition Key
  * Data Blob (up to 1MB)
* 1MB/S or 1000msg/s per Shard
* Consumers receive the messages (Records)
  * Partition Key
  * Sequence no.
  * Data Blob
* 2MB/s Per shard all consumers (receiving records)
* 2MB/s per shard per consumer (enhanced)
* Retention between 1 to 365 days
* Ability to reprocess data
* Data is immutable, it cannot be deleted once inserted
* Data that shares the same partition goes to the same shard

Capacity Modes:
* Provisioned mode
  * Choose number of Shards provisioned
  * Scale manually or using the API
  * Input: each shard gets 1MB/s (or 1000 records per second)
  * Output: each shard gets 2MB/s (classic or enhanced fanout)
  * You pây per shard provisioned per hour
* On demand
  * No need to provision or manage capacity
  * Default capacity provisioned (4MB/s in or 4000 records per second)
  * Scales automatically based on observed throughput peak during last 30 days
  * Pay per stream per hour & data in/out per GB

Security:
* Control Access / Authorization using IAM policies
* Encryption in flight using HTTPS endpoints
* Encryption at rest using KMS
* Client side encryption
* VPC Endpoints available for Kinesis to access within VPC
* Monitor API Calls using CloudTrail

## Producers

* Puts data records into data streams
* Data record consists of:
  * Sequence number (unique per partition per shard)
  * Partition Key (must specify while put records into stream)
  * Data blob (up to 1MB)
* Producers:
  * AWS SDK
  * Kinesis Producer Library
  * Kinesis Agent: monitor log files
* Write throughput: 1MB/s or 1000 records per shard
* PutRecord API
* Use batching with PutRecords API to reduce costs & increase throughput
* Use highly distributed partition key to avoid hot partition

ProvisionedThrougputExceeded:
* Exception when going over provisioned throughput
* Use highly distributed partition key
* Retries with exponential backoff
* Increase shards (scaling)

## Consumers

* Gets data records from data streams and process them
* GetRecords API

(Classic) Shared Fanout consumer - pull:
* Shared 2MB/s per shard across all consumers
* Low number of consuming applications
* Read througput: 2MB/s per shard across all consumers
* Max 5 GetRecords calls/s
* Latency ~ 200ms
* Minimize cost
* Consumers poll data from Kinesis using GetRecords API call
* Returns up to 10MB (then throttle for 5s) or up to 10000 records

Enhanced Fanout Consumer - push
* 2MB/s per shard per consumer
* SubscribeToShard API
* Latency ~ 70ms
* Higher cost
* Kinesis pushes data to consumers over HTTP/2 
* Soft limit of 5 consumer applications (KCL) per data stream (default)

Lambda:
* GetBatch API
* Supports Classic & Enhanced fanout consumers
* Read records in batches
* Configure batch size and batch window
* If error occurs, Lambda retries until succeeds or data expires
* Can process up to 10 batches per shard simultaneously

## Kinesis Client Library (KCL)

* Java library that helps reading records from a Kinesis Data Stream with distributed applications sharing the read workload
* Each shard is to be read by only one KCL instance
  * e.g. 4 shards = max 4 kcl instances
* Progress is checkpointed into DynamoDB (needs IAM access)
* Track other workers and share the work amongst shards using DynamoDB
* KCL can run on EC2, Elastic Beanstalk and on-premises
* Records are read in order at the shard level
* Versions:
  * KCL 1.x (supports shared consumer)
  * KCL 2.x (supports shared & enhanced fanout consumer)

## Operations

* Shard Splitting
  * Used to increase the Stream capacity (1MB/s per shard)
  * used to divide a hot shard
  * Increased cost
  * Old shard is closed and will be deleted once the data is expired
  * Cannot split into more than two shards in a single operation
  
* Merge Shards
  * Decrease stream capacity and save costs
  * Can be used to group two shards with low traffic (cold shards)
  * Old shards are closed and will be deleted once the data is expired
  * Cannot merge than two shards in a single operation

## Kinesis Data Firehose

* Batch writes records into Destination
* Optionally transform records using Lambda
* No need to write code
* Destinations
  * S3
  * Redshift (COPY through S3)
  * OpenSearch 
  * 3rd party destinations
  * Custom destinations using HTTP endpoint
* Optionally send all or failed records to S3 backup data
* Fully managed service, automatic scaling, serverless
* Pay for data going through Firehose
* Near real time
  * 60s min latency for non full batches
  * Or min 1MB of data at a time
* Supports many data formats, conversions, transformations

## Kinesis Data Analytics

* SQL Applications:
  * Sources:
    * Kinesis Data Streams
    * Kinesis Data Firehose
  * SQL Statements
  * Reference data in S3
  * Sinks:
    * Kinesis Data Streams
    * Kinesis Data Firehose
  * Real time analytics
  * Fully managed, serverless
  * Automatic scaling
  * Pay for actual consumption rate
  * Use cases
    * Time series analytics
    * Real time dashboards
    * ...

* Apache Flink
  * Use Flink (Java, Scala or SQL) to process and analyze streaming data
  * Sources
    * Kinesis Data Streams
    * Amazon MSK
  * Run any FLink application on managed cluster on AWS
    * Provision compute resources, parallel computation, automatic scaling
    * Application backups (implement as checkpoints and snapshots)
    * Use any Apache Flink programming features
    * Flink does not read from Firehose (use Kinesis Analytics for SQL instead)

## Ordering data into Kinesis

* Use Partition Key to guarantee ordering per key
* Same key will always go to the same shard

# Cognito

* Give users an identity to interact with our web or mobile applications
* User Pools:
  * Sign in functionality for app users
  * Integrate with API Gateway & Application Load Balancer
* Identity Pools (Federated Identity)
  * Provide AWS credentials to users so they can access AWS resources directly
  * Integrate with Cognito User Pools as an identity provider

## User Pools

* Create a serverless database of users
* Simple login
* Password reset
* Email & phone number verification
* MFA
* Federated Identities, SAML, OpenID
* Block users if their credentials are compromised elsewhere
* Login sends back a JWT
* Integrates with API Gateway and Application Load Balancer
* Lambda triggers

### Lambda Triggers

* Authentication events:
  * Pre auth 
  * post auth
  * Pre token generation
* Sign up:
  * Pre sign up
  * Post confirmation
  * Migrate user
* Messages
  * Custom message
* Token creation
  * Pre token generation
  
### Hosted UI

* Hosted UI is available to integrate Cognito
* Foundation for integration with social logins, OIDC or SAMl
* Can be customized with logo and css
* Custom domain:
  * A certificate must be created in us-east-1 (N. Virginia)
  * Must be defined in the App Integration section (global configuration)

### Adaptive Authentication

* Blocks sign-ins or requires MFA if login appears suspicious
* Each sign in is evaluated and generates a risk score
* When risk is detected, the user is prompted for a second MFA
* Risk is based on different factors, device, location, ip address...
* Integration with CloudWatch Logs

### Decoding a ID Token (JWT)

* User Pools issues JWT tokens (Base64 encoded)
  * Header 
  * Payload
  * Signature
* Signature must be verified to ensure the JWT can be trusted
* Libraries can help you verify the validity of JWT tokens issued by Cognito User Pools
* Payload will contain the user information (sub UUID, ....)
* From sub UUID, you can retrieve all user details from Cognito / OIDC

### Application Load Balancer - Authenticate Users

* You ALB can securely authenticate users
  * Offload the work of authenticating users to your load balancer
  * Your applications can focus on their business logic
* Authenticate users through:
  * Identity Provider (IdP): OIDC compliant
  * Cognito User Pools: 
    * Social IdPs
    * Corporate identities (SAML, LDAP, MS AD)
* Must use HTTPS listener to set authenticate-oidc & authenticate-cognito
* OnUnauthenticatedRequest - authenticate (default), deny, allow
  
## Identity Pools (Federated Identities)

* Get identities for users to obtain temporary AWS creds
* Your identity pool can include:
  * Public Providers
  * Users in an User Pool
  * OIDC providers & SAML providers
  * Developer Authenticated Identities
  * Allow for unauthenticated (guest access)  
* Users can then access AWS services directly or through API Gateway
  * The IAM policies applied to the creds are defined in Cognito
  * Can be customized based on the user_id for fine grained control

IAM Roles:
* Default IAM roles for authenticated and guest users
* Define rules to choose the role for each user based on the user's ID
* You can partition your users' access using policy variables
* IAM credentials are obtained by Cognito Identity Pools through STS
* Roles must have a trust policy of Cognito Identity Pools

## User Pools vs Identity Pools

* User Pools are used for Authentication
* Identity Pools are used for Authorization

# CloudWatch

CloudWatch
* Metrics: Collect and track key metrics
* Logs: Collect, monitor, analyze and store log files
* Events: Send notifications when certain events happen
* Alarams: React in real-time to metrics / events
X-Ray:
* Troubleshooting app performance and errors
* Distributed tracing of microservices
CloudTrail:
* Internal monitoring or API calls being made
* Audit changes to AWS resources by your users

## Metrics

* CloudWatch provides metrics for every service in AWS
* Metrics is a variable to monitor (CPUUtilization...)
* Metrics belong to a namespace
* Dimension is an attribute of a metric (instance id, environment,...)
* Up to 30 dimension per metrics
* Metrics have timestamps
* Can create CloudWatch dashboards of metrics

### EC Detailed monitoring

* EC2 instance metrics have metrics every 5minutes
* With detailed monitoring (for a cost), you get metrics every minute
* Use detailed monitoring if you want your ASG to scale faster
* AWS Free Tier allows 10 detailed monitoring metrics
* EC2 Memory usage is by default not pushed (must be pushed from inside the instance as a custom metric)

### Custom Metrics

* Possible to define and send custom metrics
* E.g. memory, disk space, number of logged in users
* PutMetricData API
* Ability to use dimensions (attributes) to segment metrics
  * Instance.ID
  * Environment.name
* Metric resolution (StorageResolution API parameter):
  * Standard: 1 minute
  * High Resolution: 1/5/10/30s - higher cost
* Important: accepts metric data points two weeks in the past and two hours in the future 

## Logs

* Log groups: arbitrary name, usually representing an application
* Log stream: instances within applications / log files / containers
* Can define expiration policies
  * Retention policy is defined at Log Groups level
* Can send logs to:
  * S3
  * Kinesis Data Streams
  * Kinesis Data Firehose
  * Lambda 
  * OpenSearch
* Logs are encrypted by default
* Can setup KMS-based encryption with your own keys

### Sources

* SDK, CloudWatch Logs Agent, CloudWatch Unified Agent
* Beanstalk: collection of logs from application
* ECS: collection from containers
* Lamba: function logs
* VPC Flow Logs
* AP Gateway
* CloudTrail based on filter
* Route53

### Logs Insights

* Search and analyze log data stored in CloudWatch Logs
* Provides a purpose built query language
  * Automatically discovers fields from AWS services and JSON log events
  * Fetch desired event fields, filter based on conditions, calculate aggregate statistics, sort events, limit number of events
  * Can save queries and add them to CloudWatch Dashboards
* Query multiple Log Groups in different AWS accounts
* Historic data, not real time

### S3 Export

* Log data can take up to 12 hours to become available for export
* CreateExportTask API

### Logs Subscriptions

* Get real time log events for processing and analysis
* Send to Kinesis Data Streams, Kinesis Data Firehost, Lambda
* Subscription Filter - filter which logs are events delivered to your destination

### Logs Encryption

* CloudWatch Logs can be encrypted with KMS Keys
* Enabled at the log group level, by associating a CMK with a log group
* Cannot be associated through the CloudWatch console
* Must use CloudWatch Logs API:
  * associate-kms-key: if log group already exists
  * create-log-group: if log group doesn't exist yet

### Multi-Account & Multi-Region Log Aggegration

* Cross Account Subscription
  * Send log events to resources in different AWS accounts (KDS, KDF)

### CloudWatch Agent & CloudWatch Logs Agent

* By default, no logs from your EC2 instance will go to CloudWatch
* You need to run a CloudWatch agent on EC2 to push the log files you want
* Make sure IAM permissions are correct
* CloudWatch Logs Agent can be used on-prem 

CloudWatch Logs Agent:
* Old version
* Can only send to CloudWatch Logs

CloudWatch Unified Agent:
* Collects additional system-level metrics such as RAM, processes, etc...
* Collect logs to send CloudWatch Logs
* Centralized configuration using SSM Parameter Store
* Metrics:
  * CPU (active, guest, idle, system, user, steal)
  * Disk (free, used, total) ...
  * RAM (free, inactive...)
  * Netstat
  * Processes
  * Swap Space
* Out of the box for EC2: disk, CPU, network (high level)
* Unified Agent should be used for more granularity

### Metrics Filter

* CloudWatch Logs can use filter expressions
  * Can be used to trigger Alarms
* Filters do not retroactively filter data. Filters publish metric data points for events that happen after creation of the filter
* Ability to specify up to 3 dimensions for the Metric Filter

## Alarms

* Alarms are used to trigger notifications for any metric
* Various options (sampling, %, max, min, etc...)
* Alarm States:
  * OK
  * INSUFFICIENT_DATA
  * ALARM
* Period:
  * Length of time in seconds to evaluate the metric
  * High resolution custom metrics: 10s, 30s or multiples of 60s
* Can be created based on Logs Metric Filters
* To test alarms and notifications, set alarm state using CLI - set-alarm-state

Targets:
* EC2 - Stop, Terminate, Reboot or Recover an EC2 instance
* Trigger Auto Scaling Action
* Send notification to SNS (from which you can do anything you want)

Composite Alarms:
* Alarms are on a single metric
* Composite Alarms are monitoring the states of multiple other alams
* AND and OR conditions
* Helpful to reduce alarm noise by creating complex composite alarms

Instance Recovery:
* Status Check:
  * Instance status: check EC2 vm
  * System status: check underlying hard
* Recovery: same private, public, Elastic IP, metadata, placement group

## Synthetics Canary

* Configurable script that monitors your APIs, URLs, Websites
* Reproduce what your users do programmatically to find issues
* Checks availability and latency of your endpoints and store load time data and screenshots of the UI
* Integration with with CloudWatch Alarms
* Scripts written in Node.js or Python
* Access to a headless Chrome browser
* Script can run on a schedule

Blueprints:
* Heartbeat Monitor
* API Canary
* Broken Link Checker
* Visual Monitoring
* Canary Recorder
* GUI Workflow Builder

## EventBridge

* Schedule: Cron jobs
* Event Pattern: Event rules to react to service events
* Trigger lambda functions, send SQS/SNS messages
* Each account has a default event bus
* Partner Event Bus
* Custom Event Bus
* Event Buses can be accessed by other AWS accounts using Resource based policies
* Events can be archived (all/filtered) (indefinitely or set period)
* Ability to replay archived events

### Schema Registry

* EventBridge can analyze events and infer schema
* Schema Registry allows you to generate code for your application that will know in advance how data is structered in the Event Bus
* Schema can be versioned

### Resource-based policy

* Manage permissions for a specific Event Bus
* Allow/deny events from another AWS account or AWS region
* Aggregate all events from your AWS Organization in a single AWS account or AWS region

### Multi-Account Aggregation

* Define event pattern in account and create Event Rule with target Event Bus in another account
* Create Resource policy to accept Events from another account

## AWS X-Ray

* Debugging in Production:
  * Test locally
  * Add log statements everywhere
  * Re-deploy in production
* Log formats differ across applications using CloudWatch and analytics is hard
* Debugging: monolith easy, distributed services hard
* No common views of your architecture

X-Ray:
* Visual analysis of your application
* Troubleshoot performance
* Understand dependencies in a microservice architecture
* Pinpoint service issues
* Review request behaviour
* Find errors and exceptions
* Are we meeting time SLA?
* Where am I throttled?
* Compatibility
  * Lambda
  * Beanstalk
  * ECS
  * ELB
  * API Gateway
  * EC2 or on prem servers
* Collects data from all different services
* Service map is computed from all segment and traces
* X-Ray is graphical, so even non technical people can troubleshoot

Leverages Tracing:
* Tracing is an end-to-end way to following a request
* Each component dealing with the request adds its own trace
* Tracing is made of segments (+ sub segments)
* Annotaitons can be added to traces to provide extra-information
* Ability to trace: 
  * Every request
  * Sample request
* Security
  * IAM for authorization
  * KMS for encryption at rest

How to enable X-Ray:
* Code must import X-Ray SDK
* Very little code modification needded
* Application SDK will capture:
  * Calls to AWS services
  * HTTP calls
  * Database calls
  * Queue Calls
* Install X-Ray daemon or enable X-Ray integration
  * X-Ray dameon works as a low level UDP packet interceptor
  * AWS Lambda / other AWS services already run X-Ray daemon
  * Application must have proper IAM permisions

If X-Ray is not working on EC2:
* Ensure EC2 role has proper permission
* Ensure X-Ray daemon is running
  
If X-Ray is not working on Lambda:
* Ensure Lambda execution role has proper permissions (AWSXRayWriteOnlyAccess)
* Ensure X-Ray is imported in the code
* Ensure Lambda X-Ray Active Tracing is enabled

## X-Ray Instrumentation

* Instrumentation means the measure of a products performance, diagnose errors and to write trace information
* Use X-Ray SDk
* Many SDKs require only configuration changes
* YOu can modify your application code to customize and annotation the data that the SDK sends to X-Ray using interceptors, filters, handlers, middleware

## X-Ray Concepts

* Segments: each application / service will send them
* Subsegments: for more details in your segment
* Trace: segmetns collected together to form and end-to-end trace
* Sampling: decrease the amount of requests sent to X-Ray, reduces cost
* Annotations: Key Value pairs used to index traces and use with filters
* Metadata: Key Value pairs, not indexed, not used for searching
* X-Ray daemon / agent has a config to send traces cross account:
  * Make sure IAM permissions are correct - the agent will assume the role
  * This allows to have a central account for all your application tracing

## X-Ray Sampling Rules

* With sampling rules, you control the amount of data that you records
* You can modify sampling rules without changing your code
* By default, X-Ray SDK records the first request each second and five percent of any additional requests
* One request per second is the reservoir, which ensures that at least one trace is recorded each second as long as the service is serving requests
* Five percent is the rate at which additional requests beyond the reservoir size are sampled

Rules:
* You can create your own rules with the reservoir and rate

## X-Ray APIs

Write APIs:
* PutTraceSegments: Uploads segment documents
* PutTelemetryRecords: Uploads telemetry
* GetSamplingRules: Retrieve all sampling rules
* GetSamplingTargets: advanced
* GetSamplingStatisticSummaries: advanced

Read APIs:
* GetServiceGraph: main graph
* BatchGetTraces: Retrieves a list traces specified by ID
* GetTraceSummaries: retrieves IDs and and annotations for traces available for a specified time frame using an optional filter
* GetTraceGraph: retrieves a service graph for one or more specific trace IDs

## X-Ray with Beanstalk

* Beanstalk platform includes X-Ray daemon
* Run the daemon by setting an option in the Beanstalk console or with a config file
* Make sure to give your instance profile the correct IAM permissions 
* Make sure app code is instrumented
* X-Ray daemon is not provided for multicontainer Docker

## X-Ray with ECS

ECS Cluster:
* X-Ray container as a daemon (1 container per EC2 instance)
* X-Ray container as a Side Car (1 side car per app container)

Fargate Cluster:
* X-Ray container as a Side Car (1 side car per app container)

# AWS Distro for OpenTelemetry

* Secure, production ready AWS supported distribution of the open-source OpenTelemetry project
* Provides a single set of APIs, libraries, agents and collector services
* Collects distributed traces and metrics from your apps
* Collects metadata from your AWS resources and services
* Auto-instrumentation Agents to collect traces without changing your code
* Send traces and metrics to multiple AWS services and partner solutions
  * X-Ray, CloudWatch, Prometheus
  * Instrument your apps running on AWS (EC2, ECS, EKS, Fargate, Lambda) as well as on-premises
* Migrate from X-Ray to AWS Distro for Telemetry if you want to standardize with open-source APIs from Telemetry or send traces to multiple destinations simultaneously

# CloudTrail

* Provides governance, compliance and audit for your AWS account
* CloudTrail is enabled by default
* Get an history of events / API calls made within your AWS Account by:
  * Console
  * SDK
  * CLI
  * AWS Services
* Can put logs from CloudTrail into CloudWatch Logs or S3
* A trail can be applied to all regions (default) or a single Region
* If a resource is deleted in AWS, investigate CloudTrail first

## CloudTrail Events

* Management Events
  * Operations that are performed on resources in your AWS account
  * Examples:
    * Configuring security (IAM AttachRolePolicy)
    * Configuring rules for routing data (Amazon EC2 CreateSubnet)
    * Setting up logging (AWS CloudTrail CreateTrail)
  * By default, trails are configured to log management events
  * Can separate Read Events (that don't modify resources) from Write Events (that may modify resources)
* Data Events
  * By default, data events are not logged (high volume operations)
  * S3 object level activity: can separate Read and Write Events
  * Lambda execution activity
* CloudTrail Insights Events
  * Enable CloudTrail Insights to detect unusual activity in your account
    * Inaccurate resource provisioning
    * Hitting service limits
    * Burst of AWS IAM actions
    * Gaps in periodic maintenance activity
  * CloudTrail Insights analyzes normal management events to create a baseline
  * Continuously analyzes write events to detect unusual patterns
    * Anomalies appaer in the CloudTrail console
    * Event is sent to S3
    * EventBridge event is generated (for automation needs)
* Event retention:
  * Stored for 90 days in CloudTrail
  * To keep events beyond this period, send them to S3

# KMS

## Encryption

Encryption in flight / in transit (SSL/TLS):
* Data is encrypted before sending and decrypting after receiving
* SSL certificates help with encryption (HTTPS)
* Encryption in flight ensures no man-in-the-middle attacks

Server side encryption at rest:
* Data is encrypted after being received by the server
* Data is decrypted before being set
* It is stored in an encrypted form thanks to a key
* The encryption / decryption keys must be managed somewhere and the server must have access to it

Client side encryption:
* Data is encrypted by the client and never decrypted by the server
* Data will be decrypted by a receiving client
* The server should not be able to decrypt the data
* Could leverage Envelope Encryption

## KMS (Key Management Service)

* Anytime you hear encryption for an AWS service, it's most likely KMS
* AWS manages encryption keys for us
* Fully integrated with IAM for authorization
* Easy way to control access to your data 
* Able to audit KMS Key usage using CloudTrail
* Seamlessly integrated into most AWS services (EBS, S3, RDS, SSM...)
* Never store secrets in plaintet
  * KMS Encryption available through SDK and CLI (API calls)
  * Encrypted secrets can be stored in env vars or code
* Scoped per region
* 4KB max data size

Key Types:
* KMS Keys is the new name of KMS Customer Master Key
* Symmetric (AES256):
  * Single encryption key that is used to Encrypt and Decrypt
  * AWS services that are integrated with KMS use Symmetric CMK
  * You never get access to the KMS Key unencrypted (must use KMS API)
* Assymetric (RSA & ECC key pairs)
  * Public (encrypt) & Private (Decrypt) pair
  * Used for encrypt/decrypt or sign/verify pair
  * Public key is downloadable but you cannot use private key unencrypted

Types of KMS Keys:
* AWS Owned Keys (free): SSE-S3, SSE-SQS, SSE-DDB
* AWS Managed Keys: free (aws/service-name)
* Customer managed keys created in KMS: 1$/month
* Pay for API call to KMS ($0.03 per 10000 calls)

Automatic rotation:
* AWS managed key: automatic every 1 year
* Customer managed key: (must be enabled) automatic every 1 year
* Imported KMS Key: manual rotation using alias

To copy encrypted snapshot across regions
* KMS ReEncrypt with KMS key of other region

KMS Key Policies:
* Control access to KMS keys
* Cannot control access without policy
* Default KMS Key Policy:
  * Created if you don't provide a specific KMS Key Policy
  * Complete access to the key to the root user = entire AWS account
* Custom Key Policy:
  * Define users, roles that can access the KMS key
  * Define who can administer the key
  * Useful for cross-account access of your KMS key

Copy Snapshots across accounts:
1. Create snapshot, encrypted with own KMS Key
2. Attach a KMS key policy to authorize cross-account access
3. Share encrypted snapshot
4. Create a copy of the snapshot, encrypt it with a CMK in your account
5. Create a volume from the snapshot

## Encrypt & Decrypt APIs

Encrypt envelope data:
* KMS Encrypt API call has a limit of 4KB
* If you want to encrypt data > 4KB, we need to use Envelope Encryption
* Main API is GenerateDataKey
* GenerateDataKey will return plaintext data key and encrypted data key (using CMK)
* Encrypt data using plaintext data key
* Store encrypted data and encrypted data key 

Decrypt envelope data:
* Decrypt API will decrypt data key using CMK
* Decrypt data with plaintext data key

Encryption SDK
* Implements envelope encryptions
* Exists aswell as a CLI
* Data Key Caching:
  * Re-use data keys instead of creating new ones for each encryption
  * Helps with reducing the number of calls to KMS with security trade-off
  * Use LocalCryptoMaterialsCache (max age, max bytes, max number of message)

API Summary:
* Encrypt up to 4KB of data through KMS
* GenerateDataKey:
  * Generates a unique symmetric data key (DEK)
    * Returns plaintext copy of data key
    * AND a copy that is encrypted under the CMK that you specify
* GenerateDataKeyWithoutPlaintext
  * Generate a DEK (encrypted)
    * DEK that is encrypted under the CMK that you specify
    * Must use Decrypt later
* Decrypt: decrypt up to 4KB of data (including Data Encryption Keys)
* GenerateRandom: returns random byte string

## KMS Limits

Request Quotas
* When you exceed a request quota, you get a ThrottlingException
* To respond, use exponential backoff (backoff and retry)
* For cryptographic operations, they share a quota
* This includes requests made by AWS on your behalf (SSE-KMS)
* For GenerateDataKey, consider using DEK caching from the encryption SDK
* You can request a Request Quota increase through API or AWS support

## S3 Bucket Key (SSE-KMS)

* New setting to decrease number of API calls to KMS by 99%
* Leverages data keys:
  * S3 bucket key is generated
  * That key is used to encrypt KMS objects with new data keys
* You will see less KMS CloudTrail events in CloudTrail

## Key Policies & IAM Principals

Default Key Policy:
* Principal is root

Federated user:
* Principal is federated user

Principal options:
* AWS Account 
* Account Root User
* IAM Roles
* IAM Role Sessions (assumed role)
* IAM Users
* Federated User Sessions
* AWS Services
* All Principals

## CloudHSM

* AWS provisions encryption hardware
* Dedicated Hardware 
* Manage your own encryption keys entirely
* HSM device is tamper resistant
* Supports both symmetric and asymmetric encryption (SSL/TLS)
* No free tier available
* Must use the CloudHSM Client Software
* Redshift supports CloudHSM for database encryption and key management
* Good option to use with SSE-C encryption

IAM permissions
* CRUD

CLoudHSM:
* Manages keys
* Manages users

High Availability:
* Clusters are spread across AZs

Integration:
* Through integration with KMS
* Configure KMS Custom Key Store with CloudHSM
* Ex. EBS, S3, RDS

# SSM Parameter Store

* Secure storage for configuration and secrets
* Optional Seamless Encryption using KMS
* Serverless, scalable, durable, easy SDK
* Version tracking of configurations / secrets
* Security through IAM
* Notifications through EventBridge
* Integration with CloudFormation
* Supports Parameter Hierarchy
* Access Secrets from Secrets Manager
* Public parameters
* GetParameters or GetParametersByPath
* Standard tier:
  * 10000 parameters
  * 4KB max size per parameter
  * No policies
  * No additional charge
  * Free storage pricing
* Advanced tier:
  * 100000 parameters
  * 8KB max size per parameter
  * Policies available
  * Charges apply
  * $0.05 per advanced parameter per month
  * Policies:
    * Assign TTL to force updating or deleting sensitive data
    * Can assing multiple policies
    * Notifications are sent to EventBridge
* Types:
  * String
  * StringList
  * SecureString (encrypted using KMS, must have access to KMS Decrypt)

# Secrets Manager

* Meant for storing secrets
* Capability to force rotation every X days
* Automate generation of secrets on rotation (using Lambda)
* Integration with RDS
* Secrets are encrypted using KMS
* Mostly meant for RDS integration
* Multi-Region Secrets 
  * Replicate Secrets across multiple regions
  * Secrets Manager keeps read replicas in sync with primary Secret
  * Ability to promote a read replicate to a standalone Secret

## CloudFormation Integration

* ManageMasterUserPassword - creates admin secret implicitly
  * RDS, Aurora will manage the secret in Secrets Manager and its rotation 
* Dynamic Reference

# AWS Nitro Enclaves

* Process highly sensitive data in an isolated compute environment
* Fully isolated virtual machines, hardened and highly constrained
  * no persistent storage, no interactive access, no external network
* Helps reduce attack surface
  * Cryptographic Attestion
  * Only Enclaves can access sensitive info
* Launch EC2 instance with EnclaveOptions set to true
* Use Nitro CLI to convert app to Enclave Image File (EIF)
* Use EIF as input with Nitro CLI to create an enclave

# OpenSearch

* Successor to ElasticSearch
* You can search any fields, even for partial matches
* OpenSearch can be used as a complement for another database
* Two modes:
  * managed
  * serverless
* Does not natively support SQL (can be enabled via a plugin)
* Ingestion from Kinesis Datafirehose, IAM, KMS encryption, TLS
* Comes with OpenSearch Dashboards

# Athena

* Serverless query service to analyze data stored in Amazon S3
* Uses standard SQL language to query the files (built on presto)
* Supports CSV, JSON, ORC, Avro and Parquet
* Pricing: $5.00 per TB scanned
* Commonly used with Quicksight for reporting/dashboards

## Performance Improvement

* Use columnar data for cost-saving
  * Parquet or ORC
  * Huge performance improvement
  * Use Glue to convert your data to parquet or ORC
* Compress data for smaller retrievals (bzip2, gzip, lé4, snappy, zlip, zstd)
* Partition datasets in S3 for easy querying on virtual columns

## Federated Query

* Allows you to run SQL queries across data stored in relational, non-relational, object and custom data sources
* Uses Data Source Connectors that run on AWS Lambda to run Federated Queries

# MSK

* Alternative to Kinesis
* Fully Managed Apache Kafka on AWS
  * Allows you to create, update, delete clusters
  * MSK creates & manages Kafka brokers nodes & Zookeeper nodes for you
  * Deploy the MSK cluster in your VPC, multi-AZ (up to 3 for HA)
  * Automatic recovery from common Apache Kafka failures
  * Data is stored on EBS vloumes for as long as you want
* MSK Serverless
  * Run MSK without managing capacity
  * Automatically provisions & scales resources
  
# ACM

* Let's you easily provision, manage and deploy SSL/TLS certificates
* Used to provide in flight encryption for websites (HTTPS)
* Supports both public and private TLS certificates
* Free of charge for public TLS certificates
* Automatic TLS certificate renewal
* Integrations with (load TLS certificates on)
  * Elastic Load Balancers
  * CloudFront Distributions
  * APIs on API Gateway

## ACM Private CA

* Managed service allows you to create private and CA, including root and subordinaries CAs
* Can issue deploy end-entity x.509 organizations
* Certificates are trusted only by your Organization
* Works for AWS services that are integrated with ACM

# Macie

* Fully managed data security and data privacy services that uses ML and pattern matching to discover and protect your sensitive data 
* Macie helps you identify and alert you to sensitive data, such as personally identifiable information (PII)

# AppConfig

* Configure, validate and deploy dynamic configurations
* Deploy Dynamic configuration changes to your apps independently of any code deployments
  * You don't need to restart the application
* Feature flags, application tuning, allow/block listing
* Use with apps on EC2 instances, Lambda, ECS, EKS
* Gradually deploy the configuration changes and rollback if issues occur
* Validate configuration changes before deployment
* using:
  * JSON schema
  * lambda Function
  
# CloudWatch Evidently

* Safely validate new features by serving them to a specified % of your users
  * Reduce risk and identify unintended consequences
  * Collect experiment data, analyze using stats, monitor performance
* Launches (= feature flags): enable and disable features for a subset of users
* Experiments (=A/B testings)
* Overrides: pre-define variation for a specific user
* Store evalution events in CloudWatch or S3