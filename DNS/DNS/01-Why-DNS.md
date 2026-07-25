# 1. What is DNS?

**DNS** stands for **Domain Name System**.

---

### Its job is simple:

> Convert a human-readable domain name into an IP address.

---

### Example:

`google.com`  
⬇️  
`142.250.193.78`  
*(One of many possible Google IPs.Google owns many IP addresses worldwide, so google.com can resolve to different valid IP addresses depending on your location, network, and Google's traffic management.)*

---

* **Computers** communicate using IP addresses, not names.
* **Humans** remember names much more easily than numbers.

> **DNS acts like the Internet's phonebook.**

# 2. Why do we even need DNS?

Imagine there were no **DNS**.

To open Google, you'd have to type:
`142.250.193.78`

To open YouTube:
`142.251.xxx.xxx`

To open GitHub:
`140.82.xxx.xxx`

---

Thousands of websites would mean thousands of IP addresses to memorise.

**That's unrealistic.**

---

### DNS solves this problem by mapping:

`google.com`  
↓  
`142.250.193.78`



# 3. Real-world analogy

Think of your mobile phone.

### Your contact list contains:
* **Mom**
* **Dad**
* **Friend**
* **Boss**

But the mobile network doesn't actually use those names.  
Internally it uses **phone numbers**.

`Mom`  
↓  
`+91 98xxxxxxxx`

---

### DNS works exactly the same way.

* Instead of: **Mom**
  * it stores **`google.com`**

* Instead of: **Phone Number**
  * it returns **`IP Address`**

---

### So:

`Contact Name`  
↓  
`Phone Number`

**becomes**

`Domain Name`  
↓  
`IP Address`


# 4. Why don't websites use just one IP forever?

A common misconception is:

> "One website has one IP."

In reality, large services often have many **IP addresses**.

For example:
`google.com`

may resolve to different IPs depending on:

* **your country**
* **the nearest Google data centre**
* **current server load**
* **whether a server is under maintenance**

DNS helps direct users to the most appropriate destination.

# 5. Why DNS is critical

Without **DNS**:

* **Browsers** couldn't locate websites by name.
* **APIs** would be difficult to use.
* **Email delivery** wouldn't work properly.
* **Cloud services** would be much harder to manage.
* **Kubernetes service discovery** would be impractical.
* **Load balancing and failover** would be far more complex.

> **Almost every internet service depends on DNS.**

---

# 6. Real production example

Suppose a company moves its web application from one server to another.

* **Old server:** `203.0.113.10`
* **New server:** `198.51.100.25`

**Users still visit:**  
`https://company.com`

---

### How DNS Architecture Handles This Shift:

```
                  [ User Browser ]
                          │
             Type: [https://company.com](https://company.com)
                          │
                          ▼
                 [ DNS Nameserver ]
                          │
          Points to New IP: 198.51.100.25
                          │
                          ▼
            [ New Server: 198.51.100.25 ]
```

---

The company only needs to **update the DNS record**.

Users continue using the same domain name, while DNS points them to the new server.

**This flexibility is one of DNS's biggest strengths.**

---

### ❓ Does DNS store websites?

**No.** DNS stores **mappings** and other **resource records**. The actual website is hosted on web servers.

---

# 📝 Summary

* **DNS** = Domain Name System.
* It **translates domain names into IP addresses**.
* **Humans** use names; **computers** use IP addresses.
* **DNS** acts as the **Internet's phonebook**.
* **Modern applications, cloud platforms, and distributed systems** rely heavily on DNS.



# 1. What happens when you type google.com in your browser?

Let's say you open Chrome and type:

`https://google.com`

You press Enter. Within milliseconds, many things happen before the webpage appears.

Most people think:

```text
Browser
   ↓
Google Server
```

But that's not what happens. The actual flow is more like this:

### Complete DNS Resolution Flow

![DNS Flow Diagram](http://googleusercontent.com/image_collection/image_retrieval/18245154923432327720_0)

**DNS** is the step that discovers where the browser should connect.

---

