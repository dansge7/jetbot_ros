# Project Seminar ris | Technical University of Darmstadt

This repo is originally a fork from dusty-nv/jetbot_ros that was extended to serve as starting base for the students in the project seminar
https://www.etit.tu-darmstadt.de/ris/lehre_ris/lehrveranstaltungen_ris/pro_robo_ris/index.de.jsp


# Basic Knowledge (all of you should know)

Before starting, some basic knowledge should be available. In case you are not familiar with the following, read or watch some tutorials:
- some basics in Linux (you will use Ubuntu 18.04)
    - basic console commands `cd`, `ls`, `mkdir`, `source`, `cp`, `mv`, `rm`, `chmod`, ...
    - the purpose of `sudo`
    - the purpose of `apt-get`
- some basics in C/C++
    - C/C++ compiling procedure including the purpose of `cmake`, `make` and `CmakeLists.txt`
- some basics in Python
    - the purpose of `pip`
- the purpose of Git as well as basic commands `commit`, `push`, `pull`, `clone`, `fork`, ...
- ROS (you will use the ROS1 version `melodic`)
    - *Note: How ROS is installed on the JetBot will be explained further below*
    - do the tutorials at http://wiki.ros.org/ROS/Tutorials
    - RVIZ
    - rqt (e.g. `rqt_graph` )

# Distributed Knowledge (at least one person per group should know)

At least one group participant should be familiar with:
- basic image processing routines
    - pinhole model: http://wiki.ros.org/image_pipeline/CameraInfo
    - camera calibration: http://wiki.ros.org/camera_calibration/Tutorials/MonocularCalibration
    - rectification: http://wiki.ros.org/image_proc

- coordinate systems and transforms in ROS
    - the helpful tool rqt_tf_tree

- the purpose of AprilTags
    - https://github.com/AprilRobotics/apriltag
    - *there are also some papers ...*


# Getting started

In the following, you are going to set up the basic ROS environment on the JetBot so that the robot will be able to localize itself visually (only with the help of its camera) within a given arena. 

## The arena

The size of the arena is 1.485m x 1.485m, which equals the length of 5 sheets of DIN A4 paper. Each side of the arena is built by five sheets. The sheets of paper are equipped with recursive AprilTags that are used by the JetBot to localize itself. The sheets of paper in PDF format are located in the folder `arena` and are supposed to be printed on DIN A4 sheets of paper.
The global coordinate system's origin is set in one corner of the arena.

There are differently colored bases distribued in the arena that are either located in the arena's corners or in the middle of one of the arena's sides. All bases can be printed on DIN A4 sheets of paper and cut by a pair of scissors along the dotted line to their final shape of 21cm x 21cm. The colored circles are centered in the base squares and have a radius of 6cm. The Start - Goal - circle is centered in its base square and measures 7cm in radius. The sheet of paper with the bases in PDF format is located in the folder `arena`.

