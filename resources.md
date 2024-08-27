```bash
provider "aws" {
    region = "us-east-1"
    access_key = "<aws user access key"
    secret_key = "aws user secret access key"
}

resource "aws_s3_bucket" "terraform_state" {
    bucket = "DepiLab-terraform-state-bucket"
}

resource "aws_dynamodb_table" "terraform_lock" {
    name         = "terraform-lock"
    hash_key     = "LockID"
    billing_mode = "PAY_PER_REQUEST"

    attribute {
        name = "LockID"
        type = "S"
    }
}
```
