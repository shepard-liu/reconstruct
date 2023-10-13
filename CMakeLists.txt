cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

project(reconstruct LANGUAGES CXX)

# configuring PCL
if(NOT DEFINED ENV{PCL_ROOT})
    message(FATAL_ERROR "Pass -DPCL_ROOT or configure PCL_ROOT environment variable to point to the PCL installation directory. This is automatically set if you install by PCL all-in-one installer.")
endif()

find_package(PCL 1.13 REQUIRED COMPONENTS common io visualization)
 
# executable target main
add_executable(main src/main.cpp)

target_include_directories(main PRIVATE ${PCL_INCLUDE_DIRS})

target_link_libraries(main PRIVATE
 PCL_IO_LIBRARIES
 PCL_COMMON_LIBRARIES
 PCL_SURFACE_LIBRARIES) 

set_target_properties(main PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED OFF
    CXX_EXTENSIONS OFF)