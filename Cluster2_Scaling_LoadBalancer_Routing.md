# CLUSTER 2: SCALING & LOAD DISTRIBUTION
### Topics: Vertical Scaling | Horizontal Scaling | Load Balancer | Route Routing

---

## TOPIC A: VERTICAL SCALING

# 1. Concept Overview

**Definition:** Vertical scaling (scaling up) means increasing the power of a single machine — more CPU, RAM, disk, or better hardware — instead of adding more machines.

**Why it exists:** It's the simplest way to handle more load initially — just give the existing server a bigger engine.

**Real-world analogy:** Upgrading from a small scooter to a powerful bike to carry more weight, instead of buying a second scooter.

**Where it is used:** Databases (especially early-stage or single-leader systems), monolithic applications, small startups before they need distributed systems.

**Why interviewers ask it:** To check if you understand the simplest scaling option and its hard physical/cost limits before jumping to distributed designs.

---

# 2. Core Concepts

| Concept | Explanation |
|---|---|
| **Scaling Up** | Another name for vertical scaling |
| **Single Point of Failure (SPOF)** | Since it's one machine, if it crashes, everything goes down |
| **Hardware Ceiling** | Physical limit on how much CPU/RAM one machine can have |
| **Downtime during upgrade** | Often requires restart/migration, causing temporary unavailability |
| **Cost curve** | High-end hardware costs grow non-linearly (very expensive at the top end) |
| **Vertical Scaling vs Horizontal Scaling** | Vertical = bigger machine; Horizontal = more machines |

---

# 3. Internal Working

**Simple version:** Your server is struggling → you upgrade its CPU/RAM/SSD → it can now handle more requests/data without changing your code architecture.

**Technical step-by-step:**
1. Monitor server metrics (CPU, memory, disk I/O) and identify bottleneck.
2. Provision a more powerful instance type (e.g., AWS EC2 t3.medium → m5.4xlarge).
3. Migrate data/application to new instance (often requires downtime or careful blue-green cutover).
4. Update DNS/load balancer to point to new instance if IP changes.
5. Decommission old instance.

---

# 4. Architecture / Flow

```
   Before:                     After:
   Client                      Client
     │                           │
   Server (2 vCPU, 4GB RAM)    Server (16 vCPU, 64GB RAM)
     │                           │
   Database                    Database
```
Single node, just "bigger." No change in topology — only capacity.

---

# 5. Advantages
- **Simplicity:** No code changes needed; no distributed system complexity (no data partitioning, no consistency issues).
- **No architectural rework:** Works great for monoliths and relational databases that are hard to distribute.
- **Quick short-term fix:** Easy to apply when traffic spikes are not extreme.

# 6. Disadvantages
- **Hardware ceiling:** There's a maximum amount of CPU/RAM a single machine can have.
- **Single Point of Failure:** If that one beefy machine goes down, the entire system goes down.
- **Expensive at scale:** High-end hardware costs grow much faster than linearly.
- **Downtime risk:** Resizing often requires a restart.

# 7. Trade-offs

| Approach | Best For | Downside |
|---|---|---|
| Vertical Scaling | Simplicity, early-stage apps, relational DBs | Hard ceiling, SPOF, costly at high end |
| Horizontal Scaling | Large-scale, high-availability systems | Complexity (data partitioning, consistency) |

**When to use:** Vertical scaling first for simplicity; switch to horizontal once you hit hardware limits, need high availability, or need geographic distribution.

---

# 8. Real-world Examples
- **Early-stage startups:** Often run their entire app + DB on a single large EC2/RDS instance before scaling out.
- **Traditional RDBMS (before sharding):** Companies scale up Postgres/MySQL instances before adopting read replicas or sharding.
- **Monolithic legacy systems:** Many banks still run mainframes that are scaled vertically due to the complexity of distributing transactional consistency.

---

# 9. Interview Deep Dive

**Common questions:**
- What is vertical scaling and when would you use it?
- What are its limitations?

