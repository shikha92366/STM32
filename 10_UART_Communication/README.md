
# 10_UART_Communication 🚀

> A practical and beginner-friendly guide to **UART/USART communication on STM32**, progressing from basic concepts to actual serial communication and debugging.

This repository is a continuation of my STM32 learning journey.

Here, I will learn how an STM32 microcontroller communicates with external devices and a PC using **UART/USART**.

---

## 📚 What You'll Learn

- What is UART?
- UART full form and meaning
- Synchronous vs Asynchronous communication
- TX and RX
- UART wiring
- Baud Rate
- UART Data Frame
- Start Bit
- Data Bits / Word Length
- Parity Bit
- Stop Bit
- UART vs USART
- STM32 Alternate Function (AF)
- USART2 on STM32F401
- PA2 → USART2_TX
- PA3 → USART2_RX
- UART configuration using STM32CubeMX
- UART communication using HAL
- NVIC and UART interrupts
- `HAL_UART_Transmit()`
- Serial Monitor
- PuTTY / Linux Screen
- UART-based debugging
- Basic UART troubleshooting

---

# 1. What is UART?

**UART** stands for:

> **Universal Asynchronous Receiver/Transmitter**

UART is a serial communication method used to exchange data between two devices.

It is commonly used to communicate with:

- Bluetooth modules
- GPS/GNSS modules
- GSM modules
- Wi-Fi modules
- USB-to-UART converters
- PCs
- Other microcontrollers

UART is especially useful in embedded systems because it provides a simple way to send and receive data.

---

# 2. Meaning of UART

## Universal

UART can communicate with many different types of devices.

The communication method is not restricted to one particular device or manufacturer.

## Asynchronous

UART does **not use a separate clock line** between the two communicating devices.

Instead, both devices must agree on communication parameters such as the **baud rate**.

## Receiver/Transmitter

UART contains:

- **TX → Transmitter**
- **RX → Receiver**

The transmitter sends data.

The receiver receives data.

---

# 3. Basic UART Wiring 🔌

UART mainly uses two communication lines:

| Pin | Meaning | Function |
|---|---|---|
| TX | Transmit | Sends data |
| RX | Receive | Receives data |

### Important Rule

UART connections are crossed:

```text
Device A                  Device B

TX  --------------------> RX
RX  <--------------------  TX
GND ---------------------- GND
````

### Remember

> **TX of one device connects to RX of the other device.**

---

# 4. Why is UART Called Serial Communication?

UART sends data **bit-by-bit** over a communication line.

Example:

```text
10110010
```

Instead of sending all bits simultaneously, the bits are transmitted sequentially.

```text
Bit → Bit → Bit → Bit → Bit → Bit → Bit → Bit
```

This reduces the number of communication wires.

---

# 5. UART is Asynchronous

Unlike protocols such as SPI, UART does not normally use a dedicated clock line.

### SPI

```text
SCLK
MOSI
MISO
CS
```

A clock signal is used to synchronize communication.

### UART

```text
TX
RX
```

There is no separate clock line.

Instead, both devices use the same configured communication speed.

---

# 6. Baud Rate ⚡

The **baud rate** determines the communication speed.

Common UART baud rates include:

```text
9600
19200
38400
57600
115200
```

A common configuration used in this project is:

```text
115200 baud
```

### Important

Both communicating devices must use compatible communication settings.

For example:

```text
STM32                 PC
115200 baud    ↔     115200 baud
```

If the communication settings do not match, the received data may appear corrupted.

---

# 7. UART Data Frame

UART communication sends data using a **frame**.

A basic UART frame contains:

```text
START | DATA BITS | PARITY | STOP
```

Example:

```text
      Data
       ↓
START  10101010  STOP
  ↓               ↓
