cmake_minimum_required (VERSION 3.5)

project(nccl VERSION 1.3.4)

# Define central CUDA version (allowing user overrides):
set(nccl_CUDA_MAJOR_VERSION 7 CACHE STRING "CUDA MAJOR version to build against")
set(nccl_CUDA_MINOR_VERSION 5 CACHE STRING "CUDA MINOR version to build against")
set(nccl_CUDA_VERSION "${nccl_CUDA_MAJOR_VERSION}.${nccl_CUDA_MINOR_VERSION}")
find_package(CUDA ${nccl_CUDA_VERSION} REQUIRED)
list(APPEND CUDA_NVCC_FLAGS  "-DCUDA_MAJOR=${nccl_CUDA_MAJOR_VERSION};-DCUDA_MINOR=${nccl_CUDA_MINOR_VERSION}")

# Call add_subdirectory(nccl) after nvcc flags have been set in the parent project to propagate flags to nccl
list(APPEND CUDA_NVCC_FLAGS "-DNCCL_MAJOR=${nccl_VERSION_MAJOR};-DNCCL_MINOR=${nccl_VERSION_MINOR};-DNCCL_PATCH=${nccl_VERSION_PATCH}")

file(GLOB SOURCES src/*.cu)
cuda_add_library(${PROJECT_NAME} STATIC ${SOURCES})
target_include_directories(${PROJECT_NAME} PUBLIC
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/src>"
)

option(NCCL_BUILD_TESTS "Build tests" OFF)
if(NCCL_BUILD_TESTS)
  add_subdirectory(test)
endif()

# Installation (https://github.com/forexample/package-example) {

# Layout. This works for all platforms:
#   * <prefix>/lib/cmake/<PROJECT-NAME>
#   * <prefix>/lib/
#   * <prefix>/include/
set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

# Configuration
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

# Include module with fuction 'write_basic_package_version_file'
include(CMakePackageConfigHelpers)

# Configure '<PROJECT-NAME>ConfigVersion.cmake'
# Use:
#   * PROJECT_VERSION
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Configure '<PROJECT-NAME>Config.cmake'
# Use variables:
#   * TARGETS_EXPORT_NAME
#   * PROJECT_NAME
configure_package_config_file(
    "cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

# Targets:
#   * <prefix>/lib/libnccl.a
#   * header location after install: <prefix>/include/nccl/nccl.h
#   * headers can be included by C++ code `#include <nccl/nccl.h>`
install(
    TARGETS ${PROJECT_NAME} 
    EXPORT "${TARGETS_EXPORT_NAME}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

# Headers:
install(
    DIRECTORY "src/"
    DESTINATION "${include_install_dir}/${PROJECT_NAME}"
    FILES_MATCHING PATTERN "*.h"
)

# Config
#   * <prefix>/lib/cmake/nccl/ncclConfig.cmake
#   * <prefix>/lib/cmake/nccl/ncclConfigVersion.cmake
install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

# Config
#   * <prefix>/lib/cmake/Foo/FooTargets.cmake
install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)
# }
