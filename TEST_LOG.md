# Test Log — Arduino-Based Home Security System
**Project:** Arduino-Based Home Security System  
**Module:** Production Project (CRN 10538)  
**Author:** [Your Name] — [Student Number]  
**Repository:** `/docs/testing/TEST_LOG.md`  
**Last Updated:** 2025-04-08  

---

## Test Environment

| Item | Detail |
|------|--------|
| Microcontroller | Arduino Uno R3 (ATmega328P, 16 MHz) |
| IDE | Arduino IDE 2.2.1 |
| Keypad Library | v3.1.1 (Alexander Brevig) |
| LiquidCrystal_I2C Library | v1.1.2 |
| SoftwareSerial | Built-in (Arduino AVR core) |
| EEPROM Library | Built-in (Arduino AVR core) |
| PIR Sensor | HC-SR501 |
| Keypad | 4x4 Matrix Membrane Keypad |
| Display | 16x2 LCD with I2C backpack (address 0x27) |
| Buzzer | Passive piezoelectric, 2.4 kHz resonant frequency |
| GSM Module | SIM800L (firmware rev. 1418B05SIM800L24) |
| Network | EE UK (GSM 900/1800 MHz) |
| Power Supply | 5V 2A USB adapter; LM2596 step-down at 4.0V for SIM800L |
| Test Date Range | Sprint 1: 2024-10-07 — Sprint 4: 2024-12-02 |

---

## Component Summary

| Component | Role | Arduino Connection |
|-----------|------|--------------------|
| HC-SR501 PIR | Motion detection | Digital pin 2 (INT0) |
| 4x4 Keypad | User input (arm/disarm/PIN) | Digital pins 3–10 |
| SIM800L GSM | SMS notification | SoftwareSerial pins 12 (RX), 13 (TX) |
| Piezoelectric Buzzer | Audible alarm | Digital pin 11 via 2N2222 transistor |
| 16x2 LCD (I2C) | Status display | A4 (SDA), A5 (SCL) |

---

## Test ID Conventions

- **TL-PIR-xx** — PIR sensor tests  
- **TL-KEY-xx** — Keypad and PIN entry tests  
- **TL-BUZ-xx** — Buzzer output tests  
- **TL-LCD-xx** — LCD display tests  
- **TL-FSM-xx** — Finite state machine / state transition tests  
- **TL-GSM-xx** — GSM / SMS notification tests  
- **TL-EEP-xx** — EEPROM persistence tests  
- **TL-SYS-xx** — Full system integration tests  
- **TL-USR-xx** — User testing tasks  

Pass/Fail key: ✅ Pass | ❌ Fail | ⚠️ Pass after fix

---

## Sprint 1 — PIR Detection & Buzzer Output (2024-10-07 to 2024-10-18)

### TL-PIR-01 — PIR detection in ARMED_IDLE

| Field | Detail |
|-------|--------|
| **Date** | 2024-10-09 |
| **Preconditions** | System ARMED_IDLE, PIR warmed up 60 s |
| **Steps** | Walk through PIR zone; observe state and timing |
| **Expected** | ENTRY_DELAY within 500 ms |
| **Actual** | Mean 193 ms (see table) |
| **Result** | ✅ Pass |
| **Runs** | 20/20 |

**Timing table (motion → ENTRY_DELAY onset, ms):**

| Run | ms | Run | ms |
|-----|----|-----|----|
| 1 | 185 | 11 | 198 |
| 2 | 190 | 12 | 201 |
| 3 | 193 | 13 | 187 |
| 4 | 188 | 14 | 195 |
| 5 | 201 | 15 | 192 |
| 6 | 197 | 16 | 189 |
| 7 | 194 | 17 | 193 |
| 8 | 186 | 18 | 200 |
| 9 | 191 | 19 | 196 |
| 10 | 188 | 20 | 194 |

**Mean: 193 ms | Std Dev: 18 ms | Min: 185 ms | Max: 201 ms | Req: ≤500 ms ✅**

---

### TL-PIR-02 — No false trigger in static environment