```

The exact configuration depends on the UART settings.

---

# 8. Start Bit

The **start bit** indicates the beginning of a UART frame.

Conceptually:

```text
Idle → START → DATA → STOP → Idle
```

The receiver detects the beginning of a new frame using the start bit.

---

# 9. Data Bits / Word Length

The actual information being transmitted is contained in the data bits.

Typical configurations include:

```text
7 bits
8 bits
9 bits
```

A commonly used configuration is:

```text
Word Length = 8 bits
```

---

# 10. Parity Bit

A parity bit can be used for basic error detection.

Common options:

```text
None
Even
Odd
```

### Even Parity

The parity bit is selected so that the total number of `1`s follows the even-parity rule.

### Odd Parity

The parity bit is selected so that the total number of `1`s follows the odd-parity rule.

For basic UART communication, a common configuration is:

```text
Parity = None
```

---

# 11. Stop Bit

The stop bit indicates the end of the UART frame.

Common options include:

```text
1 Stop Bit
2 Stop Bits
```

A common configuration is:

```text
Stop Bits = 1
```

---

# 12. Common UART Configuration

A typical UART configuration can look like:

```text
Baud Rate      : 115200
Word Length    : 8 Bits
Parity         : None
Stop Bits      : 1
Mode           : Asynchronous
```

This is often written as:

> **115200, 8-N-1**

Meaning:

```text
115200 baud
8 data bits
No parity
1 stop bit
```

---

# 13. UART vs USART

STM32 devices may provide **USART** peripherals.

USART stands for:

> **Universal Synchronous/Asynchronous Receiver/Transmitter**

Unlike UART, USART can support both:

```text
Synchronous
Asynchronous
```

For our basic serial communication, we use:

```text
USART → Asynchronous Mode
```

---

# 14. STM32 USART2 Example

For the STM32F401, one example configuration is:

```text
USART2
```

The corresponding pins are:

```text
PA2 → USART2_TX
PA3 → USART2_RX
```

So the basic connection is:

```text
STM32F401

PA2 → TX
PA3 → RX
```

---

# 15. Alternate Function Concept 🔄

STM32 GPIO pins can perform different functions.

A GPIO pin is not restricted to only:

```text
Input
Output
```

It can also operate in:

> **Alternate Function Mode**

For example:

```text
PA2
 │
 └── Alternate Function
       │
       └── USART2_TX
```

and:

```text
PA3
 │
 └── Alternate Function
       │
       └── USART2_RX
```

The alternate function configuration connects the GPIO pin to the required peripheral.

---

# 16. Why Alternate Function is Required?

Consider PA2.

PA2 can operate as a normal GPIO, but it can also be connected to a peripheral function.

For UART communication:

```text
PA2 → USART2_TX
PA3 → USART2_RX
```

Therefore, the pins must be configured for the appropriate alternate function.

---

# 17. STM32CubeMX Configuration

Basic procedure:

```text
Create STM32 Project
        ↓
Select MCU / Board
        ↓
Open Pinout & Configuration
        ↓
Connectivity
        ↓
USART2
        ↓
Select Asynchronous Mode
        ↓
Configure Parameters
        ↓
Configure NVIC if required
        ↓
Generate Code
```

---

# 18. USART2 Configuration

In STM32CubeMX:

```text
USART2
   ↓
Mode
   ↓
Asynchronous
```

The corresponding TX/RX pins become configured for USART2.

Example:

```text
PA2 → USART2_TX
PA3 → USART2_RX
```

---

# 19. UART Parameters

Typical configuration:

```text
Baud Rate     = 115200
Word Length   = 8 Bits
Parity        = None
Stop Bits     = 1
Mode          = TX/RX
```

These settings should match the serial terminal.

---

# 20. UART Handle

When code is generated using HAL, STM32CubeMX creates a UART handle.

Example:

```c
UART_HandleTypeDef huart2;
```

This handle is used by HAL UART functions.

---

# 21. UART Initialization

CubeMX generates the initialization code.

Conceptually:

```c
MX_USART2_UART_Init();
```

This initializes USART2 according to the configuration.

---

# 22. Sending Data Using HAL

The HAL library provides:

```c
HAL_UART_Transmit()
```

Basic structure:

```c
HAL_UART_Transmit(
    &huart2,
    data,
    size,
    timeout
);
```

The parameters represent:

| Parameter | Meaning              |
| --------- | -------------------- |
| `&huart2` | UART handle          |
| `data`    | Address of data      |
| `size`    | Number of bytes      |
| `timeout` | Maximum waiting time |

---

# 23. Simple UART Transmit Example

```c
char data[] = "Hello Code2Win\r\n";

while (1)
{
    HAL_UART_Transmit(
        &huart2,
        (uint8_t*)data,
        sizeof(data),
        HAL_MAX_DELAY
    );

    HAL_Delay(1000);
}
```

### What happens?

```text
STM32
  │
  │ UART
  ↓
USB/Serial Interface
  │
  ↓
