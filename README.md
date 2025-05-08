# ROS2-PX4-simulation-environment-multiple-drones

```
git clone https://github.com/PX4/PX4-Autopilot.git --recursive
```

```
bash ./PX4-Autopilot/Tools/setup/ubuntu.sh
```

```
zip -r PX4-Autopilot.zip PX4-Autopilot/
```

```
cd ~/PX4-Autopilot/
make px4_sitl jmavsim
```

```
sudo wget https://packages.osrfoundation.org/gazebo.gpg -O /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg
```

```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null
```

```
sudo apt-get update
```

```
sudo apt-get install gz-garden
```
before starting to make in gazebo,install the Qgroundcontrol and open it to wait the connection with px4.
```
make px4_sitl gz_x500
```

```
wget https://raw.githubusercontent.com/PX4/PX4-gazebo-models/main/simulation-gazebo
```

```
python3 simulation-gazebo
```



```
PX4_GZ_STANDALONE=1 PX4_SYS_AUTOSTART=4001 PX4_GZ_MODEL_POSE="0,1" PX4_SIM_MODEL=gz_x500 ./build/px4_sitl_default/bin/px4 -i 1
```

【简介】

    PX4作为目前全世界广泛流行的开源飞控，在无人机设计和控制方面为开发人员带来便利。ROS作为机器人操作系统，其提供的分布式通信架构为无人机编队仿真带来了便利，通过Gazebo物理仿真引擎，可以提供真实的仿真环境。

   ROS目前已经更新到了ROS2，相较于ROS1，其应用性和可维护性会更加强大。目前已有的ROS2+PX4的仿真案例并不多，且伴随二者的不断更新，有很多环境配置的问题。笔者在这里提供一种开发流程。

【Ubuntu系统的安装】

    本环境基于Ubuntu 22.04系统进行搭建。大家可以搭建虚拟机或者在Windows系统中安装双系统。在这里，笔者给出一个在Windows11下安装Ubuntu 22.04双系统的视频教程链接。
```
https://www.bilibili.com/video/BV1wo4y177Gk/?spm_id_from=333.337.search-card.all.click
```

    现在开始进行环境开发流程，这里给出一个博主的环境开发流程，与我们的环境搭建较为相似，简洁明了。如果只有单机仿真需求，也可以参考这篇资料进行环境搭建。
```
https://blog.csdn.net/weixin_55944949/article/details/140627640?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522cf0372171e9ef00ff4dd9c645f32d5b5%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=cf0372171e9ef00ff4dd9c645f32d5b5&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-3-140627640-null-null.142^v102^control&utm_term=px4%20ros2%20offboard&spm=1018.2226.3001.4187
```
    在开始下列步骤之前，建议首先保证能够科学上网，且网速流畅。否则后续从Github克隆时会中途闪退。

【PX4源码下载及编译】

    首先从Github上克隆PX4源码，并且更新子模块：
```
git clone https://github.com/PX4/PX4-Autopilot.git --recursive
```

   由于后续环境配置会装很多东西，有可能难以完全清干净，建议在这一步完成后运行以下命令，备份~/PX4-Autopilot，以免后续环境配错需要重新克隆代码：
```
zip -r PX4-Autopilot.zip PX4-Autopilot/
```
    接着安装相关依赖：
```
bash ./PX4-Autopilot/Tools/setup/ubuntu.sh
```
    此时PX4的源码已经下载好并且安装好相关依赖了，我们可以试着运行以下命令进行编译：    
```
cd ~/PX4-Autopilot/
make px4_sitl jmavsim
```
    我们会发现，开始编译PX4的源码，并且开始启动jmavsim物理仿真引擎，这是因为在我们安装依赖时就已经为我们装好了jmavsim和Gazebo-classic，如果发现终端中出现绿色的“ready for takeoff”，且jmavsim出现，则说明编译成功。如果这一步失败，请检查自己的科学上网工具。现在我们在终端中回车，在>pxh后面输入PX4的指令：
```
commander takeoff/land/shutdown
```
则可以控制无人机的起飞，降落以及关闭仿真。此时可以在jmavsim的UI中看到无人机的起降。到此步，PX4的源代码下载和编译全部完成。

【ROS2及其相关依赖的安装】

    我们使用ROS2 Humble进行开发，ROS2及其依赖的安装和环境配置有些繁琐，在此我建议各位使用“鱼香ROS”的一键安装功能，感谢其为ROS开发提供的便利。直接在Ubuntu的系统终端输入以下命令并按指令安装即可：
```
wget http://fishros.com/install -O fishros && . fishros
```
    其一键安装工具中还有Docker等工具的一键安装选项，大家可以按需使用，再次感谢。另外，使用ROS2 Foxy版本进行开发的朋友也可以使用该工具进行安装和环境配置。

