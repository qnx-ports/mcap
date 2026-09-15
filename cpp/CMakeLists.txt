cmake_minimum_required(VERSION 3.22.1)

project(Mcap VERSION 2.1.3 LANGUAGES CXX)

include(CTest)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

add_library(mcap INTERFACE)

target_include_directories(mcap INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/mcap/include>
    $<INSTALL_INTERFACE:include>
)

option(BUILD_CONFORMANCE_TESTS OFF)

set(DEPENDENCIES "")

find_library(LZ4_LIBRARY NAMES lz4)
if(LZ4_LIBRARY)
    message(STATUS "Found lz4 from ${LZ4_LIBRARY}")
    list(APPEND DEPENDENCIES ${LZ4_LIBRARY})
else()
  message(WARNING "Dependency LZ4 not found, disabling")
  target_compile_definitions(mcap INTERFACE "MCAP_COMPRESSION_NO_LZ4")
endif()

find_library(ZSTD_LIBRARY NAMES zstd)
if(ZSTD_LIBRARY)
    message(STATUS "Found zstd from ${ZSTD_LIBRARY}")
    list(APPEND DEPENDENCIES ${ZSTD_LIBRARY})
else()
  message(WARNING "Dependency zstd not found, disabling")
  target_compile_definitions(mcap INTERFACE "MCAP_COMPRESSION_NO_ZSTD")
endif()

target_link_libraries(mcap INTERFACE ${DEPENDENCIES})

if(BUILD_TESTING)
    add_subdirectory(test)
endif()

if(BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()

# Package Configuration Files
include(CMakePackageConfigHelpers)
configure_package_config_file(mcapConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/mcapConfig.cmake
    INSTALL_DESTINATION lib/cmake/mcap
)
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/mcapConfigVersion.cmake
    COMPATIBILITY AnyNewerVersion
)

# install all targets
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/mcapConfigVersion.cmake
              ${CMAKE_CURRENT_BINARY_DIR}/mcapConfig.cmake
        DESTINATION lib/cmake/mcap
)
install(TARGETS mcap
    EXPORT mcap-targets
)
install(EXPORT mcap-targets
    FILE mcap-targets.cmake
    NAMESPACE mcap::
    DESTINATION lib/cmake/mcap
)

install(DIRECTORY mcap/include/ DESTINATION include)
