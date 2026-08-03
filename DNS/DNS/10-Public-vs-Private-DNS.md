# Module 10 — Public vs Private DNS

> **Difficulty:** Beginner → Intermediate  
> **Prerequisites:** DNS Records, Forward Lookup, Reverse Lookup

---

# Learning Objectives

By the end of this chapter, you'll understand:

- What is Public DNS?
- What is Private DNS?
- Differences between them
- Public vs Private IP addresses
- AWS Route 53 Public & Private Hosted Zones
- Kubernetes DNS
- Corporate DNS
- VPN & Hybrid Cloud DNS
- Split-Horizon DNS
- Production architectures
- Troubleshooting techniques
- Interview questions

---

# Why Do We Need Two Types of DNS?

Think about these two hostnames:

```
google.com
```

and

```
db.internal.company.local
```

You can access **google.com** from almost anywhere in the world.

Can you resolve **db.internal.company.local** from your home Wi-Fi?

**No.**

That's because not every DNS record is meant to be publicly accessible.

DNS is divided into:

- 🌍 Public DNS
- 🔒 Private DNS

---

# What is Public DNS?

A **Public DNS** zone contains records that anyone on the Internet can query.

Example:

```
google.com

↓

142.250.xxx.xxx
```

Whether you're in:

- India
- Canada
- Germany
- Japan

You'll receive a DNS response.

---

# Public DNS Architecture

```
                Internet
                    │
                    ▼
        Public Recursive Resolver
                    │
                    ▼
          Authoritative DNS
                    │
                    ▼
             google.com
                    │
                    ▼
              Public IP
```

---

# Examples of Public DNS

- google.com
- amazon.com
- github.com
- openai.com
- youtube.com
- netflix.com

These domains are intended to be reachable worldwide.

---

# What is Private DNS?

Private DNS zones exist only inside trusted networks.

Example:

```
db.internal.company.local

↓

10.0.1.25
```

Only systems connected to:

- Company LAN
- VPN
- AWS VPC
- Azure VNet
- Kubernetes Cluster

can resolve these records.

The public Internet has no knowledge of these names.

---

# Private DNS Architecture

```
Employee Laptop
       │
       ▼
      VPN
       │
       ▼
Company DNS Server
       │
       ▼
db.internal.company.local
       │
       ▼
10.0.1.25
```

No request leaves the organization's private network.

---

# Real-Life Analogy

Imagine a company.

It has:

**Public Phone Number**

```
+1-800-123-456
```

Anyone can call.

Inside the company, employees use:

```
Extension 205

Extension 310

Extension 512
```

Those extensions only work internally.

DNS follows the same idea.

```
Public DNS
        │
        ▼
Everyone

Private DNS
        │
        ▼
Internal Users
```

---

# Public DNS Resolution

```
Laptop

↓

ISP DNS Resolver

↓

Internet DNS

↓

google.com

↓

Public IP Returned
```

---

# Private DNS Resolution

```
Laptop

↓

VPN

↓

Internal DNS

↓

database.internal

↓

10.10.1.50
```

Notice:

The Internet never participates.

---

# Public DNS vs Private DNS

| Feature | Public DNS | Private DNS |
|----------|------------|-------------|
| Accessible from Internet | ✅ Yes | ❌ No |
| Used for Websites | ✅ Yes | Sometimes |
| Used for Databases | Rarely | ✅ Yes |
| Used for Internal APIs | Rarely | ✅ Yes |
| Uses Public DNS Infrastructure | ✅ Yes | ❌ No |
| Common IPs | Public IP | Usually Private IP |

---

# Public DNS vs Private IP

Many beginners confuse these concepts.

They are different.

**Public DNS**

Defines **who can resolve the DNS record**.

**Private IP**

Defines **whether the IP is routable on the public Internet**.

Example:

```
example.internal

↓

54.120.x.x
```

Even if the record points to a public IP, it can still be stored in a private DNS zone.

Likewise,

```
example.com

↓

10.0.1.15
```

A public DNS record pointing to a private IP would generally be useless to Internet users because private IPs aren't publicly routable.

---

# AWS Example

Suppose you launch an EC2 instance.

AWS assigns:

```
Public IP

54.210.10.25
```

and

```
Private IP

10.0.1.20
```

Public hostname:

```
ec2-54-210-10-25.compute.amazonaws.com
```

Private hostname:

```
ip-10-0-1-20.ec2.internal
```

Resources inside the VPC use the private hostname.

Internet users use the public hostname (assuming routing and security rules allow access).

---

# AWS Route 53

AWS supports two types of Hosted Zones.

