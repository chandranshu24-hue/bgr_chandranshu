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
### 3️⃣ MOSFET Definition (PMOS Devices)
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

### 4️⃣ Resistor Definition
```spice
**resistor definition
xra ra1 qp2 gnd sky130_fd_pr__res_high_po_1p41 l=30
```
xra defines a high-poly resistor using Sky130 PDK.

Model: sky130_fd_pr__res_high_po_1p41 → High-Resistivity Polysilicon Resistor.

Nodes: Between ra1 and qp2, connected to gnd.

Parameter: l=30 → Resistor length = 30µm (resistance ∝ length).

Used to generate voltage drops or temperature-dependent resistances in the circuit.

 ### 5️⃣ BJT (PNP Transistor) Definition
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

 ### 6️⃣ Vim Command
```spice
:wq
```
Saves (:w) and quits (:q) the file in Vim or GVim editor after writing the netlist.


<img width="953" height="692" alt="Screenshot 2025-10-31 145732" src="https://github.com/user-attachments/assets/d3cd580a-22dc-4739-8d41-5de36279e459" />

## 3.4.1 ⚙️ CTAT Simulation  
### CTAT Voltage Generation with Single BJT Netlist

#### 🧠 Theory  
- A **BJT used as a diode** (by shorting its base and collector) produces a voltage that **decreases linearly with temperature**.  
- When a **constant current source (10 µA)** flows through the BJT, the **base-emitter voltage (V_BE)** shows a **negative temperature coefficient** (typically −2 mV/°C).  
- This negative slope of **V_BE vs. Temperature** represents the **CTAT characteristic**.

#### ⚡ Circuit Setup  
- **Device Used:** `sky130_fd_pr__pnp_05v5_w3p40l3p40`  
- **Bias Current:** 10 µA (constant current source)  
- **Output Measured:** Voltage across BJT (V_BE)   

#### 🧾 Expected Output  
A **straight line with a negative slope** in the **V_BE vs. Temperature** plot:  
> As temperature increases → V_BE decreases → CTAT behavior confirmed ✅  

#### 🖥️ Simulation Command  
Open your terminal and navigate to the **prelayout** directory.  
Run the following command to launch the simulation:

```bash
cd /workspaces/vsd-bandgap/bandgap/prelayout/
ngspice ctat_voltage_gen.sp
```

After simulation we can get a wavefrom like below, and from the wavefrom we can see the CTAT behaviour of the BJT, and can find the slope.

<img width="911" height="659" alt="Screenshot 2025-10-29 170822" src="https://github.com/user-attachments/assets/eaf920cc-0284-4cbc-ac74-c9c0dd8ae2f8" />

<img width="926" height="589" alt="Screenshot 2025-10-29 172431" src="https://github.com/user-attachments/assets/e4defc35-d568-459d-b876-5e8c0d4509ba" />

### CTAT Voltage generation with Multiple BJT netlist

In this simulation we will check the CTAT voltage across the 8 parallel connected BJTs
<img width="981" height="492" alt="Screenshot 2025-10-31 152650" src="https://github.com/user-attachments/assets/015da634-c3b7-43ae-9685-58a3b8af7e95" />

<img width="1171" height="761" alt="Screenshot 2025-10-31 152525" src="https://github.com/user-attachments/assets/6dfb5edd-e29b-4de2-91f1-3f8e83a1ed72" />

we can see the slope is increasing in case of multiple BJTs.

### CTAT Voltage generation with different current source values netlist
<img width="945" height="737" alt="Screenshot 2025-10-29 173850" src="https://github.com/user-attachments/assets/5aa8fa5b-4063-4cc9-a154-9024c2f36fb9" />

### 3.4.2  PTAT Simulation

#### PTAT Voltage generation with VCVS
<img width="1668" height="801" alt="Screenshot 2025-10-30 121222" src="https://github.com/user-attachments/assets/450abfb4-5814-45b1-b4b2-369d50996ab0" />
## 3.4.3 ⚙️ Resistance Temperature Coefficient (Tempco)

### 🧠 Theory  
- A resistor has a **positive temperature coefficient**, meaning its resistance increases as temperature increases.  
- When a constant current (10 µA) flows through the resistor, the voltage across it is given by:

  V_R = I × R(T)

  where R(T) is the resistance that changes with temperature.  
- As temperature rises, R(T) increases, so the voltage V_R also increases.  
- Therefore, the voltage across the resistor behaves as a **PTAT voltage**.  
- In a Bandgap Reference (BGR) circuit, this PTAT voltage adds to the thermal voltage from the BJT to cancel the CTAT behavior.

  <img width="1594" height="669" alt="Screenshot 2025-10-31 153409" src="https://github.com/user-attachments/assets/62894e4b-dc09-4867-b6da-942f0fa2965e" />

