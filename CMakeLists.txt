cmake_minimum_required (VERSION 3.26)

project (semimap VERSION 1.0.0)

set(CMAKE_CXX_FLAGS_RELEASE -Ofast -march=native)

add_executable(semimap_test test.cpp)

# Set C++ standard for all targets
set_property(TARGET semimap_test PROPERTY CXX_STANDARD 17)
