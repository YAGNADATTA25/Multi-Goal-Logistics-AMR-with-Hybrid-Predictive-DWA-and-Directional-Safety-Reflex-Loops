# Autonomous Logistics Simulation Package (`autonomous_logistics_sim`)

A high-performance ROS 2 Humble simulation package designed for multi-goal warehouse fulfillment operations. This repository implements an autonomous logistics fleet dispatch pipeline featuring synchronized temporal simulation clocks, an asynchronous multi-waypoint orchestration script, and a thread-safe telemetry feedback system for tracking real-time vehicle drift.

---

## System Architecture

The package splits multi-goal logistics workflows into decoupled, asynchronous processing components communicating natively over the high-throughput ROS 2 DDS layer:

*   **Asynchronous Navigation Thread:** Instantiates a standalone background execution thread (`threading.Thread`) utilizing the Nav2 Simple Commander API (`BasicNavigator`) to fire a sequential array of 6 unique coordinate goals. Using an async wrapper keeps blocking state checkers (`isTaskComplete()`) from stalling core sensor ingestion.
*   **Simulation Master Node:** Operates a deterministic 10Hz orchestration loop[cite: 1]. It handles localized spatial cost updates and drives 4 independent simulation threat nodes using a continuous harmonic sine-wave equation ($V(t) = A \cdot \sin(\omega t)$) to mimic real-world warehouse traffic pacing at velocities up to $0.19\text{ m/s}$[cite: 1].
*   **Closed-Loop Telemetry System:** Runs on a dedicated 0.9-second telemetry clock thread[cite: 1]. It intercepts the global `/plan` path vector and applies a thread-safe mutex barrier to compute real-time steering performance logs without inducing UI or costmap processing lag[cite: 1].

<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/3701ee92-16d0-4ddb-be57-aba9a69fa6d1" />
---

## Core Telemetry Metrics

The system streams live control-theory metrics straight to the terminal console every $0.9\text{ s}$ to evaluate localization and path tracking efficiency[cite: 1]:
1.  **Cross-Track Error (XT_Err):** Continuous perpendicular distance tracking relative to the targeted global path trajectory vector, logged in centimeters[cite: 1].
2.  **Heading Error (Head_Err):** Real-time steering alignment delta relative to the target path trajectory orientation, logged in degrees[cite: 1].
3.  **Goal Distance:** Live Euclidean distance tracking straight to the active waypoint coordinate vector[cite: 1].


---

## Repository Directory Structure

```text
autonomous_logistics_sim/
├── CMakeLists.txt             # Colcon compilation properties
├── package.xml                # ROS 2 lifecycle, nav2_msgs, and rclpy dependencies[cite: 1]
├── README.md                  # System architectural documentation
├── config/
│   └── warehouse_params.yaml  # Tuned costmap inflation layers, radii, and sync settings[cite: 1]
├── launch/
│   └── warehouse_navigation.launch.py  # Deploys Gazebo, Nav2 lifecycle nodes, and maps[cite: 1]
├── maps/
│   ├── warehouse_map.pgm      # High-fidelity binary occupancy grid map[cite: 1]
│   └── warehouse_map.yaml     # Spatial metadata anchors for localization[cite: 1]
└── scripts/
    ├── simulation_master.py   # 10Hz traffic node manager & trajectory search engine[cite: 1]
    └── waypoint_navigator.py  # Asynchronous multi-goal sequential route manager[cite: 1]




Installation & Environment Build
cd ~/turtlebot3_ws
rosdep update
rosdep install --from-paths src --ignore-src -r -y --rosdistro humble

colcon build --packages-select autonomous_logistics_sim
source install/setup.bash
Execution Guidelines
1. Launch the Warehouse Simulation & Navigation Stack

To bring up the environment, sync the simulation clocks (use_sim_time: True), and activate the Nav2 lifecycle managers[cite: 1]:
Bash

export TURTLEBOT3_MODEL=waffle_pi
ros2 launch autonomous_logistics_sim logistics_fleet.launch.py

2. Run the Logistics Mission Dispatcher

In a secondary terminal window, spin up the asynchronous coordinator to command the robot to loop through the 6 warehouse waypoints, pause for delivery dwell times, and stream tracking telemetry[cite: 1]:
Bash

source ~/turtlebot3_ws/install/setup.bash
ros2 run autonomous_logistics_sim waypoint_navigator.py
