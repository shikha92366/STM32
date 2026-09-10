# 🔘 Button Controlled LED using STM32 HAL

A beginner-friendly STM32 project demonstrating how to use **GPIO Input and GPIO Output** with the **STM32 HAL (Hardware Abstraction Layer)**.

In this project, the onboard **blue user button** is used as an input, and the onboard **green LED** is controlled as an output.

The project helps understand:

* GPIO input
* GPIO output
* `HAL_GPIO_ReadPin()`
* `HAL_GPIO_WritePin()`
* `GPIO_PIN_SET`
* `GPIO_PIN_RESET`
* `GPIO_PinState`
* `if-else` logic
* Active-low button input
* Polling using `while(1)`

---

## 📚 1. Concept

### What is GPIO?

**GPIO** stands for **General Purpose Input/Output**.

A GPIO pin can generally be configured as:

* **Input** → read a signal from a button/sensor
* **Output** → control an LED, relay, etc.

In this project:

```text
Button → GPIO Input
LED    → GPIO Output
```

### Hardware Used

| Component           | Pin  | Configuration | Label         |
| ------------------- | ---- | ------------- | ------------- |
| 🔘 Blue User Button | PC13 | GPIO Input    | `BLUE_BUTTON` |
| 🟢 Green LED (LD2)  | PA5  | GPIO Output   | `MY_LED`      |

---

# ⚙️ 2. STM32CubeIDE Configuration

Create an STM32 project for the **STM32F401RE / NUCLEO-F401RE** board.

Configure the GPIO pins as follows:

### LED — PA5

```text
PA5
Mode: GPIO Output
User Label: MY_LED
```

### Button — PC13

```text
PC13
Mode: GPIO Input
User Label: BLUE_BUTTON
```

After configuring the pins, generate the project.

STM32CubeIDE will generate the required GPIO initialization code automatically.

---

# 💻 3. Code

The main logic is placed inside the `while(1)` loop:

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

# 🧠 4. Understanding the Code

## `HAL_GPIO_ReadPin()`

```c
HAL_GPIO_ReadPin(BLUE_BUTTON_GPIO_Port, BLUE_BUTTON_Pin)
```

This function **reads the current electrical state of the button pin PC13**.

It returns a `GPIO_PinState`.

The possible values are:

```c
GPIO_PIN_RESET = 0
GPIO_PIN_SET   = 1
```

So:

```text
0 → LOW
1 → HIGH
```

---

## `GPIO_PinState`

`GPIO_PinState` is the type used by STM32 HAL to represent the state of a GPIO pin.

The enum is:

```c
typedef enum
{
    GPIO_PIN_RESET = 0U,
    GPIO_PIN_SET
} GPIO_PinState;
```

Therefore:

```text
GPIO_PIN_RESET → 0 → LOW
GPIO_PIN_SET   → 1 → HIGH
```

### Important

`GPIO_PIN_SET` and `GPIO_PIN_RESET` describe the **electrical state of a GPIO pin**.

They do **not automatically mean**:

```text
SET   = Button Pressed
RESET = Button Released
```

The actual meaning depends on how the hardware is connected.

---

# 🔘 5. What Happens When the Button is Pressed?

The onboard blue button is connected to **PC13**.

When the button is **not pressed**, PC13 is HIGH:

```text
Not Pressed → PC13 = 1 → GPIO_PIN_SET
```

When the button is **pressed**, the button connects PC13 to **GND**.

Since GND is **0 V**, PC13 becomes LOW:

```text
Pressed → PC13 = 0 → GPIO_PIN_RESET
```

Therefore:

```text
Not Pressed → PC13 = 1
Pressed     → PC13 = 0
```

This is called an **active-low input**.

> **The button does not "produce 0". Pressing the button connects PC13 to GND, and GND is 0 V.**

---

# 🔄 6. Complete Program Flow

When the button is pressed:

```text
        🔘 BUTTON PRESSED
                ↓
            PC13 = 0
                ↓
     HAL_GPIO_ReadPin() reads 0
                ↓
           if(0) → FALSE
                ↓
            else runs
                ↓
     GPIO_PIN_SET is written to PA5
                ↓
             PA5 = 1
                ↓
          🟢 LED turns ON
```

When the button is released:

```text
       🔘 BUTTON RELEASED
                ↓
            PC13 = 1
                ↓
     HAL_GPIO_ReadPin() reads 1
                ↓
           if(1) → TRUE
                ↓
       GPIO_PIN_RESET written to PA5
                ↓
             PA5 = 0
                ↓
          🟢 LED turns OFF
```

### In short:

```text
Pressed
   ↓
PC13 = 0
   ↓
else
   ↓
PA5 = 1
   ↓
LED ON
```

```text
Released
   ↓
PC13 = 1
   ↓
if
   ↓
PA5 = 0
   ↓
LED OFF
```

---

# 🔌 7. `HAL_GPIO_WritePin()`

This function controls the output GPIO pin.

```c
HAL_GPIO_WritePin(MY_LED_GPIO_Port,
                  MY_LED_Pin,
                  GPIO_PIN_SET);
```

It means:

