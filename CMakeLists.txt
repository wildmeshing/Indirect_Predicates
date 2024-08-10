# Detects whether this is a top-level project
get_directory_property(HAS_PARENT PARENT_DIRECTORY)
if(HAS_PARENT)
    set(INDIRECTPREDICATES_TOPLEVEL_PROJECT OFF)
else()
    set(INDIRECTPREDICATES_TOPLEVEL_PROJECT ON)
endif()

# Check required CMake version
set(REQUIRED_CMAKE_VERSION "3.14.0")
if(INDIRECTPREDICATES_TOPLEVEL_PROJECT)
    cmake_minimum_required(VERSION ${REQUIRED_CMAKE_VERSION})
else()
    # Don't use cmake_minimum_required here to avoid implicitly overriding parent policies
    if(${CMAKE_VERSION} VERSION_LESS ${REQUIRED_CMAKE_VERSION})
        message(FATAL_ERROR "CMake required version to build IndirectPredicates is ${REQUIRED_CMAKE_VERSION}")
    endif()
endif()

################################################################################
# CMake Policies
################################################################################

cmake_policy(SET CMP0054 NEW) # Only interpret if() arguments as variables or keywords when unquoted.
cmake_policy(SET CMP0076 NEW) # target_sources() command converts relative paths to absolute.
if(CMAKE_VERSION VERSION_GREATER_EQUAL "3.24")
  cmake_policy(SET CMP0135 NEW) # Set the timestamps of all extracted contents to the time of the extraction.
endif()

################################################################################

project(IndirectPredicates DESCRIPTION "A simple C++ library to write Paraview Files" LANGUAGES C CXX)

# option(INDIRECTPREDICATES_WITH_TESTS       "Enables unit test"                  ON)
# option(INDIRECTPREDICATES_BUILD_DOCS       "Build documentation using Doxygen" OFF)
option(INDIRECTPREDICATES_SSE2       "Use SSE2" OFF)
option(INDIRECTPREDICATES_AVX2       "Use AVX2" OFF)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/indirectpredicates/")
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/recipes/")

# Set default minimum C++ standard
if(INDIRECTPREDICATES_TOPLEVEL_PROJECT)
    set(CMAKE_CXX_STANDARD 17)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
    set(CMAKE_CXX_EXTENSIONS OFF)
endif()

### Configuration
# set(INDIRECTPREDICATES_SOURCE_DIR "${PROJECT_SOURCE_DIR}/src/indirectpredicates")
set(INDIRECTPREDICATES_INCLUDE_DIR "${PROJECT_SOURCE_DIR}/include")

list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/indirectpredicates/")
list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/recipes/")

# General CMake utils
include(indirectpredicates_cpm_cache)
include(indirectpredicates_use_colors)

# # Generate position independent code by default
# set(CMAKE_POSITION_INDEPENDENT_CODE ON)

################################################################################

################################################################################
# IndirectPredicates library
################################################################################

add_library(indirectpredicates INTERFACE)
add_library(indirectpredicates::indirectpredicates ALIAS indirectpredicates)
# set_property(TARGET indirectpredicates PROPERTY COMPILE_WARNING_AS_ERROR ON)

# Add source and header files to indirectpredicates
# add_subdirectory(${INDIRECTPREDICATES_SOURCE_DIR})

# Public include directory for indirectpredicates
target_include_directories(indirectpredicates INTERFACE ${INDIRECTPREDICATES_INCLUDE_DIR})

# Use C++20
target_compile_features(indirectpredicates INTERFACE cxx_std_17)

if (MSVC)
    add_compile_options(/bigobj)
    # grant IEEE 754 compliance
  	target_compile_options(indirectpredicates INTERFACE "/fp:strict")
	  # use intrinsic functions
	  target_compile_options(indirectpredicates INTERFACE "/Oi")
	  # set target architecture
    if(INDIRECTPREDICATES_AVX2)
      target_compile_options(indirectpredicates INTERFACE "/arch:AVX2")
    elseif(INDIRECTPREDICATES_SSE2)
      target_compile_options(indirectpredicates INTERFACE "/arch:SSE2")
	  endif()
	  # reserve enough stack size
	  target_link_options(indirectpredicates INTERFACE "/STACK:8421376")
	  # turn off annoying warnings
	  target_compile_options(indirectpredicates INTERFACE "/D _CRT_SECURE_NO_WARNINGS")
else()
    # set standard optimization level
    #target_compile_options(indirectpredicates INTERFACE -O2)
    # reserve enough stack size	
    target_compile_options(indirectpredicates INTERFACE -Wl,-z,stacksize=8421376)
    # grant IEEE 754 compliance
    target_compile_options(indirectpredicates INTERFACE -frounding-math)
    # set target architecture
    if(INDIRECTPREDICATES_AVX2)
      target_compile_options(indirectpredicates INTERFACE "-mavx2")
        target_compile_options(indirectpredicates INTERFACE "-mfma")
    elseif(INDIRECTPREDICATES_SSE2)
      target_compile_options(indirectpredicates INTERFACE "-msse2")
    endif()
endif ()

################################################################################
# Required libraries
################################################################################

# Extra warnings (link this here so it has top priority)
# include(indirectpredicates_warnings)
# target_link_libraries(indirectpredicates PRIVATE indirectpredicates::warnings)

# # Documentation
# if(INDIRECTPREDICATES_TOPLEVEL_PROJECT AND INDIRECTPREDICATES_BUILD_DOCS)
#     # check if Doxygen is installed
#     find_package(Doxygen)
#     if (DOXYGEN_FOUND)
#         add_custom_target(indirectpredicates_doc_doxygen ALL
#             COMMAND ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile
#             WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
#             COMMENT "Generating API documentation with Doxygen"
#             VERBATIM)
#     else (DOXYGEN_FOUND)
#         message("Doxygen need to be installed to generate the doxygen documentation")
#     endif (DOXYGEN_FOUND)
# endif()

# Compile extras only if this is a top-level project
# if(INDIRECTPREDICATES_TOPLEVEL_PROJECT AND INDIRECTPREDICATES_WITH_TESTS)
#     include(CTest)
#     enable_testing()
#     add_subdirectory(tests)
# endif()
