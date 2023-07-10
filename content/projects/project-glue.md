---
title: "AWS Glue Masking PII"
date: 2023-07-05T20:55:31-05:00
slug: aws-glue-masking-pii
category: AWS
tags: ["AWS","IAM", "S3", "Glue", "Terraform"]
summary: This project will walkthorugh AWS Glue products, such as data catalog, databases, tables, crawlers, and glue jobs, resulting in sensitive data found within a csv file being masked with '****'
description: 
cover:
  image:
  alt:
  caption:
  relative: true
showtoc: true
draft: false
---
Download this CSV file [Customers_Data](../../folders/customer.csv)
## Create IAM role
### Set trusted entity 
![Alt Text](../../images/Pasted%20image%2020230705172515.png)
```toml
resource "aws_iam_role" "glue_role" {
  name               = "GlueAdminRole"
  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "glue.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF
}
```
### Set permissions
- Select *AWSGlueConsoleFullAccess* from the _Attach Permissions Policies_ section. 
![Alt Text](../../images/Pasted%20image%2020230705172614.png)
```toml
resource "aws_iam_role_policy_attachment" "glue_role_attachment" {
  role       = aws_iam_role.glue_role.name
  policy_arn = "arn:aws:iam::aws:policy/AWSGlueConsoleFullAccess"
}
```
## Set up S3 Structure
Structure you S3 buckets and objects in this format
```text
glue-demo/
├─ athena_results/
├─ data/
│  ├─ customer_database/
│  │  ├─ customer_csv/
│  │  │  ├─ dataloader=20230629/
│  │  │  │  ├─ customer.csv
├─ filtered_customer_data/
```

```toml
locals {
  glue_bucket_name = "glue-demo-humailkhan829"
}

# S3 Structure
resource "aws_s3_bucket" "glue_demo" {
  bucket = local.glue_bucket_name
}

resource "aws_s3_bucket_public_access_block" "public_access_glue_bucket" {
  bucket                  = aws_s3_bucket.glue_demo.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}



resource "aws_s3_object" "athena_folder" {
  bucket = aws_s3_bucket.glue_demo.id
  key    = "athena_results/"
}

resource "aws_s3_object" "filtered_customer_data" {
  bucket = aws_s3_bucket.glue_demo.id
  key    = "filtered_customer_data/"
}


resource "aws_s3_object" "customer_database_folder" {
  bucket = aws_s3_bucket.glue_demo.id
  key    = "data/customer_database/"
}

resource "aws_s3_object" "customer_csv_folder" {
  bucket = aws_s3_bucket.glue_demo.id
  key    = "${aws_s3_object.customer_database_folder.key}customers_csv/"
}

resource "aws_s3_object" "dataloader_folder" {
  bucket = aws_s3_bucket.glue_demo.id
  key    = "${aws_s3_object.customer_csv_folder.key}dataloader=20230629/"
}

resource "aws_s3_object" "customer_csv" {
  bucket = aws_s3_bucket.glue_demo.id
  key    = "${aws_s3_object.dataloader_folder.key}customer.csv"
  source = "customer.csv"
}


```
## Create database
the database is a containers for multiple data catalog tables. A data catalog table
![Alt Text](../../images/Pasted%20image%2020230705165313.png)
```toml
resource "aws_glue_catalog_database" "glue_catalog_database" {
  name         = "customers_database"
  description  = "database for customers' information"
  location_uri = "s3://${aws_s3_bucket.glue_demo.bucket}/${aws_s3_object.customer_database_folder.key}"
}
```
## Creating a crawler
a crawler is used to infer the scheme off the csv and populate the database with tables from the inferred schema.

