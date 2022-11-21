## Build & Install

My enviroment:

- Architecture: x86_64
- OS: Ubuntu 22.04 LTS
- Editor: VS Code 1.71.0
- Compiler: gcc 11.2.0
- Build Tool: CMake 3.22.1

First install [dependencies](https://alandefreitas.github.io/matplotplusplus/integration/install/build-from-source/dependencies/) what you need.

Download the source code from [GitHub Releases](https://github.com/alandefreitas/matplotplusplus/releases). Choose _Source code(tar.gz)_.

Use `lscpu` to see how many cores on your machine, mine is `12`, so I replaced `2` by `12`:

```shell
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_FLAGS="-O2" -DBUILD_EXAMPLES=OFF -DBUILD_TESTS=OFF 
sudo cmake --build . --parallel 12 --config Release
sudo cmake --install .
```

Path to installation:

- headers: _/usr/local/include_
- libs: _/usr/local/lib_

Add "/usr/local/include" to your `includePath` in VS Code to support [IntelliSense for cross-compiling](https://code.visualstudio.com/docs/cpp/configure-intellisense-crosscompilation). Then you can add this header to your source files:

```CPP
#include <matplot/matplot.h>
```

## Find as External Package with CMake

A simple example, here _example.cpp_ and the _CMakeLists.txt_ are in the same directory:

```CMake
cmake_minimum_required(VERSION 3.14)
project(example VERSION 1.0.1)
set(CMAKE_CXX_STANDARD 17)

# Uncomment and adjust the next line if Matplot++ was not installed in a default directory
# list(APPEND CMAKE_MODULE_PATH put/your/installation/directory/here)

find_package(Matplot++)

if (Matplot++_FOUND)
    add_executable(example example.cpp)
    target_link_libraries(example PRIVATE Matplot++::matplot)
endif ()
```

You can find [C++ examples](https://alandefreitas.github.io/matplotplusplus/plot-types/line-plots/line-plot/) and [details about this CMakeLists.txt](https://alandefreitas.github.io/matplotplusplus/integration/cmake/find-as-external-package/) in the documentation.