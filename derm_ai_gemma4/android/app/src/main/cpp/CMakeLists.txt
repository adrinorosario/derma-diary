cmake_minimum_required(VERSION 3.22.1)
project(litert_bridge)

add_library(litert_bridge SHARED litert_bridge.cpp)

find_library(log-lib log)

target_link_libraries(litert_bridge ${log-lib})
