cmake_minimum_required(VERSION 3.0.2)
project(f1tenth_racecar)

add_compile_options(-std=c++17)

find_package(catkin REQUIRED
  actionlib
  rospy
  rosservice
  rostopic
  sensor_msgs
)

catkin_package(
  CATKIN_DEPENDS
    ackermann_cmd_mux
    joy
    rosbag
    rostopic
    rviz
    tf2_ros
    tf
    urg_node
)

include_directories(${catkin_INCLUDE_DIRS})

add_executable(ourdrive src/ourdrive_node.cpp)
target_link_libraries(ourdrive ${catkin_LIBRARIES})

install(PROGRAMS
  scripts/joy_teleop.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(DIRECTORY config
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

install(DIRECTORY launch
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

install(DIRECTORY maps
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

install(DIRECTORY rviz
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
