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

---

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

-Create a directory for the PDK
-Clone the SkyWater PDK repository
-Initialize submodules (if required)
-Build or install PDK libraries (optional)
-Set the PDK path so tools like Magic, Ngspice, and Netgen can locate it easily.

<img width="975" height="356" alt="Screenshot 2025-10-26 225127" src="https://github.com/user-attachments/assets/dd53a7e6-03a4-4d06-88a8-c8bf3c010cd1" />

