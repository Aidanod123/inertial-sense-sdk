set(CMAKE_CXX_STANDARD 17)
cmake_minimum_required(VERSION 3.10.0)
project(inertial_sense_ros2)
#find_package(rosidl_typesupport_c REQUIRED)
find_package(builtin_interfaces REQUIRED)
find_package(std_msgs REQUIRED)
find_package(std_srvs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(yaml-cpp REQUIRED)
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclpy REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(diagnostic_msgs REQUIRED)
find_package(rosidl_default_generators REQUIRED)
find_package(Threads REQUIRED)

add_definitions(-DROS2)

# Resolve the SDK from this file rather than the workspace's optional ros2 symlink.
# Building it as a native target supports fresh and incremental offline builds
# without writing a library into the source tree or activating a Python venv.
get_filename_component(IS_ROS_ROOT "${CMAKE_CURRENT_LIST_DIR}" REALPATH)
get_filename_component(IS_SDK_ROOT "${IS_ROS_ROOT}/../.." REALPATH)
add_subdirectory("${IS_SDK_ROOT}" "${CMAKE_CURRENT_BINARY_DIR}/is_sdk")

rosidl_generate_interfaces(${PROJECT_NAME}
    "msg/GTime.msg"
    "msg/SatInfo.msg"
    "msg/GPS.msg"
    "msg/GPSInfo.msg"
    "msg/PIMU.msg"
    "msg/RTKInfo.msg"
    "msg/RTKRel.msg"
    "msg/GlonassEphemeris.msg"
    "msg/GNSSEphemeris.msg"
    "msg/GNSSObservation.msg"
    "msg/GNSSObsVec.msg"
    "msg/INL2States.msg"
    "msg/DIDINS2.msg"
    "msg/DIDINS1.msg"
    "msg/DIDINS4.msg"
    "srv/FirmwareUpdate.srv"
    "srv/RefLLAUpdate.srv"
    DEPENDENCIES std_msgs geometry_msgs diagnostic_msgs builtin_interfaces
)
include_directories(
    "${IS_ROS_ROOT}/shared/include"
    ${YAML_CPP_INCLUDE_DIR}
)

# Preserve the wrapper's Linux SDK compatibility flags.
SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fms-extensions -Wl,--no-as-needed -DPLATFORM_IS_LINUX" )
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fms-extensions -Wl,--no-as-needed -DPLATFORM_IS_LINUX")

add_executable(${PROJECT_NAME}_node
    src/inertial_sense_ros2.cpp
    shared/src/inertial_sense_node.cpp
    shared/src/ParamHelper.cpp
    shared/src/RtkRover.cpp
    shared/src/RtkBase.cpp
)

rosidl_get_typesupport_target(cpp_typesupport_target "${PROJECT_NAME}" "rosidl_typesupport_cpp")
target_link_libraries(${PROJECT_NAME}_node "${cpp_typesupport_target}")
ament_target_dependencies(${PROJECT_NAME}_node rclcpp std_msgs geometry_msgs nav_msgs diagnostic_msgs sensor_msgs std_srvs)
target_link_libraries(${PROJECT_NAME}_node InertialSenseSDK ${YAML_CPP_LIBRARIES} pthread)


if(BUILD_TESTING)
  find_package(ament_cmake_gtest REQUIRED)
  ament_add_gtest(test_${PROJECT_NAME}
      src/inertial_sense_ros2.cpp
      shared/src/ParamHelper.cpp
      shared/src/RtkRover.cpp
      shared/src/RtkBase.cpp
      test/test_basic_unit_tests.cpp
      test/test_client_reconnect.cpp
      test/test_communications.cpp
      #test/test_main.cpp
      #test/test_main.h
      test/gtest_helpers.h
  )

  rosidl_get_typesupport_target(cpp_typesupport_target "${PROJECT_NAME}" "rosidl_typesupport_cpp")
  target_link_libraries(test_${PROJECT_NAME} "${cpp_typesupport_target}")
  ament_target_dependencies(test_${PROJECT_NAME} rclcpp std_msgs geometry_msgs nav_msgs diagnostic_msgs sensor_msgs std_srvs)
  target_link_libraries(test_${PROJECT_NAME} InertialSenseSDK ${YAML_CPP_LIBRARIES} pthread)

endif()

install(TARGETS ${PROJECT_NAME}_node
        DESTINATION lib/${PROJECT_NAME})

ament_package()
