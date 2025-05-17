# ROS2-PX4 Simulation Environment for Multiple Drones

##Chinese tutorial
the chinese tutorial:(in ZhiHu app)
[Video Tutorial](https://zhuanlan.zhihu.com/p/1904187016243045984)

## Introduction

PX4, a widely popular open-source flight control system, offers significant convenience for drone design and control. ROS (Robot Operating System), with its distributed communication architecture, facilitates drone swarm simulation. Coupled with the Gazebo physics simulation engine, it provides a realistic simulation environment.

ROS has now evolved to ROS2, which offers enhanced applicability and maintainability compared to ROS1. However, there are few existing ROS2+PX4 simulation tutorials, and with frequent updates to both systems, environment setup can be challenging. This tutorial provides a comprehensive development workflow.

## Installing Ubuntu

This environment is built on Ubuntu 22.04. You can set up a virtual machine or install Ubuntu as a dual-boot system alongside Windows. For reference, here is a video tutorial for installing Ubuntu 22.04 on Windows 11:

[Video Tutorial](https://www.bilibili.com/video/BV1wo4y177Gk/?spm_id_from=333.337.search-card.all.click)

The following outlines the environment setup process. For a similar, concise setup guide, you can refer to this blog post, which is suitable for single-drone simulation needs:

[Blog Reference](https://blog.csdn.net/weixin_55944949/article/details/140627640?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522cf0372171e9ef00ff4dd9c645f32d5b5%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=cf0372171e9ef00ff4dd9c645f32d5b5&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-3-140627640-null-null.142^v102^control&utm_term=px4%20ros2%20offboard&spm=1018.2226.3001.4187)

Before proceeding, ensure a stable internet connection and the ability to access global repositories, as cloning from GitHub may fail otherwise.

## Downloading and Compiling PX4 Source Code

Clone the PX4 source code from GitHub and update submodules:

```bash
git clone https://github.com/PX4/PX4-Autopilot.git --recursive
```

To avoid potential issues with environment setup, back up the `PX4-Autopilot` directory after cloning:

```bash
zip -r PX4-Autopilot.zip PX4-Autopilot/
```

Install dependencies:

```bash
bash ./PX4-Autopilot/Tools/setup/ubuntu.sh
```

With the source code downloaded and dependencies installed, compile the code:

```bash
cd ~/PX4-Autopilot/
make px4_sitl jmavsim
```

This command compiles the PX4 source code and launches the jmavsim physics simulation engine. If the terminal displays a green "ready for takeoff" message and the jmavsim interface appears, the compilation is successful. If this step fails, verify your internet connection. Press Enter in the terminal, and at the `>pxh` prompt, input PX4 commands:

```bash
commander takeoff/land/shutdown
```

These commands control the drone's takeoff, landing, and simulation shutdown. You should see the drone's movements in the jmavsim UI. At this point, PX4 source code download and compilation are complete.

## Installing ROS2 and Dependencies

We use ROS2 Humble for development. Installing ROS2 and its dependencies can be complex, so we recommend using the "FishROS" one-click installation tool, which simplifies the process. Run the following command in the Ubuntu terminal and follow the prompts:

```bash
wget http://fishros.com/install -O fishros && . fishros
```

This tool also supports one-click installation of Docker and other utilities, which you can use as needed. For those using ROS2 Foxy, this tool also supports installation and configuration.

## Installing Gazebo Ignition

Per the PX4 official documentation, the former Gazebo Ignition is now called Gazebo, while the original Gazebo is renamed Gazebo-classic. Ubuntu 22.04 and later versions support Gazebo (Ignition). While jmavsim and Gazebo-classic were installed during the dependency setup, we need Gazebo Ignition. Run the following commands:

```bash
cd ~/PX4-Autopilot
sudo wget https://packages.osrfoundation.org/gazebo.gpg -O /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg
sudo apt-get update
sudo apt-get install gz-garden
```

This uninstalls Gazebo-classic and installs Gazebo Ignition along with the `gz sim` simulator. Then, install the Gazebo-ROS communication bridge for this version:

```bash
sudo apt install ros-humble-ros-gzgarden
```

Gazebo Ignition installation is now complete.

## Installing MicroXRCE-DDS Agent

According to the official documentation, PX4 in ROS2 uses the uXRCE-DDS middleware to publish and subscribe to uORB messages, replacing MAVROS. Follow these steps to download, compile, and install:

Clone the source code:

```bash
git clone https://github.com/eProsima/Micro-XRCE-DDS-Agent.git
```

Compile:

```bash
cd Micro-XRCE-DDS-Agent
mkdir build
cd build
cmake ..
make
```

The compilation process may take some time. After completion, install:

```bash
sudo make install
sudo ldconfig /usr/local/lib/
```

## Installing QGroundControl (QGC)

When running PX4+Gazebo simulations, a missing QGC connection can prevent PX4 from controlling the drone. Install QGC by following the steps in this blog post:

[QGC Installation Guide](https://blog.csdn.net/weixin_55944949/article/details/130895363?spm=1001.2014.3001.5502)

## Single Drone Offboard Testing and Development Overview

With the development environment set up, we now validate PX4's Offboard mode.

**Offboard mode** is a special flight mode in PX4 that allows external systems (e.g., onboard computers or ROS2 nodes) to send control commands via MAVLink or MAVROS, enabling real-time control of the drone's attitude, position, or velocity for autonomous flight or advanced tasks.

Create a ROS2 workspace:

```bash
mkdir -p ~/ros2_ws/src
```

Clone the source code (ensure a stable internet connection):

```bash
cd ~/ros2_ws/src
git clone https://github.com/PX4/px4_msgs.git -b release/1.14
git clone https://github.com/PX4/px4_ros_com.git -b release/v1.14
```

Compile:

```bash
cd ~/ros2_ws
colcon build
```

Update and configure environment variables:

```bash
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

Launch the Offboard simulation. Open QGC, then in the first terminal, enable communication:

```bash
MicroXRCEAgent udp4 -p 8888
```

In a second terminal, start the simulation:

```bash
cd ~/PX4-Autopilot
make px4_sitl gz_x500
```

QGC connects to PX4, and Gazebo launches, displaying the x500 drone. When the PX4 terminal shows a green "ready for takeoff," the simulation is successful. We will explain the `make` command for multi-drone simulations later.

In a third terminal, run the PX4 Offboard mode example:

```bash
ros2 run px4_ros_com offboard_control
```

The drone in Gazebo should ascend 5 meters, indicating successful Offboard mode control.

### Single Drone Offboard Mode Overview

In the `~/ros2_ws/px4_ros_com` directory, locate the `CMakeLists.txt` file, which includes:

```cmake
add_executable(offboard_control src/examples/offboard/offboard_control.cpp)
ament_target_dependencies(offboard_control rclcpp px4_msgs)
install(TARGETS offboard_control DESTINATION lib/${PROJECT_NAME})
```

This compiles `offboard_control.cpp` into an executable named `offboard_control`, depending on `px4_msgs`. The source code is in `src/examples/offboard/offboard_control.cpp`.

For single-drone Offboard mode simulation, follow these steps:

1. In the configured `~/ros2_ws` workspace, create a new sub-workspace parallel to `px4_ros_com`.
2. In the sub-workspace's `/src` directory, write C++/Python code for Offboard mode control.
3. In the sub-workspace's `CMakeLists.txt`, update the following lines to point to your code:

```cmake
add_executable(offboard_control src/examples/offboard/offboard_control.cpp)
ament_target_dependencies(offboard_control rclcpp px4_msgs)
install(TARGETS offboard_control DESTINATION lib/${PROJECT_NAME})
```

Replace the path with your C++/Python code and name the executable. Recompile the ROS2 workspace before simulation:

```bash
cd ~/ros2_ws
colcon build
```

4. Open QGC, enable communication, and start the simulation as before. Then, in a new terminal:

```bash
ros2 run sub_workspace_name executable_name
```

If the drone in Gazebo follows your code's logic, the simulation is successful.

**Note 1**: When using `make px4_sitl gz_x500`, the `gz sim` simulator uses the `default` world, with its SDF file located in `./PX4-Autopilot/Tools/simulation/gz/worlds`, not `~/.gz/worlds`. Models are also in `Tools/simulation`. Modify SDF files in these directories for simulation setup.

**Note 2**: When writing C++/Python code for Offboard control, account for coordinate system differences. PX4 uses the NED (North-East-Down) coordinate system (x: north, y: east, z: down, with negative altitudes). Gazebo Ignition uses the END (East-North-Down) system (x: east, y: north, z: up, with positive altitudes). Adjust control logic accordingly.

This concludes the basic workflow for single-drone ROS2+PX4 Offboard mode simulation. You can now build upon this for further single-drone development.

## Multi-Drone (Swarm) Development Workflow

Having completed the single-drone simulation environment, we now extend to multi-drone setups for swarm control logic simulation. First, recall the key command for single-drone simulation:

```bash
cd ~/PX4-Autopilot
make px4_sitl gz_x500
```

This `make` command performs three tasks: (1) compiles PX4 source code, (2) starts PX4, and (3) launches Gazebo. For multi-drone simulation, avoid repeated `make` commands, as they launch separate Gazebo instances for each drone. Instead, aim to: (1) launch Gazebo independently, (2) start PX4 instances and add drones to Gazebo, and (3) establish communication.

Download a Gazebo startup script:

```bash
wget https://raw.githubusercontent.com/PX4/PX4-gazebo-models/main/simulation-gazebo
```

Run the script:

```bash
python3 simulation-gazebo
```

The first run downloads components to `~/.simulation-gazebo` (view hidden folders with `Ctrl+H`). The `worlds/default.sdf` file in this directory is the simulation world SDF file, which you can modify later.

After launching Gazebo, add a drone without using `make`:

```bash
cd ~/PX4-Autopilot
PX4_GZ_STANDALONE=1 PX4_SYS_AUTOSTART=4001 PX4_GZ_MODEL_POSE="0,2" PX4_SIM_MODEL=gz_x500 ./build/px4_sitl_default/bin/px4 -i 0
```

Parameters:
- `PX4_GZ_STANDALONE=1`: Starts PX4 without launching Gazebo, connecting to the existing Gazebo instance.
- `PX4_SYS_AUTOSTART=4001`: Required field.
- `PX4_GZ_MODEL_POSE="0,2"`: Sets the drone's position (ensure unique positions for each drone).
- `PX4_SIM_MODEL=gz_x500`: Specifies the x500 drone model.
- `-i 0`: Assigns the drone ID 0.

After running, a drone should appear in the Gazebo instance.

Add a second drone:

```bash
PX4_GZ_STANDALONE=1 PX4_SYS_AUTOSTART=4001 PX4_GZ_MODEL_POSE="0,1" PX4_SIM_MODEL=gz_x500 ./build/px4_sitl_default/bin/px4 -i 1
```

Two drones should now appear side-by-side in Gazebo. Test their control in their respective PX4 terminals:

```bash
commander takeoff/land
```

Finally, establish communication:

```bash
MicroXRCEAgent udp4 -p 8888
```

This completes the multi-drone simulation setup.