### 🖥️ Simulation Command
```spice 
cd /workspaces/vsd-bandgap/bandgap/prelayout/
ngspice res_tempco.sp
```
Also we can find the PTAT voltages across the resistance for different current values from the following curve.
<img width="847" height="706" alt="Screenshot 2025-10-31 171039" src="https://github.com/user-attachments/assets/ee692fff-3951-4e81-97c5-9390213c635b" />

### 3.4.4 BGR using Ideal OpAmp

Now after simulating all our components, let's quick check our BGR behaviour using one **VCVS** as an **ideal OpAmp**.

In this simulation, we should get the reference voltage as an **umbrella-shaped curve** and it should be approximately **1.2V**.
```spice
*** bgr using ideal opamp (vcvs) *****

.lib "/opt/pdk/sky130A/libs.tech/ngspice/sky130.lib.spice tt"

.global vdd gnd
.temp 27

*** vcvs definition
e1 net2 gnd ra1 qp1 gain=1000


xmp1    qp1     net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
xmp2    ra1     net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
xmp3    ref     net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4

*** bjt definition
xqp1    gnd     gnd     qp1             sky130_fd_pr__pnp_05v5_W3p40L3p40       m=1
xqp2    gnd     gnd     qp2          sky130_fd_pr__pnp_05v5_W3p40L3p40       m=8
xqp3    gnd     gnd     qp3          sky130_fd_pr__pnp_05v5_W3p40L3p40       m=1

*** high-poly resistance definition
xra1    ra1     na1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xra2    na1     na2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xra3    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xra4    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8

xrb1    ref     nb1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb2    nb1     nb2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb3    nb2     nb3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb4    nb3     nb4     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb5    nb4     nb5     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb6    nb5     nb6     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb7    nb6     nb7     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb8    nb7     nb8     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb9    nb8     nb9     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb10   nb9     nb10    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb11   nb10    nb11    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb12   nb11    nb12    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb13   nb12    nb13    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb14   nb13    nb14    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb15   nb14    nb15    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb16   nb15    nb16    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb17   nb16    nb17    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb18   nb17    nb18    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb19   nb18    nb19    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb20   nb19    nb20    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41        l=7.8
xrb21   nb20    nb21    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb22   nb21    nb22    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41        l=7.8
xrb23   nb22    qp3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8
xrb24   nb22    qp3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41       l=7.8

*** voltage source for current measurement

*** supply voltage
vsup    vdd     gnd     dc      2
*.dc    vsup    0       3.3     0.3.3

.dc     temp    -40     125     5

*vsup    vdd     gnd     pulse   0       2       10n     1u      1u      1m      100u
*.tran   5n      10u

.control
RUN
plot v(vdd) v(qp1) v(ra1) v(qp2) v(ref) v(qp3)
plot v(ref)

.endc
.end
```
<img width="849" height="729" alt="Screenshot 2025-10-30 135631" src="https://github.com/user-attachments/assets/5c92affc-e989-4db8-9814-97929159f43e" />
<img width="926" height="486" alt="Screenshot 2025-10-30 135732" src="https://github.com/user-attachments/assets/8f676c81-5902-4239-89a9-0e23c9610bcd" />
<img width="886" height="680" alt="Screenshot 2025-10-30 135839" src="https://github.com/user-attachments/assets/fac527cb-056a-4c4d-9a70-12d01c502d1c" />
<img width="1595" height="693" alt="Screenshot 2025-10-30 140003" src="https://github.com/user-attachments/assets/c4a4627c-bf06-407b-9062-8cf72d657c42" />
<img width="780" height="397" alt="Screenshot 2025-10-30 140140" src="https://github.com/user-attachments/assets/6cba83d9-dabc-4398-9eed-5fa986ca86bf" />
<img width="910" height="677" alt="Screenshot 2025-10-30 140355" src="https://github.com/user-attachments/assets/04747cd3-fe1b-4d25-8a6a-4ade8f5e17c0" />

