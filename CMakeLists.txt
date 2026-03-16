cmake_minimum_required(VERSION 3.8)

project(agriedge)

find_package(ament_cmake REQUIRED)

install(
  DIRECTORY config launch meshes urdf world
  DESTINATION share/${PROJECT_NAME}
)

ament_package()
