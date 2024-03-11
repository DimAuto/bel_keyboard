cmake_minimum_required(VERSION 3.12)

project(bel_keyboard)
set(PROJECT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/PROJECT)

set(MCU_FAMILY STM32L4xx)
set(MCU_MODEL STM32L4S5VITx)
set(CPU_PARAMETERS
    -mcpu=cortex-m4
    -DSTM32L4S5xx
    -mfpu=fpv4-sp-d16
    -mfloat-abi=hard
    -mthumb)
     
set(STARTUP_SCRIPT ${CMAKE_CURRENT_SOURCE_DIR}/Core/Startup/startup_stm32l4s5vitx.s)
set(MCU_LINKER_SCRIPT ${CMAKE_CURRENT_SOURCE_DIR}/STM32L4S5VITX_FLASH.ld)
# ------------------------------------------------------------------------------
set(EXECUTABLE ${CMAKE_PROJECT_NAME})
enable_language(C ASM)
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_EXTENSIONS ON)

#Headers
set(CORE_INCLUDE_DIRS
    ${CMAKE_CURRENT_SOURCE_DIR}/Core/Inc)
set(FREERTOS_INCLUDE_DIRS 
    ${CMAKE_CURRENT_DIR}/Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS_V2
    ${CMAKE_CURRENT_DIR}/Middlewares/Third_Party/FreeRTOS/Source/include)
set(DRIVERS_INCLUDE_DIRS
    ${CMAKE_CURRENT_DIR}/Drivers/CMSIS/Include
    ${CMAKE_CURRENT_DIR}/Drivers/${MCU_FAMILY}_HAL_Driver/Inc
    ${CMAKE_CURRENT_DIR}/Drivers/${MCU_FAMILY}_HAL_Driver/Inc/Legacy
    ${CMAKE_CURRENT_DIR}/Drivers/CMSIS/Device/ST/${MCU_FAMILY}/Include)
set(PROJECT_INCLUDE_DIRS
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_DIR})

#Sources
file(GLOB_RECURSE SOURCE_FILES
    ${CMAKE_CURRENT_SOURCE_DIR}/Core/*.c
    ${CMAKE_CURRENT_SOURCE_DIR}/Drivers/*.c
    ${CMAKE_CURRENT_SOURCE_DIR}/Middlewares/Third_Party/FreeRTOS/*.c)
file(GLOB_RECURSE PROJECT_SOURCES FOLLOW_SYMLINKS
    # ${PROJECT_DIR}/*.cpp
    ${PROJECT_DIR}/*.c)


add_executable(${EXECUTABLE}
            ${SOURCE_FILES}
            ${PROJECT_SOURCES}
            ${STARTUP_SCRIPT})


 # MACROS defines !!           
target_compile_definitions(${EXECUTABLE} PRIVATE
                            ${MCU_MODEL} USE_HAL_DRIVER)

target_include_directories(${EXECUTABLE} PRIVATE
                            ${CORE_INCLUDE_DIRS}
                            ${FREERTOS_INCLUDE_DIRS}
                            ${DRIVERS_INCLUDE_DIRS}
                            ${PROJECT_INCLUDE_DIRS})

target_compile_options(${ECECUTABLE}
                        ${CPU_PARAMETERS}
                        --std=gnu11
                        -g3
                        -O0
                        -Wall
                        -Wextra
                        -fstack-usage
                        -fcyclomatic-complexity
                        -DUSE_HAL_DRIVER)

target_link_options(${ECECUTABLE}
                    -T${MCU_LINKER_SCRIPT}
                    ${CPU_PARAMETERS}
                    --specs=nosys.specs
                    -Wl,-Map=${CMAKE_PROJECT_NAME}.map
                    --gc-sections -static
                    --specs=nano.spec
                    -mthumb -Wl,--start-group -lc -lm -Wl,--end-group)

# Calculate the size of executable
add_custom_command(TARGET ${EXECUTABLE} POST_BUILD
                    COMMAND ${CMAKE_SIZE} $<TARGET_FILE:${EXECUTABLE}>)

add_custom_command(TARGET ${EXECUTABLE} POST_BUILD
                    COMMAND ${CMAKE_OBJCOPY} -O ihex $<TARGET_FILE:${EXECUTABLE}>
                    ${EXECUTABLE}.hex
                    COMMAND ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:${EXECUTABLE}>
                    ${EXECUTABLE}.bin)