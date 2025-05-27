# AWS Notes

## Authentication & Access

1. **Every AWS user gets a profile with credentials.** There is a profile associated with each IAM user or role. A profile has its own `Access Key ID` (think of it as username) and `Secret Access Key` (think of it as password).
   ```
   [YourProfileName]
   aws_access_key_id = AKIAEXAMPLEEXAM
   aws_secret_access_key = Orwr8yWoQK7MDENG/bPxRfiCYEXAMPLEANOTHERKEY
   ```

2. **You can have multiple profiles in the same credentials file.** Put them in `~/.aws/credentials`. There is also a default profile so that if you don't specify a profile, it will be used.
   ```
   [default]                    # Default profile used when none specified
   aws_access_key_id = YOUR_KEY
   aws_secret_access_key = YOUR_SECRET
   
   [Profile1]                   # Named profile for one environment
   aws_access_key_id = PROD_KEY
   aws_secret_access_key = PROD_SECRET
   
   [Profile2]                   # Named profile for another environment
   aws_access_key_id = DEV_KEY
   aws_secret_access_key = DEV_SECRET
   ```
   The profile name can be chosen by you. AWS only identifies the key and key id not the profile name.

3. **How to create an IAM role and give access to a user, service or instance.** Create a role with trust policy. In that policy action should be `sts:AssumeRole`. In the principal, provide what service type, user or group can assume this role. Then create an access policy where you define the permissions where action is the set of permissions allowed by this policy. Also specify the resource on which you want to do this action. Do this by specifying the arn of those resources. Effect will be allow. Now for mentioned principal to assume this role, there are several methods. For ec2, they have to call `aws sts assume-role` and specify the arn of the role, aws will return the access key id and secret access key and session token. Put all this in the credentials file or export it as an env variable. Now this principal has assumed the role.

   In trust policy, you specify using principal who can assume this role. It can be user, group or service. A policy becomes a trust policy only when in the action you provide `sts:AssumeRole`. The access policy should be attached to the role, to allow the principal entity to access whatever actions are allowed by access policy.

   ```json
   // Trust Policy - who can assume this role
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "Service": "ec2.amazonaws.com"    // EC2 instances can assume this role
         },
         "Action": "sts:AssumeRole"          // This makes it a trust policy
       }
     ]
   }
   
   // Access Policy - what permissions this role has
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",                  // Allow these actions
         "Action": [
           "s3:GetObject",                   // Can download files from S3
           "s3:PutObject"                    // Can upload files to S3
         ],
         "Resource": "arn:aws:s3:::my-bucket/*"    // Only on this specific bucket
       }
     ]
   }
   ```

## Compute Services

4. **EC2 is like renting computers in the cloud.** You pick the size, operating system, and pay by the hour.
   ```bash
   # Launch an instance
   aws ec2 run-instances \
     --image-id ami-0abcdef1234567890 \      # Which operating system image to use
     --instance-type t3.micro \              # How powerful the computer should be
     --key-name my-key-pair \                # SSH key to access the instance
     --security-group-ids sg-12345678 \      # Which firewall rules to apply
     --subnet-id subnet-12345678             # Which network to put it in
   ```

5. **Security Groups are like firewalls for your instances.** They control what traffic can come in (ingress) and go out (egress).
   ```json
   {
     "GroupName": "web-server-sg",
     "Description": "Security group for web servers",
     "SecurityGroupRules": [
       {
         "IpPermissions": [
           {
             "IpProtocol": "tcp",            // Protocol type (tcp, udp, icmp)
             "FromPort": 80,                 // Starting port number
             "ToPort": 80,                   // Ending port number (same = single port)
             "IpRanges": [{"CidrIp": "0.0.0.0/0"}]    // Who can access (0.0.0.0/0 = everyone)
           },
           {
             "IpProtocol": "tcp",
             "FromPort": 443,                // HTTPS port
             "ToPort": 443,
             "IpRanges": [{"CidrIp": "0.0.0.0/0"}]
           }
         ]
       }
     ]
   }
   ```

