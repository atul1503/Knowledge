# Azure Notes

## Authentication & Access

1. **Every Azure user gets credentials through Azure Active Directory (AAD).** Think of AAD like a bouncer at a club - it checks who you are and what you're allowed to do. You can authenticate using username/password, service principals (like robot accounts), or managed identities.
   ```bash
   # Login with your user account (interactive - opens browser)
   az login
   
   # Login with service principal (for automation - no browser needed)
   # You DON'T use your personal username/password here!
   # Instead, you use the service principal's credentials:
   az login --service-principal \
     --username <app-id> \                 # This is the service principal's "username" (actually called App ID or Client ID)
     --password <password> \               # This is the service principal's "password" (actually called Client Secret)
     --tenant <tenant-id>                  # Which Azure AD tenant this belongs to
   
   # Example with real values:
   az login --service-principal \
     --username "12345678-1234-1234-1234-123456789012" \    # App ID from service principal creation
     --password "super-secret-generated-password" \          # Client Secret from service principal creation
     --tenant "87654321-4321-4321-4321-210987654321"        # Your organization's tenant ID
   ```

2. **Service Principals are like robot accounts for applications.** When your app needs to access Azure resources, create a service principal instead of using your personal account. Think of it as creating a special user account just for your application.
   ```bash
   # Create a service principal - this generates its own credentials automatically
   az ad sp create-for-rbac --name "MyAppServicePrincipal" \
     --role contributor \                    # What permissions it gets (explained below)
     --scopes /subscriptions/<subscription-id>    # Where it can access (explained below)
   
   # This command returns the credentials you'll use for login:
   # {
   #   "appId": "12345678-1234-1234-1234-123456789012",      # This becomes the --username
   #   "displayName": "MyAppServicePrincipal",               # Human-readable name
   #   "password": "super-secret-password",                   # This becomes the --password
   #   "tenant": "87654321-4321-4321-4321-210987654321"      # This becomes the --tenant
   # }
   
   # IMPORTANT: Save these credentials securely! Azure won't show the password again.
   # You'll use these exact values in the az login command above.
   
   # What each credential means:
   # - appId: The service principal's unique identifier (like a username)
   # - password: The service principal's secret key (like a password)
   # - tenant: Which Azure AD organization this belongs to
   # - displayName: Just a friendly name for humans to read
   ```

3. **Managed Identities are like automatic ID cards for Azure resources.** Azure automatically creates and manages credentials for your VMs, App Services, etc. No passwords to manage!
   ```json
   {
     "type": "SystemAssigned",              // Azure creates and manages this automatically
     "principalId": "12345678-abcd-efgh-ijkl-123456789012",    // Unique ID for this identity
     "tenantId": "87654321-4321-4321-4321-210987654321"       // Which Azure AD tenant it belongs to
   }
   ```

4. **Role-Based Access Control (RBAC) is like giving people different keys to different rooms.** Assign roles to users, groups, or service principals to control what they can do.
   ```bash
   # Give someone contributor access to a resource group
   az role assignment create \
     --assignee user@company.com \          # Who gets the permission
     --role "Contributor" \                 # What level of access (Owner, Contributor, Reader, etc.)
     --scope "/subscriptions/<sub-id>/resourceGroups/myapp-rg"    # Where they can use it
   ```

## Resource Organization

