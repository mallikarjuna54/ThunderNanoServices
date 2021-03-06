# If not stated otherwise in this file or this component's LICENSE file the
# following copyright and licenses apply:
#
# Copyright 2020 Metrological
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

project(PlayerInfo)

cmake_minimum_required(VERSION 3.3)

find_package(WPEFramework)

project_version(1.0.0)

set(MODULE_NAME ${NAMESPACE}${PROJECT_NAME})

message("Setup ${MODULE_NAME} v${PROJECT_VERSION}")

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

option(DOLBY_SUPPORT "Build the PlayerInfo with dolby support" OFF)
set(PLUGIN_PLAYERINFO_AUTOSTART true CACHE STRING "Automatically start PlayerInfo plugin")

find_package(${NAMESPACE}Plugins REQUIRED)
find_package(${NAMESPACE}Definitions REQUIRED)
find_package(CompileSettingsDebug CONFIG REQUIRED)

add_library(${MODULE_NAME} SHARED
    PlayerInfo.cpp
    Module.cpp)

set_target_properties(${MODULE_NAME} PROPERTIES
        CXX_STANDARD 11
        CXX_STANDARD_REQUIRED YES)

target_link_libraries(${MODULE_NAME}
    PRIVATE
        CompileSettingsDebug::CompileSettingsDebug
        ${NAMESPACE}Definitions::${NAMESPACE}Definitions
        ${NAMESPACE}Plugins::${NAMESPACE}Plugins)

find_package(GSTREAMER REQUIRED)

if (DOLBY_SUPPORT)

    target_sources(${MODULE_NAME}
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}/Dolby/DolbyOutput.cpp)

    target_include_directories(${MODULE_NAME}
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}/Dolby/include)

    target_compile_definitions(${MODULE_NAME}
        PRIVATE
            DOLBY_SUPPORT=1)

    add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/Dolby)

    target_link_libraries(${MODULE_NAME}
        PRIVATE
            PlayerInfoDolby)
endif()

if (USE_IARM)
    find_package(DS REQUIRED)
    find_package(IARMBus REQUIRED)
    target_include_directories(${MODULE_NAME} PRIVATE
        ${IARMBUS_INCLUDE_DIRS}
        ${DS_INCLUDE_DIRS}
        ../helpers)
    target_link_libraries(${MODULE_NAME} PRIVATE
        ${NAMESPACE}Plugins::${NAMESPACE}Plugins
        ${IARMBUS_LIBRARIES}
        ${DS_LIBRARIES})
    target_sources(${MODULE_NAME}
        PRIVATE
            IARM/PlatformImplementation.cpp
            ../helpers/utils.cpp)
elseif (GSTREAMER_FOUND)
    target_sources(${MODULE_NAME}
        PRIVATE
            GStreamer/PlatformImplementation.cpp)

    target_link_libraries(${MODULE_NAME}
        PRIVATE
            ${GSTREAMER_LIBRARIES})

    target_include_directories(${MODULE_NAME}
        PRIVATE
            ${GSTREAMER_INCLUDES})
endif ()

install(TARGETS ${MODULE_NAME}
    DESTINATION lib/${STORAGE_DIRECTORY}/plugins)

write_config()
