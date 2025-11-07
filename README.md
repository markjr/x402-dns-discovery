# Discovering x402 Resources via DNS TXT Records

**IETF Internet-Draft:** [draft-jeftovic-x402-dns-discovery](https://datatracker.ietf.org/doc/draft-jeftovic-x402-dns-discovery/)  
**Status:** Experimental (Independent Submission)  
**Author:** Mark E. Jeftovic — [easyDNS Technologies Inc.](https://easydns.com)

---

## Overview

This repository tracks the work on **x402 DNS Discovery**, an experimental proposal to enable lightweight, DNS-based discovery of [x402](https://x402.org) payment resources.

The mechanism defines the use of `_x402` DNS TXT records that advertise HTTPS URLs where clients or agents can retrieve x402 manifests — enabling autonomous agents and applications to discover payment endpoints without prior configuration.

It complements the main [x402 protocol](https://x402.org) by operating one layer earlier in the discovery stack.

---

## Specification

📄 **Current draft:**  
[`draft-jeftovic-x402-dns-discovery`](https://datatracker.ietf.org/doc/draft-jeftovic-x402-dns-discovery/)

- **Intended status:** Experimental  
- **Category:** Independent Submission  
- **Latest version:** `-00`  
- **Expires:** May 2026  

Rendered versions:
- [HTML view](https://datatracker.ietf.org/doc/html/draft-jeftovic-x402-dns-discovery)
- [Text file](https://www.ietf.org/archive/id/draft-jeftovic-x402-dns-discovery-00.txt)
- [PDF view](https://datatracker.ietf.org/doc/pdf/draft-jeftovic-x402-dns-discovery)

---

## Example

```dns
_x402.example.com. 300 IN TXT "v=x4021;url=https://example.com/.well-known/x402"
_x402.api.example.com. 300 IN TXT "v=x4021;descriptor=api;url=https://api.example.com/x402"
