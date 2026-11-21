+++
title = "Sentinel-DPI"
description = "A lightweight Deep Packet Inspection engine written from scratch in modern C++20 with zero third-party dependencies. Streams .pcap captures, decodes L2–L4 headers from raw bytes, and matches traffic against Snort-inspired rules."
date = 2026-11-21

[extra]
repo = "https://github.com/TheMatako/C_plus_plus/tree/main/Sentinel-DPI"
full_name = "Deep Packet Inspection Engine"

[[extra.status]]
state = "done"
text = "Streaming .pcap reader in constant memory (little/big endian, microsecond and nanosecond timestamps)."
[[extra.status]]
state = "done"
text = "Multi-layer binary parser decoding Ethernet, IPv4, TCP, UDP, and ICMP from raw frames."
[[extra.status]]
state = "done"
text = "Hand-written Snort-style rule parser (RuleLoader) and short-circuit matching engine (RuleEngine) with IP/port filters, TCP flag bitmasks, and case-insensitive payload search."
[[extra.status]]
state = "done"
text = "End-to-end validation against forged captures (including edge cases and false-positive checks), Valgrind leak-free verification, and v1.0.0 release."
[[extra.status]]
state = "next"
text = "Automated unit test suite, structured alert output (JSON) for SIEM/log ingestion and stateful TCP stream reassembly to inspect payloads split across multiple segments."
+++

## Why it exists

SentinelDPI is a standalone C++20 project built to understand how network intrusion detection and packet inspection work directly at the wire level. Rather than wrapping `libpcap` or relying on external parsing libraries, every layer from binary file headers to rule evaluation is implemented from scratch using only the C++ standard library.

## Core design

- **Zero third-party dependencies:** Built strictly on the C++20 standard library (`std::span`, `std::format`, `std::search`), handling binary framing, endianness, and protocol decoding manually.
- **Constant-memory streaming:** Captures are read and inspected packet by packet instead of loading entire `.pcap` files into RAM.
- **Fail-fast rule engine:** Custom DSL parser with line-numbered syntax errors, paired with short-circuit matching ordered from cheapest to most expensive check (protocol, IP, port, TCP flags, payload).
- **Defensive C++ hygiene:** Compiled under strict warnings (`-Wall`, `-Wextra`, `-Wshadow`, `-Wswitch`) and verified memory-safe with zero leaks under Valgrind.

## Pipeline & verification

Each frame flows through a single linear pipeline: `PcapReader` streams raw bytes, `PacketParser` slices them into typed protocol layers using non-owning `std::span` views, and `RuleEngine` builds a lightweight `PacketContext` once per packet before evaluating all loaded rules. To avoid extra allocations during payload inspection, signature matching runs directly on the raw byte buffer with `std::search`. The `v1.0.0` baseline was validated end-to-end against custom-forged `.pcap` captures covering SSH SYN scans, case-insensitive HTTP SQL injection with combined TCP flag masks (`PA` vs `A`), DNS queries, ICMP traffic, and benign HTTPS noise, confirming zero false positives and zero memory leaks under Valgrind.

## What comes next

The `v1.0.0` engine is complete as a stateless, per-packet inspector. Possible extensions include a unit test suite, JSON alert output, and stateful TCP stream reassembly for multi-packet payloads. I may leave the project at `v1.0.0`, so feel free to clone the repo and pick up from there.
