---
title: "Prometheus & Grafana Interview Questions & Answers (2026) Part 01"
description: "30+ Prometheus and Grafana interview questions and answers covering monitoring, alerting, PromQL, exporters, dashboards, Alertmanager, Kubernetes monitoring, troubleshooting, and observability best practices — Basic to Advanced."
date: 2025-04-26
author: "DB"
tags: ["prometheus", "grafana", "monitoring", "observability", "promql", "alertmanager", "kubernetes", "devops"]
tool: "prometheus-grafana"
level: "All Levels"
question_count: 50
draft: "false"
---

{{< qa num="1" q="Prometheus shows a target as DOWN in the Targets UI. The service is running and accessible. How do you troubleshoot?" level="intermediate" >}}
**Ans:**
**Key Discussion Points:**
- Check the error message in Prometheus Targets UI (`/targets` page)
- Common errors: connection refused, timeout, TLS mismatch, 401 Unauthorized
- Verify the scrape config `job_name`, `targets`, and `metrics_path` in `prometheus.yml`
- Confirm the exporter is listening on the correct port: `curl http://<host>:<port>/metrics`
- Check network policies/security groups blocking Prometheus → target communication
- If using service discovery, verify labels and relabeling rules
- Check `scrape_timeout` vs exporter response time

**Sample Answer:** *"I'd first check the error message on the Targets page. If it's a connection timeout, I'd verify network connectivity from Prometheus to the target. If it's an authentication error, I'd check the scrape config for bearer_token or basic_auth settings. I'd also curl the metrics endpoint directly from the Prometheus host to isolate the issue."*

{{< /qa >}}
{{< qa num="2" q="Your Prometheus server is running out of memory. Investigation reveals a metric with millions of time series. What caused this and how do you fix it?" level="advanced" >}}
**Ans:**
**Key Discussion Points:**
- High cardinality occurs when labels have many unique values (e.g., user_id, request_id, IP address)
- Each unique label combination = a new time series, consuming memory
- Identify problematic metrics with `topk(10, count by (__name__)({__name__=~".+"}))` (use TSDB status page)
- Fix: Remove high-cardinality labels from instrumentation code
- Use `metric_relabel_configs` to drop or aggregate labels before ingestion
- Use `labeldrop` or `labelkeep` in relabeling to strip unnecessary labels
- Consider using histograms instead of per-request metrics

**Sample Answer:** *"High cardinality is typically caused by adding labels with unbounded values like user IDs or session tokens. I'd use the Prometheus TSDB status page to find the top metrics by series count. Then I'd work with developers to remove those labels from instrumentation, or use `metric_relabel_configs` to drop them at the scrape level."*

{{< /qa >}}
{{< qa num="3" q="Your Grafana dashboard has a complex PromQL query that takes 30+ seconds to load because it aggregates 6 months of data. How do you fix this?" level="advanced" >}}
**Ans:**

**Key Discussion Points:**
- Create a Prometheus recording rule to pre-compute the expensive query
- Recording rules run at evaluation intervals (e.g., every 1 minute) and store results as new metrics
- Example: `record: job:http_requests:rate5m` with `expr: rate(http_requests_total[5m])`
- Replace the slow Grafana query with the pre-computed metric name
- Group recording rules in a separate rules file and reference in `prometheus.yml`
- Use `by` clause in recording rules to reduce cardinality of stored results
- Verify rules are loading: check `/rules` endpoint in Prometheus UI

**Sample Answer:** *"I'd create a recording rule that pre-computes the expensive aggregation on a 1-minute interval. The result is stored as a new metric with a much smaller dataset. Grafana then queries this pre-computed metric instead of running the complex query over 6 months of raw data, reducing load time from 30 seconds to under a second."*

{{< /qa >}}
{{< qa num="4" q="Your Prometheus setup generates 200 alerts during a scheduled maintenance window, flooding your team's Slack. How do you handle this properly?" level="advanced" >}}
**Ans:**
**Key Discussion Points:**
- Create an Alertmanager silence for the maintenance window duration
- Use the Alertmanager UI or `amtool silence add` CLI to create silences
- Define silence matchers for affected services or environments
- Use `inhibit_rules` in Alertmanager config to suppress child alerts when parent is firing
- Configure `group_wait`, `group_interval`, `repeat_interval` in routing to reduce noise
- Use `routes` in Alertmanager to route maintenance alerts to a separate low-priority channel
- After maintenance, expire the silence to resume normal alerting

