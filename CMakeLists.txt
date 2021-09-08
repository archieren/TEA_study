cmake_minimum_required(VERSION 3.10)
project(teaserpp VERSION 1.0.0)

set(CMAKE_CXX_STANDARD 14)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules/" ${CMAKE_MODULE_PATH})

# Check build types
set(CMAKE_BUILD_TYPE "Release")

# Options
option(BUILD_PYTHON_BINDINGS "Build Python bindings" ON)
option(ENABLE_DIAGNOSTIC_PRINT "Enable printing of diagnostic messages" ON)

if (ENABLE_DIAGNOSTIC_PRINT)
    message(STATUS "Enable printing of diagnostic messages.")
    add_definitions(-DTEASER_DIAG_PRINT)
endif ()

# Cache Variables
if (NOT TEASERPP_PYTHON_VERSION)
    set(TEASERPP_PYTHON_VERSION "" CACHE STRING "Python version to use for TEASER++ bindings.")
endif ()

# Find dependencies
# Eigen3
find_package(Eigen3 3.2 QUIET REQUIRED NO_MODULE)

# pmc (Parallel Maximum Clique)
configure_file(cmake/pmc.CMakeLists.txt.in pmc-download/CMakeLists.txt)
execute_process(COMMAND "${CMAKE_COMMAND}" -G "${CMAKE_GENERATOR}" .
        WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/pmc-download")
execute_process(COMMAND "${CMAKE_COMMAND}" --build .
        WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/pmc-download")
add_subdirectory("${CMAKE_BINARY_DIR}/pmc-src"
        "${CMAKE_BINARY_DIR}/pmc-build")

# spectra
configure_file(cmake/spectra.CMakeLists.txt.in spectra-download/CMakeLists.txt)
execute_process(COMMAND "${CMAKE_COMMAND}" -G "${CMAKE_GENERATOR}" .
        WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/spectra-download")
execute_process(COMMAND "${CMAKE_COMMAND}" --build .
        WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/spectra-download")
set(SPECTRA_INCLUDE_DIRS "${CMAKE_BINARY_DIR}/spectra-src/include")

# Building Targets
set(TEASERPP_ROOT ${CMAKE_CURRENT_LIST_DIR})
add_subdirectory(teaser)

if (BUILD_PYTHON_BINDINGS)
    set(PYBIND11_PYTHON_VERSION ${TEASERPP_PYTHON_VERSION})

    # download the pybind11 repo
    configure_file(cmake/pybind11.CMakeLists.txt.in pybind11-download/CMakeLists.txt)
    execute_process(COMMAND "${CMAKE_COMMAND}" -G "${CMAKE_GENERATOR}" .
            WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/pybind11-download")
    execute_process(COMMAND "${CMAKE_COMMAND}" --build .
            WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/pybind11-download")
    add_subdirectory("${CMAKE_BINARY_DIR}/pybind11-src"
            "${CMAKE_BINARY_DIR}/pybind11-build")

    message(STATUS "TEASER++ Python binding will be built.")
    add_subdirectory(python)
endif ()

# export targets
export(TARGETS ${TEASERPP_EXPORTED_TARGETS} FILE teaserpp-exports.cmake)

install(FILES cmake/teaserppConfig.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/teaserpp
        )