cmake_minimum_required(VERSION 3.8)

cmake_policy(SET CMP0048 NEW)
project(SECONDPARTYMESH VERSION 1.0.0 LANGUAGES CXX)

# Options
option(SECONDPARTYMESH_BuildTests "Build the unit tests." OFF)

# For debugging, print generator, compiler, and options
message(STATUS "Gen: ${CMAKE_GENERATOR}, CXX: ${CMAKE_CXX_COMPILER_ID}, Mode: ${CMAKE_BUILD_TYPE}")
message(STATUS "Tests: ${SECONDPARTYMESH_BuildTests}")

##################
# Checking input #
##################

include(${CMAKE_SOURCE_DIR}/cmake/CommonInputChecks.cmake)

########################
# Declare dependencies #
########################


find_package(Utility CONFIG REQUIRED HINTS "${CMAKE_SOURCE_DIR}/../utility/build" "${CMAKE_SOURCE_DIR}/../utility/install_dir")


################################
# Setting up SECONDPARTYMESH target #
################################

# Set the variables for SECONDPARTYMESH
set(SECONDPARTYMESH_INCLUDE ${CMAKE_SOURCE_DIR}/include)
set(SECONDPARTYMESH_INCLUDE_SUB ${SECONDPARTYMESH_INCLUDE}/second_party_mesh)
set(SECONDPARTYMESH_SRC ${CMAKE_SOURCE_DIR}/source)
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
# Link dependencies
target_link_libraries(secondpartymesh PUBLIC Utility::Utility)


# Assign source groups
include(${CMAKE_SOURCE_DIR}/cmake/Utilities.cmake)
assign_source_group("second_party_mesh" ${SECONDPARTYMESH_HEADER_LIST})
assign_source_group("second_party_mesh" ${SECONDPARTYMESH_SOURCE_LIST})
assign_source_group_Utility()


#################################
# Add dev and unit test targets #
#################################
    
# Build tests
if(SECONDPARTYMESH_BuildTests)
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

# Export just the secondpartymesh target (not the dev, tests, etc.)
export(
    TARGETS secondpartymesh
    FILE ${CMAKE_CURRENT_BINARY_DIR}/SecondPartyMeshTargets.cmake
    NAMESPACE SecondPartyMesh::
)

# Register package in user's package registry
export(PACKAGE SecondPartyMesh)
