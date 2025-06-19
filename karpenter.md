# Karpenter

Karpenter is a Kubernetes cluster autoscaler that automatically provisions and manages worker nodes. Instead of using fixed node groups like traditional autoscalers, Karpenter creates nodes on-demand based on what your pods actually need.

## Why use Karpenter

Traditional cluster autoscaling works with pre-defined node groups. You have to guess what instance types and sizes you'll need. Karpenter is smarter - it looks at your pending pods and creates the perfect nodes for them automatically.

**Benefits:**
- **Faster scaling**: Provisions nodes in seconds, not minutes
- **Cost optimization**: Picks the cheapest instance types that fit your workload
- **Less waste**: Creates exactly what you need, when you need it
- **No node group management**: You don't maintain fixed groups of instances

## How to install Karpenter

### Prerequisites
You need:
- A Kubernetes cluster (EKS recommended for AWS)
- `kubectl` configured to talk to your cluster  
- `helm` package manager installed

### Installation steps

1. **Add the Karpenter Helm repository:**
   ```bash
   helm repo add karpenter oci://public.ecr.aws/karpenter/karpenter
   helm repo update
   ```

2. **Install Karpenter:**
   ```bash
   helm install karpenter karpenter/karpenter \
     --version v0.32.0 \
     --namespace karpenter \
     --create-namespace \
     --set serviceAccount.annotations."eks\.amazonaws\.com/role-arn"=${KARPENTER_IAM_ROLE_ARN} \
     --set settings.aws.clusterName=${CLUSTER_NAME} \
     --set settings.aws.defaultInstanceProfile=KarpenterNodeInstanceProfile \
     --wait
   ```

   - `serviceAccount.annotations`: Connects Karpenter to AWS IAM for permissions
   - `settings.aws.clusterName`: Your EKS cluster name  
   - `settings.aws.defaultInstanceProfile`: IAM instance profile for EC2 nodes
   - `--wait`: Waits for installation to complete before returning

3. **Verify installation:**
   ```bash
   kubectl get pods -n karpenter
   ```
   You should see the Karpenter controller pod running.

## Basic Karpenter configuration

Karpenter uses two main resources to work:
- **NodePool**: Defines what kinds of nodes Karpenter can create
- **EC2NodeClass**: Defines how to configure those nodes (AWS-specific)

### Creating your first NodePool

A `NodePool` tells Karpenter what instance types, sizes, and constraints to use when creating nodes.

```yaml
apiVersion: karpenter.sh/v1beta1
kind: NodePool
metadata:
  name: default
spec:
  template:
    metadata:
      labels:
        karpenter.kwok.x-k8s.io/node: "true"
    spec:
      requirements:
        - key: kubernetes.io/arch
          operator: In
          values: ["amd64"]
        - key: kubernetes.io/os
          operator: In
          values: ["linux"]
        - key: karpenter.sh/capacity-type
          operator: In
          values: ["spot", "on-demand"]
        - key: node.kubernetes.io/instance-type
          operator: In
          values: ["m5.large", "m5.xlarge", "c5.large", "c5.xlarge"]
      nodeClassRef:
        apiVersion: karpenter.k8s.aws/v1beta1
        kind: EC2NodeClass
        name: default
      taints:
        - key: example.com/special-taint
          value: "true"
          effect: NoSchedule
  limits:
    cpu: 1000
  disruption:
    consolidationPolicy: WhenIdle
    consolidateAfter: 30s
```

**Key fields explained:**
- `requirements`: Constraints for node selection. Karpenter will only use instance types that match ALL requirements
- `kubernetes.io/arch`: CPU architecture (amd64, arm64)
- `kubernetes.io/os`: Operating system (linux, windows)
- `karpenter.sh/capacity-type`: Instance pricing model (spot = cheaper but can be interrupted, on-demand = stable but more expensive)
- `node.kubernetes.io/instance-type`: Specific EC2 instance types allowed
- `nodeClassRef`: Links to an EC2NodeClass that defines AWS-specific settings
- `taints`: Node taints that prevent certain pods from scheduling unless they have matching tolerations
- `limits`: Maximum resources this NodePool can provision
- `disruption.consolidationPolicy`: When to consolidate nodes (WhenIdle = only when nodes have no workload)
- `disruption.consolidateAfter`: How long to wait before consolidating idle nodes