5. **Subscriptions are like separate credit cards for different projects.** Each subscription has its own billing, resource limits, and access controls. You can have multiple subscriptions under one Azure account.
   ```bash
   # List all your subscriptions
   az account list --output table
   
   # Example output:
   # Name                    CloudName    SubscriptionId                        State    IsDefault
   # ----------------------  -----------  ------------------------------------  -------  -----------
   # Production Subscription AzureCloud   11111111-2222-3333-4444-555555555555  Enabled  True
   # Development Subscription AzureCloud  66666666-7777-8888-9999-000000000000  Enabled  False
   # Testing Subscription    AzureCloud   aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee  Enabled  False
   
   # Switch to a different subscription
   az account set --subscription "Production Subscription"
   # Or use the subscription ID directly:
   az account set --subscription "11111111-2222-3333-4444-555555555555"
   
   # What subscriptions are used for:
   # 1. BILLING: Each subscription gets its own bill - you can separate costs by project/department
   # 2. LIMITS: Each subscription has resource limits (e.g., max 25,000 VMs)
   # 3. ACCESS CONTROL: You can give different people access to different subscriptions
   # 4. ORGANIZATION: Group related resources together (prod vs dev vs test)
   
   # Common subscription patterns:
   # - One subscription per environment (Production, Staging, Development)
   # - One subscription per department (Engineering, Marketing, Sales)
   # - One subscription per project (Project A, Project B, Project C)
   # - One subscription per cost center for billing purposes
   
   # Subscription ID is used in:
   # - Service principal scopes: /subscriptions/<subscription-id>
   # - Resource IDs: /subscriptions/<subscription-id>/resourceGroups/...
   # - ARM templates and scripts
   # - Cost management and billing reports
   ```

## RBAC Roles Explained

**Built-in Azure Roles (Predefined):**
Azure comes with many predefined roles. Here are the most common ones:

```bash
# OWNER - Can do everything including manage access for others
# This is like being an admin - full control over resources AND who can access them
az role assignment create \
  --assignee user@company.com \
  --role "Owner" \
  --scope "/subscriptions/11111111-2222-3333-4444-555555555555"

# CONTRIBUTOR - Can create/modify/delete resources but CAN'T manage access
# This is like being a developer - you can build and deploy but can't give permissions to others
az role assignment create \
  --assignee user@company.com \
  --role "Contributor" \
  --scope "/subscriptions/11111111-2222-3333-4444-555555555555/resourceGroups/myapp-rg"

# READER - Can only view resources, cannot make any changes
# This is like being an auditor - you can see everything but can't modify anything
az role assignment create \
  --assignee user@company.com \
  --role "Reader" \
  --scope "/subscriptions/11111111-2222-3333-4444-555555555555"

# More specific predefined roles:
# - "Virtual Machine Contributor" - Can manage VMs but not networks or storage
# - "Storage Account Contributor" - Can manage storage accounts
# - "SQL DB Contributor" - Can manage SQL databases
# - "Website Contributor" - Can manage web apps
# - "Monitoring Reader" - Can read monitoring data
# - "Key Vault Administrator" - Can manage Key Vault and its contents

# You can also create custom roles if the predefined ones don't fit your needs
```

**Understanding Scopes (Where permissions apply):**
```bash
# Subscription level - access to entire subscription
--scope "/subscriptions/11111111-2222-3333-4444-555555555555"

# Resource group level - access to specific resource group and everything in it
--scope "/subscriptions/11111111-2222-3333-4444-555555555555/resourceGroups/myapp-rg"

# Resource level - access to specific resource only
--scope "/subscriptions/11111111-2222-3333-4444-555555555555/resourceGroups/myapp-rg/providers/Microsoft.Compute/virtualMachines/myapp-vm"

# Management group level - access to multiple subscriptions (enterprise feature)
--scope "/providers/Microsoft.Management/managementGroups/mycompany-mg"
```

**Real-world example of service principal with contributor role:**
```bash
# 1. Create service principal for a CI/CD pipeline
az ad sp create-for-rbac --name "MyApp-CICD-Pipeline" \
  --role "Contributor" \
  --scopes "/subscriptions/11111111-2222-3333-4444-555555555555/resourceGroups/myapp-production-rg"

# This gives the service principal permission to:
# ✅ Create, modify, and delete resources in the myapp-production-rg resource group
# ✅ Deploy applications, create VMs, modify databases, etc.
# ❌ Cannot give permissions to other users (that requires Owner role)
# ❌ Cannot access other resource groups or subscriptions

# 2. Use the service principal in your CI/CD pipeline
# The pipeline script would use these credentials:
az login --service-principal \
  --username "12345678-1234-1234-1234-123456789012" \
  --password "generated-secret-password" \
  --tenant "87654321-4321-4321-4321-210987654321"

# 3. Now the pipeline can deploy resources
az webapp create --name myapp --resource-group myapp-production-rg --plan myapp-plan
```

