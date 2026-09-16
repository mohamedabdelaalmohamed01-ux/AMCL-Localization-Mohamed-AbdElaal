# AMCL Localization — Mohamed AbdElaal

A ROS 2 package demonstrating AMCL (Adaptive Monte Carlo Localization) using a previously saved map in a TurtleBot3 simulation environment (TurtleBot3 World).

## Project Overview

This project implements robot localization using AMCL on a map generated in a previous SLAM assignment. The robot is launched in Gazebo, the saved map is loaded in RViz2, and AMCL estimates the robot's pose by matching LiDAR scans against the saved map.

The workflow demonstrates:

- Loading a previously saved map and confirming it displays correctly in RViz.
- Configuring and launching `amcl`.
- Testing localization with a wrong initial pose vs. the correct initial pose.
- Driving the robot and confirming the particle cloud converges and localization remains stable.

## Package Structure

```
amcl-localization-mohamed-abdelaal/
└── robot_localization/
    ├── config/
    │   └── amcl.yaml
    ├── launch/
    │   └── amcl.launch.py
    ├── map/
    │   ├── turtlebot3_world_map.yaml
    │   └── turtlebot3_world_map.pgm
    ├── CMakeLists.txt
    └── package.xml
├── images/
└── README.md
```

- `config/amcl.yaml` — AMCL parameter configuration.
- `launch/amcl.launch.py` — Launch file that starts `map_server`, `amcl`, and `lifecycle_manager`.
- `map/` — Map files (`.yaml` + `.pgm`) generated in the previous SLAM assignment.
- `images/` — Screenshots and videos used in this README.

## Build Instructions

1. Create a ROS 2 workspace with a `src` folder in it:
   ```bash
   mkdir -p ros2_ws/src
   ```
2. Clone the repository into your ROS 2 workspace `src` folder:
   ```bash
   cd ~/ros2_ws/src
   git clone https://github.com/Mohamed-AbdElaal207/amcl-localization-mohamed-abdelaal.git
   ```
3. Build the package:
   ```bash
   cd ~/ros2_ws
   colcon build --packages-select robot_localization
   source install/setup.bash
   ```
4. Set the TurtleBot3 model (add to `~/.bashrc` if needed):
   ```bash
   export TURTLEBOT3_MODEL=burger
   ```

## Commands Used to Launch the Simulator and AMCL

1. Launch the Gazebo simulation (terminal 1):
   ```bash
   ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
   ```
2. Launch `map_server`, `amcl`, and `lifecycle_manager` (terminal 2):
   ```bash
   source ~/ros2_ws/install/setup.bash
   ros2 launch robot_localization amcl.launch.py
   ```
3. Launch RViz (terminal 3):
   ```bash
   source ~/ros2_ws/install/setup.bash
   rviz2
   ```

## RViz Configuration

After opening RViz:

- Set **Fixed Frame** to `map`.
- Add **Map**: under QoS Settings, set **Durability Policy = Transient Local**.
- Add **TF**.
- Add **RobotModel**: under Description Topic, choose `/robot_description`.
- Add **LaserScan**.
- Add **ParticleCloud**: set its topic to `/particle_cloud`, and under QoS Settings set **Reliability Policy = Best Effort**.
<img width="992" height="733" alt="map " src="https://github.com/user-attachments/assets/edb520d7-a77a-4d14-b123-aac85374a094" />

## Testing Steps

### 1. Wrong initial pose

Using the **2D Pose Estimate** tool, click on the map where the robot is located, then drag the arrow in a direction other than the one the robot is actually facing.

<img width="972" height="738" alt="incorr" src="https://github.com/user-attachments/assets/bafd5867-dab8-4b3a-9abf-f9ae60ec5081" />


**Observation:** With the wrong initial pose, the LiDAR scan mismatches with the walls of the saved map, and the particle cloud is scattered widely around the robot's actual location.


### 2. Correct initial pose

Repeat the previous step, but point the arrow in the direction the robot is actually facing.

<img width="988" height="731" alt="corr" src="https://github.com/user-attachments/assets/de2f3648-ec6a-4054-8fd2-88551f331932" />


**Observation:** With the correct initial pose, the LiDAR scan lines up closely with the walls of the saved map, and the particle cloud is tightly clustered around the robot's actual location.



