include(FetchContent)

set(BOOST_UT_ENABLE_RUN_AFTER_BUILD OFF CACHE INTERNAL "")
set(BOOST_UT_DISABLE_MODULE ON CACHE INTERNAL "")

FetchContent_Declare(
  ut
  URL https://github.com/boost-ext/ut/archive/refs/tags/v2.0.1.zip
)

message(STATUS "Fetching dependencies...")
set_directory_properties(PROPERTIES EXCLUDE_FROM_ALL TRUE)
set(CMAKE_SKIP_INSTALL_RULES ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(ut)
set(CMAKE_SKIP_INSTALL_RULES OFF CACHE BOOL "" FORCE)
set_directory_properties(PROPERTIES EXCLUDE_FROM_ALL FALSE)
message(STATUS "...finished fetching dependencies.")

include(../cmake/code-coverage.cmake)
add_code_coverage_all_targets()

add_library(glz_test_common INTERFACE)
target_compile_features(glz_test_common INTERFACE cxx_std_20)
target_link_libraries(glz_test_common INTERFACE Boost::ut glaze::glaze)
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(glz_test_common INTERFACE -fno-exceptions -fno-rtti)
    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        target_compile_options(glz_test_common INTERFACE -Wall -Wextra -pedantic $<$<CONFIG:Debug>:-Werror>)
    else()
        target_compile_options(glz_test_common INTERFACE -Wall -Wextra -pedantic)
    endif()

#    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
#      target_compile_options(glz_test_common INTERFACE -fsanitize=address)
#      target_link_options(glz_test_common INTERFACE -fsanitize=address)
#    endif()
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_compile_options(glz_test_common INTERFACE /GR- /bigobj) #/fsanitize=address
    target_compile_options(glz_test_common INTERFACE /W4 /wd4459 /wd4805)
endif()

add_library(glz_test23_common INTERFACE)
target_compile_features(glz_test23_common INTERFACE cxx_std_23)
target_link_libraries(glz_test23_common INTERFACE Boost::ut glaze::glaze)
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(glz_test23_common INTERFACE -fno-exceptions -fno-rtti)
    if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        target_compile_options(glz_test23_common INTERFACE -Wall -Wextra -pedantic $<$<CONFIG:Debug>:-Werror>)
    else()
        target_compile_options(glz_test23_common INTERFACE -Wall -Wextra -pedantic)
    endif()

#    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
#      target_compile_options(glz_test23_common INTERFACE -fsanitize=address)
#      target_link_options(glz_test23_common INTERFACE -fsanitize=address)
#    endif()
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_compile_options(glz_test23_common INTERFACE /GR- /bigobj) #/fsanitize=address
    target_compile_options(glz_test23_common INTERFACE /W4 /wd4459 /wd4805)
endif()

add_library(glz_test_exceptions INTERFACE)
target_compile_features(glz_test_exceptions INTERFACE cxx_std_20)
target_link_libraries(glz_test_exceptions INTERFACE Boost::ut glaze::glaze)
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(glz_test_exceptions INTERFACE -fno-rtti)
    if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        target_compile_options(glz_test_exceptions INTERFACE -Wall -Wextra -pedantic $<$<CONFIG:Debug>:-Werror>)
    else()
        # GCC reflection has no means of silencing 'used but never defined', so we can't use -Werror
        target_compile_options(glz_test_exceptions INTERFACE -Wall -Wextra -pedantic)
    endif()
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_compile_options(glz_test_exceptions INTERFACE /GR- /bigobj) #/fsanitize=address
    target_compile_options(glz_test_exceptions INTERFACE /W4 /wd4459 /wd4805)
endif()

add_subdirectory(api_test)
add_subdirectory(binary_test)
add_subdirectory(compare_test)
add_subdirectory(csv_test)
add_subdirectory(eigen_test)
add_subdirectory(exceptions_test)
add_subdirectory(json_conformance)
add_subdirectory(json_performance)
add_subdirectory(json_test)
add_subdirectory(jsonrpc_test)
add_subdirectory(lib_test)
add_subdirectory(reflection_test)

# We don't run find_package_test or glaze-install_test with MSVC/Windows, because the Github action runner often chokes
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    add_test(
    NAME glaze-install_test
    COMMAND
    "${CMAKE_COMMAND}"
    --install "${PROJECT_BINARY_DIR}"
    --prefix "${CMAKE_CURRENT_BINARY_DIR}/install"
    --config $<CONFIG>
    --verbose
    )

    add_test(
    NAME find_package_test
    COMMAND
    "${CMAKE_CTEST_COMMAND}"
    --verbose
    --output-on-failure
    --build-noclean
    --build-project "${PROJECT_NAME}" # helps msvc when --build-target
    --build-generator "${CMAKE_GENERATOR}"
    --build-config $<CONFIG>
    --build-and-test
    "${CMAKE_CURRENT_SOURCE_DIR}/find_package"
    "${CMAKE_CURRENT_BINARY_DIR}/find_package"
    --build-options
    "-Dglaze_ROOT:PATH=${CMAKE_CURRENT_BINARY_DIR}/install"
    "-DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}"
    "-DBUILD_TESTING=ON"
    --test-command "${CMAKE_CTEST_COMMAND}" --verbose --output-on-failure # inner ctest command
    )

    set_tests_properties(glaze-install_test PROPERTIES FIXTURES_SETUP glaze-install-fixture)
    set_tests_properties(find_package_test PROPERTIES FIXTURES_REQUIRED glaze-install-fixture)
endif()

