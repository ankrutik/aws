# AWS Certified Solutions Architect Associate

[TOC]

# 10000ft view

Region: physical locations that contain >=2 AZs

Availability Zone: ~ discrete Data Center; redundant power, network

Edge locations: caching content; Amazon CDN

```
Region < Availability Zones < Edge Locations
```

# IAM - Identity Access Management

users; groups; policies; roles

Identity Federation: Use LDAP, Google, LinkedIn, Facebook, etc. for logins

multifactor auth

temporary access

PCI DSS: credit cards

## IAM dashboard

alias for account let's you use customized logins URLs

Users not locked to any region; always global.

5 ticks to be worked on. One of the important ones to setup MFA on the root account.

Password policies can be customized.

Users credentials can be downloaded to CSV at the time of

- creating the user
- editing password after user is created

***Credential Report* CSV download will not contain the passwords, so save the password CSV files or else you will have to regenerate them.**

User passwords:

- login password with MFA if enabled (login to console)

- access key id and secret access key (programmatic access to APIs)


## Create billing alarm

Inside CloudWatch, create alarm to check billing/cost

- ran at every configured period
- normal or detect anamoly
- SNS (simple notification service) topic
- go to email and subscribe to that topic

# S3

Object based storage. File size can be 0 bytes to 5 TB. Unlimited storage. Block based, so can't install OS.

## Bucket

- like a folder
- needs to be universally unique or unique gobally; universal namespace
- URL or web address created for it
- HTTP 200 code received if upload is successful
- by default newly created bucket is private
  - Access can be setup using:
    - bucket policies (**bucket level**)
    - Access Control Lists (**object level**)
- Access logs can be configured to be stored on another bucket in same or different account
- 100 buckets allowed per account

## Object

- key: name of the object
- value: content of the object in series of bytes
- version ID: used for version control
- metadata
- subresources: ACL, torrents

## Data consistency

read after write for PUTs

"eventual consistency" for overwrite PUTs and DELETEs means 

- if you overwrite an object and try to read immediately, you may get old version  
- takes a little time to propogate

## Guarantees

99.99% availability; 99.999..9% (11 x9) durability; Tiered storage; lifecycle management; versioning; encryption; multi factored auth for deletion; access control lists

## Path Styles

- Virtual style puts your bucket name 1st, s3 2nd, and the region 3rd.   
- Path style puts s3 1st and your bucket as a sub domain. 
- Legacy Global endpoint has no region.  
- S3 static hosting can be your own domain or your bucket name 1st, s3-website 2nd, followed by the region. 
- AWS are in the process of phasing out Path style, and support for Legacy Global  Endpoint format is limited and discouraged.  

## Storage classes

1. Standard

   99.99% availability, 99.x11 9 durability; stored redundantly across multiple devices/facilities; sustain loss of 2 facilities concurrently

2. Infrequently Accessed

   for data accessed infrequently, but need rapid access when needed; lower fee, but retreival fee applicable (previously reduced redundancy storage)

3. One Zone - Infrequently Accessed

   Lower cost than IA; multiple availability zones not needed

4. Intelligent Tiering

   Machine Learning used to understand how data is used and moved to that tier automatically

5. Glacier

   Low cost archive; store any amount of data; retrieval time configurable from hours to minutes

6. Glacier deep archive

   lowest cost archive; retrieval time is 12 hours

**See S3 tier comparison chart** https://aws.amazon.com/s3/storage-classes/#Performance_across_the_S3_Storage_Classes

**Read S3 FAQ** https://aws.amazon.com/s3/faqs/

## Charge criteria

Storage; requests; storage management pricing; data transfer pricing; 

transfer acceleration: transfer to edge location first then send to S3 bucket via Amazon's backbone network;

cross region replication pricing: replicating buckets across regions for HA or disaster recovery

## lab : Options when creating bucket

name is unique; select region; can clone

versioning maintained on same bucket; access logs; key-value pair tags; object level logging; encryption; cloudwatch metrics

blocking public access

Access to buckets can be setup via:

- bucket policies  (**bucket level**)
- ACLs (**object level**)

## Uploading files

Uploading files gives URL but URL cannot be used till object is made public

- bucket needs to be made public
- object needs to be made public

Storage class can be specified at object level

## Pricing Tiers

***exam: which tier to use for a scenarios; which is cheaper than the other***

### what are you charged for?

storage; requests and data retrievals; data transfer; management and replication

### comparisons notes

gets cheaper in order of: 
S3; S3 IA; S3 IT; S3 One Zone IA; S3 Glacier; Glacier deep archive

One Zone IA risks loss of data or unavailability if that one zone goes down.

#### S3 standard and intelligent tiering

same but intelligent tiering gives 

- access to infrequently accessed (IA) which makes IA objects cheaper to store
- management fee per 100 objects

Better to use Intelligent Tiering than S3 standard unless you have thousands/millions of objects.

## Encryption

### Encryption in transit

transfer from server to client is encrypted

achieved via SSL/TLS

### Encryption at rest

encrypted when stored; done per object; 2 forms

#### Server side

Amazon helps with encryption

Services

