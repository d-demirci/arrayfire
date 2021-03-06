FIND_PACKAGE(OpenCL REQUIRED)

ADD_DEFINITIONS(-DCL_USE_DEPRECATED_OPENCL_1_2_APIS)

IF(NOT USE_SYSTEM_CL2HPP)
    INCLUDE(build_cl2hpp)
ENDIF(NOT USE_SYSTEM_CL2HPP)
INCLUDE(CLKernelToH)

IF(USE_OPENCL_F77_BLAS)
    ADD_DEFINITIONS(-DUSE_F77_BLAS)
ENDIF()

IF(APPLE)
    FIND_PACKAGE(LAPACKE QUIET) # For finding MKL
    IF(NOT LAPACK_FOUND)
        # UNSET THE VARIABLES FROM LAPACKE
        UNSET(LAPACKE_LIB CACHE)
        UNSET(LAPACK_LIB CACHE)
        UNSET(LAPACKE_INCLUDES CACHE)
        UNSET(LAPACKE_ROOT_DIR CACHE)
        FIND_PACKAGE(LAPACK)
    ENDIF()
ELSE(APPLE) # Linux and Windows
    FIND_PACKAGE(LAPACKE)
ENDIF(APPLE)

IF(NOT LAPACK_FOUND)
    MESSAGE(WARNING "LAPACK not found. Functionality will be disabled")
ELSE(NOT LAPACK_FOUND)
  ADD_DEFINITIONS(-DWITH_OPENCL_LINEAR_ALGEBRA)

  IF(NOT USE_OPENCL_MKL)
    FIND_PACKAGE(CBLAS REQUIRED)

    IF(USE_CPU_F77_BLAS)
      ADD_DEFINITIONS(-DUSE_F77_BLAS)
    ENDIF()

    IF (NOT CBLAS_LIBRARIES)
      MESSAGE(SEND_ERROR "CBLAS Library not set")
    ENDIF()
  ENDIF()
ENDIF()

IF(USE_OPENCL_MKL) # Manual MKL Setup
    MESSAGE("OpenCL Backend Using MKL")
    ADD_DEFINITIONS(-DUSE_MKL)
ELSE(USE_OPENCL_MKL)
    IF(${MKL_FOUND}) # Automatic MKL Setup from BLAS
        MESSAGE("OpenCL Backend Using MKL RT")
        ADD_DEFINITIONS(-DUSE_MKL)
    ENDIF()
ENDIF()

IF(NOT UNIX)
    ADD_DEFINITIONS(-DAFDLL)
ENDIF()

ADD_DEFINITIONS(-DAF_OPENCL
                -D__CL_ENABLE_EXCEPTIONS)

OPTION(USE_SYSTEM_CLBLAS "Use system clBLAS" OFF)
IF(USE_SYSTEM_CLBLAS)
    FIND_PACKAGE(clBLAS REQUIRED)
ELSE()
    INCLUDE(build_clBLAS)
ENDIF()
INCLUDE_DIRECTORIES(${CLBLAS_INCLUDE_DIRS})
LINK_DIRECTORIES(${CLBLAS_LIBRARY_DIR})

OPTION(USE_SYSTEM_CLFFT "Use system clFFT" OFF)
IF(USE_SYSTEM_CLFFT)
    FIND_PACKAGE(clFFT REQUIRED)
ELSE()
    INCLUDE(build_clFFT)
ENDIF()
INCLUDE_DIRECTORIES(${CLFFT_INCLUDE_DIRS})
LINK_DIRECTORIES(${CLFFT_LIBRARY_DIR})

ADD_DEFINITIONS( -DBOOST_ALL_NO_LIB )
SET(Boost_USE_STATIC_LIBS OFF)
FIND_PACKAGE(Boost 1.48 REQUIRED)

OPTION(USE_SYSTEM_BOOST_COMPUTE "Use system BoostCompute" OFF)
IF(USE_SYSTEM_BOOST_COMPUTE)
    IF(Boost_VERSION VERSION_LESS "1.61")
        FIND_PACKAGE(BoostCompute REQUIRED)
    ENDIF()
ELSE()
    INCLUDE(build_boost_compute)
ENDIF()

SET( cl_kernel_headers
    "kernel_headers")

INCLUDE_DIRECTORIES(
    ${CMAKE_INCLUDE_PATH}
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${OpenCL_INCLUDE_DIRS}
    ${CL2HPP_INCLUDE_DIRECTORY}
    "${CMAKE_CURRENT_BINARY_DIR}"
    ${CLBLAS_INCLUDE_DIRS}
    ${CLFFT_INCLUDE_DIRS}
    ${Boost_INCLUDE_DIR}
    ${BoostCompute_INCLUDE_DIRS}
    ${CBLAS_INCLUDE_DIR}
    )
