set(PROTO_PATH "${CMAKE_CURRENT_SOURCE_DIR}/protos")
set(GENERATED_PROTOBUF_PATH "${CMAKE_CURRENT_BINARY_DIR}/generated")
file(MAKE_DIRECTORY ${GENERATED_PROTOBUF_PATH})

# Generated grpc and proto sources
set(HELLO_PROTO "${PROTO_PATH}/hello.proto")
set(HELLO_PROTO_HEADER "${GENERATED_PROTOBUF_PATH}/hello.pb.h")
set(HELLO_PROTO_SRC "${GENERATED_PROTOBUF_PATH}/hello.pb.cc")
set(HELLO_GRPC_HEADER "${GENERATED_PROTOBUF_PATH}/hello.grpc.pb.h")
set(HELLO_GRPC_SRC "${GENERATED_PROTOBUF_PATH}/hello.grpc.pb.cc")


set(ENV{PKG_CONFIG_PATH} C:/develop/vcpkg-2020.04/installed/x64-windows/lib/pkgconfig)
find_package(pkgconfig)
pkg_search_module(GRPCPP_UNSECURE REQUIRED grpc++_unsecure)

MESSAGE(STATUS ${GRPCPP_LIBRARIES})
MESSAGE(STATUS ${GRPCPP_INCLUDE_DIRS})

add_custom_command(
    OUTPUT "${HELLO_PROTO_SRC}"
           "${HELLO_PROTO_HEADER}"
    COMMAND "C:/develop/vcpkg-2020.04/installed/x64-windows/tools/protobuf/protoc.exe"
    ARGS -I "${PROTO_PATH}"
         --cpp_out="${GENERATED_PROTOBUF_PATH}"
         "${HELLO_PROTO}"
    DEPENDS "${HELLO_PROTO}"
)

add_custom_command(
    OUTPUT "${HELLO_GRPC_SRC}"
           "${HELLO_GRPC_HEADER}"
    COMMAND "C:/develop/vcpkg-2020.04/installed/x64-windows/tools/protobuf/protoc.exe"
    ARGS --grpc_out "${GENERATED_PROTOBUF_PATH}" -I "${PROTO_PATH}"
         --plugin=protoc-gen-grpc="C:/develop/vcpkg-2020.04/installed/x64-windows/tools/grpc/grpc_cpp_plugin.exe"
         "${HELLO_PROTO}"
    DEPENDS "${HELLO_PROTO}"
)

include_directories(C:/develop/vcpkg-2020.04/installed/x64-windows/include)
link_directories(C:/develop/vcpkg-2020.04/installed/x64-windows/lib)

# Generated include directory
include_directories("${GENERATED_PROTOBUF_PATH}")
add_definitions(-D_WIN32_WINNT=0x600 /std:c++17 )
foreach(_target
  async_hello_server async_hello_client)
  add_executable(${_target} "${_target}.cc"
    ${HELLO_PROTO_SRC}
    ${HELLO_GRPC_SRC})
  target_link_libraries(${_target}
    ${GRPCPP_UNSECURE_LIBRARIES}
    ${LIBCARES_LIBRARIES}
    upb
    ws2_32
    zlib
    address_sorting
    libprotobuf)
endforeach()