## Public Hosted Zone

```
mycompany.com

↓

54.210.10.20
```

Visible globally.

---

## Private Hosted Zone

```
database.internal

↓

10.0.2.50
```

Visible only to associated VPCs.

---

# Kubernetes Example

```
api.default.svc.cluster.local

↓

10.96.15.25
```

Pods inside the cluster resolve this through **CoreDNS**.

Your laptop connected to the Internet cannot resolve it.

---

# Corporate Network Example

Company DNS contains:

```
vpn.company.local

db.company.local

jenkins.company.local

grafana.company.local
```

Employees connected through VPN automatically use the company's DNS server.

Without VPN:

```
db.company.local
```

returns

```
DNS Error
```

because the public Internet has no knowledge of the private zone.

---

# Hybrid Cloud Example

A company has:

- On-Premises
- AWS
- Azure
- Kubernetes

Private DNS allows workloads across these environments to resolve:

```
db.internal

↓

10.10.5.20
```

as long as the networks are securely connected.

---

# Split-Horizon DNS

A single domain can return different answers depending on where the request originates.

Example:

Public:

```
app.company.com

↓

54.210.10.20
```

Inside the corporate network:

```
app.company.com

↓

10.0.2.15
```

This is called **Split-Horizon DNS** (also known as Split-Brain DNS).

Benefits:

- Better security
- Lower latency
- Internal traffic stays private

---

# Production Architecture

```
                 Internet Users
                        │
                        ▼
                app.company.com
                        │
                        ▼
                Public Load Balancer
                        │
        ┌───────────────┴───────────────┐
        ▼                               ▼
   Web Server 1                    Web Server 2
        │                               │
        └───────────────┬───────────────┘
                        ▼
            db.internal.company.local
                        │
                        ▼
                  Private Database
```

Only the application is publicly accessible.

The database remains hidden behind private DNS.

---

# Why Companies Use Private DNS

Common internal services include:

- Databases
- Redis
- Jenkins
- Grafana
- Prometheus
- Vault
- Active Directory
- Internal APIs
- Message Queues
- Kubernetes Services

These services should not be publicly discoverable.

---

# Troubleshooting Checklist

If a private hostname isn't resolving:

1. Are you connected to the VPN?
2. Are you using the correct DNS server?
3. Does the private zone exist?
4. Is the VPC associated with the Private Hosted Zone?
5. Is CoreDNS running (Kubernetes)?
6. Clear local DNS cache if necessary.
7. Test using:

```bash
dig hostname
```

or

```bash
nslookup hostname
```

---

# Common Misconceptions

## ❌ Private DNS means Private IP

False.

Private DNS refers to **where the DNS zone is visible**, not necessarily the type of IP it contains.

---

## ❌ Public DNS is always faster

Not always.

Performance depends on:

- Network latency
- Caching
- Resolver location
- DNS infrastructure

---

## ❌ Private DNS works automatically

False.

A private DNS service must exist.

Examples:

- Microsoft Active Directory DNS
- AWS Route 53 Private Hosted Zone
- CoreDNS
- BIND
- dnsmasq

---

# Interview Questions

## What is Public DNS?

A DNS service that provides records accessible from anywhere on the public Internet.

---

## What is Private DNS?

A DNS service that provides records only within trusted or authorized networks.

---

## Does AWS Route 53 support both?

Yes.

- Public Hosted Zones
- Private Hosted Zones

---

## Why do companies use Private DNS?

To keep internal services such as databases, monitoring systems, APIs, and infrastructure accessible only from trusted networks.

---

## What is Split-Horizon DNS?

A DNS configuration where the same domain returns different IP addresses depending on the client's network location.

---

## Can a Private DNS zone contain a Public IP?

Yes.

Although uncommon, a private DNS zone can technically contain records pointing to public IP addresses.

---

# Key Takeaways

- **Public DNS** is globally accessible.
- **Private DNS** is accessible only within trusted networks.
- Public DNS visibility and Public IP addressing are different concepts.
- AWS Route 53 supports both Public and Private Hosted Zones.
- Kubernetes relies on **CoreDNS** for internal service discovery.
- VPNs often configure clients to use internal DNS servers.
- Split-Horizon DNS allows the same hostname to resolve differently for internal and external users.

---

# What's Next?

## Module 11 — DNS Propagation & TTL

You'll learn:

- What is DNS propagation?
- TTL (Time To Live)
- Recursive resolver caching
- Browser & OS DNS cache
- Why DNS changes take time
- How to flush DNS cache
- Production migration strategies
- Common troubleshooting scenarios