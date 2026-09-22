
# 🖨️ 11_Printf_Retargeting_STM32

> **Using `printf()` with STM32 UART by retargeting output through `_io_putchar()` 🚀**

This repository is a continuation of my STM32 embedded-systems learning journey.

In normal C programming, we can easily use:

```c
printf("Hello World");
````

But on an STM32 microcontroller, `printf()` does not automatically know where the output should go.

So in this project, we will connect:

```text
printf()
   ↓
_io_putchar()
   ↓
HAL_UART_Transmit()
   ↓
USART2
   ↓
UART TX
   ↓
PC Serial Terminal
```

🎯 **Main Goal:** Learn how to use the standard C `printf()` function for debugging and serial communication on STM32.

---

# 📚 Table of Contents

* [🎯 What You Will Learn](#-what-you-will-learn)
* [🤔 Why Do We Need printf()?](#-why-do-we-need-printf)
* [🧠 What Is printf() Retargeting?](#-what-is-printf-retargeting)
* [🔌 UART as the Output Channel](#-uart-as-the-output-channel)
* [⚙️ STM32CubeMX Configuration](#️-stm32cubemx-configuration)
* [🛠️ Implementing _io_putchar()](#️-implementing-_io_putchar)
* [🔍 Understanding _io_putchar()](#-understanding-_io_putchar)
* [💻 Complete Example](#-complete-example)
* [📤 Printing Different Data Types](#-printing-different-data-types)
* [🖥️ Serial Terminal Setup](#️-serial-terminal-setup)
* [🔄 Complete Data Flow](#-complete-data-flow)
* [🐛 Common Problems](#-common-problems)
* [⚠️ Important Notes](#️-important-notes)
* [🧪 Practice Task](#-practice-task)
* [🚀 Real Embedded Use Case](#-real-embedded-use-case)
* [🎤 Interview Questions](#-interview-questions)
* [📝 Quick Revision](#-quick-revision)
* [🔜 What's Next?](#-whats-next)

---

# 🎯 What You Will Learn

After completing this repository, you will understand:

* What `printf()` is
* Why `printf()` does not directly print to an STM32 terminal
* What `printf()` retargeting means
* How `_io_putchar()` works
* How `_io_putchar()` connects `printf()` to UART
* How `HAL_UART_Transmit()` is used
* How to configure USART2
* How to send debugging messages from STM32
* How to print integers, floats and strings
* How to monitor STM32 output using a serial terminal

---

# 🤔 Why Do We Need `printf()`?

In normal C programming, we can write:

```c
printf("Hello World");
```

and see:

```text
Hello World
```

on the computer screen.

But STM32 is a microcontroller.

It does not have a normal computer console or terminal.

So STM32 needs an output path.

One convenient path is:

```text
STM32 UART → PC → Serial Terminal
```

Therefore, we want:

```c
printf("Hello STM32\r\n");
```

to eventually reach the UART peripheral.

---

# 🧠 What Is `printf()` Retargeting?

**Retargeting** means changing the destination of a standard output function.

Normally:

```text
printf()
   ↓
Standard Output
```

We want:

```text
printf()
   ↓
_io_putchar()
   ↓
UART
```

So we are essentially telling STM32:

> 🗣️ "Whenever `printf()` wants to print a character, send that character through UART."

---

# ⭐ The Main Concept

The complete idea is:

```text
             printf()
                │
                ▼
          _io_putchar()
                │
                ▼
      HAL_UART_Transmit()
                │
                ▼
             USART2
                │
                ▼
               TX
                │
                ▼
        USB / ST-LINK
                │
                ▼
                PC
                │
                ▼
        Serial Terminal
```

🔥 **Remember this flow!**

```text
printf()
   ↓
_io_putchar()
   ↓
HAL_UART_Transmit()
   ↓
UART
   ↓
