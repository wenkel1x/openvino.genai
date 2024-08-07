# Copyright (C) 2018-2024 Intel Corporation
# SPDX-License-Identifier: Apache-2.0
#

cmake_minimum_required(VERSION 3.23.0)  # The requirement comes from Jinja2Cpp

# Multi config generators such as Visual Studio ignore CMAKE_BUILD_TYPE. Multi config generators are configured with
# CMAKE_CONFIGURATION_TYPES, but limiting options in it completely removes such build options
get_property(GENERATOR_IS_MULTI_CONFIG_VAR GLOBAL PROPERTY GENERATOR_IS_MULTI_CONFIG)
if(CMAKE_GENERATOR STREQUAL "Ninja Multi-Config")
    # 'Ninja Multi-Config' specific, see:
    # https://cmake.org/cmake/help/latest/variable/CMAKE_DEFAULT_BUILD_TYPE.html
    set(CMAKE_DEFAULT_BUILD_TYPE "Release" CACHE STRING "CMake default build type")
elseif(NOT GENERATOR_IS_MULTI_CONFIG_VAR AND NOT DEFINED CMAKE_BUILD_TYPE)
    message(STATUS "CMAKE_BUILD_TYPE is not defined, 'Release' will be used")
    # Setting CMAKE_BUILD_TYPE as CACHE must go before project(). Otherwise project() sets its value and set() doesn't take an effect
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel ...")
endif()

project(OpenVINOGenAI
        VERSION 2024.3.0.0
        DESCRIPTION "OpenVINO GenAI"
        HOMEPAGE_URL "https://github.com/openvinotoolkit/openvino.genai"
        LANGUAGES CXX)

# Find OpenVINODeveloperPackage first to compile with SDL flags
find_package(OpenVINODeveloperPackage QUIET
             PATHS "${OpenVINO_DIR}")
if(NOT OpenVINODeveloperPackage_FOUND)
    find_package(OpenVINO REQUIRED COMPONENTS Runtime)
endif()

# check that SDPA to PA transformtion exists
get_target_property(ov_include_dirs openvino::runtime INTERFACE_INCLUDE_DIRECTORIES)
find_file(spda_to_pa_header sdpa_to_paged_attention.hpp
    PATHS ${ov_include_dirs}
    PATH_SUFFIXES openvino/pass
    DOC "Path to sdpa_to_paged_attention.hpp header"
    NO_CACHE REQUIRED NO_DEFAULT_PATH)

include(cmake/features.cmake)

add_subdirectory(thirdparty)
add_subdirectory(src)
add_subdirectory(samples)
add_subdirectory(tests/cpp)

install(FILES LICENSE DESTINATION licensing COMPONENT licensing_genai RENAME LICENSE-GENAI)
install(FILES third-party-programs.txt DESTINATION licensing COMPONENT licensing_genai RENAME third-party-programs-genai.txt)
set(CPACK_ARCHIVE_COMPONENT_INSTALL ON)
set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY OFF)
include(CPack)