- S3 managed keys using AES-256 (SSE-S3)
- AWS Key Mgmt Service (SSE-KMS)
- with customer provided keys (SSE-C)

##### lab :

*click on object* > Encryption > *options to use SSE-S3 or SSE-KMS*

#### Client side

client encrypts files before uploading

## Versioning

stores all versions to restore even if deleted; 

each object version maintained with permissions, storage tiers;

used for backup;

once enabled cannot be disabled, only suspended;

lifecycle rules;

delete via MFA;

### lab :

*select bucket* > Properties > Versioning > *enable or suspend*



Upload a file to this bucket; make object public; it can be accessed via URL.

URL will be with `versionId` query param. Example, `https://<bucket_url>/fileName.txt?versionId=abcversionid123`

Upload file with same name and extension to the bucket; the object is replaced; **but new version of object is no longer public**; need to make it public again to be accesible via URL; **older version will still be public**



In Bucket view, toggle Versions. You can see different versions of the same object with different version ids. 

Note that the size used by that bucket is now cumulative size of both versions of the same file. **Versioning will use up full size of each revision unlike git where only difference is saved.**



Delete file; the bucket looks empty; toggle on version and you **can see the object and its versions but with a delete marker**

## lab : Lifecycle and Glacier

*bucket view* > Management > Lifecycle

rule name: name of the rule 

rule scope: filter the objects that this rule applies to or all objects

lifecycle rule actions: what to do to what version of object at what interval; transition between states, delete, etc.

Timeline summary is displayed as describing what happens to applicable objects as time passes.

- Automates moving your objects between different storage classes.
- can be used in conjuction ith versioning.
- can be applied to current and previous versions.

## Object lock and Glacier Vault Lock

Uses WORM (write once, read many) model where you can lock modification/deletion of the object for finite time or indefinitely.

Locks can be applied on individual objects or entire buckets.

### Modes

- governance: users with right/permissions can modify object
- compliance: object can't be modified for the retention period by even the root user; retention period cannot be shortened

**Retention period** is applied by storing a timestamp on the object's metadata.

**Legal holds** do not require retention period and object is blocked from modification till legal hold exists. It can be applied by users with legal hold permission.

Vault lock policies can be applied on Glacier vaults which once applied cannot be changed.

## S3 Performance

**Prefix** is path between bucket name and object name.

S3 has extremely low latency.

3500 requests PUT/COPY/POST/DELETE per second per prefix

5500 requests GET/HEAD per second per prefix

**If you spread out your object in separate prefixes, you can get that much high read rate.**

If you use **SSE-KMS**, then 

- read will need decrypt and writes will need encrypt. 
- This puts a limit on the objects that use SSE-KMS which is the KMS quota. 
- This KMS quota depends on regions and cannot be increased.

**Multipart uploads** parallelize uploads:

- recommended for files > 100MB
- required for files > 5GB
- application will have to handle splitting of the file before multipart uploads

**Byte range fetches** are download equivalents of above:

- parallelize downloads
- failure is only for that byte range
- used to download partial amount of the file

## S3 and Glacier Select

**S3 Select**: Achieve drastic performance increase by using simple SQL expressions to fetch subset of data from object. Save money on data transfer.

**Glacier Select**: Same as above but on Glacier. Used for customer that need storage compliance.

## AWS Organizations and Consolidated Billing

Multiple AWS accounts can be consolidated into an **AWS organization**. Access to services can be given to orgs which will trickle down to accounts.

**Consolidated billing** allows easy tracking of charges, allocating costs. Volume pricing discount applicable.

Go to Services > AWS Organizations
create organizations; add accounts by sending invites;

**Root account**: always enable multi factor authentication; strong, complex passwords

**Paying account**: use for billing only; no resources deployed here

**Service Control Policies** (SCP) can be used to control access to services on OU or individual accounts. 

## lab: Sharing S3 between accounts

3 ways to share buckets between accounts:

- Programmatic access
  - Bucket policies and IAM (across entire bucket)
  - ACLs and IAM (individual objects)
- Programmatic and Console access
  - Cross account IAM Roles 



Create Role > grant permissions > give Role name > copy and save link 

Log into other account > Accoutn drop down > Switch Role OR use link to open page that has Role information already filled in

## lab: Cross Region Replication

create a new bucket B > enable versioning

go to original bucket A > go to needed version > Create Replication Rule > IAM role > select Source bucket A (limit scope or entire bucket)  > Destination bucket B

Few configurations:

- Storage class for replicated objects
- Replication Time Control (replicate 99.99% objects in 15 minutes)
- metrics and events
- encrypt with KMS
- replicate delete markers (deletion operations are replicated, lifecycle rules do not apply)

Replication only starts working when Replication Rule is turned on. Existing files, if any, are not replicated.

Permissions in source bucket and destination bucket are separate.

Versioning should be enabled on both source and destination buckets.

Deleting individual versions or delete markers are not replicated.

## Tranfer Accelaration

- Uses CloudFront Edge Network
- Upload to Edge location (instead of directly to S3 bucket). Edge location transfers to S3 bucket.

