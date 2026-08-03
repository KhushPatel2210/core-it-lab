
# Module 12 — DNS in AWS

> **Difficulty:** Intermediate  
> **Prerequisites:** DNS Fundamentals, Linux DNS, Public vs Private DNS

---

# Learning Objectives

By the end of this chapter, you'll understand:

- How AWS uses DNS
- AmazonProvidedDNS
- Public & Private DNS in AWS
- EC2 DNS hostnames
- Route 53
- Public & Private Hosted Zones
- Route 53 Resolver
- Alias Records
- VPC DNS Settings
- Hybrid DNS
- EKS & Kubernetes DNS
- Production architectures
- Troubleshooting
- Interview questions

---

# Why Does AWS Need DNS?

Imagine an EC2 instance.

Today:

```
54.210.10.20
```

Tomorrow after stopping and starting:

```
54.175.120.80
```

If applications connect directly to IP addresses:

❌ Everything breaks.

Instead applications use:

```
api.company.com
```

DNS automatically resolves the current IP.

Applications never need to know the underlying infrastructure changes.

---

# High-Level AWS DNS Architecture

```
                 Internet
                     │
                     ▼
        Route 53 Public Hosted Zone
                     │
                     ▼
             app.company.com
                     │
                     ▼
        Application Load Balancer
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
       EC2-1                 EC2-2
          │                     │
          └──────────┬──────────┘
                     ▼
            AmazonProvidedDNS
                     │
                     ▼
        Route 53 Private Hosted Zone
                     │
                     ▼
          db.internal.company.local
                     │
                     ▼
                  Amazon RDS
```

Only the application is public.

The database remains private.

---

# Public DNS in AWS

Suppose an EC2 instance has:

```
Public IP

54.210.10.20
```

AWS automatically assigns a public hostname.

Example:

```
ec2-54-210-10-20.compute.amazonaws.com
```

Anyone can resolve it (assuming networking and security allow access).

---

# Private DNS in AWS

Inside the VPC, the same instance has:

```
Private IP

10.0.1.20
```

Private hostname:

```
ip-10-0-1-20.ec2.internal
```

Only resources inside the VPC or connected networks can resolve it.

---

# One EC2, Two DNS Names

```
             EC2 Instance

      Public Hostname
               │
               ▼
ec2-54-210-10-20.compute.amazonaws.com
               │
               ▼
          54.210.10.20

-----------------------------------------

      Private Hostname
               │
               ▼
ip-10-0-1-20.ec2.internal
               │
               ▼
           10.0.1.20
```

---

# AmazonProvidedDNS

Every AWS VPC automatically includes a built-in DNS resolver called:

```
AmazonProvidedDNS
```

Responsibilities:

- Resolve EC2 private hostnames
- Resolve Route 53 Private Hosted Zones
- Resolve AWS internal service names
- Resolve VPC endpoint DNS names

You do **not** need to install a DNS server for basic VPC name resolution.

---

# Where Does AmazonProvidedDNS Live?

Example VPC:

```
10.0.0.0/16
```

Resolver:

```
10.0.0.2
```

AWS also exposes:

```
169.254.169.253
```

Both refer to the VPC DNS resolver.

Normally DHCP configures these automatically.

---

# VPC DNS Settings

Each VPC has two important settings.

## enableDnsSupport

Controls whether DNS resolution works inside the VPC.

If disabled:

❌ DNS queries fail.

---

## enableDnsHostnames

Controls whether EC2 instances receive DNS hostnames.

If disabled:

EC2 instances still receive IP addresses but may not receive DNS names.

---

# Route 53 Hosted Zones

AWS provides two Hosted Zone types.

---

## Public Hosted Zone

Example:

```
company.com

↓

Application Load Balancer
```

Accessible from anywhere.

---

## Private Hosted Zone

Example:

```
db.internal

↓

10.0.2.30
```

Accessible only from associated VPCs.

---

# Alias Records (AWS Feature)

AWS introduces **Alias Records**, which behave similarly to CNAMEs but are designed for AWS resources.

You can point a domain directly to:

- Application Load Balancer
- Network Load Balancer
- CloudFront Distribution
- S3 Static Website
- API Gateway

Example:

```
app.company.com

↓

ALB
```

Benefits:

- Supported at the zone apex (root domain)
- No extra DNS lookup
- AWS-managed target updates

---

# Route 53 Resolver

Route 53 Resolver connects AWS DNS with external DNS systems.

Supports:

- Inbound Endpoints
- Outbound Endpoints

Typical architecture:

```
On-Prem DNS

↓

Route 53 Resolver

↓

AWS Private Hosted Zone
```

This enables Hybrid Cloud DNS.

