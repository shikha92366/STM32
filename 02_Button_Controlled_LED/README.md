

# 🔘 Button Controlled LED using STM32 HAL

> A beginner STM32 GPIO project demonstrating how to read a digital input
> from a push button and control an LED using the STM32 HAL library.

---

# 📚 1. Concept

## GPIO Input and GPIO Output

GPIO stands for:

**General Purpose Input/Output**

A GPIO pin can be configured as either:

- **Input** → to read a signal
- **Output** → to generate/control a signal

In this project:

🔵 Blue User Button → GPIO Input
🟢 Green LED        → GPIO Output
````

The STM32 continuously reads the state of the blue button and controls the
green LED according to the value read from the GPIO pin.

---

## 🎯 Objective

The main objective of this project is to understand:

* GPIO input configuration
* GPIO output configuration
* Reading a GPIO pin using STM32 HAL
* Writing to a GPIO pin using STM32 HAL
* `GPIO_PinState`
* `GPIO_PIN_SET`
* `GPIO_PIN_RESET`
* `if-else` logic in embedded C
* Polling a GPIO input
* Basic input-output interfacing

---

## 🔄 Basic Embedded System Flow

```text
        INPUT
          │
          ▼
    🔵 Push Button
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

The basic pattern is:

```text
Read Input
    ↓
Process / Make Decision
    ↓
Control Output
```

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

| Device         | STM32 Pin | Configuration | User Label    |
| -------------- | --------- | ------------- | ------------- |
| LD2 Green LED  | PA5       | GPIO Output   | `MY_LED`      |
| B1 Blue Button | PC13      | GPIO Input    | `BLUE_BUTTON` |

---

## 📍 Pin Mapping

```text
                STM32F401RE
             ┌──────────────┐
             │              │
🔵 B1 Button ─────► PC13    │
             │              │
             │              │
             │       PA5 ───────► 🟢 LD2 LED
             │              │
             └──────────────┘
```

---

## STM32CubeMX Configuration

### LED

Configure:

```text
PA5 → GPIO_Output
```

User Label:

```text
MY_LED
```

---

### Blue Button

Configure:

```text
PC13 → GPIO_Input
```

User Label:

```text
BLUE_BUTTON
```

---

## Configuration Flow

```text
Create STM32 Project
        ↓
Select STM32F401RE
        ↓
Configure PA5
        ↓
PA5 → GPIO Output
        ↓
Configure PC13
        ↓
PC13 → GPIO Input
        ↓
Add User Labels
        ↓
Generate Code
```

---

# 💻 3. Code

## Main Application Code

The GPIO logic is placed inside the infinite `while(1)` loop.

```c
while (1)
{
    if (HAL_GPIO_ReadPin(BLUE_BUTTON_GPIO_Port, BLUE_BUTTON_Pin))
    {
        HAL_GPIO_WritePin(MY_LED_GPIO_Port, MY_LED_Pin, GPIO_PIN_RESET);
    }
    else
    {
        HAL_GPIO_WritePin(MY_LED_GPIO_Port, MY_LED_Pin, GPIO_PIN_SET);
    }
}
```

---

# 🧠 4. Explanation

## 4.1 `while(1)`

```c
while (1)
```

The `while(1)` loop runs continuously.

The STM32 repeatedly:

```text
Read Button
    ↓
Check Button State
    ↓
Control LED
    ↓
Repeat
```

This means the button is being checked continuously.

This method of repeatedly checking an input is called **polling**.

---

# 4.2 `HAL_GPIO_ReadPin()`

The button is read using:

```c
HAL_GPIO_ReadPin(BLUE_BUTTON_GPIO_Port, BLUE_BUTTON_Pin)
```

The function returns a value of type:

```c
GPIO_PinState
```

The GPIO pin state can be:

```text
GPIO_PIN_RESET
GPIO_PIN_SET
```

The HAL defines `GPIO_PinState` using an enum similar to:

```c
typedef enum
{
    GPIO_PIN_RESET = 0U,
    GPIO_PIN_SET
} GPIO_PinState;
```

Therefore:

```text
GPIO_PIN_RESET → 0
GPIO_PIN_SET   → 1
```

---

# 4.3 Why the `if` works without `== GPIO_PIN_SET`

The code uses:

```c
if (HAL_GPIO_ReadPin(BLUE_BUTTON_GPIO_Port, BLUE_BUTTON_Pin))
```

instead of:

```c
if (HAL_GPIO_ReadPin(BLUE_BUTTON_GPIO_Port, BLUE_BUTTON_Pin)
    == GPIO_PIN_SET)