6. **Resource Groups are like folders for organizing related resources.** Everything in Azure must belong to a resource group. Think of it as putting all resources for one project in the same folder.
   ```bash
   # Create a resource group
   az group create \
     --name myapp-rg \                      # Name for this resource group
     --location eastus                      # Which Azure region to create it in
   ```

7. **Tags are like sticky notes you put on resources.** Use them to organize, track costs, and manage resources across different resource groups.
   ```json
   {
     "tags": {
       "Environment": "Production",          // Which environment (dev, staging, prod)
       "Project": "MyWebApp",               // Which project this belongs to
       "Owner": "DevTeam",                  // Who owns this resource
       "CostCenter": "Engineering"          // For billing and cost tracking
     }
   }
   ```

## Compute Services

8. **Virtual Machines are like renting computers in the cloud.** Pick the size, operating system, and pay by the hour, just like AWS EC2.
   ```bash
   # Create a VM
   az vm create \
     --resource-group myapp-rg \
     --name myapp-vm \
     --image UbuntuLTS \                    # Which operating system image
     --size Standard_B2s \                  # How powerful (B2s = 2 vCPUs, 4GB RAM)
     --admin-username azureuser \           # Username to login with
     --generate-ssh-keys \                  # Create SSH keys automatically
     --public-ip-sku Standard              # Create a public IP address
   ```

9. **Network Security Groups (NSGs) are like firewalls for your VMs.** They control what traffic can reach your virtual machines.
   ```json
   {
     "securityRules": [
       {
         "name": "AllowHTTP",
         "protocol": "Tcp",                  // Protocol type (Tcp, Udp, *)
         "sourcePortRange": "*",             // Source port (* = any port)
         "destinationPortRange": "80",       // Destination port (80 = HTTP)
         "sourceAddressPrefix": "*",         // Source IP (* = anywhere)
         "destinationAddressPrefix": "*",    // Destination IP (* = any VM in this NSG)
         "access": "Allow",                  // Allow or Deny
         "priority": 1000,                   // Lower numbers = higher priority
         "direction": "Inbound"              // Inbound (incoming) or Outbound (outgoing)
       }
     ]
   }
   ```

10. **Virtual Machine Scale Sets automatically add or remove VMs** based on demand. Like Auto Scaling Groups in AWS.
    ```json
    {
      "sku": {
        "name": "Standard_B2s",             // Size of each VM in the scale set
        "capacity": 3                       // How many VMs to start with
      },
      "upgradePolicy": {
        "mode": "Automatic"                 // How to handle updates (Automatic, Manual, Rolling)
      },
      "virtualMachineProfile": {
        "osProfile": {
          "computerNamePrefix": "myapp",    // Prefix for VM names (myapp-1, myapp-2, etc.)
          "adminUsername": "azureuser"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "Canonical",       // Who made this image
            "offer": "UbuntuServer",        // Which OS family
            "sku": "18.04-LTS"             // Specific version
          }
        }
      },
      "autoscaleSettings": {
        "profiles": [
          {
            "capacity": {
              "minimum": "2",               // Never go below 2 VMs
              "maximum": "10",              // Never go above 10 VMs
              "default": "3"                // Normal number of VMs
            },
            "rules": [
              {
                "metricTrigger": {
                  "metricName": "Percentage CPU",    // What to measure
                  "threshold": 75                    // Scale out when CPU > 75%
                }
              }
            ]
          }
        ]
      }
    }
    ```

## App Services & Containers

