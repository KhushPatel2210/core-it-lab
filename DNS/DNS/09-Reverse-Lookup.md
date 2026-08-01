# Module 09 — Reverse Lookup

> **Difficulty:** Beginner → Intermediate  
> **Prerequisites:** DNS Records, DNS Resolution, Forward Lookup

---

# Learning Objectives

By the end of this chapter, you'll understand:

- What is Reverse Lookup?
- Why do we need it?
- How does it work?
- What is a PTR Record?
- What is `in-addr.arpa`?
- IPv6 reverse lookups
- Forward-Confirmed Reverse DNS (FCrDNS)
- Production use cases
- Common troubleshooting techniques
- Interview questions

---

# Forward Lookup vs Reverse Lookup

## Forward Lookup

```
google.com
      │
      ▼
142.250.193.78
```

Question:

> "What is the IP address of google.com?"

---

## Reverse Lookup

```
142.250.193.78
      │
      ▼
google.com
```

Question:

> "Which hostname is associated with this IP address?"

Notice that the direction is exactly the opposite.

---

# What is Reverse Lookup?

A **Reverse DNS Lookup** converts an:

```
IP Address
      │
      ▼
Hostname
```

Instead of asking:

> "Where is google.com?"

You ask:

> "Who is using this IP address?"

Unlike Forward Lookup, Reverse Lookup uses a different DNS record called a **PTR Record**.

---

# Real-Life Analogy

Imagine someone calls your phone.

Instead of seeing only:

```
+91 9876543210
```

Your phone displays:

```
Khush
```

Your contacts performed a reverse lookup.

DNS works similarly.

```
IP Address
      │
      ▼
Hostname
```

---

# Which DNS Record is Used?

Forward Lookup:

- A Record
- AAAA Record

Reverse Lookup:

- PTR Record

PTR stands for:

> **Pointer Record**

Example:

```
203.0.113.20

↓

server.company.com
```

The PTR record points an IP address back to a hostname.

---

# How Reverse Lookup Works

Suppose you have:

```
203.0.113.20
```

The DNS resolver asks:

> "Is there a PTR record for this IP?"

DNS searches the reverse lookup namespace.

If found:

```
203.0.113.20

↓

PTR

↓

server.company.com
```

The hostname is returned.

---

# Why Doesn't DNS Search Every A Record?

Imagine searching billions of domains looking for one IP address.

That would be extremely inefficient.

Instead, DNS maintains an entirely separate reverse lookup namespace dedicated to PTR records.

---

# Understanding `in-addr.arpa`

IPv4 reverse lookups don't directly search an IP address.

Instead, DNS transforms it into a special domain.

Example:

```
203.0.113.20
```

becomes

```
20.113.0.203.in-addr.arpa
```

Two things happen:

- Octets are reversed.
- `.in-addr.arpa` is appended.

---

# Why Are the Octets Reversed?

DNS is hierarchical.

It resolves domains from right to left.

Example:

```
192.168.10.25
```

becomes

```
25.10.168.192.in-addr.arpa
```

Hierarchy:

```
arpa
 │
 ▼
in-addr
 │
 ▼
192
 │
 ▼
168
 │
 ▼
10
 │
 ▼
25
```

This allows organizations to delegate reverse DNS zones efficiently.

---

# IPv6 Reverse Lookup

IPv6 uses:

```
ip6.arpa
```

instead of

```
in-addr.arpa
```

The IPv6 address is converted into a reversed hexadecimal representation.

You don't need to memorize the entire format.

Remember:

| IP Version | Reverse Zone |
|------------|--------------|
| IPv4 | `in-addr.arpa` |
| IPv6 | `ip6.arpa` |

---

# Complete Reverse Lookup Flow

```
IP Address

      │
      ▼

Convert to Reverse Zone

      │
      ▼

Query PTR Record

      │
      ▼

PTR Record Found?

 ┌──────────────┐
 │              │
 │ Yes          │
 │              │
 ▼              ▼

Hostname     NXDOMAIN / No PTR
```

---

# What Happens If No PTR Record Exists?

Many IP addresses simply don't have a PTR record.

Example:

```
dig -x 198.51.100.10
```

Output:

```
NXDOMAIN
```

or

```
No PTR Record Found
```

This is completely normal.

---

# Forward-Confirmed Reverse DNS (FCrDNS)

Many mail servers perform an additional validation.

Step 1:

Reverse Lookup

