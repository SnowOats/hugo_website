---
title: "AWS Cloud Resume Challenge"
date: 2022-04-05T23:15:00+07:00
slug: aws-cloud-resume-challenge
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
# Part 1 set up SSO, S3, Route 53, and Cloudfront
## Set up Organization and SSO
### Create Organizational accounts
[Best practices guide for setting up AWS Organization](https://github.com/org-formation/org-formation-cli/blob/master/docs/articles/aws-organizations.md):

[(Optional) use OrgFormation / AWS Control Tower with AWS SSO](https://github.com/org-formation/org-formation-cli/blob/master/docs/articles/org-formation.md) This is best used for enterprise set up

[SSO and Organization set up Guide](https://dev.to/aws-builders/minimal-aws-sso-setup-for-personal-aws-development-220k)
## Set up billing alerts
#### Creating a budget
Open the AWS Billing console at [https://console.aws.amazon.com/billing/](https://console.aws.amazon.com/billing/home?#/).
or select Select Billing Dashboard from your account or type `Billing` on Search bar
![Alt Text](../../images/Pasted%20image%2020221009164514.png)
In the navigation pane, choose **Budgets**.\
Create a budget
![Alt Text](../../images/Pasted%20image%2020221009164844.png)
if you don't have cost explorer enabled already, enable it now. 
![Alt Text](../../images/Pasted%20image%2020221009165153.png)
Select the monthly cost budget
![Alt Text](../../images/Pasted%20image%2020230531152459.png)
Fill in the details for the budget and create the budget
![Alt Text](../../images/Pasted%20image%2020230531152543.png)
#### Set up billing alerts
In the navigation pane, Preferences > Billing preferences.\
Enroll in receiving and invoice along with alerts
![Alt Text](../../images/Pasted%20image%2020221009164702.png) 
#### To enable the monitoring of estimated charges
Open the AWS Billing console at [https://console.aws.amazon.com/billing/](https://console.aws.amazon.com/billing/home?#/).\
In the navigation pane, choose **Billing Preferences**.\
By **Alert preferences** choose **Edit**.
![Alt Text](../../images/Pasted%20image%2020230605122733.png)
Choose **Receive CloudWatch Billing Alerts**.
Choose **Save preferences**.
![Alt Text](../../images/Pasted%20image%2020230605122758.png)
## Set up Access key and verify
#### Creating Access keys for IAM user
1) Go to IAM
2) under navigation pane, **Access Management > users**
3) Choose to view a user by clicking on their name
4) Select `Security Credentials` tab and scroll down to Access Key
![Alt Text](../../images/Pasted%20image%2020230531153517.png)
4) Create the access key
![Alt Text](../../images/Pasted%20image%2020230403233243.png)
#### Creating access keys for the AWS account root
1. Choose your account name in the navigation bar, and then choose **My Security Credentials**.
	![Alt Text](../../images/Pasted%20image%2020230412192125.png)
