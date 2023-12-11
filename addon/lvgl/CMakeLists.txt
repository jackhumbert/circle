cmake_minimum_required(VERSION 3.16)

set(CMAKE_SYSTEM_NAME Generic)
SET(CMAKE_SYSTEM_VERSION 1)
# set(CMAKE_HOST_SYSTEM_NAME Mac)
set(CMAKE_SYSTEM_PROCESSOR aarch64)
set(CMAKE_CROSSCOMPILING TRUE)

unset(_CMAKE_APPLE_ARCHS_DEFAULT)

set(CMAKE_C_COMPILER "/opt/homebrew/bin/aarch64-none-elf-gcc")
set(CMAKE_CXX_COMPILER "/opt/homebrew/bin/aarch64-none-elf-g++")
set(CMAKE_AR "/opt/homebrew/bin/aarch64-none-elf-ar")
set(CMAKE_LINKER "/opt/homebrew/bin/aarch64-none-elf-ld")

set(CMAKE_C_FLAGS "")
set(CMAKE_CXX_FLAGS "")
set(CMAKE_CXX_LINK_FLAGS "")

add_executable(lvgl_sample lvgl.cpp sample/kernel.cpp sample/main.cpp)

set_target_properties(lvgl_sample PROPERTIES OUTPUT_NAME "lvgl_sample/${TARGET}.elf")

add_custom_command(TARGET lvgl_sample
    POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} ARGS $<TARGET_FILE:lvgl_sample> -O binary lvgl_sample/${TARGET}.img
    WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/bin)

target_link_libraries(lvgl_sample PUBLIC 
  circle 
  lvgl     
  lvgl_demos
  # m
  # c
  # gcc
)
