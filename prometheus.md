# Prometheus Notes

1. **Prometheus is like a data collector that constantly asks your apps "How are you doing?"** It scrapes metrics from your applications and stores them in a time-series database so you can see trends and set up alerts.

2. **Metrics are just numbers with labels.** Think of them like temperature readings with tags saying where and when they were taken.
   ```yaml
   # prometheus.yml - main configuration file
   global:
     scrape_interval: 15s          # How often to collect metrics from targets
     evaluation_interval: 15s      # How often to evaluate alerting rules

   scrape_configs:                 # List of things to monitor
   - job_name: 'my-app'           # Name for this group of targets
     static_configs:              # Hardcoded list of targets
     - targets: ['localhost:8080'] # Where to find the metrics endpoint
   ```

3. **Your app needs to expose metrics on an HTTP endpoint.** Usually `/metrics`. Prometheus will visit this URL regularly to collect data.
   ```python
   # Python example with prometheus_client
   from prometheus_client import Counter, Histogram, generate_latest
   from flask import Flask, Response

   app = Flask(__name__)

   # Create metrics
   REQUEST_COUNT = Counter('http_requests_total', 'Total HTTP requests', ['method', 'endpoint'])
   REQUEST_DURATION = Histogram('http_request_duration_seconds', 'HTTP request duration')

   @app.route('/metrics')
   def metrics():
       return Response(generate_latest(), mimetype='text/plain')  # Prometheus format

   @app.route('/api/users')
   def get_users():
       REQUEST_COUNT.labels(method='GET', endpoint='/api/users').inc()  # Increment counter
       # ... your app logic
   ```

   **Function calls explained:**
   - `Counter(name, description, label_names)` - Creates a counter metric that only increases
   - `Histogram(name, description, label_names, buckets)` - Creates a histogram for measuring distributions
   - `generate_latest()` - Returns all metrics in Prometheus text format
   - `labels(**kwargs)` - Returns a metric instance with specific label values
   - `.inc(amount=1)` - Increments a counter by the specified amount (default 1)
   - `.observe(value)` - Records a value in a histogram or summary
   - `.set(value)` - Sets a gauge to a specific value

4. **There are four types of metrics you can collect:**
   - **Counter**: Only goes up (like total requests served)
   - **Gauge**: Can go up and down (like current memory usage)
   - **Histogram**: Measures distribution of values (like request durations)
   - **Summary**: Like histogram but calculates percentiles differently

   ```yaml
   # Example metrics output at /metrics endpoint
   # HELP http_requests_total Total number of HTTP requests
   # TYPE http_requests_total counter
   http_requests_total{method="GET",endpoint="/api/users"} 1234

   # HELP memory_usage_bytes Current memory usage in bytes
   # TYPE memory_usage_bytes gauge
   memory_usage_bytes 524288000

   # HELP http_request_duration_seconds HTTP request duration
   # TYPE http_request_duration_seconds histogram
   http_request_duration_seconds_bucket{le="0.1"} 100
   http_request_duration_seconds_bucket{le="0.5"} 200
   http_request_duration_seconds_bucket{le="1.0"} 250
   http_request_duration_seconds_bucket{le="+Inf"} 250
   ```

   **Metric properties explained:**
   - `# HELP` - Human-readable description of what the metric measures
   - `# TYPE` - Declares the metric type (counter, gauge, histogram, summary)
   - `{label="value"}` - Label pairs that add dimensions to metrics
   - `le="0.1"` - "Less than or equal to" bucket boundary for histograms
   - `+Inf` - Infinity bucket that contains all observations

5. **Labels are like tags that help you slice and dice your data.** Use them to add context like environment, version, or user type.
   ```python
   # Good labels - help you filter and group data
   REQUEST_COUNT = Counter('requests_total', 'Total requests', 
                          ['method', 'status_code', 'endpoint'])
   
   # Usage
   REQUEST_COUNT.labels(method='POST', status_code='200', endpoint='/api/login').inc()
   REQUEST_COUNT.labels(method='GET', status_code='404', endpoint='/api/missing').inc()
   ```

