cmake_minimum_required(VERSION 3.16)
project(mosh)

add_definitions(-std=c++11 -DHAVE_CONFIG_H -D_DEFAULT_SOURCE
                -D_XOPEN_SOURCE=600 -D_FORTIFY_SOURCE=2)
set(CMAKE_EXE_LINKER_FLAGS "-static -no-pie")

option(BUILD_SHARED_LIBS "Build shared lib" OFF)

find_package(Protobuf REQUIRED)
include_directories(${Protobuf_INCLUDE_DIRS})
include_directories(${CMAKE_CURRENT_BINARY_DIR})

file(GLOB PROTO_FILES "src/protobufs/*.proto")
foreach(PROTO_FILE in ${PROTO_FILES})
  string(REGEX REPLACE "[.]proto$" ".pb.cc" OUTPUT_SOURCE ${PROTO_FILE})
  list(APPEND OUTPUT_SOURCES ${OUTPUT_SOURCE})
endforeach()
# message(STATUS "proto files:" ${PROTO_FILES}) message(STATUS "output files:"
# ${OUTPUT_SOURCES})
set(PROTOBUF_INPUT_DIRECTORY "${PROJECT_SOURCE_DIR}/src/protobufs")
add_custom_command(
  OUTPUT ${OUTPUT_SOURCES}
  COMMAND protoc --cpp_out=src/protobufs -I${PROTOBUF_INPUT_DIRECTORY}
          ${PROTO_FILES}
  DEPENDS ${PROTO_FILES}
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
  COMMENT "Calling protoc to generate file")
file(GLOB_RECURSE MOSH_CRYPTO_SRC "src/crypto/*.cc")
file(GLOB_RECURSE MOSH_UTILS_SRC "src/util/*.cc")
file(GLOB_RECURSE MOSH_TERMINAL_SRC "src/terminal/*.cc")
file(GLOB_RECURSE MOSH_NETWORK_SRC "src/network/*.cc")
file(GLOB_RECURSE MOSH_STATESYNC_SRC "src/statesync/*.cc")

include_directories(
  src/crypto
  src/frontend
  src/include
  src/network
  src/protobufs
  src/statesync
  src/terminal
  src/util)

add_library(moshproto ${OUTPUT_SOURCES})
add_library(moshcrypto ${MOSH_CRYPTO_SRC})
add_library(moshutil ${MOSH_UTILS_SRC})
add_library(moshterminal ${MOSH_TERMINAL_SRC})
add_library(moshnetwork ${MOSH_NETWORK_SRC})
add_library(moshstatesync ${MOSH_STATESYNC_SRC})

link_libraries(
  moshcrypto
  moshnetwork
  moshstatesync
  moshterminal
  moshutil
  moshproto
  protobuf
  pthread
  ssl
  crypto
  util
  z
  util
  tinfo
  m)

add_executable(
  mosh-server src/frontend/mosh-server.cc src/frontend/stmclient.cc
              src/frontend/terminaloverlay.cc)
add_executable(
  mosh-client src/frontend/mosh-client.cc src/frontend/stmclient.cc
              src/frontend/terminaloverlay.cc)
