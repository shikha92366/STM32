




# STM32 Bare-Metal Programming

This section focuses on understanding STM32 at the **register and memory level**.

The objective is to understand how a simple hardware operation, such as turning ON an LED, is performed by directly configuring STM32 registers.

---

## 📚 Topics Covered

- Registers
- Logic Gates
- Bitwise Operators
- User Manual (UM)
- Datasheet (DS)
- Reference Manual (RM)
- Memory Map
- Peripheral Base Address
- Register Offset
- RCC
- AHB1 Bus
- GPIOA
- GPIO Registers
- GPIO Mode Register (`MODER`)
- GPIO Output Data Register (`ODR`)
- Register Reset
- Bit Manipulation
- Register-Level LED Control

---

# 1. Registers

A **register** is a storage location used by the microcontroller to store data or control the operation of a peripheral.

The basic concept is:

```text
Register
   ↓
Stores a value
   ↓
Controls a peripheral
   ↓
Affects hardware
````

For example:

```text
Register Value
      ↓
GPIO Peripheral
      ↓
GPIO Pin
      ↓
LED
```

---


## Steps to Turn ON an LED

The following steps are required to turn ON an LED using STM32 GPIO:


1. **Identify the GPIO Port and Pin**
   - Identify the port and pin connected to the LED.
   - Example: GPIOA, Pin 5 (PA5).

2. **Enable the Peripheral Clock**
   - Enable the clock for the GPIO peripheral using the RCC.

3. **Configure the GPIO Mode**
   - Configure the GPIO pin as an output.
   - For register-level programming, configure the GPIOx_MODER register.

4. **Set the GPIO Output State**
   - Set the pin HIGH to turn ON the LED.
   - Set the pin LOW to turn OFF the LED.
   - Use the GPIOx_ODR or GPIOx_BSRR register.

# 2. Logic Gates

Logic gates are important because register programming involves manipulating individual bits.

## AND

| A | B | Output |
| - | - | ------ |
| 0 | 0 | 0      |
| 0 | 1 | 0      |
| 1 | 0 | 0      |
| 1 | 1 | 1      |

## OR

| A | B | Output |
| - | - | ------ |
| 0 | 0 | 0      |
| 0 | 1 | 1      |
| 1 | 0 | 1      |
| 1 | 1 | 1      |

## NOT

| Input | Output |
| ----- | ------ |
| 0     | 1      |
| 1     | 0      |

## XOR

| A | B | Output |
| - | - | ------ |
| 0 | 0 | 0      |
| 0 | 1 | 1      |
| 1 | 0 | 1      |
| 1 | 1 | 0      |

These operations form the basis of bit manipulation used in register programming.

---

# 3. Bitwise Operators

The important bitwise operators used while learning register-level programming are:

| Operator | Operation   |
| -------- | ----------- |
| `&`      | AND         |
| `\|`     | OR          |
| `~`      | NOT         |
| `^`      | XOR         |
| `<<`     | Left Shift  |
| `>>`     | Right Shift |

---

## Left Shift `<<`

The left shift operator moves bits towards the left.

Example:

```text
7 = 00000111

7 << 1 = 00001110 = 14

7 << 2 = 00011100 = 28

7 << 3 = 00111000 = 56
```

A left shift is especially useful when placing a value at a particular register bit position.

Example:

```c
1U << 5
```

This moves bit `0` to bit `5`.

---

## Set a Bit

```c
REG |= (1U << n);
```

## Clear a Bit

```c
REG &= ~(1U << n);
```

## Toggle a Bit

```c
REG ^= (1U << n);
```

---

# 4. STM32 Documentation

While learning bare-metal programming, the STM32 documentation is used to find the information required to control the hardware.

The three important documents are:

| Abbreviation | Full Form        | Purpose                                           |
| ------------ | ---------------- | ------------------------------------------------- |
| **UM**       | User Manual      | Development board information                     |
| **DS**       | Datasheet        | MCU pin and device information                    |
| **RM**       | Reference Manual | Peripherals, registers, memory map and bit fields |

---

# 5. UM — User Manual

**UM = User Manual**

The User Manual is used to understand the **development board** and its hardware connections.

For the onboard LED:

```text
UM
 ↓
Search for LD2
 ↓
Find LED connection
 ↓
PA5
```

The documentation shows:

```text
LD2
 ↓
