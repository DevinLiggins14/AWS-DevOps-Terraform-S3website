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

<br/> Navigate to the [S3 bucket website ](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_website_configuration) configuration  <br/>
<br/> However first we need to create an index.html and error.html for our website <br/>
```Bash
# Here is an index.html for my portfolio we can use for this demo
cat <<EOF > index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Devin's Portfolio</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f4f4f4;
        }
        header {
            background-color: #333;
            color: #fff;
            padding: 10px 0;
            text-align: center;
        }
        nav {
            margin: 10px;
            text-align: center;
        }
        nav a {
            margin: 0 15px;
            text-decoration: none;
            color: #333;
            font-weight: bold;
        }
        section {
            padding: 20px;
            max-width: 1000px;
            margin: auto;
            background-color: #fff;
        }
        footer {
            background-color: #333;
            color: #fff;
            text-align: center;
            padding: 10px 0;
            position: absolute;
            bottom: 0;
            width: 100%;
        }
    </style>
</head>
<body>

<header>
    <h1>Welcome to Devin's Portfolio</h1>
</header>

<nav>
    <a href="#about">About</a>
    <a href="#projects">Projects</a>
    <a href="#contact">Contact</a>
</nav>

<section id="about">
    <h2>About Me</h2>
    <p>Hello! I'm Devin, a skilled developer with experience in cloud technologies, Linux, and networking. This portfolio showcases my work and the projects I've completed.</p>
</section>

<section id="projects">
    <h2>Projects</h2>
    <ul>
        <li><strong>Project 1:</strong> Azure Kubernetes Service Setup</li>
        <li><strong>Project 2:</strong> Linux Networking Automation</li>
        <li><strong>Project 3:</strong> Cloud Infrastructure Monitoring with Prometheus and Grafana</li>
    </ul>
</section>

<section id="contact">
    <h2>Contact Me</h2>
    <p>Email: devin@example.com</p>
</section>

<footer>
    <p>&copy; 2024 Devin. All rights reserved.</p>
</footer>

</body>
</html>
EOF

# Here is the error.html for this project
cat <<EOF > error.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Page Not Found</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f4f4f4;
            text-align: center;
            color: #333;
        }
        header {
            background-color: #ff6666;
            color: #fff;
            padding: 20px 0;
        }
        section {
            padding: 50px;
        }
        a {
            text-decoration: none;
            color: #ff6666;
            font-weight: bold;
        }
    </style>
</head>
<body>

<header>
    <h1>Error 404 - Page Not Found</h1>
</header>

<section>
    <h2>Oops! The page you're looking for doesn't exist.</h2>
    <p>It seems that you've encountered an error or the page you're trying to reach is unavailable.</p>
    <p><a href="index.html">Go back to the homepage</a></p>
</section>

</body>
</html>
EOF


```

## Step 3: Upload Website Files
<br/> Now that the html files have been created in order to add them to the S3 bucket we must create the resource [aws S3 object](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_object) <br/>

``` Bash
# Here is the example format of the S3 bucket object block
resource "aws_s3_bucket_object" "object" {
  bucket = "your_bucket_name"
  key    = "new_object_key"
  source = "path/to/file"

  # The filemd5() function is available in Terraform 0.11.12 and later
  # For Terraform 0.11.11 and earlier, use the md5() function and the file() function:
  # etag = "${md5(file("path/to/file"))}"
  etag = filemd5("path/to/file")
}
```

<br/> Note: copy the same thing to configure the error.html the format is the same <br/>

## The new configuration should look like this:

```Bash
cat <<EOF >> main.tf

#create s3 bucket
resource "aws_s3_bucket" "mybucket" {
  bucket = var.bucketname
}

# Configure ownership controls for the bucket
resource "aws_s3_bucket_ownership_controls" "mybucket" {
  bucket = aws_s3_bucket.mybucket.id

  rule {
    object_ownership = "BucketOwnerPreferred"
  }
}

# Make the bucket public by disabling access restrictions
resource "aws_s3_bucket_public_access_block" "mybucket" {
  bucket = aws_s3_bucket.mybucket.id

  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

# Set ACL to public-read for the bucket
resource "aws_s3_bucket_acl" "mybucket" {
  depends_on = [
    aws_s3_bucket_ownership_controls.mybucket,
    aws_s3_bucket_public_access_block.mybucket,
  ]

  bucket = aws_s3_bucket.mybucket.id
  acl    = "public-read"
}

# Upload index.html to the bucket
resource "aws_s3_object" "index" {
  bucket = aws_s3_bucket.mybucket.id
  key = "index.html"
  source = "index.html"
  acl = "public-read"
  content_type = "text/html"
}

# Upload error.html to the bucket
resource "aws_s3_object" "error" {
  bucket = aws_s3_bucket.mybucket.id
  key = "error.html"
  source = "error.html"
  acl = "public-read"
  content_type = "text/html"
}
EOF

# Update and apply the configurations
terraform apply -auto-approve

```

<br/> Confirm the objects have been added to the S3 bucket after running:<br/>
<img src="https://github.com/user-attachments/assets/cc0bfe8a-b715-402a-a4f5-f4271ab63551"/>
<img src="https://github.com/user-attachments/assets/8817c5ac-7976-404e-9605-3fdefd2f1b5a"/>


## Step 4: Enable Public Access


