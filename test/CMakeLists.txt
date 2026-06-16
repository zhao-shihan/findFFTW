cmake_minimum_required(VERSION 3.10)
project(test_findFFTW NONE)

# Add the parent directory (repo root) to the module path so CMake can find FindFFTW.cmake
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/..")

# Find FFTW. If FFTW_TEST_COMPONENTS is given (semicolon-separated list), those
# components are required; otherwise we just require that FFTW itself is found.
if(DEFINED FFTW_TEST_COMPONENTS)
    find_package(FFTW REQUIRED COMPONENTS ${FFTW_TEST_COMPONENTS})
else()
    find_package(FFTW REQUIRED)
endif()

# ---------------------------------------------------------------------------
# Check that the basic variables are set
# ---------------------------------------------------------------------------
if(NOT FFTW_FOUND)
    message(FATAL_ERROR "FFTW_FOUND is not TRUE after find_package(FFTW REQUIRED)")
endif()

if(NOT FFTW_INCLUDE_DIRS)
    message(FATAL_ERROR "FFTW_INCLUDE_DIRS is empty after find_package(FFTW REQUIRED)")
endif()

if(NOT FFTW_LIBRARIES)
    message(FATAL_ERROR "FFTW_LIBRARIES is empty after find_package(FFTW REQUIRED)")
endif()

# ---------------------------------------------------------------------------
# Map component names to their CMake imported target names
# ---------------------------------------------------------------------------
set(FFTW_DOUBLE_LIB_TARGET          FFTW::Double)
set(FFTW_FLOAT_LIB_TARGET           FFTW::Float)
set(FFTW_LONGDOUBLE_LIB_TARGET      FFTW::LongDouble)
set(FFTW_DOUBLE_THREADS_LIB_TARGET  FFTW::DoubleThreads)
set(FFTW_FLOAT_THREADS_LIB_TARGET   FFTW::FloatThreads)
set(FFTW_LONGDOUBLE_THREADS_LIB_TARGET FFTW::LongDoubleThreads)
set(FFTW_DOUBLE_OPENMP_LIB_TARGET   FFTW::DoubleOpenMP)
set(FFTW_FLOAT_OPENMP_LIB_TARGET    FFTW::FloatOpenMP)
set(FFTW_LONGDOUBLE_OPENMP_LIB_TARGET FFTW::LongDoubleOpenMP)
set(FFTW_DOUBLE_MPI_LIB_TARGET      FFTW::DoubleMPI)
set(FFTW_FLOAT_MPI_LIB_TARGET       FFTW::FloatMPI)
set(FFTW_LONGDOUBLE_MPI_LIB_TARGET  FFTW::LongDoubleMPI)

# ---------------------------------------------------------------------------
# Check per-component variables and targets for each expected component
# ---------------------------------------------------------------------------
foreach(component IN LISTS FFTW_TEST_COMPONENTS)
    if(NOT FFTW_${component}_FOUND)
        message(FATAL_ERROR
            "Expected FFTW component ${component} was not found: "
            "FFTW_${component}_FOUND is not TRUE")
    endif()

    # The library variable name is FFTW_<component>, e.g. FFTW_DOUBLE_LIB
    if(NOT FFTW_${component})
        message(FATAL_ERROR
            "FFTW_${component} is empty for found component ${component}")
    endif()

    set(_target "${FFTW_${component}_TARGET}")
    if(_target AND NOT TARGET ${_target})
        message(FATAL_ERROR
            "CMake imported target ${_target} was not created for component ${component}")
    endif()
endforeach()

# ---------------------------------------------------------------------------
# Informational output (also shows status of all optional components)
# ---------------------------------------------------------------------------
message(STATUS "FindFFTW test passed!")
message(STATUS "  FFTW_FOUND:        ${FFTW_FOUND}")
message(STATUS "  FFTW_INCLUDE_DIRS: ${FFTW_INCLUDE_DIRS}")
message(STATUS "  FFTW_LIBRARIES:    ${FFTW_LIBRARIES}")

set(_all_components
    DOUBLE_LIB FLOAT_LIB LONGDOUBLE_LIB
    DOUBLE_THREADS_LIB FLOAT_THREADS_LIB LONGDOUBLE_THREADS_LIB
    DOUBLE_OPENMP_LIB  FLOAT_OPENMP_LIB  LONGDOUBLE_OPENMP_LIB
    DOUBLE_MPI_LIB     FLOAT_MPI_LIB     LONGDOUBLE_MPI_LIB
)
message(STATUS "  Component status:")
foreach(_comp IN LISTS _all_components)
    message(STATUS "    FFTW_${_comp}_FOUND=${FFTW_${_comp}_FOUND}  lib=${FFTW_${_comp}}")
endforeach()
