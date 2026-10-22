# thermal-forge

Can you give an everyday household computer a supercomputer-style cooling system? This project designs the cooling system and the computer together, starting with the heat path.

## Where it stands

Research and modelling. Nothing is built yet.

The first research pass (`docs/research/`) found the following:

- **A computer's own heat can't chill that same chip below room temperature.** The temperature budget comes up short: about 52 °C is available against the 56–62 °C the best sorption pairs need.
- **The best performer per watt is the approach supercomputers use.** Run warm fluid straight onto the chips and reject the heat efficiently, with no chiller.
- **A pumpless copper two-phase loop (thermosiphon) on R134a is the lead design.** It keeps CPU and GPU in their healthy bands for near-zero wall power.
- **A copper thermosyphon study backs R134a over water.** In Rongchai & Tundee, 2022 ([doi:10.1016/j.csite.2022.102453](https://doi.org/10.1016/j.csite.2022.102453)), R134a reached 51 % effectiveness against water's 30 %.

## Layout

- `CONTEXT.md`: terms and ground rules
- `docs/adr/`: decisions
- `docs/research/`: research reports and the calculation framework (layers 0–11)
- `model/scripts/`: CoolProp models used by the research

## Run the models

```sh
python3 -m venv .venv && .venv/bin/pip install -r requirements.txt
.venv/bin/python model/scripts/fluids.py
```

## Next

1. Turn the calculation framework into a tested, parameterized model with Monte Carlo and sensitivity ranking.
2. Bench experiments: GPU clocks vs coolant temperature, then a single copper R134a loop on a CPU.
3. Co-design the case layout around the loop.

Engineering tooling comes from [MARVIN](https://github.com/G-Eskayo/marvin).