S3 Tranfer accelaration tool to test giving comparison of upload speeds when done via various Edge location against direct S3 upload.

## AWS DataSync

- move large amount of data from **on-prem to AWS**
- NFS or SMB compatible
- hourly, daily, weekly
- DataSync agent needs to be installed
- replicate EFS to another EFS

## CloudFront

- CDN (content delivery network)
- distributed servers
- deliver webpages and other web content
- optimizes based on 
  - geographic location of user
  - origin of webpage
  - content delivery server

**Edge location**: where content is cache; separate to an AWS region

**Origin**: S3 bucket, EC2 instance, Elastic load balancer, Route53

**Distribution**: CDN which is a collection of edge locations

User's request is routed to nearest edge location. Content is cached at edge location till time-to-live.

Distribution types:

- web distribution: websites
- RTMP: media streaming

Read and write(Transfer Accelaration) on edge locations.

Cache can be cleared but you will be charged.

## lab: create a CloudFront distribution

Networking > CloudFront > create distribution > Web or RTMP

origin domain name; path; origin id; TTL (min, max); signed URLs/cookies only(for users that have paid for content); WAF

Can take 15 mins - 1h 

Deleting a distribition will need disabling first

Distributions come with their own domain name. Use that domain name to access your S3 content.

Invalidations can be created to remove objects from the Edge caches.

## Signed URLS and Cookies

Used to secure content by giving access to only authorized users, commonly those that have paid for the content.

When 1 file, use signed URL. When multiple files, use cookies.

### Policy 

contains:

- URL expiration
- IP ranges
- Trusted signed (which aws accounts can crete signed urls)

### Flow

- Client authenticates with application
- application returns signed URL to client
- client uses signed URL to access content from CloudFront
- Cloudfront uses OAI to fetch from S3

### CloudFront signed URLs 

- can have different origins, not just EC2 or S3
- key-pair is account wide; managed by root user
- caching
- filter by date, path, IP, expiration, etc.

### S3 signed URLs

- issue request as IAM user who has presigned it; all permissions as that IAM user
- limited lifetime

*If user doesn't have direct access to S3, S3 signed URL cannot be used. Use CloudFront signed URLs instead.*

## Snowball

Physically transfer files to S3. Import or export.

Snowball has 50 or 80 TB

Snowball Edge has 100TB; gives compute capabilities as well

Snowmobile has 100PB capacity

Consider using these capabilities according to your network speeds to upload/download from S3.

## Storage Gateway

Used to connect on-premise software appliances to cloud storage.

Can be a virtual machine image or a physical appliance.

File gateway (NFS and SMB): store files to S3; allows all features of S3 onto the stored fies/objects; 

Volume gateway (iSCSI): store images of harddisks on S3; incremental changes are stored for versioning; uses Amazon EBS snapshots

- stored: store primary data locally; asynchronously back up to S3 in form of EBS

- cached: S3 as primary data storage; retain frequently a cessed data locally in storage gateway

Tape gateway: store tapes in virtual tapes and move to cloud; archive data; VTL interface

## Athena vs Macie

### Athena

- interactive query service using which you can anlyse and query data on S3 using standard SQL
- serverless; pay per query OR per TB scanned
- no ETL setup
- can be use on log files, business reports, AWS cost and usage reports, click-stream data

### Macie

- Personally Identifiable Information (PII) eg home address, email addres, SSN, phone number, etc
- uses ML and NLP to discover, classify, protect PII
- works on data on S3
- analyze CloudTrail logs
- gives dashboards, alerts, reports
- PCI-DSS compliance; prevent ID theft

# EC2

- resizable compute capability in the cloud
- upscale and downscale capability as needed
- Amazon Elastic Compute Cloud



### Models

#### On demand

fixed rate by the hour/second; trying things out; no long term commitments

#### Reserved

capacity reservation; significant discount on hourly charge; 1 or 3 years terms

Standard: 75% off

Convertible: 54%; change instance type

Scheduled: time window when capacity will be used

#### Spot

bid whatever prices you want for instance capacity

when the price goes above your bid, the applications are taken down. If EC2 takes the application down, you will not be charged for that partial hour used. But if you take it down, you will be charged an hour.

usefully for when applications are flexible and feasible only at very low costs

#### Dedicated hosts

physical EC2 servers dedicated for your use

no multiple tenancy

existing software licenses that restrict deployment on limited number of servers

### Instance Types

Multiple EC2 Instance types as per your needs. For example, T3 is lower cost and general purpose for web servers and small DBs; M5 is for general purpose Application servers.

## lab: Setting up EC2

#### Storage
Root volume will be SSD and magnetic. Subsequent volumes can be of more types.
IPOS: input output per second can be configurable
Delete on terminate or not, EBS is delete by default
Encrypt or not

#### Termination

Configure what happens when you terminate: stop or terminate
Termination Protection (default off) doesn't allow the usual terminate button to work. Will have to disable termination protection to actually terminate.

#### Other


Tags
Security Groups: what protocol at what port and what IP
Key pair is created
Console can be access via a console which opens in the browser, EC2 Instance Connect



Connecting via console: 

