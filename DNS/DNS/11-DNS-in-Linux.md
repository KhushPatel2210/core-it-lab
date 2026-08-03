# Module 11 — DNS in Linux

> **Difficulty:** Intermediate  
> **Prerequisites:** DNS Resolution, Forward Lookup, Reverse Lookup, Public vs Private DNS

---

# Learning Objectives

By the end of this chapter, you'll understand:

- How Linux resolves a hostname
- The role of glibc
- What `/etc/nsswitch.conf` does
- What `/etc/hosts` does
- What `/etc/resolv.conf` does
- How `systemd-resolved` works
- DNS search domains
- DNS caching
- Kubernetes & Docker DNS
- Common troubleshooting commands
- Production debugging scenarios
- Interview questions

---

# Why Learn DNS in Linux?

Imagine you're SSH'd into a production server.

A developer reports:

> "The application cannot connect to `api.company.com`."

Before blaming DNS, you should answer:

- Which DNS server is the machine using?
- Is `/etc/hosts` overriding DNS?
- Is the resolver working?
- Is `systemd-resolved` healthy?
- Is the hostname actually resolving?

Understanding Linux name resolution is the foundation of production troubleshooting.

---

# Linux DNS Resolution Flow

```
Application
      │
      ▼
glibc Resolver
      │
      ▼
/etc/nsswitch.conf
      │
      ├────────────► /etc/hosts
      │
      ▼
systemd-resolved (optional)
      │
      ▼
/etc/resolv.conf
      │
      ▼
Recursive DNS Server
      │
      ▼
Authoritative DNS
      │
      ▼
IP Address
```

---

# Step 1 — Application

Almost every network application needs DNS.

Examples:

```bash
ping google.com
curl google.com
ssh server.company.com
wget example.com
git clone https://github.com/user/repo.git
```

Applications usually **do not implement DNS themselves**.

They ask the operating system.

---

# Step 2 — glibc Resolver

Most Linux applications use the **glibc resolver library**.

Its job:

```
Hostname

↓

IP Address
```

glibc doesn't immediately contact a DNS server.

Instead, it follows the system's configured name resolution order.

---

# Step 3 — `/etc/nsswitch.conf`

This file tells Linux:

> "Where should I search for hostnames?"

Example:

```text
hosts: files dns
```

Meaning:

1. Check `/etc/hosts`
2. Query DNS

If configured as:

```text
hosts: dns files
```

Linux would:

1. Query DNS
2. Check `/etc/hosts`

Order matters.

---

# View the Configuration

```bash
cat /etc/nsswitch.conf
```

Typical output:

```text
hosts: files dns
```

---

# Step 4 — `/etc/hosts`

Before DNS, Linux checks:

```text
/etc/hosts
```

Example:

```text
127.0.0.1 localhost
192.168.1.100 dev-server
10.0.1.20 database
```

Now:

```bash
ping database
```

works without any DNS server.

---

# Common Uses of `/etc/hosts`

- Local development
- Temporary testing
- Emergency overrides
- Blocking domains
- Recovery when DNS is unavailable

Example:

```text
203.0.113.50 api.company.com
```

Every lookup for `api.company.com` now returns:

```
203.0.113.50
```

DNS is never queried.

---

# Step 5 — `/etc/resolv.conf`

If the hostname isn't found locally, Linux checks:

```text
/etc/resolv.conf
```

Example:

```text
nameserver 8.8.8.8
nameserver 1.1.1.1
search company.local
```

This file specifies:

- DNS servers
- Search domains
- Resolver options

It **does not** contain DNS records.

---

# Understanding `nameserver`

```text
nameserver 8.8.8.8
```

Primary DNS server.

If unavailable:

```
8.8.8.8 ❌

↓

1.1.1.1
```

Linux normally tries the next configured server.

---

# Understanding `search`

Example:

```text
search company.local
```

Suppose you run:

```bash
ssh server01
```

Linux automatically tries:

```
server01.company.local
```

Search domains reduce typing inside organizations.

---

# Understanding `options`

Example:

```text
options timeout:2 attempts:3
```

These settings control:

- DNS timeout
- Retry count
- Resolver behavior

---

# Step 6 — Query DNS

Now Linux sends the query.

```
google.com

↓

Recursive Resolver

↓

Internet DNS

↓

142.250.xxx.xxx
```

The application receives the IP and connects.

---

# What is `systemd-resolved`?

Modern Linux distributions often use:

```
systemd-resolved
```

Applications communicate with:

```
127.0.0.53
```

instead of directly contacting external DNS servers.

Flow:

```
Application

↓

127.0.0.53

↓

systemd-resolved

↓

Configured DNS Server

↓

Internet
```

---

