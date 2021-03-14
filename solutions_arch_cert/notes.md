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



