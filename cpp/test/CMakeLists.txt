cmake_minimum_required(VERSION 3.10)
project(McapTests LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

find_package(Catch2 3 CONFIG REQUIRED)
find_package(nlohmann_json CONFIG REQUIRED)

add_compile_options(-Wall -Wextra -pedantic -Wshadow -Wpointer-arith -Werror)

if(LZ4_LIBRARY AND ZSTD_LIBRARY)

    add_executable(unit-tests unit_tests.cpp)
    target_link_libraries(unit-tests PRIVATE mcap Catch2::Catch2WithMain)
    target_compile_definitions(unit-tests PRIVATE CATCH2_V3)
    message(STATUS "configured unit-tests with CATCH2_V3")

    add_test(NAME unit-tests COMMAND $<TARGET_FILE:unit-tests>)
else()

    add_executable(unit-tests-nocompress unit_tests.cpp)
    target_link_libraries(unit-tests-nocompress PRIVATE mcap Catch2::Catch2WithMain)
    target_compile_definitions(unit-tests-nocompress PRIVATE MCAP_COMPRESSION_NO_LZ4 MCAP_COMPRESSION_NO_ZSTD CATCH2_V3)
    message(STATUS "configured unit-tests-nocompress with CATCH2_V3")

    add_test(NAME unit-tests-nocompress COMMAND $<TARGET_FILE:unit-tests-nocompress>)
endif()

if(BUILD_MCAP_CONFORMANCE_TESTS)

    add_executable(streamed-reader-conformance streamed_reader_conformance.cpp)
    target_link_libraries(streamed-reader-conformance PRIVATE mcap nlohmann_json::nlohmann_json)

    add_executable(indexed-reader-conformance indexed_reader_conformance.cpp)
    target_link_libraries(indexed-reader-conformance PRIVATE mcap nlohmann_json::nlohmann_json)

    add_executable(streamed-writer-conformance streamed_writer_conformance.cpp)
    target_link_libraries(streamed-writer-conformance PRIVATE mcap nlohmann_json::nlohmann_json)
endif()