```
ssh ec2-username@IPADDRESS_EC2 -i PrivateKey.pem
```



## Security Groups

All inbound traffic is blocked by default, all outbound is allowed.

Define which type of connection (eg. HTTP, SSH, etc) , which protocol (eg. TCP), which port (eg. 80, 22) is allowed inbound and outbound on the instance.

**If inbound is created for a port, the outbound is created automatically. Stateful.**

No way to blacklist. Only whitelist.

Multiple security groups can be applied to same instance.

## EBS 101 

Elastic Block Storage is a harddisk on the cloud.

### General Purpose (SSD)

Most workloads; gp2 API; <=16K IOPS

### Provisional IOPS (SSD)

Databases; io1 API; <=64K IOPS

### Throughput optimised HDD

Big data, warehouses; st1 API; <=500 IOPS

### Cold HDD

File servers; sc1 API; <=250 IOPS

### Magnetic

infrequently accessed data; Standard API; 40-200 IOPS

## lab: EBS Volumes

Volume will be in the same AZ as the EC2 instance.

Volume sizes can be changed, but will take time to reflect. May need to repartition the drive to detect the added size.

Hardware assisted virtualization

Snapshots are point in time copies of Volumes. Incremental nature i.e. only changes are stored on S3. First snapshot takes time to create.

**Migrating EBS overview**: Volumes to snapshots to AMI (Amazon Machine Image) then copy AMI to destination region then launch on another AZ

For root device (device with OS) snapshots, best pratice to stop the instance before taking snapshot.

## AMI types

### Selection criteria

- region

- OS

- arch (32/64-bit)

- lauch permission

- storage for root device

  - Types and criteria

    | criteria                                                     | instance store (ephemeral) | EBS Backed  |
    | ------------------------------------------------------------ | -------------------------- | ----------- |
    | what happens to data after restart due to host failure?      | deleted                    | not deleted |
    | can it be stopped?                                           | no                         | yes         |
    | can be configured to not delete after termination on instance? | no                         | yes         |
    | can be rebooted?                                             | yes                        | yes         |



## ENI vs ENA vs EFA

| criteria | ENI                                             | ENA                                                          | EFA                                                          |
| -------- | ----------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| meaning  | Elastic network Interface                       | Enhanced networking via Elastic Network Adaptor (100 Gbps) or Virtual Function (10 Gbps) | Elastic Fabric Adaptor                                       |
| use case | basic networking; low budget; multiple networks | reliable, high throughput                                    | high-performance computing, ML applications, OS by pass (linux only) |

## lab: Encrypted root device volumes and snapshots

When creating root volumes, you have the new option to configure encryption at creation time.

For existing volumes without encryption > create snapshot > create copy > during creation mention that it needs to be encrypted > create AMI > use AMI to launch instance

Cannot take an encrypted snapshot and launch as unencrypted

Snapshots can be shared only when unencrypted.

## Spot Instances and Spot Fleets

AWS has unused capacity. This capacity can be bid on. If your price matches below your maximum bid, the capacity is provisioned for your use. When the spot prices goes above your bid, your application is brought down.

Used when you need flexible, fault tolerant applications that can be brought down in 2 minutes notice and resume when brought up again -applications that need not be online all the time.

Can save upto 90% cost of On-Demand instances.

Spot blocks can be used to stop termination for 1 to 6 hours if spot price goes above your bid.

### Spot request

Contains

- maximum price
- desired number of instances
- launch specs
- request type (application went down due to spot price increase, what to do next time that spot price goes below bid?) :
  - one time: do nothing
  - persistent: bring application up again
- valid from and until

### Spot Fleets

Collection of spot instances and on demand instances

Will try to maintain target capacity within budget

## EC2 Hibernate

**What happens when EC2 instance starts**: OS boots up, User data scripts are run, application starts

**What happens on Hibernation**: saves contents of instance RAM into EBS root volume; persists root and data volumes; on restart RAM contents are reloaded, processes are resumed, OS need not restart; **instance ID is retained**;

Use cases for hibernate: long running processes; applications that take time to initialise 

RAM must be <150GB

Hibernation time cannot exceed 60 days

Available for on-demand and reserved instances

## CloudWatch

Monitoring performance of:

- Compute:
  - EC2 instances
  - autoscaling groups
  - Elsactic load balances
  - route53
- Storage
  - EBS
  - Storeage gateways
  - CloudFront

EC2 is 5 minutes by default, configurable to 1 minute intervals.

CloudWatch alarms can be created to trigger notifications.

Events help to respond to state changes.

Logs help wit aggregate, monitor, and store logs.

Host level metrics like CPU, Network, Disk, status check

### CloudTrail (Auditing)

User and resource activity monitored by recording AWS management console actions and API calls. Unrelated to CloudWatch.

## lab: CloudWatch

Dashboards, Alarms

## lab: AWS Command Line

Needs programmatic access in IAM

ssh to EC2 instance then use the aws command

```
> aws configure
-- to configure keys

> aws s3 ls
-- <aws command> <service> <command>
```

`.aws` folder contains credentials file. Best practice not to use configure and store in credentials file. See *IAM Roles*.

