#  <ins>**Caravel Sub-Modules: Functional Summary and Hierarchical Analysis**</ins>

This document provides a detailed summary and structural insight into the key submodules of the [Caravel SoC](https://github.com/efabless/caravel), based on RTL-level analysis of the `caravel.v` file.

---

---

##  1. padframe

- **Source File:** `rtl/padframe.v`
- **Purpose:**  
  The `padframe` module interfaces internal digital logic with external I/O pins. It incorporates special I/O pad cells that perform buffering, electrostatic protection, and voltage translation.

###  Pad-Type Variants Identified:
- `gpio_pad` → `sky130_ef_io__gpiov2_pad_wrapped`  
  *(Note: Only one pad-type variant was found in the provided file.)*

---

##  2. housekeeping

- **Source File:** `rtl/housekeeping/spi_master.v` + helpers
- **Purpose:**  
  Handles low-level system operations like SPI boot, system configuration, and flash programming control.

###  SPI Interface:
- **SPI Clock:** `sck = mprj_io[32]`
- **Chip Select (CSB):** `csb = mprj_io[33]`
- **MISO (SDI):** `mprj_io[34]`
- **MOSI (SDO):** `mprj_io[35]`

- **Default SPI Mode:**  
  Mode 0 (CPOL = 0, CPHA = 0)

---

##  3. management_soc_wrapper

- **Source Directory:** `rtl/mgmt_soc/**`
- **Purpose:**  
  Contains the RISC-V based management SoC, responsible for system initialization, debug control, flash communication, and Wishbone bus interfacing.

###  Key Specs:
- **Wishbone Bus Data Width:** 32 bits  
- **IRQ Fan-in Size:** 3 (`user_irq[2:0]`)

---

##  4. user_project_wrapper

- **Source File:** `rtl/user_project_wrapper.v`
- **Purpose:**  
  Contains the user-defined logic or IP. This is the primary sandbox area for custom SoC extensions using the provided interfaces.

###  Features:
- **Steerable GPIOs:** 38 (`mprj_io[37:0]`)
- **Clock Source:** `user_clock2` (provided by `management_soc`)

---

## 🧾 Submodule Summary Table

| Sub-module               | Source File(s)                  | Notes                                                             |
|--------------------------|----------------------------------|-------------------------------------------------------------------|
| `padframe`               | `rtl/padframe.v`                | Pad type used: `gpio_pad`                                         |
| `housekeeping`           | `rtl/housekeeping/spi_master.v` | SPI pins: `mprj_io[32–35]`<br>Default Mode: 0                     |
| `management_soc_wrapper` | `rtl/mgmt_soc/**`               | Wishbone Bus: 32-bit<br>IRQ Inputs: 3                             |
| `user_project_wrapper`   | `rtl/user_project_wrapper.v`    | GPIOs: 38<br>Clock Source: `user_clock2`                          |

---

## Signals Crossing the “Management Protect” Boundary

These are key interface signals between the management domain and the user domain:

###  Wishbone Bus
- `wb_clk_i`, `wb_rst_i`, `wbs_stb_i`, `wbs_cyc_i`, `wbs_we_i`, `wbs_sel_i`, `wbs_dat_i`, `wbs_adr_i`, `wbs_dat_o`, `wbs_ack_o`

###  Logic Analyzer
- `la_data_in`, `la_data_out`, `la_oenb`

###  GPIO Interface
- `io_in`, `io_out`, `io_oeb`

###  Interrupts
- `user_irq[2:0]`

---

##  Clock and Reset Synchronization

- The main clock is received via the `clk` pad through the `padframe`.
- Synchronization and internal distribution occur in:
  - `housekeeping` → for SPI/config and clock mux control
  - `management_soc_wrapper` → distributes the final synchronized clock (`wb_clk_i`) and reset (`wb_rst_i`) across all submodules.

---

> 📂 **Reference File**: `caravel.v`  

