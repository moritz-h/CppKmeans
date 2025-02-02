include(FetchContent)
FetchContent_Declare(
  googletest
  URL https://github.com/google/googletest/archive/afd902e992b720d1b3e106bc5e425a5768872265.zip
)

# For Windows: Prevent overriding the parent project's compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

# Avoid installing GoogleTest when installing this project.
option(INSTALL_GTEST "Enable installation of googletest." OFF)

FetchContent_MakeAvailable(googletest)

enable_testing()

add_executable(
    libtest 
    src/HartiganWong.cpp
    src/Lloyd.cpp
    src/MiniBatch.cpp
    src/InitializeRandom.cpp
    src/InitializeKmeansPP.cpp
    src/InitializePCAPartition.cpp
    src/Kmeans.cpp
)

target_link_libraries(
    libtest
    gtest_main
    kmeans
)

# Also testing the custom parallelization.
add_executable(
    cuspartest
    src/HartiganWong.cpp
    src/Lloyd.cpp
    src/MiniBatch.cpp
    src/InitializeRandom.cpp
    src/InitializeKmeansPP.cpp
    src/InitializePCAPartition.cpp
    src/Kmeans.cpp
)

target_link_libraries(
    cuspartest
    gtest_main
    kmeans
)

target_compile_definitions(cuspartest PRIVATE CUSTOM_PARALLEL_TEST=1)

set(USE_OPENMP OFF CACHE BOOL "Enable OpenMP for tests")
if(USE_OPENMP)
    find_package(OpenMP REQUIRED)
    target_link_libraries(libtest OpenMP::OpenMP_CXX)
endif()

set(CODE_COVERAGE OFF CACHE BOOL "Enable coverage testing for tests")
if(CODE_COVERAGE AND CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(libtest PRIVATE -O0 -g --coverage)
    target_link_options(libtest PRIVATE --coverage)
endif()

include(GoogleTest)
gtest_discover_tests(libtest)
gtest_discover_tests(cuspartest)
