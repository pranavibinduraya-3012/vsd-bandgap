# Analog Bandgap Reference Voltage Circuit Design

This project documents my complete design and characterization of an Analog Bandgap Voltage Reference circuit using the SKY130 PDK, covering schematic design and SPICE simulations in Ngspice, PVT analysis, and layout implementation using Magic.

## Objective
To design a stable voltage reference independent of temperature and process variations.
# *INTRODUCTION*

A Bandgap Reference (BGR) is a fundamental analog IP block used in almost every mixed-signal IC and SoC. Modern systems include multiple analog and digital subsystems—such as ADCs, DACs, LDOs, PLLs, and bias networks—that require a stable and accurate reference voltage (VREF). This reference must remain consistent despite variations in temperature, supply voltage, and process corners, making BGR circuits an essential part of precision analog design.

The core principle behind a Bandgap Reference is to combine two temperature-dependent voltages whose temperature coefficients cancel each other:

### CTAT (Complementary to Absolute Temperature) Voltage  
- Derived from the base–emitter voltage (VBE) of a bipolar transistor  
- V<sub>BE</sub> decreases with increasing temperature (negative temperature coefficient).

### PTAT (Proportional to Absolute Temperature) Voltage  
- Generated from the difference in VBE (ΔVBE) of two BJTs operating at different current densities  
- ΔV<sub>BE</sub> increases with temperature (positive temperature coefficient).
  
By appropriately scaling and summing these two quantities, the negative temperature coefficient of VBE and the positive coefficient of ΔVBE cancel out, producing a **constant temperature-independent** reference voltage. This output is typically around **1.2 V**, the theoretical silicon bandgap voltage extrapolated to 0 K.
V<sub>ref</sub>(T) ≈ V<sub>BE</sub>(T) + k · ΔV<sub>BE</sub>(T)

---

## 📌 Performance Requirements

A practical BGR must exhibit:

- **High PSRR:** 40–60 dB (good supply rejection)  
- **Low Temperature Coefficient:** 10–50 ppm/°C  
- **Robust behavior across PVT corners**  
- **Stable bias currents** for driving analog and mixed-signal blocks  

---

### ⚙ Working Principle (Simplified)
At its core, the Bandgap Reference circuit works as follows:
1. Generate a CTAT voltage from a diode-connected BJT (V<sub>BE</sub>).
2. Generate a PTAT voltage using two BJTs with different emitter area ratios.
3. Add the PTAT voltage (positive TC) to the CTAT voltage (negative TC) in proper proportions.
4. The resulting sum is a **temperature-independent reference voltage**.

---

## 🗝️ Key Features of Bandgap Reference (BGR)

- Temperature-independent voltage reference circuit widely used in Integrated Circuits (ICs).  
- Produces a **constant output voltage** regardless of power supply variation, temperature changes, and circuit loading.  
- Typical **output voltage ≈ 1.2 V**, which is close to the **bandgap energy of silicon at 0 K**.  
- Used in almost all types of circuits — **analog, digital, mixed-signal, RF, and System-on-Chip (SoC)** designs.

---

##  Applications of Bandgap Reference (BGR)

- **Low Dropout Regulators (LDOs)**  
- **DC-to-DC Buck Converters**  
- **Analog-to-Digital Converters (ADCs)**  
- **Digital-to-Analog Converters (DACs)**

---

## 1. Tool and PDK Setup
### 1.1 Tools Setup

For the design, simulation, and verification of the Bandgap Reference (BGR) circuit, the following open-source EDA tools are used:

---

#### Ngspice — Circuit Simulation
**Ngspice** is an open-source SPICE-based simulator used for performing **analog circuit simulations**.  
It takes a **SPICE netlist** as input, which describes the circuit components and their connections, and then computes electrical parameters such as node voltages, currents, and transfer characteristics.  
In this project, Ngspice is used to:
- Simulate the **schematic-level design** of the BGR circuit.  
- Analyze **DC**, **AC**, and **transient** behavior.  
- Verify **temperature dependence** and output voltage stability.

-Steps to install Ngspice - Open the terminal and type the following to install Ngspice
```bash
$  sudo apt-get install ngspice
```

