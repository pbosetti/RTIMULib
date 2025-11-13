#////////////////////////////////////////////////////////////////////////////
#//
#//  This file is part of RTIMULib
#//
#//  Copyright (c) 2014, richards-tech
#//
#//  Permission is hereby granted, free of charge, to any person obtaining a copy of
#//  this software and associated documentation files (the "Software"), to deal in
#//  the Software without restriction, including without limitation the rights to use,
#//  copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the
#//  Software, and to permit persons to whom the Software is furnished to do so,
#//  subject to the following conditions:
#//
#//  The above copyright notice and this permission notice shall be included in all
#//  copies or substantial portions of the Software.
#//
#//  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED,
#//  INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A
#//  PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
#//  HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
#//  OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
#//  SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

#// The cmake support was based on work by Moritz Fischer at ettus.com.
#// Original copyright notice:
#
# Copyright 2014 Ettus Research LLC
#

########################################################################
IF(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})
    MESSAGE(FATAL_ERROR "Prevented in-tree built. This is bad practice.")
ENDIF(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})

execute_process(
  COMMAND git rev-parse --is-inside-work-tree
  WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
  OUTPUT_VARIABLE IS_GIT_REPO
  OUTPUT_STRIP_TRAILING_WHITESPACE
  ERROR_QUIET
)
if(IS_GIT_REPO STREQUAL "true")
  # Get the latest tag
  execute_process(
    COMMAND git describe --tags --dirty
    WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
    OUTPUT_VARIABLE GIT_TAG
    OUTPUT_STRIP_TRAILING_WHITESPACE
    ERROR_QUIET
  )
  if(GIT_TAG MATCHES "^[vV]([0-9]+)\\.([0-9]+)\\.([0-9]+)(.*)?")
    string(REGEX REPLACE "^[vV]([0-9]+)\\.([0-9]+)\\.([0-9]+)(.*)?" "\\1" CPACK_PACKAGE_VERSION_MAJOR "${GIT_TAG}")
    string(REGEX REPLACE "^[vV]([0-9]+)\\.([0-9]+)\\.([0-9]+)(.*)?" "\\2" CPACK_PACKAGE_VERSION_MINOR "${GIT_TAG}")
    string(REGEX REPLACE "^[vV]([0-9]+)\\.([0-9]+)\\.([0-9]+)(.*)?" "\\3" CPACK_PACKAGE_VERSION_PATCH "${GIT_TAG}")
    message(STATUS "Project root is a git repository, using version from latest tag: ${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}")
  else()
    message(WARNING "Latest git tag does not match vMAJOR.MINOR.PATCH format, using default version. Tag is: ${GIT_TAG}")
    set(CPACK_PACKAGE_VERSION_MAJOR 0)
    set(CPACK_PACKAGE_VERSION_MINOR 0)
    set(CPACK_PACKAGE_VERSION_PATCH 1)
  endif()
else()
  message(WARNING "Project root is NOT a git repository, using default version")
  set(CPACK_PACKAGE_VERSION_MAJOR 0)
  set(CPACK_PACKAGE_VERSION_MINOR 0)
  set(CPACK_PACKAGE_VERSION_PATCH 1)
  set(GIT_TAG "vUNKNOWN_GIT_VERSION")
endif()

INCLUDE(CMakeDependentOption)
INCLUDE(FeatureSummary)

########################################################################
# Project setup
########################################################################
CMAKE_MINIMUM_REQUIRED(VERSION 3.10)
PROJECT(RTIMULib CXX)
ENABLE_TESTING()

OPTION(BUILD_GL "Build RTIMULibGL" OFF)
OPTION(BUILD_DRIVE "Build RTIMULibDrive" ON)
OPTION(BUILD_DRIVE10 "Build RTIMULibDrive10" ON)
OPTION(BUILD_DRIVE11 "Build RTIMULibDrive11" ON)
OPTION(BUILD_CAL "Build RTIMULibCal" ON)
OPTION(BUILD_DEMO "Build RTIMULibDemo" OFF)
CMAKE_DEPENDENT_OPTION(BUILD_DEMOGL "Build RTIMULibDemoGL" OFF
                       "BUILD_GL" OFF)

