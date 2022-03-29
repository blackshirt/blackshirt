---
title: "Poly1305 - Introduction"
date: 2022-03-21T09:04:54+07:00
series: ["poly1305"]
categories: ["mac", "poly1305"]
comment: true
math: true
---

**Table of Contents**

- [Part 1 - Introduction](#)
- [Part 2 - Benchmarking](/series/poly1305/poly1305-benchmarking)
- [Part 3 - Optimizing and speed up](/series/poly1305/poly1305-speedup)

# About Poly1305
Poly1305 is a cryptographic message authentication code (MAC) originally created by Daniel J. Bernstein. It can be used to verify the data integrity and the authenticity of a message. A variant of Bernstein's Poly1305 that does not require AES has been standardized by the Internet Engineering Task Force in [RFC 8439](https://datatracker.ietf.org/doc/html/rfc8439).

# Algorithm
Poly1305 takes a 32-byte one-time key and an arbitrary length of message and produces a 16-byte tag. Poly1305 is a relatively simple MAC algorithm as we can look at [RFC 8439](https://datatracker.ietf.org/doc/html/rfc8439), in pseudocode (in modified and commented version):
```
fn poly1305_auth(key, msg) -> [16]byte {
    # the key partitioned into two parts, called "r" and "s".
    r, s := partitioned(key)
    
    # and then clamping the r part
    r = clamping(r)

    # set initial accumulator
    h := 0

    # next, divide the msg to 16 bytes block, and doing pad if necessary
    for len(msg) > 0 {
        # read the block as little endian number
        m := read(msg, 16)

        # Add this number to the accumulator
        h += m

        # Multiply accumulator by "r".
        h *= r

        # Set the accumulator to the result modulo p
        h %= 2¹³⁰ - 5
    }
    # Finally, the value of the secret key "s" is added to the accumulator,
    # and the 128 least significant bits are serialized in little-endian
    # order to form the tag
    h += s
    tag := serialize_to_16_le_bytes(h)

    return tag
}
```

If we look at above pseudocode, its looks nasty and simple. All the complexity is about doing performant constant-time math on numbers larger than any available numeric type.