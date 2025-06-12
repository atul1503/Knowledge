Notes:

1. **Kubernetes is like a smart manager for your containers.** It runs your apps, restarts them if they crash, and scales them up when you need more power.

2. **Pod is the smallest thing you can run.** It's like a wrapper around one or more containers. Usually just put one container per pod.
   ```yaml
   apiVersion: v1        # Which version of Kubernetes API to use
   kind: Pod             # What type of thing you're creating
   metadata:             # Info about your pod (name, labels, etc.)
     name: my-app        # What to call this pod
   spec:                 # The actual configuration of what you want
     containers:         # List of containers to run in this pod
     - name: app         # Name for this container
       image: nginx:latest    # Which Docker image to use
       ports:            # Which ports this container uses
       - containerPort: 80    # Port number the app listens on
   ```

3. **Deployment manages your pods.** It makes sure you always have the right number running and handles updates smoothly.
   ```yaml
   apiVersion: apps/v1   # Deployments use apps/v1 API version
   kind: Deployment
   metadata:
     name: my-app
   spec:
     replicas: 3         # How many copies of your pod to run
     selector:           # How to find pods that belong to this deployment
       matchLabels:      # Match pods that have these exact labels
         app: my-app
     template:           # Template for creating new pods
       metadata:
         labels:         # Labels to put on the pods we create
           app: my-app
       spec:             # Same as pod spec - what containers to run
         containers:
         - name: app
           image: nginx:latest
   ```

4. **Service exposes your pods to the network.** Think of it as a load balancer that finds your pods and sends traffic to them.
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-service
   spec:
     selector:           # Which pods this service should send traffic to
       app: my-app       # Send traffic to pods with this label
     ports:             # Which ports to expose
     - port: 80         # Port that clients connect to
       targetPort: 80   # Port on the pod to send traffic to
     type: LoadBalancer # How to expose this service (LoadBalancer, ClusterIP, NodePort)
   ```

5. **Labels are like tags you stick on everything.** Services use them to find pods, deployments use them to manage pods. Keep them simple.
   ```yaml
   metadata:
     labels:             # Key-value pairs for organizing and selecting resources
       app: my-app       # What application this belongs to
       version: v1       # Which version of the app
       environment: prod # Which environment (dev, staging, prod)
   ```

6. **Namespaces are like folders for organizing stuff.** Use them to separate different environments or teams.
   ```bash
   kubectl create namespace dev
   kubectl get pods -n dev
   kubectl apply -f app.yaml -n dev
   ```

7. **ConfigMaps store configuration data.** Put your app settings here instead of hardcoding them.
   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: app-config
   data:                 # The actual config data as key-value pairs
     database_url: "postgres://localhost:5432/mydb"
     debug_mode: "true"
   ```
   
   Use it in your pod:
   ```yaml
   containers:
   - name: app
     image: my-app:latest
     env:                # Environment variables for the container
     - name: DATABASE_URL          # Name of the env variable
       valueFrom:                  # Get value from somewhere else
         configMapKeyRef:          # Get it from a ConfigMap
           name: app-config        # Which ConfigMap to use
           key: database_url       # Which key from that ConfigMap
   ```

8. **Secrets are like ConfigMaps but for sensitive stuff.** Passwords, API keys, certificates go here.
   ```bash
   kubectl create secret generic db-secret \
     --from-literal=username=admin \
     --from-literal=password=supersecret
   ```
   
   ```yaml
   containers:
   - name: app
     image: my-app:latest
     env:
     - name: DB_PASSWORD
       valueFrom:
         secretKeyRef:     # Get value from a Secret (like ConfigMap but encrypted)
           name: db-secret # Which Secret to use
           key: password   # Which key from that Secret
   ```

