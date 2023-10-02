---
tags: [robotics]
---

An open source autopilot **flight stack**.

In general, this has two main layers:
- **Flight Stack**: An estimation and flight control system.
- **Middleware**: A general robotics layer.

# Setup
See [this link](https://docs.px4.io/main/en/dev_setup/dev_env.html).

# Architecture
![PX4 Architecture](./img/PX4%20Architecture%20(FC).svg)
- **Flight Controller**: Runs the PX4 flight stack, contains internal IMUs, compasses and barometers. Our flight controller is a [*Pixhawk 6C*](https://docs.px4.io/main/en/flight_controller/pixhawk6c.html).
- **Telemetry Radio**: For connecting to a ground station.
- **Sensors**: Connected via I2C, SPI, CAN or UART
- **Ground Station**: Typically runs QGroundControl, and is connected via [MavLink](./MAVLink.md).

A **mission computer** (or a *companion computer*) can be connected to the drone, and communicates with the flight controller via [MavLink](./MAVLink.md).

## Software Architecture
Within the flight controller, communication is done between modules with a **publish-subscribe** message bus called **uORB**.

# Simulation
PX4 supports both *software-in-the-loop* (SITL) and *hardware-in-the-loop* simulation.
- See [jMAVSim](./jMAVSim.md) for such a simulator.
- See [Simulation](./Simulation.md) for more details.

# Firmware Development
In general, builds for the desired target (including simulated drone) is done with:
```bash
make CONFIGURATION_TARGET SIMULATOR
```
- `CONFIGURATION_TARGET`: All possible targets can be seen [here](https://github.com/PX4/PX4-Autopilot/tree/main/boards). A target is entered as `VENDOR_MODEL_VARIANT`.
  - `VENDOR`: Manufacturer of the board (e.g. `px4`).
  - `MODEL`: Board model (e.g. `sitl`, `fmu-v5`, `navio2`)
  - `VARIANT`: Particular configurations (often just `default`)
  - Note that all available targets can be found with:
	```bash
	  make list_config_targets
	  ```
  - For our case, we'd either use `px4-sitl` for simulation, or `px4_fmu-v7c_default`.
- `SIMULATOR`: Entered as `VIEWER_MODEL_DEBUGGER_WORLD`.
  - `VIEWER`: The simulator to use: `gz`/`gazebo`, `jmavsim`, `none` (`none` is used if you want to launch PX4 and wait for a simulator)
  - `MODEL`: Vehicle model to use: `iris` (default), `rover`, `tailsitter`
  - `DEBUGGER`: `none` (default), `ide`, `gdb`
  - `WORLD`: Only for Gazebo Classic. This sets the world to load, with a default of `empty.world`.
  - Note that all available options can be found with:
	```bash
	make px4_sitl list_vmd_make_targets
	```

# Sources
- https://docs.px4.io/main/en/dev_setup/building_px4.html