6. **Auto Scaling Groups automatically add or remove instances** based on demand. Like having a smart assistant that hires more workers when busy.
   ```json
   {
     "AutoScalingGroupName": "web-servers-asg",
     "LaunchTemplate": {
       "LaunchTemplateName": "web-server-template",    // Template defining what instances to create
       "Version": "1"
     },
     "MinSize": 2,                          // Never go below 2 instances
     "MaxSize": 10,                         // Never go above 10 instances
     "DesiredCapacity": 3,                  // Try to maintain 3 instances normally
     "VPCZoneIdentifier": [                 // Which subnets to spread instances across
       "subnet-12345678",
       "subnet-87654321"
     ],
     "HealthCheckType": "ELB",              // Use load balancer to check if instances are healthy
     "HealthCheckGracePeriod": 300          // Wait 5 minutes before checking new instances
   }
   ```

## Storage Services

7. **S3 is like a giant file cabinet in the cloud.** You create buckets (folders) and put objects (files) in them.
   ```bash
   # Create a bucket
   aws s3 mb s3://my-unique-bucket-name-12345
   
   # Upload a file
   aws s3 cp myfile.txt s3://my-bucket/folder/myfile.txt
   
   # Download a file
   aws s3 cp s3://my-bucket/folder/myfile.txt ./downloaded-file.txt
   
   # List everything in a bucket
   aws s3 ls s3://my-bucket --recursive
   ```

8. **EBS volumes are like external hard drives for EC2 instances.** They persist data even when the instance is terminated.
   ```bash
   # Create a volume
   aws ec2 create-volume \
     --size 20 \                            # Size in GB
     --volume-type gp3 \                    # Type of storage (gp3 = general purpose SSD)
     --availability-zone us-east-1a         # Must be in same zone as instance
   
   # Attach to an instance
   aws ec2 attach-volume \
     --volume-id vol-12345678 \
     --instance-id i-87654321 \
     --device /dev/sdf                      # Where to attach it on the instance
   ```

## Networking & High Availability

9. **Where multiple zones or subnets are required.** Zones are like different parts of regions. You specify different zones or availability zone for high availability. Each zone has only one subnet. Autoscaling group is required because you distribute ec2 instances across multiple zones. Load balancer also requires multiple subnets because if one zone is down then LB can route traffic through another zone or subnet.

10. **How to restrict application instances to only get traffic from load balancer.** To make sure that the application ec2 instances only get traffic from the load balancer, then make sure that you add the load balancer's security group to the ingress (inbound) rule in the instance's security group on the specific application port. This will tell the instance's security group to allow all traffic from all instances or services who are using the load balancer's security group (including the load balancer). But with this, make sure that any malicious instances are not configured with the load balancer's security group.

11. **VPC is like your own private network in AWS.** Everything you create goes inside a VPC, like rooms in a house.
   ```json
   {
     "VpcId": "vpc-12345678",
     "CidrBlock": "10.0.0.0/16",           // IP address range for this VPC (65,536 addresses)
     "State": "available",
     "Tags": [
       {
         "Key": "Name",
         "Value": "MyAppVPC"
       }
     ]
   }
   ```

12. **Subnets divide your VPC into smaller networks.** Like dividing a house into rooms. Each subnet exists in one availability zone.
   ```json
   {
     "SubnetId": "subnet-12345678",
     "VpcId": "vpc-12345678",
     "CidrBlock": "10.0.1.0/24",           // Subset of VPC's IP range (256 addresses)
     "AvailabilityZone": "us-east-1a",     // Which physical location this subnet is in
     "MapPublicIpOnLaunch": true,          // Auto-assign public IPs to instances
     "Tags": [
       {
         "Key": "Name",
         "Value": "Public Subnet 1"
       }
     ]
   }
   ```

## Load Balancing

13. **Application Load Balancer distributes traffic across multiple instances.** Like a traffic cop directing cars to different lanes.
   ```json
   {
     "LoadBalancerName": "my-app-alb",
     "Scheme": "internet-facing",           // Can receive traffic from internet
     "Type": "application",                 // Handles HTTP/HTTPS traffic
     "Subnets": [                          // Must be in multiple availability zones
       "subnet-12345678",
       "subnet-87654321"
     ],
     "SecurityGroups": ["sg-12345678"],
     "Tags": [
       {
         "Key": "Environment",
         "Value": "production"
       }
     ]
   }
   ```

