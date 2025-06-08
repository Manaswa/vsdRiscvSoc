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

---

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

---

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


# <ins> **2. Compile “Hello, RISC-V”** </ins>

### ✅ Minimal C Program (`hello.c`)

```c
int main() {
    return 0;
}
```

*(No `printf` since there's no standard output in bare-metal unless you implement your own I/O)*

If you're targeting a **bare-metal** RV32IMC system (no OS), this is typical.

---

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

---

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


---

### ✅ 1. Generate the `.s` File (Assembly Output)

From your C file:

```bash
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -S hello.c -o hello.s
```

This will generate `hello.s`, the **assembly version** of your C code.

---

## 📄 2. Example: Assembly Output for a Minimal `main`

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

---

## 🧠 3. Prologue and Epilogue Explained

| Part         | Assembly                                      | Purpose                                                               |
| ------------ | --------------------------------------------- | --------------------------------------------------------------------- |
| **Prologue** | `addi sp, sp, -16`<br>`sw ra, 12(sp)`         | Allocate 16 bytes on stack and save the return address (`ra`)         |
| **Body**     | `li a0, 0`                                    | Load return value `0` into `a0` (RISC-V ABI: `a0` is return register) |
| **Epilogue** | `lw ra, 12(sp)`<br>`addi sp, sp, 16`<br>`ret` | Restore return address, clean up stack, and return to caller          |

> This is standard function call convention on RISC-V (RV32I-based ABI).

---

## 🔎 Optional: Want Even Cleaner Assembly?

Disable optimizations:

```bash
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -S -O0 hello.c -o hello.s
```

With optimizations (`-O2`), the compiler may **skip stack setup** entirely for such a trivial function.

---

Let me know if you’d like to explore:

* How function arguments are passed in RV32
* How recursion works in assembly
* Disassembly of more complex programs (e.g., loops, structs, etc.)





