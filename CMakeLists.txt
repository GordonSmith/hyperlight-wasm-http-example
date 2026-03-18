cmake_minimum_required(VERSION 3.25)
project(hyperlight-wasm-http-example LANGUAGES CXX)

include(ExternalProject)

# --- Locate wasi-sdk installed via vcpkg ---
if (NOT DEFINED WASI_SDK_PREFIX OR WASI_SDK_PREFIX STREQUAL "")
    set(_wasi_sdk_candidates)
    if (DEFINED VCPKG_TARGET_TRIPLET)
        list(APPEND _wasi_sdk_candidates "${CMAKE_CURRENT_BINARY_DIR}/vcpkg_installed/${VCPKG_TARGET_TRIPLET}/wasi-sdk")
    endif()
    file(GLOB _wasi_sdk_glob "${CMAKE_CURRENT_BINARY_DIR}/vcpkg_installed/*/wasi-sdk")
    list(APPEND _wasi_sdk_candidates ${_wasi_sdk_glob})
    list(FILTER _wasi_sdk_candidates EXCLUDE REGEX "^$")
    list(REMOVE_DUPLICATES _wasi_sdk_candidates)
    if (_wasi_sdk_candidates)
        list(GET _wasi_sdk_candidates 0 _wasi_sdk_default)
        set(WASI_SDK_PREFIX "${_wasi_sdk_default}" CACHE PATH "Path to the WASI SDK installed via vcpkg" FORCE)
    endif()
endif()

if (NOT WASI_SDK_PREFIX)
    message(FATAL_ERROR "WASI_SDK_PREFIX is not set. Install wasi-sdk via vcpkg or set WASI_SDK_PREFIX manually.")
endif()
message(STATUS "WASI_SDK_PREFIX: ${WASI_SDK_PREFIX}")

# --- Find Ninja for the wasm sub-build ---
if (NOT DEFINED NINJA_EXECUTABLE)
    find_program(NINJA_EXECUTABLE ninja)
endif()
if (NOT NINJA_EXECUTABLE)
    message(FATAL_ERROR "Ninja build tool not found. Install ninja-build or set NINJA_EXECUTABLE manually.")
endif()

# --- Build guest_cpp as an ExternalProject using wasi-sdk toolchain ---
ExternalProject_Add(guest_cpp_wasm
    SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/guest_cpp
    BUILD_ALWAYS TRUE
    PREFIX guest_cpp_wasm
    CMAKE_GENERATOR Ninja
    CMAKE_ARGS
        -DWASI_SDK_PREFIX=${WASI_SDK_PREFIX}
        -DCMAKE_TOOLCHAIN_FILE=${WASI_SDK_PREFIX}/share/cmake/wasi-sdk.cmake
        -DCMAKE_SYSROOT=${WASI_SDK_PREFIX}/share/wasi-sysroot
        -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}
        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        -DCMAKE_MAKE_PROGRAM=${NINJA_EXECUTABLE}
    INSTALL_COMMAND ""
)
