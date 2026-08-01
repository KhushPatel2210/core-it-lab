# Module 07 — DNS Caching

## 📁 File

```text
DNS/
├── 07-DNS-Caching.md
```

---

# Learning Objectives

After this chapter, you'll be able to answer:

- What is DNS Caching?
- Why is DNS caching important?
- Where does DNS caching happen?
- What is TTL (Time To Live)?
- Why does TTL exist?
- What is DNS Propagation?
- Why do DevOps engineers lower TTL before migrations?
- How can you clear the DNS cache?

---

# What is DNS Caching?

Imagine you ask your friend:

> "What's Khush's phone number?"

He tells you:

```text
9876543210
```

Instead of asking him again tomorrow,

you save it in your phone.

Next time you don't ask.

You already know it.

That's caching.

DNS works exactly the same way.

Instead of asking DNS servers every single time,

computers temporarily remember DNS answers.

---

# Why Do We Need DNS Caching?

Imagine there were no caching.

Every time you opened:

```text
google.com
```

your computer would have to ask:

```text
Root
 ↓
TLD
 ↓
Authoritative
```

Millions of users.

Millions of DNS lookups.

Huge load.

Slower websites.

DNS caching solves this problem.

---

# Without Caching

```text
You
 │
 ▼
Root Server
 │
 ▼
TLD Server
 │
 ▼
Authoritative
 │
 ▼
Returns IP
```

Every single visit.

Very inefficient.

---

# With Caching

### First Visit

```text
You
 │
 ▼
Recursive Resolver
 │
 ▼
Authoritative
 │
 ▼
Returns IP
 │
 ▼
Cache Saved
```

---

### Second Visit

```text
You
 │
 ▼
Recursive Resolver
 │
 ▼
Cache
 │
 ▼
Returns IP
```

Much faster.

---

# Where Does DNS Caching Happen?

Many beginners think there's only one cache.

Actually, there are multiple caches.

```text
Browser Cache
       │
       ▼
Operating System Cache
       │
       ▼
Recursive Resolver Cache
```

Let's understand each.

---

# 1. Browser Cache

Your browser remembers recently visited domains.

Example:

```text
google.com
      ↓
142.250.193.78
```

Next time,

Chrome already knows the answer.

No DNS request is needed.

---

# 2. Operating System Cache

Windows, Linux, and macOS also maintain a DNS cache.

Example:

```text
google.com
      ↓
142.250.193.78
```

Even if you close Chrome,

another browser can benefit from the OS cache.

---

# 3. Recursive Resolver Cache

This is the biggest cache.

Suppose:

One million users ask:

```text
google.com
```

Google DNS (**8.8.8.8**) stores the answer.

Instead of asking Google's authoritative server every time,

it simply replies from cache.

---

# Visual Flow

```text
                Browser Cache
                      │
                      ▼
                 OS Cache
                      │
                      ▼
           Recursive Resolver Cache
                      │
                      ▼
          Authoritative DNS Server
```

Each layer checks its own cache before moving on.

---

# What is TTL?

TTL stands for:

> **Time To Live**

It tells caches:

> "Keep this DNS answer for this amount of time."

### Example

```text
google.com
      ↓
142.250.193.78

TTL = 300
```

300 seconds

=

5 minutes.

After five minutes,

the cache expires.

The next request triggers a fresh lookup.

---

# Example

Suppose:

```text
example.com
      ↓
192.168.1.10
```

TTL:

```text
300
```

Timeline:

```text
10:00
  ↓
Resolver asks DNS
  ↓
Gets IP
  ↓
Stores Cache
  ↓
Expires
10:05
```

After **10:05**,

the next request performs another DNS lookup.

---

# Why Does TTL Exist?

Suppose your website moves to a new server.

Old IP:

```text
203.0.113.10
```

New IP:

```text
203.0.113.20
```

If caches never expired,

everyone would keep using the old IP forever.

TTL ensures caches eventually refresh.

---

# Production Example

Imagine you own:

```text
portal.company.com
```

Yesterday:

```text
A Record
      ↓
34.100.10.10
```

Today you migrate to a new server:

```text
34.100.10.20
```

But customers still reach:

```text
34.100.10.10
```

Why?

Because their recursive resolver still has the old record cached.

Nothing is "wrong" with DNS.

The cache simply hasn't expired yet.

---

# What is DNS Propagation?

This phrase is often misunderstood.

People say:

> "Wait for DNS propagation."

DNS records are **not physically copied** to every DNS server worldwide.

What's actually happening is that different caches expire at different times.

As those caches expire and perform fresh lookups, they receive the updated DNS records.

So **"DNS propagation"** is mostly **cache expiration and refresh**, not the internet slowly spreading a record everywhere.

---

# Why Do DevOps Engineers Lower TTL Before Migrations?

Suppose your TTL is:

```text
86400
```

That's:

```text
24 hours
```

If you change your server,

users may continue using the old IP for up to a day.

Instead, before the migration you lower the TTL to:

```text
300
```

(5 minutes)

Now caches refresh much sooner.

After the migration is complete and stable, you can increase the TTL again.

This is a common production practice.

---

# Can We Clear DNS Cache?

Yes.

## Windows

```powershell
ipconfig /flushdns
```

---

## Linux (depends on the DNS service)

Examples:

```bash
sudo resolvectl flush-caches
```

or

```bash
sudo systemd-resolve --flush-caches
```

Some Linux systems don't run a local DNS cache at all, while others use services like **systemd-resolved**, **dnsmasq**, or **nscd**.



# 📝 Notes (Not Added to the Main Notes)

## 1. TTL (Time To Live)

> **Note:** TTL (Time To Live) is introduced here only to explain how long DNS responses stay in cache. We'll cover TTL in more depth later, including how different DNS records use TTL, how recursive resolvers honor it, and why choosing the right TTL is important for performance and DNS migrations.

---

## 2. DNS Propagation

> **Note:** DNS propagation is introduced here because it is closely related to caching. In practice, what people call "DNS propagation" is usually cache expiration and refresh rather than DNS records being physically copied across the internet. We'll revisit this concept again when discussing DNS troubleshooting and production migrations.

---

## 3. Browser Cache vs OS Cache vs Recursive Resolver Cache

> **Note:** This chapter introduces the three main DNS cache layers at a high level. Later, when we learn DNS troubleshooting and tools like `ipconfig`, `resolvectl`, `dig`, and `nslookup`, you'll see how to inspect, clear, and verify each cache independently.

---

## 4. Cache Flushing Commands

> **Note:** The cache flush commands shown here are only examples. We'll cover platform-specific DNS troubleshooting, cache inspection, and cache clearing in a dedicated DNS troubleshooting module.

---

## 5. Recursive Resolver

> **Note:** This chapter assumes you already understand what a Recursive Resolver is. If not, review the previous module, **Recursive Resolver vs Authoritative Name Server**, before continuing.