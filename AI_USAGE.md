# AI_USAGE.md — Week 03 entries

**AI assistant:** ChatGPT  
**Review date:** 2026-10-17  
**Experiment:** Week 03 · ESP32-S3-DevKitC-1  

**Source of record:** AI responses reconstructed during the October 17 review of the Week 03 exercises; timing and GPIO statistics are from the previously collected device logs. This section can be appended to the existing Week 02 log.

---

## Entry 1 — 250 Hz ECG Sampling Program

### What I asked

> Write me an Arduino program that samples an ECG at 250 Hz, reads analog pin A0, and prints the value over the serial port.

### What it answered

The AI suggested converting 250 Hz to a 4 ms interval and placing `delay(4)` in the main loop:

```cpp
#include <Arduino.h>

const int ECG_PIN = A0;

void setup() {
    Serial.begin(115200);
    pinMode(ECG_PIN, INPUT);
}

void loop() {
    int value = analogRead(ECG_PIN);
    Serial.println(value);
    delay(4);  // 1000 ms / 250 Hz = 4 ms
}
```

The suggested code is simple and appears reasonable because 1 / 250 Hz = 4 ms. However, it assumes that the ADC read and serial output do not add any significant time to each iteration. `delay(4)` is a minimum wait after the preceding work, not a guarantee that samples will be exactly 4 ms apart.

### How I checked

I used the course's instrumented `sketch_a_delay.cpp`, which keeps the delay-based scheduling approach and adds a repeatable simulated workload. The sketch stores 1,000 successive time intervals using `micros()` and prints them only after the timing test. I then analyzed the data with `read.py` and compared it with the course's hardware-timer Sketch B.

| Measurement | Sketch A: `delay(4)` | Sketch B: timer ISR |
|---|---:|---:|
| Sample intervals (n) | 1000 | 1000 |
| Mean interval (ms) | 5.1960 | 4.0001 |
| Standard deviation (ms) | 1.0023 | 0.0023 |
| Maximum deviation from 4 ms (ms) | 7.0000 | 0.0730 |

The delay-based version missed the intended average sampling interval. Its mean of 5.1960 ms is much larger than the 4.0000 ms target, and its interval standard deviation is also much higher than that of Sketch B. The measurements were self-timed by the MCU using `micros()`; no external logic analyzer was used.

One important limitation is that Sketch B records timer ISR timestamps, while the actual `analogRead(A0)` runs later in the foreground `work()` function. Therefore, the B statistics demonstrate regular ISR timing, not independently verified ADC sampling instants.

### What I changed

I would not rely on `delay(4)` to claim accurate 250 Hz sampling. Instead, I compared it with the provided hardware-timer version, which schedules an interrupt every 4 ms without waiting for foreground processing to finish. For a real ECG acquisition system, I would also move ADC conversion timing under a suitable timer-controlled acquisition mechanism and buffer samples for later processing. This ADC-trigger improvement is a proposed next step rather than a result of the timing experiment.

---

## Entry 2 — Register-Level Timer Rewrite (250 Hz)

### What I asked

> Rewrite this ESP32-S3 timer setup at register level, without the IDF driver calls. Keep it firing at 250 Hz.

I based the requested behavior on the course's timer configuration: an 80 MHz clock divided by 80, an alarm at 4,000 timer ticks, and periodic interrupts.

### What it answered

The AI outlined a direct-register approach in which the program would stop Timer Group 0 / Timer 0, set the prescaler and up-count mode, configure the alarm for 4,000 ticks, enable auto-reload, and connect an interrupt handler. An illustrative version of the response is:

```cpp
// High-level representation of the timer configuration steps.
// Register names below are placeholders, not ESP32-S3 header symbols.

void configure_timer_250hz() {
    TIMER0.ENABLE = 0;
    TIMER0.CLOCK_SOURCE = APB_80_MHZ;
    TIMER0.DIVIDER = 80;        // 1 tick = 1 us
    TIMER0.COUNT_UP = 1;
    TIMER0.COUNTER = 0;
    TIMER0.ALARM_VALUE = 4000; // 4 ms = 250 Hz
    TIMER0.AUTO_RELOAD = 1;

    // A working implementation also needs correct interrupt routing,
    // a registered ISR, interrupt acknowledgement, and any required
    // re-arming behavior for the ESP32-S3 timer peripheral.

    TIMER0.ALARM_ENABLE = 1;
    TIMER0.ENABLE = 1;
}
```

The answer helped identify the register-level operations hidden by the Arduino timer APIs. The symbolic names above describe the sequence rather than real ESP32-S3 register definitions.

### How I checked

First, I checked the timing arithmetic:

- 80 MHz / 80 = 1 MHz timer tick rate.
- 1 / 1 MHz = 1 microsecond per tick.
- 4,000 ticks = 4,000 microseconds = 4 ms.
- 1 / 0.004 s = 250 Hz.

Second, I cross-checked the prescaler, alarm period, auto-reload setting, and ISR requirements with the Week 03 Arduino API (slide 12) and ESP-IDF configuration (slide 13). This was a code-review and calculation check, not an executed register-level timer test.

For context, the **separate** Sketch D GPIO experiment measured the cost of Arduino GPIO abstraction compared with direct GPIO register writes:

```text
L2 digitalWrite : 63440 us for 100000 toggles -> 1576.29 kHz
L3 register     : 12612 us for 100000 toggles -> 7928.96 kHz
```

The direct-register GPIO version was about 5.03 times faster in this separate benchmark. This measurement concerns GPIO access rather than the register-level timer outline.

### What I changed

I kept the working timer-API approach for the measured Sketch B. Its recorded ISR intervals had a mean of 4.0001 ms, a standard deviation of 0.0023 ms, and a maximum deviation of 0.0730 ms. For a future register-level implementation, I would replace the placeholders with actual ESP32-S3 register definitions, handle the interrupt correctly, and compare another 1,000 intervals.

This exercise showed that lower-level code is not automatically a better design: it may provide more control but also increases hardware dependence and the amount of code that must be verified.

---

## Week 03 Reflection — Principle 02

**For AI-generated code, “it runs” and “it is correct” are different claims; measurement is the referee.**

The simple `delay(4)` ECG example can compile and produce serial data while failing to maintain an accurate sampling period. Using 1,000 measured intervals made that difference visible. A hardware timer gave more regular ISR timestamps, but even that does not prove that the ADC conversion happens at precisely the same instant. For register-level code, I also learned to separate correct timing calculations from a complete, verified hardware implementation.

**Evidence referenced:** `logs/sketcha.txt`, `logs/sketchb.txt`, `logs/sketchc.txt`, `RECORD_SHEET.md`, and the Sketch D serial-monitor output.  
**Course references:** W03 slides 12–14, 21–23, 31, 41–45, and 56–59.
