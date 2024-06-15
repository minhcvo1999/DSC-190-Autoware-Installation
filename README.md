# Minh Vo, Team 4 (Autoware), DSC 190

## ROS2 Humble Autoware Installation with Eagleye on 1/5th Car, with F1Tenth Trajectory Record/Replay guide

### Prerequisites

- Ubuntu 22.04
- ROS2 Humble
- Git

### Docker Container Setup

1. **Pull the docker with the command:**

   This is the one I chose but feel free to choose whichever
   ```
   docker pull ghcr.io/autowarefoundation/autoware-universe:humble-2024.03-arm64
   ```
2. **Add the run.sh file to the jetson and edit it to include a volume for autoware and update the image name (last line).**
   ```
   docker run \
    --name [YOUR CONTAINER NAME] \
    --runtime nvidia \
    -it \
    --privileged \
    --net=host \
    -e DISPLAY=$DISPLAY \
    -v /dev/bus/usb:/dev/bus/usb \
    --device-cgroup-rule='c 189:* rmw' \
    --device /dev/video0 \
    --volume='/dev/input:/dev/input' \
    --volume='/home/jetson/.Xauthority:/root/.Xauthority:rw' \
    --volume='/tmp/.X11-unix/:/tmp/.X11-unix' \
    --volume='/home/jetson/YOUR_PATH/Autoware:/home/jetson/YOUR_PATH/Autoware' \
    ghcr.io/autowarefoundation/autoware-universe:humble-2024.03-arm64
    ```
3. **To run the docker container use:**
   ```
   sh run.sh
   ```
4. **To reenter docker container after exit use command:**
   ```
   docker exec -it [CONTAINTER NAME] bash
   ```

### Development Environment Setup

1. **Clone autoware with this command:**
   ```
   git clone https://github.com/autowarefoundation/autoware.git
   ```
2. **Install dependencies using setup script:**
   ```
   cd autoware
   ./setup-dev-env.sh
   ```

### Autoware Workspace

1. **Create src directory:**
   ```
   mkdir src
   ```
2. **VCS dependencies from **autoware.repos** into the new src directory**
   ```
   vcs import src < autoware.repos
   ```
3. **Build the Autoware workspace**
   ```
   colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release
   ```

### GPS (PointOneNav)
- To install GPS drivers use website https://github.com/PointOneNav/fusion-engine-client

### Camera/IMU (DepthAI)
- To install camera and IMU drivers use website	https://docs.luxonis.com/software/depthai/manual-install/#Manual%20DepthAI%20installation-Installing%20dependencies-Ubuntu%2FDebian

### F1Tenth Installation
1.	**Clone the F1Tenth branch into the Jetson (outside of the autoware directory):**
```
git clone https://github.com/autowarefoundation/autoware.universe.git
```

2.	**Move the f1tenth stack into autoware/src:**
```
mv f1tenth/autoware.universe/f1tenth/ autoware/src/universe/autoware.universe/
```

3.	**Install the required packages:**
```
colcon build --packages-up-to trajectory_follower_node
colcon build --packages-up-to launch_autoware_f1tenth
colcon build --packages-up-to f1tenth_stack
colcon build --packages-select recordreplay_planner
colcon build --packages-select recordreplay_planner_nodes
```

### Launch Commands
1.	**GPS:**
```
ros2 launch ntrip_client ntrip_client_launch.py host:=polaris.pointonenav.com port:=2101  mountpoint:=POLARIS  username:=[USERNAME] password:=[PASSWORD]
```

2.	**Camera (and IMU):**
```
ros2 launch depthai_ros_driver camera.launch.py
```

3.	**Eagleye:**
```
ros2 launch eagleye_rt eagleye_rt.launch.xml
```

**Note:** eagleye_rt config will need to be updated with the right topics.

**imu:**
/oak/imu/data

**gps:**
/nmea or /rtcm

4.	**Car control**
```
ros2 launch launch_autoware_f1tenth realcar_launch.py
```

5.	**Record trajectory**
```
ros2 action send_goal /planning/recordtrajectory autoware_auto_planning_msgs/action/RecordTrajectory "{record_path: "/tmp/path"}" –feedback
```

6.	**Replay trajectory**
```
ros2 action send_goal /planning/replaytrajectory autoware_auto_planning_msgs/action/ReplayTrajectory "{replay_path: "/tmp/path"}" -feedback
```