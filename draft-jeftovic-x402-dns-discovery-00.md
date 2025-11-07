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
```

Clients perform a DNS TXT lookup on `_x402.<host>` using the exact hostname from their intended HTTP request and obtain one or more URLs.  
Each URL represents an HTTPS endpoint where further x402 metadata can be retrieved using a simple GET request.

The DNS record itself is **static** and only intended to point to discovery locations.  
All dynamic data — such as pricing, currency, or facilitator credentials — are obtained over HTTPS from the URL(s) indicated.

**Discovery Scope:**  
Each hostname requiring x402 discovery MUST publish its own `_x402` TXT record. There is no inheritance from parent domains. Clients MUST query `_x402.<exact-request-host>` to discover payment resources for that specific hostname.

---

# 4. Record Format

Each `_x402` TXT record MUST contain a single string formatted as semicolon-separated key/value pairs:

```abnf
record     = "v=" version ";" [ descriptor ] "url=" https_url
version    = "x4021"
descriptor = "descriptor=" desc_value ";"
desc_value = 1*(%x20-21 / %x23-3A / %x3C-7E)
             ; printable ASCII excluding quote (0x22) and semicolon (0x3B)
https_url  = "https://" 1*(VCHAR)
```

* The optional `descriptor` provides a short, human-readable identifier for the service (e.g., "api", "shop", "tshirt").  
* Values **SHOULD NOT** contain semicolons or double quotes; if needed, publishers **MAY** percent-encode reserved characters per [RFC3986](https://www.rfc-editor.org/rfc/rfc3986).  
* Implementations **MUST** ignore any keys they do not understand.  
* Multiple TXT records at the same node are permitted and indicate multiple discovery URLs.

---

# 5. Client Processing

1. **Lookup** — perform a DNS TXT query for `_x402.<host>` using the exact hostname as the intended HTTP origin.  
2. **Parse** — for each TXT record beginning with `v=x4021;`, extract the `url` and, if present, the `descriptor`.  
3. **Fetch** — issue an HTTPS GET to each discovered URL.  
4. **Interpret** — parse the returned manifest.  
5. **Negotiate** — proceed with x402 payment handshake per core x402 protocol.

If multiple records exist, clients **SHOULD** fetch all discovered URLs unless local policy dictates otherwise.

---

# 6. Operational Considerations

* `_x402` TXT records are expected to change infrequently and MAY have long TTLs.  
* Records SHOULD be published under each subdomain where x402 resources are hosted.  
* Operators SHOULD sign DNS zones with DNSSEC.  
* Clients SHOULD prefer DNSSEC-validated responses.  
* Only HTTPS URLs are permitted; HTTP is not allowed.

---

# 7. Security Considerations

The TXT discovery mechanism does not convey sensitive or dynamic payment data.  
Compromised DNS could redirect clients to malicious endpoints. Clients SHOULD validate DNSSEC and HTTPS certificates.  
`descriptor` is informational only and MUST NOT be used for authorization.

---

# 8. IANA Considerations

IANA is requested to add this entry to the *Underscored and Globally Scoped DNS Node Names* registry ([RFC8552](https://www.rfc-editor.org/rfc/rfc8552)):

| Node Name | RR Type | Reference |
|------------|----------|------------|
| `_x402` | TXT | This document |

---

# 9. References

## 9.1 Normative

* [RFC1034] P. Mockapetris, *Domain Names – Concepts and Facilities*, 1987.  
* [RFC1035] P. Mockapetris, *Domain Names – Implementation and Specification*, 1987.  
* [RFC2119] S. Bradner, *Key Words for Use in RFCs to Indicate Requirement Levels*, 1997.  
* [RFC3986] T. Berners-Lee et al., *URI: Generic Syntax*, 2005.  
* [RFC8174] B. Leiba, *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*, 2017.  
* [RFC8552] D. Kumari, S. Rose, *Scoped Extension Mechanism for DNS (Underscored Node Names)*, 2019.

---

# 10. Acknowledgments

The author gratefully acknowledges **Tim Berners-Lee**, **Roy Fielding**, and contributors to **HTTP/1.0** and **HTTP/1.1** whose work established the foundation for HTTP 402 (“Payment Required”).  
Additional thanks to **Erik Reppel**, **Coinbase**, and the **x402.org** team for implementing and open-sourcing the modern x402 protocol, and for advancing the idea of **agentic micropayments on the web**.

---

# Appendix A. Example Flow

```text
1. A client wishes to access https://api.example.com/data
2. It queries DNS for _x402.api.example.com TXT
3. DNS responds with:
     "v=x4021;descriptor=api;url=https://api.example.com/.well-known/x402"
4. Client performs GET https://api.example.com/.well-known/x402 and receives a JSON manifest enumerating payable API endpoints.
5. Client follows x402 negotiation per core spec, receives 402 Payment Required, completes settlement, and retries successfully.
```

---

# Author’s Address

```
Mark E. Jeftovic
easyDNS Technologies Inc.
Toronto, ON, Canada
Email: markjr@easydns.com
URI: https://easydns.com
```