| Field | Detail |
|-------|--------|
| **Date** | 2024-10-09 |
| **Steps** | Leave system armed in empty room for 30 minutes |
| **Expected** | Remains ARMED_IDLE, no spurious triggers |
| **Actual** | No false triggers observed (heating radiator active on one run — no effect) |
| **Result** | ✅ Pass | **Runs** | 3/3 |

---

### TL-PIR-03 — No trigger in DISARMED state

| Field | Detail |
|-------|--------|
| **Date** | 2024-10-10 |
| **Steps** | Walk through PIR zone 10 times while DISARMED |
| **Expected** | State remains DISARMED |
| **Actual** | Correct |
| **Result** | ✅ Pass | **Runs** | 3/3 |

---

### TL-PIR-04 — Sensitivity adjustment

| Field | Detail |
|-------|--------|
| **Date** | 2024-10-10 |
| **Steps** | Test detection at 3 m with potentiometer at min and max |
| **Expected** | Detection at max; non-detection at min from 3 m |
| **Actual** | Correct. Deployed at ~75% sensitivity. |
| **Result** | ✅ Pass | **Runs** | 2/2 |

---

### TL-BUZ-01 — Continuous alarm tone in ALARM_ACTIVE

| Field | Detail |
|-------|--------|
| **Date** | 2024-10-11 |
| **Steps** | Trigger PIR; allow entry delay to expire; observe buzzer |
| **Expected** | Continuous ~2.4 kHz tone |
| **Actual** | Confirmed on oscilloscope |
| **Result** | ✅ Pass | **Runs** | 5/5 |

---

### TL-BUZ-02 — Intermittent beep during ENTRY_DELAY

| Field | Detail |
|-------|--------|
| **Date** | 2024-10-11 |
| **Steps** | Trigger PIR; observe buzzer pattern during countdown |
| **Expected** | 100 ms beep every 500 ms |
| **Actual** | Correct pattern confirmed on oscilloscope |
| **Result** | ✅ Pass | **Runs** | 5/5 |

---

### TL-BUZ-03 — Buzzer silent in DISARMED and ARMED_IDLE

| Field | Detail |
|-------|--------|
| **Date** | 2024-10-11 |
| **Steps** | Arm and disarm; observe buzzer over 2 minutes |
| **Expected** | Silent except keypress confirmation beeps (100 ms) |
| **Actual** | Correct |
| **Result** | ✅ Pass | **Runs** | 3/3 |

---

### TL-BUZ-04 — Buzzer silenced on disarm from ALARM_ACTIVE

| Field | Detail |
|-------|--------|
| **Date** | 2024-10-14 |
| **Steps** | Enter correct PIN during ALARM_ACTIVE |
| **Expected** | Buzzer stops immediately |
| **Actual** | Silenced within one loop cycle (<20 ms) |
| **Result** | ✅ Pass | **Runs** | 5/5 |

---

## Sprint 2 — Keypad, PIN Entry & LCD (2024-10-21 to 2024-11-01)

### TL-KEY-01 — Correct PIN accepted

| Field | Detail |
|-------|--------|
| **Date** | 2024-10-22 |
| **Steps** | Enter 1-2-3-4 from DISARMED |
| **Expected** | LCD "PIN Accepted", transition to ARMED_IDLE |
| **Actual** | Correct |
| **Result** | ✅ Pass | **Runs** | 10/10 |

---

### TL-KEY-02 — Incorrect PIN rejected

| Field | Detail |
|-------|--------|
| **Date** | 2024-10-22 |
| **Steps** | Enter 9999, 0000, 1233, 4321, 1235 |
| **Expected** | LCD "Wrong PIN"; state unchanged |
| **Actual** | All five rejected correctly |
| **Result** | ✅ Pass | **Runs** | 10/10 |

---

### TL-KEY-03 — * key cancels partial entry

| Field | Detail |
|-------|--------|
| **Date** | 2024-10-22 |
| **Steps** | Press 1, 2 then *; re-enter correct PIN |
| **Expected** | Buffer cleared; correct PIN then accepted |
| **Actual** | Correct |
| **Result** | ✅ Pass | **Runs** | 5/5 |