11. **App Service is like a managed web hosting platform.** You just upload your code and Azure handles the servers, scaling, and load balancing.
    ```bash
    # Create an App Service plan (the server that runs your app)
    az appservice plan create \
      --name myapp-plan \
      --resource-group myapp-rg \
      --sku B1 \                           # Pricing tier (F1=Free, B1=Basic, S1=Standard, P1=Premium)
      --is-linux                           # Use Linux servers
    
    # Create the web app
    az webapp create \
      --resource-group myapp-rg \
      --plan myapp-plan \
      --name myapp-webapp \
      --runtime "NODE|14-lts"              # Which runtime (Node.js, Python, .NET, etc.)
    ```

12. **Container Instances run Docker containers without managing VMs.** Like AWS Fargate - just give it a container image and it runs.
    ```bash
    # Run a container
    az container create \
      --resource-group myapp-rg \
      --name myapp-container \
      --image nginx:latest \               # Which Docker image to run
      --cpu 1 \                           # How many CPU cores
      --memory 1.5 \                      # How much RAM in GB
      --ports 80 \                        # Which ports to expose
      --ip-address Public                 # Give it a public IP address
    ```

13. **Azure Kubernetes Service (AKS) is managed Kubernetes.** Azure handles the control plane, you just manage your worker nodes and applications.
    ```bash
    # Create an AKS cluster
    az aks create \
      --resource-group myapp-rg \
      --name myapp-aks \
      --node-count 3 \                    # How many worker nodes to start with
      --node-vm-size Standard_B2s \       # Size of each worker node
      --enable-addons monitoring \        # Enable Azure Monitor for containers
      --generate-ssh-keys
    
    # Get credentials to use kubectl
    az aks get-credentials --resource-group myapp-rg --name myapp-aks
    ```

## Networking

14. **Virtual Networks (VNets) are like your own private network in Azure.** Similar to AWS VPC - everything you create goes inside a VNet.
    ```json
    {
      "addressSpace": {
        "addressPrefixes": ["10.0.0.0/16"]  // IP address range for this VNet (65,536 addresses)
      },
      "subnets": [
        {
          "name": "default",
          "addressPrefix": "10.0.1.0/24",   // Subnet within the VNet (256 addresses)
          "networkSecurityGroup": {
            "id": "/subscriptions/.../networkSecurityGroups/myapp-nsg"
          }
        }
      ],
      "location": "East US",
      "tags": {
        "Environment": "Production"
      }
    }
    ```

15. **Load Balancer distributes traffic across multiple VMs.** Like AWS Application Load Balancer.
    ```json
    {
      "frontendIPConfigurations": [
        {
          "name": "LoadBalancerFrontEnd",
          "publicIPAddress": {              // Public IP that clients connect to
            "id": "/subscriptions/.../publicIPAddresses/myapp-lb-ip"
          }
        }
      ],
      "backendAddressPools": [
        {
          "name": "BackEndPool",            // Pool of VMs that receive traffic
          "loadBalancingRules": [
            {
              "name": "HTTPRule",
              "frontendPort": 80,           // Port clients connect to
              "backendPort": 80,            // Port on the VMs
              "protocol": "Tcp",
              "enableFloatingIP": false,
              "idleTimeoutInMinutes": 4
            }
          ]
        }
      ],
      "probes": [
        {
          "name": "HealthProbe",
          "protocol": "Http",
          "port": 80,
          "requestPath": "/health",         // URL to check if VM is healthy
          "intervalInSeconds": 30,          // How often to check
          "numberOfProbes": 2               // How many failed checks = unhealthy
        }
      ]
    }
    ```

