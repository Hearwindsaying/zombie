cmake_minimum_required(VERSION 3.15...3.27)
project(zombie)

option(ZOMBIE_BUILD_DEMO "Build zombie demo" ON)

################################################################################
# submodule check
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to 'Release' as none was specified.")
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release"
    "MinSizeRel" "RelWithDebInfo")
endif()

if(CMAKE_BUILD_TYPE MATCHES RELEASE)
    add_definitions(-DNDEBUG)
endif()

################################################################################
# add dependencies
get_directory_property(hasParent PARENT_DIRECTORY)

set(TBB_BUILD_SHARED          ON  CACHE BOOL " " FORCE)
set(TBB_BUILD_STATIC          OFF CACHE BOOL " " FORCE)
set(TBB_BUILD_TESTS           OFF CACHE BOOL " " FORCE)
set(TBB_BUILD_TBBMALLOC       OFF CACHE BOOL " " FORCE)
set(TBB_BUILD_TBBMALLOC_PROXY OFF CACHE BOOL " " FORCE)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/deps/tbb)
set_property(TARGET tbb tbb_def_files PROPERTY FOLDER "dependencies")

if(hasParent)
    set(ZOMBIE_TBB_INCLUDES ${CMAKE_CURRENT_SOURCE_DIR}/deps/tbb/include PARENT_SCOPE)
    set(ZOMBIE_NANOFLANN_INCLUDES ${CMAKE_CURRENT_SOURCE_DIR}/deps/nanoflann/include PARENT_SCOPE)
else()
    set(ZOMBIE_TBB_INCLUDES ${CMAKE_CURRENT_SOURCE_DIR}/deps/tbb/include)
    set(ZOMBIE_NANOFLANN_INCLUDES ${CMAKE_CURRENT_SOURCE_DIR}/deps/nanoflann/include)
endif()

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/deps/fcpw)

list(APPEND ZOMBIE_DEPS_INCLUDES ${CMAKE_CURRENT_SOURCE_DIR}/deps/bessel)
list(APPEND ZOMBIE_DEPS_INCLUDES ${CMAKE_CURRENT_SOURCE_DIR}/deps/pcg32)
list(APPEND ZOMBIE_DEPS_INCLUDES ${ZOMBIE_TBB_INCLUDES})
list(APPEND ZOMBIE_DEPS_INCLUDES ${ZOMBIE_NANOFLANN_INCLUDES})
list(APPEND ZOMBIE_DEPS_INCLUDES ${FCPW_EIGEN_INCLUDES})
list(APPEND ZOMBIE_DEPS_INCLUDES ${FCPW_ENOKI_INCLUDES})

################################################################################
# build library
set(INCLUDE_CORE
    include/zombie/core/distributions.h
    include/zombie/core/geometric_queries.h
    include/zombie/core/pde.h
    include/zombie/core/sampling.h
)
set(INCLUDE_POINTESTIMATION
    include/zombie/point_estimation/common.h
    include/zombie/point_estimation/reverse_walk_on_stars.h
    include/zombie/point_estimation/walk_on_spheres.h
    include/zombie/point_estimation/walk_on_stars.h
)
set(INCLUDE_UTILS_ROBIN_BVH
    include/zombie/utils/robin_boundary_bvh/baseline.h
    include/zombie/utils/robin_boundary_bvh/bvh.h
    include/zombie/utils/robin_boundary_bvh/geometry.h
    include/zombie/utils/robin_boundary_bvh/mbvh.h
)
set(INCLUDE_UTILS
    include/zombie/utils/fcpw_boundary_handler.h
    include/zombie/utils/nearest_neighbor_finder.h
    include/zombie/utils/progress.h
)
set(INCLUDE_VARIANCE_REDUCTION
    include/zombie/variance_reduction/boundary_sampler.h
    include/zombie/variance_reduction/boundary_value_caching.h
    include/zombie/variance_reduction/domain_sampler.h
    include/zombie/variance_reduction/reverse_walk_splatter.h
)

add_library(${PROJECT_NAME} 
INTERFACE 
${INCLUDE_CORE}
${INCLUDE_POINTESTIMATION}
${INCLUDE_UTILS_ROBIN_BVH}
${INCLUDE_UTILS}
${INCLUDE_VARIANCE_REDUCTION}
)
target_include_directories(${PROJECT_NAME} INTERFACE $<BUILD_INTERFACE:${${PROJECT_NAME}_SOURCE_DIR}/include> ${ZOMBIE_DEPS_INCLUDES})
target_link_libraries(${PROJECT_NAME} INTERFACE fcpw tbb)
target_compile_features(${PROJECT_NAME} INTERFACE cxx_std_17)
target_compile_definitions(${PROJECT_NAME} 
INTERFACE 
    _USE_MATH_DEFINES
)

source_group("include/core" FILES ${INCLUDE_CORE})
source_group("include/point_estimation" FILES ${INCLUDE_POINTESTIMATION})
source_group("include/utils/robin_boundary_bvh" FILES ${INCLUDE_UTILS_ROBIN_BVH})
source_group("include/utils/" FILES ${INCLUDE_UTILS})
source_group("include/variance_reduction" FILES ${INCLUDE_VARIANCE_REDUCTION})

set_target_properties(${PROJECT_NAME} PROPERTIES
    FOLDER zombie
)

################################################################################
# build demo
if(ZOMBIE_BUILD_DEMO)
    add_subdirectory(demo)
endif()
