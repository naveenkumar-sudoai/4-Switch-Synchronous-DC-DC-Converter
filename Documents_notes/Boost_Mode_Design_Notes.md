# 4-Switch Buck-Boost — Boost-Mode Design Notes

## Objective
Verify the power stage of a **4-switch synchronous buck-boost** converter in **boost mode**
(step-up), before committing to a full controller.

| Parameter        | Value                          |
| ---------------- | ------------------------------ |
| Input voltage    | 9 V (nominal test; wide 9–24 V)|
| Output voltage   | 12 V                           |
| Output current   | 5 A (`Rload = 2.4 Ω`)          |
| Switching freq.  | 300 kHz (`T = 3.3333 µs`)      |
| Inductor         | 22 µH, DCR 10 mΩ               |
| Cin              | 38 µF, ESR 2 mΩ                |
| Cout             | 60.7 µF, ESR 5 mΩ              |
| Rsense           | 10 mΩ (in series with inductor)|

## Topology (H-bridge)
```
        Q1 (buck HS)          Q3 (boost HS)
   IN ──┤/├──── SW1 ──[L]── SW2 ──┤/├── OUT
           │                    │
        Q2 (buck LS)          Q4 (boost LS)
           │                    │
          GND                  GND
```
- **Q1** buck high-side — connects `IN` ↔ `SW1`
- **Q2** buck low-side — connects `SW1` ↔ `GND`
- **Q3** boost high-side — connects `SW2` ↔ `OUT` (synchronous rectifier)
- **Q4** boost low-side — connects `SW2` ↔ `GND` (active switch)

## Boost-mode switch states (the simulated test)
| Switch | State            | Drive        |
| ------ | ---------------- | ------------ |
| Q1     | always ON        | 10 V DC      |
| Q2     | always OFF       | 0 V          |
| Q4     | PWM (active)     | `D = 0.25`   |
| Q3     | complementary    | `1-D`, 50 ns dead time |

## Duty cycle
Boost gain: `Vout/Vin = 1/(1-D)`  →  `D = 1 − Vin/Vout`.

For **9 V → 12 V**: `D = 1 − 9/12 = 0.25`, so `Ton = 0.25 × 3.3333 µs = 833.3 ns`.

> ⚠️ A duty of **0.325** (Ton = 1.08 µs) would produce `9 / (1 − 0.325) ≈ 13.3 V`,
> not 12 V. Use `D = 0.25` (or ~0.26 to compensate real conduction losses).

## Dead time
Q3 (sync rectifier) must never overlap Q4 (active switch) — simultaneous conduction
shorts `OUT` to `GND`. In the simulation Q4 is ON `[0, 833.3 ns]`, Q3 is ON
`[883.3 ns, 3.2833 µs]`, giving **50 ns** of dead time on both edges.

## Inductor current ripple (CCM check)
`ΔIL = Vin·D·T / L = 9 × 0.25 × 3.3333 µs / 22 µH ≈ 0.34 A`
Average input current `Iin = Iout/(1−D) = 6.67 A` — deep continuous-conduction mode.

## Files
- `Simulation/Ltspice/BuckBoost_4Switch.asc` — LTspice schematic (open & Run)
- `Simulation/Ltspice/BuckBoost_4Switch_boost.cir` — SPICE netlist (same circuit)

## How to run
- **LTspice (Windows/Wine):** open the `.asc`, press Run. Probe `V(OUT)`, `V(SW2)`, `I(L1)`.
- **ngspice:** `ngspice BuckBoost_4Switch_boost.cir` then `plot v(out) v(sw2)`.

## Notes on the ideal-switch model
Switches use the LTspice `SW` voltage-controlled switch:
`.model SwitchModel SW(Ron=1m Roff=10Meg Vt=2.5)`.
Gate logic (Vt = 2.5 V): a 10 V gate = ON, 0 V gate = OFF. For a real design, replace
`SW` with MOSFET models (with `Cgs`/`Rg`) and add true gate-driver dead time.
