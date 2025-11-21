cmake_minimum_required(VERSION 3.10)

# Path to OpenSceneGraph
set(EXT_PROJ_DIR "${CMAKE_SOURCE_DIR}/../..")

# Specify critical OpenSceneGraph build variables.
set(BUILD_OSG_APPLICATIONS NO CACHE BOOL "Do not build applications")
set(EGL_LIBRARY "GL" CACHE STRING "Suppress linkage error")
set(OSG_GL1_AVAILABLE OFF CACHE BOOL "Unavailable under Emscripten")
set(OSG_GL2_AVAILABLE OFF CACHE BOOL "Unavailable under Emscripten")
set(OSG_GLES2_AVAILABLE ON CACHE BOOL "GLES2 is what Emscripten uses")
set(OSG_WINDOWING_SYSTEM "None" CACHE STRING "Since we use SDL2 for OpenGL context creation, we don't have a windowing system")
set(DYNAMIC_OPENTHREADS OFF CACHE BOOL "Link OpenThreads statically")
set(DYNAMIC_OPENSCENEGRAPH OFF CACHE BOOL "Link OpenSceneGraph statically")
# Prevent CMake error during configuration.
set(_OPENTHREADS_ATOMIC_USE_GCC_BUILTINS_EXITCODE "0" CACHE STRING "Prevent cfg error")

# Reference SDL2 during build process.
# We use SDL2 to do the following:
# * OpenGL functions' address retrieval
# * OpenGL graphics context creation
set(FLAGS "-s USE_SDL=2")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} ${FLAGS}" CACHE STRING "Use SDL2" FORCE)
# Build and reference OpenSceneGraph.
set(OSG_SOURCE_DIR "${EXT_PROJ_DIR}/OpenSceneGraph")
set(OSG_BUILD_DIR "${EXT_PROJ_DIR}/OpenSceneGraph/build/Emscripten")
file(MAKE_DIRECTORY ${OSG_BUILD_DIR})
add_subdirectory(${OSG_SOURCE_DIR} ${OSG_BUILD_DIR})

include_directories(${OSG_SOURCE_DIR}/include)
include_directories(${OSG_BUILD_DIR}/include)
link_directories(${OSG_BUILD_DIR}/lib)

# Build sample.
include_directories(src-gen)
add_executable(sample-ems src/main.cpp)
# Make Emscripten generate ready-to-open HTML page.
set(CMAKE_EXECUTABLE_SUFFIX ".html")
# Make Emscripten preload the resource.
set_target_properties(sample-ems PROPERTIES LINK_FLAGS "--preload-file box.osgt")
# Copy the resource to the build directory.
configure_file(box.osgt box.osgt COPYONLY)
# Libraries must be linked in the specified order.
# Otherwise you may get unsatisified linker errors.
target_link_libraries(
    sample-ems
    osgViewer
    osgDB
    # osgDB plugins start.
    osgdb_osg
    osgdb_serializers_osg
    # osgDB plugins end.
    osgGA
    osgText
    osgUtil
    osg
    OpenThreads
)
