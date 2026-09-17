# Learning Plan

> **Note:** This plan is not final. Phases, labs, and exit criteria will evolve
> as work progresses — topics may be reordered, split, or dropped based on what
> turns out to be useful in practice.

Detailed phase-by-phase breakdown of the **revenge** roadmap.

Each phase builds on the last. The exit criteria for every phase are concrete:
you should be able to demonstrate each listed skill on a binary you haven't
seen before, without referring back to the notes.

---

## Phase 0 — QEMU & Cross-Compilation Foundations

**Goal:** Establish a reliable cross-architecture build-and-debug loop so every
later phase can target both x86\_64 and AArch64 without physical ARM hardware.

### Topics

- QEMU user-mode emulation (`qemu-user-static`, `qemu-aarch64-static`).
- `binfmt_misc` kernel module — transparent foreign-binary execution.
- Cross-compilation with `aarch64-linux-gnu-gcc` / `aarch64-linux-gnu-g++`.
- Remote cross-debugging: `gdb-multiarch` ↔ `qemu -g <port>`.

### Labs

| Lab | Description                                                                                                                         |
| --- | ----------------------------------------------------------------------------------------------------------------------------------- |
| 0.1 | Compile a hello-world for AArch64, run it under QEMU, attach gdb-multiarch, set a breakpoint on `main`, and single-step through it. |

### Exit criteria

- [ ] Can cross-compile C programs for AArch64 in one command.
- [ ] Can launch an AArch64 binary under QEMU with a GDB stub.
- [ ] Can connect `gdb-multiarch`, set breakpoints, inspect registers.

### Key references

