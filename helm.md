Notes:

1. **Helm is like a package manager for Kubernetes.** Think of it like `npm` for Node.js or `pip` for Python, but for Kubernetes applications. Instead of writing lots of YAML files by hand, you install pre-made packages called "charts."

2. **Chart is a bundle of Kubernetes files.** It contains all the YAML files (deployments, services, etc.) needed to run an application. Charts are like templates that you can customize for different environments.
   ```bash
   helm create my-app        # Creates a new chart with template files
   helm install my-release my-app/    # Installs the chart with name "my-release"
   helm list                 # Shows all installed charts
   ```

3. **Release is a running instance of a chart.** When you install a chart, you create a release. You can have multiple releases of the same chart with different names and configurations.
   ```bash
   helm install prod-app my-app/          # Creates "prod-app" release
   helm install staging-app my-app/       # Creates "staging-app" release from same chart
   helm status prod-app                   # Shows info about the prod-app release
   ```

4. **Values.yaml is where you customize your charts.** Instead of hardcoding settings, charts use variables that you can change without editing the template files.
   ```yaml
   # values.yaml - default values for your chart
   image:
     repository: nginx      # Which Docker image to use
     tag: latest           # Which version/tag of the image
     pullPolicy: IfNotPresent    # When to pull the image (IfNotPresent, Always, Never)
   
   service:
     type: LoadBalancer    # How to expose the service (LoadBalancer, ClusterIP, NodePort)
     port: 80             # Which port to expose
   
   replicaCount: 3        # How many pods to run
   
   resources:             # CPU and memory limits
     limits:
       cpu: 500m          # Maximum CPU (500 millicores = 0.5 cores)
       memory: 512Mi      # Maximum memory (512 megabytes)
     requests:
       cpu: 250m          # Minimum CPU needed
       memory: 256Mi      # Minimum memory needed
   ```

5. **Templates use Go templating to inject values.** Template files are regular Kubernetes YAML but with `{{ }}` placeholders that get replaced with values from values.yaml.
   ```yaml
   # templates/deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: {{ .Chart.Name }}           # Gets name from Chart.yaml
     labels:
       app: {{ .Chart.Name }}
   spec:
     replicas: {{ .Values.replicaCount }}    # Gets value from values.yaml
     selector:
       matchLabels:
         app: {{ .Chart.Name }}
     template:
       metadata:
         labels:
           app: {{ .Chart.Name }}
       spec:
         containers:
         - name: {{ .Chart.Name }}
           image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"    # Combines repo and tag
           ports:
           - containerPort: 80
           resources:
             {{- toYaml .Values.resources | nindent 12 }}    # Converts resources object to YAML
   ```

6. **Chart.yaml contains metadata about your chart.** This file describes what your chart is and what version it is.
   ```yaml
   # Chart.yaml
   apiVersion: v2              # Helm chart API version (always v2 for Helm 3)
   name: my-app               # Name of the chart
   description: A simple web app chart    # What this chart does
   version: 0.1.0             # Version of the chart itself
   appVersion: "1.0"          # Version of the app that this chart installs
   ```

7. **You can override values when installing.** Use `--set` for single values or `--values` for a whole file.
   ```bash
   # Override single values
   helm install my-app ./my-chart --set replicaCount=5 --set image.tag=v2.0
   
   # Override with a file
   helm install my-app ./my-chart --values production-values.yaml
   
   # production-values.yaml might look like:
   replicaCount: 10
   image:
     tag: v2.0
   service:
     type: LoadBalancer
   resources:
     limits:
       cpu: 1000m
       memory: 1Gi
   ```