PC
```

---

# 🔌 UART as the Output Channel

UART allows the STM32 to communicate with another device using serial communication.

The two main UART signals are:

| Pin | Function |
| --- | -------- |
| TX  | Transmit |
| RX  | Receive  |

For our `printf()` output:

```text
STM32 TX ───────────► PC
```

We mainly need the **TX path** because STM32 is sending data to the computer.

---

# ⚙️ STM32CubeMX Configuration

Open your project in:

```text
STM32CubeIDE
```

Then open the `.ioc` file.

---

## 1️⃣ Enable USART2

Go to:

```text
Connectivity
      ↓
USART2
```

Select:

```text
Mode → Asynchronous
```

---

## 2️⃣ USART2 Pins

For a typical STM32F401 configuration:

```text
PA2 → USART2_TX
PA3 → USART2_RX
```

The exact pins depend on the STM32 device and board configuration.

---

## 3️⃣ Configure Baud Rate

Set:

```text
Baud Rate = 115200
```

Recommended UART configuration:

```text
Baud Rate      → 115200
Word Length    → 8 Bits
Parity         → None
Stop Bits      → 1
Mode           → TX/RX
Hardware Flow  → None
```

This is commonly written as:

```text
115200 8N1
```

Meaning:

```text
115200 → Baud rate
8      → Data bits
N      → No parity
1      → Stop bit
```

---

# 🛠️ Implementing `_io_putchar()`

Now comes the most important part.

Add:

```c
int _io_putchar(int ch)
{
    HAL_UART_Transmit(&huart2,
                      (uint8_t *)&ch,
                      1,
                      HAL_MAX_DELAY);

    return ch;
}
```

🎯 This function connects `printf()` to UART.

---

# 🔍 Understanding `_io_putchar()`

Let's break it down.

## Function

```c
int _io_putchar(int ch)
```

The parameter:

```c
int ch
```

represents the character that needs to be transmitted.

For example:

```c
printf("ABC");
```

conceptually sends:

```text
'A'
'B'
'C'
```

through the character-output mechanism.

So `_io_putchar()` handles each character.

---

# 🔄 Character-by-Character Concept

Suppose we write:

```c
printf("ABC");
```

Conceptually:

```text
'A'
 ↓
_io_putchar('A')
 ↓
UART

'B'
 ↓
_io_putchar('B')
 ↓
UART

'C'
 ↓
_io_putchar('C')
 ↓
UART
```

The serial terminal finally displays:

```text
ABC
```

---

# 📤 Inside `_io_putchar()`

Our implementation is:

```c
int _io_putchar(int ch)
{
    HAL_UART_Transmit(&huart2,
                      (uint8_t *)&ch,
                      1,
                      HAL_MAX_DELAY);

    return ch;
}
```

Now let's understand each part.

---

## 🔹 `&huart2`

```c
&huart2
```

This tells the HAL which UART peripheral should be used.

For example:

```c
UART_HandleTypeDef huart2;
```

represents USART2.

---

## 🔹 `(uint8_t *)&ch`

```c
(uint8_t *)&ch
```

This provides the address of the character data to the UART transmit function.

UART transmits data as bytes.

---

## 🔹 `1`

```c
1
```

We are transmitting one character at a time.

Therefore:

```text
Size = 1 byte
```

---

## 🔹 `HAL_MAX_DELAY`

```c
HAL_MAX_DELAY
```

This allows the HAL function to wait until the transmission is completed.

---

## 🔹 `return ch`

```c
return ch;
```

The transmitted character is returned to the calling function.

---

# 💻 Complete Example

A simple implementation looks like this:

```c
#include "main.h"
#include <stdio.h>

extern UART_HandleTypeDef huart2;

int _io_putchar(int ch)
{
    HAL_UART_Transmit(&huart2,
                      (uint8_t *)&ch,
                      1,
                      HAL_MAX_DELAY);

    return ch;
}

int main(void)
{
    HAL_Init();

    SystemClock_Config();

    MX_GPIO_Init();
    MX_USART2_UART_Init();

    printf("Hello STM32!\r\n");

    while (1)
    {
        printf("STM32 is running...\r\n");

        HAL_Delay(1000);
    }
}
```

> ⚠️ Keep the initialization functions generated by STM32CubeMX/CubeIDE. The exact generated code depends on your MCU and project.

---

# 🆚 Before and After

## ❌ Without `printf()` Retargeting

You may have to write:

```c
HAL_UART_Transmit(&huart2,
                  (uint8_t*)"Hello STM32\r\n",
                  14,
                  HAL_MAX_DELAY);
