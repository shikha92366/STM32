# 📘 STM32 ADC — Polling Mode

### Analog → Digital Conversion | STM32 HAL | Sensor Interfacing

> **Learning Path:** GPIO → CMSIS → UART → Debugging → **ADC** → ADC Interrupt → ADC DMA → Advanced Peripherals

Welcome to the next step in the STM32 learning journey! 🚀

In this repository, we learn how an STM32 reads a **real-world analog signal** such as a sensor voltage and converts it into a **digital number** that firmware can process.

The main focus here is **ADC using Polling Mode** with STM32 HAL.

---

# 🧠 1. What is an ADC?

**ADC = Analog-to-Digital Converter**

Microcontrollers understand digital values (`0` and `1`), but the real world is mostly analog.

For example:

* 🌡️ Temperature sensor → analog voltage
* 📏 Distance sensor → analog voltage
* 💡 LDR → varying voltage
* 🎚️ Potentiometer → varying voltage
* 🔋 Battery → voltage
* 🎤 Microphone → analog signal

The ADC converts:

```text
Analog Voltage
      ↓
     ADC
      ↓
Digital Number
      ↓
Firmware Processing
      ↓
Temperature / Distance / Light / Voltage etc.
```

The reference material explains this exact idea: sensors can provide information as a voltage, and the ADC allows the STM32 to read that voltage as a digital value. 

---

# 🔌 2. Real-World Example

Suppose we have a sensor:

```text
          Sensor
        ┌─────────┐
VCC ───►│         │
GND ───►│ Sensor  │
        │         │
OUT ───►│         │
        └────┬────┘
             │
             │ Analog Voltage
             ▼
        STM32 ADC Pin
             │
             ▼
       Digital ADC Value
```

The sensor may output:

```text
0.5 V
1.2 V
1.8 V
2.7 V
...
```

The STM32 ADC converts these voltages into digital values.

---

# 🎯 3. The ADC Pipeline

Think of ADC like a translator:

```text
REAL WORLD
   │
   ▼
Sensor
   │
   │ Analog Voltage
   ▼
┌─────────────┐
│ STM32 ADC   │
└─────────────┘
   │
   │ Digital Value
   ▼
Firmware
   │
   ▼
Meaningful Data
```

For example:

```text
ADC = 1000
      ↓
Firmware calculation
      ↓
Temperature = 25°C
```

The ADC itself doesn't know that `1000` means `25°C`.

**Your firmware/application algorithm gives the ADC value meaning.**

---

# 🔢 4. ADC Resolution

One of the most important ADC concepts is **resolution**.

The reference example uses a **12-bit ADC**. 

For an `N-bit` ADC:

$$
Number\ of\ levels = 2^N
$$

For a 12-bit ADC:

$$
2^{12}=4096
$$

Therefore the output range is:

```text
0 → 4095
```

because:

$$
Maximum\ ADC\ Value = 2^{12}-1
$$

$$
=4095
$$

### 🧠 Remember

```text
8-bit ADC  → 0 to 255
10-bit ADC → 0 to 1023
12-bit ADC → 0 to 4095
16-bit ADC → 0 to 65535
```

### 🔥 Interview Question

**Q: Why is the maximum value of a 12-bit ADC 4095 and not 4096?**

Because counting starts from zero:

```text
0, 1, 2, ........ 4095
```

There are still **4096 different values**.

---

# 📐 5. ADC Voltage Relationship

For an ideal ADC:

$$
ADC_{value}
=
\frac{V_{IN}}{V_{REF}}
(2^N-1)
$$

For a 12-bit ADC:

$$
ADC_{value}
=
\frac{V_{IN}}{V_{REF}}
\times4095
$$

And approximately:

$$
V_{IN}
=
\frac{ADC_{value}\times V_{REF}}{4095}
$$

### Example

Assume:

```text
VREF = 3.3 V
ADC = 2048
```

Then:

$$
V_{IN}
=
\frac{2048\times3.3}{4095}
$$

Approximately:

```text
VIN ≈ 1.65 V
```

