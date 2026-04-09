---
title: "ALB & NLB Interview Questions & Answers (2026)"
description: "65+ AWS Application Load Balancer and Network Load Balancer interview questions covering routing, health checks, SSL, security, monitoring, and common errors — Basic to Advanced."
date: 2026-04-09
author: "DB"
tags: ["aws", "alb", "nlb", "load-balancer", "interview", "networking"]
tool: "aws"
level: "All Levels"
question_count: 65
---

<div class="qa-list">

## 🟢 Basic

{{< qa num="1" q="What is an Application Load Balancer (ALB)?" level="basic" >}}
An Application Load Balancer (ALB) is an AWS managed load balancing service that operates at **Layer 7 (Application Layer)** of the OSI model. It intelligently distributes incoming HTTP/HTTPS traffic to backend targets such as EC2 instances, containers, Lambda functions, or IP addresses based on request content — including URL path, hostname, headers, query strings, and more.

ALB is best suited for:
- Microservices and container-based architectures
- HTTP/HTTPS-based APIs
- Applications requiring content-based routing
- Use cases needing WebSocket or HTTP/2 support
{{< /qa >}}

{{< qa num="2" q="What OSI layer does ALB operate on?" level="basic" >}}
ALB operates at **Layer 7 — the Application Layer** of the OSI model. This means it can inspect and make routing decisions based on the actual content of the HTTP/HTTPS request — including headers, host names, paths, methods, query parameters, and cookies.
{{< /qa >}}

