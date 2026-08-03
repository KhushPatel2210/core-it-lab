Module 15 — DNS Security
DNS/
├── 15-DNS-Security.md
Learning Objectives

By the end of this chapter, you'll understand:

Why DNS is a security target
Common DNS attacks
DNS Spoofing
DNS Cache Poisoning
DNS Hijacking
DNS Amplification Attack
DNS Tunneling
DNSSEC
Secure DNS (DoH & DoT)
Production best practices
Why is DNS a Security Risk?

Every website starts with DNS.

Before your browser connects to:

google.com

it must first ask:

"What's the IP address?"

If an attacker can change the answer,

they can redirect users to a fake server.

DNS becomes a very attractive target.

DNS Attack Surface
                User
                  │
                  ▼
           DNS Resolver
                  │
                  ▼
        Authoritative Server
                  │
                  ▼
             Website

An attacker may target:

The user's computer
The local DNS cache
The recursive resolver
The authoritative server
The network between them
1. DNS Spoofing
What is DNS Spoofing?

DNS spoofing means returning a fake DNS response.

Example:

User requests:

bank.com

Expected:

104.18.25.10

Attacker replies:

203.0.113.99

The browser connects to the attacker's server.

Visualization
User

↓

bank.com

↓

Attacker sends fake reply

↓

Fake IP

↓

Fake Website

The user may never notice if the fake site looks convincing.

2. DNS Cache Poisoning

A recursive DNS server caches answers.

Imagine the cache contains:

bank.com

↓

203.0.113.99

instead of the correct IP.

Now every user who asks that resolver gets the fake answer until the bad entry expires or is removed.

Flow
Attacker

↓

Recursive Resolver

↓

Cache Updated

↓

All Users Receive Fake IP

This attack is called cache poisoning because the resolver's cache has been "poisoned" with incorrect data.

3. DNS Hijacking

DNS hijacking happens when someone changes which DNS server a device uses or alters DNS settings.

Examples:

Malware changes your DNS server.
A compromised home router changes DNS settings.
An attacker modifies DHCP options.
A cloud VM is configured with the wrong DNS server.

Instead of:

8.8.8.8

your computer may silently start using:

203.0.113.25

controlled by an attacker.

4. DNS Amplification Attack

DNS normally uses UDP, which is connectionless.

