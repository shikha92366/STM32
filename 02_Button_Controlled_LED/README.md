
# 🔘 Button Controlled LED using STM32 HAL

> A beginner-friendly STM32 GPIO project demonstrating digital input and
> output interfacing using the STM32 HAL (Hardware Abstraction Layer).

---

## 📚 1. Concept

### GPIO Input + GPIO Output

GPIO stands for:

**General Purpose Input/Output**

A GPIO pin can be configured as:

- Input
- Output

In this project:

```text
Blue Button → GPIO Input
Green LED   → GPIO Output
````

The STM32 continuously reads the state of the blue user button and controls
the green LED according to the button state.

### Basic Input-Output Flow

```text
        INPUT
          │
          ▼
    🔵 Blue Button
          │
          ▼
        PC13
          │
          ▼
      STM32 MCU
          │
          ▼
         PA5
          │
          ▼
     🟢 Green LED
        OUTPUT
```

### Core Embedded Concept

```text
Read Input
    ↓
Process / Make Decision
    ↓
Control Output
```

This is one of the fundamental patterns used in embedded systems.

---

## 🎯 Objective

The objective of this project is to learn:

* GPIO input configuration
* GPIO output configuration
* Digital input reading
* Digital output control
* STM32 HAL GPIO APIs
* `if-else` logic for hardware control
* STM32CubeMX GPIO configuration
* Basic input-output interfacing

---

# ⚙️ 2. Configuration

## 🛠️ Hardware Used

| Component           | Details                       |
| ------------------- | ----------------------------- |
| Development Board   | STM32 NUCLEO-F401RE           |
| Microcontroller     | STM32F401RE                   |
| LED                 | LD2 – Onboard Green LED       |
| Push Button         | B1 – Onboard Blue User Button |
| Programmer/Debugger | ST-LINK                       |
| IDE                 | STM32CubeIDE                  |

---

## 🔌 GPIO Pin Configuration

| Device         | STM32 Pin | GPIO Mode   | User Label    |
| -------------- | --------- | ----------- | ------------- |
| LD2 Green LED  | PA5       | GPIO Output | `MY_LED`      |
| B1 Blue Button | PC13      | GPIO Input  | `BLUE_BUTTON` |

---

## 📍 Pin Mapping

```text
              STM32F401RE
           ┌───────────────┐
           │               │
B1 Button ─┤ PC13          │
           │               │
           │               │
           │          PA5 ─┼────► LD2 LED
           │               │
           └───────────────┘
```

---

## 🧩 STM32CubeMX Configuration

### Step 1 – Configure LED

Select:

```text
PA5 → GPIO_Output
```

Set the User Label:

```text
MY_LED
```

---

### Step 2 – Configure Blue Button

Select:

```text
PC13 → GPIO_Input
```

Set the User Label:

```text
BLUE_BUTTON
```

---

### Step 3 – Generate Code

After configuring the GPIO pins:

```text
Open STM32CubeIDE
        ↓
Create STM32 Project
        ↓
Select STM32F401RE
        ↓
Configure PA5 as GPIO Output
        ↓
Configure PC13 as GPIO Input
        ↓
Add User Labels
        ↓
Save .ioc
        ↓
Generate Code
```

---

# 💻 3. Code

## GPIO Definitions

STM32CubeMX generates GPIO definitions in `main.h`.

They will look similar to:

```c
#define MY_LED_Pin GPIO_PIN_5
#define MY_LED_GPIO_Port GPIOA

#define BLUE_BUTTON_Pin GPIO_PIN_13
#define BLUE_BUTTON_GPIO_Port GPIOC
```

These definitions allow the application code to use meaningful names instead
of directly writing GPIO port and pin numbers.

---

## Main Application Code

The application logic is placed inside the `while(1)` loop.

```c
while (1)
{
    if (HAL_GPIO_ReadPin(BLUE_BUTTON_GPIO_Port, BLUE_BUTTON_Pin) == GPIO_PIN_SET)
    {
        HAL_GPIO_WritePin(MY_LED_GPIO_Port, MY_LED_Pin, GPIO_PIN_SET);
    }
    else
    {
        HAL_GPIO_WritePin(MY_LED_GPIO_Port, MY_LED_Pin, GPIO_PIN_RESET);
    }
}
```

---

## Complete Logic

```text
             START
               │
               ▼
       Read Button PC13
               │
               ▼
       Button Pressed?
          /          \
        YES           NO
         │             │
         ▼             ▼
      LED ON         LED OFF
         │             │
         └──────┬──────┘
                │
                ▼
          Repeat Forever
