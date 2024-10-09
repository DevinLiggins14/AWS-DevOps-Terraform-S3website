# AWS-DevOps-Terraform-S3website
<h2>Description</h2>
<br/> In this project we will create a static website in AWS using Terraform
<br />
<br/> <br/>
<img src="https://github.com/user-attachments/assets/6cc07b09-9d87-4aba-9c37-b93f70bbd430"/>


<h2>Languages and Utilities Used</h2>

Bash, Amazon S3, VSCode, Terraform (Optional: AWS Cli)

<h2>Environments Used </h2>

- <b>CentOS Linux release 8.5.2111
 10, AWS portal </b>

<h2>Project walk-through:</h2>
<br/>
<p align="center">

 ##  Step 1: Create an S3 Bucket

### **Prerequisites**  
- Have an [AWS account](https://aws.amazon.com/console/).   
- Install [terraform](https://developer.hashicorp.com/terraform/install).

```Bash
# Create a dir for this project 
mkdir static-website-demo
cd static-website-demo
# Establish the AWS as the provider
cat <<EOF > provider.tf
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "5.70.0"
    }
  }
}

provider "aws" {
  # Configuration options
  region = "us-east-1"
}
EOF



```
<img src="https://github.com/user-attachments/assets/4d3c9e6f-31ee-4e03-9a5e-b63bad951c60"/>
<br/> Note: this version of the AWS provider is the latest at the time of this project. Other versions can be found [here](https://registry.terraform.io/providers/hashicorp/aws/latest/docs) <br/>
<br/> Also configure the region of the previous comand under configuration options to desired location <br/>

```Bash
# Initalize the AWS provider and download all the required dependencies and files
terraform init
``` 
<img src="https://github.com/user-attachments/assets/b26f1c4f-7408-4dd5-a9ee-3f23a82ba170"/>
<br/> Now for the S3 bucket lets create it using the AWS-Cli<br/>

```Bash
# Connect the AWS Cli with your IAM User
aws configure
# Confirm credentials
aws sts get-caller-identity
```

<br/> Make sure to fill in the credentials with the IAM user Access Key and Secret Access Keys. These can be found in the AWS IAM service for a user <br/>
<img src="https://github.com/user-attachments/assets/03525b0f-63db-4fd9-a8ad-4fdbcedfc453"/>
<img src="https://github.com/user-attachments/assets/3d1d680f-aa50-4ac9-9f41-2e102082fe11"/>

<br/> Note: to create the name must be unique across AWS  <br/>
<br/> More info on using Terraform to create an S3 bucket can be found at https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket <br/>

### However first define a variable, create the main.tf, and create our S3 bucket

```Bash
# Define the variable 
cat <<EOF > variables.tf
variable "bucketname" {
  default = "myterraformstaticweb"
}
EOF

# Create the main.tf with the variable
cat <<EOF > main.tf
# Create S3 bucket
resource "aws_s3_bucket" "mybucket" {
  bucket = var.bucketname
}
EOF

# Create the S3 bucket
terraform plan
terraform apply

```

<img src="https://github.com/user-attachments/assets/319c58f0-ccbd-43f4-a005-01324d213587"/>
<img src="https://github.com/user-attachments/assets/f4ca7509-f755-4f29-bf63-068e39c82d80"/>
<img src="https://github.com/user-attachments/assets/0aa175f5-0098-42e9-a9f8-df1e7e608ab5"/>
<br/> Confirm the bucket has been created in the console as well: <br/>
<img src ="https://github.com/user-attachments/assets/d6b44cd9-60f7-45d3-9195-c94faa028787"/>


## Step 2: Configure the Bucket for Static Website Hosting

<br/> Navigate to [s3_bucket_ownership_controls](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_ownership_controls) to find the following: <br/>


## Step 3: Upload Website Files

## Step 4: Enable Public Access

## Step 5: Configure DNS (Optional)

## Step 6: Test the Website