8. **Helm repositories are like app stores for charts.** You can add repositories to download charts made by others, or publish your own.
   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami    # Add Bitnami's chart repo
   helm repo add stable https://charts.helm.sh/stable         # Add official stable repo
   helm repo update                                            # Update repo indexes
   helm search repo nginx                                      # Search for nginx charts
   helm show chart bitnami/nginx                              # Show chart info
   helm show values bitnami/nginx                             # Show default values
   ```

9. **Installing charts from repositories is easy.** You can install popular apps like databases, monitoring tools, etc. without writing any YAML.
   ```bash
   # Install PostgreSQL database
   helm install my-postgres bitnami/postgresql
   
   # Install with custom values
   helm install my-postgres bitnami/postgresql \
     --set auth.postgresPassword=mysecretpassword \
     --set primary.persistence.size=20Gi
   
   # Install Redis
   helm install my-redis bitnami/redis --set auth.enabled=false
   ```

10. **Upgrading and rolling back releases is simple.** When you change values or chart versions, use `helm upgrade`. If something breaks, use `helm rollback`.
    ```bash
    # Upgrade a release with new values
    helm upgrade my-app ./my-chart --set image.tag=v2.0
    
    # Upgrade to a new chart version
    helm upgrade my-app ./my-chart-v2/
    
    # See upgrade history
    helm history my-app
    
    # Rollback to previous version
    helm rollback my-app 1        # Rollback to revision 1
    
    # Uninstall a release
    helm uninstall my-app
    ```

11. **Conditional logic in templates helps handle different scenarios.** Use `if`, `else`, and `with` to include or exclude parts of your YAML based on values.
    ```yaml
    # In your template file
    apiVersion: v1
    kind: Service
    metadata:
      name: {{ .Chart.Name }}-service
    spec:
      selector:
        app: {{ .Chart.Name }}
      ports:
      - port: {{ .Values.service.port }}
        targetPort: 80
      {{- if eq .Values.service.type "LoadBalancer" }}    # Only add this if type is LoadBalancer
      type: LoadBalancer
      {{- else if eq .Values.service.type "NodePort" }}   # Or if type is NodePort
      type: NodePort
      nodePort: {{ .Values.service.nodePort }}
      {{- else }}                                         # Otherwise use ClusterIP
      type: ClusterIP
      {{- end }}
    ```

12. **Helper functions make templates cleaner.** Common patterns can be defined once and reused throughout your templates.
    ```yaml
    # templates/_helpers.tpl - file for reusable template snippets
    {{- define "myapp.name" -}}     # Define a helper function
    {{- .Chart.Name | trunc 63 | trimSuffix "-" -}}    # Truncate name to 63 chars
    {{- end -}}
    
    {{- define "myapp.labels" -}}   # Define common labels
    app.kubernetes.io/name: {{ include "myapp.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
    app.kubernetes.io/version: {{ .Chart.AppVersion }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
    {{- end -}}
    
    # Use helpers in other templates
    metadata:
      name: {{ include "myapp.name" . }}    # Use the helper function
      labels:
        {{- include "myapp.labels" . | nindent 4 }}    # Use the labels helper
    ```

13. **Hooks let you run jobs before or after install/upgrade.** Use them for database migrations, cleanup tasks, or pre-install checks.
    ```yaml
    # templates/pre-install-job.yaml
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: {{ .Chart.Name }}-pre-install
      annotations:
        "helm.sh/hook": pre-install        # Run this job before installing the main app
        "helm.sh/hook-weight": "-5"        # Run this before other pre-install hooks
        "helm.sh/hook-delete-policy": hook-succeeded    # Delete job after it succeeds
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: pre-install
            image: busybox
            command: ["sh", "-c", "echo 'Running pre-install tasks'; sleep 5"]
    ```

14. **Dependencies let you include other charts.** If your app needs a database, you can include the database chart as a dependency instead of managing it separately.
    ```yaml
    # Chart.yaml
    dependencies:
    - name: postgresql                     # Include PostgreSQL chart
      version: 12.1.2                    # Which version of the chart
      repository: https://charts.bitnami.com/bitnami    # Where to find it
      condition: postgresql.enabled       # Only install if this value is true
    - name: redis
      version: 17.4.0
      repository: https://charts.bitnami.com/bitnami
      condition: redis.enabled
    ```
    
    ```yaml
    # values.yaml
    postgresql:
      enabled: true                      # Enable PostgreSQL dependency
      auth:
        postgresPassword: mypassword
        database: myapp
    
    redis:
      enabled: false                     # Disable Redis dependency
    ```
    
    ```bash
    helm dependency update              # Download the dependency charts
    helm install my-app ./my-chart     # Install with dependencies
    ```

15. **Testing your charts prevents deployment issues.** Helm can run tests to verify your deployment works correctly.
    ```yaml
    # templates/tests/test-connection.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: {{ .Chart.Name }}-test
      annotations:
        "helm.sh/hook": test            # This is a test pod, not part of the main app
    spec:
      restartPolicy: Never
      containers:
      - name: test
        image: busybox
        command: ['wget']
        args: ['{{ .Chart.Name }}-service:{{ .Values.service.port }}']    # Test if service responds
    ```
    
    ```bash
    helm test my-app                    # Run tests for the release
    ```

16. **Common commands you'll use daily:**
    ```bash
    # Creating and managing charts
    helm create my-chart                # Create new chart
    helm template my-chart ./my-chart  # See generated YAML without installing
    helm lint ./my-chart               # Check chart for problems
    helm package ./my-chart            # Create a .tgz package
    
    # Managing releases
    helm install name ./chart          # Install chart
    helm upgrade name ./chart          # Upgrade existing release
    helm rollback name 1               # Rollback to revision 1
    helm uninstall name                # Remove release
    helm list                          # Show all releases
    helm history name                  # Show release history
    
    # Working with repositories
    helm repo add name url             # Add chart repository
    helm repo list                     # Show added repositories
    helm repo update                   # Update repository indexes
    helm search repo keyword           # Search for charts
    
    # Getting info
    helm show chart repo/chartname     # Show chart metadata
    helm show values repo/chartname    # Show default values
    helm get values release-name       # Show values used by release
    helm status release-name           # Show release status
    ```

## Best practices for production

17. **Always use specific versions, not 'latest'.** This makes your deployments predictable and easier to debug.
    ```yaml
    # Bad - unpredictable
    image:
      tag: latest
    
    # Good - predictable
    image:
      tag: v1.2.3
    ```

18. **Keep values.yaml organized and documented.** Future you (and your team) will thank you.
    ```yaml
    # values.yaml
    # Application settings
    app:
      name: my-app
      version: v1.0.0
      
    # Container image settings
    image:
      repository: my-company/my-app    # Docker image repository
      tag: v1.0.0                     # Image tag (never use 'latest' in production)
      pullPolicy: IfNotPresent        # Image pull policy
      
    # Kubernetes deployment settings
    deployment:
      replicaCount: 3                 # Number of pods to run
      strategy:
        type: RollingUpdate           # How to update pods (RollingUpdate or Recreate)
        rollingUpdate:
          maxSurge: 1                 # Max extra pods during update
          maxUnavailable: 0           # Max unavailable pods during update
    ```

19. **Use separate values files for different environments.** Don't put production secrets in your main values.yaml.
    ```bash
    # Development
    helm install my-app ./chart -f values.yaml -f values-dev.yaml
    
    # Production
    helm install my-app ./chart -f values.yaml -f values-prod.yaml
    ```

20. **Validate your templates before deploying.** Use `helm template` to see the generated YAML and catch errors early.
    ```bash
    helm template my-app ./my-chart --values production-values.yaml > output.yaml
    # Review output.yaml to make sure it looks right before installing
    ``` 