### Creating an EC2NodeClass

An `EC2NodeClass` defines AWS-specific configuration for your nodes:

```yaml
apiVersion: karpenter.k8s.aws/v1beta1
kind: EC2NodeClass
metadata:
  name: default
spec:
  amiFamily: AL2 # Amazon Linux 2
  subnetSelectorTerms:
    - tags:
        karpenter.sh/discovery: "my-cluster"
  securityGroupSelectorTerms:
    - tags:
        karpenter.sh/discovery: "my-cluster"
  instanceStorePolicy: "RAID0"
  userData: |
    #!/bin/bash
    /etc/eks/bootstrap.sh my-cluster
    echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
  blockDeviceMappings:
    - deviceName: /dev/xvda
      ebs:
        volumeSize: 100Gi
        volumeType: gp3
        encrypted: true
```

**Key fields explained:**
- `amiFamily`: Operating system family (AL2 = Amazon Linux 2, Ubuntu, Bottlerocket)
- `subnetSelectorTerms`: Which VPC subnets to use (selected by tags)
- `securityGroupSelectorTerms`: Which security groups to attach (selected by tags) 
- `instanceStorePolicy`: How to configure instance storage (RAID0 = combine all disks)
- `userData`: Shell script that runs when node starts up
- `blockDeviceMappings`: EBS volume configuration for the root disk

## How Karpenter decides what nodes to create

When you create a pod that can't be scheduled, Karpenter follows this process:

1. **Analyze pod requirements**: Looks at CPU, memory, and other resource requests
2. **Check NodePool constraints**: Only considers instance types allowed by your NodePools
3. **Find compatible instances**: Matches pod requirements with available EC2 instance types
4. **Optimize for cost**: Picks the cheapest option that satisfies the requirements
5. **Create the node**: Launches EC2 instance and joins it to your cluster

**Example scenario:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        cpu: 500m      # Needs 0.5 CPU cores
        memory: 1000Mi # Needs 1GB RAM
  nodeSelector:
    kubernetes.io/arch: amd64
```

For this pod, Karpenter will:
- Look for amd64 instances (because of nodeSelector)
- Find instances with at least 0.5 CPU and 1GB RAM available
- Pick the cheapest option from your allowed instance types
- Create that node and schedule the pod on it

## Pod scheduling with Karpenter

### Node selectors
Use `nodeSelector` to force pods onto specific types of nodes:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-workload
spec:
  nodeSelector:
    node.kubernetes.io/instance-type: "p3.2xlarge"  # Only GPU instances
    kubernetes.io/arch: amd64
  containers:
  - name: ml-training
    image: tensorflow/tensorflow:latest-gpu
```

### Node affinity
Use `nodeAffinity` for more complex node selection rules:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: compute-heavy
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: node.kubernetes.io/instance-type
            operator: In
            values: ["c5.2xlarge", "c5.4xlarge", "c5.9xlarge"]  # CPU-optimized instances
          - key: karpenter.sh/capacity-type
            operator: In
            values: ["on-demand"]  # Only on-demand, no spot instances
  containers:
  - name: processor
    image: my-cpu-intensive-app
```

**Node affinity operators:**
- `In`: Value must be in the list
- `NotIn`: Value must NOT be in the list  
- `Exists`: Key must exist (ignore value)
- `DoesNotExist`: Key must not exist
- `Gt`: Numeric value must be greater than
- `Lt`: Numeric value must be less than

### Tolerations and taints
Taints prevent pods from scheduling on nodes unless pods have matching tolerations.

**Apply taint to NodePool:**
```yaml
apiVersion: karpenter.sh/v1beta1
kind: NodePool
metadata:
  name: gpu-nodes
spec:
  template:
    spec:
      taints:
        - key: nvidia.com/gpu
          value: "true"
          effect: NoSchedule  # Pods without toleration can't schedule here
      requirements:
        - key: node.kubernetes.io/instance-type
          operator: In
          values: ["p3.2xlarge", "p3.8xlarge"]
