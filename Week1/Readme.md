#  **1. <ins>Install & Sanity-Check the Toolchain</ins>** 

 ### ✅ 1. **Unpack the `.tar.gz` File**

Open a terminal and run:

```bash
tar -xvzf riscv-toolchain-rv32imac-x86_64-ubuntu.tar.gz
```

This will extract it into a directory (usually named something like `riscv-toolchain` or similar) in the current location.

> Tip: You can extract it to `/opt` or `~/tools` for a cleaner structure, e.g.:

```bash
sudo mkdir -p /opt/riscv
sudo tar -xvzf riscv-toolchain-rv32imac-x86_64-ubuntu.tar.gz -C /opt/riscv
```



 ### ✅ 2. **Add Toolchain to `PATH`**

Assuming the extracted folder contains a `bin` directory with `riscv32-unknown-elf-gcc`, add it to your PATH.

Run this (adjust the path as needed):

```bash
export PATH=/opt/riscv/riscv-toolchain-rv32imac-x86_64-ubuntu/bin:$PATH
```

To make it **permanent**, add that line to your shell config:

* For Bash:

  ```bash
  echo 'export PATH=/opt/riscv/riscv-toolchain-rv32imac-x86_64-ubuntu/bin:$PATH' >> ~/.bashrc
  source ~/.bashrc
  ```

* For Zsh:

  ```bash
  echo 'export PATH=/opt/riscv/riscv-toolchain-rv32imac-x86_64-ubuntu/bin:$PATH' >> ~/.zshrc
  source ~/.zshrc
  ```



### ✅ 3. **Verify Installation**

Run these commands to check if the tools are working:

```bash
riscv32-unknown-elf-gcc --version
riscv32-unknown-elf-objdump --version
riscv32-unknown-elf-gdb --version
```

### **✅Output and Code**

