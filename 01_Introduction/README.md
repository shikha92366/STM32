# 🚀 STM32 Basics

> A beginner-friendly yet industry-oriented introduction to STM32 microcontrollers, ARM Cortex-M architecture, memory organization, clock systems, software layers, and embedded development workflow.

---

## 📚 Table of Contents

- [What is STM32?](#-what-is-stm32)
- [Why Learn STM32?](#-why-learn-stm32)
- [STM32 Family Overview](#-stm32-family-overview)
- [ARM Cortex-M Architecture](#-arm-cortex-m-architecture)
- [STM32 Internal Architecture](#-stm32-internal-architecture)
- [Memory Organization](#-memory-organization)
- [Registers](#-registers)
- [STM32 Software Layers](#-stm32-software-layers)
- [Boot Process](#-boot-process)
- [Clock System & RCC](#-clock-system--rcc)
- [Development Workflow](#-development-workflow)
- [Important Documents](#-important-documents)
- [Interview Questions](#-interview-questions)
- [Revision Summary](#-revision-summary)

---

# 🎯 What is STM32?

STM32 is a family of **32-bit ARM Cortex-M microcontrollers** developed by **STMicroelectronics**.

The name STM32 means:

```text
ST   → STMicroelectronics
M32  → 32-bit Microcontroller
```

STM32 devices are widely used in:

- 🚗 Automotive Systems
- ✈️ Aerospace Electronics
- 🤖 Robotics
- 🏭 Industrial Automation
- 🏥 Medical Equipment
- 🌐 IoT Devices
- 📱 Consumer Electronics

---

# 🔥 Why Learn STM32?

STM32 is one of the most demanded platforms in Embedded Systems.

Most Embedded Software Engineer positions require:

- Embedded C
- STM32
- UART
- SPI
- I2C
- ADC
- Timers
- Interrupts
- FreeRTOS

---

# 🏗 STM32 Family Overview

| Series | Core | Application |
|----------|----------|----------|
| STM32F0 | Cortex-M0 | Entry Level |
| STM32F1 | Cortex-M3 | General Purpose |
| STM32F4 | Cortex-M4 | High Performance |
| STM32H7 | Cortex-M7 | Ultra High Performance |

---

## Popular STM32 Boards

### Blue Pill

```text
STM32F103C8T6
```

### NUCLEO-F401RE

```text
STM32F401RE
```

### Discovery Boards

Used for advanced development and debugging.

---

# ⚙ ARM Cortex-M Architecture

STM32 microcontrollers are built around ARM Cortex-M processors.

Examples:

```text
Cortex-M0
Cortex-M3
Cortex-M4
Cortex-M7
Cortex-M33
```

---

## Cortex-M4 Features

Used in STM32F401RE

```text
32-bit CPU
DSP Instructions
Interrupt Controller
Hardware Multiplier
Floating Point Unit (FPU)
```

---

# 🧠 STM32 Internal Architecture

```text
                CPU
                 │
                 ▼
           System Bus
                 │
 ┌───────────────┼───────────────┐
 ▼               ▼               ▼

Flash          SRAM        Peripherals

                           GPIO
                           UART
                           SPI
                           I2C
                           ADC
                           TIMERS
```

---

# 💾 Memory Organization

STM32 memory map:

```text
0x08000000 → Flash Memory

0x20000000 → SRAM

0x40000000 → Peripheral Registers
```

---

## Flash Memory

Stores:

- Program Code
- Constants

Example:

```text
512 KB Flash
```

---

## SRAM

Stores:

- Variables
- Stack
- Heap

Example:

```text
96 KB SRAM
```

---

# 📝 Registers

Registers are memory locations used to control hardware peripherals.

Examples:

```text
MODER
ODR
IDR
BSRR
PUPDR
```

---

## Register Example

Set PA5 HIGH:

```c
GPIOA->ODR |= (1 << 5);
```

Clear PA5:

```c
GPIOA->ODR &= ~(1 << 5);
```

Toggle PA5:

```c
GPIOA->ODR ^= (1 << 5);
```

---

# 🏛 STM32 Software Layers

```text
Application
     │
     ▼
HAL
     │
     ▼
LL Drivers
     │
     ▼
CMSIS
     │
     ▼
Registers
     │
     ▼
Hardware
```

---

## HAL

Hardware Abstraction Layer

Example:

```c
HAL_GPIO_WritePin(
    GPIOA,
    GPIO_PIN_5,
    GPIO_PIN_SET
);
```

### Advantages

- Easy
- Beginner Friendly
- Fast Development

---

## LL Drivers

Low Layer Drivers

Example:

```c
LL_GPIO_SetOutputPin(
    GPIOA,
    LL_GPIO_PIN_5
);
```

---

## Register Level

Example:

```c
GPIOA->ODR |= (1<<5);
```

Fastest and most efficient.

---

# 🚀 Boot Process

What happens after Power ON?

```text
Power ON
   ↓
Reset
   ↓
Bootloader
   ↓
Startup File
   ↓
main()
```

---

## Startup File Responsibilities

```text
Initialize Stack
Initialize Variables
Initialize BSS Section
Call main()
```

---

# ⏰ Clock System & RCC

CPU cannot work without a clock.

Think of the clock as the heartbeat of STM32.

---

## Clock Sources

### HSI

```text
High Speed Internal Oscillator
```

Typically:

```text
16 MHz
```

---

### HSE

```text
High Speed External Oscillator
```

Typically:

```text
8 MHz Crystal
```

---

### PLL

Phase Locked Loop

Used to increase frequency.

Example:

```text
8 MHz
  ↓
 PLL
  ↓
84 MHz
```

---

## RCC

Reset and Clock Control

Responsible for:

- Clock Generation
- Clock Distribution
- Peripheral Clock Enable
- Peripheral Reset

---

## Golden Rule

Before using any peripheral:

```text
Enable Clock
      ↓
Configure Peripheral
      ↓
Initialize Peripheral
      ↓
Use Peripheral
```

---

# 🛠 Development Workflow

```text
Read Datasheet
      ↓
Read Reference Manual
      ↓
Create Project
      ↓
Configure CubeMX
      ↓
Generate Code
      ↓
Write Application
      ↓
Debug
      ↓
Flash
      ↓
Test
```

---

# 📖 Important Documents

## Datasheet

Contains:

- Pinout
- Memory Size
- Electrical Characteristics
- Peripheral Availability

---

## Reference Manual

Contains:

- Registers
- Bit Definitions
- Peripheral Architecture

⭐ Most Important STM32 Document

---

## User Manual

Contains:

- Board Layout
- LED Connections
- Button Connections
- ST-Link Information

---

# 💡 Teaching Notes

### Common Beginner Mistake #1

Thinking STM32 and Arduino are the same.

They are not.

- Arduino → Development Platform
- STM32 → Microcontroller Family

### Common Beginner Mistake #2

Forgetting to enable peripheral clocks.

Without clocks:

```text
Peripheral Will Not Work
```

### Common Beginner Mistake #3

Learning only HAL.

Professional Engineers understand:

```text
HAL
LL
Registers
```

---

# 🎓 Interview Questions

### What is STM32?

STM32 is a family of 32-bit ARM Cortex-M based microcontrollers developed by STMicroelectronics.

### What is ARM?

ARM designs processor architectures licensed by semiconductor companies.

### Difference Between Flash and SRAM?

| Flash | SRAM |
|---------|---------|
| Non-Volatile | Volatile |
| Stores Code | Stores Variables |
| Retains Data | Loses Data After Power Off |

### What is RCC?

Reset and Clock Control peripheral responsible for clock management.

### What is a Register?

A memory location used to control hardware peripherals.

---

# 📌 Revision Summary

```text
STM32 = 32-bit ARM MCU

ARM Cortex-M = CPU Core

Flash = Program Storage

SRAM = Variable Storage

Registers = Hardware Control

RCC = Clock Management

HAL = Easy

LL = Medium

Register Programming = Advanced

Clock Required For Everything

Enable Clock First
```

---

# ✅ Progress Tracker

- [x] STM32 Introduction
- [x] STM32 Families
- [x] ARM Cortex-M
- [x] Internal Architecture
- [x] Memory Organization
- [x] Registers
- [x] Software Layers
- [x] Boot Process
- [x] Clock System
- [x] RCC
- [x] Development Workflow
- [x] Important Documents

---

## 🚀 Next Chapter

➡️ **GPIO (General Purpose Input Output)**

Learn:

- GPIO Registers
- GPIO Modes
- Push-Pull vs Open-Drain
- Pull-Up & Pull-Down
- LED Blinking
- Button Interfacing
- Register-Level GPIO Programming

⭐ If this repository helps you learn STM32, consider giving it a Star!
