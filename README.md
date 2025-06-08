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




---
## Output Screenshot

![0](https://github.com/user-attachments/assets/0d5e79dc-c389-443f-a568-9e2fe28e66fe)

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
## Output Screenshot
![0](https://github.com/user-attachments/assets/8bdb3ca3-ad3d-4b86-848c-5c8b198e3af0)



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
## Output Screenshot

!![0](https://github.com/user-attachments/assets/9eb820a6-24e9-408b-aefc-4681c1ea1a0c)
[0](https://github.com/user-attachments/assets/b7a47f7e-6fdd-4bd7-8dd8-bf33e8512c3f)

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
**hello.disasm content:**
```assembly

hello.bin:     file format binary


Disassembly of section .data:

00000000 <.data>:
       0:	1141                	addi	sp,sp,-16
       2:	4581                	li	a1,0
       4:	c422                	sw	s0,8(sp)
       6:	c606                	sw	ra,12(sp)
       8:	842a                	mv	s0,a0
       a:	7d0000ef          	jal	0x7da
       e:	d481a783          	lw	a5,-696(gp)
      12:	c391                	beqz	a5,0x16
      14:	9782                	jalr	a5
      16:	8522                	mv	a0,s0
      18:	1ca020ef          	jal	0x21e2
      1c:	00000793          	li	a5,0
      20:	c791                	beqz	a5,0x2c
      22:	6549                	lui	a0,0x12
      24:	a0e50513          	addi	a0,a0,-1522 # 0x11a0e
      28:	0810006f          	j	0x8a8
      2c:	8082                	ret
      2e:	00004197          	auipc	gp,0x4
      32:	b9e18193          	addi	gp,gp,-1122 # 0x3bcc
      36:	d4818513          	addi	a0,gp,-696
      3a:	07018613          	addi	a2,gp,112
      3e:	8e09                	sub	a2,a2,a0
      40:	4581                	li	a1,0
      42:	2571                	jal	0x6ce
      44:	00001517          	auipc	a0,0x1
      48:	86450513          	addi	a0,a0,-1948 # 0x8a8
      4c:	c519                	beqz	a0,0x5a
      4e:	00002517          	auipc	a0,0x2
      52:	90c50513          	addi	a0,a0,-1780 # 0x195a
      56:	053000ef          	jal	0x8a8
      5a:	2529                	jal	0x664
      5c:	4502                	lw	a0,0(sp)
      5e:	004c                	addi	a1,sp,4
      60:	4601                	li	a2,0
      62:	20b1                	jal	0xae
      64:	bf71                	j	0x0
      66:	1141                	addi	sp,sp,-16
      68:	c422                	sw	s0,8(sp)
      6a:	d641c783          	lbu	a5,-668(gp)
      6e:	c606                	sw	ra,12(sp)
      70:	ef91                	bnez	a5,0x8c
      72:	00000793          	li	a5,0
      76:	cb81                	beqz	a5,0x86
      78:	6549                	lui	a0,0x12
      7a:	47050513          	addi	a0,a0,1136 # 0x12470
      7e:	00000097          	auipc	ra,0x0
      82:	000000e7          	jalr	zero # 0x0
      86:	4785                	li	a5,1
      88:	d6f18223          	sb	a5,-668(gp)
      8c:	40b2                	lw	ra,12(sp)
      8e:	4422                	lw	s0,8(sp)
      90:	0141                	addi	sp,sp,16
      92:	8082                	ret
      94:	00000793          	li	a5,0
      98:	cb91                	beqz	a5,0xac
      9a:	6549                	lui	a0,0x12
      9c:	d6818593          	addi	a1,gp,-664
      a0:	47050513          	addi	a0,a0,1136 # 0x12470
      a4:	00000317          	auipc	t1,0x0
      a8:	00000067          	jr	zero # 0x0
      ac:	8082                	ret
      ae:	1141                	addi	sp,sp,-16
      b0:	c606                	sw	ra,12(sp)
      b2:	c422                	sw	s0,8(sp)
      b4:	0800                	addi	s0,sp,16
      b6:	67c9                	lui	a5,0x12
      b8:	45c78513          	addi	a0,a5,1116 # 0x1245c
      bc:	26bd                	jal	0x42a
      be:	4781                	li	a5,0
      c0:	853e                	mv	a0,a5
      c2:	40b2                	lw	ra,12(sp)
      c4:	4422                	lw	s0,8(sp)
      c6:	0141                	addi	sp,sp,16
      c8:	8082                	ret
      ca:	4501                	li	a0,0
      cc:	8082                	ret
      ce:	664d                	lui	a2,0x13
      d0:	65c5                	lui	a1,0x11
      d2:	654d                	lui	a0,0x13
      d4:	48060613          	addi	a2,a2,1152 # 0x13480
      d8:	21e58593          	addi	a1,a1,542 # 0x1121e
      dc:	49050513          	addi	a0,a0,1168 # 0x13490
      e0:	aca9                	j	0x33a
      e2:	414c                	lw	a1,4(a0)
      e4:	1141                	addi	sp,sp,-16
      e6:	c422                	sw	s0,8(sp)
      e8:	c606                	sw	ra,12(sp)
      ea:	d8018793          	addi	a5,gp,-640
      ee:	842a                	mv	s0,a0
      f0:	00f58463          	beq	a1,a5,0xf8
      f4:	076010ef          	jal	0x116a
      f8:	440c                	lw	a1,8(s0)
      fa:	de818793          	addi	a5,gp,-536
      fe:	00f58563          	beq	a1,a5,0x108
     102:	8522                	mv	a0,s0
     104:	066010ef          	jal	0x116a
     108:	444c                	lw	a1,12(s0)
     10a:	e5018793          	addi	a5,gp,-432
     10e:	00f58863          	beq	a1,a5,0x11e
     112:	8522                	mv	a0,s0
     114:	4422                	lw	s0,8(sp)
     116:	40b2                	lw	ra,12(sp)
     118:	0141                	addi	sp,sp,16
     11a:	0500106f          	j	0x116a
     11e:	40b2                	lw	ra,12(sp)
     120:	4422                	lw	s0,8(sp)
     122:	0141                	addi	sp,sp,16
     124:	8082                	ret
     126:	4501                	li	a0,0
     128:	8082                	ret
     12a:	1101                	addi	sp,sp,-32
     12c:	cc22                	sw	s0,24(sp)
     12e:	67c1                	lui	a5,0x10
     130:	d8018413          	addi	s0,gp,-640
     134:	ce06                	sw	ra,28(sp)
     136:	ca26                	sw	s1,20(sp)
     138:	c84a                	sw	s2,16(sp)
     13a:	c64e                	sw	s3,12(sp)
     13c:	c452                	sw	s4,8(sp)
     13e:	4711                	li	a4,4
     140:	18278793          	addi	a5,a5,386 # 0x10182
     144:	4621                	li	a2,8
     146:	4581                	li	a1,0
     148:	ddc18513          	addi	a0,gp,-548
     14c:	d4f1a423          	sw	a5,-696(gp)
     150:	c458                	sw	a4,12(s0)
     152:	00042023          	sw	zero,0(s0)
     156:	00042223          	sw	zero,4(s0)
     15a:	00042423          	sw	zero,8(s0)
     15e:	06042223          	sw	zero,100(s0)
     162:	00042823          	sw	zero,16(s0)
     166:	00042a23          	sw	zero,20(s0)
     16a:	00042c23          	sw	zero,24(s0)
     16e:	2385                	jal	0x6ce
     170:	67c1                	lui	a5,0x10
     172:	6a41                	lui	s4,0x10
     174:	69c1                	lui	s3,0x10
     176:	6941                	lui	s2,0x10
     178:	64c1                	lui	s1,0x10
     17a:	4e6a0a13          	addi	s4,s4,1254 # 0x104e6
     17e:	52098993          	addi	s3,s3,1312 # 0x10520
     182:	57090913          	addi	s2,s2,1392 # 0x10570
     186:	5ac48493          	addi	s1,s1,1452 # 0x105ac
     18a:	07a5                	addi	a5,a5,9 # 0x10009
     18c:	4621                	li	a2,8
     18e:	4581                	li	a1,0
     190:	e4418513          	addi	a0,gp,-444
     194:	d87c                	sw	a5,116(s0)
     196:	03442023          	sw	s4,32(s0)
     19a:	03342223          	sw	s3,36(s0)
     19e:	03242423          	sw	s2,40(s0)
     1a2:	d444                	sw	s1,44(s0)
     1a4:	cc40                	sw	s0,28(s0)
     1a6:	06042423          	sw	zero,104(s0)
     1aa:	06042623          	sw	zero,108(s0)
     1ae:	06042823          	sw	zero,112(s0)
     1b2:	0c042623          	sw	zero,204(s0)
     1b6:	06042c23          	sw	zero,120(s0)
     1ba:	06042e23          	sw	zero,124(s0)
     1be:	08042023          	sw	zero,128(s0)
     1c2:	2331                	jal	0x6ce
     1c4:	000207b7          	lui	a5,0x20
     1c8:	07c9                	addi	a5,a5,18 # 0x20012
     1ca:	de818713          	addi	a4,gp,-536
     1ce:	eac18513          	addi	a0,gp,-340
     1d2:	4621                	li	a2,8
     1d4:	4581                	li	a1,0
     1d6:	09442423          	sw	s4,136(s0)
     1da:	09342623          	sw	s3,140(s0)
     1de:	09242823          	sw	s2,144(s0)
     1e2:	08942a23          	sw	s1,148(s0)
     1e6:	0cf42e23          	sw	a5,220(s0)
     1ea:	0c042823          	sw	zero,208(s0)
     1ee:	0c042a23          	sw	zero,212(s0)
     1f2:	0c042c23          	sw	zero,216(s0)
     1f6:	12042a23          	sw	zero,308(s0)
     1fa:	0e042023          	sw	zero,224(s0)
     1fe:	0e042223          	sw	zero,228(s0)
     202:	0e042423          	sw	zero,232(s0)
     206:	08e42223          	sw	a4,132(s0)
     20a:	21d1                	jal	0x6ce
     20c:	e5018793          	addi	a5,gp,-432
     210:	0f442823          	sw	s4,240(s0)
     214:	0f342a23          	sw	s3,244(s0)
     218:	0f242c23          	sw	s2,248(s0)
     21c:	0e942e23          	sw	s1,252(s0)
     220:	40f2                	lw	ra,28(sp)
     222:	0ef42623          	sw	a5,236(s0)
     226:	4462                	lw	s0,24(sp)
     228:	44d2                	lw	s1,20(sp)
     22a:	4942                	lw	s2,16(sp)
     22c:	49b2                	lw	s3,12(sp)
     22e:	4a22                	lw	s4,8(sp)
     230:	6105                	addi	sp,sp,32
     232:	8082                	ret
     234:	d481a783          	lw	a5,-696(gp)
     238:	1101                	addi	sp,sp,-32
     23a:	c64e                	sw	s3,12(sp)
     23c:	ce06                	sw	ra,28(sp)
     23e:	cc22                	sw	s0,24(sp)
     240:	ca26                	sw	s1,20(sp)
     242:	c84a                	sw	s2,16(sp)
     244:	89aa                	mv	s3,a0
     246:	c7cd                	beqz	a5,0x2f0
     248:	694d                	lui	s2,0x13
     24a:	48090913          	addi	s2,s2,1152 # 0x13480
     24e:	54fd                	li	s1,-1
     250:	00492783          	lw	a5,4(s2)
     254:	00892403          	lw	s0,8(s2)
     258:	17fd                	addi	a5,a5,-1
     25a:	0007d763          	bgez	a5,0x268
     25e:	a8b9                	j	0x2bc
     260:	06840413          	addi	s0,s0,104
     264:	04978c63          	beq	a5,s1,0x2bc
     268:	00c41703          	lh	a4,12(s0)
     26c:	17fd                	addi	a5,a5,-1
     26e:	fb6d                	bnez	a4,0x260
     270:	77c1                	lui	a5,0xffff0
     272:	0785                	addi	a5,a5,1 # 0xffff0001
     274:	06042223          	sw	zero,100(s0)
     278:	00042023          	sw	zero,0(s0)
     27c:	00042423          	sw	zero,8(s0)
     280:	00042223          	sw	zero,4(s0)
     284:	00042823          	sw	zero,16(s0)
     288:	00042a23          	sw	zero,20(s0)
     28c:	00042c23          	sw	zero,24(s0)
     290:	c45c                	sw	a5,12(s0)
     292:	4621                	li	a2,8
     294:	4581                	li	a1,0
     296:	05c40513          	addi	a0,s0,92
     29a:	2915                	jal	0x6ce
     29c:	02042823          	sw	zero,48(s0)
     2a0:	02042a23          	sw	zero,52(s0)
     2a4:	04042223          	sw	zero,68(s0)
     2a8:	04042423          	sw	zero,72(s0)
     2ac:	40f2                	lw	ra,28(sp)
     2ae:	8522                	mv	a0,s0
     2b0:	4462                	lw	s0,24(sp)
     2b2:	44d2                	lw	s1,20(sp)
     2b4:	4942                	lw	s2,16(sp)
     2b6:	49b2                	lw	s3,12(sp)
     2b8:	6105                	addi	sp,sp,32
     2ba:	8082                	ret
     2bc:	00092403          	lw	s0,0(s2)
     2c0:	c019                	beqz	s0,0x2c6
     2c2:	8922                	mv	s2,s0
     2c4:	b771                	j	0x250
     2c6:	1ac00593          	li	a1,428
     2ca:	854e                	mv	a0,s3
     2cc:	0cf000ef          	jal	0xb9a
     2d0:	842a                	mv	s0,a0
     2d2:	c10d                	beqz	a0,0x2f4
     2d4:	4791                	li	a5,4
     2d6:	0531                	addi	a0,a0,12
     2d8:	00042023          	sw	zero,0(s0)
     2dc:	c05c                	sw	a5,4(s0)
     2de:	c408                	sw	a0,8(s0)
     2e0:	1a000613          	li	a2,416
     2e4:	4581                	li	a1,0
     2e6:	26e5                	jal	0x6ce
     2e8:	00892023          	sw	s0,0(s2)
     2ec:	8922                	mv	s2,s0
     2ee:	b78d                	j	0x250
     2f0:	3d2d                	jal	0x12a
     2f2:	bf99                	j	0x248
     2f4:	00092023          	sw	zero,0(s2)
     2f8:	47b1                	li	a5,12
     2fa:	00f9a023          	sw	a5,0(s3)
     2fe:	b77d                	j	0x2ac
     300:	595c                	lw	a5,52(a0)
     302:	c391                	beqz	a5,0x306
     304:	8082                	ret
     306:	67c1                	lui	a5,0x10
     308:	d481a703          	lw	a4,-696(gp)
     30c:	19678793          	addi	a5,a5,406 # 0x10196
     310:	d95c                	sw	a5,52(a0)
     312:	fb6d                	bnez	a4,0x304
     314:	bd19                	j	0x12a
     316:	8082                	ret
     318:	8082                	ret
     31a:	664d                	lui	a2,0x13
     31c:	65c1                	lui	a1,0x10
     31e:	48060613          	addi	a2,a2,1152 # 0x13480
     322:	17e58593          	addi	a1,a1,382 # 0x1017e
     326:	4501                	li	a0,0
     328:	a809                	j	0x33a
     32a:	664d                	lui	a2,0x13
     32c:	65c1                	lui	a1,0x10
     32e:	48060613          	addi	a2,a2,1152 # 0x13480
     332:	1da58593          	addi	a1,a1,474 # 0x101da
     336:	4501                	li	a0,0
     338:	a009                	j	0x33a
     33a:	7179                	addi	sp,sp,-48
     33c:	d04a                	sw	s2,32(sp)
     33e:	ce4e                	sw	s3,28(sp)
     340:	cc52                	sw	s4,24(sp)
     342:	ca56                	sw	s5,20(sp)
     344:	c85a                	sw	s6,16(sp)
     346:	c65e                	sw	s7,12(sp)
     348:	d606                	sw	ra,44(sp)
     34a:	d422                	sw	s0,40(sp)
     34c:	d226                	sw	s1,36(sp)
     34e:	8b2a                	mv	s6,a0
     350:	8bae                	mv	s7,a1
     352:	8ab2                	mv	s5,a2
     354:	4a01                	li	s4,0
     356:	4985                	li	s3,1
     358:	597d                	li	s2,-1
     35a:	004aa483          	lw	s1,4(s5)
     35e:	008aa403          	lw	s0,8(s5)
     362:	14fd                	addi	s1,s1,-1
     364:	0204c463          	bltz	s1,0x38c
     368:	00c45783          	lhu	a5,12(s0)
     36c:	00f9fb63          	bgeu	s3,a5,0x382
     370:	00e41783          	lh	a5,14(s0)
     374:	85a2                	mv	a1,s0
     376:	855a                	mv	a0,s6
     378:	01278563          	beq	a5,s2,0x382
     37c:	9b82                	jalr	s7
     37e:	00aa6a33          	or	s4,s4,a0
     382:	14fd                	addi	s1,s1,-1
     384:	06840413          	addi	s0,s0,104
     388:	ff2490e3          	bne	s1,s2,0x368
     38c:	000aaa83          	lw	s5,0(s5)
     390:	fc0a95e3          	bnez	s5,0x35a
     394:	50b2                	lw	ra,44(sp)
     396:	5422                	lw	s0,40(sp)
     398:	5492                	lw	s1,36(sp)
     39a:	5902                	lw	s2,32(sp)
     39c:	49f2                	lw	s3,28(sp)
     39e:	4ad2                	lw	s5,20(sp)
     3a0:	4b42                	lw	s6,16(sp)
     3a2:	4bb2                	lw	s7,12(sp)
     3a4:	8552                	mv	a0,s4
     3a6:	4a62                	lw	s4,24(sp)
     3a8:	6145                	addi	sp,sp,48
     3aa:	8082                	ret
     3ac:	7139                	addi	sp,sp,-64
     3ae:	dc22                	sw	s0,56(sp)
     3b0:	842a                	mv	s0,a0
     3b2:	852e                	mv	a0,a1
     3b4:	da26                	sw	s1,52(sp)
     3b6:	de06                	sw	ra,60(sp)
     3b8:	84ae                	mv	s1,a1
     3ba:	2e75                	jal	0x776
     3bc:	67c9                	lui	a5,0x12
     3be:	5858                	lw	a4,52(s0)
     3c0:	4585                	li	a1,1
     3c2:	00150813          	addi	a6,a0,1
     3c6:	46c78793          	addi	a5,a5,1132 # 0x1246c
     3ca:	1010                	addi	a2,sp,32
     3cc:	4689                	li	a3,2
     3ce:	d62e                	sw	a1,44(sp)
     3d0:	d026                	sw	s1,32(sp)
     3d2:	d22a                	sw	a0,36(sp)
     3d4:	ce42                	sw	a6,28(sp)
     3d6:	d43e                	sw	a5,40(sp)
     3d8:	ca32                	sw	a2,20(sp)
     3da:	cc36                	sw	a3,24(sp)
     3dc:	440c                	lw	a1,8(s0)
     3de:	c329                	beqz	a4,0x420
     3e0:	00c59783          	lh	a5,12(a1)
     3e4:	51f8                	lw	a4,100(a1)
     3e6:	6609                	lui	a2,0x2
     3e8:	01279693          	slli	a3,a5,0x12
     3ec:	0206c463          	bltz	a3,0x414
     3f0:	76f9                	lui	a3,0xffffe
     3f2:	16fd                	addi	a3,a3,-1 # 0xffffdfff
     3f4:	8fd1                	or	a5,a5,a2
     3f6:	8f75                	and	a4,a4,a3
     3f8:	00f59623          	sh	a5,12(a1)
     3fc:	d1f8                	sw	a4,100(a1)
     3fe:	8522                	mv	a0,s0
     400:	0850                	addi	a2,sp,20
     402:	034010ef          	jal	0x1436
     406:	e919                	bnez	a0,0x41c
     408:	4529                	li	a0,10
     40a:	50f2                	lw	ra,60(sp)
     40c:	5462                	lw	s0,56(sp)
     40e:	54d2                	lw	s1,52(sp)
     410:	6121                	addi	sp,sp,64
     412:	8082                	ret
     414:	01271793          	slli	a5,a4,0x12
     418:	fe07d3e3          	bgez	a5,0x3fe
     41c:	557d                	li	a0,-1
     41e:	b7f5                	j	0x40a
     420:	8522                	mv	a0,s0
     422:	c62e                	sw	a1,12(sp)
     424:	3df1                	jal	0x300
     426:	45b2                	lw	a1,12(sp)
     428:	bf65                	j	0x3e0
     42a:	85aa                	mv	a1,a0
     42c:	d3c1a503          	lw	a0,-708(gp)
     430:	bfb5                	j	0x3ac
     432:	1141                	addi	sp,sp,-16
     434:	c422                	sw	s0,8(sp)
     436:	842e                	mv	s0,a1
     438:	00e59583          	lh	a1,14(a1)
     43c:	c606                	sw	ra,12(sp)
     43e:	227d                	jal	0x5ec
     440:	00054963          	bltz	a0,0x452
     444:	483c                	lw	a5,80(s0)
     446:	40b2                	lw	ra,12(sp)
     448:	97aa                	add	a5,a5,a0
     44a:	c83c                	sw	a5,80(s0)
     44c:	4422                	lw	s0,8(sp)
     44e:	0141                	addi	sp,sp,16
     450:	8082                	ret
     452:	00c45783          	lhu	a5,12(s0)
     456:	777d                	lui	a4,0xfffff
     458:	177d                	addi	a4,a4,-1 # 0xffffefff
     45a:	8ff9                	and	a5,a5,a4
     45c:	40b2                	lw	ra,12(sp)
     45e:	00f41623          	sh	a5,12(s0)
     462:	4422                	lw	s0,8(sp)
     464:	0141                	addi	sp,sp,16
     466:	8082                	ret
     468:	4501                	li	a0,0
     46a:	8082                	ret
     46c:	00c59783          	lh	a5,12(a1)
     470:	1101                	addi	sp,sp,-32
     472:	cc22                	sw	s0,24(sp)
     474:	ca26                	sw	s1,20(sp)
     476:	c84a                	sw	s2,16(sp)
     478:	c64e                	sw	s3,12(sp)
     47a:	ce06                	sw	ra,28(sp)
     47c:	1007f713          	andi	a4,a5,256
     480:	842e                	mv	s0,a1
     482:	8932                	mv	s2,a2
     484:	89b6                	mv	s3,a3
     486:	84aa                	mv	s1,a0
     488:	e315                	bnez	a4,0x4ac
     48a:	777d                	lui	a4,0xfffff
     48c:	177d                	addi	a4,a4,-1 # 0xffffefff
     48e:	8ff9                	and	a5,a5,a4
     490:	00e41583          	lh	a1,14(s0)
     494:	00f41623          	sh	a5,12(s0)
     498:	4462                	lw	s0,24(sp)
     49a:	40f2                	lw	ra,28(sp)
     49c:	86ce                	mv	a3,s3
     49e:	864a                	mv	a2,s2
     4a0:	49b2                	lw	s3,12(sp)
     4a2:	4942                	lw	s2,16(sp)
     4a4:	8526                	mv	a0,s1
     4a6:	44d2                	lw	s1,20(sp)
     4a8:	6105                	addi	sp,sp,32
     4aa:	aabd                	j	0x628
     4ac:	00e59583          	lh	a1,14(a1)
     4b0:	4689                	li	a3,2
     4b2:	4601                	li	a2,0
     4b4:	28f5                	jal	0x5b0
     4b6:	00c41783          	lh	a5,12(s0)
     4ba:	bfc1                	j	0x48a
     4bc:	1141                	addi	sp,sp,-16
     4be:	c422                	sw	s0,8(sp)
     4c0:	842e                	mv	s0,a1
     4c2:	00e59583          	lh	a1,14(a1)
     4c6:	c606                	sw	ra,12(sp)
     4c8:	20e5                	jal	0x5b0
     4ca:	577d                	li	a4,-1
     4cc:	00c41783          	lh	a5,12(s0)
     4d0:	00e50b63          	beq	a0,a4,0x4e6
     4d4:	6705                	lui	a4,0x1
     4d6:	8fd9                	or	a5,a5,a4
     4d8:	40b2                	lw	ra,12(sp)
     4da:	c828                	sw	a0,80(s0)
     4dc:	00f41623          	sh	a5,12(s0)
     4e0:	4422                	lw	s0,8(sp)
     4e2:	0141                	addi	sp,sp,16
     4e4:	8082                	ret
     4e6:	777d                	lui	a4,0xfffff
     4e8:	177d                	addi	a4,a4,-1 # 0xffffefff
     4ea:	8ff9                	and	a5,a5,a4
     4ec:	40b2                	lw	ra,12(sp)
     4ee:	00f41623          	sh	a5,12(s0)
     4f2:	4422                	lw	s0,8(sp)
     4f4:	0141                	addi	sp,sp,16
     4f6:	8082                	ret
     4f8:	00e59583          	lh	a1,14(a1)
     4fc:	a009                	j	0x4fe
     4fe:	1141                	addi	sp,sp,-16
     500:	c422                	sw	s0,8(sp)
     502:	c226                	sw	s1,4(sp)
     504:	842a                	mv	s0,a0
     506:	852e                	mv	a0,a1
     508:	c606                	sw	ra,12(sp)
     50a:	d401a623          	sw	zero,-692(gp)
     50e:	4ab010ef          	jal	0x21b8
     512:	57fd                	li	a5,-1
     514:	00f50763          	beq	a0,a5,0x522
     518:	40b2                	lw	ra,12(sp)
     51a:	4422                	lw	s0,8(sp)
     51c:	4492                	lw	s1,4(sp)
     51e:	0141                	addi	sp,sp,16
     520:	8082                	ret
     522:	d4c1a783          	lw	a5,-692(gp)
     526:	dbed                	beqz	a5,0x518
     528:	40b2                	lw	ra,12(sp)
     52a:	c01c                	sw	a5,0(s0)
     52c:	4422                	lw	s0,8(sp)
     52e:	4492                	lw	s1,4(sp)
     530:	0141                	addi	sp,sp,16
     532:	8082                	ret
     534:	d3c1a783          	lw	a5,-708(gp)
     538:	06a78b63          	beq	a5,a0,0x5ae
     53c:	416c                	lw	a1,68(a0)
     53e:	1101                	addi	sp,sp,-32
     540:	ca26                	sw	s1,20(sp)
     542:	ce06                	sw	ra,28(sp)
     544:	cc22                	sw	s0,24(sp)
     546:	84aa                	mv	s1,a0
     548:	c59d                	beqz	a1,0x576
     54a:	c84a                	sw	s2,16(sp)
     54c:	c64e                	sw	s3,12(sp)
     54e:	4901                	li	s2,0
     550:	08000993          	li	s3,128
     554:	012587b3          	add	a5,a1,s2
     558:	4380                	lw	s0,0(a5)
     55a:	c419                	beqz	s0,0x568
     55c:	85a2                	mv	a1,s0
     55e:	4000                	lw	s0,0(s0)
     560:	8526                	mv	a0,s1
     562:	2931                	jal	0x97e
     564:	fc65                	bnez	s0,0x55c
     566:	40ec                	lw	a1,68(s1)
     568:	0911                	addi	s2,s2,4
     56a:	ff3915e3          	bne	s2,s3,0x554
     56e:	8526                	mv	a0,s1
     570:	2139                	jal	0x97e
     572:	4942                	lw	s2,16(sp)
     574:	49b2                	lw	s3,12(sp)
     576:	5c8c                	lw	a1,56(s1)
     578:	c199                	beqz	a1,0x57e
     57a:	8526                	mv	a0,s1
     57c:	2109                	jal	0x97e
     57e:	40a0                	lw	s0,64(s1)
     580:	c411                	beqz	s0,0x58c
     582:	85a2                	mv	a1,s0
     584:	4000                	lw	s0,0(s0)
     586:	8526                	mv	a0,s1
     588:	2edd                	jal	0x97e
     58a:	fc65                	bnez	s0,0x582
     58c:	44ec                	lw	a1,76(s1)
     58e:	c199                	beqz	a1,0x594
     590:	8526                	mv	a0,s1
     592:	26f5                	jal	0x97e
     594:	58dc                	lw	a5,52(s1)
     596:	c799                	beqz	a5,0x5a4
     598:	4462                	lw	s0,24(sp)
     59a:	40f2                	lw	ra,28(sp)
     59c:	8526                	mv	a0,s1
     59e:	44d2                	lw	s1,20(sp)
     5a0:	6105                	addi	sp,sp,32
     5a2:	8782                	jr	a5
     5a4:	40f2                	lw	ra,28(sp)
     5a6:	4462                	lw	s0,24(sp)
     5a8:	44d2                	lw	s1,20(sp)
     5aa:	6105                	addi	sp,sp,32
     5ac:	8082                	ret
     5ae:	8082                	ret
     5b0:	1141                	addi	sp,sp,-16
     5b2:	872e                	mv	a4,a1
     5b4:	c422                	sw	s0,8(sp)
     5b6:	c226                	sw	s1,4(sp)
     5b8:	85b2                	mv	a1,a2
     5ba:	842a                	mv	s0,a0
     5bc:	8636                	mv	a2,a3
     5be:	853a                	mv	a0,a4
     5c0:	c606                	sw	ra,12(sp)
     5c2:	d401a623          	sw	zero,-692(gp)
     5c6:	497010ef          	jal	0x225c
     5ca:	57fd                	li	a5,-1
     5cc:	00f50763          	beq	a0,a5,0x5da
     5d0:	40b2                	lw	ra,12(sp)
     5d2:	4422                	lw	s0,8(sp)
     5d4:	4492                	lw	s1,4(sp)
     5d6:	0141                	addi	sp,sp,16
     5d8:	8082                	ret
     5da:	d4c1a783          	lw	a5,-692(gp)
     5de:	dbed                	beqz	a5,0x5d0
     5e0:	40b2                	lw	ra,12(sp)
     5e2:	c01c                	sw	a5,0(s0)
     5e4:	4422                	lw	s0,8(sp)
     5e6:	4492                	lw	s1,4(sp)
     5e8:	0141                	addi	sp,sp,16
     5ea:	8082                	ret
     5ec:	1141                	addi	sp,sp,-16
     5ee:	872e                	mv	a4,a1
     5f0:	c422                	sw	s0,8(sp)
     5f2:	c226                	sw	s1,4(sp)
     5f4:	85b2                	mv	a1,a2
     5f6:	842a                	mv	s0,a0
     5f8:	8636                	mv	a2,a3
     5fa:	853a                	mv	a0,a4
     5fc:	c606                	sw	ra,12(sp)
     5fe:	d401a623          	sw	zero,-692(gp)
     602:	485010ef          	jal	0x2286
     606:	57fd                	li	a5,-1
     608:	00f50763          	beq	a0,a5,0x616
     60c:	40b2                	lw	ra,12(sp)
     60e:	4422                	lw	s0,8(sp)
     610:	4492                	lw	s1,4(sp)
     612:	0141                	addi	sp,sp,16
     614:	8082                	ret
     616:	d4c1a783          	lw	a5,-692(gp)
     61a:	dbed                	beqz	a5,0x60c
     61c:	40b2                	lw	ra,12(sp)
     61e:	c01c                	sw	a5,0(s0)
     620:	4422                	lw	s0,8(sp)
     622:	4492                	lw	s1,4(sp)
     624:	0141                	addi	sp,sp,16
     626:	8082                	ret
     628:	1141                	addi	sp,sp,-16
     62a:	872e                	mv	a4,a1
     62c:	c422                	sw	s0,8(sp)
     62e:	c226                	sw	s1,4(sp)
     630:	85b2                	mv	a1,a2
     632:	842a                	mv	s0,a0
     634:	8636                	mv	a2,a3
     636:	853a                	mv	a0,a4
     638:	c606                	sw	ra,12(sp)
     63a:	d401a623          	sw	zero,-692(gp)
     63e:	4c5010ef          	jal	0x2302
     642:	57fd                	li	a5,-1
     644:	00f50763          	beq	a0,a5,0x652
     648:	40b2                	lw	ra,12(sp)
     64a:	4422                	lw	s0,8(sp)
     64c:	4492                	lw	s1,4(sp)
     64e:	0141                	addi	sp,sp,16
     650:	8082                	ret
     652:	d4c1a783          	lw	a5,-692(gp)
     656:	dbed                	beqz	a5,0x648
     658:	40b2                	lw	ra,12(sp)
     65a:	c01c                	sw	a5,0(s0)
     65c:	4422                	lw	s0,8(sp)
     65e:	4492                	lw	s1,4(sp)
     660:	0141                	addi	sp,sp,16
     662:	8082                	ret
     664:	1141                	addi	sp,sp,-16
     666:	c422                	sw	s0,8(sp)
     668:	67cd                	lui	a5,0x13
     66a:	644d                	lui	s0,0x13
     66c:	c04a                	sw	s2,0(sp)
     66e:	47478793          	addi	a5,a5,1140 # 0x13474
     672:	47440713          	addi	a4,s0,1140 # 0x13474
     676:	c606                	sw	ra,12(sp)
     678:	c226                	sw	s1,4(sp)
     67a:	40e78933          	sub	s2,a5,a4
     67e:	00e78d63          	beq	a5,a4,0x698
     682:	40295913          	srai	s2,s2,0x2
     686:	47440413          	addi	s0,s0,1140
     68a:	4481                	li	s1,0
     68c:	401c                	lw	a5,0(s0)
     68e:	0485                	addi	s1,s1,1
     690:	0411                	addi	s0,s0,4
     692:	9782                	jalr	a5
     694:	ff24ece3          	bltu	s1,s2,0x68c
     698:	67cd                	lui	a5,0x13
     69a:	644d                	lui	s0,0x13
     69c:	47c78793          	addi	a5,a5,1148 # 0x1347c
     6a0:	47440713          	addi	a4,s0,1140 # 0x13474
     6a4:	40e78933          	sub	s2,a5,a4
     6a8:	40295913          	srai	s2,s2,0x2
     6ac:	00e78b63          	beq	a5,a4,0x6c2
     6b0:	47440413          	addi	s0,s0,1140
     6b4:	4481                	li	s1,0
     6b6:	401c                	lw	a5,0(s0)
     6b8:	0485                	addi	s1,s1,1
     6ba:	0411                	addi	s0,s0,4
     6bc:	9782                	jalr	a5
     6be:	ff24ece3          	bltu	s1,s2,0x6b6
     6c2:	40b2                	lw	ra,12(sp)
     6c4:	4422                	lw	s0,8(sp)
     6c6:	4492                	lw	s1,4(sp)
     6c8:	4902                	lw	s2,0(sp)
     6ca:	0141                	addi	sp,sp,16
     6cc:	8082                	ret
     6ce:	433d                	li	t1,15
     6d0:	872a                	mv	a4,a0
     6d2:	02c37363          	bgeu	t1,a2,0x6f8
     6d6:	00f77793          	andi	a5,a4,15
     6da:	efbd                	bnez	a5,0x758
     6dc:	e5ad                	bnez	a1,0x746
     6de:	ff067693          	andi	a3,a2,-16
     6e2:	8a3d                	andi	a2,a2,15
     6e4:	96ba                	add	a3,a3,a4
     6e6:	c30c                	sw	a1,0(a4)
     6e8:	c34c                	sw	a1,4(a4)
     6ea:	c70c                	sw	a1,8(a4)
     6ec:	c74c                	sw	a1,12(a4)
     6ee:	0741                	addi	a4,a4,16
     6f0:	fed76be3          	bltu	a4,a3,0x6e6
     6f4:	e211                	bnez	a2,0x6f8
     6f6:	8082                	ret
     6f8:	40c306b3          	sub	a3,t1,a2
     6fc:	068a                	slli	a3,a3,0x2
     6fe:	00000297          	auipc	t0,0x0
     702:	9696                	add	a3,a3,t0
     704:	00a68067          	jr	10(a3)
     708:	00b70723          	sb	a1,14(a4)
     70c:	00b706a3          	sb	a1,13(a4)
     710:	00b70623          	sb	a1,12(a4)
     714:	00b705a3          	sb	a1,11(a4)
     718:	00b70523          	sb	a1,10(a4)
     71c:	00b704a3          	sb	a1,9(a4)
     720:	00b70423          	sb	a1,8(a4)
     724:	00b703a3          	sb	a1,7(a4)
     728:	00b70323          	sb	a1,6(a4)
     72c:	00b702a3          	sb	a1,5(a4)
     730:	00b70223          	sb	a1,4(a4)
     734:	00b701a3          	sb	a1,3(a4)
     738:	00b70123          	sb	a1,2(a4)
     73c:	00b700a3          	sb	a1,1(a4)
     740:	00b70023          	sb	a1,0(a4)
     744:	8082                	ret
     746:	0ff5f593          	zext.b	a1,a1
     74a:	00859693          	slli	a3,a1,0x8
     74e:	8dd5                	or	a1,a1,a3
     750:	01059693          	slli	a3,a1,0x10
     754:	8dd5                	or	a1,a1,a3
     756:	b761                	j	0x6de
     758:	00279693          	slli	a3,a5,0x2
     75c:	00000297          	auipc	t0,0x0
     760:	9696                	add	a3,a3,t0
     762:	8286                	mv	t0,ra
     764:	fa8680e7          	jalr	-88(a3)
     768:	8096                	mv	ra,t0
     76a:	17c1                	addi	a5,a5,-16
     76c:	8f1d                	sub	a4,a4,a5
     76e:	963e                	add	a2,a2,a5
     770:	f8c374e3          	bgeu	t1,a2,0x6f8
     774:	b7a5                	j	0x6dc
     776:	00357793          	andi	a5,a0,3
     77a:	872a                	mv	a4,a0
     77c:	ef9d                	bnez	a5,0x7ba
     77e:	7f7f86b7          	lui	a3,0x7f7f8
     782:	f7f68693          	addi	a3,a3,-129 # 0x7f7f7f7f
     786:	55fd                	li	a1,-1
     788:	4310                	lw	a2,0(a4)
     78a:	0711                	addi	a4,a4,4
     78c:	00d677b3          	and	a5,a2,a3
     790:	97b6                	add	a5,a5,a3
     792:	8fd1                	or	a5,a5,a2
     794:	8fd5                	or	a5,a5,a3
     796:	feb789e3          	beq	a5,a1,0x788
     79a:	ffc74683          	lbu	a3,-4(a4)
     79e:	40a707b3          	sub	a5,a4,a0
     7a2:	ca8d                	beqz	a3,0x7d4
     7a4:	ffd74683          	lbu	a3,-3(a4)
     7a8:	c29d                	beqz	a3,0x7ce
     7aa:	ffe74503          	lbu	a0,-2(a4)
     7ae:	00a03533          	snez	a0,a0
     7b2:	953e                	add	a0,a0,a5
     7b4:	1579                	addi	a0,a0,-2
     7b6:	8082                	ret
     7b8:	d2f9                	beqz	a3,0x77e
     7ba:	00074783          	lbu	a5,0(a4)
     7be:	0705                	addi	a4,a4,1
     7c0:	00377693          	andi	a3,a4,3
     7c4:	fbf5                	bnez	a5,0x7b8
     7c6:	8f09                	sub	a4,a4,a0
     7c8:	fff70513          	addi	a0,a4,-1
     7cc:	8082                	ret
     7ce:	ffd78513          	addi	a0,a5,-3
     7d2:	8082                	ret
     7d4:	ffc78513          	addi	a0,a5,-4
     7d8:	8082                	ret
     7da:	7179                	addi	sp,sp,-48
     7dc:	cc52                	sw	s4,24(sp)
     7de:	d04a                	sw	s2,32(sp)
     7e0:	d501a903          	lw	s2,-688(gp)
     7e4:	d606                	sw	ra,44(sp)
     7e6:	04090663          	beqz	s2,0x832
     7ea:	ce4e                	sw	s3,28(sp)
     7ec:	ca56                	sw	s5,20(sp)
     7ee:	c85a                	sw	s6,16(sp)
     7f0:	c65e                	sw	s7,12(sp)
     7f2:	d422                	sw	s0,40(sp)
     7f4:	d226                	sw	s1,36(sp)
     7f6:	c462                	sw	s8,8(sp)
     7f8:	8b2a                	mv	s6,a0
     7fa:	8bae                	mv	s7,a1
     7fc:	59fd                	li	s3,-1
     7fe:	4a85                	li	s5,1
     800:	00492483          	lw	s1,4(s2)
     804:	fff48413          	addi	s0,s1,-1
     808:	00044e63          	bltz	s0,0x824
     80c:	048a                	slli	s1,s1,0x2
     80e:	94ca                	add	s1,s1,s2
     810:	020b8663          	beqz	s7,0x83c
     814:	1044a783          	lw	a5,260(s1)
     818:	03778263          	beq	a5,s7,0x83c
     81c:	147d                	addi	s0,s0,-1
     81e:	14f1                	addi	s1,s1,-4
     820:	ff341ae3          	bne	s0,s3,0x814
     824:	5422                	lw	s0,40(sp)
     826:	5492                	lw	s1,36(sp)
     828:	49f2                	lw	s3,28(sp)
     82a:	4ad2                	lw	s5,20(sp)
     82c:	4b42                	lw	s6,16(sp)
     82e:	4bb2                	lw	s7,12(sp)
     830:	4c22                	lw	s8,8(sp)
     832:	50b2                	lw	ra,44(sp)
     834:	5902                	lw	s2,32(sp)
     836:	4a62                	lw	s4,24(sp)
     838:	6145                	addi	sp,sp,48
     83a:	8082                	ret
     83c:	00492783          	lw	a5,4(s2)
     840:	40d4                	lw	a3,4(s1)
     842:	17fd                	addi	a5,a5,-1
     844:	04878c63          	beq	a5,s0,0x89c
     848:	0004a223          	sw	zero,4(s1)
     84c:	c295                	beqz	a3,0x870
     84e:	18892783          	lw	a5,392(s2)
     852:	008a9733          	sll	a4,s5,s0
     856:	00492c03          	lw	s8,4(s2)
     85a:	8ff9                	and	a5,a5,a4
     85c:	ef99                	bnez	a5,0x87a
     85e:	9682                	jalr	a3
     860:	00492703          	lw	a4,4(s2)
     864:	d501a783          	lw	a5,-688(gp)
     868:	03871763          	bne	a4,s8,0x896
     86c:	03279563          	bne	a5,s2,0x896
     870:	147d                	addi	s0,s0,-1
     872:	14f1                	addi	s1,s1,-4
     874:	f9341ee3          	bne	s0,s3,0x810
     878:	b775                	j	0x824
     87a:	18c92783          	lw	a5,396(s2)
     87e:	0844a583          	lw	a1,132(s1)
     882:	8f7d                	and	a4,a4,a5
     884:	ef19                	bnez	a4,0x8a2
     886:	855a                	mv	a0,s6
     888:	9682                	jalr	a3
     88a:	00492703          	lw	a4,4(s2)
     88e:	d501a783          	lw	a5,-688(gp)
     892:	fd870de3          	beq	a4,s8,0x86c
     896:	d7d9                	beqz	a5,0x824
     898:	893e                	mv	s2,a5
     89a:	b79d                	j	0x800
     89c:	00892223          	sw	s0,4(s2)
     8a0:	b775                	j	0x84c
     8a2:	852e                	mv	a0,a1
     8a4:	9682                	jalr	a3
     8a6:	bf6d                	j	0x860
     8a8:	85aa                	mv	a1,a0
     8aa:	4681                	li	a3,0
     8ac:	4601                	li	a2,0
     8ae:	4501                	li	a0,0
     8b0:	2a60106f          	j	0x1b56
     8b4:	1101                	addi	sp,sp,-32
     8b6:	c64e                	sw	s3,12(sp)
     8b8:	69cd                	lui	s3,0x13
     8ba:	cc22                	sw	s0,24(sp)
     8bc:	ca26                	sw	s1,20(sp)
     8be:	c84a                	sw	s2,16(sp)
     8c0:	c452                	sw	s4,8(sp)
     8c2:	ce06                	sw	ra,28(sp)
     8c4:	8a2e                	mv	s4,a1
     8c6:	892a                	mv	s2,a0
     8c8:	5b098993          	addi	s3,s3,1456 # 0x135b0
     8cc:	09b000ef          	jal	0x1166
     8d0:	0089a783          	lw	a5,8(s3)
     8d4:	6405                	lui	s0,0x1
     8d6:	143d                	addi	s0,s0,-17 # 0xfef
     8d8:	43c4                	lw	s1,4(a5)
     8da:	6785                	lui	a5,0x1
     8dc:	98f1                	andi	s1,s1,-4
     8de:	9426                	add	s0,s0,s1
     8e0:	41440433          	sub	s0,s0,s4
     8e4:	8031                	srli	s0,s0,0xc
     8e6:	147d                	addi	s0,s0,-1
     8e8:	0432                	slli	s0,s0,0xc
     8ea:	00f44b63          	blt	s0,a5,0x900
     8ee:	4581                	li	a1,0
     8f0:	854a                	mv	a0,s2
     8f2:	032010ef          	jal	0x1924
     8f6:	0089a783          	lw	a5,8(s3)
     8fa:	97a6                	add	a5,a5,s1
     8fc:	00f50e63          	beq	a0,a5,0x918
     900:	854a                	mv	a0,s2
     902:	067000ef          	jal	0x1168
     906:	40f2                	lw	ra,28(sp)
     908:	4462                	lw	s0,24(sp)
     90a:	44d2                	lw	s1,20(sp)
     90c:	4942                	lw	s2,16(sp)
     90e:	49b2                	lw	s3,12(sp)
     910:	4a22                	lw	s4,8(sp)
     912:	4501                	li	a0,0
     914:	6105                	addi	sp,sp,32
     916:	8082                	ret
     918:	408005b3          	neg	a1,s0
     91c:	854a                	mv	a0,s2
     91e:	006010ef          	jal	0x1924
     922:	57fd                	li	a5,-1
     924:	02f50963          	beq	a0,a5,0x956
     928:	eb818793          	addi	a5,gp,-328
     92c:	0089a683          	lw	a3,8(s3)
     930:	4398                	lw	a4,0(a5)
     932:	8c81                	sub	s1,s1,s0
     934:	0014e493          	ori	s1,s1,1
     938:	854a                	mv	a0,s2
     93a:	8f01                	sub	a4,a4,s0
     93c:	c2c4                	sw	s1,4(a3)
     93e:	c398                	sw	a4,0(a5)
     940:	029000ef          	jal	0x1168
     944:	40f2                	lw	ra,28(sp)
     946:	4462                	lw	s0,24(sp)
     948:	44d2                	lw	s1,20(sp)
     94a:	4942                	lw	s2,16(sp)
     94c:	49b2                	lw	s3,12(sp)
     94e:	4a22                	lw	s4,8(sp)
     950:	4505                	li	a0,1
     952:	6105                	addi	sp,sp,32
     954:	8082                	ret
     956:	4581                	li	a1,0
     958:	854a                	mv	a0,s2
     95a:	7cb000ef          	jal	0x1924
     95e:	0089a703          	lw	a4,8(s3)
     962:	46bd                	li	a3,15
     964:	40e507b3          	sub	a5,a0,a4
     968:	f8f6dce3          	bge	a3,a5,0x900
     96c:	d401a603          	lw	a2,-704(gp)
     970:	0017e793          	ori	a5,a5,1
     974:	8d11                	sub	a0,a0,a2
     976:	c35c                	sw	a5,4(a4)
     978:	eaa1ac23          	sw	a0,-328(gp)
     97c:	b751                	j	0x900
     97e:	cdf9                	beqz	a1,0xa5c
     980:	1141                	addi	sp,sp,-16
     982:	c422                	sw	s0,8(sp)
     984:	c226                	sw	s1,4(sp)
     986:	842e                	mv	s0,a1
     988:	84aa                	mv	s1,a0
     98a:	c606                	sw	ra,12(sp)
     98c:	7da000ef          	jal	0x1166
     990:	ffc42503          	lw	a0,-4(s0)
     994:	ff840713          	addi	a4,s0,-8
     998:	65cd                	lui	a1,0x13
     99a:	ffe57793          	andi	a5,a0,-2
     99e:	00f70633          	add	a2,a4,a5
     9a2:	5b058593          	addi	a1,a1,1456 # 0x135b0
     9a6:	4254                	lw	a3,4(a2)
     9a8:	0085a803          	lw	a6,8(a1)
     9ac:	00157893          	andi	a7,a0,1
     9b0:	9af1                	andi	a3,a3,-4
     9b2:	12c80063          	beq	a6,a2,0xad2
     9b6:	c254                	sw	a3,4(a2)
     9b8:	00d60833          	add	a6,a2,a3
     9bc:	00482803          	lw	a6,4(a6)
     9c0:	00187813          	andi	a6,a6,1
     9c4:	06089763          	bnez	a7,0xa32
     9c8:	ff842303          	lw	t1,-8(s0)
     9cc:	654d                	lui	a0,0x13
     9ce:	5b850513          	addi	a0,a0,1464 # 0x135b8
     9d2:	40670733          	sub	a4,a4,t1
     9d6:	00872883          	lw	a7,8(a4)
     9da:	979a                	add	a5,a5,t1
     9dc:	0ca88e63          	beq	a7,a0,0xab8
     9e0:	00c72303          	lw	t1,12(a4)
     9e4:	0068a623          	sw	t1,12(a7)
     9e8:	01132423          	sw	a7,8(t1) # 0xac
     9ec:	10080b63          	beqz	a6,0xb02
     9f0:	0017e693          	ori	a3,a5,1
     9f4:	c354                	sw	a3,4(a4)
     9f6:	c21c                	sw	a5,0(a2)
     9f8:	1ff00693          	li	a3,511
     9fc:	06f6ea63          	bltu	a3,a5,0xa70
     a00:	ff87f693          	andi	a3,a5,-8
     a04:	06a1                	addi	a3,a3,8
     a06:	41c8                	lw	a0,4(a1)
     a08:	96ae                	add	a3,a3,a1
     a0a:	4290                	lw	a2,0(a3)
     a0c:	0057d813          	srli	a6,a5,0x5
     a10:	4785                	li	a5,1
     a12:	010797b3          	sll	a5,a5,a6
     a16:	8fc9                	or	a5,a5,a0
     a18:	ff868513          	addi	a0,a3,-8
     a1c:	c710                	sw	a2,8(a4)
     a1e:	c748                	sw	a0,12(a4)
     a20:	c1dc                	sw	a5,4(a1)
     a22:	c298                	sw	a4,0(a3)
     a24:	c658                	sw	a4,12(a2)
     a26:	4422                	lw	s0,8(sp)
     a28:	40b2                	lw	ra,12(sp)
     a2a:	8526                	mv	a0,s1
     a2c:	4492                	lw	s1,4(sp)
     a2e:	0141                	addi	sp,sp,16
     a30:	af25                	j	0x1168
     a32:	02081663          	bnez	a6,0xa5e
     a36:	654d                	lui	a0,0x13
     a38:	97b6                	add	a5,a5,a3
     a3a:	5b850513          	addi	a0,a0,1464 # 0x135b8
     a3e:	4614                	lw	a3,8(a2)
     a40:	0017e893          	ori	a7,a5,1
     a44:	00f70833          	add	a6,a4,a5
     a48:	0ea68963          	beq	a3,a0,0xb3a
     a4c:	4650                	lw	a2,12(a2)
     a4e:	c6d0                	sw	a2,12(a3)
     a50:	c614                	sw	a3,8(a2)
     a52:	01172223          	sw	a7,4(a4)
     a56:	00f82023          	sw	a5,0(a6)
     a5a:	bf79                	j	0x9f8
     a5c:	8082                	ret
     a5e:	00156513          	ori	a0,a0,1
     a62:	fea42e23          	sw	a0,-4(s0)
     a66:	c21c                	sw	a5,0(a2)
     a68:	1ff00693          	li	a3,511
     a6c:	f8f6fae3          	bgeu	a3,a5,0xa00
     a70:	0097d693          	srli	a3,a5,0x9
     a74:	4611                	li	a2,4
     a76:	08d66863          	bltu	a2,a3,0xb06
     a7a:	0067d693          	srli	a3,a5,0x6
     a7e:	03968513          	addi	a0,a3,57
     a82:	050e                	slli	a0,a0,0x3
     a84:	03868613          	addi	a2,a3,56
     a88:	952e                	add	a0,a0,a1
     a8a:	4114                	lw	a3,0(a0)
     a8c:	1561                	addi	a0,a0,-8
     a8e:	00d51663          	bne	a0,a3,0xa9a
     a92:	a86d                	j	0xb4c
     a94:	4694                	lw	a3,8(a3)
     a96:	00d50663          	beq	a0,a3,0xaa2
     a9a:	42d0                	lw	a2,4(a3)
     a9c:	9a71                	andi	a2,a2,-4
     a9e:	fec7ebe3          	bltu	a5,a2,0xa94
     aa2:	46c8                	lw	a0,12(a3)
     aa4:	c748                	sw	a0,12(a4)
     aa6:	c714                	sw	a3,8(a4)
     aa8:	4422                	lw	s0,8(sp)
     aaa:	c518                	sw	a4,8(a0)
     aac:	40b2                	lw	ra,12(sp)
     aae:	8526                	mv	a0,s1
     ab0:	4492                	lw	s1,4(sp)
     ab2:	c6d8                	sw	a4,12(a3)
     ab4:	0141                	addi	sp,sp,16
     ab6:	ad4d                	j	0x1168
     ab8:	06081663          	bnez	a6,0xb24
     abc:	464c                	lw	a1,12(a2)
     abe:	4610                	lw	a2,8(a2)
     ac0:	96be                	add	a3,a3,a5
     ac2:	0016e793          	ori	a5,a3,1
     ac6:	c64c                	sw	a1,12(a2)
     ac8:	c590                	sw	a2,8(a1)
     aca:	c35c                	sw	a5,4(a4)
     acc:	9736                	add	a4,a4,a3
     ace:	c314                	sw	a3,0(a4)
     ad0:	bf99                	j	0xa26
     ad2:	96be                	add	a3,a3,a5
     ad4:	00089a63          	bnez	a7,0xae8
     ad8:	ff842503          	lw	a0,-8(s0)
     adc:	8f09                	sub	a4,a4,a0
     ade:	475c                	lw	a5,12(a4)
     ae0:	4710                	lw	a2,8(a4)
     ae2:	96aa                	add	a3,a3,a0
     ae4:	c65c                	sw	a5,12(a2)
     ae6:	c790                	sw	a2,8(a5)
     ae8:	0016e613          	ori	a2,a3,1
     aec:	d441a783          	lw	a5,-700(gp)
     af0:	c350                	sw	a2,4(a4)
     af2:	c598                	sw	a4,8(a1)
     af4:	f2f6e9e3          	bltu	a3,a5,0xa26
     af8:	d5c1a583          	lw	a1,-676(gp)
     afc:	8526                	mv	a0,s1
     afe:	3b5d                	jal	0x8b4
     b00:	b71d                	j	0xa26
     b02:	97b6                	add	a5,a5,a3
     b04:	bf2d                	j	0xa3e
     b06:	4651                	li	a2,20
     b08:	02d67363          	bgeu	a2,a3,0xb2e
     b0c:	05400613          	li	a2,84
     b10:	04d66863          	bltu	a2,a3,0xb60
     b14:	00c7d693          	srli	a3,a5,0xc
     b18:	06f68513          	addi	a0,a3,111
     b1c:	050e                	slli	a0,a0,0x3
     b1e:	06e68613          	addi	a2,a3,110
     b22:	b79d                	j	0xa88
     b24:	0017e693          	ori	a3,a5,1
     b28:	c354                	sw	a3,4(a4)
     b2a:	c21c                	sw	a5,0(a2)
     b2c:	bded                	j	0xa26
     b2e:	05c68513          	addi	a0,a3,92
     b32:	050e                	slli	a0,a0,0x3
     b34:	05b68613          	addi	a2,a3,91
     b38:	bf81                	j	0xa88
     b3a:	c9d8                	sw	a4,20(a1)
     b3c:	c998                	sw	a4,16(a1)
     b3e:	c748                	sw	a0,12(a4)
     b40:	c708                	sw	a0,8(a4)
     b42:	01172223          	sw	a7,4(a4)
     b46:	00f82023          	sw	a5,0(a6)
     b4a:	bdf1                	j	0xa26
     b4c:	0045a803          	lw	a6,4(a1)
     b50:	8609                	srai	a2,a2,0x2
     b52:	4785                	li	a5,1
     b54:	00c797b3          	sll	a5,a5,a2
     b58:	0107e7b3          	or	a5,a5,a6
     b5c:	c1dc                	sw	a5,4(a1)
     b5e:	b799                	j	0xaa4
     b60:	15400613          	li	a2,340
     b64:	00d66a63          	bltu	a2,a3,0xb78
     b68:	00f7d693          	srli	a3,a5,0xf
     b6c:	07868513          	addi	a0,a3,120
     b70:	050e                	slli	a0,a0,0x3
     b72:	07768613          	addi	a2,a3,119
     b76:	bf09                	j	0xa88
     b78:	55400613          	li	a2,1364
     b7c:	00d66a63          	bltu	a2,a3,0xb90
     b80:	0127d693          	srli	a3,a5,0x12
     b84:	07d68513          	addi	a0,a3,125
     b88:	050e                	slli	a0,a0,0x3
     b8a:	07c68613          	addi	a2,a3,124
     b8e:	bded                	j	0xa88
     b90:	3f800513          	li	a0,1016
     b94:	07e00613          	li	a2,126
     b98:	bdc5                	j	0xa88
     b9a:	7179                	addi	sp,sp,-48
     b9c:	ce4e                	sw	s3,28(sp)
     b9e:	d606                	sw	ra,44(sp)
     ba0:	d422                	sw	s0,40(sp)
     ba2:	d226                	sw	s1,36(sp)
     ba4:	d04a                	sw	s2,32(sp)
     ba6:	00b58793          	addi	a5,a1,11
     baa:	4759                	li	a4,22
     bac:	89aa                	mv	s3,a0
     bae:	04f76763          	bltu	a4,a5,0xbfc
     bb2:	44c1                	li	s1,16
     bb4:	16b4e763          	bltu	s1,a1,0xd22
     bb8:	237d                	jal	0x1166
     bba:	47e1                	li	a5,24
     bbc:	4589                	li	a1,2
     bbe:	694d                	lui	s2,0x13
     bc0:	5b090913          	addi	s2,s2,1456 # 0x135b0
     bc4:	97ca                	add	a5,a5,s2
     bc6:	43c0                	lw	s0,4(a5)
     bc8:	ff878713          	addi	a4,a5,-8 # 0xff8
     bcc:	30e40663          	beq	s0,a4,0xed8
     bd0:	405c                	lw	a5,4(s0)
     bd2:	4454                	lw	a3,12(s0)
     bd4:	4410                	lw	a2,8(s0)
     bd6:	9bf1                	andi	a5,a5,-4
     bd8:	97a2                	add	a5,a5,s0
     bda:	43d8                	lw	a4,4(a5)
     bdc:	c654                	sw	a3,12(a2)
     bde:	c690                	sw	a2,8(a3)
     be0:	00176713          	ori	a4,a4,1
     be4:	854e                	mv	a0,s3
     be6:	c3d8                	sw	a4,4(a5)
     be8:	2341                	jal	0x1168
     bea:	50b2                	lw	ra,44(sp)
     bec:	00840513          	addi	a0,s0,8
     bf0:	5422                	lw	s0,40(sp)
     bf2:	5492                	lw	s1,36(sp)
     bf4:	5902                	lw	s2,32(sp)
     bf6:	49f2                	lw	s3,28(sp)
     bf8:	6145                	addi	sp,sp,48
     bfa:	8082                	ret
     bfc:	ff87f493          	andi	s1,a5,-8
     c00:	1207c163          	bltz	a5,0xd22
     c04:	10b4ef63          	bltu	s1,a1,0xd22
     c08:	2bb9                	jal	0x1166
     c0a:	1f700793          	li	a5,503
     c0e:	3897f463          	bgeu	a5,s1,0xf96
     c12:	0094d793          	srli	a5,s1,0x9
     c16:	12078163          	beqz	a5,0xd38
     c1a:	4711                	li	a4,4
     c1c:	30f76363          	bltu	a4,a5,0xf22
     c20:	0064d793          	srli	a5,s1,0x6
     c24:	03978593          	addi	a1,a5,57
     c28:	03878813          	addi	a6,a5,56
     c2c:	00359613          	slli	a2,a1,0x3
     c30:	694d                	lui	s2,0x13
     c32:	5b090913          	addi	s2,s2,1456 # 0x135b0
     c36:	964a                	add	a2,a2,s2
     c38:	4240                	lw	s0,4(a2)
     c3a:	1661                	addi	a2,a2,-8 # 0x1ff8
     c3c:	02860163          	beq	a2,s0,0xc5e
     c40:	453d                	li	a0,15
     c42:	a039                	j	0xc50
     c44:	4454                	lw	a3,12(s0)
     c46:	26075663          	bgez	a4,0xeb2
     c4a:	00d60a63          	beq	a2,a3,0xc5e
     c4e:	8436                	mv	s0,a3
     c50:	405c                	lw	a5,4(s0)
     c52:	9bf1                	andi	a5,a5,-4
     c54:	40978733          	sub	a4,a5,s1
     c58:	fee556e3          	bge	a0,a4,0xc44
     c5c:	85c2                	mv	a1,a6
     c5e:	01092403          	lw	s0,16(s2)
     c62:	684d                	lui	a6,0x13
     c64:	5b880813          	addi	a6,a6,1464 # 0x135b8
     c68:	1f040b63          	beq	s0,a6,0xe5e
     c6c:	405c                	lw	a5,4(s0)
     c6e:	46bd                	li	a3,15
     c70:	9bf1                	andi	a5,a5,-4
     c72:	40978733          	sub	a4,a5,s1
     c76:	32e6c563          	blt	a3,a4,0xfa0
     c7a:	01092a23          	sw	a6,20(s2)
     c7e:	01092823          	sw	a6,16(s2)
     c82:	30075063          	bgez	a4,0xf82
     c86:	1ff00713          	li	a4,511
     c8a:	00492503          	lw	a0,4(s2)
     c8e:	24f76a63          	bltu	a4,a5,0xee2
     c92:	ff87f713          	andi	a4,a5,-8
     c96:	0721                	addi	a4,a4,8
     c98:	974a                	add	a4,a4,s2
     c9a:	4314                	lw	a3,0(a4)
     c9c:	0057d613          	srli	a2,a5,0x5
     ca0:	4785                	li	a5,1
     ca2:	00c797b3          	sll	a5,a5,a2
     ca6:	8d5d                	or	a0,a0,a5
     ca8:	ff870793          	addi	a5,a4,-8
     cac:	c414                	sw	a3,8(s0)
     cae:	c45c                	sw	a5,12(s0)
     cb0:	00a92223          	sw	a0,4(s2)
     cb4:	c300                	sw	s0,0(a4)
     cb6:	c6c0                	sw	s0,12(a3)
     cb8:	4025d793          	srai	a5,a1,0x2
     cbc:	4605                	li	a2,1
     cbe:	00f61633          	sll	a2,a2,a5
     cc2:	08c56263          	bltu	a0,a2,0xd46
     cc6:	00a677b3          	and	a5,a2,a0
     cca:	ef81                	bnez	a5,0xce2
     ccc:	0606                	slli	a2,a2,0x1
     cce:	99f1                	andi	a1,a1,-4
     cd0:	00a677b3          	and	a5,a2,a0
     cd4:	0591                	addi	a1,a1,4
     cd6:	e791                	bnez	a5,0xce2
     cd8:	0606                	slli	a2,a2,0x1
     cda:	00a677b3          	and	a5,a2,a0
     cde:	0591                	addi	a1,a1,4
     ce0:	dfe5                	beqz	a5,0xcd8
     ce2:	48bd                	li	a7,15
     ce4:	00359313          	slli	t1,a1,0x3
     ce8:	934a                	add	t1,t1,s2
     cea:	851a                	mv	a0,t1
     cec:	455c                	lw	a5,12(a0)
     cee:	8e2e                	mv	t3,a1
     cf0:	24f50963          	beq	a0,a5,0xf42
     cf4:	43d8                	lw	a4,4(a5)
     cf6:	843e                	mv	s0,a5
     cf8:	47dc                	lw	a5,12(a5)
     cfa:	9b71                	andi	a4,a4,-4
     cfc:	409706b3          	sub	a3,a4,s1
     d00:	24d8c863          	blt	a7,a3,0xf50
     d04:	fe06c6e3          	bltz	a3,0xcf0
     d08:	9722                	add	a4,a4,s0
     d0a:	4354                	lw	a3,4(a4)
     d0c:	4410                	lw	a2,8(s0)
     d0e:	854e                	mv	a0,s3
     d10:	0016e693          	ori	a3,a3,1
     d14:	c354                	sw	a3,4(a4)
     d16:	c65c                	sw	a5,12(a2)
     d18:	c790                	sw	a2,8(a5)
     d1a:	21b9                	jal	0x1168
     d1c:	00840513          	addi	a0,s0,8
     d20:	a029                	j	0xd2a
     d22:	47b1                	li	a5,12
     d24:	00f9a023          	sw	a5,0(s3)
     d28:	4501                	li	a0,0
     d2a:	50b2                	lw	ra,44(sp)
     d2c:	5422                	lw	s0,40(sp)
     d2e:	5492                	lw	s1,36(sp)
     d30:	5902                	lw	s2,32(sp)
     d32:	49f2                	lw	s3,28(sp)
     d34:	6145                	addi	sp,sp,48
     d36:	8082                	ret
     d38:	20000613          	li	a2,512
     d3c:	04000593          	li	a1,64
     d40:	03f00813          	li	a6,63
     d44:	b5f5                	j	0xc30
     d46:	00892403          	lw	s0,8(s2)
     d4a:	c85a                	sw	s6,16(sp)
     d4c:	405c                	lw	a5,4(s0)
     d4e:	ffc7fb13          	andi	s6,a5,-4
     d52:	009b6763          	bltu	s6,s1,0xd60
     d56:	409b0733          	sub	a4,s6,s1
     d5a:	47bd                	li	a5,15
     d5c:	12e7c663          	blt	a5,a4,0xe88
     d60:	c266                	sw	s9,4(sp)
     d62:	ca56                	sw	s5,20(sp)
     d64:	d401a703          	lw	a4,-704(gp)
     d68:	d5c1aa83          	lw	s5,-676(gp)
     d6c:	cc52                	sw	s4,24(sp)
     d6e:	c65e                	sw	s7,12(sp)
     d70:	57fd                	li	a5,-1
     d72:	9aa6                	add	s5,s5,s1
     d74:	01640a33          	add	s4,s0,s6
     d78:	2cf70263          	beq	a4,a5,0x103c
     d7c:	6785                	lui	a5,0x1
     d7e:	07bd                	addi	a5,a5,15 # 0x100f
     d80:	9abe                	add	s5,s5,a5
     d82:	77fd                	lui	a5,0xfffff
     d84:	00fafab3          	and	s5,s5,a5
     d88:	85d6                	mv	a1,s5
     d8a:	854e                	mv	a0,s3
     d8c:	399000ef          	jal	0x1924
     d90:	57fd                	li	a5,-1
     d92:	8baa                	mv	s7,a0
     d94:	32f50b63          	beq	a0,a5,0x10ca
     d98:	c462                	sw	s8,8(sp)
     d9a:	0d456563          	bltu	a0,s4,0xe64
     d9e:	eb818c13          	addi	s8,gp,-328
     da2:	000c2583          	lw	a1,0(s8)
     da6:	95d6                	add	a1,a1,s5
     da8:	00bc2023          	sw	a1,0(s8)
     dac:	872e                	mv	a4,a1
     dae:	32aa0263          	beq	s4,a0,0x10d2
     db2:	d401a683          	lw	a3,-704(gp)
     db6:	57fd                	li	a5,-1
     db8:	32f68a63          	beq	a3,a5,0x10ec
     dbc:	414b87b3          	sub	a5,s7,s4
     dc0:	97ba                	add	a5,a5,a4
     dc2:	00fc2023          	sw	a5,0(s8)
     dc6:	007bfc93          	andi	s9,s7,7
     dca:	280c8363          	beqz	s9,0x1050
     dce:	419b8bb3          	sub	s7,s7,s9
     dd2:	6585                	lui	a1,0x1
     dd4:	0ba1                	addi	s7,s7,8
     dd6:	05a1                	addi	a1,a1,8 # 0x1008
     dd8:	9ade                	add	s5,s5,s7
     dda:	419585b3          	sub	a1,a1,s9
     dde:	415585b3          	sub	a1,a1,s5
     de2:	05d2                	slli	a1,a1,0x14
     de4:	0145da13          	srli	s4,a1,0x14
     de8:	85d2                	mv	a1,s4
     dea:	854e                	mv	a0,s3
     dec:	339000ef          	jal	0x1924
     df0:	57fd                	li	a5,-1
     df2:	32f50963          	beq	a0,a5,0x1124
     df6:	41750533          	sub	a0,a0,s7
     dfa:	01450ab3          	add	s5,a0,s4
     dfe:	000c2703          	lw	a4,0(s8)
     e02:	01792423          	sw	s7,8(s2)
     e06:	001ae793          	ori	a5,s5,1
     e0a:	00ea05b3          	add	a1,s4,a4
     e0e:	00fba223          	sw	a5,4(s7)
     e12:	00bc2023          	sw	a1,0(s8)
     e16:	03240563          	beq	s0,s2,0xe40
     e1a:	46bd                	li	a3,15
     e1c:	2566fa63          	bgeu	a3,s6,0x1070
     e20:	4058                	lw	a4,4(s0)
     e22:	ff4b0793          	addi	a5,s6,-12
     e26:	9be1                	andi	a5,a5,-8
     e28:	8b05                	andi	a4,a4,1
     e2a:	8f5d                	or	a4,a4,a5
     e2c:	c058                	sw	a4,4(s0)
     e2e:	4615                	li	a2,5
     e30:	00f40733          	add	a4,s0,a5
     e34:	c350                	sw	a2,4(a4)
     e36:	c710                	sw	a2,8(a4)
     e38:	1ef6e863          	bltu	a3,a5,0x1028
     e3c:	004ba783          	lw	a5,4(s7)
     e40:	d581a683          	lw	a3,-680(gp)
     e44:	00b6f463          	bgeu	a3,a1,0xe4c
     e48:	d4b1ac23          	sw	a1,-680(gp)
     e4c:	d541a683          	lw	a3,-684(gp)
     e50:	00b6f463          	bgeu	a3,a1,0xe58
     e54:	d4b1aa23          	sw	a1,-684(gp)
     e58:	4c22                	lw	s8,8(sp)
     e5a:	845e                	mv	s0,s7
     e5c:	a811                	j	0xe70
     e5e:	00492503          	lw	a0,4(s2)
     e62:	bd99                	j	0xcb8
     e64:	25240b63          	beq	s0,s2,0x10ba
     e68:	00892403          	lw	s0,8(s2)
     e6c:	4c22                	lw	s8,8(sp)
     e6e:	405c                	lw	a5,4(s0)
     e70:	9bf1                	andi	a5,a5,-4
     e72:	40978733          	sub	a4,a5,s1
     e76:	2097e163          	bltu	a5,s1,0x1078
     e7a:	47bd                	li	a5,15
     e7c:	1ee7de63          	bge	a5,a4,0x1078
     e80:	4a62                	lw	s4,24(sp)
     e82:	4ad2                	lw	s5,20(sp)
     e84:	4bb2                	lw	s7,12(sp)
     e86:	4c92                	lw	s9,4(sp)
     e88:	0014e793          	ori	a5,s1,1
     e8c:	c05c                	sw	a5,4(s0)
     e8e:	94a2                	add	s1,s1,s0
     e90:	00992423          	sw	s1,8(s2)
     e94:	00176713          	ori	a4,a4,1
     e98:	854e                	mv	a0,s3
     e9a:	c0d8                	sw	a4,4(s1)
     e9c:	24f1                	jal	0x1168
     e9e:	50b2                	lw	ra,44(sp)
     ea0:	00840513          	addi	a0,s0,8
     ea4:	5422                	lw	s0,40(sp)
     ea6:	4b42                	lw	s6,16(sp)
     ea8:	5492                	lw	s1,36(sp)
     eaa:	5902                	lw	s2,32(sp)
     eac:	49f2                	lw	s3,28(sp)
     eae:	6145                	addi	sp,sp,48
     eb0:	8082                	ret
     eb2:	4410                	lw	a2,8(s0)
     eb4:	97a2                	add	a5,a5,s0
     eb6:	43d8                	lw	a4,4(a5)
     eb8:	c654                	sw	a3,12(a2)
     eba:	c690                	sw	a2,8(a3)
     ebc:	00176713          	ori	a4,a4,1
     ec0:	854e                	mv	a0,s3
     ec2:	c3d8                	sw	a4,4(a5)
     ec4:	2455                	jal	0x1168
     ec6:	50b2                	lw	ra,44(sp)
     ec8:	00840513          	addi	a0,s0,8
     ecc:	5422                	lw	s0,40(sp)
     ece:	5492                	lw	s1,36(sp)
     ed0:	5902                	lw	s2,32(sp)
     ed2:	49f2                	lw	s3,28(sp)
     ed4:	6145                	addi	sp,sp,48
     ed6:	8082                	ret
     ed8:	47c0                	lw	s0,12(a5)
     eda:	0589                	addi	a1,a1,2
     edc:	d88781e3          	beq	a5,s0,0xc5e
     ee0:	b9c5                	j	0xbd0
     ee2:	0097d713          	srli	a4,a5,0x9
     ee6:	4691                	li	a3,4
     ee8:	0ee6f263          	bgeu	a3,a4,0xfcc
     eec:	46d1                	li	a3,20
     eee:	18e6ed63          	bltu	a3,a4,0x1088
     ef2:	05c70613          	addi	a2,a4,92
     ef6:	060e                	slli	a2,a2,0x3
     ef8:	05b70693          	addi	a3,a4,91
     efc:	964a                	add	a2,a2,s2
     efe:	4218                	lw	a4,0(a2)
     f00:	1661                	addi	a2,a2,-8
     f02:	00e61663          	bne	a2,a4,0xf0e
     f06:	aa2d                	j	0x1040
     f08:	4718                	lw	a4,8(a4)
     f0a:	00e60663          	beq	a2,a4,0xf16
     f0e:	4354                	lw	a3,4(a4)
     f10:	9af1                	andi	a3,a3,-4
     f12:	fed7ebe3          	bltu	a5,a3,0xf08
     f16:	4750                	lw	a2,12(a4)
     f18:	c450                	sw	a2,12(s0)
     f1a:	c418                	sw	a4,8(s0)
     f1c:	c600                	sw	s0,8(a2)
     f1e:	c740                	sw	s0,12(a4)
     f20:	bb61                	j	0xcb8
     f22:	4751                	li	a4,20
     f24:	0af77c63          	bgeu	a4,a5,0xfdc
     f28:	05400713          	li	a4,84
     f2c:	16f76a63          	bltu	a4,a5,0x10a0
     f30:	00c4d793          	srli	a5,s1,0xc
     f34:	06f78593          	addi	a1,a5,111 # 0xfffff06f
     f38:	06e78813          	addi	a6,a5,110
     f3c:	00359613          	slli	a2,a1,0x3
     f40:	b9c5                	j	0xc30
     f42:	0e05                	addi	t3,t3,1
     f44:	003e7793          	andi	a5,t3,3
     f48:	0521                	addi	a0,a0,8
     f4a:	c7cd                	beqz	a5,0xff4
     f4c:	455c                	lw	a5,12(a0)
     f4e:	b34d                	j	0xcf0
     f50:	4410                	lw	a2,8(s0)
     f52:	0014e593          	ori	a1,s1,1
     f56:	c04c                	sw	a1,4(s0)
     f58:	c65c                	sw	a5,12(a2)
     f5a:	c790                	sw	a2,8(a5)
     f5c:	94a2                	add	s1,s1,s0
     f5e:	00992a23          	sw	s1,20(s2)
     f62:	00992823          	sw	s1,16(s2)
     f66:	0016e793          	ori	a5,a3,1
     f6a:	9722                	add	a4,a4,s0
     f6c:	0104a623          	sw	a6,12(s1)
     f70:	0104a423          	sw	a6,8(s1)
     f74:	c0dc                	sw	a5,4(s1)
     f76:	854e                	mv	a0,s3
     f78:	c314                	sw	a3,0(a4)
     f7a:	22fd                	jal	0x1168
     f7c:	00840513          	addi	a0,s0,8
     f80:	b36d                	j	0xd2a
     f82:	97a2                	add	a5,a5,s0
     f84:	43d8                	lw	a4,4(a5)
     f86:	854e                	mv	a0,s3
     f88:	00176713          	ori	a4,a4,1
     f8c:	c3d8                	sw	a4,4(a5)
     f8e:	2ae9                	jal	0x1168
     f90:	00840513          	addi	a0,s0,8
     f94:	bb59                	j	0xd2a
     f96:	0034d593          	srli	a1,s1,0x3
     f9a:	00848793          	addi	a5,s1,8
     f9e:	b105                	j	0xbbe
     fa0:	0014e693          	ori	a3,s1,1
     fa4:	c054                	sw	a3,4(s0)
     fa6:	94a2                	add	s1,s1,s0
     fa8:	00992a23          	sw	s1,20(s2)
     fac:	00992823          	sw	s1,16(s2)
     fb0:	00176693          	ori	a3,a4,1
     fb4:	97a2                	add	a5,a5,s0
     fb6:	0104a623          	sw	a6,12(s1)
     fba:	0104a423          	sw	a6,8(s1)
     fbe:	c0d4                	sw	a3,4(s1)
     fc0:	854e                	mv	a0,s3
     fc2:	c398                	sw	a4,0(a5)
     fc4:	2255                	jal	0x1168
     fc6:	00840513          	addi	a0,s0,8
     fca:	b385                	j	0xd2a
     fcc:	0067d713          	srli	a4,a5,0x6
     fd0:	03970613          	addi	a2,a4,57
     fd4:	060e                	slli	a2,a2,0x3
     fd6:	03870693          	addi	a3,a4,56
     fda:	b70d                	j	0xefc
     fdc:	05c78593          	addi	a1,a5,92
     fe0:	05b78813          	addi	a6,a5,91
     fe4:	00359613          	slli	a2,a1,0x3
     fe8:	b1a1                	j	0xc30
     fea:	00832783          	lw	a5,8(t1)
     fee:	15fd                	addi	a1,a1,-1
     ff0:	16679863          	bne	a5,t1,0x1160
     ff4:	0035f793          	andi	a5,a1,3
     ff8:	1361                	addi	t1,t1,-8
     ffa:	fbe5                	bnez	a5,0xfea
     ffc:	00492703          	lw	a4,4(s2)
    1000:	fff64793          	not	a5,a2
    1004:	8ff9                	and	a5,a5,a4
    1006:	00f92223          	sw	a5,4(s2)
    100a:	0606                	slli	a2,a2,0x1
    100c:	d2c7ede3          	bltu	a5,a2,0xd46
    1010:	d2060be3          	beqz	a2,0xd46
    1014:	00f67733          	and	a4,a2,a5
    1018:	e711                	bnez	a4,0x1024
    101a:	0606                	slli	a2,a2,0x1
    101c:	00f67733          	and	a4,a2,a5
    1020:	0e11                	addi	t3,t3,4
    1022:	df65                	beqz	a4,0x101a
    1024:	85f2                	mv	a1,t3
    1026:	b97d                	j	0xce4
    1028:	00840593          	addi	a1,s0,8
    102c:	854e                	mv	a0,s3
    102e:	951ff0ef          	jal	0x97e
    1032:	000c2583          	lw	a1,0(s8)
    1036:	00892b83          	lw	s7,8(s2)
    103a:	b509                	j	0xe3c
    103c:	0ac1                	addi	s5,s5,16
    103e:	b3a9                	j	0xd88
    1040:	8689                	srai	a3,a3,0x2
    1042:	4785                	li	a5,1
    1044:	00d797b3          	sll	a5,a5,a3
    1048:	8d5d                	or	a0,a0,a5
    104a:	00a92223          	sw	a0,4(s2)
    104e:	b5e9                	j	0xf18
    1050:	015b85b3          	add	a1,s7,s5
    1054:	40b005b3          	neg	a1,a1
    1058:	05d2                	slli	a1,a1,0x14
    105a:	0145da13          	srli	s4,a1,0x14
    105e:	85d2                	mv	a1,s4
    1060:	854e                	mv	a0,s3
    1062:	0c3000ef          	jal	0x1924
    1066:	57fd                	li	a5,-1
    1068:	d8f517e3          	bne	a0,a5,0xdf6
    106c:	4a01                	li	s4,0
    106e:	bb41                	j	0xdfe
    1070:	4c22                	lw	s8,8(sp)
    1072:	4785                	li	a5,1
    1074:	00fba223          	sw	a5,4(s7)
    1078:	854e                	mv	a0,s3
    107a:	20fd                	jal	0x1168
    107c:	4a62                	lw	s4,24(sp)
    107e:	4ad2                	lw	s5,20(sp)
    1080:	4b42                	lw	s6,16(sp)
    1082:	4bb2                	lw	s7,12(sp)
    1084:	4c92                	lw	s9,4(sp)
    1086:	b14d                	j	0xd28
    1088:	05400693          	li	a3,84
    108c:	06e6e363          	bltu	a3,a4,0x10f2
    1090:	00c7d713          	srli	a4,a5,0xc
    1094:	06f70613          	addi	a2,a4,111
    1098:	060e                	slli	a2,a2,0x3
    109a:	06e70693          	addi	a3,a4,110
    109e:	bdb9                	j	0xefc
    10a0:	15400713          	li	a4,340
    10a4:	06f76363          	bltu	a4,a5,0x110a
    10a8:	00f4d793          	srli	a5,s1,0xf
    10ac:	07878593          	addi	a1,a5,120
    10b0:	07778813          	addi	a6,a5,119
    10b4:	00359613          	slli	a2,a1,0x3
    10b8:	bea5                	j	0xc30
    10ba:	eb818c13          	addi	s8,gp,-328
    10be:	000c2703          	lw	a4,0(s8)
    10c2:	9756                	add	a4,a4,s5
    10c4:	00ec2023          	sw	a4,0(s8)
    10c8:	b1ed                	j	0xdb2
    10ca:	00892403          	lw	s0,8(s2)
    10ce:	405c                	lw	a5,4(s0)
    10d0:	b345                	j	0xe70
    10d2:	01451793          	slli	a5,a0,0x14
    10d6:	cc079ee3          	bnez	a5,0xdb2
    10da:	00892b83          	lw	s7,8(s2)
    10de:	015b07b3          	add	a5,s6,s5
    10e2:	0017e793          	ori	a5,a5,1
    10e6:	00fba223          	sw	a5,4(s7)
    10ea:	bb99                	j	0xe40
    10ec:	d571a023          	sw	s7,-704(gp)
    10f0:	b9d9                	j	0xdc6
    10f2:	15400693          	li	a3,340
    10f6:	02e6ed63          	bltu	a3,a4,0x1130
    10fa:	00f7d713          	srli	a4,a5,0xf
    10fe:	07870613          	addi	a2,a4,120
    1102:	060e                	slli	a2,a2,0x3
    1104:	07770693          	addi	a3,a4,119
    1108:	bbd5                	j	0xefc
    110a:	55400713          	li	a4,1364
    110e:	02f76d63          	bltu	a4,a5,0x1148
    1112:	0124d793          	srli	a5,s1,0x12
    1116:	07d78593          	addi	a1,a5,125
    111a:	07c78813          	addi	a6,a5,124
    111e:	00359613          	slli	a2,a1,0x3
    1122:	b639                	j	0xc30
    1124:	1ce1                	addi	s9,s9,-8
    1126:	9ae6                	add	s5,s5,s9
    1128:	417a8ab3          	sub	s5,s5,s7
    112c:	4a01                	li	s4,0
    112e:	b9c1                	j	0xdfe
    1130:	55400693          	li	a3,1364
    1134:	02e6e163          	bltu	a3,a4,0x1156
    1138:	0127d713          	srli	a4,a5,0x12
    113c:	07d70613          	addi	a2,a4,125
    1140:	060e                	slli	a2,a2,0x3
    1142:	07c70693          	addi	a3,a4,124
    1146:	bb5d                	j	0xefc
    1148:	3f800613          	li	a2,1016
    114c:	07f00593          	li	a1,127
    1150:	07e00813          	li	a6,126
    1154:	bcf1                	j	0xc30
    1156:	3f800613          	li	a2,1016
    115a:	07e00693          	li	a3,126
    115e:	bb79                	j	0xefc
    1160:	00492783          	lw	a5,4(s2)
    1164:	b55d                	j	0x100a
    1166:	8082                	ret
    1168:	8082                	ret
    116a:	1141                	addi	sp,sp,-16
    116c:	c606                	sw	ra,12(sp)
    116e:	c04a                	sw	s2,0(sp)
    1170:	cd89                	beqz	a1,0x118a
    1172:	c422                	sw	s0,8(sp)
    1174:	c226                	sw	s1,4(sp)
    1176:	842e                	mv	s0,a1
    1178:	84aa                	mv	s1,a0
    117a:	c119                	beqz	a0,0x1180
    117c:	595c                	lw	a5,52(a0)
    117e:	c7d1                	beqz	a5,0x120a
    1180:	00c41783          	lh	a5,12(s0)
    1184:	eb89                	bnez	a5,0x1196
    1186:	4422                	lw	s0,8(sp)
    1188:	4492                	lw	s1,4(sp)
    118a:	40b2                	lw	ra,12(sp)
    118c:	4901                	li	s2,0
    118e:	854a                	mv	a0,s2
    1190:	4902                	lw	s2,0(sp)
    1192:	0141                	addi	sp,sp,16
    1194:	8082                	ret
    1196:	85a2                	mv	a1,s0
    1198:	8526                	mv	a0,s1
    119a:	28bd                	jal	0x1218
    119c:	545c                	lw	a5,44(s0)
    119e:	892a                	mv	s2,a0
    11a0:	c791                	beqz	a5,0x11ac
    11a2:	4c4c                	lw	a1,28(s0)
    11a4:	8526                	mv	a0,s1
    11a6:	9782                	jalr	a5
    11a8:	04054663          	bltz	a0,0x11f4
    11ac:	00c45783          	lhu	a5,12(s0)
    11b0:	0807f793          	andi	a5,a5,128
    11b4:	e7b1                	bnez	a5,0x1200
    11b6:	580c                	lw	a1,48(s0)
    11b8:	c991                	beqz	a1,0x11cc
    11ba:	04040793          	addi	a5,s0,64
    11be:	00f58563          	beq	a1,a5,0x11c8
    11c2:	8526                	mv	a0,s1
    11c4:	fbaff0ef          	jal	0x97e
    11c8:	02042823          	sw	zero,48(s0)
    11cc:	406c                	lw	a1,68(s0)
    11ce:	c591                	beqz	a1,0x11da
    11d0:	8526                	mv	a0,s1
    11d2:	facff0ef          	jal	0x97e
    11d6:	04042223          	sw	zero,68(s0)
    11da:	93cff0ef          	jal	0x316
    11de:	00041623          	sh	zero,12(s0)
    11e2:	936ff0ef          	jal	0x318
    11e6:	40b2                	lw	ra,12(sp)
    11e8:	4422                	lw	s0,8(sp)
    11ea:	4492                	lw	s1,4(sp)
    11ec:	854a                	mv	a0,s2
    11ee:	4902                	lw	s2,0(sp)
    11f0:	0141                	addi	sp,sp,16
    11f2:	8082                	ret
    11f4:	00c45783          	lhu	a5,12(s0)
    11f8:	597d                	li	s2,-1
    11fa:	0807f793          	andi	a5,a5,128
    11fe:	dfc5                	beqz	a5,0x11b6
    1200:	480c                	lw	a1,16(s0)
    1202:	8526                	mv	a0,s1
    1204:	f7aff0ef          	jal	0x97e
    1208:	b77d                	j	0x11b6
    120a:	8f6ff0ef          	jal	0x300
    120e:	bf8d                	j	0x1180
    1210:	85aa                	mv	a1,a0
    1212:	d3c1a503          	lw	a0,-708(gp)
    1216:	bf91                	j	0x116a
    1218:	00c59703          	lh	a4,12(a1)
    121c:	1101                	addi	sp,sp,-32
    121e:	cc22                	sw	s0,24(sp)
    1220:	c64e                	sw	s3,12(sp)
    1222:	ce06                	sw	ra,28(sp)
    1224:	00877793          	andi	a5,a4,8
    1228:	842e                	mv	s0,a1
    122a:	89aa                	mv	s3,a0
    122c:	e7e1                	bnez	a5,0x12f4
    122e:	6785                	lui	a5,0x1
    1230:	80078793          	addi	a5,a5,-2048 # 0x800
    1234:	41d4                	lw	a3,4(a1)
    1236:	8fd9                	or	a5,a5,a4
    1238:	00f59623          	sh	a5,12(a1)
    123c:	10d05963          	blez	a3,0x134e
    1240:	02842803          	lw	a6,40(s0)
    1244:	0a080263          	beqz	a6,0x12e8
    1248:	ca26                	sw	s1,20(sp)
    124a:	01371693          	slli	a3,a4,0x13
    124e:	0009a483          	lw	s1,0(s3)
    1252:	0009a023          	sw	zero,0(s3)
    1256:	1006c363          	bltz	a3,0x135c
    125a:	4c4c                	lw	a1,28(s0)
    125c:	4601                	li	a2,0
    125e:	4685                	li	a3,1
    1260:	854e                	mv	a0,s3
    1262:	9802                	jalr	a6
    1264:	57fd                	li	a5,-1
    1266:	862a                	mv	a2,a0
    1268:	12f50163          	beq	a0,a5,0x138a
    126c:	00c41783          	lh	a5,12(s0)
    1270:	02842803          	lw	a6,40(s0)
    1274:	8b91                	andi	a5,a5,4
    1276:	c799                	beqz	a5,0x1284
    1278:	4058                	lw	a4,4(s0)
    127a:	581c                	lw	a5,48(s0)
    127c:	8e19                	sub	a2,a2,a4
    127e:	c399                	beqz	a5,0x1284
    1280:	5c5c                	lw	a5,60(s0)
    1282:	8e1d                	sub	a2,a2,a5
    1284:	4c4c                	lw	a1,28(s0)
    1286:	4681                	li	a3,0
    1288:	854e                	mv	a0,s3
    128a:	9802                	jalr	a6
    128c:	577d                	li	a4,-1
    128e:	00c41783          	lh	a5,12(s0)
    1292:	0ce51763          	bne	a0,a4,0x1360
    1296:	0009a683          	lw	a3,0(s3)
    129a:	4775                	li	a4,29
    129c:	10d76363          	bltu	a4,a3,0x13a2
    12a0:	20400737          	lui	a4,0x20400
    12a4:	0705                	addi	a4,a4,1 # 0x20400001
    12a6:	00d75733          	srl	a4,a4,a3
    12aa:	8b05                	andi	a4,a4,1
    12ac:	cb7d                	beqz	a4,0x13a2
    12ae:	4810                	lw	a2,16(s0)
    12b0:	777d                	lui	a4,0xfffff
    12b2:	7ff70713          	addi	a4,a4,2047 # 0xfffff7ff
    12b6:	8f7d                	and	a4,a4,a5
    12b8:	00e41623          	sh	a4,12(s0)
    12bc:	00042223          	sw	zero,4(s0)
    12c0:	c010                	sw	a2,0(s0)
    12c2:	01379713          	slli	a4,a5,0x13
    12c6:	00075363          	bgez	a4,0x12cc
    12ca:	cacd                	beqz	a3,0x137c
    12cc:	580c                	lw	a1,48(s0)
    12ce:	0099a023          	sw	s1,0(s3)
    12d2:	c9d5                	beqz	a1,0x1386
    12d4:	04040793          	addi	a5,s0,64
    12d8:	00f58563          	beq	a1,a5,0x12e2
    12dc:	854e                	mv	a0,s3
    12de:	ea0ff0ef          	jal	0x97e
    12e2:	44d2                	lw	s1,20(sp)
    12e4:	02042823          	sw	zero,48(s0)
    12e8:	40f2                	lw	ra,28(sp)
    12ea:	4462                	lw	s0,24(sp)
    12ec:	49b2                	lw	s3,12(sp)
    12ee:	4501                	li	a0,0
    12f0:	6105                	addi	sp,sp,32
    12f2:	8082                	ret
    12f4:	c84a                	sw	s2,16(sp)
    12f6:	0105a903          	lw	s2,16(a1)
    12fa:	04090f63          	beqz	s2,0x1358
    12fe:	ca26                	sw	s1,20(sp)
    1300:	4184                	lw	s1,0(a1)
    1302:	8b0d                	andi	a4,a4,3
    1304:	0125a023          	sw	s2,0(a1)
    1308:	412484b3          	sub	s1,s1,s2
    130c:	4781                	li	a5,0
    130e:	e311                	bnez	a4,0x1312
    1310:	49dc                	lw	a5,20(a1)
    1312:	c41c                	sw	a5,8(s0)
    1314:	00904663          	bgtz	s1,0x1320
    1318:	a83d                	j	0x1356
    131a:	992a                	add	s2,s2,a0
    131c:	02905d63          	blez	s1,0x1356
    1320:	505c                	lw	a5,36(s0)
    1322:	4c4c                	lw	a1,28(s0)
    1324:	86a6                	mv	a3,s1
    1326:	864a                	mv	a2,s2
    1328:	854e                	mv	a0,s3
    132a:	9782                	jalr	a5
    132c:	8c89                	sub	s1,s1,a0
    132e:	fea046e3          	bgtz	a0,0x131a
    1332:	00c41783          	lh	a5,12(s0)
    1336:	4942                	lw	s2,16(sp)
    1338:	0407e793          	ori	a5,a5,64
    133c:	40f2                	lw	ra,28(sp)
    133e:	00f41623          	sh	a5,12(s0)
    1342:	4462                	lw	s0,24(sp)
    1344:	44d2                	lw	s1,20(sp)
    1346:	49b2                	lw	s3,12(sp)
    1348:	557d                	li	a0,-1
    134a:	6105                	addi	sp,sp,32
    134c:	8082                	ret
    134e:	5dd4                	lw	a3,60(a1)
    1350:	eed048e3          	bgtz	a3,0x1240
    1354:	bf51                	j	0x12e8
    1356:	44d2                	lw	s1,20(sp)
    1358:	4942                	lw	s2,16(sp)
    135a:	b779                	j	0x12e8
    135c:	4830                	lw	a2,80(s0)
    135e:	bf19                	j	0x1274
    1360:	4814                	lw	a3,16(s0)
    1362:	777d                	lui	a4,0xfffff
    1364:	7ff70713          	addi	a4,a4,2047 # 0xfffff7ff
    1368:	8f7d                	and	a4,a4,a5
    136a:	00e41623          	sh	a4,12(s0)
    136e:	00042223          	sw	zero,4(s0)
    1372:	c014                	sw	a3,0(s0)
    1374:	01379713          	slli	a4,a5,0x13
    1378:	f4075ae3          	bgez	a4,0x12cc
    137c:	580c                	lw	a1,48(s0)
    137e:	c828                	sw	a0,80(s0)
    1380:	0099a023          	sw	s1,0(s3)
    1384:	f9a1                	bnez	a1,0x12d4
    1386:	44d2                	lw	s1,20(sp)
    1388:	b785                	j	0x12e8
    138a:	0009a783          	lw	a5,0(s3)
    138e:	ec078fe3          	beqz	a5,0x126c
    1392:	4775                	li	a4,29
    1394:	00e78a63          	beq	a5,a4,0x13a8
    1398:	4759                	li	a4,22
    139a:	00e78763          	beq	a5,a4,0x13a8
    139e:	00c41783          	lh	a5,12(s0)
    13a2:	0407e793          	ori	a5,a5,64
    13a6:	bf59                	j	0x133c
    13a8:	0099a023          	sw	s1,0(s3)
    13ac:	44d2                	lw	s1,20(sp)
    13ae:	bf2d                	j	0x12e8
    13b0:	1101                	addi	sp,sp,-32
    13b2:	cc22                	sw	s0,24(sp)
    13b4:	ce06                	sw	ra,28(sp)
    13b6:	842a                	mv	s0,a0
    13b8:	c119                	beqz	a0,0x13be
    13ba:	595c                	lw	a5,52(a0)
    13bc:	cf91                	beqz	a5,0x13d8
    13be:	00c59783          	lh	a5,12(a1)
    13c2:	e791                	bnez	a5,0x13ce
    13c4:	40f2                	lw	ra,28(sp)
    13c6:	4462                	lw	s0,24(sp)
    13c8:	4501                	li	a0,0
    13ca:	6105                	addi	sp,sp,32
    13cc:	8082                	ret
    13ce:	8522                	mv	a0,s0
    13d0:	4462                	lw	s0,24(sp)
    13d2:	40f2                	lw	ra,28(sp)
    13d4:	6105                	addi	sp,sp,32
    13d6:	b589                	j	0x1218
    13d8:	c62e                	sw	a1,12(sp)
    13da:	f27fe0ef          	jal	0x300
    13de:	45b2                	lw	a1,12(sp)
    13e0:	bff9                	j	0x13be
    13e2:	cd05                	beqz	a0,0x141a
    13e4:	85aa                	mv	a1,a0
    13e6:	d3c1a503          	lw	a0,-708(gp)
    13ea:	c119                	beqz	a0,0x13f0
    13ec:	595c                	lw	a5,52(a0)
    13ee:	c799                	beqz	a5,0x13fc
    13f0:	00c59783          	lh	a5,12(a1)
    13f4:	e399                	bnez	a5,0x13fa
    13f6:	4501                	li	a0,0
    13f8:	8082                	ret
    13fa:	bd39                	j	0x1218
    13fc:	1101                	addi	sp,sp,-32
    13fe:	c62e                	sw	a1,12(sp)
    1400:	c42a                	sw	a0,8(sp)
    1402:	ce06                	sw	ra,28(sp)
    1404:	efdfe0ef          	jal	0x300
    1408:	45b2                	lw	a1,12(sp)
    140a:	4522                	lw	a0,8(sp)
    140c:	00c59783          	lh	a5,12(a1)
    1410:	e385                	bnez	a5,0x1430
    1412:	40f2                	lw	ra,28(sp)
    1414:	4501                	li	a0,0
    1416:	6105                	addi	sp,sp,32
    1418:	8082                	ret
    141a:	664d                	lui	a2,0x13
    141c:	65c5                	lui	a1,0x11
    141e:	654d                	lui	a0,0x13
    1420:	48060613          	addi	a2,a2,1152 # 0x13480
    1424:	46458593          	addi	a1,a1,1124 # 0x11464
    1428:	49050513          	addi	a0,a0,1168 # 0x13490
    142c:	f0ffe06f          	j	0x33a
    1430:	40f2                	lw	ra,28(sp)
    1432:	6105                	addi	sp,sp,32
    1434:	b3d5                	j	0x1218
    1436:	461c                	lw	a5,8(a2)
    1438:	18078a63          	beqz	a5,0x15cc
    143c:	00c59703          	lh	a4,12(a1)
    1440:	7179                	addi	sp,sp,-48
    1442:	d422                	sw	s0,40(sp)
    1444:	cc52                	sw	s4,24(sp)
    1446:	ca56                	sw	s5,20(sp)
    1448:	d606                	sw	ra,44(sp)
    144a:	00877793          	andi	a5,a4,8
    144e:	8a32                	mv	s4,a2
    1450:	8aaa                	mv	s5,a0
    1452:	842e                	mv	s0,a1
    1454:	c7b5                	beqz	a5,0x14c0
    1456:	499c                	lw	a5,16(a1)
    1458:	c7a5                	beqz	a5,0x14c0
    145a:	d226                	sw	s1,36(sp)
    145c:	d04a                	sw	s2,32(sp)
    145e:	ce4e                	sw	s3,28(sp)
    1460:	c85a                	sw	s6,16(sp)
    1462:	00277793          	andi	a5,a4,2
    1466:	000a2483          	lw	s1,0(s4)
    146a:	cbbd                	beqz	a5,0x14e0
    146c:	80000b37          	lui	s6,0x80000
    1470:	c00b0b13          	addi	s6,s6,-1024 # 0x7ffffc00
    1474:	4981                	li	s3,0
    1476:	4901                	li	s2,0
    1478:	864e                	mv	a2,s3
    147a:	8556                	mv	a0,s5
    147c:	14090263          	beqz	s2,0x15c0
    1480:	800007b7          	lui	a5,0x80000
    1484:	86ca                	mv	a3,s2
    1486:	012b7463          	bgeu	s6,s2,0x148e
    148a:	c0078693          	addi	a3,a5,-1024 # 0x7ffffc00
    148e:	505c                	lw	a5,36(s0)
    1490:	4c4c                	lw	a1,28(s0)
    1492:	9782                	jalr	a5
    1494:	2ca05463          	blez	a0,0x175c
    1498:	008a2783          	lw	a5,8(s4)
    149c:	99aa                	add	s3,s3,a0
    149e:	40a90933          	sub	s2,s2,a0
    14a2:	8f89                	sub	a5,a5,a0
    14a4:	00fa2423          	sw	a5,8(s4)
    14a8:	fbe1                	bnez	a5,0x1478
    14aa:	5492                	lw	s1,36(sp)
    14ac:	5902                	lw	s2,32(sp)
    14ae:	49f2                	lw	s3,28(sp)
    14b0:	4b42                	lw	s6,16(sp)
    14b2:	4501                	li	a0,0
    14b4:	50b2                	lw	ra,44(sp)
    14b6:	5422                	lw	s0,40(sp)
    14b8:	4a62                	lw	s4,24(sp)
    14ba:	4ad2                	lw	s5,20(sp)
    14bc:	6145                	addi	sp,sp,48
    14be:	8082                	ret
    14c0:	85a2                	mv	a1,s0
    14c2:	8556                	mv	a0,s5
    14c4:	2c45                	jal	0x1774
    14c6:	1a051f63          	bnez	a0,0x1684
    14ca:	00c41703          	lh	a4,12(s0)
    14ce:	d226                	sw	s1,36(sp)
    14d0:	d04a                	sw	s2,32(sp)
    14d2:	ce4e                	sw	s3,28(sp)
    14d4:	c85a                	sw	s6,16(sp)
    14d6:	00277793          	andi	a5,a4,2
    14da:	000a2483          	lw	s1,0(s4)
    14de:	f7d9                	bnez	a5,0x146c
    14e0:	c65e                	sw	s7,12(sp)
    14e2:	c462                	sw	s8,8(sp)
    14e4:	00177793          	andi	a5,a4,1
    14e8:	e7e5                	bnez	a5,0x15d0
    14ea:	80000bb7          	lui	s7,0x80000
    14ee:	c266                	sw	s9,4(sp)
    14f0:	1bfd                	addi	s7,s7,-1 # 0x7fffffff
    14f2:	4b01                	li	s6,0
    14f4:	4901                	li	s2,0
    14f6:	0a090f63          	beqz	s2,0x15b4
    14fa:	20077793          	andi	a5,a4,512
    14fe:	4008                	lw	a0,0(s0)
    1500:	00842c03          	lw	s8,8(s0)
    1504:	18078263          	beqz	a5,0x1688
    1508:	8ce2                	mv	s9,s8
    150a:	1f896c63          	bltu	s2,s8,0x1702
    150e:	48077793          	andi	a5,a4,1152
    1512:	cba5                	beqz	a5,0x1582
    1514:	4854                	lw	a3,20(s0)
    1516:	480c                	lw	a1,16(s0)
    1518:	00169793          	slli	a5,a3,0x1
    151c:	97b6                	add	a5,a5,a3
    151e:	01f7d993          	srli	s3,a5,0x1f
    1522:	40b50c33          	sub	s8,a0,a1
    1526:	99be                	add	s3,s3,a5
    1528:	001c0793          	addi	a5,s8,1
    152c:	4019d993          	srai	s3,s3,0x1
    1530:	97ca                	add	a5,a5,s2
    1532:	864e                	mv	a2,s3
    1534:	00f9f463          	bgeu	s3,a5,0x153c
    1538:	89be                	mv	s3,a5
    153a:	863e                	mv	a2,a5
    153c:	40077713          	andi	a4,a4,1024
    1540:	1e070663          	beqz	a4,0x172c
    1544:	85b2                	mv	a1,a2
    1546:	8556                	mv	a0,s5
    1548:	e52ff0ef          	jal	0xb9a
    154c:	8caa                	mv	s9,a0
    154e:	20050a63          	beqz	a0,0x1762
    1552:	480c                	lw	a1,16(s0)
    1554:	8662                	mv	a2,s8
    1556:	2b11                	jal	0x1a6a
    1558:	00c45783          	lhu	a5,12(s0)
    155c:	b7f7f793          	andi	a5,a5,-1153
    1560:	0807e793          	ori	a5,a5,128
    1564:	00f41623          	sh	a5,12(s0)
    1568:	018c8533          	add	a0,s9,s8
    156c:	41898c33          	sub	s8,s3,s8
    1570:	01942823          	sw	s9,16(s0)
    1574:	01842423          	sw	s8,8(s0)
    1578:	c008                	sw	a0,0(s0)
    157a:	01342a23          	sw	s3,20(s0)
    157e:	8c4a                	mv	s8,s2
    1580:	8cca                	mv	s9,s2
    1582:	85da                	mv	a1,s6
    1584:	8666                	mv	a2,s9
    1586:	2121                	jal	0x198e
    1588:	401c                	lw	a5,0(s0)
    158a:	4418                	lw	a4,8(s0)
    158c:	89ca                	mv	s3,s2
    158e:	97e6                	add	a5,a5,s9
    1590:	c01c                	sw	a5,0(s0)
    1592:	008a2783          	lw	a5,8(s4)
    1596:	41870733          	sub	a4,a4,s8
    159a:	c418                	sw	a4,8(s0)
    159c:	413787b3          	sub	a5,a5,s3
    15a0:	00fa2423          	sw	a5,8(s4)
    15a4:	4901                	li	s2,0
    15a6:	9b4e                	add	s6,s6,s3
    15a8:	12078063          	beqz	a5,0x16c8
    15ac:	00c41703          	lh	a4,12(s0)
    15b0:	f40915e3          	bnez	s2,0x14fa
    15b4:	0004ab03          	lw	s6,0(s1)
    15b8:	0044a903          	lw	s2,4(s1)
    15bc:	04a1                	addi	s1,s1,8
    15be:	bf25                	j	0x14f6
    15c0:	0004a983          	lw	s3,0(s1)
    15c4:	0044a903          	lw	s2,4(s1)
    15c8:	04a1                	addi	s1,s1,8
    15ca:	b57d                	j	0x1478
    15cc:	4501                	li	a0,0
    15ce:	8082                	ret
    15d0:	4b01                	li	s6,0
    15d2:	4501                	li	a0,0
    15d4:	4c01                	li	s8,0
    15d6:	4981                	li	s3,0
    15d8:	04098e63          	beqz	s3,0x1634
    15dc:	c525                	beqz	a0,0x1644
    15de:	87da                	mv	a5,s6
    15e0:	8bce                	mv	s7,s3
    15e2:	0137f363          	bgeu	a5,s3,0x15e8
    15e6:	8bbe                	mv	s7,a5
    15e8:	4008                	lw	a0,0(s0)
    15ea:	481c                	lw	a5,16(s0)
    15ec:	4854                	lw	a3,20(s0)
    15ee:	00a7f763          	bgeu	a5,a0,0x15fc
    15f2:	00842903          	lw	s2,8(s0)
    15f6:	9936                	add	s2,s2,a3
    15f8:	07794063          	blt	s2,s7,0x1658
    15fc:	10dbcc63          	blt	s7,a3,0x1714
    1600:	505c                	lw	a5,36(s0)
    1602:	4c4c                	lw	a1,28(s0)
    1604:	8662                	mv	a2,s8
    1606:	8556                	mv	a0,s5
    1608:	9782                	jalr	a5
    160a:	892a                	mv	s2,a0
    160c:	06a05063          	blez	a0,0x166c
    1610:	412b0b33          	sub	s6,s6,s2
    1614:	4505                	li	a0,1
    1616:	0e0b0963          	beqz	s6,0x1708
    161a:	008a2783          	lw	a5,8(s4)
    161e:	9c4a                	add	s8,s8,s2
    1620:	412989b3          	sub	s3,s3,s2
    1624:	412787b3          	sub	a5,a5,s2
    1628:	00fa2423          	sw	a5,8(s4)
    162c:	f7d5                	bnez	a5,0x15d8
    162e:	4bb2                	lw	s7,12(sp)
    1630:	4c22                	lw	s8,8(sp)
    1632:	bda5                	j	0x14aa
    1634:	0044a983          	lw	s3,4(s1)
    1638:	87a6                	mv	a5,s1
    163a:	04a1                	addi	s1,s1,8
    163c:	fe098ce3          	beqz	s3,0x1634
    1640:	0007ac03          	lw	s8,0(a5)
    1644:	864e                	mv	a2,s3
    1646:	45a9                	li	a1,10
    1648:	8562                	mv	a0,s8
    164a:	2499                	jal	0x1890
    164c:	10050463          	beqz	a0,0x1754
    1650:	0505                	addi	a0,a0,1
    1652:	41850b33          	sub	s6,a0,s8
    1656:	b761                	j	0x15de
    1658:	85e2                	mv	a1,s8
    165a:	864a                	mv	a2,s2
    165c:	2e0d                	jal	0x198e
    165e:	401c                	lw	a5,0(s0)
    1660:	85a2                	mv	a1,s0
    1662:	8556                	mv	a0,s5
    1664:	97ca                	add	a5,a5,s2
    1666:	c01c                	sw	a5,0(s0)
    1668:	33a1                	jal	0x13b0
    166a:	d15d                	beqz	a0,0x1610
    166c:	00c41783          	lh	a5,12(s0)
    1670:	4bb2                	lw	s7,12(sp)
    1672:	4c22                	lw	s8,8(sp)
    1674:	5492                	lw	s1,36(sp)
    1676:	5902                	lw	s2,32(sp)
    1678:	49f2                	lw	s3,28(sp)
    167a:	4b42                	lw	s6,16(sp)
    167c:	0407e793          	ori	a5,a5,64
    1680:	00f41623          	sh	a5,12(s0)
    1684:	557d                	li	a0,-1
    1686:	b53d                	j	0x14b4
    1688:	481c                	lw	a5,16(s0)
    168a:	04a7e363          	bltu	a5,a0,0x16d0
    168e:	485c                	lw	a5,20(s0)
    1690:	04f96063          	bltu	s2,a5,0x16d0
    1694:	86ca                	mv	a3,s2
    1696:	012bf363          	bgeu	s7,s2,0x169c
    169a:	86de                	mv	a3,s7
    169c:	02f6e7b3          	rem	a5,a3,a5
    16a0:	5058                	lw	a4,36(s0)
    16a2:	4c4c                	lw	a1,28(s0)
    16a4:	865a                	mv	a2,s6
    16a6:	8556                	mv	a0,s5
    16a8:	8e9d                	sub	a3,a3,a5
    16aa:	9702                	jalr	a4
    16ac:	89aa                	mv	s3,a0
    16ae:	04a05463          	blez	a0,0x16f6
    16b2:	008a2783          	lw	a5,8(s4)
    16b6:	41390933          	sub	s2,s2,s3
    16ba:	9b4e                	add	s6,s6,s3
    16bc:	413787b3          	sub	a5,a5,s3
    16c0:	00fa2423          	sw	a5,8(s4)
    16c4:	ee0794e3          	bnez	a5,0x15ac
    16c8:	4bb2                	lw	s7,12(sp)
    16ca:	4c22                	lw	s8,8(sp)
    16cc:	4c92                	lw	s9,4(sp)
    16ce:	bbf1                	j	0x14aa
    16d0:	89e2                	mv	s3,s8
    16d2:	01897363          	bgeu	s2,s8,0x16d8
    16d6:	89ca                	mv	s3,s2
    16d8:	864e                	mv	a2,s3
    16da:	85da                	mv	a1,s6
    16dc:	2c4d                	jal	0x198e
    16de:	4018                	lw	a4,0(s0)
    16e0:	441c                	lw	a5,8(s0)
    16e2:	974e                	add	a4,a4,s3
    16e4:	413787b3          	sub	a5,a5,s3
    16e8:	c018                	sw	a4,0(s0)
    16ea:	c41c                	sw	a5,8(s0)
    16ec:	f3f9                	bnez	a5,0x16b2
    16ee:	85a2                	mv	a1,s0
    16f0:	8556                	mv	a0,s5
    16f2:	397d                	jal	0x13b0
    16f4:	dd5d                	beqz	a0,0x16b2
    16f6:	00c41783          	lh	a5,12(s0)
    16fa:	4bb2                	lw	s7,12(sp)
    16fc:	4c22                	lw	s8,8(sp)
    16fe:	4c92                	lw	s9,4(sp)
    1700:	bf95                	j	0x1674
    1702:	8c4a                	mv	s8,s2
    1704:	8cca                	mv	s9,s2
    1706:	bdb5                	j	0x1582
    1708:	85a2                	mv	a1,s0
    170a:	8556                	mv	a0,s5
    170c:	3155                	jal	0x13b0
    170e:	f00506e3          	beqz	a0,0x161a
    1712:	bfa9                	j	0x166c
    1714:	865e                	mv	a2,s7
    1716:	85e2                	mv	a1,s8
    1718:	2c9d                	jal	0x198e
    171a:	4418                	lw	a4,8(s0)
    171c:	401c                	lw	a5,0(s0)
    171e:	895e                	mv	s2,s7
    1720:	41770733          	sub	a4,a4,s7
    1724:	97de                	add	a5,a5,s7
    1726:	c418                	sw	a4,8(s0)
    1728:	c01c                	sw	a5,0(s0)
    172a:	b5dd                	j	0x1610
    172c:	8556                	mv	a0,s5
    172e:	2951                	jal	0x1bc2
    1730:	8caa                	mv	s9,a0
    1732:	e2051be3          	bnez	a0,0x1568
    1736:	480c                	lw	a1,16(s0)
    1738:	8556                	mv	a0,s5
    173a:	a44ff0ef          	jal	0x97e
    173e:	00c41783          	lh	a5,12(s0)
    1742:	4731                	li	a4,12
    1744:	4bb2                	lw	s7,12(sp)
    1746:	4c22                	lw	s8,8(sp)
    1748:	4c92                	lw	s9,4(sp)
    174a:	00eaa023          	sw	a4,0(s5)
    174e:	f7f7f793          	andi	a5,a5,-129
    1752:	b70d                	j	0x1674
    1754:	00198793          	addi	a5,s3,1
    1758:	8b3e                	mv	s6,a5
    175a:	b559                	j	0x15e0
    175c:	00c41783          	lh	a5,12(s0)
    1760:	bf11                	j	0x1674
    1762:	47b1                	li	a5,12
    1764:	00faa023          	sw	a5,0(s5)
    1768:	4bb2                	lw	s7,12(sp)
    176a:	00c41783          	lh	a5,12(s0)
    176e:	4c22                	lw	s8,8(sp)
    1770:	4c92                	lw	s9,4(sp)
    1772:	b709                	j	0x1674
    1774:	d3c1a783          	lw	a5,-708(gp)
    1778:	1141                	addi	sp,sp,-16
    177a:	c422                	sw	s0,8(sp)
    177c:	c226                	sw	s1,4(sp)
    177e:	c606                	sw	ra,12(sp)
    1780:	84aa                	mv	s1,a0
    1782:	842e                	mv	s0,a1
    1784:	c399                	beqz	a5,0x178a
    1786:	5bd8                	lw	a4,52(a5)
    1788:	cb61                	beqz	a4,0x1858
    178a:	00c41783          	lh	a5,12(s0)
    178e:	0087f713          	andi	a4,a5,8
    1792:	c315                	beqz	a4,0x17b6
    1794:	4818                	lw	a4,16(s0)
    1796:	cf05                	beqz	a4,0x17ce
    1798:	0017f713          	andi	a4,a5,1
    179c:	c32d                	beqz	a4,0x17fe
    179e:	485c                	lw	a5,20(s0)
    17a0:	00042423          	sw	zero,8(s0)
    17a4:	40f007b3          	neg	a5,a5
    17a8:	cc1c                	sw	a5,24(s0)
    17aa:	4501                	li	a0,0
    17ac:	40b2                	lw	ra,12(sp)
    17ae:	4422                	lw	s0,8(sp)
    17b0:	4492                	lw	s1,4(sp)
    17b2:	0141                	addi	sp,sp,16
    17b4:	8082                	ret
    17b6:	0107f713          	andi	a4,a5,16
    17ba:	c379                	beqz	a4,0x1880
    17bc:	0047f713          	andi	a4,a5,4
    17c0:	e721                	bnez	a4,0x1808
    17c2:	4818                	lw	a4,16(s0)
    17c4:	0087e793          	ori	a5,a5,8
    17c8:	00f41623          	sh	a5,12(s0)
    17cc:	f771                	bnez	a4,0x1798
    17ce:	2807f693          	andi	a3,a5,640
    17d2:	20000613          	li	a2,512
    17d6:	06c69063          	bne	a3,a2,0x1836
    17da:	0017f693          	andi	a3,a5,1
    17de:	c2c9                	beqz	a3,0x1860
    17e0:	4858                	lw	a4,20(s0)
    17e2:	00042423          	sw	zero,8(s0)
    17e6:	40e00733          	neg	a4,a4
    17ea:	cc18                	sw	a4,24(s0)
    17ec:	0807f713          	andi	a4,a5,128
    17f0:	df4d                	beqz	a4,0x17aa
    17f2:	0407e793          	ori	a5,a5,64
    17f6:	00f41623          	sh	a5,12(s0)
    17fa:	557d                	li	a0,-1
    17fc:	bf45                	j	0x17ac
    17fe:	8b89                	andi	a5,a5,2
    1800:	eb85                	bnez	a5,0x1830
    1802:	485c                	lw	a5,20(s0)
    1804:	c41c                	sw	a5,8(s0)
    1806:	b755                	j	0x17aa
    1808:	580c                	lw	a1,48(s0)
    180a:	cd81                	beqz	a1,0x1822
    180c:	04040713          	addi	a4,s0,64
    1810:	00e58763          	beq	a1,a4,0x181e
    1814:	8526                	mv	a0,s1
    1816:	968ff0ef          	jal	0x97e
    181a:	00c41783          	lh	a5,12(s0)
    181e:	02042823          	sw	zero,48(s0)
    1822:	4818                	lw	a4,16(s0)
    1824:	fdb7f793          	andi	a5,a5,-37
    1828:	00042223          	sw	zero,4(s0)
    182c:	c018                	sw	a4,0(s0)
    182e:	bf59                	j	0x17c4
    1830:	00042423          	sw	zero,8(s0)
    1834:	bf9d                	j	0x17aa
    1836:	8526                	mv	a0,s1
    1838:	85a2                	mv	a1,s0
    183a:	2f49                	jal	0x1fcc
    183c:	00c41783          	lh	a5,12(s0)
    1840:	4818                	lw	a4,16(s0)
    1842:	0017f693          	andi	a3,a5,1
    1846:	c685                	beqz	a3,0x186e
    1848:	4854                	lw	a3,20(s0)
    184a:	00042423          	sw	zero,8(s0)
    184e:	40d006b3          	neg	a3,a3
    1852:	cc14                	sw	a3,24(s0)
    1854:	df41                	beqz	a4,0x17ec
    1856:	bf91                	j	0x17aa
    1858:	853e                	mv	a0,a5
    185a:	aa7fe0ef          	jal	0x300
    185e:	b735                	j	0x178a
    1860:	0027f693          	andi	a3,a5,2
    1864:	ea99                	bnez	a3,0x187a
    1866:	4850                	lw	a2,20(s0)
    1868:	c410                	sw	a2,8(s0)
    186a:	d349                	beqz	a4,0x17ec
    186c:	bf3d                	j	0x17aa
    186e:	0027f693          	andi	a3,a5,2
    1872:	4601                	li	a2,0
    1874:	faf5                	bnez	a3,0x1868
    1876:	4850                	lw	a2,20(s0)
    1878:	bfc5                	j	0x1868
    187a:	00042423          	sw	zero,8(s0)
    187e:	b7bd                	j	0x17ec
    1880:	4725                	li	a4,9
    1882:	0407e793          	ori	a5,a5,64
    1886:	c098                	sw	a4,0(s1)
    1888:	00f41623          	sh	a5,12(s0)
    188c:	557d                	li	a0,-1
    188e:	bf39                	j	0x17ac
    1890:	00357713          	andi	a4,a0,3
    1894:	87aa                	mv	a5,a0
    1896:	0ff5f813          	zext.b	a6,a1
    189a:	832a                	mv	t1,a0
    189c:	c70d                	beqz	a4,0x18c6
    189e:	00c508b3          	add	a7,a0,a2
    18a2:	a039                	j	0x18b0
    18a4:	0007c683          	lbu	a3,0(a5)
    18a8:	07068c63          	beq	a3,a6,0x1920
    18ac:	cb11                	beqz	a4,0x18c0
    18ae:	87aa                	mv	a5,a0
    18b0:	00178513          	addi	a0,a5,1
    18b4:	00357713          	andi	a4,a0,3
    18b8:	ff1796e3          	bne	a5,a7,0x18a4
    18bc:	4501                	li	a0,0
    18be:	8082                	ret
    18c0:	167d                	addi	a2,a2,-1
    18c2:	961a                	add	a2,a2,t1
    18c4:	8e1d                	sub	a2,a2,a5
    18c6:	478d                	li	a5,3
    18c8:	04c7f163          	bgeu	a5,a2,0x190a
    18cc:	0ff5f593          	zext.b	a1,a1
    18d0:	00859693          	slli	a3,a1,0x8
    18d4:	96ae                	add	a3,a3,a1
    18d6:	01069713          	slli	a4,a3,0x10
    18da:	feff0337          	lui	t1,0xfeff0
    18de:	808088b7          	lui	a7,0x80808
    18e2:	85be                	mv	a1,a5
    18e4:	96ba                	add	a3,a3,a4
    18e6:	eff30313          	addi	t1,t1,-257 # 0xfefefeff
    18ea:	08088893          	addi	a7,a7,128 # 0x80808080
    18ee:	411c                	lw	a5,0(a0)
    18f0:	8fb5                	xor	a5,a5,a3
    18f2:	00678733          	add	a4,a5,t1
    18f6:	fff7c793          	not	a5,a5
    18fa:	8ff9                	and	a5,a5,a4
    18fc:	0117f7b3          	and	a5,a5,a7
    1900:	e791                	bnez	a5,0x190c
    1902:	1671                	addi	a2,a2,-4
    1904:	0511                	addi	a0,a0,4
    1906:	fec5e4e3          	bltu	a1,a2,0x18ee
    190a:	da4d                	beqz	a2,0x18bc
    190c:	962a                	add	a2,a2,a0
    190e:	a021                	j	0x1916
    1910:	0505                	addi	a0,a0,1
    1912:	fac505e3          	beq	a0,a2,0x18bc
    1916:	00054783          	lbu	a5,0(a0)
    191a:	ff079be3          	bne	a5,a6,0x1910
    191e:	8082                	ret
    1920:	853e                	mv	a0,a5
    1922:	8082                	ret
    1924:	1141                	addi	sp,sp,-16
    1926:	c422                	sw	s0,8(sp)
    1928:	c226                	sw	s1,4(sp)
    192a:	842a                	mv	s0,a0
    192c:	852e                	mv	a0,a1
    192e:	c606                	sw	ra,12(sp)
    1930:	d401a623          	sw	zero,-692(gp)
    1934:	17d000ef          	jal	0x22b0
    1938:	57fd                	li	a5,-1
    193a:	00f50763          	beq	a0,a5,0x1948
    193e:	40b2                	lw	ra,12(sp)
    1940:	4422                	lw	s0,8(sp)
    1942:	4492                	lw	s1,4(sp)
    1944:	0141                	addi	sp,sp,16
    1946:	8082                	ret
    1948:	d4c1a783          	lw	a5,-692(gp)
    194c:	dbed                	beqz	a5,0x193e
    194e:	40b2                	lw	ra,12(sp)
    1950:	c01c                	sw	a5,0(s0)
    1952:	4422                	lw	s0,8(sp)
    1954:	4492                	lw	s1,4(sp)
    1956:	0141                	addi	sp,sp,16
    1958:	8082                	ret
    195a:	1141                	addi	sp,sp,-16
    195c:	c422                	sw	s0,8(sp)
    195e:	67cd                	lui	a5,0x13
    1960:	644d                	lui	s0,0x13
    1962:	48040413          	addi	s0,s0,1152 # 0x13480
    1966:	47c78793          	addi	a5,a5,1148 # 0x1347c
    196a:	8c1d                	sub	s0,s0,a5
    196c:	c226                	sw	s1,4(sp)
    196e:	c606                	sw	ra,12(sp)
    1970:	40245493          	srai	s1,s0,0x2
    1974:	c881                	beqz	s1,0x1984
    1976:	1471                	addi	s0,s0,-4
    1978:	943e                	add	s0,s0,a5
    197a:	401c                	lw	a5,0(s0)
    197c:	14fd                	addi	s1,s1,-1
    197e:	1471                	addi	s0,s0,-4
    1980:	9782                	jalr	a5
    1982:	fce5                	bnez	s1,0x197a
    1984:	40b2                	lw	ra,12(sp)
    1986:	4422                	lw	s0,8(sp)
    1988:	4492                	lw	s1,4(sp)
    198a:	0141                	addi	sp,sp,16
    198c:	8082                	ret
    198e:	02a5f263          	bgeu	a1,a0,0x19b2
    1992:	00c58733          	add	a4,a1,a2
    1996:	00e57e63          	bgeu	a0,a4,0x19b2
    199a:	00c507b3          	add	a5,a0,a2
    199e:	ca1d                	beqz	a2,0x19d4
    19a0:	fff74683          	lbu	a3,-1(a4)
    19a4:	17fd                	addi	a5,a5,-1
    19a6:	177d                	addi	a4,a4,-1
    19a8:	00d78023          	sb	a3,0(a5)
    19ac:	fef51ae3          	bne	a0,a5,0x19a0
    19b0:	8082                	ret
    19b2:	47bd                	li	a5,15
    19b4:	02c7e163          	bltu	a5,a2,0x19d6
    19b8:	87aa                	mv	a5,a0
    19ba:	fff60693          	addi	a3,a2,-1
    19be:	c25d                	beqz	a2,0x1a64
    19c0:	0685                	addi	a3,a3,1
    19c2:	96be                	add	a3,a3,a5
    19c4:	0005c703          	lbu	a4,0(a1)
    19c8:	0785                	addi	a5,a5,1
    19ca:	0585                	addi	a1,a1,1
    19cc:	fee78fa3          	sb	a4,-1(a5)
    19d0:	fed79ae3          	bne	a5,a3,0x19c4
    19d4:	8082                	ret
    19d6:	00b567b3          	or	a5,a0,a1
    19da:	8b8d                	andi	a5,a5,3
    19dc:	88ae                	mv	a7,a1
    19de:	efbd                	bnez	a5,0x1a5c
    19e0:	ff060793          	addi	a5,a2,-16
    19e4:	ff07f813          	andi	a6,a5,-16
    19e8:	0841                	addi	a6,a6,16
    19ea:	982a                	add	a6,a6,a0
    19ec:	872a                	mv	a4,a0
    19ee:	4194                	lw	a3,0(a1)
    19f0:	05c1                	addi	a1,a1,16
    19f2:	0741                	addi	a4,a4,16
    19f4:	fed72823          	sw	a3,-16(a4)
    19f8:	ff45a683          	lw	a3,-12(a1)
    19fc:	fed72a23          	sw	a3,-12(a4)
    1a00:	ff85a683          	lw	a3,-8(a1)
    1a04:	fed72c23          	sw	a3,-8(a4)
    1a08:	ffc5a683          	lw	a3,-4(a1)
    1a0c:	fed72e23          	sw	a3,-4(a4)
    1a10:	fd071fe3          	bne	a4,a6,0x19ee
    1a14:	9bc1                	andi	a5,a5,-16
    1a16:	01178733          	add	a4,a5,a7
    1a1a:	01070593          	addi	a1,a4,16
    1a1e:	97aa                	add	a5,a5,a0
    1a20:	00c67813          	andi	a6,a2,12
    1a24:	07c1                	addi	a5,a5,16
    1a26:	8e2e                	mv	t3,a1
    1a28:	00f67693          	andi	a3,a2,15
    1a2c:	02080d63          	beqz	a6,0x1a66
    1a30:	16f1                	addi	a3,a3,-4
    1a32:	9af1                	andi	a3,a3,-4
    1a34:	9736                	add	a4,a4,a3
    1a36:	0751                	addi	a4,a4,20
    1a38:	41150833          	sub	a6,a0,a7
    1a3c:	0005a303          	lw	t1,0(a1)
    1a40:	010588b3          	add	a7,a1,a6
    1a44:	0591                	addi	a1,a1,4
    1a46:	0068a023          	sw	t1,0(a7)
    1a4a:	fee599e3          	bne	a1,a4,0x1a3c
    1a4e:	00468713          	addi	a4,a3,4
    1a52:	01c705b3          	add	a1,a4,t3
    1a56:	97ba                	add	a5,a5,a4
    1a58:	8a0d                	andi	a2,a2,3
    1a5a:	b785                	j	0x19ba
    1a5c:	fff60693          	addi	a3,a2,-1
    1a60:	87aa                	mv	a5,a0
    1a62:	bfb9                	j	0x19c0
    1a64:	8082                	ret
    1a66:	8636                	mv	a2,a3
    1a68:	bf89                	j	0x19ba
    1a6a:	00a5c7b3          	xor	a5,a1,a0
    1a6e:	8b8d                	andi	a5,a5,3
    1a70:	00c508b3          	add	a7,a0,a2
    1a74:	e7b1                	bnez	a5,0x1ac0
    1a76:	478d                	li	a5,3
    1a78:	04c7f463          	bgeu	a5,a2,0x1ac0
    1a7c:	00357793          	andi	a5,a0,3
    1a80:	872a                	mv	a4,a0
    1a82:	e7dd                	bnez	a5,0x1b30
    1a84:	ffc8f613          	andi	a2,a7,-4
    1a88:	40e606b3          	sub	a3,a2,a4
    1a8c:	02000793          	li	a5,32
    1a90:	04d7c463          	blt	a5,a3,0x1ad8
    1a94:	86ae                	mv	a3,a1
    1a96:	87ba                	mv	a5,a4
    1a98:	02c77163          	bgeu	a4,a2,0x1aba
    1a9c:	0006a803          	lw	a6,0(a3)
    1aa0:	0791                	addi	a5,a5,4
    1aa2:	0691                	addi	a3,a3,4
    1aa4:	ff07ae23          	sw	a6,-4(a5)
    1aa8:	fec7eae3          	bltu	a5,a2,0x1a9c
    1aac:	167d                	addi	a2,a2,-1
    1aae:	8e19                	sub	a2,a2,a4
    1ab0:	9a71                	andi	a2,a2,-4
    1ab2:	0591                	addi	a1,a1,4
    1ab4:	0711                	addi	a4,a4,4
    1ab6:	95b2                	add	a1,a1,a2
    1ab8:	9732                	add	a4,a4,a2
    1aba:	01176663          	bltu	a4,a7,0x1ac6
    1abe:	8082                	ret
    1ac0:	872a                	mv	a4,a0
    1ac2:	ff157ee3          	bgeu	a0,a7,0x1abe
    1ac6:	0005c783          	lbu	a5,0(a1)
    1aca:	0705                	addi	a4,a4,1
    1acc:	0585                	addi	a1,a1,1
    1ace:	fef70fa3          	sb	a5,-1(a4)
    1ad2:	fee89ae3          	bne	a7,a4,0x1ac6
    1ad6:	8082                	ret
    1ad8:	5194                	lw	a3,32(a1)
    1ada:	0005a383          	lw	t2,0(a1)
    1ade:	0045a283          	lw	t0,4(a1)
    1ae2:	0085af83          	lw	t6,8(a1)
    1ae6:	00c5af03          	lw	t5,12(a1)
    1aea:	0105ae83          	lw	t4,16(a1)
    1aee:	0145ae03          	lw	t3,20(a1)
    1af2:	0185a303          	lw	t1,24(a1)
    1af6:	01c5a803          	lw	a6,28(a1)
    1afa:	02470713          	addi	a4,a4,36
    1afe:	fed72e23          	sw	a3,-4(a4)
    1b02:	fc772e23          	sw	t2,-36(a4)
    1b06:	40e606b3          	sub	a3,a2,a4
    1b0a:	fe572023          	sw	t0,-32(a4)
    1b0e:	fff72223          	sw	t6,-28(a4)
    1b12:	ffe72423          	sw	t5,-24(a4)
    1b16:	ffd72623          	sw	t4,-20(a4)
    1b1a:	ffc72823          	sw	t3,-16(a4)
    1b1e:	fe672a23          	sw	t1,-12(a4)
    1b22:	ff072c23          	sw	a6,-8(a4)
    1b26:	02458593          	addi	a1,a1,36
    1b2a:	fad7c7e3          	blt	a5,a3,0x1ad8
    1b2e:	b79d                	j	0x1a94
    1b30:	0005c683          	lbu	a3,0(a1)
    1b34:	0705                	addi	a4,a4,1
    1b36:	00377793          	andi	a5,a4,3
    1b3a:	fed70fa3          	sb	a3,-1(a4)
    1b3e:	0585                	addi	a1,a1,1
    1b40:	d3b1                	beqz	a5,0x1a84
    1b42:	0005c683          	lbu	a3,0(a1)
    1b46:	0705                	addi	a4,a4,1
    1b48:	00377793          	andi	a5,a4,3
    1b4c:	fed70fa3          	sb	a3,-1(a4)
    1b50:	0585                	addi	a1,a1,1
    1b52:	fff9                	bnez	a5,0x1b30
    1b54:	bf05                	j	0x1a84
    1b56:	d501a783          	lw	a5,-688(gp)
    1b5a:	c3a1                	beqz	a5,0x1b9a
    1b5c:	43d8                	lw	a4,4(a5)
    1b5e:	487d                	li	a6,31
    1b60:	04e84f63          	blt	a6,a4,0x1bbe
    1b64:	00271813          	slli	a6,a4,0x2
    1b68:	c11d                	beqz	a0,0x1b8e
    1b6a:	01078333          	add	t1,a5,a6
    1b6e:	08c32423          	sw	a2,136(t1)
    1b72:	1887a883          	lw	a7,392(a5)
    1b76:	4605                	li	a2,1
    1b78:	00e61633          	sll	a2,a2,a4
    1b7c:	00c8e8b3          	or	a7,a7,a2
    1b80:	1917a423          	sw	a7,392(a5)
    1b84:	10d32423          	sw	a3,264(t1)
    1b88:	4689                	li	a3,2
    1b8a:	00d50f63          	beq	a0,a3,0x1ba8
    1b8e:	0705                	addi	a4,a4,1
    1b90:	c3d8                	sw	a4,4(a5)
    1b92:	97c2                	add	a5,a5,a6
    1b94:	c78c                	sw	a1,8(a5)
    1b96:	4501                	li	a0,0
    1b98:	8082                	ret
    1b9a:	ee018813          	addi	a6,gp,-288
    1b9e:	d501a823          	sw	a6,-688(gp)
    1ba2:	ee018793          	addi	a5,gp,-288
    1ba6:	bf5d                	j	0x1b5c
    1ba8:	18c7a683          	lw	a3,396(a5)
    1bac:	0705                	addi	a4,a4,1
    1bae:	c3d8                	sw	a4,4(a5)
    1bb0:	8ed1                	or	a3,a3,a2
    1bb2:	18d7a623          	sw	a3,396(a5)
    1bb6:	97c2                	add	a5,a5,a6
    1bb8:	c78c                	sw	a1,8(a5)
    1bba:	4501                	li	a0,0
    1bbc:	8082                	ret
    1bbe:	557d                	li	a0,-1
    1bc0:	8082                	ret
    1bc2:	7179                	addi	sp,sp,-48
    1bc4:	d226                	sw	s1,36(sp)
    1bc6:	d606                	sw	ra,44(sp)
    1bc8:	84b2                	mv	s1,a2
    1bca:	14058d63          	beqz	a1,0x1d24
    1bce:	d422                	sw	s0,40(sp)
    1bd0:	d04a                	sw	s2,32(sp)
    1bd2:	842e                	mv	s0,a1
    1bd4:	ce4e                	sw	s3,28(sp)
    1bd6:	ca56                	sw	s5,20(sp)
    1bd8:	cc52                	sw	s4,24(sp)
    1bda:	892a                	mv	s2,a0
    1bdc:	d8aff0ef          	jal	0x1166
    1be0:	ffc42703          	lw	a4,-4(s0)
    1be4:	00b48793          	addi	a5,s1,11
    1be8:	46d9                	li	a3,22
    1bea:	ffc77993          	andi	s3,a4,-4
    1bee:	ff840a93          	addi	s5,s0,-8
    1bf2:	0af6ff63          	bgeu	a3,a5,0x1cb0
    1bf6:	ff87fa13          	andi	s4,a5,-8
    1bfa:	0a07ce63          	bltz	a5,0x1cb6
    1bfe:	0a9a6c63          	bltu	s4,s1,0x1cb6
    1c02:	0d49d563          	bge	s3,s4,0x1ccc
    1c06:	67cd                	lui	a5,0x13
    1c08:	c462                	sw	s8,8(sp)
    1c0a:	5b078c13          	addi	s8,a5,1456 # 0x135b0
    1c0e:	008c2603          	lw	a2,8(s8)
    1c12:	013a86b3          	add	a3,s5,s3
    1c16:	42dc                	lw	a5,4(a3)
    1c18:	12d60e63          	beq	a2,a3,0x1d54
    1c1c:	ffe7f613          	andi	a2,a5,-2
    1c20:	9636                	add	a2,a2,a3
    1c22:	4250                	lw	a2,4(a2)
    1c24:	8a05                	andi	a2,a2,1
    1c26:	e27d                	bnez	a2,0x1d0c
    1c28:	9bf1                	andi	a5,a5,-4
    1c2a:	00f98633          	add	a2,s3,a5
    1c2e:	09465963          	bge	a2,s4,0x1cc0
    1c32:	8b05                	andi	a4,a4,1
    1c34:	e70d                	bnez	a4,0x1c5e
    1c36:	c65e                	sw	s7,12(sp)
    1c38:	ff842b83          	lw	s7,-8(s0)
    1c3c:	c85a                	sw	s6,16(sp)
    1c3e:	417a8bb3          	sub	s7,s5,s7
    1c42:	004ba703          	lw	a4,4(s7)
    1c46:	9b71                	andi	a4,a4,-4
    1c48:	97ba                	add	a5,a5,a4
    1c4a:	01378b33          	add	s6,a5,s3
    1c4e:	234b5663          	bge	s6,s4,0x1e7a
    1c52:	00e98b33          	add	s6,s3,a4
    1c56:	1d4b5463          	bge	s6,s4,0x1e1e
    1c5a:	4b42                	lw	s6,16(sp)
    1c5c:	4bb2                	lw	s7,12(sp)
    1c5e:	85a6                	mv	a1,s1
    1c60:	854a                	mv	a0,s2
    1c62:	f39fe0ef          	jal	0xb9a
    1c66:	84aa                	mv	s1,a0
    1c68:	2c050463          	beqz	a0,0x1f30
    1c6c:	ffc42783          	lw	a5,-4(s0)
    1c70:	ff850713          	addi	a4,a0,-8
    1c74:	9bf9                	andi	a5,a5,-2
    1c76:	97d6                	add	a5,a5,s5
    1c78:	18e78d63          	beq	a5,a4,0x1e12
    1c7c:	ffc98613          	addi	a2,s3,-4
    1c80:	02400793          	li	a5,36
    1c84:	1ec7e863          	bltu	a5,a2,0x1e74
    1c88:	474d                	li	a4,19
    1c8a:	16c76863          	bltu	a4,a2,0x1dfa
    1c8e:	87aa                	mv	a5,a0
    1c90:	8722                	mv	a4,s0
    1c92:	4314                	lw	a3,0(a4)
    1c94:	c394                	sw	a3,0(a5)
    1c96:	4354                	lw	a3,4(a4)
    1c98:	c3d4                	sw	a3,4(a5)
    1c9a:	4718                	lw	a4,8(a4)
    1c9c:	c798                	sw	a4,8(a5)
    1c9e:	85a2                	mv	a1,s0
    1ca0:	854a                	mv	a0,s2
    1ca2:	cddfe0ef          	jal	0x97e
    1ca6:	854a                	mv	a0,s2
    1ca8:	cc0ff0ef          	jal	0x1168
    1cac:	4c22                	lw	s8,8(sp)
    1cae:	a0a9                	j	0x1cf8
    1cb0:	4a41                	li	s4,16
    1cb2:	f49a78e3          	bgeu	s4,s1,0x1c02
    1cb6:	47b1                	li	a5,12
    1cb8:	00f92023          	sw	a5,0(s2)
    1cbc:	4481                	li	s1,0
    1cbe:	a82d                	j	0x1cf8
    1cc0:	46dc                	lw	a5,12(a3)
    1cc2:	4698                	lw	a4,8(a3)
    1cc4:	4c22                	lw	s8,8(sp)
    1cc6:	89b2                	mv	s3,a2
    1cc8:	c75c                	sw	a5,12(a4)
    1cca:	c798                	sw	a4,8(a5)
    1ccc:	004aa783          	lw	a5,4(s5)
    1cd0:	414986b3          	sub	a3,s3,s4
    1cd4:	463d                	li	a2,15
    1cd6:	8b85                	andi	a5,a5,1
    1cd8:	013a8733          	add	a4,s5,s3
    1cdc:	04d66a63          	bltu	a2,a3,0x1d30
    1ce0:	0137e7b3          	or	a5,a5,s3
    1ce4:	00faa223          	sw	a5,4(s5)
    1ce8:	435c                	lw	a5,4(a4)
    1cea:	0017e793          	ori	a5,a5,1
    1cee:	c35c                	sw	a5,4(a4)
    1cf0:	854a                	mv	a0,s2
    1cf2:	c76ff0ef          	jal	0x1168
    1cf6:	84a2                	mv	s1,s0
    1cf8:	5422                	lw	s0,40(sp)
    1cfa:	50b2                	lw	ra,44(sp)
    1cfc:	5902                	lw	s2,32(sp)
    1cfe:	49f2                	lw	s3,28(sp)
    1d00:	4a62                	lw	s4,24(sp)
    1d02:	4ad2                	lw	s5,20(sp)
    1d04:	8526                	mv	a0,s1
    1d06:	5492                	lw	s1,36(sp)
    1d08:	6145                	addi	sp,sp,48
    1d0a:	8082                	ret
    1d0c:	8b05                	andi	a4,a4,1
    1d0e:	fb21                	bnez	a4,0x1c5e
    1d10:	c65e                	sw	s7,12(sp)
    1d12:	ff842b83          	lw	s7,-8(s0)
    1d16:	c85a                	sw	s6,16(sp)
    1d18:	417a8bb3          	sub	s7,s5,s7
    1d1c:	004ba703          	lw	a4,4(s7)
    1d20:	9b71                	andi	a4,a4,-4
    1d22:	bf05                	j	0x1c52
    1d24:	50b2                	lw	ra,44(sp)
    1d26:	5492                	lw	s1,36(sp)
    1d28:	85b2                	mv	a1,a2
    1d2a:	6145                	addi	sp,sp,48
    1d2c:	e6ffe06f          	j	0xb9a
    1d30:	0147e7b3          	or	a5,a5,s4
    1d34:	00faa223          	sw	a5,4(s5)
    1d38:	014a85b3          	add	a1,s5,s4
    1d3c:	0016e693          	ori	a3,a3,1
    1d40:	c1d4                	sw	a3,4(a1)
    1d42:	435c                	lw	a5,4(a4)
    1d44:	05a1                	addi	a1,a1,8
    1d46:	854a                	mv	a0,s2
    1d48:	0017e793          	ori	a5,a5,1
    1d4c:	c35c                	sw	a5,4(a4)
    1d4e:	c31fe0ef          	jal	0x97e
    1d52:	bf79                	j	0x1cf0
    1d54:	9bf1                	andi	a5,a5,-4
    1d56:	013786b3          	add	a3,a5,s3
    1d5a:	010a0613          	addi	a2,s4,16
    1d5e:	18c6d763          	bge	a3,a2,0x1eec
    1d62:	8b05                	andi	a4,a4,1
    1d64:	ee071de3          	bnez	a4,0x1c5e
    1d68:	c65e                	sw	s7,12(sp)
    1d6a:	ff842b83          	lw	s7,-8(s0)
    1d6e:	c85a                	sw	s6,16(sp)
    1d70:	417a8bb3          	sub	s7,s5,s7
    1d74:	004ba703          	lw	a4,4(s7)
    1d78:	9b71                	andi	a4,a4,-4
    1d7a:	97ba                	add	a5,a5,a4
    1d7c:	01378b33          	add	s6,a5,s3
    1d80:	eccb49e3          	blt	s6,a2,0x1c52
    1d84:	00cba783          	lw	a5,12(s7)
    1d88:	008ba703          	lw	a4,8(s7)
    1d8c:	ffc98613          	addi	a2,s3,-4
    1d90:	02400693          	li	a3,36
    1d94:	c75c                	sw	a5,12(a4)
    1d96:	c798                	sw	a4,8(a5)
    1d98:	008b8493          	addi	s1,s7,8
    1d9c:	1cc6e763          	bltu	a3,a2,0x1f6a
    1da0:	474d                	li	a4,19
    1da2:	87a6                	mv	a5,s1
    1da4:	00c77e63          	bgeu	a4,a2,0x1dc0
    1da8:	4018                	lw	a4,0(s0)
    1daa:	47ed                	li	a5,27
    1dac:	00eba423          	sw	a4,8(s7)
    1db0:	4058                	lw	a4,4(s0)
    1db2:	00eba623          	sw	a4,12(s7)
    1db6:	1cc7ea63          	bltu	a5,a2,0x1f8a
    1dba:	0421                	addi	s0,s0,8
    1dbc:	010b8793          	addi	a5,s7,16
    1dc0:	4018                	lw	a4,0(s0)
    1dc2:	c398                	sw	a4,0(a5)
    1dc4:	4058                	lw	a4,4(s0)
    1dc6:	c3d8                	sw	a4,4(a5)
    1dc8:	4418                	lw	a4,8(s0)
    1dca:	c798                	sw	a4,8(a5)
    1dcc:	014b87b3          	add	a5,s7,s4
    1dd0:	414b0733          	sub	a4,s6,s4
    1dd4:	00fc2423          	sw	a5,8(s8)
    1dd8:	00176713          	ori	a4,a4,1
    1ddc:	c3d8                	sw	a4,4(a5)
    1dde:	004ba783          	lw	a5,4(s7)
    1de2:	854a                	mv	a0,s2
    1de4:	8b85                	andi	a5,a5,1
    1de6:	0147e7b3          	or	a5,a5,s4
    1dea:	00fba223          	sw	a5,4(s7)
    1dee:	b7aff0ef          	jal	0x1168
    1df2:	4b42                	lw	s6,16(sp)
    1df4:	4bb2                	lw	s7,12(sp)
    1df6:	4c22                	lw	s8,8(sp)
    1df8:	b701                	j	0x1cf8
    1dfa:	4014                	lw	a3,0(s0)
    1dfc:	476d                	li	a4,27
    1dfe:	c114                	sw	a3,0(a0)
    1e00:	4054                	lw	a3,4(s0)
    1e02:	c154                	sw	a3,4(a0)
    1e04:	0cc76963          	bltu	a4,a2,0x1ed6
    1e08:	00840713          	addi	a4,s0,8
    1e0c:	00850793          	addi	a5,a0,8
    1e10:	b549                	j	0x1c92
    1e12:	ffc52783          	lw	a5,-4(a0)
    1e16:	4c22                	lw	s8,8(sp)
    1e18:	9bf1                	andi	a5,a5,-4
    1e1a:	99be                	add	s3,s3,a5
    1e1c:	bd45                	j	0x1ccc
    1e1e:	00cba783          	lw	a5,12(s7)
    1e22:	008ba683          	lw	a3,8(s7)
    1e26:	ffc98613          	addi	a2,s3,-4
    1e2a:	02400593          	li	a1,36
    1e2e:	c6dc                	sw	a5,12(a3)
    1e30:	c794                	sw	a3,8(a5)
    1e32:	008b8493          	addi	s1,s7,8
    1e36:	08c5eb63          	bltu	a1,a2,0x1ecc
    1e3a:	46cd                	li	a3,19
    1e3c:	87a6                	mv	a5,s1
    1e3e:	00c6fe63          	bgeu	a3,a2,0x1e5a
    1e42:	4018                	lw	a4,0(s0)
    1e44:	47ed                	li	a5,27
    1e46:	00eba423          	sw	a4,8(s7)
    1e4a:	4058                	lw	a4,4(s0)
    1e4c:	00eba623          	sw	a4,12(s7)
    1e50:	0cc7e463          	bltu	a5,a2,0x1f18
    1e54:	0421                	addi	s0,s0,8
    1e56:	010b8793          	addi	a5,s7,16
    1e5a:	4014                	lw	a3,0(s0)
    1e5c:	c394                	sw	a3,0(a5)
    1e5e:	4054                	lw	a3,4(s0)
    1e60:	c3d4                	sw	a3,4(a5)
    1e62:	4414                	lw	a3,8(s0)
    1e64:	c794                	sw	a3,8(a5)
    1e66:	89da                	mv	s3,s6
    1e68:	8ade                	mv	s5,s7
    1e6a:	4b42                	lw	s6,16(sp)
    1e6c:	4bb2                	lw	s7,12(sp)
    1e6e:	4c22                	lw	s8,8(sp)
    1e70:	8426                	mv	s0,s1
    1e72:	bda9                	j	0x1ccc
    1e74:	85a2                	mv	a1,s0
    1e76:	3e21                	jal	0x198e
    1e78:	b51d                	j	0x1c9e
    1e7a:	46dc                	lw	a5,12(a3)
    1e7c:	4698                	lw	a4,8(a3)
    1e7e:	ffc98613          	addi	a2,s3,-4
    1e82:	02400693          	li	a3,36
    1e86:	c75c                	sw	a5,12(a4)
    1e88:	c798                	sw	a4,8(a5)
    1e8a:	008ba703          	lw	a4,8(s7)
    1e8e:	00cba783          	lw	a5,12(s7)
    1e92:	008b8493          	addi	s1,s7,8
    1e96:	c75c                	sw	a5,12(a4)
    1e98:	c798                	sw	a4,8(a5)
    1e9a:	02c6e963          	bltu	a3,a2,0x1ecc
    1e9e:	474d                	li	a4,19
    1ea0:	87a6                	mv	a5,s1
    1ea2:	00c77e63          	bgeu	a4,a2,0x1ebe
    1ea6:	4018                	lw	a4,0(s0)
    1ea8:	47ed                	li	a5,27
    1eaa:	00eba423          	sw	a4,8(s7)
    1eae:	4058                	lw	a4,4(s0)
    1eb0:	00eba623          	sw	a4,12(s7)
    1eb4:	08c7ed63          	bltu	a5,a2,0x1f4e
    1eb8:	0421                	addi	s0,s0,8
    1eba:	010b8793          	addi	a5,s7,16
    1ebe:	4018                	lw	a4,0(s0)
    1ec0:	c398                	sw	a4,0(a5)
    1ec2:	4058                	lw	a4,4(s0)
    1ec4:	c3d8                	sw	a4,4(a5)
    1ec6:	4418                	lw	a4,8(s0)
    1ec8:	c798                	sw	a4,8(a5)
    1eca:	bf71                	j	0x1e66
    1ecc:	85a2                	mv	a1,s0
    1ece:	8526                	mv	a0,s1
    1ed0:	abfff0ef          	jal	0x198e
    1ed4:	bf49                	j	0x1e66
    1ed6:	4418                	lw	a4,8(s0)
    1ed8:	c518                	sw	a4,8(a0)
    1eda:	4458                	lw	a4,12(s0)
    1edc:	c558                	sw	a4,12(a0)
    1ede:	04f60f63          	beq	a2,a5,0x1f3c
    1ee2:	01040713          	addi	a4,s0,16
    1ee6:	01050793          	addi	a5,a0,16
    1eea:	b365                	j	0x1c92
    1eec:	9ad2                	add	s5,s5,s4
    1eee:	414686b3          	sub	a3,a3,s4
    1ef2:	015c2423          	sw	s5,8(s8)
    1ef6:	0016e793          	ori	a5,a3,1
    1efa:	00faa223          	sw	a5,4(s5)
    1efe:	ffc42783          	lw	a5,-4(s0)
    1f02:	854a                	mv	a0,s2
    1f04:	84a2                	mv	s1,s0
    1f06:	8b85                	andi	a5,a5,1
    1f08:	0147e7b3          	or	a5,a5,s4
    1f0c:	fef42e23          	sw	a5,-4(s0)
    1f10:	a58ff0ef          	jal	0x1168
    1f14:	4c22                	lw	s8,8(sp)
    1f16:	b3cd                	j	0x1cf8
    1f18:	441c                	lw	a5,8(s0)
    1f1a:	00fba823          	sw	a5,16(s7)
    1f1e:	445c                	lw	a5,12(s0)
    1f20:	00fbaa23          	sw	a5,20(s7)
    1f24:	04b60863          	beq	a2,a1,0x1f74
    1f28:	0441                	addi	s0,s0,16
    1f2a:	018b8793          	addi	a5,s7,24
    1f2e:	b735                	j	0x1e5a
    1f30:	854a                	mv	a0,s2
    1f32:	a36ff0ef          	jal	0x1168
    1f36:	4481                	li	s1,0
    1f38:	4c22                	lw	s8,8(sp)
    1f3a:	bb7d                	j	0x1cf8
    1f3c:	4814                	lw	a3,16(s0)
    1f3e:	01840713          	addi	a4,s0,24
    1f42:	01850793          	addi	a5,a0,24
    1f46:	c914                	sw	a3,16(a0)
    1f48:	4854                	lw	a3,20(s0)
    1f4a:	c954                	sw	a3,20(a0)
    1f4c:	b399                	j	0x1c92
    1f4e:	4418                	lw	a4,8(s0)
    1f50:	02400793          	li	a5,36
    1f54:	00eba823          	sw	a4,16(s7)
    1f58:	4458                	lw	a4,12(s0)
    1f5a:	00ebaa23          	sw	a4,20(s7)
    1f5e:	04f60263          	beq	a2,a5,0x1fa2
    1f62:	0441                	addi	s0,s0,16
    1f64:	018b8793          	addi	a5,s7,24
    1f68:	bf99                	j	0x1ebe
    1f6a:	85a2                	mv	a1,s0
    1f6c:	8526                	mv	a0,s1
    1f6e:	a21ff0ef          	jal	0x198e
    1f72:	bda9                	j	0x1dcc
    1f74:	4818                	lw	a4,16(s0)
    1f76:	020b8793          	addi	a5,s7,32
    1f7a:	0461                	addi	s0,s0,24
    1f7c:	00ebac23          	sw	a4,24(s7)
    1f80:	ffc42703          	lw	a4,-4(s0)
    1f84:	00ebae23          	sw	a4,28(s7)
    1f88:	bdc9                	j	0x1e5a
    1f8a:	441c                	lw	a5,8(s0)
    1f8c:	00fba823          	sw	a5,16(s7)
    1f90:	445c                	lw	a5,12(s0)
    1f92:	00fbaa23          	sw	a5,20(s7)
    1f96:	02d60163          	beq	a2,a3,0x1fb8
    1f9a:	0441                	addi	s0,s0,16
    1f9c:	018b8793          	addi	a5,s7,24
    1fa0:	b505                	j	0x1dc0
    1fa2:	4818                	lw	a4,16(s0)
    1fa4:	020b8793          	addi	a5,s7,32
    1fa8:	0461                	addi	s0,s0,24
    1faa:	00ebac23          	sw	a4,24(s7)
    1fae:	ffc42703          	lw	a4,-4(s0)
    1fb2:	00ebae23          	sw	a4,28(s7)
    1fb6:	b721                	j	0x1ebe
    1fb8:	4818                	lw	a4,16(s0)
    1fba:	020b8793          	addi	a5,s7,32
    1fbe:	00ebac23          	sw	a4,24(s7)
    1fc2:	4858                	lw	a4,20(s0)
    1fc4:	0461                	addi	s0,s0,24
    1fc6:	00ebae23          	sw	a4,28(s7)
    1fca:	bbdd                	j	0x1dc0
    1fcc:	00c59783          	lh	a5,12(a1)
    1fd0:	7159                	addi	sp,sp,-112
    1fd2:	d4a2                	sw	s0,104(sp)
    1fd4:	d686                	sw	ra,108(sp)
    1fd6:	0027f713          	andi	a4,a5,2
    1fda:	842e                	mv	s0,a1
    1fdc:	cb19                	beqz	a4,0x1ff2
    1fde:	04358793          	addi	a5,a1,67
    1fe2:	4705                	li	a4,1
    1fe4:	c19c                	sw	a5,0(a1)
    1fe6:	c99c                	sw	a5,16(a1)
    1fe8:	c9d8                	sw	a4,20(a1)
    1fea:	50b6                	lw	ra,108(sp)
    1fec:	5426                	lw	s0,104(sp)
    1fee:	6165                	addi	sp,sp,112
    1ff0:	8082                	ret
    1ff2:	00e59583          	lh	a1,14(a1)
    1ff6:	d0ca                	sw	s2,96(sp)
    1ff8:	d2a6                	sw	s1,100(sp)
    1ffa:	892a                	mv	s2,a0
    1ffc:	0405cd63          	bltz	a1,0x2056
    2000:	0030                	addi	a2,sp,8
    2002:	22a9                	jal	0x214c
    2004:	04054763          	bltz	a0,0x2052
    2008:	40000593          	li	a1,1024
    200c:	854a                	mv	a0,s2
    200e:	44b2                	lw	s1,12(sp)
    2010:	b8bfe0ef          	jal	0xb9a
    2014:	00c41783          	lh	a5,12(s0)
    2018:	cd3d                	beqz	a0,0x2096
    201a:	673d                	lui	a4,0xf
    201c:	0807e793          	ori	a5,a5,128
    2020:	40000693          	li	a3,1024
    2024:	8cf9                	and	s1,s1,a4
    2026:	00f41623          	sh	a5,12(s0)
    202a:	c008                	sw	a0,0(s0)
    202c:	c808                	sw	a0,16(s0)
    202e:	c854                	sw	a3,20(s0)
    2030:	6709                	lui	a4,0x2
    2032:	08e49663          	bne	s1,a4,0x20be
    2036:	00e41583          	lh	a1,14(s0)
    203a:	854a                	mv	a0,s2
    203c:	22a1                	jal	0x2184
    203e:	6705                	lui	a4,0x1
    2040:	00c41783          	lh	a5,12(s0)
    2044:	80070713          	addi	a4,a4,-2048 # 0x800
    2048:	cd15                	beqz	a0,0x2084
    204a:	9bf1                	andi	a5,a5,-4
    204c:	0017e793          	ori	a5,a5,1
    2050:	a815                	j	0x2084
    2052:	00c41783          	lh	a5,12(s0)
    2056:	0807f493          	andi	s1,a5,128
    205a:	0014b493          	seqz	s1,s1
    205e:	409004b3          	neg	s1,s1
    2062:	3c04f493          	andi	s1,s1,960
    2066:	04048493          	addi	s1,s1,64
    206a:	854a                	mv	a0,s2
    206c:	85a6                	mv	a1,s1
    206e:	b2dfe0ef          	jal	0xb9a
    2072:	00c41783          	lh	a5,12(s0)
    2076:	c105                	beqz	a0,0x2096
    2078:	0807e793          	ori	a5,a5,128
    207c:	c008                	sw	a0,0(s0)
    207e:	c808                	sw	a0,16(s0)
    2080:	c844                	sw	s1,20(s0)
    2082:	4701                	li	a4,0
    2084:	8fd9                	or	a5,a5,a4
    2086:	50b6                	lw	ra,108(sp)
    2088:	00f41623          	sh	a5,12(s0)
    208c:	5426                	lw	s0,104(sp)
    208e:	5496                	lw	s1,100(sp)
    2090:	5906                	lw	s2,96(sp)
    2092:	6165                	addi	sp,sp,112
    2094:	8082                	ret
    2096:	2007f713          	andi	a4,a5,512
    209a:	ef19                	bnez	a4,0x20b8
    209c:	9bf1                	andi	a5,a5,-4
    209e:	04340713          	addi	a4,s0,67
    20a2:	0027e793          	ori	a5,a5,2
    20a6:	4685                	li	a3,1
    20a8:	5496                	lw	s1,100(sp)
    20aa:	5906                	lw	s2,96(sp)
    20ac:	00f41623          	sh	a5,12(s0)
    20b0:	c018                	sw	a4,0(s0)
    20b2:	c818                	sw	a4,16(s0)
    20b4:	c854                	sw	a3,20(s0)
    20b6:	bf15                	j	0x1fea
    20b8:	5496                	lw	s1,100(sp)
    20ba:	5906                	lw	s2,96(sp)
    20bc:	b73d                	j	0x1fea
    20be:	6705                	lui	a4,0x1
    20c0:	80070713          	addi	a4,a4,-2048 # 0x800
    20c4:	b7c1                	j	0x2084
    20c6:	7159                	addi	sp,sp,-112
    20c8:	d4a2                	sw	s0,104(sp)
    20ca:	842e                	mv	s0,a1
    20cc:	00e59583          	lh	a1,14(a1)
    20d0:	d2a6                	sw	s1,100(sp)
    20d2:	d0ca                	sw	s2,96(sp)
    20d4:	d686                	sw	ra,108(sp)
    20d6:	84b2                	mv	s1,a2
    20d8:	8936                	mv	s2,a3
    20da:	0205cb63          	bltz	a1,0x2110
    20de:	0030                	addi	a2,sp,8
    20e0:	20b5                	jal	0x214c
    20e2:	02054763          	bltz	a0,0x2110
    20e6:	47b2                	lw	a5,12(sp)
    20e8:	66bd                	lui	a3,0xf
    20ea:	7779                	lui	a4,0xffffe
    20ec:	8ff5                	and	a5,a5,a3
    20ee:	97ba                	add	a5,a5,a4
    20f0:	50b6                	lw	ra,108(sp)
    20f2:	5426                	lw	s0,104(sp)
    20f4:	0017b793          	seqz	a5,a5
    20f8:	00f92023          	sw	a5,0(s2)
    20fc:	40000713          	li	a4,1024
    2100:	c098                	sw	a4,0(s1)
    2102:	6505                	lui	a0,0x1
    2104:	5496                	lw	s1,100(sp)
    2106:	5906                	lw	s2,96(sp)
    2108:	80050513          	addi	a0,a0,-2048 # 0x800
    210c:	6165                	addi	sp,sp,112
    210e:	8082                	ret
    2110:	00c45783          	lhu	a5,12(s0)
    2114:	0807f793          	andi	a5,a5,128
    2118:	cf91                	beqz	a5,0x2134
    211a:	50b6                	lw	ra,108(sp)
    211c:	5426                	lw	s0,104(sp)
    211e:	4781                	li	a5,0
    2120:	00f92023          	sw	a5,0(s2)
    2124:	04000713          	li	a4,64
    2128:	c098                	sw	a4,0(s1)
    212a:	5906                	lw	s2,96(sp)
    212c:	5496                	lw	s1,100(sp)
    212e:	4501                	li	a0,0
    2130:	6165                	addi	sp,sp,112
    2132:	8082                	ret
    2134:	50b6                	lw	ra,108(sp)
    2136:	5426                	lw	s0,104(sp)
    2138:	00f92023          	sw	a5,0(s2)
    213c:	40000713          	li	a4,1024
    2140:	c098                	sw	a4,0(s1)
    2142:	5906                	lw	s2,96(sp)
    2144:	5496                	lw	s1,100(sp)
    2146:	4501                	li	a0,0
    2148:	6165                	addi	sp,sp,112
    214a:	8082                	ret
    214c:	1141                	addi	sp,sp,-16
    214e:	872e                	mv	a4,a1
    2150:	c422                	sw	s0,8(sp)
    2152:	c226                	sw	s1,4(sp)
    2154:	85b2                	mv	a1,a2
    2156:	842a                	mv	s0,a0
    2158:	853a                	mv	a0,a4
    215a:	c606                	sw	ra,12(sp)
    215c:	d401a623          	sw	zero,-692(gp)
    2160:	204d                	jal	0x2202
    2162:	57fd                	li	a5,-1
    2164:	00f50763          	beq	a0,a5,0x2172
    2168:	40b2                	lw	ra,12(sp)
    216a:	4422                	lw	s0,8(sp)
    216c:	4492                	lw	s1,4(sp)
    216e:	0141                	addi	sp,sp,16
    2170:	8082                	ret
    2172:	d4c1a783          	lw	a5,-692(gp)
    2176:	dbed                	beqz	a5,0x2168
    2178:	40b2                	lw	ra,12(sp)
    217a:	c01c                	sw	a5,0(s0)
    217c:	4422                	lw	s0,8(sp)
    217e:	4492                	lw	s1,4(sp)
    2180:	0141                	addi	sp,sp,16
    2182:	8082                	ret
    2184:	1141                	addi	sp,sp,-16
    2186:	c422                	sw	s0,8(sp)
    2188:	c226                	sw	s1,4(sp)
    218a:	842a                	mv	s0,a0
    218c:	852e                	mv	a0,a1
    218e:	c606                	sw	ra,12(sp)
    2190:	d401a623          	sw	zero,-692(gp)
    2194:	205d                	jal	0x223a
    2196:	57fd                	li	a5,-1
    2198:	00f50763          	beq	a0,a5,0x21a6
    219c:	40b2                	lw	ra,12(sp)
    219e:	4422                	lw	s0,8(sp)
    21a0:	4492                	lw	s1,4(sp)
    21a2:	0141                	addi	sp,sp,16
    21a4:	8082                	ret
    21a6:	d4c1a783          	lw	a5,-692(gp)
    21aa:	dbed                	beqz	a5,0x219c
    21ac:	40b2                	lw	ra,12(sp)
    21ae:	c01c                	sw	a5,0(s0)
    21b0:	4422                	lw	s0,8(sp)
    21b2:	4492                	lw	s1,4(sp)
    21b4:	0141                	addi	sp,sp,16
    21b6:	8082                	ret
    21b8:	1141                	addi	sp,sp,-16
    21ba:	c606                	sw	ra,12(sp)
    21bc:	c422                	sw	s0,8(sp)
    21be:	03900893          	li	a7,57
    21c2:	00000073          	ecall
    21c6:	842a                	mv	s0,a0
    21c8:	00054763          	bltz	a0,0x21d6
    21cc:	40b2                	lw	ra,12(sp)
    21ce:	8522                	mv	a0,s0
    21d0:	4422                	lw	s0,8(sp)
    21d2:	0141                	addi	sp,sp,16
    21d4:	8082                	ret
    21d6:	40800433          	neg	s0,s0
    21da:	22e1                	jal	0x23a2
    21dc:	c100                	sw	s0,0(a0)
    21de:	547d                	li	s0,-1
    21e0:	b7f5                	j	0x21cc
    21e2:	05d00893          	li	a7,93
    21e6:	00000073          	ecall
    21ea:	00054363          	bltz	a0,0x21f0
    21ee:	a001                	j	0x21ee
    21f0:	1141                	addi	sp,sp,-16
    21f2:	c422                	sw	s0,8(sp)
    21f4:	842a                	mv	s0,a0
    21f6:	c606                	sw	ra,12(sp)
    21f8:	40800433          	neg	s0,s0
    21fc:	225d                	jal	0x23a2
    21fe:	c100                	sw	s0,0(a0)
    2200:	a001                	j	0x2200
    2202:	7175                	addi	sp,sp,-144
    2204:	c326                	sw	s1,132(sp)
    2206:	c706                	sw	ra,140(sp)
    2208:	84ae                	mv	s1,a1
    220a:	c522                	sw	s0,136(sp)
    220c:	858a                	mv	a1,sp
    220e:	05000893          	li	a7,80
    2212:	00000073          	ecall
    2216:	842a                	mv	s0,a0
    2218:	00054b63          	bltz	a0,0x222e
    221c:	8526                	mv	a0,s1
    221e:	858a                	mv	a1,sp
    2220:	2231                	jal	0x232c
    2222:	40ba                	lw	ra,140(sp)
    2224:	8522                	mv	a0,s0
    2226:	442a                	lw	s0,136(sp)
    2228:	449a                	lw	s1,132(sp)
    222a:	6149                	addi	sp,sp,144
    222c:	8082                	ret
    222e:	40800433          	neg	s0,s0
    2232:	2a85                	jal	0x23a2
    2234:	c100                	sw	s0,0(a0)
    2236:	547d                	li	s0,-1
    2238:	b7d5                	j	0x221c
    223a:	7159                	addi	sp,sp,-112
    223c:	002c                	addi	a1,sp,8
    223e:	d686                	sw	ra,108(sp)
    2240:	37c9                	jal	0x2202
    2242:	57fd                	li	a5,-1
    2244:	00f50863          	beq	a0,a5,0x2254
    2248:	4532                	lw	a0,12(sp)
    224a:	50b6                	lw	ra,108(sp)
    224c:	8135                	srli	a0,a0,0xd
    224e:	8905                	andi	a0,a0,1
    2250:	6165                	addi	sp,sp,112
    2252:	8082                	ret
    2254:	50b6                	lw	ra,108(sp)
    2256:	4501                	li	a0,0
    2258:	6165                	addi	sp,sp,112
    225a:	8082                	ret
    225c:	1141                	addi	sp,sp,-16
    225e:	c606                	sw	ra,12(sp)
    2260:	c422                	sw	s0,8(sp)
    2262:	03e00893          	li	a7,62
    2266:	00000073          	ecall
    226a:	842a                	mv	s0,a0
    226c:	00054763          	bltz	a0,0x227a
    2270:	40b2                	lw	ra,12(sp)
    2272:	8522                	mv	a0,s0
    2274:	4422                	lw	s0,8(sp)
    2276:	0141                	addi	sp,sp,16
    2278:	8082                	ret
    227a:	40800433          	neg	s0,s0
    227e:	2215                	jal	0x23a2
    2280:	c100                	sw	s0,0(a0)
    2282:	547d                	li	s0,-1
    2284:	b7f5                	j	0x2270
    2286:	1141                	addi	sp,sp,-16
    2288:	c606                	sw	ra,12(sp)
    228a:	c422                	sw	s0,8(sp)
    228c:	03f00893          	li	a7,63
    2290:	00000073          	ecall
    2294:	842a                	mv	s0,a0
    2296:	00054763          	bltz	a0,0x22a4
    229a:	40b2                	lw	ra,12(sp)
    229c:	8522                	mv	a0,s0
    229e:	4422                	lw	s0,8(sp)
    22a0:	0141                	addi	sp,sp,16
    22a2:	8082                	ret
    22a4:	40800433          	neg	s0,s0
    22a8:	28ed                	jal	0x23a2
    22aa:	c100                	sw	s0,0(a0)
    22ac:	547d                	li	s0,-1
    22ae:	b7f5                	j	0x229a
    22b0:	d601a703          	lw	a4,-672(gp)
    22b4:	1141                	addi	sp,sp,-16
    22b6:	c606                	sw	ra,12(sp)
    22b8:	87aa                	mv	a5,a0
    22ba:	ef01                	bnez	a4,0x22d2
    22bc:	0d600893          	li	a7,214
    22c0:	4501                	li	a0,0
    22c2:	00000073          	ecall
    22c6:	567d                	li	a2,-1
    22c8:	872a                	mv	a4,a0
    22ca:	02c50563          	beq	a0,a2,0x22f4
    22ce:	d6a1a023          	sw	a0,-672(gp)
    22d2:	00e78533          	add	a0,a5,a4
    22d6:	0d600893          	li	a7,214
    22da:	00000073          	ecall
    22de:	d601a703          	lw	a4,-672(gp)
    22e2:	97ba                	add	a5,a5,a4
    22e4:	00f51863          	bne	a0,a5,0x22f4
    22e8:	40b2                	lw	ra,12(sp)
    22ea:	d6a1a023          	sw	a0,-672(gp)
    22ee:	853a                	mv	a0,a4
    22f0:	0141                	addi	sp,sp,16
    22f2:	8082                	ret
    22f4:	207d                	jal	0x23a2
    22f6:	40b2                	lw	ra,12(sp)
    22f8:	47b1                	li	a5,12
    22fa:	c11c                	sw	a5,0(a0)
    22fc:	557d                	li	a0,-1
    22fe:	0141                	addi	sp,sp,16
    2300:	8082                	ret
    2302:	1141                	addi	sp,sp,-16
    2304:	c606                	sw	ra,12(sp)
    2306:	c422                	sw	s0,8(sp)
    2308:	04000893          	li	a7,64
    230c:	00000073          	ecall
    2310:	842a                	mv	s0,a0
    2312:	00054763          	bltz	a0,0x2320
    2316:	40b2                	lw	ra,12(sp)
    2318:	8522                	mv	a0,s0
    231a:	4422                	lw	s0,8(sp)
    231c:	0141                	addi	sp,sp,16
    231e:	8082                	ret
    2320:	40800433          	neg	s0,s0
    2324:	28bd                	jal	0x23a2
    2326:	c100                	sw	s0,0(a0)
    2328:	547d                	li	s0,-1
    232a:	b7f5                	j	0x2316
    232c:	1141                	addi	sp,sp,-16
    232e:	419c                	lw	a5,0(a1)
    2330:	0145a383          	lw	t2,20(a1)
    2334:	0185a283          	lw	t0,24(a1)
    2338:	01c5af83          	lw	t6,28(a1)
    233c:	0205af03          	lw	t5,32(a1)
    2340:	0305ae83          	lw	t4,48(a1)
    2344:	0405ae03          	lw	t3,64(a1)
    2348:	0385a303          	lw	t1,56(a1)
    234c:	0485a803          	lw	a6,72(a1)
    2350:	04c5a883          	lw	a7,76(a1)
    2354:	4db0                	lw	a2,88(a1)
    2356:	c622                	sw	s0,12(sp)
    2358:	c426                	sw	s1,8(sp)
    235a:	4980                	lw	s0,16(a1)
    235c:	4584                	lw	s1,8(a1)
    235e:	4df4                	lw	a3,92(a1)
    2360:	55b8                	lw	a4,104(a1)
    2362:	00f51023          	sh	a5,0(a0)
    2366:	55fc                	lw	a5,108(a1)
    2368:	00951123          	sh	s1,2(a0)
    236c:	c140                	sw	s0,4(a0)
    236e:	00751423          	sh	t2,8(a0)
    2372:	00551523          	sh	t0,10(a0)
    2376:	01f51623          	sh	t6,12(a0)
    237a:	01e51723          	sh	t5,14(a0)
    237e:	01d52823          	sw	t4,16(a0)
    2382:	05c52623          	sw	t3,76(a0)
    2386:	04652423          	sw	t1,72(a0)
    238a:	01052c23          	sw	a6,24(a0)
    238e:	01152e23          	sw	a7,28(a0)
    2392:	d510                	sw	a2,40(a0)
    2394:	d554                	sw	a3,44(a0)
    2396:	4432                	lw	s0,12(sp)
    2398:	dd18                	sw	a4,56(a0)
    239a:	dd5c                	sw	a5,60(a0)
    239c:	44a2                	lw	s1,8(sp)
    239e:	0141                	addi	sp,sp,16
    23a0:	8082                	ret
    23a2:	d3c1a503          	lw	a0,-708(gp)
    23a6:	8082                	ret
    23a8:	6548                	flw	fa0,12(a0)
    23aa:	6c6c                	flw	fa1,92(s0)
    23ac:	52202c6f          	jal	s8,0x48ce
    23b0:	5349                	li	t1,-14
    23b2:	21562d43          	fmadd.s	fs10,fa2,fs5,ft4,rdn
    23b6:	0000                	unimp
    23b8:	000a                	c.slli	zero,0x2
	...
    33be:	0000                	unimp
    33c0:	00d0                	addi	a2,sp,68
    33c2:	0001                	nop
    33c4:	0148                	addi	a0,sp,132
    33c6:	0001                	nop
    33c8:	011a                	slli	sp,sp,0x6
    33ca:	0001                	nop
    33cc:	0000                	unimp
    33ce:	0000                	unimp
    33d0:	00000003          	lb	zero,0(zero) # 0x0
    33d4:	3a00                	fld	fs0,48(a2)
    33d6:	0001                	nop
	...
    33e0:	3a00                	fld	fs0,48(a2)
    33e2:	0001                	nop
    33e4:	3a68                	fld	fa0,240(a2)
    33e6:	0001                	nop
    33e8:	3ad0                	fld	fa2,176(a3)
    33ea:	0001                	nop
	...
    3474:	0001                	nop
    3476:	0000                	unimp
    3478:	0000                	unimp
    347a:	0000                	unimp
    347c:	330e                	fld	ft6,224(sp)
    347e:	abcd                	j	0x3a70
    3480:	1234                	addi	a3,sp,296
    3482:	e66d                	bnez	a2,0x356c
    3484:	deec                	sw	a1,124(a3)
    3486:	0005                	c.nop	1
    3488:	0000000b          	.insn	4, 0x000b
	...
    3504:	35b0                	fld	fa2,104(a1)
    3506:	0001                	nop
    3508:	35b0                	fld	fa2,104(a1)
    350a:	0001                	nop
    350c:	35b8                	fld	fa4,104(a1)
    350e:	0001                	nop
    3510:	35b8                	fld	fa4,104(a1)
    3512:	0001                	nop
    3514:	35c0                	fld	fs0,168(a1)
    3516:	0001                	nop
    3518:	35c0                	fld	fs0,168(a1)
    351a:	0001                	nop
    351c:	35c8                	fld	fa0,168(a1)
    351e:	0001                	nop
    3520:	35c8                	fld	fa0,168(a1)
    3522:	0001                	nop
    3524:	35d0                	fld	fa2,168(a1)
    3526:	0001                	nop
    3528:	35d0                	fld	fa2,168(a1)
    352a:	0001                	nop
    352c:	35d8                	fld	fa4,168(a1)
    352e:	0001                	nop
    3530:	35d8                	fld	fa4,168(a1)
    3532:	0001                	nop
    3534:	35e0                	fld	fs0,232(a1)
    3536:	0001                	nop
    3538:	35e0                	fld	fs0,232(a1)
    353a:	0001                	nop
    353c:	35e8                	fld	fa0,232(a1)
    353e:	0001                	nop
    3540:	35e8                	fld	fa0,232(a1)
    3542:	0001                	nop
    3544:	35f0                	fld	fa2,232(a1)
    3546:	0001                	nop
    3548:	35f0                	fld	fa2,232(a1)
    354a:	0001                	nop
    354c:	35f8                	fld	fa4,232(a1)
    354e:	0001                	nop
    3550:	35f8                	fld	fa4,232(a1)
    3552:	0001                	nop
    3554:	3600                	fld	fs0,40(a2)
    3556:	0001                	nop
    3558:	3600                	fld	fs0,40(a2)
    355a:	0001                	nop
    355c:	3608                	fld	fa0,40(a2)
    355e:	0001                	nop
    3560:	3608                	fld	fa0,40(a2)
    3562:	0001                	nop
    3564:	3610                	fld	fa2,40(a2)
    3566:	0001                	nop
    3568:	3610                	fld	fa2,40(a2)
    356a:	0001                	nop
    356c:	3618                	fld	fa4,40(a2)
    356e:	0001                	nop
    3570:	3618                	fld	fa4,40(a2)
    3572:	0001                	nop
    3574:	3620                	fld	fs0,104(a2)
    3576:	0001                	nop
    3578:	3620                	fld	fs0,104(a2)
    357a:	0001                	nop
    357c:	3628                	fld	fa0,104(a2)
    357e:	0001                	nop
    3580:	3628                	fld	fa0,104(a2)
    3582:	0001                	nop
    3584:	3630                	fld	fa2,104(a2)
    3586:	0001                	nop
    3588:	3630                	fld	fa2,104(a2)
    358a:	0001                	nop
    358c:	3638                	fld	fa4,104(a2)
    358e:	0001                	nop
    3590:	3638                	fld	fa4,104(a2)
    3592:	0001                	nop
    3594:	3640                	fld	fs0,168(a2)
    3596:	0001                	nop
    3598:	3640                	fld	fs0,168(a2)
    359a:	0001                	nop
    359c:	3648                	fld	fa0,168(a2)
    359e:	0001                	nop
    35a0:	3648                	fld	fa0,168(a2)
    35a2:	0001                	nop
    35a4:	3650                	fld	fa2,168(a2)
    35a6:	0001                	nop
    35a8:	3650                	fld	fa2,168(a2)
    35aa:	0001                	nop
    35ac:	3658                	fld	fa4,168(a2)
    35ae:	0001                	nop
    35b0:	3658                	fld	fa4,168(a2)
    35b2:	0001                	nop
    35b4:	3660                	fld	fs0,232(a2)
    35b6:	0001                	nop
    35b8:	3660                	fld	fs0,232(a2)
    35ba:	0001                	nop
    35bc:	3668                	fld	fa0,232(a2)
    35be:	0001                	nop
    35c0:	3668                	fld	fa0,232(a2)
    35c2:	0001                	nop
    35c4:	3670                	fld	fa2,232(a2)
    35c6:	0001                	nop
    35c8:	3670                	fld	fa2,232(a2)
    35ca:	0001                	nop
    35cc:	3678                	fld	fa4,232(a2)
    35ce:	0001                	nop
    35d0:	3678                	fld	fa4,232(a2)
    35d2:	0001                	nop
    35d4:	3680                	fld	fs0,40(a3)
    35d6:	0001                	nop
    35d8:	3680                	fld	fs0,40(a3)
    35da:	0001                	nop
    35dc:	3688                	fld	fa0,40(a3)
    35de:	0001                	nop
    35e0:	3688                	fld	fa0,40(a3)
    35e2:	0001                	nop
    35e4:	3690                	fld	fa2,40(a3)
    35e6:	0001                	nop
    35e8:	3690                	fld	fa2,40(a3)
    35ea:	0001                	nop
    35ec:	3698                	fld	fa4,40(a3)
    35ee:	0001                	nop
    35f0:	3698                	fld	fa4,40(a3)
    35f2:	0001                	nop
    35f4:	36a0                	fld	fs0,104(a3)
    35f6:	0001                	nop
    35f8:	36a0                	fld	fs0,104(a3)
    35fa:	0001                	nop
    35fc:	36a8                	fld	fa0,104(a3)
    35fe:	0001                	nop
    3600:	36a8                	fld	fa0,104(a3)
    3602:	0001                	nop
    3604:	36b0                	fld	fa2,104(a3)
    3606:	0001                	nop
    3608:	36b0                	fld	fa2,104(a3)
    360a:	0001                	nop
    360c:	36b8                	fld	fa4,104(a3)
    360e:	0001                	nop
    3610:	36b8                	fld	fa4,104(a3)
    3612:	0001                	nop
    3614:	36c0                	fld	fs0,168(a3)
    3616:	0001                	nop
    3618:	36c0                	fld	fs0,168(a3)
    361a:	0001                	nop
    361c:	36c8                	fld	fa0,168(a3)
    361e:	0001                	nop
    3620:	36c8                	fld	fa0,168(a3)
    3622:	0001                	nop
    3624:	36d0                	fld	fa2,168(a3)
    3626:	0001                	nop
    3628:	36d0                	fld	fa2,168(a3)
    362a:	0001                	nop
    362c:	36d8                	fld	fa4,168(a3)
    362e:	0001                	nop
    3630:	36d8                	fld	fa4,168(a3)
    3632:	0001                	nop
    3634:	36e0                	fld	fs0,232(a3)
    3636:	0001                	nop
    3638:	36e0                	fld	fs0,232(a3)
    363a:	0001                	nop
    363c:	36e8                	fld	fa0,232(a3)
    363e:	0001                	nop
    3640:	36e8                	fld	fa0,232(a3)
    3642:	0001                	nop
    3644:	36f0                	fld	fa2,232(a3)
    3646:	0001                	nop
    3648:	36f0                	fld	fa2,232(a3)
    364a:	0001                	nop
    364c:	36f8                	fld	fa4,232(a3)
    364e:	0001                	nop
    3650:	36f8                	fld	fa4,232(a3)
    3652:	0001                	nop
    3654:	3700                	fld	fs0,40(a4)
    3656:	0001                	nop
    3658:	3700                	fld	fs0,40(a4)
    365a:	0001                	nop
    365c:	3708                	fld	fa0,40(a4)
    365e:	0001                	nop
    3660:	3708                	fld	fa0,40(a4)
    3662:	0001                	nop
    3664:	3710                	fld	fa2,40(a4)
    3666:	0001                	nop
    3668:	3710                	fld	fa2,40(a4)
    366a:	0001                	nop
    366c:	3718                	fld	fa4,40(a4)
    366e:	0001                	nop
    3670:	3718                	fld	fa4,40(a4)
    3672:	0001                	nop
    3674:	3720                	fld	fs0,104(a4)
    3676:	0001                	nop
    3678:	3720                	fld	fs0,104(a4)
    367a:	0001                	nop
    367c:	3728                	fld	fa0,104(a4)
    367e:	0001                	nop
    3680:	3728                	fld	fa0,104(a4)
    3682:	0001                	nop
    3684:	3730                	fld	fa2,104(a4)
    3686:	0001                	nop
    3688:	3730                	fld	fa2,104(a4)
    368a:	0001                	nop
    368c:	3738                	fld	fa4,104(a4)
    368e:	0001                	nop
    3690:	3738                	fld	fa4,104(a4)
    3692:	0001                	nop
    3694:	3740                	fld	fs0,168(a4)
    3696:	0001                	nop
    3698:	3740                	fld	fs0,168(a4)
    369a:	0001                	nop
    369c:	3748                	fld	fa0,168(a4)
    369e:	0001                	nop
    36a0:	3748                	fld	fa0,168(a4)
    36a2:	0001                	nop
    36a4:	3750                	fld	fa2,168(a4)
    36a6:	0001                	nop
    36a8:	3750                	fld	fa2,168(a4)
    36aa:	0001                	nop
    36ac:	3758                	fld	fa4,168(a4)
    36ae:	0001                	nop
    36b0:	3758                	fld	fa4,168(a4)
    36b2:	0001                	nop
    36b4:	3760                	fld	fs0,232(a4)
    36b6:	0001                	nop
    36b8:	3760                	fld	fs0,232(a4)
    36ba:	0001                	nop
    36bc:	3768                	fld	fa0,232(a4)
    36be:	0001                	nop
    36c0:	3768                	fld	fa0,232(a4)
    36c2:	0001                	nop
    36c4:	3770                	fld	fa2,232(a4)
    36c6:	0001                	nop
    36c8:	3770                	fld	fa2,232(a4)
    36ca:	0001                	nop
    36cc:	3778                	fld	fa4,232(a4)
    36ce:	0001                	nop
    36d0:	3778                	fld	fa4,232(a4)
    36d2:	0001                	nop
    36d4:	3780                	fld	fs0,40(a5)
    36d6:	0001                	nop
    36d8:	3780                	fld	fs0,40(a5)
    36da:	0001                	nop
    36dc:	3788                	fld	fa0,40(a5)
    36de:	0001                	nop
    36e0:	3788                	fld	fa0,40(a5)
    36e2:	0001                	nop
    36e4:	3790                	fld	fa2,40(a5)
    36e6:	0001                	nop
    36e8:	3790                	fld	fa2,40(a5)
    36ea:	0001                	nop
    36ec:	3798                	fld	fa4,40(a5)
    36ee:	0001                	nop
    36f0:	3798                	fld	fa4,40(a5)
    36f2:	0001                	nop
    36f4:	37a0                	fld	fs0,104(a5)
    36f6:	0001                	nop
    36f8:	37a0                	fld	fs0,104(a5)
    36fa:	0001                	nop
    36fc:	37a8                	fld	fa0,104(a5)
    36fe:	0001                	nop
    3700:	37a8                	fld	fa0,104(a5)
    3702:	0001                	nop
    3704:	37b0                	fld	fa2,104(a5)
    3706:	0001                	nop
    3708:	37b0                	fld	fa2,104(a5)
    370a:	0001                	nop
    370c:	37b8                	fld	fa4,104(a5)
    370e:	0001                	nop
    3710:	37b8                	fld	fa4,104(a5)
    3712:	0001                	nop
    3714:	37c0                	fld	fs0,168(a5)
    3716:	0001                	nop
    3718:	37c0                	fld	fs0,168(a5)
    371a:	0001                	nop
    371c:	37c8                	fld	fa0,168(a5)
    371e:	0001                	nop
    3720:	37c8                	fld	fa0,168(a5)
    3722:	0001                	nop
    3724:	37d0                	fld	fa2,168(a5)
    3726:	0001                	nop
    3728:	37d0                	fld	fa2,168(a5)
    372a:	0001                	nop
    372c:	37d8                	fld	fa4,168(a5)
    372e:	0001                	nop
    3730:	37d8                	fld	fa4,168(a5)
    3732:	0001                	nop
    3734:	37e0                	fld	fs0,232(a5)
    3736:	0001                	nop
    3738:	37e0                	fld	fs0,232(a5)
    373a:	0001                	nop
    373c:	37e8                	fld	fa0,232(a5)
    373e:	0001                	nop
    3740:	37e8                	fld	fa0,232(a5)
    3742:	0001                	nop
    3744:	37f0                	fld	fa2,232(a5)
    3746:	0001                	nop
    3748:	37f0                	fld	fa2,232(a5)
    374a:	0001                	nop
    374c:	37f8                	fld	fa4,232(a5)
    374e:	0001                	nop
    3750:	37f8                	fld	fa4,232(a5)
    3752:	0001                	nop
    3754:	3800                	fld	fs0,48(s0)
    3756:	0001                	nop
    3758:	3800                	fld	fs0,48(s0)
    375a:	0001                	nop
    375c:	3808                	fld	fa0,48(s0)
    375e:	0001                	nop
    3760:	3808                	fld	fa0,48(s0)
    3762:	0001                	nop
    3764:	3810                	fld	fa2,48(s0)
    3766:	0001                	nop
    3768:	3810                	fld	fa2,48(s0)
    376a:	0001                	nop
    376c:	3818                	fld	fa4,48(s0)
    376e:	0001                	nop
    3770:	3818                	fld	fa4,48(s0)
    3772:	0001                	nop
    3774:	3820                	fld	fs0,112(s0)
    3776:	0001                	nop
    3778:	3820                	fld	fs0,112(s0)
    377a:	0001                	nop
    377c:	3828                	fld	fa0,112(s0)
    377e:	0001                	nop
    3780:	3828                	fld	fa0,112(s0)
    3782:	0001                	nop
    3784:	3830                	fld	fa2,112(s0)
    3786:	0001                	nop
    3788:	3830                	fld	fa2,112(s0)
    378a:	0001                	nop
    378c:	3838                	fld	fa4,112(s0)
    378e:	0001                	nop
    3790:	3838                	fld	fa4,112(s0)
    3792:	0001                	nop
    3794:	3840                	fld	fs0,176(s0)
    3796:	0001                	nop
    3798:	3840                	fld	fs0,176(s0)
    379a:	0001                	nop
    379c:	3848                	fld	fa0,176(s0)
    379e:	0001                	nop
    37a0:	3848                	fld	fa0,176(s0)
    37a2:	0001                	nop
    37a4:	3850                	fld	fa2,176(s0)
    37a6:	0001                	nop
    37a8:	3850                	fld	fa2,176(s0)
    37aa:	0001                	nop
    37ac:	3858                	fld	fa4,176(s0)
    37ae:	0001                	nop
    37b0:	3858                	fld	fa4,176(s0)
    37b2:	0001                	nop
    37b4:	3860                	fld	fs0,240(s0)
    37b6:	0001                	nop
    37b8:	3860                	fld	fs0,240(s0)
    37ba:	0001                	nop
    37bc:	3868                	fld	fa0,240(s0)
    37be:	0001                	nop
    37c0:	3868                	fld	fa0,240(s0)
    37c2:	0001                	nop
    37c4:	3870                	fld	fa2,240(s0)
    37c6:	0001                	nop
    37c8:	3870                	fld	fa2,240(s0)
    37ca:	0001                	nop
    37cc:	3878                	fld	fa4,240(s0)
    37ce:	0001                	nop
    37d0:	3878                	fld	fa4,240(s0)
    37d2:	0001                	nop
    37d4:	3880                	fld	fs0,48(s1)
    37d6:	0001                	nop
    37d8:	3880                	fld	fs0,48(s1)
    37da:	0001                	nop
    37dc:	3888                	fld	fa0,48(s1)
    37de:	0001                	nop
    37e0:	3888                	fld	fa0,48(s1)
    37e2:	0001                	nop
    37e4:	3890                	fld	fa2,48(s1)
    37e6:	0001                	nop
    37e8:	3890                	fld	fa2,48(s1)
    37ea:	0001                	nop
    37ec:	3898                	fld	fa4,48(s1)
    37ee:	0001                	nop
    37f0:	3898                	fld	fa4,48(s1)
    37f2:	0001                	nop
    37f4:	38a0                	fld	fs0,112(s1)
    37f6:	0001                	nop
    37f8:	38a0                	fld	fs0,112(s1)
    37fa:	0001                	nop
    37fc:	38a8                	fld	fa0,112(s1)
    37fe:	0001                	nop
    3800:	38a8                	fld	fa0,112(s1)
    3802:	0001                	nop
    3804:	38b0                	fld	fa2,112(s1)
    3806:	0001                	nop
    3808:	38b0                	fld	fa2,112(s1)
    380a:	0001                	nop
    380c:	38b8                	fld	fa4,112(s1)
    380e:	0001                	nop
    3810:	38b8                	fld	fa4,112(s1)
    3812:	0001                	nop
    3814:	38c0                	fld	fs0,176(s1)
    3816:	0001                	nop
    3818:	38c0                	fld	fs0,176(s1)
    381a:	0001                	nop
    381c:	38c8                	fld	fa0,176(s1)
    381e:	0001                	nop
    3820:	38c8                	fld	fa0,176(s1)
    3822:	0001                	nop
    3824:	38d0                	fld	fa2,176(s1)
    3826:	0001                	nop
    3828:	38d0                	fld	fa2,176(s1)
    382a:	0001                	nop
    382c:	38d8                	fld	fa4,176(s1)
    382e:	0001                	nop
    3830:	38d8                	fld	fa4,176(s1)
    3832:	0001                	nop
    3834:	38e0                	fld	fs0,240(s1)
    3836:	0001                	nop
    3838:	38e0                	fld	fs0,240(s1)
    383a:	0001                	nop
    383c:	38e8                	fld	fa0,240(s1)
    383e:	0001                	nop
    3840:	38e8                	fld	fa0,240(s1)
    3842:	0001                	nop
    3844:	38f0                	fld	fa2,240(s1)
    3846:	0001                	nop
    3848:	38f0                	fld	fa2,240(s1)
    384a:	0001                	nop
    384c:	38f8                	fld	fa4,240(s1)
    384e:	0001                	nop
    3850:	38f8                	fld	fa4,240(s1)
    3852:	0001                	nop
    3854:	3900                	fld	fs0,48(a0)
    3856:	0001                	nop
    3858:	3900                	fld	fs0,48(a0)
    385a:	0001                	nop
    385c:	3908                	fld	fa0,48(a0)
    385e:	0001                	nop
    3860:	3908                	fld	fa0,48(a0)
    3862:	0001                	nop
    3864:	3910                	fld	fa2,48(a0)
    3866:	0001                	nop
    3868:	3910                	fld	fa2,48(a0)
    386a:	0001                	nop
    386c:	3918                	fld	fa4,48(a0)
    386e:	0001                	nop
    3870:	3918                	fld	fa4,48(a0)
    3872:	0001                	nop
    3874:	3920                	fld	fs0,112(a0)
    3876:	0001                	nop
    3878:	3920                	fld	fs0,112(a0)
    387a:	0001                	nop
    387c:	3928                	fld	fa0,112(a0)
    387e:	0001                	nop
    3880:	3928                	fld	fa0,112(a0)
    3882:	0001                	nop
    3884:	3930                	fld	fa2,112(a0)
    3886:	0001                	nop
    3888:	3930                	fld	fa2,112(a0)
    388a:	0001                	nop
    388c:	3938                	fld	fa4,112(a0)
    388e:	0001                	nop
    3890:	3938                	fld	fa4,112(a0)
    3892:	0001                	nop
    3894:	3940                	fld	fs0,176(a0)
    3896:	0001                	nop
    3898:	3940                	fld	fs0,176(a0)
    389a:	0001                	nop
    389c:	3948                	fld	fa0,176(a0)
    389e:	0001                	nop
    38a0:	3948                	fld	fa0,176(a0)
    38a2:	0001                	nop
    38a4:	3950                	fld	fa2,176(a0)
    38a6:	0001                	nop
    38a8:	3950                	fld	fa2,176(a0)
    38aa:	0001                	nop
    38ac:	3958                	fld	fa4,176(a0)
    38ae:	0001                	nop
    38b0:	3958                	fld	fa4,176(a0)
    38b2:	0001                	nop
    38b4:	3960                	fld	fs0,240(a0)
    38b6:	0001                	nop
    38b8:	3960                	fld	fs0,240(a0)
    38ba:	0001                	nop
    38bc:	3968                	fld	fa0,240(a0)
    38be:	0001                	nop
    38c0:	3968                	fld	fa0,240(a0)
    38c2:	0001                	nop
    38c4:	3970                	fld	fa2,240(a0)
    38c6:	0001                	nop
    38c8:	3970                	fld	fa2,240(a0)
    38ca:	0001                	nop
    38cc:	3978                	fld	fa4,240(a0)
    38ce:	0001                	nop
    38d0:	3978                	fld	fa4,240(a0)
    38d2:	0001                	nop
    38d4:	3980                	fld	fs0,48(a1)
    38d6:	0001                	nop
    38d8:	3980                	fld	fs0,48(a1)
    38da:	0001                	nop
    38dc:	3988                	fld	fa0,48(a1)
    38de:	0001                	nop
    38e0:	3988                	fld	fa0,48(a1)
    38e2:	0001                	nop
    38e4:	3990                	fld	fa2,48(a1)
    38e6:	0001                	nop
    38e8:	3990                	fld	fa2,48(a1)
    38ea:	0001                	nop
    38ec:	3998                	fld	fa4,48(a1)
    38ee:	0001                	nop
    38f0:	3998                	fld	fa4,48(a1)
    38f2:	0001                	nop
    38f4:	39a0                	fld	fs0,112(a1)
    38f6:	0001                	nop
    38f8:	39a0                	fld	fs0,112(a1)
    38fa:	0001                	nop
    38fc:	39a8                	fld	fa0,112(a1)
    38fe:	0001                	nop
    3900:	39a8                	fld	fa0,112(a1)
    3902:	0001                	nop
    3904:	0000                	unimp
    3906:	0000                	unimp
    3908:	3490                	fld	fa2,40(s1)
    390a:	0001                	nop
    390c:	ffff                	.insn	2, 0xffff
    390e:	ffff                	.insn	2, 0xffff
    3910:	0000                	unimp
    3912:	0002                	c.slli64	zero
```
**hello.asm content:**
```assembly

hello.elf:     file format elf32-littleriscv


Disassembly of section .text:

000100b4 <exit>:
   100b4:	1141                	addi	sp,sp,-16
   100b6:	4581                	li	a1,0
   100b8:	c422                	sw	s0,8(sp)
   100ba:	c606                	sw	ra,12(sp)
   100bc:	842a                	mv	s0,a0
   100be:	7d0000ef          	jal	1088e <__call_exitprocs>
   100c2:	d481a783          	lw	a5,-696(gp) # 139c8 <__stdio_exit_handler>
   100c6:	c391                	beqz	a5,100ca <exit+0x16>
   100c8:	9782                	jalr	a5
   100ca:	8522                	mv	a0,s0
   100cc:	1ca020ef          	jal	12296 <_exit>

000100d0 <register_fini>:
   100d0:	00000793          	li	a5,0
   100d4:	c791                	beqz	a5,100e0 <register_fini+0x10>
   100d6:	6549                	lui	a0,0x12
   100d8:	a0e50513          	addi	a0,a0,-1522 # 11a0e <__libc_fini_array>
   100dc:	0810006f          	j	1095c <atexit>
   100e0:	8082                	ret

000100e2 <_start>:
   100e2:	00004197          	auipc	gp,0x4
   100e6:	b9e18193          	addi	gp,gp,-1122 # 13c80 <__global_pointer$>
   100ea:	d4818513          	addi	a0,gp,-696 # 139c8 <__stdio_exit_handler>
   100ee:	07018613          	addi	a2,gp,112 # 13cf0 <__BSS_END__>
   100f2:	8e09                	sub	a2,a2,a0
   100f4:	4581                	li	a1,0
   100f6:	2571                	jal	10782 <memset>
   100f8:	00001517          	auipc	a0,0x1
   100fc:	86450513          	addi	a0,a0,-1948 # 1095c <atexit>
   10100:	c519                	beqz	a0,1010e <_start+0x2c>
   10102:	00002517          	auipc	a0,0x2
   10106:	90c50513          	addi	a0,a0,-1780 # 11a0e <__libc_fini_array>
   1010a:	053000ef          	jal	1095c <atexit>
   1010e:	2529                	jal	10718 <__libc_init_array>
   10110:	4502                	lw	a0,0(sp)
   10112:	004c                	addi	a1,sp,4
   10114:	4601                	li	a2,0
   10116:	20b1                	jal	10162 <main>
   10118:	bf71                	j	100b4 <exit>

0001011a <__do_global_dtors_aux>:
   1011a:	1141                	addi	sp,sp,-16
   1011c:	c422                	sw	s0,8(sp)
   1011e:	d641c783          	lbu	a5,-668(gp) # 139e4 <completed.1>
   10122:	c606                	sw	ra,12(sp)
   10124:	ef91                	bnez	a5,10140 <__do_global_dtors_aux+0x26>
   10126:	00000793          	li	a5,0
   1012a:	cb81                	beqz	a5,1013a <__do_global_dtors_aux+0x20>
   1012c:	6549                	lui	a0,0x12
   1012e:	47050513          	addi	a0,a0,1136 # 12470 <__EH_FRAME_BEGIN__>
   10132:	00000097          	auipc	ra,0x0
   10136:	000000e7          	jalr	zero # 0 <exit-0x100b4>
   1013a:	4785                	li	a5,1
   1013c:	d6f18223          	sb	a5,-668(gp) # 139e4 <completed.1>
   10140:	40b2                	lw	ra,12(sp)
   10142:	4422                	lw	s0,8(sp)
   10144:	0141                	addi	sp,sp,16
   10146:	8082                	ret

00010148 <frame_dummy>:
   10148:	00000793          	li	a5,0
   1014c:	cb91                	beqz	a5,10160 <frame_dummy+0x18>
   1014e:	6549                	lui	a0,0x12
   10150:	d6818593          	addi	a1,gp,-664 # 139e8 <object.0>
   10154:	47050513          	addi	a0,a0,1136 # 12470 <__EH_FRAME_BEGIN__>
   10158:	00000317          	auipc	t1,0x0
   1015c:	00000067          	jr	zero # 0 <exit-0x100b4>
   10160:	8082                	ret

00010162 <main>:
   10162:	1141                	addi	sp,sp,-16
   10164:	c606                	sw	ra,12(sp)
   10166:	c422                	sw	s0,8(sp)
   10168:	0800                	addi	s0,sp,16
   1016a:	67c9                	lui	a5,0x12
   1016c:	45c78513          	addi	a0,a5,1116 # 1245c <__errno+0x6>
   10170:	26bd                	jal	104de <puts>
   10172:	4781                	li	a5,0
   10174:	853e                	mv	a0,a5
   10176:	40b2                	lw	ra,12(sp)
   10178:	4422                	lw	s0,8(sp)
   1017a:	0141                	addi	sp,sp,16
   1017c:	8082                	ret

0001017e <__fp_lock>:
   1017e:	4501                	li	a0,0
   10180:	8082                	ret

00010182 <stdio_exit_handler>:
   10182:	664d                	lui	a2,0x13
   10184:	65c5                	lui	a1,0x11
   10186:	654d                	lui	a0,0x13
   10188:	48060613          	addi	a2,a2,1152 # 13480 <__sglue>
   1018c:	21e58593          	addi	a1,a1,542 # 1121e <_fclose_r>
   10190:	49050513          	addi	a0,a0,1168 # 13490 <_impure_data>
   10194:	aca9                	j	103ee <_fwalk_sglue>

00010196 <cleanup_stdio>:
   10196:	414c                	lw	a1,4(a0)
   10198:	1141                	addi	sp,sp,-16
   1019a:	c422                	sw	s0,8(sp)
   1019c:	c606                	sw	ra,12(sp)
   1019e:	d8018793          	addi	a5,gp,-640 # 13a00 <__sf>
   101a2:	842a                	mv	s0,a0
   101a4:	00f58463          	beq	a1,a5,101ac <cleanup_stdio+0x16>
   101a8:	076010ef          	jal	1121e <_fclose_r>
   101ac:	440c                	lw	a1,8(s0)
   101ae:	de818793          	addi	a5,gp,-536 # 13a68 <__sf+0x68>
   101b2:	00f58563          	beq	a1,a5,101bc <cleanup_stdio+0x26>
   101b6:	8522                	mv	a0,s0
   101b8:	066010ef          	jal	1121e <_fclose_r>
   101bc:	444c                	lw	a1,12(s0)
   101be:	e5018793          	addi	a5,gp,-432 # 13ad0 <__sf+0xd0>
   101c2:	00f58863          	beq	a1,a5,101d2 <cleanup_stdio+0x3c>
   101c6:	8522                	mv	a0,s0
   101c8:	4422                	lw	s0,8(sp)
   101ca:	40b2                	lw	ra,12(sp)
   101cc:	0141                	addi	sp,sp,16
   101ce:	0500106f          	j	1121e <_fclose_r>
   101d2:	40b2                	lw	ra,12(sp)
   101d4:	4422                	lw	s0,8(sp)
   101d6:	0141                	addi	sp,sp,16
   101d8:	8082                	ret

000101da <__fp_unlock>:
   101da:	4501                	li	a0,0
   101dc:	8082                	ret

000101de <global_stdio_init.part.0>:
   101de:	1101                	addi	sp,sp,-32
   101e0:	cc22                	sw	s0,24(sp)
   101e2:	67c1                	lui	a5,0x10
   101e4:	d8018413          	addi	s0,gp,-640 # 13a00 <__sf>
   101e8:	ce06                	sw	ra,28(sp)
   101ea:	ca26                	sw	s1,20(sp)
   101ec:	c84a                	sw	s2,16(sp)
   101ee:	c64e                	sw	s3,12(sp)
   101f0:	c452                	sw	s4,8(sp)
   101f2:	4711                	li	a4,4
   101f4:	18278793          	addi	a5,a5,386 # 10182 <stdio_exit_handler>
   101f8:	4621                	li	a2,8
   101fa:	4581                	li	a1,0
   101fc:	ddc18513          	addi	a0,gp,-548 # 13a5c <__sf+0x5c>
   10200:	d4f1a423          	sw	a5,-696(gp) # 139c8 <__stdio_exit_handler>
   10204:	c458                	sw	a4,12(s0)
   10206:	00042023          	sw	zero,0(s0)
   1020a:	00042223          	sw	zero,4(s0)
   1020e:	00042423          	sw	zero,8(s0)
   10212:	06042223          	sw	zero,100(s0)
   10216:	00042823          	sw	zero,16(s0)
   1021a:	00042a23          	sw	zero,20(s0)
   1021e:	00042c23          	sw	zero,24(s0)
   10222:	2385                	jal	10782 <memset>
   10224:	67c1                	lui	a5,0x10
   10226:	6a41                	lui	s4,0x10
   10228:	69c1                	lui	s3,0x10
   1022a:	6941                	lui	s2,0x10
   1022c:	64c1                	lui	s1,0x10
   1022e:	4e6a0a13          	addi	s4,s4,1254 # 104e6 <__sread>
   10232:	52098993          	addi	s3,s3,1312 # 10520 <__swrite>
   10236:	57090913          	addi	s2,s2,1392 # 10570 <__sseek>
   1023a:	5ac48493          	addi	s1,s1,1452 # 105ac <__sclose>
   1023e:	07a5                	addi	a5,a5,9 # 10009 <exit-0xab>
   10240:	4621                	li	a2,8
   10242:	4581                	li	a1,0
   10244:	e4418513          	addi	a0,gp,-444 # 13ac4 <__sf+0xc4>
   10248:	d87c                	sw	a5,116(s0)
   1024a:	03442023          	sw	s4,32(s0)
   1024e:	03342223          	sw	s3,36(s0)
   10252:	03242423          	sw	s2,40(s0)
   10256:	d444                	sw	s1,44(s0)
   10258:	cc40                	sw	s0,28(s0)
   1025a:	06042423          	sw	zero,104(s0)
   1025e:	06042623          	sw	zero,108(s0)
   10262:	06042823          	sw	zero,112(s0)
   10266:	0c042623          	sw	zero,204(s0)
   1026a:	06042c23          	sw	zero,120(s0)
   1026e:	06042e23          	sw	zero,124(s0)
   10272:	08042023          	sw	zero,128(s0)
   10276:	2331                	jal	10782 <memset>
   10278:	000207b7          	lui	a5,0x20
   1027c:	07c9                	addi	a5,a5,18 # 20012 <__BSS_END__+0xc322>
   1027e:	de818713          	addi	a4,gp,-536 # 13a68 <__sf+0x68>
   10282:	eac18513          	addi	a0,gp,-340 # 13b2c <__sf+0x12c>
   10286:	4621                	li	a2,8
   10288:	4581                	li	a1,0
   1028a:	09442423          	sw	s4,136(s0)
   1028e:	09342623          	sw	s3,140(s0)
   10292:	09242823          	sw	s2,144(s0)
   10296:	08942a23          	sw	s1,148(s0)
   1029a:	0cf42e23          	sw	a5,220(s0)
   1029e:	0c042823          	sw	zero,208(s0)
   102a2:	0c042a23          	sw	zero,212(s0)
   102a6:	0c042c23          	sw	zero,216(s0)
   102aa:	12042a23          	sw	zero,308(s0)
   102ae:	0e042023          	sw	zero,224(s0)
   102b2:	0e042223          	sw	zero,228(s0)
   102b6:	0e042423          	sw	zero,232(s0)
   102ba:	08e42223          	sw	a4,132(s0)
   102be:	21d1                	jal	10782 <memset>
   102c0:	e5018793          	addi	a5,gp,-432 # 13ad0 <__sf+0xd0>
   102c4:	0f442823          	sw	s4,240(s0)
   102c8:	0f342a23          	sw	s3,244(s0)
   102cc:	0f242c23          	sw	s2,248(s0)
   102d0:	0e942e23          	sw	s1,252(s0)
   102d4:	40f2                	lw	ra,28(sp)
   102d6:	0ef42623          	sw	a5,236(s0)
   102da:	4462                	lw	s0,24(sp)
   102dc:	44d2                	lw	s1,20(sp)
   102de:	4942                	lw	s2,16(sp)
   102e0:	49b2                	lw	s3,12(sp)
   102e2:	4a22                	lw	s4,8(sp)
   102e4:	6105                	addi	sp,sp,32
   102e6:	8082                	ret

000102e8 <__sfp>:
   102e8:	d481a783          	lw	a5,-696(gp) # 139c8 <__stdio_exit_handler>
   102ec:	1101                	addi	sp,sp,-32
   102ee:	c64e                	sw	s3,12(sp)
   102f0:	ce06                	sw	ra,28(sp)
   102f2:	cc22                	sw	s0,24(sp)
   102f4:	ca26                	sw	s1,20(sp)
   102f6:	c84a                	sw	s2,16(sp)
   102f8:	89aa                	mv	s3,a0
   102fa:	c7cd                	beqz	a5,103a4 <__sfp+0xbc>
   102fc:	694d                	lui	s2,0x13
   102fe:	48090913          	addi	s2,s2,1152 # 13480 <__sglue>
   10302:	54fd                	li	s1,-1
   10304:	00492783          	lw	a5,4(s2)
   10308:	00892403          	lw	s0,8(s2)
   1030c:	17fd                	addi	a5,a5,-1
   1030e:	0007d763          	bgez	a5,1031c <__sfp+0x34>
   10312:	a8b9                	j	10370 <__sfp+0x88>
   10314:	06840413          	addi	s0,s0,104
   10318:	04978c63          	beq	a5,s1,10370 <__sfp+0x88>
   1031c:	00c41703          	lh	a4,12(s0)
   10320:	17fd                	addi	a5,a5,-1
   10322:	fb6d                	bnez	a4,10314 <__sfp+0x2c>
   10324:	77c1                	lui	a5,0xffff0
   10326:	0785                	addi	a5,a5,1 # ffff0001 <__BSS_END__+0xfffdc311>
   10328:	06042223          	sw	zero,100(s0)
   1032c:	00042023          	sw	zero,0(s0)
   10330:	00042423          	sw	zero,8(s0)
   10334:	00042223          	sw	zero,4(s0)
   10338:	00042823          	sw	zero,16(s0)
   1033c:	00042a23          	sw	zero,20(s0)
   10340:	00042c23          	sw	zero,24(s0)
   10344:	c45c                	sw	a5,12(s0)
   10346:	4621                	li	a2,8
   10348:	4581                	li	a1,0
   1034a:	05c40513          	addi	a0,s0,92
   1034e:	2915                	jal	10782 <memset>
   10350:	02042823          	sw	zero,48(s0)
   10354:	02042a23          	sw	zero,52(s0)
   10358:	04042223          	sw	zero,68(s0)
   1035c:	04042423          	sw	zero,72(s0)
   10360:	40f2                	lw	ra,28(sp)
   10362:	8522                	mv	a0,s0
   10364:	4462                	lw	s0,24(sp)
   10366:	44d2                	lw	s1,20(sp)
   10368:	4942                	lw	s2,16(sp)
   1036a:	49b2                	lw	s3,12(sp)
   1036c:	6105                	addi	sp,sp,32
   1036e:	8082                	ret
   10370:	00092403          	lw	s0,0(s2)
   10374:	c019                	beqz	s0,1037a <__sfp+0x92>
   10376:	8922                	mv	s2,s0
   10378:	b771                	j	10304 <__sfp+0x1c>
   1037a:	1ac00593          	li	a1,428
   1037e:	854e                	mv	a0,s3
   10380:	0cf000ef          	jal	10c4e <_malloc_r>
   10384:	842a                	mv	s0,a0
   10386:	c10d                	beqz	a0,103a8 <__sfp+0xc0>
   10388:	4791                	li	a5,4
   1038a:	0531                	addi	a0,a0,12
   1038c:	00042023          	sw	zero,0(s0)
   10390:	c05c                	sw	a5,4(s0)
   10392:	c408                	sw	a0,8(s0)
   10394:	1a000613          	li	a2,416
   10398:	4581                	li	a1,0
   1039a:	26e5                	jal	10782 <memset>
   1039c:	00892023          	sw	s0,0(s2)
   103a0:	8922                	mv	s2,s0
   103a2:	b78d                	j	10304 <__sfp+0x1c>
   103a4:	3d2d                	jal	101de <global_stdio_init.part.0>
   103a6:	bf99                	j	102fc <__sfp+0x14>
   103a8:	00092023          	sw	zero,0(s2)
   103ac:	47b1                	li	a5,12
   103ae:	00f9a023          	sw	a5,0(s3)
   103b2:	b77d                	j	10360 <__sfp+0x78>

000103b4 <__sinit>:
   103b4:	595c                	lw	a5,52(a0)
   103b6:	c391                	beqz	a5,103ba <__sinit+0x6>
   103b8:	8082                	ret
   103ba:	67c1                	lui	a5,0x10
   103bc:	d481a703          	lw	a4,-696(gp) # 139c8 <__stdio_exit_handler>
   103c0:	19678793          	addi	a5,a5,406 # 10196 <cleanup_stdio>
   103c4:	d95c                	sw	a5,52(a0)
   103c6:	fb6d                	bnez	a4,103b8 <__sinit+0x4>
   103c8:	bd19                	j	101de <global_stdio_init.part.0>

000103ca <__sfp_lock_acquire>:
   103ca:	8082                	ret

000103cc <__sfp_lock_release>:
   103cc:	8082                	ret

000103ce <__fp_lock_all>:
   103ce:	664d                	lui	a2,0x13
   103d0:	65c1                	lui	a1,0x10
   103d2:	48060613          	addi	a2,a2,1152 # 13480 <__sglue>
   103d6:	17e58593          	addi	a1,a1,382 # 1017e <__fp_lock>
   103da:	4501                	li	a0,0
   103dc:	a809                	j	103ee <_fwalk_sglue>

000103de <__fp_unlock_all>:
   103de:	664d                	lui	a2,0x13
   103e0:	65c1                	lui	a1,0x10
   103e2:	48060613          	addi	a2,a2,1152 # 13480 <__sglue>
   103e6:	1da58593          	addi	a1,a1,474 # 101da <__fp_unlock>
   103ea:	4501                	li	a0,0
   103ec:	a009                	j	103ee <_fwalk_sglue>

000103ee <_fwalk_sglue>:
   103ee:	7179                	addi	sp,sp,-48
   103f0:	d04a                	sw	s2,32(sp)
   103f2:	ce4e                	sw	s3,28(sp)
   103f4:	cc52                	sw	s4,24(sp)
   103f6:	ca56                	sw	s5,20(sp)
   103f8:	c85a                	sw	s6,16(sp)
   103fa:	c65e                	sw	s7,12(sp)
   103fc:	d606                	sw	ra,44(sp)
   103fe:	d422                	sw	s0,40(sp)
   10400:	d226                	sw	s1,36(sp)
   10402:	8b2a                	mv	s6,a0
   10404:	8bae                	mv	s7,a1
   10406:	8ab2                	mv	s5,a2
   10408:	4a01                	li	s4,0
   1040a:	4985                	li	s3,1
   1040c:	597d                	li	s2,-1
   1040e:	004aa483          	lw	s1,4(s5)
   10412:	008aa403          	lw	s0,8(s5)
   10416:	14fd                	addi	s1,s1,-1
   10418:	0204c463          	bltz	s1,10440 <_fwalk_sglue+0x52>
   1041c:	00c45783          	lhu	a5,12(s0)
   10420:	00f9fb63          	bgeu	s3,a5,10436 <_fwalk_sglue+0x48>
   10424:	00e41783          	lh	a5,14(s0)
   10428:	85a2                	mv	a1,s0
   1042a:	855a                	mv	a0,s6
   1042c:	01278563          	beq	a5,s2,10436 <_fwalk_sglue+0x48>
   10430:	9b82                	jalr	s7
   10432:	00aa6a33          	or	s4,s4,a0
   10436:	14fd                	addi	s1,s1,-1
   10438:	06840413          	addi	s0,s0,104
   1043c:	ff2490e3          	bne	s1,s2,1041c <_fwalk_sglue+0x2e>
   10440:	000aaa83          	lw	s5,0(s5)
   10444:	fc0a95e3          	bnez	s5,1040e <_fwalk_sglue+0x20>
   10448:	50b2                	lw	ra,44(sp)
   1044a:	5422                	lw	s0,40(sp)
   1044c:	5492                	lw	s1,36(sp)
   1044e:	5902                	lw	s2,32(sp)
   10450:	49f2                	lw	s3,28(sp)
   10452:	4ad2                	lw	s5,20(sp)
   10454:	4b42                	lw	s6,16(sp)
   10456:	4bb2                	lw	s7,12(sp)
   10458:	8552                	mv	a0,s4
   1045a:	4a62                	lw	s4,24(sp)
   1045c:	6145                	addi	sp,sp,48
   1045e:	8082                	ret

00010460 <_puts_r>:
   10460:	7139                	addi	sp,sp,-64
   10462:	dc22                	sw	s0,56(sp)
   10464:	842a                	mv	s0,a0
   10466:	852e                	mv	a0,a1
   10468:	da26                	sw	s1,52(sp)
   1046a:	de06                	sw	ra,60(sp)
   1046c:	84ae                	mv	s1,a1
   1046e:	2e75                	jal	1082a <strlen>
   10470:	67c9                	lui	a5,0x12
   10472:	5858                	lw	a4,52(s0)
   10474:	4585                	li	a1,1
   10476:	00150813          	addi	a6,a0,1
   1047a:	46c78793          	addi	a5,a5,1132 # 1246c <__errno+0x16>
   1047e:	1010                	addi	a2,sp,32
   10480:	4689                	li	a3,2
   10482:	d62e                	sw	a1,44(sp)
   10484:	d026                	sw	s1,32(sp)
   10486:	d22a                	sw	a0,36(sp)
   10488:	ce42                	sw	a6,28(sp)
   1048a:	d43e                	sw	a5,40(sp)
   1048c:	ca32                	sw	a2,20(sp)
   1048e:	cc36                	sw	a3,24(sp)
   10490:	440c                	lw	a1,8(s0)
   10492:	c329                	beqz	a4,104d4 <_puts_r+0x74>
   10494:	00c59783          	lh	a5,12(a1)
   10498:	51f8                	lw	a4,100(a1)
   1049a:	6609                	lui	a2,0x2
   1049c:	01279693          	slli	a3,a5,0x12
   104a0:	0206c463          	bltz	a3,104c8 <_puts_r+0x68>
   104a4:	76f9                	lui	a3,0xffffe
   104a6:	16fd                	addi	a3,a3,-1 # ffffdfff <__BSS_END__+0xfffea30f>
   104a8:	8fd1                	or	a5,a5,a2
   104aa:	8f75                	and	a4,a4,a3
   104ac:	00f59623          	sh	a5,12(a1)
   104b0:	d1f8                	sw	a4,100(a1)
   104b2:	8522                	mv	a0,s0
   104b4:	0850                	addi	a2,sp,20
   104b6:	034010ef          	jal	114ea <__sfvwrite_r>
   104ba:	e919                	bnez	a0,104d0 <_puts_r+0x70>
   104bc:	4529                	li	a0,10
   104be:	50f2                	lw	ra,60(sp)
   104c0:	5462                	lw	s0,56(sp)
   104c2:	54d2                	lw	s1,52(sp)
   104c4:	6121                	addi	sp,sp,64
   104c6:	8082                	ret
   104c8:	01271793          	slli	a5,a4,0x12
   104cc:	fe07d3e3          	bgez	a5,104b2 <_puts_r+0x52>
   104d0:	557d                	li	a0,-1
   104d2:	b7f5                	j	104be <_puts_r+0x5e>
   104d4:	8522                	mv	a0,s0
   104d6:	c62e                	sw	a1,12(sp)
   104d8:	3df1                	jal	103b4 <__sinit>
   104da:	45b2                	lw	a1,12(sp)
   104dc:	bf65                	j	10494 <_puts_r+0x34>

000104de <puts>:
   104de:	85aa                	mv	a1,a0
   104e0:	d3c1a503          	lw	a0,-708(gp) # 139bc <_impure_ptr>
   104e4:	bfb5                	j	10460 <_puts_r>

000104e6 <__sread>:
   104e6:	1141                	addi	sp,sp,-16
   104e8:	c422                	sw	s0,8(sp)
   104ea:	842e                	mv	s0,a1
   104ec:	00e59583          	lh	a1,14(a1)
   104f0:	c606                	sw	ra,12(sp)
   104f2:	227d                	jal	106a0 <_read_r>
   104f4:	00054963          	bltz	a0,10506 <__sread+0x20>
   104f8:	483c                	lw	a5,80(s0)
   104fa:	40b2                	lw	ra,12(sp)
   104fc:	97aa                	add	a5,a5,a0
   104fe:	c83c                	sw	a5,80(s0)
   10500:	4422                	lw	s0,8(sp)
   10502:	0141                	addi	sp,sp,16
   10504:	8082                	ret
   10506:	00c45783          	lhu	a5,12(s0)
   1050a:	777d                	lui	a4,0xfffff
   1050c:	177d                	addi	a4,a4,-1 # ffffefff <__BSS_END__+0xfffeb30f>
   1050e:	8ff9                	and	a5,a5,a4
   10510:	40b2                	lw	ra,12(sp)
   10512:	00f41623          	sh	a5,12(s0)
   10516:	4422                	lw	s0,8(sp)
   10518:	0141                	addi	sp,sp,16
   1051a:	8082                	ret

0001051c <__seofread>:
   1051c:	4501                	li	a0,0
   1051e:	8082                	ret

00010520 <__swrite>:
   10520:	00c59783          	lh	a5,12(a1)
   10524:	1101                	addi	sp,sp,-32
   10526:	cc22                	sw	s0,24(sp)
   10528:	ca26                	sw	s1,20(sp)
   1052a:	c84a                	sw	s2,16(sp)
   1052c:	c64e                	sw	s3,12(sp)
   1052e:	ce06                	sw	ra,28(sp)
   10530:	1007f713          	andi	a4,a5,256
   10534:	842e                	mv	s0,a1
   10536:	8932                	mv	s2,a2
   10538:	89b6                	mv	s3,a3
   1053a:	84aa                	mv	s1,a0
   1053c:	e315                	bnez	a4,10560 <__swrite+0x40>
   1053e:	777d                	lui	a4,0xfffff
   10540:	177d                	addi	a4,a4,-1 # ffffefff <__BSS_END__+0xfffeb30f>
   10542:	8ff9                	and	a5,a5,a4
   10544:	00e41583          	lh	a1,14(s0)
   10548:	00f41623          	sh	a5,12(s0)
   1054c:	4462                	lw	s0,24(sp)
   1054e:	40f2                	lw	ra,28(sp)
   10550:	86ce                	mv	a3,s3
   10552:	864a                	mv	a2,s2
   10554:	49b2                	lw	s3,12(sp)
   10556:	4942                	lw	s2,16(sp)
   10558:	8526                	mv	a0,s1
   1055a:	44d2                	lw	s1,20(sp)
   1055c:	6105                	addi	sp,sp,32
   1055e:	aabd                	j	106dc <_write_r>
   10560:	00e59583          	lh	a1,14(a1)
   10564:	4689                	li	a3,2
   10566:	4601                	li	a2,0
   10568:	28f5                	jal	10664 <_lseek_r>
   1056a:	00c41783          	lh	a5,12(s0)
   1056e:	bfc1                	j	1053e <__swrite+0x1e>

00010570 <__sseek>:
   10570:	1141                	addi	sp,sp,-16
   10572:	c422                	sw	s0,8(sp)
   10574:	842e                	mv	s0,a1
   10576:	00e59583          	lh	a1,14(a1)
   1057a:	c606                	sw	ra,12(sp)
   1057c:	20e5                	jal	10664 <_lseek_r>
   1057e:	577d                	li	a4,-1
   10580:	00c41783          	lh	a5,12(s0)
   10584:	00e50b63          	beq	a0,a4,1059a <__sseek+0x2a>
   10588:	6705                	lui	a4,0x1
   1058a:	8fd9                	or	a5,a5,a4
   1058c:	40b2                	lw	ra,12(sp)
   1058e:	c828                	sw	a0,80(s0)
   10590:	00f41623          	sh	a5,12(s0)
   10594:	4422                	lw	s0,8(sp)
   10596:	0141                	addi	sp,sp,16
   10598:	8082                	ret
   1059a:	777d                	lui	a4,0xfffff
   1059c:	177d                	addi	a4,a4,-1 # ffffefff <__BSS_END__+0xfffeb30f>
   1059e:	8ff9                	and	a5,a5,a4
   105a0:	40b2                	lw	ra,12(sp)
   105a2:	00f41623          	sh	a5,12(s0)
   105a6:	4422                	lw	s0,8(sp)
   105a8:	0141                	addi	sp,sp,16
   105aa:	8082                	ret

000105ac <__sclose>:
   105ac:	00e59583          	lh	a1,14(a1)
   105b0:	a009                	j	105b2 <_close_r>

000105b2 <_close_r>:
   105b2:	1141                	addi	sp,sp,-16
   105b4:	c422                	sw	s0,8(sp)
   105b6:	c226                	sw	s1,4(sp)
   105b8:	842a                	mv	s0,a0
   105ba:	852e                	mv	a0,a1
   105bc:	c606                	sw	ra,12(sp)
   105be:	d401a623          	sw	zero,-692(gp) # 139cc <errno>
   105c2:	4ab010ef          	jal	1226c <_close>
   105c6:	57fd                	li	a5,-1
   105c8:	00f50763          	beq	a0,a5,105d6 <_close_r+0x24>
   105cc:	40b2                	lw	ra,12(sp)
   105ce:	4422                	lw	s0,8(sp)
   105d0:	4492                	lw	s1,4(sp)
   105d2:	0141                	addi	sp,sp,16
   105d4:	8082                	ret
   105d6:	d4c1a783          	lw	a5,-692(gp) # 139cc <errno>
   105da:	dbed                	beqz	a5,105cc <_close_r+0x1a>
   105dc:	40b2                	lw	ra,12(sp)
   105de:	c01c                	sw	a5,0(s0)
   105e0:	4422                	lw	s0,8(sp)
   105e2:	4492                	lw	s1,4(sp)
   105e4:	0141                	addi	sp,sp,16
   105e6:	8082                	ret

000105e8 <_reclaim_reent>:
   105e8:	d3c1a783          	lw	a5,-708(gp) # 139bc <_impure_ptr>
   105ec:	06a78b63          	beq	a5,a0,10662 <_reclaim_reent+0x7a>
   105f0:	416c                	lw	a1,68(a0)
   105f2:	1101                	addi	sp,sp,-32
   105f4:	ca26                	sw	s1,20(sp)
   105f6:	ce06                	sw	ra,28(sp)
   105f8:	cc22                	sw	s0,24(sp)
   105fa:	84aa                	mv	s1,a0
   105fc:	c59d                	beqz	a1,1062a <_reclaim_reent+0x42>
   105fe:	c84a                	sw	s2,16(sp)
   10600:	c64e                	sw	s3,12(sp)
   10602:	4901                	li	s2,0
   10604:	08000993          	li	s3,128
   10608:	012587b3          	add	a5,a1,s2
   1060c:	4380                	lw	s0,0(a5)
   1060e:	c419                	beqz	s0,1061c <_reclaim_reent+0x34>
   10610:	85a2                	mv	a1,s0
   10612:	4000                	lw	s0,0(s0)
   10614:	8526                	mv	a0,s1
   10616:	2931                	jal	10a32 <_free_r>
   10618:	fc65                	bnez	s0,10610 <_reclaim_reent+0x28>
   1061a:	40ec                	lw	a1,68(s1)
   1061c:	0911                	addi	s2,s2,4
   1061e:	ff3915e3          	bne	s2,s3,10608 <_reclaim_reent+0x20>
   10622:	8526                	mv	a0,s1
   10624:	2139                	jal	10a32 <_free_r>
   10626:	4942                	lw	s2,16(sp)
   10628:	49b2                	lw	s3,12(sp)
   1062a:	5c8c                	lw	a1,56(s1)
   1062c:	c199                	beqz	a1,10632 <_reclaim_reent+0x4a>
   1062e:	8526                	mv	a0,s1
   10630:	2109                	jal	10a32 <_free_r>
   10632:	40a0                	lw	s0,64(s1)
   10634:	c411                	beqz	s0,10640 <_reclaim_reent+0x58>
   10636:	85a2                	mv	a1,s0
   10638:	4000                	lw	s0,0(s0)
   1063a:	8526                	mv	a0,s1
   1063c:	2edd                	jal	10a32 <_free_r>
   1063e:	fc65                	bnez	s0,10636 <_reclaim_reent+0x4e>
   10640:	44ec                	lw	a1,76(s1)
   10642:	c199                	beqz	a1,10648 <_reclaim_reent+0x60>
   10644:	8526                	mv	a0,s1
   10646:	26f5                	jal	10a32 <_free_r>
   10648:	58dc                	lw	a5,52(s1)
   1064a:	c799                	beqz	a5,10658 <_reclaim_reent+0x70>
   1064c:	4462                	lw	s0,24(sp)
   1064e:	40f2                	lw	ra,28(sp)
   10650:	8526                	mv	a0,s1
   10652:	44d2                	lw	s1,20(sp)
   10654:	6105                	addi	sp,sp,32
   10656:	8782                	jr	a5
   10658:	40f2                	lw	ra,28(sp)
   1065a:	4462                	lw	s0,24(sp)
   1065c:	44d2                	lw	s1,20(sp)
   1065e:	6105                	addi	sp,sp,32
   10660:	8082                	ret
   10662:	8082                	ret

00010664 <_lseek_r>:
   10664:	1141                	addi	sp,sp,-16
   10666:	872e                	mv	a4,a1
   10668:	c422                	sw	s0,8(sp)
   1066a:	c226                	sw	s1,4(sp)
   1066c:	85b2                	mv	a1,a2
   1066e:	842a                	mv	s0,a0
   10670:	8636                	mv	a2,a3
   10672:	853a                	mv	a0,a4
   10674:	c606                	sw	ra,12(sp)
   10676:	d401a623          	sw	zero,-692(gp) # 139cc <errno>
   1067a:	497010ef          	jal	12310 <_lseek>
   1067e:	57fd                	li	a5,-1
   10680:	00f50763          	beq	a0,a5,1068e <_lseek_r+0x2a>
   10684:	40b2                	lw	ra,12(sp)
   10686:	4422                	lw	s0,8(sp)
   10688:	4492                	lw	s1,4(sp)
   1068a:	0141                	addi	sp,sp,16
   1068c:	8082                	ret
   1068e:	d4c1a783          	lw	a5,-692(gp) # 139cc <errno>
   10692:	dbed                	beqz	a5,10684 <_lseek_r+0x20>
   10694:	40b2                	lw	ra,12(sp)
   10696:	c01c                	sw	a5,0(s0)
   10698:	4422                	lw	s0,8(sp)
   1069a:	4492                	lw	s1,4(sp)
   1069c:	0141                	addi	sp,sp,16
   1069e:	8082                	ret

000106a0 <_read_r>:
   106a0:	1141                	addi	sp,sp,-16
   106a2:	872e                	mv	a4,a1
   106a4:	c422                	sw	s0,8(sp)
   106a6:	c226                	sw	s1,4(sp)
   106a8:	85b2                	mv	a1,a2
   106aa:	842a                	mv	s0,a0
   106ac:	8636                	mv	a2,a3
   106ae:	853a                	mv	a0,a4
   106b0:	c606                	sw	ra,12(sp)
   106b2:	d401a623          	sw	zero,-692(gp) # 139cc <errno>
   106b6:	485010ef          	jal	1233a <_read>
   106ba:	57fd                	li	a5,-1
   106bc:	00f50763          	beq	a0,a5,106ca <_read_r+0x2a>
   106c0:	40b2                	lw	ra,12(sp)
   106c2:	4422                	lw	s0,8(sp)
   106c4:	4492                	lw	s1,4(sp)
   106c6:	0141                	addi	sp,sp,16
   106c8:	8082                	ret
   106ca:	d4c1a783          	lw	a5,-692(gp) # 139cc <errno>
   106ce:	dbed                	beqz	a5,106c0 <_read_r+0x20>
   106d0:	40b2                	lw	ra,12(sp)
   106d2:	c01c                	sw	a5,0(s0)
   106d4:	4422                	lw	s0,8(sp)
   106d6:	4492                	lw	s1,4(sp)
   106d8:	0141                	addi	sp,sp,16
   106da:	8082                	ret

000106dc <_write_r>:
   106dc:	1141                	addi	sp,sp,-16
   106de:	872e                	mv	a4,a1
   106e0:	c422                	sw	s0,8(sp)
   106e2:	c226                	sw	s1,4(sp)
   106e4:	85b2                	mv	a1,a2
   106e6:	842a                	mv	s0,a0
   106e8:	8636                	mv	a2,a3
   106ea:	853a                	mv	a0,a4
   106ec:	c606                	sw	ra,12(sp)
   106ee:	d401a623          	sw	zero,-692(gp) # 139cc <errno>
   106f2:	4c5010ef          	jal	123b6 <_write>
   106f6:	57fd                	li	a5,-1
   106f8:	00f50763          	beq	a0,a5,10706 <_write_r+0x2a>
   106fc:	40b2                	lw	ra,12(sp)
   106fe:	4422                	lw	s0,8(sp)
   10700:	4492                	lw	s1,4(sp)
   10702:	0141                	addi	sp,sp,16
   10704:	8082                	ret
   10706:	d4c1a783          	lw	a5,-692(gp) # 139cc <errno>
   1070a:	dbed                	beqz	a5,106fc <_write_r+0x20>
   1070c:	40b2                	lw	ra,12(sp)
   1070e:	c01c                	sw	a5,0(s0)
   10710:	4422                	lw	s0,8(sp)
   10712:	4492                	lw	s1,4(sp)
   10714:	0141                	addi	sp,sp,16
   10716:	8082                	ret

00010718 <__libc_init_array>:
   10718:	1141                	addi	sp,sp,-16
   1071a:	c422                	sw	s0,8(sp)
   1071c:	67cd                	lui	a5,0x13
   1071e:	644d                	lui	s0,0x13
   10720:	c04a                	sw	s2,0(sp)
   10722:	47478793          	addi	a5,a5,1140 # 13474 <__init_array_start>
   10726:	47440713          	addi	a4,s0,1140 # 13474 <__init_array_start>
   1072a:	c606                	sw	ra,12(sp)
   1072c:	c226                	sw	s1,4(sp)
   1072e:	40e78933          	sub	s2,a5,a4
   10732:	00e78d63          	beq	a5,a4,1074c <__libc_init_array+0x34>
   10736:	40295913          	srai	s2,s2,0x2
   1073a:	47440413          	addi	s0,s0,1140
   1073e:	4481                	li	s1,0
   10740:	401c                	lw	a5,0(s0)
   10742:	0485                	addi	s1,s1,1
   10744:	0411                	addi	s0,s0,4
   10746:	9782                	jalr	a5
   10748:	ff24ece3          	bltu	s1,s2,10740 <__libc_init_array+0x28>
   1074c:	67cd                	lui	a5,0x13
   1074e:	644d                	lui	s0,0x13
   10750:	47c78793          	addi	a5,a5,1148 # 1347c <__do_global_dtors_aux_fini_array_entry>
   10754:	47440713          	addi	a4,s0,1140 # 13474 <__init_array_start>
   10758:	40e78933          	sub	s2,a5,a4
   1075c:	40295913          	srai	s2,s2,0x2
   10760:	00e78b63          	beq	a5,a4,10776 <__libc_init_array+0x5e>
   10764:	47440413          	addi	s0,s0,1140
   10768:	4481                	li	s1,0
   1076a:	401c                	lw	a5,0(s0)
   1076c:	0485                	addi	s1,s1,1
   1076e:	0411                	addi	s0,s0,4
   10770:	9782                	jalr	a5
   10772:	ff24ece3          	bltu	s1,s2,1076a <__libc_init_array+0x52>
   10776:	40b2                	lw	ra,12(sp)
   10778:	4422                	lw	s0,8(sp)
   1077a:	4492                	lw	s1,4(sp)
   1077c:	4902                	lw	s2,0(sp)
   1077e:	0141                	addi	sp,sp,16
   10780:	8082                	ret

00010782 <memset>:
   10782:	433d                	li	t1,15
   10784:	872a                	mv	a4,a0
   10786:	02c37363          	bgeu	t1,a2,107ac <memset+0x2a>
   1078a:	00f77793          	andi	a5,a4,15
   1078e:	efbd                	bnez	a5,1080c <memset+0x8a>
   10790:	e5ad                	bnez	a1,107fa <memset+0x78>
   10792:	ff067693          	andi	a3,a2,-16
   10796:	8a3d                	andi	a2,a2,15
   10798:	96ba                	add	a3,a3,a4
   1079a:	c30c                	sw	a1,0(a4)
   1079c:	c34c                	sw	a1,4(a4)
   1079e:	c70c                	sw	a1,8(a4)
   107a0:	c74c                	sw	a1,12(a4)
   107a2:	0741                	addi	a4,a4,16
   107a4:	fed76be3          	bltu	a4,a3,1079a <memset+0x18>
   107a8:	e211                	bnez	a2,107ac <memset+0x2a>
   107aa:	8082                	ret
   107ac:	40c306b3          	sub	a3,t1,a2
   107b0:	068a                	slli	a3,a3,0x2
   107b2:	00000297          	auipc	t0,0x0
   107b6:	9696                	add	a3,a3,t0
   107b8:	00a68067          	jr	10(a3)
   107bc:	00b70723          	sb	a1,14(a4)
   107c0:	00b706a3          	sb	a1,13(a4)
   107c4:	00b70623          	sb	a1,12(a4)
   107c8:	00b705a3          	sb	a1,11(a4)
   107cc:	00b70523          	sb	a1,10(a4)
   107d0:	00b704a3          	sb	a1,9(a4)
   107d4:	00b70423          	sb	a1,8(a4)
   107d8:	00b703a3          	sb	a1,7(a4)
   107dc:	00b70323          	sb	a1,6(a4)
   107e0:	00b702a3          	sb	a1,5(a4)
   107e4:	00b70223          	sb	a1,4(a4)
   107e8:	00b701a3          	sb	a1,3(a4)
   107ec:	00b70123          	sb	a1,2(a4)
   107f0:	00b700a3          	sb	a1,1(a4)
   107f4:	00b70023          	sb	a1,0(a4)
   107f8:	8082                	ret
   107fa:	0ff5f593          	zext.b	a1,a1
   107fe:	00859693          	slli	a3,a1,0x8
   10802:	8dd5                	or	a1,a1,a3
   10804:	01059693          	slli	a3,a1,0x10
   10808:	8dd5                	or	a1,a1,a3
   1080a:	b761                	j	10792 <memset+0x10>
   1080c:	00279693          	slli	a3,a5,0x2
   10810:	00000297          	auipc	t0,0x0
   10814:	9696                	add	a3,a3,t0
   10816:	8286                	mv	t0,ra
   10818:	fa8680e7          	jalr	-88(a3)
   1081c:	8096                	mv	ra,t0
   1081e:	17c1                	addi	a5,a5,-16
   10820:	8f1d                	sub	a4,a4,a5
   10822:	963e                	add	a2,a2,a5
   10824:	f8c374e3          	bgeu	t1,a2,107ac <memset+0x2a>
   10828:	b7a5                	j	10790 <memset+0xe>

0001082a <strlen>:
   1082a:	00357793          	andi	a5,a0,3
   1082e:	872a                	mv	a4,a0
   10830:	ef9d                	bnez	a5,1086e <strlen+0x44>
   10832:	7f7f86b7          	lui	a3,0x7f7f8
   10836:	f7f68693          	addi	a3,a3,-129 # 7f7f7f7f <__BSS_END__+0x7f7e428f>
   1083a:	55fd                	li	a1,-1
   1083c:	4310                	lw	a2,0(a4)
   1083e:	0711                	addi	a4,a4,4
   10840:	00d677b3          	and	a5,a2,a3
   10844:	97b6                	add	a5,a5,a3
   10846:	8fd1                	or	a5,a5,a2
   10848:	8fd5                	or	a5,a5,a3
   1084a:	feb789e3          	beq	a5,a1,1083c <strlen+0x12>
   1084e:	ffc74683          	lbu	a3,-4(a4)
   10852:	40a707b3          	sub	a5,a4,a0
   10856:	ca8d                	beqz	a3,10888 <strlen+0x5e>
   10858:	ffd74683          	lbu	a3,-3(a4)
   1085c:	c29d                	beqz	a3,10882 <strlen+0x58>
   1085e:	ffe74503          	lbu	a0,-2(a4)
   10862:	00a03533          	snez	a0,a0
   10866:	953e                	add	a0,a0,a5
   10868:	1579                	addi	a0,a0,-2
   1086a:	8082                	ret
   1086c:	d2f9                	beqz	a3,10832 <strlen+0x8>
   1086e:	00074783          	lbu	a5,0(a4)
   10872:	0705                	addi	a4,a4,1
   10874:	00377693          	andi	a3,a4,3
   10878:	fbf5                	bnez	a5,1086c <strlen+0x42>
   1087a:	8f09                	sub	a4,a4,a0
   1087c:	fff70513          	addi	a0,a4,-1
   10880:	8082                	ret
   10882:	ffd78513          	addi	a0,a5,-3
   10886:	8082                	ret
   10888:	ffc78513          	addi	a0,a5,-4
   1088c:	8082                	ret

0001088e <__call_exitprocs>:
   1088e:	7179                	addi	sp,sp,-48
   10890:	cc52                	sw	s4,24(sp)
   10892:	d04a                	sw	s2,32(sp)
   10894:	d501a903          	lw	s2,-688(gp) # 139d0 <__atexit>
   10898:	d606                	sw	ra,44(sp)
   1089a:	04090663          	beqz	s2,108e6 <__call_exitprocs+0x58>
   1089e:	ce4e                	sw	s3,28(sp)
   108a0:	ca56                	sw	s5,20(sp)
   108a2:	c85a                	sw	s6,16(sp)
   108a4:	c65e                	sw	s7,12(sp)
   108a6:	d422                	sw	s0,40(sp)
   108a8:	d226                	sw	s1,36(sp)
   108aa:	c462                	sw	s8,8(sp)
   108ac:	8b2a                	mv	s6,a0
   108ae:	8bae                	mv	s7,a1
   108b0:	59fd                	li	s3,-1
   108b2:	4a85                	li	s5,1
   108b4:	00492483          	lw	s1,4(s2)
   108b8:	fff48413          	addi	s0,s1,-1
   108bc:	00044e63          	bltz	s0,108d8 <__call_exitprocs+0x4a>
   108c0:	048a                	slli	s1,s1,0x2
   108c2:	94ca                	add	s1,s1,s2
   108c4:	020b8663          	beqz	s7,108f0 <__call_exitprocs+0x62>
   108c8:	1044a783          	lw	a5,260(s1)
   108cc:	03778263          	beq	a5,s7,108f0 <__call_exitprocs+0x62>
   108d0:	147d                	addi	s0,s0,-1
   108d2:	14f1                	addi	s1,s1,-4
   108d4:	ff341ae3          	bne	s0,s3,108c8 <__call_exitprocs+0x3a>
   108d8:	5422                	lw	s0,40(sp)
   108da:	5492                	lw	s1,36(sp)
   108dc:	49f2                	lw	s3,28(sp)
   108de:	4ad2                	lw	s5,20(sp)
   108e0:	4b42                	lw	s6,16(sp)
   108e2:	4bb2                	lw	s7,12(sp)
   108e4:	4c22                	lw	s8,8(sp)
   108e6:	50b2                	lw	ra,44(sp)
   108e8:	5902                	lw	s2,32(sp)
   108ea:	4a62                	lw	s4,24(sp)
   108ec:	6145                	addi	sp,sp,48
   108ee:	8082                	ret
   108f0:	00492783          	lw	a5,4(s2)
   108f4:	40d4                	lw	a3,4(s1)
   108f6:	17fd                	addi	a5,a5,-1
   108f8:	04878c63          	beq	a5,s0,10950 <__call_exitprocs+0xc2>
   108fc:	0004a223          	sw	zero,4(s1)
   10900:	c295                	beqz	a3,10924 <__call_exitprocs+0x96>
   10902:	18892783          	lw	a5,392(s2)
   10906:	008a9733          	sll	a4,s5,s0
   1090a:	00492c03          	lw	s8,4(s2)
   1090e:	8ff9                	and	a5,a5,a4
   10910:	ef99                	bnez	a5,1092e <__call_exitprocs+0xa0>
   10912:	9682                	jalr	a3
   10914:	00492703          	lw	a4,4(s2)
   10918:	d501a783          	lw	a5,-688(gp) # 139d0 <__atexit>
   1091c:	03871763          	bne	a4,s8,1094a <__call_exitprocs+0xbc>
   10920:	03279563          	bne	a5,s2,1094a <__call_exitprocs+0xbc>
   10924:	147d                	addi	s0,s0,-1
   10926:	14f1                	addi	s1,s1,-4
   10928:	f9341ee3          	bne	s0,s3,108c4 <__call_exitprocs+0x36>
   1092c:	b775                	j	108d8 <__call_exitprocs+0x4a>
   1092e:	18c92783          	lw	a5,396(s2)
   10932:	0844a583          	lw	a1,132(s1)
   10936:	8f7d                	and	a4,a4,a5
   10938:	ef19                	bnez	a4,10956 <__call_exitprocs+0xc8>
   1093a:	855a                	mv	a0,s6
   1093c:	9682                	jalr	a3
   1093e:	00492703          	lw	a4,4(s2)
   10942:	d501a783          	lw	a5,-688(gp) # 139d0 <__atexit>
   10946:	fd870de3          	beq	a4,s8,10920 <__call_exitprocs+0x92>
   1094a:	d7d9                	beqz	a5,108d8 <__call_exitprocs+0x4a>
   1094c:	893e                	mv	s2,a5
   1094e:	b79d                	j	108b4 <__call_exitprocs+0x26>
   10950:	00892223          	sw	s0,4(s2)
   10954:	b775                	j	10900 <__call_exitprocs+0x72>
   10956:	852e                	mv	a0,a1
   10958:	9682                	jalr	a3
   1095a:	bf6d                	j	10914 <__call_exitprocs+0x86>

0001095c <atexit>:
   1095c:	85aa                	mv	a1,a0
   1095e:	4681                	li	a3,0
   10960:	4601                	li	a2,0
   10962:	4501                	li	a0,0
   10964:	2a60106f          	j	11c0a <__register_exitproc>

00010968 <_malloc_trim_r>:
   10968:	1101                	addi	sp,sp,-32
   1096a:	c64e                	sw	s3,12(sp)
   1096c:	69cd                	lui	s3,0x13
   1096e:	cc22                	sw	s0,24(sp)
   10970:	ca26                	sw	s1,20(sp)
   10972:	c84a                	sw	s2,16(sp)
   10974:	c452                	sw	s4,8(sp)
   10976:	ce06                	sw	ra,28(sp)
   10978:	8a2e                	mv	s4,a1
   1097a:	892a                	mv	s2,a0
   1097c:	5b098993          	addi	s3,s3,1456 # 135b0 <__malloc_av_>
   10980:	09b000ef          	jal	1121a <__malloc_lock>
   10984:	0089a783          	lw	a5,8(s3)
   10988:	6405                	lui	s0,0x1
   1098a:	143d                	addi	s0,s0,-17 # fef <exit-0xf0c5>
   1098c:	43c4                	lw	s1,4(a5)
   1098e:	6785                	lui	a5,0x1
   10990:	98f1                	andi	s1,s1,-4
   10992:	9426                	add	s0,s0,s1
   10994:	41440433          	sub	s0,s0,s4
   10998:	8031                	srli	s0,s0,0xc
   1099a:	147d                	addi	s0,s0,-1
   1099c:	0432                	slli	s0,s0,0xc
   1099e:	00f44b63          	blt	s0,a5,109b4 <_malloc_trim_r+0x4c>
   109a2:	4581                	li	a1,0
   109a4:	854a                	mv	a0,s2
   109a6:	032010ef          	jal	119d8 <_sbrk_r>
   109aa:	0089a783          	lw	a5,8(s3)
   109ae:	97a6                	add	a5,a5,s1
   109b0:	00f50e63          	beq	a0,a5,109cc <_malloc_trim_r+0x64>
   109b4:	854a                	mv	a0,s2
   109b6:	067000ef          	jal	1121c <__malloc_unlock>
   109ba:	40f2                	lw	ra,28(sp)
   109bc:	4462                	lw	s0,24(sp)
   109be:	44d2                	lw	s1,20(sp)
   109c0:	4942                	lw	s2,16(sp)
   109c2:	49b2                	lw	s3,12(sp)
   109c4:	4a22                	lw	s4,8(sp)
   109c6:	4501                	li	a0,0
   109c8:	6105                	addi	sp,sp,32
   109ca:	8082                	ret
   109cc:	408005b3          	neg	a1,s0
   109d0:	854a                	mv	a0,s2
   109d2:	006010ef          	jal	119d8 <_sbrk_r>
   109d6:	57fd                	li	a5,-1
   109d8:	02f50963          	beq	a0,a5,10a0a <_malloc_trim_r+0xa2>
   109dc:	eb818793          	addi	a5,gp,-328 # 13b38 <__malloc_current_mallinfo>
   109e0:	0089a683          	lw	a3,8(s3)
   109e4:	4398                	lw	a4,0(a5)
   109e6:	8c81                	sub	s1,s1,s0
   109e8:	0014e493          	ori	s1,s1,1
   109ec:	854a                	mv	a0,s2
   109ee:	8f01                	sub	a4,a4,s0
   109f0:	c2c4                	sw	s1,4(a3)
   109f2:	c398                	sw	a4,0(a5)
   109f4:	029000ef          	jal	1121c <__malloc_unlock>
   109f8:	40f2                	lw	ra,28(sp)
   109fa:	4462                	lw	s0,24(sp)
   109fc:	44d2                	lw	s1,20(sp)
   109fe:	4942                	lw	s2,16(sp)
   10a00:	49b2                	lw	s3,12(sp)
   10a02:	4a22                	lw	s4,8(sp)
   10a04:	4505                	li	a0,1
   10a06:	6105                	addi	sp,sp,32
   10a08:	8082                	ret
   10a0a:	4581                	li	a1,0
   10a0c:	854a                	mv	a0,s2
   10a0e:	7cb000ef          	jal	119d8 <_sbrk_r>
   10a12:	0089a703          	lw	a4,8(s3)
   10a16:	46bd                	li	a3,15
   10a18:	40e507b3          	sub	a5,a0,a4
   10a1c:	f8f6dce3          	bge	a3,a5,109b4 <_malloc_trim_r+0x4c>
   10a20:	d401a603          	lw	a2,-704(gp) # 139c0 <__malloc_sbrk_base>
   10a24:	0017e793          	ori	a5,a5,1
   10a28:	8d11                	sub	a0,a0,a2
   10a2a:	c35c                	sw	a5,4(a4)
   10a2c:	eaa1ac23          	sw	a0,-328(gp) # 13b38 <__malloc_current_mallinfo>
   10a30:	b751                	j	109b4 <_malloc_trim_r+0x4c>

00010a32 <_free_r>:
   10a32:	cdf9                	beqz	a1,10b10 <_free_r+0xde>
   10a34:	1141                	addi	sp,sp,-16
   10a36:	c422                	sw	s0,8(sp)
   10a38:	c226                	sw	s1,4(sp)
   10a3a:	842e                	mv	s0,a1
   10a3c:	84aa                	mv	s1,a0
   10a3e:	c606                	sw	ra,12(sp)
   10a40:	7da000ef          	jal	1121a <__malloc_lock>
   10a44:	ffc42503          	lw	a0,-4(s0)
   10a48:	ff840713          	addi	a4,s0,-8
   10a4c:	65cd                	lui	a1,0x13
   10a4e:	ffe57793          	andi	a5,a0,-2
   10a52:	00f70633          	add	a2,a4,a5
   10a56:	5b058593          	addi	a1,a1,1456 # 135b0 <__malloc_av_>
   10a5a:	4254                	lw	a3,4(a2)
   10a5c:	0085a803          	lw	a6,8(a1)
   10a60:	00157893          	andi	a7,a0,1
   10a64:	9af1                	andi	a3,a3,-4
   10a66:	12c80063          	beq	a6,a2,10b86 <_free_r+0x154>
   10a6a:	c254                	sw	a3,4(a2)
   10a6c:	00d60833          	add	a6,a2,a3
   10a70:	00482803          	lw	a6,4(a6)
   10a74:	00187813          	andi	a6,a6,1
   10a78:	06089763          	bnez	a7,10ae6 <_free_r+0xb4>
   10a7c:	ff842303          	lw	t1,-8(s0)
   10a80:	654d                	lui	a0,0x13
   10a82:	5b850513          	addi	a0,a0,1464 # 135b8 <__malloc_av_+0x8>
   10a86:	40670733          	sub	a4,a4,t1
   10a8a:	00872883          	lw	a7,8(a4)
   10a8e:	979a                	add	a5,a5,t1
   10a90:	0ca88e63          	beq	a7,a0,10b6c <_free_r+0x13a>
   10a94:	00c72303          	lw	t1,12(a4)
   10a98:	0068a623          	sw	t1,12(a7)
   10a9c:	01132423          	sw	a7,8(t1) # 10160 <frame_dummy+0x18>
   10aa0:	10080b63          	beqz	a6,10bb6 <_free_r+0x184>
   10aa4:	0017e693          	ori	a3,a5,1
   10aa8:	c354                	sw	a3,4(a4)
   10aaa:	c21c                	sw	a5,0(a2)
   10aac:	1ff00693          	li	a3,511
   10ab0:	06f6ea63          	bltu	a3,a5,10b24 <_free_r+0xf2>
   10ab4:	ff87f693          	andi	a3,a5,-8
   10ab8:	06a1                	addi	a3,a3,8
   10aba:	41c8                	lw	a0,4(a1)
   10abc:	96ae                	add	a3,a3,a1
   10abe:	4290                	lw	a2,0(a3)
   10ac0:	0057d813          	srli	a6,a5,0x5
   10ac4:	4785                	li	a5,1
   10ac6:	010797b3          	sll	a5,a5,a6
   10aca:	8fc9                	or	a5,a5,a0
   10acc:	ff868513          	addi	a0,a3,-8
   10ad0:	c710                	sw	a2,8(a4)
   10ad2:	c748                	sw	a0,12(a4)
   10ad4:	c1dc                	sw	a5,4(a1)
   10ad6:	c298                	sw	a4,0(a3)
   10ad8:	c658                	sw	a4,12(a2)
   10ada:	4422                	lw	s0,8(sp)
   10adc:	40b2                	lw	ra,12(sp)
   10ade:	8526                	mv	a0,s1
   10ae0:	4492                	lw	s1,4(sp)
   10ae2:	0141                	addi	sp,sp,16
   10ae4:	af25                	j	1121c <__malloc_unlock>
   10ae6:	02081663          	bnez	a6,10b12 <_free_r+0xe0>
   10aea:	654d                	lui	a0,0x13
   10aec:	97b6                	add	a5,a5,a3
   10aee:	5b850513          	addi	a0,a0,1464 # 135b8 <__malloc_av_+0x8>
   10af2:	4614                	lw	a3,8(a2)
   10af4:	0017e893          	ori	a7,a5,1
   10af8:	00f70833          	add	a6,a4,a5
   10afc:	0ea68963          	beq	a3,a0,10bee <_free_r+0x1bc>
   10b00:	4650                	lw	a2,12(a2)
   10b02:	c6d0                	sw	a2,12(a3)
   10b04:	c614                	sw	a3,8(a2)
   10b06:	01172223          	sw	a7,4(a4)
   10b0a:	00f82023          	sw	a5,0(a6)
   10b0e:	bf79                	j	10aac <_free_r+0x7a>
   10b10:	8082                	ret
   10b12:	00156513          	ori	a0,a0,1
   10b16:	fea42e23          	sw	a0,-4(s0)
   10b1a:	c21c                	sw	a5,0(a2)
   10b1c:	1ff00693          	li	a3,511
   10b20:	f8f6fae3          	bgeu	a3,a5,10ab4 <_free_r+0x82>
   10b24:	0097d693          	srli	a3,a5,0x9
   10b28:	4611                	li	a2,4
   10b2a:	08d66863          	bltu	a2,a3,10bba <_free_r+0x188>
   10b2e:	0067d693          	srli	a3,a5,0x6
   10b32:	03968513          	addi	a0,a3,57
   10b36:	050e                	slli	a0,a0,0x3
   10b38:	03868613          	addi	a2,a3,56
   10b3c:	952e                	add	a0,a0,a1
   10b3e:	4114                	lw	a3,0(a0)
   10b40:	1561                	addi	a0,a0,-8
   10b42:	00d51663          	bne	a0,a3,10b4e <_free_r+0x11c>
   10b46:	a86d                	j	10c00 <_free_r+0x1ce>
   10b48:	4694                	lw	a3,8(a3)
   10b4a:	00d50663          	beq	a0,a3,10b56 <_free_r+0x124>
   10b4e:	42d0                	lw	a2,4(a3)
   10b50:	9a71                	andi	a2,a2,-4
   10b52:	fec7ebe3          	bltu	a5,a2,10b48 <_free_r+0x116>
   10b56:	46c8                	lw	a0,12(a3)
   10b58:	c748                	sw	a0,12(a4)
   10b5a:	c714                	sw	a3,8(a4)
   10b5c:	4422                	lw	s0,8(sp)
   10b5e:	c518                	sw	a4,8(a0)
   10b60:	40b2                	lw	ra,12(sp)
   10b62:	8526                	mv	a0,s1
   10b64:	4492                	lw	s1,4(sp)
   10b66:	c6d8                	sw	a4,12(a3)
   10b68:	0141                	addi	sp,sp,16
   10b6a:	ad4d                	j	1121c <__malloc_unlock>
   10b6c:	06081663          	bnez	a6,10bd8 <_free_r+0x1a6>
   10b70:	464c                	lw	a1,12(a2)
   10b72:	4610                	lw	a2,8(a2)
   10b74:	96be                	add	a3,a3,a5
   10b76:	0016e793          	ori	a5,a3,1
   10b7a:	c64c                	sw	a1,12(a2)
   10b7c:	c590                	sw	a2,8(a1)
   10b7e:	c35c                	sw	a5,4(a4)
   10b80:	9736                	add	a4,a4,a3
   10b82:	c314                	sw	a3,0(a4)
   10b84:	bf99                	j	10ada <_free_r+0xa8>
   10b86:	96be                	add	a3,a3,a5
   10b88:	00089a63          	bnez	a7,10b9c <_free_r+0x16a>
   10b8c:	ff842503          	lw	a0,-8(s0)
   10b90:	8f09                	sub	a4,a4,a0
   10b92:	475c                	lw	a5,12(a4)
   10b94:	4710                	lw	a2,8(a4)
   10b96:	96aa                	add	a3,a3,a0
   10b98:	c65c                	sw	a5,12(a2)
   10b9a:	c790                	sw	a2,8(a5)
   10b9c:	0016e613          	ori	a2,a3,1
   10ba0:	d441a783          	lw	a5,-700(gp) # 139c4 <__malloc_trim_threshold>
   10ba4:	c350                	sw	a2,4(a4)
   10ba6:	c598                	sw	a4,8(a1)
   10ba8:	f2f6e9e3          	bltu	a3,a5,10ada <_free_r+0xa8>
   10bac:	d5c1a583          	lw	a1,-676(gp) # 139dc <__malloc_top_pad>
   10bb0:	8526                	mv	a0,s1
   10bb2:	3b5d                	jal	10968 <_malloc_trim_r>
   10bb4:	b71d                	j	10ada <_free_r+0xa8>
   10bb6:	97b6                	add	a5,a5,a3
   10bb8:	bf2d                	j	10af2 <_free_r+0xc0>
   10bba:	4651                	li	a2,20
   10bbc:	02d67363          	bgeu	a2,a3,10be2 <_free_r+0x1b0>
   10bc0:	05400613          	li	a2,84
   10bc4:	04d66863          	bltu	a2,a3,10c14 <_free_r+0x1e2>
   10bc8:	00c7d693          	srli	a3,a5,0xc
   10bcc:	06f68513          	addi	a0,a3,111
   10bd0:	050e                	slli	a0,a0,0x3
   10bd2:	06e68613          	addi	a2,a3,110
   10bd6:	b79d                	j	10b3c <_free_r+0x10a>
   10bd8:	0017e693          	ori	a3,a5,1
   10bdc:	c354                	sw	a3,4(a4)
   10bde:	c21c                	sw	a5,0(a2)
   10be0:	bded                	j	10ada <_free_r+0xa8>
   10be2:	05c68513          	addi	a0,a3,92
   10be6:	050e                	slli	a0,a0,0x3
   10be8:	05b68613          	addi	a2,a3,91
   10bec:	bf81                	j	10b3c <_free_r+0x10a>
   10bee:	c9d8                	sw	a4,20(a1)
   10bf0:	c998                	sw	a4,16(a1)
   10bf2:	c748                	sw	a0,12(a4)
   10bf4:	c708                	sw	a0,8(a4)
   10bf6:	01172223          	sw	a7,4(a4)
   10bfa:	00f82023          	sw	a5,0(a6)
   10bfe:	bdf1                	j	10ada <_free_r+0xa8>
   10c00:	0045a803          	lw	a6,4(a1)
   10c04:	8609                	srai	a2,a2,0x2
   10c06:	4785                	li	a5,1
   10c08:	00c797b3          	sll	a5,a5,a2
   10c0c:	0107e7b3          	or	a5,a5,a6
   10c10:	c1dc                	sw	a5,4(a1)
   10c12:	b799                	j	10b58 <_free_r+0x126>
   10c14:	15400613          	li	a2,340
   10c18:	00d66a63          	bltu	a2,a3,10c2c <_free_r+0x1fa>
   10c1c:	00f7d693          	srli	a3,a5,0xf
   10c20:	07868513          	addi	a0,a3,120
   10c24:	050e                	slli	a0,a0,0x3
   10c26:	07768613          	addi	a2,a3,119
   10c2a:	bf09                	j	10b3c <_free_r+0x10a>
   10c2c:	55400613          	li	a2,1364
   10c30:	00d66a63          	bltu	a2,a3,10c44 <_free_r+0x212>
   10c34:	0127d693          	srli	a3,a5,0x12
   10c38:	07d68513          	addi	a0,a3,125
   10c3c:	050e                	slli	a0,a0,0x3
   10c3e:	07c68613          	addi	a2,a3,124
   10c42:	bded                	j	10b3c <_free_r+0x10a>
   10c44:	3f800513          	li	a0,1016
   10c48:	07e00613          	li	a2,126
   10c4c:	bdc5                	j	10b3c <_free_r+0x10a>

00010c4e <_malloc_r>:
   10c4e:	7179                	addi	sp,sp,-48
   10c50:	ce4e                	sw	s3,28(sp)
   10c52:	d606                	sw	ra,44(sp)
   10c54:	d422                	sw	s0,40(sp)
   10c56:	d226                	sw	s1,36(sp)
   10c58:	d04a                	sw	s2,32(sp)
   10c5a:	00b58793          	addi	a5,a1,11
   10c5e:	4759                	li	a4,22
   10c60:	89aa                	mv	s3,a0
   10c62:	04f76763          	bltu	a4,a5,10cb0 <_malloc_r+0x62>
   10c66:	44c1                	li	s1,16
   10c68:	16b4e763          	bltu	s1,a1,10dd6 <_malloc_r+0x188>
   10c6c:	237d                	jal	1121a <__malloc_lock>
   10c6e:	47e1                	li	a5,24
   10c70:	4589                	li	a1,2
   10c72:	694d                	lui	s2,0x13
   10c74:	5b090913          	addi	s2,s2,1456 # 135b0 <__malloc_av_>
   10c78:	97ca                	add	a5,a5,s2
   10c7a:	43c0                	lw	s0,4(a5)
   10c7c:	ff878713          	addi	a4,a5,-8 # ff8 <exit-0xf0bc>
   10c80:	30e40663          	beq	s0,a4,10f8c <_malloc_r+0x33e>
   10c84:	405c                	lw	a5,4(s0)
   10c86:	4454                	lw	a3,12(s0)
   10c88:	4410                	lw	a2,8(s0)
   10c8a:	9bf1                	andi	a5,a5,-4
   10c8c:	97a2                	add	a5,a5,s0
   10c8e:	43d8                	lw	a4,4(a5)
   10c90:	c654                	sw	a3,12(a2)
   10c92:	c690                	sw	a2,8(a3)
   10c94:	00176713          	ori	a4,a4,1
   10c98:	854e                	mv	a0,s3
   10c9a:	c3d8                	sw	a4,4(a5)
   10c9c:	2341                	jal	1121c <__malloc_unlock>
   10c9e:	50b2                	lw	ra,44(sp)
   10ca0:	00840513          	addi	a0,s0,8
   10ca4:	5422                	lw	s0,40(sp)
   10ca6:	5492                	lw	s1,36(sp)
   10ca8:	5902                	lw	s2,32(sp)
   10caa:	49f2                	lw	s3,28(sp)
   10cac:	6145                	addi	sp,sp,48
   10cae:	8082                	ret
   10cb0:	ff87f493          	andi	s1,a5,-8
   10cb4:	1207c163          	bltz	a5,10dd6 <_malloc_r+0x188>
   10cb8:	10b4ef63          	bltu	s1,a1,10dd6 <_malloc_r+0x188>
   10cbc:	2bb9                	jal	1121a <__malloc_lock>
   10cbe:	1f700793          	li	a5,503
   10cc2:	3897f463          	bgeu	a5,s1,1104a <_malloc_r+0x3fc>
   10cc6:	0094d793          	srli	a5,s1,0x9
   10cca:	12078163          	beqz	a5,10dec <_malloc_r+0x19e>
   10cce:	4711                	li	a4,4
   10cd0:	30f76363          	bltu	a4,a5,10fd6 <_malloc_r+0x388>
   10cd4:	0064d793          	srli	a5,s1,0x6
   10cd8:	03978593          	addi	a1,a5,57
   10cdc:	03878813          	addi	a6,a5,56
   10ce0:	00359613          	slli	a2,a1,0x3
   10ce4:	694d                	lui	s2,0x13
   10ce6:	5b090913          	addi	s2,s2,1456 # 135b0 <__malloc_av_>
   10cea:	964a                	add	a2,a2,s2
   10cec:	4240                	lw	s0,4(a2)
   10cee:	1661                	addi	a2,a2,-8 # 1ff8 <exit-0xe0bc>
   10cf0:	02860163          	beq	a2,s0,10d12 <_malloc_r+0xc4>
   10cf4:	453d                	li	a0,15
   10cf6:	a039                	j	10d04 <_malloc_r+0xb6>
   10cf8:	4454                	lw	a3,12(s0)
   10cfa:	26075663          	bgez	a4,10f66 <_malloc_r+0x318>
   10cfe:	00d60a63          	beq	a2,a3,10d12 <_malloc_r+0xc4>
   10d02:	8436                	mv	s0,a3
   10d04:	405c                	lw	a5,4(s0)
   10d06:	9bf1                	andi	a5,a5,-4
   10d08:	40978733          	sub	a4,a5,s1
   10d0c:	fee556e3          	bge	a0,a4,10cf8 <_malloc_r+0xaa>
   10d10:	85c2                	mv	a1,a6
   10d12:	01092403          	lw	s0,16(s2)
   10d16:	684d                	lui	a6,0x13
   10d18:	5b880813          	addi	a6,a6,1464 # 135b8 <__malloc_av_+0x8>
   10d1c:	1f040b63          	beq	s0,a6,10f12 <_malloc_r+0x2c4>
   10d20:	405c                	lw	a5,4(s0)
   10d22:	46bd                	li	a3,15
   10d24:	9bf1                	andi	a5,a5,-4
   10d26:	40978733          	sub	a4,a5,s1
   10d2a:	32e6c563          	blt	a3,a4,11054 <_malloc_r+0x406>
   10d2e:	01092a23          	sw	a6,20(s2)
   10d32:	01092823          	sw	a6,16(s2)
   10d36:	30075063          	bgez	a4,11036 <_malloc_r+0x3e8>
   10d3a:	1ff00713          	li	a4,511
   10d3e:	00492503          	lw	a0,4(s2)
   10d42:	24f76a63          	bltu	a4,a5,10f96 <_malloc_r+0x348>
   10d46:	ff87f713          	andi	a4,a5,-8
   10d4a:	0721                	addi	a4,a4,8
   10d4c:	974a                	add	a4,a4,s2
   10d4e:	4314                	lw	a3,0(a4)
   10d50:	0057d613          	srli	a2,a5,0x5
   10d54:	4785                	li	a5,1
   10d56:	00c797b3          	sll	a5,a5,a2
   10d5a:	8d5d                	or	a0,a0,a5
   10d5c:	ff870793          	addi	a5,a4,-8
   10d60:	c414                	sw	a3,8(s0)
   10d62:	c45c                	sw	a5,12(s0)
   10d64:	00a92223          	sw	a0,4(s2)
   10d68:	c300                	sw	s0,0(a4)
   10d6a:	c6c0                	sw	s0,12(a3)
   10d6c:	4025d793          	srai	a5,a1,0x2
   10d70:	4605                	li	a2,1
   10d72:	00f61633          	sll	a2,a2,a5
   10d76:	08c56263          	bltu	a0,a2,10dfa <_malloc_r+0x1ac>
   10d7a:	00a677b3          	and	a5,a2,a0
   10d7e:	ef81                	bnez	a5,10d96 <_malloc_r+0x148>
   10d80:	0606                	slli	a2,a2,0x1
   10d82:	99f1                	andi	a1,a1,-4
   10d84:	00a677b3          	and	a5,a2,a0
   10d88:	0591                	addi	a1,a1,4
   10d8a:	e791                	bnez	a5,10d96 <_malloc_r+0x148>
   10d8c:	0606                	slli	a2,a2,0x1
   10d8e:	00a677b3          	and	a5,a2,a0
   10d92:	0591                	addi	a1,a1,4
   10d94:	dfe5                	beqz	a5,10d8c <_malloc_r+0x13e>
   10d96:	48bd                	li	a7,15
   10d98:	00359313          	slli	t1,a1,0x3
   10d9c:	934a                	add	t1,t1,s2
   10d9e:	851a                	mv	a0,t1
   10da0:	455c                	lw	a5,12(a0)
   10da2:	8e2e                	mv	t3,a1
   10da4:	24f50963          	beq	a0,a5,10ff6 <_malloc_r+0x3a8>
   10da8:	43d8                	lw	a4,4(a5)
   10daa:	843e                	mv	s0,a5
   10dac:	47dc                	lw	a5,12(a5)
   10dae:	9b71                	andi	a4,a4,-4
   10db0:	409706b3          	sub	a3,a4,s1
   10db4:	24d8c863          	blt	a7,a3,11004 <_malloc_r+0x3b6>
   10db8:	fe06c6e3          	bltz	a3,10da4 <_malloc_r+0x156>
   10dbc:	9722                	add	a4,a4,s0
   10dbe:	4354                	lw	a3,4(a4)
   10dc0:	4410                	lw	a2,8(s0)
   10dc2:	854e                	mv	a0,s3
   10dc4:	0016e693          	ori	a3,a3,1
   10dc8:	c354                	sw	a3,4(a4)
   10dca:	c65c                	sw	a5,12(a2)
   10dcc:	c790                	sw	a2,8(a5)
   10dce:	21b9                	jal	1121c <__malloc_unlock>
   10dd0:	00840513          	addi	a0,s0,8
   10dd4:	a029                	j	10dde <_malloc_r+0x190>
   10dd6:	47b1                	li	a5,12
   10dd8:	00f9a023          	sw	a5,0(s3)
   10ddc:	4501                	li	a0,0
   10dde:	50b2                	lw	ra,44(sp)
   10de0:	5422                	lw	s0,40(sp)
   10de2:	5492                	lw	s1,36(sp)
   10de4:	5902                	lw	s2,32(sp)
   10de6:	49f2                	lw	s3,28(sp)
   10de8:	6145                	addi	sp,sp,48
   10dea:	8082                	ret
   10dec:	20000613          	li	a2,512
   10df0:	04000593          	li	a1,64
   10df4:	03f00813          	li	a6,63
   10df8:	b5f5                	j	10ce4 <_malloc_r+0x96>
   10dfa:	00892403          	lw	s0,8(s2)
   10dfe:	c85a                	sw	s6,16(sp)
   10e00:	405c                	lw	a5,4(s0)
   10e02:	ffc7fb13          	andi	s6,a5,-4
   10e06:	009b6763          	bltu	s6,s1,10e14 <_malloc_r+0x1c6>
   10e0a:	409b0733          	sub	a4,s6,s1
   10e0e:	47bd                	li	a5,15
   10e10:	12e7c663          	blt	a5,a4,10f3c <_malloc_r+0x2ee>
   10e14:	c266                	sw	s9,4(sp)
   10e16:	ca56                	sw	s5,20(sp)
   10e18:	d401a703          	lw	a4,-704(gp) # 139c0 <__malloc_sbrk_base>
   10e1c:	d5c1aa83          	lw	s5,-676(gp) # 139dc <__malloc_top_pad>
   10e20:	cc52                	sw	s4,24(sp)
   10e22:	c65e                	sw	s7,12(sp)
   10e24:	57fd                	li	a5,-1
   10e26:	9aa6                	add	s5,s5,s1
   10e28:	01640a33          	add	s4,s0,s6
   10e2c:	2cf70263          	beq	a4,a5,110f0 <_malloc_r+0x4a2>
   10e30:	6785                	lui	a5,0x1
   10e32:	07bd                	addi	a5,a5,15 # 100f <exit-0xf0a5>
   10e34:	9abe                	add	s5,s5,a5
   10e36:	77fd                	lui	a5,0xfffff
   10e38:	00fafab3          	and	s5,s5,a5
   10e3c:	85d6                	mv	a1,s5
   10e3e:	854e                	mv	a0,s3
   10e40:	399000ef          	jal	119d8 <_sbrk_r>
   10e44:	57fd                	li	a5,-1
   10e46:	8baa                	mv	s7,a0
   10e48:	32f50b63          	beq	a0,a5,1117e <_malloc_r+0x530>
   10e4c:	c462                	sw	s8,8(sp)
   10e4e:	0d456563          	bltu	a0,s4,10f18 <_malloc_r+0x2ca>
   10e52:	eb818c13          	addi	s8,gp,-328 # 13b38 <__malloc_current_mallinfo>
   10e56:	000c2583          	lw	a1,0(s8)
   10e5a:	95d6                	add	a1,a1,s5
   10e5c:	00bc2023          	sw	a1,0(s8)
   10e60:	872e                	mv	a4,a1
   10e62:	32aa0263          	beq	s4,a0,11186 <_malloc_r+0x538>
   10e66:	d401a683          	lw	a3,-704(gp) # 139c0 <__malloc_sbrk_base>
   10e6a:	57fd                	li	a5,-1
   10e6c:	32f68a63          	beq	a3,a5,111a0 <_malloc_r+0x552>
   10e70:	414b87b3          	sub	a5,s7,s4
   10e74:	97ba                	add	a5,a5,a4
   10e76:	00fc2023          	sw	a5,0(s8)
   10e7a:	007bfc93          	andi	s9,s7,7
   10e7e:	280c8363          	beqz	s9,11104 <_malloc_r+0x4b6>
   10e82:	419b8bb3          	sub	s7,s7,s9
   10e86:	6585                	lui	a1,0x1
   10e88:	0ba1                	addi	s7,s7,8
   10e8a:	05a1                	addi	a1,a1,8 # 1008 <exit-0xf0ac>
   10e8c:	9ade                	add	s5,s5,s7
   10e8e:	419585b3          	sub	a1,a1,s9
   10e92:	415585b3          	sub	a1,a1,s5
   10e96:	05d2                	slli	a1,a1,0x14
   10e98:	0145da13          	srli	s4,a1,0x14
   10e9c:	85d2                	mv	a1,s4
   10e9e:	854e                	mv	a0,s3
   10ea0:	339000ef          	jal	119d8 <_sbrk_r>
   10ea4:	57fd                	li	a5,-1
   10ea6:	32f50963          	beq	a0,a5,111d8 <_malloc_r+0x58a>
   10eaa:	41750533          	sub	a0,a0,s7
   10eae:	01450ab3          	add	s5,a0,s4
   10eb2:	000c2703          	lw	a4,0(s8)
   10eb6:	01792423          	sw	s7,8(s2)
   10eba:	001ae793          	ori	a5,s5,1
   10ebe:	00ea05b3          	add	a1,s4,a4
   10ec2:	00fba223          	sw	a5,4(s7)
   10ec6:	00bc2023          	sw	a1,0(s8)
   10eca:	03240563          	beq	s0,s2,10ef4 <_malloc_r+0x2a6>
   10ece:	46bd                	li	a3,15
   10ed0:	2566fa63          	bgeu	a3,s6,11124 <_malloc_r+0x4d6>
   10ed4:	4058                	lw	a4,4(s0)
   10ed6:	ff4b0793          	addi	a5,s6,-12
   10eda:	9be1                	andi	a5,a5,-8
   10edc:	8b05                	andi	a4,a4,1
   10ede:	8f5d                	or	a4,a4,a5
   10ee0:	c058                	sw	a4,4(s0)
   10ee2:	4615                	li	a2,5
   10ee4:	00f40733          	add	a4,s0,a5
   10ee8:	c350                	sw	a2,4(a4)
   10eea:	c710                	sw	a2,8(a4)
   10eec:	1ef6e863          	bltu	a3,a5,110dc <_malloc_r+0x48e>
   10ef0:	004ba783          	lw	a5,4(s7)
   10ef4:	d581a683          	lw	a3,-680(gp) # 139d8 <__malloc_max_sbrked_mem>
   10ef8:	00b6f463          	bgeu	a3,a1,10f00 <_malloc_r+0x2b2>
   10efc:	d4b1ac23          	sw	a1,-680(gp) # 139d8 <__malloc_max_sbrked_mem>
   10f00:	d541a683          	lw	a3,-684(gp) # 139d4 <__malloc_max_total_mem>
   10f04:	00b6f463          	bgeu	a3,a1,10f0c <_malloc_r+0x2be>
   10f08:	d4b1aa23          	sw	a1,-684(gp) # 139d4 <__malloc_max_total_mem>
   10f0c:	4c22                	lw	s8,8(sp)
   10f0e:	845e                	mv	s0,s7
   10f10:	a811                	j	10f24 <_malloc_r+0x2d6>
   10f12:	00492503          	lw	a0,4(s2)
   10f16:	bd99                	j	10d6c <_malloc_r+0x11e>
   10f18:	25240b63          	beq	s0,s2,1116e <_malloc_r+0x520>
   10f1c:	00892403          	lw	s0,8(s2)
   10f20:	4c22                	lw	s8,8(sp)
   10f22:	405c                	lw	a5,4(s0)
   10f24:	9bf1                	andi	a5,a5,-4
   10f26:	40978733          	sub	a4,a5,s1
   10f2a:	2097e163          	bltu	a5,s1,1112c <_malloc_r+0x4de>
   10f2e:	47bd                	li	a5,15
   10f30:	1ee7de63          	bge	a5,a4,1112c <_malloc_r+0x4de>
   10f34:	4a62                	lw	s4,24(sp)
   10f36:	4ad2                	lw	s5,20(sp)
   10f38:	4bb2                	lw	s7,12(sp)
   10f3a:	4c92                	lw	s9,4(sp)
   10f3c:	0014e793          	ori	a5,s1,1
   10f40:	c05c                	sw	a5,4(s0)
   10f42:	94a2                	add	s1,s1,s0
   10f44:	00992423          	sw	s1,8(s2)
   10f48:	00176713          	ori	a4,a4,1
   10f4c:	854e                	mv	a0,s3
   10f4e:	c0d8                	sw	a4,4(s1)
   10f50:	24f1                	jal	1121c <__malloc_unlock>
   10f52:	50b2                	lw	ra,44(sp)
   10f54:	00840513          	addi	a0,s0,8
   10f58:	5422                	lw	s0,40(sp)
   10f5a:	4b42                	lw	s6,16(sp)
   10f5c:	5492                	lw	s1,36(sp)
   10f5e:	5902                	lw	s2,32(sp)
   10f60:	49f2                	lw	s3,28(sp)
   10f62:	6145                	addi	sp,sp,48
   10f64:	8082                	ret
   10f66:	4410                	lw	a2,8(s0)
   10f68:	97a2                	add	a5,a5,s0
   10f6a:	43d8                	lw	a4,4(a5)
   10f6c:	c654                	sw	a3,12(a2)
   10f6e:	c690                	sw	a2,8(a3)
   10f70:	00176713          	ori	a4,a4,1
   10f74:	854e                	mv	a0,s3
   10f76:	c3d8                	sw	a4,4(a5)
   10f78:	2455                	jal	1121c <__malloc_unlock>
   10f7a:	50b2                	lw	ra,44(sp)
   10f7c:	00840513          	addi	a0,s0,8
   10f80:	5422                	lw	s0,40(sp)
   10f82:	5492                	lw	s1,36(sp)
   10f84:	5902                	lw	s2,32(sp)
   10f86:	49f2                	lw	s3,28(sp)
   10f88:	6145                	addi	sp,sp,48
   10f8a:	8082                	ret
   10f8c:	47c0                	lw	s0,12(a5)
   10f8e:	0589                	addi	a1,a1,2
   10f90:	d88781e3          	beq	a5,s0,10d12 <_malloc_r+0xc4>
   10f94:	b9c5                	j	10c84 <_malloc_r+0x36>
   10f96:	0097d713          	srli	a4,a5,0x9
   10f9a:	4691                	li	a3,4
   10f9c:	0ee6f263          	bgeu	a3,a4,11080 <_malloc_r+0x432>
   10fa0:	46d1                	li	a3,20
   10fa2:	18e6ed63          	bltu	a3,a4,1113c <_malloc_r+0x4ee>
   10fa6:	05c70613          	addi	a2,a4,92
   10faa:	060e                	slli	a2,a2,0x3
   10fac:	05b70693          	addi	a3,a4,91
   10fb0:	964a                	add	a2,a2,s2
   10fb2:	4218                	lw	a4,0(a2)
   10fb4:	1661                	addi	a2,a2,-8
   10fb6:	00e61663          	bne	a2,a4,10fc2 <_malloc_r+0x374>
   10fba:	aa2d                	j	110f4 <_malloc_r+0x4a6>
   10fbc:	4718                	lw	a4,8(a4)
   10fbe:	00e60663          	beq	a2,a4,10fca <_malloc_r+0x37c>
   10fc2:	4354                	lw	a3,4(a4)
   10fc4:	9af1                	andi	a3,a3,-4
   10fc6:	fed7ebe3          	bltu	a5,a3,10fbc <_malloc_r+0x36e>
   10fca:	4750                	lw	a2,12(a4)
   10fcc:	c450                	sw	a2,12(s0)
   10fce:	c418                	sw	a4,8(s0)
   10fd0:	c600                	sw	s0,8(a2)
   10fd2:	c740                	sw	s0,12(a4)
   10fd4:	bb61                	j	10d6c <_malloc_r+0x11e>
   10fd6:	4751                	li	a4,20
   10fd8:	0af77c63          	bgeu	a4,a5,11090 <_malloc_r+0x442>
   10fdc:	05400713          	li	a4,84
   10fe0:	16f76a63          	bltu	a4,a5,11154 <_malloc_r+0x506>
   10fe4:	00c4d793          	srli	a5,s1,0xc
   10fe8:	06f78593          	addi	a1,a5,111 # fffff06f <__BSS_END__+0xfffeb37f>
   10fec:	06e78813          	addi	a6,a5,110
   10ff0:	00359613          	slli	a2,a1,0x3
   10ff4:	b9c5                	j	10ce4 <_malloc_r+0x96>
   10ff6:	0e05                	addi	t3,t3,1
   10ff8:	003e7793          	andi	a5,t3,3
   10ffc:	0521                	addi	a0,a0,8
   10ffe:	c7cd                	beqz	a5,110a8 <_malloc_r+0x45a>
   11000:	455c                	lw	a5,12(a0)
   11002:	b34d                	j	10da4 <_malloc_r+0x156>
   11004:	4410                	lw	a2,8(s0)
   11006:	0014e593          	ori	a1,s1,1
   1100a:	c04c                	sw	a1,4(s0)
   1100c:	c65c                	sw	a5,12(a2)
   1100e:	c790                	sw	a2,8(a5)
   11010:	94a2                	add	s1,s1,s0
   11012:	00992a23          	sw	s1,20(s2)
   11016:	00992823          	sw	s1,16(s2)
   1101a:	0016e793          	ori	a5,a3,1
   1101e:	9722                	add	a4,a4,s0
   11020:	0104a623          	sw	a6,12(s1)
   11024:	0104a423          	sw	a6,8(s1)
   11028:	c0dc                	sw	a5,4(s1)
   1102a:	854e                	mv	a0,s3
   1102c:	c314                	sw	a3,0(a4)
   1102e:	22fd                	jal	1121c <__malloc_unlock>
   11030:	00840513          	addi	a0,s0,8
   11034:	b36d                	j	10dde <_malloc_r+0x190>
   11036:	97a2                	add	a5,a5,s0
   11038:	43d8                	lw	a4,4(a5)
   1103a:	854e                	mv	a0,s3
   1103c:	00176713          	ori	a4,a4,1
   11040:	c3d8                	sw	a4,4(a5)
   11042:	2ae9                	jal	1121c <__malloc_unlock>
   11044:	00840513          	addi	a0,s0,8
   11048:	bb59                	j	10dde <_malloc_r+0x190>
   1104a:	0034d593          	srli	a1,s1,0x3
   1104e:	00848793          	addi	a5,s1,8
   11052:	b105                	j	10c72 <_malloc_r+0x24>
   11054:	0014e693          	ori	a3,s1,1
   11058:	c054                	sw	a3,4(s0)
   1105a:	94a2                	add	s1,s1,s0
   1105c:	00992a23          	sw	s1,20(s2)
   11060:	00992823          	sw	s1,16(s2)
   11064:	00176693          	ori	a3,a4,1
   11068:	97a2                	add	a5,a5,s0
   1106a:	0104a623          	sw	a6,12(s1)
   1106e:	0104a423          	sw	a6,8(s1)
   11072:	c0d4                	sw	a3,4(s1)
   11074:	854e                	mv	a0,s3
   11076:	c398                	sw	a4,0(a5)
   11078:	2255                	jal	1121c <__malloc_unlock>
   1107a:	00840513          	addi	a0,s0,8
   1107e:	b385                	j	10dde <_malloc_r+0x190>
   11080:	0067d713          	srli	a4,a5,0x6
   11084:	03970613          	addi	a2,a4,57
   11088:	060e                	slli	a2,a2,0x3
   1108a:	03870693          	addi	a3,a4,56
   1108e:	b70d                	j	10fb0 <_malloc_r+0x362>
   11090:	05c78593          	addi	a1,a5,92
   11094:	05b78813          	addi	a6,a5,91
   11098:	00359613          	slli	a2,a1,0x3
   1109c:	b1a1                	j	10ce4 <_malloc_r+0x96>
   1109e:	00832783          	lw	a5,8(t1)
   110a2:	15fd                	addi	a1,a1,-1
   110a4:	16679863          	bne	a5,t1,11214 <_malloc_r+0x5c6>
   110a8:	0035f793          	andi	a5,a1,3
   110ac:	1361                	addi	t1,t1,-8
   110ae:	fbe5                	bnez	a5,1109e <_malloc_r+0x450>
   110b0:	00492703          	lw	a4,4(s2)
   110b4:	fff64793          	not	a5,a2
   110b8:	8ff9                	and	a5,a5,a4
   110ba:	00f92223          	sw	a5,4(s2)
   110be:	0606                	slli	a2,a2,0x1
   110c0:	d2c7ede3          	bltu	a5,a2,10dfa <_malloc_r+0x1ac>
   110c4:	d2060be3          	beqz	a2,10dfa <_malloc_r+0x1ac>
   110c8:	00f67733          	and	a4,a2,a5
   110cc:	e711                	bnez	a4,110d8 <_malloc_r+0x48a>
   110ce:	0606                	slli	a2,a2,0x1
   110d0:	00f67733          	and	a4,a2,a5
   110d4:	0e11                	addi	t3,t3,4
   110d6:	df65                	beqz	a4,110ce <_malloc_r+0x480>
   110d8:	85f2                	mv	a1,t3
   110da:	b97d                	j	10d98 <_malloc_r+0x14a>
   110dc:	00840593          	addi	a1,s0,8
   110e0:	854e                	mv	a0,s3
   110e2:	951ff0ef          	jal	10a32 <_free_r>
   110e6:	000c2583          	lw	a1,0(s8)
   110ea:	00892b83          	lw	s7,8(s2)
   110ee:	b509                	j	10ef0 <_malloc_r+0x2a2>
   110f0:	0ac1                	addi	s5,s5,16
   110f2:	b3a9                	j	10e3c <_malloc_r+0x1ee>
   110f4:	8689                	srai	a3,a3,0x2
   110f6:	4785                	li	a5,1
   110f8:	00d797b3          	sll	a5,a5,a3
   110fc:	8d5d                	or	a0,a0,a5
   110fe:	00a92223          	sw	a0,4(s2)
   11102:	b5e9                	j	10fcc <_malloc_r+0x37e>
   11104:	015b85b3          	add	a1,s7,s5
   11108:	40b005b3          	neg	a1,a1
   1110c:	05d2                	slli	a1,a1,0x14
   1110e:	0145da13          	srli	s4,a1,0x14
   11112:	85d2                	mv	a1,s4
   11114:	854e                	mv	a0,s3
   11116:	0c3000ef          	jal	119d8 <_sbrk_r>
   1111a:	57fd                	li	a5,-1
   1111c:	d8f517e3          	bne	a0,a5,10eaa <_malloc_r+0x25c>
   11120:	4a01                	li	s4,0
   11122:	bb41                	j	10eb2 <_malloc_r+0x264>
   11124:	4c22                	lw	s8,8(sp)
   11126:	4785                	li	a5,1
   11128:	00fba223          	sw	a5,4(s7)
   1112c:	854e                	mv	a0,s3
   1112e:	20fd                	jal	1121c <__malloc_unlock>
   11130:	4a62                	lw	s4,24(sp)
   11132:	4ad2                	lw	s5,20(sp)
   11134:	4b42                	lw	s6,16(sp)
   11136:	4bb2                	lw	s7,12(sp)
   11138:	4c92                	lw	s9,4(sp)
   1113a:	b14d                	j	10ddc <_malloc_r+0x18e>
   1113c:	05400693          	li	a3,84
   11140:	06e6e363          	bltu	a3,a4,111a6 <_malloc_r+0x558>
   11144:	00c7d713          	srli	a4,a5,0xc
   11148:	06f70613          	addi	a2,a4,111
   1114c:	060e                	slli	a2,a2,0x3
   1114e:	06e70693          	addi	a3,a4,110
   11152:	bdb9                	j	10fb0 <_malloc_r+0x362>
   11154:	15400713          	li	a4,340
   11158:	06f76363          	bltu	a4,a5,111be <_malloc_r+0x570>
   1115c:	00f4d793          	srli	a5,s1,0xf
   11160:	07878593          	addi	a1,a5,120
   11164:	07778813          	addi	a6,a5,119
   11168:	00359613          	slli	a2,a1,0x3
   1116c:	bea5                	j	10ce4 <_malloc_r+0x96>
   1116e:	eb818c13          	addi	s8,gp,-328 # 13b38 <__malloc_current_mallinfo>
   11172:	000c2703          	lw	a4,0(s8)
   11176:	9756                	add	a4,a4,s5
   11178:	00ec2023          	sw	a4,0(s8)
   1117c:	b1ed                	j	10e66 <_malloc_r+0x218>
   1117e:	00892403          	lw	s0,8(s2)
   11182:	405c                	lw	a5,4(s0)
   11184:	b345                	j	10f24 <_malloc_r+0x2d6>
   11186:	01451793          	slli	a5,a0,0x14
   1118a:	cc079ee3          	bnez	a5,10e66 <_malloc_r+0x218>
   1118e:	00892b83          	lw	s7,8(s2)
   11192:	015b07b3          	add	a5,s6,s5
   11196:	0017e793          	ori	a5,a5,1
   1119a:	00fba223          	sw	a5,4(s7)
   1119e:	bb99                	j	10ef4 <_malloc_r+0x2a6>
   111a0:	d571a023          	sw	s7,-704(gp) # 139c0 <__malloc_sbrk_base>
   111a4:	b9d9                	j	10e7a <_malloc_r+0x22c>
   111a6:	15400693          	li	a3,340
   111aa:	02e6ed63          	bltu	a3,a4,111e4 <_malloc_r+0x596>
   111ae:	00f7d713          	srli	a4,a5,0xf
   111b2:	07870613          	addi	a2,a4,120
   111b6:	060e                	slli	a2,a2,0x3
   111b8:	07770693          	addi	a3,a4,119
   111bc:	bbd5                	j	10fb0 <_malloc_r+0x362>
   111be:	55400713          	li	a4,1364
   111c2:	02f76d63          	bltu	a4,a5,111fc <_malloc_r+0x5ae>
   111c6:	0124d793          	srli	a5,s1,0x12
   111ca:	07d78593          	addi	a1,a5,125
   111ce:	07c78813          	addi	a6,a5,124
   111d2:	00359613          	slli	a2,a1,0x3
   111d6:	b639                	j	10ce4 <_malloc_r+0x96>
   111d8:	1ce1                	addi	s9,s9,-8
   111da:	9ae6                	add	s5,s5,s9
   111dc:	417a8ab3          	sub	s5,s5,s7
   111e0:	4a01                	li	s4,0
   111e2:	b9c1                	j	10eb2 <_malloc_r+0x264>
   111e4:	55400693          	li	a3,1364
   111e8:	02e6e163          	bltu	a3,a4,1120a <_malloc_r+0x5bc>
   111ec:	0127d713          	srli	a4,a5,0x12
   111f0:	07d70613          	addi	a2,a4,125
   111f4:	060e                	slli	a2,a2,0x3
   111f6:	07c70693          	addi	a3,a4,124
   111fa:	bb5d                	j	10fb0 <_malloc_r+0x362>
   111fc:	3f800613          	li	a2,1016
   11200:	07f00593          	li	a1,127
   11204:	07e00813          	li	a6,126
   11208:	bcf1                	j	10ce4 <_malloc_r+0x96>
   1120a:	3f800613          	li	a2,1016
   1120e:	07e00693          	li	a3,126
   11212:	bb79                	j	10fb0 <_malloc_r+0x362>
   11214:	00492783          	lw	a5,4(s2)
   11218:	b55d                	j	110be <_malloc_r+0x470>

0001121a <__malloc_lock>:
   1121a:	8082                	ret

0001121c <__malloc_unlock>:
   1121c:	8082                	ret

0001121e <_fclose_r>:
   1121e:	1141                	addi	sp,sp,-16
   11220:	c606                	sw	ra,12(sp)
   11222:	c04a                	sw	s2,0(sp)
   11224:	cd89                	beqz	a1,1123e <_fclose_r+0x20>
   11226:	c422                	sw	s0,8(sp)
   11228:	c226                	sw	s1,4(sp)
   1122a:	842e                	mv	s0,a1
   1122c:	84aa                	mv	s1,a0
   1122e:	c119                	beqz	a0,11234 <_fclose_r+0x16>
   11230:	595c                	lw	a5,52(a0)
   11232:	c7d1                	beqz	a5,112be <_fclose_r+0xa0>
   11234:	00c41783          	lh	a5,12(s0)
   11238:	eb89                	bnez	a5,1124a <_fclose_r+0x2c>
   1123a:	4422                	lw	s0,8(sp)
   1123c:	4492                	lw	s1,4(sp)
   1123e:	40b2                	lw	ra,12(sp)
   11240:	4901                	li	s2,0
   11242:	854a                	mv	a0,s2
   11244:	4902                	lw	s2,0(sp)
   11246:	0141                	addi	sp,sp,16
   11248:	8082                	ret
   1124a:	85a2                	mv	a1,s0
   1124c:	8526                	mv	a0,s1
   1124e:	28bd                	jal	112cc <__sflush_r>
   11250:	545c                	lw	a5,44(s0)
   11252:	892a                	mv	s2,a0
   11254:	c791                	beqz	a5,11260 <_fclose_r+0x42>
   11256:	4c4c                	lw	a1,28(s0)
   11258:	8526                	mv	a0,s1
   1125a:	9782                	jalr	a5
   1125c:	04054663          	bltz	a0,112a8 <_fclose_r+0x8a>
   11260:	00c45783          	lhu	a5,12(s0)
   11264:	0807f793          	andi	a5,a5,128
   11268:	e7b1                	bnez	a5,112b4 <_fclose_r+0x96>
   1126a:	580c                	lw	a1,48(s0)
   1126c:	c991                	beqz	a1,11280 <_fclose_r+0x62>
   1126e:	04040793          	addi	a5,s0,64
   11272:	00f58563          	beq	a1,a5,1127c <_fclose_r+0x5e>
   11276:	8526                	mv	a0,s1
   11278:	fbaff0ef          	jal	10a32 <_free_r>
   1127c:	02042823          	sw	zero,48(s0)
   11280:	406c                	lw	a1,68(s0)
   11282:	c591                	beqz	a1,1128e <_fclose_r+0x70>
   11284:	8526                	mv	a0,s1
   11286:	facff0ef          	jal	10a32 <_free_r>
   1128a:	04042223          	sw	zero,68(s0)
   1128e:	93cff0ef          	jal	103ca <__sfp_lock_acquire>
   11292:	00041623          	sh	zero,12(s0)
   11296:	936ff0ef          	jal	103cc <__sfp_lock_release>
   1129a:	40b2                	lw	ra,12(sp)
   1129c:	4422                	lw	s0,8(sp)
   1129e:	4492                	lw	s1,4(sp)
   112a0:	854a                	mv	a0,s2
   112a2:	4902                	lw	s2,0(sp)
   112a4:	0141                	addi	sp,sp,16
   112a6:	8082                	ret
   112a8:	00c45783          	lhu	a5,12(s0)
   112ac:	597d                	li	s2,-1
   112ae:	0807f793          	andi	a5,a5,128
   112b2:	dfc5                	beqz	a5,1126a <_fclose_r+0x4c>
   112b4:	480c                	lw	a1,16(s0)
   112b6:	8526                	mv	a0,s1
   112b8:	f7aff0ef          	jal	10a32 <_free_r>
   112bc:	b77d                	j	1126a <_fclose_r+0x4c>
   112be:	8f6ff0ef          	jal	103b4 <__sinit>
   112c2:	bf8d                	j	11234 <_fclose_r+0x16>

000112c4 <fclose>:
   112c4:	85aa                	mv	a1,a0
   112c6:	d3c1a503          	lw	a0,-708(gp) # 139bc <_impure_ptr>
   112ca:	bf91                	j	1121e <_fclose_r>

000112cc <__sflush_r>:
   112cc:	00c59703          	lh	a4,12(a1)
   112d0:	1101                	addi	sp,sp,-32
   112d2:	cc22                	sw	s0,24(sp)
   112d4:	c64e                	sw	s3,12(sp)
   112d6:	ce06                	sw	ra,28(sp)
   112d8:	00877793          	andi	a5,a4,8
   112dc:	842e                	mv	s0,a1
   112de:	89aa                	mv	s3,a0
   112e0:	e7e1                	bnez	a5,113a8 <__sflush_r+0xdc>
   112e2:	6785                	lui	a5,0x1
   112e4:	80078793          	addi	a5,a5,-2048 # 800 <exit-0xf8b4>
   112e8:	41d4                	lw	a3,4(a1)
   112ea:	8fd9                	or	a5,a5,a4
   112ec:	00f59623          	sh	a5,12(a1)
   112f0:	10d05963          	blez	a3,11402 <__sflush_r+0x136>
   112f4:	02842803          	lw	a6,40(s0)
   112f8:	0a080263          	beqz	a6,1139c <__sflush_r+0xd0>
   112fc:	ca26                	sw	s1,20(sp)
   112fe:	01371693          	slli	a3,a4,0x13
   11302:	0009a483          	lw	s1,0(s3)
   11306:	0009a023          	sw	zero,0(s3)
   1130a:	1006c363          	bltz	a3,11410 <__sflush_r+0x144>
   1130e:	4c4c                	lw	a1,28(s0)
   11310:	4601                	li	a2,0
   11312:	4685                	li	a3,1
   11314:	854e                	mv	a0,s3
   11316:	9802                	jalr	a6
   11318:	57fd                	li	a5,-1
   1131a:	862a                	mv	a2,a0
   1131c:	12f50163          	beq	a0,a5,1143e <__sflush_r+0x172>
   11320:	00c41783          	lh	a5,12(s0)
   11324:	02842803          	lw	a6,40(s0)
   11328:	8b91                	andi	a5,a5,4
   1132a:	c799                	beqz	a5,11338 <__sflush_r+0x6c>
   1132c:	4058                	lw	a4,4(s0)
   1132e:	581c                	lw	a5,48(s0)
   11330:	8e19                	sub	a2,a2,a4
   11332:	c399                	beqz	a5,11338 <__sflush_r+0x6c>
   11334:	5c5c                	lw	a5,60(s0)
   11336:	8e1d                	sub	a2,a2,a5
   11338:	4c4c                	lw	a1,28(s0)
   1133a:	4681                	li	a3,0
   1133c:	854e                	mv	a0,s3
   1133e:	9802                	jalr	a6
   11340:	577d                	li	a4,-1
   11342:	00c41783          	lh	a5,12(s0)
   11346:	0ce51763          	bne	a0,a4,11414 <__sflush_r+0x148>
   1134a:	0009a683          	lw	a3,0(s3)
   1134e:	4775                	li	a4,29
   11350:	10d76363          	bltu	a4,a3,11456 <__sflush_r+0x18a>
   11354:	20400737          	lui	a4,0x20400
   11358:	0705                	addi	a4,a4,1 # 20400001 <__BSS_END__+0x203ec311>
   1135a:	00d75733          	srl	a4,a4,a3
   1135e:	8b05                	andi	a4,a4,1
   11360:	cb7d                	beqz	a4,11456 <__sflush_r+0x18a>
   11362:	4810                	lw	a2,16(s0)
   11364:	777d                	lui	a4,0xfffff
   11366:	7ff70713          	addi	a4,a4,2047 # fffff7ff <__BSS_END__+0xfffebb0f>
   1136a:	8f7d                	and	a4,a4,a5
   1136c:	00e41623          	sh	a4,12(s0)
   11370:	00042223          	sw	zero,4(s0)
   11374:	c010                	sw	a2,0(s0)
   11376:	01379713          	slli	a4,a5,0x13
   1137a:	00075363          	bgez	a4,11380 <__sflush_r+0xb4>
   1137e:	cacd                	beqz	a3,11430 <__sflush_r+0x164>
   11380:	580c                	lw	a1,48(s0)
   11382:	0099a023          	sw	s1,0(s3)
   11386:	c9d5                	beqz	a1,1143a <__sflush_r+0x16e>
   11388:	04040793          	addi	a5,s0,64
   1138c:	00f58563          	beq	a1,a5,11396 <__sflush_r+0xca>
   11390:	854e                	mv	a0,s3
   11392:	ea0ff0ef          	jal	10a32 <_free_r>
   11396:	44d2                	lw	s1,20(sp)
   11398:	02042823          	sw	zero,48(s0)
   1139c:	40f2                	lw	ra,28(sp)
   1139e:	4462                	lw	s0,24(sp)
   113a0:	49b2                	lw	s3,12(sp)
   113a2:	4501                	li	a0,0
   113a4:	6105                	addi	sp,sp,32
   113a6:	8082                	ret
   113a8:	c84a                	sw	s2,16(sp)
   113aa:	0105a903          	lw	s2,16(a1)
   113ae:	04090f63          	beqz	s2,1140c <__sflush_r+0x140>
   113b2:	ca26                	sw	s1,20(sp)
   113b4:	4184                	lw	s1,0(a1)
   113b6:	8b0d                	andi	a4,a4,3
   113b8:	0125a023          	sw	s2,0(a1)
   113bc:	412484b3          	sub	s1,s1,s2
   113c0:	4781                	li	a5,0
   113c2:	e311                	bnez	a4,113c6 <__sflush_r+0xfa>
   113c4:	49dc                	lw	a5,20(a1)
   113c6:	c41c                	sw	a5,8(s0)
   113c8:	00904663          	bgtz	s1,113d4 <__sflush_r+0x108>
   113cc:	a83d                	j	1140a <__sflush_r+0x13e>
   113ce:	992a                	add	s2,s2,a0
   113d0:	02905d63          	blez	s1,1140a <__sflush_r+0x13e>
   113d4:	505c                	lw	a5,36(s0)
   113d6:	4c4c                	lw	a1,28(s0)
   113d8:	86a6                	mv	a3,s1
   113da:	864a                	mv	a2,s2
   113dc:	854e                	mv	a0,s3
   113de:	9782                	jalr	a5
   113e0:	8c89                	sub	s1,s1,a0
   113e2:	fea046e3          	bgtz	a0,113ce <__sflush_r+0x102>
   113e6:	00c41783          	lh	a5,12(s0)
   113ea:	4942                	lw	s2,16(sp)
   113ec:	0407e793          	ori	a5,a5,64
   113f0:	40f2                	lw	ra,28(sp)
   113f2:	00f41623          	sh	a5,12(s0)
   113f6:	4462                	lw	s0,24(sp)
   113f8:	44d2                	lw	s1,20(sp)
   113fa:	49b2                	lw	s3,12(sp)
   113fc:	557d                	li	a0,-1
   113fe:	6105                	addi	sp,sp,32
   11400:	8082                	ret
   11402:	5dd4                	lw	a3,60(a1)
   11404:	eed048e3          	bgtz	a3,112f4 <__sflush_r+0x28>
   11408:	bf51                	j	1139c <__sflush_r+0xd0>
   1140a:	44d2                	lw	s1,20(sp)
   1140c:	4942                	lw	s2,16(sp)
   1140e:	b779                	j	1139c <__sflush_r+0xd0>
   11410:	4830                	lw	a2,80(s0)
   11412:	bf19                	j	11328 <__sflush_r+0x5c>
   11414:	4814                	lw	a3,16(s0)
   11416:	777d                	lui	a4,0xfffff
   11418:	7ff70713          	addi	a4,a4,2047 # fffff7ff <__BSS_END__+0xfffebb0f>
   1141c:	8f7d                	and	a4,a4,a5
   1141e:	00e41623          	sh	a4,12(s0)
   11422:	00042223          	sw	zero,4(s0)
   11426:	c014                	sw	a3,0(s0)
   11428:	01379713          	slli	a4,a5,0x13
   1142c:	f4075ae3          	bgez	a4,11380 <__sflush_r+0xb4>
   11430:	580c                	lw	a1,48(s0)
   11432:	c828                	sw	a0,80(s0)
   11434:	0099a023          	sw	s1,0(s3)
   11438:	f9a1                	bnez	a1,11388 <__sflush_r+0xbc>
   1143a:	44d2                	lw	s1,20(sp)
   1143c:	b785                	j	1139c <__sflush_r+0xd0>
   1143e:	0009a783          	lw	a5,0(s3)
   11442:	ec078fe3          	beqz	a5,11320 <__sflush_r+0x54>
   11446:	4775                	li	a4,29
   11448:	00e78a63          	beq	a5,a4,1145c <__sflush_r+0x190>
   1144c:	4759                	li	a4,22
   1144e:	00e78763          	beq	a5,a4,1145c <__sflush_r+0x190>
   11452:	00c41783          	lh	a5,12(s0)
   11456:	0407e793          	ori	a5,a5,64
   1145a:	bf59                	j	113f0 <__sflush_r+0x124>
   1145c:	0099a023          	sw	s1,0(s3)
   11460:	44d2                	lw	s1,20(sp)
   11462:	bf2d                	j	1139c <__sflush_r+0xd0>

00011464 <_fflush_r>:
   11464:	1101                	addi	sp,sp,-32
   11466:	cc22                	sw	s0,24(sp)
   11468:	ce06                	sw	ra,28(sp)
   1146a:	842a                	mv	s0,a0
   1146c:	c119                	beqz	a0,11472 <_fflush_r+0xe>
   1146e:	595c                	lw	a5,52(a0)
   11470:	cf91                	beqz	a5,1148c <_fflush_r+0x28>
   11472:	00c59783          	lh	a5,12(a1)
   11476:	e791                	bnez	a5,11482 <_fflush_r+0x1e>
   11478:	40f2                	lw	ra,28(sp)
   1147a:	4462                	lw	s0,24(sp)
   1147c:	4501                	li	a0,0
   1147e:	6105                	addi	sp,sp,32
   11480:	8082                	ret
   11482:	8522                	mv	a0,s0
   11484:	4462                	lw	s0,24(sp)
   11486:	40f2                	lw	ra,28(sp)
   11488:	6105                	addi	sp,sp,32
   1148a:	b589                	j	112cc <__sflush_r>
   1148c:	c62e                	sw	a1,12(sp)
   1148e:	f27fe0ef          	jal	103b4 <__sinit>
   11492:	45b2                	lw	a1,12(sp)
   11494:	bff9                	j	11472 <_fflush_r+0xe>

00011496 <fflush>:
   11496:	cd05                	beqz	a0,114ce <fflush+0x38>
   11498:	85aa                	mv	a1,a0
   1149a:	d3c1a503          	lw	a0,-708(gp) # 139bc <_impure_ptr>
   1149e:	c119                	beqz	a0,114a4 <fflush+0xe>
   114a0:	595c                	lw	a5,52(a0)
   114a2:	c799                	beqz	a5,114b0 <fflush+0x1a>
   114a4:	00c59783          	lh	a5,12(a1)
   114a8:	e399                	bnez	a5,114ae <fflush+0x18>
   114aa:	4501                	li	a0,0
   114ac:	8082                	ret
   114ae:	bd39                	j	112cc <__sflush_r>
   114b0:	1101                	addi	sp,sp,-32
   114b2:	c62e                	sw	a1,12(sp)
   114b4:	c42a                	sw	a0,8(sp)
   114b6:	ce06                	sw	ra,28(sp)
   114b8:	efdfe0ef          	jal	103b4 <__sinit>
   114bc:	45b2                	lw	a1,12(sp)
   114be:	4522                	lw	a0,8(sp)
   114c0:	00c59783          	lh	a5,12(a1)
   114c4:	e385                	bnez	a5,114e4 <fflush+0x4e>
   114c6:	40f2                	lw	ra,28(sp)
   114c8:	4501                	li	a0,0
   114ca:	6105                	addi	sp,sp,32
   114cc:	8082                	ret
   114ce:	664d                	lui	a2,0x13
   114d0:	65c5                	lui	a1,0x11
   114d2:	654d                	lui	a0,0x13
   114d4:	48060613          	addi	a2,a2,1152 # 13480 <__sglue>
   114d8:	46458593          	addi	a1,a1,1124 # 11464 <_fflush_r>
   114dc:	49050513          	addi	a0,a0,1168 # 13490 <_impure_data>
   114e0:	f0ffe06f          	j	103ee <_fwalk_sglue>
   114e4:	40f2                	lw	ra,28(sp)
   114e6:	6105                	addi	sp,sp,32
   114e8:	b3d5                	j	112cc <__sflush_r>

000114ea <__sfvwrite_r>:
   114ea:	461c                	lw	a5,8(a2)
   114ec:	18078a63          	beqz	a5,11680 <__sfvwrite_r+0x196>
   114f0:	00c59703          	lh	a4,12(a1)
   114f4:	7179                	addi	sp,sp,-48
   114f6:	d422                	sw	s0,40(sp)
   114f8:	cc52                	sw	s4,24(sp)
   114fa:	ca56                	sw	s5,20(sp)
   114fc:	d606                	sw	ra,44(sp)
   114fe:	00877793          	andi	a5,a4,8
   11502:	8a32                	mv	s4,a2
   11504:	8aaa                	mv	s5,a0
   11506:	842e                	mv	s0,a1
   11508:	c7b5                	beqz	a5,11574 <__sfvwrite_r+0x8a>
   1150a:	499c                	lw	a5,16(a1)
   1150c:	c7a5                	beqz	a5,11574 <__sfvwrite_r+0x8a>
   1150e:	d226                	sw	s1,36(sp)
   11510:	d04a                	sw	s2,32(sp)
   11512:	ce4e                	sw	s3,28(sp)
   11514:	c85a                	sw	s6,16(sp)
   11516:	00277793          	andi	a5,a4,2
   1151a:	000a2483          	lw	s1,0(s4)
   1151e:	cbbd                	beqz	a5,11594 <__sfvwrite_r+0xaa>
   11520:	80000b37          	lui	s6,0x80000
   11524:	c00b0b13          	addi	s6,s6,-1024 # 7ffffc00 <__BSS_END__+0x7ffebf10>
   11528:	4981                	li	s3,0
   1152a:	4901                	li	s2,0
   1152c:	864e                	mv	a2,s3
   1152e:	8556                	mv	a0,s5
   11530:	14090263          	beqz	s2,11674 <__sfvwrite_r+0x18a>
   11534:	800007b7          	lui	a5,0x80000
   11538:	86ca                	mv	a3,s2
   1153a:	012b7463          	bgeu	s6,s2,11542 <__sfvwrite_r+0x58>
   1153e:	c0078693          	addi	a3,a5,-1024 # 7ffffc00 <__BSS_END__+0x7ffebf10>
   11542:	505c                	lw	a5,36(s0)
   11544:	4c4c                	lw	a1,28(s0)
   11546:	9782                	jalr	a5
   11548:	2ca05463          	blez	a0,11810 <__sfvwrite_r+0x326>
   1154c:	008a2783          	lw	a5,8(s4)
   11550:	99aa                	add	s3,s3,a0
   11552:	40a90933          	sub	s2,s2,a0
   11556:	8f89                	sub	a5,a5,a0
   11558:	00fa2423          	sw	a5,8(s4)
   1155c:	fbe1                	bnez	a5,1152c <__sfvwrite_r+0x42>
   1155e:	5492                	lw	s1,36(sp)
   11560:	5902                	lw	s2,32(sp)
   11562:	49f2                	lw	s3,28(sp)
   11564:	4b42                	lw	s6,16(sp)
   11566:	4501                	li	a0,0
   11568:	50b2                	lw	ra,44(sp)
   1156a:	5422                	lw	s0,40(sp)
   1156c:	4a62                	lw	s4,24(sp)
   1156e:	4ad2                	lw	s5,20(sp)
   11570:	6145                	addi	sp,sp,48
   11572:	8082                	ret
   11574:	85a2                	mv	a1,s0
   11576:	8556                	mv	a0,s5
   11578:	2c45                	jal	11828 <__swsetup_r>
   1157a:	1a051f63          	bnez	a0,11738 <__sfvwrite_r+0x24e>
   1157e:	00c41703          	lh	a4,12(s0)
   11582:	d226                	sw	s1,36(sp)
   11584:	d04a                	sw	s2,32(sp)
   11586:	ce4e                	sw	s3,28(sp)
   11588:	c85a                	sw	s6,16(sp)
   1158a:	00277793          	andi	a5,a4,2
   1158e:	000a2483          	lw	s1,0(s4)
   11592:	f7d9                	bnez	a5,11520 <__sfvwrite_r+0x36>
   11594:	c65e                	sw	s7,12(sp)
   11596:	c462                	sw	s8,8(sp)
   11598:	00177793          	andi	a5,a4,1
   1159c:	e7e5                	bnez	a5,11684 <__sfvwrite_r+0x19a>
   1159e:	80000bb7          	lui	s7,0x80000
   115a2:	c266                	sw	s9,4(sp)
   115a4:	1bfd                	addi	s7,s7,-1 # 7fffffff <__BSS_END__+0x7ffec30f>
   115a6:	4b01                	li	s6,0
   115a8:	4901                	li	s2,0
   115aa:	0a090f63          	beqz	s2,11668 <__sfvwrite_r+0x17e>
   115ae:	20077793          	andi	a5,a4,512
   115b2:	4008                	lw	a0,0(s0)
   115b4:	00842c03          	lw	s8,8(s0)
   115b8:	18078263          	beqz	a5,1173c <__sfvwrite_r+0x252>
   115bc:	8ce2                	mv	s9,s8
   115be:	1f896c63          	bltu	s2,s8,117b6 <__sfvwrite_r+0x2cc>
   115c2:	48077793          	andi	a5,a4,1152
   115c6:	cba5                	beqz	a5,11636 <__sfvwrite_r+0x14c>
   115c8:	4854                	lw	a3,20(s0)
   115ca:	480c                	lw	a1,16(s0)
   115cc:	00169793          	slli	a5,a3,0x1
   115d0:	97b6                	add	a5,a5,a3
   115d2:	01f7d993          	srli	s3,a5,0x1f
   115d6:	40b50c33          	sub	s8,a0,a1
   115da:	99be                	add	s3,s3,a5
   115dc:	001c0793          	addi	a5,s8,1
   115e0:	4019d993          	srai	s3,s3,0x1
   115e4:	97ca                	add	a5,a5,s2
   115e6:	864e                	mv	a2,s3
   115e8:	00f9f463          	bgeu	s3,a5,115f0 <__sfvwrite_r+0x106>
   115ec:	89be                	mv	s3,a5
   115ee:	863e                	mv	a2,a5
   115f0:	40077713          	andi	a4,a4,1024
   115f4:	1e070663          	beqz	a4,117e0 <__sfvwrite_r+0x2f6>
   115f8:	85b2                	mv	a1,a2
   115fa:	8556                	mv	a0,s5
   115fc:	e52ff0ef          	jal	10c4e <_malloc_r>
   11600:	8caa                	mv	s9,a0
   11602:	20050a63          	beqz	a0,11816 <__sfvwrite_r+0x32c>
   11606:	480c                	lw	a1,16(s0)
   11608:	8662                	mv	a2,s8
   1160a:	2b11                	jal	11b1e <memcpy>
   1160c:	00c45783          	lhu	a5,12(s0)
   11610:	b7f7f793          	andi	a5,a5,-1153
   11614:	0807e793          	ori	a5,a5,128
   11618:	00f41623          	sh	a5,12(s0)
   1161c:	018c8533          	add	a0,s9,s8
   11620:	41898c33          	sub	s8,s3,s8
   11624:	01942823          	sw	s9,16(s0)
   11628:	01842423          	sw	s8,8(s0)
   1162c:	c008                	sw	a0,0(s0)
   1162e:	01342a23          	sw	s3,20(s0)
   11632:	8c4a                	mv	s8,s2
   11634:	8cca                	mv	s9,s2
   11636:	85da                	mv	a1,s6
   11638:	8666                	mv	a2,s9
   1163a:	2121                	jal	11a42 <memmove>
   1163c:	401c                	lw	a5,0(s0)
   1163e:	4418                	lw	a4,8(s0)
   11640:	89ca                	mv	s3,s2
   11642:	97e6                	add	a5,a5,s9
   11644:	c01c                	sw	a5,0(s0)
   11646:	008a2783          	lw	a5,8(s4)
   1164a:	41870733          	sub	a4,a4,s8
   1164e:	c418                	sw	a4,8(s0)
   11650:	413787b3          	sub	a5,a5,s3
   11654:	00fa2423          	sw	a5,8(s4)
   11658:	4901                	li	s2,0
   1165a:	9b4e                	add	s6,s6,s3
   1165c:	12078063          	beqz	a5,1177c <__sfvwrite_r+0x292>
   11660:	00c41703          	lh	a4,12(s0)
   11664:	f40915e3          	bnez	s2,115ae <__sfvwrite_r+0xc4>
   11668:	0004ab03          	lw	s6,0(s1)
   1166c:	0044a903          	lw	s2,4(s1)
   11670:	04a1                	addi	s1,s1,8
   11672:	bf25                	j	115aa <__sfvwrite_r+0xc0>
   11674:	0004a983          	lw	s3,0(s1)
   11678:	0044a903          	lw	s2,4(s1)
   1167c:	04a1                	addi	s1,s1,8
   1167e:	b57d                	j	1152c <__sfvwrite_r+0x42>
   11680:	4501                	li	a0,0
   11682:	8082                	ret
   11684:	4b01                	li	s6,0
   11686:	4501                	li	a0,0
   11688:	4c01                	li	s8,0
   1168a:	4981                	li	s3,0
   1168c:	04098e63          	beqz	s3,116e8 <__sfvwrite_r+0x1fe>
   11690:	c525                	beqz	a0,116f8 <__sfvwrite_r+0x20e>
   11692:	87da                	mv	a5,s6
   11694:	8bce                	mv	s7,s3
   11696:	0137f363          	bgeu	a5,s3,1169c <__sfvwrite_r+0x1b2>
   1169a:	8bbe                	mv	s7,a5
   1169c:	4008                	lw	a0,0(s0)
   1169e:	481c                	lw	a5,16(s0)
   116a0:	4854                	lw	a3,20(s0)
   116a2:	00a7f763          	bgeu	a5,a0,116b0 <__sfvwrite_r+0x1c6>
   116a6:	00842903          	lw	s2,8(s0)
   116aa:	9936                	add	s2,s2,a3
   116ac:	07794063          	blt	s2,s7,1170c <__sfvwrite_r+0x222>
   116b0:	10dbcc63          	blt	s7,a3,117c8 <__sfvwrite_r+0x2de>
   116b4:	505c                	lw	a5,36(s0)
   116b6:	4c4c                	lw	a1,28(s0)
   116b8:	8662                	mv	a2,s8
   116ba:	8556                	mv	a0,s5
   116bc:	9782                	jalr	a5
   116be:	892a                	mv	s2,a0
   116c0:	06a05063          	blez	a0,11720 <__sfvwrite_r+0x236>
   116c4:	412b0b33          	sub	s6,s6,s2
   116c8:	4505                	li	a0,1
   116ca:	0e0b0963          	beqz	s6,117bc <__sfvwrite_r+0x2d2>
   116ce:	008a2783          	lw	a5,8(s4)
   116d2:	9c4a                	add	s8,s8,s2
   116d4:	412989b3          	sub	s3,s3,s2
   116d8:	412787b3          	sub	a5,a5,s2
   116dc:	00fa2423          	sw	a5,8(s4)
   116e0:	f7d5                	bnez	a5,1168c <__sfvwrite_r+0x1a2>
   116e2:	4bb2                	lw	s7,12(sp)
   116e4:	4c22                	lw	s8,8(sp)
   116e6:	bda5                	j	1155e <__sfvwrite_r+0x74>
   116e8:	0044a983          	lw	s3,4(s1)
   116ec:	87a6                	mv	a5,s1
   116ee:	04a1                	addi	s1,s1,8
   116f0:	fe098ce3          	beqz	s3,116e8 <__sfvwrite_r+0x1fe>
   116f4:	0007ac03          	lw	s8,0(a5)
   116f8:	864e                	mv	a2,s3
   116fa:	45a9                	li	a1,10
   116fc:	8562                	mv	a0,s8
   116fe:	2499                	jal	11944 <memchr>
   11700:	10050463          	beqz	a0,11808 <__sfvwrite_r+0x31e>
   11704:	0505                	addi	a0,a0,1
   11706:	41850b33          	sub	s6,a0,s8
   1170a:	b761                	j	11692 <__sfvwrite_r+0x1a8>
   1170c:	85e2                	mv	a1,s8
   1170e:	864a                	mv	a2,s2
   11710:	2e0d                	jal	11a42 <memmove>
   11712:	401c                	lw	a5,0(s0)
   11714:	85a2                	mv	a1,s0
   11716:	8556                	mv	a0,s5
   11718:	97ca                	add	a5,a5,s2
   1171a:	c01c                	sw	a5,0(s0)
   1171c:	33a1                	jal	11464 <_fflush_r>
   1171e:	d15d                	beqz	a0,116c4 <__sfvwrite_r+0x1da>
   11720:	00c41783          	lh	a5,12(s0)
   11724:	4bb2                	lw	s7,12(sp)
   11726:	4c22                	lw	s8,8(sp)
   11728:	5492                	lw	s1,36(sp)
   1172a:	5902                	lw	s2,32(sp)
   1172c:	49f2                	lw	s3,28(sp)
   1172e:	4b42                	lw	s6,16(sp)
   11730:	0407e793          	ori	a5,a5,64
   11734:	00f41623          	sh	a5,12(s0)
   11738:	557d                	li	a0,-1
   1173a:	b53d                	j	11568 <__sfvwrite_r+0x7e>
   1173c:	481c                	lw	a5,16(s0)
   1173e:	04a7e363          	bltu	a5,a0,11784 <__sfvwrite_r+0x29a>
   11742:	485c                	lw	a5,20(s0)
   11744:	04f96063          	bltu	s2,a5,11784 <__sfvwrite_r+0x29a>
   11748:	86ca                	mv	a3,s2
   1174a:	012bf363          	bgeu	s7,s2,11750 <__sfvwrite_r+0x266>
   1174e:	86de                	mv	a3,s7
   11750:	02f6e7b3          	rem	a5,a3,a5
   11754:	5058                	lw	a4,36(s0)
   11756:	4c4c                	lw	a1,28(s0)
   11758:	865a                	mv	a2,s6
   1175a:	8556                	mv	a0,s5
   1175c:	8e9d                	sub	a3,a3,a5
   1175e:	9702                	jalr	a4
   11760:	89aa                	mv	s3,a0
   11762:	04a05463          	blez	a0,117aa <__sfvwrite_r+0x2c0>
   11766:	008a2783          	lw	a5,8(s4)
   1176a:	41390933          	sub	s2,s2,s3
   1176e:	9b4e                	add	s6,s6,s3
   11770:	413787b3          	sub	a5,a5,s3
   11774:	00fa2423          	sw	a5,8(s4)
   11778:	ee0794e3          	bnez	a5,11660 <__sfvwrite_r+0x176>
   1177c:	4bb2                	lw	s7,12(sp)
   1177e:	4c22                	lw	s8,8(sp)
   11780:	4c92                	lw	s9,4(sp)
   11782:	bbf1                	j	1155e <__sfvwrite_r+0x74>
   11784:	89e2                	mv	s3,s8
   11786:	01897363          	bgeu	s2,s8,1178c <__sfvwrite_r+0x2a2>
   1178a:	89ca                	mv	s3,s2
   1178c:	864e                	mv	a2,s3
   1178e:	85da                	mv	a1,s6
   11790:	2c4d                	jal	11a42 <memmove>
   11792:	4018                	lw	a4,0(s0)
   11794:	441c                	lw	a5,8(s0)
   11796:	974e                	add	a4,a4,s3
   11798:	413787b3          	sub	a5,a5,s3
   1179c:	c018                	sw	a4,0(s0)
   1179e:	c41c                	sw	a5,8(s0)
   117a0:	f3f9                	bnez	a5,11766 <__sfvwrite_r+0x27c>
   117a2:	85a2                	mv	a1,s0
   117a4:	8556                	mv	a0,s5
   117a6:	397d                	jal	11464 <_fflush_r>
   117a8:	dd5d                	beqz	a0,11766 <__sfvwrite_r+0x27c>
   117aa:	00c41783          	lh	a5,12(s0)
   117ae:	4bb2                	lw	s7,12(sp)
   117b0:	4c22                	lw	s8,8(sp)
   117b2:	4c92                	lw	s9,4(sp)
   117b4:	bf95                	j	11728 <__sfvwrite_r+0x23e>
   117b6:	8c4a                	mv	s8,s2
   117b8:	8cca                	mv	s9,s2
   117ba:	bdb5                	j	11636 <__sfvwrite_r+0x14c>
   117bc:	85a2                	mv	a1,s0
   117be:	8556                	mv	a0,s5
   117c0:	3155                	jal	11464 <_fflush_r>
   117c2:	f00506e3          	beqz	a0,116ce <__sfvwrite_r+0x1e4>
   117c6:	bfa9                	j	11720 <__sfvwrite_r+0x236>
   117c8:	865e                	mv	a2,s7
   117ca:	85e2                	mv	a1,s8
   117cc:	2c9d                	jal	11a42 <memmove>
   117ce:	4418                	lw	a4,8(s0)
   117d0:	401c                	lw	a5,0(s0)
   117d2:	895e                	mv	s2,s7
   117d4:	41770733          	sub	a4,a4,s7
   117d8:	97de                	add	a5,a5,s7
   117da:	c418                	sw	a4,8(s0)
   117dc:	c01c                	sw	a5,0(s0)
   117de:	b5dd                	j	116c4 <__sfvwrite_r+0x1da>
   117e0:	8556                	mv	a0,s5
   117e2:	2951                	jal	11c76 <_realloc_r>
   117e4:	8caa                	mv	s9,a0
   117e6:	e2051be3          	bnez	a0,1161c <__sfvwrite_r+0x132>
   117ea:	480c                	lw	a1,16(s0)
   117ec:	8556                	mv	a0,s5
   117ee:	a44ff0ef          	jal	10a32 <_free_r>
   117f2:	00c41783          	lh	a5,12(s0)
   117f6:	4731                	li	a4,12
   117f8:	4bb2                	lw	s7,12(sp)
   117fa:	4c22                	lw	s8,8(sp)
   117fc:	4c92                	lw	s9,4(sp)
   117fe:	00eaa023          	sw	a4,0(s5)
   11802:	f7f7f793          	andi	a5,a5,-129
   11806:	b70d                	j	11728 <__sfvwrite_r+0x23e>
   11808:	00198793          	addi	a5,s3,1
   1180c:	8b3e                	mv	s6,a5
   1180e:	b559                	j	11694 <__sfvwrite_r+0x1aa>
   11810:	00c41783          	lh	a5,12(s0)
   11814:	bf11                	j	11728 <__sfvwrite_r+0x23e>
   11816:	47b1                	li	a5,12
   11818:	00faa023          	sw	a5,0(s5)
   1181c:	4bb2                	lw	s7,12(sp)
   1181e:	00c41783          	lh	a5,12(s0)
   11822:	4c22                	lw	s8,8(sp)
   11824:	4c92                	lw	s9,4(sp)
   11826:	b709                	j	11728 <__sfvwrite_r+0x23e>

00011828 <__swsetup_r>:
   11828:	d3c1a783          	lw	a5,-708(gp) # 139bc <_impure_ptr>
   1182c:	1141                	addi	sp,sp,-16
   1182e:	c422                	sw	s0,8(sp)
   11830:	c226                	sw	s1,4(sp)
   11832:	c606                	sw	ra,12(sp)
   11834:	84aa                	mv	s1,a0
   11836:	842e                	mv	s0,a1
   11838:	c399                	beqz	a5,1183e <__swsetup_r+0x16>
   1183a:	5bd8                	lw	a4,52(a5)
   1183c:	cb61                	beqz	a4,1190c <__swsetup_r+0xe4>
   1183e:	00c41783          	lh	a5,12(s0)
   11842:	0087f713          	andi	a4,a5,8
   11846:	c315                	beqz	a4,1186a <__swsetup_r+0x42>
   11848:	4818                	lw	a4,16(s0)
   1184a:	cf05                	beqz	a4,11882 <__swsetup_r+0x5a>
   1184c:	0017f713          	andi	a4,a5,1
   11850:	c32d                	beqz	a4,118b2 <__swsetup_r+0x8a>
   11852:	485c                	lw	a5,20(s0)
   11854:	00042423          	sw	zero,8(s0)
   11858:	40f007b3          	neg	a5,a5
   1185c:	cc1c                	sw	a5,24(s0)
   1185e:	4501                	li	a0,0
   11860:	40b2                	lw	ra,12(sp)
   11862:	4422                	lw	s0,8(sp)
   11864:	4492                	lw	s1,4(sp)
   11866:	0141                	addi	sp,sp,16
   11868:	8082                	ret
   1186a:	0107f713          	andi	a4,a5,16
   1186e:	c379                	beqz	a4,11934 <__swsetup_r+0x10c>
   11870:	0047f713          	andi	a4,a5,4
   11874:	e721                	bnez	a4,118bc <__swsetup_r+0x94>
   11876:	4818                	lw	a4,16(s0)
   11878:	0087e793          	ori	a5,a5,8
   1187c:	00f41623          	sh	a5,12(s0)
   11880:	f771                	bnez	a4,1184c <__swsetup_r+0x24>
   11882:	2807f693          	andi	a3,a5,640
   11886:	20000613          	li	a2,512
   1188a:	06c69063          	bne	a3,a2,118ea <__swsetup_r+0xc2>
   1188e:	0017f693          	andi	a3,a5,1
   11892:	c2c9                	beqz	a3,11914 <__swsetup_r+0xec>
   11894:	4858                	lw	a4,20(s0)
   11896:	00042423          	sw	zero,8(s0)
   1189a:	40e00733          	neg	a4,a4
   1189e:	cc18                	sw	a4,24(s0)
   118a0:	0807f713          	andi	a4,a5,128
   118a4:	df4d                	beqz	a4,1185e <__swsetup_r+0x36>
   118a6:	0407e793          	ori	a5,a5,64
   118aa:	00f41623          	sh	a5,12(s0)
   118ae:	557d                	li	a0,-1
   118b0:	bf45                	j	11860 <__swsetup_r+0x38>
   118b2:	8b89                	andi	a5,a5,2
   118b4:	eb85                	bnez	a5,118e4 <__swsetup_r+0xbc>
   118b6:	485c                	lw	a5,20(s0)
   118b8:	c41c                	sw	a5,8(s0)
   118ba:	b755                	j	1185e <__swsetup_r+0x36>
   118bc:	580c                	lw	a1,48(s0)
   118be:	cd81                	beqz	a1,118d6 <__swsetup_r+0xae>
   118c0:	04040713          	addi	a4,s0,64
   118c4:	00e58763          	beq	a1,a4,118d2 <__swsetup_r+0xaa>
   118c8:	8526                	mv	a0,s1
   118ca:	968ff0ef          	jal	10a32 <_free_r>
   118ce:	00c41783          	lh	a5,12(s0)
   118d2:	02042823          	sw	zero,48(s0)
   118d6:	4818                	lw	a4,16(s0)
   118d8:	fdb7f793          	andi	a5,a5,-37
   118dc:	00042223          	sw	zero,4(s0)
   118e0:	c018                	sw	a4,0(s0)
   118e2:	bf59                	j	11878 <__swsetup_r+0x50>
   118e4:	00042423          	sw	zero,8(s0)
   118e8:	bf9d                	j	1185e <__swsetup_r+0x36>
   118ea:	8526                	mv	a0,s1
   118ec:	85a2                	mv	a1,s0
   118ee:	2f49                	jal	12080 <__smakebuf_r>
   118f0:	00c41783          	lh	a5,12(s0)
   118f4:	4818                	lw	a4,16(s0)
   118f6:	0017f693          	andi	a3,a5,1
   118fa:	c685                	beqz	a3,11922 <__swsetup_r+0xfa>
   118fc:	4854                	lw	a3,20(s0)
   118fe:	00042423          	sw	zero,8(s0)
   11902:	40d006b3          	neg	a3,a3
   11906:	cc14                	sw	a3,24(s0)
   11908:	df41                	beqz	a4,118a0 <__swsetup_r+0x78>
   1190a:	bf91                	j	1185e <__swsetup_r+0x36>
   1190c:	853e                	mv	a0,a5
   1190e:	aa7fe0ef          	jal	103b4 <__sinit>
   11912:	b735                	j	1183e <__swsetup_r+0x16>
   11914:	0027f693          	andi	a3,a5,2
   11918:	ea99                	bnez	a3,1192e <__swsetup_r+0x106>
   1191a:	4850                	lw	a2,20(s0)
   1191c:	c410                	sw	a2,8(s0)
   1191e:	d349                	beqz	a4,118a0 <__swsetup_r+0x78>
   11920:	bf3d                	j	1185e <__swsetup_r+0x36>
   11922:	0027f693          	andi	a3,a5,2
   11926:	4601                	li	a2,0
   11928:	faf5                	bnez	a3,1191c <__swsetup_r+0xf4>
   1192a:	4850                	lw	a2,20(s0)
   1192c:	bfc5                	j	1191c <__swsetup_r+0xf4>
   1192e:	00042423          	sw	zero,8(s0)
   11932:	b7bd                	j	118a0 <__swsetup_r+0x78>
   11934:	4725                	li	a4,9
   11936:	0407e793          	ori	a5,a5,64
   1193a:	c098                	sw	a4,0(s1)
   1193c:	00f41623          	sh	a5,12(s0)
   11940:	557d                	li	a0,-1
   11942:	bf39                	j	11860 <__swsetup_r+0x38>

00011944 <memchr>:
   11944:	00357713          	andi	a4,a0,3
   11948:	87aa                	mv	a5,a0
   1194a:	0ff5f813          	zext.b	a6,a1
   1194e:	832a                	mv	t1,a0
   11950:	c70d                	beqz	a4,1197a <memchr+0x36>
   11952:	00c508b3          	add	a7,a0,a2
   11956:	a039                	j	11964 <memchr+0x20>
   11958:	0007c683          	lbu	a3,0(a5)
   1195c:	07068c63          	beq	a3,a6,119d4 <memchr+0x90>
   11960:	cb11                	beqz	a4,11974 <memchr+0x30>
   11962:	87aa                	mv	a5,a0
   11964:	00178513          	addi	a0,a5,1
   11968:	00357713          	andi	a4,a0,3
   1196c:	ff1796e3          	bne	a5,a7,11958 <memchr+0x14>
   11970:	4501                	li	a0,0
   11972:	8082                	ret
   11974:	167d                	addi	a2,a2,-1
   11976:	961a                	add	a2,a2,t1
   11978:	8e1d                	sub	a2,a2,a5
   1197a:	478d                	li	a5,3
   1197c:	04c7f163          	bgeu	a5,a2,119be <memchr+0x7a>
   11980:	0ff5f593          	zext.b	a1,a1
   11984:	00859693          	slli	a3,a1,0x8
   11988:	96ae                	add	a3,a3,a1
   1198a:	01069713          	slli	a4,a3,0x10
   1198e:	feff0337          	lui	t1,0xfeff0
   11992:	808088b7          	lui	a7,0x80808
   11996:	85be                	mv	a1,a5
   11998:	96ba                	add	a3,a3,a4
   1199a:	eff30313          	addi	t1,t1,-257 # fefefeff <__BSS_END__+0xfefdc20f>
   1199e:	08088893          	addi	a7,a7,128 # 80808080 <__BSS_END__+0x807f4390>
   119a2:	411c                	lw	a5,0(a0)
   119a4:	8fb5                	xor	a5,a5,a3
   119a6:	00678733          	add	a4,a5,t1
   119aa:	fff7c793          	not	a5,a5
   119ae:	8ff9                	and	a5,a5,a4
   119b0:	0117f7b3          	and	a5,a5,a7
   119b4:	e791                	bnez	a5,119c0 <memchr+0x7c>
   119b6:	1671                	addi	a2,a2,-4
   119b8:	0511                	addi	a0,a0,4
   119ba:	fec5e4e3          	bltu	a1,a2,119a2 <memchr+0x5e>
   119be:	da4d                	beqz	a2,11970 <memchr+0x2c>
   119c0:	962a                	add	a2,a2,a0
   119c2:	a021                	j	119ca <memchr+0x86>
   119c4:	0505                	addi	a0,a0,1
   119c6:	fac505e3          	beq	a0,a2,11970 <memchr+0x2c>
   119ca:	00054783          	lbu	a5,0(a0)
   119ce:	ff079be3          	bne	a5,a6,119c4 <memchr+0x80>
   119d2:	8082                	ret
   119d4:	853e                	mv	a0,a5
   119d6:	8082                	ret

000119d8 <_sbrk_r>:
   119d8:	1141                	addi	sp,sp,-16
   119da:	c422                	sw	s0,8(sp)
   119dc:	c226                	sw	s1,4(sp)
   119de:	842a                	mv	s0,a0
   119e0:	852e                	mv	a0,a1
   119e2:	c606                	sw	ra,12(sp)
   119e4:	d401a623          	sw	zero,-692(gp) # 139cc <errno>
   119e8:	17d000ef          	jal	12364 <_sbrk>
   119ec:	57fd                	li	a5,-1
   119ee:	00f50763          	beq	a0,a5,119fc <_sbrk_r+0x24>
   119f2:	40b2                	lw	ra,12(sp)
   119f4:	4422                	lw	s0,8(sp)
   119f6:	4492                	lw	s1,4(sp)
   119f8:	0141                	addi	sp,sp,16
   119fa:	8082                	ret
   119fc:	d4c1a783          	lw	a5,-692(gp) # 139cc <errno>
   11a00:	dbed                	beqz	a5,119f2 <_sbrk_r+0x1a>
   11a02:	40b2                	lw	ra,12(sp)
   11a04:	c01c                	sw	a5,0(s0)
   11a06:	4422                	lw	s0,8(sp)
   11a08:	4492                	lw	s1,4(sp)
   11a0a:	0141                	addi	sp,sp,16
   11a0c:	8082                	ret

00011a0e <__libc_fini_array>:
   11a0e:	1141                	addi	sp,sp,-16
   11a10:	c422                	sw	s0,8(sp)
   11a12:	67cd                	lui	a5,0x13
   11a14:	644d                	lui	s0,0x13
   11a16:	48040413          	addi	s0,s0,1152 # 13480 <__sglue>
   11a1a:	47c78793          	addi	a5,a5,1148 # 1347c <__do_global_dtors_aux_fini_array_entry>
   11a1e:	8c1d                	sub	s0,s0,a5
   11a20:	c226                	sw	s1,4(sp)
   11a22:	c606                	sw	ra,12(sp)
   11a24:	40245493          	srai	s1,s0,0x2
   11a28:	c881                	beqz	s1,11a38 <__libc_fini_array+0x2a>
   11a2a:	1471                	addi	s0,s0,-4
   11a2c:	943e                	add	s0,s0,a5
   11a2e:	401c                	lw	a5,0(s0)
   11a30:	14fd                	addi	s1,s1,-1
   11a32:	1471                	addi	s0,s0,-4
   11a34:	9782                	jalr	a5
   11a36:	fce5                	bnez	s1,11a2e <__libc_fini_array+0x20>
   11a38:	40b2                	lw	ra,12(sp)
   11a3a:	4422                	lw	s0,8(sp)
   11a3c:	4492                	lw	s1,4(sp)
   11a3e:	0141                	addi	sp,sp,16
   11a40:	8082                	ret

00011a42 <memmove>:
   11a42:	02a5f263          	bgeu	a1,a0,11a66 <memmove+0x24>
   11a46:	00c58733          	add	a4,a1,a2
   11a4a:	00e57e63          	bgeu	a0,a4,11a66 <memmove+0x24>
   11a4e:	00c507b3          	add	a5,a0,a2
   11a52:	ca1d                	beqz	a2,11a88 <memmove+0x46>
   11a54:	fff74683          	lbu	a3,-1(a4)
   11a58:	17fd                	addi	a5,a5,-1
   11a5a:	177d                	addi	a4,a4,-1
   11a5c:	00d78023          	sb	a3,0(a5)
   11a60:	fef51ae3          	bne	a0,a5,11a54 <memmove+0x12>
   11a64:	8082                	ret
   11a66:	47bd                	li	a5,15
   11a68:	02c7e163          	bltu	a5,a2,11a8a <memmove+0x48>
   11a6c:	87aa                	mv	a5,a0
   11a6e:	fff60693          	addi	a3,a2,-1
   11a72:	c25d                	beqz	a2,11b18 <memmove+0xd6>
   11a74:	0685                	addi	a3,a3,1
   11a76:	96be                	add	a3,a3,a5
   11a78:	0005c703          	lbu	a4,0(a1)
   11a7c:	0785                	addi	a5,a5,1
   11a7e:	0585                	addi	a1,a1,1
   11a80:	fee78fa3          	sb	a4,-1(a5)
   11a84:	fed79ae3          	bne	a5,a3,11a78 <memmove+0x36>
   11a88:	8082                	ret
   11a8a:	00b567b3          	or	a5,a0,a1
   11a8e:	8b8d                	andi	a5,a5,3
   11a90:	88ae                	mv	a7,a1
   11a92:	efbd                	bnez	a5,11b10 <memmove+0xce>
   11a94:	ff060793          	addi	a5,a2,-16
   11a98:	ff07f813          	andi	a6,a5,-16
   11a9c:	0841                	addi	a6,a6,16
   11a9e:	982a                	add	a6,a6,a0
   11aa0:	872a                	mv	a4,a0
   11aa2:	4194                	lw	a3,0(a1)
   11aa4:	05c1                	addi	a1,a1,16
   11aa6:	0741                	addi	a4,a4,16
   11aa8:	fed72823          	sw	a3,-16(a4)
   11aac:	ff45a683          	lw	a3,-12(a1)
   11ab0:	fed72a23          	sw	a3,-12(a4)
   11ab4:	ff85a683          	lw	a3,-8(a1)
   11ab8:	fed72c23          	sw	a3,-8(a4)
   11abc:	ffc5a683          	lw	a3,-4(a1)
   11ac0:	fed72e23          	sw	a3,-4(a4)
   11ac4:	fd071fe3          	bne	a4,a6,11aa2 <memmove+0x60>
   11ac8:	9bc1                	andi	a5,a5,-16
   11aca:	01178733          	add	a4,a5,a7
   11ace:	01070593          	addi	a1,a4,16
   11ad2:	97aa                	add	a5,a5,a0
   11ad4:	00c67813          	andi	a6,a2,12
   11ad8:	07c1                	addi	a5,a5,16
   11ada:	8e2e                	mv	t3,a1
   11adc:	00f67693          	andi	a3,a2,15
   11ae0:	02080d63          	beqz	a6,11b1a <memmove+0xd8>
   11ae4:	16f1                	addi	a3,a3,-4
   11ae6:	9af1                	andi	a3,a3,-4
   11ae8:	9736                	add	a4,a4,a3
   11aea:	0751                	addi	a4,a4,20
   11aec:	41150833          	sub	a6,a0,a7
   11af0:	0005a303          	lw	t1,0(a1)
   11af4:	010588b3          	add	a7,a1,a6
   11af8:	0591                	addi	a1,a1,4
   11afa:	0068a023          	sw	t1,0(a7)
   11afe:	fee599e3          	bne	a1,a4,11af0 <memmove+0xae>
   11b02:	00468713          	addi	a4,a3,4
   11b06:	01c705b3          	add	a1,a4,t3
   11b0a:	97ba                	add	a5,a5,a4
   11b0c:	8a0d                	andi	a2,a2,3
   11b0e:	b785                	j	11a6e <memmove+0x2c>
   11b10:	fff60693          	addi	a3,a2,-1
   11b14:	87aa                	mv	a5,a0
   11b16:	bfb9                	j	11a74 <memmove+0x32>
   11b18:	8082                	ret
   11b1a:	8636                	mv	a2,a3
   11b1c:	bf89                	j	11a6e <memmove+0x2c>

00011b1e <memcpy>:
   11b1e:	00a5c7b3          	xor	a5,a1,a0
   11b22:	8b8d                	andi	a5,a5,3
   11b24:	00c508b3          	add	a7,a0,a2
   11b28:	e7b1                	bnez	a5,11b74 <memcpy+0x56>
   11b2a:	478d                	li	a5,3
   11b2c:	04c7f463          	bgeu	a5,a2,11b74 <memcpy+0x56>
   11b30:	00357793          	andi	a5,a0,3
   11b34:	872a                	mv	a4,a0
   11b36:	e7dd                	bnez	a5,11be4 <memcpy+0xc6>
   11b38:	ffc8f613          	andi	a2,a7,-4
   11b3c:	40e606b3          	sub	a3,a2,a4
   11b40:	02000793          	li	a5,32
   11b44:	04d7c463          	blt	a5,a3,11b8c <memcpy+0x6e>
   11b48:	86ae                	mv	a3,a1
   11b4a:	87ba                	mv	a5,a4
   11b4c:	02c77163          	bgeu	a4,a2,11b6e <memcpy+0x50>
   11b50:	0006a803          	lw	a6,0(a3)
   11b54:	0791                	addi	a5,a5,4
   11b56:	0691                	addi	a3,a3,4
   11b58:	ff07ae23          	sw	a6,-4(a5)
   11b5c:	fec7eae3          	bltu	a5,a2,11b50 <memcpy+0x32>
   11b60:	167d                	addi	a2,a2,-1
   11b62:	8e19                	sub	a2,a2,a4
   11b64:	9a71                	andi	a2,a2,-4
   11b66:	0591                	addi	a1,a1,4
   11b68:	0711                	addi	a4,a4,4
   11b6a:	95b2                	add	a1,a1,a2
   11b6c:	9732                	add	a4,a4,a2
   11b6e:	01176663          	bltu	a4,a7,11b7a <memcpy+0x5c>
   11b72:	8082                	ret
   11b74:	872a                	mv	a4,a0
   11b76:	ff157ee3          	bgeu	a0,a7,11b72 <memcpy+0x54>
   11b7a:	0005c783          	lbu	a5,0(a1)
   11b7e:	0705                	addi	a4,a4,1
   11b80:	0585                	addi	a1,a1,1
   11b82:	fef70fa3          	sb	a5,-1(a4)
   11b86:	fee89ae3          	bne	a7,a4,11b7a <memcpy+0x5c>
   11b8a:	8082                	ret
   11b8c:	5194                	lw	a3,32(a1)
   11b8e:	0005a383          	lw	t2,0(a1)
   11b92:	0045a283          	lw	t0,4(a1)
   11b96:	0085af83          	lw	t6,8(a1)
   11b9a:	00c5af03          	lw	t5,12(a1)
   11b9e:	0105ae83          	lw	t4,16(a1)
   11ba2:	0145ae03          	lw	t3,20(a1)
   11ba6:	0185a303          	lw	t1,24(a1)
   11baa:	01c5a803          	lw	a6,28(a1)
   11bae:	02470713          	addi	a4,a4,36
   11bb2:	fed72e23          	sw	a3,-4(a4)
   11bb6:	fc772e23          	sw	t2,-36(a4)
   11bba:	40e606b3          	sub	a3,a2,a4
   11bbe:	fe572023          	sw	t0,-32(a4)
   11bc2:	fff72223          	sw	t6,-28(a4)
   11bc6:	ffe72423          	sw	t5,-24(a4)
   11bca:	ffd72623          	sw	t4,-20(a4)
   11bce:	ffc72823          	sw	t3,-16(a4)
   11bd2:	fe672a23          	sw	t1,-12(a4)
   11bd6:	ff072c23          	sw	a6,-8(a4)
   11bda:	02458593          	addi	a1,a1,36
   11bde:	fad7c7e3          	blt	a5,a3,11b8c <memcpy+0x6e>
   11be2:	b79d                	j	11b48 <memcpy+0x2a>
   11be4:	0005c683          	lbu	a3,0(a1)
   11be8:	0705                	addi	a4,a4,1
   11bea:	00377793          	andi	a5,a4,3
   11bee:	fed70fa3          	sb	a3,-1(a4)
   11bf2:	0585                	addi	a1,a1,1
   11bf4:	d3b1                	beqz	a5,11b38 <memcpy+0x1a>
   11bf6:	0005c683          	lbu	a3,0(a1)
   11bfa:	0705                	addi	a4,a4,1
   11bfc:	00377793          	andi	a5,a4,3
   11c00:	fed70fa3          	sb	a3,-1(a4)
   11c04:	0585                	addi	a1,a1,1
   11c06:	fff9                	bnez	a5,11be4 <memcpy+0xc6>
   11c08:	bf05                	j	11b38 <memcpy+0x1a>

00011c0a <__register_exitproc>:
   11c0a:	d501a783          	lw	a5,-688(gp) # 139d0 <__atexit>
   11c0e:	c3a1                	beqz	a5,11c4e <__register_exitproc+0x44>
   11c10:	43d8                	lw	a4,4(a5)
   11c12:	487d                	li	a6,31
   11c14:	04e84f63          	blt	a6,a4,11c72 <__register_exitproc+0x68>
   11c18:	00271813          	slli	a6,a4,0x2
   11c1c:	c11d                	beqz	a0,11c42 <__register_exitproc+0x38>
   11c1e:	01078333          	add	t1,a5,a6
   11c22:	08c32423          	sw	a2,136(t1)
   11c26:	1887a883          	lw	a7,392(a5)
   11c2a:	4605                	li	a2,1
   11c2c:	00e61633          	sll	a2,a2,a4
   11c30:	00c8e8b3          	or	a7,a7,a2
   11c34:	1917a423          	sw	a7,392(a5)
   11c38:	10d32423          	sw	a3,264(t1)
   11c3c:	4689                	li	a3,2
   11c3e:	00d50f63          	beq	a0,a3,11c5c <__register_exitproc+0x52>
   11c42:	0705                	addi	a4,a4,1
   11c44:	c3d8                	sw	a4,4(a5)
   11c46:	97c2                	add	a5,a5,a6
   11c48:	c78c                	sw	a1,8(a5)
   11c4a:	4501                	li	a0,0
   11c4c:	8082                	ret
   11c4e:	ee018813          	addi	a6,gp,-288 # 13b60 <__atexit0>
   11c52:	d501a823          	sw	a6,-688(gp) # 139d0 <__atexit>
   11c56:	ee018793          	addi	a5,gp,-288 # 13b60 <__atexit0>
   11c5a:	bf5d                	j	11c10 <__register_exitproc+0x6>
   11c5c:	18c7a683          	lw	a3,396(a5)
   11c60:	0705                	addi	a4,a4,1
   11c62:	c3d8                	sw	a4,4(a5)
   11c64:	8ed1                	or	a3,a3,a2
   11c66:	18d7a623          	sw	a3,396(a5)
   11c6a:	97c2                	add	a5,a5,a6
   11c6c:	c78c                	sw	a1,8(a5)
   11c6e:	4501                	li	a0,0
   11c70:	8082                	ret
   11c72:	557d                	li	a0,-1
   11c74:	8082                	ret

00011c76 <_realloc_r>:
   11c76:	7179                	addi	sp,sp,-48
   11c78:	d226                	sw	s1,36(sp)
   11c7a:	d606                	sw	ra,44(sp)
   11c7c:	84b2                	mv	s1,a2
   11c7e:	14058d63          	beqz	a1,11dd8 <_realloc_r+0x162>
   11c82:	d422                	sw	s0,40(sp)
   11c84:	d04a                	sw	s2,32(sp)
   11c86:	842e                	mv	s0,a1
   11c88:	ce4e                	sw	s3,28(sp)
   11c8a:	ca56                	sw	s5,20(sp)
   11c8c:	cc52                	sw	s4,24(sp)
   11c8e:	892a                	mv	s2,a0
   11c90:	d8aff0ef          	jal	1121a <__malloc_lock>
   11c94:	ffc42703          	lw	a4,-4(s0)
   11c98:	00b48793          	addi	a5,s1,11
   11c9c:	46d9                	li	a3,22
   11c9e:	ffc77993          	andi	s3,a4,-4
   11ca2:	ff840a93          	addi	s5,s0,-8
   11ca6:	0af6ff63          	bgeu	a3,a5,11d64 <_realloc_r+0xee>
   11caa:	ff87fa13          	andi	s4,a5,-8
   11cae:	0a07ce63          	bltz	a5,11d6a <_realloc_r+0xf4>
   11cb2:	0a9a6c63          	bltu	s4,s1,11d6a <_realloc_r+0xf4>
   11cb6:	0d49d563          	bge	s3,s4,11d80 <_realloc_r+0x10a>
   11cba:	67cd                	lui	a5,0x13
   11cbc:	c462                	sw	s8,8(sp)
   11cbe:	5b078c13          	addi	s8,a5,1456 # 135b0 <__malloc_av_>
   11cc2:	008c2603          	lw	a2,8(s8)
   11cc6:	013a86b3          	add	a3,s5,s3
   11cca:	42dc                	lw	a5,4(a3)
   11ccc:	12d60e63          	beq	a2,a3,11e08 <_realloc_r+0x192>
   11cd0:	ffe7f613          	andi	a2,a5,-2
   11cd4:	9636                	add	a2,a2,a3
   11cd6:	4250                	lw	a2,4(a2)
   11cd8:	8a05                	andi	a2,a2,1
   11cda:	e27d                	bnez	a2,11dc0 <_realloc_r+0x14a>
   11cdc:	9bf1                	andi	a5,a5,-4
   11cde:	00f98633          	add	a2,s3,a5
   11ce2:	09465963          	bge	a2,s4,11d74 <_realloc_r+0xfe>
   11ce6:	8b05                	andi	a4,a4,1
   11ce8:	e70d                	bnez	a4,11d12 <_realloc_r+0x9c>
   11cea:	c65e                	sw	s7,12(sp)
   11cec:	ff842b83          	lw	s7,-8(s0)
   11cf0:	c85a                	sw	s6,16(sp)
   11cf2:	417a8bb3          	sub	s7,s5,s7
   11cf6:	004ba703          	lw	a4,4(s7)
   11cfa:	9b71                	andi	a4,a4,-4
   11cfc:	97ba                	add	a5,a5,a4
   11cfe:	01378b33          	add	s6,a5,s3
   11d02:	234b5663          	bge	s6,s4,11f2e <_realloc_r+0x2b8>
   11d06:	00e98b33          	add	s6,s3,a4
   11d0a:	1d4b5463          	bge	s6,s4,11ed2 <_realloc_r+0x25c>
   11d0e:	4b42                	lw	s6,16(sp)
   11d10:	4bb2                	lw	s7,12(sp)
   11d12:	85a6                	mv	a1,s1
   11d14:	854a                	mv	a0,s2
   11d16:	f39fe0ef          	jal	10c4e <_malloc_r>
   11d1a:	84aa                	mv	s1,a0
   11d1c:	2c050463          	beqz	a0,11fe4 <_realloc_r+0x36e>
   11d20:	ffc42783          	lw	a5,-4(s0)
   11d24:	ff850713          	addi	a4,a0,-8
   11d28:	9bf9                	andi	a5,a5,-2
   11d2a:	97d6                	add	a5,a5,s5
   11d2c:	18e78d63          	beq	a5,a4,11ec6 <_realloc_r+0x250>
   11d30:	ffc98613          	addi	a2,s3,-4
   11d34:	02400793          	li	a5,36
   11d38:	1ec7e863          	bltu	a5,a2,11f28 <_realloc_r+0x2b2>
   11d3c:	474d                	li	a4,19
   11d3e:	16c76863          	bltu	a4,a2,11eae <_realloc_r+0x238>
   11d42:	87aa                	mv	a5,a0
   11d44:	8722                	mv	a4,s0
   11d46:	4314                	lw	a3,0(a4)
   11d48:	c394                	sw	a3,0(a5)
   11d4a:	4354                	lw	a3,4(a4)
   11d4c:	c3d4                	sw	a3,4(a5)
   11d4e:	4718                	lw	a4,8(a4)
   11d50:	c798                	sw	a4,8(a5)
   11d52:	85a2                	mv	a1,s0
   11d54:	854a                	mv	a0,s2
   11d56:	cddfe0ef          	jal	10a32 <_free_r>
   11d5a:	854a                	mv	a0,s2
   11d5c:	cc0ff0ef          	jal	1121c <__malloc_unlock>
   11d60:	4c22                	lw	s8,8(sp)
   11d62:	a0a9                	j	11dac <_realloc_r+0x136>
   11d64:	4a41                	li	s4,16
   11d66:	f49a78e3          	bgeu	s4,s1,11cb6 <_realloc_r+0x40>
   11d6a:	47b1                	li	a5,12
   11d6c:	00f92023          	sw	a5,0(s2)
   11d70:	4481                	li	s1,0
   11d72:	a82d                	j	11dac <_realloc_r+0x136>
   11d74:	46dc                	lw	a5,12(a3)
   11d76:	4698                	lw	a4,8(a3)
   11d78:	4c22                	lw	s8,8(sp)
   11d7a:	89b2                	mv	s3,a2
   11d7c:	c75c                	sw	a5,12(a4)
   11d7e:	c798                	sw	a4,8(a5)
   11d80:	004aa783          	lw	a5,4(s5)
   11d84:	414986b3          	sub	a3,s3,s4
   11d88:	463d                	li	a2,15
   11d8a:	8b85                	andi	a5,a5,1
   11d8c:	013a8733          	add	a4,s5,s3
   11d90:	04d66a63          	bltu	a2,a3,11de4 <_realloc_r+0x16e>
   11d94:	0137e7b3          	or	a5,a5,s3
   11d98:	00faa223          	sw	a5,4(s5)
   11d9c:	435c                	lw	a5,4(a4)
   11d9e:	0017e793          	ori	a5,a5,1
   11da2:	c35c                	sw	a5,4(a4)
   11da4:	854a                	mv	a0,s2
   11da6:	c76ff0ef          	jal	1121c <__malloc_unlock>
   11daa:	84a2                	mv	s1,s0
   11dac:	5422                	lw	s0,40(sp)
   11dae:	50b2                	lw	ra,44(sp)
   11db0:	5902                	lw	s2,32(sp)
   11db2:	49f2                	lw	s3,28(sp)
   11db4:	4a62                	lw	s4,24(sp)
   11db6:	4ad2                	lw	s5,20(sp)
   11db8:	8526                	mv	a0,s1
   11dba:	5492                	lw	s1,36(sp)
   11dbc:	6145                	addi	sp,sp,48
   11dbe:	8082                	ret
   11dc0:	8b05                	andi	a4,a4,1
   11dc2:	fb21                	bnez	a4,11d12 <_realloc_r+0x9c>
   11dc4:	c65e                	sw	s7,12(sp)
   11dc6:	ff842b83          	lw	s7,-8(s0)
   11dca:	c85a                	sw	s6,16(sp)
   11dcc:	417a8bb3          	sub	s7,s5,s7
   11dd0:	004ba703          	lw	a4,4(s7)
   11dd4:	9b71                	andi	a4,a4,-4
   11dd6:	bf05                	j	11d06 <_realloc_r+0x90>
   11dd8:	50b2                	lw	ra,44(sp)
   11dda:	5492                	lw	s1,36(sp)
   11ddc:	85b2                	mv	a1,a2
   11dde:	6145                	addi	sp,sp,48
   11de0:	e6ffe06f          	j	10c4e <_malloc_r>
   11de4:	0147e7b3          	or	a5,a5,s4
   11de8:	00faa223          	sw	a5,4(s5)
   11dec:	014a85b3          	add	a1,s5,s4
   11df0:	0016e693          	ori	a3,a3,1
   11df4:	c1d4                	sw	a3,4(a1)
   11df6:	435c                	lw	a5,4(a4)
   11df8:	05a1                	addi	a1,a1,8
   11dfa:	854a                	mv	a0,s2
   11dfc:	0017e793          	ori	a5,a5,1
   11e00:	c35c                	sw	a5,4(a4)
   11e02:	c31fe0ef          	jal	10a32 <_free_r>
   11e06:	bf79                	j	11da4 <_realloc_r+0x12e>
   11e08:	9bf1                	andi	a5,a5,-4
   11e0a:	013786b3          	add	a3,a5,s3
   11e0e:	010a0613          	addi	a2,s4,16
   11e12:	18c6d763          	bge	a3,a2,11fa0 <_realloc_r+0x32a>
   11e16:	8b05                	andi	a4,a4,1
   11e18:	ee071de3          	bnez	a4,11d12 <_realloc_r+0x9c>
   11e1c:	c65e                	sw	s7,12(sp)
   11e1e:	ff842b83          	lw	s7,-8(s0)
   11e22:	c85a                	sw	s6,16(sp)
   11e24:	417a8bb3          	sub	s7,s5,s7
   11e28:	004ba703          	lw	a4,4(s7)
   11e2c:	9b71                	andi	a4,a4,-4
   11e2e:	97ba                	add	a5,a5,a4
   11e30:	01378b33          	add	s6,a5,s3
   11e34:	eccb49e3          	blt	s6,a2,11d06 <_realloc_r+0x90>
   11e38:	00cba783          	lw	a5,12(s7)
   11e3c:	008ba703          	lw	a4,8(s7)
   11e40:	ffc98613          	addi	a2,s3,-4
   11e44:	02400693          	li	a3,36
   11e48:	c75c                	sw	a5,12(a4)
   11e4a:	c798                	sw	a4,8(a5)
   11e4c:	008b8493          	addi	s1,s7,8
   11e50:	1cc6e763          	bltu	a3,a2,1201e <_realloc_r+0x3a8>
   11e54:	474d                	li	a4,19
   11e56:	87a6                	mv	a5,s1
   11e58:	00c77e63          	bgeu	a4,a2,11e74 <_realloc_r+0x1fe>
   11e5c:	4018                	lw	a4,0(s0)
   11e5e:	47ed                	li	a5,27
   11e60:	00eba423          	sw	a4,8(s7)
   11e64:	4058                	lw	a4,4(s0)
   11e66:	00eba623          	sw	a4,12(s7)
   11e6a:	1cc7ea63          	bltu	a5,a2,1203e <_realloc_r+0x3c8>
   11e6e:	0421                	addi	s0,s0,8
   11e70:	010b8793          	addi	a5,s7,16
   11e74:	4018                	lw	a4,0(s0)
   11e76:	c398                	sw	a4,0(a5)
   11e78:	4058                	lw	a4,4(s0)
   11e7a:	c3d8                	sw	a4,4(a5)
   11e7c:	4418                	lw	a4,8(s0)
   11e7e:	c798                	sw	a4,8(a5)
   11e80:	014b87b3          	add	a5,s7,s4
   11e84:	414b0733          	sub	a4,s6,s4
   11e88:	00fc2423          	sw	a5,8(s8)
   11e8c:	00176713          	ori	a4,a4,1
   11e90:	c3d8                	sw	a4,4(a5)
   11e92:	004ba783          	lw	a5,4(s7)
   11e96:	854a                	mv	a0,s2
   11e98:	8b85                	andi	a5,a5,1
   11e9a:	0147e7b3          	or	a5,a5,s4
   11e9e:	00fba223          	sw	a5,4(s7)
   11ea2:	b7aff0ef          	jal	1121c <__malloc_unlock>
   11ea6:	4b42                	lw	s6,16(sp)
   11ea8:	4bb2                	lw	s7,12(sp)
   11eaa:	4c22                	lw	s8,8(sp)
   11eac:	b701                	j	11dac <_realloc_r+0x136>
   11eae:	4014                	lw	a3,0(s0)
   11eb0:	476d                	li	a4,27
   11eb2:	c114                	sw	a3,0(a0)
   11eb4:	4054                	lw	a3,4(s0)
   11eb6:	c154                	sw	a3,4(a0)
   11eb8:	0cc76963          	bltu	a4,a2,11f8a <_realloc_r+0x314>
   11ebc:	00840713          	addi	a4,s0,8
   11ec0:	00850793          	addi	a5,a0,8
   11ec4:	b549                	j	11d46 <_realloc_r+0xd0>
   11ec6:	ffc52783          	lw	a5,-4(a0)
   11eca:	4c22                	lw	s8,8(sp)
   11ecc:	9bf1                	andi	a5,a5,-4
   11ece:	99be                	add	s3,s3,a5
   11ed0:	bd45                	j	11d80 <_realloc_r+0x10a>
   11ed2:	00cba783          	lw	a5,12(s7)
   11ed6:	008ba683          	lw	a3,8(s7)
   11eda:	ffc98613          	addi	a2,s3,-4
   11ede:	02400593          	li	a1,36
   11ee2:	c6dc                	sw	a5,12(a3)
   11ee4:	c794                	sw	a3,8(a5)
   11ee6:	008b8493          	addi	s1,s7,8
   11eea:	08c5eb63          	bltu	a1,a2,11f80 <_realloc_r+0x30a>
   11eee:	46cd                	li	a3,19
   11ef0:	87a6                	mv	a5,s1
   11ef2:	00c6fe63          	bgeu	a3,a2,11f0e <_realloc_r+0x298>
   11ef6:	4018                	lw	a4,0(s0)
   11ef8:	47ed                	li	a5,27
   11efa:	00eba423          	sw	a4,8(s7)
   11efe:	4058                	lw	a4,4(s0)
   11f00:	00eba623          	sw	a4,12(s7)
   11f04:	0cc7e463          	bltu	a5,a2,11fcc <_realloc_r+0x356>
   11f08:	0421                	addi	s0,s0,8
   11f0a:	010b8793          	addi	a5,s7,16
   11f0e:	4014                	lw	a3,0(s0)
   11f10:	c394                	sw	a3,0(a5)
   11f12:	4054                	lw	a3,4(s0)
   11f14:	c3d4                	sw	a3,4(a5)
   11f16:	4414                	lw	a3,8(s0)
   11f18:	c794                	sw	a3,8(a5)
   11f1a:	89da                	mv	s3,s6
   11f1c:	8ade                	mv	s5,s7
   11f1e:	4b42                	lw	s6,16(sp)
   11f20:	4bb2                	lw	s7,12(sp)
   11f22:	4c22                	lw	s8,8(sp)
   11f24:	8426                	mv	s0,s1
   11f26:	bda9                	j	11d80 <_realloc_r+0x10a>
   11f28:	85a2                	mv	a1,s0
   11f2a:	3e21                	jal	11a42 <memmove>
   11f2c:	b51d                	j	11d52 <_realloc_r+0xdc>
   11f2e:	46dc                	lw	a5,12(a3)
   11f30:	4698                	lw	a4,8(a3)
   11f32:	ffc98613          	addi	a2,s3,-4
   11f36:	02400693          	li	a3,36
   11f3a:	c75c                	sw	a5,12(a4)
   11f3c:	c798                	sw	a4,8(a5)
   11f3e:	008ba703          	lw	a4,8(s7)
   11f42:	00cba783          	lw	a5,12(s7)
   11f46:	008b8493          	addi	s1,s7,8
   11f4a:	c75c                	sw	a5,12(a4)
   11f4c:	c798                	sw	a4,8(a5)
   11f4e:	02c6e963          	bltu	a3,a2,11f80 <_realloc_r+0x30a>
   11f52:	474d                	li	a4,19
   11f54:	87a6                	mv	a5,s1
   11f56:	00c77e63          	bgeu	a4,a2,11f72 <_realloc_r+0x2fc>
   11f5a:	4018                	lw	a4,0(s0)
   11f5c:	47ed                	li	a5,27
   11f5e:	00eba423          	sw	a4,8(s7)
   11f62:	4058                	lw	a4,4(s0)
   11f64:	00eba623          	sw	a4,12(s7)
   11f68:	08c7ed63          	bltu	a5,a2,12002 <_realloc_r+0x38c>
   11f6c:	0421                	addi	s0,s0,8
   11f6e:	010b8793          	addi	a5,s7,16
   11f72:	4018                	lw	a4,0(s0)
   11f74:	c398                	sw	a4,0(a5)
   11f76:	4058                	lw	a4,4(s0)
   11f78:	c3d8                	sw	a4,4(a5)
   11f7a:	4418                	lw	a4,8(s0)
   11f7c:	c798                	sw	a4,8(a5)
   11f7e:	bf71                	j	11f1a <_realloc_r+0x2a4>
   11f80:	85a2                	mv	a1,s0
   11f82:	8526                	mv	a0,s1
   11f84:	abfff0ef          	jal	11a42 <memmove>
   11f88:	bf49                	j	11f1a <_realloc_r+0x2a4>
   11f8a:	4418                	lw	a4,8(s0)
   11f8c:	c518                	sw	a4,8(a0)
   11f8e:	4458                	lw	a4,12(s0)
   11f90:	c558                	sw	a4,12(a0)
   11f92:	04f60f63          	beq	a2,a5,11ff0 <_realloc_r+0x37a>
   11f96:	01040713          	addi	a4,s0,16
   11f9a:	01050793          	addi	a5,a0,16
   11f9e:	b365                	j	11d46 <_realloc_r+0xd0>
   11fa0:	9ad2                	add	s5,s5,s4
   11fa2:	414686b3          	sub	a3,a3,s4
   11fa6:	015c2423          	sw	s5,8(s8)
   11faa:	0016e793          	ori	a5,a3,1
   11fae:	00faa223          	sw	a5,4(s5)
   11fb2:	ffc42783          	lw	a5,-4(s0)
   11fb6:	854a                	mv	a0,s2
   11fb8:	84a2                	mv	s1,s0
   11fba:	8b85                	andi	a5,a5,1
   11fbc:	0147e7b3          	or	a5,a5,s4
   11fc0:	fef42e23          	sw	a5,-4(s0)
   11fc4:	a58ff0ef          	jal	1121c <__malloc_unlock>
   11fc8:	4c22                	lw	s8,8(sp)
   11fca:	b3cd                	j	11dac <_realloc_r+0x136>
   11fcc:	441c                	lw	a5,8(s0)
   11fce:	00fba823          	sw	a5,16(s7)
   11fd2:	445c                	lw	a5,12(s0)
   11fd4:	00fbaa23          	sw	a5,20(s7)
   11fd8:	04b60863          	beq	a2,a1,12028 <_realloc_r+0x3b2>
   11fdc:	0441                	addi	s0,s0,16
   11fde:	018b8793          	addi	a5,s7,24
   11fe2:	b735                	j	11f0e <_realloc_r+0x298>
   11fe4:	854a                	mv	a0,s2
   11fe6:	a36ff0ef          	jal	1121c <__malloc_unlock>
   11fea:	4481                	li	s1,0
   11fec:	4c22                	lw	s8,8(sp)
   11fee:	bb7d                	j	11dac <_realloc_r+0x136>
   11ff0:	4814                	lw	a3,16(s0)
   11ff2:	01840713          	addi	a4,s0,24
   11ff6:	01850793          	addi	a5,a0,24
   11ffa:	c914                	sw	a3,16(a0)
   11ffc:	4854                	lw	a3,20(s0)
   11ffe:	c954                	sw	a3,20(a0)
   12000:	b399                	j	11d46 <_realloc_r+0xd0>
   12002:	4418                	lw	a4,8(s0)
   12004:	02400793          	li	a5,36
   12008:	00eba823          	sw	a4,16(s7)
   1200c:	4458                	lw	a4,12(s0)
   1200e:	00ebaa23          	sw	a4,20(s7)
   12012:	04f60263          	beq	a2,a5,12056 <_realloc_r+0x3e0>
   12016:	0441                	addi	s0,s0,16
   12018:	018b8793          	addi	a5,s7,24
   1201c:	bf99                	j	11f72 <_realloc_r+0x2fc>
   1201e:	85a2                	mv	a1,s0
   12020:	8526                	mv	a0,s1
   12022:	a21ff0ef          	jal	11a42 <memmove>
   12026:	bda9                	j	11e80 <_realloc_r+0x20a>
   12028:	4818                	lw	a4,16(s0)
   1202a:	020b8793          	addi	a5,s7,32
   1202e:	0461                	addi	s0,s0,24
   12030:	00ebac23          	sw	a4,24(s7)
   12034:	ffc42703          	lw	a4,-4(s0)
   12038:	00ebae23          	sw	a4,28(s7)
   1203c:	bdc9                	j	11f0e <_realloc_r+0x298>
   1203e:	441c                	lw	a5,8(s0)
   12040:	00fba823          	sw	a5,16(s7)
   12044:	445c                	lw	a5,12(s0)
   12046:	00fbaa23          	sw	a5,20(s7)
   1204a:	02d60163          	beq	a2,a3,1206c <_realloc_r+0x3f6>
   1204e:	0441                	addi	s0,s0,16
   12050:	018b8793          	addi	a5,s7,24
   12054:	b505                	j	11e74 <_realloc_r+0x1fe>
   12056:	4818                	lw	a4,16(s0)
   12058:	020b8793          	addi	a5,s7,32
   1205c:	0461                	addi	s0,s0,24
   1205e:	00ebac23          	sw	a4,24(s7)
   12062:	ffc42703          	lw	a4,-4(s0)
   12066:	00ebae23          	sw	a4,28(s7)
   1206a:	b721                	j	11f72 <_realloc_r+0x2fc>
   1206c:	4818                	lw	a4,16(s0)
   1206e:	020b8793          	addi	a5,s7,32
   12072:	00ebac23          	sw	a4,24(s7)
   12076:	4858                	lw	a4,20(s0)
   12078:	0461                	addi	s0,s0,24
   1207a:	00ebae23          	sw	a4,28(s7)
   1207e:	bbdd                	j	11e74 <_realloc_r+0x1fe>

00012080 <__smakebuf_r>:
   12080:	00c59783          	lh	a5,12(a1)
   12084:	7159                	addi	sp,sp,-112
   12086:	d4a2                	sw	s0,104(sp)
   12088:	d686                	sw	ra,108(sp)
   1208a:	0027f713          	andi	a4,a5,2
   1208e:	842e                	mv	s0,a1
   12090:	cb19                	beqz	a4,120a6 <__smakebuf_r+0x26>
   12092:	04358793          	addi	a5,a1,67
   12096:	4705                	li	a4,1
   12098:	c19c                	sw	a5,0(a1)
   1209a:	c99c                	sw	a5,16(a1)
   1209c:	c9d8                	sw	a4,20(a1)
   1209e:	50b6                	lw	ra,108(sp)
   120a0:	5426                	lw	s0,104(sp)
   120a2:	6165                	addi	sp,sp,112
   120a4:	8082                	ret
   120a6:	00e59583          	lh	a1,14(a1)
   120aa:	d0ca                	sw	s2,96(sp)
   120ac:	d2a6                	sw	s1,100(sp)
   120ae:	892a                	mv	s2,a0
   120b0:	0405cd63          	bltz	a1,1210a <__smakebuf_r+0x8a>
   120b4:	0030                	addi	a2,sp,8
   120b6:	22a9                	jal	12200 <_fstat_r>
   120b8:	04054763          	bltz	a0,12106 <__smakebuf_r+0x86>
   120bc:	40000593          	li	a1,1024
   120c0:	854a                	mv	a0,s2
   120c2:	44b2                	lw	s1,12(sp)
   120c4:	b8bfe0ef          	jal	10c4e <_malloc_r>
   120c8:	00c41783          	lh	a5,12(s0)
   120cc:	cd3d                	beqz	a0,1214a <__smakebuf_r+0xca>
   120ce:	673d                	lui	a4,0xf
   120d0:	0807e793          	ori	a5,a5,128
   120d4:	40000693          	li	a3,1024
   120d8:	8cf9                	and	s1,s1,a4
   120da:	00f41623          	sh	a5,12(s0)
   120de:	c008                	sw	a0,0(s0)
   120e0:	c808                	sw	a0,16(s0)
   120e2:	c854                	sw	a3,20(s0)
   120e4:	6709                	lui	a4,0x2
   120e6:	08e49663          	bne	s1,a4,12172 <__smakebuf_r+0xf2>
   120ea:	00e41583          	lh	a1,14(s0)
   120ee:	854a                	mv	a0,s2
   120f0:	22a1                	jal	12238 <_isatty_r>
   120f2:	6705                	lui	a4,0x1
   120f4:	00c41783          	lh	a5,12(s0)
   120f8:	80070713          	addi	a4,a4,-2048 # 800 <exit-0xf8b4>
   120fc:	cd15                	beqz	a0,12138 <__smakebuf_r+0xb8>
   120fe:	9bf1                	andi	a5,a5,-4
   12100:	0017e793          	ori	a5,a5,1
   12104:	a815                	j	12138 <__smakebuf_r+0xb8>
   12106:	00c41783          	lh	a5,12(s0)
   1210a:	0807f493          	andi	s1,a5,128
   1210e:	0014b493          	seqz	s1,s1
   12112:	409004b3          	neg	s1,s1
   12116:	3c04f493          	andi	s1,s1,960
   1211a:	04048493          	addi	s1,s1,64
   1211e:	854a                	mv	a0,s2
   12120:	85a6                	mv	a1,s1
   12122:	b2dfe0ef          	jal	10c4e <_malloc_r>
   12126:	00c41783          	lh	a5,12(s0)
   1212a:	c105                	beqz	a0,1214a <__smakebuf_r+0xca>
   1212c:	0807e793          	ori	a5,a5,128
   12130:	c008                	sw	a0,0(s0)
   12132:	c808                	sw	a0,16(s0)
   12134:	c844                	sw	s1,20(s0)
   12136:	4701                	li	a4,0
   12138:	8fd9                	or	a5,a5,a4
   1213a:	50b6                	lw	ra,108(sp)
   1213c:	00f41623          	sh	a5,12(s0)
   12140:	5426                	lw	s0,104(sp)
   12142:	5496                	lw	s1,100(sp)
   12144:	5906                	lw	s2,96(sp)
   12146:	6165                	addi	sp,sp,112
   12148:	8082                	ret
   1214a:	2007f713          	andi	a4,a5,512
   1214e:	ef19                	bnez	a4,1216c <__smakebuf_r+0xec>
   12150:	9bf1                	andi	a5,a5,-4
   12152:	04340713          	addi	a4,s0,67
   12156:	0027e793          	ori	a5,a5,2
   1215a:	4685                	li	a3,1
   1215c:	5496                	lw	s1,100(sp)
   1215e:	5906                	lw	s2,96(sp)
   12160:	00f41623          	sh	a5,12(s0)
   12164:	c018                	sw	a4,0(s0)
   12166:	c818                	sw	a4,16(s0)
   12168:	c854                	sw	a3,20(s0)
   1216a:	bf15                	j	1209e <__smakebuf_r+0x1e>
   1216c:	5496                	lw	s1,100(sp)
   1216e:	5906                	lw	s2,96(sp)
   12170:	b73d                	j	1209e <__smakebuf_r+0x1e>
   12172:	6705                	lui	a4,0x1
   12174:	80070713          	addi	a4,a4,-2048 # 800 <exit-0xf8b4>
   12178:	b7c1                	j	12138 <__smakebuf_r+0xb8>

0001217a <__swhatbuf_r>:
   1217a:	7159                	addi	sp,sp,-112
   1217c:	d4a2                	sw	s0,104(sp)
   1217e:	842e                	mv	s0,a1
   12180:	00e59583          	lh	a1,14(a1)
   12184:	d2a6                	sw	s1,100(sp)
   12186:	d0ca                	sw	s2,96(sp)
   12188:	d686                	sw	ra,108(sp)
   1218a:	84b2                	mv	s1,a2
   1218c:	8936                	mv	s2,a3
   1218e:	0205cb63          	bltz	a1,121c4 <__swhatbuf_r+0x4a>
   12192:	0030                	addi	a2,sp,8
   12194:	20b5                	jal	12200 <_fstat_r>
   12196:	02054763          	bltz	a0,121c4 <__swhatbuf_r+0x4a>
   1219a:	47b2                	lw	a5,12(sp)
   1219c:	66bd                	lui	a3,0xf
   1219e:	7779                	lui	a4,0xffffe
   121a0:	8ff5                	and	a5,a5,a3
   121a2:	97ba                	add	a5,a5,a4
   121a4:	50b6                	lw	ra,108(sp)
   121a6:	5426                	lw	s0,104(sp)
   121a8:	0017b793          	seqz	a5,a5
   121ac:	00f92023          	sw	a5,0(s2)
   121b0:	40000713          	li	a4,1024
   121b4:	c098                	sw	a4,0(s1)
   121b6:	6505                	lui	a0,0x1
   121b8:	5496                	lw	s1,100(sp)
   121ba:	5906                	lw	s2,96(sp)
   121bc:	80050513          	addi	a0,a0,-2048 # 800 <exit-0xf8b4>
   121c0:	6165                	addi	sp,sp,112
   121c2:	8082                	ret
   121c4:	00c45783          	lhu	a5,12(s0)
   121c8:	0807f793          	andi	a5,a5,128
   121cc:	cf91                	beqz	a5,121e8 <__swhatbuf_r+0x6e>
   121ce:	50b6                	lw	ra,108(sp)
   121d0:	5426                	lw	s0,104(sp)
   121d2:	4781                	li	a5,0
   121d4:	00f92023          	sw	a5,0(s2)
   121d8:	04000713          	li	a4,64
   121dc:	c098                	sw	a4,0(s1)
   121de:	5906                	lw	s2,96(sp)
   121e0:	5496                	lw	s1,100(sp)
   121e2:	4501                	li	a0,0
   121e4:	6165                	addi	sp,sp,112
   121e6:	8082                	ret
   121e8:	50b6                	lw	ra,108(sp)
   121ea:	5426                	lw	s0,104(sp)
   121ec:	00f92023          	sw	a5,0(s2)
   121f0:	40000713          	li	a4,1024
   121f4:	c098                	sw	a4,0(s1)
   121f6:	5906                	lw	s2,96(sp)
   121f8:	5496                	lw	s1,100(sp)
   121fa:	4501                	li	a0,0
   121fc:	6165                	addi	sp,sp,112
   121fe:	8082                	ret

00012200 <_fstat_r>:
   12200:	1141                	addi	sp,sp,-16
   12202:	872e                	mv	a4,a1
   12204:	c422                	sw	s0,8(sp)
   12206:	c226                	sw	s1,4(sp)
   12208:	85b2                	mv	a1,a2
   1220a:	842a                	mv	s0,a0
   1220c:	853a                	mv	a0,a4
   1220e:	c606                	sw	ra,12(sp)
   12210:	d401a623          	sw	zero,-692(gp) # 139cc <errno>
   12214:	204d                	jal	122b6 <_fstat>
   12216:	57fd                	li	a5,-1
   12218:	00f50763          	beq	a0,a5,12226 <_fstat_r+0x26>
   1221c:	40b2                	lw	ra,12(sp)
   1221e:	4422                	lw	s0,8(sp)
   12220:	4492                	lw	s1,4(sp)
   12222:	0141                	addi	sp,sp,16
   12224:	8082                	ret
   12226:	d4c1a783          	lw	a5,-692(gp) # 139cc <errno>
   1222a:	dbed                	beqz	a5,1221c <_fstat_r+0x1c>
   1222c:	40b2                	lw	ra,12(sp)
   1222e:	c01c                	sw	a5,0(s0)
   12230:	4422                	lw	s0,8(sp)
   12232:	4492                	lw	s1,4(sp)
   12234:	0141                	addi	sp,sp,16
   12236:	8082                	ret

00012238 <_isatty_r>:
   12238:	1141                	addi	sp,sp,-16
   1223a:	c422                	sw	s0,8(sp)
   1223c:	c226                	sw	s1,4(sp)
   1223e:	842a                	mv	s0,a0
   12240:	852e                	mv	a0,a1
   12242:	c606                	sw	ra,12(sp)
   12244:	d401a623          	sw	zero,-692(gp) # 139cc <errno>
   12248:	205d                	jal	122ee <_isatty>
   1224a:	57fd                	li	a5,-1
   1224c:	00f50763          	beq	a0,a5,1225a <_isatty_r+0x22>
   12250:	40b2                	lw	ra,12(sp)
   12252:	4422                	lw	s0,8(sp)
   12254:	4492                	lw	s1,4(sp)
   12256:	0141                	addi	sp,sp,16
   12258:	8082                	ret
   1225a:	d4c1a783          	lw	a5,-692(gp) # 139cc <errno>
   1225e:	dbed                	beqz	a5,12250 <_isatty_r+0x18>
   12260:	40b2                	lw	ra,12(sp)
   12262:	c01c                	sw	a5,0(s0)
   12264:	4422                	lw	s0,8(sp)
   12266:	4492                	lw	s1,4(sp)
   12268:	0141                	addi	sp,sp,16
   1226a:	8082                	ret

0001226c <_close>:
   1226c:	1141                	addi	sp,sp,-16
   1226e:	c606                	sw	ra,12(sp)
   12270:	c422                	sw	s0,8(sp)
   12272:	03900893          	li	a7,57
   12276:	00000073          	ecall
   1227a:	842a                	mv	s0,a0
   1227c:	00054763          	bltz	a0,1228a <_close+0x1e>
   12280:	40b2                	lw	ra,12(sp)
   12282:	8522                	mv	a0,s0
   12284:	4422                	lw	s0,8(sp)
   12286:	0141                	addi	sp,sp,16
   12288:	8082                	ret
   1228a:	40800433          	neg	s0,s0
   1228e:	22e1                	jal	12456 <__errno>
   12290:	c100                	sw	s0,0(a0)
   12292:	547d                	li	s0,-1
   12294:	b7f5                	j	12280 <_close+0x14>

00012296 <_exit>:
   12296:	05d00893          	li	a7,93
   1229a:	00000073          	ecall
   1229e:	00054363          	bltz	a0,122a4 <_exit+0xe>
   122a2:	a001                	j	122a2 <_exit+0xc>
   122a4:	1141                	addi	sp,sp,-16
   122a6:	c422                	sw	s0,8(sp)
   122a8:	842a                	mv	s0,a0
   122aa:	c606                	sw	ra,12(sp)
   122ac:	40800433          	neg	s0,s0
   122b0:	225d                	jal	12456 <__errno>
   122b2:	c100                	sw	s0,0(a0)
   122b4:	a001                	j	122b4 <_exit+0x1e>

000122b6 <_fstat>:
   122b6:	7175                	addi	sp,sp,-144
   122b8:	c326                	sw	s1,132(sp)
   122ba:	c706                	sw	ra,140(sp)
   122bc:	84ae                	mv	s1,a1
   122be:	c522                	sw	s0,136(sp)
   122c0:	858a                	mv	a1,sp
   122c2:	05000893          	li	a7,80
   122c6:	00000073          	ecall
   122ca:	842a                	mv	s0,a0
   122cc:	00054b63          	bltz	a0,122e2 <_fstat+0x2c>
   122d0:	8526                	mv	a0,s1
   122d2:	858a                	mv	a1,sp
   122d4:	2231                	jal	123e0 <_conv_stat>
   122d6:	40ba                	lw	ra,140(sp)
   122d8:	8522                	mv	a0,s0
   122da:	442a                	lw	s0,136(sp)
   122dc:	449a                	lw	s1,132(sp)
   122de:	6149                	addi	sp,sp,144
   122e0:	8082                	ret
   122e2:	40800433          	neg	s0,s0
   122e6:	2a85                	jal	12456 <__errno>
   122e8:	c100                	sw	s0,0(a0)
   122ea:	547d                	li	s0,-1
   122ec:	b7d5                	j	122d0 <_fstat+0x1a>

000122ee <_isatty>:
   122ee:	7159                	addi	sp,sp,-112
   122f0:	002c                	addi	a1,sp,8
   122f2:	d686                	sw	ra,108(sp)
   122f4:	37c9                	jal	122b6 <_fstat>
   122f6:	57fd                	li	a5,-1
   122f8:	00f50863          	beq	a0,a5,12308 <_isatty+0x1a>
   122fc:	4532                	lw	a0,12(sp)
   122fe:	50b6                	lw	ra,108(sp)
   12300:	8135                	srli	a0,a0,0xd
   12302:	8905                	andi	a0,a0,1
   12304:	6165                	addi	sp,sp,112
   12306:	8082                	ret
   12308:	50b6                	lw	ra,108(sp)
   1230a:	4501                	li	a0,0
   1230c:	6165                	addi	sp,sp,112
   1230e:	8082                	ret

00012310 <_lseek>:
   12310:	1141                	addi	sp,sp,-16
   12312:	c606                	sw	ra,12(sp)
   12314:	c422                	sw	s0,8(sp)
   12316:	03e00893          	li	a7,62
   1231a:	00000073          	ecall
   1231e:	842a                	mv	s0,a0
   12320:	00054763          	bltz	a0,1232e <_lseek+0x1e>
   12324:	40b2                	lw	ra,12(sp)
   12326:	8522                	mv	a0,s0
   12328:	4422                	lw	s0,8(sp)
   1232a:	0141                	addi	sp,sp,16
   1232c:	8082                	ret
   1232e:	40800433          	neg	s0,s0
   12332:	2215                	jal	12456 <__errno>
   12334:	c100                	sw	s0,0(a0)
   12336:	547d                	li	s0,-1
   12338:	b7f5                	j	12324 <_lseek+0x14>

0001233a <_read>:
   1233a:	1141                	addi	sp,sp,-16
   1233c:	c606                	sw	ra,12(sp)
   1233e:	c422                	sw	s0,8(sp)
   12340:	03f00893          	li	a7,63
   12344:	00000073          	ecall
   12348:	842a                	mv	s0,a0
   1234a:	00054763          	bltz	a0,12358 <_read+0x1e>
   1234e:	40b2                	lw	ra,12(sp)
   12350:	8522                	mv	a0,s0
   12352:	4422                	lw	s0,8(sp)
   12354:	0141                	addi	sp,sp,16
   12356:	8082                	ret
   12358:	40800433          	neg	s0,s0
   1235c:	28ed                	jal	12456 <__errno>
   1235e:	c100                	sw	s0,0(a0)
   12360:	547d                	li	s0,-1
   12362:	b7f5                	j	1234e <_read+0x14>

00012364 <_sbrk>:
   12364:	d601a703          	lw	a4,-672(gp) # 139e0 <heap_end.0>
   12368:	1141                	addi	sp,sp,-16
   1236a:	c606                	sw	ra,12(sp)
   1236c:	87aa                	mv	a5,a0
   1236e:	ef01                	bnez	a4,12386 <_sbrk+0x22>
   12370:	0d600893          	li	a7,214
   12374:	4501                	li	a0,0
   12376:	00000073          	ecall
   1237a:	567d                	li	a2,-1
   1237c:	872a                	mv	a4,a0
   1237e:	02c50563          	beq	a0,a2,123a8 <_sbrk+0x44>
   12382:	d6a1a023          	sw	a0,-672(gp) # 139e0 <heap_end.0>
   12386:	00e78533          	add	a0,a5,a4
   1238a:	0d600893          	li	a7,214
   1238e:	00000073          	ecall
   12392:	d601a703          	lw	a4,-672(gp) # 139e0 <heap_end.0>
   12396:	97ba                	add	a5,a5,a4
   12398:	00f51863          	bne	a0,a5,123a8 <_sbrk+0x44>
   1239c:	40b2                	lw	ra,12(sp)
   1239e:	d6a1a023          	sw	a0,-672(gp) # 139e0 <heap_end.0>
   123a2:	853a                	mv	a0,a4
   123a4:	0141                	addi	sp,sp,16
   123a6:	8082                	ret
   123a8:	207d                	jal	12456 <__errno>
   123aa:	40b2                	lw	ra,12(sp)
   123ac:	47b1                	li	a5,12
   123ae:	c11c                	sw	a5,0(a0)
   123b0:	557d                	li	a0,-1
   123b2:	0141                	addi	sp,sp,16
   123b4:	8082                	ret

000123b6 <_write>:
   123b6:	1141                	addi	sp,sp,-16
   123b8:	c606                	sw	ra,12(sp)
   123ba:	c422                	sw	s0,8(sp)
   123bc:	04000893          	li	a7,64
   123c0:	00000073          	ecall
   123c4:	842a                	mv	s0,a0
   123c6:	00054763          	bltz	a0,123d4 <_write+0x1e>
   123ca:	40b2                	lw	ra,12(sp)
   123cc:	8522                	mv	a0,s0
   123ce:	4422                	lw	s0,8(sp)
   123d0:	0141                	addi	sp,sp,16
   123d2:	8082                	ret
   123d4:	40800433          	neg	s0,s0
   123d8:	28bd                	jal	12456 <__errno>
   123da:	c100                	sw	s0,0(a0)
   123dc:	547d                	li	s0,-1
   123de:	b7f5                	j	123ca <_write+0x14>

000123e0 <_conv_stat>:
   123e0:	1141                	addi	sp,sp,-16
   123e2:	419c                	lw	a5,0(a1)
   123e4:	0145a383          	lw	t2,20(a1)
   123e8:	0185a283          	lw	t0,24(a1)
   123ec:	01c5af83          	lw	t6,28(a1)
   123f0:	0205af03          	lw	t5,32(a1)
   123f4:	0305ae83          	lw	t4,48(a1)
   123f8:	0405ae03          	lw	t3,64(a1)
   123fc:	0385a303          	lw	t1,56(a1)
   12400:	0485a803          	lw	a6,72(a1)
   12404:	04c5a883          	lw	a7,76(a1)
   12408:	4db0                	lw	a2,88(a1)
   1240a:	c622                	sw	s0,12(sp)
   1240c:	c426                	sw	s1,8(sp)
   1240e:	4980                	lw	s0,16(a1)
   12410:	4584                	lw	s1,8(a1)
   12412:	4df4                	lw	a3,92(a1)
   12414:	55b8                	lw	a4,104(a1)
   12416:	00f51023          	sh	a5,0(a0)
   1241a:	55fc                	lw	a5,108(a1)
   1241c:	00951123          	sh	s1,2(a0)
   12420:	c140                	sw	s0,4(a0)
   12422:	00751423          	sh	t2,8(a0)
   12426:	00551523          	sh	t0,10(a0)
   1242a:	01f51623          	sh	t6,12(a0)
   1242e:	01e51723          	sh	t5,14(a0)
   12432:	01d52823          	sw	t4,16(a0)
   12436:	05c52623          	sw	t3,76(a0)
   1243a:	04652423          	sw	t1,72(a0)
   1243e:	01052c23          	sw	a6,24(a0)
   12442:	01152e23          	sw	a7,28(a0)
   12446:	d510                	sw	a2,40(a0)
   12448:	d554                	sw	a3,44(a0)
   1244a:	4432                	lw	s0,12(sp)
   1244c:	dd18                	sw	a4,56(a0)
   1244e:	dd5c                	sw	a5,60(a0)
   12450:	44a2                	lw	s1,8(sp)
   12452:	0141                	addi	sp,sp,16
   12454:	8082                	ret

00012456 <__errno>:
   12456:	d3c1a503          	lw	a0,-708(gp) # 139bc <_impure_ptr>
   1245a:	8082                	ret
```

---
## Output Screenshot
![0](https://github.com/user-attachments/assets/292248cb-7399-4759-969c-7eaaef374b78)

![0](https://github.com/user-attachments/assets/53e54dcf-d509-4dba-a956-70cedcf2ef09)

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
## Output Screenshot
![0](https://github.com/user-attachments/assets/89bc18d6-3228-46c3-a6c0-16ab4f5e190d)
![0](https://github.com/user-attachments/assets/371684e6-0060-4887-a46e-1fb87f860a74)

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
## Output Screenshot
![0](https://github.com/user-attachments/assets/1bfe6e9c-a549-4a03-a721-6dc83cb12657)

## Task 8:Compare -O0 vs -O2 Optimization in RISC-V
🛠️ Compiling with Different Optimization Levels
bash# Compile with no optimizations (-O0)
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -O0 -S test.c -o test_O0.s

# Compile with high-level optimizations (-O2)
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -O2 -S test.c -o test_O2.s
🔍 What to Observe in Assembly (test_O0.s vs test_O2.s)

-O0:

Preserves all code exactly as written
Includes full function calls, stack setup, and variable usage
Useful for debugging and step-by-step learning


-O2:

Optimizes the code for performance
Performs several optimizations:

🔸 Dead Code Elimination: removes unused variables or expressions
🔸 Inlining: replaces small function calls with their code body
🔸 Register Allocation: minimizes stack use by using registers efficiently





📄 Open Both Assembly Files Side-by-Side in VS Code
bashcode test_O0.s test_O2.s
Analysis Tips

Count the instructions: -O2 typically produces fewer instructions
Check stack usage: Look for differences in stack pointer manipulation
Function calls: Notice if small functions are inlined in the -O2 version
Register usage: -O2 should show more efficient register allocation
Loop optimizations: Look for loop unrolling or other loop optimizations

Expected Differences
Aspect-O0-O2Code SizeLargerSmaller (usually)ReadabilityHighLowerDebug InfoCompleteOptimized awayPerformanceSlowerFasterCompilation TimeFastSlower

---
## Output Screenshot
![0](https://github.com/user-attachments/assets/d1b1b9bd-1fc4-403d-9a17-adefd6a5c47c)


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
## Output Screenshot
![0](https://github.com/user-attachments/assets/1305f767-fa19-42ee-bc4a-d76644ba8808)

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

**Gpio.c content**
```c
#include <stdint.h>
#include <stdio.h>

// GPIO register address
#define GPIO_BASE_ADDR    0x10012000
#define GPIO_OUTPUT_REG   (GPIO_BASE_ADDR + 0x08)  // Typical output register offset
#define GPIO_DIRECTION    (GPIO_BASE_ADDR + 0x04)  // Direction register
#define GPIO_INPUT_EN     (GPIO_BASE_ADDR + 0x00)  // Input enable

// Method 1: Using volatile pointer (RECOMMENDED)
static inline void gpio_write_volatile(uint32_t addr, uint32_t value) {
    volatile uint32_t *reg = (volatile uint32_t *)addr;
    *reg = value;
}

static inline uint32_t gpio_read_volatile(uint32_t addr) {
    volatile uint32_t *reg = (volatile uint32_t *)addr;
    return *reg;
}

// Method 2: Using inline assembly (MOST ROBUST)
static inline void gpio_write_asm(uint32_t addr, uint32_t value) {
    asm volatile (
        "sw %1, 0(%0)"          // Store word: sw rs2, offset(rs1)
        :                       // No outputs
        : "r"(addr), "r"(value) // Inputs: address in register, value in register  
        : "memory"              // Memory clobber - prevents reordering
    );
}

static inline uint32_t gpio_read_asm(uint32_t addr) {
    uint32_t value;
    asm volatile (
        "lw %0, 0(%1)"          // Load word: lw rd, offset(rs1)
        : "=r"(value)           // Output: value in register
        : "r"(addr)             // Input: address in register
        : "memory"              // Memory clobber
    );
    return value;
}

// Method 3: Memory barrier functions (ALTERNATIVE)
static inline void memory_barrier(void) {
    asm volatile ("fence" ::: "memory");
}

void gpio_toggle_demo(void) {
    printf("=== GPIO Toggle Demo ===\n");
    printf("GPIO Base Address: 0x%08X\n", GPIO_BASE_ADDR);
    
    // Method 1: Using volatile pointers
    printf("\n--- Method 1: Volatile Pointers ---\n");
    
    // Configure GPIO direction (set as output)
    gpio_write_volatile(GPIO_DIRECTION, 0xFFFFFFFF);  // All pins as output
    
    // Read current state
    uint32_t current_state = gpio_read_volatile(GPIO_OUTPUT_REG);
    printf("Current GPIO state: 0x%08X\n", current_state);
    
    // Toggle all bits
    uint32_t new_state = ~current_state;
    gpio_write_volatile(GPIO_OUTPUT_REG, new_state);
    printf("Toggled GPIO state: 0x%08X\n", new_state);
    
    // Method 2: Using inline assembly
    printf("\n--- Method 2: Inline Assembly ---\n");
    
    // Read using assembly
    uint32_t asm_state = gpio_read_asm(GPIO_OUTPUT_REG);
    printf("Assembly read state: 0x%08X\n", asm_state);
    
    // Toggle using assembly
    gpio_write_asm(GPIO_OUTPUT_REG, ~asm_state);
    printf("Assembly toggled to: 0x%08X\n", ~asm_state);
    
    // Method 3: Specific bit manipulation
    printf("\n--- Method 3: Bit Manipulation ---\n");
    
    // Toggle specific bits (e.g., bits 0, 1, 2)
    uint32_t mask = 0x07;  // Bits 2:0
    current_state = gpio_read_volatile(GPIO_OUTPUT_REG);
    new_state = current_state ^ mask;  // XOR to toggle
    gpio_write_volatile(GPIO_OUTPUT_REG, new_state);
    printf("Toggled bits 2:0 from 0x%08X to 0x%08X\n", current_state, new_state);
}

// Demonstration of different toggle patterns
void gpio_pattern_demo(void) {
    printf("\n=== GPIO Pattern Demo ===\n");
    
    uint32_t patterns[] = {
        0x55555555,  // Alternating pattern
        0xAAAAAAAA,  // Inverse alternating
        0xFF00FF00,  // Byte alternating
        0x00000000,  // All off
        0xFFFFFFFF   // All on
    };
    
    int num_patterns = sizeof(patterns) / sizeof(patterns[0]);
    
    for (int i = 0; i < num_patterns; i++) {
        printf("Setting pattern %d: 0x%08X\n", i, patterns[i]);
        gpio_write_volatile(GPIO_OUTPUT_REG, patterns[i]);
        
        // Add a small delay (cycle counting)
        for (volatile int delay = 0; delay < 1000; delay++);
        
        // Verify the write
        uint32_t readback = gpio_read_volatile(GPIO_OUTPUT_REG);
        printf("Readback: 0x%08X %s\n", readback, 
               (readback == patterns[i]) ? "✓" : "✗");
    }
}

int main(void) {
    printf("=== RISC-V Memory-Mapped I/O Demo ===\n");
    printf("Target: GPIO at 0x10012000\n\n");
    
    // Note: In Spike simulation, this address might not be mapped
    // The demo shows the correct techniques even if hardware isn't present
    
    gpio_toggle_demo();
    gpio_pattern_demo();
    
    printf("\n=== Demo Complete ===\n");
    printf("Note: In Spike simulation, GPIO hardware may not respond,\n");
    printf("but the memory operations are performed correctly.\n");
    
    return 0;
}
```
---
## Output Screenshot
![0](https://github.com/user-attachments/assets/19d4a74c-fa7b-47b2-a375-b03136a79940)


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

## Output Screenshot
![0](https://github.com/user-attachments/assets/81e5e111-93dd-4e34-be20-e6734416d334)
![0](https://github.com/user-attachments/assets/6963aad0-48bc-4dcc-a574-1813b94a4b69)

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

# RISC-V Linker Script Template

## Basic Linker Script (`linker.ld`)

```ld
ENTRY(_start)

MEMORY {
   FLASH (rx) : ORIGIN = 0x00000000, LENGTH = 64K
   RAM (rwx)  : ORIGIN = 0x10000000, LENGTH = 8K
}

SECTIONS {
   .text : {
       *(.text*)
       *(.rodata*)
   } > FLASH
   
   __data_load_start = LOADADDR(.data);
   
   .data : {
       __data_start = .;
       *(.data*)
       *(.sdata*)
       __data_end = .;
   } > RAM AT > FLASH
   
   .bss : {
       __bss_start = .;
       *(.bss*)
       *(.sbss*)
       __bss_end = .;
   } > RAM
}

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
## Output Screenshot
![0](https://github.com/user-attachments/assets/0b9978e9-c8c0-44e9-8a7c-839fa21b4e23)
![0](https://github.com/user-attachments/assets/98eb340d-3933-4bc0-8e10-117c80e634df)

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
## Output ScreenShot
![0](https://github.com/user-attachments/assets/83c650a3-a4c9-4312-be33-53ffb9f21469)
![0](https://github.com/user-attachments/assets/1083ab97-19f5-4dbe-9e63-1b1b1528e779)
![0](https://github.com/user-attachments/assets/22fddc4f-40fa-40dc-9eca-8289499dc164)

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


# atomic_test.s
.text
.globl _start
_start:
    # Test atomic add
    li t0, 0x1000        # Load address
    li t1, 5             # Value to add
    amoadd.w t2, t1, (t0) # Atomic add, result in t2
    
    # Exit system call
    li a7, 93            # exit syscall number
    li a0, 0             # exit status
    ecall

# lr_sc_test.s
.text
.globl _start
_start:
    li t0, 0x1000        # Memory address
    li t1, 42            # Initial value
    sw t1, 0(t0)         # Store initial value
    
retry:
    lr.w t2, (t0)        # Load reserved
    addi t2, t2, 1       # Increment
    sc.w t3, t2, (t0)    # Store conditional
    bnez t3, retry       # Retry if failed
    
    # Exit
    li a7, 93
    li a0, 0
    ecall
// atomic_counter.c
#include <pthread.h>
#include <stdio.h>

volatile int counter = 0;

void* increment_atomic(void* arg) {
    for(int i = 0; i < 1000; i++) {
        __sync_fetch_and_add(&counter, 1);  // Uses amoadd.w
    }
    return NULL;
}

int main() {
    pthread_t threads[4];
    
    for(int i = 0; i < 4; i++) {
        pthread_create(&threads[i], NULL, increment_atomic, NULL);
    }
    
    for(int i = 0; i < 4; i++) {
        pthread_join(threads[i], NULL);
    }
    
    printf("Final counter value: %d\n", counter);
    return 0;
}
---
## Output ScreenShot
![0](https://github.com/user-attachments/assets/9e220505-176a-475b-b3a5-41b7a9c53d8b)

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
## Output ScreenShot
![0](https://github.com/user-attachments/assets/ce851c17-a37d-4785-9fa5-83dddd7965ad)
![0](https://github.com/user-attachments/assets/506e6600-2c5a-4b90-ac56-984a11fd8bef)
![0](https://github.com/user-attachments/assets/95fe9dda-8734-45e8-85cb-3836ea9c0a82)
![0](https://github.com/user-attachments/assets/5b37204d-1990-4c92-84c4-b61ccbc00a67)

## Task 16:
```markdown
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

## Task 1: Install RISC-V Toolchain (RV32IMAC)

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

## Task 8: Compare `-O0` vs `-O2` Optimization in RISC-V

### 🛠️ Compiling with Different Optimization Levels

```bash
# Compile with no optimizations (-O0)
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -O0 -S test.c -o test_O0.s

# Compile with high-level optimizations (-O2)
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -O2 -S test.c -o test_O2.s
```

### 🔍 What to Observe in Assembly (`test_O0.s` vs `test_O2.s`)

* **`-O0`:**
   * Preserves all code exactly as written
   * Includes full function calls, stack setup, and variable usage
   * Useful for debugging and step-by-step learning

* **`-O2`:**
   * Optimizes the code for performance
   * Performs several optimizations:
      * 🔸 *Dead Code Elimination*: removes unused variables or expressions
      * 🔸 *Inlining*: replaces small function calls with their code body
      * 🔸 *Register Allocation*: minimizes stack use by using registers efficiently

### 📄 Open Both Assembly Files Side-by-Side in VS Code

```bash
code test_O0.s test_O2.s
```

### Analysis Tips
1. **Count the instructions**: `-O2` typically produces fewer instructions
2. **Check stack usage**: Look for differences in stack pointer manipulation
3. **Function calls**: Notice if small functions are inlined in the `-O2` version
4. **Register usage**: `-O2` should show more efficient register allocation
5. **Loop optimizations**: Look for loop unrolling or other loop optimizations

### Expected Differences

| Aspect | `-O0` | `-O2` |
|--------|-------|-------|
| Code Size | Larger | Smaller (usually) |
| Readability | High | Lower |
| Debug Info | Complete | Optimized away |
| Performance | Slower | Faster |
| Compilation Time | Fast | Slower |

---

## RISC-V Linker Script Template

### Basic Linker Script (`linker.ld`)

```ld
ENTRY(_start)

MEMORY {
    FLASH (rx) : ORIGIN = 0x00000000, LENGTH = 64K
    RAM (rwx)  : ORIGIN = 0x10000000, LENGTH = 8K
}

SECTIONS {
    .text : {
        *(.text*)
        *(.rodata*)
    } > FLASH
    
    __data_load_start = LOADADDR(.data);
    
    .data : {
        __data_start = .;
        *(.data*)
        *(.sdata*)
        __data_end = .;
    } > RAM AT > FLASH
    
    .bss : {
        __bss_start = .;
        *(.bss*)
        *(.sbss*)
        __bss_end = .;
    } > RAM
}
```

### Linker Script Explanation

#### Memory Layout
- **FLASH**: Code and constants stored in non-volatile memory
  - Origin: `0x00000000` 
  - Size: `64K` (65,536 bytes)
  - Permissions: `rx` (read + execute)

- **RAM**: Variables and stack in volatile memory  
  - Origin: `0x10000000`
  - Size: `8K` (8,192 bytes)
  - Permissions: `rwx` (read + write + execute)

#### Section Mapping

##### `.text` Section
```ld
.text : {
    *(.text*)     // All code sections
    *(.rodata*)   // All read-only data (constants, strings)
} > FLASH
```
- Contains executable code and constants
- Stored in FLASH memory
- Automatically loaded at startup

##### `.data` Section  
```ld
__data_load_start = LOADADDR(.data);

.data : {
    __data_start = .;
    *(.data*)     // Initialized global variables
    *(.sdata*)    // Small initialized data
    __data_end = .;
} > RAM AT > FLASH
```
- Contains initialized global/static variables
- **Stored in FLASH** but **copied to RAM** at startup
- Requires startup code to copy from FLASH to RAM

##### `.bss` Section
```ld
.bss : {
    __bss_start = .;
    *(.bss*)      // Uninitialized global variables  
    *(.sbss*)     // Small uninitialized data
    __bss_end = .;
} > RAM
```
- Contains uninitialized global/static variables
- Located in RAM only
- Must be zero-initialized by startup code

### Usage with GCC

#### Compile and Link
```bash
# Compile source files
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -c main.c -o main.o
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -c startup.s -o startup.o

# Link with custom linker script
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -nostartfiles \
    -T linker.ld main.o startup.o -o program.elf
```

#### Required Startup Code Symbols
Your startup assembly must handle data copying:

```assembly
# startup.s example
.globl _start
_start:
    # Copy .data section from FLASH to RAM
    la t0, __data_load_start  # Source address in FLASH
    la t1, __data_start       # Destination address in RAM  
    la t2, __data_end         # End of data section
    
copy_data:
    beq t1, t2, zero_bss      # If start == end, skip
    lw t3, 0(t0)              # Load from FLASH
    sw t3, 0(t1)              # Store to RAM
    addi t0, t0, 4            # Increment source
    addi t1, t1, 4            # Increment destination
    j copy_data
    
zero_bss:
    # Zero .bss section
    la t0, __bss_start
    la t1, __bss_end
    
clear_bss:
    beq t0, t1, main_call
    sw zero, 0(t0)
    addi t0, t0, 4
    j clear_bss
    
main_call:
    call main                 # Jump to main function
```

### Memory Map Verification

#### Check Section Addresses
```bash
# View section information
riscv32-unknown-elf-objdump -h program.elf

# Check symbol addresses  
riscv32-unknown-elf-nm program.elf | grep -E "__(data|bss)_"
```

#### Expected Output
```
Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 .text         00001000  00000000  00000000  00001000  2**2
  1 .data         00000100  10000000  00001000  00002000  2**2  
  2 .bss          00000200  10000100  10000100  00000000  2**2
```

### Customization Notes

#### Adjust Memory Sizes
```ld
MEMORY {
    FLASH (rx) : ORIGIN = 0x00000000, LENGTH = 128K  // Increase FLASH
    RAM (rwx)  : ORIGIN = 0x10000000, LENGTH = 32K   // Increase RAM  
}
```

#### Add Stack and Heap
```ld
SECTIONS {
    /* ... existing sections ... */
    
    .heap : {
        __heap_start = .;
        . += 0x1000;          // 4KB heap
        __heap_end = .;
    } > RAM
    
    .stack : {
        . += 0x800;           // 2KB stack  
        __stack_top = .;
    } > RAM
}
```

#### RISC-V Specific Sections
```ld
.text : {
    *(.text.init)             // Initialization code first
    *(.text*)
    *(.rodata*)
} > FLASH
```

---

## Using Newlib printf Without an OS

### Question
"How do I retarget `_write` so that printf sends bytes to my memory-mapped UART?"

### Answer Outline

#### Step 1: Implement `_write` Function
Create a custom `_write()` function that redirects output to your UART:

```c
// syscalls.c or retarget.c
#include <sys/stat.h>
#include <sys/types.h>
#include <errno.h>

// Define your UART base address
#define UART_BASE_ADDR 0x10000000
#define UART_TX_REG    (*(volatile uint32_t*)(UART_BASE_ADDR + 0x00))
#define UART_STATUS    (*(volatile uint32_t*)(UART_BASE_ADDR + 0x04))
#define UART_TX_READY  (1 << 5)  // TX ready bit

// Implement _write system call
int _write(int fd, char* buf, int len) {
    int i;
    
    // Only handle stdout and stderr
    if (fd == 1 || fd == 2) {
        for (i = 0; i < len; i++) {
            // Wait until UART is ready to transmit
            while (!(UART_STATUS & UART_TX_READY)) {
                // Busy wait for TX ready
            }
            
            // Send byte to UART
            UART_TX_REG = buf[i];
            
            // Handle newline conversion (optional)
            if (buf[i] == '\n') {
                while (!(UART_STATUS & UART_TX_READY)) {
                    // Wait for TX ready
                }
                UART_TX_REG = '\r';  // Send carriage return
            }
        }
        return len;  // Return number of bytes written
    }
    
    // For other file descriptors, return error
    errno = EBADF;
    return -1;
}
```

#### Step 2: Implement Other Required System Calls

```c
// Additional system calls required by newlib
int _close(int fd) {
    return -1;
}

int _fstat(int fd, struct stat* st) {
    st->st_mode = S_IFCHR;
    return 0;
}

int _isatty(int fd) {
    return 1;  // Assume all file descriptors are TTY
}

off_t _lseek(int fd, off_t ptr, int dir) {
    return 0;
}

int _read(int fd, char* ptr, int len) {
    return 0;  // No input implementation
}

// Heap management (if using malloc)
extern char _end;  // Defined by linker
static char* heap_ptr = &_end;

void* _sbrk(int incr) {
    char* prev_heap = heap_ptr;
    heap_ptr += incr;
    return prev_heap;
}
```

#### Step 3: Compile and Link with Custom System Calls

```bash
# Compile your main program and syscalls
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -c main.c -o main.o
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -c syscalls.c -o syscalls.o

# Link with -nostartfiles and include your syscalls
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -nostartfiles -T linker.ld \
    main.o syscalls.o -o program.elf
```

#### Step 4: Alternative Method - Using `-specs=nosys.specs`

```bash
# Use nosys specs to avoid undefined syscalls
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -specs=nosys.specs \
    main.c syscalls.c -T linker.ld -o program.elf
```

#### Step 5: Test Your printf Implementation

```c
// main.c
#include <stdio.h>

int main() {
    printf("Hello, RISC-V World!\n");
    printf("Counter: %d\n", 42);
    printf("Hex value: 0x%08x\n", 0xDEADBEEF);
    
    while(1) {
        // Main loop
    }
    
    return 0;
}
```

### Key Points

#### Memory-Mapped UART Considerations
- **UART Base Address**: Update `UART_BASE_ADDR` to match your hardware
- **Register Offsets**: Adjust `UART_TX_REG` and `UART_STATUS` offsets
- **Status Bits**: Modify `UART_TX_READY` bit mask for your UART controller
- **Polling vs Interrupts**: This implementation uses polling; consider interrupt-driven I/O for better performance

#### Linking Options
- **`-nostartfiles`**: Skip standard system startup files
- **`-specs=nosys.specs`**: Use "no system" specifications
- **Custom Linker Script**: Ensure proper memory layout with `linker.ld`

#### Debug Tips
1. **Verify UART Configuration**: Ensure UART is properly initialized before calling printf
2. **Check Memory Map**: Confirm UART addresses match your SoC documentation  
3. **Test Simple Output**: Start with single character output before using printf
4. **Linker Symbols**: Make sure `_end` symbol is defined in your linker script

#### Common Issues and Solutions

| Issue | Solution |
|-------|----------|
| `undefined reference to '_write'` | Include syscalls.c in compilation |
| No output on UART | Verify UART initialization and base address |
| Garbled output | Check baud rate and UART configuration |
| Linker errors | Use appropriate `-specs` or `-nostartfiles` |
```

---
## Output ScreenShot
![0](https://github.com/user-attachments/assets/5a357617-4b07-4b1b-93f3-8b774c809be2)


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
## Output ScreenShot
![0](https://github.com/user-attachments/assets/42032f17-f292-4162-b714-b52f5a414c80)

![0](https://github.com/user-attachments/assets/d25e5004-f07e-4774-a2bd-45382721ea19)

**endian_test.c ** 
```c
Conversation opened. 1 read message. 

Skip to content
Using Vellore Institute of Technology Mail with screen readers

2 of 9,497
(no subject)
Inbox

Yashawini Venkatasubramaniam 22BEC1253 <yashawini.v2022@vitstudent.ac.in>
Attachments
11:04 PM (12 minutes ago)
to me


 5 Attachments
  •  Scanned by Gmail
// endian_test.c - RISC-V Endianness Verification
#include <stdio.h>
#include <stdint.h>

// Union to overlay uint32_t with byte array
union endian_test {
    uint32_t word;      // 32-bit word
    uint8_t bytes[4];   // 4 individual bytes
};

// Function to print binary representation
void print_binary(uint32_t value) {
    printf("Binary: ");
    for (int i = 31; i >= 0; i--) {
        printf("%d", (value >> i) & 1);
        if (i % 8 == 0) printf(" ");
    }
    printf("\n");
}

// Function to determine endianness
const char* get_endianness() {
    union endian_test test;
    test.word = 0x01020304;
    
    if (test.bytes[0] == 0x04) {
        return "Little-endian";
    } else if (test.bytes[0] == 0x01) {
        return "Big-endian";
    } else {
        return "Unknown";
    }
}

int main() {
    printf("=== RISC-V RV32IMAC Endianness Test ===\n\n");
    
    // Create union instance
    union endian_test test;
    
    // Store the test value
    test.word = 0x01020304;
    
    printf("Test value: 0x%08X\n", test.word);
    print_binary(test.word);
    printf("\n");
    
    // Print memory layout
    printf("Memory layout (address -> byte):\n");
    printf("Address  | Byte Value | Hex\n");
    printf("---------|------------|----\n");
    
    for (int i = 0; i < 4; i++) {
        printf("bytes[%d] | %3d        | 0x%02X\n", 
               i, test.bytes[i], test.bytes[i]);
    }
    
    printf("\nByte ordering analysis:\n");
    printf("If little-endian: bytes[0] should be 0x04 (LSB first)\n");
    printf("If big-endian:    bytes[0] should be 0x01 (MSB first)\n");
    printf("\nActual bytes[0] = 0x%02X\n", test.bytes[0]);
    
    // Determine and print endianness
    printf("\nResult: This system is %s\n", get_endianness());
    
    // Additional verification with different values
    printf("\n=== Additional Tests ===\n");
    
    uint32_t test_values[] = {0x12345678, 0xDEADBEEF, 0xCAFEBABE};
    int num_tests = sizeof(test_values) / sizeof(test_values[0]);
    
    for (int i = 0; i < num_tests; i++) {
        test.word = test_values[i];
        printf("\nValue: 0x%08X\n", test.word);
        printf("Bytes: [0x%02X, 0x%02X, 0x%02X, 0x%02X]\n",
               test.bytes[0], test.bytes[1], test.bytes[2], test.bytes[3]);
    }
    
    // Show struct packing example
    printf("\n=== Struct Packing Example ===\n");
    
    struct packed_struct {
        uint8_t  byte1;
        uint16_t word1;
        uint8_t  byte2;
        uint32_t word2;
    } __attribute__((packed));
    
    struct normal_struct {
        uint8_t  byte1;
        uint16_t word1;
        uint8_t  byte2;
        uint32_t word2;
    };
    
    printf("Normal struct size: %zu bytes\n", sizeof(struct normal_struct));
    printf("Packed struct size: %zu bytes\n", sizeof(struct packed_struct));
    
    return 0;
}
endian_test.c
Displaying endian_test.c.

```
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