Expand the Access keys (access key ID and secret access key) section.
Choose **Create New Access Key**.
#### AWS CLI
download [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) and launch it
type  `aws configure --profile iamadmin-general` 
Replace `iamadmin-general` with the name of your profile.
![Alt Text](../../images/Pasted%20image%2020230403233450.png)
verify if account is configured successfully: `aws s3 ls --profile iamadmin-general`
![Alt Text](../../images/Pasted%20image%2020230403233507.png)
[AWS Documentation](https://docs.aws.amazon.com/accounts/latest/reference/root-user-access-key.html)
## Deploy Static Website to AWS S3 with HTTPS using CloudFront
### Set up S3
Create two buckets. One for the domain name and another starting with the prefix (www.)
![Alt Text](../../images/Pasted%20image%2020230605124226.png)
### Route 53 Domain registration and hosted zones
open up https://us-east-1.console.aws.amazon.com/route53/v2/home
on the navigation pane, select **Registered domains**\
Register a custom domain with Route 53 or transfer one in
![Alt Text](../../images/Pasted%20image%2020230605124148.png)
on the navigation pane of Route 53, select **Hosted zones**\
Create a new hosted zone by providing the domain name and ensure it is set to type **public hosted zone**
![Alt Text](../../images/Pasted%20image%2020230605140535.png)
### Adding CDN to S3
[Reference](https://docs.aws.amazon.com/AmazonS3/latest/userguide/website-hosting-cloudfront-walkthrough.html)
CDN is needed to speed up website, enforce HTTPS only, and block public access to S3 by using origin control.
#### Getting a Secure Sockets Layer (SSL) Certificate
ACM is used to secure a website from HTTP to HTTPS

Search for ACM or Amazon Certificate Manager in the search bar and click on certificate manager in the AWS management console

Request for a new certificate
![Alt Text](../../images/Pasted%20image%2020230605141101.png)
Request for public certificate
![Alt Text](../../images/Pasted%20image%2020230605141115.png)
Enter for the fully qualified domain name
`*.humailkhan829.net`
This certificate will secure any subdomain of *humailkhan829.net*, such as *sub.humailkhan829.net* or *test.humailkhan829.net*.
#### Creating distribution
Go to cloud front > Distribution > create distribution
![Alt Text](../../images/Pasted%20image%2020230605141836.png)
Specify the name, and ensure origin access control setting is enabled
![Alt Text](../../images/Pasted%20image%2020230605142003.png)
Under **Default cache behavior > Viewer**, enforce HTTPS only
![Alt Text](../../images/Pasted%20image%2020230605142048.png)
Under **Settings**, Specify custom domain names and SSL cert
![Alt Text](../../images/Pasted%20image%2020230605142312.png)
### Create CNAME redirect and Cloudfront redirect
open up https://us-east-1.console.aws.amazon.com/route53/v2/home\
on the navigation pane of Route 53, select **Hosted zones**\
Selected hosted zone and view details\
Create these type of records 
1) route for the domain to route to cloud front
![Alt Text](../../images/Pasted%20image%2020230605143002.png)
2) redirect any sub domain to root domain with CNAME record
![Alt Text](../../images/Pasted%20image%2020230605143210.png)
Also, the ACM should populate a CNAME entry like the following
![Alt Text](../../images/Pasted%20image%2020230605143416.png)
if not, go to your certificate under ACM and create a record using the CNAME name and CNAME value
![Alt Text](../../images/Pasted%20image%2020230605143656.png)
### Errors
#### Changes to S3 is not reflecting to CloudFront
Invalidate Cloudfront cache
https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Invalidation.html#invalidating-objects-api
`aws cloudfront create-invalidation --distribution-id E3HKHQS3APLTXO --paths "/*"`
![Alt Text](Bins/Images/../../images/Pasted%20image%2020230202130333.png)

#### Permission denied error
Check to see if name servers for host zone is the same for the registered domain

Route 53 > Registered Domain > select domain
![Alt Text](../../images/Pasted%20image%2020230516222217.png)
Route 53 > Hosted Zones 
![Alt Text](../../images/Pasted%20image%2020230516222243.png)

#### S3 bucket blocked public access is still public
![Alt Text](../../images/Pasted%20image%2020230529151206.png)
Caching problem. Wait a while and refresh the page. 

#### Cloudfront OAI not accessing subdomain correctly
There is a feature in CloudFront called the [default root object](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/DefaultRootObject.html) that allows you to specify an index document that applies to the root object only, but not on any subfolders.

if a user goes to `www.example.com/blog`, this request is no longer on the root directory, and therefore CloudFront does not rewrite this URL and instead sends it to the origin as is

Lambda Function
```js
exports.handler = (event, context, callback) => {
  const request = event.Records[0].cf.request;

  let prefixPath; // needed for 2nd condition

  if (request.uri.match('.+/$')) {
    request.uri += 'index.html';
    callback(null, request);
  } else if (prefixPath = request.uri.match('(.+)/index.html')) {
    const modifiedPrefixPath = prefixPath[1].replace(/^\/+/, '/');
    const response = {
      status: '301',
      statusDescription: 'Found',
      headers: {
        location: [{
          key: 'Location', value: modifiedPrefixPath + '/',
        }],
      }
    };
    callback(null, response);
  } else if (request.uri.match('/[^/.]+$')) {
    const modifiedRequestURI = request.uri.replace(/^\/+/, '/');
    const response = {
      status: '301',
      statusDescription: 'Found',
      headers: {
        location: [{
          key: 'Location', value: modifiedRequestURI + '/',
        }],
      }
    };
    callback(null, response);
  } else {
    callback(null, request);
  }
}
```
add trust relation for lambda function
```json
{
	"Effect": "Allow",
	"Principal": {
		"Service": "edgelambda.amazonaws.com"
	},
	"Action": "sts:AssumeRole"
}
```
Add Trigger for cloud front and set up lambda edge for origin request
![Alt Text](../../images/Pasted%20image%2020230529162036.png)
# Set up DynamoDB, lambda api
### Create dynamoDB table item
![Alt Text](../../images/Pasted%20image%2020230529182253.png)
### lambda function
```python
import json
import boto3
import os
table_name = os.environ.get('DB_TABLE')
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(table_name)
def lambda_handler(event, context):
    response = table.get_item(Key={
        'id':'0'
    })
    views = response['Item']['views']
    views = views + 1
    print(views)
    response = table.put_item(Item={
            'id':'0',
            'views': views
    })

    return views
```
You can create the environment variable under Configuration > Environment variables\

