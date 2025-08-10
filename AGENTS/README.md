# videoP2Proxy Repository Overview

This repository contains `videop2proxy`, a proxy that allows Xiaomi P2P cameras to be accessed using standard protocols such as RTSP or raw H264 over stdout.

## Layout
- `main.c` – command-line interface and connection loop.
- `client.c` / `client.h` – orchestrates P2P client operations.
- `iotc.c` / `iotc.h` – wrappers around IOTC API calls.
- `av.c` / `av.h` – handles AV client setup and receiving frames.
- `avframe.c` / `avframe.h` – frame parsing and communication with other modules.
- `rtsp.cxx` / `rtsp.h` – optional RTSP server built with **live555** when `ENABLE_RTSP` is defined.
- `include/` – third‑party headers for IOTC/AV APIs.
- `lib/` – vendor libraries required at link time.
- Build scripts: `autogen.sh`, `configure.ac`, and `Makefile.am` (Autotools).
- `PROTOCOL.md` – Xiaomi P2P handshake and frame format notes for refactors.

## Build & Test
1. Run `./autogen.sh` once to generate the `configure` script and initial Makefile.
2. Build with `make`. This ensures the project compiles cleanly.

## Protocol Reference
Detailed Xiaomi P2P connection steps and the metadata layout of incoming frames are documented in `PROTOCOL.md`. This overview is useful when re-implementing the proxy in languages such as Python, Go, or Rust.

## Coding Guidelines
- The codebase is primarily C with one C++ file (`rtsp.cxx`).
- Source files specify `indent-tabs-mode: t` and `tab-width: 4`; use tabs for indentation at a width of four spaces.
- Keep functions small and avoid trailing whitespace.

## Notes
This document is intended for developers and Codex agents to understand the structure and build process of the project. Future contributions should follow the build and coding guidelines above.