16. **Application Gateway is like a smart load balancer with extra features.** It can do SSL termination, URL-based routing, and Web Application Firewall (WAF).
    ```json
    {
      "sku": {
        "name": "Standard_v2",              // Which version (v1 or v2)
        "tier": "Standard_v2",
        "capacity": 2                       // How many instances
      },
      "gatewayIPConfigurations": [
        {
          "name": "appGatewayIpConfig",
          "subnet": {
            "id": "/subscriptions/.../subnets/appgw-subnet"    // Dedicated subnet for App Gateway
          }
        }
      ],
      "frontendIPConfigurations": [
        {
          "name": "appGwPublicFrontendIp",
          "publicIPAddress": {
            "id": "/subscriptions/.../publicIPAddresses/appgw-ip"
          }
        }
      ],
      "backendAddressPools": [
        {
          "name": "appGwBackendPool",
          "backendAddresses": [
            {"ipAddress": "10.0.1.4"},      // IP addresses of backend servers
            {"ipAddress": "10.0.1.5"}
          ]
        }
      ],
      "httpListeners": [
        {
          "name": "appGwHttpListener",
          "frontendIPConfiguration": {
            "id": "/subscriptions/.../frontendIPConfigurations/appGwPublicFrontendIp"
          },
          "frontendPort": {
            "id": "/subscriptions/.../frontendPorts/port_80"
          },
          "protocol": "Http"
        }
      ]
    }
    ```

## Storage Services

17. **Storage Accounts are like containers for all your data.** You create different types of storage (blobs, files, queues, tables) inside a storage account.
    ```bash
    # Create a storage account
    az storage account create \
      --name myappstorageaccount \         # Must be globally unique, lowercase, no special chars
      --resource-group myapp-rg \
      --location eastus \
      --sku Standard_LRS \                 # Replication type (LRS=Local, GRS=Geo, ZRS=Zone)
      --kind StorageV2                     # Storage account type (StorageV2 is recommended)
    ```

18. **Blob Storage is like AWS S3 - for storing files and objects.** You create containers (like S3 buckets) and put blobs (files) in them.
    ```bash
    # Create a container
    az storage container create \
      --name myapp-files \
      --account-name myappstorageaccount \
      --public-access blob                 # Public access level (off, blob, container)
    
    # Upload a file
    az storage blob upload \
      --file ./myfile.txt \
      --name myfile.txt \
      --container-name myapp-files \
      --account-name myappstorageaccount
    
    # Download a file
    az storage blob download \
      --name myfile.txt \
      --container-name myapp-files \
      --account-name myappstorageaccount \
      --file ./downloaded-file.txt
    ```

19. **Managed Disks are like external hard drives for VMs.** They persist data even when the VM is deleted, similar to AWS EBS.
    ```bash
    # Create a managed disk
    az disk create \
      --resource-group myapp-rg \
      --name myapp-data-disk \
      --size-gb 128 \                      # Size in GB
      --sku Premium_LRS                    # Disk type (Standard_LRS, Premium_LRS, StandardSSD_LRS)
    
    # Attach to a VM
    az vm disk attach \
      --resource-group myapp-rg \
      --vm-name myapp-vm \
      --name myapp-data-disk
    ```

## Database Services

20. **Azure SQL Database is like AWS RDS for SQL Server.** Fully managed SQL database with automatic backups, scaling, and patching.
    ```bash
    # Create a SQL server (logical container for databases)
    az sql server create \
      --name myapp-sql-server \
      --resource-group myapp-rg \
      --location eastus \
      --admin-user sqladmin \
      --admin-password SuperSecret123!
    
    # Create a database
    az sql db create \
      --resource-group myapp-rg \
      --server myapp-sql-server \
      --name myapp-database \
      --service-objective S1              # Performance tier (Basic, S0-S12, P1-P15)
    ```

21. **Cosmos DB is like AWS DynamoDB - a NoSQL database that scales globally.** It supports multiple APIs (SQL, MongoDB, Cassandra, etc.).
    ```json
    {
      "databaseAccountOfferType": "Standard",
      "locations": [
        {
          "locationName": "East US",        // Primary region
          "failoverPriority": 0,            // Priority for failover (0 = highest)
          "isZoneRedundant": false
        },
        {
          "locationName": "West US",        // Secondary region for geo-replication
          "failoverPriority": 1,
          "isZoneRedundant": false
        }
      ],
      "consistencyPolicy": {
        "defaultConsistencyLevel": "Session",    // Consistency level (Strong, Session, Eventual, etc.)
        "maxIntervalInSeconds": 300,
        "maxStalenessPrefix": 100000
      },
      "capabilities": [
        {
          "name": "EnableServerless"       // Enable serverless billing (pay per request)
        }
      ]
    }
    ```

