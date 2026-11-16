cmake_minimum_required(VERSION 3.20)
project(storage_engine LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Compiler flags for high-performance systems programming and safety
add_compile_options(
    -Wall
    -Wextra
    -Wpedantic
    -Werror
    -Wno-unused-parameter
    -Wno-unused-variable
)

# Optional AddressSanitizer and ThreadSanitizer flags
option(ENABLE_ASAN "Enable AddressSanitizer" OFF)
option(ENABLE_TSAN "Enable ThreadSanitizer" OFF)

if(ENABLE_ASAN)
    add_compile_options(-fsanitize=address,undefined -fno-omit-frame-pointer)
    add_link_options(-fsanitize=address,undefined)
endif()

if(ENABLE_TSAN)
    add_compile_options(-fsanitize=thread -fno-omit-frame-pointer)
    add_link_options(-fsanitize=thread)
endif()

# Global include directory
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src/include)

# Collect source files
file(GLOB_RECURSE STORAGE_ENGINE_SOURCES 
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
)

# Only build library if source files exist
if(STORAGE_ENGINE_SOURCES)
    add_library(storage_engine STATIC ${STORAGE_ENGINE_SOURCES})
    target_include_directories(storage_engine PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src/include)
else()
    add_custom_target(storage_engine_placeholder
        COMMAND ${CMAKE_COMMAND} -E echo "No source files in src/ yet."
    )
endif()

# Enable Testing
enable_testing()

file(GLOB_RECURSE TEST_SOURCES
    "${CMAKE_CURRENT_SOURCE_DIR}/test/*.cpp"
)

foreach(TEST_FILE ${TEST_SOURCES})
    get_filename_component(TEST_NAME ${TEST_FILE} NAME_WE)
    add_executable(${TEST_NAME} ${TEST_FILE})
    if(TARGET storage_engine)
        target_link_libraries(${TEST_NAME} PRIVATE storage_engine pthread)
    else()
        target_link_libraries(${TEST_NAME} PRIVATE pthread)
    endif()
    add_test(NAME ${TEST_NAME} COMMAND ${TEST_NAME})
endforeach()
