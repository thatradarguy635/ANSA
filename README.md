# ANSA — Adaptive Null-Steering Antenna System

ANSA is a hardware-and-software research project exploring adaptive interference suppression using antenna-array signal combining and controllable RF phase/amplitude relationships. The current prototype work includes antenna hardware, passive RF networks, a DC-bias/phase-control board, and a controller/power-distribution board.

> **Project status:** Prototype hardware assembled; characterization, calibration, and end-to-end adaptive null-steering validation are ongoing. Performance claims should be based on documented VNA and over-the-air measurements rather than component appearance alone.

## Project overview

The intended system workflow is:

1. **Receive RF signals** with the antenna element or array.
2. **Condition and combine RF paths** using passive RF networks.
3. **Adjust controllable RF parameters** through a bias/control circuit.
4. **Measure the received output** with suitable RF test equipment or an SDR.
5. **Run an adaptive algorithm** to update control settings and reduce the measured interference while preserving the desired signal as far as the available hardware allows.

The exact control law, receiver interface, operating band, and final system-level performance should be specified in the project configuration and test reports once they are verified.

## Hardware and prototype photographs

### 1. Antenna element

![ANSA antenna element](assets/antenna.jpg)

The photographed board contains a rectangular conductive patch and a feed line terminating in an SMA connector. The patch is the radiating/receiving element; its dimensions, substrate properties, feed geometry, and ground-plane arrangement determine its resonant frequency, impedance match, bandwidth, and radiation pattern.

**Role in ANSA**
- Receives incident RF energy and converts it to an electrical RF signal.
- Provides the antenna element response used by the downstream RF and signal-processing chain.
- Must be characterized for resonant frequency, return loss (`S11`), impedance bandwidth, gain, and radiation pattern before array-level results can be interpreted.

**Recommended verification**
- Measure `S11` using a calibrated VNA.
- Record the substrate material, thickness, patch dimensions, connector launch geometry, and target frequency.
- If this is intended to operate as one element in an array, document element spacing and mutual-coupling measurements.

### 2. RF power combiner

![ANSA RF power combiner](assets/power-combiner.jpg)

The board has four SMA ports and a printed multi-branch transmission-line network. The visible layout appears to combine two pairs of RF paths and then combine those intermediate paths at a common port. The cross-branch resistors appear to provide isolation or resistive combining behaviour; the precise topology and design values should be confirmed against the schematic.

**Role in ANSA**
- Combines RF signals from multiple paths into a single output.
- Enables a single receiver/measurement chain to observe the combined result.
- The relative phase and amplitude of the input signals determine whether they add constructively or partially cancel at the output.

**Important engineering note**
A passive combiner does not itself perform adaptive null steering. It provides the RF combining network; adaptive behaviour requires controllable path weights (for example, phase and/or amplitude adjustment) and a feedback algorithm.

**Recommended verification**
- Identify and label the common/output port and each input port.
- Measure insertion loss, input return loss, isolation between input ports, and amplitude/phase balance across the operating band.
- Compare measured S-parameters with the simulated model. Do not label this as a Wilkinson combiner unless the schematic and impedance/resistor design confirm that topology.

### 3. 90-degree hybrid branch-line coupler

![ANSA four-port hybrid coupler](assets/90-degree-hybrid-coupler.jpg)

The board is a four-port printed transmission-line network with SMA connectors at the four corners, consistent in appearance with a branch-line hybrid coupler. A 90-degree hybrid ideally divides an input into two outputs with approximately equal amplitudes and a phase difference of 90 degrees, while isolating the fourth port under its design conditions.

**Role in ANSA**
- Produces two RF output signals with a defined relative phase relationship.
- Can support quadrature signal generation, RF comparison, or phase-sensitive network experiments.
- Provides a useful building block for testing RF phase relationships in the ANSA prototype.

**Preliminary measurement note**
The project notes report `S11` of approximately `−29 dB` and a measured phase difference of `84.6°` between `S21` and `S31`. These are preliminary measured values and should be accompanied in the final report by the measurement frequency, calibration method, port numbering, magnitude balance, and uncertainty. A phase difference of `84.6°` is 5.4° away from the ideal 90° value.

**Recommended verification**
- Document the port numbering and define which port is the input, through, coupled, and isolated port.
- Plot all relevant S-parameter magnitudes and the phase difference over frequency.
- Verify output amplitude balance and isolation, not only input matching and phase difference.

### 4. DC-bias / phase-control PCB

![ANSA DC-bias and phase-control PCB](assets/dc-bias-phase-control-pcb.jpg)

The photograph shows a custom PCB with two SMA RF ports, a small breakout board labelled `MCP4725`, and a discrete RF/control circuit. The MCP4725 is a digital-to-analogue converter (DAC) breakout; it can generate a programmable analogue voltage from digital commands over I²C. The remaining discrete circuit appears to interface the bias/control voltage with the RF path, but its exact function should be confirmed from the schematic and component values.

