cmake_minimum_required(VERSION 3.28)

# ------------------------------------------------------------------------------
# Set default build type
# ------------------------------------------------------------------------------
if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR AND NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build." FORCE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

# ------------------------------------------------------------------------------
# Project metadata
# ------------------------------------------------------------------------------
project(dddmp
  VERSION 4.0.0
  DESCRIPTION "DDDMP Library"
  HOMEPAGE_URL "https://github.com/cuddorg/dddmp"
  LANGUAGES C
)

# ------------------------------------------------------------------------------
# Prevent in-source builds
# ------------------------------------------------------------------------------
if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
  message(FATAL_ERROR "In-source builds are not allowed. Please use a separate build directory.")
endif()

option(DDDMP_BUILD_SHARED_LIBS "Build shared library for the DDDMP library" OFF)
option(DDDMP_BUILD_TESTS "Build tests for the DDDMP library" OFF) # TODO

# ------------------------------------------------------------------------------
# Compiler settings
# ------------------------------------------------------------------------------
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)
set(DDDMP_C_STANDARD ${CMAKE_C_STANDARD})
set(DDDMP_C_STANDARD_REQUIRED ${CMAKE_C_STANDARD_REQUIRED})
set(DDDMP_C_EXTENSIONS ${CMAKE_C_EXTENSIONS})

set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# ------------------------------------------------------------------------------
# Headers and sources
# ------------------------------------------------------------------------------
set(DDDMP_PUBLIC_HEADERS
  include/cudd/dddmp.h
)

set(DDDMP_SOURCES
  src/dddmpBinary.c
  src/dddmpConvert.c
  src/dddmpDbg.c
  src/dddmpLoad.c
  src/dddmpLoadCnf.c
  src/dddmpNodeAdd.c
  src/dddmpNodeBdd.c
  src/dddmpNodeCnf.c
  src/dddmpStoreAdd.c
  src/dddmpStoreBdd.c
  src/dddmpStoreCnf.c
  src/dddmpStoreMisc.c
  src/dddmpUtil.c
  src/dddmpInt.h
)

# ------------------------------------------------------------------------------
# Get dependencies
# ------------------------------------------------------------------------------
include(FetchContent)

set(CUDD_BUILD_SHARED_LIBS ${DDDMP_BUILD_SHARED_LIBS} CACHE BOOL "" FORCE)

FetchContent_Declare(
  cudd
  GIT_REPOSITORY https://github.com/cuddorg/cudd.git
  GIT_TAG        pitchfork-layout # 4.0.0-rc1
  SYSTEM
  EXCLUDE_FROM_ALL
  FIND_PACKAGE_ARGS
) 
FetchContent_MakeAvailable(cudd)

# ------------------------------------------------------------------------------
# Library target
# ------------------------------------------------------------------------------

if(DDDMP_BUILD_SHARED_LIBS)
  add_library(dddmp SHARED ${DDDMP_SOURCES})
  add_library(cudd::dddmp_shared ALIAS dddmp)
else()
  add_library(dddmp STATIC ${DDDMP_SOURCES})
  add_library(cudd::dddmp_static ALIAS dddmp)
endif()
add_library(cudd::dddmp ALIAS dddmp)

target_include_directories(dddmp
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  PRIVATE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
)

target_compile_definitions(dddmp
  PUBLIC
    DDDMP_EXPORTS
)

set_target_properties(dddmp PROPERTIES
  OUTPUT_NAME               "dddmp"
  C_STANDARD                "${DDDMP_C_STANDARD}"
  C_STANDARD_REQUIRED       "${DDDMP_C_STANDARD_REQUIRED}"
  C_EXTENSIONS              "${DDDMP_C_EXTENSIONS}"
  VERSION                   "${PROJECT_VERSION}"
  SOVERSION                 "${PROJECT_VERSION_MAJOR}"
  POSITION_INDEPENDENT_CODE ON
  PUBLIC_HEADER             "${DDDMP_PUBLIC_HEADERS}"
)

target_link_libraries(dddmp
  PUBLIC
    cudd::cudd
)

# ------------------------------------------------------------------------------
# Install rules
# ------------------------------------------------------------------------------
include(GNUInstallDirs)

install(TARGETS dddmp
  EXPORT dddmpTargets
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT Runtime
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT Runtime
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT Development
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/cudd COMPONENT Development
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# ------------------------------------------------------------------------------
# Export configuration
# ------------------------------------------------------------------------------
install(EXPORT cuddTargets
  FILE dddmpTargets.cmake
  NAMESPACE cudd::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/cudd
)
