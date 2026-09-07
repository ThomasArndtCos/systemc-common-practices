cmake_minimum_required(VERSION 3.14 FATAL_ERROR)
project(scp VERSION 1.0 LANGUAGES CXX C)

set(CMAKE_CXX_STANDARD 14 CACHE STRING "C++ standard to build all targets.")

set(GITHUB "https://github.com/" CACHE STRING "github base url")

include(FetchContent)
include(CTest)

FetchContent_Declare(
  cpm-cmake
  GIT_REPOSITORY ${GITHUB}cpm-cmake/CPM.cmake.git
  GIT_SHALLOW True
  GIT_TAG v0.31.1
)

FetchContent_MakeAvailable(cpm-cmake)
include(${cpm-cmake_SOURCE_DIR}/cmake/CPM.cmake)

cpmaddpackage("${GITHUB}TheLartians/PackageProject.cmake.git@1.4.1")

# Prefer an already-installed SystemC.
if(DEFINED ENV{SYSTEMC_HOME})
  if(DEFINED SystemCLanguage_DIR)
    list(PREPEND CMAKE_PREFIX_PATH SystemCLanguage_DIR)
  endif()
endif()
find_package(SystemCLanguage CONFIG QUIET)
if(NOT SystemCLanguage_FOUND)
  message(STATUS "SystemC not found locally; fetching it from Git")
  cpmaddpackage(
    NAME SystemCLanguage
    GIT_REPOSITORY ${GITHUB}accellera-official/systemc.git
    GIT_SHALLOW True
    GIT_TAG main
  )
else()
  message(STATUS "Using installed SystemC from ${SystemCLanguage_DIR}")
endif()

# Prefer an already-installed CCI.
if(DEFINED ENV{CCI_HOME})
  if(DEFINED SystemCCCI_DIR)
    list(PREPEND CMAKE_PREFIX_PATH SystemCCCI_DIR)
  endif()
endif()
find_package(SystemCCCI CONFIG QUIET)
if(NOT SystemCCCI_FOUND)
  message(STATUS "CCI not found locally; fetching it from Git")
  cpmaddpackage(
    NAME SystemCCCI
    GIT_REPOSITORY  ${GITHUB}accellera-official/cci.git
    GIT_SHALLOW True
    GIT_TAG main
  )
else()
  message(STATUS "Using installed CCI from ${SystemCCCI_DIR}")
endif()

cpmaddpackage(
  NAME initiator_id
  SOURCE_DIR ${PROJECT_SOURCE_DIR}/tlm_extensions/initiator_id
)
cpmaddpackage(
  NAME path_trace
  SOURCE_DIR ${PROJECT_SOURCE_DIR}/tlm_extensions/path_trace
)
cpmaddpackage(
  NAME report
  SOURCE_DIR ${PROJECT_SOURCE_DIR}/report
)

if("${PROJECT_NAME}" STREQUAL "${CMAKE_PROJECT_NAME}")
  add_subdirectory(examples)
  if(BUILD_TESTING)
    enable_testing()
  endif()
endif()
