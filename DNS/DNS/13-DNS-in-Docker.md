# Module 13 — DNS in Docker

> **Difficulty:** Intermediate  
> **Prerequisites:** Linux DNS, AWS DNS, Docker Networking Basics

---

# Learning Objectives

By the end of this chapter, you'll understand:

- Why Docker needs DNS
- Docker Embedded DNS
- Container Name Resolution
- `/etc/resolv.conf` inside containers
- User-defined vs Default Bridge Networks
- Docker Compose DNS
- Multi-network DNS
- External DNS Resolution
- Docker DNS Configuration
- Production Use Cases
- Troubleshooting
- Interview Questions

---

# Why Does Docker Need DNS?

Imagine you have two containers.

```
Frontend

Backend
```

How should Frontend connect to Backend?

Option 1:

```
172.18.0.3
```

❌ Bad practice.

Container IPs are not guaranteed to remain the same after recreation.

Instead:

```
http://backend
```

Docker resolves:

```
backend

↓

172.18.0.3
```

Applications use stable names instead of changing IP addresses.

---

# Docker Networking Overview

Create a custom network.

```bash
docker network create app-network
```

Containers:

```
frontend

backend

database
```

All containers on the same user-defined network can communicate using their names.

---

# Docker DNS Architecture

```
                Docker Network

       ┌───────────────────────────┐

        frontend
            │
            ▼
         backend
            │
            ▼
        database

       └───────────────────────────┘
```

No hardcoded IP addresses are required.

---

# DNS Resolution Flow

```
Application

↓

glibc Resolver

↓

/etc/resolv.conf

↓

127.0.0.11

↓

Docker Embedded DNS

      │
      ├────────► Container Names
      │
      ▼
Host DNS Resolver

↓

Internet DNS
```

Docker decides whether the query is:

- Internal
- External

---

# Docker Embedded DNS

Every **user-defined bridge network** includes a built-in DNS server.

Responsibilities:

- Resolve container names
- Resolve Docker Compose service names
- Forward Internet DNS queries

Think of it as a lightweight DNS server built into Docker.

---

# `/etc/resolv.conf`

Inside a container:

```bash
cat /etc/resolv.conf
```

Example:

```text
nameserver 127.0.0.11
options ndots:0
```

```
127.0.0.11
```

is Docker's embedded DNS resolver.

It is **not**:

- Google DNS
- Cloudflare DNS
- Your router

---

# External DNS Resolution

Suppose the container requests:

```
google.com
```

Flow:

```
Container

↓

127.0.0.11

↓

Host DNS

↓

Recursive Resolver

↓

Internet

↓

Google IP
```

Docker forwards external queries to the host's configured DNS servers unless overridden.

---

# Internal Container Resolution

Application:

```
frontend
```

requests:

```
database
```

Docker immediately returns:

```
database

↓

172.18.0.5
```

No Internet request is made.

---

# Container Name vs Hostname

These are different concepts.

Container Name:

```
backend
```

Hostname:

```bash
hostname
```

By default they are often related, but they can be configured independently.

Example:

```bash
docker run \
  --name backend \
  --hostname api-server
```

```
Container Name

backend

Hostname

api-server
```

---

# Default Bridge Network

Docker's default bridge:

```
docker0
```

Limitations:

- Automatic service discovery is limited.
- Name resolution between unrelated containers is not as seamless.
- Older workflows often relied on the deprecated `--link` feature.

---

# User-Defined Bridge Network

Create:

```bash
docker network create app-network
```

Now:

```
frontend

↓

backend
```

works automatically.

Best Practice:

Always use user-defined bridge networks for multi-container applications.

---

# Docker Compose DNS

Compose file:

```yaml
services:
  frontend:
  backend:
  redis:
```

Docker automatically creates:

- Network
- DNS entries

Application:

```
http://backend:8080
```

works immediately.

The service name becomes the hostname.

---

# Environment Variables

Instead of:

```text
DATABASE_HOST=172.18.0.5
```

Use:

```text
DATABASE_HOST=database
```