<br/> Next we need to add [S3 bucket website](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_website_configuration) configuration <br/>
<br/> This is for our index.html and error.html files <br/>

```Bash
# Example usage
resource "aws_s3_bucket_website_configuration" "example" {
  bucket = aws_s3_bucket.example.id

  index_document {
    suffix = "index.html"
  }

  error_document {
    key = "error.html"
  }

  routing_rule {
    condition {
      key_prefix_equals = "docs/"
    }
    redirect {
      replace_key_prefix_with = "documents/"
    }
  }
}
```

<br/> Note: the reson why there is no VPC or Subnet defined is because S3 does not deploy within VPC it deploys outside of VPC also these resources were not configured prior to this demo <br/>

<br/> Below is the final configuration of main.tf for this demo <br/>

```Bash
cat <<EOF >> main.tf

#create s3 bucket
resource "aws_s3_bucket" "mybucket" {
  bucket = var.bucketname
}

# Configure ownership controls for the bucket
resource "aws_s3_bucket_ownership_controls" "mybucket" {
  bucket = aws_s3_bucket.mybucket.id

  rule {
    object_ownership = "BucketOwnerPreferred"
  }
}

# Make the bucket public by disabling access restrictions
resource "aws_s3_bucket_public_access_block" "mybucket" {
  bucket = aws_s3_bucket.mybucket.id

  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

# Set ACL to public-read for the bucket
resource "aws_s3_bucket_acl" "mybucket" {
  depends_on = [
    aws_s3_bucket_ownership_controls.mybucket,
    aws_s3_bucket_public_access_block.mybucket,
  ]

  bucket = aws_s3_bucket.mybucket.id
  acl    = "public-read"
}

# Upload index.html to the bucket
resource "aws_s3_object" "index" {
  bucket = aws_s3_bucket.mybucket.id
  key = "index.html"
  source = "index.html"
  acl = "public-read"
  content_type = "text/html"
}

# Upload error.html to the bucket
resource "aws_s3_object" "error" {
  bucket = aws_s3_bucket.mybucket.id
  key = "error.html"
  source = "error.html"
  acl = "public-read"
  content_type = "text/html"
}

# Configure website hosting for the S3 bucket
resource "aws_s3_bucket_website_configuration" "website" {
  bucket = aws_s3_bucket.mybucket.id

  index_document {
    suffix = "index.html"
  }

  error_document {
    key = "error.html"
  }

  depends_on = [ aws_s3_bucket_acl.mybucket ]
}

EOF

# Approve and apply the changes
terraform apply -auto-approve
```

<br/> Confirm: <br/>

<img src="https://github.com/user-attachments/assets/e2f866ec-6db2-4e7e-8029-02e60cdb708b"/>
<img src="https://github.com/user-attachments/assets/519db9df-3368-4c1e-bb16-04aa5441f967"/>
<img src="https://github.com/user-attachments/assets/41e994c0-3b37-4d05-9dfb-b6dc57a8a312"/>


### Success!


## Step 6: Test the Website


<br/> Now to test our website I am going to edit the contents of the index.html file to add an image <br/>

```Bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Devin's Portfolio</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f0f8ff; /* Light blue background */
            color: #333; /* Dark grey text */
            margin: 0;
            padding: 0;
        }
        header {
            background-color: #4682b4; /* Steel blue header */
            color: white;
            padding: 20px;
            text-align: center;
        }
        .container {
            text-align: center;
            padding: 50px;
        }
        h1 {
            font-size: 2.5em;
            color: #ff4500; /* Orange title */
        }
        p {
            font-size: 1.2em;
            color: #2e8b57; /* Sea green text */
        }
        img {
            width: 400px;
            height: auto;
            margin-top: 30px;
            border-radius: 10px;
        }
        footer {
            background-color: #4682b4;
            color: white;
            padding: 10px;
            position: fixed;
            bottom: 0;
            width: 100%;
            text-align: center;
        }
    </style>
</head>
<body>

<header>
    <h1>Welcome to Devin's Portfolio</h1>
</header>

<div class="container">
    <p>Explore my work and projects below!</p>
    <!-- Random image -->
    <img src="https://i.imgflip.com/2d57wq.jpg" alt="Placeholder Image">
</div>

<footer>
    <p>Contact: devin@example.com</p>
</footer>

</body>
</html>


```


<br/> Now we can run the following to destroy our current S3 bucket so we can create our static website from scratch <br/>

```Bash
# Destroy current S3 Bucket
terraform destroy -auto-approve
```


<img src="https://github.com/user-attachments/assets/1b388a6e-448a-4f56-9772-d65adc725843"/>
<img src="https://github.com/user-attachments/assets/87f81ada-8b63-4306-85ef-63b70622e2ef"/>


<br/> Now let's create our static website again from scratch <br/>

```Bash
# Create static website in AWS with Terraform
terraform apply -auto-approve

```

<img src="https://github.com/user-attachments/assets/fe2b91fa-f4ad-4ff0-837b-316c290dbb2f"/>
<img src="https://github.com/user-attachments/assets/a43e5e0b-0316-4184-8b1a-f7b11f4f8e53"/>
<img src="https://github.com/user-attachments/assets/f36b4ebc-2ba9-4393-a843-6379e48d962c"/>
<img src="https://github.com/user-attachments/assets/5f0021e1-6209-40fb-bfe9-7f15ff74f0e4"/>
