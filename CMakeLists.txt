#          Copyright The ViaDuck Project 2016 - 2018.
# Distributed under the Boost Software License, Version 1.0.
#    (See accompanying file LICENSE or copy at
#          http://www.boost.org/LICENSE_1_0.txt)

cmake_minimum_required(VERSION 3.1)
project(mariadbclientpp)

# Threads
find_package(Threads)

option(MARIADDPP_BUILD_TEST "Build tests" ON)
option(MARIADDPP_BUILD_DOCS "Build Doxygen documentation" ON)
option(MARIADDPP_INSTALL_TARGETS "Install headers to target" OFF)
option(MARIADDPP_SEARCH_LIB "Use find library" ON)

# add additional cmake modules
list(INSERT CMAKE_MODULE_PATH 0 "${CMAKE_CURRENT_SOURCE_DIR}/external/cmake-modules")

# c++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# dependencies
if (NOT MARIADDPP_SEARCH_LIB)
	SET(mariadb_LIBRARY "mariadbclient")
else()
	# Use the default find library
	find_library(mariadb_LIBRARY mariadbclient)
	if (NOT mariadb_LIBRARY)

		find_library(mariadb_LIBRARY mysqlclient)
		if (NOT mariadb_LIBRARY)
			message(FATAL_ERROR "Could not find mysqlclient lib")
		endif()
	endif()
endif()

# find files
file(GLOB mariadbclientpp_PUBLIC_HEADERS include/mariadb++/*)
file(GLOB mariadbclientpp_PRIVATE_HEADERS src/*.hpp)
file(GLOB mariadbclientpp_SOURCES src/*.cpp)

# set up target
add_library(mariadbclientpp STATIC ${mariadbclientpp_SOURCES} ${mariadbclientpp_PUBLIC_HEADERS} ${mariadbclientpp_PRIVATE_HEADERS})

# target options
target_link_libraries(mariadbclientpp ${mariadb_LIBRARY} ${CMAKE_THREAD_LIBS_INIT})
target_include_directories(mariadbclientpp PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)

if (NOT MARIADDPP_SEARCH_LIB)
	target_include_directories(mariadbclientpp
		PUBLIC 
			${MARIADB_LIB_INCLUDE_PATH} 
			${MARIADB_LIB_INCLUDE_PATH_BIN}
	)
	
	target_compile_definitions(mariadbclientpp
		PUBLIC
			MARIADBPP_HAS_DIRECT_INCLUDE=1
	)
endif()

# install configuration
if (MARIADDPP_INSTALL_TARGETS)
install(FILES ${mariadbclientpp_PUBLIC_HEADERS} DESTINATION include/mariadb++)
	install(TARGETS mariadbclientpp
		ARCHIVE DESTINATION lib
		RUNTIME DESTINATION bin
		LIBRARY DESTINATION lib
	)
endif()

# tests
if (MARIADDPP_BUILD_TEST)
	add_subdirectory(test)
endif()

# doxygen
if (MARIADDPP_BUILD_DOCS)
	include(Doxygen)
	if (DOXYGEN_FOUND)
		setup_doxygen(mariadbpp_doc ${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in)
	endif()
endif()