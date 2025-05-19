# Copyright 2025 ReZTI CO LTD
cmake_minimum_required(VERSION 3.5)

string(TIMESTAMP BUILD_TIME "%Y%m%d%H%M")
set(PROJECT_VER "v0.0-${BUILD_TIME}")


if(DEFINED ENV{ESP_MATTER_DEVICE_PATH})
    # Read environment variable into a normal CMake variable
    set(ESP_MATTER_DEVICE_PATH $ENV{ESP_MATTER_DEVICE_PATH})
    message(FATAL_ERROR "set matter device path to ${ESP_MATTER_DEVICE_PATH}")
else()
    set(ENV{ESP_MATTER_DEVICE_PATH}  ${CMAKE_CURRENT_LIST_DIR}/components/device_hal/device/esp32h2_occupancy)
    message(STATUS "matter device path is set to default: $ENV{ESP_MATTER_DEVICE_PATH}")
endif(DEFINED ENV{ESP_MATTER_DEVICE_PATH})


if(DEFINED ENV{RMAKER_PATH})
    set(RMAKER_PATH $ENV{RMAKER_PATH})
else()
    set(RMAKER_PATH ${CMAKE_CURRENT_LIST_DIR}/../../..)
endif(DEFINED ENV{RMAKER_PATH})

if(NOT DEFINED ENV{ESP_MATTER_PATH})
    message(FATAL_ERROR "Please set ESP_MATTER_PATH to the path of esp-matter repo")
endif(NOT DEFINED ENV{ESP_MATTER_PATH})

set(ESP_MATTER_DEVICE_PATH components/device_hal/device/esp32h2_occupancy)

set(ESP_MATTER_PATH $ENV{ESP_MATTER_PATH})
set(MATTER_SDK_PATH ${ESP_MATTER_PATH}/connectedhomeip/connectedhomeip)

# This should be done before using the IDF_TARGET variable.
include($ENV{IDF_PATH}/tools/cmake/project.cmake)
include($ENV{ESP_MATTER_DEVICE_PATH}/esp_matter_device.cmake)

idf_build_set_property(RAINMAKER_ENABLED 1)

set(EXTRA_COMPONENT_DIRS
    "${MATTER_SDK_PATH}/config/esp32/components"
    "${ESP_MATTER_PATH}/components"
#    "${ESP_MATTER_PATH}/device_hal/device"
    "components/device_hal/device"
    "${ESP_MATTER_PATH}/examples/common"
    "${RMAKER_PATH}/examples/common/app_insights"
    "${RMAKER_PATH}/examples/matter/common"
    "${RMAKER_PATH}/components"
    ${extra_components_dirs_append})

# Include insights only for IDF <= v4.4 as it is already included by esp-matter for >= v5.0
if("${IDF_VERSION_MAJOR}.${IDF_VERSION_MINOR}" VERSION_LESS_EQUAL "4.4")
    list(APPEND EXTRA_COMPONENT_DIRS "${RMAKER_PATH}/components/esp-insights/components")
endif()

project(rezti_occupancy_v1)

idf_build_set_property(CXX_COMPILE_OPTIONS "-std=gnu++17;-Os;-DCHIP_HAVE_CONFIG_H" APPEND)
idf_build_set_property(C_COMPILE_OPTIONS "-Os" APPEND)
# For RISCV chips, project_include.cmake sets -Wno-format, but does not clear various
# flags that depend on -Wformat
idf_build_set_property(COMPILE_OPTIONS "-Wno-format-nonliteral;-Wno-format-security" APPEND)
