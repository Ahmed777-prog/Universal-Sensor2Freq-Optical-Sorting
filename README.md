# 🚀 Universal Data Acquisition & Optical Sorting Platform
## A Hardware-Level, Frequency-Based Approach for Automated Material Classification

The **Universal Data Acquisition & Optical Sorting Platform** is an advanced, strictly **microcontroller-free (MCU-less)** industrial control topology engineered for high-reliability material classification. Completely bypassing the vulnerabilities of software stacks, interrupt service routines (ISRs), and traditional analog-to-digital converters (ADCs), this platform executes multi-variable data acquisition, synchronous free-space optical data transmission (Li-Fi), analog signal conditioning, and decoding entirely within the **hardware domain** using discrete CMOS/TTL digital logic and precision analog networks.

This architecture demonstrates that pure hardware configurations achieve absolute deterministic real-time parallel processing speeds, total immunity to software crashes or bit-flips, and robust operational resilience in harsh industrial environments.

---

## 👥 Core Engineering Team

* **Ahmed Akram** (Computers & Control Systems Engineering, Benha University)
* **Alsaeed Hasan** (Computers & Control Systems Engineering, Benha University)
* **Alaa Sayed** (Computers & Control Systems Engineering, Benha University)
* **Gannah Allah Hussein** (Computers & Control Systems Engineering, Benha University)

---

## 🛠️ Key Technical Features & Engineering Highlights

* **MCU-Less Control Topology:** Eliminates firmware dependencies, clock jitter from software loops, and propagation execution latency.
* **Resistance-to-Frequency Quantization:** Instantly translates variable physical parameters (such as Light via LDRs, Temperature via Thermistors, or Force via FSRs) into a highly precise, continuous frequency footprint.
* **Dual-Channel Free-Space Optics (Li-Fi Link):** Establishes a synchronous optical communication link utilizing dedicated 650nm lasers for parallel `DATA` and `CLOCK` streaming, completely eliminating the need for complex, latency-inducing Phase-Locked Loop (PLL) clock recovery circuits.
* **Transimpedance Amplifier (TIA) Front-End:** Incorporates a high-speed current-to-voltage conversion stage that neutralizes parasitic photodiode capacitance and eliminates detrimental RC time-constant propagation delays.
* **Active Anti-Desync Watchdog:** Implements an automated frame-completion detection and auto-reload hardware sub-system ensuring zero synchronization drift under intense ambient noise.
* **3-to-8 Line Matrix Classification:** Employs high-speed current-sinking digital decoders for decisive, bounce-free, and deterministic physical sorting boundaries.

---

## 📐 Detailed Hardware Architecture & Sub-Systems

### 1. Transmitter (TX) Architecture

The transmitter sub-system captures environmental analog sensor gradients and digitizes them by modulating a time-gated frequency burst into serialized optical pulses.

#### A. Precision Analog-to-Frequency Conversion (VCO Stage)
An **NE555P Timer** configured in an optimized **Astable Multivibrator Topology** performs real-time Resistance-to-Frequency conversion. The generated continuous square wave features a frequency directly proportional to the instantaneous physical state of the sensor:

$$f = \frac{1.44}{(R_1 + 2(RV_1 + R_{\text{sensor}})) \cdot C_2}$$

* **Dynamic Adaptability:** The highly interchangeable sensor front-end seamlessly accommodates LDRs (for optical/color sorting), Thermistors (for thermal threshold classification), or FSRs (for weight/pressure grading).
* **Hardware Calibration ($RV_1$):** A 3.1kΩ precision potentiometer establishes the baseline frequency calibration to counteract hardware component tolerances.

#### B. Deterministic Time-Gating & Sampling Window Generation
A secondary **NE555P Timer** operating in **Monostable Mode** generates a strict, highly stable sampling time window ($T_{\text{WIN}} \approx 10\mu\text{s}$) upon the arrival of a material trigger pulse captured via an input edge sensor.
* **Time-Frequency Intersection:** The continuous variable frequency (`M_Freq`) and the precision sampling window (`T_WIN`) are intersected via a **74HC08 AND Gate**, producing a discrete, finite packet of clock pulses designated as `BURST_CLK`.

#### C. High-Speed Synchronous Digital Quantization
The discrete `BURST_CLK` pulse packet is injected directly into a cascaded network of **Dual 74HC161 Synchronous 4-Bit Binary Counters**.
* **Zero-Ripple Design:** Synchronous state updating eliminates propagation "ghost values" and transient glitch states. At the falling edge of $T_{\text{WIN}}$, the counter outputs freeze into a rock-solid, static 8-bit binary word representing the digitized physical footprint of the material.

#### D. Asynchronous Gating Control & Zero-Overflow Frame Termination
To serialize and transmit the captured byte securely without data corruption, a **74HC74 D-Flip-Flop** operates as an active hardware gatekeeper:
* **The Engineering Problem:** Preventing asynchronous bit-shifting errors or "Dancing Output" anomalies during frame preparation.
* **The Solution:** A specialized **RC Spiker network** generates a nanosecond-scale edge-triggered load pulse (`PL_EN`) immediately prior to transmission. The D-Flip-Flop opens the gating window (`TX_ACTIVE`) allowing a stable transmission clock (`RAW_CLK`) to pass as a clean `SAFE_CLK`. A dedicated **74HC161 Pulse Counter** monitors `SAFE_CLK` to count exactly 8 pulses, then instantly issues a hardware deactivator pulse back into the Flip-Flop's asynchronous reset line, freezing the clock line immediately and terminating the frame with zero bit overflow.

