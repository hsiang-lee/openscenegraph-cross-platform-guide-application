cmake_minimum_required(VERSION 3.4.1)
set(BUILD_SIMULATOR YES CACHE BOOL "Build for simulator. Use false to build for real device")

# Path to external projects:
# * OpenSceneGraph
set(EXT_PROJ_DIR "${CMAKE_SOURCE_DIR}/../..")

# Specify critical OpenSceneGraph build variables.
set(OPENGL_PROFILE "GLES2" CACHE STRING "OpenGL variant to use")
set(DYNAMIC_OPENTHREADS OFF CACHE BOOL "Link OpenThreads statically")
set(DYNAMIC_OPENSCENEGRAPH OFF CACHE BOOL "Link OpenSceneGraph statically")
set(BUILD_OSG_APPLICATIONS OFF CACHE BOOL "Do not build OpenSceneGraph samples")
# Configure OpenSceneGraph for simulator or real device.
if(BUILD_SIMULATOR)
    # Prevent macOS SDK includes that lead to build errors.
    set(CURL_INCLUDE_DIR "" CACHE STRING "Prevent wrong macOS SDK include" FORCE)
    set(ZLIB_INCLUDE_DIR "" CACHE STRING "Prevent wrong macOS SDK include" FORCE)
    set(OSG_BUILD_PLATFORM_IPHONE_SIMULATOR YES CACHE BOOL "Build OSG for simulator")
else()
    set(OSG_BUILD_PLATFORM_IPHONE YES CACHE BOOL "Build OSG for device")
endif()

# Build and reference OpenSceneGraph.
set(OSG_SOURCE_DIR "${EXT_PROJ_DIR}/OpenSceneGraph")
if(BUILD_SIMULATOR)
    set(BUILD_VARIANT "Simulator")
else()
    set(BUILD_VARIANT "Device")
endif()
set(OSG_BUILD_DIR "${EXT_PROJ_DIR}/OpenSceneGraph/build/${BUILD_VARIANT}")
file(MAKE_DIRECTORY ${OSG_BUILD_DIR})
add_subdirectory(${OSG_SOURCE_DIR} ${OSG_BUILD_DIR})
include_directories(${OSG_SOURCE_DIR}/include)
include_directories(${OSG_BUILD_DIR}/include)

# Reference OpenSceneGraph cross-platform guide application.
include_directories("${EXT_PROJ_DIR}/openscenegraph-cross-platform-guide-application/ios/src")
include_directories("${EXT_PROJ_DIR}/openscenegraph-cross-platform-guide-application/ios/src-gen")

# Build osgNativeLib.
add_library(osgNativeLib STATIC src/osgNativeLib.cpp)
# WARNING: This does not actually link anything. However, this command
# WARNING: makes sure all necessary libraries are built.
target_link_libraries(
    osgNativeLib
    osgViewer
    osgDB
    # osgDB plugins start.
    osgdb_osg
    osgdb_serializers_osg
    # osgDB plugins end.
    osgGA
    osgShadow
    osgText
    osgUtil
    osg
    OpenThreads
)

# Generate single osglib after osgNativeLib has been built.
add_custom_command(
    target osgNativeLib
    post_build
    command "${CMAKE_SOURCE_DIR}/create_single_osglib" "." "${OSG_BUILD_DIR}")