**Follow-ups:**
- At what point would you switch to horizontal scaling?
- How do you scale up a database without downtime?

**Hidden edge cases:**
- Stateful applications relying on local disk/session data break easily if you try to later switch to horizontal scaling — vertical scaling can mask underlying architecture problems.

**Frequently misunderstood:**
- People think vertical scaling is "bad" — it's not; it's appropriate for many real systems, especially relational databases requiring strong consistency.

---

# 10. Important Terminology

| Term | Definition |
|---|---|
| SPOF | Single Point of Failure |
| Instance Type | Predefined hardware configuration (cloud) |
| Downtime | Period when system is unavailable |
| Hardware Ceiling | Max capacity a single machine can offer |

# 11. Time Complexity / Performance
- No algorithmic complexity — purely hardware-bound.
- Latency improves with faster CPU/RAM/SSD.
- Throughput capped by max hardware specs available.
- Scalability impact: Limited — eventually you WILL hit a ceiling.

# 12. Common Mistakes
- Saying vertical scaling has "no limit."
- Forgetting to mention SPOF risk.
- Not distinguishing it clearly from horizontal scaling in answers.

# 13. Revision Sheet
- Vertical scaling = bigger machine, same architecture
- Simple but has a hardware ceiling and SPOF risk
- Good for early-stage systems and relational DBs
- Cost grows non-linearly at high end

# 14. Memory Tricks
**"Scale UP = Bigger CUP (CPU/RAM/Disk)"** — one machine just gets stronger.

---

# 15. Interview Questions

**Basic (5):**
1. What is vertical scaling?
2. Give a real-world analogy for vertical scaling.
3. What is a hardware ceiling?
4. What is SPOF?
5. When is vertical scaling preferred?

**Medium (5):**
1. What are the limitations of vertical scaling?
2. How do you scale up a database with minimal downtime?
3. Compare vertical vs horizontal scaling cost curves.
4. Why is vertical scaling simpler architecturally?
5. Can vertical scaling solve high-availability needs? Why/why not?

**Advanced (3):**
1. How would you migrate a vertically-scaled monolith to a horizontally-scaled system?
2. What database engines are hardest to scale horizontally and why?
3. How do cloud providers handle live resizing of instances?

**Scenario (2):**
1. Your database CPU is maxed out and queries are slow — vertical or horizontal scaling first? Justify.
2. You're told "just buy a bigger server" by management — what risks would you flag?

---

# 16. Coding Connections
- **Databases:** Increasing instance size for RDS/PostgreSQL before considering replicas/sharding.
- **Operating Systems:** Resource allocation (CPU scheduling, memory management) benefits directly from more hardware.
- **Distributed Systems:** Often the "before" state compared against horizontally-scaled architectures.

---
---

## TOPIC B: HORIZONTAL SCALING

# 1. Concept Overview

**Definition:** Horizontal scaling (scaling out) means adding more machines/servers to handle increased load, rather than making one machine bigger.

**Why it exists:** Single machines have hardware ceilings; horizontal scaling allows near-infinite growth by distributing load across many commodity machines.

**Real-world analogy:** Instead of one super-fast cashier, you open more checkout counters at a supermarket to handle more customers simultaneously.

**Where it is used:** Web servers, microservices, NoSQL databases, distributed caches, almost all modern large-scale systems (Netflix, Amazon, Google).

**Why interviewers ask it:** It's central to almost every system design problem — tests if you understand statelessness, load balancing, and distributed system trade-offs (CAP theorem, data partitioning).

---

# 2. Core Concepts