9. **Ingress is like a smart router.** It takes HTTP requests from outside and routes them to the right service based on the URL.
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: my-ingress
   spec:
     rules:               # List of routing rules
     - host: myapp.com    # Which domain name this rule applies to
       http:              # HTTP-specific routing
         paths:           # List of URL paths to match
         - path: /        # Which URL path to match (/ means everything)
           pathType: Prefix    # How to match the path (Prefix, Exact, etc.)
           backend:            # Where to send matching requests
             service:          # Send to a service
               name: my-service     # Which service to send to
               port:               # Which port on that service
                 number: 80        # Port number
   ```

10. **Volumes let containers store data that survives restarts.** Without them, everything gets wiped when a pod dies.
    ```yaml
    spec:
      containers:
      - name: app
        image: my-app:latest
        volumeMounts:         # Where to attach volumes inside the container
        - name: data-storage  # Which volume to mount (matches volumes list below)
          mountPath: /app/data     # Where inside container to mount it
      volumes:                # List of volumes available to this pod
      - name: data-storage    # Name for this volume
        persistentVolumeClaim:     # Use a persistent volume claim
          claimName: my-pvc        # Which PVC to use
    ```

11. **Resource limits prevent one app from eating all your CPU and memory.** Always set these in production.
    ```yaml
    containers:
    - name: app
      image: my-app:latest
      resources:            # How much CPU and memory this container needs/can use
        requests:           # Minimum resources needed (used for scheduling)
          memory: "64Mi"    # 64 megabytes of RAM minimum
          cpu: "250m"       # 250 millicores (0.25 CPU cores) minimum
        limits:             # Maximum resources allowed
          memory: "128Mi"   # Can't use more than 128MB RAM
          cpu: "500m"       # Can't use more than 0.5 CPU cores
    ```

12. **Health checks tell Kubernetes if your app is working.** Use `livenessProbe` to restart broken pods and `readinessProbe` to know when they're ready for traffic.
    ```yaml
    containers:
    - name: app
      image: my-app:latest
      livenessProbe:        # Check if container is still alive (restart if fails)
        httpGet:            # Make an HTTP request to check health
          path: /health     # Which URL path to check
          port: 8080        # Which port to check on
        initialDelaySeconds: 30    # Wait 30 seconds before first check
        periodSeconds: 10          # Check every 10 seconds after that
      readinessProbe:       # Check if container is ready for traffic
        httpGet:
          path: /ready      # Different endpoint for readiness
          port: 8080
        initialDelaySeconds: 5     # Start checking after 5 seconds
        periodSeconds: 5           # Check every 5 seconds
    ```

13. **HorizontalPodAutoscaler (HPA) automatically scales your app** based on CPU usage or other metrics.
    ```yaml
    apiVersion: autoscaling/v2
    kind: HorizontalPodAutoscaler
    metadata:
      name: my-app-hpa
    spec:
      scaleTargetRef:       # What to scale
        apiVersion: apps/v1 # API version of the target
        kind: Deployment    # Type of resource to scale
        name: my-app        # Name of the deployment to scale
      minReplicas: 2        # Never scale below this number
      maxReplicas: 10       # Never scale above this number
      metrics:              # What metrics to use for scaling decisions
      - type: Resource      # Use built-in resource metrics
        resource:
          name: cpu         # Scale based on CPU usage
          target:
            type: Utilization        # Use percentage utilization
            averageUtilization: 70   # Scale up when average CPU > 70%
    ```

14. **Jobs run tasks that need to complete and then stop.** Like batch processing or database migrations.
    ```yaml
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: data-migration
    spec:
      template:             # Template for the pod that will run the job
        spec:
          containers:
          - name: migrator
            image: my-migrator:latest
            command: ["python", "migrate.py"]    # What command to run
          restartPolicy: Never    # Don't restart if it fails (OnFailure is another option)
    ```

15. **CronJobs run jobs on a schedule.** Like cron but for Kubernetes.
    ```yaml
    apiVersion: batch/v1
    kind: CronJob
    metadata:
      name: backup-job
    spec:
      schedule: "0 2 * * *"  # When to run (cron format: min hour day month weekday)
      jobTemplate:           # Template for creating jobs
        spec:                # Job specification
          template:          # Pod template for the job
            spec:
              containers:
              - name: backup
                image: backup-tool:latest
                command: ["./backup.sh"]
              restartPolicy: OnFailure    # Restart if it fails, but not if it succeeds
    ```

16. **DaemonSet runs one pod on every node.** Good for logging agents, monitoring, or node-level services.
    ```yaml
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: log-collector
    spec:
      selector:
        matchLabels:
          app: log-collector
      template:
        metadata:
          labels:
            app: log-collector
        spec:
          containers:
          - name: logger
            image: fluentd:latest
    ```

17. **StatefulSet is for apps that need stable network identities or persistent storage.** Like databases.
    ```yaml
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: postgres
    spec:
      serviceName: postgres
      replicas: 1
      selector:
        matchLabels:
          app: postgres
      template:
        metadata:
          labels:
            app: postgres
        spec:
          containers:
          - name: postgres
            image: postgres:13
            env:
            - name: POSTGRES_PASSWORD
              value: "password"
            volumeMounts:
            - name: postgres-storage
              mountPath: /var/lib/postgresql/data
      volumeClaimTemplates:
      - metadata:
          name: postgres-storage
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 10Gi
    ```

18. **Use kubectl to talk to your cluster.** These are the commands you'll use every day:
    ```bash
    # See what's running
    kubectl get pods
    kubectl get services
    kubectl get deployments
    
    # Get detailed info
    kubectl describe pod my-pod
    kubectl logs my-pod
    kubectl logs -f my-pod  # Follow logs
    
    # Apply changes
    kubectl apply -f my-app.yaml
    kubectl delete -f my-app.yaml
    
    # Debug stuff
    kubectl exec -it my-pod -- /bin/bash
    kubectl port-forward my-pod 8080:80
    
    # Scale things
    kubectl scale deployment my-app --replicas=5
    ```

19. **Rolling updates happen automatically** when you change your deployment. Kubernetes gradually replaces old pods with new ones.
    ```bash
    # Update image
    kubectl set image deployment/my-app app=my-app:v2
    
    # Check rollout status
    kubectl rollout status deployment/my-app
    
    # Rollback if something breaks
    kubectl rollout undo deployment/my-app
    ```

20. **Node affinity lets you control which nodes your pods run on.** Useful for putting GPU workloads on GPU nodes.
    ```yaml
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-type
                operator: In
                values:
                - gpu-node
    ```

21. **Taints and tolerations work together.** Nodes have taints (like "no regular pods allowed"), pods have tolerations (like "I can handle that taint").
    ```bash
    # Taint a node
    kubectl taint nodes node1 special=true:NoSchedule
    ```
    
    ```yaml
    # Pod that tolerates the taint
    spec:
      tolerations:
      - key: "special"
        operator: "Equal"
        value: "true"
        effect: "NoSchedule"
    ```

22. **RBAC controls who can do what in your cluster.** Create roles and bind them to users or service accounts.
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: pod-reader
    rules:
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get", "watch", "list"]
    ```

