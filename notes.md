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
  
## Elastic Load Balancer

A load balancer distributes network traffic across a group of servers.

Types:
* Application Load Balancer
  * HTTP and HTTPS
* Network Load Balancer
  * TCP and high performance
* Classic Load Balancer
  * HTTP/HTTPS and TCP
* Gateway Load Balancer
  * Virtual Applicances
  * Virtual Firewalls
  * IDS/IPS systems

### Types
#### Application Load Balancer

* Used for load balancing HTTP/HTTPS
* Operate at layer 7 (application layer)
* Supports advanced request routing based on the HTTP header

#### Network Load Balancer

* High performance
* Load balances TCP traffic
* Operates at layer 4 (network layer)
* Capable of handling millions of requests per second while mainting ultra-low latencies
* Most expensive option

#### Classic Load Balancer

* Supports layer 7 features, such as X-Forwarded-For and sticky sessions
* Supports layer 4 load balancing

### X-Forwarded-For header

Identifies originating IP address of client connecting through a load balancer

### Sticky Sessions

Requests from a specific client are forwarded to the same target

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

## Route 53

AWS DNS service

Allows to map domain names to IP addresses, EC2 instances, ALBs, S3 buckets...

Supported record types:
* A record type
* AAAA record type
* CAA record type
* CNAME record type
* DS record type
* MX record type
* NAPTR record type
* NS record type
* PTR record type
* SOA record type
* SPF record type
* SRV record type
* TXT record type

### Hosted zone

Container for DNS records

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

### Encryption

#### Encryption at Rest

* Enabled by default at creation time
* Uses KMS (AES-256)
* Includes all underlying storage, automated backups, snapshots, logs and read replicas
* Cannot be enabled on an unencrypted RDS instance
  * To encrypt an unencrypted DB, take an unencrypted snapshot, create an encrypted snapshot and restore to a new encrypted DB

### Deletion protection

* Disabled by default

### Multi-AZ and Read Replicas

#### Multi-AZ

Exact standby copy of database in another AZ

* Provides resilience
* Supported by all RDS types
* RDS will automatically failover to standby without intervention
* For DR, not for increasing performance

#### Read Replica

Read-only copy of your primary database

* Provides read performance
* Can be located in the same or another AZ as primary DB
* Can be cross-region
* Each read replica has its own DNS name
* Can be promoted to be their own DB
  * Breaks replication
* Requires automatic backup
* Up to 5 replicas are supported

### RDS Proxy Pooling

RDS Proxy pools and shares DB connections

* Serverless and scales automatically through pooling and sharing DB connections
* Preserves application connections (during failover)
* Detects failover and routes requests
* Deployable over Multi-AZ for protection
* Up to 66% faster failover times

## ElastiCache

Stores frequently accessed data in in-memory cache

* In-Memory Cache (Key Value)
* Improves database performance
* Great for read-heavy database workloads

Types:
* Memcached
  * Basic object caching
  * Scales horizontally
  * No persistance, Multi-AZ or failover
* Redis
  * Enterprise features like persistance, replication, Multi-AZ and failover
  * Supports sorting, ranking and complex data types like lists and hash
  
ElastiCache is a good choice if DB is read-heavy and not prone to frequent changing

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

* Allows to connect to AWS services using private network
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
  
# Beanstalk

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
* Encryption at rest - Server Side
  * SSE-S3 
    * AES 256
    * S3 managed keys
    * Enabled by default
  * SSE-KMS
    * KMS managed keys
  * SSE-C
    * Customer managed keys
* Encryption at rest - Client Side

## CORS

* Cross-Origin Resource Sharing is enabled on buckets
* Needs to be configured to allow resources in one bucket to access resources in another bucket

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

### Edge Locations

* Users access content from Edge Locations
* Spread around the world
* Keep a cache of copies of your objects

### Origins

An origin is the location where content is stored, and from which CloudFront gets content to serve to viewers. To specify an origin:

* Use S3OriginConfig to specify an Amazon S3 bucket that is not configured with static website hosting.
  
* Use CustomOriginConfig to specify all other kinds of origins, including:
  * An Amazon S3 bucket that is configured with static website hosting
  * An Elastic Load Balancing load balancer
  * An AWS Elemental MediaPackage endpoint
  * An AWS Elemental MediaStore container
  * Any other HTTP server, running on an Amazon EC2 instance or any other kind of host
  * Route 53 address

### Distribution

* Name given to origin
* Configuration settings for content you wish to distribute

### Origin Failover

### TTL (Time To Live)

* Objects are cached for a period of time which is defined by the TTL
* Cache can be invalidated (incurs fee)

### S3 Transfer Acceleration

* Enables fast transfer of date file over long distances through Edge Locations

### SSL/TLS - HTTPS

* Certificates need to be created in us-east-1 (N. Virginia)

### Viewer protocol policy

Options:
* HTTP and HTTPS
* Redirect HTTP to HTTPS
* HTTPS only

### Allowed HTTP methods

Options:
* GET, HEAD
* GET, HEAD, OPTIONS
* GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE

### Signed URLs

### Signed Cookies

### Origin Request policies

Options:
* Headers
* Query Strings
* Cookies

### Function Associations

* Viewer request
* Viewer response
* Origin request
* Origin response

### Price class

* All edge locaiton
* NA and Europe
* NA, EMEA

### AWS WAF

* Protects against web attacks 

### AWS Shield

* Protects against DDOS attacks

### Supported HTTP versions

* HTTP/2
* HTTP/3

### Geographic restrictions

* Allow or deny countries

### Origin Access Control

* Force all requests to go through Cloudfront instead of directly to your origin

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
## Athena

Enables you to run SQL queries on data stored in S3

* Serverless
* Pay per query and TB scanned
* Easy to use
* Integrated with S3

# Lambda

# DynamoDB

Serverless NoSql Database

# CodeBuild

# CodeDeploy

# CodePipeline

# CloudFormation

# SNS

# SQS

# Kinesis