#### 'Internal Server Error' when calling lambda function to update dynamoDB
Missing permission. 
Lambda function > Permission > execution role
![Alt Text](../../images/Pasted%20image%2020230529171333.png)
Attach the following policies
![Alt Text](../../images/Pasted%20image%2020230529171309.png)

### API Gateway
#### Set up API call
on the navigation pane, select **Resources**
Click Actions > Create Method
![Alt Text](../../images/Pasted%20image%2020230605131725.png)
connect API Gateway to Lambda function
![Alt Text](../../images/Pasted%20image%2020230605135636.png)
#### Enable CORS
Click Actions > Enable CORS\
once set up, enable CORS so that cloud front origin is allowed to interact with API Gateway
![Alt Text](../../images/Pasted%20image%2020230605135855.png)
#### Test API
![Alt Text](../../images/Pasted%20image%2020230605132231.png)
##### Request Body
```
{
    
}
```
##### Response Body
```
1
```

##### Response Headers
```json
{
    "Access-Control-Allow-Origin": [
        "*"
    ],
    "Content-Type": [
        "application/json"
    ],
    "X-Amzn-Trace-Id": [
        "Root=1-647e2790-8f1e856b24006082d3123ec6;Sampled=0;lineage=60feb345:0"
    ]
}
```
### Website modification
#### index.html
```html
<div class="counter-number">Couldn't read the counter</div>
```
Insert html text for your homepage
#### index.js
```js
const counter = document.querySelector(".counter-number");
const API_GATEWAY = "https://dhytdk89e1.execute-api.us-east-1.amazonaws.com/dev"
async function updateCounter() {
    var myHeaders = new Headers();
    myHeaders.append("Content-Type", "application/json");
    var raw = JSON.stringify({});
    var requestOptions = {
        method: 'POST',
        Headers: myHeaders,
        body: raw,
        redirect: 'follow'
    }
    const response = await fetch(API_GATEWAY,requestOptions);
    let data = await response.json();
    counter.innerHTML = `👀 Views: ${data}`;
}
updateCounter();
```
> replace **API_GATEWAY** with your URL. 
Alternatively, instead of using two files, you can embed within **index.html** by inserting the <script> tag
# Set up CI/CD
#### Setting up GitHub actions
Create the file `.github/workflows` inside your project and create `main.yaml` with the following config
**Main.yaml**
```yaml
name: Upload Website onto S3

on:
  push:
    branches:
    - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@master
    - uses: jakejarvis/s3-sync-action@master
      with:
        args: --acl public-read --follow-symlinks --delete --exclude '.git/*'
      env:
        AWS_S3_BUCKET: ${{ secrets.AWS_S3_BUCKET }}
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        AWS_REGION: 'us-west-1'
        SOURCE_DIR: 'public'
```
#### Setting up Secrets
On github page, move to Settings\
On the left navigation pane, Security > Secrets and variables > Actions\
Create new repository secrets
![Alt Text](../../images/Pasted%20image%2020230531174217.png)
>AWS_S3_BUCKET is the name of the s3 bucket, not the ARN.
#### Verify changes
Create a new PR to the repo and go to actions. The workflow will start deploying the PR onto the S3.
![Alt Text](../../images/Pasted%20image%2020230531174459.png)
### Errors
#### ACL blocking github/floworks
upload failed: public/page/index.html to s3://***/page/index.html An error occurred (AccessDenied) when calling the PutObject operation: Access Denied

1) Configure S3 bucket to allow public access to ACL
![Alt Text](../../images/Pasted%20image%2020230529193837.png)
2) Change the ownership of the bucket to enable ACLs
![Alt Text](../../images/Pasted%20image%2020230529193857.png)
3) Verify that the bucket owner has Write access to the bucket. This assumes the access keys belongs to the bucket owner.
![Alt Text](../../images/Pasted%20image%2020230529193914.png)

#### ERROR: {"message":"Missing Authentication Token"}
The API gateway link cannot directly be called, instead pass in a Request Body