## Serverless Computing

22. **Azure Functions are like AWS Lambda - run code without managing servers.** They trigger on events like HTTP requests, timer schedules, or storage changes.
    ```python
    # Azure Function code (Python)
    import logging
    import json
    import azure.functions as func
    
    def main(req: func.HttpRequest) -> func.HttpResponse:    # Entry point for HTTP trigger
        logging.info('Python HTTP trigger function processed a request.')
        
        name = req.params.get('name')       # Get parameter from query string
        if not name:
            try:
                req_body = req.get_json()   # Try to get from request body
                name = req_body.get('name') if req_body else None
            except ValueError:
                pass
        
        if name:
            return func.HttpResponse(       # Return HTTP response
                json.dumps({
                    "message": f"Hello, {name}!",
                    "status": "success"
                }),
                status_code=200,
                headers={"Content-Type": "application/json"}
            )
        else:
            return func.HttpResponse(
                "Please pass a name in the query string or request body",
                status_code=400
            )
    ```

23. **Logic Apps are like visual workflows for connecting different services.** Think of it as a flowchart that automatically does things when events happen.
    ```json
    {
      "definition": {
        "triggers": {
          "When_a_HTTP_request_is_received": {    // What starts this workflow
            "type": "Request",
            "kind": "Http",
            "inputs": {
              "schema": {
                "type": "object",
                "properties": {
                  "email": {"type": "string"},      // Expected input format
                  "message": {"type": "string"}
                }
              }
            }
          }
        },
        "actions": {
          "Send_an_email": {                      // What to do when triggered
            "type": "ApiConnection",
            "inputs": {
              "host": {
                "connection": {
                  "name": "@parameters('$connections')['outlook']['connectionId']"
                }
              },
              "method": "post",
              "path": "/Mail",
              "body": {
                "To": "@triggerBody()?['email']",   // Use email from trigger input
                "Subject": "New Message",
                "Body": "@triggerBody()?['message']"
              }
            }
          }
        }
      }
    }
    ```

## Content Delivery & DNS

24. **Azure CDN caches your content worldwide.** Like AWS CloudFront - copies of your files stored in edge locations for faster delivery.
    ```bash
    # Create a CDN profile
    az cdn profile create \
      --name myapp-cdn \
      --resource-group myapp-rg \
      --sku Standard_Microsoft            # Pricing tier (Standard_Microsoft, Standard_Akamai, Premium_Verizon)
    
    # Create a CDN endpoint
    az cdn endpoint create \
      --name myapp-cdn-endpoint \
      --profile-name myapp-cdn \
      --resource-group myapp-rg \
      --origin myapp.azurewebsites.net \  # Where to get the original content
      --origin-host-header myapp.azurewebsites.net
    ```

25. **Azure DNS manages your domain names.** Like AWS Route 53 - translates domain names to IP addresses.
    ```bash
    # Create a DNS zone
    az network dns zone create \
      --resource-group myapp-rg \
      --name myapp.com
    
    # Add an A record (domain to IP address)
    az network dns record-set a add-record \
      --resource-group myapp-rg \
      --zone-name myapp.com \
      --record-set-name www \              # Subdomain (www.myapp.com)
      --ipv4-address 203.0.113.1          # IP address to point to
    
    # Add a CNAME record (domain to another domain)
    az network dns record-set cname set-record \
      --resource-group myapp-rg \
      --zone-name myapp.com \
      --record-set-name blog \             # Subdomain (blog.myapp.com)
      --cname myapp.azurewebsites.net      # Point to Azure web app
    ```

## Monitoring & Logging