```
203.0.113.25

↓

mail.company.com
```

Step 2:

Forward Lookup

```
mail.company.com

↓

203.0.113.25
```

If both match:

✅ Forward-Confirmed Reverse DNS

This helps reduce spam and spoofing.

---

# Who Manages PTR Records?

This is a common interview question.

Unlike A Records, PTR records are usually managed by whoever controls the IP address allocation.

Examples:

- ISP
- Cloud Provider
- Hosting Provider
- Enterprise Network Administrator

If you own a domain but not the IP range, you usually cannot create PTR records yourself.

---

# Production Example 1 — Mail Servers

A mail server sends email from:

```
203.0.113.25
```

Receiving mail server performs:

```
PTR Lookup

↓

mail.company.com
```

If no valid PTR exists, the email may be marked as suspicious or spam.

---

# Production Example 2 — Firewall Logs

Firewall log:

```
203.0.113.20
```

Reverse Lookup:

```
vpn.company.com
```

The logs become much easier to read.

---

# Production Example 3 — Monitoring

Monitoring Alert:

```
10.0.4.25
```

Reverse DNS:

```
db-prod.company.local
```

Engineers immediately know which server generated the alert.

---

# Production Example 4 — AWS

Suppose an EC2 instance has an Elastic IP.

```
54.x.x.x
```

AWS can associate a reverse DNS record with the Elastic IP (for supported services and configurations), which is especially useful for outbound mail servers.

---

# Production Example 5 — Kubernetes

Inside Kubernetes, reverse lookups can also exist depending on cluster DNS configuration.

Example:

```
10.96.5.12

↓

api.default.svc.cluster.local
```

This helps with diagnostics and service discovery.

---

# Troubleshooting Commands

### dig

```bash
dig -x 8.8.8.8
```

Output:

```
dns.google
```

---

### nslookup

```bash
nslookup 8.8.8.8
```

---

### host

```bash
host 8.8.8.8
```

---

# Reverse DNS Troubleshooting Checklist

When reverse DNS isn't working:

1. Verify the IP address.
2. Check whether a PTR record exists.
3. Confirm the correct reverse zone (`in-addr.arpa` or `ip6.arpa`).
4. Validate that the hostname resolves back to the same IP (FCrDNS).
5. Check DNS propagation and caching.
6. Ensure the cloud provider or ISP has configured the PTR record.

---

# Common Misconceptions

## ❌ Every IP has a PTR Record

False.

Many IP addresses do not have reverse DNS configured.

---

## ❌ Reverse Lookup Always Returns the Same Domain

Not necessarily.

Forward:

```
app.company.com

↓

203.0.113.20
```

Reverse:

```
203.0.113.20

↓

server-01.company.com
```

Both can be perfectly valid.

---

## ❌ PTR Records Prove Ownership

False.

A PTR record only indicates the hostname configured for an IP address.

It does **not** prove legal ownership of a domain or IP address.

---

# Interview Questions

## What is Reverse DNS Lookup?

Converting an IP address into a hostname using a PTR record.

---

## Which DNS record is used?

PTR Record.

---

## What is `in-addr.arpa`?

A special DNS namespace used for IPv4 reverse lookups.

---

## Why do mail servers rely on Reverse DNS?

To verify sender configuration and reduce spam.

---

## What is FCrDNS?

A security check where:

- Reverse Lookup → Hostname
- Forward Lookup → Same IP

Both should match.

---

## Which command performs a Reverse Lookup?

```bash
dig -x <IP_ADDRESS>
```

Example:

```bash
dig -x 8.8.8.8
```

---

# Key Takeaways

- Reverse Lookup converts **IP Address → Hostname**.
- It uses **PTR Records**, not A or AAAA records.
- IPv4 reverse lookups use **`in-addr.arpa`**.
- IPv6 reverse lookups use **`ip6.arpa`**.
- Mail servers, monitoring tools, security platforms, SIEMs, and firewalls frequently rely on reverse DNS.
- A missing PTR record is common and does not necessarily indicate a problem.
- FCrDNS is widely used to validate mail server identity.

---

# What's Next?

## Module 10 — DNS Propagation & TTL

You'll learn:

- What is DNS propagation?
- Why DNS changes aren't visible immediately
- TTL (Time To Live)
- Recursive resolver caching
- Browser & OS DNS cache behavior
- Cache invalidation
- How to speed up DNS migrations
- Real-world troubleshooting scenarios