23. **NetworkPolicies control traffic between pods.** By default, all pods can talk to each other. Use these to lock things down.
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: deny-all
    spec:
      podSelector: {}
      policyTypes:
      - Ingress
      - Egress
    ```

24. **Always use specific image tags in production.** Don't use `latest` because you won't know what version is actually running.
    ```yaml
    # Good
    image: nginx:1.21.6
    
    # Bad for production
    image: nginx:latest
    ```

25. **Kubernetes has three main parts:**
    - **Control Plane**: The brain that makes decisions (API server, scheduler, controller manager)
    - **Nodes**: The workers that run your pods (kubelet, kube-proxy, container runtime)
    - **etcd**: The database that stores all cluster state

26. **Common gotchas:**
    - Pods in different namespaces can't find each other by service name alone. Use `service-name.namespace-name.svc.cluster.local`
    - Resource requests are what the scheduler uses to place pods. Limits are what actually gets enforced.
    - If you don't set resource requests, your pods might get scheduled on overloaded nodes.
    - Always set `imagePullPolicy: Always` if you're using mutable tags like `latest`.
    - Services only route to pods that are ready (pass readiness checks).

## How to expose your application to the outside world

27. **NodePort is the simplest way but not great for production.** It opens a specific port on every node in your cluster. Anyone can access your app by hitting any node's IP address plus that port.
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-app-nodeport
    spec:
      type: NodePort
      selector:
        app: my-app
      ports:
      - port: 80          # Internal cluster port
        targetPort: 8080  # Port on your pod
        nodePort: 30080   # Port exposed on each node (30000-32767 range)
    ```
    ```bash
    # Access your app at any node's IP
    curl http://node-ip:30080
    # If on AWS, use EC2 instance public IP
    curl http://ec2-instance-ip:30080
    ```
    **Problems with NodePort:** Users need to remember weird ports, no load balancing between nodes, security groups need to allow these ports.