6. **PromQL is the query language for asking questions about your data.** It's like SQL but for time-series data.
   ```promql
   # Basic queries
   http_requests_total                           # Get all request counts
   http_requests_total{method="GET"}            # Only GET requests
   http_requests_total{method="GET",status="200"} # GET requests that succeeded

   # Rate calculations
   rate(http_requests_total[5m])                # Requests per second over last 5 minutes
   sum(rate(http_requests_total[5m])) by (method) # Requests per second grouped by method

   # Aggregations
   sum(http_requests_total)                     # Total requests across all instances
   avg(memory_usage_bytes)                      # Average memory usage
   max(cpu_usage_percent) by (instance)        # Highest CPU usage per instance
   ```

   **PromQL functions explained:**
   - `rate(metric[duration])` - Calculates per-second rate of increase over time range
   - `sum(metric) by (label)` - Sums values grouped by specified labels
   - `avg(metric)` - Calculates average value across all series
   - `max(metric)` - Returns maximum value across all series
   - `min(metric)` - Returns minimum value across all series
   - `count(metric)` - Counts number of series
   - `increase(metric[duration])` - Total increase over time range
   - `irate(metric[duration])` - Instantaneous rate using last two data points
   - `histogram_quantile(quantile, metric)` - Calculates quantiles from histogram buckets

7. **Alerting rules let Prometheus notify you when things go wrong.** Define conditions and Prometheus will fire alerts when they're met.
   ```yaml
   # alerting_rules.yml
   groups:
   - name: my-app-alerts              # Group name for organizing rules
     rules:
     - alert: HighErrorRate           # Name of this alert
       expr: |                        # PromQL expression that triggers alert
         sum(rate(http_requests_total{status=~"5.."}[5m])) /
         sum(rate(http_requests_total[5m])) > 0.1
       for: 2m                        # Alert only if condition is true for 2 minutes
       labels:                        # Labels to add to the alert
         severity: critical
         team: backend
       annotations:                   # Human-readable information about the alert
         summary: "High error rate detected"
         description: "Error rate is {{ $value | humanizePercentage }} for the last 5 minutes"

     - alert: HighMemoryUsage
       expr: memory_usage_bytes / memory_total_bytes > 0.9
       for: 5m
       labels:
         severity: warning
       annotations:
         summary: "Memory usage is high on {{ $labels.instance }}"
   ```

   **Alert rule properties explained:**
   - `groups` - Top-level container for organizing related rules
   - `name` - Identifier for the rule group
   - `interval` - How often to evaluate rules (defaults to global evaluation_interval)
   - `alert` - Name of the alert (becomes alertname label)
   - `expr` - PromQL expression that must be true to fire alert
   - `for` - How long condition must be true before firing
   - `labels` - Additional labels to attach to the alert
   - `annotations` - Metadata for alert descriptions and runbooks
   - `{{ $value }}` - Template variable containing the expression result
   - `{{ $labels.labelname }}` - Template variable for accessing label values
   - `status=~"5.."` - Regex match for status codes starting with 5

8. **Alertmanager handles alert notifications.** It receives alerts from Prometheus and sends them to Slack, email, PagerDuty, etc.
   ```yaml
   # alertmanager.yml
   global:
     smtp_smarthost: 'localhost:587'    # Email server for sending notifications
     smtp_from: 'alerts@mycompany.com'

   route:                               # How to route alerts to different receivers
     group_by: ['alertname']            # Group alerts with same name together
     group_wait: 10s                    # Wait 10s before sending first notification
     group_interval: 10s                # Wait 10s between notifications for same group
     repeat_interval: 1h                # Resend alert every hour if still firing
     receiver: 'web.hook'               # Default receiver for all alerts

   receivers:                           # Where to send notifications
   - name: 'web.hook'
     slack_configs:                     # Send to Slack
     - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
       channel: '#alerts'
       title: 'Alert: {{ .GroupLabels.alertname }}'
       text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
   ```

   **Alertmanager properties explained:**
   - `global` - Default configuration applied to all receivers
   - `smtp_smarthost` - SMTP server address and port for email
   - `smtp_from` - Default sender email address
   - `route` - Tree structure defining how alerts are routed
   - `group_by` - Labels used to group alerts together
   - `group_wait` - Time to wait before sending first notification for a group
   - `group_interval` - Time between notifications for the same group
   - `repeat_interval` - How often to resend notifications for ongoing alerts
   - `receiver` - Name of receiver to handle alerts
   - `receivers` - List of notification destinations
   - `slack_configs` - Configuration for Slack notifications
   - `api_url` - Slack webhook URL
   - `channel` - Slack channel to send to
   - `{{ .GroupLabels.alertname }}` - Template for accessing grouped alert labels
   - `{{ range .Alerts }}` - Template loop over all alerts in group

