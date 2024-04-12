cmake_minimum_required(VERSION 3.18)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Set extension name here
set(TARGET_NAME quack)

# DuckDB's extension distribution supports vcpkg. As such, dependencies can be added in ./vcpkg.json and then
# used in cmake with find_package. Feel free to remove or replace with other dependencies.
# Note that it should also be removed from vcpkg.json to prevent needlessly installing it..
find_package(OpenSSL REQUIRED)

set(EXTENSION_NAME ${TARGET_NAME}_extension)
set(LOADABLE_EXTENSION_NAME ${TARGET_NAME}_loadable_extension)

project(${TARGET_NAME})

set(EXTENSION_SOURCES src/quack_extension.cpp)

if(NOT TARGET duckdb_static)
  add_subdirectory(duckdb)
endif()

get_target_property(DUCKDB_INCLUDES duckdb INCLUDE_DIRECTORIES)

include_directories(
  src/include
  duckdb/src/include
  ${DUCKDB_INCLUDES})

set(SOURCES src/quack_extension.cpp)


add_library(${LOADABLE_EXTENSION_NAME} SHARED ${SOURCES})

target_link_libraries(${LOADABLE_EXTENSION_NAME}
  duckdb
  OpenSSL::SSL OpenSSL::Crypto)


add_library(${EXTENSION_NAME} STATIC ${SOURCES})

target_link_libraries(${EXTENSION_NAME}
  duckdb_static,
  OpenSSL::SSL OpenSSL::Crypto)

install(
  TARGETS ${EXTENSION_NAME}
  EXPORT "${DUCKDB_EXPORT_SET}"
  LIBRARY DESTINATION "${INSTALL_LIB_DIR}"
  ARCHIVE DESTINATION "${INSTALL_LIB_DIR}")