### 3.4.5 BGR with selfbias current mirror
Now we will replace the ideal Op-Amp with self-biased current mirror which is our proposed design. We expect same type of output as in case of ideal OpAmp based BGR. We will also check for different corners, and will see how our circuit is performing in different corners. 
 ### tt corner stimulation
 ```spice
 **** bandgap reference circuit using self-biase current mirror *****

.lib "/opt/pdk/sky130A/libs.tech/ngspice/sky130.lib.spice tt"

.global vdd gnd
.temp 27

*** circuit definition ***

*** mosfet definitions self-biased current mirror and output branch
xmp1    net1    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
xmp2    net2    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
xmp3    net3    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
xmn1    net1    net1    q1      gnd     sky130_fd_pr__nfet_01v8_lvt     l=1     w=5     m=8
xmn2    net2    net1    q2      gnd     sky130_fd_pr__nfet_01v8_lvt     l=1     w=5     m=8

*** start-upcircuit
xmp4    net4    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=1
xmp5    net5    net2    net4    vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=1
xmp6    net7    net6    net2    vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=2
xmn3    net6    net6    net8    gnd     sky130_fd_pr__nfet_01v8_lvt     l=7     w=1     m=1
xmn4    net8    net8    gnd     gnd     sky130_fd_pr__nfet_01v8_lvt     l=7     w=1     m=1

*** bjt definition
xqp1    gnd     gnd     qp1             sky130_fd_pr__pnp_05v5_W3p40L3p40       m=1
xqp2    gnd     gnd     qp2          sky130_fd_pr__pnp_05v5_W3p40L3p40       m=8
xqp3    gnd     gnd     qp3          sky130_fd_pr__pnp_05v5_W3p40L3p40       m=1

*** high-poly resistance definition
xra1    ra1     na1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra2    na1     na2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra3    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra4    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb1    vref    nb1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb2    nb1     nb2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb3    nb2     nb3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb4    nb3     nb4     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb5    nb4     nb5     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb6    nb5     nb6     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb7    nb6     nb7     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb8    nb7     nb8     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb9    nb8     nb9     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb10   nb9     nb10    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb11   nb10    nb11    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb12   nb11    nb12    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb13   nb12    nb13    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb14   nb13    nb14    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb15   nb14    nb15    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb16   nb15    nb16    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb17   nb16    qp3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb18   nb16    qp3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8

*** voltage source for current measurement
vid1    q1      qp1     dc      0
vid2    q2      ra1     dc      0
vid3    net3    vref    dc      0
vid4    net7    net1    dc      0
vid5    net5    net6    dc      0

*** supply voltage
vsup    vdd     gnd     dc      2
*.dc    vsup    0       3.3     0.3.3
.dc     temp    -40     125     5

*vsup   vdd     gnd     pulse   0       2       10n     1u      1u      1m      100u
*.tran  5n      10u

.control
run

plot v(vdd) v(net1) v(net2) v(qp1) v(ra1) v(qp2) v(vref) v(qp3)
plot vid1#branch vid2#branch vid3#branch vid4#branch vid5#branch

.endc
.end
```
<img width="1530" height="698" alt="Screenshot 2025-10-31 173959" src="https://github.com/user-attachments/assets/f10be323-9177-4953-a5e4-badaa7a12ef2" />
<img width="861" height="663" alt="Screenshot 2025-10-31 174112" src="https://github.com/user-attachments/assets/94d77d82-baee-44ae-83d6-a4ea4626ffac" />
Tempco. Of Vref = ~21.7 PPM

### Behaviour in FF corner
``` spice
**** bandgap reference circuit using self-biase current mirror at ff corner*****

.lib "/opt/pdk/sky130A/libs.tech/ngspice/sky130.lib.spice ff"

.global vdd gnd
.temp 27

*** circuit definition ***

*** mosfet definitions self-biased current mirror and output branch
xmp1    net1    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
xmp2    net2    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
xmp3    net3    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
xmn1    net1    net1    q1      gnd     sky130_fd_pr__nfet_01v8_lvt     l=1     w=5     m=8
xmn2    net2    net1    q2      gnd     sky130_fd_pr__nfet_01v8_lvt     l=1     w=5     m=8

*** start-upcircuit
xmp4    net4    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=1
xmp5    net5    net2    net4    vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=1
xmp6    net7    net6    net2    vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=2
xmn3    net6    net6    net8    gnd     sky130_fd_pr__nfet_01v8_lvt     l=7     w=1     m=1
xmn4    net8    net8    gnd     gnd     sky130_fd_pr__nfet_01v8_lvt     l=7     w=1     m=1

*** bjt definition
xqp1    gnd     gnd     qp1             sky130_fd_pr__pnp_05v5_W3p40L3p40       m=1
xqp2    gnd     gnd     qp2          sky130_fd_pr__pnp_05v5_W3p40L3p40       m=8
xqp3    gnd     gnd     qp3          sky130_fd_pr__pnp_05v5_W3p40L3p40       m=1

*** high-poly resistance definition
xra1    ra1     na1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra2    na1     na2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra3    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra4    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb1    vref    nb1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb2    nb1     nb2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb3    nb2     nb3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb4    nb3     nb4     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb5    nb4     nb5     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb6    nb5     nb6     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb7    nb6     nb7     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb8    nb7     nb8     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb9    nb8     nb9     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb10   nb9     nb10    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb11   nb10    nb11    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb12   nb11    nb12    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb13   nb12    nb13    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb14   nb13    nb14    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb15   nb14    nb15    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb16   nb15    nb16    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb17   nb16    qp3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb18   nb16    qp3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8

*** voltage source for current measurement
vid1    q1      qp1     dc      0
vid2    q2      ra1     dc      0
vid3    net3    vref    dc      0
vid4    net7    net1    dc      0
vid5    net5    net6    dc      0

*** supply voltage
vsup    vdd     gnd     dc      2
*.dc    vsup    0       3.3     0.3.3
.dc     temp    -40     125     5

*vsup   vdd     gnd     pulse   0       2       10n     1u      1u      1m      100u
*.tran  5n      10u
.control
run

plot v(vdd) v(net1) v(net2) v(qp1) v(ra1) v(qp2) v(vref) v(qp3)
plot vid1#branch vid2#branch vid3#branch vid4#branch vid5#branch

.endc
.end
`````
<img width="799" height="698" alt="Screenshot 2025-10-30 150616" src="https://github.com/user-attachments/assets/19282c78-5c5c-411f-a69f-4e25440c206d" />