IF(LAPACK_FOUND)
    INCLUDE_DIRECTORIES(${LAPACK_INCLUDE_DIR})
ENDIF()

FILE(GLOB opencl_headers
  "*.hpp"
  "*.h")

FILE(GLOB opencl_sources
    "*.cpp")

FILE(GLOB jit_sources
    "jit/*.hpp")

FILE(GLOB kernel_headers
    "kernel/*.hpp")

FILE(GLOB opencl_kernels
    "kernel/*.cl")

FILE(GLOB kernel_sources
     "kernel/*.cpp")

FILE(GLOB conv_ker_headers
    "kernel/convolve/*.hpp")

FILE(GLOB conv_ker_sources
     "kernel/convolve/*.cpp")

FILE(GLOB cpu_headers
     "cpu/*.hpp")

FILE(GLOB cpu_sources
     "cpu/*.cpp")

LIST(SORT opencl_headers)
LIST(SORT opencl_sources)
LIST(SORT jit_sources)
LIST(SORT kernel_headers)
LIST(SORT opencl_kernels)
LIST(SORT kernel_sources)
LIST(SORT conv_ker_headers)
LIST(SORT conv_ker_sources)
LIST(SORT cpu_headers)
LIST(SORT cpu_sources)

source_group(backend\\opencl\\Headers FILES ${opencl_headers})
source_group(backend\\opencl\\Sources FILES ${opencl_sources})
source_group(backend\\opencl\\JIT FILES ${jit_sources})
source_group(backend\\opencl\\kernel\\Headers FILES ${kernel_headers})
source_group(backend\\opencl\\kernel\\cl FILES ${opencl_kernels})
source_group(backend\\opencl\\kernel\\Sources FILES ${kernel_sources})
source_group(backend\\opencl\\kernel\\convolve\\Headers FILES ${conv_ker_headers})
source_group(backend\\opencl\\kernel\\convolve\\Sources FILES ${conv_ker_sources})
source_group(backend\\opencl\\cpu\\Headers FILES ${cpu_headers})
source_group(backend\\opencl\\cpu\\Sources FILES ${cpu_sources})

IF(LAPACK_FOUND)
    FILE(GLOB magma_sources
        "magma/*.cpp")

    FILE(GLOB magma_headers
        "magma/*.h")

    LIST(SORT magma_headers)
    LIST(SORT magma_sources)

    source_group(backend\\opencl\\magma\\Sources FILES ${magma_sources})
    source_group(backend\\opencl\\magma\\Headers FILES ${magma_headers})
ELSE()
    SET(magma_sources)
    SET(magma_headers)
ENDIF()

FILE(GLOB backend_headers
    "../*.hpp"
    "../*.h"
    )

FILE(GLOB backend_sources
    "../*.cpp"
    )

LIST(SORT backend_headers)
LIST(SORT backend_sources)

source_group(backend\\Headers FILES ${backend_headers})
source_group(backend\\Sources FILES ${backend_sources})

FILE(GLOB c_headers
    "../../api/c/*.hpp"
    "../../api/c/*.h"
    )

FILE(GLOB c_sources
    "../../api/c/*.cpp"
    )

LIST(SORT c_headers)
LIST(SORT c_sources)

source_group(api\\c\\Headers FILES ${c_headers})
source_group(api\\c\\Sources FILES ${c_sources})

FILE(GLOB cpp_sources
    "../../api/cpp/*.cpp"
    )

LIST(SORT cpp_sources)

source_group(api\\cpp\\Sources FILES ${cpp_sources})

FILE(GLOB kernel_src ${opencl_kernels} "kernel/KParam.hpp")

LIST(SORT kernel_src)

CL_KERNEL_TO_H(
    SOURCES ${kernel_src}
    VARNAME kernel_files
    EXTENSION "hpp"
    OUTPUT_DIR ${cl_kernel_headers}
    TARGETS cl_kernel_targets
    NAMESPACE "opencl"
    )

# OS Definitions
IF(UNIX)
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC -pthread -Wno-comment")
    # GCC 6.0 and above enable -Wignored-attributes by default causing a lot of warnings
    # Disable the trigger for gcc >= 6.0.0
    IF("${CMAKE_C_COMPILER_ID}" STREQUAL "GNU" AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER "6.0.0")
        ADD_DEFINITIONS(-Wno-ignored-attributes)
    ENDIF()
ENDIF()

INCLUDE("${CMAKE_CURRENT_SOURCE_DIR}/kernel/sort_by_key/CMakeLists.txt")