**Sample Answer:** *"I'd create an Alertmanager silence before the maintenance window starts, matching the affected services using label matchers. During the window, all matching alerts are suppressed. I'd also review `inhibit_rules` to prevent cascading alerts and configure `group_by` to batch related alerts into a single notification."*

{{< /qa >}}
{{< qa num="5" q="You have 10 Kubernetes clusters each with its own Prometheus. You need a global view of all clusters. How do you architect this?" level="advanced" >}}
**Ans:**

**Key Discussion Points:**
- Set up a global Prometheus server that federates from cluster-level Prometheus instances
- Use `/federate` endpoint with `match[]` parameter to pull aggregated metrics
- Cluster Prometheus instances scrape detailed metrics; global pulls recording rule results only
- Use `cluster` label to distinguish metrics across clusters in the global instance
- Consider Thanos or Cortex for production-grade multi-cluster long-term storage
- Thanos Sidecar + Query component provides deduplication and global query capability
- Use remote_write to push metrics to a central Thanos Receiver

**Sample Answer:** *"For a simple setup, I'd use Prometheus federation where a global Prometheus scrapes pre-aggregated metrics from each cluster's Prometheus via the `/federate` endpoint. For production scale, I'd deploy Thanos with sidecars in each cluster, use object storage for long-term retention, and a global Thanos Query layer for unified querying across all clusters."*

{{< /qa >}}
{{< qa num="6" q="Your Prometheus server is consuming 32GB of RAM and the team is concerned. What strategies do you use to reduce memory usage?" level="advanced" >}}
**Ans:**

**Key Discussion Points:**
- Identify high-cardinality metrics using Prometheus TSDB status page (`/tsdb-status`)
- Reduce scrape intervals for non-critical targets (e.g., 60s instead of 15s)
- Drop unused metrics using `metric_relabel_configs` with `action: drop`
- Reduce `--storage.tsdb.retention.time` to store less historical data in memory
- Use recording rules to replace high-cardinality queries with aggregated metrics
- Tune `--query.max-samples` and `--storage.tsdb.max-block-duration`
- Offload long-term storage to Thanos/Cortex and reduce local retention

**Sample Answer:** *"I'd start with the TSDB status page to find which metrics are using the most series. I'd then work with teams to drop unused metrics via relabeling and reduce scrape frequency for less critical jobs. For long-term data, I'd offload to Thanos and reduce local retention to 15 days, significantly cutting memory usage."*

{{< /qa >}}
{{< qa num="7" q="A new microservice is deployed in Kubernetes but Prometheus isn't scraping it. The pod is running. What do you check?" level="intermediate" >}}
**Ans:**

**Key Discussion Points:**
- Verify pod has Prometheus scrape annotations: `prometheus.io/scrape: "true"`, `prometheus.io/port`, `prometheus.io/path`
- Check if using Prometheus Operator: ensure a `ServiceMonitor` or `PodMonitor` CRD is created
- Verify the ServiceMonitor's `namespaceSelector` and `selector` match the service labels
- Check Prometheus `serviceMonitorSelector` in the Prometheus CR matches ServiceMonitor labels
- Confirm the metrics endpoint is reachable: `kubectl exec` into Prometheus pod and curl target
- Review RBAC: Prometheus service account needs `get`, `list`, `watch` on pods/services/endpoints

**Sample Answer:** *"I'd first check if the pod has the required Prometheus annotations for annotation-based discovery. If using Prometheus Operator, I'd verify a ServiceMonitor exists with correct label selectors matching the service. I'd also check RBAC permissions and confirm the metrics endpoint is accessible within the cluster network."*

{{< /qa >}}
{{< qa num="8" q="Your team is building a payment processing service in Go. What custom Prometheus metrics would you instrument and how?" level="advanced" >}}
**Ans:**
**Key Discussion Points:**
- Use `prometheus/client_golang` library
- Counter: `payment_requests_total` with labels `{status="success|failed", payment_method}`
- Histogram: `payment_processing_duration_seconds` to track latency percentiles
- Gauge: `active_payment_sessions` for current in-flight payments
- Counter: `payment_amount_total` for business KPI tracking
- Expose `/metrics` endpoint using `promhttp.Handler()`
- Define meaningful bucket boundaries for histograms based on SLO targets
- Use `MustRegister` at startup to fail fast if metrics conflict