```

Both approaches can be used here.

The reason the first version works is because of how C evaluates values
inside an `if` statement.

In C:

```text
0       → FALSE
Non-zero → TRUE
```

Since:

```text
GPIO_PIN_RESET = 0
GPIO_PIN_SET   = 1
```

the following happens:

```text
HAL_GPIO_ReadPin()
        │
        ├── GPIO_PIN_SET (1)
        │       ↓
        │     TRUE
        │       ↓
        │      IF
        │
        └── GPIO_PIN_RESET (0)
                ↓
              FALSE
                ↓
               ELSE
```

---

# 4.4 Understanding the `if` Block

The code is:

```c
if (HAL_GPIO_ReadPin(BLUE_BUTTON_GPIO_Port, BLUE_BUTTON_Pin))
{
    HAL_GPIO_WritePin(MY_LED_GPIO_Port, MY_LED_Pin, GPIO_PIN_RESET);
}
```

If the button reading is non-zero:

```text
Button State
     ↓
GPIO_PIN_SET
     ↓
1
     ↓
TRUE
     ↓
IF block executes
     ↓
LED = GPIO_PIN_RESET
```

So the LED is written with:

```c
GPIO_PIN_RESET
```

---

# 4.5 Understanding the `else` Block

The code is:

```c
else
{
    HAL_GPIO_WritePin(MY_LED_GPIO_Port, MY_LED_Pin, GPIO_PIN_SET);
}
```

If the button reading is zero:

```text
Button State
     ↓
GPIO_PIN_RESET
     ↓
0
     ↓
FALSE
     ↓
ELSE block executes
     ↓
LED = GPIO_PIN_SET
```

So the LED is written with:

```c
GPIO_PIN_SET
```

---

# 4.6 Complete Logic

Your program therefore follows this logic:

```text
                 Read PC13
                     │
                     ▼
              Is value non-zero?
                /           \
              YES            NO
               │              │
               ▼              ▼
        LED = RESET      LED = SET
               │              │
               ▼              ▼
            LED OFF          LED ON
```

In table form:

| Button GPIO State    | C `if` Result | LED Output       | LED |
| -------------------- | ------------- | ---------------- | --- |
| `GPIO_PIN_SET` (1)   | TRUE          | `GPIO_PIN_RESET` | OFF |
| `GPIO_PIN_RESET` (0) | FALSE         | `GPIO_PIN_SET`   | ON  |

---

# 4.7 `HAL_GPIO_WritePin()`

The LED is controlled using:

```c
HAL_GPIO_WritePin(MY_LED_GPIO_Port,
                  MY_LED_Pin,
                  GPIO_PIN_RESET);
```

The function takes three important arguments:

```text
HAL_GPIO_WritePin(
        GPIO Port,
        GPIO Pin,
        Pin State
);
```

---

## First Argument – GPIO Port

```c
MY_LED_GPIO_Port
```

This tells HAL which GPIO port contains the LED.

For PA5, this corresponds to:

```text
GPIOA
```

---

## Second Argument – GPIO Pin

```c
MY_LED_Pin
```

This identifies the particular GPIO pin.

For the onboard LED:

```text
PA5
```

---

## Third Argument – Pin State

The third argument specifies the state to write:

```c
GPIO_PIN_SET
```

or:

```c
GPIO_PIN_RESET
```

---

# 4.8 `GPIO_PinState`

`GPIO_PinState` is a type defined using an enumeration.

Conceptually:

```c
typedef enum
{
    GPIO_PIN_RESET = 0U,
    GPIO_PIN_SET
} GPIO_PinState;
```

So:

```text
GPIO_PinState
      │
      ├── GPIO_PIN_RESET
      │       = 0
      │
      └── GPIO_PIN_SET
              = 1