```

**Add toleration to pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  tolerations:
  - key: nvidia.com/gpu
    operator: Equal
    value: "true"
    effect: NoSchedule  # This pod can tolerate the taint
  containers:
  - name: cuda-app
    image: nvidia/cuda:11.0-runtime-ubuntu20.04
```

**Taint effects:**
- `NoSchedule`: New pods can't be scheduled unless they tolerate the taint
- `PreferNoSchedule`: Kubernetes avoids scheduling pods here but will if needed
- `NoExecute`: Existing pods without toleration get evicted

## Cost optimization with Karpenter

### Spot instances
Spot instances are up to 90% cheaper but can be interrupted by AWS. Great for fault-tolerant workloads:

```yaml
apiVersion: karpenter.sh/v1beta1
kind: NodePool
metadata:
  name: spot-nodes
spec:
  template:
    spec:
      requirements:
        - key: karpenter.sh/capacity-type
          operator: In
          values: ["spot"]  # Only use spot instances
        - key: node.kubernetes.io/instance-type
          operator: In
          values: ["m5.large", "m5.xlarge", "m4.large", "m4.xlarge"]  # Multiple types for better availability
  disruption:
    consolidationPolicy: WhenEmpty  # Consolidate more aggressively to save money
```

### Mixed instance types
Allow multiple instance types so Karpenter can pick the cheapest available:

```yaml
spec:
  template:
    spec:
      requirements:
        - key: node.kubernetes.io/instance-type
          operator: In
          values: 
            - "m5.large"
            - "m5.xlarge" 
            - "m4.large"
            - "m4.xlarge"
            - "c5.large"
            - "c5.xlarge"  # Many options = better pricing
        - key: karpenter.sh/capacity-type
          operator: In
          values: ["spot", "on-demand"]  # Allow both pricing models
```

### Consolidation
Karpenter can consolidate workloads onto fewer nodes to save money:

```yaml
spec:
  disruption:
    consolidationPolicy: WhenIdle      # Move pods from idle nodes to other nodes
    consolidateAfter: 30s              # Wait 30 seconds before consolidating
    expireAfter: 2160h                 # Replace nodes after 90 days (for security updates)
```

**Consolidation policies:**
- `WhenIdle`: Only consolidate nodes with no workload
- `WhenEmpty`: More aggressive - consolidate nodes even if it requires moving pods
- `WhenUnderutilized`: Consolidate based on resource utilization thresholds

## Troubleshooting common issues

### Pods stuck in Pending state
If pods aren't getting scheduled, check:

1. **View pending pods:**
   ```bash
   kubectl get pods --field-selector=status.phase=Pending
   ```

2. **Check pod events:**
   ```bash
   kubectl describe pod <pod-name>
   ```
   Look for scheduling errors in the Events section.

3. **Check Karpenter logs:**
   ```bash
   kubectl logs -n karpenter -l app.kubernetes.io/name=karpenter
   ```

4. **Common causes:**
   - Pod resource requests exceed what any allowed instance type can provide
   - NodePool requirements are too restrictive  
   - No NodePool matches the pod's nodeSelector or nodeAffinity
   - AWS service limits reached (EC2 instance limits)

### Nodes not being created
Check these things:

1. **Verify NodePool exists and is valid:**
   ```bash
   kubectl get nodepool
   kubectl describe nodepool <nodepool-name>
   ```

2. **Check EC2NodeClass configuration:**
   ```bash
   kubectl get ec2nodeclass  
   kubectl describe ec2nodeclass <nodeclass-name>
   ```

3. **Verify AWS permissions:** Karpenter needs IAM permissions to create EC2 instances, security groups, etc.

4. **Check AWS service quotas:** You might have hit EC2 instance limits in your AWS account.

### High costs
If your AWS bill is higher than expected:

1. **Check instance types being created:**
   ```bash
   kubectl get nodes --show-labels
   ```
   Look for expensive instance types.

