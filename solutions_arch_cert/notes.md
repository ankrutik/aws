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

