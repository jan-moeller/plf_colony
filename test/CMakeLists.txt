# Hack! Remove the getchar() calls from the test executable to make it fit for CI.
file(READ ${CMAKE_CURRENT_LIST_DIR}/plf_colony_test_suite.cpp PLF_COLONY_TEST_SOURCE)
string(REPLACE "getchar()" "/* getchar() */" PLF_COLONY_TEST_SOURCE "${PLF_COLONY_TEST_SOURCE}")
file(GENERATE OUTPUT test.cpp CONTENT "${PLF_COLONY_TEST_SOURCE}")

add_executable(${PROJECT_NAME}-tests ${CMAKE_CURRENT_BINARY_DIR}/test.cpp)
target_link_libraries(${PROJECT_NAME}-tests ${PROJECT_NAME})

if (PLF_COLONY_USE_ASAN)
    target_compile_options(${PROJECT_NAME}-tests INTERFACE
            $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-fsanitize=address>
    )
    target_link_options(${PROJECT_NAME}-tests INTERFACE
            $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-fsanitize=address>
    )
endif ()
if (PLF_COLONY_USE_UBSAN)
    target_compile_options(${PROJECT_NAME}-tests INTERFACE
            $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-fsanitize=undefined>
    )
    target_link_options(${PROJECT_NAME}-tests INTERFACE
            $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-fsanitize=undefined>
    )
endif ()
if (PLF_COLONY_USE_LSAN)
    target_compile_options(${PROJECT_NAME}-tests INTERFACE
            $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-fsanitize=leak>
    )
    target_link_options(${PROJECT_NAME}-tests INTERFACE
            $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-fsanitize=leak>
    )
endif ()

# Hack! Add the inner include path since the test executable includes it like that.
target_include_directories(${PROJECT_NAME}-tests PRIVATE ../include/plf)

add_test(NAME test COMMAND ${PROJECT_NAME}-tests)
