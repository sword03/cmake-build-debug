# the `pkg_check_modules` function is created with this call
set(CMAKE_CXX_STANDARD 11)
message(STATUS "CMAKE_CXX_COMPILER = ${CMAKE_CXX_COMPILER}")
set(CMAKE_PREFIX_PATH "/opt/protobuf/lib/pkgconfig")
set(ENV{PKG_CONFIG_PATH}  "/opt/protobuf/lib/pkgconfig")
set(PROTOC  "/opt/protobuf/bin/protoc")
find_package(PkgConfig REQUIRED)
pkg_search_module(PROTOBUF_OE_ENCLAVE REQUIRED protobuf_oe_enclave)  # this looks for opencv.pc file


add_custom_command(OUTPUT hello.pb.cc hello.pb.h
        DEPENDS ${CMAKE_SOURCE_DIR}/hello.proto
        COMMAND ${PROTOC} -I=${CMAKE_SOURCE_DIR} --cpp_out=${CMAKE_CURRENT_BINARY_DIR} hello.proto)
message(STATUS "COMMAND: ${PROTOC} -I=${CMAKE_SOURCE_DIR} --cpp_out=${CMAKE_CURRENT_BINARY_DIR} hello.proto")

# Use the edger8r to generate C bindings from the EDL file.
add_custom_command(OUTPUT helloworld_t.h helloworld_t.c helloworld_args.h
        DEPENDS ${CMAKE_SOURCE_DIR}/helloworld.edl
        COMMAND openenclave::oeedger8r --trusted ${CMAKE_SOURCE_DIR}/helloworld.edl)
message(STATUS "COMMAND: openenclave::oeedger8r --trusted ${CMAKE_SOURCE_DIR}/helloworld.edl")

add_executable(enclave enc.cpp ${CMAKE_CURRENT_BINARY_DIR}/helloworld_t.c ${CMAKE_CURRENT_BINARY_DIR}/hello.pb.cc)

if (WIN32)
    maybe_build_using_clangw(enclave)
endif ()

include_directories(${PROTOBUF_OE_ENCLAVE_INCLUDE_DIRS})

target_compile_definitions(enclave PUBLIC OE_API_VERSION=2)

message(STATUS "CMAKE_CURRENT_SOURCE_DIR = ${CMAKE_CURRENT_SOURCE_DIR}")
message(STATUS "CMAKE_CURRENT_BINARY_DIR = ${CMAKE_CURRENT_BINARY_DIR}")
message(STATUS "PROTOBUF_OE_ENCLAVE_INCLUDE_DIRS = ${PROTOBUF_OE_ENCLAVE_INCLUDE_DIRS}")
message(STATUS "PROTOBUF_OE_ENCLAVE_LIBRARY_DIRS = ${PROTOBUF_OE_ENCLAVE_LIBRARY_DIRS}")
message(STATUS "PROTOBUF_OE_ENCLAVE_LIBRARIES = ${PROTOBUF_OE_ENCLAVE_LIBRARIES}")
message(STATUS "PROTOBUF_OE_ENCLAVE_LDFLAGS = ${PROTOBUF_OE_ENCLAVE_LDFLAGS}")


# Need for the generated file helloworld_t.h
target_include_directories(enclave PRIVATE ${CMAKE_CURRENT_BINARY_DIR})


target_link_libraries(enclave ${PROTOBUF_OE_ENCLAVE_LDFLAGS} openenclave::oeenclave openenclave::oelibc openenclave::oelibcxx)

message(STATUS "\n\n")