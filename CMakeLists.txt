cmake_minimum_required (VERSION 3.16)

project (semimap)

set(SEMIMAP_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)

# By default, we build in Release mode.
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "" FORCE)
endif()

# Versioning
# ==========

file(
    STRINGS "${SEMIMAP_INCLUDE_DIR}/semimap/semimap.h" semimap_version_defines
    REGEX "#define SEMIMAP_VERSION_(MAJOR|MINOR|PATCH)"
)

foreach(ver ${semimap_version_defines})
    if(ver MATCHES "#define SEMIMAP_VERSION_(MAJOR|MINOR|PATCH) +([^ ]+)$")
        set(SEMIMAP_VERSION_${CMAKE_MATCH_1} "${CMAKE_MATCH_2}" CACHE INTERNAL "")
    endif()
endforeach()

set(
    ${PROJECT_NAME}_VERSION
    ${SEMIMAP_VERSION_MAJOR}.${SEMIMAP_VERSION_MINOR}.${SEMIMAP_VERSION_PATCH}
)

message(STATUS "semimap v${${PROJECT_NAME}_VERSION}")


# Build
# =====

set(
    SEMIMAP_HEADERS
    ${SEMIMAP_INCLUDE_DIR}/semimap/semimap.h
)

add_library(semimap INTERFACE)

target_include_directories(
    semimap INTERFACE
    $<BUILD_INTERFACE:${SEMIMAP_INCLUDE_DIR}>
    $<INSTALL_INTERFACE:include>
)

set_target_properties(
    semimap PROPERTIES
    PUBLIC_HEADER "${SEMIMAP_HEADERS}"
)

target_compile_features(semimap INTERFACE cxx_std_17)

# TODO: add test suite


# Installation
# ============

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

install(
    TARGETS semimap
    EXPORT ${PROJECT_NAME}-targets
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
)

# Makes the project importable from the build directory
export(
    EXPORT ${PROJECT_NAME}-targets
    FILE "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-Targets.cmake"
)

set(
    SEMIMAP_CMAKECONFIG_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}"
    CACHE STRING "install path for semimapConfig.cmake"
)

configure_package_config_file(
    ${PROJECT_NAME}Config.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
    INSTALL_DESTINATION ${SEMIMAP_CMAKECONFIG_INSTALL_DIR}
)

# semimap is header-only and does not depend on the architecture.
# Remove CMAKE_SIZEOF_VOID_P from semimapConfigVersion.cmake so that an
# architecture-independent semimapConfig.cmake is generated.
set(_SEMIMAP_CMAKE_SIZEOF_VOID_P ${CMAKE_SIZEOF_VOID_P})
unset(CMAKE_SIZEOF_VOID_P)

write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
    VERSION ${${PROJECT_NAME}_VERSION}
    COMPATIBILITY AnyNewerVersion
    ARCH_INDEPENDENT
)

set(CMAKE_SIZEOF_VOID_P ${_SEMIMAP_CMAKE_SIZEOF_VOID_P})
install(
    FILES
        ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
    DESTINATION ${SEMIMAP_CMAKECONFIG_INSTALL_DIR}
)
install(
    EXPORT ${PROJECT_NAME}-targets
    FILE ${PROJECT_NAME}Targets.cmake
    DESTINATION ${SEMIMAP_CMAKECONFIG_INSTALL_DIR}
)
