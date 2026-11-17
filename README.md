# ROS 2 Manipulator: Arduinobot

A ROS 2 Humble workspace for a 3-DOF robotic arm with a two-finger gripper. The robot is simulated in Gazebo, controlled through `ros2_control`, planned with MoveIt 2, and commanded through a custom action server that runs predefined tasks.

<!-- Add demo GIF here: ![Task server demo](docs/demo.gif) -->

## Features

- **Robot model:** URDF/Xacro description with STL meshes and a simulated RGB camera publishing `/image_raw`
- **Simulation:** Gazebo (Ignition) with `ros2_control` joint trajectory controllers for the arm and gripper
- **Motion planning:** MoveIt 2 configuration with `arm` and `gripper` planning groups
- **Task server:** C++ action server that plans and executes a predefined arm and gripper pose for each task ID
- **ROS 2 basics:** Python and C++ examples of services, actions, and parameters, plus Euler and quaternion conversion services

## System Overview

```text
 ros2 action send_goal /task_server {task_number}
                    |
                    v
        +-----------------------+
        |  Task Server (C++)    |  arduinobot_remote
        |  ArduinobotTask action|
        +-----------------------+
                    |  MoveGroupInterface (arm, gripper)
                    v
        +-----------------------+
        |  MoveIt 2 move_group  |  arduinobot_moveit
        +-----------------------+
                    |  FollowJointTrajectory
                    v
        +-----------------------+
        |  ros2_control         |  arduinobot_controller
        |  arm / gripper        |
        +-----------------------+
                    |
                    v
        +-----------------------+
        |  Gazebo simulation    |  arduinobot_description
        +-----------------------+
```

## Packages

| Package | Description |
|---|---|
| `arduinobot_description` | URDF/Xacro model, meshes, RViz and Gazebo launch files, simulated RGB camera |
| `arduinobot_controller` | `ros2_control` configuration and controller spawners |
| `arduinobot_moveit` | MoveIt 2 configuration (SRDF, kinematics, joint limits, controllers) |
| `arduinobot_msgs` | Custom service and action interfaces, including `ArduinobotTask.action` |
| `arduinobot_remote` | Task server that executes predefined tasks through MoveIt 2 |
| `arduinobot_utils` | Euler-to-quaternion and quaternion-to-Euler conversion services |
| `arduinobot_cpp_examples` | MoveIt 2 C++ API example (`simple_moveit_interface`) |
| `arduinobot_py_examples` | Python examples of services, actions, and parameters |

## Task Server

The task server receives a task number through the `/task_server` action, plans both groups with MoveIt 2, and executes the trajectories.

| Task | Pose | Arm joints (rad) | Gripper joints (rad) |
|---:|---|---|---|
| 0 | Home, gripper open | `[0.0, 0.0, 0.0]` | `[-0.7, 0.7]` |
| 1 | Pick, gripper closed | `[-1.14, -0.6, -0.07]` | `[0.0, 0.0]` |
| 2 | Rest, gripper closed | `[-1.57, 0.0, -0.9]` | `[0.0, 0.0]` |

Action definition (`arduinobot_msgs/action/ArduinobotTask.action`):

```text
int32 task_number
---
bool success
---
int32 percentage
```

The server is written in C++ because `moveit_py` is not available on ROS 2 Humble. A Python version (`task_server.py`) is included for ROS 2 Iron and later.

## Requirements

- Ubuntu 22.04
- ROS 2 Humble
- MoveIt 2
- `ros2_control`, `ros2_controllers`
- Gazebo (Ignition) with `ign_ros2_control`
- `xacro`, `joint_state_publisher_gui`

## Build

```bash
mkdir -p ~/arduinobot_ws/src
cd ~/arduinobot_ws/src
git clone https://github.com/Heedong-Lee5909/ROS2_Manipulator.git
cd ~/arduinobot_ws
rosdep install --from-paths src --ignore-src -r -y
colcon build
source install/setup.bash
```

## Run

Run each command in a separate terminal after sourcing the workspace (`source install/setup.bash`).

```bash
# 1. Gazebo simulation
ros2 launch arduinobot_description gazebo.launch.py

# 2. Controllers
ros2 launch arduinobot_controller controller.launch.py

# 3. MoveIt 2 and RViz
ros2 launch arduinobot_moveit moveit.launch.py

# 4. Task server (C++)
ros2 launch arduinobot_remote remote_interface.launch.py
```

On ROS 2 Iron or later, use `use_python:=true` to run the Python task server.

Send a task:

```bash
ros2 action send_goal /task_server arduinobot_msgs/action/ArduinobotTask "{task_number: 1}"
```

To view the model only in RViz:

```bash
ros2 launch arduinobot_description display.launch.py
```