cmake_minimum_required(VERSION 3.25)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
project(
	TeensyLogger
	VERSION 1.0
	LANGUAGES C CXX
)

set(MCU IMXRT1062)
set(MCU_LD imxrt1062.ld)
set(MCU_DEF ARDUINO_TEENSY40)
set(TEENSYDUINO_VERSION 159)
set(ARDUINO_VERSION 10813)

# MCU configuration
add_definitions(
	-D__${MCU}__
	-DARDUINO=${ARDUINO_VERSION}
	-DTEENSYDUINO=${TEENSYDUINO_VERSION}
	-D${MCU_DEF}
	-DF_CPU=600000000
	-DUSB_SERIAL
	-DLAYOUT_US_ENGLISH
	-DUSING_MAKEFILE
)
# Compile options
add_compile_options(
	$<$<COMPILE_LANGUAGE:C>:-std=gnu11>
	$<$<COMPILE_LANGUAGE:CXX>:-std=gnu++20>
	$<$<COMPILE_LANGUAGE:CXX>:-fno-exceptions>
	$<$<COMPILE_LANGUAGE:CXX>:-felide-constructors>
	$<$<COMPILE_LANGUAGE:CXX>:-Wno-error=narrowing>
	$<$<COMPILE_LANGUAGE:CXX>:-fno-rtti>
	$<$<COMPILE_LANGUAGE:CXX>:-Wno-volatile>
	$<$<COMPILE_LANGUAGE:CXX>:-fpermissive>
	-g 
	-Os 
	-Wno-psabi 
	-mthumb 
	-ffunction-sections 
	-fdata-sections 
	-MMD
	-mcpu=cortex-m7
	-mfloat-abi=hard 
	-mfpu=fpv5-d16
)
# Linking
set(LINKER_SCRIPT "${CMAKE_SOURCE_DIR}/teensy4/imxrt1062.ld")
add_link_options(
	-Os
	-Wl,--gc-sections,--relax
	--specs=nano.specs
	-mcpu=cortex-m7
	-mfloat-abi=hard
	-mfpu=fpv5-d16
	-mthumb
	-T${LINKER_SCRIPT}
)
link_libraries(
	-lm
	-lstdc++
)

# Teensy files
add_subdirectory(teensy4)

# Logger library
add_library(ArduinoLog
	${CMAKE_SOURCE_DIR}/src/ArduinoLog.cpp
)

target_include_directories(ArduinoLog PUBLIC
	$<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
	$<INSTALL_INTERFACE:include>
)

target_link_libraries(ArduinoLog PUBLIC
	TeensyLib
)

if (PROJECT_NAME STREQUAL CMAKE_PROJECT_NAME)
	add_executable(LoggingTest
		${CMAKE_CURRENT_SOURCE_DIR}/examples/Log-basic/Log-basic.cpp
	)

	target_link_libraries(LoggingTest PRIVATE
		ArduinoLog
	)

	# Make test program
	include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/mcu_upload.cmake)
	flashMcu(LoggingTest)
endif()
