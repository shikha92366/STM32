# CMSIS Programming

This section focuses on **CMSIS-based register-level programming** on STM32 microcontrollers.

The goal is to understand how CMSIS simplifies direct hardware register access while keeping the programming approach close to the STM32 hardware.

---

## Prerequisites

Before starting CMSIS programming, you should have a basic understanding of:

* Embedded C
* STM32 GPIO
* Microcontroller registers
* Register-level / Bare-Metal programming
* RCC
* GPIO registers
* Bitwise operators
* STM32 Reference Manual
* STM32 Datasheet

> **Recommended:** Complete the Bare-Metal Programming and SFR Debugging sections before starting this section. CMSIS becomes much easier to understand once the underlying registers are familiar.

---

## What is CMSIS?

**CMSIS** stands for:

> **Cortex Microcontroller Software Interface Standard**

CMSIS provides standardized definitions and interfaces for ARM Cortex-M microcontrollers.

For STM32 development, the device-specific CMSIS headers provide access to:

* Peripheral registers
* Register structures
* Register names
* Bit definitions
* Core definitions

Instead of manually calculating peripheral addresses and register offsets, we can access registers using predefined names.

For example:

```c
RCC->AHB1ENR
GPIOA->MODER
GPIOA->ODR
```

---

# CMSIS vs Bare-Metal Programming

### Bare-Metal

In bare-metal programming, we work directly with hardware registers and may manually define:

* Peripheral base addresses
* Register offsets
* Register addresses
* Bit masks

Example:

```c
*(volatile uint32_t *)ADDRESS |= MASK;
```

### CMSIS

CMSIS provides predefined structures and definitions for the same hardware registers.

Example:

```c
RCC->AHB1ENR
```

```c
GPIOA->MODER
```

```c
GPIOA->ODR
```

Therefore, CMSIS is still **low-level register programming**. It simply provides a cleaner and more readable way to access the hardware.

---

# First CMSIS Program – GPIO LED

## Objective

Turn ON the onboard LED using CMSIS register-level programming.

The example uses:

* STM32F4 series
* GPIOA
* PA5
* LD2 onboard LED
* STM32CubeIDE

The onboard LD2 LED is connected to **PA5** on the referenced board. 

---

## Project Setup

Create a new STM32 project in STM32CubeIDE.

Select the required STM32 device/board and generate the project.

For this exercise, the GPIO configuration will be performed directly through CMSIS registers rather than relying on HAL GPIO functions.

---

# Step 1 – Include the Device Header

The STM32 device header provides the CMSIS definitions required to access the microcontroller registers.

Example:

```c
#include "stm32f4xx.h"
```

The exact header depends on the STM32 device/family being used.

---

# Step 2 – Enable the GPIO Clock

Before using GPIOA, its peripheral clock must be enabled.

The clock is controlled through the **RCC (Reset and Clock Control)** peripheral.

The register path is:

```text
RCC
 ↓
AHB1ENR
 ↓
GPIOAEN
```

CMSIS provides access to this register using:

```c
RCC->AHB1ENR
```

The GPIOA clock can then be enabled using:

```c
RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;
```

The source material explains that the GPIOA clock must first be enabled through the RCC AHB1 peripheral clock enable register. 

---

# Step 3 – Configure PA5 as Output

After enabling the GPIOA clock, PA5 needs to be configured as an output.

The GPIO mode is controlled using:

```text
GPIOA
 ↓
MODER
```

CMSIS provides direct access through:

```c
GPIOA->MODER
```

The configuration process is:

```text
GPIOA
 ↓
MODER
 ↓
PA5
 ↓
Output Mode
```

The source material emphasizes that the CMSIS register names correspond directly to the registers described in the STM32 Reference Manual. 

---

# Step 4 – Write to ODR

After configuring PA5 as an output, the output value can be controlled through the **Output Data Register (ODR)**.

The register path is:

```text
GPIOA
 ↓
ODR
 ↓
Bit 5
```

CMSIS access:

```c
GPIOA->ODR
```

To set PA5 HIGH:

```c
GPIOA->ODR |= (1U << 5);
```

The source material identifies ODR as the Output Data Register and uses PA5 as the output pin for controlling the LED. 

---

# Complete Program

```c
#include "stm32f4xx.h"

int main(void)
{
    /* Enable GPIOA clock */
    RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;

    /* Configure PA5 as output */
    GPIOA->MODER |= (1U << 10);

    /* Set PA5 HIGH */
    GPIOA->ODR |= (1U << 5);

    /* Infinite loop */
    while (1)
    {
    }
}
```

---

# Program Flow

```text
Start
  ↓
Include STM32 CMSIS Header
  ↓
Enable GPIOA Clock
  ↓
Configure PA5 as Output
  ↓
Write HIGH to PA5
  ↓
LED ON
  ↓
Infinite Loop
```

---

# Registers Used

| Peripheral | Register  | Purpose              |
| ---------- | --------- | -------------------- |
| RCC        | `AHB1ENR` | Enable GPIOA clock   |
| GPIOA      | `MODER`   | Configure PA5 mode   |
| GPIOA      | `ODR`     | Set PA5 output value |

---

# Understanding the CMSIS Syntax

### Peripheral

```c
GPIOA
```

Represents the GPIOA peripheral.

### Register

```c
GPIOA->MODER
```

Accesses the GPIOA Mode Register.

### RCC Register

```c
RCC->AHB1ENR
```

Accesses the AHB1 peripheral clock enable register.

### Set a Bit

```c
register |= mask;
```

### Clear a Bit

```c
register &= ~mask;
```

---

# CMSIS and the Reference Manual

A useful learning approach is:

```text
Reference Manual
       ↓
Find Peripheral
       ↓
Find Register
       ↓
Find Required Bit
       ↓
Find CMSIS Definition
       ↓
Write C Code
```

For example:

```text
RCC
 ↓
AHB1ENR
 ↓
GPIOAEN
 ↓
RCC->AHB1ENR
```

Similarly:

```text
GPIOA
 ↓
MODER
 ↓
PA5 configuration
 ↓
GPIOA->MODER
```

And:

```text
GPIOA
 ↓
ODR
 ↓
Bit 5
 ↓
GPIOA->ODR
```

---

# Why CMSIS?

CMSIS makes register-level programming easier to read and maintain.

It provides:

* Predefined peripheral structures
* Register definitions
* Bit definitions
* Device-specific hardware mappings
* Easier register access
* Better code readability

It also removes the need to manually calculate peripheral base addresses and register offsets when accessing the registers. 

---

# Key Learning

The main concept is:

> **CMSIS does not replace register-level programming. It provides predefined structures and definitions that make register-level programming easier.**

The learning progression is:

```text
Bare-Metal Programming
        ↓
Understand Registers
        ↓
Understand Addresses & Bits
        ↓
CMSIS
        ↓
Use Predefined Register Structures
        ↓
Cleaner Register-Level Programming
```

---

# Next Step – GPIO Input

After controlling an LED as an output, the next step is to learn how to work with a GPIO input.

The next exercise will follow:

```text
Button Press
     ↓
GPIO Input
     ↓
Read Pin State
     ↓
Process Input
     ↓
Control LED
```

This will introduce GPIO input handling and reading the state of a peripheral pin. The source material identifies button-controlled LED operation as the next step after the CMSIS LED example. 


* [ ] DMA
* [ ] RTOS
