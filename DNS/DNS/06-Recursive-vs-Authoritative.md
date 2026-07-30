# Module 06 — Recursive Resolver vs Authoritative Name Server

## 📁 File

```text
DNS/
├── 06-Recursive-vs-Authoritative.md
```

---

# Learning Objectives

After this chapter, you'll be able to answer:

- What is a Recursive Resolver?
- What is an Authoritative Name Server?
- How are they different?
- Who owns them?
- Which one caches data?
- Which one is the source of truth?
- How do they work together?

---

# The Biggest Misconception

Many beginners think:

> "A DNS server is just a DNS server."

Not true.

There are different types of DNS servers, each with a specific responsibility.

The two you'll encounter the most are:

1. Recursive Resolver
2. Authoritative Name Server

---

# Think of a Restaurant

Imagine you go to a restaurant.

You don't walk into the kitchen.

Instead,

You ask the waiter:

> "I'd like a pizza."

The waiter doesn't cook it.

He goes to the kitchen.

The chef prepares it.

The waiter brings it back.

DNS works exactly the same way.

```text
You
 │
 ▼
Recursive Resolver (Waiter)
 │
 ▼
Authoritative Server (Chef)
 │
 ▼
Returns Answer
```

The waiter doesn't create the food.

The chef does.

---

# What is a Recursive Resolver?

A **Recursive Resolver** is the DNS server your computer talks to.

### Examples

**Google DNS**

```text
8.8.8.8
```

---

**Cloudflare**

```text
1.1.1.1
```

---

**Quad9**

```text
9.9.9.9
```

---

**Your ISP DNS**

Its job is simple:

> Find the answer for the client.

If it doesn't know the answer,

it will:

- Ask Root Servers
- Ask TLD Servers
- Ask Authoritative Servers

until it finds the answer.

---

# Example

You ask:

```text
google.com
```

Your laptop asks:

```text
8.8.8.8
```

Google DNS says:

> "Don't worry.
>
> I'll find it."

It does all the work.

Then replies:

```text
142.250.193.78
```

---

# Does the Recursive Resolver Store Data?

**Yes.**

This is one of its biggest jobs.

It caches answers.

### Example

Yesterday:

```text
google.com
        ↓
142.250.193.78
```

Today,

another user asks:

```text
google.com
```

Instead of asking Google again,

the resolver immediately replies from cache.

Much faster.

---

# What is an Authoritative Name Server?

The **Authoritative Name Server** is the official owner of the DNS records for a domain.

It stores the real records.

### Example

```text
example.com
      ↓
A Record
      ↓
192.168.1.20
```

Nobody else decides this IP.

Only the authoritative server.

It is the **source of truth**.

---

# Example

Suppose you own:

```text
mycompany.com
```

Inside your DNS provider:

```text
A Record
      ↓
34.210.10.15
```

This record exists on the authoritative server.

When the recursive resolver asks:

> "What's the IP of mycompany.com?"

The authoritative server replies:

```text
34.210.10.15
```

---

# Visual Flow

```text
               User
                │
                ▼
      Recursive Resolver
                │
                ▼
        Root DNS Server
                │
                ▼
          TLD Server
                │
                ▼
     Authoritative Server
                │
                ▼
      Returns Official IP
                │
                ▼
      Recursive Resolver
                │
                ▼
             User
```

Notice something.

The user never contacts the authoritative server directly.

The recursive resolver does.

---

# Real Production Example

Suppose your company uses **AWS Route 53**.

Your DNS records are:

```text
portal.company.com
        ↓
52.12.10.20
```

These records live in Route 53's authoritative name servers.

Now your customer opens:

```text
portal.company.com
```

Their laptop asks:

```text
8.8.8.8
```

Google DNS (recursive resolver) eventually contacts Route 53's authoritative server.

Route 53 replies:

```text
52.12.10.20
```

Google DNS caches the answer.

Customer gets the IP.

---

# Who Owns Which Server?

| Server | Usually Owned By |
|----------|------------------|
| Recursive Resolver | ISP, Google DNS, Cloudflare, Quad9, Company |
| Authoritative Server | Domain owner or DNS provider (Cloudflare, Route 53, GoDaddy, etc.) |

---

# Which One Caches?

| Server | Cache? |
|----------|---------|
| Recursive Resolver | ✅ Yes |
| Authoritative Server | ❌ No (it serves the official records) |

Think of it like this:

```text
Recursive Resolver
        ↓
      Memory
        ↓
       Fast

------------------------

Authoritative Server
        ↓
 Official Database
        ↓
      Truth
```

---

# Which One Can Change Records?

Only the authoritative server.

### Example

Old:

```text
example.com
      ↓
10.0.0.10
```

New:

```text
example.com
      ↓
10.0.0.20
```

You change it on the authoritative server.

Recursive resolvers cannot edit your records.

They only cache the responses they receive.

---

# Production Troubleshooting Scenario

A customer reports:

> "Your website still opens the old server."

You check the authoritative DNS:

```text
example.com
      ↓
10.0.0.20
```

Correct.

But the customer's recursive resolver still has:

```text
10.0.0.10
```

cached.

The issue isn't the authoritative server—it's stale cache in the recursive resolver.

This is why **TTL** matters.

---

# Comparison Table

| Feature | Recursive Resolver | Authoritative Server |
|----------|--------------------|----------------------|
| Talks to users? | ✅ Yes | ❌ Usually no |
| Stores official records? | ❌ No | ✅ Yes |
| Caches answers? | ✅ Yes | ❌ No |
| Can modify DNS records? | ❌ No | ✅ Yes |
| Performs DNS lookups? | ✅ Yes | ❌ No |
| Source of truth? | ❌ No | ✅ Yes |

---

# Interview Questions

## Q1. What is a Recursive Resolver?

A DNS server that receives queries from clients, performs DNS lookups if needed, caches the results, and returns the answer.

---

## Q2. What is an Authoritative Name Server?

A DNS server that stores the official DNS records for a domain and provides the authoritative answer.

---

## Q3. Which DNS server caches responses?

The Recursive Resolver.

---

## Q4. Which DNS server is the source of truth?

The Authoritative Name Server.

---

## Q5. Can a recursive resolver change a domain's DNS records?

No. It can only cache and return responses from authoritative servers.

---

# Summary

Think of the relationship like this:

```text
You
 │
 ▼
Recursive Resolver
(Finds the answer)
 │
 ▼
Authoritative Server
(Owns the answer)
 │
 ▼
Returns the official DNS record
 │
 ▼
Recursive Resolver caches it
 │
 ▼
You receive the IP
```

---

# One important note before the next module

In the DNS lookup process we've learned so far, the recursive resolver performs the entire lookup on your behalf. But there's another concept called an **iterative query**, where a DNS server doesn't fetch the final answer—it simply tells the requester where to ask next.

That's exactly what the **Root** and **TLD** servers do.

Understanding **Recursive vs Iterative Queries** is the missing piece that will make **`dig +trace`** completely intuitive, because `dig +trace` lets you watch those iterative referrals happen step by step.

That should be your next chapter before moving on to TTL or the DNS command-line tools.