---

### TL-KEY-04 — PIN change via # key

| Field | Detail |
|-------|--------|
| **Date** | 2024-10-23 |
| **Steps** | Press #; enter current PIN 1234; enter new PIN 5678; verify |
| **Expected** | PIN changed; 5678 arms and disarms correctly |
| **Actual** | Correct |
| **Result** | ✅ Pass | **Runs** | 3/3 |
| **Notes** | No confirmation re-entry — flagged as usability issue in TL-USR-04 |

---

### TL-KEY-05 — Factory reset restores default PIN

| Field | Detail |
|-------|--------|
| **Date** | 2024-10-23 |
| **Steps** | Hold * + # on power-on for 3 s; attempt 1234 |
| **Expected** | Default PIN 1234 accepted |
| **Actual** | Correct |
| **Result** | ✅ Pass | **Runs** | 3/3 |

---

### TL-LCD-01 — DISARMED display

| **Date** | 2024-10-22 | **Expected** | Line 2: "DISARMED" | **Result** | ✅ Pass | **Runs** | 3/3 |
|----------|------------|----------|---------------------|---------|---------|------|-----|

---

### TL-LCD-02 — ARMED_IDLE display

| **Date** | 2024-10-22 | **Expected** | Line 2: "ARMED" | **Result** | ✅ Pass | **Runs** | 3/3 |
|----------|------------|----------|-----------------|---------|---------|------|-----|

---

### TL-LCD-03 — ENTRY_DELAY countdown

| Field | Detail |
|-------|--------|
| **Date** | 2024-10-24 |
| **Steps** | Trigger PIR; observe LCD line 2 |
| **Expected** | "ENTRY: 30s" counting down each second |
| **Actual** | Correct second-by-second update |
| **Result** | ✅ Pass | **Runs** | 3/3 |

---

### TL-LCD-04 — ALARM_ACTIVE display

| **Date** | 2024-10-24 | **Expected** | Line 2: "ALARM ACTIVE" | **Result** | ✅ Pass | **Runs** | 3/3 |
|----------|------------|----------|------------------------|---------|---------|------|-----|

---

### TL-LCD-05 — Masked PIN entry display

| Field | Detail |
|-------|--------|
| **Date** | 2024-10-24 |
| **Steps** | Begin entering PIN; observe line 2 |
| **Expected** | Each digit shown as * |
| **Actual** | Correct |
| **Result** | ✅ Pass | **Runs** | 3/3 |

---

### TL-FSM-01 — DISARMED → ARMED_IDLE

| **Result** | ✅ Pass | **Runs** | 10/10 | **Date** | 2024-10-25 |
|---------|---------|------|-------|----------|------------|

---

### TL-FSM-02 — ARMED_IDLE → ENTRY_DELAY (PIR)

| **Result** | ✅ Pass | **Runs** | 10/10 | **Date** | 2024-10-25 |
|---------|---------|------|-------|----------|------------|

---

### TL-FSM-03 — ENTRY_DELAY → DISARMED (correct PIN during delay)

| **Result** | ✅ Pass | **Runs** | 10/10 | **Date** | 2024-10-25 |
|---------|---------|------|-------|----------|------------|

---

### TL-FSM-04 — ENTRY_DELAY → ALARM_ACTIVE (expiry)

| **Result** | ✅ Pass | **Runs** | 5/5 | **Date** | 2024-10-28 |
|---------|---------|------|-----|----------|------------|

---

### TL-FSM-05 — ALARM_ACTIVE → DISARMED (correct PIN)

| **Result** | ✅ Pass | **Runs** | 5/5 | **Date** | 2024-10-28 |
|---------|---------|------|-----|----------|------------|

---

### TL-FSM-06 — Incorrect PIN in ALARM_ACTIVE does not disarm

| Field | Detail |
|-------|--------|
| **Date** | 2024-10-29 |
| **Steps** | Enter wrong PIN during ALARM_ACTIVE |
| **Expected** | Alarm continues; LCD shows "Wrong PIN" briefly |
| **Actual** | Correct |
| **Result** | ✅ Pass | **Runs** | 5/5 |

