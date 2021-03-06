cmake_minimum_required(VERSION 2.8)
project(pilotguru)

include(ExternalProject)
ExternalProject_Add(can-utils
  PREFIX            ${CMAKE_BINARY_DIR}/can-utils
  GIT_REPOSITORY    https://github.com/waiwnf/can-utils.git
  GIT_TAG           master
  UPDATE_COMMAND    ""
  CONFIGURE_COMMAND autoreconf --install -Wall && ./configure --prefix=<INSTALL_DIR>
  BUILD_IN_SOURCE   1
)
ExternalProject_Get_Property(can-utils source_dir)
add_library(socketcanlib STATIC IMPORTED)
set_property(TARGET socketcanlib PROPERTY IMPORTED_LOCATION ${source_dir}/.libs/libcan.a)
include_directories(
  ${source_dir}/include
)

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Release)
ENDIF()

MESSAGE("Build type: " ${CMAKE_BUILD_TYPE})
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall -O3 -march=native -g")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -O3 -march=native -g -std=c++11")

add_subdirectory(thirdparty/DBoW2)
add_subdirectory(thirdparty/g2o)
add_subdirectory(thirdparty/orb-slam2)

include_directories(
${PROJECT_SOURCE_DIR}/include
${PROJECT_SOURCE_DIR}/thirdparty/orb-slam2/include
${PROJECT_SOURCE_DIR}/thirdparty/DBoW2
${PROJECT_SOURCE_DIR}/thirdparty/g2o
${PROJECT_SOURCE_DIR}/thirdparty/LBFGS
${EIGEN3_INCLUDE_DIR}
${CMAKE_CURRENT_BINARY_DIR}/thirdparty/orb-slam2  # For generated proto files.
)

find_package(OpenCV 2.4.3 REQUIRED)

add_library(pilotgurulib SHARED
  ${PROJECT_SOURCE_DIR}/src/calibration/rotation.cc
  ${PROJECT_SOURCE_DIR}/src/calibration/velocity.cc
  ${PROJECT_SOURCE_DIR}/src/geometry/geometry.cc
  ${PROJECT_SOURCE_DIR}/src/io/image_sequence_reader.cc
  ${PROJECT_SOURCE_DIR}/src/io/image_sequence_writer.cc
  ${PROJECT_SOURCE_DIR}/src/interpolation/align_time_series.cc
  ${PROJECT_SOURCE_DIR}/src/io/json_converters.cc
  ${PROJECT_SOURCE_DIR}/src/optimization/gradient_descent.cc
  ${PROJECT_SOURCE_DIR}/src/slam/horizontal_flatten.cc
  ${PROJECT_SOURCE_DIR}/src/slam/track_image_sequence.cc
  ${PROJECT_SOURCE_DIR}/src/slam/smoothing.cc
)

add_library(kiacanlib SHARED ${PROJECT_SOURCE_DIR}/src/car/kia_can.cc)
add_library(canlib SHARED ${PROJECT_SOURCE_DIR}/src/car/can.cc)

add_executable(annotate_frames src/annotate_frames.cc)
add_executable(calibrate src/calibrate.cc)
add_executable(fit_motion src/fit_motion.cc)
add_executable(integrate_motion src/integrate_motion.cc)
add_executable(interpolate_velocity src/interpolate_velocity.cc)
add_executable(kia_steering_angle src/kia_steering_angle.cc)
add_executable(optical_trajectories src/optical_trajectories.cc)
add_executable(project_translations src/project_translations.cc)
add_executable(render_motion src/render_motion.cc)
add_executable(smooth_heading_directions src/smooth_heading_directions.cc)

add_dependencies(pilotgurulib ORB_SLAM2)
add_dependencies(annotate_frames pilotgurulib)
add_dependencies(fit_motion pilotgurulib ORB_SLAM2)
add_dependencies(integrate_motion pilotgurulib ORB_SLAM2)
add_dependencies(interpolate_velocity pilotgurulib ORB_SLAM2)
add_dependencies(kia_steering_angle socketcanlib)
add_dependencies(optical_trajectories pilotgurulib ORB_SLAM2)
add_dependencies(project_translations pilotgurulib ORB_SLAM2)
add_dependencies(render_motion pilotgurulib ORB_SLAM2)
add_dependencies(smooth_heading_directions pilotgurulib ORB_SLAM2)

target_link_libraries(annotate_frames pilotgurulib ORB_SLAM2 ${OpenCV_LIBS} glog gflags)
target_link_libraries(calibrate ${OpenCV_LIBS} glog gflags)
target_link_libraries(fit_motion pilotgurulib ORB_SLAM2 glog gflags)
target_link_libraries(integrate_motion pilotgurulib ORB_SLAM2 glog gflags)
target_link_libraries(interpolate_velocity pilotgurulib ORB_SLAM2 glog gflags)
target_link_libraries(kia_steering_angle canlib kiacanlib socketcanlib glog gflags)
target_link_libraries(optical_trajectories pilotgurulib ORB_SLAM2 ${OpenCV_LIBS} glog gflags)
target_link_libraries(project_translations pilotgurulib ORB_SLAM2 ${OpenCV_LIBS} glog gflags)
target_link_libraries(render_motion pilotgurulib ORB_SLAM2 ${OpenCV_LIBS} glog gflags)
target_link_libraries(smooth_heading_directions pilotgurulib ORB_SLAM2 glog gflags)
