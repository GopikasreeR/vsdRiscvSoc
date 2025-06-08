# vsdRiscvSoc
# RISC-V DIR-V Grand Challenge – Week 1 Tasks

This repository documents my Week 1 progress in the C2S RISC-V (DIR-V) Grand Challenge. The tasks explore the RISC-V toolchain, bare-metal programming, inline assembly, memory-mapped I/O, and emulator usage.

---

## 📁 Task List


| ✅ | Task # | Title                             | Summary                                                  |
|----|--------|-----------------------------------|----------------------------------------------------------|
| ✅ | 1      | 🔧 Toolchain Setup                | Install, add to PATH, verify gcc/gdb/objdump             |
| ✅ | 2      | 🖥️ Hello World (Cross-Compilation)| Build a minimal RISC-V ELF using C                       |
| ✅ | 3      | 🧠 C to Assembly                  | Convert C to assembly and analyze function structure     |
| ✅ | 4      | 🔍 Disassembly & Hex Dump        | Dump ELF to human-readable and hex formats               |
| ✅ | 5      | 🧾 ABI & Register Cheatsheet     | RV32I register roles and calling conventions             |
| ✅ | 6      | 🐞 GDB Debugging                 | Set breakpoints, step through execution                  |
| ✅ | 7      | 🧪 QEMU Simulation                | Boot and run ELF on QEMU or Spike emulator               |
| ✅ | 8      | ⚡ GCC Optimizations             | Compare `-O0` and `-O2` optimizations                    |
| ✅ | 9      | 🛠️ Inline Assembly              | Read CPU cycle count with inline asm                     |
| ✅ | 10     | 💡 Memory-Mapped GPIO           | Toggle GPIO via memory-mapped I/O                        |
| ✅ | 11     | 🗺️ Linker Script Basics         | Map sections to memory manually                          |
| ✅ | 12     | 🧬 crt0 & Startup Code          | Bootstrap the program from `_start`                      |
| ✅ | 13     | ⏱️ Timer Interrupts             | Enable MTIP & write interrupt handlers                   |
| ✅ | 14     | 🔐 Atomic Extension ("A")       | Learn lr.w, sc.w, and AMO instructions                   |
| ✅ | 15     | 🤝 Mutex with Atomics           | Implement spinlocks using atomic operations              |
| ✅ | 16     | 📤 Retarget printf to UART      | Implement `_write()` to print to UART                    |
| ✅ | 17     | 🔁 Endianness & Struct Packing  | Verify little-endian with unions                         |

---

### ✅ Task 1: 🔧 Toolchain Setup  
- Installed `riscv-none-elf` toolchain
- Added to PATH via `~/.bashrc`
- Verified using:
  ```bash
  riscv32-unknown-elf-gcc --version
  riscv32-unknown-elf-objdump --version
  riscv32-unknown-elf-gdb --version
![image](https://github.com/user-attachments/assets/bc889cc1-6916-435d-801e-b2697c592962)
![image](https://github.com/user-attachments/assets/2c63d152-8ece-4dea-8832-2a7365c9eebd)
![image](https://github.com/user-attachments/assets/5b4b6efb-600a-4112-a92e-ba6d79412a71)

### ✅ Task 2: 🖥️ Hello World (Cross-Compilation)

Step 1: Create the Hello World C Program
Create a minimal C program that demonstrates basic functionality and printf usage.
Create a file named `hello.c`:

Copy and paste the following code into the editor:
```
#include <stdio.h>
int main() {
printf("Hello, RISC-V!\n");
return 0;
}
```
Save and exit nano:

Press Ctrl + O then Enter to save.
Press Ctrl + X to exit the editor.

Step 2: Compile the code
Run the following command to compile the program for the RISC-V architecture:
```
riscv-none-elf-gcc -march=rv32imc -mabi=ilp32 -o hello.elf hello.c
```
-march=rv32imc specifies the architecture (RV32IMC).

-mabi=ilp32 specifies the calling convention/ABI.

-o hello.elf sets the output file name.

Step 3: Verify the ELF file
Run the following command to check the file type of the compiled executable:
```
file hello.elf
```

You should see output indicating that the file is an ELF executable for the RISC-V architecture.

![image](https://github.com/user-attachments/assets/87bcad12-f21a-4f29-b989-355615312ecc)

### ✅ Task 3: 🧠 C to Assembly
Step 1: Generate the Assembly (.s) file
If your source file is hello.c, run:
```
riscv-none-elf-gcc  -S -O0 hello.c
```
 -S: Compile to assembly, not machine code
 -O0: Disable optimization (shows all instructions clearly)

Step 2: Open hello.s
You can open the file with:
```
cat hello.s
```
Look for a section like this inside main::
```
main:
    addi    sp,sp,-16
    sw      ra,12(sp)
    sw      s0,8(sp)
    addi    s0,sp,16
    ...
    lw      s0,8(sp)
    lw      ra,12(sp)
    addi    sp,sp,16
    ret
