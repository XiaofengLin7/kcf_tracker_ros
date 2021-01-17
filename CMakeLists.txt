cmake_minimum_required(VERSION 2.8)
project(kcf_target_tracker)
set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")
# Try to see if user has OpenCV installed
# if yes, default callback will display the image
find_package(OpenCV REQUIRED)
if (OpenCV_FOUND)
  message( "\n${PROJECT_NAME}...")
  message( STATUS " - Includes: ${OpenCV_INCLUDE_DIRS}")
  message( STATUS " - Libraries: ${OpenCV_LIBRARIES}")
else()
  message( STATUS "OpenCV is required for this sample to do the target tracking")
endif ()


unset(KCFcpp_DIR)
find_path(KCFcpp_DIR
        NAMES
        KCFCpp.sh
        PATHS
        ${CMAKE_CURRENT_SOURCE_DIR}/KCFcpp
        )

if(NOT KCFcpp_DIR)
  message(STATUS "Target Tracking sample depends on KCFcpp library from https://github.com/joaofaro/KCFcpp.git")
  message(STATUS "When you run make, we will try to clone the repo")
  message(STATUS "This will happen only once as long as you do not delete it")

  include(${CMAKE_MODULE_PATH}/DownloadProject/DownloadProject.cmake)
  download_project(PROJ                kcf_cpp
          GIT_REPOSITORY      https://github.com/joaofaro/KCFcpp.git
          GIT_TAG             master
          ${UPDATE_DISCONNECTED_IF_AVAILABLE}
          SOURCE_DIR          ${CMAKE_CURRENT_SOURCE_DIR}/KCFcpp
          )

endif()



## To use this this repo, you need to run
##  git clone https://github.com/joaofaro/KCFcpp.git
## under the current foler
find_package(Boost REQUIRED COMPONENTS thread)
find_package(catkin REQUIRED
  COMPONENTS
    cv_bridge
    roscpp
    rospy
    std_msgs
    actionlib
    image_transport
)
catkin_package(
  CATKIN_DEPENDS
    cv_bridge
    roscpp
    actionlib
    rospy
    std_msgs
  ##  darknet_ros_msgs
    image_transport
  DEPENDS
    Boost
)
include_directories(KCFcpp/src)
include_directories(
  ${OpenCV_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}

)

add_library(kcf_tracker STATIC
  KCFcpp/src/fhog.cpp
  KCFcpp/src/kcftracker.cpp)
target_link_libraries(kcf_tracker ${OpenCV_LIBRARIES})

add_executable(${PROJECT_NAME}
  target_tracking.cpp
  tracking_utility.cpp
  )
target_link_libraries(${PROJECT_NAME}
  kcf_tracker
  ${Boost_LIBRARIES}
  ${OpenCV_LIBRARIES}
  ${catkin_LIBRARIES}
  ${OpenCV_LIBS}
  )
