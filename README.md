# ros2_rover_localization

ROS 2 package for rover localization using robot_localization.

This package provides a local EKF configuration to fuse motor odometry and IMU data in order to publish a filtered odometry topic and the odom -> base_link transform.

## Features

- Fuse motor odometry and IMU data
- Publish filtered odometry on /odometry/filtered
- Publish TF odom -> base_link
- Designed for tracked or differential-drive rover platforms
- Uses robot_localization
- Configurable through YAML files

## Inputs

### Odometry

Default topic:

    /odom

Expected message type:

    nav_msgs/msg/Odometry

Expected frames:

    header.frame_id: odom
    child_frame_id: base_link

The odometry source should publish raw motor or wheel odometry.

Important: when this EKF is active, the raw odometry node should not publish the odom -> base_link TF. The EKF node will publish this transform.

### IMU

Default topic:

    /imu/raw

Expected message type:

    sensor_msgs/msg/Imu

Expected frame:

    header.frame_id: imu_link

## Outputs

### Filtered odometry

    /odometry/filtered

Type:

    nav_msgs/msg/Odometry

### TF

    odom -> base_link

This transform is published by the EKF node.

## Frame tree

Expected local frame tree:

    odom
    └── base_link
        ├── imu_link
        ├── gps_link
        └── lidar_link

Later, when GPS/global localization is added, the full frame tree can become:

    map
    └── odom
        └── base_link
            ├── imu_link
            ├── gps_link
            └── lidar_link

## Package structure

    ros2_rover_localization
    ├── config
    │   └── ekf_local.yaml
    ├── launch
    │   └── ekf_local.launch.py
    ├── ros2_rover_localization
    │   └── __init__.py
    ├── resource
    │   └── ros2_rover_localization
    ├── package.xml
    ├── setup.py
    ├── setup.cfg
    ├── README.md
    └── LICENSE

## Installation

Clone the package into your ROS 2 workspace:

    cd ~/ros2_ws/src
    git clone <your-repository-url>

Install dependencies:

    sudo apt update
    sudo apt install ros-$ROS_DISTRO-robot-localization

Build the package:

    cd ~/ros2_ws
    colcon build --packages-select ros2_rover_localization
    source install/setup.bash

## Usage

Make sure your raw odometry and IMU topics are running:

    ros2 topic list

You should see at least:

    /odom
    /imu/raw

Launch the local EKF:

    ros2 launch ros2_rover_localization ekf_local.launch.py

## Verification

Check the filtered odometry output:

    ros2 topic echo /odometry/filtered

Check the TF:

    ros2 run tf2_ros tf2_echo odom base_link

Generate the TF tree:

    ros2 run tf2_tools view_frames

Expected result:

    odom -> base_link

## RViz

Set the fixed frame to:

    odom

Recommended displays:

- TF
- Odometry /odom
- Odometry /odometry/filtered
- IMU /imu/raw
- PointCloud2 /points

## Configuration

The EKF configuration is located in:

    config/ekf_local.yaml

By default, it uses:

    odom0: /odom
    imu0: /imu/raw

If your topics are different, edit the YAML file.

## EKF behavior

The local EKF fuses:

    raw motor odometry + IMU

into:

    /odometry/filtered

and publishes:

    odom -> base_link

This setup is intended for local navigation and RViz visualization with odom as the fixed frame.

For GPS/global localization, another configuration should be added later using navsat_transform_node and a global EKF to publish:

    map -> odom

## Important notes

Only one node should publish a given TF transform.

Correct setup:

    raw odometry node:
      publishes /odom
      does not publish TF

    imu node:
      publishes /imu/raw

    ekf_node:
      subscribes to /odom and /imu/raw
      publishes /odometry/filtered
      publishes TF odom -> base_link

Incorrect setup:

    raw odometry node publishes TF odom -> base_link
    ekf_node also publishes TF odom -> base_link

This would create a TF conflict.

## Topic requirements

The /odom topic should use:

    nav_msgs/msg/Odometry

The /imu/raw topic should use:

    sensor_msgs/msg/Imu

Check topic types with:

    ros2 topic info /odom
    ros2 topic info /imu/raw

Check message content with:

    ros2 topic echo /odom --once
    ros2 topic echo /imu/raw --once

## License

MIT
