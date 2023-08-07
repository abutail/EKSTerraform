# EKSTerraform
Multiple EKS Clusters in a single VPC
### Step 1: Install Terraform

#### On macOS (using Homebrew):

```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

#### On Windows (using Chocolatey):

```bash
choco install terraform
```

#### On Linux (download binary):

```bash
wget https://releases.hashicorp.com/terraform/1.0.6/terraform_1.0.6_linux_amd64.zip
unzip terraform_1.0.6_linux_amd64.zip
sudo mv terraform /usr/local/bin/
```

You can verify the installation with:

```bash
terraform -v
```

### Step 2: Terraform Code to Create Two EKS Clusters with Version 1.27 in the Same VPC

```hcl
provider "aws" {
  region = "us-west-2"
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "2.77.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-west-2a", "us-west-2b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.3.0/24", "10.0.4.0/24"]
}

module "eks_cluster_1" {
  source          = "terraform-aws-modules/eks/aws"
  version         = "17.23.0"
  cluster_name    = "my-cluster-1"
  cluster_version = "1.27"
  subnets         = module.vpc.private_subnets
  node_groups = {
    eks_nodes = {
      desired_capacity = 2
      max_capacity     = 3
      min_capacity     = 1
      key_name         = var.key_name
      instance_type    = "m5.large"
      key_pair_name    = var.key_pair_name
    }
  }
}

module "eks_cluster_2" {
  source          = "terraform-aws-modules/eks/aws"
  version         = "17.23.0"
  cluster_name    = "my-cluster-2"
  cluster_version = "1.27"
  subnets         = module.vpc.private_subnets
  node_groups = {
    eks_nodes = {
      desired_capacity = 2
      max_capacity     = 3
      min_capacity     = 1
      key_name         = var.key_name
      instance_type    = "m5.large"
      key_pair_name    = var.key_pair_name
    }
  }
}
```

### Step 3: Apply the Terraform Configuration

Run the following commands in the directory where your Terraform configuration is located:

```bash
terraform init
terraform apply
```

Enter "yes" when prompted to create the resources.

Make sure to adjust the `key_name` and `key_pair_name` variables to match your environment, and validate the EKS version as per your AWS region, as not all regions might support the specified version at the time of writing.

That's it! The code will create two EKS clusters with version 1.27 within the same VPC.
