# Copyright (C) 2018 Intel Corporation
#
#
# SPDX-License-Identifier: Apache-2.0
#

cmake_minimum_required(VERSION 3.2)

project(ade)

option(ENABLE_TESTING      "Build tests, require google test" OFF)
option(BUILD_TUTORIAL      "Build tutorial samples"           OFF)
option(FORCE_ADE_ASSERTS   "Always enable ADE_ASSERT"         OFF)
option(BUILD_DOCUMENTATION "Build doxygen documentation"      OFF)

# TODO: this is horrible hack, we must follow cmake
# build/install policy
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

function(add_security_flags target)
    if(UNIX)
        target_compile_definitions( ${target} PRIVATE _FORTIFY_SOURCE=2 )
        if( ${CMAKE_CXX_COMPILER_ID} STREQUAL "GNU")
            if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS 4.9)
                target_compile_options( ${target} PRIVATE -fstack-protector)
            else()
                target_compile_options( ${target} PRIVATE -fstack-protector-strong)
            endif()
        elseif(${CMAKE_CXX_COMPILER_ID} STREQUAL "Intel")
            target_compile_options( ${target} PRIVATE -fstack-protector-strong)
        endif()
    elseif(WIN32)
        target_compile_options( ${target} PRIVATE /GS /DynamicBase)
        if("${CMAKE_SIZEOF_VOID_P}" STREQUAL "4")
            # These options for 32 bit builds only
            target_compile_options( ${target} PRIVATE /SAFESEH /NXCOMPAT )
        endif()
    endif()
endfunction()

set(CMAKE_CXX_STANDARD 11)

if (MSVC)
    add_definitions(-D_SCL_SECURE_NO_WARNINGS)
endif(MSVC)

add_subdirectory(sources/ade/)

if(ENABLE_TESTING)
    enable_testing()
    add_subdirectory(sources/tests)
endif()

if(BUILD_TUTORIAL)
    add_subdirectory(tutorial)
endif()

# cpack

if(CMAKE_BUILD_TYPE STREQUAL "Release")
  set(CPACK_STRIP_FILES ON)
endif()
set(CPACK_PACKAGE_VERSION ${THE_PROJECT_VERSION})
set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY OFF)
set(CPACK_ARCHIVE_COMPONENT_INSTALL ON)
set(CPACK_GENERATOR "ZIP")

include(CPack)
