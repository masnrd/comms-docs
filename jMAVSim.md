---
tags: [robotics]
---

A simulation for multirotor drones that can connect to a SITL version of [PX4](./PX4.md).
- This can connect to QGroundControl, or be connected to an external controller.

# Setup
Refer to the Setup section of [PX4](./PX4.md).

# Usage
To initialise jMAVSim:
```bash
cd PX4-Autopilot
make px4_sitl jmavsim
```

This will build the firmware for the **simulated** drone (i.e. the [SITL](./Simulation.md) PX4 firmware for the jMAVSim drone). The files for the simulated firmware are in [`PX4-Autopilot/boards/px4/sitl`](https://github.com/PX4/PX4-Autopilot/tree/main/boards/px4/sitl).

# Further Reading
- https://docs.px4.io/main/en/simulation/jmavsim.html