PA5
```

Therefore:

```text
Port = GPIOA
Pin  = 5
```

### What UM tells us

> Which component on the development board is connected to which MCU pin?

---

# 6. DS — Datasheet

**DS = Datasheet**

The Datasheet provides information about the **STM32 microcontroller**.

It is used to find:

* MCU pin information
* Pinout
* GPIO pins
* Pin functions
* Alternate functions
* Device information

For the LED pin:

```text
PA5
 ↓
GPIOA
 ↓
MCU Pin
```

The Datasheet also provides the alternate-function mapping for the GPIO pins.

For PA5, the pin mapping table can be used to understand the available functions of the pin.

---

# 7. RM — Reference Manual

**RM = Reference Manual**

The Reference Manual is used to understand the STM32 peripherals and their registers.

It provides information about:

* Memory map
* Peripheral addresses
* Register addresses
* Register offsets
* Register reset values
* Register bit fields
* RCC
* GPIO
* Peripheral configuration

The Reference Manual is the main document used for register-level programming.

---

# 8. Documentation Flow


The complete documentation flow used during this learning process is:



```text


UM
User Manual
   ↓
Find LED
   ↓
Identify PA5

DS
Datasheet
   ↓
Understand PA5
   ↓
Identify GPIOA

RM
Reference Manual
   ↓
Find GPIOA
   ↓
Find RCC
   ↓
Find Memory Map
   ↓
Find Base Address
   ↓
Find Register Offset
   ↓
Find Required Bit
   ↓
Write Register-Level Code
```

A simple way to remember:

```text
UM → Board

DS → MCU

RM → Peripheral + Registers
```

---

# 9. Memory Map
Identify Port A, Pin 5 (PA5) – User LED (LD2) Connection
<img width="947" height="213" alt="ld2" src="https://github.com/user-attachments/assets/788d75a5-c0f3-4ee7-b08b-0a387677c763" />
<img width="1411" height="842" alt="port a pin 5" src="https://github.com/user-attachments/assets/09ea982c-58aa-4dbe-890e-dbb398279d6d" />
Find GPIOA Peripheral on AHB1 Bus
<img width="1155" height="475" alt="AHB1" src="https://github.com/user-attachments/assets/282de250-6fe2-4334-bf4a-5d8114dcb4f5" />
<img width="1567" height="800" alt="gpio offset" src="https://github.com/user-attachments/assets/398f3c6f-259e-4b43-b923-3a8819018ae2" />




The STM32 memory map shows where different peripherals are located in the MCU address space.

From the Reference Manual, the peripheral boundary addresses show:

```text
RCC
0x4002 3800 - 0x4002 3BFF

GPIOA
0x4002 0000 - 0x4002 03FF
```

Both RCC and GPIOA are located on:

```text
AHB1
```

---

# 10. GPIOA Base Address

From the STM32F401xB/C and STM32F401xD/E register boundary address table:

```text
GPIOA
0x4002 0000 - 0x4002 03FF
```

Therefore:

```text
GPIOA Base Address = 0x40020000
```

The base address is the starting address of the GPIOA peripheral register block.

---

# 11. RCC Base Address

From the same memory map:

```text
RCC
0x4002 3800 - 0x4002 3BFF
```

Therefore:

```text
RCC Base Address = 0x40023800
```

---

# 12. AHB1 Bus

GPIOA and RCC are located on the **AHB1 bus**.

```text
AHB1
 │
 ├── RCC
 │
 ├── GPIOA
 ├── GPIOB
 ├── GPIOC
 ├── GPIOD
 ├── GPIOE
 └── GPIOH
```

The STM32 documentation also shows GPIOA as an AHB1 peripheral.

---

# 13. RCC AHB1 Peripheral Clock Enable

Before using GPIOA, its peripheral clock needs to be enabled.

The relevant register is:

```text
RCC_AHB1ENR
```

The register information is found in the RCC section of the Reference Manual.

The RCC AHB1 peripheral clock enable register contains enable bits for AHB1 peripherals.

For GPIOA:

```text
GPIOAEN = Bit 0
```

Therefore:

```text
Bit 0 = 1
```

is used to enable the GPIOA clock.

---

# 14. RCC Register Address

The RCC base address is:

```text
0x40023800
```

The `RCC_AHB1ENR` register has:

```text
Address Offset = 0x30
```

Therefore:

```text
RCC_AHB1ENR Address
=
RCC Base Address + Offset