```

---

# 🧠 4. Explanation

## 4.1 `HAL_GPIO_ReadPin()`

```c
HAL_GPIO_ReadPin(BLUE_BUTTON_GPIO_Port, BLUE_BUTTON_Pin);
```

This function is used to read the state of the GPIO input.

In this project, it reads the blue button connected to:

```text
PC13
```

The returned GPIO state can be:

```text
GPIO_PIN_SET
GPIO_PIN_RESET
```

The result is checked using an `if` condition.

---

## 4.2 Checking the Button

```c
if (HAL_GPIO_ReadPin(BLUE_BUTTON_GPIO_Port, BLUE_BUTTON_Pin) == GPIO_PIN_SET)
```

This means:

```text
Read PC13
   ↓
Is the input SET?
   ↓
YES → Execute if block
NO  → Execute else block
```

---

## 4.3 `HAL_GPIO_WritePin()`

```c
HAL_GPIO_WritePin(MY_LED_GPIO_Port,
                  MY_LED_Pin,
                  GPIO_PIN_SET);
```

This function is used to control the state of a GPIO output.

In this project, the output is:

```text
PA5 → LD2 Green LED
```

---

## 4.4 Turning LED ON

```c
HAL_GPIO_WritePin(MY_LED_GPIO_Port,
                  MY_LED_Pin,
                  GPIO_PIN_SET);
```

This sets the LED GPIO output.

Conceptually:

```text
PA5 → SET
     ↓
LED → ON
```

---

## 4.5 Turning LED OFF

```c
HAL_GPIO_WritePin(MY_LED_GPIO_Port,
                  MY_LED_Pin,
                  GPIO_PIN_RESET);
```

This resets the LED GPIO output.

Conceptually:

```text
PA5 → RESET
     ↓
LED → OFF
```

---

# 🔄 4.6 Complete Working

### When the Button is Pressed

```text
🔵 Blue Button Pressed
          ↓
        PC13
          ↓
  HAL_GPIO_ReadPin()
          ↓
    GPIO_PIN_SET
          ↓
   if condition TRUE
          ↓
 HAL_GPIO_WritePin()
          ↓
        PA5 SET
          ↓
     🟢 LED ON
```

---

### When the Button is Released

```text
🔵 Blue Button Released
          ↓
        PC13
          ↓
  HAL_GPIO_ReadPin()
          ↓
   GPIO_PIN_RESET
          ↓
  if condition FALSE
          ↓
 HAL_GPIO_WritePin()
          ↓
       PA5 RESET
          ↓
      🟢 LED OFF
```

---

# 📚 4.7 HAL Functions Used

| HAL Function           | Purpose                      |
| ---------------------- | ---------------------------- |
| `HAL_GPIO_Init()`      | Initializes/configures GPIO  |
| `HAL_GPIO_ReadPin()`   | Reads a GPIO input           |
| `HAL_GPIO_WritePin()`  | Sets or resets a GPIO output |
| `HAL_GPIO_TogglePin()` | Toggles a GPIO output        |

The main application APIs used in this project are:

```c
HAL_GPIO_ReadPin()
HAL_GPIO_WritePin()
```

---

# 🏗️ 4.8 HAL Architecture

The application does not directly manipulate GPIO registers.

Instead, the application uses HAL functions.

```text
        Application
             │
             ▼
           main.c
             │
             ▼
        STM32 HAL
             │
       ┌─────┴─────┐
       ▼           ▼
Read GPIO      Write GPIO
       │           │
       ▼           ▼
  GPIO Peripheral
             │
             ▼
       STM32 Hardware
```

The HAL provides an abstraction layer between the application and the
microcontroller hardware.

---

# 📂 4.9 Important Project Files

The STM32CubeIDE project contains generated files such as:

```text
STM32_Project/
│
├── Core/
│   ├── Inc/
│   │   └── main.h
│   │
│   └── Src/
│       └── main.c
│
├── Drivers/
│
└── STM32_Project.ioc
```

### `main.c`

Contains the main application code and program execution flow.

### `main.h`

Contains generated definitions such as GPIO pins and ports.

Example:

```c
#define MY_LED_Pin GPIO_PIN_5
#define MY_LED_GPIO_Port GPIOA
```

### `.ioc`

Contains the STM32CubeMX configuration.

### `Drivers/`

Contains the STM32 HAL and CMSIS driver files.

---

# 🛡️ 4.10 USER CODE Sections

STM32CubeMX automatically generates a large part of the project.

Custom application code should be placed inside the appropriate:

```c
/* USER CODE BEGIN */

/* USER CODE END */
```

sections.

For example:

```c
/* USER CODE BEGIN WHILE */

while (1)
{
    if (HAL_GPIO_ReadPin(BLUE_BUTTON_GPIO_Port, BLUE_BUTTON_Pin) == GPIO_PIN_SET)
    {
        HAL_GPIO_WritePin(MY_LED_GPIO_Port, MY_LED_Pin, GPIO_PIN_SET);
    }
    else
    {
        HAL_GPIO_WritePin(MY_LED_GPIO_Port, MY_LED_Pin, GPIO_PIN_RESET);
    }
}