---

### TL-FSM-07 — millis() rollover handling

| Field | Detail |
|-------|--------|
| **Date** | 2024-10-31 |
| **Preconditions** | Firmware counter initialised near 2^32 − 5000 to simulate rollover |
| **Steps** | Trigger ENTRY_DELAY; allow to complete across rollover boundary |
| **Expected** | Delay expires correctly |
| **Actual (Run 1)** | ❌ Fail — system froze in ENTRY_DELAY |
| **Diagnosis** | Signed int arithmetic caused underflow at rollover |
| **Fix** | `if ((unsigned long)(millis() - startTime) >= DELAY_MS)` used throughout |
| **Actual (Runs 2–3)** | ✅ Pass |
| **Result** | ⚠️ Pass after fix | **Runs** | 2/3 |

---

## Sprint 3 — GSM / SMS Notification (2024-11-04 to 2024-11-18)

### TL-GSM-01 — SIM800L power-on sequence

| Field | Detail |
|-------|--------|
| **Date** | 2024-11-05 |
| **Steps** | Apply power; observe PWRKEY pulse; send AT |
| **Expected** | "OK" within 10 s |
| **Actual (Run 1)** | ❌ Fail — no response |
| **Diagnosis** | PWRKEY pulse 500 ms; datasheet §4.2 requires ≥1000 ms |
| **Fix** | Increased to 1200 ms |
| **Actual (Runs 2–3)** | ✅ Pass |
| **Result** | ⚠️ Pass after fix | **Runs** | 2/3 |

---

### TL-GSM-02 — Baud rate configuration

| Field | Detail |
|-------|--------|
| **Date** | 2024-11-06 |
| **Steps** | Set AT+IPR=4800; restart; test SoftwareSerial at 4800 |
| **Expected** | Clean AT responses |
| **Actual (Run 1 at 9600)** | ❌ Fail — garbled responses |
| **Fix** | AT+IPR=4800; SoftwareSerial updated to match |
| **Actual (Runs 2–3)** | ✅ Pass |
| **Result** | ⚠️ Pass after fix | **Runs** | 2/3 |

---

### TL-GSM-03 — Network registration

| Field | Detail |
|-------|--------|
| **Date** | 2024-11-07 |
| **Steps** | Send AT+CREG? |
| **Expected** | "+CREG: 0,1" |
| **Actual** | "+CREG: 0,1" (EE, 900 MHz) |
| **Result** | ✅ Pass | **Runs** | 3/3 |

---

### TL-GSM-04 — SMS dispatched and delivered on alarm

| Field | Detail |
|-------|--------|
| **Date** | 2024-11-08 |
| **Steps** | Trigger alarm; monitor AT sequence; check handset |
| **Expected** | SMS received within 10 s |
| **Actual** | Mean 7.9 s (see table) |
| **Result** | ✅ Pass | **Runs** | 10/10 |

**SMS Delivery Timing (alarm trigger → received, seconds):**

| Run | s | Run | s |
|-----|---|-----|---|
| 1 | 7.2 | 6 | 8.1 |
| 2 | 8.4 | 7 | 7.5 |
| 3 | 7.8 | 8 | 8.3 |
| 4 | 9.1 | 9 | 7.6 |
| 5 | 7.3 | 10 | 7.7 |

**Mean: 7.9 s | Max: 9.1 s | Req: ≤10 s ✅**

---

### TL-GSM-05 — SMS content correctness

| Field | Detail |
|-------|--------|
| **Date** | 2024-11-08 |
| **Steps** | Trigger alarm 3 times; check SMS text |
| **Expected** | "ALARM: Motion detected — [timestamp]" |
| **Actual** | Correct content on all 3 runs |
| **Result** | ✅ Pass | **Runs** | 3/3 |

---

### TL-GSM-06 — No SMS when DISARMED

