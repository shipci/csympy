cmake_minimum_required(VERSION 2.8)

set(CMAKE_USER_MAKE_RULES_OVERRIDE ${CMAKE_SOURCE_DIR}/cmake/UserOverride.cmake)

project(csympy)

set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

# Make sure that CMAKE_BUILD_TYPE is either Debug or Release:
if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release
        CACHE STRING "Build type (Debug, Release)" FORCE)
endif ()
if (NOT (CMAKE_BUILD_TYPE STREQUAL "Debug" OR
        CMAKE_BUILD_TYPE STREQUAL "Release"))
    message("${CMAKE_BUILD_TYPE}")
    message(FATAL_ERROR "CMAKE_BUILD_TYPE must be one of: Debug, Release (current value: '${CMAKE_BUILD_TYPE}')")
endif ()

if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    set(CMAKE_CXX_FLAGS_DEBUG
        "${CMAKE_CXX_FLAGS_DEBUG} -Wno-unused-parameter -ggdb")
    set(CMAKE_CXX_FLAGS_RELEASE
        "${CMAKE_CXX_FLAGS_RELEASE} -Wno-unused-parameter")
endif()

if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    set(CMAKE_CXX_FLAGS_DEBUG
        "${CMAKE_CXX_FLAGS_DEBUG} -Wno-unused-parameter -ggdb")
    set(CMAKE_CXX_FLAGS_RELEASE
        "${CMAKE_CXX_FLAGS_RELEASE} -Wno-unused-parameter")
endif()

# GMP
find_package(GMP REQUIRED)
include_directories(${GMP_INCLUDE_DIRS})
set(LIBS ${LIBS} ${GMP_LIBRARIES})

#ECM
set(WITH_ECM no
    CACHE BOOL "Build with ECM (libecm-dev) support")

if (WITH_ECM)
    find_package(ECM REQUIRED)
    include_directories(${ECM_INCLUDE_DIRS})
    set(LIBS ${LIBS} ${ECM_LIBRARY})
    set(HAVE_CSYMPY_ECM yes)
endif()

#PRIMESIEVE
set(WITH_PRIMESIEVE no
    CACHE BOOL "Build with primesieve")

if (WITH_PRIMESIEVE)
    find_package(PRIMESIEVE REQUIRED)
    include_directories(${PRIMESIEVE_INCLUDE_DIRS})
    set(LIBS ${LIBS} ${PRIMESIEVE_LIBRARY})
    set(HAVE_CSYMPY_PRIMESIEVE yes)
endif()

# Python
set(WITH_PYTHON no
    CACHE BOOL "Build with Python wrappers")

# BFD
set(WITH_BFD no
    CACHE BOOL "Build with BFD (binutils-dev) support")

if (WITH_BFD)
    find_package(BFD REQUIRED)
    find_package(LINKH REQUIRED)
    include_directories(${BFD_INCLUDE_DIRS} ${LINKH_INCLUDE_DIRS})
    set(LIBS ${LIBS} ${BFD_LIBRARIES})
    set(HAVE_TEUCHOS_BFD yes)
    set(HAVE_TEUCHOS_LINK yes)
endif()

# Doxygen
set(BUILD_DOXYGEN no
    CACHE BOOL "Create C++ API Doxgyen documentation.")

if (BUILD_DOXYGEN)
    find_package(Doxygen REQUIRED)
    if(DOXYGEN_FOUND)
        configure_file(${CMAKE_CURRENT_SOURCE_DIR}/doc/Doxyfile
            ${CMAKE_CURRENT_BINARY_DIR}/doc/Doxyfile @ONLY)
        add_custom_target(doc ALL
            ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/doc/Doxyfile
            WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/doc
            COMMENT "Generating API documentation with Doxygen" VERBATIM)
        install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/doc/html/ DESTINATION doc)
    endif(DOXYGEN_FOUND)
endif()

# CSYMPY_ASSERT
set(WITH_CSYMPY_ASSERT no
    CACHE BOOL "Enable CSYMPY_ASSERT macro")

# CSYMPY_RCP
set(WITH_CSYMPY_RCP yes
    CACHE BOOL "Enable CSYMPY_RCP support")

# CSYMPY_THREAD_SAFE
set(WITH_CSYMPY_THREAD_SAFE no
    CACHE BOOL "Enable CSYMPY_THREAD_SAFE support")

if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(HAVE_TEUCHOS_DEBUG yes)
    set(HAVE_TEUCHOS_DEBUG_RCP_NODE_TRACING yes)
    set(HAVE_CSYMPY_RCP yes)
endif()

enable_testing()
add_subdirectory(src)
add_subdirectory(benchmarks)

if (WITH_PYTHON)
    # We need to execute the find_package() commands here, so that the
    # variables like ${PYTHON_INSTALL_PATH} are defined here and in all cmake
    # files (as opposed to just subset of cmake files).
    find_package(Python REQUIRED)
    find_package(Cython REQUIRED)
    add_subdirectory(csympy)
endif ()


message("\n")
message("Configuration results")
message("---------------------")
message("C++ compiler: ${CMAKE_CXX_COMPILER}")
message("Build type: ${CMAKE_BUILD_TYPE}")
if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    message("C++ compiler flags: ${CMAKE_CXX_FLAGS_DEBUG}")
else ()
    message("C++ compiler flags: ${CMAKE_CXX_FLAGS_RELEASE}")
endif ()
message("Installation prefix: ${CMAKE_INSTALL_PREFIX}")

message("WITH_CSYMPY_ASSERT: ${WITH_CSYMPY_ASSERT}")
message("WITH_CSYMPY_RCP: ${WITH_CSYMPY_RCP}")
message("WITH_CSYMPY_THREAD_SAFE: ${WITH_CSYMPY_THREAD_SAFE}")

message("GMP_INCLUDE_DIRS: ${GMP_INCLUDE_DIRS}")
message("GMP_LIBRARIES: ${GMP_LIBRARIES}")

message("WITH_BFD: ${WITH_BFD}")
if (WITH_BFD)
    message("BFD_INCLUDE_DIRS: ${BFD_INCLUDE_DIRS}")
    message("BFD_LIBRARIES: ${BFD_LIBRARIES}")
    message("LINKH_INCLUDE_DIRS: ${LINKH_INCLUDE_DIRS}")
endif()

message("WITH_PYTHON: ${WITH_PYTHON}")
if (WITH_PYTHON)
    message("Python library: ${PYTHON_LIBRARY}")
    message("Python include path: ${PYTHON_INCLUDE_PATH}")
    message("Python install path: ${PYTHON_INSTALL_PATH}")

    message("Cython path: ${CYTHON_BIN}")
endif ()

message("WITH_ECM: ${WITH_ECM}")
if (WITH_ECM)
    message("ECM_INCLUDE_DIRS: ${ECM_INCLUDE_DIRS}")
    message("ECM_LIBRARIES: ${ECM_LIBRARY}")
endif()

message("WITH_PRIMESIEVE: ${WITH_PRIMESIEVE}")
if (WITH_PRIMESIEVE)
    message("PRIMESIEVE_INCLUDE_DIRS: ${PRIMESIEVE_INCLUDE_DIRS}")
    message("PRIMESIEVE_LIBRARIES: ${PRIMESIEVE_LIBRARY}")
endif()

message("Copying source of python wrappers into: ${CMAKE_CURRENT_BINARY_DIR}")
file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/csympy DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