9. **Service discovery automatically finds targets to monitor.** Instead of hardcoding IP addresses, let Prometheus discover them.
   ```yaml
   # Kubernetes service discovery
   scrape_configs:
   - job_name: 'kubernetes-pods'
     kubernetes_sd_configs:            # Use Kubernetes API to find targets
     - role: pod                       # Look for pods
     relabeling_configs:               # Rules for processing discovered targets
     - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
       action: keep                    # Only keep pods with this annotation
       regex: true
     - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
       action: replace                 # Use custom metrics path if specified
       target_label: __metrics_path__
       regex: (.+)

   # File-based service discovery
   - job_name: 'file-discovery'
     file_sd_configs:                  # Read targets from files
     - files:
       - '/etc/prometheus/targets/*.json'  # Watch these files for changes
       refresh_interval: 30s           # Check for file changes every 30s
   ```

   **Service discovery properties explained:**
   - `kubernetes_sd_configs` - Configuration for Kubernetes service discovery
   - `role` - What Kubernetes objects to discover (pod, service, endpoints, node)
   - `relabeling_configs` - Rules to modify or filter discovered targets
   - `source_labels` - Labels to read values from
   - `action` - What to do with the labels (keep, drop, replace, labelmap)
   - `regex` - Regular expression to match against source labels
   - `target_label` - Label to write the result to
   - `__meta_kubernetes_*` - Metadata labels provided by Kubernetes discovery
   - `__metrics_path__` - Special label that sets the metrics endpoint path
   - `file_sd_configs` - Configuration for file-based service discovery
   - `files` - List of file patterns to watch
   - `refresh_interval` - How often to check files for changes

10. **Recording rules pre-calculate expensive queries.** Instead of running complex queries every time, calculate them once and store the result.
    ```yaml
    # recording_rules.yml
    groups:
    - name: performance-rules
      interval: 30s                    # How often to evaluate these rules
      rules:
      - record: job:http_requests:rate5m    # Name of the new metric to create
        expr: sum(rate(http_requests_total[5m])) by (job)  # Query to calculate

      - record: instance:cpu_usage:avg5m
        expr: avg(cpu_usage_percent) by (instance)

      # Use recording rules in alerts for better performance
      - alert: HighRequestRate
        expr: job:http_requests:rate5m > 100    # Use pre-calculated metric
        for: 1m
    ```

    **Recording rule properties explained:**
    - `record` - Name of the new metric to create (follows naming convention)
    - `expr` - PromQL expression to evaluate and store
    - `labels` - Additional labels to add to the recorded metric
    - Naming convention: `level:metric:operations` (e.g., `job:http_requests:rate5m`)

11. **Grafana is the pretty face for your Prometheus data.** It creates dashboards and graphs from your metrics.
    ```json
    // Grafana dashboard panel configuration
    {
      "title": "Request Rate",
      "type": "graph",
      "targets": [
        {
          "expr": "sum(rate(http_requests_total[5m])) by (method)",  // PromQL query
          "legendFormat": "{{ method }} requests/sec",              // How to label lines
          "refId": "A"
        }
      ],
      "yAxes": [
        {
          "label": "Requests per second",    // Y-axis label
          "min": 0                          // Start Y-axis at 0
        }
      ]
    }
    ```

    **Grafana panel properties explained:**
    - `title` - Display name for the panel
    - `type` - Panel type (graph, singlestat, table, heatmap, etc.)
    - `targets` - Array of data sources/queries for the panel
    - `expr` - PromQL query to execute
    - `legendFormat` - Template for legend labels using query result labels
    - `refId` - Unique identifier for this query within the panel
    - `yAxes` - Configuration for Y-axis display
    - `label` - Y-axis title
    - `min`/`max` - Y-axis range limits
    - `gridPos` - Panel position and size on dashboard
    - `datasource` - Which Prometheus instance to query

12. **Exporters collect metrics from third-party systems.** They translate other systems' data into Prometheus format.
    ```yaml
    # Common exporters to add to prometheus.yml
    scrape_configs:
    - job_name: 'node-exporter'        # System metrics (CPU, memory, disk)
      static_configs:
      - targets: ['localhost:9100']

    - job_name: 'postgres-exporter'    # PostgreSQL database metrics
      static_configs:
      - targets: ['localhost:9187']

    - job_name: 'redis-exporter'       # Redis metrics
      static_configs:
      - targets: ['localhost:9121']

    - job_name: 'nginx-exporter'       # Nginx web server metrics
      static_configs:
      - targets: ['localhost:9113']
    ```

    **Common exporter ports and metrics:**
    - `node-exporter:9100` - System metrics (node_cpu_seconds_total, node_memory_MemAvailable_bytes)
    - `postgres-exporter:9187` - Database metrics (pg_up, pg_stat_database_tup_returned)
    - `redis-exporter:9121` - Redis metrics (redis_up, redis_connected_clients)
    - `nginx-exporter:9113` - Web server metrics (nginx_http_requests_total, nginx_connections_active)
    - `blackbox-exporter:9115` - Endpoint probing (probe_success, probe_duration_seconds)