| Concept | Explanation |
|---|---|
| **Scaling Out** | Another name for horizontal scaling |
| **Statelessness** | Servers must not store client-specific state locally so any server can handle any request |
| **Load Balancer** | Required component to distribute requests across multiple servers |
| **Sharding** | Splitting data across multiple database nodes |
| **Replication** | Copying data across multiple nodes for availability/read scaling |
| **Auto-scaling** | Automatically adding/removing servers based on load |
| **Consistent Hashing** | Technique to distribute data/load evenly while minimizing reshuffling when nodes change |
| **Distributed Consensus** | Coordinating state across multiple nodes (e.g., Raft, Paxos) |
| **CAP Theorem** | In a distributed system, you can only guarantee 2 of Consistency, Availability, Partition Tolerance |
| **Eventual Consistency** | Data becomes consistent across nodes eventually, not instantly |

---

# 3. Internal Working

**Simple version:** Instead of one big server, you run many smaller servers behind a traffic director (load balancer) that spreads requests among them.

**Technical step-by-step:**
1. Application is made stateless (no local session storage; use shared cache/DB for state).
2. Multiple identical server instances are deployed (often containerized — Docker/Kubernetes).
3. A Load Balancer sits in front, distributing incoming requests (round robin, least connections, etc.).
4. Auto-scaling groups monitor metrics (CPU, request count) and spin up/down instances dynamically.
5. Shared data layer (DB, cache) is accessed by all instances — often itself horizontally scaled via sharding/replication.
6. Health checks continuously verify each instance is alive; unhealthy ones are removed from rotation.

---

# 4. Architecture / Flow

```
                  Client Requests
                        │
                        ▼
                 Load Balancer
            ┌───────────┼───────────┐
            ▼           ▼           ▼
        Server 1     Server 2     Server 3
            │           │           │
            └───────────┼───────────┘
                        ▼
                 Shared Database
                (Sharded/Replicated)
```

**Component explanation:**
- **Load Balancer:** Distributes traffic, performs health checks, removes dead nodes.
- **Server instances:** Identical, stateless, can be scaled in/out dynamically.
- **Shared Database:** Common data layer; itself needs scaling strategies (replication, sharding).

---

# 5. Advantages
- **Near-infinite scalability:** Just keep adding machines (within reason) — no hardware ceiling like vertical scaling.
- **High availability:** If one server dies, others keep serving — no SPOF.
- **Cost-efficient:** Commodity hardware is cheaper per unit than premium high-end machines.
- **Elastic scaling:** Auto-scaling adjusts capacity dynamically based on real-time demand.

# 6. Disadvantages
- **Complexity:** Requires statelessness, load balancing, data partitioning — much harder to design and operate.
- **Data consistency challenges:** Distributed data needs careful handling (CAP theorem trade-offs).
- **Network overhead:** Inter-service communication adds latency compared to a single in-memory call.
- **Operational overhead:** More servers = more monitoring, deployment, and failure scenarios to manage.

# 7. Trade-offs

| Approach | Best For | Downside |
|---|---|---|
| Horizontal Scaling | High traffic, high availability, elasticity | Complex, needs statelessness + data partitioning |
| Vertical Scaling | Simplicity, strong consistency needs | Hardware ceiling, SPOF |

**When to use:** Horizontal scaling for systems expecting unpredictable or massive growth (Netflix, Amazon); vertical scaling as a quick early fix or for naturally single-node workloads.

---

# 8. Real-world Examples
- **Netflix:** Thousands of microservice instances horizontally scaled across AWS regions using auto-scaling groups.
- **Amazon:** Order processing, catalog services all horizontally scaled with load balancers and sharded databases.
- **Google Search:** Massively horizontally scaled index serving across thousands of machines globally.
- **Uber:** Horizontally scales ride-matching services per city/region to handle surge demand.

---

# 9. Interview Deep Dive

**Common questions:**
- What is horizontal scaling and why is it preferred for large systems?
- What makes an application "horizontally scalable"?

**Follow-ups:**
- How do you handle session data in a horizontally scaled system?
- How would you scale a stateful service horizontally (e.g., WebSocket servers)?

**Hidden edge cases:**
- What happens during auto-scaling events when new instances aren't "warmed up" yet (cold start problem)?
- How do you handle distributed transactions across horizontally scaled services?

