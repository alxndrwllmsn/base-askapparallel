cmake_minimum_required (VERSION 3.12.0)
option(UPDATE_CMAKE "Update the cmake modules" ON)
 
if (UPDATE_CMAKE)
# Reuse common cmake infrastructure

if (EXISTS ${CMAKE_SOURCE_DIR}/askap-cmake)
    execute_process(COMMAND git pull 
	WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/askap-cmake)
else ()	
    execute_process(COMMAND git clone https://bitbucket.csiro.au/scm/askapsdp/askap-cmake.git
	WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})
endif ()
endif ()

set (CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/askap-cmake")
include(version_utils)
get_version_string()


project(askap-parallel VERSION ${VERSION_SHORT} LANGUAGES C CXX)

# Required Version of ASKAP dependencies is the MAJOR and MINOR version
# of this package. This allows the PATCH versions to change on everything
# but the version number cannot be less or more than the current version
# 
set(REQUIRED_VERSION "${VERSION_MAJOR}.${VERSION_MINOR}")


configure_file(askap_askapparallel.cc.in ${CMAKE_CURRENT_BINARY_DIR}/askap/askapparallel/askap_askapparallel.cc)

option (CXX11 "Compile as C++11 if possible" YES)
option (ENABLE_SHARED "Build shared libraries" YES)
option (ENABLE_RPATH "Include rpath in executables and shared libraries" YES)
option (ENABLE_OPENMP "Build with OPENMP Support" NO)
# YandaSoft dependencies .. in order
find_package(lofar-blob REQUIRED)
find_package(askap-askap ${REQUIRED_VERSION} REQUIRED)
find_package(log4cxx REQUIRED)
find_package(Casacore REQUIRED COMPONENTS  ms images mirlib coordinates fits lattices measures scimath scimath_f tables casa)
find_package(CPPUnit)
find_package(MPI)

# uninstall target
if(NOT TARGET uninstall)
    configure_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
        "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
        IMMEDIATE @ONLY)

    add_custom_target(uninstall
        COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
endif()

if (ENABLE_OPENMP)
	find_package(OpenMP)
    if (OPENMP_FOUND)
        if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Intel")
            set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -qopenmp")
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -qopenmp")
        else( )
            set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
        endif( )
    else (OPENMP_FOUND)
        message(WARNING "Cannot fullfill USE_OPENMP, compiler does not support it")
    endif (OPENMP_FOUND)
endif()


if (ENABLE_SHARED)
option (BUILD_SHARED_LIBS "" YES)
    if (ENABLE_RPATH)
        # Set RPATH to use for installed targets; append linker search path
        set(CMAKE_INSTALL_NAME_DIR "${CMAKE_INSTALL_PREFIX}/lib" )
        set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
        set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
    endif (ENABLE_RPATH)
endif(ENABLE_SHARED)


if (CASACORE3 OR CXX11)
	set(CMAKE_CXX_STANDARD 11)
	set(CMAKE_CXX_STANDARD_REQUIRED ON)
endif()

set (_modules askapparallel)

# add some more tests and sub-directories

foreach (module ${_modules})
	add_subdirectory(askap/${module})
endforeach (module)


include(yanda_export)
yanda_export(askap)