14. **Target Groups define which instances receive traffic from the load balancer.** Like a list of workers who can handle requests.
   ```json
   {
     "TargetGroupName": "web-servers-tg",
     "Protocol": "HTTP",                    // How to communicate with targets
     "Port": 80,                           // Which port to send traffic to
     "VpcId": "vpc-12345678",
     "HealthCheckPath": "/health",          // URL to check if target is healthy
     "HealthCheckIntervalSeconds": 30,      // How often to check health
     "HealthyThresholdCount": 2,            // How many successful checks = healthy
     "UnhealthyThresholdCount": 5,          // How many failed checks = unhealthy
     "Targets": [                          // Which instances to send traffic to
       {
         "Id": "i-12345678",               // Instance ID
         "Port": 80                        // Port on the instance
       }
     ]
   }
   ```

## Services

15. **CloudFront is used as a CDN** which caches your content in multiple edge locations and delivers it when it's requested. But it must have some origin. The origin can be an S3 bucket or even an ELB. For S3, provide S3 bucket name to be used as origin, for ELB, you can take help of CloudFront so that ELB doesn't have to serve static content every time. You can hit the CloudFront domain name to access the content served by its origin, but you can also make its DNS entry in Route 53.
   ```json
   {
     "DistributionConfig": {
       "CallerReference": "my-app-cdn-2024",    // Unique identifier for this distribution
       "Comment": "CDN for my web application",
       "Origins": [                             // Where to get the original content
         {
           "Id": "S3-my-bucket",               // Name for this origin
           "DomainName": "my-bucket.s3.amazonaws.com",    // Where the content lives
           "S3OriginConfig": {
             "OriginAccessIdentity": ""        // Leave empty for public buckets
           }
         }
       ],
       "DefaultCacheBehavior": {               // How to handle requests
         "TargetOriginId": "S3-my-bucket",     // Which origin to use
         "ViewerProtocolPolicy": "redirect-to-https",    // Force HTTPS
         "CachePolicyId": "managed-caching-optimized",   // How long to cache content
         "Compress": true                      // Compress files for faster delivery
       },
       "Enabled": true,                        // Turn on the distribution
       "PriceClass": "PriceClass_100"          // Use cheapest edge locations only
     }
   }
   ```

## DNS Management

