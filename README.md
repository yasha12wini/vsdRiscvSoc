# RISC-V Development Guide - Complete Tutorial

## Prerequisites and Setup

### Install RISC-V Toolchain
```bash
# Download and install RISC-V GNU toolchain
sudo apt-get update
sudo apt-get install gcc-riscv64-linux-gnu gcc-riscv64-unknown-elf
# Or for 32-bit specifically:
sudo apt-get install gcc-riscv32-unknown-elf gcc-riscv32-unknown-linux-gnu

# Install QEMU for RISC-V emulation
sudo apt-get install qemu-system-riscv32 qemu-system-riscv64 qemu-user

# Install additional dependencies
sudo apt-get install libboost-regex1.74.0 libboost-system1.74.0 libboost-filesystem1.74.0
```

### Set up PATH
```bash
# Add toolchain to PATH
export PATH=$HOME/riscv_custom_tools/bin:$PATH
export PATH=~/Downloads/riscv/bin:$PATH
echo 'export PATH=$HOME/riscv_custom_tools/bin:$PATH' >> ~/.bashrc
```

---

## Task 1: 
Absolutely! Here's a clean, copy-paste-ready set of commands to **unpack**, **set up**, and **verify** the RISC-V RV32IMAC toolchain for your GitHub README:

---

### 🚀 Install RISC-V Toolchain (RV32IMAC)

```bash
# 1. Extract the prebuilt toolchain archive
tar -xzf riscv-toolchain-rv32imac-x86_64-ubuntu.tar.gz -C $HOME

# 2. Add the toolchain to your PATH temporarily
export PATH=$HOME/riscv/bin:$PATH

# 3. (Optional) Add to ~/.bashrc to make it permanent
echo 'export PATH=$HOME/riscv/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# 4. Verify installation
riscv32-unknown-elf-gcc --version
riscv32-unknown-elf-objdump --version
riscv32-unknown-elf-gdb --version
```

---

✅ Make sure the extracted directory is actually `$HOME/riscv`. If it's named differently (e.g., `riscv-toolchain`), update the `PATH` accordingly:

```bash
export PATH=$HOME/<your-toolchain-folder>/bin:$PATH
```

Let me know if you'd like a similar block for running on Spike or compiling C programs for RV32.


---

## Task 2: Basic RISC-V Compilation

### Create Hello World Program
```bash
nano hello.c
```

**hello.c content:**
```c
#include <stdio.h>

int main() {
    printf("Hello, RISC-V!\n");
    return 0;
}
```

### Compile for RISC-V
```bash
# Basic compilation
riscv32-unknown-elf-gcc -o hello.elf hello.c

# Check file type
file hello.elf
```

**Expected output:**
```
hello.elf: ELF 32-bit LSB executable, UCB RISC-V, RVC, soft-float ABI, version 1 (SYSV), statically linked, not stripped
```

### Compilation with specific architecture (if needed)
```bash
# This might fail with multilib error
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -o hello.elf hello.c
# Error: Cannot find suitable multilib set for '-march=rv32imc'/'-mabi=ilp32'
```

---

## Task 3: Generate Assembly Code

### Generate Assembly from C
```bash
# Generate assembly file
riscv32-unknown-elf-gcc -S hello.c -o hello.s

# View the assembly
cat hello.s
```

**hello.s content:**
```assembly
.file "hello.c"
.option nopic
.attribute arch, "rv32i2p1_m2p0_a2p1_c2p0"
.attribute unaligned_access, 0
.attribute stack_align, 16
.text
.section .rodata
.align 2
.LC0:
.string "Hello, RISC-V!"
.text
.align 1
.globl main
.type main, @function
main:
addi sp,sp,-16
sw ra,12(sp)
sw s0,8(sp)
addi s0,sp,16
lui a5,%hi(.LC0)
addi a0,a5,%lo(.LC0)
call puts
li a5,0
mv a0,a5
lw ra,12(sp)
lw s0,8(sp)
addi sp,sp,16
jr ra
.size main, .-main
.ident "GCC: (g04696df096) 14.2.0"
.section .note.GNU-stack,"",@progbits
```

---

## Task 4: Binary Analysis and Disassembly

