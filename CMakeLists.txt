# Copyright(c) 2015-2016, NVIDIA CORPORATION. All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#  * Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
#  * Redistributions in binary form must reproduce the above copyright
#    notice, this list of conditions and the following disclaimer in the
#    documentation and/or other materials provided with the distribution.
#  * Neither the name of NVIDIA CORPORATION nor the names of its
#    contributors may be used to endorse or promote products derived
#    from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS ``AS IS'' AND ANY
# EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
# PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR
# CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
# EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
# PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
# PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY
# OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

cmake_minimum_required(VERSION 3.2)

project(VulkanHppGenerator)

get_filename_component(PROJECT_HOME "${CMAKE_SOURCE_DIR}"   ABSOLUTE)
get_filename_component(CMAKE_DIR    "${PROJECT_HOME}/cmake" ABSOLUTE)
get_filename_component(SRC_DIR      "${PROJECT_HOME}/src"   ABSOLUTE)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_DIR})

file(GLOB_RECURSE header_files "${SRC_DIR}/*.h")
file(GLOB_RECURSE source_files "${SRC_DIR}/*.cpp")
list(APPEND executable_sources "${SRC_DIR}/VulkanHppGenerator.cpp")

set(source_files_no_main ${source_files})
list(REMOVE_ITEM source_files_no_main "${executable_sources}")

file(TO_NATIVE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/Vulkan-Docs/src/spec/vk.xml vk_spec)
string(REPLACE "\\" "\\\\" vk_spec ${vk_spec})
add_definitions(-DVK_SPEC="${vk_spec}")

file(TO_NATIVE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/vulkan/vulkan.hpp vulkan_hpp)
string(REPLACE "\\" "\\\\" vulkan_hpp ${vulkan_hpp})
add_definitions(-DVULKAN_HPP="${vulkan_hpp}")

find_package(TinyXML2 REQUIRED)

set(LOCAL_INCLUDE_DIRS ${SRC_DIR})
set(SYSTEM_INCLUDE_DIRS ${TinyXML2_INCLUDE_DIR})
set(SYSTEM_LIBRARIES ${TinyXML2_LIBRARIES})


# compile and link static library (all code excluding mains)
if (source_files_no_main)
    set(CORE_LIB "core")
    add_library(${CORE_LIB} STATIC ${source_files_no_main} ${header_files})
    target_include_directories(${CORE_LIB}        PRIVATE ${LOCAL_INCLUDE_DIRS})
    target_include_directories(${CORE_LIB} SYSTEM PRIVATE ${SYSTEM_INCLUDE_DIRS}) 
    target_link_libraries(${CORE_LIB} PRIVATE ${SYSTEM_LIBRARIES})
    set_property(TARGET ${CORE_LIB} PROPERTY CXX_STANDARD 11)
endif()

# executables
foreach(main_source ${executable_sources})
    get_filename_component(exec_name ${main_source} NAME_WE)
    add_executable(${exec_name} ${main_source})
    set_property(TARGET ${exec_name} PROPERTY CXX_STANDARD 11)
    target_link_libraries(${exec_name} ${SYSTEM_LIBRARIES} ${CORE_LIB})
endforeach()