![Arena layout](https://github.com/NikHoh/jetbot_ros/blob/melodic/images/arena_setup.png)

## The robot


- flash your Nano's SD card with NVIDIA's JetPack image - see the [Getting Started](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit) guide.

> **Note**:  the process below will likely exceed the disk capacity of a 16GB filesystem,  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; so a larger SD card should be used.  If using the 'Etcher' method with JetPack-L4T image,  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; the APP partition will automatically be resized to fill the SD card upon first booting the system.  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Otherwise flash with L4T using the -S option (example given for 64GB SD card):  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `sudo ./flash.sh -S 58GiB jetson-nano-sd mmcblk0p1` 

- connect the JetBot to power, mouse, keyboard and a monitor

- you should now be able to direclty boot Ubuntu 18.04 on the JetBot (Attention: the following instruction only work on Ubuntu 18.04)

- during the following guide you will create a file structure that looks like:  


### File structure
```
|-- workspace
  |-- jetson-inference
  |   |-- ```
  |-- catkin_ws
  |   |-- build
  |   |   |-- ```
  |   |-- devel
  |   |   |-- ```
  |   |-- logs
  |   |   |-- ```
  |   |-- src
  |   |   |-- jetbot_ros (*this repo*)
  |   |   |   |-- gazebo
  |   |   |   |-- launch
  |   |   |   |-- scripts
  |   |   |   |-- src
  |   |   |   |-- CMakeLists.txt
  |   |   |   |-- ost.yaml
  |   |   |   |-- package.xml
  |   |   |   |-- README.md
  |   |   |-- ros_deep_learning
  |   |   |-- apriltag
  |   |   |-- apriltag_ros

```

Carefully follow the the following instructions. **IMPORTANT: Don't copy paste commands blindfold. Try to understand what's the purpose of the command and also read what happens in the console output.** In case of some errors:
```python
def in_case_of_error():
    if an_error_occured_before_that_you_missed:
        in_case_of_error()
    
    read_the_error_message # very important!!!
    
    if you_can_find_out_where_the_error_originates_from:
        solve_the_error()
        return

    for _ in range(5):
        if you_can_google_the_error_and_find_some_help_in_an_online_forum:
            solve_the_error()
            return 

    post_a_message_in_the_moodle_forum()

    if get_help_from_moodle_forum:
        solve_the_error()
        return
    else: 
        solve_the_error()
        write_answer_to_moodle_forum()
        return
```

### Install catkin_tools 
(https://catkin-tools.readthedocs.io/en/latest/installing.html)
```bash
sudo sh \
    -c 'echo "deb http://packages.ros.org/ros/ubuntu `lsb_release -sc` main" \
        > /etc/apt/sources.list.d/ros-latest.list'
wget http://packages.ros.org/ros.key -O - | sudo apt-key add -
sudo apt-get update
sudo apt-get install python3-catkin-tools
```

### Install git
```bash
sudo apt install git
```

### Install ROS Melodic

```bash
# enable all Ubuntu packages:
sudo apt-add-repository universe
sudo apt-add-repository multiverse
sudo apt-add-repository restricted

# add ROS repository to apt sources
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

# install ROS Base
sudo apt-get update
sudo apt-get install ros-melodic-ros-base

# add ROS paths to environment
sudo sh -c 'echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc'
```

Close and restart the terminal.

### Installing some other packages
```bash
# http://wiki.ros.org/rosdep
sudo apt-get install python-rosdep
sudo rosdep init
rosdep update

# http://wiki.ros.org/image_common
sudo apt-get install ros-melodic-image-common

# http://wiki.ros.org/image_pipeline
sudo apt-get install ros-melodic-image-pipeline

# http://wiki.ros.org/rviz
sudo apt-get install ros-melodic-rviz

# http://wiki.ros.org/rqt
sudo apt-get install ros-melodic-rqt ros-melodic-rqt-common-plugins ros-melodic-rqt-robot-plugins
```


### Install Adafruit Libraries

These Python libraries from Adafruit support the TB6612/PCA9685 motor drivers and the SSD1306 debug OLED:

```bash
# pip should be installed
sudo apt-get install python-pip

# install Adafruit libraries
pip install Adafruit-MotorHAT
pip install Adafruit-SSD1306
```

Grant your user access to the i2c bus:

```bash
sudo usermod -aG i2c $USER
```

Reboot the system for the changes to take effect.

### Create catkin workspace

Create a ROS Catkin workspace to contain our ROS packages:

```bash
# create the catkin workspace
mkdir -p ~/workspace/catkin_ws/src
cd ~/workspace/catkin_ws
catkin init
catkin build

# add catkin_ws path to bashrc
sudo sh -c 'echo "source ~/workspace/catkin_ws/devel/setup.bash" >> ~/.bashrc'

```
> Note:  out of personal preference, the catkin_ws is created as a subdirectory under ~/workspace

Close and open a new terminal window.
Verify that your catkin_ws is visible to ROS:
```bash
echo $ROS_PACKAGE_PATH 
/home/nvidia/workspace/catkin_ws/src:/opt/ros/melodic/share
```

### Build jetson-inference

Clone and build the [`jetson-inference`](https://github.com/dusty-nv/jetson-inference) repo:

```bash
# git and cmake should be installed
sudo apt-get install git cmake

# clone the repo and submodules
cd ~/workspace
git clone https://github.com/dusty-nv/jetson-inference -b nvmm-disabled
cd jetson-inference
git submodule update --init

# build from source
mkdir build
cd build
cmake ../  # during this command a dialog window will open, you can proceed with the preselected neural networks, select pytorch (maybe you will need this later)
make

# install libraries
sudo make install
```

### Build ros_deep_learning

Clone and build the [`ros_deep_learning`](https://github.com/dusty-nv/ros_deep_learning) repo:

```bash
# install dependencies
sudo apt-get install ros-melodic-vision-msgs ros-melodic-image-transport ros-melodic-image-publisher

# clone the repo
cd ~/workspace/catkin_ws/src
git clone https://github.com/dusty-nv/ros_deep_learning

# make ros_deep_learning
cd ../    # cd ~/workspace/catkin_ws
catkin build

# confirm that the package can be found
rospack find ros_deep_learning
/home/nvidia/workspace/catkin_ws/src/ros_deep_learning
```

### Build jetbot_ros

Clone and build the [`jetbot_ros`](https://github.com/dusty-nv/jetbot_ros) repo:

```bash
# clone the repo (this repo)
cd ~/workspace/catkin_ws/src
git clone https://github.com/NikHoh/jetbot_ros.git

# build the package
cd ../    # cd ~/workspace/catkin_ws
catkin build

# confirm that jetbot_ros package can be found
rospack find jetbot_ros
/home/nvidia/workspace/catkin_ws/src/jetbot_ros
```

### Testing JetBot (Part I)

Next, let's check that the different components of the robot are working under ROS.

First open a new terminal, and start `roscore`
```bash
roscore
```

#### Running the Motors

Open a new terminal, and start the `jetbot_motors` node:
```bash
rosrun jetbot_ros jetbot_motors.py
```

The `jetbot_motors` node will listen on the following topics:
* `/jetbot_motors/cmd_dir`     relative heading (degree `[-180.0, 180.0]`, speed `[-1.0, 1.0]`)
* `/jetbot_motors/cmd_raw`     raw L/R motor commands  (speed `[-1.0, 1.0]`, speed `[-1.0, 1.0]`)
* `/jetbot_motors/cmd_str`     simple string commands (left/right/forward/backward/stop)

> Note:  currently only `cmd_str` method is implemented.


#### Test Motor Commands
Open a new terminal, and run some test commands:
(it is recommended to initially test with JetBot up on blocks, wheels not touching the ground)  

```bash
rostopic pub /jetbot_motors/cmd_str std_msgs/String --once "forward"
rostopic pub /jetbot_motors/cmd_str std_msgs/String --once "backward"
rostopic pub /jetbot_motors/cmd_str std_msgs/String --once "left"
rostopic pub /jetbot_motors/cmd_str std_msgs/String --once "right"
rostopic pub /jetbot_motors/cmd_str std_msgs/String --once "stop"
```
Terminate the jetbot_motors node by hitting Strg+C in the respective console window.


#### Using the Debug OLED

If you have an SSD1306 debug OLED on your JetBot, you can run the `jetbot_oled` node to display system information and user-defined text:

```bash
rosrun jetbot_ros jetbot_oled.py
```

By default, `jetbot_oled` will refresh the display every second with the latest memory usage, disk space, and IP addresses.

The node will also listen on the `/jetbot_oled/user_text` topic to recieve string messages from the user that it will display:

```bash
rostopic pub /jetbot_oled/user_text std_msgs/String --once "HELLO!"
```

#### Using the Keyboard control

(it is recommended to initially test with JetBot up on blocks, wheels not touching the ground)  
Open a console and start a motor controller that listens to a `/cmd_vel` topic by
```bash
rosrun jetbot_ros motors_waveshare.py
```

Next, in another console start a node that publishes `/cmd_vel` messages by pressind the W A S D X keys. 
W: positive linear velocity increment
A: negative angular velocity increment
S: stop all velocities
D: positive angular velocity increment
X: negative linear velocity increment
```bash
rosrun jetbot_ros teleop_keyboard.py
```
In the active console by pressing the WASDX keys the JetBot now move its wheels accordingly.

#### Using the Camera

To begin streaming the JetBot camera, start the `jetbot_camera` node:

```bash
rosrun jetbot_ros jetbot_camera
```

The video frames will be published to the `/camera/image_raw` topic as [`sensor_msgs::Image`](http://docs.ros.org/melodic/api/sensor_msgs/html/msg/Image.html) messages with BGR8 encoding.  To test the camera feed, install the [`image_view`](http://wiki.ros.org/image_view?distro=melodic) package and then subscribe to `/camera/image_raw` from a new terminal:

```bash
# first open a new terminal
sudo apt-get install ros-melodic-image-view
rosrun image_view image_view image:=/camera/image_raw
```

A window should then open displaying the live video from the camera.  By default, the window may appear smaller than the video feed.  Click on the terminal or maximize button on the window to enlarge the window to show the entire frame.


### Build apriltag

Follow the instructions from:

https://github.com/AprilRobotics/apriltag_ros

Make sure that you don't copy/paste comments from the section "Quickstart" blindfold. Use the correct `src` folder (see the file structure above).

#### Test apriltag

More information about apriltag_ros: http://wiki.ros.org/apriltag_ros

Have a look at the tutorial to get an idea of what is happening: http://wiki.ros.org/apriltag_ros/Tutorials/Detection%20in%20a%20video%20stream

To work properly, apriltag_ros needs:
- a **calibrated camera**
    - a default camera calibration is already available in the repo (ost.yaml)
    - **BUT**: every camera behaves differently, so you should calibrate your camera yourselves following
    - http://library.isr.ist.utl.pt/docs/roswiki/camera_calibration(2f)Tutorials(2f)MonocularCalibration.html
    - Hint: large checkerboards for calibration are available at the institute, you can use them there (write an e-mail before)
    - Hint: make sure the camera node is running and publishing image_raw and camera_info topics
    - replace the existing ost.yaml
    - **Important**: 
        - change line 75 in `src/jetbot_camera.cpp` to match the correct path
        - rebuild the catkin_ws: `catkin build`
- a **rectified image**
    - the image `image_raw` can be rectified with the help of the ROS image_proc library: http://wiki.ros.org/image_proc?distro=melodic
    - the respective node is already included in the launch file, which is described next
    - you don't have to do something here but should understand how image_proc is integrated into the launch file
- a settings.yaml and tags.yaml configuration set
    - you can find them both in the folder `apriltag_ros_config` in this repo
    - **Important**: copy them to the correct position at `workspace/catkin_ws/src/apriltag_ros/apriltag_ros/config`

### Testing JetBot (Part II)

Now everything should be ready to test the localization of the JetBot in the arena. 

All needed ROS nodes are started with a single launch file: `/launch/visual_localization.launch`

(This will also launch the keyboard control. To avoid this comment out the according lines in the launch file.)

```bash
# in a terminal window
roscore

# in another terminal window
roslaunch jetbot_ros visual_localization.launch

# in another terminal window, test if all topics and nodes work
rostopic list
rosnode list

# visualize the results with RVIZ
# in another terminal window, run
rviz
```

In the appearing RVIZ window you can add the an image view to view the `tag_detections_image` topic and adding a `TF` visualization. Holding one of the arena's wall elements in front of the camera, there should now be tags detected, and a respective coordinate frame should appear. 

![RVIZ window](https://github.com/NikHoh/jetbot_ros/blob/melodic/images/rviz)

Helpful tools for your further work with ROS are:
- rqt 
    - run `rqt_graph` to visualize how nodes interact with each other
    ![rqt graph](https://github.com/NikHoh/jetbot_ros/blob/melodic/images/rqt_graph.png)

    - run `rosrun rqt_tf_tree rqt_tf_tree` to see current TFs
        - if an apriltag is in the image, there should appear a TF from the camera frame `csi://0` to the arena frame `arena`, which is the arena's origin
        ![rqt tf tree](https://github.com/NikHoh/jetbot_ros/blob/melodic/images/rqt_tf_tree.png)


Close everything by hitting Strg+C in the console where you launched the launch file.



## Not tested but recommended: JetBot Model for Gazebo Robotics Simulator

<img src="https://github.com/dusty-nv/jetbot_ros/raw/master/gazebo/jetbot_gazebo_0.png" width="700">

See the [`gazebo`](gazebo) directory of the repo for instructions on loading the JetBot simulator model for Gazebo.

For example, you could build the arena in gazebo (PDF files of the arena walls available in the folder `arena`), simulate the camera there, and set up the localization there as well. Then you would be able to do further developments solely in the simulation.

# Things you could/should do next

- install Ubuntu 18.04 and ROS melodic on another computer (virtual maschine possible) and try to connect the robot the the computer via ROS:
    - http://wiki.ros.org/ROS/Tutorials/MultipleMachines
    - Hints:
        - both machines must run Ubuntu 18.04 
        - check that openssh-client and openssh-server are installed on both devices (https://www.cyberciti.biz/faq/how-to-install-ssh-on-ubuntu-linux-using-apt-get/)
        - check with `sudo service ssh status` on both devices (https://kinsta.com/knowledgebase/ssh-connection-refused/)
        - if JetBot and your PC are connectd to the same network and you have retrieved the IP adresses of both devices for example as:
            - JetBot: `172.25.1.217` and
            - PC: `172.25.1.108`
        - you should then be able to ssh from your PC to the JetBot by `ssh jetbot@172.25.1.217`

        - for more troubleshotting this may be helpful: 
            - http://wiki.ros.org/ROS/NetworkSetup
            - http://wiki.ros.org/ROS/Troubleshooting
    - then you can test the JetBot without using its GUI

- implement Extended Kalman Filter for localization and SLAM algorithm
    - find inspiration: https://github.com/gargrohin/ROS-Navigation-and-Planning-with-SLAM

- implement other stuff
    - find inspiration at other forks from the original repo: https://github.com/dusty-nv/jetbot_ros/forks
    or the dusty-nv/jetbot_ros master branch (note that this repo is a fork of the branch "melodic")
