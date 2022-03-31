---
title: "Poly1305 - Introduction"
date: 2022-03-21T09:04:54+07:00
series: ["poly1305"]
categories: ["poly1305"]
comment: true
math: true
---
This post is first series i'm trying to write about of my intense works in writing (actually porting) `poly1305` module for [V language](https://github.com/vlang/v). I hope i can make series of writing in this topics. 
If you look this part is not worth to read, you can skip this part and goes to others part in contents below. Currently, its hasn't been published, i would publish it when its ready.

**Table of Contents**
- [Part 1 - Introduction -**You're here**](#)
- [Part 2 - Benchmarking](/series/poly1305-benchmarking)
- [Part 3 - Optimizing and speed up](/series/poly1305-speedup)

Let's begin by background about what is `Poly1305` and related concepts.

### What is message authentication code (MAC)
In cryptography world, a message authentication code (MAC), sometimes known as a tag, is a short piece of information used for authenticating a message. In other words, to confirm that the message came from the stated sender (its authenticity) and has not been changed. The MAC value protects a message's data integrity, as well as its authenticity, by allowing verifiers (who also possess the secret key) to detect any changes to the message content.

Generally speaking, message authentication works with three mechanism.
- First, there is a key generation mechanism that selects a key from the keyspace uniformly at random. 
- Second, there is also a signing mechanism that generates a tag given the key and message. 
- Finally, there is a verifying mechanism that helps to authenticate a message that is given the key and the tag. 

Generally, MAC functions are similar to cryptographic hash functions that provides sufficient integrity, but its way more by providing authenticity. 

### What's Poly1305
Fundamentally, Poly1305 is a different class of MAC that's originally designed and created by Daniel J. Bernstein. It can be used to verify the data integrity and the authenticity of a message. Poly1305 has been standardized by the Internet Engineering Task Force in [RFC 8439](https://datatracker.ietf.org/doc/html/rfc8439).

### How poly1305 algorithm works

Poly1305 is a relatively simple MAC algorithm. Poly1305 takes a 32-byte one-time key (**see warning below** why its called one-time key) and an arbitrary length of message and produces a 16-byte tag.  As we can look at [RFC 8439](https://datatracker.ietf.org/doc/html/rfc8439), in pseudocode (in modified, simplified and commented version):
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

> **⚠ WARNING:**
>Using the same key to generate tags for multiple messages allows an attacker to forge tags. Always generate a new key per message you want to authenticate. If you are using this as a MAC for symmetric encryption please use `chacha20poly1305` instead.

### State of poly1305 module for V language
Luckily, we already have working `poly1305` module in pure V recently. If you wanna to try, you can look and read it on github repos at [poly1305](https://github.com/blackshirt/poly1305.git). As a notice, even its pass the tests from RFC test vectors, its not proven in productions and maybe still contains some buggy things.

> **⛔ DANGER**: 
>Security and cryptography was considerabled as “Hazardous Materials” module. You should **ONLY** use it if you’re 100% absolutely sure that you know what you’re doing because this module was written and provided as is without any guarantees and risks in your own. Just looks at the code and see what happens in it. 

Let's take break a down the pseudocode into real code. I'm not going to the technical detail about internal machinery of poly1305, if you wanna to know in depth and detail, you should read original papers from DJ Bernstein [poly1305.pdf](https://cr.yp.to/mac/poly1305-20050329.pdf), and read for this great writing from Loup Vaillant blog, [poly1305-design](https://loup-vaillant.fr/tutorials/poly1305-design).

#### Take look in a depth
The V's poly1305 module fundamentally contains two basic functions:
- `create_mac(msg, key)`  
- `verify_mac(mac, msg, key)`

`create_mac` function intended to create 16 byte length of tag/mac, where msg was messages you want to auth and key was **one-time** key you should provide, where `verify_mac` was for verifying mac provided with msg and key supplied, its return true when valid and false otherwise.

> **Note**: the function provided in the poly1305 module maybe changed in the future.

Under the hood, for creating tag/mac with `create_mac` its consists of step :
- initialize poly1305 state by calling `new_with_key(key)` function.
- feed poly1305 state by supplying its with messages want to auth, by calling `write(msg)` method.
- and finally, calling `finalize()` on the poly1305 state to return tag as a result.