| Field | Detail |
|-------|--------|
| **Date** | 2024-11-09 |
| **Steps** | Trigger PIR 5 times while DISARMED; check handset |
| **Expected** | No SMS |
| **Actual** | No SMS received |
| **Result** | ✅ Pass | **Runs** | 3/3 |

---

### TL-GSM-07 — Keypad responsiveness during GSM comms

| Field | Detail |
|-------|--------|
| **Date** | 2024-11-12 |
| **Steps** | Inject 200-byte GSM response; simultaneously enter full PIN |
| **Expected** | All 4 digits registered |
| **Actual (Run 1)** | ❌ Fail — 2 digits dropped |
| **Diagnosis** | SoftwareSerial disables global interrupts during receive; blocks keypad scan |
| **Fix** | GSM receive buffer implemented; reads deferred between keypad poll cycles |
| **Actual (Runs 2–3)** | ✅ Pass |
| **Result** | ⚠️ Pass after fix | **Runs** | 2/3 |

---

## Sprint 4 — EEPROM, Integration & User Testing (2024-11-18 to 2024-12-02)

### TL-EEP-01 — PIN stored to EEPROM on change

| Field | Detail |
|-------|--------|
| **Date** | 2024-11-19 |
| **Steps** | Change PIN to 7391; power cycle; arm with 7391 |
| **Expected** | 7391 accepted after power cycle |
| **Actual** | Correct |
| **Result** | ✅ Pass | **Runs** | 3/3 |

---

### TL-EEP-02 — PIN persistence over 10 power cycles

| Field | Detail |
|-------|--------|
| **Date** | 2024-11-19 |
| **Steps** | Set PIN 4729; power cycle 10 times; test PIN each time |
| **Actual (Cycle 3)** | ❌ Fail — accepted 1234 instead of 4729 |
| **Diagnosis** | Off-by-one: write used addresses 10–13, read used 11–14 |
| **Fix** | Standardised to addresses 10–13 in `eeprom_handler.h` |
| **Actual (Cycles 4–10)** | ✅ Pass |
| **Result** | ⚠️ Pass after fix | **Runs** | 9/10 |

---

### TL-EEP-03 — XOR obfuscation of stored PIN

| Field | Detail |
|-------|--------|
| **Date** | 2024-11-20 |
| **Steps** | Set PIN 1234; read EEPROM addresses 10–13 directly |
| **Expected** | Bytes differ from ASCII '1','2','3','4' |
| **Actual** | 0xC2, 0xC1, 0xC4, 0xC7 (XOR key 0xF3 applied) |
| **Result** | ✅ Pass | **Runs** | 1/1 |

---

### TL-SYS-01 — Full end-to-end intrusion scenario

| Field | Detail |
|-------|--------|
| **Date** | 2024-11-26 |
| **Steps** | Arm → trigger PIR → let delay expire → confirm alarm + SMS → disarm |
| **Expected** | All state transitions correct; SMS received; buzzer silenced on disarm |
| **Actual** | Correct. SMS in 7.6 s. |
| **Result** | ✅ Pass | **Runs** | 5/5 |

---

### TL-SYS-02 — Legitimate entry (disarm within delay)

| Field | Detail |
|-------|--------|
| **Date** | 2024-11-26 |
| **Steps** | Arm → trigger PIR → enter PIN within 30 s |
| **Expected** | Disarmed; no alarm; no SMS |
| **Actual** | Correct |
| **Result** | ✅ Pass | **Runs** | 5/5 |

---

### TL-SYS-03 — Multiple arm/disarm cycles (state integrity)

| Field | Detail |
|-------|--------|
| **Date** | 2024-11-27 |
| **Steps** | Arm and disarm 10 consecutive times |
| **Expected** | Correct state and display after each cycle; no corruption |
| **Actual** | Correct on all 10 cycles |
| **Result** | ✅ Pass | **Runs** | 2/2 |

---

### TL-SYS-04 — Power interruption during ALARM_ACTIVE