28. **LoadBalancer service creates a cloud load balancer automatically.** This is the easiest way for production if you're on a cloud provider like AWS.
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-app-loadbalancer
    spec:
      type: LoadBalancer
      selector:
        app: my-app
      ports:
      - port: 80          # Port on the load balancer
        targetPort: 8080  # Port on your pod
    ```
    
    **On AWS EKS:** Creates a Classic Load Balancer (CLB) automatically. You get a public DNS name like `a1b2c3d4e5f6g7h8-1234567890.us-west-2.elb.amazonaws.com`
    
    **On-premises:** LoadBalancer type doesn't work unless you have something like MetalLB installed. It will stay in "Pending" state forever.
    
    ```bash
    # Check if your LoadBalancer got an external IP
    kubectl get svc my-app-loadbalancer
    # On AWS, EXTERNAL-IP shows the ELB DNS name
    ```

29. **Ingress is the smart way for HTTP/HTTPS traffic.** It acts like a reverse proxy that can route different domains/paths to different services. Much better than LoadBalancer for web apps.
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: my-app-ingress
      annotations:
        kubernetes.io/ingress.class: "nginx"    # Which ingress controller to use
    spec:
      rules:
      - host: myapp.example.com     # Domain name for your app
        http:
          paths:
          - path: /                 # Route all traffic under this domain
            pathType: Prefix        # How to match paths (Prefix, Exact)
            backend:
              service:
                name: my-app-service    # Which service to route to
                port:
                  number: 80           # Which port on that service
      - host: api.example.com       # Different domain for API
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-api-service
                port:
                  number: 80
    ```
    
    **You need an Ingress Controller** to make Ingress work. Popular options:
    - **NGINX Ingress Controller** (works everywhere)
    - **AWS Load Balancer Controller** (AWS-specific, creates ALB/NLB)
    - **Traefik** (good for simple setups)

