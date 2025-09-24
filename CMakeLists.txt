cmake_minimum_required(VERSION 3.24)

option(OUTDATED "Applies 1.5.97 offsets/IDs." ON)

# -------- Project ----------
project(
	CurrencySwapper
	VERSION 2.2.0
	LANGUAGES CXX
)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

get_filename_component(
    Skyrim64Path
    "[HKEY_LOCAL_MACHINE\\SOFTWARE\\WOW6432Node\\Bethesda Softworks\\Skyrim Special Edition;installed path]"
    ABSOLUTE
)

set(Skyrim64Path "${Skyrim64Path}" CACHE PATH "Skyrim SE install directory")

if(NOT IS_DIRECTORY "${Skyrim64Path}")
    message(WARNING "Skyrim SE install path not found; Papyrus scripts will not be compiled or archived.")
endif()

list(APPEND CMAKE_MODULE_PATH
	"${PROJECT_SOURCE_DIR}/tools/SKSE-CMakeModules/Modules"
)
list(APPEND CMAKE_MODULE_PATH
	"${PROJECT_SOURCE_DIR}/tools/Bethesda-CMakeModules/Modules"
)

include(SKSEPlugin)
set(SKSE_USE_XBYAK ON)
add_compile_definitions(SKSE_SUPPORT_XBYAK)
add_compile_definitions(SKYRIM_AE)
add_compile_definitions(SKYRIM_SUPPORT_AE)

SKSEPlugin_Add(
	${PROJECT_NAME}
	SOURCE_DIR src
	SOURCES
		README.md
		vcpkg.json
	PRECOMPILE_HEADERS
		src/common/PCH.h
)

find_path(CLIB_UTIL_INCLUDE_DIRS "ClibUtil/utils.hpp")
find_package(jsoncpp CONFIG REQUIRED)

target_link_libraries(
		"${PROJECT_NAME}"
		PRIVATE
			JsonCpp::JsonCpp
)

target_include_directories(
	${PROJECT_NAME}
	PRIVATE
		${CLIB_UTIL_INCLUDE_DIRS}
)

set_target_properties(CommonLibSSE PROPERTIES
	FOLDER External
)

set(DATA_DIR "${PROJECT_SOURCE_DIR}/Data")
set(SKSE_DIR "${DATA_DIR}/SKSE")
set(SKSE_PLUGINS_DIR "${SKSE_DIR}/Plugins")

set(INI_PATH "${SKSE_PLUGINS_DIR}/${PROJECT_NAME}.ini")
set(PAPYRUS_SOURCE_DIR "${DATA_DIR}/Source/Scripts")
set(CUSTOM_CONSOLE_DIR "${SKSE_DIR}/CustomConsole")

if (IS_DIRECTORY PAPYRUS_SOURCE_DIR)
	install(
		DIRECTORY "${DATA_DIR}/Source"
		DESTINATION "."
		COMPONENT components
	)
endif()

if (IS_DIRECTORY CUSTOM_CONSOLE_DIR)
	install(
		DIRECTORY "${SKSE_DIR}/CustomConsole"
		DESTINATION "."
		COMPONENT components
	)
endif()

include(components.cmake)