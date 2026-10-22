# Household supercomputer cooling: calculation framework

Started 2026-10-21. These are the base calculations for the virtual model, to be done before any hardware is built. Each layer feeds the next. Gil and a friend chose to over-parameterize on purpose: put every variable in, then let sensitivity analysis show which ones matter.

Companion research: `heat-driven-pc-cooling-research-2026-10-21.md`. Property data: CoolProp (`pip install -r requirements.txt`). Existing scripts are in `model/scripts/`.

## Layer 0: the feasibility question

This answers whether the temperature differences can theoretically do the job. It is three inequalities. Write them first, because everything else only refines them.

**A. Passive loop (thermosiphon). No work input.**

    T_die = T_amb + Q · R_total  ≤  T_die,max
    R_total = R_jc + R_TIM + R_base + R_evap + R_vapour + R_cond + R_air

Feasible whenever T_die,max > T_amb. The question is whether R_total is small enough.

**B. Work-driven heat pump** (compressor or Peltier), for going below ambient.

    COP_Carnot = T_c / (T_h − T_c)        (kelvin)
    W = Q_c / COP_real,   COP_real ≈ 0.3–0.6 · COP_Carnot (compressor), ≈0.1–0.2 (Peltier)
    Q_rejected = Q_c + W

**C. Heat-driven chiller** (sorption or absorption, three temperatures). This is the self-powered idea.

    COP_max = (1 − T_a/T_g) · T_c / (T_a − T_c)

Carnot alone says yes once T_g > T_a. The real gate is the **temperature budget**:

    T_g,available = T_die,max − ΔT_stack(die→coolant) − ΔT_approach(generator HX)
    Feasible only if  T_g,available ≥ T_g,min(working pair, T_a, T_c)

Worked example with today's numbers: 65 − 8 − 5 = **52 °C available**, against **56–62 °C required** (best pairs, 25 °C room). It fails by 4–10 K. Every lever in the model either widens the left side or lowers the right side.

## Layer 1: heat sources

- **Chip power.** P = P_dyn + P_leak, with P_dyn = α·C·V²·f.
- **Leakage vs temperature.** P_leak(T) ≈ P_leak,0 · e^{k(T−T_0)}, k ≈ 0.01–0.03 /K. This is an assumption; measure it per chip.
- **Electro-thermal feedback.** T = T_a + R·P(T). It is stable only if R · dP/dT < 1, which is the runaway criterion.
- **Sources table.** For each component (CPU, GPU, VRM, PSU, chipset), record W, max temperature, healthy band and load profile over time.

## Layer 2: conduction stack, die to fluid

- **Slab:** R = t / (k·A). Covers the die, TIM (paste, LM or solder), IHS and copper base.
- **Spreading resistance:** small die on a large base (Lee/Song–Au–Moran correlation).
- **Contact resistance:** set by mounting pressure and flatness.
- **Copper:** k ≈ 390–400 W/m·K. The model varies base thickness and fin geometry.

## Layer 3: evaporator (boiling on the cold plate)

- **Pool-boiling HTC (Cooper):** h = 55 · p_r^0.12 · (−log10 p_r)^−0.55 · M^−0.5 · q″^0.67
- **Flow boiling** (if pumped or in microchannels): Kandlikar or Kim–Mudawar correlations.
- **Critical heat flux (Zuber):** q″_CHF = 0.131 · h_fg · ρ_v^½ · [σ·g·(ρ_l − ρ_v)]^¼. Must exceed the chip's flux, about 30–50 W/cm², with margin. Enhanced surfaces raise it.
- **Fin efficiency** of skived or pin fins: η_f = tanh(mL)/(mL), with m = √(hP/kA_c).

## Layer 4: condenser and heat rejection

- **Film condensation (Nusselt):** h = 0.943 · [ρ_l(ρ_l − ρ_v)·g·h_fg·k_l³ / (μ_l·L·ΔT)]^¼
- **Air or water side, ε-NTU:** for a condensing stream C_r = 0, so ε = 1 − e^−NTU, with NTU = UA / C_min and Q = ε · C_min · (T_sat − T_air,in). This is the method from the Rongchai & Tundee paper.
- **Fan or pump power:** W = Δp · V̇ / η. This feeds the wall-power metric.

## Layer 5: loop hydraulics (what makes it run without a pump)

