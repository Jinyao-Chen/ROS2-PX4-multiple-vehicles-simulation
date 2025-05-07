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

