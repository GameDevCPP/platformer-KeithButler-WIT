cmake_minimum_required(VERSION 3.9)
# Compiler flags
if (MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP /EHsc" CACHE INTERNAL "" FORCE)
elseif(APPLE)
    message("hello apple")
    set(CMAKE_OSX_DEPLOYMENT_TARGET 10.15 CACHE STRING "")
endif()
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

project(Platformer_for_wit)

#### setup Directories ####
#Main output directory
set(OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin/")
# Ouput all DLLs from all libs into main build folder
set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${OUTPUT_DIRECTORY})

add_custom_target(copy_resources ALL COMMAND ${CMAKE_COMMAND}
        -E copy_directory
        "${PROJECT_SOURCE_DIR}/res"
        ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/$<CONFIGURATION>/res
)
add_custom_target(copy_resources_rootdir ALL COMMAND ${CMAKE_COMMAND}
        -E copy_directory
        "${PROJECT_SOURCE_DIR}/res"
        ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/res
)

#### Add External Dependencies ####
# SFML

include(FetchContent)
FetchContent_Declare(SFML
        GIT_REPOSITORY https://github.com/SFML/SFML.git
        GIT_TAG 2.6.x)
FetchContent_MakeAvailable(SFML)

FetchContent_Declare(BOX2D
        GIT_REPOSITORY      https://github.com/dooglz/Box2D.git
        GIT_TAG             v2.3.1)
#FetchContent_MakeAvailable(BOX2D)
if(NOT box2d_POPULATED)
    FetchContent_Populate(box2d)
    add_subdirectory(${box2d_SOURCE_DIR}/Box2D ${box2d_BINARY_DIR})
endif()
fetchcontent_makeavailable(BOX2D)




include_directories("lib/b2d/Box2D/")

#### Add Engine and our own library projects####

file(GLOB_RECURSE RESOURCE_FILES "res/*.*")
####
file(GLOB_RECURSE SOURCE_FILES engine/*.cpp engine/*.h)
add_library(lib_engine STATIC ${SOURCE_FILES} ${RESOURCE_FILES})
target_include_directories(lib_engine INTERFACE "${CMAKE_SOURCE_DIR}/engine/" PUBLIC SYSTEM ${B2D_INCS} )
target_link_libraries(lib_engine lib_tile_level_loader lib_maths lib_ecm Box2D sfml-graphics sfml-audio)

## Maths lib
add_library(lib_maths INTERFACE)
target_sources(lib_maths INTERFACE "${CMAKE_SOURCE_DIR}/lib_maths/maths.h")


# Include the directories of lib_maths and SFML
target_include_directories(lib_maths INTERFACE
        "${CMAKE_SOURCE_DIR}/lib_maths"
        ${SFML_SOURCE_DIR}/include
)

## Tile loader lib
file(GLOB_RECURSE SOURCE_FILES lib_tile_level_loader/*.cpp lib_tile_level_loader/*.h)
add_library(lib_tile_level_loader STATIC ${SOURCE_FILES} ${RESOURCE_FILES})
target_include_directories(lib_tile_level_loader INTERFACE "${CMAKE_SOURCE_DIR}/lib_tile_level_loader/" )
target_link_libraries(lib_tile_level_loader lib_maths sfml-graphics)

## ECM lib
file(GLOB_RECURSE SOURCE_FILES lib_ecm/*.cpp lib_ecm/*.h)
add_library(lib_ecm STATIC ${SOURCE_FILES})
target_include_directories(lib_ecm INTERFACE "${CMAKE_SOURCE_DIR}/lib_ecm" )
target_link_libraries(lib_ecm PRIVATE lib_maths)



file(GLOB_RECURSE SOURCES lab_7_platformer/*.cpp practical_6_platformer/*.h)
file(GLOB_RECURSE CMPNTS lab_7_platformer/components/*.cpp practical_6_platformer/components/*.h)
file(GLOB_RECURSE SCENES lab_7_platformer/scenes/*.cpp practical_6_platformer/scenes/*.h)
add_executable(PRACTICAL_7_PLATFORMER ${SOURCES} ${RESOURCE_FILES})

source_group("components" FILES ${CMPNTS})
source_group("resources" FILES ${RESOURCE_FILES})

target_link_libraries(PRACTICAL_7_PLATFORMER PRIVATE lib_engine  sfml-graphics Box2D)
set(EXECUTABLES ${EXECUTABLES} PRACTICAL_7_PLATFORMER)

add_dependencies(PRACTICAL_7_PLATFORMER copy_resources copy_resources_rootdir )


set_target_properties(PRACTICAL_7_PLATFORMER
        PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY
        ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/$(Configuration)
)