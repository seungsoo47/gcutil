PROJECT(gc)
CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

INCLUDE_DIRECTORIES(bdwgc/include)

FILE(GLOB GCUTIL_BDWGC_SRC bdwgc/*.c)
FILE(GLOB GCUTIL_SRC *.cpp)

#SET (GCUTIL_MODE "release" CACHE STRING "GCUTIL_MODE")
SET (GCUTIL_ENABLE_THREADING "0" CACHE STRING "GCUTIL_ENABLE_THREADING")
SET (GCUTIL_ENABLE_THREAD_LOCAL_ALLOC "0" CACHE STRING "GCUTIL_ENABLE_THREAD_LOCAL_ALLOC")

SET (GCUTIL_CFLAGS_INTERNAL -g3 -fdata-sections -ffunction-sections -DESCARGOT -Wno-unused-variable -fno-strict-aliasing -DGC_DLL=1 -fvisibility=hidden)
SET (GCUTIL_CFLAGS_INTERNAL ${GCUTIL_CFLAGS_INTERNAL} -DALL_INTERIOR_POINTERS=0 -DENABLE_DISCLAIM=1 -DGC_ATOMIC_UNCOLLECTABLE=1 -DGC_DONT_REGISTER_MAIN_STATIC_DATA=1 -DGC_ENABLE_SUSPEND_THREAD=1)
SET (GCUTIL_CFLAGS_INTERNAL ${GCUTIL_CFLAGS_INTERNAL} -DGC_NO_THREADS_DISCOVERY=1 -DGC_VERSION_MAJOR=8 -DGC_VERSION_MICRO=4 -DGC_VERSION_MINOR=0)
SET (GCUTIL_CFLAGS_INTERNAL ${GCUTIL_CFLAGS_INTERNAL} -DHAVE_DLADDR=1 -DHAVE_DLFCN_H=1 -DHAVE_DL_ITERATE_PHDR=1 -DHAVE_INTTYPES_H=1 -DHAVE_MEMORY_H=1 -DHAVE_STDINT_H=1 -DHAVE_STDLIB_H=1 -DHAVE_STRINGS_H=1)
SET (GCUTIL_CFLAGS_INTERNAL ${GCUTIL_CFLAGS_INTERNAL} -DHAVE_STRING_H=1 -DHAVE_SYS_STAT_H=1 -DHAVE_SYS_TYPES_H=1 -DHAVE_UNISTD_H=1)
SET (GCUTIL_CFLAGS_INTERNAL ${GCUTIL_CFLAGS_INTERNAL} -DIGNORE_DYNAMIC_LOADING=1 -DJAVA_FINALIZATION=1 -DMUNMAP_THRESHOLD=1 -DNO_EXECUTE_PERMISSION=1 -DSTDC_HEADERS=1 -DUSE_MMAP=1 -DUSE_MUNMAP=1)
SET (GCUTIL_CFLAGS_INTERNAL ${GCUTIL_CFLAGS_INTERNAL} -DHAVE_PTHREAD_GETATTR_NP=1 -DUSE_GET_STACKBASE_FOR_MAIN=1)

IF (${GCUTIL_ENABLE_THREADING} STREQUAL "1")
    SET (GCUTIL_CFLAGS_INTERNAL ${GCUTIL_CFLAGS_INTERNAL} -DGC_BUILTIN_ATOMIC=1 -DGC_THREADS=1 -DPARALLEL_MARK=1 -D_REENTRANT=1 -DGC_ENABLE_INCREMENTAL=1)

    IF (${GCUTIL_ENABLE_THREAD_LOCAL_ALLOC} STREQUAL "1")
        SET (GCUTIL_CFLAGS_INTERNAL ${GCUTIL_CFLAGS_INTERNAL} -DTHREAD_LOCAL_ALLOC=1)
    ENDIF()
ENDIF()

IF (${GCUTIL_MODE} STREQUAL "debug")
    SET (GCUTIL_CFLAGS_INTERNAL ${GCUTIL_CFLAGS_INTERNAL} -DKEEP_BACK_PTRS=1 -DSAVE_CALL_COUNT=8 -DDBG_HDRS_ALL=1 -DGC_DEBUG -O0)
ELSEIF (${GCUTIL_MODE} STREQUAL "release")
    SET (GCUTIL_CFLAGS_INTERNAL ${GCUTIL_CFLAGS_INTERNAL} -DNO_DEBUGGING=1 -O2)
ENDIF()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -fno-rtti")

add_compile_options(${GCUTIL_CFLAGS_INTERNAL})
add_compile_options(${GCUTIL_CFLAGS})
SET (GCUTIL_CFLAGS_FROM_ENV $ENV{CFLAGS})
SEPARATE_ARGUMENTS(GCUTIL_CFLAGS_FROM_ENV)
add_compile_options(${GCUTIL_CFLAGS_FROM_ENV})

ADD_LIBRARY(gc-lib STATIC ${GCUTIL_BDWGC_SRC} ${GCUTIL_SRC})
