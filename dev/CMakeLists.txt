add_executable(secondpartymesh_dev source/main.cpp)
target_link_libraries(secondpartymesh_dev PRIVATE SecondPartyMesh::SecondPartyMesh)

include(${CMAKE_SOURCE_DIR}/cmake/Utilities.cmake)
assign_source_group("second_party_mesh" ${SECONDPARTYMESH_HEADER_LIST})
assign_source_group("second_party_mesh" ${SECONDPARTYMESH_SOURCE_LIST})
assign_source_group_Utility()