### Generate Binary and Hex Files
```bash
# Disassemble ELF file
riscv32-unknown-elf-objdump -d hello.elf > hello.asm

# Create binary file
riscv32-unknown-elf-objcopy -O binary hello.elf hello.bin

# View binary as hex
xxd hello.bin > hello.hex
hexdump -C hello.bin

# Disassemble binary (correct command)
riscv32-unknown-elf-objdump -D -b binary -m riscv:rv32 hello.bin > hello.disasm
cat hello.disasm
```

### Common Errors and Solutions
```bash
# Wrong command (will fail):
riscv32-elf-unknown-objdump -D -b binary -m riscv32 hello.bin > hello.disasm
# Error: command not found

# Wrong architecture (will fail):
riscv32-unknown-elf-objdump -D -b binary -m riscv32 hello.bin > hello.disasm
# Error: can't use supplied machine riscv32
```

**Note:** hello.asm and hello.disasm files are attached separately due to length.

---

## Task 5: RISC-V Register Reference

### RV32 Integer Register Table (x0 to x31)

| Register | ABI Name | Role / Purpose |
|----------|----------|----------------|
| x0 | zero | Constant zero (always 0) |
| x1 | ra | Return address |
| x2 | sp | Stack pointer |
| x3 | gp | Global pointer |
| x4 | tp | Thread pointer |
| x5 | t0 | Temporary / caller-saved |
| x6 | t1 | Temporary / caller-saved |
| x7 | t2 | Temporary / caller-saved |
| x8 | s0/fp | Saved register / frame pointer |
| x9 | s1 | Saved register / callee-saved |
| x10 | a0 | Argument 0 / return value 0 |
| x11 | a1 | Argument 1 / return value 1 |
| x12 | a2 | Argument 2 |
| x13 | a3 | Argument 3 |
| x14 | a4 | Argument 4 |
| x15 | a5 | Argument 5 |
| x16 | a6 | Argument 6 |
| x17 | a7 | Argument 7 |
| x18 | s2 | Saved register / callee-saved |
| x19 | s3 | Saved register / callee-saved |
| x20 | s4 | Saved register / callee-saved |
| x21 | s5 | Saved register / callee-saved |
| x22 | s6 | Saved register / callee-saved |
| x23 | s7 | Saved register / callee-saved |
| x24 | s8 | Saved register / callee-saved |
| x25 | s9 | Saved register / callee-saved |
| x26 | s10 | Saved register / callee-saved |
| x27 | s11 | Saved register / callee-saved |
| x28 | t3 | Temporary / caller-saved |
| x29 | t4 | Temporary / caller-saved |
| x30 | t5 | Temporary / caller-saved |
| x31 | t6 | Temporary / caller-saved |

### Calling Convention Summary

- **a0–a7 (x10–x17):** Used to pass arguments to functions and receive return values (first 8 arguments)
- **s0–s11 (x8, x9, x18–x27):** Callee-saved registers. A function must restore their original values before returning
- **t0–t6 (x5–x7, x28–x31):** Caller-saved registers. Can be freely used by the callee; the caller must save them if needed
- **sp (x2):** Stack pointer – points to the top of the stack
- **ra (x1):** Return address – used to return from function calls
- **fp (x8 / s0):** Frame pointer – used when accessing local stack frames
- **gp (x3):** Global pointer – used in position-independent code for accessing global variables
- **tp (x4):** Thread pointer – for thread-local storage

### Example Code Snippet (Understanding ABI Roles)

```c
// C code
int add(int a, int b) {
    return a + b;
}
```

```assembly
# Corresponding RISC-V Assembly
add:
    add a0, a0, a1   # a0 = a0 + a1 (result in a0)
    ret              # return to address in ra
```

- `a0, a1` contain the input arguments
- The result is returned in `a0`
- `ret` jumps to the address in `ra`

---

## Task 6: Debugging with GDB and QEMU

### Run Program with QEMU
```bash
# Simple execution
qemu-riscv32 hello.elf
# Output: Hello, RISC-V!
```

### Debug with GDB
```bash
# Terminal 1: Start QEMU with GDB server
qemu-riscv32 -g 1234 hello.elf

# Terminal 2: Connect GDB
riscv32-unknown-elf-gdb hello.elf

# GDB commands:
target remote localhost:1234   # connect to QEMU's GDB server
break main                    # set breakpoint
continue                     # run until breakpoint
step                         # step through program
stepi                        # step single instruction
info reg a0                  # inspect register a0
disassemble                  # see assembly
```

---

## Task 6 (Extended): Build RISC-V Proxy Kernel (pk)

