Here's a professional **README.md** for your next STM32 learning repository topic. You can copy and paste it directly into GitHub.

````markdown
# 06_SFR_Debugging

## 📌 Overview

This section focuses on **SFR (Special Function Register) Debugging** in STM32 using STM32CubeIDE.

In register-level programming, we directly modify microcontroller registers to configure peripherals and control hardware. SFR debugging helps us monitor these registers and observe how their values change during program execution.

This practical demonstrates how to debug GPIO register operations and verify LED control at the hardware register level.

---

## 🎯 Learning Objectives

- Understand SFR (Special Function Registers).
- Learn how to open and use the SFR window in STM32CubeIDE.
- Monitor STM32 peripheral register values.
- Understand RCC_AHB1ENR register.
- Configure GPIO pins using the MODER register.
- Control GPIO output using the ODR register.
- Observe register changes during step-by-step debugging.
- Connect register-level programming with the STM32 reference manual.

---

## 🛠️ Software and Hardware

### Software
- STM32CubeIDE
- STM32 Debugger
- STM32 Reference Manual

### Hardware
- STM32F4 Microcontroller
- NUCLEO-F401RE Development Board
- On-board LED (PA5)

---

## 📚 Key Concepts

### 1. SFR (Special Function Register)

SFRs are registers used to control and configure microcontroller peripherals.

The SFR window allows us to monitor register values during debugging.

---

### 2. RCC_AHB1ENR Register

The RCC_AHB1ENR register controls clock enable for peripherals connected to the AHB1 bus.

For GPIOA:

| Bit | Name | Description |
|-----|------|-------------|
| 0 | GPIOAEN | GPIOA Clock Enable |

```c
RCC->AHB1ENR |= (1 << 0);
````

This enables the clock for GPIOA.

---

### 3. GPIOA_MODER Register

The MODER register configures the operating mode of GPIO pins.

Each GPIO pin uses two bits.

| Bits | Mode                   |
| ---- | ---------------------- |
| 00   | Input                  |
| 01   | General Purpose Output |
| 10   | Alternate Function     |
| 11   | Analog                 |

For PA5:

```c
GPIOA->MODER &= ~(3 << (5 * 2));
GPIOA->MODER |= (1 << (5 * 2));
```

This configures PA5 as a general-purpose output.

---

### 4. GPIOA_ODR Register

The Output Data Register (ODR) controls the output logic state of GPIO pins.

```c
GPIOA->ODR |= (1 << 5);
```

This sets PA5 HIGH.

| Bit 5 | Output |
| ----- | ------ |
| 0     | LOW    |
| 1     | HIGH   |

---

## 🔍 SFR Debugging Workflow

1. Open the STM32 project in STM32CubeIDE.

2. Start Debug Mode.

3. Open the SFR window:

   `Window → Show View → SFR`

4. Locate the RCC and GPIOA register groups.

5. Execute the program step by step.

6. Observe register value changes.

7. Verify GPIO configuration and LED output.

---

## 📊 Register Changes During Debugging

| Step | Register    | Expected Change   |
| ---- | ----------- | ----------------- |
| 1    | RCC_AHB1ENR | GPIOAEN: 0 → 1    |
| 2    | GPIOA_MODER | PA5 Mode: 00 → 01 |
| 3    | GPIOA_ODR   | Bit 5: 0 → 1      |

---

## 🧠 Important Learning

SFR debugging helps connect:

**C Code → Register Operations → Hardware Behavior**

By observing register values, we can understand how register-level programming affects the microcontroller.

This is useful for debugging embedded systems without relying entirely on HAL libraries.

---

## 📖 Reference Documents

* STM32F401xD/E Datasheet
* STM32F4 Reference Manual (RM0368)
* NUCLEO-F401RE User Manual (UM1724)
* STM32CubeIDE Documentation

---

## 🚀 Next Steps

* Explore GPIOx_IDR (Input Data Register).
* Understand BSRR for atomic GPIO operations.
* Compare HAL-based programming with register-level programming.
* Practice debugging RCC and GPIO registers.

````