So:

```text
0 V       → ~0
1.65 V    → ~2048
3.3 V     → ~4095
```

---

# ⚠️ IMPORTANT HARDWARE NOTE

The reference demonstration mentions applying **5 V** to an ADC input while observing the maximum ADC value. 

**Do NOT assume an STM32 ADC pin can safely accept 5 V.**

For most STM32 systems using a 3.3 V supply, the ADC input must remain within the MCU's specified analog-input range, typically related to `VDDA/VREF+`.

If you need to measure a voltage higher than the ADC range, use an appropriate:

```text
Voltage Divider
       ↓
ADC Pin
```

For example:

```text
Battery
  │
 R1
  │──────► ADC
 R2
  │
 GND
```

Always check the **specific STM32 datasheet** before connecting an external voltage.

---

# 🧩 6. Three Ways to Read ADC Data

There are three important methods discussed in the reference:

```text
                 ADC Data
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
   POLLING       INTERRUPT       DMA
       │            │            │
     Simple       Async         Fast
     Blocking     Efficient      Efficient
```

The reference identifies these three approaches as:

1. Polling
2. Interrupt
3. DMA 

---

# 🟢 7. ADC Polling

Polling is the simplest method to understand.

The CPU basically says:

> **"ADC, give me the conversion result. I'll wait here."**

Conceptually:

```text
CPU
 │
 ├── Start ADC
 │
 ├── Wait...
 │
 ├── Wait...
 │
 ├── Conversion complete?
 │
 ├── YES
 │
 └── Read ADC value
```

During the waiting operation, CPU execution is blocked.

The reference describes polling as easy to implement but slower because the CPU waits for the conversion. 

---

# 🆚 Polling vs Interrupt vs DMA

| Method       | CPU Waiting? | Complexity | Suitable For                  |
| ------------ | ------------ | ---------- | ----------------------------- |
| 🟢 Polling   | Yes          | Low        | Simple/slow measurements      |
| 🟡 Interrupt | No           | Medium     | Periodic/event-based sampling |
| 🔵 DMA       | Minimal      | Higher     | High-speed continuous data    |

### When should you use polling?

Good examples:

```text
✔ Read potentiometer occasionally
✔ Read a slow sensor
✔ One-time measurement
✔ Simple learning projects
✔ Debugging ADC
```

Not ideal for:

```text
✘ High-speed sampling
✘ Audio
✘ Large ADC buffers
✘ Real-time waveform acquisition
```

The reference specifically discusses DMA for high-speed applications such as audio, high-speed sensors and real-time data acquisition. 

---

# 🧠 8. Polling Analogy

Imagine ordering food 🍔.

### Polling

You stand at the counter asking:

> "Is my food ready?"

```text
Are you ready?
NO

Are you ready?
NO

Are you ready?
NO

Are you ready?
YES!
```

You're wasting your time waiting.

### Interrupt

You give your phone number:

> "Call me when the food is ready."

You can do something else.

### DMA

You tell the delivery system:

> "Keep delivering the food directly to the storage area."

CPU doesn't need to handle every transfer.

---

# 🛠️ 9. STM32CubeMX Configuration

For the basic polling experiment:

### Step 1 — Select your STM32 board

Open:

```text
STM32CubeMX / STM32CubeIDE
```

Select the appropriate MCU/board.

---

### Step 2 — Enable ADC

Go to:

```text
Pinout & Configuration
        ↓
Analog
        ↓
ADC
```

Select the required ADC channel.

For example:

```text
ADC1_IN0
```

The reference demonstration uses the first ADC channel, `ADC1_IN0`. 

---

# ⚙️ 10. Important ADC Settings

For this learning experiment:

```text
ADC
├── Resolution       → 12-bit
├── Conversion Mode  → Polling
├── Interrupt        → OFF
├── DMA              → OFF
└── Channel          → ADC input channel
```

The reference specifically starts with polling and does not enable interrupt or DMA. 

---

# 📦 11. What CubeMX Generates?