**Sample Answer:** *"I'd instrument at minimum: a counter for total payment attempts labeled by status and method, a histogram for processing duration with buckets aligned to our SLO (e.g., 0.1s, 0.5s, 1s, 5s), and a gauge for active sessions. I'd expose these on `/metrics` and add a ServiceMonitor for Prometheus to auto-discover the service."*

{{< /qa >}}
{{< qa num="9" q="You need 2 years of metrics retention for compliance but your Prometheus local disk is only 500GB. How do you handle this?" level="advanced" >}}
**Ans:**

**Key Discussion Points:**
- Prometheus local storage is not designed for multi-year retention
- Implement remote_write to send metrics to long-term storage: Thanos, Cortex, or VictoriaMetrics
- Thanos: deploy Sidecar + Object Store (S3) for unlimited retention at low cost
- Use `--storage.tsdb.retention.time=15d` for local storage; rely on remote for older data
- Implement downsampling in Thanos Compactor for older data (5min, 1hr resolution)
- Use Grafana with Thanos Querier as data source for transparent querying across retention tiers
- Estimate storage: ~1-3 bytes per sample; 1M series × 1 sample/15s × 2yr ≈ multiple TB

**Sample Answer:** *"I'd set up Prometheus with a 15-day local retention and configure remote_write to Thanos backed by S3 for 2-year retention. Thanos Compactor handles downsampling of older data to reduce storage costs. Grafana queries the Thanos Querier, which transparently merges local and long-term data."*

{{< /qa >}}
{{< qa num="10" q="A developer asks you which PromQL function to use for alerting on HTTP error rate. They're confused between rate(), irate(), and increase(). How do you explain and recommend?" level="intermediate" >}}
**Ans:**

**Key Discussion Points:**
- `rate(metric[5m])`: per-second rate averaged over 5 minutes — smooth, good for alerting
- `irate(metric[5m])`: per-second rate of last 2 data points — highly responsive, volatile, good for graphing spikes
- `increase(metric[5m])`: total increase over 5 minutes — useful for "how many errors in last 5 min"
- For alerting: use `rate()` to avoid false alerts from momentary spikes
- For dashboards showing current throughput: `irate()` for real-time panels
- For "errors per window" business metrics: `increase()`
- All three handle counter resets (e.g., pod restarts) automatically

**Sample Answer:** *"For alerting on error rate, I'd use `rate()` with a 5-minute window because it averages out momentary spikes and reduces alert flapping. `irate()` is too sensitive for alerting but great for real-time dashboards. `increase()` is best when you want the raw count over a time window, like 'how many 5xx errors in the last hour'."*


{{< /qa >}}
{{< qa num="11" q="Your Prometheus remote_write to Thanos is falling behind, and the queue is growing. How do you diagnose and resolve this?" level="advanced" >}}
**Ans:**

**Key Discussion Points:**
- Check `prometheus_remote_storage_queue_highest_sent_timestamp_seconds` metric
- Monitor `prometheus_remote_storage_samples_failed_total` for send failures
- Check Thanos Receiver logs for ingestion bottlenecks
- Tune `remote_write` config: increase `max_shards`, `capacity`, `max_samples_per_send`
- Check network bandwidth between Prometheus and Thanos Receiver
- If Thanos Receiver is overloaded, scale horizontally using hashring configuration
- Enable WAL-based remote_write for durability and replay on failure

**Sample Answer:** *"I'd check the remote_write queue metrics to see how far behind we are and whether there are send failures. If the queue is growing due to throughput, I'd increase `max_shards` in the remote_write config to parallelize writes. If Thanos Receiver is the bottleneck, I'd scale it out using a hashring setup."*

{{< /qa >}}
{{< qa num="12" q="You need to monitor SSL certificate expiry and HTTP response codes for 50 external URLs. How do you implement this with Prometheus?" level="advanced" >}}
**Ans:**