## lab: IAM Roles

Roles are more secure than stpring access keys and secret access keys on individual EC2 instances

Roles are easier to manage

Roles can be assigned to EC2 instance using both command line and console

Roles are universal.

## lab: Using Boot Strap Scripts

Configure instance details > Advanced Details > user data

```bash
#!/bin/bash
yum update -y
yum install httpd -y
service httpd start
chkconfig httpd on
cd /var/www/html
echo "Jai mata di" > index.html
aws s3 mb s3://<domain>
aws s3 cp index.html s3://<domain>
```

## EC2 Instance Metadata

 ```bash
curl http://<169.254.169.254>/latest/meta-data
curl http://<169.254.169.254>/latest/user-data
 ```

## lab: Elastic File System

Supports NFSv4

Multiple EC2 instances cannot share an EBS volume but can share a EFS volume.

Grows and reduces automatically, no preprovision of space needed. Pay for only what you use.

Supports thousands of concurrent NFS connections.

Data stored across multiple AZs within a region.

Read after write cosistency.

## FSX for Windows and FSX for Lustre

FSX for Windows: Windows file server designed for Windows based applications. SMB based.

FSX for Lustre: for HPC and ML applications. Can stored data directly on S3.

## EC2 Placement Groups

**Cluster Placement group** is grouping of instances within a single AZ. Applications that need low latency, high n/w thruput.

**Spread Placement Group** are group of instances each placed on distinct underlying hardware. Used when you have small number of critical instances that should be kept separate from each other.

**Partitioned Placement groups** divides each group into logical segment partitions such that each partition that its own rack (each rack has own network and power source). Used for HDFS, HBase, Cassandra.

## HPC on AWS

### Data Store

Snowball, AWS Data Sync, Direct connect

AWS Direct Connect is a dedicated line between from your premises to AWS rather than using Internet.

### Compute

EC2 instances that are GPU or CPU optimised; EC2 fleets; Cluster Placement groups; ENA; ENI; EFA

### Orchestration and Automation

AWS batch; AWS ParallelCluster

## AWS WAF

Web application firewall to monitor HTTP and HTTPS requests that are forwarded to CloudFront, Application Load balance or API gateway.

Control access to content.

Firewall on what IP addresses should be allowed or what query parameters should be passed.

Whitelist, Blacklist, counter.

Block access to geography, malicious IPs.

# Databases on AWS

On AWS

- Microsoft SQL Server
- Oracle
- MySQL Server
- PostgreSQL
- Amazon Aurora 
- MariaDB

Multi-AZ for disaster recevery

Read replicas for performance