Create a crawler with data source of **/data/customer_database/customers_csv/**, attach the Glue IAM role, and output data to **customers_database**.
![Alt Text](../../images/Pasted%20image%2020230705170000.png)
```toml
resource "aws_glue_crawler" "crawler_customer_csv" {
  name          = "crawler_customer_csv"
  role          = aws_iam_role.glue_role.arn
  database_name = aws_glue_catalog_database.glue_catalog_database.name
  description   = "Crawler for customer CSV files"

  s3_target {
    path = "s3://${aws_s3_bucket.glue_demo.bucket}/${aws_s3_object.customer_csv_folder.key}"
  }
  # execute crawler upon creation
  provisioner "local-exec" {
    command = "aws glue start-crawler --name ${self.name}"
  }
}
```
### Run the crawler
![Alt Text](../../images/Pasted%20image%2020230705170350.png)
Once the crawler is complete, it will be listed under Crawler runs and the database will be populated with a table.
![Alt Text](../../images/Pasted%20image%2020230705170449.png)
![Alt Text](../../images/Pasted%20image%2020230705170527.png)
```toml
# Since the table is created by the crawler, the table has to be imported. NOTE: define your bare-bone aws_glue_catalog_table before importing.    
# terraform import aws_glue_catalog_table.customer_csv <catalog_id>:<database_name>:<table_name>
# terraform import aws_glue_catalog_table.customer_csv 338697520941:customers_database:customers_csv
# After importing, do terraform plan. The imported object will not match your stated template. Copy the missing configurations and add them to the resource.
resource "aws_glue_catalog_table" "customer_csv" {
  name          = "customers_csv"
  database_name = aws_glue_catalog_database.glue_catalog_database.name
  table_type    = "EXTERNAL_TABLE"
  owner         = "owner"

  parameters = {
    "classification"                   = "csv"
    "typeOfData"                       = "file"
    "UPDATED_BY_CRAWLER"               = "${aws_glue_crawler.crawler_customer_csv.name}"
    "areColumnsQuoted"                 = "false"
    "averageRecordSize"                = "197"
    "columnsOrdered"                   = "true"
    "compressionType"                  = "none"
    "delimiter"                        = ","
    "objectCount"                      = "1"
    "partition_filtering.enabled"      = "true"
    "recordCount"                      = "998"
    "sizeKey"                          = "196676"
    "skip.header.line.count"           = "1"
    "CrawlerSchemaDeserializerVersion" = "1.0"
    "CrawlerSchemaSerializerVersion"   = "1.0"
  }

  partition_keys {
    name = "dataloader"
    type = "string"
  }

  storage_descriptor {
    location                  = "s3://${aws_s3_bucket.glue_demo.bucket}/${aws_s3_object.customer_csv_folder.key}"
    input_format              = "org.apache.hadoop.mapred.TextInputFormat"
    output_format             = "org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat"
    compressed                = false
    number_of_buckets         = -1



    parameters = {
      "classification"                   = "csv"
      "typeOfData"                       = "file"
      "UPDATED_BY_CRAWLER"               = "${aws_glue_crawler.crawler_customer_csv.name}"
      "areColumnsQuoted"                 = "false"
      "averageRecordSize"                = "197"
      "columnsOrdered"                   = "true"
      "compressionType"                  = "none"
      "delimiter"                        = ","
      "objectCount"                      = "1"
      "partition_filtering.enabled"      = "true"
      "recordCount"                      = "998"
      "sizeKey"                          = "196676"
      "skip.header.line.count"           = "1"
      "CrawlerSchemaDeserializerVersion" = "1.0"
      "CrawlerSchemaSerializerVersion"   = "1.0"
    }

    columns {
      name = "customerid"
      type = "bigint"
    }
    columns {
      name = "namestyle"
      type = "boolean"
    }
    columns {
      name = "title"
      type = "string"
    }
    columns {
      name = "firstname"
      type = "string"
    }
    columns {
      name = "middlename"
      type = "string"
    }
    columns {
      name = "lastname"
      type = "string"
    }
    columns {
      name = "suffix"
      type = "string"
    }
    columns {
      name = "companyname"
      type = "string"
    }
    columns {
      name = "salesperson"
      type = "string"
    }
    columns {
      name = "emailaddress"
      type = "string"
    }
    columns {
      name = "phone"
      type = "string"
    }
    columns {
      name = "passwordhash"
      type = "string"
    }
    columns {
      name = "passwordsalt"
      type = "string"
    }
    columns {
      name = "rowguid"
      type = "string"
    }
    columns {
      name = "modifieddate"
      type = "string"
    }
    
    ser_de_info {
      serialization_library = "org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe"
      
      parameters = {
        "field.delim" = ","
      }
    }
  }
}
```
## Creating Glue Job
### Visual Editor
The data catalog table will be read, filtered by **HIPAA**, and then outputted to a S3 bucket
![Alt Text](../../images/Pasted%20image%2020230705173737.png)
#### Data Source
Set *S3 source type* to *customers_database*
Set *Table* to *customers_csv*
#### Transform
Set the data source to *Node parents*
Under *Types of sensitive information to detect* > select *select categories* > *HIPAA*
Under *Actions* > select *Redact detected text* 
#### Job Details
![Alt Text](../../images/Pasted%20image%2020230705174257.png)
Attach the IAM role. The rest can remains as default.
```toml
resource "aws_glue_job" "mask_pii_glue_job" {
  name     = "mask_pii"
  role_arn = "${aws_iam_role.glue_role.arn}"
  command {
    script_location = "s3://${aws_s3_bucket.example.bucket}/mask_pii.py"
  }
}
```
The scripts can be copied from the script tab.
#### Ouput
![Alt Text](../../images/Pasted%20image%2020230705174943.png)
Sample output of 2nd row
```json
{
    "customerid": "2",
    "namestyle": "false",
    "title": "Mr.",
    "firstname": "****",
    "middlename": "",
    "lastname": "****",
    "suffix": "",
    "companyname": "Progressive Sports",
    "salesperson": "adventure-works\\david8",
    "emailaddress": "keith0@adventure-works.com",
    "phone": "170-555-0127",
    "passwordhash": "YPdtRdvqeAhj6wyxEsFdshBDNXxkCXn+CRgbvJItknw=",
    "passwordsalt": "fs1ZGhY=",
    "rowguid": "{E552F657-A9AF-4A7D-A645-C429D6E02491}",
    "modifieddate": "2006-08-01 00:00:00",
    "dataloader": "20230629"
}
```

