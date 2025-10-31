# Bandgap Reference Design

This project documents my work on the Bandgap Reference circuit using the Sky130 PDK.

## Objective
To design a stable voltage reference independent of temperature and process variations.
# *INTRODUCTION*
A **Bandgap Reference (BGR)** circuit is an essential building block in analog and mixed-signal integrated circuits.  
Its main function is to generate a **stable reference voltage** that remains **independent of temperature, supply voltage, and process variations**.  
This stable voltage is used in many critical applications such as **ADCs, DACs, voltage regulators, and biasing circuits**.

The concept of the Bandgap Reference is based on combining two temperature-dependent voltages:

1. **CTAT (Complementary to Absolute Temperature) Voltage:**  
   - Typically derived from the **base-emitter voltage (V<sub>BE</sub>)** of a bipolar transistor.  
   - V<sub>BE</sub> decreases with increasing temperature (negative temperature coefficient).

2. **PTAT (Proportional to Absolute Temperature) Voltage:**  
   - Generated from the **difference in base-emitter voltages (ΔV<sub>BE</sub>)** between two transistors operating at different current densities.  
   - ΔV<sub>BE</sub> increases with temperature (positive temperature coefficient).

By carefully scaling and summing these two voltages, the opposing temperature effects cancel out, resulting in a **constant output voltage**—typically around **1.2 V**, which corresponds to the bandgap voltage of silicon at 0 K.

---

### ⚙️ Key Features
- Provides a **temperature-stable voltage reference** (~1.2 V)
- **Insensitive** to power supply and process variations
- Can be implemented using **bipolar or CMOS processes**
- Widely used in **precision analog and mixed-signal ICs**



### 🧠 Working Principle (Simplified)
At its core, the Bandgap Reference circuit works as follows:
1. Generate a CTAT voltage from a diode-connected BJT (V<sub>BE</sub>).
2. Generate a PTAT voltage using two BJTs with different emitter area ratios.
3. Add the PTAT voltage (positive TC) to the CTAT voltage (negative TC) in proper proportions.
4. The resulting sum is a **temperature-independent reference voltage**.


   
<img width="698" height="268" alt="Screenshot 2025-10-31 095749" src="https://github.com/user-attachments/assets/29a07377-a79a-483f-ac21-dac1115c8883" />

## ⚙️ Features of Bandgap Reference (BGR)

- Temperature-independent voltage reference circuit widely used in Integrated Circuits (ICs).  
- Produces a **constant output voltage** regardless of power supply variation, temperature changes, and circuit loading.  
- Typical **output voltage ≈ 1.2 V**, which is close to the **bandgap energy of silicon at 0 K**.  
- Used in almost all types of circuits — **analog, digital, mixed-signal, RF, and System-on-Chip (SoC)** designs.

---

## 🧩 Applications of Bandgap Reference (BGR)

- **Low Dropout Regulators (LDOs)**  
- **DC-to-DC Buck Converters**  
- **Analog-to-Digital Converters (ADCs)**  
- **Digital-to-Analog Converters (DACs)**

## 📚 Contents

