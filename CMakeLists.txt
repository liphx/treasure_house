cmake_minimum_required(VERSION 3.11 FATAL_ERROR)

project(treasure_house)

add_subdirectory(src)

set(PROJECT_ROOT ${CMAKE_CURRENT_SOURCE_DIR})

# we will use the network to fetch Google Test sources
# make it possible to disable unit tests when not on network
option(ENABLE_UNIT_TESTS "Enable unit tests" ON)
message(STATUS "Enable testing: ${ENABLE_UNIT_TESTS}")

if(ENABLE_UNIT_TESTS)
  # the following code to fetch googletest
  # is inspired by and adapted after:
  #   - https://cmake.org/cmake/help/v3.11/module/FetchContent.html
  include(FetchContent)

  FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG        release-1.10.0
  )

  FetchContent_GetProperties(googletest)

  if(NOT googletest_POPULATED)
    FetchContent_Populate(googletest)
    # Prevent GoogleTest from overriding our compiler/linker options
    # when building with Visual Studio
    set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
    # Prevent GoogleTest from using PThreads
    set(gtest_disable_pthreads ON CACHE BOOL "" FORCE)

    # adds the targers: gtest, gtest_main, gmock, gmock_main
    message('${googletest_SOURCE_DIR}')
    add_subdirectory(${googletest_SOURCE_DIR} ${googletest_BINARY_DIR})

    # Silence std::tr1 warning on MSVC
    if(MSVC)
        foreach(_tgt gtest gtest_main gmock gmock_main)
        target_compile_definitions(${_tgt}
            PRIVATE
            "_SILENCE_TR1_NAMESPACE_DEPRECATION_WARNING"
            )
        endforeach()
    endif()
  endif()

  enable_testing()
  add_subdirectory(tests)
endif()