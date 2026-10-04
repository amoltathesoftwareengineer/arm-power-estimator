# arm-power-estimator

Static and profiling-based estimation of the energy and battery impact of code on ARM processors. Maps instruction mix and memory access patterns to power models, helping developers optimize mobile and embedded workloads.

> **Status:** early development. The analyzers, power models and validation setup are being built in the open. See the [Roadmap](#roadmap).

---

## Why this exists

Most phones, wearables, sensors and many IoT devices run on ARM processors and on a battery. For these devices, **energy use is a feature**: it decides battery life, heat and cost. Yet developers usually find out about energy problems late:

- Measuring real power needs **special hardware** (power monitors, instrumented boards) that many developers do not have.
- Tools that report energy are often **tied to one vendor or one chip.**
- Common advice such as "fewer instructions means less energy" is **not always true.** Memory access, cache misses and idle time often matter more.
- Energy results are rarely shared with enough detail to be **reproduced.**

`arm-power-estimator` aims to give developers an early, approximate view of energy cost before they own a power meter:

- **Static estimation** from compiled code, without running it.
- **Profiling-based estimation** from performance counters when the code can run.
- **Transparent power models** that show how instruction mix and memory behaviour turn into energy.
- **Validation against real measurements,** with error reported openly.
- **Reproducible experiments** from a plain config file.

---

## What it estimates

| Quantity | Meaning |
|---|---|
| **Energy per run** | Estimated energy (joules) used by a function or program |
| **Average power** | Estimated power draw (watts) during execution |
| **Breakdown** | Share of energy from compute, memory access, and idle or stall time |
| **Hotspots** | Functions or loops that contribute the most energy |
| **Battery impact** | Rough battery drain per hour for a given usage pattern and battery capacity |

All outputs are estimates with an error range, not exact measurements.

---

## Data and reference benchmarks

- **Benchmark programs:** open embedded and energy benchmark suites such as BEEBS, Embench and MiBench, used as test workloads
- **Hardware measurements:** power traces measured on real ARM boards (for example, Cortex-M boards with an on-board or external power monitor, and Raspberry Pi class boards) to calibrate and validate the models
- **Contributed measurements:** the project will accept power traces from other users, with the board, clock speed and measurement setup documented

Measurement data is stored with the full setup description so results can be repeated. Benchmark suites are not redistributed here, and each keeps its own license.

---

## Planned approach

### Static estimation

- Disassemble compiled ARM code and count instructions by class (arithmetic, load/store, branch, floating point, SIMD)
- Estimate memory behaviour (access patterns, likely cache hits and misses)
- Apply an instruction-class energy model fitted to measurements

### Profiling-based estimation

- Read hardware performance counters (cycles, instructions, cache misses, branch misses) with standard Linux tools where available
- Fit a regression model from counters to measured power
- Report estimated energy per function

### Metrics

- **Estimation error:** MAPE of energy estimates against measured energy
- **Ranking accuracy:** does the tool correctly rank which of two code versions uses less energy? This often matters more than exact numbers
- **Cross-board generalization:** how well a model fitted on one board works on another
- **Stability:** repeated runs give similar estimates

---

## How it works

```
compiled ARM binary / source
            |
      +-----+-------------+
      |                   |
      v                   v
 static analysis     profiling run
 (instruction mix,   (performance counters
  memory pattern)     per function)
      |                   |
      +---------+---------+
                |
                v
   power model (fitted on measured traces)
                |
                v
 energy estimate + breakdown + hotspots + error range
```

---

## Planned repository structure

```
arm-power-estimator/
├── data/
│   ├── traces/           # measured power traces with setup metadata
│   └── models/           # fitted model parameters
├── arm_power/
│   ├── static/           # disassembly parsing, instruction classes
│   ├── profiling/        # performance counter collection
│   ├── models/           # instruction-class and regression models
│   ├── battery.py        # battery impact calculation
│   └── evaluate.py       # error and ranking metrics
├── benchmarks/           # benchmark programs and scripts
├── notebooks/            # exploration and plots
├── tests/
├── docs/
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

---

## Quickstart (planned)

```bash
git clone https://github.com/amoltathesoftwareengineer/arm-power-estimator.git
cd arm-power-estimator
pip install -r requirements.txt

# estimate energy from a compiled binary (static)
python -m arm_power.static --binary ./app.elf --model cortex-m4

# estimate energy from a profiling run
python -m arm_power.profiling --command "./app" --model cortex-a53

# compare two versions of the same code
python -m arm_power.compare --a ./app_v1.elf --b ./app_v2.elf
```

---

## Results

Results will be added once measurements on real boards are collected.

| Board | Workload set | Energy error (MAPE) | Ranking accuracy |
|---|---|---|---|
| _coming soon_ | - | - | - |

---

## Roadmap

- [ ] Parse ARM disassembly and classify instructions
- [ ] Set up a repeatable power measurement process on one board
- [ ] Collect traces for open benchmark programs
- [ ] Fit and validate an instruction-class energy model
- [ ] Add performance-counter profiling and a regression model
- [ ] Add a memory-behaviour model
- [ ] Publish the first validated results with error ranges
- [ ] Add battery impact calculator
- [ ] Support more boards and cores, including contributed traces

---

## Limitations and responsible use

- These are **estimates.** Real energy depends on the chip, voltage, temperature, clock speed, operating system and other running software. Always measure on real hardware for final decisions.
- A model fitted on one board may be wrong on another. Reported error applies to the tested boards only.
- Static estimation cannot know run-time behaviour such as input-dependent branches or cache contents.
- This project is not affiliated with or endorsed by Arm Ltd or any chip vendor. "ARM" is used to describe the processor architecture.
- This project is for research and education. It is not a certified power analysis tool.

---

## Contributing

Contributions are welcome, especially:

- Power measurements from new boards (with full setup details)
- New power models and validation methods
- Support for more ARM cores and instruction sets
- Bug reports and reproducibility fixes

Please open an issue to discuss before sending a large pull request.

---

## License

MIT License. See [LICENSE](LICENSE).

---

## Author

**Amol Tathe**
[LinkedIn](https://linkedin.com/in/amoltathesoftwareengineer) · [GitHub](https://github.com/amoltathesoftwareengineer)
