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



### Example: UART Write in Bare-Metal C

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
#  **8. <ins>Exploring GCC Optimisation</ins>**



###  Step 1: Create a Minimal C File

```c
// hello_opt.c
int square(int x) {
    int y = x * x;
    return y;
}

int main() {
    int result = square(5);
    return result;
}
```



###  Step 2: Compile with -O0 and -O2

```bash
riscv32-unknown-elf-gcc -S -O0 -o hello_O0.s hello_opt.c
riscv32-unknown-elf-gcc -S -O2 -o hello_O2.s hello_opt.c
```

* `-S`: Generate assembly code (`.s` file)
* `-O0`: No optimization (default)
* `-O2`: High optimization (inlines, removes dead code, etc.)



###  Step 3: Compare the Outputs

You can run:

```bash
diff hello_O0.s hello_O2.s
```

Or inspect manually.



###  What You'll Notice:

#### 🔴 `-O0` (No Optimization):

* Function calls are preserved (e.g., `square()` is called from `main()`)
* Variables are stored in the stack (e.g., `x`, `y`, `result`)
* Prologue/epilogue code is longer (push/pop, save/restore)

**Example (simplified)**:

```asm
square:
  addi sp, sp, -16
  sw ra, 12(sp)
  ...
  mul a0, a0, a0
  ...
  lw ra, 12(sp)
  addi sp, sp, 16
  ret
```



#### 🟢 `-O2` (Optimized):

* `square()` is **inlined** into `main()` — no actual function call
* Stack is not used unless absolutely necessary
* Variables might be stored in **registers only**
* Prologue/epilogue might disappear

**Example (simplified)**:

```asm
main:
  li a5, 5
  mul a0, a5, a5
  ret
```

✅ Much shorter and faster code



###  Why This Happens:

| Flag  | Behavior                                                                                                          |
| ----- | ----------------------------------------------------------------------------------------------------------------- |
| `-O0` | Keeps code simple and debuggable; uses stack and avoids reordering.                                               |
| `-O2` | Aggressively inlines functions, uses registers, eliminates unused variables, and reorders instructions for speed. |



### 📌 Summary

| Aspect         | `-O0`     | `-O2`               |
| -------------- | --------- | ------------------- |
| Function calls | Preserved | Inlined (if simple) |
| Stack usage    | High      | Minimal             |
| Code size      | Larger    | Smaller             |
| Speed          | Slower    | Faster              |
| Debuggable     | ✅ Yes     | ❌ Harder            |

---
### **✅Output and Code**

