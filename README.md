# agriedge

`agriedge` is a ROS 2 Jazzy robot description and simulation package for an agricultural ground robot.

The platform is a **4-wheel skid-steer robot** designed for outdoor/agri use, with a complete simulated sensor stack in New Gazebo (`gz-sim`) and control through `ros2_control`.

## Robot Overview

The robot model includes:

- **4 driven wheels** (front/rear, left/right) for skid-steer motion.
- **Main chassis** based on CAD meshes.
- **3D LiDAR** (`gpu_lidar`) for environment perception.
- **RGB front camera** for visual sensing.
- **RGB-D / depth camera** for close-range 3D perception.
- **GPS (NavSat)** sensor for global position simulation.

It is intended for workflows such as:

- autonomous field navigation,
- mapping and localization experiments,
- perception pipeline testing,
- controller tuning in simulation before real hardware.

## Package Structure

- `urdf/agriedge.xacro`: main robot model (links, joints, sensors).
- `urdf/gz_plugins.xacro`: Gazebo visuals, wheel friction/contact, and sensor plugins.
- `urdf/ros2_control.xacro`: control interfaces for simulation and hardware mode.
- `launch/rsp.launch.py`: robot_state_publisher launch.
- `launch/display.launch.py`: RViz-only visualization launch.
- `launch/sim.launch.py`: full simulation launch (`gz-sim` + spawn + bridge + controllers).
- `config/my_controllers.yaml`: `diff_drive_controller` + `joint_state_broadcaster`.
- `config/gz_bridge.yaml`: Gazebo-to-ROS topic bridge config.
- `world/agriedge_empty.sdf`: default simulation world.

## Requirements

- ROS 2 Jazzy
- `ros_gz_sim`
- `ros_gz_bridge`
- `gz_ros2_control`
- `ros2_control`
- `ros2_controllers`
- `xacro`

## Build

From the workspace root:

```bash
source /opt/ros/jazzy/setup.bash
colcon build --packages-select agriedge
source install/setup.bash
```

## Run


### Full New Gazebo simulation

```bash
ros2 launch agriedge sim.launch.py
```

This launch starts:

- robot_state_publisher,
- New Gazebo with `agriedge_empty.sdf`,
- robot spawning from `robot_description`,
- `ros_gz_bridge` for sensors/clock,
- controller spawners (`diff_cont`, `joint_broad`),
- static TF links for scoped Gazebo sensor frames.

## Keyboard Teleop

`diff_cont` is configured for stamped velocity commands (`TwistStamped`):

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard \
  --ros-args -r /cmd_vel:=/diff_cont/cmd_vel -p stamped:=true
```

## Main ROS Topics

- `/diff_cont/cmd_vel`: velocity command input for the mobile base controller.
- `/odom`: wheel odometry output.
- `/lidar_3d/scan`: LiDAR scan stream.
- `/lidar_3d/points`: LiDAR point cloud stream.
- `/camera/image_raw`: RGB camera image.
- `/depth_camera/image`: depth camera image.
- `/depth_camera/points`: depth point cloud.
- `/gps/fix`: GPS/NavSat fix data.
- `/clock`: simulation clock.

## TF Frames

Key frames in the setup:

- `odom`
- `base_footprint`
- `base_link`
- `lidar_Link`
- `cam_Link`
- `camnav_Link`
- `GPS_Link`

## Quick Troubleshooting

- Robot does not move:
  - Check controller state (`diff_cont` must be active).
  - Confirm teleop is publishing to `/diff_cont/cmd_vel` with stamped messages.
- No model in RViz:
  - Check `Fixed Frame` (`base_link` or `odom`).
  - Ensure `robot_state_publisher` is running.
- Missing GPS data:
  - Confirm `/gps/fix` is present in `ros2 topic list`.
  - Verify `sim.launch.py` is used (includes bridge and sensor plugins).
- Simulation issues after edits:
  - Rebuild and re-source `install/setup.bash`.