```

This is why the same values can be used both when reading and writing
GPIO states.

---

# 4.9 Read vs Write

The two main HAL functions used in this project have different jobs.

### Reading the Button

```c
HAL_GPIO_ReadPin(
    BLUE_BUTTON_GPIO_Port,
    BLUE_BUTTON_Pin
);
```

Meaning:

```text
"Tell me the current state of this pin."
```

---

### Controlling the LED

```c
HAL_GPIO_WritePin(
    MY_LED_GPIO_Port,
    MY_LED_Pin,
    GPIO_PIN_SET
);
```

Meaning:

```text
"Set this output pin."
```

---

# 4.10 Complete Input → Output Relationship

```text
          BUTTON
            │
            ▼
           PC13
            │
            ▼
  HAL_GPIO_ReadPin()
            │
            ▼
      GPIO_PinState
            │
       ┌────┴────┐
       │         │
      SET      RESET
       │         │
       ▼         ▼
      TRUE      FALSE
       │         │
       ▼         ▼
 LED = RESET  LED = SET
       │         │
       ▼         ▼
    LED OFF    LED ON
```

---

# 🧪 5. Result

After building and flashing the program to the STM32 NUCLEO-F401RE:

```text
Button State → LED Response
```

The program continuously reads the button and updates the LED.

### When the GPIO reads `GPIO_PIN_SET`

```text
PC13
 ↓
GPIO_PIN_SET
 ↓
if = TRUE
 ↓
PA5 = GPIO_PIN_RESET
 ↓
LED OFF
```

### When the GPIO reads `GPIO_PIN_RESET`

```text
PC13
 ↓
GPIO_PIN_RESET
 ↓
if = FALSE
 ↓
PA5 = GPIO_PIN_SET
 ↓
LED ON
```

---

# 🧠 6. Key Learning

This project introduced the basic **GPIO input-output relationship**.

The important pattern is:

```text
INPUT
  ↓
READ
  ↓
PROCESS
  ↓
WRITE
  ↓
OUTPUT
```

In this project:

```text
Button
  ↓
PC13
  ↓
HAL_GPIO_ReadPin()
  ↓
GPIO_PinState
  ↓
if-else
  ↓
HAL_GPIO_WritePin()
  ↓
PA5
  ↓
LED
```

---

## Important C Concept Learned

The following:

```c
if (HAL_GPIO_ReadPin(...))
```

works because C evaluates:

```text
0       → FALSE
non-zero → TRUE
```

Since:

```text
GPIO_PIN_RESET = 0
GPIO_PIN_SET   = 1
```

the returned GPIO state can directly be used as the condition.

---

## Important HAL Concepts Learned

### Read GPIO

```c
HAL_GPIO_ReadPin();
```

### Write GPIO

```c
HAL_GPIO_WritePin();
```

### GPIO State

```c
GPIO_PIN_SET
GPIO_PIN_RESET
```

### GPIO State Type

```c
GPIO_PinState
```

---

# 📌 7. HAL vs Bare Metal

In this project, GPIO is controlled through the STM32 HAL.

```text
Application Code
       ↓
STM32 HAL API
       ↓
HAL GPIO Driver
       ↓
GPIO Registers
       ↓
STM32 Hardware
```

For example:

```c
HAL_GPIO_ReadPin();
```

hides the direct register-level operations from the application.

Similarly:

```c
HAL_GPIO_WritePin();
```

provides an abstraction for controlling the GPIO hardware.

---

# 🛡️ 8. USER CODE Sections

STM32CubeMX generates a large portion of the STM32 project automatically.

Custom application code should be placed inside the appropriate USER CODE
sections.

For example:

```c
/* USER CODE BEGIN WHILE */

while (1)
{
    if (HAL_GPIO_ReadPin(BLUE_BUTTON_GPIO_Port, BLUE_BUTTON_Pin))
    {
        HAL_GPIO_WritePin(MY_LED_GPIO_Port, MY_LED_Pin, GPIO_PIN_RESET);
    }
    else
    {
        HAL_GPIO_WritePin(MY_LED_GPIO_Port, MY_LED_Pin, GPIO_PIN_SET);
    }
}

