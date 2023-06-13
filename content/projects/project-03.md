---
title: "AWS Macie Demo"
date: 2023-06-13T12:00:01-05:00
slug: aws-macie-demo
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

# Amazon Macie
Goal: use Amazon Macie to identify sensitive information stored in an S3 bucket and notify subscribers
**Credit cards - cc.txt**
```js
American Express
5135725008183484 09/26
CVE: 550

American Express
347965534580275 05/24
CCV: 4758

Mastercard
5105105105105100
Exp: 01/27
Security code: 912
```

**Employee information - employees.txt**

```js
74323 Julie Field
Lake Joshuamouth, OR 30055-3905
1-196-191-4438x974
53001 Paul Union
New John, HI 94740
Amanda Wells

354-70-6172
242 George Plaza
East Lawrencefurt, VA 37287-7620
GB73WAUS0628038988364
587 Silva Village
Pearsonburgh, NM 11616-7231
LDNM1948227117807
Brett Garza
```
**Access credentials - keys.txt**

```js
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
AWS_SESSION_TOKEN=AQoDYXdzEPT//////////wEXAMPLEtc764bNrC9SAPBSM22wDOk4x4HIZ8j4FZTwdQWLWsKWHGBuFqwAeMicRXmxfpSPfIeoIYRqTflfKD8YUuwthAx7mSEI/qkPpKPi/kMcGdQrmGdeehM4IC1NtBmUpp2wUE8phUZampKsburEDy0KPkyQDYwT7WZ0wq5VSXDvp75YU9HFvlRd8Tx6q6fE8YQcHNVXAkiY9q6d+xo0rKwT38xVqr7ZD0u0iPPkUL64lIZbqBAz+scqKmlzm8FDrypNC9Yjc8fPOLn9FX9KSYvKTr4rvx3iSIlTJabIQwj2ICCR/oLxBA==
github_key: c8a2f31d8daeb219f623f484f1d0fa73ae6b4b5a
github_api_key: c8a2f31d8daeb219f623f484f1d0fa73ae6b4b5a
github_secret: c8a2f31d8daeb219f623f484f1d0fa73ae6b4b5a
```

**Custom data (Australian licence plates) - plates.txt**

```js
# Victoria
1BE8BE
ABC123
DEF-456

# New South Wales
AO31BE
AO-15-EB
BU-60-UB

# Queensland
123ABC
000ZZZ
987-YXW
```
![Alt Text](../../images/Pasted%20image%2020221009164844.png)

## Set up SNS for email alerts
### Create SNS alerts
Amazon SNS > Topics > Create Topic
![Alt Text](../../images/Pasted%20image%2020230515191059.png)

![Alt Text](../../images/Pasted%20image%2020230515191257.png)

![Alt Text](../../images/Pasted%20image%2020230515191334.png)

### Create a subscription
SNS > Subscriptions > create Subscription
![Alt Text](../../images/Pasted%20image%2020230515191443.png)


### Set up event bridge
![Alt Text](../../images/Pasted%20image%2020230515191553.png)
##### define rule detail
![Alt Text](../../images/Pasted%20image%2020230515191608.png)
##### build event pattern
![Alt Text](../../images/Pasted%20image%2020230515191634.png)
	![Alt Text](../../images/Pasted%20image%2020230515191650.png)
##### Select Targets
![Alt Text](../../images/Pasted%20image%2020230515191744.png)
## Macie
#### Create Custom Data identifiers
![Alt Text](../../images/Pasted%20image%2020230515191847.png)
#### Create Job
Amazon Macie > S3 Buckets > create jobs
![Alt Text](../../images/Pasted%20image%2020230515190518.png)
##### refine the scope
![Alt Text](../../images/Pasted%20image%2020230515190538.png)
##### select data identifiers
![Alt Text](../../images/Pasted%20image%2020230515190600.png)
##### select custom data identifiers
![Alt Text](../../images/Pasted%20image%2020230515191924.png)
### Findings of completed jobs
![Alt Text](../../images/Pasted%20image%2020230515191955.png)



Sample of some terraform commands. This is not a complete version. 
```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.27"
    }
  }

  required_version = ">= 0.14.9"
}

provider "aws" {
  profile = "default"
  region  = "us-east-1"
}

# Create an SNS topic
resource "aws_sns_topic" "macie_alert_topic" {
  name = "macie-alert-topic"
}

# Create SNS topic policy to allow Eventbridge to publish to the SNS topic
resource "aws_sns_topic_policy" "default" {
  arn    = aws_sns_topic.macie_alert_topic.arn
  policy = <<POLICY
{
  "Version": "2008-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "events.amazonaws.com"
      },
      "Action": "sns:Publish",
      "Resource": "${aws_sns_topic.macie_alert_topic.arn}",
      "Condition": {
        "ArnEquals": {
          "aws:SourceArn": "${aws_cloudwatch_event_rule.macie_alert_rule.arn}"
        }
      }
    }
  ]
}
POLICY  
}

# Create an EventBridge rule to trigger the SNS topic when MACIE findings have severity above or equal to "MEDIUM"
resource "aws_cloudwatch_event_rule" "macie_alert_rule" {
  name        = "macie-alert-rule"
  description = "Trigger SNS alert for MACIE findings with severity above or equal to MEDIUM"

  event_pattern = <<PATTERN
{
  "detail-type": ["Macie Findings"],
  "detail": {
    "severity": ["MEDIUM", "HIGH", "CRITICAL"]
  }
}
PATTERN
}

# Add a target to the EventBridge rule to send the event to the SNS topic
resource "aws_cloudwatch_event_target" "macie_alert_target" {
  rule      = aws_cloudwatch_event_rule.macie_alert_rule.name
  target_id = "macie-alert-target"

  arn = aws_sns_topic.macie_alert_topic.arn
}


# display the Name and ARN of the SNS topic
output "SNS-Topic" {
  value       = aws_sns_topic.macie_alert_topic.name
  description = "The SNS Topic Name"
}

output "SNS-Topic-ARN" {
  value       = aws_sns_topic.macie_alert_topic.arn
  description = "The SNS Topic ARN"
}
```