=
0x40023800 + 0x30

=
0x40023830
```

So:

```text
RCC_AHB1ENR = 0x40023830
```

---

# 15. GPIOA Clock Enable

The GPIOA clock-enable bit is:

```text
GPIOAEN → Bit 0
```

Therefore the operation conceptually becomes:

```c
RCC_AHB1ENR |= (1U << 0);
```

or:

```c
RCC_AHB1ENR |= 0x01;
```

This enables the clock for GPIOA.

---

# 16. GPIOA Reset Register

The Reference Manual also contains the:

```text
RCC_AHB1RSTR
```

register.

Its:

```text
Address Offset = 0x10
```

The GPIOA reset bit is:

```text
GPIOARST → Bit 0
```

The documentation specifies:

```text
0 → Does not reset GPIOA

1 → Resets GPIOA
```

This is different from the GPIOA **clock-enable bit**.

### Important distinction

```text
RCC_AHB1ENR
        ↓
GPIOA Clock Enable

RCC_AHB1RSTR
        ↓
GPIOA Reset
```

For normal GPIO operation, the important step is enabling the GPIOA clock.

---

# 17. GPIO Port Mode Register — MODER

The GPIO mode register is:

```text
GPIOx_MODER
```

For GPIOA:

```text
GPIOA_MODER
```

The Reference Manual shows:

```text
Address Offset = 0x00
```

Therefore:

```text
GPIOA_MODER Address
=
GPIOA Base Address + 0x00

=
0x40020000 + 0x00

=
0x40020000
```

So:

```text
GPIOA_MODER = 0x40020000
```

---

# 18. GPIO Mode Configuration

Each GPIO pin uses **2 bits** in the `MODER` register.

For pin `y`:

```text
Bits = 2y : 2y+1
```

The available modes are:

| Value | Mode                   |
| ----- | ---------------------- |
| `00`  | Input                  |
| `01`  | General Purpose Output |
| `10`  | Alternate Function     |
| `11`  | Analog                 |

---

# 19. PA5 Mode Configuration

The LED is connected to:

```text
PA5
```

For pin 5:

```text
2 × 5 = 10
```

Therefore PA5 uses:

```text
MODER5[1:0]
```

which corresponds to:

```text
Bits 11:10
```

To configure PA5 as a general-purpose output:

```text
MODER5 = 01
```

Conceptually:

```text
Bits 11:10

01
```

The `01` value must be placed at the correct bit position using a left shift.

```c
1U << (5 * 2)
```

---

# 20. GPIO Output Data Register — ODR

After configuring PA5 as an output, the output state must be controlled.

The GPIO output data register is:

```text
GPIOx_ODR
```

For GPIOA:

```text
GPIOA_ODR
```

The register is used to control the output state of GPIO pins.

Conceptually:

```text
ODR Bit = 0
     ↓
GPIO Output LOW

ODR Bit = 1
     ↓
GPIO Output HIGH
```

---

# 21. PA5 Output

For PA5:

```text
Pin Number = 5
```

Therefore the corresponding ODR bit is:

```text
Bit 5
```

To set PA5 HIGH:

```c
GPIOA_ODR |= (1U << 5);
```

To set PA5 LOW:

```c
GPIOA_ODR &= ~(1U << 5);
```

According to the board documentation for LD2:

```text
I/O HIGH → LED ON

I/O LOW → LED OFF
```

---

# 22. Complete Register-Level Flow

The complete process is:

```text
                 UM
                 ↓
             Find LD2
                 ↓
                PA5
                 ↓
                 DS
                 ↓
            Understand PA5
                 ↓
                 RM
                 ↓
             Find GPIOA
                 ↓
          Find GPIOA Address
                 ↓
          0x40020000
                 ↓
          Find RCC Address
                 ↓
          0x40023800
                 ↓
       Find AHB1 Clock Register
                 ↓
          RCC_AHB1ENR
                 ↓
          Offset = 0x30
                 ↓
          Address = 0x40023830
                 ↓
        Set GPIOAEN Bit 0
                 ↓
          GPIOA Clock ON
                 ↓
        GPIOA_MODER
                 ↓
          PA5 → Output
                 ↓
          GPIOA_ODR
                 ↓
          Set Bit 5
                 ↓
              PA5 HIGH
                 ↓
             LED ON
