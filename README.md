# 350W MPPT 4-Switch Buck-Boost Converter PCB

[![Hardware-Status](https://shields.io)](#)
[![PCB-CAD](https://shields.io)](#)
[![License](https://shields.io)](#)

A high-efficiency, digitally controlled **350W 4-Switch Synchronous Buck-Boost DC-DC Converter** designed specifically for Maximum Power Point Tracking (MPPT) solar charging applications. This project transitions a high-frequency switching design from prototype evaluation into a highly optimized, custom multi-layer PCB. 

The architecture features **dual-ended voltage/current tracking**, an ultra-fast **hardware-isolated analog fault protection sub-system**, and optimized low-inductance parallel switching loops running at **150 kHz**.

---

## 📊 Core System Specifications
* **Input Voltage Operating Range (Vin):** 24V to 48V DC
* **Output Voltage Operating Range (Vout):** 12V to 48V DC
* **Maximum Output Continuous Power (Pmax):** 350 Watts
* **Switching Frequency (fsw):** 150 kHz
* **Max Input Continuous Current:** ~14.6 Amps (at Vin, min = 24V)
* **Max Output Continuous Current:** ~29.1 Amps (at Vout, min = 12V)

---

## 🛠️ Component Selection Matrix
To optimize the high-frequency parameters of the board, through-hole components are utilized for major power components to simplify thermal management, while the signal processing and driver support structures are executed strictly in low-parasitic SMD footprints.

| Component Function | Selected Part | Package Type | Supplier Reference | Key Design Factor |
| :--- | :--- | :--- | :--- | :--- |
| **Power MOSFETs** | Slkor SL102N10 | TO-220-3L | Robu.in | 100V Vds, 102A Id, ultra-low 6.4 mΩ Rds(on), tiny 32.5 nC Qg. Used in parallel pairs. |
| **Gate Driver IC** | Infineon IR2110 | DIP-14 | Robu.in | High/Low side independent floating driver; 2.0A source / 2.5A sink capability to snap open parallel gates. |
| **Input Current Sensor** | Allegro ACS712-20A | SOIC-8 | Robu.in | Hall-Effect sensor; 1.2 mΩ path eliminates thermal loss; provides 2.1 kV galvanic safety isolation from input spikes. |
| **Output Current Sensor** | Analog Devices MAX4080SASA+T | SOIC-8 | Robu.in | High-side shunt amplifier; 76V common-mode input tolerance easily handles 48V output swing. |
| **Window Comparator** | TI LM393DR | SOIC-8 | Robu.in | Open-drain dual outputs; wired-AND hardware node configuration for instantaneous dual fault tracking. |
| **Hardware Safety Latch** | Nexperia 74HC74D | SOIC-14 | Robu.in | Dual D-Type Flip-Flop wired as an asynchronous SR latch with a lightning-fast 14 ns propagation lockout delay. |
| **Bootstrap Diode** | Slkor M7 | SMA (DO-214AC) | Robu.in | Ultra-fast recovery surface mount variant rated for 1000V/1A to seal off the floating high-side gate charge. |
| **High-Speed Bootstrap Cap** | FH 0.1µF (100nF) 50V | 0805 SMD | Robu.in | Multi-layer ceramic capacitor (MLCC); low-ESR variant for high-frequency instantaneous gate current bursts. |
| **Bulk Storage Bootstrap Cap**| Lelon 10µF 50V | SMD Radial | Robu.in | Aluminum electrolytic reservoir capacitor maintaining solid voltage hold throughout prolonged ON-duty cycles. |

---

## 📐 Circuit Sub-System Architecture

### 1. Dual Parallel MOSFET Driving Config
To handle 350W efficiently, two SL102N10 MOSFETs run in parallel to drop path resistance to a mere 3.2 mΩ (6.4 mΩ / 2). To eliminate high-frequency parasitic cross-talk oscillation between the tied gates, independent 10 Ω 0805 SMD resistors isolate each gate pin:

### 2. High-Side Floating Bootstrap Network
The IR2110 floating channel relies on a precise, low-impedance SMD component cluster to charge and retain gate potential when the floating node shifts dynamically up to the 48V input potential.
* **Diode Voltage Drop Offset:** The M7 diode acts as a fast-acting one-way check-valve. The +12V gate auxiliary supply undergoes a 1.1V Vf reduction. The bootstrap reservoir capacitors lock in at 12V - 1.1V = 10.9V. This drives the Slkor gates efficiently above their 10V full conduction plateau.

### 3. Comprehensive 4-Point Data Sensing
Implementing an MPPT tracking algorithm requires calculating true live input power (P = V × I) alongside tracking dynamic battery charging performance.

#### Output Shunt Resistor Dimensioning:
The MAX4080SASA possesses a fixed gain factor of 60 V/V, saturating its differential measurement window at 100mV. At maximum power drop configuration (350W / 12V = 29.1A max output continuous load):
Rsense = Vsense_max / Imax = 100 mV / 29.1 A = 3.43 mΩ
* **Design Implementation:** A standard 3 mΩ 2512 SMD Shunt Resistor (Rated for ≥ 3W) is populated on the output line. At peak load, it drops a clean 87.3 mV, presenting low insertion loss and translating to an amplified 5.23V maximum scale output directly mapping to safety handling circuitry.

#### Wide-Swing Resistor Voltage Dividers:
To safely profile swinging rails (Vin: 24V -> 48V and Vout: 12V -> 48V), dividers scale maximum 48V lines down safely. They are over-engineered for a 60V absolute maximum surge ceiling to safeguard microcontroller ADC interfaces:
* **For 3.3V Logic Microcontrollers (ESP32 / STM32 / RP2040):** 
  * Rtop = 180 kΩ, Rbottom = 10 kΩ
  * *Scale Performance:* 12V Rail -> 0.63V | 48V Rail -> 2.52V | 60V Spike -> 3.15V (Safely below 3.3V).
* **For 5V Logic Microcontrollers (Arduino Atmega328P / Mega):**
  * Rtop = 120 kΩ, Rbottom = 10 kΩ
  * *Scale Performance:* 12V Rail -> 0.92V | 48V Rail -> 3.69V | 60V Spike -> 4.61V (Safely below 5.0V).

---

### 4. Nanosecond Hardware Protection & Latch Loop
Software failsafe deployment via microcontroller polling cycles loops is far too sluggish to mitigate short-circuits. This system utilizes a dedicated high-speed analog window comparator interlocked with a hardware latch to ground the driver pins instantly.


[VOLTAGE MONITOR STAGE]Output Voltage Divider ──► Pin 5 (IN2+) ───┐Overvoltage Trip Ref   ──► Pin 6 (IN2-) ───┼─► Pin 7 (OUT2) ──┬──┐│  │[CURRENT MONITOR STAGE]                                        │  │MAX4080 Output (Amps)  ──► Pin 3 (IN1+) ───┐                  │  │Overcurrent Trip Ref   ──► Pin 2 (IN1-) ───┼─► Pin 1 (OUT1) ──┴──┤▼Combined Fault Line (Active Low)│├──► [ Pull-Up Resistor: 10 kΩ to 5V ]│▼74HC74D Latch Pin 1 (/PRE1)


1. **Wired-AND Fault Triggering:** The open-drain outputs (Pins 1 and 7) of the LM393DR are tied together on a single track pulled high via a 10 kΩ resistor to +5V. If output current breaches the threshold set by the trimmer potentiometer or output voltage breaks past its target ceiling, the trace voltage instantly collapses to 0V Ground.
2. **Instant Asynchronous Latching:** This low node drops into Pin 1 (/PRE1) of the 74HC74D Latch. It instantly sets the flip-flop asynchronously, bypassing any microprocessor clock dependency.
3. **Driver Lockout:** Latch output Pin 5 (Q1) shoots directly to HIGH (5V), feeding into the SD (Shutdown) Pin 11 of the IR2110 Gate Driver. The IR2110 overrides all microcontroller PWM activity within nanoseconds, forcing its high-side and low-side driver channels hard to 0V and isolating the parallel MOSFET gates from power lines.
4. **Microcontroller Overlook Loop:** The driver remains firmly disabled until the microcontroller scans the environmental conditions and executes an explicit recovery task by pulsing Pin 1 (/CLR1) LOW to safely clear the latch lockout state.

---

## 📐 PCB Layout & Manufacturing Directives

To maintain complete signal stability under full 350W loads at 150 kHz, adhere to these explicit PCB routing constraints:

* **Pure Kelvin Sensing Connections:** When routing tracks from the 2512 SMD current shunt back to Pins 3 and 4 of the MAX4080 IC, do not branch off heavy power planes. Tap the copper traces directly from the interior pads of the shunt resistor landing pattern. Run these two tracks side-by-side as a tightly grouped differential pair to cancel out ambient EMI noise from the switching loops.
* **High-Current Path Optimization:** At maximum output configuration, traces will carry up to 29.1A. Use wide copper polygon pours rather than standard tracks for the power rails. For standard 1oz copper weights, ensure power tracks are properly dimensioned using high-current clearance calculators (or utilize 2oz heavy copper weight manufacturing processes) to keep trace temperature rises below 10°C.
* **Under-Chip Component Placement Trick:** To minimize parasitic loop area and inductive ringing, mount the SMD bootstrap cluster (M7 SMA Diode and 0805 100nF Ceramic capacitor) on the bottom layer of the PCB layout, positioned directly beneath the through-hole pins of the DIP-14 IR2110 socket.
* **Split Ground Plane Separation:** Implement a dedicated split-plane grounding method. Keep noisy, high-power return paths (COM) isolated from quiet analog sensor grounding returns (VSS / Analog Ground). Bridge them together at exactly one single star ground node positioned directly beneath the ground connections of the IR2110 gate driver IC.
