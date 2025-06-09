# *<ins>Caravel Sub-Modules: Functional Summary and Analysis</ins>*

---

### 1. **padframe**

**Source File:** `rtl/padframe.v`

**Purpose:**
The `padframe` module interfaces internal logic with external pins. It includes special I/O pad cells that handle buffering, protection, and voltage translation.

**Pad-Type Variants Found:**

* `gpio_pad` → `sky130_ef_io__gpiov2_pad_wrapped`
 
---

### 2. **housekeeping**

**Source File:** `rtl/housekeeping/spi_master.v` + helpers

**Purpose:**
Manages low-level SPI operations and system configuration. Used for tasks like boot control and SPI flash interaction.

**SPI Signals:**

* SPI Clock = `sck = mprj_io[32]`
* SPI Chip Select (CSB) = `csb = mprj_io[33]`
* SPI MISO (SDI) = `mprj_io[34]`
* SPI MOSI (SDO) = `mprj_io[35]`

**Default SPI Mode:**

* Mode 0 (CPOL = 0, CPHA = 0)

---

### 3. **management\_soc\_wrapper**

**Source Directory:** `rtl/mgmt_soc/**`

**Purpose:**
Wraps the Management SoC which runs a RISC-V core. It manages communication, programming, debugging, and controls the user area.

**Specs:**

* Wishbone Bus Data Width: **32 bits**
* IRQ Fan-in Size: **3** (via `user_irq[2:0]`)

---

### 4. **user\_project\_wrapper**

**Source File:** `rtl/user_project_wrapper.v`

**Purpose:**
Encapsulates the user logic/IP. It's the main area provided to users in Caravel for integrating their custom design.

**Features:**

* Number of steerable GPIOs: **38** (mapped as `mprj_io[37:0]`)
* Clock Source: **`user_clock2`** (provided by the management SoC)

---

### Summary Table

| Sub-module               | Source File(s)                  | Notes                                                         |
| ------------------------ | ------------------------------- | ------------------------------------------------------------- |
| padframe                 | `rtl/padframe.v`                | Found pad type: `gpio_pad`                                    |
| housekeeping             | `rtl/housekeeping/spi_master.v` | SPI pins: `sck = mprj_io[32]`, `csb = mprj_io[33]`SPI mode: 0 |
| management\_soc\_wrapper | `rtl/mgmt_soc/**`               | Wishbone: 32-bit, IRQs: 3                                     |
| user\_project\_wrapper   | `rtl/user_project_wrapper.v`    | 38 GPIOs steerable, clock from `user_clock2`                  |

---