PC Serial Terminal
```

The message appears repeatedly on the serial terminal.

---

# 24. Why `(uint8_t*)` is Used?

Our data is declared as:

```c
char data[];
```

But the HAL transmit function expects a byte-oriented pointer.

Therefore, the pointer can be type-cast:

```c
(uint8_t*)data
```

This tells the compiler to treat the data as bytes for transmission.

---

# 25. Finding HAL Function Definitions

Instead of memorizing every function, inspect the HAL driver.

Typical path:

```text
Drivers
   ↓
STM32F4xx_HAL_Driver
   ↓
Inc
   ↓
stm32f4xx_hal_uart.h
```

Here you can inspect:

```c
HAL_UART_Transmit()
HAL_UART_Receive()
HAL_UART_Transmit_IT()
HAL_UART_Receive_IT()
```

This is a useful habit while learning embedded development.

---

# 26. `HAL_MAX_DELAY`

For the timeout parameter, HAL provides:

```c
HAL_MAX_DELAY
```

Example:

```c
HAL_UART_Transmit(
    &huart2,
    (uint8_t*)data,
    sizeof(data),
    HAL_MAX_DELAY
);
```

It allows the function to wait for the transmission without using a short application-defined timeout.

---

# 27. Adding a New Line

When sending text to a serial terminal:

```c
"\r\n"
```

is commonly used.

### `\r`

Carriage Return

Moves the cursor back to the beginning of the line.

### `\n`

New Line

Moves the cursor to the next line.

Therefore:

```c
"Hello\r\n"
```

produces:

```text
Hello
```

and the next message starts on a new line.

---

# 28. UART Interrupts

UART can also work using interrupts.

The basic idea is:

```text
Normal Program
      │
      ↓
UART Event
      │
      ↓
Interrupt
      │
      ↓
UART Handler
      │
      ↓
Return to Program
```

This allows the MCU to respond to UART events without continuously polling the peripheral.

---

# 29. NVIC

NVIC stands for:

> **Nested Vectored Interrupt Controller**

It manages interrupts inside the Cortex-M processor.

For UART interrupt-based communication, the corresponding interrupt needs to be enabled/configured as required.

In CubeMX:

```text
USART2
   ↓
NVIC Settings
   ↓
Enable required USART2 interrupt
```

---

# 30. Polling vs Interrupt

### Polling

The CPU continuously checks whether UART data is ready.

```text
CPU
 ↓
Check UART
 ↓
Check UART
 ↓
Check UART
 ↓
Process data
```

### Interrupt

The CPU can perform other work and respond when the UART generates an interrupt.

```text
CPU doing normal work
        ↓
UART event
        ↓
Interrupt
        ↓
UART processing
        ↓
Return
```

---

# 31. UART with PC 💻

One of the simplest applications is:

> **STM32 → PC**

```text
STM32
  │
  │ UART
  ↓
USB-to-UART / Board USB Interface
  │
  ↓
PC
  │
  ↓
Serial Terminal
```

The PC acts as the receiving device.

---

# 32. Serial Terminal

A serial terminal can be used to view UART data.

Examples:

* PuTTY
* Screen
* Other serial terminal applications

For Windows, PuTTY can be configured using:

```text
Connection Type → Serial
COM Port        → STM32 COM Port
Speed           → 115200
```

---

# 33. Linux `screen`

On Linux, a serial terminal can be opened using:

```bash
sudo screen /dev/ttyACM0 115200
```

The exact device name may vary.

For example:

```text
/dev/ttyACM0
/dev/ttyUSB0
```

---

# 34. Serial Monitor Configuration

The terminal settings must match the STM32 UART settings.

Example:

```text
Baud Rate  → 115200
Data Bits  → 8
Parity     → None
Stop Bits  → 1
```

If the settings do not match, communication may not work correctly.

---

# 35. UART as a Debugging Tool 🐛

UART is extremely useful for embedded debugging.

Unlike a normal desktop C program, an STM32 application does not automatically have a normal terminal for:

```c
printf()
```

UART can be used to send debugging messages.

Example:

```c
char msg[] = "System Started\r\n";

HAL_UART_Transmit(
    &huart2,
    (uint8_t*)msg,
    sizeof(msg),
    HAL_MAX_DELAY
);
```

The message can then be viewed on the PC.

---

# 36. Practical Debugging Example

Suppose your system contains:

```text
STM32
 ├── Sensor
 ├── GPS
 ├── Bluetooth
 └── UART
