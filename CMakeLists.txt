cmake_minimum_required(VERSION 3.5)
project(webrtc-build)

if ("${WEBRTC_ROOT_PATH}" STREQUAL "" )
    message(FATAL_ERROR "\"$\{WEBRTC_ROOT_PATH\}\" can not be empty")
endif()

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH}
    ${CMAKE_SOURCE_DIR}/CMakeModules)

find_package(Git REQUIRED)
find_package(PythonInterp 2.7 REQUIRED)

include(Options)
include(TargetOsAndCpu)

if (HAS_OWN_DEPOT_TOOLS)
  find_package(DepotTools REQUIRED)
endif (HAS_OWN_DEPOT_TOOLS)

include(ExternalProject)
if (NOT HAS_OWN_DEPOT_TOOLS)
  ExternalProject_Add(
      depot-tools

      GIT_REPOSITORY https://chromium.googlesource.com/chromium/tools/depot_tools

      PREFIX ${CMAKE_BINARY_DIR}/depot_tools

      CONFIGURE_COMMAND ""
      UPDATE_COMMAND ""
      PATCH_COMMAND ""
      BUILD_COMMAND ""
      INSTALL_COMMAND ""
  )

  set(_NEXT_DEPENDS depot-tools)
  set(DEPOT_TOOLS_PATH ${CMAKE_BINARY_DIR}/depot_tools/src/depot-tools)
endif (NOT HAS_OWN_DEPOT_TOOLS)

include(Gn)
include(LibWebRTCCommand)

libwebrtc_command(
    NAME webrtc-generate
    COMMAND ${_GEN_COMMAND}
    WORKING_DIRECTORY "${WEBRTC_ROOT_PATH}"
    COMMENT "Generating build files"
    DEPENDS ${_NEXT_DEPENDS}
)

set(_NINJA_COMMAND ninja ${NINJA_ARGS} -C ${_NINJA_BUILD_DIR})

libwebrtc_command(
    NAME webrtc-build
    COMMAND ${_NINJA_COMMAND}
    WORKING_DIRECTORY "${WEBRTC_ROOT_PATH}"
    COMMENT "Running ninja"
    DEPENDS webrtc-generate
)

ExternalProject_Add(
    libwebrtc
    DEPENDS webrtc-build

    INSTALL_COMMAND ""

    INSTALL_DIR ${CMAKE_BINARY_DIR}
    SOURCE_DIR ${CMAKE_SOURCE_DIR}/libwebrtc
    BINARY_DIR ${CMAKE_BINARY_DIR}/libwebrtc

    CMAKE_ARGS
    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_BINARY_DIR}
    -DCMAKE_MODULE_PATH:PATH=${CMAKE_MODULE_PATH}
    -DINSTALL_CMAKE_DIR:PATH=${CMAKE_BINARY_DIR}/lib/cmake/LibWebRTC
    -DTARGET_OS:STRING=${TARGET_OS}
    -DWEBRTC_OUTPUT_DIR:PATH=${CMAKE_BINARY_DIR}/webrtc/src/out/${_CONFIG}
    -DWEBRTC_SOURCE_DIR:PATH=${CMAKE_BINARY_DIR}/webrtc/src/webrtc
)


  
