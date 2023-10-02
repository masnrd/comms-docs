---
tags: [robotics]
---

A network protocol for communicating with drones.
- Data streams are sent and received through a **publish-subscribe model**.
- Configuration sub-protocols, like mission protocols are **point-to-point** with **retransmission**.

Messages are defined with [XML files](https://mavlink.io/en/messages/common.html).
- **Dialect**: The message set supported by a MAVLink system.
- A *code generator* generates software libraries from these message definitions, which can then be used by drones and ground stations.

# MAVSDK
A high-level Python API. 
- Documentation is [here](http://mavsdk-python-docs.s3-website.eu-central-1.amazonaws.com/).
