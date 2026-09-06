# xv6 System Call Extension: Trace & History

## Overview
This repository contains an extended version of the xv6 operating system. Two new system calls, `trace` and `history`, have been implemented to monitor and track system call activity. This is particularly useful for tracking process behavior and aggregating system call statistics.

## Task 1: System Call Tracing (`trace`)
The `trace` system call enables the tracking of specific system calls made by a user process.

### Features
* **Targeted Tracing:** Accepts a single integer argument representing the system call number to trace.
* **Detailed Output:** When a traced system call returns, the kernel prints:
  * Process ID
  * System call name
  * Typed arguments (integers, pointers, or strings)
  * Return value
* **Process Isolation:** Tracing is enabled only for the process that calls `trace` and does not affect other concurrent processes.

### Implementation Details
* **Process State:** Added a `trace_num` field to the `proc` structure to track the active trace target.
* **Argument Parsing:** Implemented an `argtypes` mapping matrix in `kernel/syscall.c` to format and print arguments dynamically based on the system call signature.
* **User Utility:** A user-space program `trace` (`user/trace.c`) is provided to easily run commands with tracing enabled (e.g., `trace 5 grep hello README`).

## Task 2: System Call History (`history`)
The `history` system call aggregates and retrieves statistics about system call usage across the entire operating system.

### Features
* **Aggregated Statistics:** Tracks the total number of times each system call has been invoked since boot.
* **Time Tracking:** Measures the total CPU time (in xv6 ticks) consumed by each system call.
* **Query Utility:** The `history` user program (`user/history.c`) fetches and displays statistics for a specific system call or all system calls.

### Implementation Details
* **Data Structure:** Utilizes a `syscall_stat` structure containing the system call name, invocation count, and accumulated time.
* **Concurrency & Locking:** Since xv6 is a multiprocessor environment, a dedicated array of spinlocks (`stat_lock`) is used to prevent race conditions when multiple CPUs update system call statistics simultaneously.
* **Global Tracking:** Statistics are stored globally in `syscall_stats` and updated safely inside the main `syscall()` handler.

## File Structure
Key modifications and additions include:
* `Makefile`: Added compilation targets for `trace` and `history` user programs.
* `kernel/syscall.c` & `kernel/syscall.h`: Core logic, data structures, and handlers for `SYS_trace` and `SYS_history`.
* `kernel/proc.h` & `kernel/proc.c`: Process state tracking for tracing.
* `kernel/stat.h`: Definition of `struct syscall_stat`.
* `user/trace.c` & `user/history.c`: Command-line utilities for user interaction.
* `user/usys.pl` & `user/user.h`: User-space stubs mapping to the kernel system calls.

## Building and Running
1. Ensure you have the xv6-riscv build environment setup.
2. Build the OS and boot into qemu:
   ```bash
   make qemu
   ```
3. Use the utilities in the xv6 shell:
   ```bash
   $ trace 15 grep hello README
   $ history 5
   $ history
   ```
