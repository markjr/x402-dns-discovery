%%%
title = "Discovering x402 Resources via DNS TXT Records"
abbrev = "x402 DNS Discovery"
category = "exp"
docName = "draft-jeftovic-x402-dns-discovery-00"
ipr = "trust200902"
area = "Applications and Real-Time"
workgroup = "Independent Submission"
date = 2025-11-07
keyword = ["x402", "DNS", "TXT", "payment", "discovery"]
[author]
initials = "M.E."
surname = "Jeftovic"
fullname = "Mark E. Jeftovic"
organization = "easyDNS Technologies Inc."
email = "markjr@easydns.com"
uri = "https://easydns.com"
%%%

# Abstract

This document defines a DNS-based discovery mechanism for locating **x402 payment resources** associated with a domain.  
Domains publish one or more `_x402` TXT records containing URLs where x402-compatible clients can obtain resource manifests and metadata over HTTPS.  
The goal is to provide a lightweight, cache-friendly discovery vector that enables automated payment negotiation using the x402 protocol while keeping DNS records static and non-sensitive.

---

# 1. Introduction

The **x402 protocol** reactivates HTTP status code 402 ("Payment Required") as a mechanism for metered and token-based access control.  
In this model, clients interact directly with an origin server or a designated facilitator to complete small, machine-to-machine payments before accessing a protected resource.

This draft defines a complementary DNS discovery mechanism for x402.  
By publishing an `_x402` TXT record as defined in the Domain Name System ([RFC1034](https://www.rfc-editor.org/rfc/rfc1034), [RFC1035](https://www.rfc-editor.org/rfc/rfc1035)), a domain can advertise one or more URLs where clients may retrieve x402 resource manifests and related metadata.  
This allows agents and applications to locate payment gateways before making application requests, reducing latency and enabling autonomous client behavior.

The TXT layer is a **pointer-only** facility; all dynamic data (prices, nonces, credentials) are obtained via HTTPS from the discovered URL(s).

---

# 2. Conventions and Terminology

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC2119](https://www.rfc-editor.org/rfc/rfc2119) and [RFC8174](https://www.rfc-editor.org/rfc/rfc8174).

* **Origin** — the HTTP host (and optional port) that serves content or APIs protected by x402.  
* **x402 Resource** — any endpoint implementing the x402 payment protocol.  
* **Manifest URL** — an HTTPS URL published via `_x402` TXT that, when fetched, returns a structured manifest describing available x402 resources.  
* **Descriptor** — a short, optional free-text token that semantically identifies the service (e.g., "api", "shop", "tshirt").

---

# 3. Overview

To enable discovery, an origin publishes one or more TXT records under the underscored node name `_x402` at the specific hostname where x402 resources are served.

```dns
_x402.example.com.  300  IN  TXT  "v=x4021;url=https://example.com/.well-known/x402"
_x402.shop.example.com.  300  IN  TXT  "v=x4021;descriptor=tshirt;url=https://shop.example.com/x402-discovery"