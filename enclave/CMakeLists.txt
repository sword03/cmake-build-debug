# the `pkg_check_modules` function is created with this call

set(CMAKE_PREFIX_PATH "/opt/protobuf/lib/pkgconfig")
set(ENV{PKG_CONFIG_PATH}  "/opt/protobuf/lib/pkgconfig")
find_package(PkgConfig REQUIRED)
pkg_search_module(PROTOBUF_OE_ENCLAVE REQUIRED protobuf_oe_enclave)  # this looks for opencv.pc file

# Use the edger8r to generate C bindings from the EDL file.
message(STATUS ${CMAKE_CURRENT_SOURCE_DIR})
message(STATUS ${CMAKE_CURRENT_BINARY_DIR})
add_custom_command(OUTPUT helloworld_t.h helloworld_t.c helloworld_args.h
        DEPENDS ${CMAKE_SOURCE_DIR}/helloworld.edl
        COMMAND openenclave::oeedger8r --trusted ${CMAKE_SOURCE_DIR}/helloworld.edl)

add_executable(enclave enc.cpp ${CMAKE_CURRENT_BINARY_DIR}/helloworld_t.c hello.pb.cc)

if (WIN32)
    maybe_build_using_clangw(enclave)
endif ()

include_directories(${PROTOBUF_OE_ENCLAVE_INCLUDE_DIRS})

target_compile_definitions(enclave PUBLIC OE_API_VERSION=2)

message(STATUS ${PROTOBUF_OE_ENCLAVE_INCLUDE_DIRS})
message(STATUS ${PROTOBUF_OE_ENCLAVE_LIBRARY_DIRS})
message(STATUS ${PROTOBUF_OE_ENCLAVE_LIBRARIES})
message(STATUS ${PROTOBUF_OE_ENCLAVE_LDFLAGS})


# Need for the generated file helloworld_t.h
#target_include_directories(enclave PRIVATE ${CMAKE_CURRENT_BINARY_DIR} ${PROTOBUF_OE_ENCLAVE_LIBRARY_DIRS})
target_include_directories(enclave PRIVATE ${CMAKE_CURRENT_BINARY_DIR})


target_link_libraries(enclave openenclave::oeenclave openenclave::oelibc openenclave::oelibcxx ${PROTOBUF_OE_ENCLAVE_LDFLAGS})# libprotobuf_oe_enclave.a)