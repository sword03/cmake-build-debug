# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License.
set(CMAKE_CXX_STANDARD 11)

set(CMAKE_PREFIX_PATH "/opt/protobuf/lib/pkgconfig")
set(ENV{PKG_CONFIG_PATH}  "/opt/protobuf/lib/pkgconfig")
set(PROTOC  "/opt/protobuf/bin/protoc")
find_package(PkgConfig REQUIRED)
pkg_search_module(PROTOBUF REQUIRED protobuf)  # this looks for opencv.pc file

add_custom_command(OUTPUT hello.pb.cc hello.pb.h
        DEPENDS ${CMAKE_SOURCE_DIR}/hello.proto
        COMMAND ${PROTOC} -I=${CMAKE_SOURCE_DIR} --cpp_out=${CMAKE_CURRENT_BINARY_DIR} hello.proto)
message(STATUS "COMMAND: ${PROTOC} -I=${CMAKE_SOURCE_DIR} --cpp_out=${CMAKE_CURRENT_BINARY_DIR} hello.proto")

add_custom_command(OUTPUT helloworld_u.h helloworld_u.c helloworld_args.h
        DEPENDS ${CMAKE_SOURCE_DIR}/helloworld.edl
        COMMAND openenclave::oeedger8r --untrusted ${CMAKE_SOURCE_DIR}/helloworld.edl)
message(STATUS "COMMAND: openenclave::oeedger8r --untrusted ${CMAKE_SOURCE_DIR}/helloworld.edl")

add_executable(helloworld_host host.cpp ${CMAKE_CURRENT_BINARY_DIR}/helloworld_u.c ${CMAKE_CURRENT_BINARY_DIR}/hello.pb.cc)

include_directories(${PROTOBUF_INCLUDE_DIRS} ${CMAKE_CURRENT_BINARY_DIR})

message(STATUS "CMAKE_CURRENT_SOURCE_DIR = ${CMAKE_CURRENT_SOURCE_DIR}")
message(STATUS "CMAKE_CURRENT_BINARY_DIR = ${CMAKE_CURRENT_BINARY_DIR}")
message(STATUS "PROTOBUF_INCLUDE_DIRS = ${PROTOBUF_INCLUDE_DIRS}")
message(STATUS "PROTOBUF_LIBRARY_DIRS = ${PROTOBUF_LIBRARY_DIRS}")
message(STATUS "PROTOBUF_LIBRARIES = ${PROTOBUF_LIBRARIES}")
message(STATUS "PROTOBUF_LDFLAGS = ${PROTOBUF_LDFLAGS}")

target_include_directories(helloworld_host PRIVATE
        # Needed for the generated file helloworld_u.h
        ${CMAKE_CURRENT_BINARY_DIR} ${PROTOBUF_LIBRARY_DIRS})

target_link_libraries(helloworld_host openenclave::oehostapp ${PROTOBUF_LDFLAGS})

message(STATUS "\n\n")