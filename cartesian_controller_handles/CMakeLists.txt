cmake_minimum_required(VERSION 3.20)
project(cartesian_controller_handles)

set(CMAKE_CXX_STANDARD 17)
set(ADDITIONAL_COMPILE_OPTIONS -Wall -Wextra -Wpedantic -Wno-unused-parameter)
add_compile_options(${ADDITIONAL_COMPILE_OPTIONS})

find_package(ament_cmake REQUIRED)
find_package(cartesian_controller_base REQUIRED) # For ROS2 version handling
find_package(controller_interface REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(interactive_markers REQUIRED)
find_package(kdl_parser REQUIRED)
find_package(pluginlib REQUIRED)
find_package(rclcpp REQUIRED)
find_package(urdf REQUIRED)

#--------------------------------------------------------------------------------
# Libraries
#--------------------------------------------------------------------------------

add_library(${PROJECT_NAME} SHARED
  src/motion_control_handle.cpp
)

target_include_directories(${PROJECT_NAME}
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

target_link_libraries(${PROJECT_NAME} PUBLIC
  cartesian_controller_base::cartesian_controller_base
  controller_interface::controller_interface
  geometry_msgs::geometry_msgs__rosidl_generator_cpp
  interactive_markers::interactive_markers
  kdl_parser::kdl_parser
  pluginlib::pluginlib
  rclcpp::rclcpp
  urdf::urdf
)

# Prevent pluginlib from using boost
target_compile_definitions(${PROJECT_NAME} PUBLIC "PLUGINLIB__DISABLE_BOOST_FUNCTIONS")

#--------------------------------------------------------------------------------
# Install and export
#--------------------------------------------------------------------------------

pluginlib_export_plugin_description_file(controller_interface cartesian_controller_handles_plugin.xml)

install(
  DIRECTORY include/
  DESTINATION include
)

install(
  TARGETS ${PROJECT_NAME}
  EXPORT export_${PROJECT_NAME}
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)
ament_export_dependencies(
  cartesian_controller_base
  controller_interface
  geometry_msgs
  interactive_markers
  kdl_parser
  pluginlib
  rclcpp
  urdf
)

ament_package()