2. **Enable more spot instances:**
   ```yaml
   - key: karpenter.sh/capacity-type
     operator: In  
     values: ["spot"]  # Remove "on-demand" if possible
   ```

3. **Add more instance type options:**
   More choices = better chance of finding cheaper instances.

4. **Tune consolidation settings:**
   ```yaml
   disruption:
     consolidationPolicy: WhenEmpty  # More aggressive consolidation
     consolidateAfter: 15s           # Consolidate faster
   ```

## Advanced Karpenter patterns

### Multi-zone deployment
Spread nodes across availability zones for high availability:

```yaml
apiVersion: karpenter.sh/v1beta1
kind: NodePool
metadata:
  name: multi-zone
spec:
  template:
    spec:
      requirements:
        - key: topology.kubernetes.io/zone
          operator: In
          values: ["us-west-2a", "us-west-2b", "us-west-2c"]  # Multiple AZs
```

### GPU workloads  
Configure nodes specifically for GPU workloads:

```yaml
apiVersion: karpenter.sh/v1beta1
kind: NodePool
metadata:
  name: gpu-nodes
spec:
  template:
    metadata:
      labels:
        node-type: gpu
    spec:
      taints:
        - key: nvidia.com/gpu
          value: "true"
          effect: NoSchedule
      requirements:
        - key: node.kubernetes.io/instance-type
          operator: In
          values: ["p3.2xlarge", "p3.8xlarge", "p3.16xlarge"]
        - key: karpenter.sh/capacity-type
          operator: In
          values: ["on-demand"]  # GPU instances on spot can be expensive when interrupted
  nodeClassRef:
    name: gpu-nodeclass
---
apiVersion: karpenter.k8s.aws/v1beta1
kind: EC2NodeClass
metadata:
  name: gpu-nodeclass
spec:
  amiFamily: AL2
  userData: |
    #!/bin/bash
    /etc/eks/bootstrap.sh my-cluster
    # Install NVIDIA drivers
    sudo yum install -y nvidia-driver nvidia-docker2
    sudo systemctl restart docker
```

### Batch workloads
Optimize for batch processing jobs that can tolerate interruptions:

```yaml
apiVersion: karpenter.sh/v1beta1
kind: NodePool  
metadata:
  name: batch-processing
spec:
  template:
    metadata:
      labels:
        workload-type: batch
    spec:
      taints:
        - key: batch-only
          value: "true"
          effect: NoSchedule
      requirements:
        - key: karpenter.sh/capacity-type
          operator: In
          values: ["spot"]  # Cheaper for batch jobs
        - key: node.kubernetes.io/instance-type
          operator: In
          values: ["c5.large", "c5.xlarge", "c5.2xlarge", "c5.4xlarge"]  # CPU-optimized
  disruption:
    consolidationPolicy: WhenEmpty
    consolidateAfter: 10s  # Quick consolidation for batch workloads
```

## Best practices

1. **Use multiple NodePools**: Create different pools for different workload types (web apps, batch jobs, GPU workloads).

2. **Allow multiple instance types**: More options = better availability and pricing.

3. **Mix spot and on-demand**: Use spot for fault-tolerant workloads, on-demand for critical services.

4. **Set appropriate resource requests**: Accurate requests help Karpenter pick the right instance sizes.

5. **Use taints for specialized nodes**: Keep expensive nodes (GPU, large instances) for workloads that actually need them.

6. **Monitor and tune consolidation**: Start conservative, then make it more aggressive as you get comfortable.

7. **Set resource limits**: Prevent runaway costs with NodePool limits:
   ```yaml
   limits:
     cpu: 1000      # Max CPU cores across all nodes in this pool
     memory: 1000Gi # Max memory across all nodes in this pool
   ```

8. **Use topology spread constraints**: Spread workloads across zones and nodes for resilience:
   ```yaml
   apiVersion: v1
   kind: Pod
   spec:
     topologySpreadConstraints:
     - maxSkew: 1
       topologyKey: topology.kubernetes.io/zone
       whenUnsatisfiable: DoNotSchedule
       labelSelector:
         matchLabels:
           app: my-app
   ``` 