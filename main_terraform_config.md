```
# Configure the AWS provider
provider "aws" {
    region = "us-east-1"
    access_key = "<aws user access key"
    secret_key = "<aws user secret access key"
}

# S3 Backend Configuration
terraform {
    backend "s3" {
        bucket = "DepiLab-terraform-state-bucket"
        key    = "network/terraform.tfstate"
        region = "us-east-1"
        dynamodb_table = "terraform-lock"
    }
}

# VPC Creation
resource "aws_vpc" "tf_vpc" {
    cidr_block = "10.0.0.0/16"
    tags = {
        Name = "tf_vpc"
    }
}

# Subnets
resource "aws_subnet" "tf_vpc_pub_subnet" {
    vpc_id = aws_vpc.tf_vpc.id
    cidr_block = "10.0.1.0/24"
    map_public_ip_on_launch = true
    tags = {
        Name = "tf_vpc_pub_subnet"
    }
}

resource "aws_subnet" "tf_vpc_prv_subnet" {
    vpc_id = aws_vpc.tf_vpc.id
    cidr_block = "10.0.2.0/24"
    tags = {
        Name = "tf_vpc_prv_subnet"
    }
}

# Internet Gateway
resource "aws_internet_gateway" "tf_vpc_igw" {
    vpc_id = aws_vpc.tf_vpc.id
    tags = {
        Name = "tf_vpc_igw"
    }
}

# Route Table for Public Subnet
resource "aws_route_table" "tf_vpc_pub_route" {
    vpc_id = aws_vpc.tf_vpc.id

    route {
        cidr_block = "0.0.0.0/0"
        gateway_id = aws_internet_gateway.tf_vpc_igw.id
    }

    tags = {
        Name = "tf_vpc_pub_route"
    }
}

resource "aws_route_table_association" "tf_vpc_pub_route_assoc" {
    subnet_id      = aws_subnet.tf_vpc_pub_subnet.id
    route_table_id = aws_route_table.tf_vpc_pub_route.id
}

# NAT Gateway
resource "aws_eip" "tf_vpc_nat_eip" {
    tags = {
        Name = "tf_vpc_nat_eip"
    }
}

resource "aws_nat_gateway" "tf_vpc_nat_gw" {
    allocation_id = aws_eip.tf_vpc_nat_eip.id
    subnet_id     = aws_subnet.tf_vpc_pub_subnet.id
    tags = {
        Name = "tf_vpc_nat_gw"
    }
}

# Route Table for Private Subnet
resource "aws_route_table" "tf_vpc_prv_route" {
    vpc_id = aws_vpc.tf_vpc.id

    route {
        cidr_block     = "0.0.0.0/0"
        nat_gateway_id = aws_nat_gateway.tf_vpc_nat_gw.id
    }

    tags = {
        Name = "tf_vpc_prv_route"
    }
}

resource "aws_route_table_association" "tf_vpc_prv_route_assoc" {
    subnet_id      = aws_subnet.tf_vpc_prv_subnet.id
    route_table_id = aws_route_table.tf_vpc_prv_route.id
}

# Security Group
resource "aws_security_group" "tf_vpc_sg" {
    vpc_id = aws_vpc.tf_vpc.id

    ingress {
        from_port   = 22
        to_port     = 22
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    }

    egress {
        from_port   = 0
        to_port     = 0
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
    }

    tags = {
        Name = "tf_vpc_sg"
    }
}

# Bonus Task - PEM File
resource "aws_key_pair" "tf_vpc_key" {
    key_name   = "tf_vpc_key"
    public_key = file("~/.ssh/tf_vpc_id_rsa.pub")
    tags = {
        Name = "tf_vpc_key"
    }
}
```