/* USER CODE END WHILE */
```

This helps prevent custom application code from being lost when STM32CubeMX
regenerates the project.

---

# 🧪 5. Result

After building and flashing the program to the STM32 NUCLEO-F401RE:

### Button Released

```text
🔵 B1 → RELEASED

🟢 LD2 → OFF
```

### Button Pressed

```text
🔵 B1 → PRESSED

🟢 LD2 → ON
```

### Final Behavior

```text
Press Button
     ↓
LED ON

Release Button
     ↓
LED OFF
```

The button successfully controls the onboard LED.

---

# 🧠 6. Key Learning

This project introduced the basic:

```text
INPUT → PROCESS → OUTPUT
```

model used in embedded systems.

```text
       INPUT
         │
         ▼
   Read GPIO Pin
         │
         ▼
   Make a Decision
         │
         ▼
  Write GPIO Output
         │
         ▼
      OUTPUT
```

The STM32 continuously reads the hardware input and uses the result to
control the hardware output.

---

## Important Concepts Learned

### GPIO Input

A GPIO configured as an input allows the microcontroller to read a digital
signal.

Examples:

```text
Push Button
Switch
Digital Sensor
```

---

### GPIO Output

A GPIO configured as an output allows the microcontroller to generate a
digital signal.

Examples:

```text
LED
Buzzer
Relay
Control Signal
```

---

## HAL Learning

I learned how to use:

```c
HAL_GPIO_ReadPin()
```

to read a GPIO input and:

```c
HAL_GPIO_WritePin()
```

to control a GPIO output.

I also learned how STM32CubeMX generates the GPIO initialization and pin
definitions required by the HAL-based application.

---

# 🚀 7. How to Run

1. Open the project in STM32CubeIDE.
2. Open the `.ioc` file.
3. Verify `PA5` is configured as GPIO Output.
4. Verify `PC13` is configured as GPIO Input.
5. Verify the user labels.
6. Generate the code.
7. Add the application logic inside the USER CODE section.
8. Build the project.
9. Connect the NUCLEO-F401RE board through USB.
10. Flash the program using ST-LINK.
11. Press the blue B1 button.
12. Observe the LD2 green LED.

---

# 📌 8. Project Summary

| Category             | Details               |
| -------------------- | --------------------- |
| Microcontroller      | STM32F401RE           |
| Board                | NUCLEO-F401RE         |
| Programming Language | Embedded C            |
| Framework            | STM32 HAL             |
| Input                | B1 Blue Button        |
| Input Pin            | PC13                  |
| Output               | LD2 Green LED         |
| Output Pin           | PA5                   |
| HAL Input API        | `HAL_GPIO_ReadPin()`  |
| HAL Output API       | `HAL_GPIO_WritePin()` |
| IDE                  | STM32CubeIDE          |

---

# 📖 9. What I Learned from This Project

This project helped me move from controlling a simple GPIO output to
**interfacing a GPIO input with an output**.

The main learning flow was:

```text
GPIO Output
     ↓
GPIO Input
     ↓
Read Input
     ↓
Process Input
     ↓
Control Output
```

This forms the foundation for more advanced embedded concepts.

---

# ➡️ 10. Next Concept

## Bare-Metal / Register-Level GPIO

The next step is to implement the same Button + LED functionality **without
using HAL GPIO functions**.

Instead of:

```c
HAL_GPIO_ReadPin()
HAL_GPIO_WritePin()
```

the next project will directly work with STM32 GPIO registers.

### Current Approach – HAL

```text
Application
     ↓
HAL API
     ↓
GPIO Driver
     ↓
GPIO Registers
     ↓
Hardware
```

### Next Approach – Bare Metal

```text
Application
     ↓
GPIO Registers
     ↓
Hardware
```

The goal is to understand what happens **underneath the HAL abstraction**.

---

## 🔜 Next Project

```text
03_Button_Controlled_LED_Bare_Metal
```

Topics to learn next:

* STM32 GPIO registers
* GPIO port registers
* GPIO mode configuration
* Input data register
* Output data register
* Register-level bit manipulation
* Reading a GPIO register
* Writing to a GPIO register
* HAL vs Bare-Metal programming

---

# 📈 STM32 Learning Progress

```text
[✓] GPIO Output – LED Blink using HAL
[✓] GPIO Input + Output – Button Controlled LED using HAL
[ ] GPIO Output – Bare Metal
[ ] GPIO Input – Bare Metal
[ ] Button Interrupt
[ ] External LED
[ ] GPIO Register Programming
[ ] Timers
[ ] PWM
[ ] ADC
[ ] UART
[ ] SPI
[ ] I2C
[ ] CAN
[ ] FreeRTOS
```

---

# 📚 References

* STM32F401RE Datasheet
* STM32F4 Reference Manual
* STM32 NUCLEO-F401RE User Manual
* STM32 HAL GPIO Documentation
* STM32CubeIDE Documentation

