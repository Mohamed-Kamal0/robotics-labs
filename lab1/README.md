# Robotics Assignment 1 - Requirement 1 Report & Detailed Guide

This document provides a comprehensive report for **Requirement 1** of Robotics Assignment 1. It contains detailed answers to the assignment questions, line-by-line explanations of every code file created, and an overview of ROS concepts utilized in this project.

---

## Table of Contents
1. [ROS Topic Exploration & Question Answers](#1-ros-topic-exploration--question-answers)
2. [Workspace & Package Architecture](#2-workspace--package-architecture)
3. [Package Configuration Files Explained](#3-package-configuration-files-explained)
   - [package.xml](#packagexml)
   - [CMakeLists.txt](#cmakeliststxt)
4. [Python Controller Node (`move_turtle.py`) Detailed Line-by-Line Explanation](#4-python-controller-node-move_turtlepy-detailed-line-by-line-explanation)
5. [Launch File (`move.launch`) Detailed Explanation](#5-launch-file-movelaunch-detailed-explanation)
6. [Step-by-Step Running & Testing Guide](#6-step-by-step-running--testing-guide)

---

## 1. ROS Topic Exploration & Question Answers

### Discovery Commands
In ROS, nodes communicate by sending messages over channels called **topics**. To inspect topics running under `turtlesim_node`:

```bash
# List all active ROS topics
rostopic list

# Output includes:
# /rosout
# /rosout_agg
# /turtle1/cmd_vel
# /turtle1/color_sensor
# /turtle1/pose
```

Checking topic details using `rostopic info`:

```bash
rostopic info /turtle1/cmd_vel
```

**Output breakdown:**
- **Type**: `geometry_msgs/Twist`
- **Publishers**: None (until a controller node runs)
- **Subscribers**: `/turtlesim`

---

### Questions & Answers

#### Question 1: Which topic has `/turtlesim` node as subscriber?
> **Answer**: `/turtle1/cmd_vel`

#### Question 2: What is the message type of this topic?
> **Answer**: `geometry_msgs/Twist`

---

### Understanding the Message Structure (`geometry_msgs/Twist`)
The `geometry_msgs/Twist` message represents linear and angular velocity vectors in 3D space:

```
geometry_msgs/Vector3 linear
  float64 x   # Linear velocity along X-axis (forward/backward or right/left in 2D grid)
  float64 y   # Linear velocity along Y-axis (up/down in 2D grid)
  float64 z   # Linear velocity along Z-axis (unused in 2D turtlesim)
geometry_msgs/Vector3 angular
  float64 x   # Roll rate (unused in 2D turtlesim)
  float64 y   # Pitch rate (unused in 2D turtlesim)
  float64 z   # Yaw rate (rotation speed around vertical axis)
```

In 2D Turtlesim:
- Setting `linear.x = 0.5` pushes the turtle horizontally to the right.
- Setting `linear.y = 0.5` pushes the turtle vertically upwards.

---

## 2. Workspace & Package Architecture

A ROS **Catkin Workspace** is a folder where ROS packages are built and organized.

### Directory Tree:
```
catkin_ws/
├── build/                 # Auto-generated CMake/Build artifacts
├── devel/                 # Auto-generated setup scripts & executables
└── src/                   # Source code folder
    └── turtle_req/        # Custom ROS package
        ├── CMakeLists.txt # CMake build rules
        ├── package.xml    # Package metadata & dependencies
        ├── launch/
        │   └── move.launch
        └── scripts/
            └── move_turtle.py
```

---

## 3. Package Configuration Files Explained

### `package.xml`
The `package.xml` manifest defines metadata and dependencies required by ROS:

```xml
<?xml version="1.0"?>
<package format="2">
  <name>turtle_req</name>
  <version>0.0.0</version>
  <description>The turtle_req package for turtlesim movement automation</description>

  <maintainer email="user@todo.todo">mohamedkamal</maintainer>
  <license>BSD</license>

  <!-- Build Tool Dependency: Required to build catkin packages -->
  <buildtool_depend>catkin</buildtool_depend>
  
  <!-- Build Dependencies: Headers/libraries needed at compile time -->
  <build_depend>rospy</build_depend>
  <build_depend>std_msgs</build_depend>
  <build_depend>geometry_msgs</build_depend>
  
  <!-- Export Dependencies: Packages needed by dependent packages -->
  <build_export_depend>rospy</build_export_depend>
  <build_export_depend>std_msgs</build_export_depend>
  <build_export_depend>geometry_msgs</build_export_depend>
  
  <!-- Execution Dependencies: Packages needed at runtime -->
  <exec_depend>rospy</exec_depend>
  <exec_depend>std_msgs</exec_depend>
  <exec_depend>geometry_msgs</exec_depend>
  <exec_depend>turtlesim</exec_depend>
</package>
```

---

### `CMakeLists.txt`
`CMakeLists.txt` instructs Catkin how to build and install the package:

```cmake
cmake_minimum_required(VERSION 3.0.2)
project(turtle_req)

# Finds dependent ROS packages and cmake components
find_package(catkin REQUIRED COMPONENTS
  rospy
  std_msgs
  geometry_msgs
)

# Declare Catkin package build rules
catkin_package()

# Include ROS standard directory paths
include_directories(
  ${catkin_INCLUDE_DIRS}
)

# Marks Python scripts as executable programs and installs them to devel space
catkin_install_python(PROGRAMS
  scripts/move_turtle.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
```

---

## 4. Python Controller Node (`move_turtle.py`) Detailed Line-by-Line Explanation

Below is the complete implementation of `move_turtle.py` with line-by-line explanations:

```python
#!/usr/bin/env python3
# Line 1: Shebang header telling Linux to execute this script using Python 3.

import rospy
# Line 4: Imports rospy, the Python client library for ROS.

from geometry_msgs.msg import Twist
# Line 5: Imports the Twist message class from geometry_msgs.

def move_turtle():
    # Line 7: Define main controller function.

    rospy.init_node('move_turtle', anonymous=True)
    # Line 9: Registers node named 'move_turtle' with the ROS master.
    # anonymous=True appends a unique random number to prevent name collisions.

    pub = rospy.Publisher('/turtle1/cmd_vel', Twist, queue_size=10)
    # Line 12: Creates a ROS publisher object.
    # Topic: '/turtle1/cmd_vel'
    # Message type: Twist
    # queue_size=10: Buffers up to 10 outgoing messages if network is congested.

    rate = rospy.Rate(10)
    # Line 17: Defines loop frequency to 10 Hz (10 iterations per second).

    start_time = rospy.get_time()
    # Line 19: Records current ROS system time (in seconds).

    move_x = True
    # Line 22: Boolean flag tracking state:
    # True  -> Move along X-axis (horizontal)
    # False -> Move along Y-axis (vertical)

    rospy.loginfo("Turtle controller node started. Toggling motion every 2.5 seconds.")
    # Line 26: Logs an informational message to the ROS console/log topic.

    while not rospy.is_shutdown():
        # Line 28: Main loop runs until Ctrl+C or node shutdown signal.

        current_time = rospy.get_time()
        # Line 30: Fetches latest time stamp.

        if current_time - start_time >= 2.5:
            # Line 33: Check if 2.5 seconds have passed since last state change.
            move_x = not move_x
            # Line 35: Invert state (True becomes False, False becomes True).
            start_time = current_time
            # Line 36: Reset state clock.
            
            axis_name = "X (horizontal)" if move_x else "Y (vertical)"
            rospy.loginfo("Switched movement to %s axis", axis_name)
            # Line 39: Log state switch.

        vel_msg = Twist()
        # Line 41: Instantiate new empty Twist message object (all fields default to 0.0).

        if move_x:
            vel_msg.linear.x = 0.5
            vel_msg.linear.y = 0.0
        else:
            vel_msg.linear.x = 0.0
            vel_msg.linear.y = 0.5
        # Lines 43-49: Assign 0.5 velocity to active axis and 0.0 to inactive axis.

        pub.publish(vel_msg)
        # Line 51: Publish velocity message to /turtle1/cmd_vel.

        rate.sleep()
        # Line 53: Pauses execution to maintain exact 10 Hz rate.

if __name__ == '__main__':
    try:
        move_turtle()
    except rospy.ROSInterruptException:
        pass
    # Lines 55-59: Standard Python entry point with ROS interrupt handling.
```

---

## 5. Launch File (`move.launch`) Detailed Explanation

ROS launch files use XML format to start multiple nodes, set parameters, and automate system bringup without opening multiple terminal windows manually.

```xml
<launch>
    <!-- 
      Node 1: Starts the official Turtlesim GUI simulator.
      pkg: Package name containing executable ('turtlesim')
      type: Executable program name ('turtlesim_node')
      name: ROS node name inside graph ('turtlesim')
      output="screen": Redirects stdout/stderr to terminal screen
    -->
    <node pkg="turtlesim" type="turtlesim_node" name="turtlesim" output="screen"/>

    <!-- 
      Node 2: Starts our custom python script node.
      pkg: Package name ('turtle_req')
      type: Python script name ('move_turtle.py')
      name: ROS node name inside graph ('move_turtle')
      output="screen": Displays log messages in terminal window
    -->
    <node pkg="turtle_req" type="move_turtle.py" name="move_turtle" output="screen"/>
</launch>
```

---

## 6. Step-by-Step Running & Testing Guide

run roscore in terminal 
run rosrun turtlesim turtlesim_node in another 
run python lab1/Requirement_1_Submission/turtle_req/scripts/move_turtle.py in another

### Expected Behavior
1. The **turtlesim** canvas opens automatically.
2. The turtle moves **right (X-axis)** with speed 0.5 for **2.5 seconds**.
3. The turtle switches to move **up (Y-axis)** with speed 0.5 for **2.5 seconds**.
4. The pattern repeats continuously, creating a **staircase trajectory**.
