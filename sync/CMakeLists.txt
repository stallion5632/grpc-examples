set(PROTO_PATH "${CMAKE_CURRENT_SOURCE_DIR}/protos")
set(GENERATED_PROTOBUF_PATH "${CMAKE_CURRENT_BINARY_DIR}/generated")
file(MAKE_DIRECTORY ${GENERATED_PROTOBUF_PATH})

# Generated grpc and proto sources
set(COMPANY_PROTO "${PROTO_PATH}/company.proto")
set(COMPANY_PROTO_HEADER "${GENERATED_PROTOBUF_PATH}/company.pb.h")
set(COMPANY_PROTO_SRC "${GENERATED_PROTOBUF_PATH}/company.pb.cc")
set(COMPANY_GRPC_HEADER "${GENERATED_PROTOBUF_PATH}/company.grpc.pb.h")
set(COMPANY_GRPC_SRC "${GENERATED_PROTOBUF_PATH}/company.grpc.pb.cc")

set(ENV{PKG_CONFIG_PATH} C:/develop/vcpkg-2020.04/installed/x64-windows/lib/pkgconfig)
find_package(pkgconfig)
pkg_search_module(GRPCPP_UNSECURE REQUIRED grpc++_unsecure)
pkg_search_module(LIBCARES REQUIRED libcares)

MESSAGE(STATUS ${GRPCPP_LIBRARIES})
MESSAGE(STATUS ${GRPCPP_INCLUDE_DIRS})

add_custom_command(
    OUTPUT "${COMPANY_PROTO_SRC}"
           "${COMPANY_PROTO_HEADER}"
    COMMAND "C:/develop/vcpkg-2020.04/installed/x64-windows/tools/protobuf/protoc.exe"
    ARGS -I "${PROTO_PATH}"
         --cpp_out="${GENERATED_PROTOBUF_PATH}"
         "${COMPANY_PROTO}"
    DEPENDS "${COMPANY_PROTO}"
)

add_custom_command(
    OUTPUT "${COMPANY_GRPC_SRC}"
           "${COMPANY_GRPC_HEADER}"
    COMMAND "C:/develop/vcpkg-2020.04/installed/x64-windows/tools/protobuf/protoc.exe"
    ARGS --grpc_out "${GENERATED_PROTOBUF_PATH}" -I "${PROTO_PATH}"
         --plugin=protoc-gen-grpc="C:/develop/vcpkg-2020.04/installed/x64-windows/tools/grpc/grpc_cpp_plugin.exe"
         "${COMPANY_PROTO}"
    DEPENDS "${COMPANY_PROTO}"
)
include_directories(C:/develop/vcpkg-2020.04/installed/x64-windows/include)
link_directories(C:/develop/vcpkg-2020.04/installed/x64-windows/lib)
# Generated include directory
include_directories("${GENERATED_PROTOBUF_PATH}") 
add_definitions(-D_WIN32_WINNT=0x600 /std:c++17 )

foreach(_target
  company_server company_client)
  add_executable(${_target} "${_target}.cc"
    ${COMPANY_PROTO_SRC}
    ${COMPANY_GRPC_SRC})
  target_link_libraries(${_target}
  ${GRPCPP_UNSECURE_LIBRARIES}
  ${LIBCARES_LIBRARIES}
  upb
  ws2_32
  zlib
  address_sorting
  libprotobuf)
endforeach()