**Key Discussion Points:**
- Deploy Prometheus Blackbox Exporter with `http_probe`, `tcp_probe`, `icmp_probe` modules
- Configure HTTP probe module to follow redirects and check SSL
- Use metric `probe_ssl_earliest_cert_expiry` to track certificate expiration
- Alert: `(probe_ssl_earliest_cert_expiry - time()) / 86400 < 30` for certs expiring in 30 days
- Use `probe_success == 0` to alert on endpoint failures
- Define 50 targets in Prometheus scrape config using `blackbox` job with `params`
- Use file_sd or HTTP SD for dynamic URL management

**Sample Answer:** *"I'd deploy the Blackbox Exporter and configure an HTTP probe module with TLS verification enabled. I'd add all 50 URLs as targets in the blackbox scrape job. For SSL expiry, I'd create a Prometheus alert on `probe_ssl_earliest_cert_expiry` giving 30 days' warning, and a separate alert on `probe_success == 0` for availability monitoring."*


{{< /qa >}}
{{< qa num="13" q="Your application stops publishing metrics when it crashes, so Prometheus shows no data rather than an alert. How do you alert on missing metrics?" level="intermediate" >}}
**Ans:**

**Key Discussion Points:**
- Use `absent()` function: `absent(up{job="my-app"})` fires when the metric disappears
- Or `absent_over_time(metric[5m])` for metrics that may be intermittently absent
- Common scenario: alert fires when all instances of a job are down
- Combine with `up` metric: `up{job="my-app"} == 0` for targets that are scraped but down
- For expected absences (e.g., batch jobs), use `absent_over_time` with longer windows
- Add `for` clause to avoid flapping on brief metric gaps
- Ensure alert labels match Alertmanager routing rules for proper notification

**Sample Answer:** *"I'd use the `absent()` function in an alert rule: `absent(http_requests_total{job='my-app'})`. This fires when Prometheus stops seeing the metric entirely. I'd add a `for: 5m` clause to avoid false alerts from brief scrape gaps. For the `up` metric specifically, I'd use `up{job='my-app'} == 0` as a simpler alternative."*

{{< /qa >}}
{{< qa num="14" q="Your SRE team wants to be alerted when any server's disk is 85% full. You have 200 servers. How do you set this up at scale?" level="intermediate" >}}
**Ans:**

**Key Discussion Points:**
- Deploy Node Exporter on all servers via Ansible/Chef/Puppet or as a DaemonSet in Kubernetes
- Key disk metric: `(1 - node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100`
- Alert rule: `node_filesystem_avail_bytes{fstype!~"tmpfs|devtmpfs"} / node_filesystem_size_bytes < 0.15`
- Group by `instance` and `mountpoint` for precise identification
- Use file-based service discovery (`file_sd_configs`) to auto-register new servers
- Add `for: 5m` to avoid alerts from momentary spikes
- Include `instance` label in alert message for actionable notifications

**Sample Answer:** *"I'd deploy Node Exporter as a systemd service on all servers using Ansible, with file-based service discovery so Prometheus auto-discovers new hosts. I'd create a single alert rule using `node_filesystem_avail_bytes / node_filesystem_size_bytes < 0.15` grouped by instance and mountpoint, providing exactly which server and partition needs attention."*

{{< /qa >}}
{{< qa num="15" q="Your single Prometheus instance is a SPOF (Single Point of Failure). How do you make it highly available?" level="advanced" >}}
**Ans:**

**Key Discussion Points:**
- Run 2 identical Prometheus instances scraping the same targets (active-active)
- Both instances are independent — no native built-in clustering in Prometheus
- Use Alertmanager cluster mode to deduplicate alerts from both instances
- Alertmanager HA: run 3 instances with `--cluster.peer` flags for gossip protocol
- Use Thanos or Cortex for deduplication of metrics from dual Prometheus instances
- Thanos Query deduplicates using `replica` external label
- Configure identical `external_labels` except for `replica` label to enable dedup

**Sample Answer:** *"I'd run two Prometheus instances with identical configs, differentiated by a `replica` external label. Both scrape all targets and both remote_write to Thanos. The Thanos Querier deduplicates results using the replica label. For alerting, I'd run Alertmanager in cluster mode with 3 nodes to deduplicate and route alerts from both Prometheus instances."*


