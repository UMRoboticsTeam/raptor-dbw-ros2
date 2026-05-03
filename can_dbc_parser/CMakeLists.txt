cmake_minimum_required(VERSION 3.5)
project(can_dbc_parser)

# LTO objects seem to be breaking emulated builds with Github actions, I think this is related https://github.com/software-mansion/scarb/issues/2548
set(CMAKE_INTERPROCEDURAL_OPTIMIZATION OFF CACHE BOOL "Disable LTO" FORCE)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Parse version number from the file
file(READ "version" version_input)
string(REGEX MATCH "([0-9]+)\.([0-9]+)\.([0-9]+)" _ ${version_input})

# Fill version number into package.xml
configure_file(
        package.in.xml
        ${CMAKE_CURRENT_SOURCE_DIR}/package.xml
        @ONLY
)

# find dependencies
find_package(ament_cmake_auto REQUIRED)
find_package(can_msgs REQUIRED)
ament_auto_find_build_dependencies()

ament_auto_add_library(${PROJECT_NAME} SHARED
  src/DbcMessage.cpp
  src/DbcSignal.cpp
  src/Dbc.cpp
  src/LineParser.cpp
  src/DbcBuilder.cpp
)

target_compile_options(${PROJECT_NAME} PRIVATE -Wno-unused-function)

#run colcon test to run linters against code
if(BUILD_TESTING)
  find_package(ament_lint_auto)
  ament_lint_auto_find_test_dependencies()
endif()

# Setup files to satisfy rosdep
# This is pretty hacky...
# Package name format comes from "rosify_package_name" function in https://github.com/ros-infrastructure/bloom/blob/master/bloom/generators/rosdebian.py
write_file("${CMAKE_CURRENT_BUILD_DIR}/rosdep/sources.list.d/99-${PROJECT_NAME}.list" "yaml file:///etc/ros/rosdep/${PROJECT_NAME}.yaml")
write_file("${CMAKE_CURRENT_BUILD_DIR}/rosdep/${PROJECT_NAME}.yaml" "${PROJECT_NAME}:\n  ubuntu: [ros-$ENV{ROS_DISTRO}-${PROJECT_NAME}]")
install(
  DIRECTORY ${CMAKE_CURRENT_BUILD_DIR}/rosdep
  DESTINATION /etc/ros/
)

ament_auto_package(INSTALL_TO_SHARE
  # launch
)