```
Step 3: Understand the Prologue and Epilogue
   Prologue (at the start of main)
```
addi    sp,sp,-16     # Make space on the stack (16 bytes)
sw      ra,12(sp)     # Save return address (ra) at offset 12
sw      s0,8(sp)      # Save frame pointer (s0) at offset 8
addi    s0,sp,16      # Set up new frame pointer (s0 = sp + 16)
```
This sets up a stack frame, preserving the return address (ra) and old frame pointer (s0), preparing for function calls or local variables.

Epilogue (at the end of main)
```
lw      s0,8(sp)      # Restore frame pointer
lw      ra,12(sp)     # Restore return address
addi    sp,sp,16      # Deallocate stack space
ret                   # Return to caller
```

This restores the state before returning from main, undoing the changes made by the prologue.

![image](https://github.com/user-attachments/assets/1b6c420c-98ac-4cc5-8a1d-92af2c0ff8f0)
![image](https://github.com/user-attachments/assets/ea51715b-dfbc-4b60-a3df-262f3888dbb4)

### ✅ Task 4: 🔍 Disassembly & Hex Dump 

Step 1. Disassemble your ELF file into readable format:
```
riscv-none-elf-objdump -d hello.elf > hello.dump
```
Now open hello.dump:
```
cat hello.dump
```
![image](https://github.com/user-attachments/assets/65abc345-5afa-417b-b71f-ecf354795b5a)

Step 2. Create a HEX file (raw machine code):
```
riscv-none-elf-objcopy -O ihex hello.elf hello.hex
cat hello.hex
```
This produces the file in Intel HEX format — this is useful if you're programming actual hardware.

🧾 What You’ll See in hello.dump
Let’s take a sample disassembly line:
```
00000000 <main>:
   0: 1141                 addi	sp,sp,-16
   2: c606                 sw	ra,12(sp)
   4: c422                 sw	s0,8(sp)
   ...