- **Driving head:** Δp_drive = (ρ_l − ρ_mix) · g · H
- **Must exceed losses:** Δp_friction (two-phase, Friedel or Lockhart–Martinelli) + Δp_acceleration + Δp_minor (bends, fittings)
- **Mass flow:** ṁ = Q / (h_fg · x_out)
- **Operating limits:**
  - entrainment/flooding (Wallis, Kutateladze)
  - dry-out
  - geyser boiling at low load
  - fill ratio: optimum is about 30–40 % for closed tubes and 55–75 % for loops

## Layer 6: working fluid and pressure safety

- **Properties from CoolProp:** T_sat(p), h_fg, ρ_l, ρ_v, μ, k, σ, c_p.
- **Candidates:** R134a (buyable without certification), R1233zd(E), isobutane, water, ethanol.
- **Charge:** m = FR · V_evap · ρ_l
- **Tube burst (Barlow):** p_burst = 2·t·S / D. Size for the worst case, e.g. R134a at 70 °C (21 bar) or a fire, with a safety factor of 3–4 and a relief valve.

## Layer 7: heat pump and chiller cycles (only if going below ambient)

- **Vapour compression, states 1–4:**
  - Q_evap = ṁ(h1 − h4)
  - W = ṁ(h2s − h1) / η_isen
  - COP = Q_evap / W
- **Adsorption (Dubinin–Astakhov):**
  - uptake W = W_0 · exp[−(A/E)^n], with A = R·T·ln(p_sat/p)
  - cycle: Q_des, Q_evap, and COP = Q_evap / Q_des
- **Absorption:** Dühring charts for LiBr–water; this gives T_g,min(T_a, T_c).

## Layer 8: limits from humidity and the chips

- **Dew point (Magnus):** γ = ln(RH/100) + a·T/(b + T), T_d = b·γ / (a − γ), with a = 17.62, b = 243.12 °C. Rule: every cold surface ≥ T_d + 3 K.
- **Healthy bands:** CPU case 35–50 °C, GPU die 45–65 °C. The VRMs and capacitors near the socket need their own airflow.

## Layer 9: payoff, and whether the build is worth it

- **Clocks vs temperature:** boost-bin tables (NVIDIA GPU Boost, AMD PowerTune). Overclock headroom vs T.
- **Lifetime, Arrhenius acceleration factor:** AF = exp[(E_a/k_B)(1/T_1 − 1/T_2)]. Electromigration follows Black's equation. Capacitor life doubles per 10 K.
- **Wall power:** P_wall = P_chips/η_PSU + P_pumps + P_fans (+ compressor).
- **Metrics:**
  - W per K gained (lower is better)
  - performance gained per W at the wall

## Layer 10: transients

- **Lumped model:** C · dT/dt = Q_in(t) − (T − T_a)/R, with τ = R·C.
- **Load profiles:** gaming, CPU-only and GPU compute, plus boost spikes.
- **Thermosiphon behaviour:** start-up, low-load instability, and thermal mass and storage if a chiller is involved.

## Layer 11: uncertainty and sensitivity (the reason to over-parameterize)

- Give every input a range, not a point: TIM resistance, HTCs, leakage k, room temperature, RH and fill ratio.
- Run Monte Carlo through layers 0–9. Output the distributions of T_die, W/K and feasibility margin.
- Run a global sensitivity analysis (Sobol indices, SALib) to rank the parameters that move the answer. Measure those first on the bench.

## Fidelity ladder (maps onto the engineering toolchain survey)

1. **1D resistance network plus cycle model** in Python with CoolProp. This covers layers 0–11, and Monte Carlo runs in seconds.
2. **2D/3D conduction in the copper cold plate:** build123d → gmsh → scikit-fem (or CalculiX).
3. **CFD for condenser airflow and conjugate heat transfer:** OpenFOAM on the Mac Mini.
4. **Bench validation:** wall meter, thermocouples, dew point and GPU clocks vs coolant temperature.

## Open fork: design the computer around the heat pump

Instead of retrofitting a cooler onto a PC, place the components for the thermal system:

- Hot sources and cold loads get their own loops.
- The evaporator sits low and the condenser high, to maximise the thermosiphon head.
- Short vapour lines.
- The board layout follows the heat paths.

Candidate for its own project (see `project-heat-driven-self-cooling-pc` memory).
