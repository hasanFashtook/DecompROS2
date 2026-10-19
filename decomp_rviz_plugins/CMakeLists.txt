################################################################################
# Set minimum required version of cmake, project name and compile options
################################################################################
cmake_minimum_required(VERSION 3.16)
project(decomp_rviz_plugins)


set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17 -Wall -Wno-deprecated-declarations")
find_package(ament_cmake REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(PCL REQUIRED)
find_package(decomp_util REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rviz_common REQUIRED)
find_package(rviz_default_plugins REQUIRED)
find_package(rviz_ogre_vendor REQUIRED)
find_package(Qt6 QUIET COMPONENTS Widgets Core)
if(NOT Qt6_FOUND)
  find_package(Qt5 REQUIRED COMPONENTS Widgets Core)
endif()
find_package(decomp_ros_msgs REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(pcl_conversions REQUIRED)
find_package(rcpputils)
find_package(pluginlib)

include_directories(${EIGEN3_INCLUDE_DIRS} ${DECOMP_UTIL_INCLUDE_DIRS}  ${PCL_INCLUDE_DIRS})

add_definitions(-DQT_NO_KEYWORDS)
link_directories(${PCL_LIBRARY_DIRS})
add_definitions(${PCL_DEFINITIONS})
set(CMAKE_AUTOMOC ON)

include_directories(
  include
  ${rclcpp_INCLUDE_DIRS}
  ${rviz_common_INCLUDE_DIRS}
)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(decomp_rviz_plugins_headers_to_moc
  include/${PROJECT_NAME}/bound_visual.hpp
  include/${PROJECT_NAME}/ellipsoid_array_display.hpp
  include/${PROJECT_NAME}/mesh_visual.hpp
  include/${PROJECT_NAME}/mesh_shape.hpp
  include/${PROJECT_NAME}/vector_visual.hpp
  include/${PROJECT_NAME}/data_ros_utils.hpp
  include/${PROJECT_NAME}/ellipsoid_array_visual.hpp
  include/${PROJECT_NAME}/polyhedron_array_display.hpp
)

set(decomp_rviz_plugins_source_files
  src/bound_visual.cpp
  src/mesh_shape.cpp 
  src/mesh_visual.cpp
  src/vector_visual.cpp
  src/ellipsoid_array_visual.cpp
  src/ellipsoid_array_display.cpp
  src/polyhedron_array_display.cpp
  ${decomp_rviz_plugins_headers_to_moc}
)

add_library(${PROJECT_NAME} SHARED ${decomp_rviz_plugins_source_files})

target_include_directories(${PROJECT_NAME} PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
            $<INSTALL_INTERFACE:include/${PROJECT_NAME}>
)

if(TARGET Qt6::Widgets)
  target_link_libraries(${PROJECT_NAME} PUBLIC Qt6::Widgets Qt6::Core)
elseif(TARGET Qt5::Widgets)
  target_link_libraries(${PROJECT_NAME} PUBLIC Qt5::Widgets Qt5::Core)
endif()

target_link_libraries(${PROJECT_NAME} PUBLIC
            rviz_ogre_vendor::OgreMain
            rviz_ogre_vendor::OgreOverlay
            rviz_common::rviz_common ${PCL_LIBRARIES}
)

target_compile_definitions(${PROJECT_NAME}
                 PRIVATE "RVIZ_DEFAULT_PLUGINS_BUILDING_LIBRARY")

target_link_libraries(${PROJECT_NAME}
        PUBLIC
        rclcpp::rclcpp
        rclcpp_components::component
        rviz_common::rviz_common
        rviz_default_plugins::rviz_default_plugins
        rviz_rendering::rviz_rendering
        ${decomp_ros_msgs_TARGETS}
        tf2::tf2
        tf2_ros::tf2_ros
        sensor_msgs::sensor_msgs__rosidl_generator_c
        sensor_msgs::sensor_msgs__rosidl_typesupport_c
        sensor_msgs::sensor_msgs__rosidl_typesupport_cpp
        pcl_conversions::pcl_conversions
        nav_msgs::nav_msgs__rosidl_generator_c
        nav_msgs::nav_msgs__rosidl_typesupport_c
        nav_msgs::nav_msgs__rosidl_typesupport_cpp
)

## Here we export the plugins and meshes so they can be found by RViz at runtime.
pluginlib_export_plugin_description_file(rviz_common plugin_description.xml)

ament_export_include_directories("include/${PROJECT_NAME}")

# Export modern CMake targets
ament_export_targets(${PROJECT_NAME} HAS_LIBRARY_TARGET)

## Install
ament_export_dependencies(${PROJECT_NAME}
  geometry_msgs
  rclcpp
  rviz_common
  sensor_msgs
  rviz_default_plugins
  nav_msgs
  rviz_rendering
  pcl_conversions)

install(TARGETS ${PROJECT_NAME}
  EXPORT ${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

install(
  DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/icons"
  DESTINATION "share/${PROJECT_NAME}"
)

install(
  DIRECTORY include/
  DESTINATION include
)


ament_package()

