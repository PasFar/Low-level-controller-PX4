# PX4 Low Level Offboard Control using ROS 2

## Setup Instructions

1. **Clone the support utilities repository:**

   ```bash
   git clone https://github.com/Prisma-Drone-Team/sitl_utils.git
   ```

2. **Clone the PX4 Autopilot repository:**

   ```bash
   cd sitl_utils
   git clone https://github.com/PX4/PX4-Autopilot.git --recursive
   cd PX4-Autopilot
   git checkout v1.15.4 --f
   git submodule update --recursive
   ```

3. **Clone the custom PX4 offboard controller:**

   ```bash
   cd ..
   mkdir -p ros2_ws-src/pkg
   cd ros2_ws-src/pkg
   git clone https://github.com/PasFar/px4_offboard_lowlevel.git
   ```

4. **Set up custom simulation assets and configuration:**

   Replace `/PATH/TO/` with the absolute path to your `sitl_utils` folder:

   ```bash
   cp /PATH/TO/sitl_utils/ros2_ws-src/pkg/px4_offboard_lowlevel/px4-resources/px4_humble_dockerfile-gazebo-classic_FSR.txt /PATH/TO/sitl_utils/docker/
   cp /PATH/TO/sitl_utils/ros2_ws-src/pkg/px4_offboard_lowlevel/px4-resources/leo_race_field /PATH/TO/sitl_utils/PX4-Autopilot/Tools/simulation/gazebo-classic/sitl_gazebo-classic/models/ -r
   cp /PATH/TO/sitl_utils/ros2_ws-src/pkg/px4_offboard_lowlevel/px4-resources/empty.world /PATH/TO/sitl_utils/PX4-Autopilot/Tools/simulation/gazebo-classic/sitl_gazebo-classic/worlds/
   cp /PATH/TO/sitl_utils/ros2_ws-src/pkg/px4_offboard_lowlevel/px4-resources/iris /PATH/TO/PX4-Autopilot/Tools/simulation/gazebo-classic/sitl_gazebo-classic/models/ -r
   
   ```

5. **Build the Docker image:**

   ```bash
   cd /PATH/TO/sitl_utils/docker
   docker build -t leo-img -f px4_humble_dockerfile-gazebo-classic_FSR.txt .
   ```

6. **Run the Docker container:**

   ```bash
   cd /PATH/TO/sitl_utils
   ./run_cnt.sh
   ```

7. **Setup PX4 developmnent enviromnent**

    ```bash
   bash ./PX4-Autopilot/Tools/setup/ubuntu.sh
   ```
   

---

## Simulation Instructions

Open **five separate terminals** inside the running Docker container and follow these steps:

### Terminal 1 - Launch PX4 Simulation

Launch PX4 with Gazebo Classic and the custom world:

```bash
cd PX4-Autopilot
make px4_sitl gazebo-classic
```

After the controller is launched, remember to arm and set the offboard mode in this terminal.

### Terminal 2 - Start the microRTPS Agent

Start the communication bridge between ROS 2 and PX4:

```bash
MicroXRCEAgent udp4 -p 8888
```

### Terminal 3 - Launch the Offboard Controller

Compile the custom controller and launch it:

```bash
cd ros2_ws
. install/setup.bash
colcon build --packages-select px4_msgs px4_offboard_lowlevel
. install/setup.bash
ros2 launch px4_offboard_lowlevel iris_sitl.launch.py
```


### Terminal 4 - Set Controller Parameters

Change the control mode to thrust-and-torque mode:

```bash
ros2 param set /offboard_controller control_mode 2
```

This terminal can also be used to tune gains and switch controller types. 

```bash
ros2 param set /offboard_controller control_gains.controller_type geometric
```

### Terminal 5 - Launch a Trajectory

Source the workspace:

```bash
cd ros2_ws
. install/setup.bash
```

Then, run a trajectory:

* **Circular trajectory:**

  ```bash
  ros2 run px4_offboard_lowlevel circle_trajectory_node
  ```

* **Bernoulli's Lemniscate trajectory:**

  ```bash
  ros2 run px4_offboard_lowlevel lemniscate_trajectory_node
  ```

### Troubleshooting

While running the simulation on PX4 and gazebo, if gazebo is not found, execute the following:
```bash
sudo apt remove gz-garden -y
sudo apt-get update -y
sudo aptitude install -y gazebo libgazebo11 libgazebo-dev
```

If the models of the custom world are not found, in a terminal:
```bash
export GAZEBO_MODEL_PATH="/root/PX4-Autopilot/Tools/simulation/gazebo-classic/sitl_gazebo-classic/models/leo_race_field:/root/PX4-Autopilot/Tools/simulation/gazebo-classic/sitl_gazebo-classic/models:$GAZEBO_MODEL_PATH"
source ~/.bashrc
```