![Screenshot from 2025-06-03 19-22-29](https://github.com/user-attachments/assets/1e5980df-7472-437c-8d68-ecf93f97e574)

![Screenshot from 2025-06-03 19-16-33](https://github.com/user-attachments/assets/e16f291a-8331-44ea-847b-9c845fcb66ea)
![Screenshot from 2025-06-03 19-16-02](https://github.com/user-attachments/assets/1a642c09-5312-4ff1-9afd-982c4dd333ea)
![Screenshot from 2025-06-03 19-15-03](https://github.com/user-attachments/assets/b489dc2e-03be-4959-941c-0a123dffc35c)

---


#  **2. <ins> Compile “Hello, RISC-V”</ins>** 

### ✅ Minimal C Program (`hello.c`)

```c
int main() {
    return 0;
}
```

*(No `printf` since there's no standard output in bare-metal unless you implement your own I/O)*

If you're targeting a **bare-metal** RV32IMC system (no OS), this is typical.



### ✅ GCC Cross-Compilation Command

Assuming you're using `riscv32-unknown-elf-gcc` for **RV32IMC**:

```bash
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -nostdlib -o hello.elf hello.c
```

### 🔍 Breakdown of flags:

| Flag             | Meaning                                                    |
| ---------------- | ---------------------------------------------------------- |
| `-march=rv32imc` | Target architecture: 32-bit RISC-V with IMC extensions     |
| `-mabi=ilp32`    | Integer/pointer ABI (standard for RV32)                    |
| `-nostdlib`      | No standard library (for bare-metal, no `libc`, no `crt0`) |
| `-o hello.elf`   | Output file                                                |



### ✅ Optional: Inspect the ELF

Use `objdump` to check the binary:

```bash
riscv32-unknown-elf-objdump -d hello.elf
```

You can also check the ELF header:

```bash
riscv32-unknown-elf-readelf -h hello.elf
```

---
### **✅Output and Code**
![Screenshot from 2025-06-03 19-45-51](https://github.com/user-attachments/assets/b3fbbb23-353c-494a-9922-3a5f60ae1137)
![Screenshot from 2025-06-03 19-51-57](https://github.com/user-attachments/assets/359ded54-8e44-4a96-8ed6-ad466cbf7fff)
![Screenshot from 2025-06-03 21-44-15](https://github.com/user-attachments/assets/3005d738-d158-43a5-8f8e-324d2d8146e9)

---

#  **3. <ins>From C to Assembly</ins>** 


### ✅ 1. Generate the `.s` File (Assembly Output)

From your C file:

```bash
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -S hello.c -o hello.s
```

This will generate `hello.s`, the **assembly version** of your C code.


###  2. Example: Assembly Output for a Minimal `main`

### Input C code (`hello.c`):

```c
int main() {
    return 0;
}
```

### Output Assembly (`hello.s`), typically looks like:

```asm
    .file   "hello.c"
    .text
    .align  2
    .globl  main
    .type   main, @function
main:
    addi    sp, sp, -16     # Prologue: allocate stack
    sw      ra, 12(sp)      # Save return address
    li      a0, 0           # Return 0
    lw      ra, 12(sp)      # Restore return address
    addi    sp, sp, 16      # Epilogue: deallocate stack
    ret                     # Return to caller
```



### 3. Prologue and Epilogue Explained

| Part         | Assembly                                      | Purpose                                                               |
| ------------ | --------------------------------------------- | --------------------------------------------------------------------- |
| **Prologue** | `addi sp, sp, -16`<br>`sw ra, 12(sp)`         | Allocate 16 bytes on stack and save the return address (`ra`)         |
| **Body**     | `li a0, 0`                                    | Load return value `0` into `a0` (RISC-V ABI: `a0` is return register) |
| **Epilogue** | `lw ra, 12(sp)`<br>`addi sp, sp, 16`<br>`ret` | Restore return address, clean up stack, and return to caller          |

> This is standard function call convention on RISC-V (RV32I-based ABI).

---
### **✅Output and Code**

![Screenshot from 2025-06-03 21-49-07](https://github.com/user-attachments/assets/cfc7fbd8-a6e6-4a3e-b032-5f7a3649a209)
![Screenshot from 2025-06-03 21-52-20](https://github.com/user-attachments/assets/01ebe261-9a7f-4ee5-b8d5-eed2f6148bc1)

---

#  **4. <ins>Hex Dump & Disassembly</ins>** 



### ✅ 1. Convert ELF to Raw Hex

### Option A: Use `objcopy` to get raw binary

```bash
riscv32-unknown-elf-objcopy -O binary hello.elf hello.bin
```

### Option B: Get Intel HEX (if needed for flashing tools)

```bash
riscv32-unknown-elf-objcopy -O ihex hello.elf hello.hex
```

You can view the hex with:

```bash
hexdump -C hello.bin
```

Or:

```bash
cat hello.hex
```



### ✅ 2. Disassemble with `objdump`

Use:

```bash
riscv32-unknown-elf-objdump -d hello.elf
```

This shows **disassembled machine code**.

You can add symbols with:

```bash
riscv32-unknown-elf-objdump -d -S hello.elf
```

That shows interleaved **C code and assembly**, if debug info is included (`-g` during compile).


### ✅ 3. Understand `objdump` Columns

Example output:

```asm
00010074 <main>:
   10074:   1141        addi    sp,sp,-16
   10076:   c606        sw      ra,12(sp)
   10078:   4501        li      a0,0
   1007a:   40b2        lw      ra,12(sp)
   1007c:   0141        addi    sp,sp,16
   1007e:   8082        ret
```

| Column           | Meaning                                      |
| ---------------- | -------------------------------------------- |
| `10074:`         | Address in memory (offset in ELF section)    |
| `1141`           | Machine code (hex encoding of instruction)   |
| `addi sp,sp,-16` | Assembly instruction (disassembled mnemonic) |

So for:

```
10074:   1141        addi    sp,sp,-16
```

* `10074` = address
* `1141` = binary opcodes (hex)
* `addi sp,sp,-16` = instruction

---
### **✅Output and Code**

![Screenshot from 2025-06-03 23-31-11](https://github.com/user-attachments/assets/582918af-a562-463a-9d15-fb4e1c4360f0)
![Screenshot from 2025-06-03 23-33-14](https://github.com/user-attachments/assets/031dd86f-1f77-486f-9bfb-ae2d51a9b6bc)

---


#  **5. <ins>ABI & Register Cheat-Sheet</ins>** 


### ✅ RV32I Register Table

| xN  | ABI Name     | Role (Calling Convention)                     |
| --- | ------------ | --------------------------------------------- |
| x0  | `zero`       | Constant zero                                 |
| x1  | `ra`         | Return address (used by `call` and `ret`)     |
| x2  | `sp`         | Stack pointer                                 |
| x3  | `gp`         | Global pointer                                |
| x4  | `tp`         | Thread pointer                                |
| x5  | `t0`         | Temporary register (caller-saved)             |
| x6  | `t1`         | Temporary register (caller-saved)             |
| x7  | `t2`         | Temporary register (caller-saved)             |
| x8  | `s0` or `fp` | Saved register / frame pointer (callee-saved) |
| x9  | `s1`         | Saved register (callee-saved)                 |
| x10 | `a0`         | Argument 0 / return value 0                   |
| x11 | `a1`         | Argument 1 / return value 1                   |
| x12 | `a2`         | Argument 2                                    |
| x13 | `a3`         | Argument 3                                    |
| x14 | `a4`         | Argument 4                                    |
| x15 | `a5`         | Argument 5                                    |
| x16 | `a6`         | Argument 6                                    |
| x17 | `a7`         | Argument 7                                    |
| x18 | `s2`         | Saved register (callee-saved)                 |
| x19 | `s3`         | Saved register (callee-saved)                 |
| x20 | `s4`         | Saved register (callee-saved)                 |
| x21 | `s5`         | Saved register (callee-saved)                 |
| x22 | `s6`         | Saved register (callee-saved)                 |
| x23 | `s7`         | Saved register (callee-saved)                 |
| x24 | `s8`         | Saved register (callee-saved)                 |
| x25 | `s9`         | Saved register (callee-saved)                 |
| x26 | `s10`        | Saved register (callee-saved)                 |
| x27 | `s11`        | Saved register (callee-saved)                 |
| x28 | `t3`         | Temporary register (caller-saved)             |
| x29 | `t4`         | Temporary register (caller-saved)             |
| x30 | `t5`         | Temporary register (caller-saved)             |
| x31 | `t6`         | Temporary register (caller-saved)             |

---

### 📌 Summary of Roles

* **zero (x0):** Always 0 (hardwired)
* **ra (x1):** Return address from function calls
* **sp (x2):** Stack pointer
* **a0–a7 (x10–x17):** Argument and return value registers
* **t0–t6 (x5–x7, x28–x31):** Temporaries (not preserved by callee)
* **s0–s11 (x8–x9, x18–x27):** Saved registers (callee must preserve)
* **gp, tp (x3, x4):** Used internally (global/thread pointer)

---

#  **6. <ins>Stepping with GDB</ins>** 

### ✅ Step-by-Step GDB Walkthrough

Assuming you have an ELF file called `hello.elf`:

### 🔹 1. Start GDB

```bash
riscv32-unknown-elf-gdb hello.elf
```

You’ll enter the GDB prompt:

```
(gdb)
```

### 🔹 2. Set a Breakpoint at `main`

```gdb
(gdb) break main
```

You should see:

```
Breakpoint 1 at 0x00010074: file hello.c, line 1.
```

### 🔹 3. Start Running the Program

**If running in a simulator like QEMU:**

You need to start QEMU with GDB support and connect it — ask me if you want that.

**If using `gdb` without execution support** (e.g., no simulator), you can still **step through** the ELF statically:

```gdb
(gdb) layout asm      # (optional) shows disassembly layout in terminal UI
(gdb) start
```

### 🔹 4. Step Through Instructions

```gdb
(gdb) stepi      # Step one instruction
(gdb) nexti      # Step over function calls
```

or

```gdb
(gdb) step       # Step into C function
(gdb) next       # Step over C function
```


### 🔹 5. Inspect Registers

```gdb
(gdb) info registers
```

Output looks like:

```
ra             0x00010078
sp             0x80001000
a0             0x0
...
```

You can also inspect individual registers:

```gdb
(gdb) print $a0
(gdb) print $sp
```

###  Example GDB Session

```bash
riscv32-unknown-elf-gdb hello.elf
```

Then:


```gdb
(gdb) break main
(gdb) run
(gdb) stepi
(gdb) info registers
```
---
### **✅Output and Code**
![Screenshot from 2025-06-04 00-30-28](https://github.com/user-attachments/assets/a94031c4-ebeb-4972-bb44-65c8eb3c7d4f)
![Screenshot from 2025-06-04 00-30-51](https://github.com/user-attachments/assets/9d9390e5-73e1-4d96-983e-064737c70729)
![Screenshot from 2025-06-04 13-57-50](https://github.com/user-attachments/assets/fd773271-7eac-4763-8f71-7af81d24db4b)
![Screenshot from 2025-06-04 13-58-03](https://github.com/user-attachments/assets/43ffbfe6-bab8-4ea9-a027-106ab158e627)
![Screenshot from 2025-06-04 13-58-24](https://github.com/user-attachments/assets/6e978586-3e58-46f4-9601-5b3dde26c220)
![Screenshot from 2025-06-04 14-00-10](https://github.com/user-attachments/assets/bc364cdd-0862-4a1f-88f8-04b05b738dd9)
![Screenshot from 2025-06-04 14-00-47](https://github.com/user-attachments/assets/1865c92e-0bab-475b-a880-aede239ebaf4)
![Screenshot from 2025-06-04 14-09-37](https://github.com/user-attachments/assets/6f191447-5f5a-4387-a243-9cc8ee332417)

---

#  **7. <ins>Running Under an Emulator</ins>** 

### ✅ OPTION 1: Using QEMU (Recommended for UART console)

Assuming:

* You compiled your ELF with a `putchar()`/`printf()` that writes to `UART` mapped at `0x10000000` (e.g., SiFive UART)
* You have a working `main()` and `-T linker.ld`

### 🛠 QEMU Command

```bash
qemu-system-riscv32 \
  -nographic \
  -machine sifive_e \
  -kernel your_baremetal.elf
```

### ✅ UART Output

* QEMU will automatically emulate the SiFive UART at `0x10000000`
* Any `putchar()` or `printf()` writing to that address will print **directly to your terminal**
* No need for `-S -gdb` unless you're debugging



### ✅ OPTION 2: Using Spike (requires proxy kernel or test device emulation)

Spike is **not ideal for UART output from raw bare-metal**, unless:

### A. You are using a **proxy kernel (PK)**:

```bash
spike pk your_program.elf
```

…but that's **not bare-metal** anymore — that’s "newlib + pk" environment.

### B. You're using **Spike with MMIO traps** (advanced):

You can write a Spike-compatible MMIO trap model to print when writing to `0x10000000`, but that involves:

* Custom Spike build with `--enable-commitlog`
* Writing traps in C++

This is **non-trivial** and not ideal unless you're developing on real hardware.



### 🔧 Example: UART Write in Bare-Metal C

If you're using SiFive's UART at `0x10000000`:

```c
#define UART_TX  (*(volatile unsigned int *)0x10000000)

void putchar(char c) {
    UART_TX = c;
}

int main() {
    const char *msg = "Hello from RISC-V UART!\n";
    while (*msg) putchar(*msg++);
    while (1); // Loop forever
}
```
---
### **✅Output and Code**


![Screenshot from 2025-06-04 16-48-49](https://github.com/user-attachments/assets/1bcc65cc-5e75-442d-932f-3300f0389893)
![Screenshot from 2025-06-04 17-41-26](https://github.com/user-attachments/assets/5e725322-a877-4060-ac1e-cc150aa2c7e4)
![Screenshot from 2025-06-04 17-42-57](https://github.com/user-attachments/assets/815e7aab-e39e-4ec1-9305-c8c083cec0c2)
![Screenshot from 2025-06-04 17-43-17](https://github.com/user-attachments/assets/23563a26-d3b0-4197-96dd-f277426db65e)

---






