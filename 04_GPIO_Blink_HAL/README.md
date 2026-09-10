



## Overview

This project demonstrates how to blink the onboard user LED of the **NUCLEO-F401RE** using the **STM32 HAL (Hardware Abstraction Layer)**.

The main purpose of this project is not only to blink an LED, but also to understand how to:

- Create an STM32 project using STM32CubeMX/CubeIDE
- Identify the correct LED pin from the board documentation
- Configure a GPIO pin as an output
- Use a GPIO user label
- Find HAL functions in the STM32 HAL driver files
- Understand the generated `main.c` structure
- Use `HAL_GPIO_TogglePin()` for LED blinking
- Use `HAL_Delay()` to control the blink speed

---

## Hardware Used

- **Microcontroller:** STM32F401RE
- **Development Board:** NUCLEO-F401RE
- **User LED:** LD2
- **LED GPIO:** PA5
- **Arduino Signal:** D13

### LD2 Pin Mapping

According to the NUCLEO-F401RE board documentation, the green user LED **LD2** is connected to the Arduino **D13** signal, which corresponds to **PA5** of the STM32F401RE.

```text
LD2
 ↓
Arduino D13
 ↓
STM32 PA5
 ↓
GPIO Output
````

> The board documentation should always be checked when identifying the pin connected to an onboard peripheral.

---

# Project Creation

Create the project using the STM32 project wizard:

```text
File
 ↓
New
 ↓
STM32 Project
 ↓
Target Selector
```

Search for the required STM32 device/board.

For this project:

```text
STM32F401RE
```

Select the appropriate STM32F401RE target.

Then provide the project name:

```text
01_GPIO_Blink_HAL
```

Keep the remaining project settings at their default values unless required otherwise.

---

# GPIO Configuration

<img width="631" height="422" alt="Screenshot 2026-09-08 230455" src="https://github.com/user-attachments/assets/09c7a275-509b-4b72-9861-b6cc0f279df3" />

After creating the project, open the `.ioc` configuration file.

The MCU pinout will be displayed.

Locate:

```text
PA5
```

Configure it as:

```text
GPIO_Output
```

PA5 is now configured as a GPIO output for controlling the onboard LD2 LED.

---

## Adding a User Label

<img width="1077" height="676" alt="Screenshot 2026-09-08 230714" src="https://github.com/user-attachments/assets/bbcd31cb-0600-41e4-bec0-650020ad8287" />


To make the generated code easier to understand, assign a user label to PA5.

Right-click on **PA5** and select:

```text
Enter User Label
```

Enter:

```text
MY_LED
```

The configuration becomes:

```text
PA5
 └── GPIO_Output
      └── User Label: MY_LED
```

Save the `.ioc` file:

```text
Ctrl + S
```

If asked to generate code, select:

```text
Yes
```

---

# Generated Project Structure

After code generation, the project contains the generated source and header files.

The important directories are:

```text
01_GPIO_Blink_HAL/
│
├── Core/
│   ├── Inc/
│   │   └── main.h
│   │
│   └── Src/
│       └── main.c
│
├── Drivers/
│   └── STM32F4xx_HAL_Driver/
│       ├── Inc/
│       └── Src/
│
└── Startup/
```

For this project, the most important files are:

```text
Core/Src/main.c
Core/Inc/main.h
Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_gpio.c
Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal.c
```

---

# Understanding `main.c`

Open:

```text
Core
 └── Src
      └── main.c
```

The generated `main.c` contains initialization code followed by an infinite loop.

The basic structure is:

```c
int main(void)
{
    HAL_Init();

    SystemClock_Config();

    MX_GPIO_Init();

    while (1)
    {
        // Application code
    }
}
```

---

## Arduino Comparison

If you have previously worked with Arduino, the STM32 structure can be understood approximately as:

```text
Arduino                  STM32 HAL
------------------------------------------------
setup()          →       Initialization code

loop()           →       while(1)
```

The initialization section contains functions such as:

```c
HAL_Init();
SystemClock_Config();
MX_GPIO_Init();
```

The application code runs continuously inside:

```c
while (1)
{
}
```

---

# Understanding HAL

HAL stands for:

**Hardware Abstraction Layer**

HAL provides functions that allow us to interact with STM32 peripherals without directly manipulating every hardware register ourselves.

For example, instead of directly modifying GPIO registers, we can use:

```c
HAL_GPIO_TogglePin();
```

This makes peripheral programming easier and more readable.

---

# How to Find HAL GPIO Functions

Instead of memorizing every HAL function, you can find the function inside the HAL driver source code.

Navigate to:

```text
Drivers
 └── STM32F4xx_HAL_Driver
      └── Src
           └── stm32f4xx_hal_gpio.c
```

This file contains the GPIO-related HAL functions.

Examples include:

```c
HAL_GPIO_WritePin();
HAL_GPIO_TogglePin();
HAL_GPIO_ReadPin();
```

---

# `HAL_GPIO_WritePin()`

The GPIO HAL driver contains the function:

```c
HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx,
                 uint16_t GPIO_Pin,
                 GPIO_PinState PinState);
```

The function requires:

1. GPIO port
2. GPIO pin
3. Pin state

Conceptually:

```text
GPIO Port
    +
GPIO Pin
    +
