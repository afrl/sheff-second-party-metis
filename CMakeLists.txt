cmake_minimum_required(VERSION 3.8)

cmake_policy(SET CMP0048 NEW)
project(SECONDPARTYMETIS VERSION 1.0.0 LANGUAGES C)

# Options
option(SHEFF_BuildTests "Build the unit tests." OFF)

# For debugging, print generator, compiler, and options
message(STATUS "Gen: ${CMAKE_GENERATOR}, C: ${CMAKE_C_COMPILER_ID}, Mode: ${CMAKE_BUILD_TYPE}")
message(STATUS "Tests: ${SHEFF_BuildTests}")

# Add cmake dir to module path
list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")

##################
# Checking input #
##################

include(${PROJECT_SOURCE_DIR}/cmake/CommonInputChecks.cmake)

################################
# Setting up SECONDPARTYMETIS target #
################################

# Required for some non-standard functions in GNU
if (${CMAKE_C_COMPILER_ID} STREQUAL "GNU" )
    add_definitions(-D_XOPEN_SOURCE=700)
endif()

# Set the variables for SECONDPARTYMETIS
set(SECONDPARTYMETIS_INCLUDE ${PROJECT_SOURCE_DIR}/include)
set(SECONDPARTYMETIS_SRC ${PROJECT_SOURCE_DIR}/source)
file (GLOB_RECURSE SECONDPARTYMETIS_HEADER_LIST "${SECONDPARTYMETIS_INCLUDE}/*.h")
file (GLOB_RECURSE SECONDPARTYMETIS_SOURCE_LIST "${SECONDPARTYMETIS_SRC}/*.h" "${SECONDPARTYMETIS_SRC}/*.c")
    
# Create target
add_library(secondpartymetis STATIC)
set_target_properties(secondpartymetis PROPERTIES LINKER_LANGUAGE C)
# Special definitions for metis needed for Windows
if (WIN32)
    target_compile_definitions(secondpartymetis PUBLIC USE_GKREGEX)
    target_compile_definitions(secondpartymetis PUBLIC "__thread=__declspec(thread)")
endif()
# Add the header includes and headers to the library
target_include_directories(secondpartymetis PUBLIC
    $<BUILD_INTERFACE:${SECONDPARTYMETIS_INCLUDE}>
    $<INSTALL_INTERFACE:include>)
# Add headers in the include folder as PUBLIC so downstream
# targets have access to them
target_sources(secondpartymetis PUBLIC ${SECONDPARTYMETIS_HEADER_LIST})
# Add headers and sources in the source folder as PRIVATE as they
# will not be needed downstream
target_sources(secondpartymetis PRIVATE ${SECONDPARTYMETIS_SOURCE_LIST})
# Add an alias so that a library can be used inside the build tree, eg when testing
add_library(SecondPartyMetis::SecondPartyMetis ALIAS secondpartymetis)
set_target_properties(secondpartymetis PROPERTIES FOLDER "sheff")

###########################
# Assigning source groups #
###########################

include("$ENV{SHEFF_TOOLS}/cmake/IncludeSHEFF.cmake")
register_sources_sheff("second_party_metis"
                       "${SECONDPARTYMETIS_HEADER_LIST}"
                       "${SECONDPARTYMETIS_SOURCE_LIST}")
assign_source_groups_sheff()

#################################
# Add unit test targets #
#################################
    
# Build tests
if(SHEFF_BuildTests)
    include(CTest)
    enable_testing()
    add_subdirectory(unit_tests)
endif()

################
# Installation #
################

include(GNUInstallDirs)
set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/SecondPartyMetis)

# Install the secondpartymetis target (library and headers)
install(
    TARGETS secondpartymetis
    EXPORT SecondPartyMetisTargets
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    INCLUDES DESTINATION include
)
# This is required so that the exported target has the name SecondPartyMetis and not secondpartymetis
set_target_properties(secondpartymetis PROPERTIES EXPORT_NAME SecondPartyMetis)

# Create a ConfigVersion.cmake file
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/SecondPartyMetisConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

# Create the package config file for find_package(SecondPartyMetis)
configure_package_config_file(
    ${CMAKE_CURRENT_LIST_DIR}/cmake/SecondPartyMetisConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/SecondPartyMetisConfig.cmake
    INSTALL_DESTINATION ${INSTALL_CONFIGDIR}
)

# Install the config and config version files
# Note: if you write custom find modules, add them here
# Refer to https://github.com/pabloariasal/modern-cmake-sample/blob/cfda7b51ff915409fd61d4f2e75b63032bedb182/libjsonutils/CMakeLists.txt
install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/SecondPartyMetisConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/SecondPartyMetisConfigVersion.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/SecondPartyMetisTargets.cmake
    DESTINATION ${INSTALL_CONFIGDIR}
)

###########
# Exports #
###########

# Export just the secondpartymetis target (not tests, etc.)
export(
    TARGETS secondpartymetis
    FILE ${CMAKE_CURRENT_BINARY_DIR}/SecondPartyMetisTargets.cmake
    NAMESPACE SecondPartyMetis::
)