13. **Federation lets multiple Prometheus servers share data.** Useful for scaling across multiple data centers or teams.
    ```yaml
    # Global Prometheus federating from local ones
    scrape_configs:
    - job_name: 'federate'
      scrape_interval: 15s
      honor_labels: true               # Keep original labels from federated servers
      metrics_path: '/federate'        # Special endpoint for federation
      params:
        'match[]':                     # Which metrics to pull from other servers
          - '{job=~"mysql.*"}'         # All MySQL-related metrics
          - '{__name__=~"job:.*"}'     # All recording rule results
      static_configs:
      - targets:
        - 'prometheus-dc1:9090'        # Prometheus server in datacenter 1
        - 'prometheus-dc2:9090'        # Prometheus server in datacenter 2
    ```

    **Federation properties explained:**
    - `honor_labels` - Preserve original labels from federated instances
    - `metrics_path: '/federate'` - Special endpoint that returns filtered metrics
    - `params` - URL parameters to pass to the federate endpoint
    - `'match[]'` - Array of PromQL selectors for which metrics to federate
    - `{job=~"mysql.*"}` - Regex selector for all jobs starting with "mysql"
    - `{__name__=~"job:.*"}` - Selector for all recording rule metrics

14. **Retention and storage management keep your disk from filling up.** Configure how long to keep data and where to store it.
    ```bash
    # Prometheus startup flags
    prometheus \
      --config.file=/etc/prometheus/prometheus.yml \
      --storage.tsdb.path=/var/lib/prometheus/ \      # Where to store data
      --storage.tsdb.retention.time=30d \             # Keep data for 30 days
      --storage.tsdb.retention.size=50GB \            # Or until 50GB used
      --web.console.libraries=/etc/prometheus/console_libraries \
      --web.console.templates=/etc/prometheus/consoles \
      --web.listen-address=0.0.0.0:9090              # What port to listen on
    ```

    **Prometheus startup flags explained:**
    - `--config.file` - Path to main configuration file
    - `--storage.tsdb.path` - Directory where time-series data is stored
    - `--storage.tsdb.retention.time` - How long to keep data (e.g., 30d, 1y)
    - `--storage.tsdb.retention.size` - Maximum disk space to use
    - `--web.listen-address` - IP and port for web interface
    - `--web.external-url` - URL that users will use to reach Prometheus
    - `--web.route-prefix` - Prefix for web endpoints
    - `--log.level` - Logging level (debug, info, warn, error)
    - `--query.timeout` - Maximum query execution time
    - `--query.max-concurrency` - Maximum concurrent queries

15. **Common metric naming conventions help keep things organized:**
    ```python
    # Good metric names - clear and consistent
    http_requests_total              # Counter: ends with _total
    http_request_duration_seconds    # Histogram: includes unit
    memory_usage_bytes              # Gauge: includes unit
    database_connections_active     # Gauge: describes what it measures

    # Bad metric names - avoid these
    requests                        # Too vague
    response_time                   # No unit specified
    db_conn                        # Abbreviated and unclear
    ```

    **Metric naming best practices:**
    - Use `snake_case` for metric names
    - Include units in the name (`_seconds`, `_bytes`, `_total`)
    - Counters should end with `_total`
    - Use descriptive names that explain what is measured
    - Avoid abbreviations that aren't universally understood
    - Group related metrics with common prefixes (`http_`, `database_`)

16. **Health checks and monitoring Prometheus itself.** Make sure your monitoring system is healthy.
    ```yaml
    # Monitor Prometheus with itself
    scrape_configs:
    - job_name: 'prometheus'
      static_configs:
      - targets: ['localhost:9090']    # Prometheus monitors itself

    # Key metrics to watch
    # prometheus_tsdb_head_samples_appended_total - samples being stored
    # prometheus_config_last_reload_successful - config reloads working
    # prometheus_rule_evaluation_failures_total - alerting rules failing
    # up{job="your-app"} - whether your targets are reachable
    ```

    **Important Prometheus self-monitoring metrics:**
    - `up` - Whether a target is reachable (1 = up, 0 = down)
    - `prometheus_config_last_reload_successful` - Whether config reload succeeded
    - `prometheus_tsdb_head_samples_appended_total` - Rate of samples being ingested
    - `prometheus_rule_evaluation_failures_total` - Failed rule evaluations
    - `prometheus_notifications_total` - Alerts sent to Alertmanager
    - `prometheus_tsdb_compactions_total` - Database compaction operations
    - `prometheus_tsdb_head_series` - Number of active time series
    - `prometheus_query_duration_seconds` - Query execution time

