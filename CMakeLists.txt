# Copyright(c) 2020 eaibot limited.
cmake_minimum_required(VERSION 3.20)
project(ydlidar_ros2_driver C CXX)

##################ros2#############################################
# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

##################build sdk##########################################
set(MY_PATCH_FILE "${CMAKE_CURRENT_SOURCE_DIR}/patches/ydlidar_sdk.patch")
include(ExternalProject)
ExternalProject_Add(ydlidar-sdk
  GIT_REPOSITORY https://github.com/YDLIDAR/YDLidar-SDK
  UPDATE_DISCONNECTED ON
  PATCH_COMMAND ${CMAKE_COMMAND} -E chdir
          ${CMAKE_CURRENT_BINARY_DIR}/ydlidar-sdk-prefix/src
	  patch -p0 < ${MY_PATCH_FILE}
  CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}/ydlidar-sdk-install
)
SET(YDLIDAR_SDK_INCLUDE_DIRS ${CMAKE_CURRENT_BINARY_DIR}/ydlidar-sdk-install/include)
SET(YDLIDAR_SDK_LIB_DIRS ${CMAKE_CURRENT_BINARY_DIR}/ydlidar-sdk-install/lib)

include_directories(
  include
  ${YDLIDAR_SDK_INCLUDE_DIRS}
  ${CMAKE_CURRENT_BINARY_DIR}/ydlidar-sdk-prefix/src/ydlidar-sdk
)

####################find package#####################################
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rmw REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(visualization_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(std_srvs REQUIRED)

############## YDLIDAR SDK START#####################################
#find ydlidar_sdk package
#find_package(ydlidar_sdk REQUIRED)
############## YDLIDAR SDK END#####################################

#Include directories
include_directories(
  ${PROJECT_SOURCE_DIR}
  ${PROJECT_SOURCE_DIR}/src
  ${YDLIDAR_SDK_INCLUDE_DIRS})

#link library directories
link_directories(${YDLIDAR_SDK_LIBRARY_DIRS})

#---------------------------------------------------------------------------------------
# generate excutable and add libraries
#---------------------------------------------------------------------------------------
add_executable(${PROJECT_NAME}_node
    src/${PROJECT_NAME}_node.cpp)
#---------------------------------------------------------------------------------------
# link libraries
#--------------------------------------------------------------------------------------
add_dependencies(${PROJECT_NAME}_node ydlidar-sdk)
target_link_libraries(ydlidar_ros2_driver_node PUBLIC
  ${geometry_msgs_TARGETS}
  ${sensor_msgs_TARGETS}
  ${std_srvs_TARGETS}
  ${visualization_msgs_TARGETS}
  rclcpp::rclcpp
  sensor_msgs::sensor_msgs_library
  ${YDLIDAR_SDK_LIB_DIRS}/libydlidar_sdk.a
)
#---------------------------------------------------------------------------------------
# generate excutable and add libraries
#---------------------------------------------------------------------------------------
add_executable(${PROJECT_NAME}_client
    src/${PROJECT_NAME}_client.cpp)
#---------------------------------------------------------------------------------------
# link libraries
#--------------------------------------------------------------------------------------
target_link_libraries(ydlidar_ros2_driver_client PUBLIC
  ${geometry_msgs_TARGETS}
  ${sensor_msgs_TARGETS}
  ${std_srvs_TARGETS}
  ${visualization_msgs_TARGETS}
  rclcpp::rclcpp
  sensor_msgs::sensor_msgs_library
)
#---------------------------------------------------------------------------------------
# Install
#---------------------------------------------------------------------------------------
install(TARGETS
    ${PROJECT_NAME}_node ${PROJECT_NAME}_client
    DESTINATION lib/${PROJECT_NAME})

install(DIRECTORY launch params startup config
    DESTINATION share/${PROJECT_NAME})

if(BUILD_TESTING)
    find_package(ament_lint_auto REQUIRED)
    ament_lint_auto_find_test_dependencies()
endif()

ament_package()

