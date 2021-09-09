cmake_minimum_required(VERSION 3.8)

cmake_policy(SET CMP0048 NEW)
project(SECONDPARTYMESH VERSION 1.0.0 LANGUAGES CXX)

# Options
option(SHEFF_BuildTests "Build the unit tests." OFF)

# For debugging, print generator, compiler, and options
message(STATUS "Gen: ${CMAKE_GENERATOR}, CXX: ${CMAKE_CXX_COMPILER_ID}, Mode: ${CMAKE_BUILD_TYPE}")
message(STATUS "Tests: ${SHEFF_BuildTests}")

# Add cmake dir to module path
list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")

##################
# Checking input #
##################

include(${PROJECT_SOURCE_DIR}/cmake/CommonInputChecks.cmake)

########################
# Declare dependencies #
########################

if(NOT TARGET Utility::Utility)
    message(STATUS "Utility did not already exist, finding package.")
    find_package(Utility CONFIG REQUIRED NO_CMAKE_SYSTEM_PACKAGE_REGISTRY HINTS 
        "${PROJECT_SOURCE_DIR}/../../first_party/utility/build"
        NO_CMAKE_PACKAGE_REGISTRY)
endif()

################################
# Setting up SECONDPARTYMESH target #
################################

# Set the variables for SECONDPARTYMESH
set(SECONDPARTYMESH_INCLUDE ${PROJECT_SOURCE_DIR}/include)
set(SECONDPARTYMESH_SRC ${PROJECT_SOURCE_DIR}/source)
file (GLOB_RECURSE SECONDPARTYMESH_HEADER_LIST "${SECONDPARTYMESH_INCLUDE}/*.h")
file (GLOB_RECURSE SECONDPARTYMESH_SOURCE_LIST "${SECONDPARTYMESH_SRC}/*.h" "${SECONDPARTYMESH_SRC}/*.cpp")
    
# Create target
add_library(secondpartymesh STATIC)
# Add the header includes and headers to the library
target_include_directories(secondpartymesh PUBLIC
    $<BUILD_INTERFACE:${SECONDPARTYMESH_INCLUDE}>
    $<INSTALL_INTERFACE:include>)
# Add headers in the include folder as PUBLIC so downstream
# targets have access to them
target_sources(secondpartymesh PUBLIC ${SECONDPARTYMESH_HEADER_LIST})
# Add headers and sources in the source folder as PRIVATE as they
# will not be needed downstream
target_sources(secondpartymesh PRIVATE ${SECONDPARTYMESH_SOURCE_LIST})
# Add an alias so that a library can be used inside the build tree, eg when testing
add_library(SecondPartyMesh::SecondPartyMesh ALIAS secondpartymesh)
# Add C++ 14 flag
target_compile_features(secondpartymesh PUBLIC cxx_std_14)
# Define TETLIBRARY, which macros the interface in place for the main entry point
target_compile_definitions(secondpartymesh PUBLIC TETLIBRARY)
set_target_properties(secondpartymesh PROPERTIES FOLDER "sheff")
# Link dependencies
target_link_libraries(secondpartymesh PUBLIC Utility::Utility)

###########################
# Assigning source groups #
###########################

include("$ENV{SHEFF_TOOLS}/cmake/IncludeSHEFF.cmake")
register_sources_sheff("second_party_mesh"
                       "${SECONDPARTYMESH_HEADER_LIST}"
                       "${SECONDPARTYMESH_SOURCE_LIST}")
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
set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/SecondPartyMesh)

# Install the secondpartymesh target (library and headers)
install(
    TARGETS secondpartymesh
    EXPORT SecondPartyMeshTargets
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    INCLUDES DESTINATION include
)
# This is required so that the exported target has the name SecondPartyMesh and not secondpartymesh
set_target_properties(secondpartymesh PROPERTIES EXPORT_NAME SecondPartyMesh)

# Create a ConfigVersion.cmake file
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/SecondPartyMeshConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

# Create the package config file for find_package(SecondPartyMesh)
configure_package_config_file(
    ${CMAKE_CURRENT_LIST_DIR}/cmake/SecondPartyMeshConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/SecondPartyMeshConfig.cmake
    INSTALL_DESTINATION ${INSTALL_CONFIGDIR}
)

# Install the config and config version files
# Note: if you write custom find modules, add them here
# Refer to https://github.com/pabloariasal/modern-cmake-sample/blob/cfda7b51ff915409fd61d4f2e75b63032bedb182/libjsonutils/CMakeLists.txt
install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/SecondPartyMeshConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/SecondPartyMeshConfigVersion.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/SecondPartyMeshTargets.cmake
    DESTINATION ${INSTALL_CONFIGDIR}
)

###########
# Exports #
###########

# Export just the secondpartymesh target (not tests, etc.)
export(
    TARGETS secondpartymesh
    FILE ${CMAKE_CURRENT_BINARY_DIR}/SecondPartyMeshTargets.cmake
    NAMESPACE SecondPartyMesh::
)