![Screenshot from 2025-06-05 12-35-51](https://github.com/user-attachments/assets/302f172c-614d-4bbf-8aa1-db76e70600c9)
![Screenshot from 2025-06-05 12-36-41](https://github.com/user-attachments/assets/3baba9ba-62a1-454a-b3d7-1e5bda3ff40b)
![Screenshot from 2025-06-05 12-37-19](https://github.com/user-attachments/assets/996f4d4b-6df3-4379-affd-68ab5fb2c1a9)
![Screenshot from 2025-06-05 12-37-34](https://github.com/user-attachments/assets/a90f242b-ab4d-45a7-ad87-701ab77b7d31)

---

#  **9. <ins> Inline Assembly Basics</ins>** 


### ✅ C Function to Read `cycle` CSR

```c
unsigned int read_cycle() {
    unsigned int cycle;
    asm volatile ("csrr %0, cycle" : "=r"(cycle));
    return cycle;
}
```

###  Explanation

### 🔹 `asm volatile`

* `asm`: Tells the compiler this is an inline assembly block.
* `volatile`: Prevents the compiler from optimizing away or reordering this instruction, which is important for hardware timing-related instructions.

### 🔹 `"csrr %0, cycle"`

* `csrr` = "Control and Status Register Read"
* `%0` is a placeholder for the first output operand, i.e., where the result goes
* `cycle` is a **named alias** for CSR address `0xC00`

> You could also use `"csrr %0, 0xC00"` if the name isn't available.

### 🔹 `: "=r"(cycle)`

* This is the **output operand** section of the inline assembly.

#### Breakdown:

* `=` → This tells the compiler it’s an output.
* `r` → Use a general-purpose **register**.
* `(cycle)` → Store the result of the `csrr` instruction in the `cycle` variable.



### 🧪 Example Usage

```c
#include <stdio.h>

unsigned int read_cycle();

int main() {
    unsigned int c = read_cycle();
    printf("Cycle count: %u\n", c);
    return 0;
}
```

> If you're doing bare-metal development, `printf()` might not work unless redirected to UART or semihosting.



### 📌 Summary of Inline Assembly Constraints

| Constraint | Meaning                      |
| ---------- | ---------------------------- |
| `=`        | Output operand               |
| `r`        | Any general-purpose register |
| `(var)`    | Corresponding C variable     |

---

#  **10. <ins>Memory-Mapped I/O Demo</ins>**


### ✅ C Code Snippet: Toggle GPIO Register

```c
#include <stdint.h>

// Define the GPIO address as a volatile pointer
#define GPIO_REG (*(volatile uint32_t*)0x10012000)

void toggle_gpio() {
    GPIO_REG = 1;        // Set the GPIO pin (e.g., high)
    for (volatile int i = 0; i < 100000; ++i); // Delay loop
    GPIO_REG = 0;        // Clear the GPIO pin (e.g., low)
}
```


### 🔐 Preventing Compiler Optimization

The key is:

### 🔸 `volatile`

* Tells the compiler:

  > “Do not optimize or cache reads/writes to this address.”
* Without `volatile`, the compiler might skip writes if it thinks the memory isn't used.

### 🔸 `for (volatile int i = ...)`

* This prevents the delay loop from being optimized away.

---

### 🔁 Optional: Looping Toggle

To toggle it repeatedly:

```c
void _start() {
    while (1) {
        toggle_gpio();
    }
}
```

---

### ⚙️ Compiling & Running

Assume you have a `linker.ld` and `riscv32-unknown-elf-gcc`:

```bash
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -T linker.ld \
  -nostdlib -o gpio_toggle.elf gpio_toggle.c
```

Run with QEMU:

```bash
qemu-system-riscv32 -nographic -machine sifive_e -kernel gpio_toggle.elf
```

---

#  **11. <ins>Linker Script 101</ins>** 


### ✅ Minimal Linker Script (`linker.ld`)

```ld
SECTIONS {
  /* Code (Flash) at address 0x00000000 */
  .text 0x00000000 : {
    *(.text*)
  }

  /* Initialized data (SRAM) at address 0x10000000 */
  .data 0x10000000 : {
    *(.data*)
  }

  /* Uninitialized data (BSS) follows .data in RAM */
  .bss : {
    *(.bss*)
    *(COMMON)
  }
}
```


###  How to Use It

Compile your C file with:

```bash
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -T linker.ld -nostdlib -o myprog.elf myprog.c
```

Run it with QEMU:

```bash
qemu-system-riscv32 -nographic -machine sifive_e -kernel myprog.elf
```


###  Explanation: Flash vs SRAM Addressing

| Section | Memory Type | Address      | Why?                                                                           |
| ------- | ----------- | ------------ | ------------------------------------------------------------------------------ |
| `.text` | Flash / ROM | `0x00000000` | Program code lives in **non-volatile Flash** so it's retained after power off. |
| `.data` | SRAM        | `0x10000000` | Data must be **read/write at runtime**, so it's placed in **RAM (SRAM)**.      |

### Note:

* On real hardware, `.data` is usually **copied from Flash to RAM** during startup.
* `.bss` is zero-initialized in RAM.

---

### **✅Output and Code**

![Screenshot from 2025-06-05 17-22-11](https://github.com/user-attachments/assets/63ac0a71-2980-499e-a880-2dbc453a3a63)

---


#  **12. <ins>Start-up Code & crt0</ins>** 

### ✅ What `crt0.S` Does in a Bare-Metal RISC-V Program

`crt0.S` (also known as the "C runtime zero") is **startup assembly code** that's executed **before `main()`**. It sets up the **runtime environment** needed for your C code to run properly.


###  Typical Responsibilities of `crt0.S`

1. **Set up the stack pointer**

   * The CPU doesn’t know where the stack is. `crt0.S` sets it up, often like:

     ```asm
     la sp, _stack_top
     ```

2. **Zero the `.bss` section**

   * `.bss` contains uninitialized global/static variables which must be zeroed manually:

     ```asm
     la a0, __bss_start
     la a1, __bss_end
     loop:
       beq a0, a1, done
       sw zero, 0(a0)
       addi a0, a0, 4
       j loop
     done:
     ```

3. **Copy `.data` from Flash to RAM** (if applicable)

   * For initialized globals:

     ```asm
     la a0, _data_load_start
     la a1, _data_start
     la a2, _data_end
     loop_copy:
       beq a1, a2, done_copy
       lw t0, 0(a0)
       sw t0, 0(a1)
       addi a0, a0, 4
       addi a1, a1, 4
       j loop_copy
     done_copy:
     ```

4. **Call `main()`**

   ```asm
   call main
   ```

5. **Hang if `main()` returns**

   ```asm
   loop_forever:
     j loop_forever
   ```



###  Where to Get `crt0.S`

* ✅ **Newlib** provides portable `crt0.S` implementations under `libgloss/`:

  * GitHub: [newlib/libgloss/riscv/crt0.S](https://sourceware.org/git/?p=newlib-cygwin.git;a=blob;f=libgloss/riscv/crt0.S)

* ✅ **Platform SDKs / BSPs (Board Support Packages)**:

  * SiFive Freedom E SDK
  * Kendryte SDK (for K210 chips)
  * Renode simulation templates

* ✅ **Write your own** if you’re learning or on custom hardware.



###  Summary

| Feature              | Purpose                              |
| -------------------- | ------------------------------------ |
| Set stack pointer    | For function calls & local variables |
| Zero `.bss`          | C standard compliance                |
| Copy `.data`         | Load initialized globals             |
| Call `main()`        | Start the C program                  |
| Trap if `main` exits | Infinite loop                        |

---

#  **13. <ins>Interrupt Primer</ins>** 

### ✅ Step-by-Step: MTIP Setup with a Simple ISR

### What is MTIP?

* MTIP (Machine Timer Interrupt Pending) gets set when `mtime >= mtimecmp`.
* You must:

  1. Configure the CLINT (`mtimecmp` and `mtime`)
  2. Enable timer interrupt in `mie`
  3. Enable global interrupts in `mstatus`
  4. Set `mtvec` to your handler



###  1. `main.c` – Timer Setup and Handler

```c
#include <stdint.h>

// Memory-mapped CLINT registers (QEMU/Spike default)
#define MTIME      (*(volatile uint64_t*)(0x0200BFF8))
#define MTIMECMP   (*(volatile uint64_t*)(0x02004000))

volatile int tick = 0;

void enable_mtip() {
    // Schedule timer interrupt (e.g., 100000 ticks in the future)
    MTIMECMP = MTIME + 100000;

    // Enable machine-timer interrupt
    asm volatile("csrs mie, %0" :: "r"(1 << 7));

    // Enable global interrupt
    asm volatile("csrs mstatus, %0" :: "r"(1 << 3));

    // Set mtvec to address of timer handler (direct mode)
    extern void timer_isr();
    asm volatile("la t0, timer_isr\ncsrw mtvec, t0");
}

int main() {
    enable_mtip();
    while (1) {
        // Tick updated by interrupt
        if (tick >= 5) {
            // Done after 5 ticks
            while (1);
        }
    }
}
```


###  2. `isr.S` – Timer Interrupt Handler in ASM

```asm
.section .text
.global timer_isr
timer_isr:
    // Save registers
    addi sp, sp, -16
    sw ra, 12(sp)
    sw t0, 8(sp)
    sw t1, 4(sp)

    // Acknowledge timer interrupt: set mtimecmp = mtime + 100000
    li t0, 0x0200BFF8       # mtime
    ld t1, 0(t0)
    li t0, 0x02004000       # mtimecmp
    addi t1, t1, 100000
    sd t1, 0(t0)

    // Increment C variable 'tick'
    la t0, tick
    lw t1, 0(t0)
    addi t1, t1, 1
    sw t1, 0(t0)

    // Restore and return
    lw t1, 4(sp)
    lw t0, 8(sp)
    lw ra, 12(sp)
    addi sp, sp, 16
    mret
```


### 3. `linker.ld` – Minimal Linker Script

```ld
ENTRY(_start)

MEMORY
{
  RAM (rwx) : ORIGIN = 0x80000000, LENGTH = 64K
}

SECTIONS
{
  . = ORIGIN(RAM);

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


### 🏗 4. Compile and Run (QEMU)

### 🔧 Compile:

```bash
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -T linker.ld \
  -nostartfiles -nostdlib -o timer.elf main.c isr.S
```

###  Run on QEMU:

```bash
qemu-system-riscv32 -nographic -machine sifive_e -kernel timer.elf
```

You can print a message inside the ISR by writing to a UART MMIO location (e.g., `0x10000000`).



###  Summary

| Component     | Action                        |
| ------------- | ----------------------------- |
| `mtimecmp`    | Set to future value           |
| `mie` CSR     | Enable timer interrupt        |

| `mstatus` CSR | Enable global interrupts      |
| `mtvec` CSR   | Point to your ISR             |
| `ISR`         | Acknowledge + action + `mret` |

---
### **✅Output and Code**

![Screenshot from 2025-06-06 20-04-07](https://github.com/user-attachments/assets/d2a7fcd7-872c-450e-b57f-2e295607e194)
![Screenshot from 2025-06-06 20-12-18](https://github.com/user-attachments/assets/f534f716-a517-4247-a62a-02ea8ff1bdad)
![Screenshot from 2025-06-06 20-15-45](https://github.com/user-attachments/assets/40b697f3-d9db-4ac2-a292-1b62b453d428)
![Screenshot from 2025-06-06 20-18-42](https://github.com/user-attachments/assets/e3e319ed-da57-4729-8275-6765594e23fd)
![Screenshot from 2025-06-06 20-24-40](https://github.com/user-attachments/assets/f79b1118-8627-4420-a5fe-25a71bd909f6)

---