```

Every time you want to send something.

😵 That's a lot of code for a simple message!

---

## ✅ With `_io_putchar()`

Once `_io_putchar()` is implemented:

```c
printf("Hello STM32\r\n");
```

Much cleaner! 😎

---

# 📤 Printing Different Data Types

One of the biggest advantages of `printf()` is formatted output.

---

## 🔢 Integer

```c
int temperature = 25;

printf("Temperature = %d\r\n", temperature);
```

Output:

```text
Temperature = 25
```

---

## 🔢 Unsigned Integer

```c
uint32_t counter = 100;

printf("Counter = %lu\r\n", counter);
```

Output:

```text
Counter = 100
```

---

## 🔤 Character

```c
char ch = 'A';

printf("Character = %c\r\n", ch);
```

Output:

```text
Character = A
```

---

## 📝 String

```c
char name[] = "STM32";

printf("MCU = %s\r\n", name);
```

Output:

```text
MCU = STM32
```

---

## 🔢 Float

```c
float voltage = 3.30f;

printf("Voltage = %.2f V\r\n", voltage);
```

Output:

```text
Voltage = 3.30 V
```

> ⚠️ Floating-point support may need to be enabled in the project linker settings depending on your toolchain configuration.

---

# 🖥️ Serial Terminal Setup

After programming the STM32:

1. Connect the STM32 board to your computer.
2. Find the correct COM port.
3. Open a serial terminal.
4. Select the correct UART interface.
5. Set the baud rate to `115200`.

For Windows you may see:

```text
COM3
COM4
COM5
```

On Linux you may see:

```text
/dev/ttyACM0
```

---

## Terminal Settings

```text
┌──────────────────────────────┐
│      SERIAL TERMINAL         │
├──────────────────────────────┤
│ Baud Rate : 115200           │
│ Data Bits : 8                │
│ Parity    : None             │
│ Stop Bits : 1                │
│ Flow Ctrl : None             │
└──────────────────────────────┘
```

---

# 🧪 Expected Output

If your code contains:

```c
printf("Hello STM32!\r\n");

while (1)
{
    printf("STM32 is running...\r\n");

    HAL_Delay(1000);
}
```

Your terminal should show:

```text
Hello STM32!
STM32 is running...
STM32 is running...
STM32 is running...
STM32 is running...
```

🎉 Your `printf()` is now working through UART!

---

# 🔄 Complete Data Flow

This is the most important diagram in this project:

```text
                    YOUR CODE
                       │
                       ▼
              printf("Hello STM32")
                       │
                       ▼
              ┌─────────────────┐
              │  _io_putchar()  │
              └─────────────────┘
                       │
                       ▼
             HAL_UART_Transmit()
                       │
                       ▼
                    USART2
                       │
                       ▼
                      TX
                       │
                       ▼
                 USB / ST-LINK
                       │
                       ▼
                      PC
                       │
                       ▼
               Serial Terminal
                       │
                       ▼
                 Hello STM32
```

---

# 🧠 What Is Actually Happening?

When you write:

```c
printf("Hello");
```

you are **not directly calling UART**.

Instead, the output is routed through the character output function provided by your setup:

```text
printf()
   ↓
_io_putchar()
   ↓
HAL_UART_Transmit()
   ↓
UART hardware
```

This is called **retargeting standard output**.

---

# 🐛 Common Problems

## ❌ 1. Nothing Appears

Check:

```text
✔ STM32 is connected
✔ Correct COM port selected
✔ Correct UART selected
✔ USART2 enabled
✔ TX pin configured
✔ Baud rate = 115200
✔ Terminal settings are correct
```

---

# ❌ 2. Garbage Characters

Example:

```text
▒▒▒▒╫╫╫@#@
```

Possible causes:

* Wrong baud rate
* Incorrect clock configuration
* Incorrect UART configuration
* Wrong COM port
* Wrong serial terminal settings

Make sure both sides use:

```text
115200 8N1
```

---

# ❌ 3. Wrong UART Handle

If your project contains:

```c
UART_HandleTypeDef huart2;
```

then use:

```c
&huart2
```

inside:

```c
HAL_UART_Transmit()
```

For example:

```c
HAL_UART_Transmit(&huart2,
                  (uint8_t *)&ch,
                  1,
                  HAL_MAX_DELAY);
