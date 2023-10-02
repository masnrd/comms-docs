# wireshark-dissector

This is a dissector to parse MAVLink packets -- that is, it allows Wireshark to read anything that is a MAVLink packet.

# Installation
1. Find the *Global Lua Plugins* directory in Wireshark.
   - In Wireshark, go to `Help > About Wireshark`, and view the `Folders` tab.
2. Create the folder if it's missing.
3. Copy `mavlink_2_common.lua` there.

Re-open Wireshark.

# Usage
Like other protocols in Wireshark, you can filter for MAVLink data with `mavlink_proto`. Alternatively, you can refer to [this link](../Simulation.md) to identify the UDP ports to filter for.
- **Example**: Filter for MAVLink messages from QGroundControl to PX4 on SITL.
  - `mavlink_proto && udp.srcport=14550`
- **Example**: Filter for MAVLink messages from offboard communication to PX4 on SITL.
  - `mavlink_proto && udp.srcport >= 14540 && udp.srcport <= 14549`.
  
