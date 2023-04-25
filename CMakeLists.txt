cmake_minimum_required (VERSION 3.16)

# TODO: Should we instead parse the version from the header file?
project (semimap VERSION 1.0.0)

set(SEMIMAP_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)

set(CMAKE_CXX_FLAGS_RELEASE -Ofast -march=native)

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
    VERSION ${PROJECT_VERSION}
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