#### E. High-Speed Parallel-to-Serial Conversion & Laser Driving Topologies
* The static 8-bit parallel footprint is latched into a **74HC165 Parallel-In/Serial-Out (PISO) Shift Register**. 
* Driven synchronously by `SAFE_CLK`, the data is shifted out serially. 
* High-speed **PN2222A BJT Transistors** drive two separate **650nm Laser Diodes** (`DATA_LASER` and `CLK_LASER`), translating the electrical bitstream into high-intensity, synchronized optical pulses across free space.

---

### 2. Receiver (RX) Architecture

The receiver subsystem performs high-sensitivity optical-to-electrical transduction, precision waveform reconstruction, and hardware-governed frame validation.

#### A. Analog Front-End Conditioning (TIA Stage)
When the transmitted laser streams impinge upon the receiver's clear IR Photodiodes, they operate in **Reverse-Bias Mode** to maximize responsiveness. The weak generated micro-currents are processed via a **Dual-Channel LM358 Operational Amplifier** configured as a high-gain **Transimpedance Amplifier (TIA)**:
* **Virtual Ground Shifting (Biased TIA):** To accommodate single-supply rails (0-5V) and prevent output clipping against the ground rail, a rigid voltage divider establishes an artificial reference offset at 2.5V on the non-inverting inputs.
* **RC Mitigation:** The TIA configuration absorbs the photodiode's internal junction capacitance, eliminating the slow RC charge curves that plague traditional resistor-divider sensor layouts.
* **Active Low-Pass Filtering:** A **100pF Symmetrical Feedback Capacitor** in parallel with a 100kΩ feedback resistor active-filters ambient optical glare, cross-talk, and high-frequency breadboard noise.

#### B. Precision Waveform Reconstruction (High-Speed Comparators)
The conditioned analog voltages from the TIA pass into dual **LM311 High-Speed Comparators** to extract crisp square waves.
* **Response Speed:** Response times within $\approx 200\text{ns}$ secure vertical logic edges.
* **Hysteresis Implementation:** A localized feedback resistor network inserts a firm noise margin, preventing output chattering or "double-clocking" when dealing with small input signal deltas. High-performance pull-up resistors convert the open-collector stages into sharp 5V logic levels: `RX_DATA` and `RX_CLK`.

#### C. Synchronous Byte Reassembly & Hardware Watchdog
* The reconstructed serial data stream enters a **74HC595 Serial-In/Parallel-Out (SIPO) Shift Register**, clocked directly by the recovered `RX_CLK`.
* **Watchdog Auto-Reload Sub-system:** A dedicated **74LS161 RX Frame Counter** monitors the incoming clock edge pulses. When a full 8-bit block is verified, it asserts a frame completion flag via an inverter network, latching the valid byte onto the parallel storage registers and performing an automatic reset. This keeps the receiver perfectly in-frame even under ambient light fluctuations.

#### D. Deterministic Decision & Actuation Matrix
Rather than executing complex software comparison loops, the platform offloads classification directly onto a **74HC138 3-to-8 Line Binary Decoder**:
* **Data Quantization:** The system taps into the 3 Most Significant Bits (**MSBs**: `RX_Bit5`, `RX_Bit6`, `RX_Bit7`) from the SIPO register. Tapping the MSBs breaks down the full 256-value density spectrum into **8 wide, highly stable sorting zones**, making the actuation immune to minor, low-level sensor frequency fluctuations.
* **Safety Interlock:** The decoder's hardware enable pins are hardwired to the Watchdog frame flag. This guarantees the classification matrix **ONLY** triggers when a verified, error-free 8-bit frame is latched, totally preventing false actuator strikes during transit.
* **Current-Sinking Matrix (Active-Low Actuation):** Since CMOS decoders exhibit far superior current-sinking capability compared to current-sourcing, the output indicators/relays are tied directly to VCC, and the 74HC138 sinks current to ground to actuate the distinct classifications:
  * **Zones 0–2 (Green LEDs):** Low Reflectance / High Absorption Materials.
  * **Zones 3–5 (Yellow LEDs):** Medium Reflectance / Standard Materials.
  * **Zones 6–7 (Red LEDs):** High Reflectance / Intense Glare Materials.

---

## 📁 Repository Structure

```text
├── Schematics/
│   └── Frequency-to-Digital_Converter_via_Li-Fi_Latest_Schematic.pdf  # Comprehensive system schematic diagrams
├── Simulation/
│   └── simulation.pdsprj                                              # Complete Proteus schematic capture & logic simulation file
├── Documentation/
│   └── Universal_Data_Acquisition_&_Optical_Sorting_Platform.pdf      # Official project defense presentation slides
├── Hardware Implementation/
│   └── 1.jpeg                                                         # Hardware Implementation Image
└── README.md                                                          # System documentation and engineering overview
```

---

## 🔬 Reliability & Noise Mitigation Implementations

* **Inrush & Ground Bounce Suppression:** Strategic deployment of 100nF (104) ceramic decoupling capacitors directly across the VCC and GND pins of all digital logic and op-amp ICs to handle rapid logic gate switching transients.
* **Breadboard Parasitic Protection:** Short-run, tightly grouped logic wiring paths to limit mutual inductance and stray capacitance from acting as ambient EM antennas.
* **Physical Alignment Geometry:** Rigidly integrated laser/photodiode physical alignment channels to ensure maximum optical flux transfer and block out stray room fluorescent lighting interference.

---

## 📜 Academic Disclaimer

This project was developed as an official undergraduate engineering presentation for the department of **Computers and Control Systems Engineering, Faculty of Engineering, Benha University, Egypt**. All rights reserved by the respective authors.