### 3. Driving with teleop

```bash
ros2 run turtlebot3_teleop teleop_keyboard
```

<img width="1772" height="491" alt="Annotation 2026-09-16 022740" src="https://github.com/user-attachments/assets/3e5921d1-c985-4e3c-8ca0-7b6e06736739" />


**Observation:** As the robot moves, the particle cloud converges further, narrowing around the estimated pose — confirming AMCL is successfully tracking the robot's position.

## TF Tree

```bash
ros2 run tf2_tools view_frames
```

<img width="936" height="540" alt="Annotation 2026-09-16 022854" src="https://github.com/user-attachments/assets/904a603c-ab0c-4a89-8b7f-96a3df584cab" />


**Observation:** The TF tree confirms the expected chain: `map → odom → base_footprint → base_link → ...`, with AMCL publishing the `map → odom` transform.

## Required Topic and Transform Outputs

Confirm `/scan` and `/odom` are available:

```bash
ros2 topic list
```

Confirm `/amcl_pose` updates while the robot moves:

```bash
ros2 topic echo /amcl_pose --once
```

Expected output shape:

```
header:
  stamp:
    sec: <sec>
    nanosec: <nanosec>
  frame_id: map
pose:
  pose:
    position:
      x: <x>
      y: <y>
      z: 0.0
    orientation:
      x: 0.0
      y: 0.0
      z: <z>
      w: <w>
```

Confirm AMCL publishes the `map → odom` transform:

```bash
ros2 run tf2_ros tf2_echo map odom
```

Output:

```
At time 2655.0
- Translation: [-0.002, 0.027, 0.000]
- Rotation: in Quaternion (xyzw) [0.000, 0.000, 0.004, 1.000]
- Rotation: in RPY (radian) [0.000, -0.000, 0.008]
- Rotation: in RPY (degree) [0.000, -0.000, 0.478]
- Matrix:
  1.000 -0.008  0.000 -0.002
  0.008  1.000  0.000  0.027
  0.000  0.000  1.000  0.000
  0.000  0.000  0.000  1.000
```

> Note: the first line printed a one-time `Invalid frame ID "map"` warning because `tf2_echo` started before AMCL had published its first transform — this is expected on startup and clears up automatically once AMCL is active and the map → odom transform is published.

Check the map topic:

```bash
ros2 topic info -v /map
```

Output:

```
Type: nav_msgs/msg/OccupancyGrid
Publisher count: 1
  Node name: map_server
  Node namespace: /
  QoS: Reliability=RELIABLE, Durability=TRANSIENT_LOCAL

Subscription count: 2
  Node name: rviz
  Node name: amcl
  QoS (both): Reliability=RELIABLE, Durability=TRANSIENT_LOCAL
```

Check the estimated pose topic:

```bash
ros2 topic info -v /amcl_pose
```

Output:

```
Type: geometry_msgs/msg/PoseWithCovarianceStamped
Publisher count: 1
  Node name: amcl
  Node namespace: /
  QoS: Reliability=RELIABLE, Durability=TRANSIENT_LOCAL

Subscription count: 0
```

Check the particle cloud topic:

```bash
ros2 topic info -v /particle_cloud
```

Output:

```
Type: nav2_msgs/msg/ParticleCloud
Publisher count: 1
  Node name: amcl
  QoS: Reliability=BEST_EFFORT, Durability=VOLATILE

Subscription count: 1
  Node name: rviz
  QoS: Reliability=BEST_EFFORT, Durability=VOLATILE
```

## Common Problems Faced and How They Were Solved

- **No map appears in RViz:** Check the Map display settings — Map Topic = `/map`, Durability Policy = Transient Local.
- **No particle cloud appears:** Check the Particle Cloud display settings — Topic = `/particle_cloud`, Reliability Policy = Best Effort. Don't forget to set an initial pose using the **2D Pose Estimate** tool first.
- **LiDAR scan does not match the map:** Make sure the initial pose was set correctly (position and orientation).
- **AMCL node does not start:** Make sure the workspace is sourced:
  ```bash
  source ~/ros2_ws/install/setup.bash
  ```
<img width="992" height="733" alt="map " src="https://github.com/user-attachments/assets/d1d36b94-1b7d-4ddd-be8d-e3baf88fc46f" />