### Install and Build pk
```bash
# Make and install tools
make install
export PATH=$HOME/riscv_custom_tools/bin:$PATH

# Clone and build proxy kernel
git clone https://github.com/riscv-software-src/riscv-pk.git
cd riscv-pk
mkdir build-rv32imac-xpack
cd build-rv32imac-xpack

# Configure and build
../configure --prefix=$HOME/riscv_custom_tools --host=riscv32-unknown-elf --with-arch=rv32imac_zicsr_zifencei --with-abi=ilp32 CC=riscv32-unknown-elf-gcc

make -j$(nproc)
```

### Create UART Console Program
```bash
cat > hello_uart.c << 'EOF'
#include <stdio.h>

int main() {
    printf("Hello from RISC-V UART console!\n");
    printf("This is bare-metal ELF running on Spike.\n");
    return 0;
}
EOF
```

### Create Linker Script
```bash
cat > bare_metal.ld << 'EOF'
ENTRY(_start)

MEMORY {
    RAM : ORIGIN = 0x80000000, LENGTH = 128M
}

SECTIONS {
    .text : {
        *(.text*)
    } > RAM
   
    .data : {
        *(.data*)
    } > RAM
   
    .bss : {
        *(.bss*)
    } > RAM
}
EOF
```

### Run with Spike Simulator
```bash
# Compile UART program
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -o hello_uart.elf hello_uart.c

# Run with Spike
spike --isa=rv32imac /home/yasha/riscv_custom_tools/riscv32-unknown-elf/bin/pk ./hello_uart.elf
```

---

## Task 7: Bare-Metal Assembly Programming

### Assemble and Link Bare-Metal Code
```bash
# Assemble startup code
riscv32-unknown-elf-as -march=rv32imac -mabi=ilp32 -o bare_code.o start.S

# Compile and link bare-metal program
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -nostdlib -T linker.ld -o bare_code.elf bare_code.o bare_code.c
```

**Note:** start.S, linker.ld, and bare_code.c files to be attached.

---

## Task 8: [Placeholder]
*Content for Task 8*

---

## Task 9: Cycle Counter and Performance Testing

### Create Enhanced Cycle Test
```bash
nano enhanced_cycle_test.c
```

**enhanced_cycle_test.c:**
```c
#include <stdio.h>
#include <stdint.h>

static inline uint32_t rdcycle(void) {
    uint32_t c;
    asm volatile ("csrr %0, cycle" : "=r"(c));
    return c;
}

int main(void) {
    uint32_t start, end;
   
    printf("=== RISC-V Cycle Counter Test ===\n");
   
    start = rdcycle();
    printf("Start cycles: %u\n", start);
   
    // Do some work
    volatile int sum = 0;
    for (volatile int i = 0; i < 1000; i++) {
        sum += i;
    }
   
    end = rdcycle();
    printf("End cycles: %u\n", end);
    printf("Elapsed cycles: %u\n", end - start);
    printf("Sum result: %d\n", sum);
    
    return 0;
}
```

### Compile and Run
```bash
# Compile with CSR and counter extensions
riscv32-unknown-elf-gcc -o enhanced_cycle_test.elf enhanced_cycle_test.c -march=rv32imac_zicsr_zicntr -mabi=ilp32

# Run with Spike
spike --isa=rv32imac_zicntr $HOME/riscv_custom_tools/riscv32-unknown-elf/bin/pk enhanced_cycle_test.elf
```

---

## Task 10: GPIO Simulation

### Create GPIO Demo
```bash
nano gpio_demo.c
```

### Compile and Run GPIO Demo
```bash
# Compile GPIO demo
riscv32-unknown-elf-gcc -o gpio_demo.elf gpio_demo.c -march=rv32imac_zicsr_zicntr -mabi=ilp32

# Run with Spike
spike --isa=rv32imac_zicntr $HOME/riscv_custom_tools/riscv32-unknown-elf/bin/pk gpio_demo.elf
```

**Note:** gpio_demo.c code to be attached.

---

## Task 11: Memory Layout and Linker Scripts

### Create Test Program
```bash
cat > test.c << 'EOF'
int global_data = 42;
int uninitialized_data;

void _start(void) {
    global_data = 100;
    while(1);
}
EOF
```

### Create Custom Linker Script
```bash
nano linker.ld
```