```
![image](https://github.com/user-attachments/assets/e4925f6f-463a-4268-bcd7-4b294f49944b)

### ✅ Task 5:🧠 RISC-V ABI & Register Cheat-Sheet

This cheat-sheet provides a quick reference for all **32 RV32 integer registers** (`x0`–`x31`) along with their **ABI names** and **typical roles** in the calling convention.

---

## 🧾 Register Table (x0 - x31)

| Register | ABI Name | Role                             | Description                     |
|----------|----------|----------------------------------|---------------------------------|
| x0       | zero     | Constant zero                    | Always zero                     |
| x1       | ra       | Return address                   | Stores return address for calls |
| x2       | sp       | Stack pointer                    | Points to top of stack          |
| x3       | gp       | Global pointer                   | Points to global data           |
| x4       | tp       | Thread pointer                   | Points to thread data           |
| x5       | t0       | Temporary                        | Caller-saved temporary          |
| x6       | t1       | Temporary                        | Caller-saved temporary          |
| x7       | t2       | Temporary                        | Caller-saved temporary          |
| x8       | s0/fp    | Saved register / Frame pointer   | Callee-saved / frame pointer    |
| x9       | s1       | Saved register                   | Callee-saved                    |
| x10      | a0       | Function argument / return value | 1st argument / return value     |
| x11      | a1       | Function argument / return value | 2nd argument / return value     |
| x12      | a2       | Function argument                | 3rd argument                    |
| x13      | a3       | Function argument                | 4th argument                    |
| x14      | a4       | Function argument                | 5th argument                    |
| x15      | a5       | Function argument                | 6th argument                    |
| x16      | a6       | Function argument                | 7th argument                    |
| x17      | a7       | Function argument                | 8th argument                    |
| x18      | s2       | Saved register                   | Callee-saved                    |
| x19      | s3       | Saved register                   | Callee-saved                    |
| x20      | s4       | Saved register                   | Callee-saved                    |
| x21      | s5       | Saved register                   | Callee-saved                    |
| x22      | s6       | Saved register                   | Callee-saved                    |
| x23      | s7       | Saved register                   | Callee-saved                    |
| x24      | s8       | Saved register                   | Callee-saved                    |
| x25      | s9       | Saved register                   | Callee-saved                    |
| x26      | s10      | Saved register                   | Callee-saved                    |
| x27      | s11      | Saved register                   | Callee-saved                    |
| x28      | t3       | Temporary                        | Caller-saved temporary          |
| x29      | t4       | Temporary                        | Caller-saved temporary          |
| x30      | t5       | Temporary                        | Caller-saved temporary          |
| x31      | t6       | Temporary                        | Caller-saved temporary          |

---

## 📜 RISC-V Calling Convention Summary

| Category      | Registers         | Description                                                  |
|---------------|------------------|--------------------------------------------------------------|
| Return value  | `a0–a1`          | Used to return values from functions                         |
| Arguments     | `a0–a7`          | Used to pass the first 8 arguments to functions              |
| Caller-saved  | `t0–t6`, `a0–a7` | Must be saved/restored by the caller if needed after a call  |
| Callee-saved  | `s0–s11`         | Must be saved/restored by the callee                         |
| Stack pointer | `sp`             | Points to the top of the stack                               |
| Frame pointer | `s0/fp`          | Used to reference the current function’s stack frame         |
| Return addr   | `ra`             | Holds the return address for function calls                  |

---

### ✅ Task 6:🐞 GDB Debugging  
Step 1: Install QEMU (User-Mode)
```
sudo apt update
sudo apt install qemu-user
```
This installs qemu-riscv32, which runs RISC-V ELF binaries in user mode.

Verify the install
```
qemu-riscv32 --version
```
Step 2: Create hello.c
```
#include <stdio.h>

int main() {
    printf("Hello, RISC-V!\n");
    return 0;
}
```
Save this as hello.c

Step 3: Compile with Debug Symbols
```
xpack-riscv-none-elf-gcc-14.2.0-3/bin/riscv-none-elf-gcc \
  -march=rv32imc -mabi=ilp32 -g -o hello.elf hello.c
```
This produces a hello.elf

Step 4: Start QEMU with GDB Stub
```
qemu-riscv32 -g 1234 hello.elf
```
It will now wait for GDB to connect on port 1234.

Step 5: Start GDB in Another Terminal
```
xpack-riscv-none-elf-gcc-14.2.0-3/bin/riscv-none-elf-gdb hello.elf
```
Then inside GDB:
```
(gdb) target remote :1234
(gdb) break main
(gdb) continue
```
Step 6: Step & Inspect
```
(gdb) stepi
(gdb) info registers
```
Step 7: Exit
In GDB: quit
In QEMU terminal: Ctrl + C

![image](https://github.com/user-attachments/assets/e07a8090-1603-4748-8895-d9ac40243f8c)
![image](https://github.com/user-attachments/assets/05b2da04-0e67-4897-b9a7-b3a523ea6cf9)
![image](https://github.com/user-attachments/assets/6bc22107-9a39-4d00-a887-7a437c1bb805)
![image](https://github.com/user-attachments/assets/03d37167-8a29-4906-96da-2dd4c86915fa)
![image](https://github.com/user-attachments/assets/0c4d3902-c2f0-4f20-9eb6-0fcccd20d80a)

### ✅ Task 7: 🧪 QEMU Simulation  
STEP 1: Verify Your Emulator & ELF
```
which qemu-system-riscv32
qemu-system-riscv32 --version

