cmake_minimum_required(VERSION 3.20)
project(cartesian_controller_base)

set(CMAKE_CXX_STANDARD 17)

set(ADDITIONAL_COMPILE_OPTIONS -Wall -Wextra -Wpedantic -Wno-unused-parameter)
add_compile_options(${ADDITIONAL_COMPILE_OPTIONS})

# Set build type to "Release" by default
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif(NOT CMAKE_BUILD_TYPE)

# Use CMake to pass the current ROS_DISTRO via variables into a preprocessor template.
# We then include this file and switch between the different APIs.
if($ENV{ROS_DISTRO} STREQUAL "jazzy")
  set(CARTESIAN_CONTROLLERS_JAZZY TRUE)
elseif($ENV{ROS_DISTRO} STREQUAL "iron")
  set(CARTESIAN_CONTROLLERS_IRON TRUE)
elseif($ENV{ROS_DISTRO} STREQUAL "humble")
  set(CARTESIAN_CONTROLLERS_HUMBLE TRUE)
elseif($ENV{ROS_DISTRO} STREQUAL "lyrical")
  set(CARTESIAN_CONTROLLERS_LYRICAL TRUE)
else()
  message(WARNING "ROS2 version must be {jazzy|iron|humble}")
endif()
configure_file(include/cartesian_controller_base/ROS2VersionConfig.h.in ROS2VersionConfig.h)

find_package(Eigen3 3.3 REQUIRED NO_MODULE)
add_definitions(-DEIGEN_MPL2_ONLY)

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(controller_interface REQUIRED)
find_package(hardware_interface REQUIRED)
find_package(kdl_parser REQUIRED)
find_package(trajectory_msgs REQUIRED)
find_package(pluginlib REQUIRED)
find_package(urdf REQUIRED)
find_package(realtime_tools REQUIRED)

#--------------------------------------------------------------------------------
# Libraries
#--------------------------------------------------------------------------------
add_library(ik_solvers SHARED
  src/IKSolver.cpp
  src/ForwardDynamicsSolver.cpp
  src/JacobianTransposeSolver.cpp
  src/DampedLeastSquaresSolver.cpp
  src/SelectivelyDampedLeastSquaresSolver.cpp
)

target_include_directories(ik_solvers
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
    $<BUILD_INTERFACE:${CMAKE_BINARY_DIR}>  # ROS2VersionConfig.h
)

target_link_libraries(ik_solvers PUBLIC
  hardware_interface::hardware_interface
  rclcpp::rclcpp
  trajectory_msgs::trajectory_msgs__rosidl_generator_cpp
  kdl_parser::kdl_parser
  pluginlib::pluginlib
  Eigen3::Eigen
)

add_library(${PROJECT_NAME} SHARED
  src/cartesian_controller_base.cpp
  src/SpatialPDController.cpp
  src/PDController.cpp
  src/IKSolver.cpp
)

# Manual includes for local directories and non-ament packages
target_include_directories(${PROJECT_NAME}
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
    $<BUILD_INTERFACE:${CMAKE_BINARY_DIR}>  # ROS2VersionConfig.h
)

target_link_libraries(${PROJECT_NAME} PUBLIC
  ik_solvers
  rclcpp::rclcpp
  controller_interface::controller_interface
  kdl_parser::kdl_parser
  trajectory_msgs::trajectory_msgs__rosidl_generator_cpp
  pluginlib::pluginlib
  urdf::urdf
  realtime_tools::realtime_tools
  Eigen3::Eigen
)

# Prevent pluginlib from using boost
target_compile_definitions(${PROJECT_NAME} PUBLIC "PLUGINLIB__DISABLE_BOOST_FUNCTIONS")

#--------------------------------------------------------------------------------
# Install and export
#--------------------------------------------------------------------------------

pluginlib_export_plugin_description_file(cartesian_controller_base ik_solver_plugin.xml)

install(
  DIRECTORY include/
  DESTINATION include
)

install(
  FILES ${CMAKE_BINARY_DIR}/ROS2VersionConfig.h
  DESTINATION include/${PROJECT_NAME}
)

install(
  TARGETS ${PROJECT_NAME} ik_solvers
  EXPORT export_${PROJECT_NAME}
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)
ament_export_dependencies(
  controller_interface
  hardware_interface
  kdl_parser
  trajectory_msgs
  pluginlib
  urdf
  Eigen3
  realtime_tools
)

ament_package()
