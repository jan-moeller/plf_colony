cmake_minimum_required(VERSION 3.14 FATAL_ERROR)
project(plf_colony
        VERSION 7.4.3
        LANGUAGES CXX
)

#############################################################################################################
# Prevent in-source builds
#############################################################################################################
if (PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
    message(
            FATAL_ERROR
            "In-source builds not allowed. Please make a new directory (called a build directory) and run CMake from there."
    )
endif ()

#############################################################################################################
# Build configuration
#############################################################################################################
option(PLF_COLONY_BUILD_TESTS "Include test target in main build" OFF)
option(PLF_COLONY_USE_ASAN "Enable address sanitizer in the test build" OFF)
option(PLF_COLONY_USE_UBSAN "Enable UB sanitizer in the test build" OFF)
option(PLF_COLONY_USE_LSAN "Enable leak sanitizer in the test build" OFF)

message(STATUS "------------------------------------------------------------------------------")
message(STATUS "    ${PROJECT_NAME} (${PROJECT_VERSION})")
message(STATUS "------------------------------------------------------------------------------")
message(STATUS "Build type:                ${CMAKE_BUILD_TYPE}")
message(STATUS "Build tests:               ${PLF_COLONY_BUILD_TESTS}")
message(STATUS "Use address sanitizer      ${PLF_COLONY_USE_ASAN}")
message(STATUS "Use UB sanitizer           ${PLF_COLONY_USE_UBSAN}")
message(STATUS "Use leak sanitizer         ${PLF_COLONY_USE_LSAN}")

#############################################################################################################
# Main target
#############################################################################################################
add_library(${PROJECT_NAME} INTERFACE
        include/plf/colony.hpp
        include/plf/plf_colony.h
)
target_include_directories(${PROJECT_NAME}
        INTERFACE
        $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include/${PROJECT_NAME}-${PROJECT_VERSION}>
)
set_target_properties(${PROJECT_NAME} PROPERTIES
        LINKER_LANGUAGE CXX
        CXX_STANDARD 98
        CXX_STANDARD_REQUIRED YES
        CXX_EXTENSIONS NO
)
add_library(plf::colony ALIAS ${PROJECT_NAME})

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
        "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
        VERSION ${CMAKE_PROJECT_VERSION}
        COMPATIBILITY SameMajorVersion ARCH_INDEPENDENT
)
install(TARGETS ${PROJECT_NAME}
        EXPORT ${PROJECT_NAME}Targets
        LIBRARY DESTINATION lib COMPONENT Runtime
        ARCHIVE DESTINATION lib COMPONENT Development
        RUNTIME DESTINATION bin COMPONENT Runtime
        PUBLIC_HEADER DESTINATION include COMPONENT Development
        BUNDLE DESTINATION bin COMPONENT Runtime
)
configure_package_config_file(
        "${PROJECT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in"
        "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
        INSTALL_DESTINATION lib/cmake/${PROJECT_NAME}
)

install(EXPORT ${PROJECT_NAME}Targets DESTINATION lib/cmake/${PROJECT_NAME})
install(FILES "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
        "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
        DESTINATION lib/cmake/${PROJECT_NAME})
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/ DESTINATION include)

#############################################################################################################
# Test target (if configured that way)
#############################################################################################################
if (${PLF_COLONY_BUILD_TESTS})
    enable_testing()
    add_subdirectory(test)
endif ()