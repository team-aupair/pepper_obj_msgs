cmake_minimum_required(VERSION 2.4.6)
project(ORB_SLAM2)

# Check C++11 or C++0x support
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
   add_definitions(-DCOMPILEDWITHC11)
   message(STATUS "Using flag -std=c++11.")
elseif(COMPILER_SUPPORTS_CXX0X)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
   add_definitions(-DCOMPILEDWITHC0X)
   message(STATUS "Using flag -std=c++0x.")
else()
   message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

#LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/../../../cmake_modules)

find_package(OpenCV 3.0 QUIET)
if(NOT OpenCV_FOUND)
   find_package(OpenCV 2.4.3 QUIET)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 2.4.3 not found.")
   endif()
endif()

find_package(Eigen3 3.1.0 REQUIRED)
find_package(Pangolin REQUIRED)
find_package(catkin REQUIRED COMPONENTS tf roscpp sensor_msgs image_transport cv_bridge)

set(PROJECT_SOURCE_DIR $ENV{ORB_PATH}/Examples/ROS/ORB_SLAM2)

catkin_package(
#  INCLUDE_DIRS include
  LIBRARIES ORB_SLAM2
  CATKIN_DEPENDS tf roscpp sensor_msgs image_transport cv_bridge pepper_obj_msgs
#  DEPENDS system_lib
)

include_directories(
${catkin_INCLUDE_DIRS}
${PROJECT_SOURCE_DIR}
${PROJECT_SOURCE_DIR}/../../../
${PROJECT_SOURCE_DIR}/../../../include
${Pangolin_INCLUDE_DIRS}
$ENV{CATKIN_PATH}/devel/include
)

set(LIBS
${catkin_LIBRARIES}
${OpenCV_LIBS}
${EIGEN3_LIBS}
${Pangolin_LIBRARIES}
${PROJECT_SOURCE_DIR}/../../../Thirdparty/DBoW2/lib/libDBoW2.so
${PROJECT_SOURCE_DIR}/../../../Thirdparty/g2o/lib/libg2o.so
${PROJECT_SOURCE_DIR}/../../../lib/libORB_SLAM2.so
-lboost_system
)

# Node for monocular camera
add_executable(Mono
${PROJECT_SOURCE_DIR}/src/ros_mono.cc
)

target_link_libraries(Mono
${LIBS}
)

# Node for monocular camera (Augmented Reality Demo)
add_executable(MonoAR
${PROJECT_SOURCE_DIR}/src/AR/ros_mono_ar.cc
${PROJECT_SOURCE_DIR}/src/AR/ViewerAR.h
${PROJECT_SOURCE_DIR}/src/AR/ViewerAR.cc
)

target_link_libraries(MonoAR
${LIBS}
)

# Node for stereo camera
add_executable(Stereo
${PROJECT_SOURCE_DIR}/src/ros_stereo.cc
)

target_link_libraries(Stereo
${LIBS}
)

# Node for RGB-D camera
add_executable(RGBD
${PROJECT_SOURCE_DIR}/src/ros_rgbd.cc
)

add_dependencies(RGBD pepper_obj_msgs_generate_messages_cpp)

target_link_libraries(RGBD
${LIBS}
)

# Node for RVIZ MAP
add_executable(MAP
${PROJECT_SOURCE_DIR}/src/ros_draw_map.cc
)

target_link_libraries(MAP
${LIBS}
)
