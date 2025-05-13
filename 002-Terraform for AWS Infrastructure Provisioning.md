# 📋 **1. PLAN: Terraform for AWS Infrastructure Provisioning**

We'll use **Terraform** to automate the creation of the AWS EC2 instance, EBS volume, security groups, and any other resources required for the DevSecOps pipeline.

#### Key Terraform Tasks:

1. Provision an **AWS EC2 instance** (Ubuntu 24.04 LTS).
2. Attach a **50 GB EBS volume**.
3. Configure **Security Groups** to allow access to necessary ports (e.g., SSH, HTTP, Docker/Kubernetes).
4. Set up **IAM roles** for secure access to AWS resources.

---

### 🔑 **Terraform Configuration:**

#### 1. **Provider Setup**

The first step is configuring the **AWS provider** in Terraform:

```hcl
provider "aws" {
  region = "us-east-1"  # Change to your preferred AWS region
  access_key = "your_access_key"  # Replace with actual AWS access key (use environment variables for security)
  secret_key = "your_secret_key"  # Replace with actual AWS secret key (use environment variables for security)
}
```

#### 2. **AWS EC2 Instance Configuration**

Define the AWS EC2 instance with the required specs (Ubuntu 24.04 LTS, t2.medium):

```hcl
resource "aws_instance" "devsecops_instance" {
  ami           = "ami-xxxxxxxxxxxxxxxxx"  # Replace with the Ubuntu 24.04 LTS AMI ID for your region
  instance_type = "t2.medium"
  key_name      = "your_ssh_key"  # Replace with your SSH key pair name
  tags = {
    Name = "DevSecOps-EC2"
  }

  # Security group setup (will define in next section)
  security_groups = [aws_security_group.devsecops_sg.name]

  # Attach an EBS volume
  ebs_block_device {
    device_name = "/dev/sda1"
    volume_size = 50  # 50 GB EBS volume
    volume_type = "gp2"
  }
}
```

#### 3. **Security Group Configuration**

Create a **Security Group** to allow necessary access:

```hcl
resource "aws_security_group" "devsecops_sg" {
  name        = "devsecops_sg"
  description = "Allow SSH, HTTP, Docker, Kubernetes ports"

  ingress {
    from_port   = 22    # SSH
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80    # HTTP (for Jenkins, etc.)
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443   # HTTPS
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 6443  # Kubernetes API server
    to_port     = 6443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

#### 4. **IAM Role for EC2 Instance**

(Optional) Set up an IAM role if you want the EC2 instance to interact with AWS services securely, such as accessing S3 or CloudWatch logs.

```hcl
resource "aws_iam_role" "devsecops_role" {
  name               = "devsecops-role"
  assume_role_policy = data.aws_iam_policy_document.assume_role_policy.json
}

data "aws_iam_policy_document" "assume_role_policy" {
  statement {
    actions   = ["sts:AssumeRole"]
    principals = [
      {
        type        = "Service"
        identifiers = ["ec2.amazonaws.com"]
      }
    ]
  }
}
```

#### 5. **Terraform Output**

You can output the public IP of the EC2 instance once it’s provisioned:

```hcl
output "instance_ip" {
  value = aws_instance.devsecops_instance.public_ip
}
```

---

### 🔄 **Terraform Commands**

Now, to apply this configuration, use the following Terraform commands:

1. **Initialize Terraform** (downloads required providers):

   ```bash
   terraform init
   ```

2. **Preview the changes** Terraform will apply:

   ```bash
   terraform plan
   ```

3. **Apply the configuration** to provision the resources:

   ```bash
   terraform apply
   ```

   Terraform will prompt you to confirm. Type `yes` to proceed.

---

### 💡 Notes:

* Make sure your **AWS credentials** are properly set up. You can use environment variables, `~/.aws/credentials`, or configure through AWS CLI.
* The **AMI ID** for Ubuntu 24.04 LTS varies by region. You can find the correct AMI ID by searching the [AWS Marketplace](https://aws.amazon.com/marketplace) or using the AWS EC2 console.

---