#### Magic — Layout Design and DRC
**Magic** is a VLSI layout editor developed by Berkeley, primarily used for **IC layout design** in open-source PDKs such as **Sky130**.  
It provides interactive tools for drawing transistors, interconnects, and layers according to process design rules.  
Magic is used here to:
- Create the **physical layout** of the BGR circuit.  
- Perform **Design Rule Check (DRC)** to ensure the layout complies with the fabrication constraints of the Sky130 process.  
- Extract the layout to generate a **SPICE netlist** for post-layout simulations.

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

#### Netgen — LVS (Layout vs. Schematic)
**Netgen** is a layout verification tool used for **Layout Versus Schematic (LVS)** comparison.  
It compares the netlist extracted from the layout (using Magic) with the schematic netlist (used in Ngspice simulation) to verify connectivity and device matching.  
A successful LVS ensures that the **layout accurately represents the schematic**, confirming the design’s electrical integrity before fabrication.

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

---

## 1. BGR Introduction

###  BGR Principle

The **Bandgap Reference (BGR)** circuit generates a temperature-independent reference voltage by combining two voltage components with **opposite temperature coefficients**.

The basic operation principle of a BGR circuit is to **sum a voltage with a negative temperature coefficient (CTAT)** and another with a **positive temperature coefficient (PTAT)** such that their variations cancel each other.

<img width="1304" height="712" alt="Screenshot 2025-11-15 153624" src="https://github.com/user-attachments/assets/59185030-7951-4355-9f07-c82240fe3590" />


---

####  Key Concepts

- **CTAT (Complementary to Absolute Temperature):**  
  A voltage that **decreases** as temperature **increases**.  
  Typically, the **base-emitter voltage (V<sub>BE</sub>)** of a bipolar junction transistor (BJT) or a diode exhibits CTAT behavior.
  

- **PTAT (Proportional to Absolute Temperature):**  
  A voltage that **increases** as temperature **increases**.  
  This can be generated using the **difference between two V<sub>BE</sub>** voltages of transistors operating at different current densities.

---

####  Principle of Operation

The BGR circuit operates by **adding** the CTAT and PTAT voltages in proper proportion so that the resulting voltage remains constant over temperature.
####  1.1.1 CTAT VOLTAGE GENERATION
Semiconductor diodes typically exhibit CTAT (Complementary to Absolute Temperature) behavior. When a constant current flows through a forward-biased diode, an increase in temperature causes the voltage across the diode to decrease. Experimentally, the rate of decrease of the diode’s forward voltage with temperature is approximately –2 mV/°C.

