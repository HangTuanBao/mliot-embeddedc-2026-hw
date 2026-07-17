cmake_minimum_required(VERSION 3.22)

# Set the project name
set(CMAKE_PROJECT_NAME app_firmware)

# Define the build type
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Debug")
endif()

# Include toolchain file
include("cmake/gcc-arm-none-eabi.cmake")

# Enable compile command to ease indexing with e.g. clangd
set(CMAKE_EXPORT_COMPILE_COMMANDS TRUE)

# Core project settings
project(${CMAKE_PROJECT_NAME})
message("Build type: " ${CMAKE_BUILD_TYPE})

# Enable CMake support for ASM and C languages
enable_language(C ASM)

# Create an executable object type
add_executable(${CMAKE_PROJECT_NAME})

# Project defines (symbols)
set(PROJECT_DEFINES 
    STM32F103xB
    USE_HAL_DRIVER
)

# Project include paths (all files are local now)
set(PROJECT_INCLUDES
    ${CMAKE_SOURCE_DIR}
)

# Application source files
set(PROJECT_SOURCES
    ${CMAKE_SOURCE_DIR}/main.c
    ${CMAKE_SOURCE_DIR}/startup_stm32f103xb.s
    ${CMAKE_SOURCE_DIR}/system_stm32f1xx.c
    ${CMAKE_SOURCE_DIR}/stm32f1xx_hal.c
    ${CMAKE_SOURCE_DIR}/stm32f1xx_hal_gpio.c
    ${CMAKE_SOURCE_DIR}/stm32f1xx_hal_cortex.c
    ${CMAKE_SOURCE_DIR}/stm32f1xx_hal_rcc.c
    ${CMAKE_SOURCE_DIR}/stm32f1xx_hal_rcc_ex.c
)

# Add sources to target
target_sources(${CMAKE_PROJECT_NAME} PRIVATE
    ${PROJECT_SOURCES}
)

# Add include paths to target
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE
    ${PROJECT_INCLUDES}
)

# Add compile definitions to target
target_compile_definitions(${CMAKE_PROJECT_NAME} PRIVATE
    ${PROJECT_DEFINES}
)

# Post-build commands to generate hex and bin files
add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O ihex $<TARGET_FILE:${CMAKE_PROJECT_NAME}> ${CMAKE_PROJECT_NAME}.hex
    COMMAND ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:${CMAKE_PROJECT_NAME}> ${CMAKE_PROJECT_NAME}.bin
    COMMENT "Building HEX and BIN files"
)

# Print executable size
add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_SIZE} $<TARGET_FILE:${CMAKE_PROJECT_NAME}>
    COMMENT "Program size:"
)

# Add output files to clean target
set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES 
    ADDITIONAL_CLEAN_FILES "${CMAKE_PROJECT_NAME}.map;${CMAKE_PROJECT_NAME}.hex;${CMAKE_PROJECT_NAME}.bin"
)
