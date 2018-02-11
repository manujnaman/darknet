cmake_minimum_required(VERSION 2.8)
project(darknet C CXX)

add_definitions(-DGPU)
add_definitions(-DCUDNN)
add_definitions(-DOPENCV)


# Double find_package is being done to remove the error "/usr/bin/ld: cannot find -lopencv_dep_cudart"
find_package(CUDA REQUIRED)
unset(CUDA_USE_STATIC_CUDA_RUNTIME CACHE)
option(CUDA_USE_STATIC_CUDA_RUNTIME OFF)
find_package(CUDA)

find_package(OpenCV 2.4.13 REQUIRED)
find_package(Threads REQUIRED)

include_directories(${CUDA_INCLUDE_DIRS})
include_directories(${OpenCV_INCLUDE_DIR})
include_directories(src)
include_directories(include)
#set(SRC_DIR src)


file(GLOB SRC_FILES ${PROJECT_SOURCE_DIR}/src/*.c ${PROJECT_SOURCE_DIR}/examples/*.c)
list(REMOVE_ITEM SRC_FILES "${PROJECT_SOURCE_DIR}/src/compare.c")
list(REMOVE_ITEM SRC_FILES "${PROJECT_SOURCE_DIR}/examples/voxel.c")
list(REMOVE_ITEM SRC_FILES "${PROJECT_SOURCE_DIR}/examples/rnn_vid.c")
list(REMOVE_ITEM SRC_FILES "${PROJECT_SOURCE_DIR}/examples/swag.c")
list(REMOVE_ITEM SRC_FILES "${PROJECT_SOURCE_DIR}/examples/writing.c")
list(REMOVE_ITEM SRC_FILES "${PROJECT_SOURCE_DIR}/examples/dice.c")

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR})

#include(CheckCXXCompilerFlag)
#CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
#CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
#if(COMPILER_SUPPORTS_CXX11)
#    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
#    set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -std=c++11")
#elseif(COMPILER_SUPPORTS_CXX0X)
#    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
#    set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -std=c++0x")
#else()
#        message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support.")
#endif()

## Compiler settings
#if(CMAKE_COMPILER_IS_GNUCXX)
#    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wl,--export-dynamic -Wall -Wno-sign-compare -fPIC")
#endif()

set(CMAKE_CXX_FLAGS "-O3 -Wall -Wno-unknown-pragmas -std=c++11")


#thread
#FIND_PACKAGE(Threads)
list(APPEND LIBRARIES ${CMAKE_THREAD_LIBS_INIT})

# CUDA
#FIND_PACKAGE(CUDA)
#if (CUDA_FOUND)
#    message(STATUS "CUDA Version: " ${CUDA_VERSION_STRINGS})
#    message(STATUS "CUDA Libararies: " ${CUDA_LIBRARIES})
#    include_directories(SYSTEM ${CUDA_INCLUDE_DIRS})
    list(APPEND LIBRARIES ${CUDA_LIBRARIES} ${CUDA_CUBLAS_LIBRARIES} ${CUDA_curand_LIBRARY} ${CUDA_cusparse_LIBRARY})
#    list(APPEND CUDA_NVCC_FLAGS "-arch=sm_35;-O2;-Xcompiler;-fPIC;")
    set(CUDA_PROPAGATE_HOST_FLAGS OFF)
#    cuda_include_directories(src)
#else()
#    list(APPEND LIBRARIES "m")
#endif()

#BOOST
#FIND_PACKAGE(Boost REQUIRED python)
#FIND_PACKAGE(PythonLibs REQUIRED)

#include_directories(SYSTEM ${Boost_INCLUDE_DIR})
#include_directories(SYSTEM ${PYTHON_INCLUDE_DIR})
#include_directories(SYSTEM ${OpenCV_INCLUDE_DIR})

#include_directories(src)
#set(SRC_DIR src)

#FILE(GLOB SRC_FILES ${PROJECT_SOURCE_DIR}/${SRC_DIR}/*.c)
#LIST(REMOVE_ITEM SRC_FILES "${PROJECT_SOURCE_DIR}/src/server.c")
#LIST(REMOVE_ITEM SRC_FILES "${PROJECT_SOURCE_DIR}/src/old.c")
#LIST(REMOVE_ITEM SRC_FILES "${PROJECT_SOURCE_DIR}/src/cpu_gemm.c")
##LIST(REMOVE_ITEM SRC_FILES "${PROJECT_SOURCE_DIR}/src/darknet.c")

#if (CUDA_FOUND)

FILE(GLOB CU_SRC_FILES ${PROJECT_SOURCE_DIR}/src/*.cu)
    
CUDA_INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR}/src)
CUDA_INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR}/include)

set(CUDA_ARCH_BIN "61" CACHE STRING "Specify 'real' GPU arch to build binaries for, BIN(PTX) format is supported. Example: 1.3 2.1(1.3) or 13 21(13)")
set(CUDA_ARCH_PTX "" CACHE STRING "Specify 'virtual' PTX arch to build PTX intermediate code for. Example: 1.0 1.2 or 10 12")              

include(CudaComputeTargetFlags)
APPEND_TARGET_ARCH_FLAGS()

#set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS}  "-Xcompiler;-fPIC;")           
#set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "--ftz=true;--prec-div=false;--prec-sqrt=false") 
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS}  "-D_FORCE_INLINES;-D_MWAITXINTRIN_H_INCLUDED;-Xcompiler;-fPIC;-std=c++11;--default-stream per-thread")
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "--ftz=true;--prec-div=false;--prec-sqrt=false") 

CUDA_COMPILE(cuda_objs ${CU_SRC_FILES})

    
#message(STATUS "CUDA FOUND")
#endif()


add_library(dnl SHARED ${SRC_FILES} ${CU_SRC_FILES} ${cuda_objs})
target_link_libraries(dnl ${LIBRARIES} ${OpenCV_LIBS} cudnn)

add_executable(darknet ${PROJECT_SOURCE_DIR}/examples/darknet.c)
target_link_libraries(darknet ${LIBRARIES} dnl)


