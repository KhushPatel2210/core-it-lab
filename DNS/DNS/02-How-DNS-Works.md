# 2. Step-by-step DNS Resolution

Let's go through the complete process.

### Step 1 — User enters a domain
Example: `google.com`

The browser does not know Google's IP address. So it asks:
> *"Where is google.com?"*

### Step 2 — Browser Cache
The browser first checks: **Browser DNS Cache**

Imagine you visited Google 10 minutes ago. The browser may already remember the IP.

Example:  
`google.com` → `142.250.193.78`

* **If found:** ✅ No internet lookup.
* **If not:** Move to the operating system.

### Step 3 — Operating System Cache
The operating system also stores recently resolved domains. Windows, Linux, and macOS all have DNS caches.

If the OS already knows:  
`google.com` → `142.250.193.78`

It immediately returns the answer. Again: **No internet query.**

### Step 4 — Hosts File
Before asking DNS servers, the operating system checks the hosts file.

* **Linux:** `/etc/hosts`
* **Windows:** `C:\Windows\System32\drivers\etc\hosts`

Example entry:
```text
127.0.0.1 localhost
192.168.1.50 company.local
```

If `company.local` exists here, the system does not ask DNS at all. It simply uses: `192.168.1.50`

#### Why is the hosts file useful?
Developers often map domains to local machines.  
Example: `127.0.0.1 myapp.local`

Now `http://myapp.local` opens your local application.

### Step 5 — DNS Resolver
If nothing is found, the OS sends the request to a **DNS Resolver**. Usually this is provided by:
* Your ISP
* Google DNS (`8.8.8.8`)
* Cloudflare DNS (`1.1.1.1`)
* Quad9 (`9.9.9.9`)

Example:  
`Laptop` → `8.8.8.8`

The resolver's job is to find the answer for you.

### Step 6 — Resolver Cache
Resolvers also cache answers.

Imagine one million people ask for `google.com` every hour. Instead of asking Google's authoritative servers one million times, the resolver remembers the answer for a while (based on the DNS record's TTL, which we'll study later).

If cached:  
`Resolver` → Returns IP immediately *(Very fast)*.

### Step 7 — Recursive Lookup
If the resolver doesn't know, it starts asking other DNS servers. It doesn't magically know Google's IP; it has to discover it.

This process is called **DNS Resolution** (or recursive resolution). We'll study the full hierarchy in the next module.

### Step 8 — IP Address Returned
Eventually the resolver receives:  
`google.com` → `142.250.193.78`

It sends this back to your computer.

### Step 9 — Browser Connects
Now DNS is finished. The browser opens a TCP connection to the IP address. If you're using HTTPS, it performs the TLS handshake. 

Only after that does the browser send:
```http
GET /
Host: google.com
```

> **Notice something important:** The browser still includes `Host: google.com` even though it's connecting to an IP address. This allows one server to host many websites (virtual hosting).

---

## Summary Flow Chart

* User types `google.com`
* Browser Cache
* OS Cache
* Hosts File
* DNS Resolver
* Resolver Cache
* DNS Hierarchy
* Gets IP Address
* Browser connects to server
* Website loads

---

# Real Production Example

Imagine your company has: `api.company.com`

* **Yesterday** it pointed to: `10.10.10.15`
* **Today** DevOps deployed a new API server: `10.10.10.30`

They update the DNS record. Users don't need to change bookmarks or application settings—they still use: `https://api.company.com`

The DNS resolver simply returns the new IP after the old record expires.


# 📁 What is the Hosts File?

The **hosts file** is a local text file on your computer that **manually maps domain names to IP addresses**.

Think of it as your personal **mini DNS server**. Your computer always checks this file **before** querying any public DNS.

> 💡 **Sticky Note Analogy:**  
> Instead of asking the phone company for a contact number, you check a note on your desk:  
> `Mom` $\rightarrow$ `+91 9876543210`. The hosts file works the exact same way.

---

### 📍 Location & Format

* **Linux / macOS:** `/etc/hosts`
* **Windows:** `C:\Windows\System32\drivers\etc\hosts`

```text
# Example /etc/hosts file entries
127.0.0.1       localhost
192.168.1.50    dev.company.local
34.100.200.20   company.com
```

---

### ⚡ Request Flow

```
[ Web Browser ]
      │
      ▼
  [ OS Cache ]
      │
      ▼
 [ Hosts File ]  ── (Match Found?) ──► ✅ [ Directly Uses IP ]
      │
 (Not Found)
      ▼
 [ DNS Lookup ]
```

---

### 🎯 Common Use Cases

* **Local Development:** Map local IPs to clean domain names (e.g., `127.0.0.1 myapp.local`).
* **Pre-deployment Testing:** Point a live domain to a staging IP on your machine only.
* **Overriding DNS & Blocking:** Redirect unwanted domains to `127.0.0.1` to block them locally.