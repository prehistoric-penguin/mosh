cmake_minimum_required(VERSION 3.16)
project(mosh)

add_definitions (-no-pie -std=c++11 -static -DHAVE_CONFIG_H -D_DEFAULT_SOURCE -D_XOPEN_SOURCE=600 -D_FORTIFY_SOURCE=2 )
set(CMAKE_EXE_LINKER_FLAGS -static)
option(BUILD_SHARED_LIBS "Build static lib" OFF)

file(GLOB_RECURSE GEN_PROTO_SRC "src/protobufs/*.cc")
file(GLOB_RECURSE MOSH_CRYPTO_SRC "src/crypto/*.cc")
file(GLOB_RECURSE MOSH_UTILS_SRC "src/util/*.cc")
file(GLOB_RECURSE MOSH_TERMINAL_SRC "src/terminal/*.cc")
file(GLOB_RECURSE MOSH_NETWORK_SRC "src/network/*.cc")
file(GLOB_RECURSE MOSH_STATESYNC_SRC "src/statesync/*.cc")

include_directories(src/crypto src/frontend src/include src/network src/protobufs src/statesync src/terminal src/util)

add_library(moshproto
	${GEN_PROTO_SRC}
	)


add_library(moshcrypto
	${MOSH_CRYPTO_SRC})

add_library(moshutil
	${MOSH_UTILS_SRC}
	)

add_library(moshterminal
	${MOSH_TERMINAL_SRC}
	)

add_library(moshnetwork
	${MOSH_NETWORK_SRC}
	)

add_library(moshstatesync
	${MOSH_STATESYNC_SRC}
	)

link_libraries(moshcrypto moshnetwork moshstatesync moshterminal moshutil moshproto 
	protobuf pthread ssl crypto util z util tinfo m)

add_executable(mosh-server src/frontend/mosh-server.cc
	src/frontend/stmclient.cc src/frontend/terminaloverlay.cc)
add_executable(mosh-client src/frontend/mosh-client.cc
	src/frontend/stmclient.cc src/frontend/terminaloverlay.cc)