**Frequently misunderstood:**
- People think horizontal scaling is "always better" — it's not free; it adds real architectural complexity and cost in engineering effort.
- Confusing horizontal scaling of compute with horizontal scaling of data (sharding) — they're related but distinct problems.

---

# 10. Important Terminology

| Term | Definition |
|---|---|
| Sharding | Splitting data across multiple DB nodes |
| Replication | Copying data to multiple nodes |
| Auto-scaling | Dynamically adjusting server count based on load |
| Statelessness | No server-specific session data stored locally |
| Consistent Hashing | Algorithm to evenly distribute keys/load across nodes |

# 11. Time Complexity / Performance
- Adding nodes increases throughput roughly linearly (until bottlenecked elsewhere, e.g., DB).
- Latency may slightly increase due to network hops between LB and servers.
- Scalability impact: Very high — this is THE primary technique for massive scale systems.

# 12. Common Mistakes
- Forgetting to mention statelessness as a prerequisite.
- Not discussing how the database also needs to scale (forgetting data layer entirely).
- Assuming horizontal scaling solves all problems without trade-offs.

# 13. Revision Sheet
- Horizontal scaling = more machines, not bigger ones
- Requires: statelessness, load balancer, often sharded/replicated DB
- Pros: near-infinite scale, high availability, cost-efficient
- Cons: complexity, consistency challenges, network overhead

# 14. Memory Tricks
**"Scale OUT = More Mouths to feed (Servers), need a Waiter (Load Balancer) to direct traffic"**

---

# 15. Interview Questions

**Basic (5):**
1. What is horizontal scaling?
2. How is it different from vertical scaling?
3. Why is statelessness important for horizontal scaling?
4. What is auto-scaling?
5. Name a real company that uses horizontal scaling heavily.

**Medium (5):**
1. How do you manage session data in a horizontally scaled app?
2. What is sharding and how does it relate to horizontal scaling?
3. What is the role of a load balancer in horizontal scaling?
4. What are the challenges of horizontally scaling a database?
5. Explain auto-scaling triggers and cooldown periods.

**Advanced (3):**
1. How would you horizontally scale a stateful WebSocket service?
2. Explain consistent hashing and why it's preferred over simple modulo hashing for scaling.
3. How does CAP theorem affect your horizontal scaling design decisions?

**Scenario (2):**
1. Your e-commerce site gets 10x traffic during a flash sale — how would horizontal scaling + auto-scaling handle this?
2. You need to scale a service that maintains in-memory user carts — what challenges arise with horizontal scaling, and how do you solve them?

---

# 16. Coding Connections
- **Backend Development:** Designing stateless REST APIs.
- **Spring Boot:** Externalizing session state (Spring Session + Redis) to support horizontal scaling.
- **Distributed Systems:** Core topic — sharding, replication, consensus.
- **Databases:** NoSQL databases (Cassandra, DynamoDB) designed natively for horizontal scaling.
- **Networking:** Load balancer algorithms, health checks.
- **Operating Systems:** Container orchestration (Kubernetes) for managing many instances.

---
---

## TOPIC C: LOAD BALANCER

# 1. Concept Overview

**Definition:** A Load Balancer is a system component that distributes incoming network traffic across multiple servers to ensure no single server is overwhelmed.

**Why it exists:** Without it, horizontal scaling would be pointless — you'd have multiple servers but no way to intelligently route traffic among them.

**Real-world analogy:** A traffic police officer at a busy intersection directing cars to different lanes so no single lane gets jammed.

**Where it is used:** Virtually every large-scale web application — Amazon, Netflix, Google, banking systems.

**Why interviewers ask it:** It's a cornerstone component in almost any system design answer involving scale; tests understanding of algorithms, health checks, and failure handling.

---

# 2. Core Concepts

