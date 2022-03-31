---
title: "Poly1305 - Benchmarking"
date: 2022-03-23T09:05:07+07:00
series: ["poly1305"]
categories: ["mac", "poly1305"]
draft: true
---


Its on windows amd64 machine, from original poly1305 github repo.
```bash
$ v run bench.v
Stats of benchmarking of poly32_create_mac_1k_msg_bench function
Number of iterations: 100000 x
Average:        139 μs
Minimal:        0 μs
Maximal:        155000 μs
```
On poly1305-optimize branch (windows amd64 machine)
```bash
$ v run bench.v
Stats of benchmarking of poly32_create_mac_1k_msg_bench function
Number of iterations: 100000 x
Average:        106 μs
Minimal:        0 μs
Maximal:        180000 μs
```

```bash
Stats of benchmarking of poly32_create_mac_1k_msg_bench function
Number of iterations: 100000 x
Average:        206 μs
Minimal:        0 μs
Maximal:        304000 μs
```

```bash
Stats of benchmarking of poly32_compute_mac_1k_msg_bench function
Number of iterations: 100000 x
Average:        70 μs
Minimal:        0 μs
Maximal:        555000 μs
```

After modified and refactored to use `State` 
```bash
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

```bash
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


```bash
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

this is 10x iteration of sorted profiles `sorted.txt`
```
            11          0.001ms             90ns blackshirt__poly1305__Poly1305_finalize 
           715          0.087ms            122ns crypto__internal__subtle__constant_time_copy 
            11          0.002ms            185ns blackshirt__poly1305__new_with_key 
             1          0.000ms            216ns rand__seed__time_seed_64 
             1          0.000ms            261ns time__init_win_time_start 
             2          0.001ms            265ns rand__seed__time_seed_array 
             1          0.000ms            310ns add_vectored_exception_handler 
             1          0.000ms            317ns add_unhandled_exception_handler 
             1          0.000ms            411ns rand__new_default 
             1          0.000ms            431ns time__make_unix_time 
             2          0.001ms            474ns rand__read_64 
             2          0.001ms            482ns rand__read_internal 
             2          0.001ms            582ns rand__PRNG_bytes 
             2          0.001ms            587ns rand__bytes 
             1          0.001ms            607ns is_terminal 
             5          0.004ms            817ns StrIntpData_process_str_intp_data 
             5          0.005ms           1015ns str_intp 
             1          0.001ms           1482ns rand__init 
             1          0.005ms           4686ns builtin_init 
            11          0.150ms          13629ns blackshirt__poly1305__Poly1305_write 
            11          0.153ms          13915ns blackshirt__poly1305__create_mac 
             5          0.095ms          19059ns _write_buf_to_fd 
             5          0.096ms          19111ns _writeln_to_fd 
             5          0.096ms          19115ns println 
             1          0.031ms          30679ns time__local_as_unix_time 
             1          0.077ms          76978ns main__print_bench_results 
             1          0.256ms         255866ns main__poly32_create_mac_1k_msg_bench 
             1          0.256ms         256013ns main__main 
```

with output benchmarking
```bash
$ v run bench.v
Stats of benchmarking of poly32_create_mac_1k_msg_bench function
Number of iterations: 10 x
Average:        110672 ns
Minimal:        89300 ns
Maximal:        170500 ns
```

Its golang bench (default Sum) thats mimics v version
```bash
$ go run gobench.go 
Stats of benchmarking of benchmark_1kSum function
Number of iterations: 100000 x
Average:         1180.159 ns
Minimal:         0.000 ns
Maximal:         4029200.000 ns
```


This is modified go poly1305's Sum function, to get more appropriate apple to apple comparisons.
```bash
$ go run gobench.go 
Stats of benchmarking of benchmark_1kSum function
Number of iterations: 100000 x
Average:         1071.032 ns
Minimal:         0.000 ns
Maximal:         1023700.000 ns
Stats of benchmarking of benchmark_1k_SumGeneric function
Number of iterations: 100000 x
Average:         2039.255 ns
Minimal:         0.000 ns
Maximal:         26011500.000 ns
```