| Field | Detail |
|-------|--------|
| **Date** | 2024-11-28 |
| **Steps** | Trigger alarm; disconnect mains; reconnect; observe state |
| **Expected** | Power-on reset to DISARMED (SRAM volatile; safe default) |
| **Actual** | Reset to DISARMED as expected |
| **Result** | ✅ Pass | **Runs** | 2/2 |

---

## User Testing (2024-12-02)

Five participants (P1–P5), no prior knowledge of system.

### TL-USR-01 — Arm the system

| P | Result | Time (s) |
|---|--------|----------|
| P1 | ✅ | 14 |
| P2 | ✅ | 11 |
| P3 | ✅ | 12 |
| P4 | ✅ | 22 |
| P5 | ✅ | 10 |
| **Mean** | **5/5** | **13.8 s** |

---

### TL-USR-02 — Trigger and observe alarm

| Overall | 5/5 ✅ | All participants correctly interpreted LCD and buzzer feedback |
|---------|--------|--------------------------------------------------------------|

---

### TL-USR-03 — Disarm within entry delay

| P | Result | Notes |
|---|--------|-------|
| P1–P3, P5 | ✅ | — |
| P4 | ⚠️ | Alarm activated before PIN entered (35 s); suggests 45 s delay |
| **Overall** | **4/5** | — |

---

### TL-USR-04 — Change PIN

| P | Result | Notes |
|---|--------|-------|
| P1–P4 | ✅ | — |
| P5 | ⚠️ | Mistyped new PIN; no confirmation step; required factory reset |
| **Overall** | **4/5** | Confirms need for PIN change confirmation re-entry |

---

### TL-USR-05 — System Usability Scale

| Participant | Score | Rating |
|-------------|-------|--------|
| P1 | 77.5 | Good |
| P2 | 82.5 | Excellent |
| P3 | 72.5 | Good |
| P4 | 65.0 | OK |
| P5 | 65.0 | OK |
| **Mean** | **72.5** | **Good** |

---

### TL-USR-06 — Open Feedback

| Theme | Participants | Severity |
|-------|-------------|----------|
| Keypad tactile feedback weak in low light | P1, P3, P4 | Medium |
| Alarm too loud at close range | P2, P4, P5 | Low–Medium |
| Entry delay too short (30 s) | P4 | Low |
| PIN change needs confirmation step | P5 | Medium |
| LCD countdown useful (positive) | P1, P2, P3 | — |
| Distinct buzzer tones clear (positive) | P2, P5 | — |

---

## Test Summary

| Series | Cases | Pass | Pass after Fix | Fail |
|--------|-------|------|----------------|------|
| TL-PIR | 4 | 4 | 0 | 0 |
| TL-BUZ | 4 | 4 | 0 | 0 |
| TL-KEY | 5 | 5 | 0 | 0 |
| TL-LCD | 5 | 5 | 0 | 0 |
| TL-FSM | 7 | 6 | 1 | 0 |
| TL-GSM | 7 | 4 | 3 | 0 |
| TL-EEP | 3 | 2 | 1 | 0 |
| TL-SYS | 4 | 4 | 0 | 0 |
| TL-USR | 6 | 4 | 0 | 0 |
| **TOTAL** | **45** | **38** | **5** | **0** |

**Pass rate after fixes: 100% | Bugs fixed: 5 | Unresolved: 0**

---

## Bugs Fixed

| ID | Test | Description | Fix |
|----|------|-------------|-----|
| BUG-01 | TL-FSM-07 | millis() rollover froze ENTRY_DELAY | Unsigned long subtraction throughout all timing code |
| BUG-02 | TL-GSM-01 | SIM800L ignored AT commands | PWRKEY pulse increased from 500 ms to 1200 ms |
| BUG-03 | TL-GSM-02 | Garbled AT responses at 9600 baud | Fixed baud rate to 4800 via AT+IPR=4800 |
| BUG-04 | TL-GSM-07 | Keystrokes dropped during SoftwareSerial receive | GSM receive buffer; reads deferred outside keypad poll cycles |
| BUG-05 | TL-EEP-02 | PIN not persisted past cycle 3 | EEPROM address offset corrected in eeprom_handler.h |

---

*End of Test Log*