{{< /qa >}}
{{< qa num="16" q="A Grafana dashboard that worked yesterday is now showing No data for all panels. The data source is Prometheus. What do you investigate?" level="basic" >}}
**Ans:**


**Key Discussion Points:**
- Check data source health: Settings → Data Sources → Test connection
- Verify Grafana can reach Prometheus (network, URL, port)
- Check Prometheus is running and healthy: `/healthz` endpoint
- Review the time range in Grafana — data may exist but time range is wrong
- Inspect the panel query directly in Query Inspector for errors
- Check if Prometheus metrics were renamed or labels changed
- Verify Grafana service account/user has query permissions
- Check Prometheus retention — data older than retention period disappears

**Sample Answer:** *"I'd start by testing the Prometheus data source connection in Grafana settings. Then I'd open the Query Inspector on a failing panel to see the raw query and any error messages. I'd also verify the dashboard time range matches when data exists and check whether any metric names or labels changed recently."*


{{< /qa >}}
{{< qa num="17" q="You need a Grafana alert that fires only when BOTH CPU > 80% AND Memory > 90% simultaneously on the same host. How do you configure this?" level="advanced" >}}
**Ans:**


**Key Discussion Points:**
- Use Grafana Unified Alerting (Grafana 8+) with multi-condition support
- Define two queries (A: CPU metric, B: Memory metric) in the alert rule
- Use `Reduce` expressions to get scalar values from each query
- Use `Math` expression: `$A > 80 && $B > 90` to combine conditions
- Ensure both queries use the same `instance` label for correlation
- Set evaluation interval and pending period to avoid flapping
- Route to appropriate contact point via notification policy

**Sample Answer:** *"In Grafana Unified Alerting, I'd define two queries — one for CPU and one for Memory — both labeled by instance. I'd add Reduce expressions to get current values, then a Math expression combining them: `$cpuReduce > 80 && $memReduce > 90`. This fires only when both thresholds are breached simultaneously on the same host."*

{{< /qa >}}
{{< qa num="18" q="You need to give 5 different teams access to Grafana but ensure Team A cannot see Team B's dashboards or data. How do you implement this?" level="advanced" >}}
**Ans:**

**Key Discussion Points:**
- Use Grafana Organizations to completely isolate teams
- Each org has its own data sources, dashboards, users, and teams
- Create separate org per team; assign users with appropriate role (Viewer/Editor/Admin)
- Configure data source per org pointing to team-specific Prometheus or namespace
- Grafana Enterprise: use RBAC and Data Source permissions for finer-grained control
- Use SSO (OAuth/SAML) with `auto_assign_org` to auto-provision users to correct org
- For shared infrastructure teams, use a separate "Platform" org with cross-team dashboards

**Sample Answer:** *"I'd create a separate Grafana Organization for each team. Each org gets its own data sources configured to query only team-relevant data (e.g., namespace-scoped Prometheus queries). Users are assigned to their org via SSO group mapping. Teams cannot see other orgs' dashboards since organizations are fully isolated in Grafana."*

{{< /qa >}}

{{< qa num="19" q="You have a dashboard that hardcodes a specific service name in every panel query. The team wants one dashboard that works for all 20 microservices. How do you redesign it?" level="intermediate" >}}
**Ans:**


**Key Discussion Points:**
- Create a Grafana dashboard variable `$service` of type `Query`
- Query: `label_values(http_requests_total, service)` to dynamically populate dropdown
- Replace hardcoded service name in all panel queries with `$service`
- Enable multi-value and "All" option for selecting multiple services simultaneously
- Add additional variables for `$environment`, `$cluster` for further filtering
- Chain variables using `$service` in the query of dependent variables
- Use `__text` and `__value` for display vs query value mapping

**Sample Answer:** *"I'd add a template variable called `service` using `label_values(http_requests_total, service)` to dynamically pull all service names from Prometheus. Then I'd replace the hardcoded service name in every panel query with `$service`. Users can select any service from the dropdown, or select 'All' to see aggregated metrics across all services."*
{{< /qa >}}

{{< qa num="20" q="You want to view application logs alongside metrics in Grafana without deploying a full ELK stack. What would you recommend?" level="intermediate" >}}
**Ans:**

