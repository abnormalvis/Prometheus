cmake_minimum_required(VERSION 2.8.3)
project(prometheus_gazebo)

find_package(catkin REQUIRED COMPONENTS
  gazebo_dev
  gazebo_msgs
  gazebo_ros
  message_generation      
  roscpp
  geometry_msgs
  sensor_msgs
  std_msgs
  gazebo_msgs
  prometheus_msgs
)

## System dependencies are found with CMake's conventions
find_package(Boost REQUIRED COMPONENTS system)

catkin_package(
  INCLUDE_DIRS include
  CATKIN_DEPENDS message_runtime
)

include_directories(
    include
    ${catkin_INCLUDE_DIRS}
)
###############################
##      声明可执行cpp文件     ##
###############################
add_executable(set_model_state_example cpp_nodes/set_model_state_example.cpp)
add_dependencies(set_model_state_example prometheus_gazebo_gencpp)
target_link_libraries(set_model_state_example ${catkin_LIBRARIES})

## Mark cpp header files for installation
install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h"
)

install(DIRECTORY launch/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
)

## Keep the package's source-relative resource paths in the install space.
install(DIRECTORY launch_basic/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch_basic
)
install(DIRECTORY launch_uav_with_sensor/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch_uav_with_sensor
)
install(DIRECTORY config/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/config
)
install(DIRECTORY gazebo_models/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/gazebo_models
)
install(DIRECTORY gazebo_worlds/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/gazebo_worlds
)

install(PROGRAMS scripts/jinja_gen.py
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/scripts
)