```

You can send debugging information:

```text
System Started
GPS Initialized
Sensor Initialized
GPS Data Received
Temperature = 25.4 C
```

This makes it easier to understand what the firmware is doing.

---

# 37. UART with External Modules

Once UART is understood, it can be used to interface STM32 with many modules.

### Bluetooth

```text
STM32 ↔ Bluetooth Module
```

### GPS/GNSS

```text
STM32 ↔ GPS/GNSS Module
```

### GSM

```text
STM32 ↔ GSM Module
```

### Wi-Fi

```text
STM32 ↔ Wi-Fi Module
```

This is one reason UART is an important embedded-systems interface.

---

# 38. UART Communication Flow

```text
                STM32
                  │
          ┌───────┴───────┐
          │    USART2     │
          └───────┬───────┘
                  │
          ┌───────┴───────┐
          │               │
        PA2             PA3
         TX              RX
          │               │
          ↓               ↑
        RX                TX
      External          External
       Device             Device
```

---

# 39. Important Concepts to Remember ⭐

### UART

```text
Universal Asynchronous Receiver/Transmitter
```

### USART

```text
Universal Synchronous/Asynchronous
Receiver/Transmitter
```

### Main UART lines

```text
TX → Transmit
RX → Receive
```

### Connection

```text
TX → RX
RX → TX
```

### Asynchronous communication

```text
No dedicated clock line
```

### Speed

```text
Baud Rate
```

### STM32F401 Example

```text
USART2
PA2 → TX
PA3 → RX
```

### Common configuration

```text
115200
8 Data Bits
No Parity
1 Stop Bit
```

### HAL transmission

```c
HAL_UART_Transmit()
```

### Interrupt controller

```text
NVIC
```

---

# 40. Quick Revision Diagram

```text
             UART / USART
                  │
       ┌──────────┴──────────┐
       │                     │
      TX                    RX
Transmit Data          Receive Data
       │                     │
       └─────────┬───────────┘
                 │
          Asynchronous
          Communication
                 │
            Baud Rate
                 │
        ┌────────┴────────┐
        │                 │
     Data Frame       Configuration
        │                 │
   Start Bit          Baud Rate
   Data Bits          Word Length
   Parity             Parity
   Stop Bit           Stop Bits
```

---

# 41. Practical Project

## 📌 Project: STM32 UART Hello World

### Objective

Send a message from STM32 to a PC using USART2.

### Hardware

* STM32F401 development board
* USB cable
* PC

### Configuration

```text
USART2
Mode       → Asynchronous
Baud Rate  → 115200
Word Length → 8 Bits
Parity     → None
Stop Bits  → 1
```

### Expected Output

```text
Hello Code2Win
Hello Code2Win
Hello Code2Win
Hello Code2Win
```

---

# 42. Learning Progression

```text
UART Basics
     ↓
TX / RX
     ↓
Baud Rate
     ↓
Data Frame
     ↓
USART
     ↓
Alternate Function
     ↓
USART2
     ↓
PA2 / PA3
     ↓
STM32CubeMX
     ↓
HAL UART
     ↓
UART Transmit
     ↓
Serial Terminal
     ↓
UART Debugging
     ↓
UART Interrupts
     ↓
UART + External Modules
```

---

# 43. Common Mistakes ❌

### 1. TX connected to TX

Incorrect:

```text
TX → TX
RX → RX
```

Correct:

```text
TX → RX
RX → TX
```

---

### 2. Different baud rates

STM32:

```text
115200
```

PC:

```text
9600
```

This can cause incorrect communication.

---

### 3. Incorrect UART pins

Always verify the MCU pin mapping.

For the example used here:

```text
PA2 → USART2_TX
PA3 → USART2_RX
```

---

### 4. Forgetting Alternate Function

A GPIO pin must be configured for the appropriate peripheral function.

---

### 5. Incorrect terminal settings

Check:

```text
Baud Rate
Data Bits
Parity
Stop Bits
COM Port
```

---

# 44. Key Takeaways 🧠

> **UART is a simple asynchronous serial communication interface.**

> **TX sends data and RX receives data.**

> **UART does not use a separate clock line.**

> **Both devices must agree on communication settings such as baud rate.**

> **STM32 GPIO pins can be configured in Alternate Function mode to connect them to peripherals.**

> **USART can support both synchronous and asynchronous communication.**

> **UART is extremely useful for communicating with modules and for debugging embedded firmware.**


