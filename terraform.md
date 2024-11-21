`terraform init` is used to download provider plugins and setup backend etc. It doesnot create or destroy resources etc. Use `-upgrade` option only when you are changing provider versions.

Adding this is required to any of your .tf files in current directory

```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0.0"
    }
  }
}


provider "aws" {
        region = "eu-north-1"
        profile = "terraform"
    }
```

version can be any stable version. Obviously, to apply the changes use `terraform init`.

Terraform does not need to start with an entry tf file, it uses all files in the current directory and does it thing. So you can keep any resources,variable in any file you want. But the file extension should be .tf .It also uses the tf files in subdirectories also.
