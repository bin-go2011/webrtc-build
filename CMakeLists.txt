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

  
