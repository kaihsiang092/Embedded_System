# W03 Record Sheet

**Name** 楊凱翔  **Student ID** Q86144072 **Date** 2026-09-24

Every cell below is measured by you, in this room, today. There is no reference answer on
any slide in this deck, and none in this folder. If a cell is empty at 12:00, say so —
an empty cell is a result, a guessed cell is not.

**Units: milliseconds, four decimal places.** The raw files are microseconds; `read.py`
does the conversion. Do not mix the two.

---

## 1 · The three numbers (slide 31)

Fill each row as soon as you have it. A first, then B, then C.

| Version | Mean (ms) | Std dev (ms) | Max dev (ms) | n |
|---|---|---|---|---|
| A · `delay(4)` |5.196 | 1.0023| 7.0|1000 |
| B · timer ISR |4.0001 |0.0023 |0.0730 |1000 |
| C · `millis()` busy-wait |4.0023 |0.5280 |3.2530 |1000 |

What each one answers, so you know what you are looking at:

- **Mean** — am I sampling at the rate I claimed? Should land near 4.0000 ms for 250 Hz.
  A mean that is off is a **rate** error, not a jitter error.
- **Std dev** — how much does it wander, typically? This is the jitter figure, and it is
  the one that becomes noise floor in cycle 3.
- **Max dev** — what is the worst single interval? The hard real-time number. One bad
  interval can move an R-peak, and a good average will not save you.

## 2 · Side by side (slide 42)

| Version | Std dev (ms) | Max dev (ms) | Ratio to B | Usable for HRV? |
|---|---|---|---|---|
| A · `delay(4)` | 1.0023 | 7.0000 | 435.78 | No |
| B · timer ISR | 0.0023 | 0.0730 | 1.00 | Yes |
| C · `millis()` | 0.5280 | 3.2530 | 229.57 | No |

**Which version has the smallest standard deviation, and by what factor?**

> _______________________________________________________________

**A healthy adult's beat-to-beat variation is tens of milliseconds. Is your worst
version's max deviation small compared to that, or not?**

> _______________________________________________________________

## 3 · The price of being correct (slide 43)

Check these against your own build output. This is the one pair of numbers the deck gives
you. **Your RAM figures should match exactly** — they were measured on two different machines
and came out the same to the byte. **Your flash figures will be within a few tens of bytes**,
because your project's own file path is compiled into the binary.

| | Flash (bytes) | RAM (bytes) |
|---|---|---|
| Sketch A · `delay(4)` — deck | 273,421 | 22,808 |
| Sketch A — **yours** |273389 |22808 |
| Sketch B · timer ISR — deck | 279,385 | 22,848 |
| Sketch B — **yours** |279353 |22848 |

Difference in flash, yours: 5,964  bytes.

The deck's difference is about **5,950 bytes**. If yours is within a few tens of bytes of
that, you have reproduced it. If it differs by a lot — hundreds or thousands — something is
wrong and you should find out what before you write it down.

## 4 · Noise floors (slides 54–55)

| File | Floor (dB below tone) |
|---|---|
| `a-delay.txt` |-63.95 |
| `b-timer.txt` |-163.22 |
| **Difference** |99.27  |

`floor.py` normalises the 10 Hz tone to 0 dB and fixes both axes at `-140 … +5 dB`, so the
two plots are drawn on the same ruler and can be laid side by side as they come out. Do not
change the axis settings — if one plot auto-scales and the other does not, the cleaner
version can end up *looking* noisier, which is the opposite of what you measured.

## 5 · The W02 debt (slide 58)

Paste the two lines Sketch D printed, verbatim:

```
L2 digitalWrite :63440 us for 100000 toggles -> 1576.29 kHz
L3 register     :12612 us for 100000 toggles -> 7928.96 kHz
```

L3-to-L2 ratio: 5.03

## 6 · Method note (one sentence, required)

> Self-timed with `micros()`, no external instrument.

Anything else about how you measured that a reader would need in order to repeat it:
> Each sketch recorded 1000 sampling intervals using `micros()`, stored the intervals without printing during measurement, and printed them only after the run for analysis with `read.py`.

> _______________________________________________________________

## 7 · Judgement (one sentence, required)

Name the version you would **not** ship for HRV, and why — **with your own numbers in
the sentence.** "A is worse" is not an answer. "A's max deviation of 4.9 ms is a
significant fraction of a 30 ms beat-to-beat variation, so it would corrupt the HRV
estimate" is.
> **I would not ship Sketch A (`delay(4)`) for HRV because its standard deviation was 1.0023 ms and its maximum deviation reached 7.0000 ms, which is a significant fraction of a beat-to-beat variation of tens of milliseconds and could corrupt the HRV estimate.**
> _______________________________________________________________
>
> _______________________________________________________________

---

## Before you leave

- [ ] `a-delay.txt`, `b-timer.txt`, `c-millis.txt` in `w03/` — 1000 raw intervals each
- [ ] Both spectrum PNGs in `w03/`
- [ ] `read.py` and `floor.py` committed, so your numbers can be reproduced
- [ ] Two entries in `AI_USAGE.md` — the 250 Hz prompt and the register-level rewrite
- [ ] This sheet, filled in
- [ ] Google Form submitted **before you leave the room**

Files in the repo the same day. You cannot re-create any of this once the board is reflashed.
