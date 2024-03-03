# ensf400-lab6-terraform

## Objectives
This lab will give us an overview of Infrastructure-as-Code (IaC) with examples using [Terraform](https://developer.hashicorp.com/terraform/docs).
Terraform works primarily with public cloud services which would incur charges for the creation of
cloud-based resources. To avoid the costs of practicing Terraform tasks and to be able to 
work on the lab tasks freely, we will use [LocalStack](https://docs.localstack.cloud/overview/) to
create a simulated environment of AWS services.

## Environment

### Set Up Your GitHub CodeSpaces Instance
Same as Lab 5, this labs will be performed in [GitHub CodeSpaces](https://github.com/codespaces). Create an instance using GitHub Codespaces. Choose repository \texttt{denoslab/ensf400-lab6-terraform}.

### Install LocalStack

We will use PyPI to install LocalStack. Full installation guide can be found [here](https://github.com/localstack/localstack?tab=readme-ov-file#installation).

```bash
pip install localstack
```
After installation, start LocalStack inside a Docker container by running:
```bash
 % localstack start -d

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
% localstack status services
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
pip install awscli-local[ver1]
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
$ echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
$ sudo apt update && sudo apt install terraform
```
Type `terraform` to see if Terraform can run correctly. You should see output like this.

```bash
$ terraform
Usage: terraform [global options] <subcommand> [args]

The available commands for execution are listed below.
The primary workflow commands are given first, followed by
less common or more advanced commands.

Main commands:
  init          Prepare your working directory for other commands
  validate      Check whether the configuration is valid
  plan          Show changes required by the current configuration
  apply         Create or update infrastructure
  destroy       Destroy previously-created infrastructure
...
Global options (use these before the subcommand, if any):
  -chdir=DIR    Switch to a different working directory before executing the
                given subcommand.
  -help         Show this help output, or the help for a specified subcommand.
  -version      An alias for the "version" subcommand.
```



### Install `tflocal` - Terraform LocalStack Wrapper

LocalStack supports Terraform via the AWS provider through custom service endpoints. We will use the `tflocal` wrapper script to automatically configure the service endpoints.

`tflocal` is a small wrapper script to run Terraform against LocalStack. `tflocal` script uses the Terraform Override mechanism and creates a temporary file localstack_providers_override.tf to configure the endpoints for the AWS provider section. The endpoints for all services are configured to point to the LocalStack API (http://localhost:4566 by default). It allows us to easily deploy your unmodified Terraform scripts against LocalStack.

To install the `tflocal` command, we will use PyPI:
```bash
pip install terraform-local
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
tflocal init
```

Dry-run the configuration to see what will be changed:
```bash
$ tflocal plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_s3_bucket.test-bucket will be created
  + resource "aws_s3_bucket" "test-bucket" {
      + acceleration_status         = (known after apply)
      + acl                         = (known after apply)
      + arn                         = (known after apply)
      + bucket                      = "my-bucket"
      + bucket_domain_name          = (known after apply)
      + bucket_prefix               = (known after apply)
      + bucket_regional_domain_name = (known after apply)
      + force_destroy               = false
      + hosted_zone_id              = (known after apply)
      + id                          = (known after apply)
      + object_lock_enabled         = (known after apply)
      + policy                      = (known after apply)
      + region                      = (known after apply)
      + request_payer               = (known after apply)
      + tags_all                    = (known after apply)
      + website_domain              = (known after apply)
      + website_endpoint            = (known after apply)
    }

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

  # aws_s3_bucket.test-bucket will be created
  + resource "aws_s3_bucket" "test-bucket" {
      + acceleration_status         = (known after apply)
      + acl                         = (known after apply)
      + arn                         = (known after apply)
      + bucket                      = "my-bucket"
      + bucket_domain_name          = (known after apply)
      + bucket_prefix               = (known after apply)
      + bucket_regional_domain_name = (known after apply)
      + force_destroy               = false
      + hosted_zone_id              = (known after apply)
      + id                          = (known after apply)
      + object_lock_enabled         = (known after apply)
      + policy                      = (known after apply)
      + region                      = (known after apply)
      + request_payer               = (known after apply)
      + tags_all                    = (known after apply)
      + website_domain              = (known after apply)
      + website_endpoint            = (known after apply)
    }

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
╵

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

## Example 2 - API Gateway DynamoDB

This example will create an AWS API Gateway with a DynamoDB backend. Here is the [source](https://github.com/localstack-samples/localstack-terraform-samples/tree/master/apigateway-dynamodb) of this example.

Go to the `apigateway-dynamodb` directory, then apply Terraform configurations.
```bash
cd apigateway-dynamodb
tflocal init
tflocal plan
tflocal apply --auto-approve
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
APIKEY=$(tflocal output -json | jq -r .apigw_key.value)
echo $APIKEY
1UA2BJMkLoV7cyTnG3Xxzjr9QYt8gWdKNibm6lhs
```

```bash
RESTAPI=$(awslocal apigateway get-rest-apis | jq -r .items[0].id)
echo $RESTAPI
leycl5nd00
```

Next, use these two variables to create a data record:
```bash
curl ${RESTAPI}.execute-api.localhost.localstack.cloud:4566/v1/pets -H "x-api-key: ${APIKEY}" -H 'Content-Type: application/json' --request POST --data-raw '{ "PetType": "dog", "PetName": "tito", "PetPrice": 250 }'
{}
```

Finally, verify the creation of the record by querying it:
```bash
curl -H "x-api-key: ${APIKEY}" --request GET ${RESTAPI}.execute-api.localhost.localstack.cloud:4566/v1/pets/dog
{"pets": [{"id": "fd67ab41", "PetType": "dog", "PetName": "tito", "PetPrice": "250"}]}
```