Tempco. Of Vref = ~10 PPM
### Behaviour in ss corner
``` spice
**** bandgap reference circuit using self-biase current mirror at ss corner****

.lib "/opt/pdk/sky130A/libs.tech/ngspice/sky130.lib.spice ss"

.global vdd gnd
.temp 27

*** circuit definition ***

*** mosfet definitions self-biased current mirror and output branch
xmp1	net1	net2	vdd	vdd	sky130_fd_pr__pfet_01v8_lvt	l=2	w=5	m=4
xmp2    net2    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5    	m=4
xmp3    net3    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
xmn1    net1    net1    q1      gnd     sky130_fd_pr__nfet_01v8_lvt     l=1     w=5     m=8
xmn2    net2    net1    q2      gnd     sky130_fd_pr__nfet_01v8_lvt     l=1     w=5     m=8

*** start-upcircuit
xmp4    net4    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=1
xmp5    net5    net2    net4    vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=1
xmp6    net7    net6    net2    vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=2
xmn3    net6    net6    net8    gnd     sky130_fd_pr__nfet_01v8_lvt     l=7     w=1     m=1
xmn4    net8    net8    gnd     gnd     sky130_fd_pr__nfet_01v8_lvt     l=7     w=1     m=1

*** bjt definition
xqp1	gnd	gnd	qp1		sky130_fd_pr__pnp_05v5_W3p40L3p40	m=1
xqp2    gnd     gnd     qp2          sky130_fd_pr__pnp_05v5_W3p40L3p40       m=8
xqp3    gnd     gnd     qp3          sky130_fd_pr__pnp_05v5_W3p40L3p40       m=1

*** high-poly resistance definition
xra1	ra1	na1	vdd	sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra2	na1     na2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra3    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra4    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8

xrb1    vref    nb1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb2    nb1     nb2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb3    nb2     nb3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb4    nb3     nb4     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb5    nb4     nb5     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb6    nb5     nb6     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb7    nb6     nb7     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb8    nb7     nb8     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb9    nb8     nb9     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb10   nb9     nb10    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb11   nb10    nb11    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb12   nb11    nb12    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb13   nb12    nb13    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb14   nb13    nb14    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb15   nb14    nb15    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb16   nb15    nb16    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb17   nb16    qp3   	vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb18   nb16    qp3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8

*** voltage source for current measurement
vid1	q1	qp1	dc	0
vid2    q2      ra1     dc      0
vid3    net3    vref    dc      0
vid4    net7	net1	dc	0
vid5	net5	net6	dc	0

*** supply voltage
vsup	vdd	gnd	dc 	2
*.dc	vsup	0	3.3	0.3.3
.dc	temp	-40	125	5

*vsup	vdd	gnd	pulse	0	2	10n	1u	1u	1m	100u
*.tran	5n	10u

.control
run

plot v(vdd) v(net1) v(net2) v(qp1) v(ra1) v(qp2) v(vref) v(qp3)
plot vid1#branch vid2#branch vid3#branch vid4#branch vid5#branch

.endc
.end

```
<img width="843" height="706" alt="Screenshot 2025-10-30 150017" src="https://github.com/user-attachments/assets/9394d754-b91d-4c96-92a2-56527d8950fe" />
Tempco. Of Vref = ~45 PPM







---





















