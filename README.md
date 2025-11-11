# SUMO-RL

Lightweight reinforcement-learning environments for traffic-signal control built on SUMO (Simulator of Urban MObility). Provides Gymnasium and PettingZoo style APIs plus example agents and experiments.

## Quick start

1. Install SUMO (Ubuntu example)
```bash
sudo add-apt-repository ppa:sumo/stable
sudo apt-get update
sudo apt-get install sumo sumo-tools sumo-doc
```

2. Set SUMO_HOME (add to shell rc)
```bash
export SUMO_HOME="/usr/share/sumo"
echo 'export SUMO_HOME="/usr/share/sumo"' >> ~/.bashrc
source ~/.bashrc
```

3. Use a Python virtual environment (recommended)
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -U pip setuptools wheel
pip install -e .
```
Note: the package does not expose an "all" extra; install optional RL libs manually if needed.

4. (Optional, headless performance) enable libsumo
```bash
export LIBSUMO_AS_TRACI=1
```
Warning: libsumo speeds up simulation but disables the SUMO GUI and parallel SUMO processes.

## Run an example

- Headless (fast):
```bash
export LIBSUMO_AS_TRACI=1
python experiments/ql_2way-single-intersection.py \
    -route sumo_rl/nets/2way-single-intersection/single-intersection-horizontal.rou.xml \
    -s 600
```

- With GUI (visualize; slower). Ensure libsumo is disabled and DISPLAY is available:
```bash
unset LIBSUMO_AS_TRACI
python experiments/ql_2way-single-intersection.py -gui \
    -route sumo_rl/nets/2way-single-intersection/single-intersection-horizontal.rou.xml \
    -s 600
```

- Run sarsa_double experiment (uses Fire CLI):
```bash
# headless quick run
export LIBSUMO_AS_TRACI=1
python experiments/sarsa_double.py run --use_gui=False --runs=1
# GUI (requires X or xvfb)
unset LIBSUMO_AS_TRACI
python experiments/sarsa_double.py run --use_gui=True --runs=1
```

- Run 4x4 Q‑learning sample (edit script to set --gui/--runs/--seconds or pass args if updated):
```bash
# headless quick test
export LIBSUMO_AS_TRACI=1
python experiments/ql_4x4grid-mo.py --runs 1 --episodes 1 --seconds 600
```

## Files of interest

- sumo_rl/environment/env.py — SumoEnvironment (core)
- sumo_rl/environment/traffic_signal.py — agent observation/action/reward logic
- sumo_rl/environment/observations.py — observation functions (extend by subclassing ObservationFunction)
- experiments/ — runnable example scripts (QL, DQN, SARSA, etc.)
- sumo_rl/nets/ — bundled nets and routes for quick testing
- outputs/ — CSV logs created by experiments

## Troubleshooting

- FatalTraCIError: Connection closed by SUMO.
  - Run the same net+route with sumo-gui or sumo to see the SUMO error:
    sumo -n path/to/net.xml -r path/to/route.rou.xml 2>&1 | tee sumo.log
  - Check that route edges and net edges match and that SUMO can open the GUI (DISPLAY set).
- "Please declare SUMO_HOME": export SUMO_HOME correctly and run in same shell.
- Permission errors during pip install: use a venv or pip install --user.
- If GUI fails on a headless server, use xvfb-run:
```bash
xvfb-run -s "-screen 0 1400x900x24" python experiments/ql_2way-single-intersection.py -gui -s 600
```

## Extending

- Custom observations: implement an ObservationFunction and pass instance to SumoEnvironment.
- Add or edit nets/routes under sumo_rl/nets/, then run experiments pointing to those files.

## Inspecting outputs

After a run, open CSVs under outputs/<scenario>/ and plot columns like system_total_arrived, system_total_departed, system_mean_waiting_time to evaluate performance. Example quick check:
```python
import pandas as pd
df = pd.read_csv("outputs/2way-single-intersection/<file>.csv")
df.plot(x="step", y=["system_total_arrived","system_total_departed","system_mean_waiting_time"])
```

License and contributions
- See project metadata for license and contribution guidelines.
