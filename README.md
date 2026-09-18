<h1>
    <picture>
        <source media="(prefers-color-scheme: dark)" srcset=".github/assets/revenge.svg" />
        <source media="(prefers-color-scheme: light)" srcset=".github/assets/revenge.svg" />
        <img alt="revenge" src=".github/assets/revenge.svg" width="280" />
    </picture>
</h1>

**rev**erse **eng**ineering — a learning project.

A structured, hands-on roadmap for learning systems programming, binary
reverse engineering, dynamic instrumentation, network analysis, and performance
profiling.

> **Audience:** me, first. If you stumbled on this and it helps you too, great.

## Motivation

Large, stripped, closed-source binaries require comfort across the full stack
to understand: compilers, ABIs, memory layout, IPC, OS internals, and the
tools built to peer inside running processes.

This repository is the lab notebook for building that comfort, one compilable
exercise at a time.

## Methodology

```
Write  →  Compile  →  Reverse
```

Every concept is learned by **writing** minimal C/C++ (or Zig/Rust) programs,
**compiling** them for x86\_64 and AArch64, then **reversing** the resulting
binaries with objdump, Ghidra, GDB, and Frida — comparing what the source
_intended_ with what the machine _actually does_.

Difficulty increases progressively across seven phases.
See **[PLAN.md](PLAN.md)** for the full breakdown.

## Roadmap at a glance

| #   | Phase                                                                                             |
| :-- | :------------------------------------------------------------------------------------------------ |
| 0   | [QEMU & Cross-Compilation Foundations](PLAN.md#phase-0--qemu--cross-compilation-foundations)      |
| 1   | [Systems Programming & Memory Layout](PLAN.md#phase-1--systems-programming--memory-layout)        |
| 2   | [Compiler Patterns & Disassembly](PLAN.md#phase-2--compiler-patterns--disassembly)                |
| 3   | [Debugging & State Manipulation](PLAN.md#phase-3--debugging--state-manipulation)                  |
| 4   | [Static & Dynamic Reversing](PLAN.md#phase-4--static--dynamic-reversing)                          |
| 5   | [IPC, Offloading & Performance Profiling](PLAN.md#phase-5--ipc-offloading--performance-profiling) |
| 6   | [Cross-Platform Expansion](PLAN.md#phase-6--cross-platform-expansion)                             |

## Host environment

| Property             | Value                                         |
| -------------------- | --------------------------------------------- |
| Host OS              | Debian 12 (Proxmox VE, `cpu=host`)            |
| Target architectures | x86\_64, AArch64 (ARM64 via qemu-user-static) |
| Cross toolchain      | `aarch64-linux-gnu-gcc`, `gdb-multiarch`      |
| Emulation            | QEMU user-static + binfmt\_misc               |

## Repository layout

```
revenge/
├── README.md
├── PLAN.md
├── LICENSE
├── phase-00-qemu/
├── phase-01-memory/
├── phase-02-disasm/
├── phase-03-debug/
├── phase-04-reversing/
├── phase-05-ipc-perf/
├── phase-06-cross-plat/
└── tools/
```

Each `phase-NN-*/` directory will contain:

- `README.md` — phase-specific objectives, notes, and observations.
- `lab-XX/` — self-contained exercises with source, Makefile, and a
  short writeup of findings.

## Prerequisites

- Comfortable reading C (doesn't have to be pretty — just has to compile).
- A Linux box or VM with root access.
- `gcc`, `aarch64-linux-gnu-gcc`, `gdb-multiarch`, `qemu-user-static`,
  `binutils`, `objdump`, `readelf`.
- Ghidra (Phase 4+), Frida (Phase 4+), Wireshark (Phase 5+).

## License

This project is licensed under the [MIT License](LICENSE).
Code samples, lab exercises, and documentation are all covered under the same
terms — use, modify, and share freely.
