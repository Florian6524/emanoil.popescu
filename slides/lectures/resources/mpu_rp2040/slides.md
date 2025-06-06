---
layout: section
---

# MPU for RP2040
Protected Memory System Architecture v7 (PMSAv7)


---
---
# Bibliography
for this section

**Joseph Yiu**, *The Definitive Guide to ARM® Cortex®-M0 and Cortex-M0+ Processors, 2nd Edition* 
   - Chapter 12 - *Memory Protection Unit*

---
layout: two-cols
---
# MPU for RP2040
Cortex-M0+ works in three modes

- **handler** mode - *no restrictions* - used while executing ISRs and Exception Handlers
- **thread** mode
  - **privileged** *no restrictions* - usually used for the operating system
  - **unprivileged** mode - *allows only ALU and memory access through Memory Protection* - used for applications

MPU allows 8 regions
- each region has up to 8 subregions
- permissions R W X

:: right ::

<img src="./mpu_rp2040.svg" class="w-120 rounded">


---
layout: two-cols
---
# Memory Protection Unit
Cortex-M MPU (PMSAv7-m)

<style>
.two-columns {
    grid-template-columns: 3fr 2fr;
}
</style>

<img src="./mpu.svg" class="w-120 rounded">

- allows the definition of *memory regions*
- regions can overlap, *highest region number* takes *priority*
- regions have access permissions (similar to rwx)

$$ region\_size = 2^{size}, size >= 8 $$
$$ base\_address = region\_size \times N $$

:: right ::

<img src="./mpu_regions.svg" class="w-70 m-5 rounded">

---
layout: two-cols
---

# Memory Protection Unit
Access Protection

<img src="./mpu.svg" class="w-120 rounded">

**AP** Access Protection

**XN** eXecute Never 
  - faults if MCU has to read the next instruction from an *XN* region

:: right ::

| **AP** | Privileged Mode | Unprivileged Mode |
|-------|------------|--------------|
| `000` | No Access | No Access |
| `001` | Read/Write | No Access |
| `010` | Read/Write | Read only |
| `011` | Read/Write | Read/Write |
| <span color="red">`100`</span> | <span color="red">Do not use</span> | <span color="red">Do not use</span> |
| `101` | Read only | No Access |
| `110` | Read only | Read only |
| `111` | Read/Write | Read only |

---
layout: two-cols
---
# Subregions

<style>
.two-columns {
    grid-template-columns: 5fr 3fr;
}
</style>

- each region is divided in 8 subregion
- each bit in `Subregion Disable` disables a subregion
- a disabled subregion triggers a fault if accessed

<img src="./mpu.svg" class="w-120 rounded">

:: right ::

<img src="./subregions.svg" class="w-70 rounded">

---
layout: two-cols
---
# Subregions' Usage
improve granularity

<style>
.two-columns {
    grid-template-columns: 5fr 3fr;
}
</style>

$$ region\_size = 2^{size}, size >= 8 $$
$$ base\_address = region\_size \times N $$
$$ subregion\_size = \frac{region\_size}{8} $$

- a 5K region is not allowed (5K is not a power of 2)
- use two 4K regions back to back
- disable 6 of the subregions (subregion is 512B)

:: right ::

<img src="./regions_and_subregions.svg" class="w-70 rounded">

---
layout: two-cols
---
# Memory Layout
protection

<style>
.two-columns {
    grid-template-columns: 5fr 2fr;
}
</style>

### Flash

- **Code** - *read* and *execute*
- **.rodata** - constants - *read only*
- **.data** - *in flash* - initialized global variables
  - is copied to RAM at startup by the `init` function
  - *should not be accessed after startup*

### RAM
- **stack** - *read* and *write*
  - *usually protected by unaccessible memory before and after*
- **.data** - *in RAM* - global variables - *read* and write
- **.bss** - global variables (not initialized or initialized to `0`) - *read* and *write*

:: right ::

<img src="./layout.svg" class="w-64 rounded">
