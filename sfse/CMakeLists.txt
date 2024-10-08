cmake_minimum_required(VERSION 3.18)

# ---- Project ----

include(${CMAKE_CURRENT_SOURCE_DIR}/../cmake/versioning.cmake)

project(
	sfse
	VERSION ${SFSE_VERSION_MAJOR}.${SFSE_VERSION_MINOR}.${SFSE_VERSION_PATCH}
	LANGUAGES CXX
)

# ---- Include guards ----

if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
	message(
		FATAL_ERROR
			"In-source builds not allowed. Please make a new directory (called a build directory) and run CMake from there."
)
endif()

# ---- Build options ----

set(CMAKE_INTERPROCEDURAL_OPTIMIZATION ON)
set(CMAKE_INTERPROCEDURAL_OPTIMIZATION_DEBUG OFF)

# ---- Dependencies ----

if (NOT TARGET sfse_common)
	add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../sfse_common sfse_common)	# bundled
endif()

if (NOT TARGET xbyak)
	add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../xbyak xbyak)	# bundled
endif()

# ---- Add source files ----

file(GLOB headers CONFIGURE_DEPENDS *.h)
file(GLOB sources CONFIGURE_DEPENDS *.cpp)

source_group(
	${PROJECT_NAME}
	FILES
		sfse.cpp
		sfse.def
		sfse_version.rc
)

source_group(
	${PROJECT_NAME}/api
	FILES
		GameChargen.cpp
		GameChargen.h
		GameData.cpp
		GameData.h
		GameConsole.cpp
		GameConsole.h
		GameEvents.h
		GameExtraData.cpp
		GameExtraData.h
		GameFormComponents.h
		GameForms.cpp
		GameForms.h
		GameLock.cpp
		GameLock.h
		GameMemory.cpp
		GameMemory.h
		GameMenu.h
		GameObjects.h
		GameReferences.h
		GameReflection.h
		GameRTTI.cpp
		GameRTTI.h
		GameRTTI.inl
		GameScript.cpp
		GameScript.h
		GameSettings.cpp
		GameSettings.h
		GameStreams.h
		GameSingleton.h
		GameTypes.cpp
		GameTypes.h
		GameUI.cpp
		GameUI.h
		NiExtraData.cpp
		NiExtraData.h
		NiObject.cpp
		NiObject.h
		NiRTTI.cpp
		NiRTTI.h
)

source_group(
	${PROJECT_NAME}/hooks
	FILES
		Hooks_Data.cpp
		Hooks_Data.h
		Hooks_Version.cpp
		Hooks_Version.h
		Hooks_Script.cpp
		Hooks_Script.h
		Hooks_Scaleform.cpp
		Hooks_Scaleform.h
		Hooks_Command.cpp
		Hooks_Command.h
		Hooks_Serialization.cpp
		Hooks_Serialization.h
)

source_group(
	${PROJECT_NAME}/internal
	FILES
		PluginAPI.h
		PluginManager.cpp
		PluginManager.h
		Serialization.cpp
		Serialization.h
)

source_group(
	${PROJECT_NAME}/netimmerse
	FILES
		NiTypes.h
)

source_group(
	${PROJECT_NAME}/papyrus/functions
	FILES
)

source_group(
	${PROJECT_NAME}/papyrus/vm
	FILES
		PapyrusNativeFunctions.h
)

source_group(
	${PROJECT_NAME}/scaleform
	FILES
		ScaleformFunctions.h
		ScaleformLog.h
		ScaleformMemory.cpp
		ScaleformMemory.h
		ScaleformMovie.h
		ScaleformState.h
		ScaleformTypes.h
		ScaleformValue.h
		ScaleformLoader.h
		ScaleformManager.h
		ScaleformTranslator.h
)

# ---- Create library ----

add_library(
	${PROJECT_NAME}
	SHARED
	${headers}
	${sources}
	sfse.def
	sfse_version.rc
)

add_library(sfse64::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

include(${CMAKE_CURRENT_SOURCE_DIR}/../cmake/configuration.cmake)

set_target_properties(
	${PROJECT_NAME}
	PROPERTIES
		OUTPUT_NAME "${PROJECT_NAME}_${RUNTIME_VERSION_MAJOR}_${RUNTIME_VERSION_MINOR}_${RUNTIME_VERSION_PATCH}"
)

target_compile_definitions(
	${PROJECT_NAME}
	PRIVATE
		RUNTIME
		RUNTIME_VERSION=${RUNTIME_VERSION_PACKED}
)

target_compile_features(
	${PROJECT_NAME}
	PUBLIC
		cxx_std_17
)

target_include_directories(
	${PROJECT_NAME}
	PUBLIC
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/..>
		$<INSTALL_INTERFACE:include>
)

target_link_libraries(
	${PROJECT_NAME}
	PUBLIC
		sfse::sfse_common
		xbyak::xbyak
)

target_compile_options(
	${PROJECT_NAME}
	PUBLIC
		/wd4200
)

# ---- Configure all targets ----

set_target_properties(
	${PROJECT_NAME}
	sfse_common
	PROPERTIES
		MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL"
)

# ---- Create an installable target ----

include(GNUInstallDirs)

install(
	TARGETS
		${PROJECT_NAME}
	EXPORT
		${PROJECT_NAME}-targets
)

install(
	FILES
		${headers}
	DESTINATION
		${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
)

configure_file(
	cmake/config.cmake.in
	${PROJECT_NAME}-config.cmake
	@ONLY
)

install(
	FILES
		${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config.cmake
	DESTINATION
		${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)

install(
	EXPORT
		${PROJECT_NAME}-targets
	NAMESPACE
		sfse::
	DESTINATION
		${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)

include(${CMAKE_CURRENT_SOURCE_DIR}/../cmake/installation.cmake)
