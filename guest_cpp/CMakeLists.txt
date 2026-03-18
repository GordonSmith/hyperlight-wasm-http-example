cmake_minimum_required(VERSION 3.25)
project(guest_cpp LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXECUTABLE_SUFFIX ".wasm")

add_executable(guest_cpp
    root.cpp
)

target_include_directories(guest_cpp PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})

# Reactor model: exports functions instead of running main()
target_link_options(guest_cpp PRIVATE -mexec-model=reactor)

# Link the wit-bindgen generated component-type object
target_link_libraries(guest_cpp PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/root_component_type.o)