ADD_FEATURE_INFO(RTIMULibGL BUILD_GL "")
ADD_FEATURE_INFO(RTIMULibDrive BUILD_DRIVE "App that shows how to use the RTIMULib library in a basic way.")
ADD_FEATURE_INFO(RTIMULibDrive10 BUILD_DRIVE10 "App that shows to use  pressure/temperature sensors.")
ADD_FEATURE_INFO(RTIMULibDrive11 BUILD_DRIVE11 "App that shows to use  pressure/temperature/humidity sensors.")
ADD_FEATURE_INFO(RTIMULibCal BUILD_CAL "Command line calibration tool for the magnetometers and accelerometers.")
ADD_FEATURE_INFO(RTIMULibDemo BUILD_DEMO "GUI app that displays the fused IMU data in real-time")
ADD_FEATURE_INFO(RTIMULibDemoGL BUILD_DEMOGL "RTIMULibDemo with OpenGL visualization")

INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR}/../RTIMULib)

ADD_SUBDIRECTORY(${CMAKE_CURRENT_SOURCE_DIR}/../RTIMULib RTIMULib)

IF(BUILD_GL)
    INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR}/RTIMULibGL)
    INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR}/RTIMULibGL/QtGLLib)
    INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR}/RTIMULibGL/VRWidgetLib)
    ADD_SUBDIRECTORY(${CMAKE_CURRENT_SOURCE_DIR}/RTIMULibGL)
ENDIF(BUILD_GL)

IF(BUILD_DRIVE)
    ADD_SUBDIRECTORY(RTIMULibDrive)
ENDIF(BUILD_DRIVE)

IF(BUILD_DRIVE10)
    ADD_SUBDIRECTORY(RTIMULibDrive10)
ENDIF(BUILD_DRIVE10)

IF(BUILD_DRIVE11)
    ADD_SUBDIRECTORY(RTIMULibDrive11)
ENDIF(BUILD_DRIVE11)

IF(BUILD_CAL)
    ADD_SUBDIRECTORY(RTIMULibCal)
ENDIF(BUILD_CAL)

IF(BUILD_DEMO)
    ADD_SUBDIRECTORY(RTIMULibDemo)
ENDIF(BUILD_DEMO)

IF(BUILD_DEMOGL)
    ADD_SUBDIRECTORY(RTIMULibDemoGL)
ENDIF(BUILD_DEMOGL)

FEATURE_SUMMARY(WHAT ENABLED_FEATURES DESCRIPTION "Enabled features:")


IF(UNIX)
    # Packaging
    SET(CPACK_PACKAGE_NAME RTIMULib)
    SET(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${GIT_TAG}-${CMAKE_SYSTEM_NAME}-${CMAKE_SYSTEM_PROCESSOR}")
    SET(CPACK_PACKAGE_VENDOR "RTIMULib Developers")
    SET(CPACK_PACKAGE_DIRECTORY ${CMAKE_SOURCE_DIR}/../packages)
    SET(CPACK_STRIP_FILES TRUE)
    SET(CPACK_PACKAGING_INSTALL_PREFIX "/usr/local")
    SET(CPACK_PACKAGE_CONTACT "paolo.bosetti@unitn.it")
    SET(CMAKE_PROJECT_HOMEPAGE_URL "https://github.com/RTIMULib/RTIMULib")
    SET(CPACK_PACKAGE_DESCRIPTION "RTIMULib installer: a library for interfacing with IMU sensors")
    SET(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_LIST_DIR}/../LICENSE")
    SET(CPACK_DEBIAN_PACKAGE_NAME RTIMULib)
    SET(CPACK_GENERATOR "DEB")
    INCLUDE(CPack)
ENDIF(UNIX)

MESSAGE(STATUS "Using install prefix: ${CMAKE_INSTALL_PREFIX}")