which spike
spike --help
```
Expected:

ELF is RISC-V 32-bit
qemu-system-riscv32 shows version like 8.0.4

STEP 2: Install QEMU & OpenSBI Firmware
```
sudo apt update
sudo apt install qemu-system-misc opensbi
```
STEP 3: Download OpenSBI Firmware
```
curl -LO https://github.com/qemu/qemu/raw/v8.0.4/pc-bios/opensbi-riscv32-generic-fw_dynamic.bin
ls -la opensbi-riscv32-generic-fw_dynamic.bin
```
✅ STEP 4: Create Bare-Metal ELF
`hello.c`

```
volatile char *uart = (char *)0x10000000;

void main() {
    const char *msg = "Hello, RISC-V!\n";
    while (*msg) {
        *uart = *msg++;
    }
    while (1);
}
```

`startup.s`

```
.section .text
.global _start

_start:
    la sp, stack_top     # Set up stack pointer
    call main            # Call main()
    j .                  # Infinite loop

.section .bss
.space 4096
stack_top:
```

`linker.ld`

```
ENTRY(_start)

MEMORY {
  RAM (rwx) : ORIGIN = 0x80200000, LENGTH = 128K
}

SECTIONS {
  .text : {
    *(.text*)
    *(.rodata*)
  } > RAM

  .data : {
    *(.data*)
  } > RAM

  .bss : {
    *(.bss*)
    *(COMMON)
  } > RAM
}
```
And now compile:
```
riscv-none-elf-gcc -g -march=rv32im -mabi=ilp32 -nostdlib -T linker.ld -o hello.elf hello.c startup.s
file hello.elf
```
STEP 5: Run With QEMU + OpenSBI
```
qemu-system-riscv32 \
  -nographic \
  -machine virt \
  -bios opensbi-riscv32-generic-fw_dynamic.bin \
  -kernel hello.elf
```

EXPECTED OUTPUT
```
OpenSBI v1.2 ...
Hello, RISC-V!
```
![image](https://github.com/user-attachments/assets/bddd958d-8bb6-4e3b-9770-365227fde402)
![image](https://github.com/user-attachments/assets/ecc97a58-f348-48b1-82c6-c40a5601c463)

### ✅ Task 8: ⚡ GCC Optimizations

STEP 1: Prepare a C File to Observe Optimization Effects
Create a file named opt_test.c with the following code:
```
int add(int a, int b) {
    int temp = 0;
    temp = a + b;
    return temp;
}

