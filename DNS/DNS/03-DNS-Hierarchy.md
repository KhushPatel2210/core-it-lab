# The Biggest Question

When you type:

```text
google.com
```

How does your DNS resolver know Google's IP?

Did Google send its IP to every computer in the world?

**No.**

Imagine the Internet has:

- Billions of websites
- Millions of DNS servers

No single server can store every website's IP.

So, the Internet divides the work.

Think of it as a **hierarchy**.

---

# Imagine a Huge Library

Suppose you walk into the world's largest library.

You ask:

> "Where can I find the book Harry Potter?"

The receptionist doesn't know where every single book is.

Instead, she says:

> Go to Floor 3.

Floor 3 says:

> Go to Shelf B.

Shelf B says:

> Go to Row 5.

Row 5 finally gives you the book.

DNS works exactly the same way.

Nobody knows everything.

Each server only knows where to ask next.

---

# DNS Hierarchy

There are three main levels.

```text
            Root Server
                 │
                 ▼
          TLD Server (.com)
                 │
                 ▼
  Authoritative Server (google.com)
                 │
                 ▼
      Returns Google's IP
```

Let's understand each one.

---

# Level 1 — Root DNS Server

Think of the **Root Server** as the reception desk of the Internet.

It does **not** know Google's IP.

Instead, it knows:

> "If someone asks for a `.com` domain, ask the `.com` server."

For example, if you ask:

```text
google.com
```

The Root Server replies:

> I don't know Google's IP.
>
> But I know who manages `.com`.
>
> Go ask them.

That's all.

It doesn't answer the final question.

## What does the Root Server know?

It knows only where to find **Top-Level Domain (TLD)** servers.

### Examples

- `.com`
- `.net`
- `.org`
- `.edu`
- `.gov`
- `.in`

---

# Level 2 — TLD Server

**TLD** means:

> Top-Level Domain

### Examples

- `.com`
- `.org`
- `.net`
- `.io`
- `.dev`
- `.in`

Suppose you ask the `.com` server:

```text
google.com
```

The `.com` server replies:

> I don't know Google's IP.
>
> But I know who manages `google.com`.
>
> Go ask Google's Authoritative Server.

Notice something important.

Again...

**No IP address.**

Just directions.

---

# Level 3 — Authoritative Server

Finally, the resolver reaches Google's DNS server.

This server owns the DNS records for:

```text
google.com
```

Now it replies:

```text
google.com
↓
142.250.193.78
```

Finally...

The resolver has the answer.

---

# Complete Journey

Let's see everything together.

You type:

```text
google.com
```

Resolver asks:

> Root Server

Root replies:

> Ask `.com`

Resolver asks:

> `.com` Server

Reply:

> Ask Google's DNS Server

Resolver asks:

> Google Authoritative Server

Reply:

```text
142.250.193.78
```

The resolver stores it in cache.

The browser connects.

The website opens.

---

# Visual Diagram

```text
                google.com
                     │
                     ▼
               DNS Resolver
                     │
                     ▼
              Root DNS Server
                     │
      "Ask the .com server"
                     │
                     ▼
              .com TLD Server
                     │
  "Ask Google's DNS server"
                     │
                     ▼
    Google's Authoritative Server
                     │
      "142.250.193.78"
                     │
                     ▼
               DNS Resolver
                     │
                     ▼
                Browser
                     │
                     ▼
               Google Website
```

---

# Real-World Analogy

Imagine you're looking for your friend's house.

You ask the police station.

Police:

> I don't know the house.
>
> But I know which city.

You go to the city office.

City office:

> I don't know the house.
>
> But I know which street.

You go to the street.

Street office:

> House Number 45.

Found it.

DNS is exactly this process.

Each level gives you the next clue.

---

# Why Doesn't the Root Server Store Every Website?

Imagine storing information for:

- Google
- Amazon
- Netflix
- Microsoft
- Facebook
- Every university
- Every startup
- Every company
- Every personal website

That's billions of domains.

One server cannot handle that amount of data or traffic.

Instead, DNS is distributed.

Each server has a small, specific responsibility.

---

# Does the Root Server Ever Change?

Rarely.

There are only **13 logical root server identities**, named:

- A Root
- B Root
- C Root
- ...
- M Root

Each logical server is actually backed by many physical servers around the world using **Anycast**, so users reach a nearby instance rather than a single machine.

---

# What is an Authoritative Server?

This is the source of truth for a domain.

Example:

```text
google.com
```

Google owns DNS records like:

- A Record
- AAAA Record
- MX Record
- TXT Record
- CNAME Record

We'll study these records in the next module.

---

# Does Every Website Have Its Own Authoritative Server?

Not necessarily.

A company can:

- Run its own DNS servers, or
- Use a DNS provider such as AWS Route 53 or Cloudflare to host its authoritative DNS.

The key point is that the authoritative server is responsible for the domain's official DNS records.

---

# Production Example

Suppose your company owns:

```text
api.company.com
```

Your DNS provider stores:

```text
api.company.com
↓
34.120.10.15
```

When a user's resolver reaches the authoritative server, it receives:

```text
34.120.10.15
```

The resolver caches it.

The browser connects.