When ADC is enabled, CubeMX creates an ADC handle.

Example:

```c
ADC_HandleTypeDef hadc1;
```

Think of the handle as the software object used by HAL to control ADC1.

```text
hadc1
  │
  ├── ADC configuration
  ├── ADC state
  ├── ADC registers/interface
  └── HAL control
```

---

# 💻 12. Basic ADC Polling Code

The basic HAL flow is:

```c
uint32_t value = 0;

while (1)
{
    HAL_ADC_Start(&hadc1);

    HAL_ADC_PollForConversion(&hadc1, 20);

    value = HAL_ADC_GetValue(&hadc1);

    HAL_Delay(1000);
}
```

### The three important functions are:

```c
HAL_ADC_Start()
```

↓

```c
HAL_ADC_PollForConversion()
```

↓

```c
HAL_ADC_GetValue()
```

---

# 🔍 13. Understand Every Function

## ① `HAL_ADC_Start()`

```c
HAL_ADC_Start(&hadc1);
```

Starts the ADC conversion process.

Think:

> **"ADC, start working!"**

---

## ② `HAL_ADC_PollForConversion()`

```c
HAL_ADC_PollForConversion(&hadc1, 20);
```

Waits until the ADC conversion is complete.

The second argument is the timeout.

```text
20
↓
timeout value
```

The reference demonstrates this polling step with a timeout value. 

---

## ③ `HAL_ADC_GetValue()`

```c
value = HAL_ADC_GetValue(&hadc1);
```

Reads the converted ADC result.

Example:

```text
ADC → 0
ADC → 512
ADC → 2048
ADC → 3500
ADC → 4095
```

The value is stored in our variable.

---

# 🔄 14. Complete Execution Flow

```text
                START
                  │
                  ▼
         HAL_ADC_Start()
                  │
                  ▼
    Start ADC Conversion
                  │
                  ▼
   HAL_ADC_PollForConversion()
                  │
             Conversion?
             /          \
           NO            YES
           │              │
           └──────┐       ▼
                  │  HAL_ADC_GetValue()
                  │       │
                  │       ▼
                  │   ADC Result
                  │       │
                  └──────►▼
                         Delay
                           │
                           ▼
                     Repeat Loop
```

---

# 🧪 15. Example: Potentiometer

A potentiometer is one of the easiest ADC experiments.

```text
        3.3V
         │
         │
      ┌───────┐
      │ POT   │
      └───┬───┘
          │
          ├────────► ADC_IN
          │
         GND
```

Turn the knob:

```text
      ADC
       │
       │        /
       │       /
       │      /
       │     /
       │____/____________ Time
```

The ADC value changes according to the potentiometer voltage.

---

# 💡 16. Example: LDR

An LDR changes resistance depending on light.

Usually we create a voltage divider:

```text
             VCC
              │
             R1
              │
              ├──────────► ADC
              │
             LDR
              │
             GND
```

Then:

```text
Light changes
      ↓
LDR resistance changes
      ↓
Voltage changes
      ↓
ADC value changes
      ↓
Firmware interprets value
```

---

# 📏 17. Example: Analog Distance Sensor

The reference demonstrates an analog IR sensor whose output provides an analog value related to distance. 

Conceptually:

```text
Object
  │
  ▼
IR Sensor
  │
  │ Analog Voltage
  ▼
STM32 ADC
  │
  │ Digital Value
  ▼
Firmware
  │
  ▼
Distance Calculation
```

For example:

```text
ADC = 3000
       ↓
Algorithm
       ↓
Distance ≈ X cm
```

The exact relationship depends on the particular sensor.

---

# 📊 18. What Happens When Voltage Changes?

For a 12-bit ADC:

```text
VIN increases
      ↓
ADC value increases
```

and generally:

```text
VIN decreases
      ↓
ADC value decreases
```

For a 3.3 V reference:

```text
0 V       → ~0
0.825 V   → ~1024
1.65 V    → ~2048
2.475 V   → ~3072
3.3 V     → ~4095
```

These are ideal approximate values.

---

