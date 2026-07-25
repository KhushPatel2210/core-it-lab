# The Complete Journey

Imagine you type:

```text
https://google.com
```

and press **Enter**.

Here's the high-level flow:

```text
You
 │
 ▼
Browser
 │
 ▼
Browser Cache
 │
 ▼
OS Cache
 │
 ▼
Hosts File
 │
 ▼
Recursive DNS Resolver
 │
 ▼
Root Server
 │
 ▼
TLD Server (.com)
 │
 ▼
Authoritative Server
 │
 ▼
Returns IP
 │
 ▼
Resolver Cache
 │
 ▼
Your Computer
 │
 ▼
Browser
 │
 ▼
TCP Connection
 │
 ▼
TLS Handshake (HTTPS)
 │
 ▼
HTTP Request
 │
 ▼
Google Website
```

Notice something:

DNS is only one part of loading a website.

After DNS finishes, networking and HTTP begin.

---

# Step 1 — Browser

You type:

```text
https://google.com
```

The browser first asks:

> "Do I already know Google's IP?"

It checks its own cache.

---

# Step 2 — Browser Cache

Example:

```text
google.com
      ↓
142.250.193.78
```

If found:

- ✔ Done.
- No DNS lookup.

If not:

Continue.

---

# Step 3 — OS Cache

Windows, Linux, and macOS also remember recent DNS lookups.

Example:

```text
google.com
      ↓
142.250.193.78
```

If found:

- Browser gets the IP.
- Done.

---

# Step 4 — Hosts File

The operating system checks:

### Linux

```text
/etc/hosts
```

### Windows

```text
C:\Windows\System32\drivers\etc\hosts
```

Suppose:

```text
192.168.1.50 google.com
```

exists.

Now your computer believes:

```text
google.com
      ↓
192.168.1.50
```

DNS servers are never contacted.

---

# Step 5 — Recursive DNS Resolver

Nothing found locally.

Now your computer asks its configured DNS resolver.

Maybe:

```text
8.8.8.8

or

1.1.1.1

or your ISP's DNS
```

Think of the resolver as your assistant.

Instead of you asking ten different DNS servers,

the resolver does all the work.

---

# Step 6 — Resolver Cache

Resolvers also have memory.

Suppose someone else asked:

```text
google.com
```

five seconds ago.

The resolver already knows.

It simply replies:

```text
142.250.193.78
```

Very fast.

No Root Server.

No TLD.

No Authoritative Server.

---

# Step 7 — Root Server

If the resolver has no cached answer,

it asks a Root Server.

### Question

> Where is `google.com`?

### Root replies

> I don't know.
>
> Ask the `.com` server.

---

# Step 8 — TLD Server

Resolver asks:

```text
.com server
```

### Question

> Where is `google.com`?

### Reply

> Ask Google's Authoritative Server.

Again,

no IP yet.

---

# Step 9 — Authoritative Server

Resolver asks Google's authoritative DNS server:

```text
google.com
```

Now finally:

```text
google.com
      ↓
142.250.193.78
```

Success.

---

# Step 10 — Cache the Answer

Before replying,

the resolver saves the answer in its cache.

Your operating system may also cache it.

Your browser may cache it too.

This prevents repeated lookups.

---

# Step 11 — Browser Gets the IP

Browser now knows:

```text
google.com
      ↓
142.250.193.78
```

DNS is finished.

---

# Step 12 — TCP Connection

Now the browser opens a TCP connection to:

```text
142.250.193.78
```

Usually on:

```text
Port 443
```

because HTTPS is being used.

No HTTP request has been sent yet.

---

# Step 13 — TLS Handshake

Because it's HTTPS,

the browser verifies:

- Server certificate
- Encryption
- Identity

Only after TLS succeeds is the connection considered secure.

---

# Step 14 — HTTP Request

Now the browser sends:

```http
GET /
Host: google.com
```

Notice something important.

The browser connected to:

```text
142.250.193.78
```

But it still sends:

```http
Host: google.com
```

This allows one server (or one load balancer) to host many websites.

Example:

```text
142.250.193.78
      ↓
google.com
gmail.com
maps.google.com
```

The server uses the **Host** header to determine which site the client wants.

---

# Step 15 — Website Loads

Finally:

Google sends:

- HTML
- CSS
- JavaScript
- Images
- Fonts

The browser renders the page.

You're now looking at Google.

---

# Visual Timeline

```text
User
 │
 ▼
Type google.com
 │
 ▼
Browser Cache
 │
 ▼
OS Cache
 │
 ▼
Hosts File
 │
 ▼
Recursive Resolver
 │
 ▼
Root Server
 │
 ▼
TLD Server
 │
 ▼
Authoritative Server
 │
 ▼
IP Address
 │
 ▼
TCP Connection
 │
 ▼
TLS Handshake
 │
 ▼
HTTP GET Request
 │
 ▼
Website Response
```

---

# Real Production Example

Suppose your company has:

```text
portal.company.com
```

The DNS record changes from:

```text
203.0.113.10
```

to

```text
203.0.113.20
```

A user may continue reaching the old server until cached DNS entries expire.

That's why DNS changes sometimes don't appear instantly.

This delay depends on the **TTL (Time To Live)** value, which we'll cover in the next module.


---

# 📝 Notes (Not Added to the Main Notes)

## 1. TTL (Time To Live)

> **Note:** TTL (Time To Live) is introduced here only to explain why DNS changes are not visible immediately. We'll cover TTL in detail in the next module, including how it controls how long DNS records remain cached.

---

## 2. Beyond DNS

> **Note:** The sections **TCP Connection**, **TLS Handshake**, and **HTTP Request** are included only to show the complete journey after DNS resolution. These topics belong to the Networking and HTTP modules and are intentionally kept at a high level here.