| Concept | Explanation |
|---|---|
| **Layer 4 Load Balancing** | Operates at transport layer (TCP/UDP); routes based on IP/port, fast but less intelligent |
| **Layer 7 Load Balancing** | Operates at application layer (HTTP); can route based on URL, headers, cookies |
| **Round Robin** | Requests distributed sequentially across servers |
| **Weighted Round Robin** | Servers with more capacity get more requests |
| **Least Connections** | Routes to server with fewest active connections |
| **IP Hash** | Routes based on hash of client IP (useful for session stickiness) |
| **Health Check** | Periodic check to see if a server is alive before routing to it |
| **Sticky Sessions (Session Affinity)** | Ensures a client's requests always go to the same server |
| **SSL Termination** | LB decrypts HTTPS traffic so backend servers handle plain HTTP (offloads crypto work) |
| **Active-Passive vs Active-Active LB** | Active-Passive: backup LB takes over on failure; Active-Active: multiple LBs share load simultaneously |
| **Global Server Load Balancing (GSLB)** | Distributes traffic across multiple data centers/regions |

---

# 3. Internal Working

**Simple version:** A request comes in → the load balancer decides which server should handle it based on a rule (round robin, least busy, etc.) → forwards the request → returns the response to the client.

**Technical step-by-step:**
1. Client sends request to a single LB endpoint (often resolved via DNS).
2. LB performs health checks periodically and maintains a list of healthy backend servers.
3. LB applies a routing algorithm (round robin/least connections/IP hash) to pick a target server.
4. If Layer 7, LB may inspect HTTP headers/URL path to make smarter routing decisions (e.g., route `/api/*` to API servers, `/static/*` to static servers).
5. LB forwards the request to the chosen server.
6. Server processes and responds; LB relays the response back to the client.
7. If a server fails health checks, LB removes it from rotation until it recovers.

---

# 4. Architecture / Flow

```
                 Client
                    │
                    ▼
             Load Balancer
        (Health Checks + Routing Algorithm)
            ┌───────┼───────┐
            ▼        ▼        ▼
        Server A  Server B  Server C
         (healthy) (healthy) (DOWN - removed)
```

**Component explanation:**
- **Load Balancer:** Single entry point; decides routing, performs health checks.
- **Servers:** Identical backend instances; LB treats unhealthy ones as unavailable.

---

# 5. Advantages
- **High availability:** Automatically routes around failed servers.
- **Scalability:** Enables horizontal scaling by distributing load evenly.
- **Flexibility:** Layer 7 LBs enable smart routing (A/B testing, canary deployments).
- **Security:** Can handle SSL termination, DDoS mitigation at the edge.

# 6. Disadvantages
- **Single point of failure (if not redundant):** The LB itself can become a bottleneck/SPOF unless made highly available.
- **Added latency:** Extra network hop for every request.
- **Complexity:** Sticky sessions can complicate stateless design goals.

# 7. Trade-offs

| Algorithm | Best For | Downside |
|---|---|---|
| Round Robin | Simple, uniform server capacity | Ignores actual server load |
| Least Connections | Uneven request durations | Slightly more overhead to track |
| IP Hash | Session stickiness needs | Uneven distribution if IPs are unevenly distributed |
| Layer 4 | Speed, simple TCP routing | No application-level intelligence |
| Layer 7 | Smart, content-based routing | Slower (more processing per request) |

---

# 8. Real-world Examples
- **AWS ELB/ALB:** Application Load Balancer offers Layer 7 routing; Network Load Balancer offers Layer 4 for ultra-low latency.
- **Netflix:** Uses internal load balancing combined with client-side load balancing (Eureka + Ribbon historically) for microservices.
- **Google:** Uses Maglev, a custom Layer 4 load balancer, plus global load balancing for routing across data centers.
- **NGINX/HAProxy:** Popular open-source software load balancers used by countless companies.

---

# 9. Interview Deep Dive

**Common questions:**
- What is a load balancer and why do we need it?
- Difference between Layer 4 and Layer 7 load balancing?

