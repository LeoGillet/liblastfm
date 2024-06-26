cmake_minimum_required(VERSION 2.8.6)
project(liblastfm)

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules)
set(CMAKE_CXX_STANDARD 11)

# general settings
set(LASTFM_SOVERSION 1)
set(LASTFM_VERSION "0x00010100")

set(LASTFM_MAJOR_VERSION "1")
set(LASTFM_MINOR_VERSION "1")
set(LASTFM_PATCH_VERSION "0")
set(LASTFM_VERSION_STRING "${LASTFM_MAJOR_VERSION}.${LASTFM_MINOR_VERSION}.${LASTFM_PATCH_VERSION}")

# options
option(BUILD_FINGERPRINT "Build the lastfm-fingerprint library" ON)
option(BUILD_DEMOS "Build the lastfm example programs" OFF)
option(BUILD_TESTS "Build liblastfm tests" ON)


# installation dirs
include(GNUInstallDirs)
include(FeatureSummary)

#cmake module path
set(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH}" "${PROJECT_SOURCE_DIR}/cmake/Modules")

# setup qt stuff
set(CMAKE_AUTOMOC ON)

option(BUILD_WITH_QT4 "Build liblastfm with Qt4" OFF)

if(NOT BUILD_WITH_QT4)
    find_package(Qt5 REQUIRED COMPONENTS Core Network Xml Test CONFIG)

    if(BUILD_FINGERPRINT)
        find_package(Qt5Sql REQUIRED CONFIG)
    endif()

    set(LASTFM_LIB_VERSION_SUFFIX 5)
    if(UNIX AND NOT APPLE)
        find_package(Qt5DBus REQUIRED CONFIG)
    endif()

#     macro(qt_wrap_ui)
#         qt5_wrap_ui(${ARGN})
#     endmacro()
#
#     macro(qt_add_resources)
#         qt5_add_resources(${ARGN})
#     endmacro()
#
#     find_package(Qt5LinguistTools REQUIRED)
#     macro(qt_add_translation)
#         qt5_add_translation(${ARGN})
#     endmacro()
else()
    find_package(Qt4 COMPONENTS QtCore QtNetwork QtXml REQUIRED)
endif()


if(CMAKE_COMPILER_IS_GNUCXX)
    add_definitions("-fno-operator-names -fvisibility-inlines-hidden -fvisibility=hidden")
endif()
if(UNIX AND NOT APPLE)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wl,--no-undefined")
endif()

if(MSVC)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /Zc:wchar_t-")
endif(MSVC)

set(LASTFM_LIB_TARGET_NAME lastfm${LASTFM_LIB_VERSION_SUFFIX} CACHE
    INTERNAL "Target name of liblastfm" FORCE)

set(FINGERPRINT_LIB_TARGET_NAME lastfm_fingerprint${LASTFM_LIB_VERSION_SUFFIX} CACHE
    INTERNAL "Target name of liblastfm_fingerprint" FORCE)

# main library
add_subdirectory(src)

# lastfm_fingerprint library
if(BUILD_FINGERPRINT)
    add_subdirectory(src/fingerprint)
endif()

# demos
if(BUILD_DEMOS)
    add_subdirectory(demos)
endif()

# tests
if(BUILD_TESTS)
    enable_testing()
    add_subdirectory(tests)
endif()

feature_summary(WHAT ALL FATAL_ON_MISSING_REQUIRED_PACKAGES)
