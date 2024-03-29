cmake_minimum_required(VERSION 3.14)
project(ProtobufTutorial)

set(CMAKE_CXX_STANDARD 11)

include(FindProtobuf)
find_package(Protobuf REQUIRED)
include_directories(${PROTOBUF_INCLUDE_DIR})

if(protobuf_VERBOSE)
    message(STATUS "Using Protocol Buffers ${Protobuf_VERSION}")
endif()

set(CMAKE_INCLUDE_CURRENT_DIR TRUE)

if(MSVC AND protobuf_MSVC_STATIC_RUNTIME)
    foreach(flag_var
            CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
            CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO)
        if(${flag_var} MATCHES "/MD")
            string(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
        endif(${flag_var} MATCHES "/MD")
    endforeach()
endif()

foreach(example add_person list_people guide_client)
    set(${example}_SRCS ${example}.cc)
    set(${example}_PROTOS addressbook.proto guide.proto)

    #Code Generation
    if(protobuf_MODULE_COMPATIBLE) #Legacy Support
        protobuf_generate_cpp(${example}_PROTO_SRCS ${example}_PROTO_HDRS ${${example}_PROTOS})
        list(APPEND ${example}_SRCS ${${example}_PROTO_SRCS} ${${example}_PROTO_HDRS})
    endif()

    #Executable setup
    set(executable_name ${example}_cpp)
    add_executable(${executable_name} ${${example}_SRCS} ${${example}_PROTOS})
    if(protobuf_MODULE_COMPATIBLE) #Legacy mode
        target_include_directories(${executable_name} PUBLIC ${PROTOBUF_INCLUDE_DIRS})
        target_link_libraries(${executable_name} ${PROTOBUF_LIBRARIES})
    else()
        target_link_libraries(${executable_name} protobuf::libprotobuf)
        protobuf_generate(TARGET ${executable_name})
    endif()

endforeach()