1. [Tool and PDK Setup](#1-tool-and-pdk-setup)  
   1.1 [Tools Setup](#11-tools-setup)  
   1.2 [PDK Setup](#12-pdk-setup)

2. [Bandgap Reference (BGR) Introduction](#2-bandgap-reference-bgr-introduction)  
   2.1 [BGR Principle](#21-bgr-principle)  
   2.2 [Types of BGR](#22-types-of-bgr)  
   2.3 [Self-Biased Current Mirror Based BGR](#23-self-biased-current-mirror-based-bgr)

3. [Design and Pre-Layout Simulation](#3-design-and-pre-layout-simulation)

4. [Layout Design](#4-layout-design)

5. [LVS and Post-Layout Simulation](#5-lvs-and-post-layout-simulation)

## 1. Tool and PDK Setup
### 1.1 Tools Setup

For the design, simulation, and verification of the Bandgap Reference (BGR) circuit, the following open-source EDA tools are used:

---

#### 🧪 Ngspice — Circuit Simulation
**Ngspice** is an open-source SPICE-based simulator used for performing **analog circuit simulations**.  
<img width="289" height="123" alt="Screenshot 2025-10-31 101341" src="https://github.com/user-attachments/assets/f4615124-2ed7-4b6a-b25b-c889e7c5b86f" />
It takes a **SPICE netlist** as input, which describes the circuit components and their connections, and then computes electrical parameters such as node voltages, currents, and transfer characteristics.  
In this project, Ngspice is used to:
- Simulate the **schematic-level design** of the BGR circuit.  
- Analyze **DC**, **AC**, and **transient** behavior.  
- Verify **temperature dependence** and output voltage stability.

-Steps to install Ngspice - Open the terminal and type the following to install Ngspice
```bash
$  sudo apt-get install ngspice
```

#### 🧩 Magic — Layout Design and DRC
**Magic** is a VLSI layout editor developed by Berkeley, primarily used for **IC layout design** in open-source PDKs such as **Sky130**.  
It provides interactive tools for drawing transistors, interconnects, and layers according to process design rules.  
Magic is used here to:
- Create the **physical layout** of the BGR circuit.  
- Perform **Design Rule Check (DRC)** to ensure the layout complies with the fabrication constraints of the Sky130 process.  
- Extract the layout to generate a **SPICE netlist** for post-layout simulations.
<img width="274" height="111" alt="Screenshot 2025-10-31 101451" src="https://github.com/user-attachments/assets/a78093f7-4b13-406b-b854-ec6c8117bc11" />

-Steps to install Magic - Open the terminal and type the following to install Magic
```bash
$  wget http://opencircuitdesign.com/magic/archive/magic-8.3.32.tgz
$  tar xvfz magic-8.3.32.tgz
$  cd magic-8.3.28
$  ./configure
$  sudo make
$  sudo make install
```

---

#### 🔗 Netgen — LVS (Layout vs. Schematic)
**Netgen** is a layout verification tool used for **Layout Versus Schematic (LVS)** comparison.  
It compares the netlist extracted from the layout (using Magic) with the schematic netlist (used in Ngspice simulation) to verify connectivity and device matching.  
A successful LVS ensures that the **layout accurately represents the schematic**, confirming the design’s electrical integrity before fabrication.

---<img width="284" height="103" alt="Screenshot 2025-10-31 101535" src="https://github.com/user-attachments/assets/8f03273d-ce0b-4531-81f3-4f387b05238d" />

-Steps to install Netgen - Open the terminal and type the following to insatll Netgen.
```bash
$  git clone git://opencircuitdesign.com/netgen
$  cd netgen
$  ./configure
$  sudo make
$  sudo make install 
```

In summary, these tools together provide a **complete open-source analog design flow** — from schematic simulation (Ngspice) → layout creation (Magic) → verification (Netgen).

### 1.2 PDK Setup 

The SkyWater **sky130** PDK provides process design data (layers, device rules, models) required for layout, extraction and simulation.  
Below are typical steps to obtain and prepare the SkyWater-130 PDK on a Linux development environment.
**Steps:**
- Create a directory for the PDK.  
- Clone the SkyWater PDK repository.  
- Initialize submodules (if required).  
- Build or install PDK libraries (optional).  
- Set the PDK path so tools like Magic, Ngspice, and Netgen can locate it easily. 

<img width="975" height="356" alt="Screenshot 2025-10-26 225127" src="https://github.com/user-attachments/assets/dd53a7e6-03a4-4d06-88a8-c8bf3c010cd1" />

<img width="970" height="651" alt="Screenshot 2025-10-26 230151" src="https://github.com/user-attachments/assets/2b2cb94a-b2ef-4135-8f42-42a58acddaf4" />

## 2. BGR Introduction

### 2.1 BGR Principle

The **Bandgap Reference (BGR)** circuit generates a temperature-independent reference voltage by combining two voltage components with **opposite temperature coefficients**.

The basic operation principle of a BGR circuit is to **sum a voltage with a negative temperature coefficient (CTAT)** and another with a **positive temperature coefficient (PTAT)** such that their variations cancel each other.

<img width="937" height="571" alt="Screenshot 2025-10-31 114203" src="https://github.com/user-attachments/assets/fd8e82f0-ac14-4d01-8c6c-c3544f9b369a" />

---

#### 🧩 Key Concepts

- **CTAT (Complementary to Absolute Temperature):**  
  A voltage that **decreases** as temperature **increases**.  
  Typically, the **base-emitter voltage (V<sub>BE</sub>)** of a bipolar junction transistor (BJT) or a diode exhibits CTAT behavior.
  

- **PTAT (Proportional to Absolute Temperature):**  
  A voltage that **increases** as temperature **increases**.  
  This can be generated using the **difference between two V<sub>BE</sub>** voltages of transistors operating at different current densities.

---

#### ⚙️ Principle of Operation

The BGR circuit operates by **adding** the CTAT and PTAT voltages in proper proportion so that the resulting voltage remains constant over temperature.
####  2.1.1 CTAT VOLTAGE GENERATION
Semiconductor diodes typically exhibit CTAT (Complementary to Absolute Temperature) behavior. When a constant current flows through a forward-biased diode, an increase in temperature causes the voltage across the diode to decrease. Experimentally, the rate of decrease of the diode’s forward voltage with temperature is approximately –2 mV/°C.
<img width="825" height="295" alt="Screenshot 2025-10-31 114812" src="https://github.com/user-attachments/assets/37ece2d7-02f6-4462-9eb4-e0d2c31d0e94" />

####  2.1.2 PTAT VOLTAGE GENERATION
<img width="328" height="627" alt="Screenshot 2025-10-31 115134" src="https://github.com/user-attachments/assets/cde662c9-566f-4728-8542-b3e8e4a2aec4" />

From the diode current equation, it can be observed that the diode voltage consists of two main temperature-dependent components:

Thermal voltage (Vₜ) — This term is directly proportional to temperature (approximately of order ~1).

Reverse saturation current (Iₛ) — This term increases with temperature approximately with an order of ~2.5.

Since Iₛ appears in the denominator of the logarithmic term (ln(I₀/Iₛ)), an increase in temperature causes this term to decrease, resulting in the CTAT behavior of the diode voltage.

Therefore, to design a PTAT (Proportional to Absolute Temperature) voltage generation circuit, we need a method to isolate the Vₜ component from the Iₛ dependence.

The following approach describes how this separation can be achieved.

<img width="452" height="412" alt="Screenshot 2025-10-31 115401" src="https://github.com/user-attachments/assets/d22c07fa-9154-4592-a5df-ccbc76e92de8" />

In the above circuit same amount of current I is flowing in both the branches. So the node voltage A and B are going to be same V. Now in the B branch if we substract V1 from V, we get Vt independent of Is.

<img width="640" height="383" alt="Screenshot 2025-10-31 115505" src="https://github.com/user-attachments/assets/8257068f-df17-4cad-b717-eaf2363d28c0" />

V= Combined Voltage across R1 and Q2 (CTAT in nature but less sloppy)
V1= Voltage across Q2 (CTAT in nature but more sloppy)
V-V1= Voltage across R1 (PTAT in nature)

From the above analysis, it is evident that the voltage difference (V – V₁) exhibits a PTAT (Proportional to Absolute Temperature) behavior. However, its slope is relatively small compared to the CTAT (Complementary to Absolute Temperature) characteristic of a diode.

To enhance the PTAT slope, multiple BJTs configured as diodes can be used in parallel. This reduces the current flowing through each individual diode, which in turn increases the slope of the (V – V₁) characteristic, thereby improving the PTAT response.
<img width="1002" height="507" alt="Screenshot 2025-10-31 115754" src="https://github.com/user-attachments/assets/d3e2de54-c961-4d9c-a93e-421788587f0b" />



#### 🧠 Summary

- **Diode / BJT junction** provides the **CTAT** component.  
- **Difference in V<sub>BE</sub>** between transistors provides the **PTAT** component.  
- When both are combined properly, the **temperature variations cancel**, producing a **constant reference voltage (~1.2 V)** close to the **bandgap voltage of silicon**.

---

📘 *In simple terms, the BGR circuit uses one voltage that decreases with temperature and another that increases with temperature — when added in the right ratio, the overall result stays constant.*

### 2.2 Types of Bandgap Reference (BGR)

The **Bandgap Reference (BGR)** circuit can be classified in different ways depending on its **circuit architecture** and **application requirements**.

---

#### 🧩 Architecture-wise Classification

Based on the circuit implementation approach, BGR circuits are commonly designed using:

1. **Self-Biased Current Mirror Architecture**  
   - Uses transistor-level biasing without external amplifiers.  
   - Offers simplicity and good stability.  
   - Suitable for integration in analog and mixed-signal ICs.

2. **Operational Amplifier-Based Architecture**  
   - Uses an op-amp to control node voltages precisely.  
   - Provides better accuracy and matching.  
   - Often used in precision reference applications.

---

#### ⚙️ Application-wise Classification

Depending on design goals and target specifications, BGR circuits can be categorized as:

1. **Low-Voltage BGR** — Optimized to operate at reduced supply voltages.  
2. **Low-Power BGR** — Designed for minimal power consumption, suitable for battery-powered systems.  
3. **High-PSRR and Low-Noise BGR** — Provides improved noise performance and power supply rejection ratio.  
4. **Curvature-Compensated BGR** — Includes additional circuitry to minimize second-order temperature effects.

---

#### 🧠 Our Design Choice

In this project, we implement the **Bandgap Reference (BGR)** circuit using a **Self-Biased Current Mirror Architecture**,  
as it provides a good balance between **simplicity**, **power efficiency**, and **temperature stability** for integrated circuit applications.

### 2.3 Self-Biased Current Mirror Based BGR
The **Self-Biased Current Mirror Based Bandgap Reference (BGR)** circuit is composed of several functional sub-blocks that together generate a stable, temperature-independent reference voltage.

---

#### 🧩 Main Components

1. **CTAT Voltage Generation Circuit**  
   Produces a voltage that decreases with increasing temperature.

2. **PTAT Voltage Generation Circuit**  
   Produces a voltage that increases with temperature.

3. **Self-Biased Current Mirror Circuit**  
   Establishes a stable bias current without relying on an external source.

4. **Reference Branch Circuit**  
   Combines the PTAT and CTAT voltages to generate a constant reference voltage.

5. **Start-Up Circuit**  
### 2.3.1 CTAT Voltage Generation Circuit
<img width="222" height="292" alt="Screenshot 2025-10-31 120511" src="https://github.com/user-attachments/assets/f81c739d-d139-4442-9e03-85e1de35a908" />

### 2.3.2 PTAT Voltage Generation Circuit
<img width="487" height="488" alt="Screenshot 2025-10-31 120612" src="https://github.com/user-attachments/assets/03a0037b-3e24-4529-837d-b01bb89097ca" />

### 2.3.3 Self-Biased Current Mirror Circuit

The **Self-Biased Current Mirror** is a special type of current mirror that does **not require any external biasing source**.  
Instead, it **automatically establishes its own bias current** through internal feedback, achieving a stable operating point without relying on an external reference.

#### ⚙️ Working Principle

In a self-biased current mirror, the **bias current** is generated internally by the circuit configuration itself.  
This is typically achieved using **transistor feedback loops**, where one branch sets the reference voltage or current, and the mirror branch replicates it.

The circuit adjusts itself until the **voltages and currents stabilize** at a desired value — a state known as **self-biasing equilibrium**.  
This eliminates the need for an external current reference, making the design **compact, power-efficient, and self-sufficient**.
<img width="447" height="376" alt="Screenshot 2025-10-31 120810" src="https://github.com/user-attachments/assets/1d49e411-6297-44c1-a7b0-9ba2fd790ab9" />
---
### 2.3.4 Reference Branch Circuit

The **Reference Branch Circuit** is the core part of the Bandgap Reference (BGR) that performs the **addition of CTAT and PTAT voltages** to produce the final **constant reference voltage**.

This branch typically consists of a **mirror transistor** and a **BJT configured as a diode**.  
The mirror transistor ensures that the **same current** flowing through the current mirror branches also flows through the reference branch, maintaining bias symmetry across the circuit.

#### ⚙️ Working Principle

From the **PTAT generation circuit**, we obtain a **PTAT voltage** and a **PTAT current**.  
This PTAT current is mirrored into the reference branch, where it flows through a **resistor** connected in series with the **CTAT diode**.

However, the **slope of the PTAT voltage** is much smaller compared to that of the **CTAT voltage**.  
To balance these effects and achieve temperature independence, the **resistance value is increased** — since the current is constant, a higher resistance increases the voltage drop proportionally.
As a result, the total output voltage across the resistor becomes the **sum of the PTAT and CTAT components**, yielding a **temperature-stable reference voltage**.

<img width="168" height="487" alt="Screenshot 2025-10-31 122920" src="https://github.com/user-attachments/assets/77e86414-5d1f-42a8-8b0f-2a0c96c72d2b" />
---

### 2.3.5 Start-up Circuit

The **Start-up Circuit** is an essential part of the Bandgap Reference (BGR) design that ensures the **self-biased current mirror** starts operating correctly from power-up.

#### ⚙️ Function

In self-biased current mirrors, there exists a **degenerative bias point** where the circuit can settle into an **unwanted zero-current state**.  
Without intervention, the mirror could remain in this state indefinitely, preventing the circuit from reaching its intended operating condition.

To avoid this, a **start-up circuit** is introduced.  
This circuit **forces a small initial current** into the self-biased current mirror when it detects that the mirror current is zero.  
This small perturbation shifts the mirror out of the zero-current equilibrium point.

Once the circuit begins to conduct, the **self-biasing mechanism** of the current mirror takes over and automatically stabilizes the current to its desired operating value.

<img width="652" height="548" alt="Screenshot 2025-10-31 123141" src="https://github.com/user-attachments/assets/0367290b-ed4e-469e-bb0e-2898c3b3a790" />

### 2.3.6 Complete BGR Circuit

By combining all the previously discussed building blocks, we can construct the **Complete Bandgap Reference (BGR) Circuit**.

---

#### ⚙️ Circuit Composition

The complete BGR circuit integrates the following components:

1. **CTAT Voltage Generation Circuit** — provides a voltage that decreases with temperature using a BJT diode.  
2. **PTAT Voltage Generation Circuit** — produces a voltage that increases with temperature using resistors and matched BJTs.  
3. **Self-Biased Current Mirror Circuit** — establishes and maintains stable current levels without the need for external biasing.  
4. **Reference Branch Circuit** — sums the CTAT and PTAT components to generate the temperature-independent reference voltage.  
5. **Start-up Circuit** — ensures the self-biased current mirror starts correctly by eliminating the zero-current operating point.

---

#### 🧩 Working Principle

- The **CTAT** and **PTAT** voltages are carefully scaled and summed to achieve a **temperature-stable output voltage**.  
- The **current mirror** maintains proper biasing across all branches.  
- The **start-up circuit** guarantees reliable operation from power-up.  

Together, these components form a **fully functional Bandgap Reference circuit**, producing a **constant output voltage (~1.2 V)** that remains stable over variations in **temperature, supply voltage, and load conditions**.

<img width="940" height="612" alt="Screenshot 2025-10-31 123818" src="https://github.com/user-attachments/assets/dc1935a4-7a6e-4256-a452-1da815851cf2" />

#### ✅ Advantages of SBCM BGR

- **Simplest Topology:** The circuit structure is straightforward, making it easy to implement.  
- **Ease of Design:** Requires fewer components and has a simpler biasing mechanism compared to op-amp-based BGRs.  
- **Always Stable:** The self-biasing mechanism ensures a stable operating point once the circuit starts.  

#### ⚠️ Limitations of SBCM BGR

- **Low Power Supply Rejection Ratio (PSRR):** More sensitive to supply voltage fluctuations.  
- **Cascode Design Required:** A cascode structure may be added to improve PSRR performance.  
- **Voltage Headroom Issue:** Limited voltage swing can affect proper operation in low-voltage designs.  
- **Requires Start-up Circuit:** Essential to prevent the circuit from remaining in the zero-current state.

## 3. Design and Pre-layout Simulation

For the practical implementation of the Bandgap Reference (BGR) circuit, the **SkyWater SKY130 (130 nm)** PDK is used.  
Before designing the complete circuit, we must first define the **design requirements** that our circuit should meet.

---

### 3.1 Design Requirements

| Parameter | Specification |
|------------|----------------|
| Supply Voltage (VDD) | 1.8 V |
| Temperature Range | -40°C to 125°C |
| Power Consumption | < 60 µW |
| Off Current | < 2 µA |
| Start-up Time | < 2 µs |
| Temperature Coefficient (Tempco) of Vref | < 50 ppm/°C |

---

### 3.2 Device Data Sheet

#### 1. MOSFET

| Parameter | NFET | PFET |
|------------|-------|-------|
| Type | LVT | LVT |
| Voltage Rating | 1.8 V | 1.8 V |
| Threshold Voltage (Vt0) | ~0.4 V | ~-0.6 V |
| Model | sky130_fd_pr__nfet_01v8_lvt | sky130_fd_pr__pfet_01v8_lvt |

---

#### 2. Bipolar Junction Transistor (PNP)

| Parameter | PNP |
|------------|------|
| Current Rating | 1 µA – 10 µA/µm² |
| Beta (β) | ~12 |
| Area | 11.56 µm² |
| Model | sky130_fd_pr__pnp_05v5_W3p40L3p40 |

---

#### 3. Resistor (RPOLYH)

| Parameter | RPOLYH |
|------------|----------|
| Sheet Resistance | ~350 Ω/sq |
| Tempco | 2.5 Ω/°C |
| Available Widths | 0.35 µm, 0.69 µm, 1.41 µm, 5.37 µm |
| Model | sky130_fd_pr__res_high_po |

---

### 3.3 Circuit Design

#### 1. Current Calculation

Maximum Power Consumption = 60 µW  
Supply Voltage = 1.8 V  

Total Current = 60 µW / 1.8 V = 33.33 µA  

Hence, 10 µA per branch is selected (3 × 10 = 30 µA)  
Start-up current = 1–2 µA  

---

#### 2. Choosing Number of BJTs in Branch 2

- Fewer BJTs → smaller resistance but poorer matching  
- More BJTs → higher resistance but better matching  

Chosen compromise: **8 BJTs** in parallel for good matching and moderate resistance.

---

#### 3. Calculation of R1

R1 = (Vt × ln(8)) / I  
R1 = (26 mV × ln(8)) / 10.7 µA ≈ 5 kΩ  

R1 Size:  
- W = 1.41 µm  
- L = 7.8 µm  
- Unit resistance = 2 kΩ  

Resistor implementation: 2 in series and 2 in parallel (2 + 2 + (2‖2))

---

#### 4. Calculation of R2

Current through reference branch:  
I3 = I2 = (Vt × ln(8)) / R1  

Voltage across R2:  
VR2 = R2 × I3 = (R2 / R1) × (Vt × ln(8))  

Slope of VR2 = (R2 / R1) × (ln(8) × 115 µV/°C)  
Slope of VQ3 = -1.6 mV/°C  

For zero temperature coefficient,  
Total slope = 0 → R2 ≈ 33 kΩ  

Resistor implementation: 16 in series and 2 in parallel (2 + 2 + … + 2 + (2‖2))

---

#### 5. Self-Biased Current Mirror (SBCM) Design

##### A. PMOS Design (MP1, MP2)

- Operate both transistors in saturation region.  
- Increase channel length to reduce channel length modulation.  
- Final size: L = 2 µm, W = 5 µm, M = 4  

##### B. NMOS Design (MN1, MN2)

- Operate both transistors either in saturation or deep subthreshold region.  
- Here, they are designed to work in deep subthreshold region.  
- Increase channel length to improve stability.  
- Final size: L = 1 µm, W = 5 µm, M = 8  

---
### 3.3.1 Final Circuit
<img width="1027" height="647" alt="Screenshot 2025-10-31 144626" src="https://github.com/user-attachments/assets/bcb02bda-5746-41d8-a6f6-3e4952a4ed85" />

### 3.4 Writing Spice netlist and Pre-layout simulation

#### Steps to write a netlist

1. Create a file with `.sp` extension, open with any editor like `gvim` / `vim` / `nano`.
2. The 1st line of the Spice netlist is by default a comment line.
3. To write a valid netlist we must include the library file (with absolute path) and mention the corner name (tt, ff or ss).

<img width="712" height="487" alt="Screenshot 2025-10-31 145512" src="https://github.com/user-attachments/assets/f2c4606a-b998-4c49-b491-bb91ff42a173" />
## 🔍 Netlist Explanation (Sky130 BGR Subcircuit)

### 1️⃣ Global and Temperature Setup
- `.global vdd gnd` → Declares **VDD** and **GND** as global nodes, accessible throughout the design.  
- `.temp 27` → Sets the **simulation temperature** to **27°C (room temperature)**.

---

### 2️⃣ Voltage-Controlled Voltage Source (VCVS)
```spice
*** vcvs definition
e1 ra1 qp1 net2 gnd gain=1000
````
e1 defines a VCVS (Voltage-Controlled Voltage Source).

Input nodes: qp1 and net2

Output nodes: ra1 and gnd

gain=1000 → Output voltage = 1000 × (V(qp1) - V(net2))

Used for amplification or feedback control in analog reference circuits.
3️⃣ MOSFET Definition (PMOS Devices)
```spice
*** mosfet definition
xmp1 q1 net2 vdd vdd sky130_fd_pr__pfet_01v8_lvt l=2 w=5 m=4
xmp2 q2 net2 vdd vdd sky130_fd_pr__pfet_01v8_lvt l=2 w=5 m=4
```
xmp1, xmp2 are PMOS transistors used in bias or mirror configurations.

Model: sky130_fd_pr__pfet_01v8_lvt → 1.8V Low-Threshold PMOS (from Sky130 PDK).

Node order: Drain → Gate → Source → Bulk

Parameters:

l=2 → Channel Length = 2µm

w=5 → Channel Width = 5µm

m=4 → 4 parallel transistors for higher drive strength and better matching.

Both transistors share the same gate (net2) to form a current mirror or load pair.

4️⃣ Resistor Definition
```spice
**resistor definition
xra ra1 qp2 gnd sky130_fd_pr__res_high_po_1p41 l=30
```
xra defines a high-poly resistor using Sky130 PDK.

Model: sky130_fd_pr__res_high_po_1p41 → High-Resistivity Polysilicon Resistor.

Nodes: Between ra1 and qp2, connected to gnd.

Parameter: l=30 → Resistor length = 30µm (resistance ∝ length).

Used to generate voltage drops or temperature-dependent resistances in the circuit.

5️⃣ BJT (PNP Transistor) Definition
```spice
**bjt definition
xqp1 gnd gnd qp1 gnd sky130_fd_pr__pnp_05v5_w3p40l3p40 m=1
xqp2 gnd gnd qp2 gnd sky130_fd_pr__pnp_05v5_w3p40l3p40 m=8
```
xqp1, xqp2 are PNP BJTs used for CTAT and PTAT voltage generation.

Model: sky130_fd_pr__pnp_05v5_w3p40l3p40 → 5V PNP transistor from SkyWater PDK.

Node order: Collector → Base → Emitter → Substrate

Parameters:

m=1 → Single transistor (for base reference branch).

m=8 → 8 parallel BJTs (used to adjust emitter area and current density).

Increasing m improves matching and modifies Vbe slope for temperature compensation.

6️⃣ Vim Command
```spice
:wq
```
Saves (:w) and quits (:q) the file in Vim or GVim editor after writing the netlist.





<img width="953" height="692" alt="Screenshot 2025-10-31 145732" src="https://github.com/user-attachments/assets/d3cd580a-22dc-4739-8d41-5de36279e459" />