int main() {
    int result = add(10, 20);
    return 0;
}
```

STEP 2: Compile with -O0 and -O2
Generate assembly (.s) files:
```
riscv-none-elf-gcc -S -O0 -o opt_O0.s opt_test.c
riscv-none-elf-gcc -S -O2 -o opt_O2.s opt_test.c
```
Now you will have two assembly files to compare: `opt_O0.s` and `opt_O2.s`

![image](https://github.com/user-attachments/assets/5f7bae97-f889-402e-baa2-bc0e8e826622)

✅ -O0 (no optimization):
•	Every variable is kept
•	Uses memory and stack even for simple operations
•	square() is kept as a separate function
✅ -O2 (optimized):
•	Function square(5) gets inlined
•	result might directly store 25 (constant folding)
•	Code is tighter, uses fewer instructions

### ✅ Task 9: 🛠️ Inline Assembly 
Create files `hello2.c` , `cycle_counter` , `linker.ld` and `startup.s` and compile them using 
```
riscv-none-elf-gcc -g -O0 -march=rv32im -mabi=ilp32 -nostdlib -T linker.ld -o hello2.elf hello2.c startup.s
qemu-system-riscv32 -nographic -machine virt -bios none -kernel hello2.elf
```

![image](https://github.com/user-attachments/assets/0f5c3493-9cb1-47a3-a457-b8ea887cc124)
![image](https://github.com/user-attachments/assets/4c008425-f73d-4f94-bce8-0bb9bfd9893c)
![image](https://github.com/user-attachments/assets/941500f2-76e9-4ece-b0a5-ee4b76fd374c)
![image](https://github.com/user-attachments/assets/3eea0c9b-2e06-4cc8-af81-41d077f8aaf8)
![image](https://github.com/user-attachments/assets/bd65518e-dc69-4772-bd5a-cd33a74f5a97)

EXPECTED OUTCOME:

![image](https://github.com/user-attachments/assets/2fefbdde-3aaa-42f2-ae9c-d8b2aee2efca)

### ✅ Task 10: 💡 Memory-Mapped GPIO 
Step 1: Create Memory-Mapped I/O Program and compile it along with `linker.ld` and `startup.s`

![image](https://github.com/user-attachments/assets/9ba4f233-e6ed-4039-a443-dd2d7b160480)

```
riscv-none-elf-gcc -g -O2 -march=rv32im -mabi=ilp32 -nostdlib -T linker.ld -o gpio_toggle.elf gpio_toggle.c startup.s
riscv-none-elf-readelf -h gpio_toggle.elf
```
![image](https://github.com/user-attachments/assets/1ce8b302-75cd-4a90-87ba-8c23ba25eda4)

```
qemu-system-riscv32 -nographic -machine virt -bios none -kernel ~/Downloads/gpio_toggle.elf
```
EXPECTED OUTCOME : GPIO TOGGLED

### ✅ Task 11: 🗺️ Linker Script Basics

`Minimal.ld code`

```
/*

Minimal Linker Script for RV32IMC

Places .text at 0x00000000 (Flash/ROM)

Places .data at 0x10000000 (SRAM)
*/

ENTRY(_start)

MEMORY
{
    FLASH (rx) : ORIGIN = 0x00000000, LENGTH = 256K
    SRAM  (rwx): ORIGIN = 0x10000000, LENGTH = 64K
}

SECTIONS
{
    /* Text section in Flash at 0x00000000 */
    .text 0x00000000 : {
        (.text.start)    / Entry point first */
        (.text)         /* All other text */
        (.rodata)       /* Read-only data */
    } > FLASH

    /* Data section in SRAM at 0x10000000 */
    .data 0x10000000 : {
        _data_start = .;
        (.data)         /* Initialized data */
        _data_end = .;
    } > SRAM

    /* BSS section in SRAM */
    .bss : {
        _bss_start = .;
        (.bss)
        *(COMMON)
        _bss_end = .;
    } > SRAM

    /* Stack pointer symbol */
    _stack_top = ORIGIN(SRAM) + LENGTH(SRAM);
}
```

`test_linker.c code`

```
#include <stdint.h>

// Initialized global variable (.data section)
uint32_t global_var = 0x12345678;

// Uninitialized global variable (.bss section)
uint32_t bss_var;

// Function in .text section
void test_function(void) {
    global_var = 0xABCDEF00;
    bss_var = 0x11111111;
}

// Main function
void main(void) {
    test_function();
    while(1) {
        // Loop forever
    }
}
```

`start.s code`

```
.section .text.start
.global _start

_start:
    # Setup stack pointer
    lui sp, %hi(_stack_top)
    addi sp, sp, %lo(_stack_top)

    # Call main()
    call main

1:  j 1b   # Infinite loop

.size _start, . - _start
```

Compile assembly and C code with custom linker script

```
riscv32-unknown-elf-gcc -c start.s -o start.o
riscv32-unknown-elf-gcc -c test_linker.c -o test_linker.o
```
Link with custom linker script
```
riscv32-unknown-elf-ld -T minimal.ld start.o test_linker.o -o test_linker.elf
```

Create complete working build script

```
cat << 'EOF' > build_linker_test.sh
#!/bin/bash
echo "=== Task 11: Linker Script Implementation ==="

