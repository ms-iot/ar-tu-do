cmake_minimum_required(VERSION 2.8.3)
project(simulation_tools)

## Compile as C++11, supported in ROS Kinetic and newer
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 11)
endif()

## Find catkin macros and libraries
find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  std_msgs
  gazebo_msgs
)

find_package(gazebo REQUIRED)
find_package(TBB CONFIG)
if(NOT TBB_FOUND)
  pkg_check_modules(TBB tbb)
endif()

if(NOT TBB_FOUND)
  message(FATAL_ERROR "TBB is required.")
endif()

if(NOT TARGET TBB::tbb)
  add_library(TBB::tbb IMPORTED)
  set_target_properties(TBB::tbb PROPERTIES
    INTERFACE_INCLUDE_DIRECTORIES ${TBB_INCLUDE_DIRS}
    IMPORTED_LINK_INTERFACE_LIBRARIES ${TBB_LIBRARIES}
  )
endif()

## Errors and Warnings

set ( GCC_CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wchar-subscripts -Wchkp -Wdouble-promotion -Wformat -Wnonnull -Wmain -Wswitch-bool -Winvalid-memory-model -Wunknown-pragmas -Warray-bounds -Wfloat-equal -Wlogical-op -Wpacked ")
# -Wpedantic cant be used because of ROS
if(NOT MSVC)
  set(CMAKE_CXX_FLAGS "${GCC_CMAKE_CXX_FLAGS}")
else()
  add_compile_options(/IGNORE:4251)
endif()

catkin_python_setup()

###################################
## catkin specific configuration ##
###################################

## Declare things to be passed to dependent projects
catkin_package(
  INCLUDE_DIRS include
  CATKIN_DEPENDS roscpp rospy std_msgs
)


###########
## Build ##
###########

## Specify additional locations of header files
include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${GAZEBO_INCLUDE_DIRS}
)

# Declare crash detector node executable
add_executable(crash_detector src/crash_detector.cpp)
target_link_directories(crash_detector PRIVATE TBB::tbb)
target_link_libraries(crash_detector ${catkin_LIBRARIES} ${GAZEBO_LIBRARIES} TBB::tbb)
add_dependencies(crash_detector ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(chase_cam src/chase_cam.cpp)
target_link_directories(chase_cam PRIVATE TBB::tbb)
target_link_libraries(chase_cam ${catkin_LIBRARIES} ${GAZEBO_LIBRARIES} TBB::tbb)
add_dependencies(chase_cam ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})