# STM32 Documentation

This section contains the essential documentation required for
understanding and developing applications for the STM32F401RE
microcontroller and the NUCLEO-F401RE development board.

Before starting peripheral-level programming, it is recommended to
become familiar with these documents.

---

## 1. STM32F401xD/E Datasheet

The **STM32F401xD/E Datasheet** provides information specific to the
STM32F401RE microcontroller.

It contains:

- MCU pinout
- Memory organization
- Electrical characteristics
- Power supply requirements
- GPIO and peripheral availability
- Package information
- Absolute maximum ratings

**Use the datasheet when you need to know:**

> What pins, memory, peripherals, and electrical specifications does
> the STM32F401RE have?

---

## 2. STM32F4 Reference Manual – RM0368

The **STM32F4 Reference Manual (RM0368)** provides detailed information
about how the STM32F401/F411 microcontroller peripherals work.

It covers:

- Memory and bus architecture
- Reset and Clock Control (RCC)
- GPIO
- External interrupts
- Timers
- USART/UART
- SPI
- I²C
- ADC
- DMA
- Power control
- Other STM32 peripherals

**Use the Reference Manual when you need to know:**

> How does a particular peripheral work and which registers do I need
> to configure?

This document is especially important for **register-level programming**.

---

## 3. NUCLEO-F401RE User Manual – UM1724

The **NUCLEO-F401RE User Manual (UM1724)** describes the development
board rather than the STM32F401RE MCU itself.

It contains information about:

- Board layout
- ST-LINK/V2-1
- USB interface
- Power supply
- User LED
- User push button
- Arduino connectors
- ST morpho connectors
- Board configuration
- Pin connections

**Use the Nucleo User Manual when you need to know:**

> How is the STM32F401RE connected to the NUCLEO-F401RE development
> board?

---

## Quick Reference

| Document | Main Purpose |
|---|---|
| **STM32F401xD/E Datasheet** | MCU specifications, pins, memory and electrical characteristics |
| **STM32F4 Reference Manual (RM0368)** | Peripheral operation and register-level configuration |
| **NUCLEO-F401RE User Manual (UM1724)** | Development board hardware, connections and configuration |

---

## Recommended Reading Order

Follow the documentation in this order:

### 1. STM32F401xD/E Datasheet
Understand the MCU specifications, pinout, memory, peripherals, and electrical characteristics.

⬇️

### 2. STM32F4 Reference Manual (RM0368)
Understand how the peripherals work and how they are configured through registers.

⬇️

### 3. NUCLEO-F401RE User Manual (UM1724)
Understand the development board, ST-LINK, connectors, power, LEDs, buttons, and pin connections.

⬇️

### 4. STM32 Peripheral Programming
Apply the information from the above documents while programming STM32 peripherals.
