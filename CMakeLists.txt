cmake_minimum_required(VERSION 3.25)
include(FetchContent)
# Includes toolchain, linker scripts, and CLI tools
FetchContent_Declare(
	teensytools
	GIT_REPOSITORY https://github.com/jmjohnson1/TeensyTools.git
	GIT_TAG v1.0.1
)
FetchContent_MakeAvailable(teensytools)
set(CMAKE_TOOLCHAIN_FILE "${teensytools_SOURCE_DIR}/cmake/mcu_toolchain.cmake")
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
project(
	TeensyLogger
	VERSION 1.0
	LANGUAGES C CXX
)
# Sets compiler and linker options as well as some definitions
include("${teensytools_SOURCE_DIR}/cmake/mcu_config.cmake")
configMCU(${teensytools_SOURCE_DIR})

FetchContent_Declare(
	teensy4
	GIT_REPOSITORY https://github.com/jmjohnson1/Teensy4.git
	GIT_TAG v1.0.0
)
FetchContent_MakeAvailable(teensy4)

# Logger library
add_library(ArduinoLog
	${CMAKE_SOURCE_DIR}/src/ArduinoLog.cpp
)

target_include_directories(ArduinoLog PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
	$<INSTALL_INTERFACE:include>
)

target_link_libraries(ArduinoLog PUBLIC
	Teensy4
)

if (PROJECT_NAME STREQUAL CMAKE_PROJECT_NAME)
	add_executable(LoggingTest
		${CMAKE_CURRENT_SOURCE_DIR}/examples/Log-basic/Log-basic.cpp
	)

	target_link_libraries(LoggingTest PRIVATE
		ArduinoLog
	)

	# Make test program
	include(${teensytools_SOURCE_DIR}/cmake/mcu_upload.cmake)
	flashMcu(LoggingTest ${CMAKE_SOURCE_DIR})
endif()
