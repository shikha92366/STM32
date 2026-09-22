
# 09_CMSIS_GPIO_Input_Button

> **STM32F401RE • CMSIS • Register-Level GPIO • Button Input • Bit Manipulation**

This repository is a continuation of the previous **CMSIS Programming** repository.

In the previous repo, we controlled an LED using GPIO registers.  
Here, we extend that knowledge to work with a **GPIO input** and interface the **user push button** with the STM32.

---

## 📌 What You Will Learn

- GPIO Input configuration using CMSIS
- GPIO Output configuration
- RCC clock enable
- `MODER` register configuration
- `IDR` — Input Data Register
- `ODR` — Output Data Register
- Bit masking
- Bit shifting
- `AND`, `OR`, and `NOT` operations
- Reading a specific GPIO pin
- Controlling an LED using a push button
- Writing register-level code in a cleaner/professional way

---

## 🧠 Hardware Used

| Component | STM32 Pin |
|---|---|
| User LED (LD2) | `PA5` |
| User Button (B1) | `PC13` |
| MCU | STM32F401RE |
| Board | NUCLEO-F401RE |

### Basic Operation

```text
        Push Button
            │
            ▼
          PC13
            │
            ▼
       GPIOC → IDR
            │
            │ Read Input
            ▼
       Program Logic
            │
            ▼
       GPIOA → ODR
            │
            ▼
          PA5
            │
            ▼
           LED
````

---

# 1. GPIO Clock Enable

Before using a GPIO peripheral, its clock must be enabled through the RCC.

### GPIOA

```c
RCC->AHB1ENR |= (1U << 0);
```

### GPIOC

```c
RCC->AHB1ENR |= (1U << 2);
```

### Why?

```text
RCC
 │
 ├── GPIOA Clock
 │
 └── GPIOC Clock
```

Without enabling the peripheral clock, GPIO registers cannot be used normally.

---

# 2. GPIO Mode Configuration

The `MODER` register controls the mode of each GPIO pin.

Each GPIO pin uses **2 bits** in `MODER`.

| MODER Bits | Mode                   |
| ---------- | ---------------------- |
| `00`       | Input                  |
| `01`       | General Purpose Output |
| `10`       | Alternate Function     |
| `11`       | Analog                 |

---

## 📍 PA5 as Output

PA5 is the LED pin.

Each pin occupies two bits:

```text
Pin 5 → 2 × 5 = Bit 10, 11
```

### Clear the two mode bits

```c
GPIOA->MODER &= ~(3U << (2U * 5U));
```

### Set Output Mode

```c
GPIOA->MODER |= (1U << (2U * 5U));
```

So:

```text
PA5
 │
 └── MODER[11:10] = 01
             ↓
          Output
```

---

# 3. PC13 as Input

The user button is connected to `PC13`.

For pin 13:

```text
2 × 13 = 26
```

Therefore, PC13 uses:

```text
MODER[27:26]
```

To configure PC13 as input:

```c
GPIOC->MODER &= ~(3U << (2U * 13U));
```

This makes:

```text
MODER[27:26] = 00
```

Therefore:

```text
PC13 → INPUT
```

---

# 4. Why `3U`?

Two bits are used for every GPIO pin.

Two bits containing `1` are:

```text
11₂ = 3₁₀
```

Therefore:

```c
3U
```

represents:

```text
00000011
```

When shifted:

```c
3U << (2U * 13U)
```

the `11` moves to the position of PC13.

---

# 5. Bit Masking

Bit masking allows us to modify specific bits without affecting other bits.

### Clear bits

```c
GPIOC->MODER &= ~(3U << (2U * 13U));
```

Meaning:

```text
AND with 0 → Bit becomes 0
AND with 1 → Bit remains unchanged
```

### Set a bit

```c
GPIOA->ODR |= (1U << 5U);
```

Meaning:

```text
OR with 1 → Bit becomes 1
OR with 0 → Bit remains unchanged
```

---

# 6. ODR — Output Data Register

`ODR` is used to control GPIO output states.

For PA5:

```text
PA5 → ODR bit 5
```

### LED ON

```c
GPIOA->ODR |= (1U << 5U);
```

### LED OFF

```c
GPIOA->ODR &= ~(1U << 5U);
```

### Important

Avoid blindly writing the entire `ODR` when you only want to change one pin.

Prefer:

```c
GPIOA->ODR |= (1U << 5U);
```

or

```c
GPIOA->ODR &= ~(1U << 5U);
```

because these modify only the required bit.

---

# 7. IDR — Input Data Register

`IDR` means:

> **Input Data Register**

It is used to read the current state of GPIO input pins.

For the user button:

```text
PC13 → GPIOC → IDR bit 13
```

To check PC13:

```c
GPIOC->IDR & (1U << 13U)
```

---

# 8. Reading PC13

```c
if (GPIOC->IDR & (1U << 13U))
{
    // PC13 is HIGH
}
else
{
    // PC13 is LOW
}
```

### Why `&`?

The `AND` operation isolates only bit 13.

Example:

```text
IDR:
xxxx xxxx xxxx xxxx
              ↑
             PC13