- [QEMU User-Mode Documentation](https://www.qemu.org/docs/master/user/main.html)
- [Debian binfmt-support](https://wiki.debian.org/QemuUserEmulation)

---

## Phase 1 — Systems Programming & Memory Layout

**Goal:** Build a working mental model of how C and C++ programs are laid out
in memory — from the ELF header down to individual struct fields — so you can
reason about offsets, alignment, and pointer arithmetic the way a reverse
engineer does.

### Topics

- **C memory model:** stack vs. heap vs. BSS vs. data vs. text segments.
- **Struct layout:** padding, alignment, `__attribute__((packed))`,
  `#pragma pack`, `offsetof()`, `sizeof()` surprises.
- **Pointer arithmetic:** array decay, pointer-to-member, void pointers,
  aliasing rules.
- **Dynamic allocation:** `malloc`/`free` internals (glibc arena overview),
  `mmap` anonymous pages.
- **ELF binary structure:** headers, sections, segments, symbol tables,
  relocation entries, GOT/PLT.
- **Mach-O structure** (overview): segments, load commands, `__TEXT`/`__DATA`,
  comparison with ELF.
- **Tooling:** `readelf`, `objdump -d`, `nm`, `size`, `hexdump`, `pahole`.

### Labs

| Lab | Description                                                                                                                                                                |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.1 | Write a struct with mixed types (char, int, double, pointer). Print `sizeof`, `offsetof`, and the address of each field. Compile for x86\_64 and AArch64, compare layouts. |
| 1.2 | Allocate a struct on the stack and on the heap. Use `hexdump`/`xxd` (or GDB `x/` command) to view raw memory around each, identifying padding bytes.                       |
| 1.3 | Write a program that stores data in global, static, stack, and heap variables. Use `readelf -S` and `nm` to map each variable to its ELF section.                          |
| 1.4 | Manually walk an ELF binary with `readelf -a` and `objdump -d`. Identify `.text`, `.rodata`, `.data`, `.bss`, `.got`, `.plt`. Diagram the virtual memory layout.           |
| 1.5 | Write a C program that casts a struct pointer to `char*` and reconstructs field values using raw offset arithmetic. Verify with GDB.                                       |
| 1.6 | Compare the ELF output of `gcc -O0` vs `gcc -O2` for the same source. Note differences in section sizes, symbol visibility, and inlined functions.                         |

### Exit criteria

- [ ] Can predict `sizeof` and `offsetof` for any struct without running code.
- [ ] Can identify which ELF section a variable lives in by looking at its
      storage class and linkage.
- [ ] Can extract a field value from a raw memory dump given only the struct
      definition and a base address.
- [ ] Can explain GOT/PLT lazy binding in one paragraph.

### Key references

- _Computer Systems: A Programmer's Perspective_ (CS:APP) — Chapters 7, 9.
- ELF specification: [man 5 elf](https://man7.org/linux/man-pages/man5/elf.5.html)
- `pahole` (dwarves) for struct layout analysis.

---

## Phase 2 — Compiler Patterns & Disassembly

**Goal:** Read x86\_64 and ARM64 disassembly fluently — recognize compiler
idioms, calling conventions, and control flow patterns so you can reconstruct
high-level logic from stripped binaries.

### Topics

- **x86\_64 ISA essentials:** registers (RAX–R15), addressing modes, common
  instructions (MOV, LEA, CMP, JMP/Jcc, CALL, RET, PUSH/POP), REP prefixes,
  SIMD overview (SSE/AVX).
- **AArch64 ISA essentials:** general registers (X0–X30, SP, LR, PC),
  load/store architecture, condition flags, branch instructions (B, BL, CBZ,
  TBZ), ADRP+ADD literal pools.
- **Calling conventions:** System V AMD64 ABI, AAPCS64. Parameter passing,
  return values, callee/caller-saved registers, red zone.
- **Stack frame anatomy:** prologue (push rbp / stp x29,x30), epilogue,
  local variable allocation, frame pointer vs. frame-pointer-omission.
- **Control flow patterns:** if/else → CMP+Jcc, switch → jump tables,
  loops → back-edges, short-circuit evaluation.
- **Compiler optimizations:** Godbolt exploration of `-O0` through `-O3`,
  inlining, tail calls, strength reduction, loop unrolling, dead code
  elimination.

### Labs

| Lab | Description                                                                                                                                                            |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2.1 | Write 5 small C functions (arithmetic, branching, loops, switch, recursion). Compile at `-O0` and `-O2` for both architectures. Annotate the disassembly line-by-line. |
| 2.2 | Write a function that takes 8+ arguments. Disassemble and trace which go in registers vs. on the stack, for both ABIs.                                                 |
| 2.3 | Write a function with a `switch` over 10+ cases. Identify the jump table in the disassembly and reconstruct the case values.                                           |
| 2.4 | Compile with `-O3` and identify at least three optimization patterns (inlining, tail call, strength reduction). Document each with before/after disassembly.           |
| 2.5 | Use [Godbolt Compiler Explorer](https://godbolt.org) to compare GCC vs. Clang output for the same source at matching optimization levels. Note codegen differences.    |
| 2.6 | Strip a compiled binary (`strip -s`). Reconstruct function boundaries using only `objdump -d` and heuristics (prologue patterns, alignment, call targets).             |

### Exit criteria

- [ ] Can read x86\_64 and ARM64 disassembly and write equivalent C pseudocode.
- [ ] Can identify the calling convention in use from a function's prologue.
- [ ] Can spot at least 3 optimization patterns in `-O2`/`-O3` output.
- [ ] Can locate function boundaries in a stripped binary.

### Key references

- [x86\_64 System V ABI](https://gitlab.com/x86-psABIs/x86-64-ABI)
- [ARM Architecture Reference Manual](https://developer.arm.com/documentation/ddi0487/latest)
- [Godbolt Compiler Explorer](https://godbolt.org)

---

## Phase 3 — Debugging & State Manipulation

**Goal:** Use GDB and LLDB as offensive tools — not just for finding bugs, but
for inspecting, modifying, and hijacking program state at runtime.

### Topics

- **GDB power usage:** TUI mode, Python scripting, pretty-printers, custom
  commands, `.gdbinit` automation.
- **Breakpoints:** software (`int3` / `BRK`) vs. hardware (debug registers),
  conditional breakpoints, watchpoints (data breakpoints).
- **Live memory patching:** `set {int}0x... = value`, `call` arbitrary
  functions, writing to process memory at runtime.
- **Register manipulation:** modify return values, redirect control flow by
  changing RIP/PC, fake function arguments.
- **Execution flow hijack:** overwrite GOT entries, redirect function calls,
  trampoline techniques.
- **LLDB:** command mapping from GDB, scripting via Python, `process`
  commands.
- **Core dumps:** generating, loading, and analyzing post-mortem state.

### Labs

| Lab | Description                                                                                                                                                                                          |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 3.1 | Compile a "password check" program. Use GDB to bypass the check by (a) modifying the comparison register, (b) patching the branch instruction in memory, (c) overwriting the stored password string. |
| 3.2 | Write a program with a global counter. Use a GDB watchpoint to break every time the counter changes, and log the call stack at each hit using a GDB Python script.                                   |
| 3.3 | Write a program that calls `exit(1)`. Use GDB to redirect execution past the exit call so the program prints a success message instead.                                                              |
| 3.4 | Attach GDB to a running process (a simple loop), modify a local variable to change its behavior, then detach cleanly.                                                                                |
| 3.5 | Generate a core dump from a crashing program. Load it in GDB and reconstruct the exact state (registers, stack, heap) at the moment of the crash.                                                    |
| 3.6 | Repeat Lab 3.1 using LLDB instead of GDB. Document command equivalences.                                                                                                                             |

### Exit criteria

- [ ] Can set hardware watchpoints and conditional breakpoints from memory.
- [ ] Can modify registers and memory to alter program behavior at runtime.
- [ ] Can write a GDB Python script that automates a multi-step inspection.
- [ ] Can analyze a core dump and identify root cause of a crash.

### Key references

- [GDB User Manual](https://sourceware.org/gdb/current/onlinedocs/gdb.html)
- [GDB Python API](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Python.html)
- [LLDB to GDB Command Map](https://lldb.llvm.org/use/map.html)

---

## Phase 4 — Static & Dynamic Reversing

**Goal:** Combine static analysis (Ghidra decompilation) with dynamic
instrumentation (Frida, LD\_PRELOAD) to understand and modify closed-source
binaries.

### Topics

- **Ghidra:** project setup, auto-analysis, navigating the decompiler,
  renaming/retyping, applying struct definitions, scripting via Java/Python.
- **Stripped binary analysis:** recovering function signatures, identifying
  vtables (C++), string cross-references, data flow analysis.
- **LD\_PRELOAD injection:** writing shared libraries that intercept libc
  calls (`malloc`, `open`, `send`/`recv`), function interposition.
- **Frida:** JavaScript agent injection, `Interceptor.attach`,
  `Interceptor.replace`, `NativeFunction`, `Memory.read*`/`Memory.write*`,
  stalking (instruction tracing).
- **Hooking strategies:** inline hooks, import table patching, PLT/GOT
  overwriting, detour trampolines.
- **Anti-reversing awareness:** common obfuscation patterns, `ptrace`
  anti-debug, integrity checks — and how to defeat them.

### Labs

| Lab | Description                                                                                                                                                                                  |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 4.1 | Compile a non-trivial C++ program (~500 LOC), strip it, and load it in Ghidra. Recover at least 5 function names and reconstruct 2 struct definitions using only Ghidra's decompiler output. |
| 4.2 | Write an `LD_PRELOAD` library that intercepts `malloc` and logs every allocation (size + returned pointer) to stderr. Load it against a target program and analyze the allocation pattern.   |
| 4.3 | Write an `LD_PRELOAD` library that intercepts `send()`/`recv()` and hexdumps all network traffic to a log file.                                                                              |
| 4.4 | Use Frida to attach to a running process, hook a known function, log its arguments on every call, and modify the return value.                                                               |
| 4.5 | Combine Ghidra + Frida: find an interesting function in Ghidra (static), then use Frida to hook it at runtime and observe real arguments and return values (dynamic).                        |
| 4.6 | Write a simple anti-debug check (`ptrace(PTRACE_TRACEME)`). Then bypass it with (a) `LD_PRELOAD`, (b) GDB patching, (c) Frida.                                                               |

### Exit criteria

- [ ] Can navigate a stripped binary in Ghidra and produce useful pseudocode.
- [ ] Can write an `LD_PRELOAD` interposer for any libc function.
- [ ] Can use Frida to hook, trace, and modify behavior of a running process.
- [ ] Can bypass at least two anti-debugging techniques.

### Key references

- [Ghidra Docs](https://ghidra-sre.org)
- [Frida Docs](https://frida.re/docs/home/)
- _Practical Binary Analysis_ — Dennis Andriesse

---

## Phase 5 — IPC, Offloading & Performance Profiling

**Goal:** Profile real workloads, identify bottlenecks, and build IPC
mechanisms that can offload computation from a running process to an
external one.

### Topics

- **Linux perf:** `perf stat`, `perf record`, `perf report`, hardware
  counters (cache misses, branch mispredictions, IPC).
- **FlameGraphs:** generating, reading, and acting on CPU and off-CPU flame
  graphs.
- **Wireshark / tshark:** capturing network traffic, protocol dissection,
  writing custom Lua dissectors, traffic pattern analysis.
- **Shared memory IPC:** `shm_open`, `mmap`, `ftruncate`, lock-free ring
  buffers, `futex`-based synchronization.
- **ZeroMQ (libzmq):** PUB/SUB, REQ/REP, PUSH/PULL patterns for
  multi-process spatial offloading.
- **eBPF (overview):** `bpftrace` one-liners for tracing syscalls, function
  latency, and memory allocation without modifying the target.

### Labs

| Lab | Description                                                                                                                                                                                 |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 5.1 | Write a CPU-intensive loop (matrix multiply or similar). Profile it with `perf stat` and `perf record`. Generate a FlameGraph. Optimize based on findings and re-profile.                   |
| 5.2 | Run a local UDP echo server. Capture traffic with tshark, write a Lua dissector that parses a custom packet header, display decoded fields.                                                 |
| 5.3 | Build a shared-memory ring buffer (`shm_open` + `mmap`). One process writes structured records, another reads and prints them. Measure throughput.                                          |
| 5.4 | Implement the same producer/consumer from Lab 5.3 using ZeroMQ PUB/SUB. Compare latency and throughput with the shared-memory version.                                                      |
| 5.5 | Attach `bpftrace` to a running process and trace all `malloc` calls with sizes and return addresses. Compare with the LD\_PRELOAD approach from Phase 4.                                    |
| 5.6 | **Integration lab:** Hook a target process with Frida (Phase 4), extract structured runtime data, publish it over ZeroMQ to an external consumer process. Profile the overhead with `perf`. |

### Exit criteria

- [ ] Can profile a workload with `perf` and identify the top hotspot.
- [ ] Can generate and interpret a FlameGraph.
- [ ] Can build a working shared-memory IPC channel between two processes.
- [ ] Can capture and dissect custom UDP traffic with Wireshark/tshark.
- [ ] Can articulate the tradeoff between shm, ZeroMQ, and pipe-based IPC.

### Key references

- [Brendan Gregg — perf examples](https://www.brendangregg.com/perf.html)
- [FlameGraph repo](https://github.com/brendangregg/FlameGraph)
- [ZeroMQ Guide](https://zguide.zeromq.org)
- [bpftrace Reference Guide](https://github.com/bpftrace/bpftrace/blob/master/docs/reference_guide.md)

---

## Phase 6 — Cross-Platform Expansion

**Goal:** Extend the reverse engineering and instrumentation skills built on
Linux/ELF to Windows (PE), Android (DEX/ART + native .so), and iOS (Mach-O).

### Topics

- **Windows / PE:** PE file format, Import Address Table (IAT), WinDbg
  basics, Windows calling convention (Microsoft x64), DLL injection
  (LoadLibrary, manual mapping), API hooking (Detours / MinHook).
- **Android:** APK structure, DEX bytecode, ART runtime, JNI and native .so
  analysis, `frida-server` on Android, `adb` workflow, smali/baksmali.
- **iOS / Mach-O:** Mach-O binary format, dyld shared cache, Objective-C
  runtime introspection, `class-dump`, Frida on jailbroken / non-jailbroken
  (sideloading), Swift metadata.
- **Cross-platform Frida:** writing portable Frida agents that detect the
  host OS/arch and adapt hooks accordingly.

### Labs

| Lab | Description                                                                                                                                                                      |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 6.1 | Compile a simple C program as a Windows PE (cross-compile with MinGW or use a Windows VM). Analyze it with `objdump -p` and compare IAT/PE headers with an ELF binary's GOT/PLT. |
| 6.2 | Set up WinDbg (or WinDbg Preview) and debug a Windows executable. Set breakpoints, inspect the TEB/PEB, walk the loaded module list.                                             |
| 6.3 | Build a simple Android NDK app with a native .so. Pull the APK, extract the .so, and analyze it in Ghidra. Hook a JNI function with Frida on the device/emulator.                |
| 6.4 | Decompile an Android APK with jadx. Identify an interesting method, then hook it at the Java level with Frida's Java API (`Java.perform`).                                       |
| 6.5 | Analyze a Mach-O binary (compile on macOS or obtain a sample). Compare its structure (load commands, segments) with ELF and PE using a side-by-side table.                       |
| 6.6 | Write a single Frida script that hooks the same logical function (e.g., a crypto call) on both Linux and Android, detecting the platform at runtime.                             |

### Exit criteria

- [ ] Can identify imports, exports, and entry points in PE, ELF, and Mach-O.
- [ ] Can use WinDbg for basic debugging of a Windows binary.
- [ ] Can use Frida on Android to hook both Java and native functions.
- [ ] Can articulate key structural differences between PE, ELF, and Mach-O.

### Key references

- [PE Format — Microsoft Docs](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format)
- [Android NDK Docs](https://developer.android.com/ndk)
- [Frida Android Examples](https://frida.re/docs/examples/android/)
- _macOS and iOS Internals_ — Jonathan Levin

---

## Appendix: Tool inventory

A running list of tools used across all phases.

| Tool                         | Phase | Purpose                              |
| ---------------------------- | ----- | ------------------------------------ |
| `gcc` / `g++`                | 0–6   | Native and cross compilation         |
| `aarch64-linux-gnu-gcc`      | 0–6   | AArch64 cross compilation            |
| `qemu-user-static`           | 0–6   | ARM64 user-mode emulation            |
| `gdb-multiarch`              | 0–6   | Cross-architecture debugging         |
| `readelf` / `objdump` / `nm` | 1–6   | ELF inspection                       |
| `pahole`                     | 1     | Struct layout analysis (DWARF)       |
| `hexdump` / `xxd`            | 1–4   | Raw memory inspection                |
| Godbolt                      | 2     | Compiler output comparison           |
| GDB (Python scripting)       | 3     | Automated debug workflows            |
| LLDB                         | 3, 6  | Alternative debugger (macOS primary) |
| Ghidra                       | 4–6   | Static reverse engineering           |
| Frida                        | 4–6   | Dynamic instrumentation              |
| `LD_PRELOAD` libs            | 4–5   | Function interposition               |
| `perf`                       | 5     | Performance profiling                |
| FlameGraph                   | 5     | Visual profiling                     |
| Wireshark / tshark           | 5     | Network capture and analysis         |
| `shm_open` / `mmap`          | 5     | Shared memory IPC                    |
| ZeroMQ                       | 5     | Message-based IPC                    |
| `bpftrace`                   | 5     | eBPF-based tracing                   |
| WinDbg                       | 6     | Windows debugging                    |
| jadx / smali                 | 6     | Android APK decompilation            |
| `class-dump`                 | 6     | Objective-C header recovery          |