# 🔬 19. ADC Resolution / LSB

The smallest voltage step that can ideally be represented is approximately:

$$
LSB = \frac{V_{REF}}{2^N-1}
$$

For a 12-bit ADC with 3.3 V reference:

$$
LSB \approx \frac{3.3}{4095}
$$

Approximately:

```text
0.806 mV / count
```

So approximately every:

```text
0.806 mV
```

changes the ADC result by one count.

> For practical calculations, always consider the actual STM32 ADC/reference configuration and datasheet specifications.

---

# 🧮 20. Convert ADC Value → Voltage

Use:

$$
V_{IN} =
\frac{ADC \times V_{REF}}{4095}
$$

Example:

```text
ADC = 2500
VREF = 3.3V
```

Therefore:

$$
V_{IN}
=
\frac{2500\times3.3}{4095}
$$

```text
VIN ≈ 2.015 V
```

### C code

```c
float voltage;

voltage = ((float)value * 3.3f) / 4095.0f;
```

Now:

```text
ADC value
    ↓
Voltage
```

---

# 🌡️ 21. Convert ADC → Sensor Parameter

The ADC only gives:

```text
Digital Number
```

Your application converts it into:

```text
Temperature
Distance
Pressure
Light intensity
Battery voltage
etc.
```

Example:

```c
uint32_t adc_value;
float voltage;

HAL_ADC_Start(&hadc1);

HAL_ADC_PollForConversion(&hadc1, 20);

adc_value = HAL_ADC_GetValue(&hadc1);

voltage = ((float)adc_value * 3.3f) / 4095.0f;
```

Then your sensor-specific equation can convert:

```text
Voltage → Physical Quantity
```

---

# 🐛 22. Debugging ADC Using Live Expressions

One useful technique from the reference is checking the ADC variable using **Live Expressions** while debugging. 

Example:

```c
uint32_t value = 0;
```

Add:

```text
value
```

to Live Expressions.

Then observe:

```text
value = 574
value = 621
value = 701
value = 850
...
```

This makes it easy to verify that the ADC is actually responding to the sensor.

---

# 🎯 23. What Should You Observe?

If you connect an analog sensor and change its input:

```text
Sensor changes
      ↓
Analog voltage changes
      ↓
ADC conversion changes
      ↓
value changes
```

For example:

```text
Sensor condition     ADC
──────────────────────────
Low input             200
                       ↓
                       800
                       ↓
                       1500
                       ↓
                       2500
                       ↓
High input            3500
```

The reference demonstrates exactly this kind of behavior with an analog sensor: changing the sensor condition causes the ADC value to increase/decrease. 

---

# 🧠 24. Why `uint32_t`?

A common declaration is:

```c
uint32_t value = 0;
```

A 12-bit ADC result only requires 12 bits.

But using:

```c
uint32_t
```

is convenient because STM32 HAL ADC APIs commonly use a 32-bit unsigned type for the returned ADC value.

So:

```text
ADC result
   ↓
uint32_t
   ↓
0 → 4095
```

---

# 🚨 25. Common Beginner Mistakes

## ❌ Mistake 1 — Applying excessive voltage

Never assume:

```text
5V → ADC pin
```

is safe.

Check your STM32 datasheet and board circuitry first.

---

## ❌ Mistake 2 — Forgetting GND

Sensor and STM32 need a common reference.

```text
Sensor GND ───────── STM32 GND
```

Without a proper common ground, ADC readings can be incorrect.

---

## ❌ Mistake 3 — Wrong ADC channel

You configured:

```text
ADC1_IN0
```

but physically connected the sensor to another ADC pin.

Result:

```text
Wrong / unexpected reading
```

Always match:

```text
Physical Pin
      ↕
ADC Channel
      ↕
CubeMX Configuration
```

---

## ❌ Mistake 4 — Forgetting to start conversion

```c
HAL_ADC_GetValue(&hadc1);
```

doesn't replace starting/polling the conversion.

Understand the sequence:

