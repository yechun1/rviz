cmake_minimum_required(VERSION 3.5)

project(rviz_assimp_vendor)

find_package(ament_cmake REQUIRED)

# TODO(wjwood): remove when windows build is fixed
if(DEFINED ENV{RVIZ_BUILD_ON_WINDOWS_OVERRIDE})
  set(RVIZ_BUILD_ON_WINDOWS_OVERRIDE "$ENV{RVIZ_BUILD_ON_WINDOWS_OVERRIDE}")
endif()
if(WIN32 AND NOT DEFINED RVIZ_BUILD_ON_WINDOWS_OVERRIDE)
  message(STATUS "${PROJECT_NAME} is not supported on Windows, skipping...")
  return()
endif()

set(PACKAGE_VERSION "1.0.0")

macro(build_assimp)
  set(extra_cmake_args)
  if(DEFINED CMAKE_BUILD_TYPE)
    list(APPEND extra_cmake_args -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE})
  endif()
  if(WIN32)
    list(APPEND extra_cmake_args "-DCMAKE_CXX_FLAGS=/w")
    list(APPEND extra_cmake_args "-DCMAKE_C_FLAGS=/w")
  else()
    list(APPEND extra_cmake_args "-DCMAKE_CXX_FLAGS=-std=c++14 -w")
    list(APPEND extra_cmake_args "-DCMAKE_C_FLAGS=-w")
  endif()

  list(APPEND extra_cmake_args "-DASSIMP_BUILD_ASSIMP_TOOLS=OFF")
  list(APPEND extra_cmake_args "-DASSIMP_BUILD_TESTS=OFF")
  list(APPEND extra_cmake_args "-DASSIMP_BUILD_SAMPLES=OFF")

  include(ExternalProject)
  ExternalProject_Add(assimp-4.1.0
    URL https://github.com/assimp/assimp/archive/v4.1.0.tar.gz
    URL_MD5 83b53a10c38d964bd1e69da0606e2727
    TIMEOUT 600
    CMAKE_ARGS
      -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}/assimp_install
      ${extra_cmake_args}
      -Wno-dev
  )

  # The external project will install to the build folder, but we'll install that on make install.
  install(
    DIRECTORY
      ${CMAKE_CURRENT_BINARY_DIR}/assimp_install/
    DESTINATION
      ${CMAKE_INSTALL_PREFIX}/opt/rviz_assimp_vendor
  )
endmacro()

find_package(assimp QUIET)

if(NOT assimp_FOUND OR "${assimp_VERSION}" VERSION_LESS 4.1.0)
  build_assimp()

  if(WIN32)
    # TODO(wjwwood): do this for Windows too
    # ament_environment_hooks(env_hook/rviz_assimp_vendor_library_path.bat)
  else()
    ament_environment_hooks(env_hook/rviz_assimp_vendor_library_path.sh)
  endif()
endif()

ament_package(
  CONFIG_EXTRAS "rviz_assimp_vendor-extras.cmake.in"
)