Compile everything
echo "1. Compiling with custom linker script..."
riscv32-unknown-elf-gcc -c start.s -o start.o
riscv32-unknown-elf-gcc -c test_linker.c -o test_linker.o
riscv32-unknown-elf-ld -T minimal.ld start.o test_linker.o -o test_linker.elf

echo "✓ Compilation successful!"

Verify results
echo -e "\n2. Verifying memory layout:"
echo "Text section should be at 0x00000000:"
riscv32-unknown-elf-objdump -h test_linker.elf | grep ".text"

echo "Data section should be at 0x10000000:"
riscv32-unknown-elf-objdump -h test_linker.elf | grep -E ".(s)?data"

echo -e "\n3. Symbol addresses:"
riscv32-unknown-elf-nm test_linker.elf | head -10

echo -e "\n✓ Linker script working correctly!"
EOF

chmod +x build_linker_test.sh
./build_linker_test.sh
```
![image](https://github.com/user-attachments/assets/59e11671-a867-407e-955d-2236f9a0f7af)

### ✅ Task 12: 🧬 crt0 & Startup Code 

## 🏁 What is `crt0.S` and What Does It Do in a Bare-Metal RISC-V Program?

In a **bare-metal RISC-V program** (i.e., without an operating system), `crt0.S` is the **startup assembly file** that prepares the environment **before `main()` is called**.

---

### 📋 Responsibilities of `crt0.S`

Here’s what a typical `crt0.S` does:

1. 🧱 **Set Up the Stack Pointer**
   - Initializes the `sp` (stack pointer) register to a valid memory location, usually at the top of RAM.
   - This is essential before any C code runs, since functions and local variables rely on the stack.

2. 🧼 **Zero Out the `.bss` Section**
   - `.bss` contains uninitialized global/static variables.
   - `crt0.S` loops through this region and sets all values to zero, as per the C standard.

3. 📦 **Copy `.data` Section to RAM (optional)**
   - If variables have initialized values (stored in Flash), it copies them into RAM for read-write access.

4. ▶️ **Call the `main()` Function**
   - Once memory is initialized and the CPU is ready, control is transferred to the user-defined `main()` function.

5. 🔁 **Handle Exit / Infinite Loop**
   - If `main()` ever returns, `crt0.S` usually puts the CPU into a harmless infinite loop (`wfi` or `j .`), since there's no OS to return to.

---


### 🛠️ Minimal Example `crt0.S`

```assembly
.section .init
.globl _start

_start:
  la sp, _stack_top       # Initialize stack pointer
  call main               # Call main()
  j .                     # Infinite loop if main returns
