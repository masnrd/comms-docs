---
tags: [robotics]
---

In [PX4]("./PX4.md"), simulation can be:
- **Software-in-the-Loop** (SITL): Flight stack runs on the computer.
- **Hardware-in-the-loop** (HITL): Flight stack runs on actual flight controller.

For now, I'm working with [jMAVSim]("./jMAVSim.md") for basic testing, but I'll probably have to transition to Gazebo if I want to test with anything other than a flat plane.

# SITL
![PX4 SITL Communication]("./img/PX4 SITL Communication.svg")

The direct communication between the simulator (jMAVSim, Gazebo, etc) and PX4 on SITL is TCP port 4560. This shouldn't be important.
- This is over MAVLink unless the simulator is Gazebo, but this really isn't important (I think).

More importantly:
- QGroundControl Communication with PX4 on SITL:
  - QGC Port: 14550
  - PX4 SITL Port: 18570
- API/Offboard Communication with PX4 on SITL:
  - Offboard Port(s): 14540-14549
  - PX4 SITL Port: 14580
- These ports use UDP and communicate with [MAVLink]("./MAVLink.md").

To build the simulator, see [PX4]("./PX4.md").

# HITL
![PX4 HITL Communication]("./img/PX4 HITL Communication.svg")

Here, the PX4 firmware runs on the real flight controller, with the simulator (on a development computer) connected via USB/UART.

See [this link](https://docs.px4.io/main/en/simulation/hitl.html) for details.
