---
layout: post
title: A C++ Triton Backend for TensorRT-LLM BLS
subtitle: Reimplementing the BLS pipeline in C++ so it can handle hundreds of concurrent requests
cover-img: /assets/img/coroutine-cover.png
thumbnail-img: /assets/img/tensorrtllm-bls-cover.png
share-img: /assets/img/tensorrtllm-bls-cover.png
tags: [Triton, TensorRT-LLM, C++, LLM inference, coroutines]
author: Binh Tran
gh-repo: binhtranmcs/tensorrtllm_bls_backend
gh-badge: [star, fork, follow]
---

If you've deployed an LLM behind [Triton Inference Server](https://github.com/triton-inference-server/server) using [TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM), you've likely run into the `tensorrt_llm_bls` model. It's the piece of BLS (Business Logic Scripting) glue that chains `preprocessing` → `tensorrt_llm` → `postprocessing` together into a single request/response contract for the client. The reference implementation NVIDIA ships for this is a Python backend.

That Python implementation works fine at low concurrency, but it doesn't scale well once you're pushing hundreds of concurrent streaming requests through a single model instance. Thread-per-request overhead, the GIL, and Python-level scheduling all become the bottleneck before the GPU ever does. I wanted the orchestration layer to stop being the ceiling, so I wrote [`tensorrtllm_bls_backend`](https://github.com/binhtranmcs/tensorrtllm_bls_backend) — the same BLS pipeline, reimplemented as a native C++ Triton backend.

## What it does

`tensorrtllm_bls_backend` is a compiled Triton backend (a `.so`, built against Triton's in-process C-API) that reproduces the full `tensorrt_llm_bls` contract:

- Orchestrates `preprocessing` → `tensorrt_llm` → `postprocessing`, streaming tokens back to the client as they're generated
- Supports multimodal input (raw image tensors or base64), routed to a multimodal encoder either in-process or over HTTP to an external Triton server
- Computes top-k log-probabilities for context and generation logits, synchronously or asynchronously
- Handles guided decoding, LoRA task ids, embedding bias, and the draft-token fields needed for speculative decoding inputs
- Correctly stitches together partial UTF-8 token boundaries across streaming chunks
- Reports per-request and per-batch statistics back to Triton, and traces latency through every stage of the pipeline

In other words, it's meant to be a drop-in replacement for the Python BLS model — same inputs, same outputs, same deployment shape — just implemented differently underneath.

## Why C++ and coroutines

The core problem with the Python backend is concurrency model: each in-flight request effectively needs its own thread to sit and wait on I/O (calls to the other models, waiting for tokens to stream back). That's expensive to scale.

This backend instead spawns one **coroutine** per request on a shared `boost::asio::io_context`, using Boost.Asio's C++20 coroutine support. Every call out to another model — `preprocessing`, `tensorrt_llm`, `postprocessing`, the multimodal encoder — goes through Triton's in-process C-API and gets bridged into an awaitable result via a `concurrent_channel`. Nothing blocks a thread while waiting; the coroutine just suspends until its result is ready and control goes back to the scheduler to service other requests.

The result is that a single model instance can hold a large number of concurrent streaming requests open at once, with far less per-request overhead than the thread-based Python approach — which is exactly what's needed to keep the orchestration layer from becoming the bottleneck at scale.

## Where it stands

It's functional and covers the bulk of the `tensorrt_llm_bls` feature surface — streaming, multimodal, log-probs, guided decoding are all working. Still on my list: async CUDA memory operations, GPU-batched log-prob calculation, actual speculative decoding support (the input/output fields are there, the logic isn't yet), stop-word-triggered cancellation, and finer-grained metrics like TTFT/TPOT.

If you're running TensorRT-LLM behind Triton and the BLS model is showing up as your bottleneck under load, take a look: [github.com/binhtranmcs/tensorrtllm_bls_backend](https://github.com/binhtranmcs/tensorrtllm_bls_backend).