**Key Discussion Points:**
- Deploy Grafana Loki for log aggregation (lightweight, label-based, Prometheus-inspired)
- Use Promtail as the log shipping agent on each host/pod (similar to Node Exporter)
- In Kubernetes, deploy Promtail as a DaemonSet to collect all pod logs
- Add Loki as a data source in Grafana for log querying with LogQL
- Use Explore view to correlate Prometheus metrics and Loki logs side by side
- Use `{app="my-service"} |= "ERROR"` LogQL to filter error logs
- Derived fields: link log entries to traces in Tempo using trace IDs

**Sample Answer:** *"I'd deploy Loki + Promtail using the official Helm chart. Promtail runs as a DaemonSet collecting all pod logs and shipping them to Loki with Kubernetes labels. In Grafana, I'd add Loki as a data source and use the Explore view to correlate metrics and logs. This gives us log aggregation without the operational complexity of Elasticsearch."*

{{< /qa >}}
{{< qa num="21" q="Users report intermittent slowness in a microservices application with 15 services. How do you use Grafana Tempo to identify the bottleneck?" level="advanced" >}}
**Ans:**

**Key Discussion Points:**
- Deploy Grafana Tempo as the distributed tracing backend
- Instrument services with OpenTelemetry SDK to emit traces
- Configure services to send spans to Tempo via OTLP protocol
- In Grafana, use Explore → Tempo data source → search by trace ID or service
- Use TraceQL to search: `{duration > 5s && resource.service.name = "payment-service"}`
- Enable Trace to Logs correlation: click a span to jump to Loki logs for that time range
- Use Metrics from Traces (RED metrics) to identify high-latency service spans
- Service graph view shows inter-service dependencies and latency distribution

**Sample Answer:** *"I'd use Grafana Tempo with OpenTelemetry instrumentation. When a user reports slowness, I can pull the trace ID from the request and search Tempo to see the full trace waterfall across all 15 services. The span timeline immediately reveals which service is adding the most latency, down to the individual database query level."*


{{< /qa >}}
{{< qa num="22" q="Your product manager wants key metrics displayed on the company's internal wiki (Confluence). How do you embed live Grafana panels?" level="intermediate" >}}
**Ans:**

**Key Discussion Points:**
- Enable `allow_embedding = true` in Grafana config (`grafana.ini`)
- Use Grafana's panel share → Embed tab to generate `<iframe>` HTML
- For public dashboards: enable anonymous access or use Grafana Public Dashboards feature
- Pass variables via URL parameters in the embed URL: `&var-service=payments`
- For Confluence: use the HTML macro to embed the iframe
- Set `cookie_samesite = none` and `cookie_secure = true` for cross-origin embedding
- Consider Grafana Enterprise's Reporting feature for scheduled PDF reports instead

**Sample Answer:** *"I'd enable embedding in Grafana's config, then use the panel Share dialog to get the iframe embed code. For Confluence, I'd use the HTML macro to paste the iframe. For authentication, I'd configure a read-only Grafana service account or enable Public Dashboards for the specific dashboard. Variables can be pre-filtered via URL parameters."*

{{< /qa >}}
{{< qa num="23" q="After every deployment, engineers spend 20 minutes correlating whether a metric change happened before or after the deploy. How do you automate this?" level="intermediate" >}}
**Ans:**

**Key Discussion Points:**
- Use Grafana Annotations API to programmatically add deployment markers
- In CI/CD pipeline (Jenkins/GitHub Actions), POST to `/api/annotations` after deploy
- Annotation shows as a vertical line on all dashboard panels at deploy time
- Include metadata: service name, version, deployer, commit SHA in annotation text
- Use tags (e.g., `deployment`, `production`) to filter annotations per dashboard
- Configure dashboards to show annotations from a specific tag automatically
- Use Grafana's built-in annotation query with `type: tags` matching `deployment`

**Sample Answer:** *"I'd add a step in our CI/CD pipeline that calls the Grafana Annotations API immediately after deployment, passing the service name, version, and commit hash. This creates a vertical marker on all dashboard panels. Engineers can instantly see exactly when the deploy happened relative to any metric change — no more manual correlation."*

{{< /qa >}}
{{< qa num="24" q="Every time Grafana is redeployed, all dashboards and data sources are lost. How do you make Grafana configuration persistent and version-controlled?" level="advanced" >}}
**Ans:**

