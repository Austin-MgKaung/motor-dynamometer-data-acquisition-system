# FMC Motor Dynamometer DAQ (fmc-motor-dyno-daq)

**A high-performance, modular mixed-signal Data Acquisition System for 3-phase motor characterization.**

This repository contains the hardware design (schematics/layout), FPGA gateware, and host software for a custom DAQ system based on the **Xilinx Artix-7** (Nexys Video) and the **FMC** (FPGA Mezzanine Card) standard.

---

## 1. Project Overview
The system is designed to measure 3-phase voltages and currents with high bandwidth and strict galvanic isolation. It utilizes a centralized **FMC Carrier** architecture that feeds power to and receives signals from two external **Combo Sensor Boards**.

* **Target Application:** Motor Control Loop verification, Inverter efficiency analysis, Dynamometer instrumentation.
* **Host Platform:** Digilent Nexys Video (Artix-7 XC7A200T).
* **Interface:** FMC Low Pin Count (LPC).

## 2. System Architecture
The system consists of three main hardware PCBs:

### A. FMC ADC Carrier Board (Qty: 1)
The central hub that plugs into the FPGA.
* **ADCs:** 2x **ADS8528** (12-bit, 8-channel, Simultaneous Sampling, Parallel Interface).
* **Power Management:** High-power switching regulators generating low-noise $\pm15V$ and $+5V$ rails distributed to sensor boards.
* **Signal Conditioning:** Input protection (RC filters + Clamps) for all 12 analog channels.

### B. Combo Sensor Boards (Qty: 2)
External boards placed near the motor/inverter to minimize noise pickup. Each board handles one 3-phase set (Voltage + Current).
* **Voltage Channels (3x):**
    * Range: **0–50V** Nominal.
    * Chain: Divider $\rightarrow$ **ISO224** (Galvanic Isolation) $\rightarrow$ **OPA2197** (Diff Amp).
    * Bandwidth: ~100 kHz.
* **Current Channels (3x):**
    * Range: **$\pm$70A**.
    * Sensor: **LA 55-P** (Closed-loop Hall Effect) passing through PCB aperture.
    * Chain: Burden ($90\Omega$) $\rightarrow$ Low-Pass Filter $\rightarrow$ **OPA2197**.
    * Bandwidth: ~2 kHz.

---

## 3. Key Specifications

| Parameter | Specification | Notes |
| :--- | :--- | :--- |
| **Analog Channels** | **12 Total** | 6 Voltage + 6 Current |
| **Sampling Rate** | $\ge$ 200 kS/s per channel | All channels simultaneous |
| **Input Voltage** | 0V to 50V | Isolated ($\pm12V$ ISO224 input range) |
| **Input Current** | $\pm$ 70A | Via LA 55-P Aperture |
| **ADC Resolution** | 12-bit | Bipolar $\pm10V$ Range |
| **Host Interface** | FMC LPC | 3.3V LVCMOS |
| **Power Input** | 12V DC | External Supply (approx 20W max) |

---

## 4. Repository Structure

This repository is organized into three main technology layers:

```text
/
├── hardware/                 # PCB Design Source Files
│   ├── carrier-board/        # Altium/KiCad project for FMC Carrier
│   ├── combo-sensor-board/   # Altium/KiCad project for Sensor Board
│   └── docs/                 # BOMs, Schematics (PDF), and Calculations
│
├── gateware/                 # FPGA Logic (HDL)
│   ├── src/                  # Verilog/VHDL source (ADS8528 Drivers)
│   ├── simul/                # Testbenches and Simulation Waveforms
│   └── constraints/          # XDC Constraints for Nexys Video
│
└── software/                 # Host PC & Embedded Software
    ├── drivers/              # C/C++ Drivers (PCIe/USB)
    └── ui/                   # Python/Qt Dashboard for Data Viz