**Follow-ups:**
- How do you make the load balancer itself highly available?
- How does a load balancer handle a server that's "alive" but slow (not technically down)?

**Hidden edge cases:**
- Sticky sessions can cause uneven load if one "sticky" server gets overloaded by a popular user session.
- Health checks passing but the app being functionally broken (e.g., DB connection lost) — need application-level health checks, not just TCP ping.

**Frequently misunderstood:**
- People think Round Robin is always good — it ignores actual server load/capacity differences.
- Confusing Load Balancer with API Gateway (LB just distributes traffic; API Gateway does much more — auth, rate limiting, routing logic).

---

# 10. Important Terminology

| Term | Definition |
|---|---|
| Health Check | Periodic check of server availability |
| Sticky Session | Routing a client consistently to the same server |
| SSL Termination | Decrypting HTTPS at the LB |
| L4/L7 | Transport layer vs Application layer balancing |

# 11. Time Complexity / Performance
- Routing decision: O(1) for round robin/random; O(n) worst case for least-connections scan (often optimized with heaps)
- Adds minor latency (single extra hop, ~1-5ms typically)
- Massively improves throughput and availability at scale

# 12. Common Mistakes
- Not mentioning health checks when describing LB behavior.
- Confusing LB with API Gateway.
- Forgetting that the LB itself needs redundancy (DNS-based failover or multiple LB instances).

# 13. Revision Sheet
- LB distributes traffic across multiple servers
- L4 = fast, IP/port based | L7 = smart, content-based
- Algorithms: Round Robin, Weighted RR, Least Connections, IP Hash
- Health checks remove dead servers from rotation
- SSL termination offloads encryption work from backend

# 14. Memory Tricks
**"L4 = Lightning fast, L7 = Logic-based"**

---

# 15. Interview Questions

**Basic (5):**
1. What is a load balancer?
2. Why do we need load balancers?
3. What is Round Robin?
4. What is a health check?
5. What's the difference between Layer 4 and Layer 7 LB?

**Medium (5):**
1. What is sticky session and when would you use it?
2. What is SSL termination?
3. Compare Least Connections vs Round Robin algorithms.
4. How does a load balancer detect a failed server?
5. What is the difference between LB and API Gateway?

**Advanced (3):**
1. How would you make a load balancer highly available (no SPOF)?
2. Explain Global Server Load Balancing (GSLB) and its use cases.
3. How would you design a load balancer for a system with WebSocket connections?

**Scenario (2):**
1. One of your servers is alive (passes TCP health check) but its database connection is broken — how do you prevent the LB from routing traffic to it?
2. You notice one server getting overloaded due to sticky sessions — how would you redesign this?

---

# 16. Coding Connections
- **Backend Development:** Designing health check endpoints (`/health`, `/ready`).
- **Spring Boot:** Spring Cloud LoadBalancer, integrating with service discovery.
- **Distributed Systems:** Core building block for any scaled system.
- **Networking:** TCP/HTTP routing, SSL/TLS termination.
- **Operating Systems:** Connection handling, socket management at scale.

---
---

## TOPIC D: ROUTE ROUTING

# 1. Concept Overview

**Definition:** Routing refers to the logic that determines which server, service, or handler should process a given request based on its path, headers, or other attributes.

**Why it exists:** In a system with many services/endpoints, you need rules to direct each type of request to the correct destination — whether at the network level (which data center) or application level (which microservice/controller).

**Real-world analogy:** A receptionist at a large hospital directing patients to the correct department (cardiology, pediatrics) based on their symptoms.

**Where it is used:** API Gateways, Load Balancers (Layer 7), web frameworks (Express, Spring MVC), microservice architectures.

**Why interviewers ask it:** Tests understanding of how requests get correctly dispatched in complex, multi-service systems — crucial for microservices and API Gateway designs.

---

# 2. Core Concepts