INCLUDE("${CMAKE_CURRENT_SOURCE_DIR}/kernel/scan_by_key/CMakeLists.txt")
IF(DEFINED BLAS_SYM_FILE)

    ADD_LIBRARY(afopencl_static STATIC
                ${opencl_headers}
                ${opencl_sources}
                ${jit_sources}
                ${kernel_headers}
                ${opencl_kernels}
                ${kernel_sources}
                ${conv_ker_headers}
                ${conv_ker_sources}
                ${cpu_headers}
                ${cpu_sources}
                ${backend_headers}
                ${backend_sources}
                ${magma_sources}
                ${magma_headers}
                ${SORT_BY_KEY_OBJECTS}
                ${SCAN_BY_KEY_OBJECTS})

    ADD_LIBRARY(afopencl SHARED
               ${c_headers}
               ${c_sources}
               ${cpp_sources})


    IF(FORGE_FOUND AND NOT USE_SYSTEM_FORGE)
        ADD_DEPENDENCIES(afopencl_static forge)
    ENDIF()

    IF(APPLE)
    SET_TARGET_PROPERTIES(afopencl_static
        PROPERTIES LINK_FLAGS -Wl,-exported_symbols_list,${BLAS_SYM_FILE})
    TARGET_LINK_LIBRARIES(afopencl PUBLIC $<TARGET_FILE:afopencl_static>)
    ELSE(APPLE)
    add_custom_command(OUTPUT ${PROJECT_BINARY_DIR}/afopencl_static.renamed
      COMMAND objcopy --redefine-syms ${BLAS_SYM_FILE} $<TARGET_FILE:afopencl_static> ${PROJECT_BINARY_DIR}/afopencl_static.renamed
      DEPENDS $<TARGET_FILE:afopencl_static>)
      TARGET_LINK_LIBRARIES(afopencl PUBLIC ${PROJECT_BINARY_DIR}/afopencl_static.renamed)
    ENDIF(APPLE)


ELSE(DEFINED BLAS_SYM_FILE)

    ADD_LIBRARY(afopencl SHARED
                ${opencl_headers}
                ${opencl_sources}
                ${jit_sources}
                ${kernel_headers}
                ${opencl_kernels}
                ${kernel_sources}
                ${conv_ker_headers}
                ${conv_ker_sources}
                ${cpu_sources}
                ${cpu_sources}
                ${backend_headers}
                ${backend_sources}
                ${c_headers}
                ${c_sources}
                ${cpp_sources}
                ${magma_sources}
                ${magma_headers}
                ${SORT_BY_KEY_OBJECTS}
                ${SCAN_BY_KEY_OBJECTS})

ENDIF()

IF(NOT USE_SYSTEM_CL2HPP)
    ADD_DEPENDENCIES(afopencl cl2hpp)
ENDIF(NOT USE_SYSTEM_CL2HPP)

ADD_DEPENDENCIES(afopencl ${cl_kernel_targets})

TARGET_LINK_LIBRARIES(afopencl
                                PRIVATE ${OpenCL_LIBRARIES}
                                PRIVATE ${CLBLAS_LIBRARIES}
                                PRIVATE ${CLFFT_LIBRARIES}
                                PRIVATE ${CMAKE_DL_LIBS}
                                PRIVATE ${FreeImage_LIBS}
                     )

IF(LAPACK_FOUND)
    TARGET_LINK_LIBRARIES(afopencl
                                PRIVATE ${LAPACK_LIBRARIES}
                                PRIVATE ${CBLAS_LIBRARIES})
ENDIF(LAPACK_FOUND)

LIST(LENGTH GRAPHICS_DEPENDENCIES GRAPHICS_DEPENDENCIES_LEN)
IF(${GRAPHICS_DEPENDENCIES_LEN} GREATER 0)
    ADD_DEPENDENCIES(afopencl ${GRAPHICS_DEPENDENCIES})
ENDIF(${GRAPHICS_DEPENDENCIES_LEN} GREATER 0)

IF(FORGE_FOUND)
    TARGET_LINK_LIBRARIES(afopencl
                                PRIVATE ${GRAPHICS_LIBRARIES})
ENDIF(FORGE_FOUND)

SET_TARGET_PROPERTIES(afopencl PROPERTIES
    VERSION "${AF_VERSION}"
    SOVERSION "${AF_VERSION_MAJOR}")

INSTALL(TARGETS afopencl EXPORT OpenCL DESTINATION "${AF_INSTALL_LIB_DIR}"
        COMPONENT libraries)

IF(APPLE)
    INSTALL(SCRIPT "${PROJECT_SOURCE_DIR}/CMakeModules/osx_install/InstallTool.cmake")
ENDIF(APPLE)

export(TARGETS afopencl FILE ArrayFireOpenCL.cmake)
INSTALL(EXPORT OpenCL DESTINATION "${AF_INSTALL_CMAKE_DIR}"
    COMPONENT cmake
    FILE ArrayFireOpenCL.cmake)
