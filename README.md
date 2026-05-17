# Arduino-soil-weather-station

This repository contains an Arduino-based environmental monitoring sketch that integrates an ambient light sensor (LDR), a DHT11 temperature and humidity sensor, and an analog soil moisture probe into a single microcontroller setup. 

The project reads raw environmental data, performs localized mathematical conversions for physical units (Lux and Moisture %), and outputs the telemetry directly to the hardware Serial Monitor.

---

## 🛠️ Hardware Requirements

* **Microcontroller:** Arduino Uno, Nano, or compatible board
* [cite_start]**Light Sensor:** Standard Light Dependent Resistor (LDR) + 5.03 kΩ precision resistor [cite: 55]
* [cite_start]**Climate Sensor:** DHT11 Temperature & Humidity Module [cite: 63]
* [cite_start]**Soil Probe:** Analog Resistive/Capacitive Soil Moisture Sensor [cite: 57]
* **Connecting Hardware:** Breadboard and jumper wires

---

## 📌 Hardware Configuration & Pinout

| Component | Component Pin | Arduino Pin | Description |
| :--- | :--- | :--- | :--- |
| **LDR Circuit** | Analog Out | [cite_start]**A0** (`0`) [cite: 55] | Reads ambient light voltage |
| **DHT11 Sensor** | Data Pin | [cite_start]**D11** (`gate`) [cite: 55] | [cite_start]Manually timed 1-wire data stream [cite: 63] |
| **Soil Moisture** | Analog Out | [cite_start]**A0** (`sensor_pin`) [cite: 57] | Reads volumetric soil water content |

> [cite_start]⚠️ **Hardware Bug Alert:** As currently written in the code, both the LDR [cite: 55] [cite_start]and the Soil Moisture sensor [cite: 57] are assigned to **Analog Pin A0**. To deploy this physically, you must reassign one of these sensors to another analog pin (e.g., `A2`) in the variable declarations.

---

## 🔬 Sensor Mechanics & Math

### 1. Ambient Light Calculation (Lux)
[cite_start]Instead of relying on arbitrary raw values, the sketch converts the LDR analog voltage into standard lux using a log-log regression power law[cite: 59, 60]:

* [cite_start]**Reference Balance Resistor:** 5030 Ω ($5.03\text{ k}\Omega$) [cite: 55]
* [cite_start]**Lux Conversion Scalar:** 12,518,931 [cite: 55]
* [cite_start]**Lux Conversion Exponent:** -1.405 [cite: 55]

[cite_start]The code calculates the sensor's current operational resistance ($R_{\text{ldr}}$) and applies the formula[cite: 59, 60]:
$$\text{Lux} = 12518931 \times (R_{\text{ldr}})^{-1.405}$$

### 2. Manual DHT11 Bit-Banging
[cite_start]The code bypasses external libraries to decode the DHT11 data stream manually using microsecond pulse timing via `pulseIn()`[cite: 63, 64]. 
* [cite_start]It triggers the sensor with a 20ms `LOW` pulse[cite: 63].
* [cite_start]It reads a 40-bit data packet[cite: 56, 65].
* [cite_start]Binary `0` is determined by high pulse durations between 20–26µs[cite: 65].
* [cite_start]Binary `1` is determined by high pulse durations between 65–74µs[cite: 65].

### 3. Soil Moisture Mapping
[cite_start]The soil moisture framework uses a hardcoded calibration scale to map the analog input into a human-readable percentage[cite: 67]:
* [cite_start]**Dry Baseline (0%):** An ADC reading of `550` [cite: 67]
* [cite_start]**Saturated Baseline (100%):** An ADC reading of `0` [cite: 67]

---

## 🚀 How to Set Up and Run

1. **Clone the Repository:** Create a folder named `allsensorcode` on your computer and place the `allsensorcode.ino` file inside it.
2. **Wire the Circuit:** Build your hardware layout according to the pinout table (ensuring you resolve the `A0` conflict mentioned above).
3. **Compile:** Open `allsensorcode.ino` in the Arduino IDE. 
4. **Fix Compilation Errors:**
   * [cite_start]Add a missing semicolon `;` at the end of line 12 (`int sensor_pin = A0`)[cite: 57].
   * [cite_start]Move the misplaced closing bracket `}` on line 42 to the very end of the file so the DHT11 [cite: 63] [cite_start]and Soil Moisture [cite: 67] code segments are included inside the main `loop()` scope.
   * [cite_start]Remove the infinite `while(1)` trap loop inside the DHT segment [cite: 63] to allow all three sensors to continuously update.
5. [cite_start]**Upload & Monitor:** Upload the code to your Arduino and open your Serial Monitor set to a baud rate of **9600**[cite: 57].
