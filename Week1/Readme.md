# <ins> **1. Install & Sanity-Check the Toolchain** </ins>

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