**Role in ANSA**
- Provides a hardware path for experimenting with voltage-controlled RF behaviour.
- May supply the bias voltage for a varactor or another tunable RF component if that is what the final schematic specifies.
- Bridges digital control commands and analogue bias settings, enabling a software controller to sweep or update a hardware setting.

**Important engineering note**
The photo alone does not establish the achievable phase-shift range, tuning linearity, insertion loss, or whether the circuit is a complete phase shifter. These properties must be measured or simulated. Ensure the DAC output range, RF/DC isolation, bias filtering, component voltage limits, and grounding are appropriate before connecting it to the RF chain.

**Recommended verification**
- Publish a schematic and component-value table.
- Record the DAC supply/reference voltage, output range, I²C address, and control interface.
- Measure RF transmission phase and insertion loss versus DAC voltage at the intended operating frequency.
- Check that DC bias cannot reach the VNA, SDR, or other RF instrument input.

### 5. Controller and power-distribution board

![ANSA controller and power-distribution board](assets/control-power-distribution-board.jpg)

This assembly appears to integrate an ESP32 development board, a battery holder, a power switch, a DC–DC converter module, connectors, and wiring for the prototype subsystems. The ESP32 provides a programmable controller and communication interface; the converter and distribution wiring provide the required supply rails to connected modules.

**Role in ANSA**
- Hosts control firmware for setting or updating hardware parameters.
- Can communicate with a host computer over USB/serial or another supported interface.
- Distributes power to the prototype modules and provides a platform for integration testing.

**Recommended verification**
- Publish a power-tree diagram listing each rail, voltage, current capability, and connected load.
- Document the battery chemistry, cell count, protection, charging method, and fuse/current-limit arrangements.
- Identify the exact DC–DC converter and confirm its output voltage and ripple under load.
- Label each connector and record its pinout. Avoid relying on wire colour alone.
- Keep RF grounds, digital grounds, and supply returns arranged according to the validated PCB/system design.

## System integration

The hardware blocks can be represented at a high level as:

```text
Antenna / RF inputs
        |
        v
Passive RF network(s)
(coupler / combiner)
        |
        v
RF measurement or receiver chain
        |
        v
Host-side measurement and adaptive algorithm
        |
        v
Controller (e.g., ESP32) + DAC / bias-control PCB
        |
        +---- updates tunable RF hardware
```

This is a conceptual architecture, not a verified wiring diagram. The final connection order depends on the actual RF topology and on which paths are controllable. A fixed passive coupler or combiner alone cannot create a dynamically steerable null.

## Characterization and validation plan

| Stage | What to measure | Evidence to store |
|---|---|---|
| Antenna | `S11`, resonant frequency, bandwidth, radiation pattern | VNA plots, dimensions, substrate details |
| Hybrid coupler | `S11`–`S44`, insertion loss, isolation, output balance, phase difference | Touchstone `.s4p` file, port map, plots |
| Power combiner | Return loss, insertion loss, isolation, amplitude/phase balance | Touchstone file, port map, plots |
| Bias/phase-control PCB | Phase shift and insertion loss versus control voltage; bias limits | Schematic, sweep data, plots |
| Controller/power board | Supply rails, ripple, current draw, communications, safe startup | Wiring diagram, firmware version, test log |
| Integrated system | Output power/RSSI versus control settings; repeatability; desired-signal preservation | Test procedure, raw measurements, plots, software commit |

### Suggested project data structure

```text
ANSA/
├── README.md
├── assets/
│   ├── antenna.jpg
│   ├── power-combiner.jpg
│   ├── 90-degree-hybrid-coupler.jpg
│   ├── dc-bias-phase-control-pcb.jpg
│   └── control-power-distribution-board.jpg
├── hardware/
│   ├── antenna/
│   ├── coupler/
│   ├── power-combiner/
│   └── bias-control/
├── firmware/
├── software/
├── simulations/
├── measurements/
│   ├── vna/
│   └── system-tests/
└── docs/
```

## Reproducibility

For each hardware revision, keep the following with the repository:

- CAD/source files and exported Gerbers.
- Schematics, PCB revision, bill of materials, and assembly notes.
- CST/HFSS or other simulation projects and their key settings.
- VNA calibration details, measurement frequency range, port mapping, and `.sNp` files.
- Test scripts, firmware version, control settings, and raw measurement data.
- A dated record of any hardware rework, connector changes, or component substitutions.

## Safety and responsible testing

Use conducted tests with suitable attenuation, loads, shielding, or a controlled RF test setup where possible. Do not intentionally interfere with GNSS, cellular, Wi-Fi, public-safety, or other third-party communications. Perform over-the-air tests only in a lawful, authorized environment and within applicable spectrum rules.

## Current limitations

- The photographs document the physical prototype but do not, by themselves, prove the electrical function or measured performance of every circuit.
- The exact operating band and final system topology should be explicitly documented.
- Claims of adaptive null depth, interference suppression, or communication improvement require repeatable system-level measurements against a defined baseline.

## Contributing

As the project develops, use issues for design questions and bugs, branches for experimental changes, and tagged releases for report-ready hardware/firmware revisions. Include measurement evidence when changing RF layouts or control algorithms.
