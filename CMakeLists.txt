cmake_minimum_required(VERSION 2.8.8)

project (dispatch)
set (dispatch_VERSION "1.2")

list (APPEND CMAKE_MODULE_PATH "${dispatch_SOURCE_DIR}/cmake")

if (CMAKE_SYSTEM_NAME MATCHES "Linux")
  add_definitions(-D_GNU_SOURCE)
endif ()

include_directories("${CMAKE_CURRENT_BINARY_DIR}/src")
include_directories("${DARLING_TOP_DIRECTORY}/src/libc/include")
include_directories("${DARLING_TOP_DIRECTORY}/src/libc/pthreads")
include_directories("${DARLING_TOP_DIRECTORY}/src/kernel/emulation/linux/ext")
include_directories("${DARLING_TOP_DIRECTORY}/src/libclosure")

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -nostdinc -fblocks -ggdb -O0")
set(CMAKE_SHARED_LINKER_FLAGS "-nostdlib -Wl,--version-script=${DARLING_TOP_DIRECTORY}/darwin.map")

#
# On Mac OS X, libdispatch_init is automatically invoked during libSystem
# process initialization.  On other systems, it is tagged as a library
# constructor to be run by automatically by the runtime linker.
#

set (USE_LIBDISPATCH_INIT_CONSTRUCTOR OFF)
set (USE_APPLE_TSD_OPTIMIZATIONS ON)

#
# On Mac OS X libdispatch can use the non-portable direct pthread TSD functions
#
option(USE_APPLE_TSD_OPTIMIZATIONS "Use non-portable pthread TSD optimizations for Mac OS X." OFF)

find_program(MIG_EXECUTABLE mig)

################################################################################

#
# Find libraries we will need
#

set (CBLOCKS_COMPILER_SUPPORT_FOUND ON)

# hack for xnu/bsd/sys/event.h EVFILT_SOCK declaration
if (EXISTS "${apple_xnu_source_bsd_path}")
  add_definitions(-DPRIVATE=1)
endif ()

set (HAVE_LIBKERN_OSBYTEORDER_H 1)
set (HAVE_MACH_ABSOLUTE_TIME 1)
set (HAVE_PTHREAD_MAIN_NP 1)
set (HAVE_GETPROGNAME 1)
set (HAVE_DECL_FD_COPY 1)
set (HAVE_STRLCPY 1)
set (HAVE_MACH_MACH_H ON)
set (HAVE_MACH 1)
set (USE_MIG ON)
set (__DARWIN_NON_CANCELABLE 1)

add_definitions(-D__ENVIRONMENT_MAC_OS_X_VERSION_MIN_REQUIRED__=1080)
add_definitions(-DHAVE_PTHREAD_WORKQUEUES=1)
set (HAVE_PTHREAD_WORKQUEUES 1)
set (HAVE_PTHREAD_WORKQUEUE_SETDISPATCH_NP 1)

if (HAVE_MACH)
  set (USE_MACH_SEM 1)
elseif (HAVE_SEM_INIT)
  set (USE_POSIX_SEM 1)
else ()
  message(FATAL_ERROR "no supported semaphore type")
endif ()

configure_file("config/config.h.cmake.in"
  "${dispatch_BINARY_DIR}/config/config.h"
  @ONLY
)

################################################################################

include_directories(SYSTEM fixincludes)

add_subdirectory(src)
add_subdirectory(testing)

# Install man pages
# add_subdirectory(man)

# Install public headers
# add_subdirectory(dispatch) 
# add_subdirectory(os)