Attackers exploit this by sending small queries with a spoofed source IP (the victim's IP).

The DNS server sends a much larger response to the victim.

Example

Attacker sends:

60 Bytes

DNS server replies:

4000 Bytes

If this happens thousands or millions of times,

the victim becomes overwhelmed.

This is called a DNS Amplification DDoS Attack.

Visualization
Attacker
     │
     │ (Spoofed Source IP = Victim)
     ▼
DNS Server
     │
     ▼
Huge Response
     │
     ▼
Victim

The DNS server is being abused as an amplifier.

5. DNS Tunneling

DNS is usually allowed through firewalls.

Attackers can misuse DNS queries to secretly move data.

Example:

Instead of asking:

google.com

they send something like:

c2VjcmV0LWRhdGE.example.com

The encoded label may actually contain hidden data.

A controlled DNS server receives those queries and reconstructs the information.

Why It's Dangerous

Attackers can:

Exfiltrate sensitive files
Bypass firewall restrictions
Establish covert command-and-control channels

DNS tunneling is uncommon in normal environments but important for security teams to understand.

6. DNSSEC (DNS Security Extensions)

Traditional DNS provides no way to verify that a DNS response is authentic.

DNSSEC adds digital signatures to DNS records.

Resolvers that validate DNSSEC can detect if a response has been tampered with.

Simplified Flow
Domain Owner
      │
Signs DNS Records
      │
      ▼
Authoritative DNS Server
      │
      ▼
Resolver Validates Signature
      │
      ▼
Trusted Response

DNSSEC provides authentication and integrity of DNS data.

It does not encrypt DNS traffic.

7. DNS over HTTPS (DoH)

Traditional DNS queries are often sent in plain text.

DNS over HTTPS (DoH):

Encrypts DNS queries
Sends them over HTTPS (port 443)
Helps protect against eavesdropping on untrusted networks

Example:

Laptop
   │
HTTPS
   ▼
DNS Resolver

Someone monitoring the network cannot easily read the DNS queries.

8. DNS over TLS (DoT)

Similar goal.

Instead of HTTPS,

DNS uses TLS directly (commonly port 853).

Client

↓

Encrypted TLS

↓

Resolver
DoH vs DoT
DNS over HTTPS (DoH)	DNS over TLS (DoT)
Uses HTTPS (443)	Uses TLS (853)
Blends with normal web traffic	Dedicated DNS encryption
Harder to distinguish from web traffic	Easier to identify and manage
Encrypts DNS	Encrypts DNS

Both protect DNS queries in transit.

Security Best Practices

A production environment should:

Enable DNSSEC where supported
Keep recursive resolvers updated
Restrict zone transfers
Monitor unusual DNS traffic
Detect DNS tunneling
Use trusted recursive resolvers
Avoid exposing unnecessary DNS servers
Use encrypted DNS (DoH or DoT) where appropriate
Log DNS activity for investigations
AWS DNS Security

In AWS, common practices include:

Restrict Route 53 modifications using IAM.
Use Security Groups and NACLs to limit unnecessary DNS-related access.
Protect public applications with services like Shield and WAF (where applicable).
Monitor Route 53 changes with CloudTrail.
Use Route 53 Resolver DNS Firewall (if available in your architecture) to block malicious domains.
Kubernetes DNS Security

Inside Kubernetes:

Limit Pod permissions.
Restrict egress with Network Policies where appropriate.
Protect CoreDNS from unnecessary exposure.
Monitor DNS traffic for anomalies.
Avoid granting unrestricted Internet access to every workload.
Real Production Scenario

A company notices:

bank.company.com

suddenly resolves to an unexpected IP.

Investigation reveals:

A recursive resolver was compromised.
Its cache was poisoned.
Users were redirected to a phishing site.

The response:

Clear the poisoned cache.
Restore trusted DNS data.
Validate DNSSEC where supported.
Investigate the compromise.
Rotate affected credentials if users may have been exposed.
Common Misconceptions
❌ DNSSEC encrypts DNS traffic.

No.

DNSSEC verifies authenticity and integrity.

It does not provide encryption.

❌ DNS over HTTPS prevents phishing.

No.

It encrypts DNS queries in transit but does not guarantee that the destination website is legitimate.

❌ DNS attacks only affect websites.

No.

They can impact:

APIs
Email systems
Kubernetes clusters
Cloud infrastructure
Internal enterprise services
❌ Internal DNS doesn't need protection.

Wrong.

Private DNS often exposes critical internal systems such as databases, CI/CD servers, and monitoring platforms.

Interview Questions
Q1. What is DNS Spoofing?

Providing a forged DNS response so a client is redirected to an incorrect destination.

Q2. What is DNS Cache Poisoning?

Injecting false DNS data into a recursive resolver's cache so future queries receive incorrect answers.

Q3. What problem does DNSSEC solve?

It helps verify that DNS responses are authentic and have not been modified.

Q4. Does DNSSEC encrypt DNS traffic?

No.

It provides authentication and integrity, not confidentiality.

Q5. What is DNS Amplification?

A DDoS technique where attackers send small spoofed DNS requests that trigger much larger responses to a victim.

Q6. What is DNS Tunneling?

Using DNS queries and responses to covertly transfer data or communicate with malicious infrastructure.

Q7. Difference between DoH and DoT?
DoH encrypts DNS over HTTPS (port 443).
DoT encrypts DNS using TLS directly (commonly port 853).
Summary
                    DNS Security

             ┌────────────────────┐
             │                    │
             ▼                    ▼

        DNS Attacks          DNS Protection

        Spoofing             DNSSEC
        Cache Poisoning      DoH
        Hijacking            DoT
        Amplification        Monitoring
        Tunneling            Logging
Key Takeaways
DNS is a foundational service and a common attack target.
Common attacks include spoofing, cache poisoning, hijacking, amplification, and tunneling.
DNSSEC verifies the authenticity of DNS responses but does not encrypt them.
DoH and DoT encrypt DNS traffic between clients and resolvers.
Monitoring and securing DNS is a critical responsibility in cloud, DevOps, and enterprise environments.