```c
HAL_ADC_Start();

HAL_ADC_PollForConversion();

HAL_ADC_GetValue();
```

---

## ❌ Mistake 5 — Wrong resolution assumption

If your ADC is configured as:

```text
12-bit
```

don't expect:

```text
0 → 1023
```

That's a 10-bit range.

For 12-bit:

```text
0 → 4095
```

---

# 🧩 26. Polling vs Interrupt vs DMA — Mental Model

### 🟢 Polling

```text
CPU
 │
 ├── Start ADC
 ├── WAIT
 ├── WAIT
 ├── WAIT
 └── Read
```

### 🟡 Interrupt

```text
CPU ───────────────► Other Work
                         │
                         │ ADC completes
                         ▼
                       IRQ
                         │
                         ▼
                       CPU
```

### 🔵 DMA

```text
ADC
 │
 │
 ▼
DMA ─────────► RAM Buffer
                  │
                  │
                  ▼
                 CPU
```

This is one of the most important concepts to remember before moving to **ADC Interrupt** and **ADC DMA**.

---

# 🏗️ 27. When to Use Which?

```text
                    Need ADC?
                       │
                       ▼
             Is measurement slow?
                  /         \
                YES         NO
                 │           │
                 ▼           ▼
             POLLING     Need periodic /
                         asynchronous data?
                            /       \
                          YES       NO
                           │         │
                           ▼         ▼
                       INTERRUPT   High-speed/
                                   continuous?
                                      │
                                      ▼
                                      DMA
```

---

# 🧪 28. Mini Project — ADC Sensor Monitor

### 🎯 Goal

Read an analog sensor using ADC polling and display/observe the value.

### Hardware

```text
STM32
  │
  ├── ADC Input
  │
  ├── GND
  │
  └── 3.3V

Sensor
  │
  ├── VCC
  ├── GND
  └── Analog OUT
```

### Firmware

```c
uint32_t adc_value;
float voltage;

while (1)
{
    HAL_ADC_Start(&hadc1);

    if (HAL_ADC_PollForConversion(&hadc1, 20) == HAL_OK)
    {
        adc_value = HAL_ADC_GetValue(&hadc1);

        voltage = ((float)adc_value * 3.3f) / 4095.0f;
    }

    HAL_Delay(1000);
}
```

---

# 🧠 29. Challenge Yourself

Try these without looking at the answer.

### Challenge 1

A 12-bit ADC has how many levels?

<details>
<summary>💡 Answer</summary>

$$
2^{12}=4096
$$

So:

```text
4096 levels
```

</details>

---

### Challenge 2

What is the maximum digital value?

<details>
<summary>💡 Answer</summary>

```text
4095
```

</details>

---

### Challenge 3

What does `HAL_ADC_PollForConversion()` do?

<details>
<summary>💡 Answer</summary>

It waits/polls until the ADC conversion is completed or the timeout occurs.

</details>

---

### Challenge 4

What does `HAL_ADC_GetValue()` do?

<details>
<summary>💡 Answer</summary>

It retrieves the converted ADC result.

</details>

---

### Challenge 5

Why is polling not ideal for high-speed applications?

<details>
<summary>💡 Answer</summary>

Because the CPU waits for the conversion and is therefore blocked during the polling operation.

</details>

---

# 🎤 30. Interview Questions

### Q1. What is ADC?

**Answer:**
ADC stands for Analog-to-Digital Converter. It converts an analog electrical signal, such as sensor voltage, into a digital value that the microcontroller can process.

---

### Q2. What is ADC resolution?

**Answer:**
Resolution determines the number of discrete digital levels available.

For an `N-bit` ADC:

$$
2^N
$$

A 12-bit ADC provides:

```text
4096 levels
0–4095
```

---

### Q3. Why does a 12-bit ADC have a maximum value of 4095?

**Answer:**

$$
2^{12}-1=4095
$$

because the counting starts from zero.

---

### Q4. What is ADC polling?

**Answer:**
Polling means the CPU waits for the ADC conversion to complete and then reads the result.

---

### Q5. What is the disadvantage of polling?

