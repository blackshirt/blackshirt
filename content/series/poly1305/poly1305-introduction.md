---
title: "Poly1305 - Introduction"
date: 2022-03-21T09:04:54+07:00
series: ["poly1305"]
categories: ["mac", "poly1305"]
comment: true
---

**Table of Contents**

- [Part 1 - Introduction](#)
- [Part 2 - Benchmarking](/series/poly1305/poly1305-benchmarking)
- [Part 3 - Optimizing and speed up](/series/poly1305/poly1305-speedup)

# About Poly1305
Poly1305 is a cryptographic message authentication code (MAC) created by Daniel J. Bernstein. It can be used to verify the data integrity and the authenticity of a message. A variant of Bernstein's Poly1305 that does not require AES has been standardized by the Internet Engineering Task Force in [RFC 8439](https://datatracker.ietf.org/doc/html/rfc8439).