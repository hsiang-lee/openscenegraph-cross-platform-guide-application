project(SAMPLE_APPLICATION)
cmake_minimum_required(VERSION 3.28)

# OpenSceneGraph directories.
set(OSG_INC_DIR "" CACHE PATH "Path to OpenSceneGraph include directory")
set(OSG_LIB_DIR "" CACHE PATH "Path to OpenSceneGraph lib directory")
include_directories(${OSG_INC_DIR})
link_directories(${OSG_LIB_DIR})

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Reference own inlcudes.
include_directories(${CMAKE_SOURCE_DIR}/desktop/src-gen)
# Create executable.
add_executable(sampleApplication desktop/src/main.cpp)
# Link executable with OpenSceneGraph libraries.
target_link_libraries(sampleApplication osg osgDB osgGA osgViewer)