**linker.ld:**
```ld
/* Minimal linker script for RV32IMAC */
ENTRY(_start)

SECTIONS
{
    .text 0x0 : {
        *(.text*)
        *(.rodata*)
    }
   
    .data 0x10000000 : {
        *(.data*)
        *(.sdata*)
    }
   
    .bss 0x10000004 : {
        *(.bss*)
        *(.sbss*)
    }
}
```

### Compile and Analyze
```bash
# Compile object file
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -c test.c -o test.o

# Link with custom script
riscv32-unknown-elf-ld -T linker.ld test.o -o test.elf

# Analyze sections
riscv32-unknown-elf-objdump -h test.elf
```

### Memory Layout Explanation

The address separation between `.text` (0x00000000) and `.data` (0x10000000) reflects different memory types in embedded systems:

**Flash Memory (0x00000000 region):**
- Non-volatile storage for program code and constants
- Read-only during normal execution
- Slower access compared to SRAM
- Retains data when power is removed
- Used for `.text` (code) and `.rodata` (read-only data)

**SRAM (0x10000000 region):**
- Volatile memory for runtime data
- Read-write access required
- Much faster access than Flash
- Loses data when power is removed
- Used for `.data` (initialized variables) and `.bss` (uninitialized variables)

The large address gap (256MB) ensures clear memory region separation, preventing accidental overlap and allowing memory management units to apply different access permissions and caching policies to each region.

---

## Task 12: C Runtime Startup (crt0.S)

### What crt0.S Does in Bare-Metal RISC-V

The `crt0.S` (C runtime startup) file is the first code that runs after reset in a bare-metal RISC-V program. It performs essential system initialization before your `main()` function can execute.

#### Typical crt0.S Tasks:

1. **Stack Pointer Setup**
   - Initializes the stack pointer (sp) to the top of RAM
   - Essential for function calls and local variables

2. **BSS Section Zeroing**
   - Clears uninitialized global/static variables to zero
   - Required by C standard for proper variable initialization

3. **Data Section Initialization**
   - Copies initialized data from Flash to SRAM
   - Ensures global variables have correct initial values

4. **Main Function Call**
   - Calls your `main()` function
   - May set up argc/argv for command-line arguments (usually NULL in bare-metal)

5. **Post-main Handling**  
   - Enters infinite loop or reset after `main()` returns
   - Prevents processor from executing random memory

### Where to Get crt0.S:

**Device/Board Specific Sources:**
- SiFive Freedom SDK - Provides crt0.S for SiFive boards
- Nuclei SDK - For Nuclei RISC-V processors
- RT-Thread - Includes RISC-V startup files
- Vendor BSPs - Check your specific RISC-V chip manufacturer

**Generic Sources:**
- Newlib - GNU C library includes basic RISC-V crt0
- RISC-V GNU Toolchain - Contains reference implementations
- OpenOCD examples - Often includes startup code templates

**Example repositories:**
- `riscv-gnu-toolchain/riscv-newlib/libgloss/riscv/`
- `SiFive's freedom-e-sdk/bsp/env/`
- Various GitHub projects with "riscv-crt0" or "riscv-startup"

### Create Complete Startup System

```bash
# Create startup assembly
nano crt0.S
```

**crt0.S:**
```assembly
.section .text
.global _start

_start:
    # 1. Set up stack pointer
    lui sp, %hi(0x10002000)
    addi sp, sp, %lo(0x10002000)
   
    # 2. Zero out .bss section
    la t0, __bss_start
    la t1, __bss_end
   
zero_bss:
    beq t0, t1, bss_done
    sw zero, 0(t0)
    addi t0, t0, 4
    j zero_bss
   
bss_done:
    # 3. Copy .data section from Flash to RAM
    la t0, __data_start
    la t1, __data_end
    la t2, __data_load_start
   
copy_data:
    beq t0, t1, data_done
    lw t3, 0(t2)
    sw t3, 0(t0)
    addi t0, t0, 4
    addi t2, t2, 4
    j copy_data
   
data_done:
    # 4. Call main function
    call main
   
    # 5. Handle main return - infinite loop
_exit:
    j _exit
```

### Create Test Main Program
```bash
nano main1.c
```

**main1.c:**
```c
int global_var = 42;        // Goes to .data
int uninit_var;             // Goes to .bss

int main(void) {
    global_var = 100;
    uninit_var = 200;
    return 0;
}
```

### Complete Linker Script
```bash
nano complete_linker.ld
```

