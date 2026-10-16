# ROSOrin Pro Simulation — Part 1

ROS 2 simulation workspace for the Hiwonder ROSOrin Pro mecanum robot. This repository preserves the simulation source code and documents the setup and commands used for Gazebo simulation, RViz, LiDAR, SLAM mapping, and navigation.

## 1. Environment

- **Operating system:** Ubuntu 22.04 in VMware
- **ROS 2:** Humble
- **Robot:** ROSOrin Pro Mecanum
- **Packages:** `robot_gazebo`, `rosorin_description`
- **Environment configuration:** `.typerc`
- **LiDAR configuration:** MS200
- **Saved map:** `map_01.yaml` and `map_01.pgm`, if included in the repository

Gazebo performance in a virtual machine may be lower than on a native Linux installation.

## 2. Repository layout

The main source code is expected under `src/simulations/`.

```text
.
├── .typerc
├── .gitignore
├── README.md
└── src/
    └── simulations/
        ├── robot_gazebo/
        └── rosorin_description/
```

The exact subdirectories depend on the version of the Hiwonder source package. Map files may be located under `src/simulations/robot_gazebo/maps/`.

## 3. Prerequisites

Prepare Ubuntu 22.04, ROS 2 Humble, `colcon`, `rosdep`, and the dependencies required by the Hiwonder simulation package.

Source ROS 2:

```bash
source /opt/ros/humble/setup.bash
```

## 4. Build the workspace

These commands assume the workspace is located at `~/ros2_ws` inside Ubuntu:

```bash
cd ~/ros2_ws
source /opt/ros/humble/setup.bash

rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install

source ~/ros2_ws/install/setup.bash
source ~/ros2_ws/.typerc
```

To inspect the launch files available in the local source tree:

```bash
find ~/ros2_ws/src/simulations/robot_gazebo/launch -type f -name '*.launch.py' -print
```

## 5. Launch the room simulation

```bash
cd ~/ros2_ws
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
source ~/ros2_ws/.typerc

ros2 launch robot_gazebo room_worlds.launch.py
```

If the launch file cannot be found, check that the workspace was built and sourced and that the launch file exists in the package version in this repository.

### Keyboard teleoperation

Open a second terminal and run:

```bash
cd ~/ros2_ws
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
source ~/ros2_ws/.typerc

ros2 run robot_gazebo teleop_key_control
```

Keep the terminal focused while sending keyboard input. Follow the controls printed by the teleoperation node.

## 6. LiDAR and RViz

The configured simulated LiDAR publishes data on `/scan`. Check the topic with:

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
source ~/ros2_ws/.typerc

ros2 topic list
ros2 topic info /scan
ros2 topic echo /scan --once
```

If `/scan` is missing, confirm the simulation is running and inspect the launch output for sensor or bridge errors.

RViz configuration files may be provided in the `robot_gazebo/rviz/` directory (for example, `slam.rviz` or `nav_nav2.rviz`, depending on the package version). Use the files that actually exist in the checked-out source.

## 7. SLAM and saving a map

Start the room simulation in one terminal:

```bash
cd ~/ros2_ws
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
source ~/ros2_ws/.typerc

ros2 launch robot_gazebo room_worlds.launch.py
```

In a second terminal, start the SLAM launch file provided by the package:

```bash
cd ~/ros2_ws
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
source ~/ros2_ws/.typerc

ros2 launch robot_gazebo slam.launch.py
```

In a third terminal, start teleoperation and drive around the room:

```bash
cd ~/ros2_ws
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
source ~/ros2_ws/.typerc

ros2 run robot_gazebo teleop_key_control
```

Save the map:

```bash
ros2 run nav2_map_server map_saver_cli \
  -f ~/ros2_ws/src/simulations/robot_gazebo/maps/map_01
```

This writes `map_01.yaml` and `map_01.pgm` to the specified location. Make sure the destination directory exists. Inspect the saved map because coverage depends on the path driven and the sensor observations.

If `slam.launch.py` is not present in this package version, inspect the available launch files and use the SLAM launch file supplied by the checked-out source.

## 8. Navigation using a saved map

**Terminal 1 — launch the room with navigation enabled:**

```bash
cd ~/ros2_ws
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
source ~/ros2_ws/.typerc

ros2 launch robot_gazebo room_worlds.launch.py nav:=true
```

**Terminal 2 — launch navigation with the saved map:**

```bash
cd ~/ros2_ws
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
source ~/ros2_ws/.typerc

ros2 launch robot_gazebo navigation.launch.py map:=map_01
```

In the package version used during setup, `navigation.launch.py` was located under `robot_gazebo/launch/include/` rather than necessarily at the top level. The command above depends on the package exposing that launch file through its launch configuration. If ROS 2 cannot find it, inspect the actual launch files and include structure before editing source files.

In RViz, use **2D Pose Estimate** to initialize the robot pose if required, then **2D Nav Goal** to send a target. Navigation depends on the saved map, localization, transforms, sensor data, and navigation configuration.

**Multi-point waypoint navigation has not been confirmed as working in this handover.** Do not describe it as completed unless it is successfully demonstrated.

## 9. Troubleshooting

### Package or launch file not found

```bash
cd ~/ros2_ws
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
ros2 pkg list | grep robot_gazebo
ros2 pkg prefix robot_gazebo
find src/simulations/robot_gazebo/launch -type f -name '*.launch.py' -print
```

If source files have changed, rebuild and source the workspace again.

### `/scan` is missing

- Confirm Gazebo is still running.
- Check the terminal output for sensor or bridge errors.
- Use `ros2 topic list` to confirm the topic name used by this package version.

### Navigation does not move

- Confirm the map is loaded and visible in RViz.
- Initialize the robot pose with **2D Pose Estimate** if required.
- Check that TF, `/scan`, odometry, and navigation nodes are active.
- Inspect the navigation terminal for lifecycle, costmap, or transform errors.

### Low simulation frame rate

Gazebo performance was limited in the virtual-machine environment. This can reduce visual smoothness and slow testing; it should be described as a VM performance limitation, not as a measured robot-control frequency.

## 10. Notes

- Commands assume the workspace is at `~/ros2_ws` inside Ubuntu, not the Windows download directory.
- Do not commit generated `build/`, `install/`, or `log/` directories.
- Check `.typerc` and all configuration files for credentials or sensitive information before making the repository public.
- Only report features that were successfully demonstrated and can be reproduced from the submitted source.