Pin State
```

For example:

```c
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
```

This sets PA5 to the specified output state.

---

# `HAL_GPIO_TogglePin()`

For blinking an LED, we can use:

```c
HAL_GPIO_TogglePin();
```

Toggle means changing the current output state:

```text
LOW  → HIGH
HIGH → LOW
```

Therefore, repeatedly calling:

```c
HAL_GPIO_TogglePin();
```

will produce:

```text
ON
 ↓
OFF
 ↓
ON
 ↓
OFF
 ↓
...
```

---

# Finding the LED Port and Pin

Because we assigned the user label:

```text
MY_LED
```

CubeMX generates corresponding definitions in:

```text
Core
 └── Inc
      └── main.h
```

You should find definitions similar to:

```c
#define MY_LED_Pin GPIO_PIN_5
#define MY_LED_GPIO_Port GPIOA
```

Therefore:

```text
MY_LED_GPIO_Port → GPIOA
MY_LED_Pin       → GPIO_PIN_5
```

This allows us to write:

```c
HAL_GPIO_TogglePin(MY_LED_GPIO_Port, MY_LED_Pin);
```

instead of directly writing:

```c
HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
```

Using the generated label makes the application code easier to understand.

---

# Adding the Blink Code

Open:

```text
Core/Src/main.c
```

Locate:

```c
while (1)
{
}
```

Add:

```c
while (1)
{
    HAL_GPIO_TogglePin(MY_LED_GPIO_Port, MY_LED_Pin);
    HAL_Delay(500);
}
```

---

# Understanding `HAL_Delay()`

The delay function is:

```c
HAL_Delay();
```

The delay value is specified in milliseconds.

For example:

```c
HAL_Delay(500);
```

means:

```text
500 ms
=
0.5 seconds
```

The HAL delay function can be found in:

```text
Drivers
 └── STM32F4xx_HAL_Driver
      └── Src
           └── stm32f4xx_hal.c
```

---

# Complete Blink Logic

The program executes:

```text
Start
  ↓
Initialize HAL
  ↓
Configure system clock
  ↓
Initialize GPIO
  ↓
Enter while(1)
  ↓
Toggle LED
  ↓
Wait 500 ms
  ↓
Toggle LED
  ↓
Wait 500 ms
  ↓
Repeat forever
```

Therefore:

```text
LED ON
  ↓
500 ms
  ↓
LED OFF
  ↓
500 ms
  ↓
LED ON
  ↓
500 ms
  ↓
Repeat
```

---

# Final Code

The important application code inside `main.c` is:

```c
while (1)
{
    HAL_GPIO_TogglePin(MY_LED_GPIO_Port, MY_LED_Pin);
    HAL_Delay(500);
}
```

---

# How the Code Works

### `HAL_GPIO_TogglePin()`

```c
HAL_GPIO_TogglePin(MY_LED_GPIO_Port, MY_LED_Pin);
```

Toggles the output state of the LED pin.

For this project:

```text
MY_LED_GPIO_Port → GPIOA
MY_LED_Pin       → GPIO_PIN_5
```

So the function effectively operates on:

```c
GPIOA, GPIO_PIN_5
```

---

### `HAL_Delay(500)`

```c
HAL_Delay(500);
```

pauses execution for approximately:

```text
500 milliseconds
```

This makes the LED transition slow enough for us to see.

---

# Important Files to Explore

For learning how STM32 HAL works, explore these files:

### GPIO HAL implementation

```text
Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_gpio.c
```

Contains GPIO functions such as:

```c
HAL_GPIO_WritePin()
HAL_GPIO_TogglePin()
HAL_GPIO_ReadPin()
```

### Common HAL implementation

```text
Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal.c
```

Contains common HAL functions such as:

```c
HAL_Init()
HAL_Delay()
```

### Pin definitions

```text
Core/Inc/main.h
```

Contains generated definitions such as:

```c
#define MY_LED_Pin GPIO_PIN_5
#define MY_LED_GPIO_Port GPIOA
```

### Application code

```text
Core/Src/main.c
```

Contains:

```c
int main(void)
```

and the application logic inside:

```c
while (1)
{
}
```

---

# Key Learning

The main purpose of this exercise is not just blinking the LED.

The important workflow is:

```text
Need a peripheral function
        ↓
Identify the peripheral
        ↓
Open the corresponding HAL driver
        ↓
Find the required function
        ↓
Check the function parameters
        ↓
Find the generated PORT/PIN definitions
        ↓
Use the HAL function in main.c
```

For GPIO:

```text
GPIO
 ↓
stm32f4xx_hal_gpio.c
 ↓
HAL_GPIO_TogglePin()
 ↓
main.h
 ↓
MY_LED_GPIO_Port
MY_LED_Pin
 ↓
main.c
```

This same approach can later be used when working with other STM32 peripherals such as:

```text
UART
SPI
I2C
ADC
TIMERS
PWM
CAN
```

---

## Expected Result


https://github.com/user-attachments/assets/9e41ba62-3f8b-4fcd-8725-2b8559ab2801



After building and programming the STM32F401RE, the onboard **LD2 green user LED** should blink approximately every **500 ms**.

```text
LD2:  ON → OFF → ON → OFF → ...
       500ms  500ms
```

This completes the basic **STM32 GPIO LED Blink using HAL** project.

```
```