{{< qa num="3" q="What protocols does ALB support?" level="basic" >}}
- **HTTP** (port 80 by default)
- **HTTPS** (port 443 by default, with TLS termination)
- **WebSocket** (ws://, wss://)
- **HTTP/2** (between client and ALB; ALB downgrades to HTTP/1.1 to targets)
- **gRPC** (ALB natively supports gRPC routing as of 2020)
{{< /qa >}}

{{< qa num="4" q="What are the key components of ALB?" level="basic" >}}
| Component | Description |
|-----------|-------------|
| **Load Balancer** | The entry point that receives traffic |
| **Listener** | Checks for connection requests on a protocol/port |
| **Listener Rules** | Defines routing conditions and actions |
| **Target Group** | Group of registered targets (EC2, containers, Lambda, IPs) |
| **Targets** | Actual backend endpoints receiving traffic |
| **Health Checks** | Periodic checks to ensure targets are healthy |
{{< /qa >}}

{{< qa num="5" q="What is a Target Group in ALB?" level="basic" >}}
A Target Group is a logical grouping of backend resources to which the ALB routes requests. Each target group specifies:
- **Target type**: `instance`, `ip`, or `lambda`
- **Protocol & Port**: Used to communicate with targets
- **Health check configuration**: How ALB monitors target health
- **Load balancing algorithm**: Round robin or LOR

One target can be registered in multiple target groups. Target groups are the key unit for weighted routing, blue/green deployments, and canary releases.
{{< /qa >}}

{{< qa num="6" q="What target types does ALB support?" level="basic" >}}
| Target Type | Description |
|-------------|-------------|
| `instance` | Routes to EC2 instances by instance ID |
| `ip` | Routes to any IP address (on-premises, ECS tasks) |
| `lambda` | Routes to a Lambda function |

You cannot mix target types within a single target group.
{{< /qa >}}

{{< qa num="7" q="What is a Listener in ALB?" level="basic" >}}
A Listener is a process that checks for incoming connection requests on a specified **protocol and port**. An ALB can have multiple listeners (e.g., one on port 80 for HTTP and one on port 443 for HTTPS). Each listener has one or more rules that determine how to route requests to target groups.
{{< /qa >}}

{{< qa num="8" q="What are Listener Rules?" level="basic" >}}
Listener Rules define **conditions** and **actions**:

**Conditions** can match on: Host header, path pattern, HTTP method, query string parameter, source IP CIDR, and HTTP headers.

**Actions** can:
- `forward` — Route to one or more target groups (with weights)
- `redirect` — Redirect to a URL (301/302)
- `fixed-response` — Return a custom HTTP response
- `authenticate-cognito` — Authenticate via Cognito
- `authenticate-oidc` — Authenticate via OIDC

Rules are evaluated in **priority order** (lowest number = highest priority). The **default rule** has no conditions and acts as a catch-all.
{{< /qa >}}

{{< qa num="9" q="What is a Network Load Balancer (NLB)?" level="basic" >}}
A Network Load Balancer (NLB) is an AWS load balancing service that operates at **Layer 4 (Transport Layer)** of the OSI model. It handles TCP, UDP, and TLS traffic with **ultra-high performance and ultra-low latency**. NLB is designed for extreme performance — capable of handling millions of requests per second while maintaining static IP addresses per AZ.

NLB is best suited for:
- Real-time gaming, VoIP, IoT
- Financial trading platforms requiring microsecond latency
- Applications requiring static IPs
- TCP/UDP protocols (not HTTP-specific)
{{< /qa >}}

{{< qa num="10" q="What OSI layer does NLB operate on?" level="basic" >}}
NLB operates at **Layer 4 — the Transport Layer** of the OSI model. It makes routing decisions based on IP protocol (TCP, UDP, TLS), source/destination IP address, and source/destination port.

NLB does **not** inspect HTTP headers, cookies, or paths — it routes based on connection-level information only.
{{< /qa >}}

{{< qa num="11" q="What protocols does NLB support?" level="basic" >}}
- **TCP** — Standard TCP connections
- **UDP** — User Datagram Protocol (for DNS, gaming, streaming)
- **TLS** — TLS termination at the load balancer
- **TCP_UDP** — Same port for both TCP and UDP traffic
{{< /qa >}}

{{< qa num="12" q="What are the key components of NLB?" level="basic" >}}
| Component | Description |
|-----------|-------------|
| **Load Balancer** | Entry point with one static IP per AZ |
| **Listener** | Listens on a protocol and port |
| **Target Group** | Backend resources (instances, IPs, ALB) |
| **Elastic IPs** | Optional static IPs per AZ |
| **Health Checks** | TCP, HTTP, HTTPS, or custom checks |
{{< /qa >}}

{{< qa num="13" q="What is the difference between ALB and NLB?" level="basic" >}}
| Feature | ALB | NLB |
|---------|-----|-----|
| OSI Layer | Layer 7 (Application) | Layer 4 (Transport) |
| Protocols | HTTP, HTTPS, WebSocket, gRPC | TCP, UDP, TLS |
| Routing | Content-based (path, host, headers) | Flow hash (IP, port) |
| Static IP | ❌ No | ✅ Yes (per AZ) |
| Performance | High | Ultra-high (millions RPS) |
| WAF Support | ✅ Yes | ❌ No |
| Auth Offload | ✅ Yes (Cognito/OIDC) | ❌ No |
| Lambda Target | ✅ Yes | ❌ No |
| PrivateLink | ❌ No | ✅ Yes |
| Source IP | Via X-Forwarded-For | Preserved natively |
| Cross-zone LB | Enabled (free) | Disabled (charged when on) |
{{< /qa >}}

## 🟡 Intermediate

{{< qa num="1" q="What routing algorithms does ALB use?" level="intermediate" >}}
| Algorithm | Description |
|-----------|-------------|
| **Round Robin** (default) | Distributes requests sequentially across targets |
| **Least Outstanding Requests (LOR)** | Routes to the target with fewest active requests |
| **Weighted Random** (for target groups) | Routes based on assigned weights |

LOR is ideal when targets have varying request processing times (e.g., mixed EC2 sizes or Lambda).
{{< /qa >}}

{{< qa num="2" q="What is content-based routing in ALB?" level="intermediate" >}}
Content-based routing means ALB inspects the **content of the HTTP request** — such as URL path, host header, HTTP method, query strings, or custom headers — to decide which target group receives the request. This allows a single ALB to serve multiple microservices:

- `example.com/api/*` → API Target Group
- `example.com/static/*` → CDN/S3 Target Group
- `images.example.com` → Image Service Target Group
{{< /qa >}}

{{< qa num="3" q="What is host-based routing?" level="intermediate" >}}
Host-based routing routes requests based on the **HTTP `Host` header** (i.e., the domain/subdomain). This allows one ALB to serve multiple domains:

```
api.example.com      → API Target Group
admin.example.com    → Admin Target Group
www.example.com      → Frontend Target Group
```

This is useful for multi-tenant applications and microservices architectures.
{{< /qa >}}

{{< qa num="4" q="What is path-based routing?" level="intermediate" >}}
Path-based routing routes requests based on the **URL path**:

```
example.com/api/*      → API backend
example.com/images/*   → Image service
example.com/auth/*     → Auth service
example.com/           → Frontend app
```

Rules use glob-style patterns: `/api/*` matches `/api/users`, `/api/orders`, etc.
{{< /qa >}}

{{< qa num="5" q="What is weighted target group routing?" level="intermediate" >}}
Weighted target group routing allows a single listener rule to **forward traffic to multiple target groups with defined weights**. This is commonly used for:
- **Canary releases**: Send 5% traffic to new version, 95% to old
- **Blue/green deployments**: Gradually shift traffic
- **A/B testing**: Split traffic between variants

```
Target Group A (v1): weight 90  → 90% of traffic
Target Group B (v2): weight 10  → 10% of traffic
```
{{< /qa >}}

{{< qa num="6" q="How does ALB handle sticky sessions?" level="intermediate" >}}
Sticky sessions (session affinity) ensure a client always reaches the **same target** within a target group. ALB supports two types:
1. **Duration-based stickiness**: ALB generates a cookie (`AWSALB`) that expires after a set time
2. **Application-based stickiness**: ALB uses a cookie generated by your application

Sticky sessions are configured at the **target group** level and are useful for stateful applications that store session data in memory.

| Feature | Duration-Based | Application-Based |
|---------|---------------|-------------------|
| Cookie generated by | ALB (`AWSALB`) | Your application |
| Cookie forwarded to target | No | Yes |
| Expiry control | ALB-managed | App-managed |
| Use case | Simple stateful apps | Apps with custom session cookies |
{{< /qa >}}

{{< qa num="7" q="Does ALB support WebSockets and HTTP/2?" level="intermediate" >}}
**WebSockets:** Yes. ALB natively supports WebSocket connections. When a client upgrades an HTTP connection to WebSocket (`Upgrade: websocket` header), ALB maintains the persistent connection and routes it to the appropriate target. Both `ws://` (unencrypted) and `wss://` (encrypted) are supported.

**HTTP/2:** Yes. ALB supports HTTP/2 between the client and the load balancer. However, ALB communicates with backend targets over HTTP/1.1 by default. HTTP/2 benefits include multiplexing, header compression, and reduced latency.
{{< /qa >}}

{{< qa num="21" q="What is ALB redirect action and fixed-response action?" level="intermediate" >}}
**Redirect action:** Returns an HTTP redirect response to the client. Common use cases:
- **HTTP → HTTPS redirect**: Port 80 rule redirects to `https://#{host}:443/#{path}?#{query}`
- **www to non-www redirect**
- Status code: `301` (Permanent) or `302` (Temporary)

**Fixed-response action:** Returns a custom HTTP response directly without forwarding to any target. Use cases:
- Return `200 OK` for health check endpoints
- Return `403 Forbidden` for blocked paths
- Return `404` for unmatched routes

```json
{
  "Type": "fixed-response",
  "FixedResponseConfig": {
    "StatusCode": "200",
    "ContentType": "text/plain",
    "MessageBody": "OK"
  }
}
```
{{< /qa >}}

{{< qa num="8" q="How does ALB handle SSL/TLS termination?" level="intermediate" >}}
ALB performs **SSL/TLS termination at the load balancer** level. The client connects to ALB over HTTPS; ALB decrypts the traffic and forwards to targets over HTTP or HTTPS. This:
- Offloads crypto processing from backend servers
- Centralizes certificate management (via ACM)
- Allows ALB to inspect request content for routing decisions

You can attach **multiple certificates** to a single HTTPS listener using **SNI** (Server Name Indication), which allows ALB to host multiple SSL certificates on a single listener. The client includes the target hostname in the TLS handshake, and ALB selects the matching certificate automatically. ALB supports up to **25 certificates per listener**.
{{< /qa >}}

{{< qa num="23" q="How does ALB integrate with AWS WAF?" level="intermediate" >}}
ALB integrates natively with **AWS WAF (Web Application Firewall)**. You can attach a WAF Web ACL to an ALB to:
- Block/allow requests based on IP, country, URI, headers
- Protect against SQL injection, XSS, DDoS
- Rate-limit requests
- Apply managed AWS rule sets

WAF rules are evaluated **before** ALB routes the request. If WAF blocks a request, it returns a configurable response (default: `403`).
{{< /qa >}}

{{< qa num="9" q="How do health checks work in ALB and what are the parameters?" level="intermediate" >}}
ALB periodically sends requests to each registered target to determine if it's healthy. Targets that fail health checks are removed from rotation. Health checks are configured at the **target group** level.

A target is marked **healthy** after N consecutive successful responses, and **unhealthy** after N consecutive failures.

| Parameter | Description | Default |
|-----------|-------------|---------|
| Protocol | HTTP or HTTPS | HTTP |
| Port | Port to check | `traffic-port` |
| Path | URL path to check | `/` |
| Healthy threshold | Consecutive successes needed | 5 |
| Unhealthy threshold | Consecutive failures before marking unhealthy | 2 |
| Timeout | Time to wait for response | 5 seconds |
| Interval | Time between checks | 30 seconds |
| Success codes | HTTP codes for healthy | `200` |
{{< /qa >}}

{{< qa num="10" q="What CloudWatch metrics are available for ALB?" level="intermediate" >}}
| Metric | Description |
|--------|-------------|
| `RequestCount` | Total requests processed |
| `TargetResponseTime` | Time to receive response from target |
| `HTTPCode_Target_2XX_Count` | 2xx responses from targets |
| `HTTPCode_Target_4XX_Count` | 4xx responses from targets |
| `HTTPCode_Target_5XX_Count` | 5xx responses from targets |
| `HTTPCode_ELB_5XX_Count` | 5xx errors generated by ALB itself |
| `HealthyHostCount` | Number of healthy targets |
| `UnHealthyHostCount` | Number of unhealthy targets |
| `ActiveConnectionCount` | Active TCP connections |
| `ProcessedBytes` | Total bytes processed |
{{< /qa >}}

{{< qa num="11" q="What target types does NLB support?" level="intermediate" >}}
| Target Type | Description |
|-------------|-------------|
| `instance` | Route to EC2 by instance ID |
| `ip` | Route to any IP (on-prem, containers) |
| `alb` | Route to an ALB (NLB → ALB chaining) |

The `alb` target type (added in 2021) allows NLB to front an ALB, combining static IPs with HTTP-level routing.
{{< /qa >}}

{{< qa num="12" q="Does NLB support security groups?" level="intermediate" >}}
**Yes, as of 2023**, NLB supports security groups. Previously, NLB passed source IPs directly to targets, requiring targets to manage their own security group rules. With SG support on NLB:
- You can attach security groups to NLB itself
- Target security groups can reference the NLB's SG
- Provides centralized inbound/outbound filtering

For older NLBs without SG support, you must add NLB node IP CIDRs to target security groups.
{{< /qa >}}

{{< qa num="13" q="What is a static IP in NLB?" level="intermediate" >}}
NLB provides a **single static IP address per Availability Zone**. You can optionally assign **Elastic IP addresses (EIPs)** to NLB nodes. Benefits:
- Clients can whitelist specific IPs (firewall rules)
- No DNS TTL concerns — IP never changes
- Essential for compliance and on-premise integrations

ALB does not support static IPs (only a static DNS name).
{{< /qa >}}

{{< qa num="14" q="What routing algorithm does NLB use?" level="intermediate" >}}
NLB uses a **flow hash algorithm** based on:
- Protocol
- Source IP and port
- Destination IP and port
- TCP sequence number (for TCP)

This ensures that **all packets in a connection/flow go to the same target** (maintaining TCP state). It is essentially source-IP affinity at the flow level.
{{< /qa >}}

{{< qa num="15" q="What is source IP preservation in NLB?" level="intermediate" >}}
NLB **preserves the client's source IP address** by default when forwarding traffic to targets (for TCP and UDP). This means:
- Targets see the actual client IP in the connection (not the NLB IP)
- No need to check `X-Forwarded-For` headers (unlike ALB)
- Target security groups must allow traffic from client IP ranges

When using **TLS termination**, the source IP is still preserved downstream.
{{< /qa >}}

{{< qa num="16" q="Does NLB support sticky sessions?" level="intermediate" >}}
Yes. NLB supports **source IP-based sticky sessions** (not cookie-based like ALB). Connections from the same source IP are consistently routed to the same target for the duration of the stickiness window.

- For TCP/TLS: Stickiness can be enabled per target group
- Stickiness duration: 1–604800 seconds (up to 7 days)
- This differs from ALB where stickiness is cookie-based
{{< /qa >}}

{{< qa num="17" q="How do health checks work in NLB and what protocols can they use?" level="intermediate" >}}
NLB sends periodic health checks to targets. Unlike ALB (which only does HTTP/HTTPS), NLB supports multiple health check protocols. A target must pass N consecutive checks to be considered healthy. NLB removes unhealthy targets from rotation within **10 seconds** of failure (fast failover).

| Protocol | Description |
|----------|-------------|
| TCP | Opens and closes a connection; success = connection established |
| HTTP | Sends HTTP request to specified path; checks status code |
| HTTPS | Same as HTTP but over TLS |

For **UDP targets**, NLB uses TCP health checks (UDP itself is connectionless).
{{< /qa >}}

{{< qa num="18" q="What CloudWatch metrics are available for NLB?" level="intermediate" >}}
| Metric | Description |
|--------|-------------|
| `ActiveFlowCount` | Active TCP/UDP flows |
| `NewFlowCount` | New TCP/UDP flows per second |
| `ProcessedBytes` | Total bytes processed |
| `HealthyHostCount` | Number of healthy targets |
| `UnHealthyHostCount` | Number of unhealthy targets |
| `TCP_Client_Reset_Count` | Client-sent RST packets |
| `TCP_Target_Reset_Count` | Target-sent RST packets |
| `TCP_ELB_Reset_Count` | RST packets sent by NLB |
| `ConsumedLCUs` | Load balancer capacity units |
{{< /qa >}}

{{< qa num="19" q="What is cross-zone load balancing and how does it differ between ALB and NLB?" level="intermediate" >}}
Cross-zone load balancing allows a load balancer to **distribute traffic evenly across all registered targets in all enabled Availability Zones**, regardless of which AZ received the incoming request.

- **ALB**: Cross-zone load balancing is **enabled by default** and there is **no charge** for inter-AZ data transfer
- **NLB**: Cross-zone load balancing is **disabled by default**. When enabled, **inter-AZ data transfer is charged**

Without cross-zone balancing, each LB node only routes to targets in its own AZ, potentially creating imbalance if target counts differ per AZ.
{{< /qa >}}

{{< qa num="35" q="What is ALB request tracing?" level="intermediate" >}}
ALB automatically adds an **`X-Amzn-Trace-Id`** header to each incoming request. This trace ID can be used to:
- Correlate ALB access logs with application logs
- Trace requests through distributed systems (with AWS X-Ray)
- Debug latency and error issues

If the request already has a trace ID, ALB updates it. The format is: `Root=1-<timestamp>-<unique-id>`.
{{< /qa >}}

{{< qa num="20" q="What ALB access logs contain?" level="intermediate" >}}
ALB access logs (stored in S3) record detailed information per request:
- Timestamp, client IP, target IP
- Request method, URL, protocol, HTTP version
- Response status code (ALB and target)
- Bytes sent/received
- Request processing time, target processing time, response time
- SSL cipher and protocol
- `X-Amzn-Trace-Id` (request trace ID)
- User agent, redirect URL

Access logs are stored in **S3** and are useful for debugging, compliance, and traffic analysis.
{{< /qa >}}

## 🔴 Advanced

{{< qa num="1" q="What is ALB Least Outstanding Requests (LOR) routing?" level="advanced" >}}
LOR (Least Outstanding Requests) is an optional routing algorithm that sends new requests to the target with the **fewest in-progress requests**. This is beneficial when:
- Targets have varying capacity (e.g., different EC2 sizes)
- Requests have highly variable processing times
- You want to prevent overwhelming slower targets

Default is Round Robin; LOR is set per target group. LOR is especially valuable when using mixed instance sizes in a target group or routing to Lambda functions with varying execution times.
{{< /qa >}}

{{< qa num="2" q="What is connection draining / deregistration delay in ALB?" level="advanced" >}}
Connection draining (also called **deregistration delay**) ensures that ALB continues routing **in-flight requests** to a target that is being deregistered or marked unhealthy. During the delay period:
- No new requests are sent to the draining target
- Existing connections are allowed to complete

Default: **300 seconds**. Range: 0–3600 seconds. Set to 0 to deregister immediately — useful for Lambda targets or fast deployments where in-flight requests complete within milliseconds.
{{< /qa >}}

{{< qa num="3" q="How does ALB integrate with ECS and Lambda?" level="advanced" >}}
**ECS integration:** ALB integrates tightly with Amazon ECS:
- ECS automatically registers/deregisters task IPs with target groups
- Supports **dynamic port mapping** — multiple containers on the same EC2 host can use different ports
- When using Fargate, target type must be `ip`
- ECS service rolling updates work with ALB to drain connections before stopping old tasks

**Lambda integration:** ALB can invoke Lambda functions as targets. When a request matches a rule pointing to a Lambda target group:
- ALB serializes the HTTP request into a **JSON event** and invokes the Lambda
- Lambda processes and returns a JSON response
- ALB deserializes the response back to HTTP

This enables serverless HTTP applications without API Gateway. Lambda target groups don't support health checks in the traditional sense — Lambda must respond correctly.
{{< /qa >}}

{{< qa num="4" q="What is the difference between internal and internet-facing ALB?" level="advanced" >}}
| Feature | Internet-Facing | Internal |
|---------|----------------|----------|
| DNS resolution | Public IP | Private IP |
| Accessible from | Internet | VPC / connected networks |
| Subnet requirement | Public (IGW attached) | Private |
| Use case | Public-facing web apps | Internal microservices |

An **internal ALB** is deployed in private subnets and is only accessible from within the VPC (or connected networks via VPN/Direct Connect). Use cases: internal microservices communication, internal API gateways, tier-to-tier routing in multi-tier architectures.
{{< /qa >}}

{{< qa num="5" q="What is idle timeout in ALB and how does it affect connections?" level="advanced" >}}
Idle timeout is the time ALB waits before closing an **idle TCP connection** (no data sent/received). Default is **60 seconds**, configurable up to 4,000 seconds.

**Critical considerations:**
- For HTTP keep-alive connections, ensure your backend server's keep-alive timeout is **greater** than ALB's idle timeout to avoid race conditions. If the backend closes the connection at exactly 60 seconds and ALB is still expecting it to be open, you get 502 errors.
- For WebSockets, set idle timeout high enough to accommodate long-lived connections
- For long-running API calls or file uploads, increase the timeout to match your maximum expected response time
{{< /qa >}}

{{< qa num="6" q="What is desync mitigation mode in ALB?" level="advanced" >}}
Desync mitigation mode protects against **HTTP desync attacks** (HTTP request smuggling) by controlling how ALB handles non-conformant HTTP requests. Three modes:

- **Strictest**: Closes connections for any non-RFC-compliant requests — maximum security, may break legacy clients
- **Defensive** (default): Closes connections for most attacks but allows some minor deviations — balanced security
- **Monitor**: Logs issues without blocking — use for testing rule impact before enabling stricter modes

HTTP desync attacks exploit inconsistencies in how HTTP requests are parsed between the load balancer and backend servers, allowing attackers to inject requests into other users' sessions.
{{< /qa >}}

{{< qa num="7" q="What are the limits of ALB?" level="advanced" >}}
| Resource | Default Limit |
|----------|--------------|
| Load balancers per region | 50 |
| Listeners per load balancer | 50 |
| Rules per listener | 100 (up to 1,000 with quota increase) |
| Conditions per rule | 5 |
| Target groups per region | 3,000 |
| Targets per target group | 1,000 |
| Certificates per listener | 25 |

Each rule can have up to **5 conditions** and **5 actions**. A single ALB can have up to **50 listeners** by default.
{{< /qa >}}

{{< qa num="8" q="What is the zonal DNS name in NLB and when would you use it?" level="advanced" >}}
Each NLB has a **regional DNS name** (e.g., `my-nlb-xxxx.elb.amazonaws.com`) and **zonal DNS names** (e.g., `us-east-1a.my-nlb-xxxx.elb.amazonaws.com`) for each AZ. Zonal DNS names resolve to the NLB node IP in a specific AZ.

**Use cases:**
- Reduce cross-AZ data transfer costs by pinning clients to a specific AZ node
- Route clients in the same AZ to the local NLB node to improve latency
- Improve latency for latency-sensitive applications like gaming or trading
{{< /qa >}}

{{< qa num="9" q="Can NLB be used with AWS PrivateLink?" level="advanced" >}}
Yes. NLB is the **backbone of AWS PrivateLink**. Service providers expose their services via NLB, and service consumers access them through VPC Endpoints. This allows:
- Exposing services to other AWS accounts/VPCs without VPC peering
- No traffic traverses the public internet
- Highly secure, scalable service exposure model

This is the standard pattern for SaaS providers on AWS — expose your service via NLB, and customers subscribe via PrivateLink without any network complexity.
{{< /qa >}}

{{< qa num="10" q="When would you choose NLB over ALB?" level="advanced" >}}
Choose NLB when you need:

1. **Static IP addresses** — for firewall whitelisting or compliance requirements
2. **Extreme performance** — millions of requests/second, microsecond latency
3. **TCP/UDP protocols** — not HTTP/HTTPS (e.g., MQTT, DNS, custom TCP protocols)
4. **AWS PrivateLink** — exposing services across accounts/VPCs
5. **On-premises integration** — with Direct Connect or VPN using static IPs
6. **Preserve source IP natively** — without header manipulation like X-Forwarded-For
7. **TLS pass-through** — when targets should handle TLS themselves and you need end-to-end encryption
{{< /qa >}}

{{< qa num="11" q="Can NLB handle millions of requests per second?" level="advanced" >}}
Yes. NLB is designed to scale automatically to handle **tens of millions of TCP/UDP flows** per second with ultra-low latency (single-digit milliseconds). It scales instantly without a warm-up period (unlike Classic LB). This makes it ideal for:
- High-frequency trading platforms
- Real-time gaming backends
- Large-scale DNS servers
- IoT telemetry platforms

ALB requires pre-warming for very sudden traffic spikes — you can contact AWS Support to request pre-warming if you expect a large traffic event.
{{< /qa >}}

{{< qa num="12" q="What is UDP load balancing in NLB?" level="advanced" >}}
NLB supports **UDP** protocol (added in 2019). UDP is connectionless, so NLB uses a **flow-based hash** (source IP, source port, destination IP, destination port) to consistently route UDP packets from the same client to the same target.

**Use cases:**
- DNS load balancing (port 53)
- Real-time streaming (RTP, RTSP)
- IoT sensor data
- Network monitoring tools (SNMP, syslog)

Health checks for UDP targets use TCP (since UDP is connectionless and cannot confirm a successful connection the same way TCP can).
{{< /qa >}}

{{< qa num="13" q="What is TLS connection termination vs passthrough in NLB?" level="advanced" >}}
| Mode | Description |
|------|-------------|
| **TLS Termination** | NLB decrypts TLS traffic, forwards plain TCP to targets |
| **TCP Passthrough** | NLB forwards raw TLS bytes to targets; targets handle decryption |

For passthrough, set the **listener protocol to TCP** (not TLS). Targets receive the full TLS handshake and must have valid certificates. Useful when end-to-end encryption is required and targets need the original TLS connection — such as mutual TLS (mTLS) authentication where the target needs to inspect the client certificate.
{{< /qa >}}

{{< qa num="14" q="Can ALB and NLB be chained together?" level="advanced" >}}
Yes. You can use **NLB as a frontend for ALB** by creating a target group of type `alb` in NLB (introduced 2021). This provides:
- **Static IPs** (from NLB) + **Content-based routing** (from ALB)
- Useful when clients need to whitelist IPs but you need HTTP routing rules
- NLB handles TCP forwarding, ALB handles HTTP routing to microservices

Architecture: `Client → NLB (static IP) → ALB (HTTP routing) → Targets`

This is a common pattern when you have existing firewall rules that whitelist specific IPs but also need path-based or host-based routing for your microservices.
{{< /qa >}}

{{< qa num="15" q="What is the difference between NLB and CLB (Classic Load Balancer)?" level="advanced" >}}
| Feature | NLB | CLB |
|---------|-----|-----|
| Architecture | VPC only, AZ-aware | Legacy EC2-Classic |
| Layer | Layer 4 | Layer 4 & 7 (limited) |
| Performance | Extreme | Moderate |
| Static IPs | ✅ Yes | ❌ No |
| UDP | ✅ Yes | ❌ No |
| Multiple ports on one listener | ✅ Yes | ❌ Limited |
| AWS recommendation | ✅ Use NLB/ALB | ❌ Deprecated — migrate away |

CLB is legacy and AWS recommends migrating to ALB or NLB. CLB does not support modern features like path-based routing, Lambda targets, WebSockets, or static IPs.
{{< /qa >}}

{{< qa num="16" q="What are the limits of NLB?" level="advanced" >}}
| Resource | Default Limit |
|----------|--------------|
| Load balancers per region | 50 |
| Listeners per load balancer | 50 |
| Target groups per region | 3,000 |
| Targets per target group | 1,000 |
| Certificates per TLS listener | 25 |
{{< /qa >}}

{{< qa num="17" q="HTTP 502 Bad Gateway — what causes it and how do you fix it?" level="advanced" >}}
**Where:** ALB

**Meaning:** ALB received an **invalid response from the target** or the target closed the connection without responding. ALB itself is running, but the backend is misbehaving.

**Common Causes:**
- Target application crashed or returned malformed HTTP response
- Keep-alive timeout mismatch — backend closed the connection before ALB's idle timeout expired
- Target returned a response ALB couldn't parse (e.g., HTTP/0.9)
- Target is overloaded and not responding properly

**Solutions:**
1. Check application logs on target instances/containers
2. Ensure backend sends valid HTTP responses with proper headers
3. Set backend keep-alive timeout **greater** than ALB idle timeout (60s default)
4. Check if targets are healthy in the target group console
5. Review ALB access logs — look for `target_status_code` field
6. Scale out target group if overloaded
{{< /qa >}}

{{< qa num="18" q="HTTP 503 Service Unavailable — what causes it and how do you fix it?" level="advanced" >}}
**Where:** ALB

**Meaning:** ALB has **no healthy targets** to route the request to. All registered targets are unhealthy or no targets are registered.

**Common Causes:**
- All targets failed health checks
- Target group is empty (no registered targets)
- Health check path is wrong (returning 4xx/5xx)
- Security group not allowing health check traffic from ALB
- New deployment is rolling out and old tasks are draining

**Solutions:**
1. Check `UnHealthyHostCount` metric in CloudWatch
2. Verify health check path returns `200 OK`
3. Ensure security groups allow ALB → target traffic on health check port
4. Check if targets have sufficient capacity (CPU/memory)
5. Register at least one healthy target in the target group
6. Review target logs for startup errors
{{< /qa >}}

{{< qa num="18" q="HTTP 504 Gateway Timeout — what causes it and how do you fix it?" level="advanced" >}}
**Where:** ALB

**Meaning:** ALB forwarded the request to a target, but the target **did not respond within the idle timeout period**.

**Common Causes:**
- Target is processing a slow/long-running request
- Target is under heavy load and can't respond in time
- Application deadlock or blocking operation
- ALB idle timeout (default 60s) too short for the use case

**Solutions:**
1. Increase ALB idle timeout to match max expected response time
2. Optimize slow queries or background jobs on targets
3. Add async processing — respond 202 immediately, process asynchronously
4. Scale out target group to handle load
5. Check for database slow queries causing application delays
6. Review VPC network ACLs for blocking traffic
{{< /qa >}}

{{< qa num="20" q="HTTP 400, 403, and 408 errors in ALB — causes and solutions?" level="advanced" >}}
**HTTP 400 Bad Request:** ALB rejected the request because it was malformed or violated HTTP standards. ALB itself generates this error without forwarding to targets.

Causes: Invalid HTTP headers (non-printable characters), request URI too long (>8KB), duplicate content-length headers, HTTP desync attack attempt.

Fix: Validate and sanitize client requests. Reduce URL/header size. Review ALB desync mitigation mode settings.

---

**HTTP 403 Forbidden:** The request was blocked by **AWS WAF** or a listener rule returned a fixed `403` response.

Causes: WAF rule matched and blocked the request, geographic restriction rule blocking client IP.

Fix: Check AWS WAF logs to identify the triggering rule. Add IP to WAF whitelist if legitimate. Adjust WAF rules or change from BLOCK to COUNT mode for testing.

---

**HTTP 408 Request Timeout:** ALB didn't receive the complete request from the client within the timeout period.

Causes: Slow client connection, large request body being uploaded slowly, client sent partial request headers and stalled.

Fix: Increase ALB idle timeout for upload-heavy applications. Use multipart uploads via S3 pre-signed URLs for large files.
{{< /qa >}}

{{< qa num="21" q="HTTP 460, 463, 464, and 561 errors in ALB — causes and solutions?" level="advanced" >}}
**HTTP 460 Client Closed Connection:** The client closed the connection before ALB could deliver the response. ALB had already received a response from the target but couldn't deliver it.

Causes: Client timeout too short, long-running requests, network instability. Fix: Optimize response times or increase client-side timeout.

---

**HTTP 463 Too Many IP Addresses in X-Forwarded-For:** The `X-Forwarded-For` header contains more than **30 IP addresses** (ALB's limit). ALB rejects the request.

Causes: Long proxy chains before ALB, intermediate proxies not trimming XFF headers. Fix: Trim XFF headers at an upstream proxy before they reach ALB.

---

**HTTP 464 Incompatible Protocol:** Target group protocol is incompatible with the listener protocol. For example, an HTTP listener routing to an HTTPS target group with strict protocol matching.

Fix: Ensure listener and target group protocols are compatible. For HTTPS listeners routing to HTTP targets, set target group protocol to HTTP.

---

**HTTP 561 Unauthorized (ALB Auth):** ALB received an error from the IdP (Identity Provider) during the Cognito/OIDC authentication flow.

Causes: Wrong client ID/secret, redirect URI not registered in IdP, OIDC discovery document unreachable.

Fix: Verify Cognito App Client ID, secret, and callback URLs. Ensure the redirect URI `https://<alb-dns>/oauth2/idpresponse` is registered in IdP.
{{< /qa >}}

{{< qa num="22" q="NLB — Connection Refused and Health Check Failure errors?" level="advanced" >}}
**NLB Connection Refused:** The NLB successfully routes the connection but the **target actively rejects it** (TCP RST).

Causes: Application not listening on the configured port, application crashed, OS-level firewall (iptables) blocking the port, wrong port configured in target group.

Fix: Verify the application is running and listening on the correct port using `ss -tlnp` or `netstat -tlnp`. Check target security groups allow NLB traffic.

---

**NLB Health Check Failure (TCP):** NLB's TCP health check cannot establish a connection to the target.

Causes: Application not started, listening on wrong port, target security group blocking NLB health check IPs, OS firewall rules.

Fix: Ensure app starts on boot (systemd). Verify target security group allows NLB node IPs. Check `HealthyHostCount` and `UnHealthyHostCount` metrics. For NLBs without security groups, add NLB IP ranges to target security group.
{{< /qa >}}

{{< qa num="23" q="NLB — Asymmetric Routing / Hairpinning and Cross-Zone Imbalance issues?" level="advanced" >}}
**Asymmetric Routing / Hairpinning:**
Targets within the same VPC sending traffic to the NLB's IP and receiving responses from a different IP than expected — causing connection drops.

Causes: Target trying to reach itself via NLB (hairpinning), source IP preservation causing return traffic to go through a different path.

Fix: Enable proxy protocol v2 on target group so targets can identify the actual source. Targets should not connect to themselves via NLB — use direct communication instead. Enable client IP preservation = false if source IP is not needed.

---

**Cross-Zone Load Balancing Imbalance:**
Traffic is unevenly distributed because cross-zone load balancing is disabled (NLB default) and one AZ has more traffic or targets than another.

Causes: Cross-zone LB disabled, DNS-based client routing sending most traffic to one AZ, unequal number of targets registered per AZ.

Fix: Enable cross-zone load balancing on NLB (note: incurs inter-AZ data transfer cost). Register an equal number of targets per AZ. Use Auto Scaling to maintain equal target counts across AZs.
{{< /qa >}}

{{< qa num="24" q="SSL/TLS Handshake Failure and Listener Certificate Not Found errors?" level="advanced" >}}
**SSL/TLS Handshake Failure:** The SSL/TLS handshake between the client and ALB/NLB fails — no encrypted connection is established.

Causes: Client presenting a certificate the LB doesn't trust (for mTLS), cipher suite mismatch, TLS protocol version mismatch (client uses TLS 1.0/1.1 when LB requires 1.2+), expired certificate on either side.

Fix: Check ALB security policy — update to support required cipher suites. Use ACM to renew expired certificates. Enable debug logging to capture handshake failure details.

---

**Listener Certificate Not Found:** The load balancer cannot find or access the SSL/TLS certificate configured for a listener, preventing HTTPS/TLS listeners from functioning.

Causes: ACM certificate deleted or in pending validation state, certificate in wrong AWS region, IAM role permissions don't allow LB to access certificate.

Fix: Check ACM console — ensure certificate status is `Issued`. ACM certificates must be in the **same region** as the load balancer. Re-add certificate to listener if accidentally removed.
{{< /qa >}}

</div>
