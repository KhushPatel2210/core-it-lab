# What is a DNS Record?

A **DNS record** is simply an entry stored on the authoritative DNS server.

Example:

```text
google.com
      ↓
142.250.193.78
```

That mapping is stored as a DNS record.

Different record types store different kinds of information.

---

# Think of DNS as an Employee Database

Imagine a company database.

| Employee | Information |
|----------|-------------|
| Khush | Phone Number |
| Rahul | Email |
| Priya | Address |
| Amit | Department |

Every row contains different information.

DNS works exactly the same way.

```text
google.com
      ↓
   A Record
      ↓
142.250.193.78
```

Another record might store Google's mail server.

```text
google.com
      ↓
   MX Record
      ↓
mail.google.com
```

Same domain.

Different information.

---

# The Most Important DNS Records

- A
- AAAA
- CNAME
- MX
- TXT
- NS
- PTR
- SOA

Let's learn each one.

---

# 1. A Record (Address Record)

This is the most common DNS record.

Its job:

> Map a domain name to an IPv4 address.

## Example

```text
example.com
      ↓
192.168.10.15
```

### Meaning

When someone visits:

```text
https://example.com
```

Connect to:

```text
192.168.10.15
```

## Real Production Example

Suppose your AWS EC2 instance has:

```text
54.210.100.20
```

You don't want users to type:

```text
http://54.210.100.20
```

Instead you create:

```text
A Record

mycompany.com
      ↓
54.210.100.20
```

Now users simply visit:

```text
https://mycompany.com
```

---

# 2. AAAA Record

Exactly the same idea as an A record.

The only difference:

It stores an IPv6 address.

## Example

```text
example.com
      ↓
2001:db8::1234
```

Think of it as:

| Record | IP Version |
|---------|------------|
| A | IPv4 |
| AAAA | IPv6 |

---

# 3. CNAME Record (Canonical Name)

This one confuses many beginners.

A **CNAME** does **not** point directly to an IP.

Instead, it points to another domain name.

## Example

```text
www.example.com
        ↓
example.com
```

Now the resolver looks up:

```text
example.com
```

and finds its **A** or **AAAA** record.

## Why use CNAME?

Suppose your IP changes.

Without CNAME:

```text
www.example.com
        ↓
192.168.1.10

api.example.com
        ↓
192.168.1.10
```

You must update multiple records.

With CNAME:

```text
www.example.com
        ↓
example.com
```

Only the **A record** for `example.com` needs updating.

---

# 4. MX Record (Mail Exchange)

MX records tell the internet:

> Which server receives email for this domain?

## Example

```text
example.com
      ↓
mail.example.com
```

When someone sends an email to:

```text
khush@example.com
```

The sender's mail server first asks DNS:

> "Who handles email for example.com?"

DNS replies:

```text
mail.example.com
```

The email is delivered there.

Without MX records, email delivery wouldn't know where to go.

---

# 5. TXT Record

TXT records store text information.

Originally they were just arbitrary text.

Today they're widely used for:

- SPF (who can send email)
- DKIM (email signing)
- DMARC (email policies)
- Domain ownership verification
- Google Workspace verification
- Microsoft 365 verification

## Example

```text
TXT

google-site-verification=abc123xyz
```

Google checks this value to confirm you own the domain.

---

# 6. NS Record (Name Server)

NS records answer:

> Which authoritative DNS server manages this domain?

## Example

```text
example.com
      ↓
ns1.cloudflare.com
ns2.cloudflare.com
```

These are the servers that hold the official DNS records.

When you buy a domain and use Cloudflare, AWS Route 53, or another DNS provider, changing the NS records effectively tells the internet:

> "These servers are now the source of truth for my domain."

---

# 7. PTR Record (Pointer Record)

PTR records perform the opposite of an **A record**.

Instead of:

```text
Domain
   ↓
IP
```

They do:

```text
IP
 ↓
Domain
```

This is called a **reverse DNS lookup**.

## Example

```text
192.0.2.15
     ↓
mail.example.com
```

Mail servers often check PTR records to help verify the identity of sending servers.

---

# 8. SOA Record (Start of Authority)

This is one of the most important records, but beginners rarely need to edit it.

It contains administrative information about the DNS zone, such as:

- Primary authoritative server
- Responsible contact
- Serial number
- Refresh interval
- Retry interval
- Expire time
- Default TTL

DNS servers use this information to coordinate and keep zone data up to date.

---

# Summary Table

| Record | Purpose | Example |
|---------|---------|---------|
| A | Domain → IPv4 | `example.com → 192.0.2.10` |
| AAAA | Domain → IPv6 | `example.com → 2001:db8::10` |
| CNAME | Domain → Another domain | `www.example.com → example.com` |
| MX | Mail server | `mail.example.com` |
| TXT | Verification / email policies | `google-site-verification=...` |
| NS | Authoritative name servers | `ns1.cloudflare.com` |
| PTR | IP → Domain | `192.0.2.10 → mail.example.com` |
| SOA | Zone administration | `Serial number, TTL, etc.` |

---

# Production Scenario

Imagine you own:

```text
mycompany.com
```

Your DNS records might look like this:

### A Record

```text
mycompany.com
      ↓
54.210.100.20
```

---

### CNAME Record

```text
www.mycompany.com
        ↓
mycompany.com
```

---

### MX Record

```text
mycompany.com
      ↓
mail.mycompany.com
```

---

### TXT Record

```text
mycompany.com
      ↓
google-site-verification=abc123
```

---

### NS Record

```text
mycompany.com
      ↓
ns1.cloudflare.com
ns2.cloudflare.com
```

Every record serves a different purpose, but together they define how the domain behaves on the internet.

> **Note:** IPv4 and IPv6 are covered in detail in a later module. For now, just remember that **A records store IPv4 addresses**, while **AAAA records store IPv6 addresses**.