DynamoDB (Amazon's no SQL solution)

Red Shift OLAP (Amazon's data warehousing solution)

Elasticache: MemcacheD, Redis; to speed up existing databases using caching

## Lab: create RDS instance

Configure your DB security group to talk with application security group when creating the RDS instance

RDS runs onn virtual machine, can't ssh into that

Not serverless

## Back Ups, Multi-AZ, and read replicas

Backup needs to be enabled for read replicas.

2 types of backup:

- automated backups
- database snapshots

Read replicas on read replicas are possible. Too many can bring latency.

- can be multi-AZ
- increase performance
- can be in different region
- MySQL, PostgreSQL, MariaDB, Oracle, Aurora
- can be promoted to master, this will break the read replica

MultiAZ:

- used for Disaster recovery
- force failover from one AZ to another by rebooting RDS instance

Encryption available for all 6 DB types. Uses KMS. Automated backup, read replicas, snapshots are encrypted too.

## DynamoDB

single-digit millisecond latency

supports both documents and key-value data models

uses SSDs

spread across 3 distinct geographic locations

eventually consistent reads: consistency across all copies reached within 1 second; default

strongly consistent reads: returns result that reflects all writes that received successful response prior to read; sub 1 second

## Advanced DynamoDB

### DynamoDb Accelarator (DAX)

in-mem cache; 10x performance; microseconds response; no need for devs to manage separate caching logic

### Transactions

Multiple "all or nothing" operations like in financial transactions or fulfilling orders

2 underlying reads/writes: prepare and commit

upto 25 items or 4MB of data

### On-Demand Capacity

pay-per-request pricing

balance cost and performance

no minimum capacity

**use for new product launches, then observe and switch to provisioned according to waht you learned with on-demand**

### On-Demand backup and Restore

Full backups at any time

0 impact on table performance and availability

consistent within seconds, retained until deleted

same region as source table

### Point in time Recovery

protects against accidental writes or deletes

restore to any point in the last 35 days to 5 minutes 

incremental backups

not enabled by deafult

### Streams

time-ordered **sequence** of item-level **changes** in a table

inserts, updated, deletes

stored for 24 hours

stream records grouped into shards

combine with lambda functions for functionality like stored procedures

### Global Tables

globally distributed appls

multi region redundancy for DR or HA

no application rewrites

replication latency under 1 second

needs streams enabled

### Database migration service (DMS)

Source and destination DBs can be on prem or on AWS (EC2 or RDS)

### Security

KMS used for encryption

site-to-site VPN, Direct Connect

IAM policies and Roles

Fine-grained access

CloudWatch and CloudTrail

VPC endpoints

## Redshift

petabyte scale data warehouse service

single node config (160Gb)

multi-node config:

- leader node: manage client requests
- compute nodes: store data and perform queries and computations

Individual columns are compressed

Massively parralel processing

backups:

- enabled by default
- maximum retention of 35 days
- maintains at least 3 copies of data
- async replicate your snapshots to S3 in another region

Pricing:

- compute node hours: 1 unit per node per hour; leader node is not charged
- backup
- data transfer

Security

- ssl
- aes-256 encryption
- KMS

Only available in 1 AZ currently

## Aurora

MySQL and PostgreSQL compatible RDS engine

Starts at 10GB, scales in 10GB increaments, upto 64TB

compute resources upto 32vCPUs and 244GB RAM

2 copies of data in each AZ, minimum 3 AZs

share snapshots with other AWS accounts

types of aurora replicas:

- aurora replicas (15)
- MySQL read replicas (5)
- PostgreSQL (1)

autmated backups on by default

serverless

suited for infrequent, intermittent, or unpredictable workloads

## Elasticache

web service to deploy, operate, and scal in-memory cache in the cloud

supports memcached and redis

memcached: simple, multithreaded performance

redis: advanced data type, sorting

redis is multi AZ and allows back/restores

## Database Migration Service

homogenous (eg Oracle to Oracle) or heterogenous (eg. Oracle to Aurora) migration

Heterogenous needs Schema Conversion Tool

## Caching strategies

Services that have caching capabilites:

- CloudFront
- API Gateway
- Elasticache
- DynamoDB Accelarator

Add caching more towards the front to decrease latency.

## Elastic Map Reduce

Big Data scenarios

Cluster components:

- Master node: manage and track status of tasks; monitor health of cluster; at least one in each cluster
- core node: runs tasks and stores data on Hadoop Distributed File System
- task node: only runs tasks and does not store data; optional

Master node stored log data. 

replication can be configured to replicate log data every 5 mintues on S3; can on;ly be set up during initial creation

# Advanced IAM

## AWS Directory Service

AWS Managed Microsoft AD

Simple AD

AD Connector

Cloud Directory

Cognito user pools

AD compatible:

-  microsoft AD
- ad connector
- simple ad

Not AD compatible:

- cloud directory
- cognito user pools

AD Trust is used to extend AWS AD onto your on-prem.

## IAM policies

AWS policies are policies created by AWS for convienences. Not editable.

Customer managed policies are used for customizations.

If not explicitly allowed, it is implicitly denied

Policies only work when applied to a group or role.

AWS joins all applicable policies

Permission boundaries

- used to delegate administration to other users
- prevent privilege escalation or unnecessarily broad permissions
- control maximum permissions that a IAM policy can apply

## Resource Access Manager

resource sharing between accounts

only some services can be shared

## AWS Single Sign-On

centrally manage access to AWS accoutns and business applications

# Route53

## DNS 101

DNS is used to convert human readable domain names to IPv4 (32 bits) or IPv6 (128 bits) addresses.

.com .edu .co.uk is top level domain name

Elastic Load balancers do not have pre-defined IPv4 addresses, uyou resolve them using a DNS name

**Top level domain names** controlled by Internet Assigned Numbers Authority (IANA)

A **registrar** (Amazon, GoDaddy) assigns domain names under one of the top level domain names. Domains are registered with interNIC (service of ICANN) which enforees uniqueness of domain names across the Internet. Registered with a central database called WhoIS database.

**SOA** (start of authority) records stores:

- name of server that supplied data for zone
- administrator of zone
- current version of the data file
- default TTL seconds

**NS** (name server) records used by top level domain servers to direct traffic to the content dns server which contains authorative DNS records.

User enters URL -> Browser goes to top level domain -> TLD returns NS record -> Browser looks up NS record to get SOA record -> SOA record contains DNS records

- **"A" Record**: Address record; used to convert name of domain to an IP address
- **TTL**
- **CName** (canonical name): Used to resolve one domain to another; instead of using multiple IP address, use one IP and have 2 domain names on it. CNAME can't be used for naked domain name (`http://google.com`). It must be either an "A" record or alias.

- **Alias** records are used to map resource records sets in your hosted zone to ELB, CloudFront distributions, or S3 buckets. Alias records are like CNAME records.
- **PTR** records are used to look up domain names for IP addresses
- **MX** records are used for mail

## lab: Register a domain name

domain registration can be bought from the AWS console for hosted zones (.com addressed for 20USD)

can take upto 3 days

_3 EC2 instances were brought up in different regions with different IP addresses and different index.html contents. One domain name was registered (hosted zone). Route 53 was used to configure the 3 different IP address to the same domain name (hosted zone) to try out the following routing policies. Hit the address, wait for DNS flush (TTL) or manually flush, hit again and see if it DNS resolve a different IP address._

50 domain names can be managed using the AWS console but this limit can be increased by contacting AWS support.

Records are created for each IP in Route 53 > Hosted zones.

Health checks can be configured for each IP; can configure notifications.

## Route53 Routing policies available on AWS

### Simple

one record with multiple IP adddresses

for multiple values in record, Route 53 will return all values in random order

### Weighted 

split traffic based on different weights assigned to each IP address

Heath checks can be created and configured with records in your hosted zones.

### Latency based

route traffic based on lowest network latency for your end user i.e. route to region that will give them the fastest response time

Latency records are created for each IP address. Regions are automatically detected when IP addresses are entered.

### Failover

active and passive site setup. health checks are configured with the record set for the active site which help route to passive site incase the active's health goes down.

### Geolocation

routing of traffic based on geographical location of your user. Not to be confused with latency based, here we are restricting users to a site based on their geograhical location and not worrying about latency.

### Geoproximity (Traffic flow only)

routing based on geographical location of your user and your resources. 

optionally choose to route more or kess to a given resource based on a bias value. 

bias expands or shrinks the **size of the geographical location from which traffic is routed** to a resource. 

must use Route 53 traffic flow

### Multivalue Answer

same as simple routing policy (one domain name returns multiple IP addresses) but associate health check with each record. this policy will then only return those records that are healthy.

 # VPC

Virtual private cloud let's you provision a loggically isloated section of AWS services. Select your own IP address range, create subnets, configure routing tables and network gateways.

For example, configure web server instances on public subnets and DB instances on private subnet with no Internet access.

Hardware VPC between corporate DC and your VPC. Leverage AWS cloud as extension of your corporate DC.

Internet/Virtual Private gateway -> Router -> route table -> Network ACL -> Public/Private subnet (security group applied)

Bastion subnet is when you connect to a public subnet to connect to a private subnet.

CIDR blocks: 10.0.0.0/16 subnet is the largest allowed. 10.0.0.0/28 is the smallest.

Bring up instances in subnet of your choice. Attach network Access Control Lists (ACLs) to control access to those subnets.

Default VPC has subnets that connect to Internet.

VPC peering uses private IP address to allow one VPC to connect to another via a direct network route. Uses star configuration (1 central with *n* others) not transitive.

1 subnet = 1 AZ

Security groups are stateful; Network ACL are stateless.

## lab: build a custom VPC

Networking > VPC

**Create VPC**: name, IPv4 CIDR block, IPv6 CIDR, tenancy

Above creates **route table**, **network ACL** and **security group** by default.

**Create subnet**: name, VPC, AZ, IPv4 CIDR block, IPv6 CIDR block

can create multiple subnets for each VPC

5 IP addresses are reserved by Amazon (10.0.0.0-3, 10.0.0.255)

**Modify auto-assign IP settings**

**Create Internet Gateway**: name

attach IG to VPC; allows only 1 IG to each VPC

Best practice, keep main route table private.

**Create route table**: name, VPC

Edit routes and connect outbound IPv4(0.0.0.0/0) and IPv6(::/0) to required IG.

Subnet association: connect subnet to required route table

When creating Instance from AMI, select:

- network IG
- subnet
- auto assign public IP address if needed (eg disable for DB instances)

Security groups are limited to one VPC. Cannot be used in multiple VPCs.

Create security group for private subnet such that it will accept requests from within our public subnet i.e. our application instances talking to our DB instance.

create security group for private subnet: name, VPC, inbound rules {ICMP IPv4, public subnet}, {HTTP, public subnet}, {HTTPS, public subnet}, {SSH, public subnet}, {MySQL/Aurora, public subnet},
allow all outbound rules

Change security group of DB instance to this new security group.

Copy private key of DB instance into App instance. Then ssh into app instance and then ssh into db instance. Alternative to this is to use bastion hosts.

## lab: NAT instances and NAT Gateways

Network Address Translation

These instances allow private subnets to communicate to the Internet. Eg when you want to run yum to update/patch software within instances on a private subnet.

NAT instance is a single EC2 instance. NAT gateways are multiple instances spread across multiple AZ with high availability.

### NAT Instance

NAT instance must be in a public subnet

Should be a route out of the private subnet to the NAT instance.

Bring up a NAT instance from the one available under Community AMIs. Bring it up under the same VPC. Disable source/destination check.

Create a new Route such that destination is 0.0.0.0/0 (all outbound traffic on all port) and target is our NAT instance.

Issues:

- multiple EC2 instances using the same NAT instance can overwhelm it, can increase isntance size
- can create HA by using autoscaling groups, multiple subnets in different AZs, and script to automate failover
- behind security groups

### NAT Gateway

Create a new Gateway: subnet mention public subnet, create new elastic ip allocation ID

Edit route table: edit main, destination is 0.0.0.0/0 (all outbound traffic on all port) and target is our NAT gateway

NAT gateways cannot span AZs. Redundant inside the AZ.

thruput is 5Gbps to 45Gbps

no need to patch

not associated with security groups

automatically assigned IP addresses

no need to disable source and destination checks

When you have one NAT Gateway being used by instances in multiple AZs, then failure of that NAT gateway's AZ will block others. Better to bring up multiple NAT gateways in multiple AZs.

## Access Control Lists

When a VPC is created, a Network ACL is created by default. 

When you add a subnet to a VPC, it is added to the default NACL. Subnets should always be associated with a NACL.

By default, a NACL denies everything inbound and outbound.

A subnet can be associated with only one NACL at any given time. But a NACL can have multiple subnets associated with it.

NACL > Edit Subnet associations:  to add or remove subnets from NACL

The rule sequence matters since it checks the first passing rule, the rules are evaluated in ascending order of rule number starting with 100. Deny rules need to be before Allow rules.

Ephemeral ports are a range of ports that are randomly allocated to incoming sessions to server. The port is then used for the lifetime of the communication session.

NACLs are stateless i.e. inbound and outbound rules need to be defined separately, creation of inbound rule doesn't create corresponding outbound rule.

Internet Gateway > Route Table > **NACL** > Security Group

### Contrast with Security groups

- NACLs act before Security Groups.
- NACLs can have IP block lists
- NACLs are at subnet level while Security Groups are at instance level

## lab: Custom VPCs and ELBs

When provisioning a Load Balancer, you need at least 2 public subnets.

## lab: VPC Flow Logs 

logs information on IP traffic going to and from your network interfaces

logs data using Amazon CloudWatch

VPC > Actions > Create Flow Log: accepted/rejected traffic, CloudWatch Logs/ S3 bucket (CloudWatch log group will need to be created); IAM role

Peered VPC need to be in your account for flow logs on them.

Tags on flow logs.

Can't reconfigure flow logs after creation eg. changing IAM role

IP traffic that is not monitored:

- Amazon DNS. If using own DNS server, that will be logged
- Amazon Windows license activation
- 169.254.169.254 for instance metadata
- DHCP traffic
- to reserved IPS address for default VPC router

### Levels

- VPC
- subnet
- network interface

## Bastions Hosts

Special purpose computer specifically designed and configured to withstand attacks. Eg. hosts a single application like a Proxy server and all services are removed to the bare essentials to reduce threat to the computer. Can be outside the firewall or inside a DMZ and involves access from untrusted networks/computers.

NAT Gateway/Instance is used to provide Internet traffic to EC2 instances in a private subnet.

Bastion/Jumpbox is used to securely administer EC2 instances using SSH or RDP.

NAT gateway cannot be used as a bastion host.

Bastion hosts community AMIs are available.

## Direct Connect

Private connection between AWS and your DC/office/colocation environment. Helps reduce network costs, increase bandwidth, improve latency.

### Setting up Direct Connect

- create **public virtual interface** in DC console
- VPC > VPN connections > create **customer gateway**
- create **virtual private gateway**
- attach virtual private gateway to desired VPC
- VPC > VPN connections > create new **VPN connection**
- select virtual private gateway and customer gateway
- once VPN is available, setup VPN on customer gateway/firewall

## Global Accelerator

Directs traffic to the optimal endpoint in terms of proximity to client, health, endpoint weights

Components:

- static ip addresses: AWS brings 2 IPs or you can bring your own
- accelerator
- DNS name: assigns you a DNS name
- network zone: isolated unit with own physical infrastructure
- listener: port, TCP/UDP, client affinity
- endpoint group: regions with **traffic dials**
- endpoint: the EC2 instance or application load balancer or network load balance or Elastic IP address, **weights**

Disable, wait for some time, delete

## VPC endpoints

privately connection your VPC to **only certain supported** AWS services and VPC endpoint services without without internet gateway, NAT, VPN, Direct Connect. 

No public IP needed.

Traffic doesn't leave amazon network.

Instance in private subnet sends message to VPC endpoint gateway. The gateway then sends the message  to the AWS service.

Endpoints are virtual devices.

Types of endpoints:

- interface
- gateway: S3, DynamoDB

## AWS PrivateLink

Ways to connect to other VPC:

- open up to Internet via Internet Gateway: problems with security, have to setup firewall, etc.
- use VPC peering: will have to create that many star config connections and gets hard to manage for large number of peers

Use PrivateLink instead:

- network load balancer on service side
- Elastic network interface (ENI) on customer side

Take static IP of network load balancer and open it up on the ENI

Useful to connect your VPC to 10s, 100s, 1000s of customer VPCs.

No VPC peering, NAT, Internet gateways, route tables, etc.

## AWS Transit Gateway

Network topologies can get very complicated.

Transit gateway is "**hub-spoke**" like topology to **simplify**.

Transitive peering between 1000s of VPC and on-prem DCs.

Cross regional

Across multiple AWS accounts using Resource Access Manager.

Use route tables to limit communication.

Works with Direct Connect, VPN.

Supports **IP multicast** (not supported by any other AWS service).

## AWS VPN CloudHub

If you have **multiple sites each with its own VPN**, you can use AWS VPN CloudHub to connect these sites.

Hub-spoke model

Operates over Internet, but customer gateway to AWS VPN CloudHub is encrypted (since it's a VPN).

## AWS Network Costs

- use private IP address over public IP address to save costs. this uses AWS backbone network.
- to cut network costs, have all your EC2 instances in the same availability zone and use private IP address. 100% cost free but has **single point of failure issues**.