17. **Docker setup for quick testing.** Get Prometheus running locally in minutes.
    ```yaml
    # docker-compose.yml
    version: '3.8'
    services:
      prometheus:
        image: prom/prometheus:latest
        container_name: prometheus
        ports:
          - "9090:9090"              # Access Prometheus UI at localhost:9090
        volumes:
          - ./prometheus.yml:/etc/prometheus/prometheus.yml  # Mount config file
          - prometheus_data:/prometheus    # Persist data
        command:
          - '--config.file=/etc/prometheus/prometheus.yml'
          - '--storage.tsdb.path=/prometheus'
          - '--web.console.libraries=/etc/prometheus/console_libraries'
          - '--web.console.templates=/etc/prometheus/consoles'

      grafana:
        image: grafana/grafana:latest
        container_name: grafana
        ports:
          - "3000:3000"              # Access Grafana UI at localhost:3000
        environment:
          - GF_SECURITY_ADMIN_PASSWORD=admin    # Default login: admin/admin
        volumes:
          - grafana_data:/var/lib/grafana

    volumes:
      prometheus_data:
      grafana_data:
    ```

    **Docker Compose properties explained:**
    - `version` - Docker Compose file format version
    - `services` - Container definitions
    - `image` - Docker image to use
    - `container_name` - Custom name for the container
    - `ports` - Port mapping from host to container (host:container)
    - `volumes` - Mount points for persistent data and configuration
    - `command` - Override default container command
    - `environment` - Environment variables to set in container
    - `depends_on` - Service startup dependencies
    - `restart` - Restart policy (no, always, on-failure, unless-stopped)

18. **Best practices for production deployments:**
    - **Use recording rules** for expensive queries that run frequently
    - **Set up proper retention** based on your storage capacity and needs
    - **Monitor Prometheus itself** - set up alerts for when it's down or struggling
    - **Use service discovery** instead of static configs for dynamic environments
    - **Keep cardinality low** - don't create metrics with too many unique label combinations
    - **Use federation** for scaling across multiple teams or data centers
    - **Back up your data** - Prometheus data is valuable for debugging and analysis

19. **Additional PromQL operators and functions for advanced queries:**
    ```promql
    # Comparison operators
    http_requests_total > 100        # Greater than
    memory_usage_bytes < 1000000000  # Less than
    cpu_usage_percent >= 80          # Greater than or equal
    disk_free_bytes <= 1000000000    # Less than or equal
    up == 1                          # Equal to
    up != 0                          # Not equal to

    # Logical operators
    up == 1 and http_requests_total > 100    # Both conditions true
    up == 0 or cpu_usage_percent > 90        # Either condition true
    up == 1 unless cpu_usage_percent > 95    # First true and second false

    # Time functions
    time()                           # Current Unix timestamp
    hour()                          # Hour of day (0-23)
    day_of_week()                   # Day of week (0-6, Sunday=0)
    days_in_month()                 # Number of days in current month

    # Math functions
    abs(cpu_usage_percent - 50)      # Absolute value
    ceil(memory_usage_bytes / 1000000) # Round up
    floor(response_time_seconds)     # Round down
    round(cpu_usage_percent, 0.1)    # Round to nearest 0.1

    # String functions
    label_replace(up, "env", "prod", "instance", ".*prod.*")  # Add/modify labels
    label_join(up, "target", ":", "instance", "job")         # Join label values
    ```

    **Advanced PromQL functions explained:**
    - `label_replace(metric, dst_label, replacement, src_label, regex)` - Create/modify labels
    - `label_join(metric, dst_label, separator, src_label1, src_label2)` - Combine labels
    - `on(labels)` - Specify which labels to match in binary operations
    - `ignoring(labels)` - Ignore specific labels when matching
    - `group_left(labels)` / `group_right(labels)` - Many-to-one / one-to-many joins
    - `offset` - Look at data from a specific time in the past
    - `@` - Evaluate expression at a specific timestamp
    