```

---

# 23. Register Address Calculation

The general formula is:

```text
Register Address
=
Peripheral Base Address
+
Register Offset
```

### RCC AHB1ENR

```text
RCC Base Address = 0x40023800

AHB1ENR Offset = 0x30

Register Address:

0x40023800 + 0x30
= 0x40023830
```

### GPIOA MODER

```text
GPIOA Base Address = 0x40020000

MODER Offset = 0x00

Register Address:

0x40020000 + 0x00
= 0x40020000
```

This is the process used to locate registers instead of memorizing their addresses.

---

# 24. Pointer-Based Register Access

A memory-mapped register can be accessed through a pointer.

Conceptually:

```c
volatile uint32_t *REG =
    (volatile uint32_t *)(BASE_ADDRESS + OFFSET);
```

The register can then be accessed using:

```c
*REG
```

For example:

```c
*REG |= (1U << n);
```

---

# 25. Why `volatile`?

Peripheral registers are hardware-controlled memory locations.

The `volatile` keyword tells the compiler that the value at that memory location should be accessed as specified and should not be treated like an ordinary variable.

Example:

```c
volatile uint32_t *REG;
```

---

# 26. Read-Modify-Write

When only particular bits of a register need to be changed, a read-modify-write operation can be used.

```text
Read Register
      ↓
Modify Required Bits
      ↓
Write Register
```

Example:

```c
REG |= (1U << n);
```

This sets the selected bit while preserving the other bits.

---

# 27. Bare-Metal LED Control

The complete hardware operation can be summarized as:

```text
RCC
 ↓
Enable GPIOA Clock
 ↓
GPIOA
 ↓
MODER
 ↓
Configure PA5 as Output
 ↓
ODR
 ↓
Set Bit 5
 ↓
PA5 = HIGH
 ↓
LD2 = ON
```

---

# 🎯 Main Learning

The important part of bare-metal programming is **not memorizing addresses**.

The important skill is knowing how to find the required information from the STM32 documentation.

```text
UM
 ↓
Find the board component

DS
 ↓
Understand the MCU pin

RM
 ↓
Find the peripheral

Memory Map
 ↓
Find the peripheral base address

Register Map
 ↓
Find the register offset

Register Description
 ↓
Find the required bit / bit field

Bitwise Operations
 ↓
Modify the register

C Code
 ↓
Control the hardware
```

---

# 🧠 Key Points

```text
UM  → User Manual
DS  → Datasheet
RM  → Reference Manual
```

```text
GPIOA Base Address = 0x40020000
```

```text
RCC Base Address = 0x40023800
```

```text
RCC_AHB1ENR Offset = 0x30
```

```text
RCC_AHB1ENR Address = 0x40023830
```

```text
GPIOAEN = Bit 0
```

```text
GPIOA_MODER Offset = 0x00
```

```text
PA5 → MODER Bits 11:10
```

```text
PA5 Output → ODR Bit 5
```

---

# 📖 Learning Sequence

```text
Registers
   ↓
Logic Gates
   ↓
Bitwise Operators
   ↓
UM
   ↓
DS
   ↓
RM
   ↓
Memory Map
   ↓
Peripheral Base Address
   ↓
Register Offset
   ↓
RCC
   ↓
GPIOA Clock Enable
   ↓
GPIOA MODER
   ↓
PA5 Output Mode
   ↓
GPIOA ODR
   ↓
PA5 HIGH
   ↓
LED ON
```

---

# 🚀 Next Step

The next step is to implement the above process in **C using direct register access** and verify the register values using the debugger.

````

### One important correction in your documentation

Your screenshots show two different RCC registers:

```text
RCC_AHB1RSTR
Offset = 0x10
GPIOARST = Bit 0
````

and

```text
RCC_AHB1ENR
Offset = 0x30
GPIOAEN = Bit 0
```

They are **not the same thing**:

```text
AHB1RSTR → Reset GPIOA
AHB1ENR  → Enable GPIOA clock
```

For turning the LED on, your actual sequence should focus on **`RCC_AHB1ENR` → GPIOAEN bit 0**, then `GPIOA_MODER`, then `GPIOA_ODR`.

Also, the screenshots you provided are excellent to include in the GitHub documentation because they show your **actual process of deriving the values from the official STM32 documentation**, rather than just presenting the final code.