```

### ✅ Task 13: ⏱️ Timer Interrupts 

Set `mtimecmp`
Enable `mie.MTIE`
Write __attribute__((interrupt)) handler in C or ASM

![image](https://github.com/user-attachments/assets/2219d3d2-7cb2-4cd2-8993-5241fe5c383b)
![image](https://github.com/user-attachments/assets/7dca40b8-2430-46aa-8366-c227485c7a86)

![image](https://github.com/user-attachments/assets/71f53c01-8514-4d01-81ab-539c6aff8ca2)

PARTIAL OUTPUT USING GDB:
![image](https://github.com/user-attachments/assets/7741cf58-d20b-4c9c-8261-a5eb1a1d2f23)

### ✅ Task 14: 🔐 Atomic Extension ("A") 

## ⚙️ The 'A' Extension in RISC-V (Atomic Instructions)

The `'A'` extension stands for **Atomic operations**, and when added to `RV32IMC`, it forms `RV32IMAC**A**`, enabling support for **atomic read-modify-write instructions** directly in hardware.

These instructions are crucial in **multi-threaded**, **multi-core**, or **interrupt-driven** environments for **safe and efficient synchronization**.

---

### 📌 Why Is the 'A' Extension Useful?

- ✅ Enables **lock-free data structures** like queues, stacks, and ring buffers.
- ✅ Essential for building **OS kernels**, semaphores, mutexes, and spinlocks.
- ✅ Prevents **race conditions** during concurrent access to shared memory.

---

### 📋 Key Instructions Introduced by the 'A' Extension

| Instruction  | Meaning                         | Use Case                          |
|--------------|----------------------------------|------------------------------------|
| `lr.w`       | Load Reserved (word)             | Begin an atomic operation         |
| `sc.w`       | Store Conditional (word)         | End an atomic operation           |
| `amoadd.w`   | Atomic Add (word)                | Atomic increment or add           |
| `amoswap.w`  | Atomic Swap (word)               | Swap values atomically            |
| `amoxor.w`   | Atomic XOR (word)                | Atomic bitwise XOR                |
| `amoand.w`   | Atomic AND (word)                | Atomic bitwise AND                |
| `amoor.w`    | Atomic OR (word)                 | Atomic bitwise OR                 |
| `amomin.w`   | Atomic Minimum (signed word)     | Atomically compute minimum        |
| `amomax.w`   | Atomic Maximum (signed word)     | Atomically compute maximum        |
| `amominu.w`  | Atomic Minimum (unsigned word)   | Unsigned atomic min               |
| `amomaxu.w`  | Atomic Maximum (unsigned word)   | Unsigned atomic max               |

---

### 🔁 How `lr.w` and `sc.w` Work Together

These two form a **load-link/store-conditional (LL/SC)** pair, commonly used to implement **atomic compare-and-swap (CAS)**.

#### Example: Atomic Counter Increment

```assembly
atomic_inc:
    again:
        lr.w t0, (a0)       # Load current value
        addi t0, t0, 1      # Increment it
        sc.w t1, t0, (a0)   # Try to store it
        bnez t1, again      # If store fails, retry
    ret
```

### ✅ Task 15: 🤝 Mutex with Atomics 
Write a two-thread style pseudo test that uses lr.w / sc.w to build an atomic lock in RV32IMAC (where A = Atomic extension).
________________________________________
🧠 What are lr.w and sc.w?
Instruction	Meaning
lr.w (Load-Reserved)	Reads a word from memory and marks it for exclusive access
sc.w (Store-Conditional)	Writes to that memory only if no one else modified it
They are used for building locks without hardware mutex or disabling interrupts — perfect for multicore sync or thread-safe code.

![image](https://github.com/user-attachments/assets/b1c14a14-f99c-4079-ba6f-e89ec54c2577)
![image](https://github.com/user-attachments/assets/1571818e-c74c-4fa2-a234-9b5d44ea75b5)

![image](https://github.com/user-attachments/assets/85259476-db95-4eac-bc85-d3fd1fc05539)
![image](https://github.com/user-attachments/assets/985148f3-4a35-482f-9c1a-71eeb40e9a0f)
![image](https://github.com/user-attachments/assets/ee897e83-c280-4462-a05e-a5a1d958de5d)

### ✅ Task 16: 📤 Retarget printf to UART 

Create `_write()`:

```
int _write(int fd, char *buf, int len) {
    volatile char *uart = (char *)0x10000000;
    for (int i = 0; i < len; i++)
        uart[i] = buf[i];
    return len;
}
```
Compile:
```
riscv-none-elf-gcc -nostartfiles -T linker.ld -o print.elf startup.o uart_printf.o
```
### ✅ Task 17: 🔁 Endianness & Struct Packing
Verify that RV32 is little-endian by default using the union trick in C. Demonstrate byte ordering verification by storing a 32-bit value and examining individual bytes.
![image](https://github.com/user-attachments/assets/69660b18-d231-4dbb-8840-334c3a119a3d)
![image](https://github.com/user-attachments/assets/654a784d-0b40-40b8-8f9f-a840ef7a4245)

and then use qemu:
```
qemu-system-riscv32 -nographic -machine virt -bios none -kernel output.elf
```

![image](https://github.com/user-attachments/assets/c9d30eaa-9aa4-4984-9f1d-96794cb95a9a)