**Answer:**
The CPU is blocked while waiting for the conversion, so it cannot efficiently perform other tasks during that time.

---

### Q6. Polling vs Interrupt?

**Answer:**

**Polling:**

```text
CPU continuously waits/checks
```

**Interrupt:**

```text
CPU performs other work
        ↓
ADC completes
        ↓
Interrupt occurs
        ↓
CPU handles result
```

---

### Q7. Polling vs DMA?

**Answer:**

Polling requires the CPU to manage the ADC conversion and read the result.

DMA can transfer ADC results directly into memory with minimal CPU involvement, making it suitable for high-speed or continuous acquisition.

---

### Q8. Why is DMA useful for ADC?

**Answer:**
DMA can continuously transfer ADC results into RAM without requiring the CPU to manually handle every conversion.

---

### Q9. What is `VREF`?

**Answer:**
`VREF` is the reference voltage used by the ADC to determine the input voltage range and conversion scale.

---

### Q10. How do you convert ADC value into voltage?

For a 12-bit ADC:

$$
V_{IN}=
\frac{ADC\times V_{REF}}{4095}
$$

---

# 📝 31. Quick Revision Sheet

```text
ADC
│
├── Analog → Digital
│
├── Used for sensor interfacing
│
├── Resolution
│      └── N-bit → 2^N levels
│
├── 12-bit
│      ├── 4096 levels
│      └── 0–4095
│
├── Reading Methods
│      ├── Polling
│      ├── Interrupt
│      └── DMA
│
└── HAL Polling Flow
       │
       ├── HAL_ADC_Start()
       │
       ├── HAL_ADC_PollForConversion()
       │
       └── HAL_ADC_GetValue()
```

---

# ⚡ 32. One-Minute Revision

If you remember only one thing from this repository, remember this:

> **ADC converts an analog voltage into a digital number.**

For a 12-bit ADC:

```text
Analog Input
     ↓
ADC
     ↓
0 → 4095
```

With HAL polling:

```c
HAL_ADC_Start(&hadc1);

HAL_ADC_PollForConversion(&hadc1, 20);

value = HAL_ADC_GetValue(&hadc1);
```

And the three major ways of acquiring ADC data are:

```text
🟢 Polling   → Simple but CPU waits
🟡 Interrupt → CPU gets notified
🔵 DMA       → Data moves directly to memory
```

---

# 🧭 33. What Comes Next?

This repository is the foundation for the next two ADC techniques:

```text
          ADC
           │
           ▼
   ┌────────────────┐
   │    POLLING     │ ← YOU ARE HERE
   └───────┬────────┘
           │
           ▼
   ┌────────────────┐
   │    INTERRUPT   │
   └───────┬────────┘
           │
           ▼
   ┌────────────────┐
   │      DMA       │
   └────────────────┘
```

So don't just memorize the HAL functions.

Understand **why polling works, why it blocks the CPU, and why interrupt/DMA become important as the application gets faster and more complex.**

---

## 📚 Learning Checklist

* [ ] Understand analog vs digital signals
* [ ] Understand what ADC does
* [ ] Understand ADC resolution
* [ ] Calculate `2^N`
* [ ] Remember 12-bit → `0–4095`
* [ ] Understand `VREF`
* [ ] Understand ADC → voltage conversion
* [ ] Configure ADC in CubeMX
* [ ] Select ADC channel
* [ ] Understand `ADC_HandleTypeDef`
* [ ] Use `HAL_ADC_Start()`
* [ ] Use `HAL_ADC_PollForConversion()`
* [ ] Use `HAL_ADC_GetValue()`
* [ ] Debug ADC using Live Expressions
* [ ] Interface an analog sensor
* [ ] Understand polling limitations
* [ ] Know when to use polling vs interrupt vs DMA
* [ ] Be ready for ADC interview questions

---

## 🚀 Repository Goal

> **Don't just make the ADC work. Understand what happens between the sensor pin and the number you see in the debugger.**

**Next:** `ADC Interrupt Mode` ⚡
