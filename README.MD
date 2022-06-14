# My journey

## Cmake configuration
I've successfully modified and compiled one of the examples of [this](https://github.com/ObKo/stm32-cmake) repo. Had to add the ARM toolchain, make, ninja to the PATH, but after that, it went smoothly.
I used the `fetch-cmsis-hal` example on an STM32F303K8. I've generated an almost empty project in CubeIDE, where I blinked an LED. The final cmake file is in the project's folder.

## Debugging
I have followed [this](https://github.com/MaJerle/stm32-cube-cmake-vscode) repo to configure vscode. Python 2.7 is needed on the path, otherwise the GDB server won't start up for some reason. Tried both stlink, and openocd. Based on [this SO question](https://stackoverflow.com/questions/72525534/gdb-disconnects-when-trying-to-debug-via-openocd), the culprit is the ARM toolchain.


## Extensions
+ Cmake Tools by Microsoft
+ Cortex-Debug by marus25

## Step by step
1. Install cmake, arm toolchain, CubeIDE and ninja/make
1. Make sure you can compile
1. install openocd
1. install python 2.7
1. Add launch.json, tasks.json was not needed
1. pray it works


## Files

### launch.json for stlink
```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug Microcontroller - STLink-V3",
            "cwd": "${workspaceFolder}",        //Path from where commands are executed
            "type": "cortex-debug",             //Debug 
            "executable": "${command:cmake.launchTargetPath}", //or fixed file path: build/project-name.elf
            "request": "launch",                //Use "attach" to connect to target w/o elf download
            "servertype": "stlink",             //Use stlink setup of cortex-M debug
            "device": "STM32F303K8",            //MCU used
            "interface": "swd",                 //Interface setup
            "serialNumber": "",                 //Set ST-Link ID if you use multiple at the same time
            "runToEntryPoint": "main",          //Run to main and stop there
            "v1": false,
            "showDevDebugOutput": "both",

            /* Will get automatically detected if STM32CubeIDE is installed to default directory
               or it can be manually provided if necessary.. */
            //"serverpath": "c:\\ST\\STM32CubeIDE_1.7.0\\STM32CubeIDE\\plugins\\com.st.stm32cube.ide.mcu.externaltools.stlink-gdb-server.win32_2.0.100.202109301221\\tools\\bin\\ST-LINK_gdbserver.exe",
            //"armToolchainPath": "c:\\ST\\STM32CubeIDE_1.7.0\\STM32CubeIDE\\plugins\\com.st.stm32cube.ide.mcu.externaltools.gnu-tools-for-stm32.9-2020-q2-update.win32_2.0.0.202105311346\\tools\\bin",
            //"stm32cubeprogrammer": "c:\\Program Files\\STMicroelectronics\\STM32Cube\\STM32CubeProgrammer\\bin",

            /* If you use external loader, add additional arguments */
            //"serverArgs": ["--extload", "path/to/ext/loader.stldr"],
        }
    ]
}
```

### launch.json for openocd
```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug Microcontroller - STLink-V3",
            "cwd": "${workspaceFolder}",        //Path from where commands are executed
            "type": "cortex-debug",             //Debug 
            "executable": "${command:cmake.launchTargetPath}", //or fixed file path: build/project-name.elf
            "request": "launch",                //Use "attach" to connect to target w/o elf download
            "servertype": "openocd",             //Use stlink setup of cortex-M debug
            "device": "STM32F303K8",            //MCU used
            "interface": "swd",                 //Interface setup
            "configFiles": [
                "interface/stlink-v2-1.cfg",
                "target/stm32f3x.cfg",
            ],
            "serialNumber": "",                 //Set ST-Link ID if you use multiple at the same time
            "runToEntryPoint": "main",          //Run to main and stop there
            "v1": false,
            "showDevDebugOutput": "both",

            /* Will get automatically detected if STM32CubeIDE is installed to default directory
               or it can be manually provided if necessary.. */
            //"serverpath": "c:\\ST\\STM32CubeIDE_1.7.0\\STM32CubeIDE\\plugins\\com.st.stm32cube.ide.mcu.externaltools.stlink-gdb-server.win32_2.0.100.202109301221\\tools\\bin\\ST-LINK_gdbserver.exe",
            //"armToolchainPath": "c:\\ST\\STM32CubeIDE_1.7.0\\STM32CubeIDE\\plugins\\com.st.stm32cube.ide.mcu.externaltools.gnu-tools-for-stm32.9-2020-q2-update.win32_2.0.0.202105311346\\tools\\bin",
            //"stm32cubeprogrammer": "c:\\Program Files\\STMicroelectronics\\STM32Cube\\STM32CubeProgrammer\\bin",

            /* If you use external loader, add additional arguments */
            //"serverArgs": ["--extload", "path/to/ext/loader.stldr"],
        }
    ]
}
```

### CMakeLists.txt
```
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
```