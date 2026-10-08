cmake_minimum_required(VERSION 3.0.2)
project(turtle_req)

find_package(catkin REQUIRED COMPONENTS
  rospy
  std_msgs
  geometry_msgs
)

catkin_package()

include_directories(
  ${catkin_INCLUDE_DIRS}
)

catkin_install_python(PROGRAMS
  scripts/move_turtle.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