**Note:** complete_linker.ld code to be attached.

### Build Complete System
```bash
# Compile startup code
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -c crt0.S -o crt0.o

# Compile main program
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -c main1.c -o main1.o

# Link everything
riscv32-unknown-elf-ld -T complete_linker.ld crt0.o main1.o -o complete.elf

# Analyze the result
echo "=== SECTION HEADERS ==="
riscv32-unknown-elf-objdump -h complete.elf

echo -e "\n=== SYMBOL TABLE ==="
riscv32-unknown-elf-objdump -t complete.elf | grep -E "(bss|data|start|main)"

echo -e "\n=== CRT0 STARTUP CODE ==="
riscv32-unknown-elf-objdump -d complete.elf

echo -e "\n=== MEMORY SECTIONS ==="
riscv32-unknown-elf-readelf -S complete.elf
```

---

## Task 13: Interrupt Handling and Timers

### Create Timer Demo with Interrupts
```bash
nano timer_demo.c
nano crt0_interrupt.S
nano interrupt_linker.ld
```

### Build Interrupt System
```bash
# Compile startup code with zicsr extension
riscv32-unknown-elf-gcc -march=rv32imac_zicsr -mabi=ilp32 -c crt0_interrupt.S -o crt0_interrupt.o

# Compile timer demo with zicsr extension
riscv32-unknown-elf-gcc -march=rv32imac_zicsr -mabi=ilp32 -O2 -c timer_demo.c -o timer_demo.o

# Link everything
riscv32-unknown-elf-ld -T interrupt_linker.ld crt0_interrupt.o timer_demo.o -o timer_interrupt.elf

# Analyze interrupt system
echo "=== RISC-V TIMER INTERRUPT DEMO ===" && \
riscv32-unknown-elf-objdump -h timer_interrupt.elf && \
echo -e "\n=== INTERRUPT FUNCTIONS ===" && \
riscv32-unknown-elf-objdump -t timer_interrupt.elf | grep -E "(interrupt|timer|trap)" && \
echo -e "\n=== CSR INSTRUCTIONS ===" && \
riscv32-unknown-elf-objdump -d timer_interrupt.elf | grep -E "(csrr|csrw|csrs)" | head -10
```

**Note:** timer_demo.c, crt0_interrupt.S, and interrupt_linker.ld codes to be attached.

---

## Task 14: Atomic Operations and Load-Reserved/Store-Conditional

### Navigate to Home Directory
```bash
cd ~
```

### Create Atomic Assembly Test
```bash
nano atomic_test.s
```

### Create Load-Reserved/Store-Conditional Test
```bash
nano lr_sc_test.s
```

### Create Atomic Counter in C
```bash
nano atomic_counter.c
```

### Build and Run Atomic Tests
```bash
# Assemble atomic test
riscv32-unknown-linux-gnu-as -o atomic_test.o atomic_test.s
riscv32-unknown-linux-gnu-ld -o atomic_test atomic_test.o

# Compile atomic counter (with threading)
riscv32-unknown-linux-gnu-gcc -pthread -o atomic_counter atomic_counter.c

# For static linking (better compatibility with QEMU)
riscv32-unknown-linux-gnu-gcc -static -pthread -o atomic_counter atomic_counter.c

# Run with QEMU
qemu-riscv32 ./atomic_counter
```

**Note:** atomic_test.s, lr_sc_test.s, and atomic_counter.c codes to be attached.

---

## Task 15: Mutex Implementation with Atomic Operations

### Create Basic Mutex Test
```bash
nano rv32_mutex.c

# Compile and run
riscv32-unknown-linux-gnu-gcc -static -o rv32_mutex rv32_mutex.c
qemu-riscv32 ./rv32_mutex
```

### Create Concurrent Mutex Test
```bash
nano rv32_mutex_concurrent.c

# Compile with threading support
riscv32-unknown-linux-gnu-gcc -static -pthread -o rv32_mutex_concurrent rv32_mutex_concurrent.c
qemu-riscv32 ./rv32_mutex_concurrent
```

### Analyze Atomic Instructions
```bash
# View disassembly of atomic operations
riscv32-unknown-linux-gnu-objdump -d rv32_mutex_concurrent | grep -A10 -B5 "lr.w\|sc.w"
```

### Test Multiple Runs
```bash
# Run multiple times to test consistency
for i in {1..3}; do
    echo "=== Run $i ==="
    qemu-riscv32 ./rv32_mutex_concurrent
    echo ""
done
```

