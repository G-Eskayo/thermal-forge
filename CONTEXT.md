# thermal-forge: context

Co-design a computer and its cooling system as one machine. Start from the heat path, then place the parts.

## Terms

- **Heat source:** any component that makes heat (CPU, GPU, VRM, PSU, chipset). Each has a wattage, a load profile and a healthy band.
- **Healthy band:** the temperature range where a part runs best without trading away lifetime. CPU case is about 35–50 °C. GPU die is about 45–65 °C.
- **No-sacrifice rule:** no component is pushed out of its healthy band to help another one or the system. This is a hard constraint.
- **Temperature budget:** the chain of temperature drops from a chip to the room or to a chiller (die → TIM → base → fluid → heat exchanger). The feasibility test for heat-driven cooling compares what's left against what the cycle needs.
- **Thermosiphon / loop thermosiphon:** a sealed, pumpless two-phase loop. Fluid boils at the cold plate, the vapour rises to a condenser, and the liquid falls back by gravity.
- **Heat-driven chiller:** a sorption or absorption cycle that makes cold from heat input (the generator) instead of a compressor.
- **Wall power:** electricity drawn from the outlet. It's a first-class metric next to temperature, reported as W per K gained.
- **Attainable:** a fluid, part or material a civilian can buy legally, in the needed quantity, at a sane price.

## Ground rules

- **Model before build.** Over-parameterize the virtual model, find the sensitive parameters, and measure those on the bench first.
- **Compact.** No multi-square-metre solar.
- **Any attainable fluid.** Don't default to water.
- **Safety.** Pressure-rated parts and a relief device on anything above atmospheric pressure.

## Tooling

MARVIN (a separate project) builds the general engineering tools this project uses: CAD, meshing, thermal and CFD analysis, and experiment design. Project-specific models, designs and data live here.