26. **Azure Monitor collects metrics and logs from all your resources.** Like AWS CloudWatch - helps you understand what's happening in your infrastructure.
    ```json
    {
      "alertRule": {
        "name": "High CPU Alert",
        "description": "Alert when VM CPU is high",
        "isEnabled": true,
        "condition": {
          "dataSource": {
            "resourceUri": "/subscriptions/.../virtualMachines/myapp-vm",
            "metricName": "Percentage CPU"    // What to measure
          },
          "operator": "GreaterThan",          // Comparison operator
          "threshold": 80,                    // Trigger when CPU > 80%
          "windowSize": "PT5M",               // Time window (5 minutes)
          "timeAggregation": "Average"        // How to aggregate data
        },
        "actions": [
          {
            "sendToServiceOwners": true,      // Email the service owners
            "customEmails": ["admin@company.com"],
            "webhookProperties": {}
          }
        ]
      }
    }
    ```

27. **Application Insights monitors your applications.** It tracks performance, errors, and user behavior in your web apps and APIs.
    ```javascript
    // Add to your web application
    const appInsights = require('applicationinsights');
    appInsights.setup('your-instrumentation-key')    // Unique key for your app
        .setAutoDependencyCorrelation(true)          // Track dependencies automatically
        .setAutoCollectRequests(true)                // Track HTTP requests
        .setAutoCollectPerformance(true)             // Track performance counters
        .setAutoCollectExceptions(true)              // Track exceptions
        .start();
    
    // Custom tracking
    const client = appInsights.defaultClient;
    client.trackEvent({name: "UserLogin", properties: {userId: "123"}});    // Track custom events
    client.trackMetric({name: "ProcessingTime", value: 150});               // Track custom metrics
    ```

## Key Concepts

28. **Availability Zones are like different buildings in the same city.** Each zone has its own power and network. If one fails, others keep running. Similar to AWS AZs.

29. **Regions are like different countries.** Each region is completely separate with its own set of availability zones. Choose regions close to your users for better performance.

30. **Resource IDs are like postal addresses for Azure resources.** Every resource has a unique ID that looks like: `/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/{provider}/{resource-type}/{resource-name}`

31. **Azure Resource Manager (ARM) templates are like blueprints for your infrastructure.** Similar to AWS CloudFormation - define your resources in JSON and Azure creates them.
    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "vmName": {
          "type": "string",                 // Parameter type
          "defaultValue": "myVM",           // Default value if none provided
          "metadata": {
            "description": "Name of the virtual machine"
          }
        }
      },
      "resources": [
        {
          "type": "Microsoft.Compute/virtualMachines",
          "apiVersion": "2021-03-01",
          "name": "[parameters('vmName')]",  // Use parameter value
          "location": "[resourceGroup().location]",    // Use resource group's location
          "properties": {
            "hardwareProfile": {
              "vmSize": "Standard_B2s"      // VM size
            },
            "osProfile": {
              "computerName": "[parameters('vmName')]",
              "adminUsername": "azureuser"
            }
          }
        }
      ]
    }
    ```

## Security Best Practices

32. **Use Managed Identities instead of storing credentials.** Let Azure handle authentication automatically for your VMs and App Services.

33. **Enable Azure Security Center for security recommendations.** It scans your resources and tells you how to improve security.

34. **Use Key Vault to store secrets, keys, and certificates.** Never put passwords or API keys in your code or configuration files.
    ```bash
    # Create a Key Vault
    az keyvault create \
      --name myapp-keyvault \
      --resource-group myapp-rg \
      --location eastus
    
    # Store a secret
    az keyvault secret set \
      --vault-name myapp-keyvault \
      --name "DatabasePassword" \
      --value "SuperSecretPassword123!"
    
    # Retrieve a secret
    az keyvault secret show \
      --vault-name myapp-keyvault \
      --name "DatabasePassword" \
      --query value -o tsv
    ```

35. **Enable encryption everywhere.** Use encryption at rest for storage accounts, databases, and managed disks. Use HTTPS/TLS for all network traffic.

36. **Use Network Security Groups and Application Security Groups** to control network traffic. Follow the principle of least privilege - only allow the minimum required access.

37. **Monitor everything with Azure Monitor and Security Center.** Enable diagnostic logging for all resources and set up alerts for suspicious activities. 