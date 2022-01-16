cmake_minimum_required(VERSION 3.5.1)

project(lanelet2)

set(CMAKE_CXX_STANDARD 17)

# by setting this, we don't need to install the generated c++ library to /usr/local
# LINKED_LIBRARIES will be installed by side with the python api binding libraries
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-rpath='$ORIGIN'")

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
find_package(Boost REQUIRED COMPONENTS python serialization system filesystem)
find_package(PythonLibs 3.5 REQUIRED) # sets ${PYTHON_INCLUDE_DIRS}
find_package(Eigen3 QUIET)
if(NOT EIGEN3_FOUND)
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(EIGEN3 REQUIRED eigen3)
    set(EIGEN3_INCLUDE_DIR ${EIGEN3_INCLUDE_DIRS})
endif()
find_package(GeographicLib REQUIRED)
find_package(pugixml REQUIRED)

include_directories(
        ${Boost_INCLUDE_DIRS}
        ${PYTHON_INCLUDE_DIRS}
        ${EIGEN3_INCLUDE_DIR}
        ${GeographicLib_INCLUDE_DIRS}
        ${PugiXML_INCLUDE_DIRS})

include_directories(
        lanelet2_core/include
        lanelet2_io/include
        lanelet2_projection/include
        lanelet2_traffic_rules/include
        lanelet2_routing/include
        lanelet2_validation/include
        lanelet2_matching/include
        lanelet2_python/include)

file(GLOB CORESRCS lanelet2_core/src/*.cpp)
add_library(lanelet2_core SHARED ${CORESRCS})
target_link_libraries(lanelet2_core
        ${Boost_LIBRARIES}
        ${PYTHON_LIBRARY}
        ${GeographicLib_LIBRARIES}
        ${PugiXML_LIBRARIES}
        )

file(GLOB IOSRCS lanelet2_io/src/*.cpp)
add_library(lanelet2_io SHARED ${IOSRCS})
target_link_libraries(lanelet2_io
        ${Boost_LIBRARIES}
        ${PYTHON_LIBRARY}
        ${GeographicLib_LIBRARIES}
        ${PugiXML_LIBRARIES}
        lanelet2_core)

file(GLOB PRJSRCS lanelet2_projection/src/*.cpp)
add_library(lanelet2_projection SHARED ${PRJSRCS})
target_link_libraries(lanelet2_projection
        ${Boost_LIBRARIES}
        ${PYTHON_LIBRARY}
        ${GeographicLib_LIBRARIES}
        ${PugiXML_LIBRARIES}
        lanelet2_core)

file(GLOB RULESSRCS lanelet2_traffic_rules/src/*.cpp)
add_library(lanelet2_traffic_rules SHARED ${RULESSRCS})
target_link_libraries(lanelet2_traffic_rules
        ${Boost_LIBRARIES}
        ${PYTHON_LIBRARY}
        ${GeographicLib_LIBRARIES}
        ${PugiXML_LIBRARIES}
        lanelet2_core)

file(GLOB ROUTESRCS lanelet2_routing/src/*.cpp)
add_library(lanelet2_routing SHARED ${ROUTESRCS})
target_link_libraries(lanelet2_routing
        ${Boost_LIBRARIES}
        ${PYTHON_LIBRARY}
        ${GeographicLib_LIBRARIES}
        ${PugiXML_LIBRARIES}
        lanelet2_core
        lanelet2_traffic_rules)

file(GLOB VALISRCS lanelet2_validation/src/*.cpp)
add_library(lanelet2_validation SHARED ${VALISRCS})
target_link_libraries(lanelet2_validation
        ${Boost_LIBRARIES}
        ${PYTHON_LIBRARY}
        ${GeographicLib_LIBRARIES}
        ${PugiXML_LIBRARIES}
        lanelet2_core lanelet2_io lanelet2_routing lanelet2_traffic_rules lanelet2_projection)


file(GLOB MATCHSRCS lanelet2_matching/src/*.cpp)
add_library(lanelet2_matching SHARED ${MATCHSRCS})
target_link_libraries(lanelet2_matching
        ${Boost_LIBRARIES}
        ${PYTHON_LIBRARY}
        ${GeographicLib_LIBRARIES}
        ${PugiXML_LIBRARIES})
target_include_directories(lanelet2_matching PRIVATE lanelet2_matching/include/lanelet2_matching)


## build each python api binding submodule
set(MODUELS core geometry io matching projection traffic_rules routing)
foreach(SUBMODULE ${MODUELS})
    set(API_LIBRARY_NAME "${PROJECT_NAME}_${SUBMODULE}_pyapi")
    add_library(${API_LIBRARY_NAME} SHARED
            lanelet2_python/python_api/${SUBMODULE}.cpp  # binding codes
            )
    target_compile_definitions(${API_LIBRARY_NAME} PRIVATE -DPYTHON_API_MODULE_NAME=lib${API_LIBRARY_NAME})
    target_link_libraries(${API_LIBRARY_NAME}
            ${Boost_LIBRARIES}
            ${PYTHON_LIBRARY}
            lanelet2_matching
            lanelet2_projection
            lanelet2_validation
            lanelet2_routing
            lanelet2_traffic_rules
            lanelet2_io
            lanelet2_core
            )
endforeach()