This survives container recreation.

---

# Multi-Network Containers

A container can belong to multiple networks.

Example:

```
frontend

│

├── frontend-network

└── backend-network
```

Docker chooses the appropriate DNS records based on the network used for communication.

---

# Production Example 1

Application:

```
Frontend
```

Needs:

```
Redis
```

Configuration:

```text
REDIS_HOST=redis
```

Docker resolves:

```
redis

↓

172.18.0.5
```

Redis restarts:

```
172.18.0.9
```

Application still connects using:

```
redis
```

---

# Production Example 2

Microservices:

```
Frontend

↓

API

↓

Auth

↓

Database
```

Connections:

```
http://api

http://auth

database
```

Never:

```
172.x.x.x
```

---

# Production Example 3

Command:

```bash
curl http://backend:8080
```

Flow:

```
curl

↓

127.0.0.11

↓

Docker DNS

↓

172.18.0.3

↓

HTTP Connection
```

---

# Docker DNS Configuration

Override DNS:

```bash
docker run \
  --dns 8.8.8.8
```

Search domain:

```bash
docker run \
  --dns-search company.local
```

Custom hostname:

```bash
docker run \
  --hostname api-server
```

These options are useful in enterprise environments.

---

# Useful Commands

View resolver:

```bash
cat /etc/resolv.conf
```

Inspect network:

```bash
docker network inspect app-network
```

Inspect container:

```bash
docker inspect backend
```

View hostname:

```bash
hostname
```

Test DNS:

```bash
nslookup backend
```

or

```bash
getent hosts backend
```

---

# Troubleshooting Checklist

When DNS fails:

1. Verify both containers are on the same network.
2. Check the network with `docker network inspect`.
3. Inspect `/etc/resolv.conf`.
4. Verify container names.
5. Test with `getent hosts`.
6. Check upstream DNS if external lookups fail.
7. Restart the affected container if necessary.

---

# Docker vs Kubernetes DNS

| Docker | Kubernetes |
|---------|------------|
| Embedded DNS (`127.0.0.11`) | CoreDNS |
| Container names | Service names |
| User-defined networks | Cluster-wide DNS |
| Simple service discovery | Advanced service discovery |

Understanding Docker DNS makes learning Kubernetes DNS much easier.

---

# Common Misconceptions

## ❌ Containers use Google DNS

False.

Containers usually query Docker's embedded resolver.

---

## ❌ Container IPs never change

False.

IP addresses often change after recreation.

---

## ❌ Docker Compose requires IP addresses

False.

Compose automatically provides DNS-based service discovery.

---

## ❌ Docker DNS and Kubernetes DNS are identical

False.

Docker provides simple name resolution.

Kubernetes provides cluster-wide service discovery through CoreDNS.

---

# Interview Questions

### What DNS server do Docker containers use?

```
127.0.0.11
```

on user-defined bridge networks.

---

### Why use container names instead of IPs?

Container IPs change.

Names remain stable.

---

### Does Docker Compose provide automatic DNS?

Yes.

Service names automatically become resolvable hostnames.

---

### Can Docker resolve Internet domains?

Yes.

Unknown names are forwarded to upstream DNS servers.

---

### Why are user-defined bridge networks recommended?

They provide automatic DNS-based service discovery between containers.

---

### What command shows network details?

```bash
docker network inspect app-network
```

---

# Key Takeaways

- Docker provides an embedded DNS resolver (`127.0.0.11`) on user-defined networks.
- Containers should communicate using container or service names, not IP addresses.
- Docker Compose automatically creates DNS records for service names.
- External DNS queries are forwarded to the host's configured DNS servers.
- Multi-container applications become easier to manage when they rely on DNS instead of static IP addresses.

---

# What's Next?

## Module 14 — DNS in Kubernetes

You'll learn:

- CoreDNS
- Kubernetes Service Discovery
- Cluster DNS Architecture
- Service DNS Names
- Pod DNS
- Search Domains
- Headless Services
- StatefulSets
- ExternalName Services
- Production DNS Troubleshooting