cmake_minimum_required(VERSION 3.16)
set(CMAKE_TOOLCHAIN_FILE ${CMAKE_CURRENT_SOURCE_DIR}/../../cmake/stm32_gcc.cmake)

project(stm32-fetch-cmsis-hal C ASM)

stm32_fetch_cmsis(F3)
stm32_fetch_hal(F3)

find_package(CMSIS COMPONENTS STM32F303K8 REQUIRED)
find_package(HAL COMPONENTS STM32F3 REQUIRED)
set(CMAKE_INCLUDE_CURRENT_DIR TRUE)


file(GLOB PROJECT_SOURCES CONFIGURE_DEPENDS  "*.c")


add_executable(stm32-fetch-f3 ${PROJECT_SOURCES} stm32f3xx_hal_conf.h)
target_link_libraries(stm32-fetch-f3 CMSIS::STM32::F303K8 STM32::NoSys 
        HAL::STM32::F3::UART 
        HAL::STM32::F3::GPIO 
        HAL::STM32::F3::CORTEX 
        HAL::STM32::F3::RCC 
    )
stm32_print_size_of_target(stm32-fetch-f3)
stm32_generate_binary_file(stm32-fetch-f3)