**Key Discussion Points:**
- Use Grafana Provisioning: YAML files for data sources, dashboards, alerting
- Mount provisioning configs via ConfigMaps in Kubernetes
- Store dashboard JSON in Git repository; mount via ConfigMap or use Grafana's `dashboardProviders`
- Use `editable: false` to prevent manual edits being lost on restart
- Use `Grafana Operator` (Kubernetes) to manage Grafana resources as CRDs
- Alternatively use Terraform `grafana` provider to manage dashboards as code
- Implement CI pipeline to validate dashboard JSON and apply changes on merge

**Sample Answer:** *"I'd configure Grafana provisioning by mounting data source and dashboard YAML files as Kubernetes ConfigMaps. Dashboard JSON files are stored in Git and synced to the provisioning directory. Any change to a dashboard goes through a PR, gets reviewed, and is automatically applied on the next Grafana restart or via a config reload — full GitOps with audit trail."*

{{< /qa >}}
{{< qa num="25" q="Your team is migrating from legacy Grafana alerting (pre-v8) to Unified Alerting. What are the key considerations and migration steps?" level="advanced" >}}
**Ans:**

**Key Discussion Points:**
- Unified Alerting uses Alertmanager natively, replacing legacy notification channels
- Migrate notification channels → Contact Points in Unified Alerting
- Recreate routing logic using Notification Policies with label matchers
- Legacy alerts are panel-based; Unified alerts are standalone with folder organization
- Use Grafana's built-in migration tool: `grafana-cli admin migrate-alerts`
- Test in staging: run both legacy and unified alerting in parallel temporarily
- Update alert rules to use multi-dimensional alerting (fires per label set, not just one alert)
- Silences and inhibition rules now managed via Alertmanager UI in Grafana

**Sample Answer:** *"I'd start by mapping our legacy notification channels to Unified Alerting Contact Points — Slack, PagerDuty, email. Then I'd recreate routing rules as Notification Policies using label matchers. I'd use the built-in migration tool to convert existing panel alerts and test everything in staging before cutting over production. Unified Alerting's multi-dimensional alerts also mean fewer rules to manage overall."*

{{< /qa >}}
{{< qa num="26" q="A Grafana dashboard with 40 panels takes 3 minutes to load, causing frustration. How do you optimize it?" level="advanced" >}}
**Ans:**

**Key Discussion Points:**
- Reduce panel count: merge related panels or use table panels for multiple metrics
- Use recording rules in Prometheus to pre-compute expensive queries
- Increase Prometheus query step to reduce data point resolution for long time ranges
- Enable Grafana query caching (Enterprise) or use Prometheus response caching
- Split the dashboard into multiple focused dashboards with drill-down links
- Use `$__interval` variable instead of hardcoded step to auto-adjust resolution
- Avoid high-cardinality label selectors that force full TSDB scans
- Enable lazy loading: use Grafana's row collapse feature to defer loading hidden panels

**Sample Answer:** *"I'd audit each panel's query with the Query Inspector to find the slowest ones and convert them to recording rules. For the dashboard structure, I'd split it into logical sub-dashboards with drill-down links, reducing panels per page. I'd also ensure all queries use `$__interval` for auto-adjusted resolution and enable row collapsing so only visible panels load initially."*

{{< /qa >}}
{{< qa num="27" q="You want developers to view dashboards but not edit them, while SREs can create/edit dashboards but not manage users. How do you configure this?" level="intermediate" >}}
**Ans:**

**Key Discussion Points:**
- Use Grafana built-in roles: Viewer (read-only), Editor (create/edit), Admin (full access)
- Assign developers as `Viewer` role at org level
- Assign SREs as `Editor` role — can create dashboards but cannot manage users
- Use Teams to group users and assign folder-level permissions
- Give SREs `Admin` on specific dashboard folders but `Viewer` on others
- Grafana Enterprise RBAC: create custom roles with fine-grained permissions
- Integrate with SSO groups (LDAP/OAuth) to auto-assign roles based on group membership

**Sample Answer:** *"I'd assign developers the Viewer role at the organization level, granting read-only access to all dashboards. SREs would get the Editor role, allowing them to create and modify dashboards but not manage users. I'd use Teams and folder-level permissions so SREs have Editor access only to their team's folders, keeping other teams' dashboards read-only for everyone."*