【Gazebo Ignition的安装】

    根据PX4的官方文档，之前的Gazebo Ignition命名为Gazebo，以前的 Gazebo现在叫Gazebo-classic，而Ubuntu22.04及以后的版本就支持 Gazebo(Gazebo Ignition) ，在之前安装依赖时，自动安装了

jmavsim和Gazebo-classic，但我们是要使用的是Gazebo Ignition，因此在终端中采用以下命令：
```
cd ~/PX4-Autopilot
sudo wget https://packages.osrfoundation.org/gazebo.gpg -O /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg
sudo apt-get update
sudo apt-get install gz-garden
```
    此时系统会自动卸载Gazebo-classic，同时安装Gazebo Ignition以及gazebo的仿真器gz sim，再使用以下命令安装此版本的Gazebo-ROS的通信桥：
```
sudo apt install ros-humble-ros-gzgarden
```
至此，我们需要的Gazebo Ignition完成安装。 

【MicroXRCE-DDS Agent的安装】

    根据官方文档说明，

在ROS2中PX4使用uXRCE-DDS中间件来允许在配套计算机上发布和订阅uORB消息，而不再使用MAVROS，我们按照官方案例进行下载、编译及安装。

    下载源码：
```
git clone https://github.com/eProsima/Micro-XRCE-DDS-Agent.git
```
     编译：
```
cd Micro-XRCE-DDS-Agent
mkdir build
cd build
cmake ..
make  
```
    编译的过程会较长，需要进行一定等待，编译结束后进行安装：
```
sudo make install
sudo ldconfig /usr/local/lib/ 
```
【QGC地面站的安装】

    在后续启动PX4+gazebo仿真时，如果丢失地面站的连接，会导致无法启动PX4飞控对无人机进行控制，因此需要在系统中安装QGC地面站。安装的方法我给出一个CSDN博主的链接，按照上面的步骤操作即可：
```
https://blog.csdn.net/weixin_55944949/article/details/130895363?spm=1001.2014.3001.5502
```
【单机Offboard测试流程以及开发简介】

  现在所有的开发环境已经配置好了，我们现在进行PX4的Offboard模式的验证。

  Offboard模式

是PX4中的一种特殊飞行模式，允许外部系统（如机载计算机、ROS2节点等）通过MAVLink或MAVROS

直接发送控制指令，实时控制无人机的姿态、位置或速度，从而实现自主飞行或高级任务。

首先创建ROS2的工作空间：
```
mkdir -p ~/ros2_ws/src
```
随后下载源码（注意科学上网）：
```
cd ~/ros2_ws/src
git clone https://github.com/PX4/px4_msgs.git -b release/1.14
git clone https://github.com/PX4/px4_ros_com.git -b release/v1.14
```
进行编译：
```
cd ~/ros2_ws
colcon build
```
更新环境并配置环境变量：
```
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc 
source ~/.bashrc 
```
现在我们启动Offboard 仿真，首先打开已经装好的QGC地面站，打开第一个终端，运行以下命令，打开通信：
```
MicroXRCEAgent udp4 -p 8888
```
紧接着打开第二个终端，启动仿真：
```
cd ~/PX4_Autopilot
make px4_sitl gz_x500
```
此时地面站会与PX4进行连接，Gazebo启动，出现了x500的无人机。当PX4终端出现绿色的“ready for takeoff”时，说明仿真启动成功。有关这个使用“make"命令启动仿真的环节，我们在后续进行多无人机仿真时会详细解释。

现在我们打开第三个终端，运行下载好的PX4的Offboard模式源码：
```
ros2 run px4_ros_com offboard_control
```
我们可以发现Gazebo中无人机上升5米，Offboard模式控制成功。现在介绍一下单机Offboard模式简介。

我们在刚刚创建并下载好源码的~/ros2_ws里找到px4_ros_com目录下的CMakelist.txt文件，找到这一行：
```
add_executable(offboard_control src/examples/offboard/offboard_control.cpp)
ament_target_dependencies(offboard_control rclcpp px4_msgs)
install(TARGETS offboard_control DESTINATION lib/${PROJECT_NAME})
```
这是将src/examples/offboard中的offboard_control.cpp文件编译成可执行文件并命名为offboard_control，同时依赖来源于px4_msgs，我们可以打开

src/examples/offboard目录便可以找到

offboard_control.cpp源代码。

如果进行单机使用Offboard模式进行仿真，便可以按照以下流程：

在已经创建并且配置好环境的~/ros2_ws这个ros2工作空间下再创建一个工作空间（和px4_ros_com目录并行）

在新创建的子工作空间中，在/src目录下编写基于Offboard模式进行控制的c++/python代码

