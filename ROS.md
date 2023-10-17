---
tags: [robotics]
---

# Usage
## Building a Workspace
After sourcing ROS's `setup.bash`, in the project root:
```bash
colcon build
```

This builds all the folders under `./src`.

## Usage
In a new terminal, source ROS's `setup.bash`, and source the overlay.

```bash
source install/local_setup.bash
```

This provides access to the environment hooks for the current workspace, making the executables available.
- This is an "overlay" on top of the underlying ROS2 environment from `setup.bash`.
- Note that sourcing the original installation is not necessary if we `source install/setup.bash`.

Then, `ros2 launch` launches a specific launch file. A launch file is typically a file that executes multiple processes along with the necessary ROS nodes ([Example](https://github.com/PX4/px4_ros_com/blob/main/launch/sensor_combined_listener.launch.py)).

# Controlling Applications
To interface with PX4, ROS2 can subscribe to topics published by PX4, or publish to topics that PX4 listens to. These topics are defined in [`PX4-Autopilot/src/modules/uxrce_dds_client/dds_topics.yaml`](https://github.com/PX4/PX4-Autopilot/blob/main/src/modules/uxrce_dds_client/dds_topics.yaml), and more information is available at the [uORB Message Reference](http://docs.px4.io/main/en/msg_docs/).

## Warnings
### Differing QoS Settings
ROS2 code subscribing to PX4 topics should do so using the ROS2 predefined QoS sensor data.

```c++
rmw_qos_profile_t qos_prof = rmw_qos_profile_sensor_data;
auto qos = rclcpp::QoS(rclcpp::QoSInitialization(qos_prof.history, 5), qos_prof);

sub_ = this->create_subscription<px4_msgs::msg::SensorCombined>("/fmu/out/sensor_combined", qos, ...);
```

### Different Frame Conventions
| Frame |   PX4   |   ROS   |
| Body  |   FRD   |   FLU   |
| World | FRD/NED | FLU/ENU |

- **FRD**: X Forward, Y Right, Z Down
- **NED**: X North, Y East, Z Down
- **FLU**: X Forward, Y Left, Z Up
- **ENU**: X East, Y North, Z Up

To rotate a vector from ENU to NED:
1. Rotate $\frac{\pi}{2}$ around the $z$-axis (up)
2. Rotate $\pi$ around the $x$-axis (old East/new North)

To rotate a vector from NED to ENU:
1. Rotate $\frac{\pi}{2}$ around the $z$-axis (down)
2. Rotate $\pi$ around the $x$-axis (old East/new North)

To rotate a vector from FLU to FRD: Rotate $\pi$ around the $x$-axis (front).

To rotate a vector from FRD to FLU: Rotate $\pi$ around the $x$ axis (front).

**Further Reading**: [ROS 2 & PX4 Frame Conventions](http://docs.px4.io/main/en/ros/ros2_comm.html#ros-2-px4-frame-conventions).

### Time Synchronisation
Automatically handled by the uXRCE-DDS middleware. 
- These statistics are reported to the topic `/fmu/out/timesync_status`.
- Note that when simulating with Gazebo, there could be issues. Refer [here](http://docs.px4.io/main/en/ros/ros2_comm.html#ros-gazebo-and-px4-time-synchronization) for possible workarounds.

# Examples
## Listener
A listener is a ROS node that listens to topics published by [PX4](PX4.md).

Here, we have a listener for the `SensorCombined` uORB message. Other examples are included [here](https://github.com/PX4/px4_ros_com/tree/main/src/examples/listeners).
```c++
// C++ Libraries to interface with ROS2
#include <rclcpp/rclcpp.hpp>

// Header file for SensorCombined message that node subscribes to
#include <px4_msgs/msg/sensor_combined.hpp>

// Class that subclasses generic rclcpp::Node base class.
class SensorCombinedListener : public rclcpp::Node {
	public:
	
		// Callback function, called when SensorCombined uORB messages are received (as micro XRCE-DDS messages)
		explicit SensorCombinedListener() : Node("sensor_combined_listener")
		{
			// Set QoS Settings based on rmw_qos_profile_sensor_data, since default ROS2 QoS profile is incompatible.
			rmw_qos_profile_t qos_prof = rmw_qos_profile_sensor_data;
			auto qos = rclcpp::QoS(rclcpp::QoSInitialization(qos_prof.history, 5), qos_prof);
			
			// Create subscription
			sub_ = this->create_subscription<px4_msgs::msg::SensorCombined>(
				"/fmu/out/sensor_combined", 
				qos, 
				[this](const px4_msgs::msg::SensorCombined::UniquePtr msg) {
					std::cout << "ts: " << msg->timestamp << std::endl;
				}
			);
		}
	private:
		rclcpp::Subscription<px4_msgs::msg::SensorCombined>::SharedPtr sub_;
}

int main(int argc, char** argv)
{
	setvbuf(stdout, NULL, _IONBF, BUFSIZ);
	rclcpp::init(argc, argv);
	
	// Run listener ROS node on loop.
	rclcpp::spin(std::make_shared<SensorCombinedListener>()); 
	
	rclcpp::shutdown();
	return 0;
}
```

## Advertiser
An advertiser is a ROS2 node that publishes to the DDS/RTPS network, and hence to PX4.

Here, we have an example that `DebugVect` messages to the firmware.
```c++
#include <chrono>
#include <rclcpp/rclcpp.hpp>
#include <px4_msgs/msg/debug_vect.hpp>

using namespace std::chrono_literals; // Replaces time literals with the necessary data (e.g. milliseconds)

class DebugVectAdvertiser : public rclcpp::Node {
	public:
		// Sets up the timer callback.
		DebugVectAdvertiser() : Node("debug_vect_advertiser") 
		{
			publisher_ = this->create_publisher<px4_msgs::msg::DebugVect>("fmu/debug_vect/in", 10);
			
			// Callback function called by timer
			auto timer_callback = [this]()->void {
				auto dbvec = px4_msgs::msg::DebugVect();
				dbvec.timestamp = std::chrono::time_point_cast<std::chrono::microseconds>(std::chrono::steady_clock::now()).time_since_epoch().count(); // Current time
				std::string name = "test";
				std::copy(name.begin(), name.end(), dbvec.name.begin());
				dbvec.x = 1.0; dbvec.y = 2.0; dbvec.z = 3.0;
				
				// Report to log
				RCLCPP_INFO(this->get_logger(), "\033[97m Publish dbvec: ts %llu, x %f, y %f, z %f \033[0m", dbvec.timestamp, dbvec.x, dbvec.y, dbvec.z);
				
				// Publish!
				this->publisher_->publish(db_vec);
			};
			
			// Set up timer
			timer_ = this->create_wall_timer(500ms, timer_callback);)
		}
	
	private:
		rclcpp::TimerBase::SharedPtr timer_;
		rclcpp::Publisher<px4_msgs::msg::DebugVect>::SharedPtr publisher_;
}

int main(int argc, char** argv)
{
	rclcpp::init(argc, argv);
	
	// Instantiate the DebugVectAdvertiser
	rclcpp::spin(std::make_shared<DebugVectAdvertiser>());
	
	rclcpp::shutdown();
	return 0;
}
```