![IMG-20251115-WA0003 1](https://github.com/user-attachments/assets/6e13b34f-88b8-4988-bf0f-1da965326dd7)

![IMG-20251115-WA0002 1](https://github.com/user-attachments/assets/9dbf44ee-6a47-479d-be32-10705858b2f0)

####   PTAT VOLTAGE GENERATION

From the diode current equation, it can be observed that the diode voltage consists of two main temperature-dependent components:

Thermal voltage (Vₜ) — This term is directly proportional to temperature (approximately of order ~1).

Reverse saturation current (Iₛ) — This term increases with temperature approximately with an order of ~2.5.

Since Iₛ appears in the denominator of the logarithmic term (ln(I₀/Iₛ)), an increase in temperature causes this term to decrease, resulting in the CTAT behavior of the diode voltage.

Therefore, to design a PTAT (Proportional to Absolute Temperature) voltage generation circuit, we need a method to isolate the Vₜ component from the Iₛ dependence.

The following approach describes how this separation can be achieved.

In the above circuit same amount of current I is flowing in both the branches. So the node voltage A and B are going to be same V. Now in the B branch if we substract V1 from V, we get Vt independent of Is.


V= Combined Voltage across R1 and Q2 (CTAT in nature but less sloppy)
V1= Voltage across Q2 (CTAT in nature but more sloppy)
V-V1= Voltage across R1 (PTAT in nature)

From the above analysis, it is evident that the voltage difference (V – V₁) exhibits a PTAT (Proportional to Absolute Temperature) behavior. However, its slope is relatively small compared to the CTAT (Complementary to Absolute Temperature) characteristic of a diode.

To enhance the PTAT slope, multiple BJTs configured as diodes can be used in parallel. This reduces the current flowing through each individual diode, which in turn increases the slope of the (V – V₁) characteristic, thereby improving the PTAT response.

![IMG-20251115-WA0001 1](https://github.com/user-attachments/assets/f3bb9d6c-0150-484b-8466-e0d912e3d1c9)

<img width="412" height="395" alt="Screenshot 2025-11-15 155345" src="https://github.com/user-attachments/assets/2223fb30-0c49-4a29-9d26-07584e1bb1f2" />



####  Summary

- **Diode / BJT junction** provides the **CTAT** component.  
- **Difference in V<sub>BE</sub>** between transistors provides the **PTAT** component.  
- When both are combined properly, the **temperature variations cancel**, producing a **constant reference voltage (~1.2 V)** close to the **bandgap voltage of silicon**.

---

 *In simple terms, the BGR circuit uses one voltage that decreases with temperature and another that increases with temperature — when added in the right ratio, the overall result stays constant.*

###  Types of Bandgap Reference (BGR)

The **Bandgap Reference (BGR)** circuit can be classified in different ways depending on its **circuit architecture** and **application requirements**.

---

####  Architecture-wise Classification

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

####  Application-wise Classification

Depending on design goals and target specifications, BGR circuits can be categorized as:

1. **Low-Voltage BGR** — Optimized to operate at reduced supply voltages.  
2. **Low-Power BGR** — Designed for minimal power consumption, suitable for battery-powered systems.  
3. **High-PSRR and Low-Noise BGR** — Provides improved noise performance and power supply rejection ratio.  
4. **Curvature-Compensated BGR** — Includes additional circuitry to minimize second-order temperature effects.

---

####  Our Design Choice

In this project, we implement the **Bandgap Reference (BGR)** circuit using a **Self-Biased Current Mirror Architecture**,  
as it provides a good balance between **simplicity**, **power efficiency**, and **temperature stability** for integrated circuit applications.

###  Self-Biased Current Mirror Based BGR
The **Self-Biased Current Mirror Based Bandgap Reference (BGR)** circuit is composed of several functional sub-blocks that together generate a stable, temperature-independent reference voltage.

---

####  Main Components

1. **CTAT Voltage Generation Circuit**  
   Produces a voltage that decreases with increasing temperature.

2. **PTAT Voltage Generation Circuit**  
   Produces a voltage that increases with temperature.

3. **Self-Biased Current Mirror Circuit**  
   Establishes a stable bias current without relying on an external source.

4. **Reference Branch Circuit**  
   Combines the PTAT and CTAT voltages to generate a constant reference voltage.

5. **Start-Up Circuit**  

###  Self-Biased Current Mirror Circuit

The **Self-Biased Current Mirror** is a special type of current mirror that does **not require any external biasing source**.  
Instead, it **automatically establishes its own bias current** through internal feedback, achieving a stable operating point without relying on an external reference.

####  Working Principle

In a self-biased current mirror, the **bias current** is generated internally by the circuit configuration itself.  
This is typically achieved using **transistor feedback loops**, where one branch sets the reference voltage or current, and the mirror branch replicates it.

The circuit adjusts itself until the **voltages and currents stabilize** at a desired value — a state known as **self-biasing equilibrium**.  
This eliminates the need for an external current reference, making the design **compact, power-efficient, and self-sufficient**.

<img width="552" height="467" alt="Screenshot 2025-11-15 155742" src="https://github.com/user-attachments/assets/873902b4-3495-4b36-929d-cf1b8839efb3" />


---
###  Reference Branch Circuit

The **Reference Branch Circuit** is the core part of the Bandgap Reference (BGR) that performs the **addition of CTAT and PTAT voltages** to produce the final **constant reference voltage**.

This branch typically consists of a **mirror transistor** and a **BJT configured as a diode**.  
The mirror transistor ensures that the **same current** flowing through the current mirror branches also flows through the reference branch, maintaining bias symmetry across the circuit.

####  Working Principle

From the **PTAT generation circuit**, we obtain a **PTAT voltage** and a **PTAT current**.  
This PTAT current is mirrored into the reference branch, where it flows through a **resistor** connected in series with the **CTAT diode**.

However, the **slope of the PTAT voltage** is much smaller compared to that of the **CTAT voltage**.  
To balance these effects and achieve temperature independence, the **resistance value is increased** — since the current is constant, a higher resistance increases the voltage drop proportionally.
As a result, the total output voltage across the resistor becomes the **sum of the PTAT and CTAT components**, yielding a **temperature-stable reference voltage**.

---

###  Start-up Circuit

The **Start-up Circuit** is an essential part of the Bandgap Reference (BGR) design that ensures the **self-biased current mirror** starts operating correctly from power-up.

####  Function

In self-biased current mirrors, there exists a **degenerative bias point** where the circuit can settle into an **unwanted zero-current state**.  
Without intervention, the mirror could remain in this state indefinitely, preventing the circuit from reaching its intended operating condition.

To avoid this, a **start-up circuit** is introduced.  
This circuit **forces a small initial current** into the self-biased current mirror when it detects that the mirror current is zero.  
This small perturbation shifts the mirror out of the zero-current equilibrium point.

Once the circuit begins to conduct, the **self-biasing mechanism** of the current mirror takes over and automatically stabilizes the current to its desired operating value.



###  Complete BGR Circuit

By combining all the previously discussed building blocks, we can construct the **Complete Bandgap Reference (BGR) Circuit**.

<img width="804" height="525" alt="Screenshot 2025-11-15 155851" src="https://github.com/user-attachments/assets/4e1ab1fa-db51-4b8e-a94d-881461b2df23" />

---

####  Circuit Composition

The complete BGR circuit integrates the following components:

- **CTAT Voltage Generation Circuit** — provides a voltage that decreases with temperature using a BJT diode.  
- **PTAT Voltage Generation Circuit** — produces a voltage that increases with temperature using resistors and matched BJTs.  
- **Self-Biased Current Mirror Circuit** — establishes and maintains stable current levels without the need for external biasing.  
- **Reference Branch Circuit** — sums the CTAT and PTAT components to generate the temperature-independent reference voltage.  
- **Start-up Circuit** — ensures the self-biased current mirror starts correctly by eliminating the zero-current operating point.

---

####  Working Principle

- The **CTAT** and **PTAT** voltages are carefully scaled and summed to achieve a **temperature-stable output voltage**.  
- The **current mirror** maintains proper biasing across all branches.  
- The **start-up circuit** guarantees reliable operation from power-up.  

Together, these components form a **fully functional Bandgap Reference circuit**, producing a **constant output voltage (~1.2 V)** that remains stable over variations in **temperature, supply voltage, and load conditions**.



####  Advantages of SBCM BGR

- **Simplest Topology:** The circuit structure is straightforward, making it easy to implement.  
- **Ease of Design:** Requires fewer components and has a simpler biasing mechanism compared to op-amp-based BGRs.  
- **Always Stable:** The self-biasing mechanism ensures a stable operating point once the circuit starts.  

####  Limitations of SBCM BGR

- **Low Power Supply Rejection Ratio (PSRR):** More sensitive to supply voltage fluctuations.  
- **Cascode Design Required:** A cascode structure may be added to improve PSRR performance.  
- **Voltage Headroom Issue:** Limited voltage swing can affect proper operation in low-voltage designs.  
- **Requires Start-up Circuit:** Essential to prevent the circuit from remaining in the zero-current state.

## 2. Design and Pre-layout Simulation

For the practical implementation of the Bandgap Reference (BGR) circuit, the **SkyWater SKY130 (130 nm)** PDK is used.  
Before designing the complete circuit, we must first define the **design requirements** that our circuit should meet.

---

###  Design Requirements

| Parameter | Specification |
|------------|----------------|
| Supply Voltage (VDD) | 1.8 V |
| Temperature Range | -40°C to 125°C |
| Power Consumption | < 60 µW |
| Off Current | < 2 µA |
| Start-up Time | < 2 µs |
| Temperature Coefficient (Tempco) of Vref | < 50 ppm/°C |

---

###  Device Data Sheet

#### 1. MOSFET

| Parameter | NFET | PFET |
|------------|-------|-------|
| Type | LVT | LVT |
| Voltage Rating | 1.8 V | 1.8 V |
| Threshold Voltage (Vt0) | ~0.4 V | ~-0.6 V |
| Model | sky130_fd_pr__nfet_01v8_lvt | sky130_fd_pr__pfet_01v8_lvt |

---

####  Bipolar Junction Transistor (PNP)

| Parameter | PNP |
|------------|------|
| Current Rating | 1 µA – 10 µA/µm² |
| Beta (β) | ~12 |
| Area | 11.56 µm² |
| Model | sky130_fd_pr__pnp_05v5_W3p40L3p40 |

---

####  Resistor (RPOLYH)

| Parameter | RPOLYH |
|------------|----------|
| Sheet Resistance | ~350 Ω/sq |
| Tempco | 2.5 Ω/°C |
| Available Widths | 0.35 µm, 0.69 µm, 1.41 µm, 5.37 µm |
| Model | sky130_fd_pr__res_high_po |

---

###  Circuit Design

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
###  Final Circuit

<img width="827" height="517" alt="Screenshot 2025-11-15 160048" src="https://github.com/user-attachments/assets/80f99e5a-b1f8-4c43-b219-257184b7d6b5" />

##   CTAT Simulation  
### CTAT Voltage Generation with Single BJT Netlist

####  Theory  
- A **BJT used as a diode** (by shorting its base and collector) produces a voltage that **decreases linearly with temperature**.  
- When a **constant current source (10 µA)** flows through the BJT, the **base-emitter voltage (V_BE)** shows a **negative temperature coefficient** (typically −2 mV/°C).  
- This negative slope of **V_BE vs. Temperature** represents the **CTAT characteristic**.

####  Circuit Setup  
- **Device Used:** `sky130_fd_pr__pnp_05v5_w3p40l3p40`  
- **Bias Current:** 10 µA (constant current source)  
- **Output Measured:** Voltage across BJT (V_BE)   

####  Expected Output  
A **straight line with a negative slope** in the **V_BE vs. Temperature** plot:  
> As temperature increases → V_BE decreases → CTAT behavior confirmed ✅  



After simulation we can get a wavefrom like below, and from the wavefrom we can see the CTAT behaviour of the BJT, and can find the slope.

<img width="911" height="659" alt="ctat crt result.png
" src="https://github.com/bharathmr1012/Bharath_Bandgap_workshop/blob/386a181c6a5718a9c053b2ac9cefe59d88d2b788/pics/results/ctat%20crt%20result.png" />



### CTAT Voltage generation with Multiple BJT netlist

In this simulation we will check the CTAT voltage across the 8 parallel connected BJTs


<img width="1171" height="761" alt="ctar crt result with multipel bjt" src="https://github.com/bharathmr1012/Bharath_Bandgap_workshop/blob/386a181c6a5718a9c053b2ac9cefe59d88d2b788/pics/results/ctar%20crt%20result%20with%20multipel%20bjt.png" />

we can see the slope is increasing in case of multiple BJTs.

### CTAT Voltage generation with different current source values netlist
<img width="945" height="737" alt="ctat with variabel current " src="https://github.com/bharathmr1012/Bharath_Bandgap_workshop/blob/efa588ab52ad45d6e491f80d98ba0fd075d69305/pics/results/ctat%20with%20variabel%20current%20.png" />

###   PTAT Simulation

#### PTAT Voltage generation with VCVS
<img width="1668" height="801" alt="PTAT crt" src="https://github.com/bharathmr1012/Bharath_Bandgap_workshop/blob/18d6a615ceb38911106dda71ad8d2fff0e34aafe/pics/results/ptat1.png" />

<img width="1668" height="801" alt="PTAT crt2" src="https://github.com/bharathmr1012/Bharath_Bandgap_workshop/blob/18d6a615ceb38911106dda71ad8d2fff0e34aafe/pics/results/ptat2.png" />



###  BGR using Ideal OpAmp

Now after simulating all our components, let's quick check our BGR behaviour using one **VCVS** as an **ideal OpAmp**.

In this simulation, we should get the reference voltage as an **umbrella-shaped curve** and it should be approximately **1.2V**.
```spice
*** Full BGR crt using ideal opamp (vcvs) *****

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
<img width="849" height="729" alt="BGR_Ideal1" src="https://github.com/bharathmr1012/Bharath_Bandgap_workshop/blob/8262bed31f23bacdc71e3e677131178a26020b28/pics/results/BGR_Ideal1.png" />
<img width="926" height="486" alt="BGR_Ideal2" src="https://github.com/bharathmr1012/Bharath_Bandgap_workshop/blob/8262bed31f23bacdc71e3e677131178a26020b28/pics/results/BGR_Ideal2.png" />

###  BGR with selfbias current mirror
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
<img width="1530" height="698" alt="bgr tt1" src="https://github.com/bharathmr1012/Bharath_Bandgap_workshop/blob/fc9133678516c799e57a4e3378b48a2d54655896/pics/results/bgr%20tt1.png" />

<img width="1530" height="698" alt="bgr tt2" src="https://github.com/bharathmr1012/Bharath_Bandgap_workshop/blob/fc9133678516c799e57a4e3378b48a2d54655896/pics/results/bgr%20tt2.png" />

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
<img width="799" height="698" alt="bgr ff1" src="https://github.com/bharathmr1012/Bharath_Bandgap_workshop/blob/ddd5f99614d7c82901366941ffb2a49987ace1eb/pics/results/bgr%20ff1.png" />

<img width="799" height="698" alt="bgr ff2" src="https://github.com/bharathmr1012/Bharath_Bandgap_workshop/blob/ddd5f99614d7c82901366941ffb2a49987ace1eb/pics/results/bgr%20ff2.png" />

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

## 4. Layout Design

Now after getting our final **netlist**, we have to design the **layout** for our **Bandgap Reference (BGR)** circuit.  
Layout is the graphical representation of the physical masks used in **IC fabrication**.  
We are going to use the **Magic VLSI tool** for our layout design.

---




##  Blocks Design

###  Design of NFETs

We have designed the **NFET layout** by placing all the transistors in a single well-defined region to ensure proper matching and compactness.  
The layout is carefully optimized to maintain **symmetry**, **matching accuracy**, and **noise immunity**.

####  Design Details:
1. **Common Centroid Matching:**  
   - All NFETs are placed following the **common centroid** technique to minimize mismatch due to process gradients.  
   - This helps in achieving balanced electrical characteristics across devices.

2. **Dummy Devices:**  
   - **Dummy transistors** are added at the edges of the active device array.  
   - These prevent **diffusion edge effects** and ensure that active devices experience uniform process conditions.

3. **Guard Ring:**  
   - A **p+ guard ring** is added around the NFET array to protect the layout from substrate noise and improve device isolation.  
   - It also helps in reducing latch-up susceptibility.

4. **Diffusion Continuity:**  
   - The layout ensures **no diffusion breaks**, which helps maintain low series resistance and better current matching.

5. **Layout Orientation:**  
   - All gates are oriented in the **same direction** for consistent channel stress.  
   - Source and drain regions are shared between adjacent devices to reduce area and parasitics.

``` bash
@chandranshu24-hue ➜ /workspaces/vsd-bandgap/bandgap/layout (main) $ magic -T sky130A.tech  nfets.mag
```
<img width="1217" height="740" alt="NFETS" src="https://github.com/bharathmr1012/Bharath_Bandgap_workshop/blob/7343661f156ccdb7f8250aa00120981e5913610a/pics/results/nfets.png" />

###  Design of PFETs

We have designed the **PFET layout block** by grouping all PFETs together in a symmetric and well-matched arrangement.  
The layout emphasizes **device matching**, **isolation**, and **robustness against noise** to ensure accurate current mirroring and stable biasing.

####  Design Details:
1. **Matching Arrangement:**  
   - All PFETs are placed in a **symmetrical array** using a **common centroid pattern** to reduce gradient-induced mismatches.  
   - Ensures that variations in oxide thickness or dopant concentration are evenly distributed across all devices.

2. **Dummy Transistors:**  
   - **Dummy PFETs** are added at the edges of the active device array.  
   - These dummies improve **process uniformity** and minimize edge effects in the active transistors.

3. **Guard Ring Implementation:**  
   - A **n+ guard ring** is placed around the PFET block to protect it from substrate noise and potential coupling from nearby circuits.  
   - It provides better **isolation** and enhances overall layout reliability.

4. **Shared Source/Drain Regions:**  
   - Adjacent transistors share common diffusion areas to minimize parasitic resistance and save layout area.

5. **Orientation Consistency:**  
   - All gates are aligned in the same direction for uniform channel characteristics and ease of routing.

<img width="1217" height="740" alt="pfets" src="https://github.com/bharathmr1012/Bharath_Bandgap_workshop/blob/7343661f156ccdb7f8250aa00120981e5913610a/pics/results/pfets.png" />

###  Design of RESBANK
<img width="1182" height="752" alt="resbank1" src="https://github.com/bharathmr1012/Bharath_Bandgap_workshop/blob/7343661f156ccdb7f8250aa00120981e5913610a/pics/results/resbank1.png" />
<img width="1182" height="752" alt="resbankk2" src="https://github.com/bharathmr1012/Bharath_Bandgap_workshop/blob/7343661f156ccdb7f8250aa00120981e5913610a/pics/results/resbank2.png" />

###  Design of PNP10
We have created the layout by putting all the PNPs together, with appropriate matching, and used dummies to enhance noise performance.
<img width="1196" height="690" alt="pnp10" src="https://github.com/bharathmr1012/Bharath_Bandgap_workshop/blob/0472ad8b2a3a13e316923d9f10820049e1a99410/pics/results/pnp10.png" />

###  Design of STARTERNFET
We placed the the two w=1, l=7 NFETs together with a guardring to desingn the STATRTERNFET.
<img width="962" height="315" alt="starternfet" src="https://github.com/bharathmr1012/Bharath_Bandgap_workshop/blob/0472ad8b2a3a13e316923d9f10820049e1a99410/pics/results/starnet.png" />

###  TOP LEVEL DESIGN
The **top-level layout** integrates all the sub-blocks of the Bandgap Reference (BGR) circuit — including NFETs, PFETs, resistor bank, BJT, and the startup circuit — into a single unified layout using **Magic VLSI**.

####  Layout Overview:
The layout is organized for optimal matching, symmetry, and noise immunity. Each functional block is carefully placed to minimize parasitic effects and ensure consistent temperature behavior.

####  Key Components:
1. **NFET Block (Bottom Section):**
   - Contains all NMOS transistors arranged in a **common centroid** configuration.
   - Includes dummy devices and a guard ring for isolation and matching.
   - Used primarily in the current mirror and startup circuit.

2. **PFET Block (Middle Section):**
   - PFETs are placed symmetrically to ensure equal current distribution.
   - Guard ring provided to suppress substrate coupling noise.
   - Used in the mirror and biasing circuits.

3. **Resistor Bank (Top Section):**
   - Houses all resistors in a matched array configuration.
   - Edge dummies used to avoid process variations.
   - Provides PTAT and CTAT voltage scaling.

4. **BJT Block:**
   - Diode-connected BJT for CTAT voltage generation.
   - Placed close to the resistor bank to ensure uniform temperature tracking.

5. **Startup Circuit (Center):**
   - Labeled as **starternfet** in the layout.
   - Ensures proper startup of the self-biased current mirror.
   - Connected to NFET region for bias initialization.

6. **Guard Rings and Isolation:**
   - Full perimeter **p+ guard ring** implemented for substrate noise isolation.
   - Ensures reliable and low-noise reference operation.

####  Layout Details:
| Parameter | Value / Description |
|------------|---------------------|
| Tool Used | Magic VLSI |
| Technology | SkyWater SKY130 |
| DRC Status | Clean (No Design Rule Errors) |
| File Name | `bgr_top.mag` |
| Layout Dimensions | ~85 µm × 73 µm |

####  Layout Visualization:
The image below shows the **complete top-level BGR layout**, where all components are interconnected and verified for DRC cleanliness.
<img width="1163" height="774" alt="top" src="https://github.com/bharathmr1012/Bharath_Bandgap_workshop/blob/fb8a5d9c6791a77ece683f55d6c34babbd3e9fd2/pics/results/top.png" />
 *This top-level layout ensures electrical symmetry, thermal stability, and process tolerance for a robust and accurate Bandgap Reference circuit.*





