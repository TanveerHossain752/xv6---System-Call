# xv6 System Call Extension: Trace & History

This repository contains an extended version of the xv6 operating system for **Assignment 2: xv6 System Call** (CSE 314). Two new system calls, `trace` and `history`, have been implemented to monitor process behavior and aggregate system call statistics across the OS.

---

## Table of Contents
1. [Setup & Patch Application](#setup--patch-application)
2. [Task 1: System Call Tracing (`trace`)](#task-1-system-call-tracing-trace)
3. [Task 2: System Call History (`history`)](#task-2-system-call-history-history)
4. [File Structure & Modifications](#file-structure--modifications)
5. [Building, Running, and Submitting](#building-running-and-submitting)

---

## Setup & Patch Application

To clone the official base repository:

```bash
git clone https://github.com/shuaibw/xv6-riscv --depth=1
cd xv6-riscv
```

To apply the implementation patch (`2205112.patch`):

```bash
git apply 2205112.patch
```

---

## Task 1: System Call Tracing (`trace`)
The `trace` system call enables tracking of specific system calls made by a user process.

### Features
* **Targeted Tracing:** Accepts an integer argument representing the system call number to trace.
* **Detailed Output:** When a traced system call returns, the kernel prints:
  * Process ID
  * System call name
  * Typed arguments (integers, pointers, strings)
  * Return value
* **Process Isolation:** Tracing is enabled strictly for the calling process and does not affect other processes.

### Implementation Details
* **Process State:** Added a `trace_num` field to `struct proc` (`kernel/proc.h`), initialized to `-1` in `allocproc()` (`kernel/proc.c`).
* **Argument Parsing & Printing:** Added `syscallnames[]` and an `argtypes` mapping matrix (`ARG_NONE`, `ARG_INT`, `ARG_PTR`, `ARG_STR`) in `kernel/syscall.c`. The helper function `print_syscall_args()` handles typed argument extraction and uses `fetchstr()` to fetch string values safely.
* **Interception:** In `syscall()` (`kernel/syscall.c`), argument values are captured prior to execution, and formatted output is printed upon completion if `p->trace_num == num`.
* **User Utility:** Provided `user/trace.c` which invokes `trace()` and executes commands using `exec()`.

---

## Task 2: System Call History (`history`)
The `history` system call aggregates and retrieves statistics about system call usage across the operating system.

### Features
* **Aggregated Statistics:** Tracks total call count for each system call since boot-up.
* **Time Tracking:** Measures CPU time (in xv6 ticks) consumed by each system call.
* **User-Space Output:** Fetches data from kernel space and formats output strictly in user mode (`user/history.c`).

### Implementation Details
* **Data Structure:** Defined `struct syscall_stat` in `kernel/stat.h`:
  ```c
  struct syscall_stat {
    char syscall_name[16];
    int count;
    int accum_time;
  };
  ```
* **Timing & Execution:** Updated `syscall()` in `kernel/syscall.c` to capture start (`ticks0`) and end (`ticks1`) ticks via `tickslock` and update statistics using `record_syscall_stat()`.
* **Concurrency & Locking:** Initialized an array of fine-grained spinlocks (`stat_lock[NELEM(syscalls)]`) during kernel boot via `syscallstatsinit()` (invoked in `kernel/main.c`) to safely support multiprocessor execution.
* **Memory Transfer:** `history()` uses `copyout()` to safely pass struct data to user space memory.

---

## File Structure & Modifications

| File | Type | Modifications |
|---|---|---|
| `Makefile` | Build | Added `$U/_trace` and `$U/_history` targets to `UPROGS`. |
| `kernel/defs.h` | Header | Added prototypes for `syscallstatsinit()` and `history()`. |
| `kernel/main.c` | Kernel Init | Called `syscallstatsinit()` during boot initialization. |
| `kernel/proc.h` & `kernel/proc.c` | Kernel Core | Added `trace_num` to `struct proc` and initialized it in `allocproc()`. |
| `kernel/stat.h` | Header | Defined `struct syscall_stat`. |
| `kernel/syscall.h` | Header | Assigned `#define SYS_trace 22` and `#define SYS_history 23`. |
| `kernel/syscall.c` | Kernel Core | Implemented argument formatting, timer tracking, spinlock handling, and `history()` logic. |
| `kernel/sysproc.c` | Handlers | Added system call wrappers `sys_trace()` and `sys_history()`. |
| `user/user.h` & `user/usys.pl` | User Interface | Declared prototypes and generated user assembly stubs. |
| `user/trace.c` | User Command | CLI tool to execute commands with active tracing. |
| `user/history.c` | User Command | CLI tool to query single or all system call statistics. |

---

## Building, Running, and Submitting

### 1. Build and Run in QEMU
```bash
make qemu
```

### 2. Shell Example Commands
```bash
$ trace 15 grep hello README
$ history 5
$ history
```

### 3. Generating Patch File
```bash
git add --all
git diff HEAD > 2205112.patch
```

### 4. Verifying Patch Application
```bash
git clone https://github.com/shuaibw/xv6-riscv test-repo --depth=1
cd test-repo
git apply ../2205112.patch
make qemu
```
