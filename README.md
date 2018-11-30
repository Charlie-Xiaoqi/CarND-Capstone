# **CarND-Capstone**

This is the project repo for the final project of the Udacity Self-Driving Car Nanodegree: Programming a Real Self-Driving Car. For more information about the project, see the project introduction [here](https://youtu.be/UT34zkxfS_M).

### Team Members

* [Xiaoqi Li](https://github.com/Charlie-Xiaoqi) - Team lead
* [Mohammad Bahrami](https://github.com/mhBahrami) - Trajectory Planner, Controller, and Integrator
* [Nikola Noxon](https://github.com/nikolanoxon) - Traffic Light Detection
* [Tamoghna Das](https://github.com/tamoghna21) - Traffic Light Detection and Controller

### Project Components

* Diagram of nodes and messages

#### Visualization

- [Highway](https://youtu.be/hxx9eVh6IdU)

#### Traffic Light Detection

* We adopted the inception v2 model from the tensorflow zoo, the architecture of the model can be found over here: 
[Inception v2 architecture](https://towardsdatascience.com/a-simple-guide-to-the-versions-of-the-inception-network-7fc52b863202)

#### Traffic Light Classification

| Simulator model         | Test site model                            |
|:-----------------------:|:--------------------------------------:|
| ![simulator model](https://github.com/Charlie-Xiaoqi/CarND-Capstone-1/blob/master/Classification_image/Simulator_image.png)          | ![Test site model](https://github.com/Charlie-Xiaoqi/CarND-Capstone-1/blob/master/Classification_image/Testsite_image.png)

* Four output classes: `GREEN`, `YELLOW`, `RED`, `UNKNOWN`.
* Test accuracy: lowest precision was red at 98%, lowest recall was green at 88%.
* See model and training code in the Nikola's repo in [`traffic_light_classifier.ipynb`](https://github.com/nikolanoxon/CarND-Traffic-Light-Classifier/blob/master/traffic_light_classifier.ipynb).
* See inference code in [`tl_classifier.py`](https://github.com/mhBahrami/CarND-Capstone/blob/master/ros/src/tl_detector/light_classification/tl_classifier.py).

#### Trajectory Planner

Trajectory planner is implemented in [`waypoint_updater.py`](https://github.com/mhBahrami/CarND-Capstone/blob/master/ros/src/waypoint_updater/waypoint_updater.py). It specifies the speed of waypoints based on the current speed of the vehicle, distance between car and stop line (or traffic light), and the color of the traffic light (the traffic light status).

The code will publish waypoints generated by [`generate_lane()`](https://github.com/mhBahrami/CarND-Capstone/blob/master/ros/src/waypoint_updater/waypoint_updater.py#L91). The most important parts of this function are as following:

- [`should_accelerate()`](https://github.com/mhBahrami/CarND-Capstone/blob/master/ros/src/waypoint_updater/waypoint_updater.py#L103) function: It determines either the car should accelerate to the cruise speed or decelerate (and be prepared to stop).

  > You can find it [from Line 136 to Line 165](https://github.com/mhBahrami/CarND-Capstone/blob/master/ros/src/waypoint_updater/waypoint_updater.py#L136).

  The decision is made based on the stop line index, the current speed of car, and the traffic light status (`GREEN`, `YELLOW`, `RED`, `UNKNOWN`). 

  - If it's `GREEN` or `UNKNOWN` the car would accelerate to cruise speed (From [Lines 140 to 145](https://github.com/mhBahrami/CarND-Capstone/blob/master/ros/src/waypoint_updater/waypoint_updater.py#L140)).

  - If it's `YELLOW` and the distance between the car and stop line is less than `16 m`, safe distance for the `YELLOW` ([Lines 153 & 154](https://github.com/mhBahrami/CarND-Capstone/blob/master/ros/src/waypoint_updater/waypoint_updater.py#L153)), there is not enough time to pass the traffic line before it turns to `RED`. So, the car will decelerate to stop behind the stop line. Otherwise the car will keep going.
  - If the traffic light is `RED`, the safe distance to stop a car is calculated using kinematic equations of motion which is a function of car's velocity ([Line 168](https://github.com/mhBahrami/CarND-Capstone/blob/master/ros/src/waypoint_updater/waypoint_updater.py#L168)). Again, if car is within this distance the car should decelerate to stop behind the stop line safely.

- Then based on output of the pervious function the waypoints to accelerate (function `accelerate_waypoints()` from [Line 186 to 197](https://github.com/mhBahrami/CarND-Capstone/blob/master/ros/src/waypoint_updater/waypoint_updater.py#L186)) or decelerate (function `decelerate_waypoints()` from [Line 200 to 219](https://github.com/mhBahrami/CarND-Capstone/blob/master/ros/src/waypoint_updater/waypoint_updater.py#L200)) will be generated.

#### Control Subsystem

This subsystem publishes control commands for the vehicle’s steering, throttle, and brakes based on a list of waypoints to follow.

#### Waypoint Follower Node

This node was given to us by Udacity. It parses the list of waypoints to follow and publishes proposed linear and angular velocities to the /twist_cmd topic

#### Drive By Wire (DBW) Node

The DBW node is the final step in the self driving vehicle’s system. At this point we have a target linear and angular velocity and must adjust the vehicle’s controls accordingly. In this project we control 3 things: throttle, steering, brakes. As such, we have 3 distinct controllers to interface with the vehicle.

#### Throttle Controller

The throttle controller is a simple PID controller that compares the current velocity with the target velocity and adjusts the throttle accordingly. The throttle gains were tuned using trial and error for allowing reasonable acceleration without oscillation around the set-point.

#### Steering Controller

This controller translates the proposed linear and angular velocities into a steering angle based on the vehicle’s steering ratio and wheelbase length. To ensure our vehicle drives smoothly, we cap the maximum linear and angular acceleration rates. The steering angle computed by the controller is also passed through a low pass filter to reduce possible jitter from noise in velocity data.

#### Braking Controller

This is the simplest controller of the three - we simply proportionally brake based on the throttle and the brake deadband.

Please use **one** of the two installation options, either native **or** docker installation.

### Native Installation

* Be sure that your workstation is running Ubuntu 16.04 Xenial Xerus or Ubuntu 14.04 Trusty Tahir. [Ubuntu downloads can be found here](https://www.ubuntu.com/download/desktop).
* If using a Virtual Machine to install Ubuntu, use the following configuration as minimum:
  * 2 CPU
  * 2 GB system memory
  * 25 GB of free hard drive space

  The Udacity provided virtual machine has ROS and Dataspeed DBW already installed, so you can skip the next two steps if you are using this.

* Follow these instructions to install ROS
  * [ROS Kinetic](http://wiki.ros.org/kinetic/Installation/Ubuntu) if you have Ubuntu 16.04.
  * [ROS Indigo](http://wiki.ros.org/indigo/Installation/Ubuntu) if you have Ubuntu 14.04.
* [Dataspeed DBW](https://bitbucket.org/DataspeedInc/dbw_mkz_ros)

  * Use this option to install the SDK on a workstation that already has ROS installed: [One Line SDK Install (binary)](https://bitbucket.org/DataspeedInc/dbw_mkz_ros/src/81e63fcc335d7b64139d7482017d6a97b405e250/ROS_SETUP.md?fileviewer=file-view-default)
* Download the [Udacity Simulator](https://github.com/udacity/CarND-Capstone/releases).

### Docker Installation
[Install Docker](https://docs.docker.com/engine/installation/)

Build the docker container
```bash
docker build . -t capstone
```

Run the docker file
```bash
docker run -p 4567:4567 -v $PWD:/capstone -v /tmp/log:/root/.ros/ --rm -it capstone
```

### Port Forwarding
To set up port forwarding, please refer to the [instructions from term 2](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/16cf4a78-4fc7-49e1-8621-3450ca938b77)

### Usage

1. Clone the project repository
```bash
git clone https://github.com/udacity/CarND-Capstone.git
```

2. Install python dependencies
```bash
cd CarND-Capstone
pip install -r requirements.txt
```
3. Make sure `*.py` files and `*.sh` files are executable
```bash
find [/YOUR/DIRECTORY/TO]/CarND-Capstone -type f -iname "*.py" -exec chmod +x {} \;
find [/YOUR/DIRECTORY/TO]/CarND-Capstone -type f -iname "*.sh" -exec chmod +x {} \;
```
4. Make and run styx
```bash
cd ros
catkin_make
source devel/setup.sh
roslaunch launch/styx.launch
```
5. Run the simulator

### Real world testing
1. Download [training bag](https://s3-us-west-1.amazonaws.com/udacity-selfdrivingcar/traffic_light_bag_file.zip) that was recorded on the Udacity self-driving car.
2. Unzip the file
```bash
unzip traffic_light_bag_file.zip
```
3. Play the bag file
```bash
rosbag play -l traffic_light_bag_file/traffic_light_training.bag
```
4. Launch your project in site mode
```bash
cd CarND-Capstone/ros
roslaunch launch/site.launch
```
5. Confirm that traffic light detection works on real life images