/* USER CODE END WHILE */
```

Using the USER CODE sections helps protect custom code when STM32CubeMX
regenerates the project.

---

# 📂 9. Project Structure

```text
02_Button_Controlled_LED_HAL/
│
├── README.md
│
└── STM32_Project/
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

### Important Files

| File/Folder | Purpose                       |
| ----------- | ----------------------------- |
| `main.c`    | Main application logic        |
| `main.h`    | GPIO pin and port definitions |
| `.ioc`      | STM32CubeMX configuration     |
| `Drivers/`  | HAL and CMSIS drivers         |

---

# 🚀 10. How to Run

1. Open the project in STM32CubeIDE.
2. Open the `.ioc` file.
3. Configure PA5 as `GPIO_Output`.
4. Configure PC13 as `GPIO_Input`.
5. Set the appropriate user labels.
6. Generate the code.
7. Add the application logic inside the USER CODE section.
8. Build the project.
9. Connect the NUCLEO-F401RE through USB.
10. Flash the program using ST-LINK.
11. Observe the LED while changing the button state.

---

# 📌 11. Project Summary

| Category        | Details                          |
| --------------- | -------------------------------- |
| Board           | STM32 NUCLEO-F401RE              |
| MCU             | STM32F401RE                      |
| Language        | Embedded C                       |
| Framework       | STM32 HAL                        |
| Input           | B1 Blue User Button              |
| Input Pin       | PC13                             |
| Output          | LD2 Green LED                    |
| Output Pin      | PA5                              |
| GPIO Input API  | `HAL_GPIO_ReadPin()`             |
| GPIO Output API | `HAL_GPIO_WritePin()`            |
| GPIO State Type | `GPIO_PinState`                  |
| GPIO States     | `GPIO_PIN_SET`, `GPIO_PIN_RESET` |
| Input Method    | Polling                          |

---

# 📖 12. What I Learned

From this project, I learned how to:

* Configure a GPIO as an input.
* Configure a GPIO as an output.
* Read the state of a button using HAL.
* Control an LED using HAL.
* Understand `GPIO_PinState`.
* Understand `GPIO_PIN_SET` and `GPIO_PIN_RESET`.
* Use GPIO state directly inside an `if` condition.
* Understand the relationship between `0`/`1` and `FALSE`/`TRUE` in C.
* Implement basic GPIO polling.
* Understand the Input → Process → Output model.

---

# ➡️ 13. Next Concept

## Bare-Metal / Register-Level GPIO

The next step is to implement the same GPIO functionality **without using
the HAL GPIO APIs**.

Currently:

```text
Application
     ↓
HAL_GPIO_ReadPin()
HAL_GPIO_WritePin()
     ↓
HAL Driver
     ↓
GPIO Registers
     ↓
Hardware
```

Next:

```text
Application
     ↓
GPIO Registers
     ↓
Hardware
```

The next project will focus on understanding the GPIO registers directly.

### Topics to Learn

* GPIO registers
* GPIO port registers
* GPIO mode configuration
* Input Data Register
* Output Data Register
* Register-level bit manipulation
* Reading GPIO registers
* Writing GPIO registers
* HAL vs Bare-Metal programming

---

## 🔜 Next Project

```text
03_Button_Controlled_LED_Bare_Metal
```

The goal is not just to make the LED work again.

The goal is to understand:

> **What is actually happening inside the STM32 when a GPIO pin is read and
> written?**

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

````

### One thing to remember

Your **actual code** is:

```c
if (HAL_GPIO_ReadPin(BLUE_BUTTON_GPIO_Port, BLUE_BUTTON_Pin))
````

and not:

```c
if (HAL_GPIO_ReadPin(...) == GPIO_PIN_SET)
```

I've explained the exact reason for that in the README under **“Why the `if` works without `== GPIO_PIN_SET`”**. That is worth keeping because it demonstrates that you're learning **C behavior + STM32 HAL**, rather than simply copying HAL code.
