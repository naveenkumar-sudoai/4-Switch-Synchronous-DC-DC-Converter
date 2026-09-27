# 4-Switch Synchronous Buck-Boost DC-DC Converter

A high-power, high-efficiency **four-switch synchronous buck-boost** DC-DC converter capable of stepping a DC input up or down to the desired output level with synchronous rectification on all four switches.

## Key Specifications

| Parameter        | Value            |
| ---------------- | ---------------- |
| Input Voltage    | 24 V – 48 V      |
| Output Voltage   | 12 V – 48 V      |
| Output Current   | 30 A             |
| Topology         | 4-Switch Buck-Boost (H-bridge) |
| Rectification    | Synchronous (all switches) |

The four-switch (H-bridge) buck-boost topology allows the converter to operate in three modes — **buck**, **boost**, and **buck-boost** (transition) — enabling the output to be either above or below the input while maintaining high efficiency across the entire range.

## Project Structure

```
.
├── Documents_notes/          # Datasheets, notes, design calculations, references
├── Design/
│   └── kicad/                # KiCad schematic + PCB layout
├── Simulation/
│   ├── Ltspice/              # SPICE simulation of power stage & control loop
│   └── matlab/               # Control design, transfer functions, waveforms
├── LICENSE
└── README.md
```

## Getting Started

- **Hardware design** lives in `Design/kicad/`.
- **Circuit-level simulation** (switching waveforms, efficiency, losses) lives in `Simulation/Ltspice/`.
- **Control-system modeling** (compensator design, bode plots, stability) lives in `Simulation/matlab/`.
- **Documentation & notes** live in `Documents_notes/`.

## License

See [LICENSE](LICENSE).