30. **NGINX Ingress Controller is the most common choice.** Works on-premises and in cloud. Creates a LoadBalancer service that routes to your apps based on domains/paths.
    ```bash
    # Install NGINX Ingress Controller
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml
    
    # On AWS, this creates a Classic Load Balancer
    # On-premises, you need to expose it somehow (NodePort, external LB, etc.)
    ```
    
    **HTTPS/TLS with cert-manager:**
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: my-app-https
      annotations:
        kubernetes.io/ingress.class: "nginx"
        cert-manager.io/cluster-issuer: "letsencrypt-prod"    # Automatic SSL certificates
    spec:
      tls:                          # Enable HTTPS
      - hosts:
        - myapp.example.com
        secretName: myapp-tls       # Where to store the SSL certificate
      rules:
      - host: myapp.example.com
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-service
                port:
                  number: 80
    ```

31. **AWS Load Balancer Controller is better if you're on AWS.** It creates Application Load Balancers (ALB) or Network Load Balancers (NLB) instead of the old Classic Load Balancers.
    ```bash
    # Install AWS Load Balancer Controller (requires IRSA setup)
    helm repo add eks https://aws.github.io/eks-charts
    helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
      -n kube-system \
      --set clusterName=my-cluster \
      --set serviceAccount.create=false \
      --set serviceAccount.name=aws-load-balancer-controller
    ```
    
    **Create an ALB with Ingress:**
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: my-app-alb
      annotations:
        kubernetes.io/ingress.class: alb
        alb.ingress.kubernetes.io/scheme: internet-facing    # Public ALB
        alb.ingress.kubernetes.io/target-type: ip           # Route to pod IPs directly
        alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-west-2:123456789:certificate/12345678-1234-1234-1234-123456789012    # SSL cert from ACM
    spec:
      tls:
      - hosts:
        - myapp.example.com
      rules:
      - host: myapp.example.com
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-service
                port:
                  number: 80
    ```
    
    **Create an NLB with LoadBalancer service:**
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-app-nlb
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-type: "nlb"    # Create NLB instead of CLB
        service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"    # Public NLB
    spec:
      type: LoadBalancer
      selector:
        app: my-app
      ports:
      - port: 80
        targetPort: 8080
    ```

32. **Choose the right approach based on your setup:**

    **On-premises cluster:**
    - Use **NodePort** for development/testing
    - Use **Ingress with NGINX controller** + external load balancer for production
    - Use **MetalLB** if you want LoadBalancer services to work
    
    **AWS EKS cluster:**
    - Use **ALB Ingress** for web applications (HTTP/HTTPS)
    - Use **NLB LoadBalancer** for non-HTTP traffic or when you need static IPs
    - Use **CLB LoadBalancer** (default) for simple cases
    
    **Cost considerations on AWS:**
    - **NodePort:** Free (just uses EC2 instances)
    - **Classic Load Balancer:** ~$18/month + data transfer
    - **Application Load Balancer:** ~$16/month + LCU usage + data transfer
    - **Network Load Balancer:** ~$16/month + NLCU usage + data transfer

33. **Multiple apps on one load balancer with Ingress.** Save money by routing multiple applications through one ALB/NLB instead of creating separate LoadBalancer services.
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: multi-app-ingress
      annotations:
        kubernetes.io/ingress.class: alb
        alb.ingress.kubernetes.io/scheme: internet-facing
    spec:
      rules:
      - host: app1.example.com      # First app
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app1-service
                port:
                  number: 80
      - host: app2.example.com      # Second app
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app2-service
                port:
                  number: 80
      - host: api.example.com       # API on same ALB
        http:
          paths:
          - path: /v1               # API v1
            pathType: Prefix
            backend:
              service:
                name: api-v1-service
                port:
                  number: 80
          - path: /v2               # API v2
            pathType: Prefix
            backend:
              service:
                name: api-v2-service
                port:
                  number: 80
    ```

34. **Security considerations for external access:**
    ```yaml
    # Restrict ALB to specific IP ranges
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: restricted-app
      annotations:
        kubernetes.io/ingress.class: alb
        alb.ingress.kubernetes.io/scheme: internet-facing
        alb.ingress.kubernetes.io/inbound-cidrs: "10.0.0.0/8,192.168.0.0/16"    # Only allow private networks
    spec:
      rules:
      - host: internal.example.com
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: internal-service
                port:
                  number: 80
    ```
    
    **Use AWS Security Groups:**
    ```yaml
    # Control which security groups can access your NLB
    apiVersion: v1
    kind: Service
    metadata:
      name: secure-app
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-type: nlb
        service.beta.kubernetes.io/aws-load-balancer-security-groups: "sg-12345678"    # Specific security group
    spec:
      type: LoadBalancer
      selector:
        app: secure-app
      ports:
      - port: 443
        targetPort: 8080
    ```

35. **Quick decision guide for exposing apps:**

    **Use NodePort when:**
    - Testing/development
    - On-premises with existing external load balancer
    - Need non-HTTP protocols and don't want cloud load balancer costs
    
    **Use LoadBalancer service when:**
    - Simple app, just need basic load balancing
    - Non-HTTP traffic (TCP/UDP)
    - Don't need fancy routing rules
    
    **Use Ingress when:**
    - HTTP/HTTPS traffic
    - Multiple apps/domains
    - Need path-based routing
    - Want SSL termination
    - Want to save money (one load balancer for multiple apps)
    
    **Example commands to check what you have:**
    ```bash
    # See all services and their external access
    kubectl get svc -A
    
    # See all ingresses
    kubectl get ingress -A
    
    # Check ingress controller pods
    kubectl get pods -n ingress-nginx
    
    # Check AWS Load Balancer Controller
    kubectl get pods -n kube-system | grep aws-load-balancer
    ``` 