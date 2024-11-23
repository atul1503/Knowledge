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


## Application load balancing in AWS

Dont worry about load balancer scaling. It is managed by AWS. AWS scales load balancers if traffic is more.
In Terraform for EC2 applications, you can setup an autoscaling group and create a cloud watch alarm and attach the alarm using alarm actions to an autoscaling policy. The policy is connected to your autoscaling group by its name. So when this alarm gets triggered, it triggers your auto scaling policy and this autoscaling policy is the one who will force autoscaling group to scale up or down the number of applications.