```

---

# ❌ 4. New Line Is Not Working Properly

Prefer:

```c
printf("Hello\r\n");
```

instead of only:

```c
printf("Hello\n");
```

For UART terminals:

```text
\r → Carriage Return
\n → New Line
```

Together:

```text
\r\n
```

give the expected new-line behavior in many serial terminals.

---

# ⚠️ Important Notes

## 1️⃣ `_io_putchar()` Is Toolchain/Setup Dependent

You may see different retargeting methods in STM32 projects.

For example:

```c
_io_putchar()
```

or:

```c
__io_putchar()
```

or:

```c
_write()
```

These are not three names for the same function in every setup.

They depend on the library/toolchain and project configuration.

### In this repository

We are specifically learning:

```c
_io_putchar(int ch)
```

because this is the method used in this learning path.

---

# ⚡ Why Is `printf()` Useful in Embedded Systems?

`printf()` is extremely useful during development and debugging.

For example:

```c
printf("System Started\r\n");
```

```c
printf("Temperature = %d\r\n", temperature);
```

```c
printf("ADC Value = %lu\r\n", adc_value);
```

```c
printf("Battery = %.2f V\r\n", voltage);
```

You can observe internal values without needing an LCD or other display.

---

# 🛰️ Real Embedded-System Example

Imagine an STM32 system monitoring a sensor.

You can write:

```c
printf("Temperature = %.2f C\r\n", temperature);

printf("Pressure = %.2f hPa\r\n", pressure);

printf("Battery = %.2f V\r\n", battery_voltage);
```

The PC terminal becomes your debugging window.

For an avionics system, you could similarly inspect:

```text
Altitude
Temperature
Battery Voltage
GPS Data
IMU Data
Sensor Status
System State
```

Example:

```text
==============================
       STM32 AVIONICS
==============================

Altitude    : 125.40 m
Temperature : 28.50 C
Battery     : 7.82 V
GPS         : FIXED
IMU         : OK
System      : RUNNING

==============================
```

🚀 This is one of the practical uses of `printf()` in embedded development.

---

# ⚠️ `printf()` Is Not Always Ideal

Although `printf()` is very convenient, it can be relatively heavy for a microcontroller.

It can consume:

* Flash memory
* RAM
* CPU time

Floating-point formatting can be particularly expensive.

So:

```text
During learning/debugging
        ↓
      printf()
        ↓
       ✅
```

But in performance-sensitive production firmware:

```text
Use carefully
        ↓
Optimized logging may be preferred
```

---

# 🆚 `HAL_UART_Transmit()` vs `printf()`

| Feature                | `HAL_UART_Transmit()` | `printf()` |
| ---------------------- | --------------------- | ---------- |
| Direct UART control    | ✅                     | ❌          |
| Easy text output       | 😐                    | ✅          |
| Formatted output       | ❌                     | ✅          |
| Print variables easily | 😐                    | ✅          |
| Debugging              | ✅                     | ⭐⭐⭐        |
| Code readability       | Lower                 | Higher     |
| Overhead               | Lower                 | Higher     |

Example:

### Direct UART

```c
HAL_UART_Transmit(&huart2,
                  (uint8_t*)"Hello\r\n",
                  7,
                  HAL_MAX_DELAY);
```

### `printf()`

```c
printf("Hello\r\n");
```

---

# 🧩 The Role of Each Function

| Function                | Purpose                          |
| ----------------------- | -------------------------------- |
| `printf()`              | Formats and requests output      |
| `_io_putchar()`         | Handles character output         |
| `HAL_UART_Transmit()`   | Sends the character through UART |
| `MX_USART2_UART_Init()` | Initializes USART2               |
| `HAL_Delay()`           | Creates a delay                  |

Remember:

```text
printf()
   ↓
