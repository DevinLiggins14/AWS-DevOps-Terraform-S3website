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

<br/> Navigate to [s3 bucket ownership](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_ownership_controls) as well as the [public access](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_public_access_block) blocks and add and configure the example block so it can be added into the main.tf <br/>

```Bash
# aws_s3_bucket_ownership_controls example use
resource "aws_s3_bucket" "example" {
  bucket = "example"
}

resource "aws_s3_bucket_ownership_controls" "example" {
  bucket = aws_s3_bucket.example.id

  rule {
    object_ownership = "BucketOwnerPreferred"
  }
}

# aws_s3_bucket_public_access_block example use
resource "aws_s3_bucket" "example" {
  bucket = "example"
}

resource "aws_s3_bucket_public_access_block" "example" {
  bucket = aws_s3_bucket.example.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```






<br/> However we are not done, we need to apply [ACLs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_acl) to our S3 bucket configuration as well  <br/>
<br/> For this project we will use a PUBLIC-read configuration for testing only: <br/>
```Bash
# With public-read ACL
resource "aws_s3_bucket_acl" "example" {
  depends_on = [
    aws_s3_bucket_ownership_controls.example,
    aws_s3_bucket_public_access_block.example,
  ]

  bucket = aws_s3_bucket.example.id
  acl    = "public-read"
}
```

<br/> The complete configuration now looks like this: <br/>

```Bash
cat <<EOF >> main.tf

# Create S3 bucket
resource "aws_s3_bucket" "mybucket" {
  bucket = var.bucketname
}

# Configure ownership controls for mybucket
resource "aws_s3_bucket_ownership_controls" "mybucket" {
  bucket = aws_s3_bucket.mybucket.id

  rule {
    object_ownership = "BucketOwnerPreferred"
  }
}

# Make mybucket public by adjusting public access block settings
resource "aws_s3_bucket_public_access_block" "mybucket" {
  bucket = aws_s3_bucket.mybucket.id

  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

# Apply public-read ACL to mybucket
resource "aws_s3_bucket_acl" "mybucket" {
  depends_on = [
    aws_s3_bucket_ownership_controls.mybucket,
    aws_s3_bucket_public_access_block.mybucket,
  ]

  bucket = aws_s3_bucket.mybucket.id
  acl    = "public-read"
}

EOF

# Apporve and apply the changes
terraform apply -auto-approve


```

<br/> Confirm the bucket configurations in the console after applying the changes <br/>
<img src="https://github.com/user-attachments/assets/e814168c-0d13-4024-aee1-97d0a194ec54"/>
<img src="https://github.com/user-attachments/assets/f610938e-4790-4174-9568-41b3b892cea9"/>

<br/> Next we need to enable static website hosting because it is currently disabled <br/>
<img src="https://github.com/user-attachments/assets/08027c5e-1375-4bea-8ca0-cc5df2618bff"/>

## Step 3: Upload Website Files

## Step 4: Enable Public Access

## Step 5: Configure DNS (Optional)

## Step 6: Test the Website


