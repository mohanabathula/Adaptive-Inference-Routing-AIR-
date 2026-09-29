# Performance Aware Load Balancer 
# Real - Time control
..................
____________________________
# CARLA (0.9.13) with ROS Bridge for Ubuntu Noetic

This guide explains how to set up and run **CARLA 0.9.13 with ROS Noetic and the CARLA ROS Bridge** for the **ALB (Adaptive Load Balancer) / AIR (Adaptive Inference Routing)** experiments. It covers launching the CARLA simulator, connecting it to ROS, spawning the ego and traffic vehicles, visualizing camera data, and manually controlling the ego vehicle.


---

## Documentation Links

- **CARLA 0.9.13 Documentation**: [CARLA 0.9.13 Build Guide](https://carla.readthedocs.io/en/0.9.13/build_linux/)
- **CARLA ROS Bridge Documentation**: [ROS Bridge Docs](https://carla.readthedocs.io/projects/ros-bridge/en/latest/ros_installation_ros1/)
- **CARLA ROS Bridge GitHub Repository**: [GitHub - carla-simulator/ros-bridge](https://github.com/carla-simulator/ros-bridge)

---

## System Requirements

- **Ubuntu 20.04** or later
- **ROS Noetic** installed
- **CARLA 0.9.13** installed
- **Python 3.8** or later
- **CARLA ROS Bridge** installed

> **Project naming:** The project was originally called **ALB (Adaptive Load Balancer)** and is now called **AIR (Adaptive Inference Routing)**. Some existing packages and scripts still use the old `carla_lb` naming.

---

## 1. System Requirements

* Ubuntu 20.04
* ROS Noetic
* CARLA 0.9.13
* Python 3.8+
* CARLA ROS Bridge
* ALB/AIR ROS workspace and dependencies

### Useful Documentation

* [CARLA 0.9.13 Documentation](https://carla.readthedocs.io/en/0.9.13/)
* [CARLA ROS Bridge Documentation](https://carla.readthedocs.io/projects/ros-bridge/en/latest/ros_installation_ros1/)
* [CARLA ROS Bridge GitHub](https://github.com/carla-simulator/ros-bridge)

---

# 2. Start CARLA

CARLA must be running before starting the ROS Bridge.

### Terminal 1 — Start CARLA

Navigate to the CARLA installation:

```bash
cd ~/carla-0.9.13
```

Launch CARLA:

```bash
make launch
```

This starts the CARLA server/simulator.

> **VNC:** If CARLA is running on a remote machine, log in through VNC and make sure the CARLA simulator is running before continuing.

Wait until CARLA is fully loaded before launching the ROS Bridge.

---

# 3. Start CARLA ROS Bridge

Open a **new terminal**.

### Terminal 2 — ROS Bridge + Ego Vehicle

Navigate to the CARLA ROS Bridge workspace:

```bash
cd ~/carla-ros-bridge/catkin_ws
```

Source the workspace:

```bash
source devel/setup.bash
```

Launch the ROS Bridge with the example ego vehicle:

```bash
roslaunch carla_ros_bridge carla_ros_bridge_with_example_ego_vehicle.launch
```

This:

* Starts the CARLA ROS Bridge.
* Connects ROS to the CARLA simulator.
* Spawns the example **ego vehicle**.
* Publishes CARLA vehicle and sensor information as ROS topics.

---

# 4. Manual Ego Vehicle Control

Once the ego vehicle is spawned, it can be controlled using the keyboard.

| Key | Action                        |
| --- | ----------------------------- |
| `W` | Move forward / accelerate     |
| `A` | Turn left                     |
| `S` | Brake                         |
| `D` | Turn right                    |
| `Q` | Reverse                       |
| `B` | Enable/disable manual control |

> **Tip:** If keyboard control is not responding, press `B` to enable manual control mode.

---

# 5. View CARLA Camera Streams

To view the camera streams published through ROS:

```bash
rqt_image_view
```

Select the required topic from the dropdown.

| Camera                | ROS Topic                                              |
| --------------------- | ------------------------------------------------------ |
| RGB                   | `/carla/ego_vehicle/rgb_front/image`                   |
| Depth                 | `/carla/ego_vehicle/depth_front/image`                 |
| Semantic Segmentation | `/carla/ego_vehicle/semantic_segmentation_front/image` |

The RGB camera stream is particularly useful for monitoring the perception pipeline during ALB/AIR experiments.

---

# 6. Spawn Traffic for ALB/AIR Experiments

After CARLA and the ROS Bridge are running, traffic vehicles can be spawned in front of the ego vehicle.

### Terminal 3 — Spawn Traffic

Source the ALB/AIR workspace:

```bash
source ~/catkin_ws/devel/setup.bash
```

Run:

```bash
rosrun carla_lb spwan_traffic.py
```

> **Note:** `spwan` is the existing script name in the project and is intentionally retained.

### Default Traffic Setup

The script spawns **two vehicles in front of the ego vehicle**, with approximately **10 m spacing**.

```text
              🚗 Vehicle 1    🚗 Vehicle 2
                     │             │
                     └──── 10 m ───┘
                            │
                            ▼
                       🚙 Ego Vehicle
```

### Modify Traffic Scenario

The traffic-spawning code can be modified to change:

* Number of traffic vehicles.
* Distance between vehicles.
* Distance from the ego vehicle.
* Vehicle spawn positions.
* Vehicle placement/orientation.

Modify the corresponding parameters in the spawning script and run:

```bash
rosrun carla_lb spwan_traffic.py
```

again.

---

# 7. ALB / AIR Experiment

Once CARLA, the ROS Bridge, the ego vehicle, and traffic vehicles are running, start the required **ALB/AIR perception and inference nodes**.

The overall system is:

```text
                CARLA 0.9.13
                     │
                     ▼
              ROS CARLA Bridge
                     │
          ┌──────────┼──────────┐
          │          │          │
          ▼          ▼          ▼
       Ego Car    Camera     Vehicle State
          │          │
          │          ▼
          │     Perception
          │          │
          └──────────┼──────────┐
                     ▼          │
                  ALB / AIR     │
                     │          │
                     ▼          │
              Inference Routing │
                     │          │
                     ▼          │
              Onboard / Edge    │
                                │
                     Traffic Scenario
```

---

# 8. Typical Terminal Setup

For a standard ALB/AIR experiment, use separate terminals as follows:

### Terminal 1 — CARLA

```bash
cd ~/carla-0.9.13
make launch
```

### Terminal 2 — CARLA ROS Bridge

```bash
cd ~/carla-ros-bridge/catkin_ws
source devel/setup.bash
roslaunch carla_ros_bridge carla_ros_bridge_with_example_ego_vehicle.launch
```

### Terminal 3 — Spawn Traffic

```bash
source ~/catkin_ws/devel/setup.bash
rosrun carla_lb spwan_traffic.py
```

### Terminal 4+ — ALB/AIR Nodes

Launch the required ALB/AIR perception, monitoring, and inference nodes according to the experiment being performed.

---

# 9. Useful ROS Commands

### Check Running ROS Nodes

```bash
rosnode list
```

### Check Available Topics

```bash
rostopic list
```

### Check a Specific Topic

```bash
rostopic echo <topic_name>
```

### View Camera

```bash
rqt_image_view
```

---

# 10. Manual Vehicle Control via ROS

The ego vehicle can also be controlled by publishing a ROS command.

Example — left turn:

```bash
rostopic pub /carla/ego_vehicle/vehicle_control_cmd_manual carla_msgs/CarlaEgoVehicleControl \
"{throttle: 0.3, steer: 0.5, brake: 0.0, hand_brake: false, reverse: false, gear: 0, manual_gear_shift: false}"
```

Important fields:

| Field      | Meaning              |
| ---------- | -------------------- |
| `throttle` | Forward acceleration |
| `steer`    | Steering command     |
| `brake`    | Braking command      |
| `reverse`  | Reverse mode         |
| `gear`     | Vehicle gear         |

---

# 11. Troubleshooting

### CARLA Not Starting

Check that:

```bash
cd ~/carla-0.9.13
make launch
```

starts successfully and that the CARLA simulator is fully loaded before starting the ROS Bridge.

### ROS Bridge Cannot Connect

Check:

* CARLA is already running.
* The CARLA version is **0.9.13**.
* The ROS Bridge workspace is correctly sourced.
* CARLA ROS Bridge is compatible with CARLA 0.9.13.

### ROS Package Not Found

Source the appropriate workspace:

```bash
source ~/catkin_ws/devel/setup.bash
```

Then check:

```bash
rospack find carla_lb
```

### Traffic Not Spawning

Check that:

```bash
rosnode list
```

shows the CARLA ROS Bridge nodes and that the ego vehicle is already spawned.

Then run:

```bash
rosrun carla_lb spwan_traffic.py
```

### Camera Not Visible

Run:

```bash
rqt_image_view
```

and verify that the required camera topic is available:

```bash
rostopic list | grep image
```

---

# 12. Complete Startup Flow

For a normal ALB/AIR CARLA experiment:

```text
1. Start CARLA
       ↓
2. Start CARLA ROS Bridge
       ↓
3. Spawn Ego Vehicle
       ↓
4. Spawn Traffic Vehicles
       ↓
5. Start Camera / Perception
       ↓
6. Start ALB / AIR
       ↓
7. Run Experiment
```

### Quick Reference

```bash
# Terminal 1
cd ~/carla-0.9.13
make launch
```

```bash
# Terminal 2
cd ~/carla-ros-bridge/catkin_ws
source devel/setup.bash
roslaunch carla_ros_bridge carla_ros_bridge_with_example_ego_vehicle.launch
```

```bash
# Terminal 3
source ~/catkin_ws/devel/setup.bash
rosrun carla_lb spwan_traffic.py
```

```bash
# Camera visualization
rqt_image_view
```

Then launch the required **ALB/AIR nodes** for the experiment.

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
