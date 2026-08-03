Module 14 — DNS in Kubernetes
DNS/
├── 14-DNS-in-Kubernetes.md
Learning Objectives

By the end of this chapter, you'll understand:

Why Kubernetes needs DNS
What CoreDNS is
How Pods resolve names
Service Discovery
ClusterIP and DNS
FQDN in Kubernetes
Search Domains
/etc/resolv.conf inside Pods
DNS resolution flow
Real production examples
Why Kubernetes Needs DNS

Imagine a cluster:

Frontend Pod

Backend Pod

Redis Pod

Database Pod

How does the frontend find the backend?

Using:

10.244.2.35

No.

Because Pods are temporary.

If a Pod dies:

10.244.2.35

may become:

10.244.5.87

Hardcoding Pod IPs would constantly break applications.

Instead Kubernetes uses Services and DNS.

High-Level Architecture
                 Kubernetes Cluster

        ┌──────────────────────────────────┐

            frontend Pod

                  │
                  ▼

              backend Service

                  │
                  ▼

             backend Pods

                  │
                  ▼

             database Service

                  │
                  ▼

             database Pods

        └──────────────────────────────────┘

Applications talk to Service names, not Pod IPs.

What is CoreDNS?

Every Kubernetes cluster includes a DNS server.

Today, that server is usually:

CoreDNS

CoreDNS is responsible for:

Resolving Service names
Resolving Pod names (when configured)
Forwarding external DNS requests
Providing service discovery

Think of it as the cluster's DNS server.

DNS Resolution Flow

Suppose:

frontend

needs:

backend

Flow:

Frontend Pod
      │
      ▼
CoreDNS
      │
      ▼
backend.default.svc.cluster.local
      │
      ▼
ClusterIP
      │
      ▼
Backend Pods

The application only knows:

backend

CoreDNS does the rest.

Service Discovery

Suppose you create:

apiVersion: v1
kind: Service

metadata:
  name: backend

Kubernetes automatically creates:

backend

as a DNS name.

Now any Pod can connect using:

http://backend

No IP addresses required.

Fully Qualified Domain Name (FQDN)

Every Service gets a full DNS name.

Example:

backend.default.svc.cluster.local

Let's break it down.

Part	Meaning
backend	Service name
default	Namespace
svc	Indicates a Service
cluster.local	Cluster DNS domain (default)
Kubernetes Search Domains

Inside every Pod:

cat /etc/resolv.conf

Example:

search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5

Notice the search domains.

If the application asks for:

backend

Kubernetes automatically tries:

backend.default.svc.cluster.local

Then:

backend.svc.cluster.local

Then:

backend.cluster.local

This is why short names usually work inside the same namespace.

Nameserver

Example:

nameserver 10.96.0.10

This IP belongs to:

CoreDNS Service

Every Pod sends DNS queries to CoreDNS.

How External DNS Works

Suppose the Pod requests:

google.com

Flow:

Pod
 │
 ▼
CoreDNS
 │
 ▼
Upstream DNS
 │
 ▼
Internet
 │
 ▼
Google IP

CoreDNS forwards queries that it cannot answer locally.

Pod vs Service

Pods change.

Services stay.

Example:

Pod

10.244.1.25

Restart:

10.244.6.10

But the Service remains:

backend.default.svc.cluster.local

Applications should always use the Service name.

Production Example 1

Application:

Frontend

needs:

Redis

Instead of:

10.244.5.15

Configuration:

REDIS_HOST=redis

CoreDNS resolves:

redis.default.svc.cluster.local
↓

10.96.25.10

The Service forwards traffic to healthy Redis Pods.

Production Example 2

Microservices:

Frontend

↓

Auth Service

↓

User Service

↓

Database

Connections:

http://auth

http://users

database

Every name is resolved through CoreDNS.

Production Example 3

Suppose the Backend Pod crashes.

Old Pod:

10.244.1.12

New Pod:

10.244.8.30

Nothing changes for the Frontend.

It still connects to:

backend

because the Service selects the new healthy Pod.

Kubernetes DNS Resolution Flow
Application
      │
      ▼
Pod Resolver
      │
      ▼
CoreDNS
      │
      ├────────► Kubernetes Services
      │
      ▼
Upstream DNS
      │
      ▼
Internet
Kubernetes vs Docker DNS
Docker	Kubernetes
Embedded DNS (127.0.0.11)	CoreDNS
Resolves container names	Resolves Service names
User-defined networks	Cluster-wide networking
Service names from Compose	Service names from Kubernetes Services
Simpler	More scalable and feature-rich
Common Misconceptions
❌ Pods should communicate using Pod IPs.

Wrong.

Pods are ephemeral.

Always communicate using Services.

❌ CoreDNS only resolves Kubernetes Services.

No.

It also forwards external DNS requests to upstream resolvers.

❌ backend is just an alias.

No.

It is a real DNS record created by Kubernetes for the Service.

❌ Every Pod has a fixed IP.

Wrong.

Pod IPs change when Pods are recreated.

Service DNS names remain stable.

Troubleshooting DNS in Kubernetes

View DNS configuration inside a Pod:

kubectl exec -it nginx -- cat /etc/resolv.conf

Check if a Service resolves:

kubectl exec -it nginx -- nslookup backend

Or with dig:

kubectl exec -it nginx -- dig backend

List CoreDNS Pods:

kubectl get pods -n kube-system

Typical output:

coredns-7f89d4d6c9-abc12
coredns-7f89d4d6c9-def34

Check the CoreDNS Service:

kubectl get svc -n kube-system

Look for a Service named:

kube-dns

Despite the name, this Service usually points to CoreDNS Pods.

Interview Questions
Q1. What DNS server does Kubernetes use?

CoreDNS (modern clusters).

Q2. Why should applications use Service names instead of Pod IPs?

Because Pods are temporary and their IP addresses can change.

Q3. What is the default Kubernetes cluster domain?
cluster.local

(Although it can be customized.)

Q4. What is the FQDN of a Service?

Example:

backend.default.svc.cluster.local
Q5. Does CoreDNS resolve external domains?

Yes.

If it cannot resolve a Kubernetes name, it forwards the request to upstream DNS servers.

Summary
                Kubernetes Cluster

             Frontend Pod
                  │
                  ▼
              CoreDNS
                  │
      ┌───────────┴───────────┐
      ▼                       ▼
 Kubernetes Services      External DNS
      │                       │
      ▼                       ▼
  Backend Service         google.com
      │
      ▼
  Backend Pods
Key Takeaways
CoreDNS is the DNS server for Kubernetes.
Applications communicate using Service names, not Pod IPs.
Every Service receives a stable DNS name.
Pods use CoreDNS through the nameserver entry in /etc/resolv.conf.
CoreDNS also forwards external DNS queries.
Understanding Kubernetes DNS is essential for debugging microservices and production clusters.
Where You Are Now

At this point, you've mastered DNS across multiple environments:

✅ Internet DNS
✅ Linux DNS
✅ AWS DNS
✅ Docker DNS
✅ Kubernetes DNS

This is far beyond what most beginners learn.