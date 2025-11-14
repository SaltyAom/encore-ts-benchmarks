# Encore Benchmark

Fork from https://github.com/encoredev/ts-benchmarks

![Benchmark Results](./results.png)

This benchmark purpose is to compare performance between EncoreTS and Elysia

- Benchmark Date: 14 Nov 2025
- Machine specification: Intel i7-13700K, DDR5 32GB 5600MHz
- OS: Debian 11 (bullseye) on WSL - 5.15.167.4-microsoft-standard-WSL2
- Encore: 1.5.17
- Rust: 1.91.1
- Elysia: 1.4.16
- Bun: 1.3.2

## Changes from upstream
1. Update Elysia version, add Elysia build script for production performance

2. Update Elysia bare request to use static resources

3. Due to machine specification, I update `oha` concurrency from `150` to `450` due to scale upper limit

Upstream:
```bash
PORT=3000 oha -c 150 -z 10s -m GET "http://127.0.0.1:$PORT/hello"
```

This variant:
```bash
PORT=3000 oha -c 450 -z 10s -m GET "http://127.0.0.1:$PORT/hello"
```

## Prerequisted
Here's the required tools for testing
- [oha](https://github.com/hatoo/oha)
- [Encore](https://encore.dev/docs/ts/quick-start)
- [Bun](https://bun.sh)

It's recommended to update all of the dependencies to the latest version

## Benchmarking Encore
Navigate to encore source code
```bash
cd requests/encore

# Install necessary dependencies
bun install
```

Upstream recommendation:

To start the server:
```bash
$ ENCORE_LOG=off ENCORE_NOTRACE=1 ENCORE_RUNTIME_LOG=debug encore run
[... more logs...]
DBG encore_runtime_core::api::manager > api server listening for incoming requests addr=127.0.0.1:52680
```

Default encore port goes to proxy, so we should use a random assigned port, in this case `52680`
```bash
PORT=52680 oha -c 450 -z 10s -m GET "http://127.0.0.1:$PORT/hello"
```

For schema validation:
```bash
PORT=52680 oha -c 450 -z 10s -m POST -H 'Content-Type: application/json' -H 'x-foo: test' "http://127.0.0.1:$PORT/schema?name=test&excitement=123" -d '{"someKey": "test", "someOtherKey": 123, "requiredKey": [123, 456, 789], "nullableKey": null, "multipleTypesKey": true, "multipleRestrictedTypesKey": "test", "enumKey": "John"}'
```

## Benchmarking Elysia
Navigate to Elysia source code
```bash
# For bare request
cd requests/elysia

# For schema validation
cd requests/elysia-schema
```

Run a build command to generate a server binary for production usage:
```bash
# Install necessary dependencies
bun install

bun run build
```

Running the benchmark:

For bare request
```bash
PORT=3000 oha -c 150 -z 10s -m GET "http://127.0.0.1:$PORT/hello"
```

For schema validation
```bash
PORT=3000 oha -c 450 -z 10s -m POST -H 'Content-Type: application/json' -H 'x-foo: test' "http://127.0.0.1:$PORT/schema?name=test&excitement=123" -d '{"someKey": "test", "someOtherKey": 123, "requiredKey": [123, 456, 789], "nullableKey": null, "multipleTypesKey": true, "multipleRestrictedTypesKey": "test", "enumKey": "John"}'
```

## Why is the results difference from upstream Encore benchmark
The upstream benchmark use Elysia 1.1.16, and this benchmark use Elysia 1.4.16.

There are a lot of improvement to Elysia performance between both version.
- In [Elysia 1.3](https://elysiajs.com/blog/elysia-13.html#exact-mirror), Elysia introduce [Exact Mirror](https://github.com/elysiajs/exact-mirror) for data normalization using JIT compilation instead of dynamic data mutation
    - Exact Mirror significantly improve schema validation performance
- [Improved Sucrose performance](https://elysiajs.com/blog/elysia-13.html#performance-improvement), Elysia 1.3 introduces general performance improvement for post-JIT code, increasing performance by 40% in the best case
- [System Router](https://elysiajs.com/blog/elysia-13.html#system-router), Elysia 1.3 introduce System Router which use Bun native routing when possible to increase performance
- Bun version, in a single year, Bun improve its general-purpose performance significantly

We highly recommended running the benchmark on your machine for the most accurate result
