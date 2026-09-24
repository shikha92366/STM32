
# 🐛 11 — STM32 Debugging: Live Expressions & SWV

> **Stop guessing what your STM32 is doing. Start watching it. 🔍**

This repository covers **STM32 debugging using STM32CubeIDE**, focusing on two powerful debugging techniques:

1. 🔎 **Live Expressions**
2. 📡 **Serial Wire Viewer (SWV)**

We will also learn how to use:

- SWD debugging
- ITM (Instrumentation Trace Macrocell)
- SWO (Serial Wire Output)
- ITM Stimulus Port 0
- SWV ITM Data Console
- `printf()` retargeting
- `_write()`
- `ITM_SendChar()`

The goal is to understand how to observe the internal behavior of an STM32 application **without depending completely on UART**.

---

# 📚 Table of Contents

- [1. Why Debugging Matters](#1-why-debugging-matters)
- [2. Two Debugging Methods](#2-two-debugging-methods)
- [3. Live Expressions](#3-live-expressions)
- [4. Live Expressions Example](#4-live-expressions-example)
- [5. Limitations of Live Expressions](#5-limitations-of-live-expressions)
- [6. Serial Wire Viewer — SWV](#6-serial-wire-viewer--swv)
- [7. SWV Configuration](#7-swv-configuration)
- [8. Understanding SysTick](#8-understanding-systick)
- [9. Retargeting printf to SWV](#9-retargeting-printf-to-swv)
- [10. `_write()` Function](#10-write-function)
- [11. ITM_SendChar()](#11-itm_sendchar)
- [12. Complete SWV printf Example](#12-complete-swv-printf-example)
- [13. SWV ITM Data Console](#13-swv-itm-data-console)
- [14. Configure Trace](#14-configure-trace)
- [15. Start Trace](#15-start-trace)
- [16. Important Build Step](#16-important-build-step)
- [17. Live Expressions vs SWV](#17-live-expressions-vs-swv)
- [18. Debugging Workflow](#18-debugging-workflow)
- [19. Common Problems](#19-common-problems)
- [20. Practical Experiment](#20-practical-experiment)
- [21. Real-World Application](#21-real-world-application)
- [22. Interview Questions](#22-interview-questions)
- [23. Quick Revision](#23-quick-revision)
- [24. Learning Progress](#24-learning-progress)

---

# 1. Why Debugging Matters

When we write embedded programs, **debugging is one of the most important parts of development**.

A program can:

- Compile successfully ✅
- Flash successfully ✅
- Start running ✅

and still produce the wrong result ❌.

For example:

```c
int counter = 0;

while (1)
{
    counter++;
    HAL_Delay(2000);
}
````

We may want to know:

```text
Is counter changing?

What is its current value?

How frequently is it changing?

Is the program actually reaching this part of the code?
```

Instead of modifying the program repeatedly, STM32CubeIDE provides debugging tools that allow us to observe the program while it is running.

---

# 2. Two Debugging Methods

This module focuses on:

```text
                 STM32 DEBUGGING
                       │
            ┌──────────┴──────────┐
            │                     │
            ▼                     ▼
    Live Expressions             SWV
            │                     │
            │                     ├── ITM
            │                     ├── SWO
            │                     └── ITM Data Console
            │
            ▼
     Variable Monitoring
```

### 🔎 Live Expressions

Used mainly to observe variables and expressions.

Example:

```text
counter = 5
```

### 📡 SWV

Used to send runtime information from the STM32 to STM32CubeIDE.

For example:

```text
Value of counter = 5
Value of ADC = 1234
Sensor initialized
System started
```

The reference specifically introduces these as the two debugging approaches covered in this lesson. 

---

# 3. Live Expressions

## 🔎 What are Live Expressions?

**Live Expressions** allow us to monitor the value of variables while the MCU is executing.

This is extremely useful when debugging:

* Counters
* Flags
* Sensor values
* ADC values
* State variables
* Calculated values

---

# 4. Live Expressions Example

First create a variable **outside `main()`**:

```c
uint16_t counter = 0;
```

Then:

```c
int main(void)
{
    HAL_Init();

    while (1)
    {
        counter++;

        HAL_Delay(2000);
    }
}
```

### What happens?

Initially:

```text
counter = 0
```

After the first loop:

```text
counter = 1
```

After another 2 seconds:

```text
counter = 2
```

Then:

```text
3
4
5
...
```

So the expected behavior is:

```text
0 → 1 → 2 → 3 → 4 → 5 → ...
```

The reference uses this same simple counter example and a 2-second delay to demonstrate Live Expressions. 

---

## 🛠️ How to Open Live Expressions

Start a debugging session.

Then go to:

```text
Window
   ↓
Show View
   ↓
Live Expressions
```

The reference follows this same workflow. 

---

## ➕ Add an Expression

Click:

```text
New Expression
```

Enter:

```text
counter
```

The debugger will identify the variable and show its current value.

Example:

```text
┌──────────────────────────────┐
│ Live Expressions             │
├──────────────────────────────┤
│ Expression       Value       │
│                              │
│ counter          0           │
└──────────────────────────────┘
```

Now press:

```text
▶ Resume
```

The value should update:

```text
0
1
2
3
4
5
...
```

The reference demonstrates that the counter updates live every two seconds after pressing Resume. 

---

# 5. Limitations of Live Expressions

Live Expressions are very useful, but there is a limitation.

They are excellent for:

```text
counter
ADC_Value
temperature
state
voltage
flag
```

But they are not ideal when we want **formatted output**.

For example, suppose we want:

```text
Counter = 10
Temperature = 28°C
Voltage = 7.4V
State = FLIGHT
```

Live Expressions mainly show individual expressions.

For formatted messages, **SWV + `printf()`** is more useful.

The reference makes this exact distinction: Live Expressions can show variables, but formatted output similar to `printf()` requires SWV. 

---

# 6. Serial Wire Viewer — SWV

## 📡 What is SWV?

**SWV = Serial Wire Viewer**

SWV provides a way to observe runtime information from a supported STM32/Cortex-M system through the debugging infrastructure.

Conceptually:

```text
STM32
  │
  ▼
Cortex-M Core
  │
  ▼
ITM
  │
  ▼
SWO
  │
  ▼
ST-LINK
  │
  ▼
USB
  │
  ▼
STM32CubeIDE
  │
  ▼
SWV ITM Data Console
```

This allows us to send debugging information without using a normal UART communication path.

---

# 7. SWV Configuration

For Live Expressions, we can use the basic debug configuration.

For SWV, additional configuration is required.

The reference configures SWV through the STM32 configuration interface:

```text
System Core
     ↓
SYS
     ↓
Debug
     ↓
Serial Wire
```

The reference also uses:

```text
Timebase Source → SysTick
```

and saves the configuration. 

---

# 8. Understanding SysTick

## ⏱️ What is SysTick?

**SysTick = System Tick Timer**

It is a timer available in ARM Cortex-M processors.

It is commonly used to generate periodic system ticks.

In STM32 HAL applications, SysTick is commonly associated with the HAL time base.

Conceptually:

```text
System Clock
     ↓
   SysTick
     ↓
Periodic Tick
     ↓
HAL Time Base
```

In the SWV configuration shown in the reference, **SysTick is selected as the timebase source**. 

---

# 9. Retargeting printf to SWV

Now comes the important part.

We already learned how `printf()` can be redirected to UART.

For SWV, we do something similar.

But instead of:

```c
_io_putchar()
```

or:

```c
HAL_UART_Transmit()
```

we use:

```c
ITM_SendChar()
```

The reference specifically changes the output function from `IO_PutChar` to `ITM_SendChar`. 

---

# 10. `_write()` Function

The C library can use `_write()` as the low-level output function behind `printf()`.

A typical implementation for SWV is:

```c
int _write(int file, char *ptr, int len)
{
    for (int i = 0; i < len; i++)
    {
        ITM_SendChar(ptr[i]);
    }

    return len;
}
```

---

## 🔍 Understanding the Function

### Function:

```c
_write()
```

receives:

```text
file
ptr
len
```

The important parameter for us is:

```c
ptr
```

because it points to the characters that need to be transmitted.

---

## The loop

```c
for (int i = 0; i < len; i++)
{
    ITM_SendChar(ptr[i]);
}
```

This sends each character one by one.

For example:

```c
printf("HELLO");
```

conceptually becomes:

```text
'H' → ITM_SendChar()
'E' → ITM_SendChar()
'L' → ITM_SendChar()
'L' → ITM_SendChar()
'O' → ITM_SendChar()
```

---

# 11. ITM_SendChar()

## What is ITM?

**ITM = Instrumentation Trace Macrocell**

ITM is part of the Cortex-M debug/trace infrastructure.

For our application, we mainly care about:

```c
ITM_SendChar()
```

Example:

```c
ITM_SendChar('A');
```

This sends the character through the ITM trace mechanism.

---

# 🔄 Complete Data Flow

The most important flow to remember is:

```text
printf()
   │
   ▼
_write()
   │
   ▼
ITM_SendChar()
   │
   ▼
ITM
   │
   ▼
SWO
   │
   ▼
ST-LINK
   │
   ▼
STM32CubeIDE
   │
   ▼
SWV ITM Data Console
```

### 🧠 Remember:

> `printf()` → `_write()` → `ITM_SendChar()` → SWV → ITM Console

---

# 12. Complete SWV printf Example

```c
#include "main.h"
#include <stdio.h>

int _write(int file, char *ptr, int len)
{
    for (int i = 0; i < len; i++)
    {
        ITM_SendChar(ptr[i]);
    }

    return len;
}

uint16_t counter = 0;

int main(void)
{
    HAL_Init();

    while (1)
    {
        counter++;

        printf("Value of counter = %d\r\n", counter);

        HAL_Delay(2000);
    }
}
```

Expected output:

```text
Value of counter = 1
Value of counter = 2
Value of counter = 3
Value of counter = 4
Value of counter = 5
...
```

The reference uses the same idea: increment the counter, print its value using formatted `printf()`, and delay for two seconds. 

---

# 13. SWV ITM Data Console

After configuring SWV and adding `_write()`, start the debugger.

Then open:

```text
Window
   ↓
Show View
   ↓
SWV
   ↓
SWV ITM Data Console
```

The reference specifically uses the **SWV ITM Data Console** for displaying the formatted output. 

---

# 14. Configure Trace

Opening the console is not enough.

We need to configure the ITM stimulus port.

Inside:

```text
SWV ITM Data Console
```

find:

```text
Configure Trace
```

Then:

```text
ITM Stimulus Ports
        ↓
Port 0
        ↓
Enable / Tick
        ↓
OK
```

The reference enables **ITM Stimulus Port 0** before starting the trace. 

---

# 15. Start Trace

After configuring Port 0:

```text
Start Trace
     ↓
Resume
     ↓
Program executes
     ↓
printf()
     ↓
_write()
     ↓
ITM_SendChar()
     ↓
SWV Console
```

You should now see:

```text
Value of counter = 1
Value of counter = 2
Value of counter = 3
...
```

The reference follows this exact sequence: configure the trace, start the trace, then resume execution. 

---

# 16. Important Build Step

## ⚠️ Don't Forget to Build!

This is one of the most important practical lessons from the debugging session.

If you add or modify `_write()` but don't build the project, the debugger may still be using the old compiled code.

The reference demonstrates exactly this issue: the `_write()` function was added but not rebuilt, causing `printf()` not to work initially. 

### Correct workflow:

```text
Modify Code
    ↓
Save
    ↓
Build
    ↓
Build Successful
    ↓
Start Debug
    ↓
Configure SWV
    ↓
Start Trace
    ↓
Resume
```

### ⭐ Golden Rule

> **If you change the code, rebuild before debugging.**

---

# 17. Live Expressions vs SWV

| Feature                    | Live Expressions | SWV + ITM |
| -------------------------- | ---------------- | --------- |
| Monitor variables          | ✅                | ✅         |
| Formatted output           | ❌                | ✅         |
| `printf()`                 | ❌                | ✅         |
| Runtime messages           | Limited          | ✅         |
| Sensor values              | ✅                | ✅         |
| Debugging                  | ✅                | ✅         |
| Uses normal UART           | ❌                | ❌         |
| ITM required               | ❌                | ✅         |
| ITM Data Console           | ❌                | ✅         |
| Best for variable watching | ⭐⭐⭐              | ⭐⭐        |
| Best for formatted logs    | ⭐                | ⭐⭐⭐       |

---

# 18. Debugging Workflow

## 🔎 Method 1 — Live Expressions

```text
Write Code
    ↓
Build
    ↓
Start Debug
    ↓
Open Live Expressions
    ↓
Add Variable
    ↓
Resume
    ↓
Observe Value
```

---

## 📡 Method 2 — SWV

```text
Write Code
    ↓
Configure SYS → Debug → Serial Wire
    ↓
Implement _write()
    ↓
Use ITM_SendChar()
    ↓
Build
    ↓
Debug
    ↓
Open SWV ITM Data Console
    ↓
Configure ITM Port 0
    ↓
Start Trace
    ↓
Resume
    ↓
Observe printf()
```

---

# 19. Common Problems

## ❌ Problem 1 — Live Expression is not updating

Check:

* Debug session is running
* Correct variable name is entered
* Variable is visible to the debugger
* Program is executing
* Code has been built

---

## ❌ Problem 2 — `printf()` does not appear

Check:

```text
Did I implement _write()?
        ↓
Did I use ITM_SendChar()?
        ↓
Did I build the project?
        ↓
Is SWV enabled?
        ↓
Is ITM Port 0 enabled?
        ↓
Did I Start Trace?
        ↓
Did I Resume execution?
```

---

## ❌ Problem 3 — SWV Console is empty

Check:

```text
SWV enabled?
     ↓
ITM Port 0 enabled?
     ↓
Start Trace?
     ↓
Resume?
     ↓
printf() executing?
```

---

## ❌ Problem 4 — Code changed but output is unchanged

Most likely:

```text
Code modified
     ↓
❌ Build forgotten
     ↓
Old binary still running
```

Solution:

```text
Build → Debug → Run
```

---

# 20. Practical Experiment

## 🧪 Experiment 1 — Live Counter

```c
uint16_t counter = 0;

while (1)
{
    counter++;

    HAL_Delay(2000);
}
```

Monitor:

```text
counter
```

Expected:

```text
0
1
2
3
4
5
...
```

---

## 🧪 Experiment 2 — SWV Counter

```c
uint16_t counter = 0;

while (1)
{
    counter++;

    printf("Value of counter = %d\r\n", counter);

    HAL_Delay(2000);
}
```

Expected:

```text
Value of counter = 1
Value of counter = 2
Value of counter = 3
Value of counter = 4
...
```

---

# 21. Real-World Application

This becomes especially useful when working with sensors.

Suppose we have:

```c
uint16_t adc_value;
float temperature;
float voltage;
uint8_t state;
```

### Live Expressions

We can monitor:

```text
adc_value
temperature
voltage
state
```

### SWV

We can print:

```text
ADC = 2356
Temperature = 28.4
Voltage = 3.31
State = 2
```

This is particularly useful for the next stage of STM32 learning because ADC values can be analyzed using the same debugging techniques. The reference explicitly points toward using these debugging methods to analyze ADC values in subsequent lessons. 

---

# 🚀 22. Interview Questions

## Q1. What is Live Expressions?

**Answer:**

Live Expressions is a debugging feature in STM32CubeIDE that allows us to monitor the values of variables or expressions while the MCU is executing.

---

## Q2. What is SWV?

**Answer:**

SWV stands for Serial Wire Viewer. It provides runtime trace and debugging information through the supported Cortex-M debug/trace infrastructure.

---

## Q3. What is ITM?

**Answer:**

ITM stands for Instrumentation Trace Macrocell. It is a Cortex-M trace component that can be used to send instrumentation data from the MCU to the debugger.

---

## Q4. What is SWO?

**Answer:**

SWO stands for Serial Wire Output. It is the trace output used for transmitting SWV information from the target MCU.

---

## Q5. How do you redirect `printf()` to SWV?

**Answer:**

Implement `_write()` and send each character using `ITM_SendChar()`:

```c
int _write(int file, char *ptr, int len)
{
    for (int i = 0; i < len; i++)
    {
        ITM_SendChar(ptr[i]);
    }

    return len;
}
```

---

## Q6. Why do we use ITM_SendChar()?

**Answer:**

`ITM_SendChar()` sends a character through the ITM trace mechanism, allowing it to be viewed through the SWV debugging infrastructure.

---

## Q7. Why is Port 0 enabled?

**Answer:**

The ITM stimulus port used by the character output mechanism needs to be enabled so that the transmitted ITM data can be received by the SWV console.

---

## Q8. Why is `printf()` not working after implementing `_write()`?

Possible reasons:

* Project was not rebuilt
* SWV is not configured
* ITM Port 0 is disabled
* Trace was not started
* Debug session is not running
* Program has not been resumed

---

## Q9. What is the difference between UART debugging and SWV debugging?

### UART

```text
MCU
 ↓
UART TX
 ↓
USB-UART
 ↓
PC Terminal
```

### SWV

```text
MCU
 ↓
ITM
 ↓
SWO
 ↓
ST-LINK
 ↓
STM32CubeIDE
```

UART is a normal communication peripheral, while SWV uses the MCU's debug/trace infrastructure.

---

# 🧠 23. Quick Revision

## Live Expressions

> **Monitor variable values during debugging.**

```text
counter → 1 → 2 → 3 → 4
```

---

## SWD

> **Serial Wire Debug**

Used for:

```text
Programming
Debugging
```

---

## SWV

> **Serial Wire Viewer**

Used for:

```text
Runtime debugging
Trace
Data output
```

---

## SWO

> **Serial Wire Output**

Trace output path.

---

## ITM

> **Instrumentation Trace Macrocell**

Provides instrumentation/trace functionality.

---

## ITM_SendChar()

```c
ITM_SendChar(ch);
```

Sends a character through ITM.

---

## `_write()`

Used as a low-level output function for `printf()`.

```text
printf()
   ↓
_write()
   ↓
ITM_SendChar()
```

---

## ITM Data Console

Used to view the SWV/ITM output in STM32CubeIDE.

---

# ⭐ Most Important Flow

Memorize this:

```text
             LIVE EXPRESSIONS
                    │
                    ▼
             Monitor Variables


              printf()
                  │
                  ▼
               _write()
                  │
                  ▼
           ITM_SendChar()
                  │
                  ▼
                 ITM
                  │
                  ▼
                 SWO
                  │
                  ▼
               ST-LINK
                  │
                  ▼
           STM32CubeIDE
                  │
                  ▼
        SWV ITM Data Console
```

---

# 🎯 24. Learning Progress

### STM32 Debugging Checklist

* [x] Understand why debugging is important
* [x] Understand Live Expressions
* [x] Monitor a variable using Live Expressions
* [x] Understand limitations of Live Expressions
* [x] Understand SWV
* [x] Understand SWO
* [x] Understand ITM
* [x] Configure Serial Wire debugging
* [x] Configure ITM Stimulus Port 0
* [x] Open SWV ITM Data Console
* [x] Implement `_write()`
* [x] Use `ITM_SendChar()`
* [x] Retarget `printf()` to SWV
* [x] Build before debugging
* [x] Start SWV trace
* [x] Analyze runtime values

---



# 🏁 Final Takeaway

> **Debugging is not about finding errors after the program fails. It is about observing what the MCU is actually doing.**

### Live Expressions:

```text
"What is the value of my variable?"
```

### SWV:

```text
"What is my program telling me?"
```

### ITM:

```text
"How do I send debug information?"
```

### `_write()`:

```text
"Where should printf() send its data?"
```

### ITM_SendChar():

```text
"Send this character through the ITM trace system."
```

### Build:

```text
"Make sure my latest code is actually compiled."
```


## 🔥 The Golden Debugging Rule

```text
CHANGE CODE
    ↓
SAVE
    ↓
BUILD
    ↓
DEBUG
    ↓
CONFIGURE TRACE
    ↓
START TRACE
    ↓
RESUME
    ↓
OBSERVE
    ↓
UNDERSTAND
    ↓
FIX
```

> 🚀 **Don't guess what your STM32 is doing — debug it, observe it, understand it.**