16. **Route53 is used for mapping IP address or other domains to other DNS names.** Records are created in hosted zones. Each hosted zone is like a main domain like `develgs.com` or `google.com` and inside it you define subdomains for that hosted zone. There are two types of records inside each hosted zone. One is CNAME and other A record. Use CNAME to map a URL to another URL. Use A record for mapping IP address to a URL. TTL is the amount of time in seconds your website responses will be cached in browsers or other HTTP clients.
   ```json
   {
     "HostedZone": {
       "Name": "myapp.com.",                 // Domain name (note the trailing dot)
       "CallerReference": "myapp-2024-01",   // Unique identifier
       "Config": {
         "Comment": "DNS for my application",
         "PrivateZone": false                // Public zone (accessible from internet)
       }
     },
     "RecordSets": [
       {
         "Name": "myapp.com.",               // Root domain
10. **VPC is like your own private network in AWS.** Everything you create goes inside a VPC, like rooms in a house.
    ```json
    {
      "VpcId": "vpc-12345678",
      "CidrBlock": "10.0.0.0/16",           // IP address range for this VPC (65,536 addresses)
      "State": "available",
      "Tags": [
        {
          "Key": "Name",
          "Value": "MyAppVPC"
        }
      ]
    }
    ```

11. **Subnets divide your VPC into smaller networks.** Like dividing a house into rooms. Each subnet exists in one availability zone.
    ```json
    {
      "SubnetId": "subnet-12345678",
      "VpcId": "vpc-12345678",
      "CidrBlock": "10.0.1.0/24",           // Subset of VPC's IP range (256 addresses)
      "AvailabilityZone": "us-east-1a",     // Which physical location this subnet is in
      "MapPublicIpOnLaunch": true,          // Auto-assign public IPs to instances
      "Tags": [
        {
          "Key": "Name",
          "Value": "Public Subnet 1"
        }
      ]
    }
    ```

12. **Internet Gateway connects your VPC to the internet.** Like the front door of your house.
    ```bash
    # Create internet gateway
    aws ec2 create-internet-gateway
    
    # Attach it to your VPC
    aws ec2 attach-internet-gateway \
      --internet-gateway-id igw-12345678 \
      --vpc-id vpc-12345678
    ```

13. **Route Tables tell traffic where to go.** Like GPS directions for network packets.
    ```json
    {
      "RouteTableId": "rtb-12345678",
      "VpcId": "vpc-12345678",
      "Routes": [
        {
          "DestinationCidrBlock": "10.0.0.0/16",    // Traffic going to VPC addresses
          "GatewayId": "local"                      // Stay local (don't leave VPC)
        },
        {
          "DestinationCidrBlock": "0.0.0.0/0",      // All other traffic (internet)
          "GatewayId": "igw-12345678"               // Send through internet gateway
        }
      ]
    }
    ```

## Load Balancing

14. **Application Load Balancer distributes traffic across multiple instances.** Like a traffic cop directing cars to different lanes.
    ```json
    {
      "LoadBalancerName": "my-app-alb",
      "Scheme": "internet-facing",           // Can receive traffic from internet
      "Type": "application",                 // Handles HTTP/HTTPS traffic
      "Subnets": [                          // Must be in multiple availability zones
        "subnet-12345678",
        "subnet-87654321"
      ],
      "SecurityGroups": ["sg-12345678"],
      "Tags": [
        {
          "Key": "Environment",
          "Value": "production"
        }
      ]
    }
    ```

15. **Target Groups define which instances receive traffic from the load balancer.** Like a list of workers who can handle requests.
    ```json
    {
      "TargetGroupName": "web-servers-tg",
      "Protocol": "HTTP",                    // How to communicate with targets
      "Port": 80,                           // Which port to send traffic to
      "VpcId": "vpc-12345678",
      "HealthCheckPath": "/health",          // URL to check if target is healthy
      "HealthCheckIntervalSeconds": 30,      // How often to check health
      "HealthyThresholdCount": 2,            // How many successful checks = healthy
      "UnhealthyThresholdCount": 5,          // How many failed checks = unhealthy
      "Targets": [                          // Which instances to send traffic to
        {
          "Id": "i-12345678",               // Instance ID
          "Port": 80                        // Port on the instance
        }
      ]
    }
    ```

## Content Delivery

16. **CloudFront is a CDN that caches your content worldwide.** Like having copies of your store in every major city.
    ```json
    {
      "DistributionConfig": {
        "CallerReference": "my-app-cdn-2024",    // Unique identifier for this distribution
        "Comment": "CDN for my web application",
        "Origins": [                             // Where to get the original content
          {
            "Id": "S3-my-bucket",               // Name for this origin
            "DomainName": "my-bucket.s3.amazonaws.com",    // Where the content lives
            "S3OriginConfig": {
              "OriginAccessIdentity": ""        // Leave empty for public buckets
            }
          }
        ],
        "DefaultCacheBehavior": {               // How to handle requests
          "TargetOriginId": "S3-my-bucket",     // Which origin to use
          "ViewerProtocolPolicy": "redirect-to-https",    // Force HTTPS
          "CachePolicyId": "managed-caching-optimized",   // How long to cache content
          "Compress": true                      // Compress files for faster delivery
        },
        "Enabled": true,                        // Turn on the distribution
        "PriceClass": "PriceClass_100"          // Use cheapest edge locations only
      }
    }
    ```

## DNS Management

17. **Route 53 is like a phone book for the internet.** It translates domain names to IP addresses.
    ```json
    {
      "HostedZone": {
        "Name": "myapp.com.",                 // Domain name (note the trailing dot)
        "CallerReference": "myapp-2024-01",   // Unique identifier
        "Config": {
          "Comment": "DNS for my application",
          "PrivateZone": false                // Public zone (accessible from internet)
        }
      },
      "RecordSets": [
        {
          "Name": "myapp.com.",               // Root domain
          "Type": "A",                        // A record maps domain to IP address
          "TTL": 300,                         // How long browsers should cache this (seconds)
          "ResourceRecords": [
            {"Value": "203.0.113.1"}          // IP address to point to
          ]
        },
        {
          "Name": "www.myapp.com.",           // Subdomain
          "Type": "CNAME",                    // CNAME maps domain to another domain
          "TTL": 300,
          "ResourceRecords": [
            {"Value": "myapp.com."}           // Point to root domain
          ]
        }
      ]
    }
    ```

## Database Services

18. **RDS manages databases for you.** Like having a database administrator who handles backups, updates, and scaling.
    ```json
    {
      "DBInstanceIdentifier": "myapp-database",
      "DBInstanceClass": "db.t3.micro",      // Size of the database server
      "Engine": "mysql",                     // Database type (mysql, postgres, etc.)
      "EngineVersion": "8.0.35",             // Specific version of the database
      "MasterUsername": "admin",             // Root username
      "MasterUserPassword": "supersecret123", // Root password
      "AllocatedStorage": 20,                // Storage size in GB
      "StorageType": "gp2",                  // Type of storage (gp2 = general purpose SSD)
      "VpcSecurityGroupIds": ["sg-12345678"], // Which security groups to apply
      "DBSubnetGroupName": "my-db-subnet-group", // Which subnets database can use
      "BackupRetentionPeriod": 7,            // Keep backups for 7 days
      "MultiAZ": true,                       // Create standby in different zone for high availability
      "StorageEncrypted": true               // Encrypt data at rest
    }
    ```

## Serverless Computing

19. **Lambda runs your code without managing servers.** Like having a magic function that runs when triggered.
    ```python
    # Lambda function code
    import json
    
    def lambda_handler(event, context):      # Entry point for Lambda function
        # event = data passed to function (HTTP request, S3 event, etc.)
        # context = runtime information (time remaining, request ID, etc.)
        
        name = event.get('name', 'World')    # Get name from input, default to 'World'
        
        return {
            'statusCode': 200,               # HTTP status code
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*'    # Allow CORS
            },
            'body': json.dumps({             # Response body (must be string)
                'message': f'Hello, {name}!',
                'requestId': context.aws_request_id
            })
        }
    ```

20. **API Gateway creates REST APIs that trigger Lambda functions.** Like a receptionist that routes requests to the right department.
    ```json
    {
      "RestApiId": "abc123def4",
      "Name": "MyAppAPI",
      "Description": "API for my application",
      "Resources": [
        {
          "PathPart": "users",               // URL path segment
          "Methods": [
            {
              "HttpMethod": "GET",           // HTTP method
              "Integration": {
                "Type": "AWS_PROXY",         // Forward everything to Lambda
                "IntegrationHttpMethod": "POST", // Lambda always uses POST
                "Uri": "arn:aws:lambda:us-east-1:123456789012:function:GetUsers"
              },
              "MethodResponses": [
                {
                  "StatusCode": "200",       // Expected response code
                  "ResponseModels": {
                    "application/json": "Empty"
                  }
                }
              ]
            }
          ]
        }
      ]
    }
    ```

## Monitoring & Logging

21. **CloudWatch monitors your AWS resources.** Like having security cameras and sensors throughout your infrastructure.
    ```json
    {
      "MetricName": "CPUUtilization",       // What to measure
      "Namespace": "AWS/EC2",               // Which service this metric belongs to
      "Dimensions": [                       // Additional details about what's being measured
        {
          "Name": "InstanceId",
          "Value": "i-12345678"             // Which specific instance
        }
      ],
      "Statistic": "Average",               // How to aggregate data (Average, Sum, Maximum, etc.)
      "Period": 300,                        // Time period in seconds (5 minutes)
      "EvaluationPeriods": 2,               // How many periods to look at
      "Threshold": 80.0,                    // Trigger alarm when CPU > 80%
      "ComparisonOperator": "GreaterThanThreshold",
      "AlarmActions": [                     // What to do when alarm triggers
        "arn:aws:sns:us-east-1:123456789012:cpu-alerts"    // Send notification
      ]
    }
    ```

## Key Concepts

22. **Availability Zones are like different buildings in the same city.** Each zone has its own power and network, so if one fails, others keep running.

23. **Regions are like different countries.** Each region is completely separate, with its own set of availability zones. Choose regions close to your users for better performance.

24. **ARNs (Amazon Resource Names) are like postal addresses for AWS resources.** Every resource has a unique ARN that looks like: `arn:aws:service:region:account-id:resource-type/resource-name`

25. **Tags are like sticky notes you put on resources.** Use them to organize, track costs, and manage resources. Common tags: Environment (dev/prod), Owner, Project, Cost-Center.

## Security Best Practices

26. **Never put credentials in your code.** Use IAM roles for EC2 instances, environment variables for Lambda, or AWS Secrets Manager for sensitive data.

27. **Use least privilege principle.** Only give the minimum permissions needed. Start with no access and add permissions as needed.

28. **Enable MFA (Multi-Factor Authentication) on all accounts.** Especially for root accounts and admin users.

29. **Encrypt everything.** Use encryption at rest for S3, EBS, RDS. Use encryption in transit with HTTPS/TLS.

30. **Monitor and log everything.** Enable CloudTrail for API calls, VPC Flow Logs for network traffic, and CloudWatch for metrics and logs.