**Note:** rv32_mutex.c and rv32_mutex_concurrent.c codes to be attached.

---

## Task 16: [To be completed]
*Placeholder for Task 16 content*

---

## Task 17: Endianness Testing

### Create Endianness Test
```bash
cd ~
nano endian_test.c
```

### Compile and Run
```bash
# Compile with optimization
riscv32-unknown-linux-gnu-gcc -march=rv32imac -mabi=ilp32 -O2 -o endian_test endian_test.c

# Static linking for better QEMU compatibility
riscv32-unknown-linux-gnu-gcc -march=rv32imac -mabi=ilp32 -static -o endian_test endian_test.c

# Run with QEMU
qemu-riscv32 endian_test
```

**Note:** endian_test.c code to be attached.

---

## Useful Commands Reference

### File Finding
```bash
# Find specific files
find ~ -name hello.c
find ~ -name "*.elf"
find ~ -name "*.s"
```

### ELF Analysis Commands
```bash
# View section headers
riscv32-unknown-elf-objdump -h file.elf

# View symbol table
riscv32-unknown-elf-objdump -t file.elf

# Disassemble code sections
riscv32-unknown-elf-objdump -d file.elf

# View all sections (including data)
riscv32-unknown-elf-objdump -D file.elf

# View ELF header information
riscv32-unknown-elf-readelf -h file.elf

# View section information
riscv32-unknown-elf-readelf -S file.elf
```

### QEMU Commands
```bash
# Run 32-bit RISC-V binary
qemu-riscv32 ./program

# Run with GDB debugging
qemu-riscv32 -g 1234 ./program

# Run system emulation
qemu-system-riscv32 -M virt -kernel kernel.elf
```

### GDB Commands for RISC-V
```gdb
# Connect to QEMU
target remote localhost:1234

# Set breakpoints
break main
break *_start

# View registers
info registers
info reg a0 a1 a2

# Step execution
step          # Source level
stepi         # Instruction level
continue      # Continue execution

# View memory
x/10i $pc     # 10 instructions at PC
x/16xw 0x1000 # 16 words at address 0x1000

# Disassemble
disassemble main
disassemble _start
```

## Troubleshooting Common Issues

### Multilib Errors
If you get multilib errors, try:
```bash
# Use default architecture
riscv32-unknown-elf-gcc -o program program.c

# Or specify compatible options
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -o program program.c
```

### Missing Libraries
```bash
# Install missing boost libraries
sudo apt-get install libboost-all-dev

# Install additional RISC-V tools
sudo apt-get install gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu
```

### QEMU Issues
```bash
# Make sure binary is statically linked for user mode
riscv32-unknown-linux-gnu-gcc -static -o program program.c

# Check if QEMU supports the architecture
qemu-riscv32 -cpu help
```

---

## Summary

This guide covers comprehensive RISC-V development from basic compilation to advanced topics like atomic operations, interrupt handling, and bare-metal programming. Each task builds upon previous knowledge, providing hands-on experience with:

- Cross-compilation for RISC-V
- Assembly language programming
- Memory layout and linker scripts
- Debugging with GDB and QEMU
- Bare-metal programming techniques
- Atomic operations and synchronization
- Performance analysis with cycle counters

The combination of theoretical knowledge and practical implementation provides a solid foundation for RISC-V embedded systems development.

---

## Files to be Attached

The following code files are referenced in this guide and should be attached separately:

1. **hello.asm** - Disassembled hello world (Task 4)
2. **hello.disasm** - Binary disassembly output (Task 4)
3. **gpio_demo.c** - GPIO simulation code (Task 10)
4. **complete_linker.ld** - Complete linker script (Task 12)
5. **timer_demo.c** - Timer interrupt demo (Task 13)
6. **crt0_interrupt.S** - Interrupt startup code (Task 13)
7. **interrupt_linker.ld** - Interrupt linker script (Task 13)
8. **atomic_test.s** - Atomic assembly test (Task 14)
9. **lr_sc_test.s** - Load-reserved/store-conditional test (Task 14)
10. **atomic_counter.c** - Atomic counter implementation (Task 14)
11. **rv32_mutex.c** - Basic mutex implementation (Task 15)
12. **rv32_mutex_concurrent.c** - Concurrent mutex test (Task 15)
13. **endian_test.c** - Endianness testing code (Task 17)
