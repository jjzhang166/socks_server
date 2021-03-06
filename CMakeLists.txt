cmake_minimum_required(VERSION 3.0)

option(USE_HUNTER "Use hunter" OFF)

if (USE_HUNTER)
	include("cmake/HunterGate.cmake")
	HunterGate(
		URL "https://github.com/ruslo/hunter/archive/v0.17.18.tar.gz"
		SHA1 "ffd57842beb628402ea918508718c6f84abe3771"
	)
endif()

project(socks_server)

if (USE_HUNTER)
	hunter_add_package(Boost COMPONENTS system filesystem)
endif()

if (NOT MSVC)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -static-libstdc++ -static-libgcc")
else()
	set(CompilerFlags
			CMAKE_CXX_FLAGS_DEBUG
			CMAKE_CXX_FLAGS_RELEASE
			CMAKE_CXX_FLAGS_MINSIZEREL
			CMAKE_CXX_FLAGS_RELWITHDEBINFO
			CMAKE_C_FLAGS_DEBUG
			CMAKE_C_FLAGS_RELEASE
			CMAKE_C_FLAGS_MINSIZEREL
			CMAKE_C_FLAGS_RELWITHDEBINFO
		)

	foreach(CompilerFlag ${CompilerFlags})
		string(REPLACE "/MD" "/MT" ${CompilerFlag} "${${CompilerFlag}}")
	endforeach()

	add_definitions(-D_CRT_SECURE_NO_WARNINGS)
	add_definitions(-D_WIN32_WINNT=0x0501)

	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")
endif()

set(Boost_USE_STATIC_LIBS ON)
set(Boost_USE_STATIC_RUNTIME ON)

find_package(Boost COMPONENTS system filesystem REQUIRED)
include_directories(${Boost_INCLUDE_DIRS})

add_executable(socks_server
  src/main.cpp
  src/io.hpp
  src/socks_server.hpp
)

if(UNIX)
	target_link_libraries(socks_server ${Boost_LIBRARIES} pthread)
else()
	target_link_libraries(socks_server ${Boost_LIBRARIES})
endif()

install(TARGETS socks_server RUNTIME DESTINATION bin)

