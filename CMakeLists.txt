cmake_minimum_required(VERSION 3.22)

project(neovim_client_cpp)

find_package(Boost 1.55.0 REQUIRED COMPONENTS system coroutine context thread filesystem timer)

set(CMAKE_CXX_STANDARD 17)
set(MSGPACK_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/external/msgpack-c)

#add_subdirectory(${MSGPACK_SOURCE_DIR})
set(CMAKE_CXX_FLAGS "-DMSGPACK_USE_BOOST ${CMAKE_CXX_FLAGS}") #TODO: import every flags directly from msgpack-c

add_executable(test 
    src/main.cpp
    src/socket.cpp
)

target_link_libraries(test PRIVATE Boost::boost Boost::system Boost::filesystem Boost::timer Boost::thread)
target_include_directories(test PUBLIC ${MSGPACK_SOURCE_DIR}/include include gen)

find_package(Python3 COMPONENTS Interpreter REQUIRED)
if (Python3_Interpreter_FOUND)
    add_custom_target(auto_gen_api
        COMMAND ${Python3_EXECUTABLE} ${CMAKE_SOURCE_DIR}/gen_api.py
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
        COMMENT "Autogen nvim interface"
    )
endif()

add_dependencies(test auto_gen_api)