# Why Does `/etc/resolv.conf` Show `127.0.0.53`?

Example:

```text
nameserver 127.0.0.53
```

This is **not** Google DNS.

It is the local stub resolver provided by `systemd-resolved`.

The real upstream DNS servers can be viewed with:

```bash
resolvectl status
```

---

# DNS Caching

Some Linux systems cache DNS responses.

Examples:

- systemd-resolved
- nscd
- dnsmasq

Benefits:

- Faster lookups
- Reduced DNS traffic

---

# Search Domains in Kubernetes

Typical Pod:

```text
search default.svc.cluster.local svc.cluster.local cluster.local
```

Application requests:

```
redis
```

Linux expands it to:

```
redis.default.svc.cluster.local
```

This is why Pods can use short service names.

---

# Docker DNS

Containers use an embedded DNS server.

Typical:

```text
127.0.0.11
```

Docker automatically resolves container names on user-defined networks.

---

# Useful Commands

View DNS servers:

```bash
cat /etc/resolv.conf
```

View hosts file:

```bash
cat /etc/hosts
```

View NSS configuration:

```bash
cat /etc/nsswitch.conf
```

View resolver status:

```bash
resolvectl status
```

Query DNS directly:

```bash
dig google.com
```

Simple lookup:

```bash
nslookup google.com
```

Another lookup tool:

```bash
host google.com
```

---

# `getent hosts` (Very Important)

Most beginners test only with `dig`.

But applications use the NSS stack.

Better test:

```bash
getent hosts google.com
```

`getent` follows:

- `/etc/hosts`
- NSS
- DNS

making it closer to how real applications resolve names.

---

# Production Example 1

Developer:

> "`api.company.com` opens the wrong server."

Check:

```bash
cat /etc/hosts
```

Output:

```text
192.168.1.50 api.company.com
```

Problem solved.

Local override.

---

# Production Example 2

```
dig google.com
```

works.

But:

```bash
ping google.com
```

fails.

Possible causes:

- Broken NSS configuration
- Incorrect `/etc/hosts`
- `systemd-resolved` failure
- Resolver misconfiguration

---

# Production Example 3

Kubernetes Pod:

```text
nameserver 10.96.0.10
```

```
search default.svc.cluster.local
```

Application requests:

```
redis
```

Automatically becomes:

```
redis.default.svc.cluster.local
```

---

# Production Troubleshooting Checklist

When DNS fails:

1. Check `/etc/hosts`
2. Check `/etc/resolv.conf`
3. Verify DNS server reachability
4. Inspect `systemd-resolved`
5. Test with `getent hosts`
6. Test with `dig`
7. Check firewall rules
8. Verify search domains
9. Flush DNS cache if needed

---

# Common Misconceptions

## ❌ `/etc/resolv.conf` stores DNS records

False.

It stores DNS **server configuration**, not records.

---

## ❌ `/etc/hosts` is a DNS server

False.

It's only a local hostname mapping file.

---

## ❌ Linux always uses Google DNS

False.

Linux uses whatever resolver is configured.

---

## ❌ `127.0.0.53` is Google's DNS

False.

It's the local `systemd-resolved` stub resolver.

---

## ❌ `dig` proves applications can resolve names

Not always.

`dig` queries DNS directly.

Applications usually use NSS.

Use:

```bash
getent hosts hostname
```

for realistic testing.

---

# Interview Questions

### Which file defines DNS servers?

```
/etc/resolv.conf
```

---

### Which file overrides DNS locally?

```
/etc/hosts
```

---

### What does `/etc/nsswitch.conf` do?

Defines the order of hostname resolution sources such as local files and DNS.

---

### What is `127.0.0.53`?

The local stub resolver used by `systemd-resolved`.

---

### Why is `/etc/hosts` checked first?

Because many systems configure:

```text
hosts: files dns
```

---

### Why use `getent hosts` instead of only `dig`?

Because `getent` follows the same NSS resolution path used by most Linux applications.

---

# Key Takeaways

- Applications typically use the **glibc resolver**.
- `/etc/nsswitch.conf` determines the hostname lookup order.
- `/etc/hosts` can override DNS.
- `/etc/resolv.conf` defines DNS servers and search domains.
- `systemd-resolved` often acts as a local DNS stub resolver.
- `getent hosts` is the best way to test hostname resolution as applications see it.
- Kubernetes and Docker both provide their own DNS mechanisms for service discovery.

---

# What's Next?

## Module 12 — DNS Troubleshooting

You'll learn how to troubleshoot DNS failures using:

- `dig`
- `nslookup`
- `host`
- `getent`
- `resolvectl`
- `tcpdump`
- `curl --resolve`
- `ping`
- `traceroute`

along with real-world production scenarios and a systematic debugging workflow.