cmake_minimum_required(VERSION 3.10)
project(McapExamples LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
# set(CONAN_SYSTEM_INCLUDES ON) # avoid warnings in protobuf headers

if (BUILD_TESTING)
    find_package(Catch2 3 CONFIG REQUIRED)
endif()

find_package(Protobuf 3 REQUIRED)
find_package(nlohmann_json CONFIG REQUIRED)

if (MSVC)
  add_compile_options(/W4 /WX
    /wd4251 # suppress warning about having template instances (such as std::string) as public class members
  )
  add_compile_definitions(_CRT_SECURE_NO_WARNINGS)
else()
  add_compile_options(-Wall -Wextra -pedantic -Wshadow -Wpointer-arith -Werror)
endif()


add_executable(bag2mcap bag2mcap.cpp)
target_link_libraries(bag2mcap mcap)

add_executable(mcapdump mcapdump.cpp)
target_link_libraries(mcapdump mcap)

add_executable(rotatemcap rotatemcap.cpp)
target_link_libraries(rotatemcap mcap)

add_subdirectory(protobuf)
add_subdirectory(jsonschema)

install(
	TARGETS
        bag2mcap
        mcapdump
        rotatemcap
        example_json_writer
        example_json_reader
        example_protobuf_writer
        example_protobuf_dynamic_reader
        example_protobuf_static_reader
	RUNTIME DESTINATION lib/mcap-cpp/examples
	COMPONENT examples
)