{{< /qa >}}
{{< qa num="28" q="You want your SREs to be able to click on a metric spike in Grafana and immediately jump to the relevant logs and traces for that time window. How do you implement this?" level="advanced" >}}
**Ans:**

**Key Discussion Points:**
- Enable Grafana's Exemplars feature: Prometheus stores trace IDs alongside metrics
- Configure `exemplars` in Prometheus scrape config with `traceIDLabelName`
- In Grafana, link Prometheus data source to Tempo data source via `traceToLogs`
- Clicking an exemplar on a metric graph opens the corresponding trace in Tempo
- Configure `Derived Fields` in Loki data source to extract trace IDs from log lines
- Clicking a trace ID in Loki logs opens the full trace in Tempo
- Use consistent trace ID format (W3C TraceContext) across all three pillars

**Sample Answer:** *"I'd implement the three pillars of observability with Grafana's native correlation features. Prometheus exemplars carry trace IDs with metric data points — clicking a spike shows the actual traces causing it. Loki derived fields extract trace IDs from logs linking to Tempo. Tempo traces link back to logs for the same time window. The result is seamless navigation across metrics, logs, and traces without leaving Grafana."*

{{< /qa >}}
{{< qa num="29" q="Your CTO wants a weekly PDF report every Monday showing the previous week's SLA performance metrics. How do you automate this?" level="intermediate" >}}
**Ans:**

**Key Discussion Points:**
- Grafana Enterprise: use built-in Reporting feature with scheduled PDF generation
- Configure report: select dashboard, time range `last 7 days`, schedule `Every Monday 8 AM`
- Add recipients' email addresses directly in the report config
- For Grafana OSS: use `grafana-image-renderer` plugin + custom Lambda/cron job
- Use Grafana API to render panels as PNG images: `/render/d-solo/{uid}`
- Compile images into a PDF using a script (WeasyPrint, ReportLab) and email via SES/SMTP
- Consider using Grafana's Public Dashboards feature for self-service stakeholder access

**Sample Answer:** *"With Grafana Enterprise, I'd use the built-in Reporting feature to configure a scheduled PDF report of the SLA dashboard with a 'last 7 days' time range, set to send every Monday morning. For OSS Grafana, I'd build a Lambda function that uses the Grafana Image Renderer API to capture panel screenshots, compiles them into a PDF, and sends it via SES."*

{{< /qa >}}
{{< qa num="30" q="Your team needs a custom Grafana panel that displays a real-time topology map of your microservices with live health status. No existing plugin meets your needs. How do you approach building a custom plugin?" level="advanced" >}}
**Ans:**

**Key Discussion Points:**
- Use Grafana Plugin SDK for React to scaffold a new panel plugin: `npx @grafana/create-plugin`
- Plugin types: Panel (visualization), Data Source (new backend), App (collection of panels)
- Use D3.js or React Flow inside the panel for interactive topology visualization
- Query multiple data sources in the plugin using Grafana's `useQuery` hook
- Map service health based on Prometheus `up` metric colors (green/red)
- Use Grafana's `PanelProps` interface to receive data from configured queries
- Sign the plugin for production use: `npx @grafana/sign-plugin`
- Deploy as a custom plugin volume mount in Kubernetes Grafana deployment

**Sample Answer:** *"I'd scaffold a new Grafana panel plugin using the official create-plugin CLI. Using React and React Flow library, I'd build an interactive topology graph. The panel would query Prometheus via configured data sources to get service health status and dynamically color nodes. Once built and signed, it's deployed as a custom plugin volume in our Grafana Kubernetes deployment."*

{{< /qa >}}


## 📝 Summary

| Tool | Key Topics Covered |
|------|-------------------|
| **AWS CloudWatch** | Alarms, Logs, Synthetics, Metric Math, Anomaly Detection, Cross-Account, Container Insights |
| **Prometheus** | PromQL, Recording Rules, Alertmanager, Federation, Thanos, High Cardinality, HA Setup |
| **Grafana** | Unified Alerting, Provisioning, Loki, Tempo, RBAC, Variables, Correlation, Plugins |

---

*Happy interviewing! ⭐ Star this repo if you found it helpful.*
