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
- needs to be universally unique or unique gobally
- URL or web address created for it
- HTTP 200 code received if upload is successful

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