---

# Production Architecture

```
Internet Users
        │
        ▼
 shop.company.com
        │
        ▼
Route 53 Public Zone
        │
        ▼
Application Load Balancer
        │
 ┌──────┴──────┐
 ▼             ▼
EC2         EC2
        │
        ▼
AmazonProvidedDNS
        │
        ▼
db.internal.company.local
        │
        ▼
Amazon RDS
```

---

# Auto Scaling & DNS

Auto Scaling launches:

```
Old Instance

10.0.1.20
```

Later:

```
New Instance

10.0.1.45
```

Applications should never use those IPs directly.

Instead they connect through:

- DNS
- Load Balancers
- Service Discovery

Infrastructure changes become invisible.

---

# DNS for AWS Services

AWS services are usually accessed by hostname.

Examples:

```
s3.amazonaws.com

ec2.amazonaws.com

sts.amazonaws.com

dynamodb.amazonaws.com
```

AWS SDKs resolve these names automatically.

---

# Interface VPC Endpoints

Suppose your EC2 instance accesses:

```
S3
```

Instead of using the Internet:

```
EC2

↓

Private Endpoint

↓

AWS Service
```

AWS automatically creates private DNS entries for many Interface Endpoints, allowing applications to continue using familiar AWS service hostnames while traffic stays on the AWS network.

---

# Hybrid Cloud DNS

```
On-Prem

↓

Corporate DNS

↓

Route 53 Resolver

↓

AWS Private Hosted Zone

↓

EC2
```

Both environments can resolve each other's private names.

---

# Kubernetes (EKS)

Inside Amazon EKS:

CoreDNS resolves:

```
api.default.svc.cluster.local
```

Applications still rely on AmazonProvidedDNS for VPC-level resolution outside the cluster.

---

# Route 53 Routing Policies

Route 53 supports intelligent routing.

Examples:

- Simple
- Weighted
- Latency
- Failover
- Geolocation
- Geoproximity
- Multi-Value Answer

These are widely used in production for high availability and traffic management.

---

# Common Troubleshooting

Check DNS server:

```bash
cat /etc/resolv.conf
```

Query Route 53:

```bash
dig app.company.com
```

Verify private DNS:

```bash
nslookup db.internal
```

Check VPC DNS settings.

Verify Security Groups and NACLs if connectivity still fails after successful DNS resolution.

---

# Common Misconceptions

## ❌ Route 53 is required for all AWS DNS

False.

Every VPC already includes AmazonProvidedDNS.

---

## ❌ Every EC2 gets a Public DNS Name

False.

Only instances with public IP addresses receive public hostnames.

---

## ❌ Applications should use Private IPs

Not recommended.

Use stable DNS names whenever possible.

---

## ❌ AmazonProvidedDNS and Route 53 are the same

False.

| AmazonProvidedDNS | Route 53 |
|-------------------|----------|
| VPC DNS Resolver | Managed DNS Service |
| Exists in every VPC | Optional AWS service |
| Resolves names | Hosts DNS zones |
| Used automatically | Configured by you |

---

# Interview Questions

### What DNS service exists in every VPC?

AmazonProvidedDNS.

---

### What are the two important VPC DNS settings?

- `enableDnsSupport`
- `enableDnsHostnames`

---

### What is the difference between Public and Private Hosted Zones?

Public Hosted Zones are accessible from the Internet.

Private Hosted Zones are accessible only from associated VPCs.

---

### What is an Alias Record?

An AWS-specific DNS record that points directly to AWS resources such as ALBs, CloudFront distributions, and API Gateways without the limitations of a traditional CNAME.

---

### Why use DNS instead of IP addresses?

Cloud infrastructure changes frequently.

DNS provides stable names while underlying IP addresses change.

---

### What is Route 53 Resolver?

A managed DNS resolver that enables DNS resolution between AWS and external networks, supporting hybrid cloud deployments.

---

# Key Takeaways

- Every VPC includes **AmazonProvidedDNS** by default.
- EC2 instances can have both public and private DNS names.
- Route 53 provides both **Public** and **Private Hosted Zones**.
- Alias Records simplify routing to AWS resources.
- Applications should use DNS names rather than hardcoded IP addresses.
- Hybrid environments often use Route 53 Resolver to integrate AWS and on-premises DNS.
- Route 53 routing policies support high availability, disaster recovery, and global traffic management.

---

# What's Next?

## Module 13 — DNS Troubleshooting & Debugging

You'll learn:

- `dig`
- `nslookup`
- `host`
- `getent`
- `resolvectl`
- `tcpdump`
- `curl --resolve`
- DNS propagation
- Real-world production debugging workflows