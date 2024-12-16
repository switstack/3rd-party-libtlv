cmake_minimum_required(VERSION 3.21)

project(
    tlv
    LANGUAGES CXX
)

# Define some paths
set(PROJECT_DIRECTORY "${CMAKE_CURRENT_LIST_DIR}")
set(INCLUDE_DIRECTORY ${PROJECT_DIRECTORY}/include)
set(SOURCE_DIRECTORY ${PROJECT_DIRECTORY}/src)
set(TEST_SOURCE_DIRECTORY ${PROJECT_DIRECTORY}/test/src)
set(CMAKE_DIRECTORY ${PROJECT_DIRECTORY}/cmake)

# Include external modules
include(GNUInstallDirs)

# Define project options
set(TLV_INSTALL ON CACHE BOOL "Indicate whether output files must be installed")
set(TLV_BUILD_TESTS OFF CACHE BOOL "Whether to build the test application")

# Set up targets
add_library(
    ${PROJECT_NAME}
    ${SOURCE_DIRECTORY}/tlv.cpp
)
target_include_directories(
    ${PROJECT_NAME}
    PUBLIC
    $<BUILD_INTERFACE:${INCLUDE_DIRECTORY}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_compile_options(
        ${PROJECT_NAME}
        PUBLIC
        --std=c++11 -Werror -Wall -Wextra -pedantic -pedantic-errors
    )
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_compile_options(
        ${PROJECT_NAME}
        PUBLIC
        /std:c++14 /WX /W4 /EHsc
    )
endif()

if(TLV_INSTALL)
    include(CMakePackageConfigHelpers)

    install(
        DIRECTORY
            ${INCLUDE_DIRECTORY}/tlv
        DESTINATION
            ${CMAKE_INSTALL_INCLUDEDIR}
    )
    install(
        TARGETS
            ${PROJECT_NAME}
        EXPORT
            ${PROJECT_NAME}-targets
        ARCHIVE
            DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY
            DESTINATION ${CMAKE_INSTALL_LIBDIR}
        PUBLIC_HEADER
            DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    )
    install(
        EXPORT
            ${PROJECT_NAME}-targets
        FILE
            ${PROJECT_NAME}-targets.cmake
        NAMESPACE
            ${PROJECT_NAME}::
        DESTINATION
            ${CMAKE_INSTALL_LIBDIR}/cmake
    )
    configure_package_config_file(
        ${CMAKE_DIRECTORY}/Config.cmake.in
        ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config.cmake
        INSTALL_DESTINATION
            ${CMAKE_INSTALL_LIBDIR}/cmake
    )
    install(
        FILES
            ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config.cmake
        DESTINATION
            ${CMAKE_INSTALL_LIBDIR}/cmake
    )
endif()

if(TLV_BUILD_TESTS)
    # FIXME Commented as CMake script seems broken (see https://github.com/cpputest/cpputest/issues/1639)
    #find_package(CppUTest REQUIRED)

    add_executable(
        ${PROJECT_NAME}-test
        ${TEST_SOURCE_DIRECTORY}/test.cpp
    )
    target_link_libraries(
        ${PROJECT_NAME}-test
        PRIVATE
        ${PROJECT_NAME}
        CppUTestExt
        CppUTest
    )

    enable_testing()

    # FIXME Doing it the old way as CppUTest package is not found by CMake out-of-the-box
    add_test(
        NAME
            ${PROJECT_NAME}-test
        COMMAND
            ${PROJECT_NAME}-test -v
    )
endif()
