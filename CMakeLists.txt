cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

project(reconstruct LANGUAGES CXX)

# -------------------------
# Configure PCL
# -------------------------
if(WIN32 AND NOT DEFINED ENV{PCL_ROOT})
    message(FATAL_ERROR "Pass -DPCL_ROOT or configure PCL_ROOT environment variable to point to the PCL installation directory. This is automatically set if you install by PCL all-in-one installer.")
endif()

list(APPEND PCL_IMPORT_COMPONENTS
    common
    io
    surface
    visualization)

set(PCL_FIND_QUIETLY ON)
find_package(PCL 1.13 QUIET REQUIRED COMPONENTS ${PCL_IMPORT_COMPONENTS})

message(STATUS "PCL_LIBRARIES: ${PCL_LIBRARIES}")
message(STATUS "PCL_LIBRARY_DIRS: ${PCL_LIBRARY_DIRS}")
message(STATUS "PCL_IO_LINK_LIBRARIES: ${PCL_IO_LINK_LIBRARIES}")
message(STATUS "PCL_INCLUDE_DIRS: ${PCL_INCLUDE_DIRS}")
message(STATUS "VTK_LIBRARIES: ${VTK_LIBRARIES}")

# -------------------
# TARGET main
# -------------------
add_executable(main src/main.cpp)

target_include_directories(main PRIVATE
    ${PCL_INCLUDE_DIRS}
    ${PCL_ROOT}/3rdParty/FLANN/include/)

target_link_libraries(main PRIVATE ${PCL_LIBRARIES})

set_target_properties(main PROPERTIES
    OUTPUT_NAME main
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
    CXX_EXTENSIONS OFF)

include(cmake/utils/print_all_props.cmake)

# copy dlls for linking on Windows
if(WIN32)
    foreach(_dep_target IN LISTS PCL_LIBRARIES)
        message(STATUS "PCL dependency target: ${_dep_target}")

        if(${_dep_target} MATCHES "^pcl_.*$")
            list(APPEND TARGET_MAIN_RUNTIME_DLLS "${PCL_ROOT}/bin/$<TARGET_FILE_BASE_NAME:${_dep_target}>$<IF:$<CONFIG:Debug>,d, >.dll")
        elseif(${_dep_target} MATCHES "^(VTK|vtk).*$")
            list(APPEND TARGET_MAIN_RUNTIME_DLLS "${PCL_ROOT}/3rdParty/VTK/bin/$<TARGET_FILE_BASE_NAME:${_dep_target}>$<IF:$<CONFIG:Debug>,d, >.dll")
        elseif(${_dep_target} MATCHES "^(FLANN|[fF]lann).*$")
            list(APPEND TARGET_MAIN_RUNTIME_DLLS "${PCL_ROOT}/3rdParty/FLANN/bin/$<TARGET_FILE_BASE_NAME:${_dep_target}>$<IF:$<CONFIG:Debug>,d, >.dll")
        elseif(${_dep_target} MATCHES "^(QHULL|[qQ]hull).*$")
            list(APPEND TARGET_MAIN_RUNTIME_DLLS "${PCL_ROOT}/3rdParty/Qhull/bin/$<TARGET_FILE_BASE_NAME:${_dep_target}>$<IF:$<CONFIG:Debug>,d, >.dll")
        elseif(${_dep_target} MATCHES "^(BOOST|[bB]oost).*$")
        # do nothing bcz boost is linked statically by PCL
        elseif(${_dep_target} MATCHES "(OPENNI2|[oO]pen(ni|NI)2)")
        # do nothing bcz OPENNI2 is linked statically by PCL
        else()
            message(FATAL_ERROR "Unknown dependency target of PCL: ${_dep_target}")
        endif()
    endforeach()

    add_custom_command(TARGET main POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        ${TARGET_MAIN_RUNTIME_DLLS}
        $<TARGET_FILE_DIR:main>)
endif()