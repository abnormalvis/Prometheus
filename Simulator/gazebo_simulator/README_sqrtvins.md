# Prometheus Gazebo + SqrtVINS with external PX4

This integration keeps the Prometheus Gazebo world, P450_D435i model, simulated camera and IMU, and ROS 1 MAVROS/VIO stack. PX4 is started as an external PX4 v1.14 SITL process; the ROS launch does not require the Prometheus `px4` ROS package.

## Environment

Use a ROS Noetic shell. The PX4 v1.14 checkout in this workspace has Gazebo Classic plugins built at `build/px4_sitl_default/build_gazebo-classic`. The Prometheus P450 model contains the Gazebo MAVLink interface plugin and listens on TCP 4560. MAVROS uses UDP 14540/14580 for vehicle 1.

In both terminals, source the PX4 Gazebo environment and expose the Prometheus and SqrtVINS source packages. Replace paths if the workspace is elsewhere:

```bash
source /opt/ros/noetic/setup.bash
export UAV_WS=/home/idris-24-04/uav_ws
export PX4_ROOT=/home/idris-24-04/px4_ws/PX4-Autopilot
export PX4_BUILD="$PX4_ROOT/build/px4_sitl_default"
export PROMETHEUS_ROOT="$UAV_WS/src/rmUAV/rm_drone_ros/src/Prometheus"
export SQRTVINS_ROOT="$UAV_WS/src/rmUAV/rm_drone_ros/src/sqrtVINS"
source "$PX4_ROOT/Tools/simulation/gazebo-classic/setup_gazebo.bash" "$PX4_ROOT" "$PX4_BUILD"
export GAZEBO_MODEL_PATH="$PROMETHEUS_ROOT/Simulator/gazebo_simulator/gazebo_models/uav_models:$PROMETHEUS_ROOT/Simulator/gazebo_simulator/gazebo_models/sensor_models:$GAZEBO_MODEL_PATH"
export ROS_PACKAGE_PATH="$PROMETHEUS_ROOT/Modules:$PROMETHEUS_ROOT/Simulator:$SQRTVINS_ROOT:$ROS_PACKAGE_PATH"
source "$UAV_WS/src/rmUAV/rm_drone_ros/devel/setup.bash"
export GAZEBO_PLUGIN_PATH="$UAV_WS/src/rmUAV/rm_drone_ros/devel/lib:$GAZEBO_PLUGIN_PATH"
```

## Start

Terminal 1 starts Gazebo with the Prometheus P450_D435i model, MAVROS, SqrtVINS, the Prometheus controller and pose adapter. The `start_px4:=false` setting deliberately skips the ROS `px4` package node:

```bash
roslaunch prometheus_gazebo sqrtvins_sitl.launch start_px4:=false
```

Terminal 2 starts the external PX4 process in the same shell environment, so it can load the Gazebo Classic plugin paths. The installed PX4 tree has an Iris airframe, but no P450 airframe:

```bash
cd "$PX4_BUILD"
PX4_SIM_MODEL=gazebo-classic_iris PX4_SIM_SPEED_FACTOR=1.0 \
  ./bin/px4 -s etc/init.d-posix/rcS -i 0 -w sitl_prometheus_0
```

This Iris/P450 combination is only for checking whether the external PX4 process connects to the Prometheus Gazebo MAVLink plugin and whether camera/IMU topics flow. Iris rotor geometry and control allocation do not match P450. Do not arm, take off, or use autonomous control until a P450-matched PX4 v1.14 airframe has been configured and validated.

If `prometheus_uav_control` has not been built yet, validate the simulator and VIO pipeline without it:

```bash
roslaunch prometheus_gazebo sqrtvins_sitl.launch \
  start_px4:=false start_uav_control:=false joy_enable:=false
```

## Check

```bash
rostopic hz /uav1/camera/color/image_raw
rostopic hz /uav1/mavros/imu/data_raw
rostopic echo -n 1 /uav1/mavros/state
rostopic hz /ov_srvins/odomimu
rostopic hz /vins_estimator/imu_propagate
```

Expected topics are the Prometheus simulated D435i color image, MAVROS raw IMU, SqrtVINS odometry, and the adapted Prometheus vision pose. If PX4 does not connect, check that Gazebo loaded `libgazebo_mavlink_interface.so`, the simulator TCP port is 4560, and PX4 was started with the same `PX4_BUILD` environment.

SqrtVINS calibration values in `sqrtVINS/config/gazebo_p450` are initial estimates and still require comparison against Gazebo ground truth. This setup is a simulation integration, not a flight-ready calibration or airframe configuration.