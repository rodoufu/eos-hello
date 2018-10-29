file(GLOB hello_CURRENT_SRC ABSOLUTE "*.cpp")

message(STATUS "Files contracts.cmake ${CURRENT_SRC}")

createContract(hello ${CMAKE_CURRENT_BINARY_DIR} ${hello_CURRENT_SRC})
pushContract(hello hi bob bob)
