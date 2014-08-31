if (NOT DISPATCH_ENABLE_TEST_SUITE)
    return ()
endif ()

if (NOT COMPILER_HAS_BLOCKS_SUPPORT)
    message(WARNING
        "No compiler support for Blocks; test suite will not be built.")
    return ()
endif ()
#///////////////////////////////////////////////////////////////////////////////

# Aggregate target that collects all the test executables.
add_custom_target(tests)

execute_process(
    # Give the test suite access to dispatch's private headers.
    COMMAND ${CMAKE_COMMAND} -E
        create_symlink "${dispatch_SOURCE_DIR}/private" "dispatch"
)

add_library(bsdtests OBJECT
    bsdtests.c
    bsdtests.h
    dispatch_test.c
    dispatch_test.h
    shims.h
    shims/asprintf.c
    shims/asprintf.h
    shims/arc4random.c
    shims/arc4random.h
    shims/fgetln.c
    shims/fgetln.h
)

function (dispatch_test test_name)
    if (ARGN)
        set (sources "${ARGN}")
    else ()
        set (sources "${test_name}")
    endif ()

    add_executable("${test_name}" ${sources} $<TARGET_OBJECTS:bsdtests>)
    add_dependencies(tests "${test_name}")
    set_property(TARGET "${test_name}" PROPERTY RUNTIME_OUTPUT_DIRECTORY tests)
endfunction ()

add_definitions(
    -Wall
    -Wextra
    -Wno-unused-parameter
    -fblocks
)

# Sometimes we need to link additional libraries for maths functions
dispatch_search_libs(sin LIBRARIES m REQUIRED)
set (MATH_LIBRARIES "${SIN_LIBRARIES}")

link_libraries(libdispatch_shared ${MATH_LIBRARIES})

include_directories(
    ${dispatch_BINARY_DIR}
    ${dispatch_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_BINARY_DIR}
)

include_directories(SYSTEM
    ${KQUEUE_INCLUDE_DIRS}
    ${PTHREAD_WORKQUEUE_INCLUDE_DIRS}
    ${BLOCKS_RUNTIME_INCLUDE_DIRS}
)

add_executable(bsdtestharness bsdtestharness.c $<TARGET_OBJECTS:bsdtests>)
add_executable(bsdtestsummarize bsdtestsummarize.c $<TARGET_OBJECTS:bsdtests>)
add_dependencies(tests bsdtestharness bsdtestsummarize)

dispatch_test( dispatch_apply )
dispatch_test( dispatch_api )
dispatch_test( dispatch_c99 )
dispatch_test( dispatch_debug )
dispatch_test( dispatch_queue_finalizer )
dispatch_test( dispatch_group )
dispatch_test( dispatch_overcommit )
dispatch_test( dispatch_pingpong )
dispatch_test( dispatch_plusplus )

dispatch_test( dispatch_priority )
dispatch_test( dispatch_priority2 dispatch_priority.c )
set_property(TARGET dispatch_priority2 APPEND PROPERTY
    COMPILE_DEFINITIONS USE_SET_TARGET_QUEUE=1)

dispatch_test( dispatch_concur )
dispatch_test( dispatch_context_for_key )
dispatch_test( dispatch_proc )
dispatch_test( dispatch_read )
dispatch_test( dispatch_read2 )
dispatch_test( dispatch_after )
dispatch_test( dispatch_timer )
dispatch_test( dispatch_timer_short )
dispatch_test( dispatch_timer_timeout )
dispatch_test( dispatch_sema )
dispatch_test( dispatch_suspend_timer )
dispatch_test( dispatch_timer_bit31 )
dispatch_test( dispatch_timer_bit63 )
dispatch_test( dispatch_timer_set_time )
dispatch_test( dispatch_starfish )
dispatch_test( dispatch_cascade )
dispatch_test( dispatch_drift )
dispatch_test( dispatch_readsync )
dispatch_test( dispatch_data )
dispatch_test( dispatch_io )
dispatch_test( dispatch_io_net )
dispatch_test( dispatch_vm )
dispatch_test( dispatch_vnode )
dispatch_test( dispatch_select )

if (HAVE_MACH)
    dispatch_test( dispatch_deadname )
endif ()