| Concept | Explanation |
|---|---|
| **Path-based Routing** | Routes based on URL path (e.g., `/users/*` → User Service) |
| **Host-based Routing** | Routes based on domain/subdomain (e.g., `api.example.com` vs `admin.example.com`) |
| **Header-based Routing** | Routes based on HTTP headers (e.g., API version header) |
| **Weighted Routing** | Splits traffic by percentage across versions (canary deployments, A/B testing) |
| **Geo-based Routing** | Routes based on user's geographic location |
| **Dynamic Routing** | Routes determined at runtime based on service registry/discovery |
| **Static Routing** | Hardcoded routing rules, rarely change |
| **Service Discovery** | Mechanism by which routing rules find the current location/IP of services (especially important since services scale up/down dynamically) |
| **Reverse Proxy** | A server that routes client requests to backend servers, often combined with routing logic |

---

# 3. Internal Working

**Simple version:** When a request comes in, the system looks at its URL/headers and decides "this goes to Service A" or "this goes to Service B," then forwards it there.

**Technical step-by-step:**
1. Request arrives at a router (could be API Gateway, Load Balancer, or web framework router).
2. Router inspects request attributes — path, host, headers, query params.
3. Router matches against a defined routing table/rule set.
4. If using service discovery, router queries a registry (e.g., Eureka, Consul) for current healthy instances of the target service.
5. Request is forwarded to the matched service/instance.
6. Response is routed back to the client.

---

# 4. Architecture / Flow

```
                    Client Request: /orders/123
                              │
                              ▼
                       API Gateway / Router
              ┌───────────────┼───────────────┐
              ▼                ▼                ▼
        /users/*          /orders/*          /payments/*
       User Service      Order Service     Payment Service
```

**Component explanation:**
- **Router/Gateway:** Inspects incoming requests and matches against rules.
- **Target Services:** Independent microservices handling specific domains.

---

# 5. Advantages
- **Modularity:** Enables microservices architecture by cleanly separating concerns.
- **Flexibility:** Supports canary releases, A/B testing, versioning via weighted/header-based routing.
- **Scalability:** Combined with service discovery, routing adapts dynamically as services scale.

# 6. Disadvantages
- **Complexity:** More routing rules = more potential for misconfiguration.
- **Latency overhead:** Extra hop for routing decision, especially with dynamic service discovery lookups.
- **Debugging difficulty:** Hard to trace a request across many routing layers without proper distributed tracing.

# 7. Trade-offs

| Approach | Best For | Downside |
|---|---|---|
| Static Routing | Simple, stable systems | Doesn't adapt to dynamic scaling |
| Dynamic Routing (Service Discovery) | Microservices, auto-scaling environments | More complex, dependent on registry health |
| Path-based | REST APIs with clear resource boundaries | Can get unwieldy with too many nested rules |
| Header-based | API versioning, feature flagging | Less visible/debuggable than path-based |

---

# 8. Real-world Examples
- **Netflix:** Uses Zuul/Spring Cloud Gateway for dynamic routing across hundreds of microservices.
- **Amazon:** API Gateway routes requests to Lambda functions or backend services based on path/method.
- **Uber:** Routes ride requests to region-specific services based on geo-routing.
- **Kubernetes Ingress:** Routes external traffic to the correct internal service based on host/path rules.

---

# 9. Interview Deep Dive

**Common questions:**
- What is routing in the context of system design?
- Difference between path-based and host-based routing?

**Follow-ups:**
- How does routing work with service discovery in a dynamic environment?
- How would you implement canary deployment using routing?

**Hidden edge cases:**
- What happens if two routing rules overlap/conflict (e.g., `/users/*` and `/users/admin`)? Need rule precedence (most specific wins).
- Routing during a deployment when old and new service versions coexist (version skew).

**Frequently misunderstood:**
- People confuse "routing" with "load balancing" — routing decides WHICH service, load balancing decides WHICH instance of that service.

---

# 10. Important Terminology

