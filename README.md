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

---

## 📌 Key Performance Requirements

A practical BGR must exhibit:

- **High PSRR:** 40–60 dB (good supply rejection)  
- **Low Temperature Coefficient:** 10–50 ppm/°C  
- **Robust behavior across PVT corners**  
- **Stable bias currents** for driving analog and mixed-signal blocks  

---

## 🎯 Project Summary

Thus, the Bandgap Reference serves as the backbone of reliable precision systems.  
This project implements and characterizes a complete BGR using the **SKY130 open-source PDK**, covering:

- Schematic design  
- SPICE simulation in ngspice  
- Temperature analysis  
- Line and load regulation  
- Layout design using Magic  


A Bandgap Reference (BGR) is a foundational analog block that provides a stable DC reference voltage with minimal sensitivity to temperature, supply (line) variation, and process spread. Such a stable reference is critical for mixed-signal systems—powering ADCs, DACs, LDO/voltage regulators, and bias networks—where accuracy and repeatability matter.
The concept of the Bandgap Reference is based on combining two temperature-dependent voltages:

1. **CTAT (Complementary to Absolute Temperature) Voltage:**  
   - Typically derived from the **base-emitter voltage (V<sub>BE</sub>)** of a bipolar transistor.  
   - V<sub>BE</sub> decreases with increasing temperature (negative temperature coefficient).

2. **PTAT (Proportional to Absolute Temperature) Voltage:**  
   - Generated from the **difference in base-emitter voltages (ΔV<sub>BE</sub>)** between two transistors operating at different current densities.  
   - ΔV<sub>BE</sub> increases with temperature (positive temperature coefficient).

By carefully scaling and summing these two voltages, the opposing temperature effects cancel out, resulting in a **constant output voltage**—typically around **1.2 V**, which corresponds to the bandgap voltage of silicon at 0 K.
V<sub>ref</sub>(T) ≈ V<sub>BE</sub>(T) + k · ΔV<sub>BE</sub>(T)
