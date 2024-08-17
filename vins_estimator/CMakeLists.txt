cmake_minimum_required(VERSION 2.8.3)
project(vins)

set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CXX_FLAGS "-std=c++14") #11改为14以支持pangolin
#-DEIGEN_USE_MKL_ALL")
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -Wall -g")

## 一些库（比如glog）并不总能在系统中找到FindXXX.cmake之类的文件，我们在这里手动提供
message(STATUS "## CMAKE_SOURCE_DIR   : " ${CMAKE_SOURCE_DIR})
message(STATUS "## PROJECT_SOURCE_DIR : " ${PROJECT_SOURCE_DIR})
list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/../cmake)
list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

find_package(Eigen3)
message(STATUS "## EIGEN3_FOUND        : " ${EIGEN3_FOUND})
message(STATUS "## EIGEN3_INCLUDE_DIR  : " ${EIGEN3_INCLUDE_DIR})

find_package(OpenCV REQUIRED)
message(STATUS "## OpenCV_VERSION      : " ${OpenCV_VERSION})
message(STATUS "## OpenCV_INCLUDE_DIRS : " ${OpenCV_INCLUDE_DIRS})
message(STATUS "## OpenCV_LIBRARIES    : " ${OpenCV_LIBRARIES})
message(STATUS "## OpenCV_LIBS         : " ${OpenCV_LIBS})

find_package(Ceres REQUIRED)
message(STATUS "## CERES_VERSION      : " ${CERES_VERSION})
message(STATUS "## CERES_INCLUDE_DIRS : " ${CERES_INCLUDE_DIRS})
message(STATUS "## CERES_LIBRARIES    : " ${CERES_LIBRARIES})

find_package(Glog REQUIRED)
message(STATUS "## GLOG_FOUND        : " ${GLOG_FOUND})
message(STATUS "## GLOG_INCLUDE_DIRS : " ${GLOG_INCLUDE_DIRS})
message(STATUS "## GLOG_INCLUDE_DIR  : " ${GLOG_INCLUDE_DIR})
message(STATUS "## GLOG_LIBRARIES    : " ${GLOG_LIBRARIES})

find_package(Gflags REQUIRED)
message(STATUS "## GFLAGS_FOUND        : " ${GFLAGS_FOUND})
message(STATUS "## GFLAGS_INCLUDE_DIRS : " ${GFLAGS_INCLUDE_DIRS})
message(STATUS "## GFLAGS_LIBRARIES    : " ${GFLAGS_LIBRARIES})

find_package(Pangolin REQUIRED) #QUIET
message(STATUS "## Pangolin INCLUDE DIRS: " ${Pangolin_INCLUDE_DIRS})
message(STATUS "## Pangolin LIBRARIES   : " ${Pangolin_LIBRARIES})
message(STATUS "## Pangolin FOUND       : " ${Pangolin_FOUND})
if(${Pangolin_FOUND}) 
  message(STATUS "## yes, found Pangolin!")
  add_definitions(-DUSE_PANGOLIN)
endif()

find_package(catkin REQUIRED COMPONENTS
  roscpp
  std_msgs
  geometry_msgs
  nav_msgs
  tf
  cv_bridge
  camera_models
  image_transport
)

include_directories(
  ${catkin_INCLUDE_DIRS}
  ${CERES_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
  ${Glog_INCLUDE_DIRS}
  ${Pangolin_INCLUDE_DIRS}
)

catkin_package()

add_library(vins_lib
  src/estimator/parameters.cpp
  src/estimator/estimator.cpp
  src/estimator/feature_manager.cpp
  src/featureTracker/feature_tracker.cpp
  src/initial/solve_5pts.cpp
  src/initial/initial_alignment.cpp
  src/initial/initial_sfm.cpp
  src/initial/initial_ex_rotation.cpp
  src/factor/pose_local_parameterization.cpp
  src/factor/projectionTwoFrameOneCamFactor.cpp
  src/factor/projectionTwoFrameTwoCamFactor.cpp
  src/factor/projectionOneFrameTwoCamFactor.cpp
  src/factor/marginalization_factor.cpp
  src/utility/utility.cpp
  src/utility_ros/RosVisualization.cpp
  src/utility_ros/CameraPoseVisualization.cpp
  src/ui_pangolin/ui_window_impl.cpp
  src/ui_pangolin/ui_window.cpp
)

# if (${Pangolin_FOUND}) {
#     add_library(vins_lib
#       src/ui_pangolin/ui_window_impl.cpp
#       src/ui_pangolin/ui_window.cpp
#   )
# }

target_link_libraries(vins_lib 
  ${catkin_LIBRARIES} 
  ${OpenCV_LIBS} 
  ${CERES_LIBRARIES}
  ${Pangolin_LIBRARIES}
)


add_executable(vins_node src/rosNodeTest.cpp)
target_link_libraries(vins_node vins_lib) 

add_executable(kitti_odom_test src/KITTIOdomTest.cpp)
target_link_libraries(kitti_odom_test vins_lib) 

add_executable(kitti_gps_test src/KITTIGPSTest.cpp)
target_link_libraries(kitti_gps_test vins_lib) 

