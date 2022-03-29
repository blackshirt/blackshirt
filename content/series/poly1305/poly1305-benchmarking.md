---
title: "Poly1305 - Benchmarking"
date: 2022-03-23T09:05:07+07:00
series: ["poly1305"]
categories: ["mac", "poly1305"]
draft: true
---

```
Stats of benchmarking of poly32_create_mac_1k_msg_bench function
Number of iterations: 100000 x
Average:        206 μs
Minimal:        0 μs
Maximal:        304000 μs
```

```
Stats of benchmarking of poly32_compute_mac_1k_msg_bench function
Number of iterations: 100000 x
Average:        70 μs
Minimal:        0 μs
Maximal:        555000 μs
```

After modified and refactored to use `State` 
```
Stats of benchmarking of poly32_create_mac_1k_msg_bench function
Number of iterations: 100000 x
Average:        127 μs
Minimal:        0 μs
Maximal:        63000 μs
Stats of benchmarking of poly32_compute_mac_1k_msg_bench function
Number of iterations: 100000 x
Average:        63 μs
Minimal:        0 μs
Maximal:        22000 μs
```

The Go version benchmark (only for 1024K message).
I'm on windows, commonly its differs when its run on other os or platform.
Just makes sure, run benchmark yourself.

```
$ go test -bench=Benchmark1K
goos: windows
goarch: amd64
pkg: golang.org/x/crypto/internal/poly1305
cpu: AMD A4-9125 RADEON R3, 4 COMPUTE CORES 2C+2G
Benchmark1K-2                     765384              1332 ns/op         768.51 MB/s
Benchmark1KUnaligned-2            891763              1349 ns/op         759.36 MB/s
PASS
ok      golang.org/x/crypto/internal/poly1305   3.826s
```


```
$ v -cc gcc -gc boehm run bench.v
Stats of benchmarking of poly32_create_mac_1k_msg_bench function
Number of iterations: 100000 x
Average:        80 μs
Minimal:        0 μs
Maximal:        38000 μs
Stats of benchmarking of poly32_compute_mac_1k_msg_bench function
Number of iterations: 100000 x
Average:        42 μs
Minimal:        0 μs
Maximal:        10000 μs
Stats of benchmarking of poly32_create_mac_2m_msg_bench function
Number of iterations: 1000 x
Average:        184836 μs
Minimal:        143000 μs
Maximal:        1299000 μs
Stats of benchmarking of poly32_compute_mac_2m_msg_bench function
Number of iterations: 1000 x
Average:        96972 μs
Minimal:        77000 μs
Maximal:        504000 μs
```