_io_putchar()
   ↓
HAL_UART_Transmit()
   ↓
UART
```

---

# 🧪 Practice Task

Create a program that prints:

```text
========================
      STM32 MONITOR
========================

System Started!

Counter = 0
Counter = 1
Counter = 2
Counter = 3
Counter = 4
```

Use:

```c
printf()
```

for all messages.

---

# 🔥 Challenge

Create these variables:

```c
int temperature = 28;
float voltage = 3.30f;
uint32_t counter = 0;
```

Then continuously print:

```text
Temperature = 28 C
Voltage = 3.30 V
Counter = 0
```

and increment the counter every second.

---

# 🚀 Mini Debugging Project

Try creating a simple system monitor:

```text
================================
          STM32 DEBUG
================================

System Status : OK
Temperature   : 28.50 C
Voltage       : 3.30 V
Counter       : 25
UART          : OK

================================
```

This will help you practice:

* `printf()`
* Format specifiers
* Variables
* UART
* Debugging

---

# 🎤 Interview Questions

## Q1. What is `printf()`?

`printf()` is a standard C library function used for formatted output.

---

## Q2. Why doesn't `printf()` directly display output on STM32?

Because STM32 does not have a conventional PC console. The output must be redirected to an interface such as UART.

---

## Q3. What is `printf()` retargeting?

Retargeting is the process of redirecting standard output from `printf()` to a hardware interface such as UART.

---

## Q4. What is `_io_putchar()`?

`_io_putchar()` is a character-output function that can be implemented to send each character generated by `printf()` through a selected peripheral.

---

## Q5. How do you redirect `printf()` to UART?

A common implementation is:

```c
int _io_putchar(int ch)
{
    HAL_UART_Transmit(&huart2,
                      (uint8_t *)&ch,
                      1,
                      HAL_MAX_DELAY);

    return ch;
}
```

---

## Q6. Why is `HAL_UART_Transmit()` used?

It is used to transmit data through the STM32 UART peripheral using the HAL library.

---

## Q7. What does `115200 8N1` mean?

```text
115200 → Baud rate
8      → Data bits
N      → No parity
1      → Stop bit
```

---

## Q8. What is the main advantage of `printf()` over direct UART transmission?

It provides convenient formatted output, making debugging and displaying variables much easier.

---

## Q9. What is one disadvantage of `printf()`?

It can have significant memory and execution-time overhead, especially when floating-point formatting is used.

---

# 📝 Quick Revision

### 🔹 Main Function

```c
_io_putchar(int ch)
```

### 🔹 UART Transmission

```c
HAL_UART_Transmit()
```

### 🔹 Output Function

```c
printf()
```

### 🔹 Typical UART

```text
USART2
```

### 🔹 Typical Pins

```text
PA2 → TX
PA3 → RX
```

### 🔹 Typical Baud Rate

```text
115200
```

### 🔹 Typical Format

```text
8N1
```

---

# 🧠 One-Minute Revision

If you remember only one thing from this repository, remember this:

```text
          printf()
             │
             ▼
      _io_putchar()
             │
             ▼
   HAL_UART_Transmit()
             │
             ▼
          USART2
             │
             ▼
          UART TX
             │
             ▼
             PC
             │
             ▼
      Serial Terminal
```

### In one line:

> **`_io_putchar()` acts as the bridge that allows `printf()` output to be transmitted through UART.**


# 🏆 Key Takeaway

The entire project can be remembered with just this:

```text
printf()
   ↓
_io_putchar()
   ↓
HAL_UART_Transmit()
   ↓
UART
   ↓
PC
```

We have taken the simple C function:

```c
printf("Hello STM32");
```

and connected it to actual STM32 hardware.

🎯 **This is `printf()` retargeting.**

---

## 🚀 Keep Learning • Keep Debugging • Keep Building

```text
        CODE
         │
         ▼
      DEBUG
         │
         ▼
      UNDERSTAND
         │
         ▼
       BUILD
         │
         ▼
        🚀

