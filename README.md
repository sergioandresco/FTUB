# Hydrogen Gas Turbine Monitor (digital twin)

Static single-page dashboard that simulates monitoring a 50 MW, 3-stage hydrogen gas turbine
(Turbomáquinas course project). All data is simulated from the project's design values.

No build step: the site is just `index.html`.

## Deploy on Vercel
1. Push this folder to a GitHub repository.
2. In Vercel: Add New > Project > import the repository.
3. Framework Preset: **Other**. Leave build command and output directory empty. Deploy.

---

# Project context: Hydrogen gas turbine (Turbomáquinas "proyecto-misión")

Escuela Colombiana de Ingeniería Julio Garavito. Team: Dani and Juan Manuel Maldonado.

## 1. Mission
Reduce emissions of thermal generation that backs up Colombian hydroelectric plants (about 70 % of the grid is hydro; in dry years such as El Niño, thermal plants run on gas or coal). The proposal is a **50 MW, 3-stage axial gas turbine fueled by hydrogen** that burns without CO2 and acts as dispatchable backup (dispatched through XM / SIN, framed by Ley 2099/2021).

## 2. What was done in each delivery
- 1st/2nd delivery: IEEE report (.docx), calculation report (.docx), velocity-triangle workbook (.xlsx), slides structure and a two-speaker script, CAD model (GrabCAD SLDPRT opened in Autodesk Inventor).
- 3rd (final) delivery:
  1. Add the hydrogen pipes (tank -> injectors) to the CAD. **Pending: done in Inventor.**
  2. Monitoring web page (this repo): a **digital twin**, i.e. a simulation, not real telemetry.

## 3. Design data (source of every nominal value)
| Quantity | Value |
|---|---|
| Power | 50 MW |
| Speed N | 9000 rpm |
| Mean diameter Dm | 1.00 m (U = 471.2 m/s) |
| Turbine inlet T01 / P01 | 1400 K / 10 bar |
| Exhaust pressure P02 | 1.2 bar |
| Mass flow | 84.35 kg/s (Q = 33.9 m3/s at inlet) |
| Total enthalpy drop | 592.8 kJ/kg (197.6 per stage) |
| Efficiency, gamma, cp, R | 0.88, 1.33, 1150 J/kg K, 287 J/kg K |
| Degree of reaction | 0.5 (mirror triangles) |
| Blade heights | 35 / 61 / 106 mm |
| Angles | alpha1 40.1, alpha2 2.8, beta1 -2.8, beta2 -40.1 deg |
| Mach (stator exit / relative) | 1.02 / 0.78 |

Stage outlets: stage 1 = 4.93 bar, 1201.9 K; stage 2 = 2.43 bar, 1031.8 K; stage 3 = 1.20 bar, 885.8 K.
Combustion chamber (assumed): T = 600 K, Vref = 20 m/s, L/D = 2, injector pitch 18 mm, 3 rings -> ~500 injectors, Dref 0.96 m, L 1.92 m.
H2 line: shutoff valve -> regulator (12-15 bar) -> flow control valve -> annular manifold -> injectors, with N2 purge and flame arrestors. Materials: SS 316/316L lines, Inconel injector tips, metal/PTFE seals. H2 stored as compressed gas.

## 4. How the dashboard data is generated
The page runs a small simulation in the browser (JavaScript, 1 step = 1 s of simulated time, speed 1x/3x/8x).

**Turbine expansion (same chain as the calculation report).** For a given T01 and P01, the pressure ratio is split equally over 3 stages: PR = (P01/P02)^(1/3). Per stage: T_out_isentropic = T * (P_out/P)^((gamma-1)/gamma); T_out = T - eta (T - T_out_isentropic). Electrical power = mdot * cp * (T01 - T_exit). At nominal inputs this returns 49.9 MW and stage outlets equal to the design values above.

**Axial velocity per stage.** Ca = mdot / (rho * A), with rho = P/(R T) and a flow area A per stage (0.114, 0.189, 0.341 m2). These areas are **derived** from the design Ca, rho and mdot, not independent data. Result: 517 / 543 / 524 m/s, matching the report.

**Part load (assumption).** Mass flow falls to 70 % at the lowest load: mdot = 84.35 * (0.7 + 0.3 L). T01 is found by bisection so that power = 49.9 MW * L, and P01 scales as mdot * sqrt(T01/1400) (choked-flow behavior of the first nozzle). Load range 40-100 %.

**Hydrogen flow.** Energy balance in the chamber: mh2 = mdot * cp * (T01 - T_compressor_exit) / LHV_H2, with LHV = 120 MJ/kg and compressor exit temperature 300 + 300 (P01/10)^0.25 K (a simple stand-in, not a compressor model). At nominal this gives about 0.65 kg/s. The tank holds 20 t (an example number) and drains at that rate.

**Other signals.** NOx = 9 ppm * exp((T01 - 1400)/55) (rises steeply with flame temperature, the Micromix risk). Vibration = 1.8 mm/s * (rpm/9000)^2 plus fault contribution. H2 in air (% LEL) rises about 2.5 per second when a leak is injected. Regulator and manifold pressures follow the line state. Random noise is added to every signal.

**CO2 avoided.** Heat released by H2 is converted to the methane that would give the same heat (LHV 50 MJ/kg) times 2.75 kg CO2 per kg CH4. This is a comparison against natural gas, not a measured emission.

**Operating states.** PARADA -> PURGA (8 s N2 purge) -> ENCENDIDO (valve opens, flame detected) -> ACELERACION (to 9000 rpm) -> SINCRONIZACION -> CARGA (ramp) -> OPERACION. Any critical alarm or the emergency button -> DISPARO (fuel cut, N2 purge, spin down) -> PARADA. Start is blocked while faults are active or LEL > 10 %.

**Alarms (example thresholds, not from a standard).** T01 > 1430 K warn, > 1470 K trip. Vibration > 4.5 warn, > 7.1 mm/s trip. H2 ambient > 10 % LEL warn, > 25 % trip. NOx > 25 ppm warn. Manifold-to-P01 margin < 1.2 bar warn.

## 5. Limits to state in the presentation
- Everything is simulated; no real turbine or sensors exist.
- Part-load behavior, alarm limits, tank size and compressor temperature are assumptions.
- It is a steady, mean-line model: no real transients, blade-row losses or combustion kinetics.
- Next real step, if built: replace the `step()` function with readings from a data source (PLC/OPC UA, MQTT or a REST API) and keep the same UI.

## 6. Files in this repo
- `index.html`: the full dashboard (HTML + CSS + JS, no dependencies).
- `assets/turbine-cad.jpg`: cutaway view of the Inventor CAD model, shown on the dashboard.
- `README.md`: this file (deployment steps and project context).
- `CLAUDE.md`: notes for Claude Code (not used by the site).

## 7. Suggested way to present it (1-2 minutes)
1. Show the nominal state and say the numbers come from the calculation report.
2. Start from PARADA and run the start sequence (use 8x).
3. Inject "Fuga de H2": LEL rises, trip at 25 %, N2 purge.
4. Inject "Sobretemperatura": NOx alarm, then trip at 1470 K, linking to the Micromix NOx risk.
5. Point at the CO2-avoided counter as the mission result.

## 8. Still to do
- Add the H2 pipes to the CAD (tank -> regulator -> valve -> manifold -> injectors), using SS 316L and a flame arrestor before the manifold.
- Later: CAD cuts and labels, update report and slides for the 3rd delivery, deploy the site on Vercel.
