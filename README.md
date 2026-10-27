# Operating Systems Assignment 9: Memory Management & Page Tables in xv6

This repository contains implementations for **xv6 (x86)** covering Page Table Inspection, Lazy Memory Allocation via Page Fault Handling, and Page Replacement Algorithm Simulations (FIFO & LRU).

---

## Environment Setup & macOS Troubleshooting

### Prerequisites

* **Toolchain:** `x86_64-elf-gcc` (GCC 16+), `x86_64-elf-binutils`
* **Emulator:** `qemu-system-i386`
* **Host OS:** macOS / Linux

## Assignment Tasks & Implementation Details


### Question 1: Printing Process Page Tables (`vmprint`)

#### Overview

Implemented a system call `int vmprint(void)` that walks the two-level page table of the calling process, displaying virtual-to-physical mappings, page permissions (`P`, `W`, `U`), and total page frame statistics.

#### Implementation Details

1. **Kernel Page Table Walk (`vm.c`):**
* Traversing the process's page directory (`p->pgdir`).
* Inspecting Page Directory Entries (PDEs) and Page Table Entries (PTEs) below `KERNBASE` (`0x80000000`).
* Decoding page flags: `PTE_P` (Present), `PTE_W` (Writable), `PTE_U` (User).


2. **System Call Plumbing:**
* Declared `sys_vmprint(void)` in `sysproc.c`.
* Added `SYS_vmprint` entry in `syscall.h` and `syscall.c`.
* Added user stub `int vmprint(void);` in `user.h` and `usys.S`.


3. **User Test Program (`pttest.c`):**
* Invokes `vmprint()` at process start.
* Calls `sbrk(4 * 4096)` to request 4 additional memory pages.
* Invokes `vmprint()` again to verify exactly 4 new mappings appear.



---

### Question 2: Lazy Allocation & Page Fault Handling (`T_PGFLT`)

#### Overview

Modified `sbrk()` to lazily defer physical memory allocation until page access, handling page faults (`T_PGFLT`) on demand within the kernel trap handler.

#### Implementation Details

1. **Defer Allocation in `sys_sbrk()` (`sysproc.c`):**


2. **Update Memory Copy/Deallocation Routines (`vm.c`):**
* Modified `deallocuvm()` and `copyuvm()` to ignore unmapped pages instead of triggering a kernel panic when encountering non-present PTEs (`!(pte & PTE_P)`).


3. **Page Fault Trap Handler (`trap.c`):**
* Caught `T_PGFLT` exceptions.
* Read faulting virtual address using CR2 register (`rcr2()`).
* Checked if `fault_addr < myproc()->sz` and above stack guard pages.
* Allocated a new physical frame with `kalloc()`, cleared it with `memset()`, and mapped it via `mappages()` with `PTE_W | PTE_U`.
* Incremented `p->page_faults` counter and resumed execution.


4. **System Call `getfaults()`:**
* Returns `myproc()->page_faults`.


5. **User Test Program (`faulttest.c`):**
* Calls `sbrk(32 * 4096)`.
* Writes to 5 distinct virtual pages.
* Asserts `getfaults() == 5`.
* Attempts out-of-bounds access beyond `myproc()->sz` to confirm process termination.



---

### Question 3: Page Replacement Algorithms (FIFO & LRU)

#### Overview

Created a standalone user-level simulator (`pagerepl.c`) to compare First-In-First-Out (FIFO) and Least Recently Used (LRU) page replacement policies across various frame allocations.

#### Reference String Test Case

* **Reference Sequence:** `7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1`
* **Frames:** 3, 4, and 5

#### Expected Results (3 Frames)

* **FIFO:** 15 Page Faults
* **LRU:** 12 Page Faults

---

## Build & Run Instructions

```bash
# Clean previous build artifacts
make clean

# Compile kernel and utility programs
make TOOLPREFIX=x86_64-elf-

# Launch xv6 inside QEMU (No-GUI console mode)
make qemu-nox TOOLPREFIX=x86_64-elf-

```

---

## Submission Checklist

* Modified Kernel Source Files (`vm.c`, `trap.c`, `proc.h`, `sysproc.c`, `syscall.c`, `syscall.h`, `defs.h`, `user.h`, `usys.S`)
* User Programs (`pttest.c`, `faulttest.c`, `pagerepl.c`)
* Terminal Output Screenshots
* 1-Page Summary Report Document
