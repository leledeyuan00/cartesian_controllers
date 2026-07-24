cmake_minimum_required(VERSION 3.20)
project(cartesian_controller_utilities)

# Default to C++20
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 20)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()


find_package(ament_cmake REQUIRED)
find_package(rclpy REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)


#--------------------------------------------------------------------------------
# Install and export
#--------------------------------------------------------------------------------

install(PROGRAMS
  scripts/buttons.py
  scripts/converter.py
  scripts/pose.py
  DESTINATION lib/${PROJECT_NAME}
)

install(
  DIRECTORY etc launch
  DESTINATION share/${PROJECT_NAME}
)

ament_package()
