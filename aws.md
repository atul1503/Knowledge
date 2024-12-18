There is a profile assocated with each IAM user or role.
A profile has its own `Access Key ID`(think of it as username) and `Secret Access Key`(think of it as password).  
To use these 2 credentials you can put them in `~/.aws/credentials` like this:
```
[YourProfileName]
aws_access_key_id = AKIAEXAMPLEEXAM
aws_secret_access_key = Orwr8yWoQK7MDENG/bPxRfiCYEXAMPLEANOTHERKEY
```

There is also a default profile so that if you don't specify a profile, it will be used.

You can specify multiple profile in the same credentials file, like this:
```
[Profile1]
aws_access_key_id =
aws_secret_access_key = 

[Profile2]
aws_access_key_id =
aws_secret_access_key = 
```

The profile name can be chosen by you. AWS only identifies the key and key id not the profile name.


## Where is multiple zones or subnets required
Zones are like different parts of regions. You specify different zones or availability zone for high availability. 
Each zone has only subnet.
Autoscaling group is required because you distribute ec2 instances across multiple zones.
Load balancer also requires multiple subnets because if one zone is down then LB can route traffic through another zone or subnet.

## How to restrict application instances to only get traffic from load balancer

To make sure that the application ec2 intances only get traffic from the load balancer, then make sure that you add the load balancer's security group to the ingress(inbound) rule in the instance's security group on the specific application port. 
This will tell the instance's security group to allow all traffic from all instances or services who are using the load balancer's security group(including the load balancer). But with this, make sure that any malicious instances are not configured with the load balancers's security group. 


## How to create an IAM role and give access to a user,service or instance.

Create a role with trust policy. In that policy action should be `sts:AssumeRole`. In the principal, provide what service type, user or group can assume this role. Then create an access policy where you define the permissions where action is the set of permissions allowed by this policy. Also specify the resource on which you can to do this action. Do this by specifying the arn of those resources. Effect will be allow. Now for mentioned principal to assume this role, there are several methods. For ec2,they have to call `aws sts assume-role` and specify the arn of the role, aws will return the access key id and secret access key and session token. Put all this in the crendentials file or export it as an env variable. Now this principal has assumed the role.


In trust policy, you specify using principal who can assume this role. It can user, group or service.
A policy becomes a trust policy only when in the action you provide `sts:AssumeRole`.
The access policy should be attached to the role, to allow the principal entity to access whatever actions are allowed by access policy.


# Services

1. Cloudfront is used as a CDN which caches your content in multiple edge locations and delivers it when its requested. But it must have some origin. The origin can be an s3 bucket or even a ELB. For s3, provide s3 bucket name to be used as origin, for elb, you can take help of cloud front so that elb doesnot have to server static content everytime. You can hit the cloud front domain name to access the content served by its origin, but you can also make its dn entry in route 53.