在新创建的子工作空间中的CMakelist.txt中找到以下几行：
```
add_executable(offboard_control src/examples/offboard/offboard_control.cpp)
ament_target_dependencies(offboard_control rclcpp px4_msgs)
install(TARGETS offboard_control DESTINATION lib/${PROJECT_NAME})
```
将路径改为c++/python代码的地址，并给他命名一个可执行文件的名字。在进行仿真前，一定要重新编译ROS2工作空间！
```
cd ~/ros2_ws
colcon build
```
    最后，仿照之前的方法打开QGC地面站、通信以及启动仿真，随后再打开一个新的终端：
```
ros2 run 子工作空间名称 可执行文件名称
```
    如果在Gazebo中看见无人机按照代码逻辑进行飞行，则仿真成功。

补充1：使用make px4_sitl gz_x500命令启动仿真时，自动启动的gz sim仿真器的世界为default，其sdf文件不再在~/.gz/worlds里，而是在./PX4-Autopilot/Tools/simulation/gz/worlds中，同理，所有的模型也均在Tools/simulation里，因此，在搭建仿真环境时，应该去到这些文件夹中直接修改sdf文件。

补充2：在编写c++/python代码进行offboard模式控制时，需要注意坐标系的变换。在PX4中，坐标系为NED坐标系，即North-East-Down，因此x坐标指向北，y坐标指向东，z坐标指向下（所有的高度为负值）；而在Gazebo Ignition中，坐标系为END坐标系，此时x坐标指向东，y坐标指向北，z坐标指向上（高度为正值）。整理控制逻辑时需要注意坐标的换算。

至此，ROS2+PX4在Offboard模式下进行单机仿真的基本流程就全部介绍完了。大家如果成功地跑通以上流程，接下来就可以基于此进行更多的单机仿真开发了。

【多机（编队）的开发流程】
在前面的部分，我们完成了基于ROS2+PX4的单机仿真环境开发。现在我们开始进行多机的环境开发，为后续编队的控制逻辑仿真进行预先准备。现在我们先回顾一下我们启动单机仿真一个最主要的指令：
```
cd ~/PX4-Autopilot
make px4_sitl gz_x500
```
这个make指令其实一共完成了三件事G：1-编译PX4源代码 2-启动PX4 3-启动Gazebo仿真，但在多机仿真中，我们不需要反复的使用make指令，这样会导致每一架x500无人机单独启动了一个Gazebo仿真，而这不是我们想要的。因此在Gazebo环境中启动多机仿真，我们的目标为以下几个：1-单独启动Gazebo仿真 2-分别启动PX4，添加无人机进入Gazebo环境 3-建立通信
现在我们尝试单独启动一下Gazebo仿真，首先我们需要下载一个Gazebo的启动脚本(simulation-gazebo)：
```
wget https://raw.githubusercontent.com/PX4/PX4-gazebo-models/main/simulation-gazebo
```
随后在脚本所在的目录终端运行以下指令：
```
python3 simulation-gazebo
```
首次运行脚本后，会下载一些组建放在.simulation-gazebo下（在主目录里用Ctrl+H打开隐藏文件夹），其中的/worlds/default.sdf即为仿真的世界sdf文件，后续在这个文件里进行修改。

在单独启动了Gazebo环境后，我们往其中添加无人机，此时我们不再使用make命令，而是用以下指令：
```
cd ~/PX4-Autopilot
PX4_GZ_STANDALONE=1 PX4_SYS_AUTOSTART=4001 PX4_GZ_MODEL_POSE="0,2" PX4_SIM_MODEL=gz_x500 ./build/px4_sitl_default/bin/px4 -i 0
```
其中有四个参数，PX4_GZ_STANDALONE=1表明我们使用standalone启动（不启动仿真，只启动PX4，等待单独启动的仿真，当我们前面已经单独启动了Gazebo环境之后，会自动在仿真环境中添加飞机），PX4_SYS_AUTOSTART=4001为必须字段，PX4_GZ_MODEL_POSE="0,2"表示无人机的位置，注意不同的飞机位置应该不一样，PX4_SIM_MODEL=gz_x500指定了我们的飞机模型为x500，同时，我们需要为多机仿真的每一架无人机给定一个单独的编号，这里为 -i 0，即第0号无人机。运行该指令后，我们应该可以在我们之前单独打开的Gazebo里看见出现了一架飞机。

现在我们再添加第二架飞机：
```
PX4_GZ_STANDALONE=1 PX4_SYS_AUTOSTART=4001 PX4_GZ_MODEL_POSE="0,1" PX4_SIM_MODEL=gz_x500 ./build/px4_sitl_default/bin/px4 -i 1
```
现在我们会发现在Gazebo里出现了两架并排的无人机，可以分别在他们启动PX4后的终端输入：
```
commander takeoff/land
```
可以分别观察他们的起降情况。最后添加通信：
```
MicroXRCEAgent udp4 -p 8888
```
