---
title: "Implementing DNS SEC in AWS"
date: 2023-06-10T23:15:00+07:00
slug: dns-sec
category: AWS
summary:
description:
cover:
  image:
  alt:
  caption:
  relative: true
showtoc: true
draft: false
---
## Implementing DNSSEC using Route53
#### verify no DNS SEC is present
run command on AWS CLI 
`dig animlas4life.org dnskey +dnssec`
![Alt Text](../../images/Pasted%20image%2020230523213350.png)
#### Set up DNS SEC
Route53 > Hosted zones
![Alt Text](../../images/Pasted%20image%2020230523213251.png)
go inside `animals4life.org`

![Alt Text](../../images/Pasted%20image%2020230523213438.png)
DNSSEC signing > enable DNSSEC signing 
##### Create KSK
![Alt Text](../../images/Pasted%20image%2020230523213630.png)
#### Verify DNSSEC Keys
run command on AWS CLI 
`dig animlas4life.org dnskey +dnssec`
![Alt Text](../../images/Pasted%20image%2020230523213714.png)

#### Get KSK signing algorithm & public key
Route 53 > hosted zone > animlas4life.org > DNSSEC signing > view information to create DS record
![Alt Text](../../images/Pasted%20image%2020230523214049.png)
![Alt Text](../../images/Pasted%20image%2020230523214158.png)
#### Create train of trust with TLD
DNS > registered domains > DNSSEC Status
![Alt Text](../../images/Pasted%20image%2020230523213914.png)
![Alt Text](../../images/Pasted%20image%2020230523214353.png)
this will modify the TLD of animals4lyfe domain zone by adding DS record
this process might take a hour to complete. 
![Alt Text](../../images/Pasted%20image%2020230523214719.png)
![Alt Text](../../images/Pasted%20image%2020230530200455.png)
#### Verify DS record
Get TLD
`dig org NS +short`
![Alt Text](../../images/Pasted%20image%2020230523214819.png)

Results take a while to propagate
AUTHORITY SECTION is empty because it has not established a train of trust to the TLD
![Alt Text](../../images/Pasted%20image%2020230523214928.png)
Later, it get changed to ANSWER SECTION:
![Alt Text](../../images/Pasted%20image%2020230523215029.png)

![Alt Text](../../images/Pasted%20image%2020230530211247.png)