| Term | Definition |
|---|---|
| Routing Table | Set of rules mapping requests to destinations |
| Service Discovery | Mechanism to find current location of services |
| Canary Routing | Sending a small % of traffic to a new version |
| Reverse Proxy | Server routing client requests to backend services |

# 11. Time Complexity / Performance
- Routing lookup: typically O(1) with hash-based matching, O(log n) with trie-based path matching for complex rule sets
- Adds minor latency per hop; dynamic service discovery lookups can add more if not cached
- Scalability impact: Essential for microservices to scale independently

# 12. Common Mistakes
- Confusing routing with load balancing.
- Forgetting rule precedence/conflict handling.
- Not considering how routing adapts when services scale dynamically (treating it as static).

# 13. Revision Sheet
- Routing = deciding WHICH service handles a request
- Load Balancing = deciding WHICH instance of that service
- Types: path-based, host-based, header-based, weighted, geo-based
- Dynamic routing relies on service discovery (Eureka, Consul)

# 14. Memory Tricks
**"Routing picks WHICH door, Load Balancing picks WHICH person behind that door"**

---

# 15. Interview Questions

**Basic (5):**
1. What is routing in system design?
2. What is path-based routing?
3. What is host-based routing?
4. Difference between routing and load balancing?
5. What is a reverse proxy?

**Medium (5):**
1. What is weighted routing and where is it used?
2. How does service discovery relate to dynamic routing?
3. What is canary deployment and how does routing enable it?
4. How would you route traffic based on API version?
5. What challenges arise from overlapping routing rules?

**Advanced (3):**
1. How would you design routing for a multi-region, multi-service architecture?
2. Explain how Kubernetes Ingress routing works internally.
3. How would you implement zero-downtime deployment using routing strategies?

**Scenario (2):**
1. You want to test a new version of a service with 5% of production traffic — how would you set up routing?
2. Your routing rules have a conflict between `/api/users/*` and `/api/users/admin` — how do you resolve precedence?

---

# 16. Coding Connections
- **Backend Development:** Defining routes in Express.js, Flask, Spring MVC controllers.
- **Spring Boot:** `@RequestMapping`, Spring Cloud Gateway routing rules.
- **Distributed Systems:** Service discovery integration (Eureka, Consul, Zookeeper).
- **Databases:** Routing read/write queries to correct DB replicas/shards.
- **Networking:** Reverse proxies (NGINX), Ingress controllers.
- **Operating Systems:** Kernel-level routing tables (lower-level analog, less relevant to app routing).

---

# 17. Related Topics (for entire cluster)
API Gateway, Microservices, Service Discovery, Consistent Hashing, CAP Theorem, Auto-scaling, Database Sharding/Replication.

# 18. Cheat Sheet (Cluster Summary)
```
VERTICAL SCALING  = bigger machine, simple, hardware ceiling, SPOF
HORIZONTAL SCALING = more machines, complex, near-infinite scale, needs statelessness
LOAD BALANCER     = distributes traffic across instances (L4 fast / L7 smart)
ROUTING           = decides WHICH service handles a request (path/host/header-based)

Routing → picks the service
Load Balancer → picks the instance within that service
```

# 19. Placement Rating
- Importance: ★★★★★
- Interview Frequency: ★★★★★
- Difficulty: ★★★☆☆
- Why: Scaling and load balancing form the backbone of almost every "design X at scale" interview question.

# 20. Final Interview Summary
1. Vertical scaling = bigger machine (simple but has a hard ceiling + SPOF).
2. Horizontal scaling = more machines (scalable but requires statelessness + data partitioning).
3. Load Balancer distributes traffic across instances using algorithms like Round Robin, Least Connections (L4 = fast/dumb, L7 = smart/slower).
4. Routing decides WHICH service handles a request; Load Balancing decides WHICH instance of that service.
5. Always mention health checks, statelessness, and service discovery — these come up as follow-ups in nearly every scaling question.
