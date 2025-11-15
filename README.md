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
 
### 🗝️ Key Features
- Provides a **temperature-stable voltage reference** (~1.2 V)
- **Insensitive** to power supply and process variations
- Can be implemented using **bipolar or CMOS processes**
- Widely used in **precision analog and mixed-signal ICs**

---

### ⚙ Working Principle (Simplified)
At its core, the Bandgap Reference circuit works as follows:
1. Generate a CTAT voltage from a diode-connected BJT (V<sub>BE</sub>).
2. Generate a PTAT voltage using two BJTs with different emitter area ratios.
3. Add the PTAT voltage (positive TC) to the CTAT voltage (negative TC) in proper proportions.
4. The resulting sum is a **temperature-independent reference voltage**.

<img width="698" height="268" alt="Screenshot 2025-10-31 095749" src="https://github.com/user-attachments/assets/29a07377-a79a-483f-ac21-dac1115c8883" />






