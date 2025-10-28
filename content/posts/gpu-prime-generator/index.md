---
title: "GPU Prime Generator"
date: 2025-10-28
summary: "Finding prime numbers with Python and CUDA — because math runs faster when it’s parallel."
tags: ["Python", "CUDA", "Numba", "GPU", "Parallel Computing", "Cryptography"]
image: "gpu-prime-generator-banner.png"

---

# GPU Prime Generator

Sometimes you just want to see your GPU *do math*.  
**GPU Prime Generator** is a Python project that uses **Numba’s CUDA backend** to search for large prime numbers in parallel — no C++, no custom kernels, just high-level Python code running directly on your NVIDIA GPU.

---

## The Idea

Prime generation has always been a fun problem. It’s simple, infinite, and surprisingly good at stress-testing hardware.  
Traditionally, you’d write a sieve or a primality test and let your CPU churn away — one number at a time. But GPUs are designed for massive, embarrassingly parallel workloads, which makes them perfect for range testing.

So this project asks a simple question:

> *How fast can a Python script find primes if we hand the math to the GPU?*

---

## How It Works

Under the hood, `gpu-prime-generator` uses:
- **Numba** to JIT-compile Python functions into CUDA kernels.
- **CuPy** and **NumPy** for fast array operations and data handling.
- Thousands of GPU threads, each testing different numbers at once.

You can specify a starting number and range size, and the tool will fan out those checks across your GPU cores. The results come back almost instantly for mid-sized ranges.

```bash
python gpu_prime_generator.py --start 1000000000 --count 10
```

Example output:

```
GPU Prime Generator starting at 1,000,000,000
Found 10 primes in 0.42s
Peak GPU threads: 32768
```

---

## Why Bother?

There are faster ways to find primes — but that’s not the point.  
This project exists because it’s *cool* to watch number theory scale across thousands of CUDA cores.

It’s a great sandbox if you’re curious about:
- How to map **serial math to parallel compute**
- How **Numba** bridges Python and GPU code
- Or just how quickly your GPU can brute-force something with minimal overhead

---

## Setup

Requirements:
- Python 3.10+
- NVIDIA GPU with CUDA Compute Capability 6.0+
- `numba`, `numpy`, `cupy`
- CUDA Toolkit 12+

```bash
git clone https://github.com/mgelsinger/gpu-prime-generator.git
cd gpu-prime-generator
pip install -r requirements.txt
```

---

## Takeaways

Even a small experiment like this highlights how accessible GPU programming has become in Python.  
You don’t need C++ or deep CUDA knowledge — just a few decorators and some vectorized logic.

The result? A tool that crunches through primes at ridiculous speeds and doubles as a lightweight benchmark or teaching demo for parallel computing.

---

Source:  
👉 [github.com/mgelsinger/gpu-prime-generator](https://github.com/mgelsinger/gpu-prime-generator)

---

*Part of the [glsngr.xyz](https://glsngr.xyz) portfolio — open tools and experiments built to explore performance, parallelism, and practical computing.*