> Set the LED GPIO pin to HIGH.

And:

```c
HAL_GPIO_WritePin(MY_LED_GPIO_Port,
                  MY_LED_Pin,
                  GPIO_PIN_RESET);
```

means:

> Set the LED GPIO pin to LOW.

### Notice the difference

`HAL_GPIO_ReadPin()` → **READS the button**

`HAL_GPIO_WritePin()` → **WRITES to the LED**

```text
BUTTON
  ↓
PC13
  ↓
READ
  ↓
if / else
  ↓
WRITE
  ↓
PA5
  ↓
LED
```

We are **not resetting the button**.

We are setting/resetting the **LED pin** depending on the button state.

---

# 🧩 8. Why Does `if()` Work Here?

The code is:

```c
if (HAL_GPIO_ReadPin(BLUE_BUTTON_GPIO_Port, BLUE_BUTTON_Pin))
```

The function returns either:

```text
GPIO_PIN_RESET → 0
GPIO_PIN_SET   → 1
```

In C:

```text
0       → FALSE
non-zero → TRUE
```

Therefore:

```c
if (1)
```

is true.

And:

```c
if (0)
```

is false.

So the code can be understood as:

```c
if (button_state == 1)
{
    // LED LOW
}
else
{
    // LED HIGH
}
```

---

# 🔁 9. Why `while(1)`?

The code is inside:

```c
while (1)
{
    ...
}
```

`1` is always true, so the loop runs continuously.

The STM32 repeatedly checks the button:

```text
READ BUTTON
     ↓
CHECK STATE
     ↓
CONTROL LED
     ↓
READ BUTTON AGAIN
     ↓
CHECK STATE
     ↓
CONTROL LED
     ↓
...
```

This method of continuously checking the input is called **polling**.

---

# 🧪 10. Expected Result

### Button Released

```text
PC13 = 1
PA5  = 0
LED  = OFF
```

### Button Pressed

```text
PC13 = 0
PA5  = 1
LED  = ON
```

Therefore, the LED follows the button state:

```text
🔘 Released → 🟢 LED OFF

🔘 Pressed  → 🟢 LED ON
```

---

# 📝 11. Key Takeaways

### GPIO

GPIO allows the microcontroller to interact with external hardware.

### Input

The button is configured as an input because the STM32 needs to **read** its state.

### Output

The LED is configured as an output because the STM32 needs to **control** it.

### HAL GPIO Functions

```c
HAL_GPIO_ReadPin()
```

→ Reads a GPIO pin.

```c
HAL_GPIO_WritePin()
```

→ Sets or resets a GPIO output pin.

### GPIO States

```text
GPIO_PIN_RESET = 0 = LOW
GPIO_PIN_SET   = 1 = HIGH
```

### Active-Low Input

For this button:

```text
Pressed     → 0
Not Pressed → 1
```

### Main Concept

> **Read the button → make a decision → control the LED.**

---

# 📂 12. Project Structure

A typical STM32CubeIDE project contains:

```text
Button_Controlled_LED/
│
├── Core/
│   ├── Inc/
│   │   ├── main.h
│   │   └── stm32f4xx_it.h
│   │
│   └── Src/
│       ├── main.c
│       ├── stm32f4xx_it.c
│       └── system_stm32f4xx.c
│
├── Drivers/
│   └── STM32F4xx_HAL_Driver/
│
├── Button_Controlled_LED.ioc
└── README.md
```

---

# 🎯 13. What I Learned

Through this project, I learned how to:

* Configure GPIO input using STM32CubeIDE
* Configure GPIO output using STM32CubeIDE
* Read a button using `HAL_GPIO_ReadPin()`
* Control an LED using `HAL_GPIO_WritePin()`
* Understand `GPIO_PinState`
* Understand `GPIO_PIN_SET` and `GPIO_PIN_RESET`
* Understand HIGH and LOW
* Understand active-low inputs
* Use `if-else` with GPIO states
* Continuously monitor an input using polling

---

# ➡️ 14. Next Concept

After understanding GPIO using HAL, the next step is to understand **GPIO at the register level (Bare-Metal Programming)**.

Instead of:

```c
HAL_GPIO_ReadPin()
HAL_GPIO_WritePin()
```

we will learn how the STM32 GPIO registers themselves work.

### Learning Path

```text
GPIO Output using HAL
        ↓
GPIO Input + Output using HAL  ← This Project
        ↓
GPIO Output using Registers
        ↓
GPIO Input using Registers
        ↓
GPIO Interrupts
        ↓
Timers
        ↓
PWM
        ↓
ADC
        ↓
UART
        ↓
SPI
        ↓
I2C
        ↓
CAN
        ↓
FreeRTOS
```

---

## ⭐ Project Summary

**Project:** Button Controlled LED
**Microcontroller:** STM32F401RE
**Board:** NUCLEO-F401RE
**Framework:** STM32 HAL
**Input:** PC13 — Blue User Button
**Output:** PA5 — Green LED
**Concept:** GPIO Input + GPIO Output + Polling

> **Core idea:** Read the button state from PC13 and use that information to control the LED on PA5.
