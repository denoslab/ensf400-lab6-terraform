# ensf400-lab6-terraform

## Objectives
This lab will give us an overview of Infrastructure-as-Code (IaC) with examples using [Terraform](https://developer.hashicorp.com/terraform/docs).
Terraform works primarily with public cloud services which would incur charges for the creation of
cloud-based resources. To avoid the costs of practicing Terraform tasks and to be able to 
work on the lab tasks freely, we will use [LocalStack](https://docs.localstack.cloud/overview/) to
create a simulated environment of AWS services.

## Environment

### Set Up Your GitHub CodeSpaces Instance
Same as Lab 5, this labs will be performed in [GitHub CodeSpaces](https://github.com/codespaces). Create an instance using GitHub Codespaces. Choose repository `denoslab/ensf400-lab6-terraform`.

### Install LocalStack

We will use PyPI to install LocalStack. Full installation guide can be found [here](https://github.com/localstack/localstack?tab=readme-ov-file#installation).

```bash
$ pip install localstack
```
After installation, start LocalStack inside a Docker container by running:
```bash
$ localstack start -d

     __                     _______ __             __
    / /   ____  _________ _/ / ___// /_____ ______/ /__
   / /   / __ \/ ___/ __ `/ /\__ \/ __/ __ `/ ___/ //_/
  / /___/ /_/ / /__/ /_/ / /___/ / /_/ /_/ / /__/ ,<
 /_____/\____/\___/\__,_/_//____/\__/\__,_/\___/_/|_|

 💻 LocalStack CLI 3.2.0
 👤 Profile: default

[12:47:13] starting LocalStack in Docker mode 🐳
           preparing environment
           configuring container
           starting container
[12:47:15] detaching
```
You can query the status of respective services on LocalStack by running:
```bash
$ localstack status services
┏━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━┓
┃ Service                  ┃ Status      ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━┩
│ acm                      │ ✔ available │
│ apigateway               │ ✔ available │
│ cloudformation           │ ✔ available │
│ cloudwatch               │ ✔ available │
│ config                   │ ✔ available │
│ dynamodb                 │ ✔ available │
...
```

### Install AWS CLI and LocalStack AWS CLI Wrapper
Now that we have LocalStack installed, the next step is to install the AWS CLI along with the LocalStack wrapper.

```bash
$ pip install awscli-local[ver1]
```
Check if `awslocal` command can correctly run. The following output is expected.
```bash
$ awslocal
Note: AWS CLI version 2, the latest major version of the AWS CLI, is now stable and recommended for general use. For more information, see the AWS CLI version 2 installation instructions at: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html

usage: aws [options] <command> <subcommand> [<subcommand> ...] [parameters]
To see help text, you can run:

  aws help
  aws <command> help
  aws <command> <subcommand> help
aws: error: the following arguments are required: command
```

### Install Terraform
The previous steps have created a simulated local AWS environment that can provide the APIs for provisioning AWS cloud services. Now we will install Terraform. 

Terraform is an Infrastructure-as-Code (IaC) framework developed by HashiCorp. It enables users to define and provision infrastructure using a high-level configuration language. Terraform uses HashiCorp Configuration Language (HCL) as its configuration syntax. HCL is a domain-specific language designed for writing configurations that define infrastructure elements and their relationships.

Run the following command on your CodeSpaces instance.

```bash
$ wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
```
```bash
$ echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```
```bash
$ sudo apt update && sudo apt install terraform
```
Type `terraform` to see if Terraform can run correctly. You should see output like this.

```bash
$ terraform
Usage: terraform [global options] <subcommand> [args]
...
```



### Install `tflocal` - Terraform LocalStack Wrapper

LocalStack supports Terraform via the AWS provider through custom service endpoints. We will use the `tflocal` wrapper script to automatically configure the service endpoints.

`tflocal` is a small wrapper script to run Terraform against LocalStack. `tflocal` script uses the Terraform Override mechanism and creates a temporary file localstack_providers_override.tf to configure the endpoints for the AWS provider section. The endpoints for all services are configured to point to the LocalStack API (http://localhost:4566 by default). It allows us to easily deploy your unmodified Terraform scripts against LocalStack.

To install the `tflocal` command, we will use PyPI:
```bash
$ pip install terraform-local
```

After installation, we can use the `tflocal` command, which has the same interface as the terraform command line.
```bash
$ tflocal --help
Usage: terraform [global options] <subcommand> [args]
...
```
## Example 1 - Managing An S3 Bucket

### List Existing S3 Buckets
Before we create any configuration using Terraform, first let us list the existing S3 buckets.
```bash
$ awslocal s3 ls
```
The output should be empty, meaning that there is no bucket.

### Create a Terraform configuration

Create a new file named `main.tf` and add a minimal AWS S3 bucket configuration to it. To create the file, use
```bash
$ touch main.tf
```

Then, the following contents should be added in the main.tf file:
```tf
resource "aws_s3_bucket" "test-bucket" {
  bucket = "my-bucket"
}
```
Initialize Terraform using the following command:
```bash
$ tflocal init
```

Dry-run the configuration to see what will be changed:
```bash
$ tflocal plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:
...

Plan: 1 to add, 0 to change, 0 to destroy.
╷
│ Warning: Invalid Attribute Combination
│ 
│   with provider["registry.terraform.io/hashicorp/aws"],
│   on localstack_providers_override.tf line 2, in provider "aws":
│    2: provider "aws" {
│ 
│ Only one of the following attributes should be set: "endpoints[0].configservice", "endpoints[0].config"
│ 
│ This will be an error in a future release.
│ 
│ (and one more similar warning elsewhere)
```
**NOTE:** there is a warning irrelevant to our lab. Ignore the warning as it does not affect our steps.

We can now provision the S3 bucket specified in the configuration:
```bash
$ tflocal apply 

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:
...

Plan: 1 to add, 0 to change, 0 to destroy.
...

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: 
```
Enter `yes` to confirm performing the planned actions.

Now list the buckets again using the AWS CLI:

```bash
$ awslocal s3 ls
2024-03-03 04:41:29 my-bucket
```
We can see that the bucket has been created as an AWS resource.

Now, change the name of the S3 bucket from `my-bucket` to `my-bucket2` by modifying `main.tf`:
```tf
resource "aws_s3_bucket" "test-bucket" {
  bucket = "my-bucket2"
}
```
Then plan and apply the change again and observe the output.
```bash
tflocal plan
tflocal apply -auto-approve
```
Then list the buckets again using the AWS CLI:

```bash
$ awslocal s3 ls
2024-03-03 05:05:28 my-bucket2
```

## Example 2 - Creating a Static Website 

We will create a simple static website using plain HTML to get started. To create a static website deployed over S3, we need to create an index document and a custom error document. We will name our index document index.html and our error document error.html. Optionally, you can create a folder called assets to store images and other assets.

Let’s go to the directory `s3-static-website` where we’ll store our static website files.
```bash
$ cd /workspaces/ensf400-lab6-terraform/s3-static-website
```

Create an `index.html` file in the `www` sub directory (empty file already created for you):

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta http-equiv="Content-Type" content="text/html" />
    <meta  charset="utf-8"  />
    <title>Static Website</title>
  </head>
  <body>
    <p>Static Website deployed locally over S3 using LocalStack</p>
  </body>
</html>
```

S3 will serve this file when a user visits the root URL of your static website, serving as the default page. In a similar fashion, we can configure a custom error document that contains a user-friendly error message. Let’s create a file named `error.html` (empty file already created for you) and add the following code:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>404</title>
  </head>
  <body>
    <p>Something is amiss.</p>
  </body>
</html>
```

With the provider configured, we can now configure the variables for our S3 bucket. Go back to the `s3-static-website` directory. Create a new file named `variables.tf` (empty file already created for you) and add the following code:

```tf
# Input variable definitions

variable "bucket_name" {
  description = "Name of the s3 bucket. Must be unique."
  type        = string
}

variable "tags" {
  description = "Tags to set on the bucket."
  type        = map(string)
  default     = {}
}

```

We take user input for the bucket name and tags. Next, we will define the output variables for our Terraform configuration. Create a new file named `outputs.tf` (empty file already created for you) and add the following code:

```tf
# Output variable definitions

output "arn" {
  description = "ARN of the bucket"
  value       = aws_s3_bucket.s3_bucket.arn
}

output "name" {
  description = "Name (id) of the bucket"
  value       = aws_s3_bucket.s3_bucket.id
}

output "domain" {
  description = "Domain name of the bucket"
  value       = aws_s3_bucket_website_configuration.s3_bucket.website_domain
}

output "website_endpoint" {
  value = aws_s3_bucket_website_configuration.s3_bucket.website_endpoint
}

```

The output variables are the ARN, name, domain name, and website endpoint of the bucket. With all the configuration files in place, we can now create the S3 bucket. Create a new file named `main.tf` (empty file already created for you) and create the S3 bucket using the following code:

```tf
resource "aws_s3_bucket" "s3_bucket" {
  bucket = var.bucket_name
  tags   = var.tags
}

```

To configure the static website hosting, we will use the `aws_s3_bucket_website_configuration` resource. Add the following code to the `main.tf` file:

```tf
resource "aws_s3_bucket_website_configuration" "s3_bucket" {
  bucket = aws_s3_bucket.s3_bucket.id

  index_document {
    suffix = "index.html"
  }

  error_document {
    key = "error.html"
  }

}
```

To set the bucket policy, we will use the `aws_s3_bucket_policy` resource. Add the following code to the `main.tf` file:

```tf
resource "aws_s3_bucket_acl" "s3_bucket" {
  bucket = aws_s3_bucket.s3_bucket.id
  acl    = "public-read"
}

resource "aws_s3_bucket_policy" "s3_bucket" {
  bucket = aws_s3_bucket.s3_bucket.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid       = "PublicReadGetObject"
        Effect    = "Allow"
        Principal = "*"
        Action    = "s3:GetObject"
        Resource = [
          aws_s3_bucket.s3_bucket.arn,
          "${aws_s3_bucket.s3_bucket.arn}/*",
        ]
      },
    ]
  })
}
```

In the above code, we are setting the ACL of the bucket to `public-read` and setting the bucket policy to allow public access to the bucket. Let’s use the `aws_s3_object` resource to upload the files to the bucket. Add the following code to the `main.tf` file:

```tf
resource "aws_s3_object" "object_www" {
  depends_on   = [aws_s3_bucket.s3_bucket]
  for_each     = fileset("${path.root}", "www/*.html")
  bucket       = var.bucket_name
  key          = basename(each.value)
  source       = each.value
  etag         = filemd5("${each.value}")
  content_type = "text/html"
  acl          = "public-read"
}
```

The above code uploads all our html files to the bucket. We are also setting the ACL of the files to `public-read`.

With all the configuration files in place, we can now initialize the Terraform configuration. Run the following command to initialize the Terraform configuration:

```bash
$ tflocal init

...
Terraform has been successfully initialized!
...
```

We can create an execution plan based on our Terraform configuration for the AWS resources. Run the following command to create an execution plan:

```bash
$ tflocal plan
```

Finally, we can apply the Terraform configuration to create the AWS resources. Run the following command to apply the Terraform configuration:

```bash
$ tflocal apply -auto-approve

var.bucket_name
  Name of the s3 bucket. Must be unique.

  Enter a value: testbucket
...
arn = "arn:aws:s3:::testbucket"
domain = "s3-website-us-east-1.amazonaws.com"
name = "testbucket"
website_endpoint = "testbucket.s3-website-us-east-1.amazonaws.com"
```

In the above command, we specified `testbucket` as the bucket name. You can specify any bucket name since LocalStack is ephemeral, and stopping your LocalStack container will delete all the created resources. The above command output includes the ARN, name, domain name, and website endpoint of the bucket. You can see the website_endpoint configured to use AWS S3 Website Endpoint. You can now access the website using the bucket name `testbucket` in the following format: 
```bash
$ curl http://testbucket.s3-website.localhost.localstack.cloud:4566
```

Since the endpoint is configured to use `localhost.localstack.cloud`, no real AWS resources have been created.


## Example 3 - API Gateway DynamoDB

This example will create an AWS API Gateway with a DynamoDB backend. Here is the [source](https://github.com/localstack-samples/localstack-terraform-samples/tree/master/apigateway-dynamodb) of this example.

Go to the `apigateway-dynamodb` directory, then apply Terraform configurations.
```bash
$ cd /workspaces/ensf400-lab6-terraform/apigateway-dynamodb
$ tflocal init
$ tflocal plan
$ tflocal apply -auto-approve
```
As we can see, the output will include the API key of the created service and its REST API endpoint:

```bash
Apply complete! Resources: 16 added, 0 changed, 0 destroyed.

Outputs:

apigw_endpoint = "https://leycl5nd00.execute-api.us-east-1.amazonaws.com/v1/pets"
apigw_key = "1UA2BJMkLoV7cyTnG3Xxzjr9QYt8gWdKNibm6lhs"
```

Now, we will store the API key of the service and only the first part of the REST API endpoint for accessing the LocalStack service:
```bash
$ APIKEY=$(tflocal output -json | jq -r .apigw_key.value)
$ echo $APIKEY
```
You should see the output like this (API key will be different):
```bash
1UA2BJMkLoV7cyTnG3Xxzjr9QYt8gWdKNibm6lhs
```

```bash
$ RESTAPI=$(awslocal apigateway get-rest-apis | jq -r .items[0].id)
$ echo $RESTAPI
```
You should see the output like this:
```bash
leycl5nd00
```
The values above will be different since each time the API key and the endpoint will be generated randomly.

Next, use these two variables to create a data record:
```bash
$ curl ${RESTAPI}.execute-api.localhost.localstack.cloud:4566/v1/pets -H "x-api-key: ${APIKEY}" -H 'Content-Type: application/json' --request POST --data-raw '{ "PetType": "dog", "PetName": "tito", "PetPrice": 250 }'
```
You should see the output below indicating the record is created successfully:
```bash
{}
```

Finally, verify the creation of the record by querying it:
```bash
$ curl -H "x-api-key: ${APIKEY}" --request GET ${RESTAPI}.execute-api.localhost.localstack.cloud:4566/v1/pets/dog
```
You should see the query result below:
```bash
{"pets": [{"id": "fd67ab41", "PetType": "dog", "PetName": "tito", "PetPrice": "250"}]}
```

## Have Your Work Checked By a TA
The TA will check the completion of the following tasks:
- Output of Example 1.
- Output of Example 2.
- Output of Example 3.

## Cleanups

Finally, clean up resources under each working directory created by Terraform:
```bash
$ cd /workspaces/ensf400-lab6-terraform
$ tflocal destroy -auto-approve
$ cd /workspaces/ensf400-lab6-terraform/s3-static-website
$ tflocal destroy -auto-approve
$ cd /workspaces/ensf400-lab6-terraform/apigateway-dynamodb
$ tflocal destroy -auto-approve
```
