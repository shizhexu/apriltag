cmake_minimum_required(VERSION 2.8.3)
project(apriltag)

# @todo this does NOT work, since it should set ${CMAKE_THREAD_LIBS_INIT}, but
# it does NOT!
#find_package(Threads REQUIRED)

find_package(catkin REQUIRED)

find_package(OpenCV REQUIRED)

catkin_package(
  INCLUDE_DIRS . common
  LIBRARIES ${PROJECT_NAME} ${PROJECT_NAME}-common)

include_directories(
  .
  common
  ${catkin_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS})

add_library(${PROJECT_NAME}-common
  common/g2d.c
  common/getopt.c
  common/homography.c
  common/image_f32.c
  common/image_u8.c
  common/image_u8x3.c
  common/image_u8x4.c
  common/matd.c
  common/pam.c
  common/pjpeg.c
  common/pjpeg-idct.c
  common/pnm.c
  common/string_util.c
  common/svd22.c
  common/time_util.c
  common/unionfind.c
  common/workerpool.c
  common/zarray.c
  common/zhash.c
  common/zmaxheap.c)
target_link_libraries(${PROJECT_NAME}-common pthread m)
set_target_properties(${PROJECT_NAME}-common PROPERTIES COMPILE_FLAGS "-std=gnu99 -Wall -Wno-unused-parameter -Wno-unused-function -pthread -O4 -fno-strict-overflow")

add_library(${PROJECT_NAME}
  apriltag.c
  apriltag_quad_thresh.c
  tag16h5.c
  tag25h9.c
  tag25h7.c
  tag36h10.c
  tag36artoolkit.c
  tag36h11.c)
target_link_libraries(${PROJECT_NAME}
  ${PROJECT_NAME}-common ${catkin_LIBRARIES})
set_target_properties(${PROJECT_NAME} PROPERTIES COMPILE_FLAGS "-std=gnu99 -Wall -Wno-unused-parameter -Wno-unused-function -pthread -O4 -fno-strict-overflow")

add_executable(apriltag_demo example/apriltag_demo.c)
target_link_libraries(apriltag_demo ${PROJECT_NAME} ${catkin_LIBRARIES})
set_target_properties(apriltag_demo PROPERTIES COMPILE_FLAGS "-std=gnu99 -Wall -Wno-unused-parameter -Wno-unused-function -pthread -O4 -fno-strict-overflow")

add_executable(opencv_demo example/opencv_demo.cc)
target_link_libraries(opencv_demo ${PROJECT_NAME} ${catkin_LIBRARIES} ${OpenCV_LIBRARIES})
set_target_properties(opencv_demo PROPERTIES COMPILE_FLAGS "-Wall -pthread -O4")

install(TARGETS ${PROJECT_NAME} ${PROJECT_NAME}-common apriltag_demo opencv_demo
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

# Install the root and common headers:
# Note that we need to exclude the 'debian' folder (from bloom-release) to
# avoid an infinite recursion in the installation process.
# See https://cmake.org/cmake/help/v3.0/command/install.html
# Note that we set the destination to ${CATKIN_GLOBAL_INCLUDE_DESTINATION}
# instead of the common ${CATKIN_PACKAGE_INCLUDE_DESTINATION} (which is equal to
# ${CATKIN_GLOBAL_INCLUDE_DESTINATION}/${PROJECT_NAME}) because we need to
# install the headers on the global/top directory.
install(DIRECTORY ./
  DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h"
  PATTERN "debian" EXCLUDE)

if (CATKIN_ENABLE_TESTING)
  catkin_add_gtest(quad_goodness_test test/quad_goodness_test.cpp)
  target_link_libraries(quad_goodness_test ${PROJECT_NAME} ${catkin_LIBRARIES})
  set_target_properties(quad_goodness_test PROPERTIES COMPILE_FLAGS "-Wall -Wno-unused-parameter -Wno-unused-function -pthread -O4")
endif()
