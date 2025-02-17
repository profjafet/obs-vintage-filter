cmake_minimum_required(VERSION 3.16)

# Change obs-plugintemplate to your plugin's name in a machine-readable format
# (e.g.: obs-myawesomeplugin) and set 
project(obs-vintage-filter VERSION 2.0.0)
add_library(${CMAKE_PROJECT_NAME} MODULE)

# Replace `Your Name Here` with the name (yours or your organization's) you want
# to see as the author of the plugin (in the plugin's metadata itself and in the installers)
set(PLUGIN_AUTHOR "cg2121")

# Replace `com.example.obs-plugin-template` with a unique Bundle ID for macOS releases
# (used both in the installer and when submitting the installer for notarization)
set(MACOS_BUNDLEID "com.obs-plugin.obs-vintage-filter")

# Replace `me@contoso.com` with the maintainer email address you want to put in Linux packages
set(LINUX_MAINTAINER_EMAIL "claytong1214@gmail.com")

# Add your custom source files here - header files are optional and only required for visibility
# e.g. in Xcode or Visual Studio
target_sources(${CMAKE_PROJECT_NAME}
	PRIVATE
		src/plugin-main.c)

# /!\ TAKE NOTE: No need to edit things past this point /!\

find_package(libobs REQUIRED)
find_package(obs-frontend-api REQUIRED)
include(external/ObsPluginHelpers.cmake)
find_package(Qt5Core REQUIRED)
find_package(Qt5Widgets REQUIRED)


configure_file(
	src/plugin-macros.h.in
	${CMAKE_SOURCE_DIR}/src/plugin-macros.generated.h)

target_sources(${CMAKE_PROJECT_NAME}
	PRIVATE
		src/plugin-macros.generated.h)

# --- Platform-independent build settings ---

target_include_directories(${CMAKE_PROJECT_NAME}
	PRIVATE ${CMAKE_SOURCE_DIR}/src)

target_link_libraries(${CMAKE_PROJECT_NAME}
	PRIVATE
		OBS::libobs
		OBS::obs-frontend-api
		Qt5::Core
		Qt5::Widgets)

set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES
	AUTOMOC ON
	AUTOUIC ON
	AUTORCC ON)

target_compile_features(${CMAKE_PROJECT_NAME}
	PRIVATE
		cxx_std_17)

# --- End of section ---

# --- Windows-specific build settings and tasks ---
if(OS_WINDOWS)
	configure_file(
		installer/installer-Windows.iss.in
		${CMAKE_SOURCE_DIR}/installer/installer-Windows.generated.iss)

	configure_file(
		CI/include/build_environment.ps1.in
		${CMAKE_SOURCE_DIR}/CI/include/build_environment.ps1)

	if(MSVC)
		target_compile_options(${CMAKE_PROJECT_NAME}
			PRIVATE
				/MP
				/d2FH4-)
	endif()
# --- End of section ---

# -- macOS specific build settings and tasks --
elseif(OS_MACOS)
	configure_file(
		bundle/installer-macOS.pkgproj.in
		${CMAKE_SOURCE_DIR}/bundle/installer-macOS.generated.pkgproj)

	configure_file(
		CI/include/build_environment.sh.in
		${CMAKE_SOURCE_DIR}/CI/include/build_environment.sh
	)

	set(MACOSX_PLUGIN_GUI_IDENTIFIER "${MACOS_BUNDLEID}")
	set(MACOSX_PLUGIN_BUNDLE_VERSION "${CMAKE_PROJECT_VERSION}")
	set(MACOSX_PLUGIN_SHORT_VERSION_STRING "1")

	target_compile_options(${CMAKE_PROJECT_NAME}
		PRIVATE
			-Wall
			-Wextra
			-Werror-implicit-function-declaration
			-stdlib=libc++
			-fvisibility=default)

	set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES PREFIX "")
# --- End of section ---

# --- Linux-specific build settings and tasks ---
else()
	configure_file(
		CI/include/build_environment.sh.in
		${CMAKE_SOURCE_DIR}/CI/include/build_environment.sh
	)

	target_compile_options(${CMAKE_PROJECT_NAME}
		PRIVATE
			-Wall
			-Wextra)

	set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES PREFIX "")
endif()
# --- End of section ---

setup_plugin_target(${CMAKE_PROJECT_NAME})