Mask:
0010 0000 0000 0000
              ↑
             Bit 13
```

The result tells us whether PC13 is:

```text
HIGH → 1
LOW  → 0
```

---

# 9. Button Controlled LED

The basic logic is:

```text
          PC13
           │
           ▼
          IDR
           │
      Read Button
           │
     ┌─────┴─────┐
     │           │
   HIGH         LOW
     │           │
     ▼           ▼
   LED ON      LED OFF
```

### Code

```c
while (1)
{
    if (GPIOC->IDR & (1U << 13U))
    {
        GPIOA->ODR |= (1U << 5U);
    }
    else
    {
        GPIOA->ODR &= ~(1U << 5U);
    }
}
```

---

# 10. Complete CMSIS Program

```c
#include "stm32f401xe.h"

int main(void)
{
    /* Enable GPIOA and GPIOC clocks */
    RCC->AHB1ENR |= (1U << 0);   // GPIOA
    RCC->AHB1ENR |= (1U << 2);   // GPIOC

    /* PA5 -> Output */
    GPIOA->MODER &= ~(3U << (2U * 5U));
    GPIOA->MODER |=  (1U << (2U * 5U));

    /* PC13 -> Input */
    GPIOC->MODER &= ~(3U << (2U * 13U));

    /* Initial LED state = OFF */
    GPIOA->ODR &= ~(1U << 5U);

    while (1)
    {
        /* Read PC13 */
        if (GPIOC->IDR & (1U << 13U))
        {
            /* Button HIGH */
            GPIOA->ODR |= (1U << 5U);
        }
        else
        {
            /* Button LOW */
            GPIOA->ODR &= ~(1U << 5U);
        }
    }
}
```

---

# 🔍 Register Summary

| Register       | Purpose                 | Used For     |
| -------------- | ----------------------- | ------------ |
| `RCC->AHB1ENR` | Enable peripheral clock | GPIOA, GPIOC |
| `GPIOA->MODER` | Configure pin mode      | PA5 Output   |
| `GPIOC->MODER` | Configure pin mode      | PC13 Input   |
| `GPIOA->ODR`   | Control output          | LED          |
| `GPIOC->IDR`   | Read input              | Button       |

---

# 🧩 Important Bit Operations

### Shift

```c
1U << 5U
```

Moves `1` to bit 5.

```text
00000001
    ↓
00100000
```

---

### OR `|`

Used mainly to **set bits**.

```c
GPIOA->ODR |= (1U << 5U);
```

```text
0 OR 1 = 1
1 OR 1 = 1
```

---

### AND `&`

Used to **check or preserve bits**.

```c
GPIOC->IDR & (1U << 13U)
```

---

### NOT `~`

Used to create an inverse mask.

```c
~(1U << 5U)
```

Useful for clearing a bit.

---

### AND + NOT

```c
GPIOA->ODR &= ~(1U << 5U);
```

This clears bit 5.

```text
Bit 5 → 0
Other bits → unchanged
```

---

# ⭐ Professional Register-Level Pattern

Instead of manually calculating large hexadecimal masks every time:

```c
GPIOA->MODER &= ~(3U << (2U * 5U));
```

This is easier to understand and maintain.

### General formula

For a GPIO pin `n`:

```c
2U * n
```

gives the starting bit position in `MODER`.

### Example

For PC13:

```c
2U * 13U = 26
```

Therefore:

```c
GPIOC->MODER &= ~(3U << 26U);
```

---

# 🧠 Quick Revision

```text
RCC
 ↓
Enable GPIO Clock
 ↓
MODER
 ↓
Configure Pin Mode
 ↓
IDR / ODR
 ↓
Read Input / Control Output
```

### LED

```text
PA5
 ↓
MODER → Output
 ↓
ODR → Control LED
```

### Button

```text
PC13
 ↓
MODER → Input
 ↓
IDR → Read Button
```

---

# ⚠️ Common Mistakes

### 1. Forgetting GPIO clock

```c
RCC->AHB1ENR |= ...
```

must be done before GPIO configuration.

### 2. Wrong pin number

```text
PA5 → 5
PC13 → 13
```

### 3. Forgetting that MODER uses 2 bits per pin

```text
Pin n → bits (2n + 1 : 2n)
```

### 4. Using the wrong register

```text
Input  → IDR
Output → ODR
Mode   → MODER
Clock  → RCC
```

### 5. Accidentally changing other pins

Prefer bit masking:

```c
GPIOA->ODR |= (1U << 5U);
```

instead of rewriting the entire register.

---

# 📚 Key Takeaways

* **RCC** enables the GPIO peripheral clock.
* **MODER** selects Input/Output/AF/Analog mode.
* **IDR** reads the physical input state.
* **ODR** controls GPIO output state.
* Every GPIO pin uses **2 bits in MODER**.
* Bit shifting makes register manipulation easier.
* `|=` is commonly used to set bits.
* `&=` with an inverted mask is commonly used to clear bits.
* `&` can be used to test a specific input bit.
* CMSIS provides a cleaner way to access STM32 peripheral registers while still working close to the hardware.

---
