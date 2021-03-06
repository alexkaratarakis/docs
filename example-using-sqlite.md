## Example: Using Sqlite

  - Step 1: Install
  - Step 2: Use
    - [VS/MSBuild Project (User-wide integration)](#msbuild)
    - [CMake (Toolchain file)](#cmake)

### Step 1: Install

First, we need to know what name [Sqlite](https://sqlite.org) goes by in the ports tree. To do that, we'll run the `search` command and inspect the output:
```
PS D:\src\vcpkg> .\vcpkg search sqlite
libodb-sqlite        2.4.0            Sqlite support for the ODB ORM library
sqlite3              3.15.0           SQLite is a software library that implements a se...

If your library is not listed, please open an issue at:
    https://github.com/Microsoft/vcpkg/issues
```
Looking at the list, we can see that the port is named "sqlite3". You can also run the `search` command without arguments to see the full list of packages.

Installing is then as simple as using the `install` command.
```
PS D:\src\vcpkg> .\vcpkg install sqlite3
-- CURRENT_INSTALLED_DIR=D:/src/vcpkg/installed/x86-windows
-- DOWNLOADS=D:/src/vcpkg/downloads
-- CURRENT_PACKAGES_DIR=D:/src/vcpkg/packages/sqlite3_x86-windows
-- CURRENT_BUILDTREES_DIR=D:/src/vcpkg/buildtrees/sqlite3
-- CURRENT_PORT_DIR=D:/src/vcpkg/ports/sqlite3/.
-- Downloading https://sqlite.org/2016/sqlite-amalgamation-3150000.zip...
-- Downloading https://sqlite.org/2016/sqlite-amalgamation-3150000.zip... OK
-- Testing integrity of downloaded file...
-- Testing integrity of downloaded file... OK
-- Extracting source D:/src/vcpkg/downloads/sqlite-amalgamation-3150000.zip
-- Extracting done
-- Configuring x86-windows-rel
-- Configuring x86-windows-rel done
-- Configuring x86-windows-dbg
-- Configuring x86-windows-dbg done
-- Build x86-windows-rel
-- Build x86-windows-rel done
-- Build x86-windows-dbg
-- Build x86-windows-dbg done
-- Package x86-windows-rel
-- Package x86-windows-rel done
-- Package x86-windows-dbg
-- Package x86-windows-dbg done
-- Performing post-build validation
-- Performing post-build validation done
Package sqlite3:x86-windows is installed
```

We can check that sqlite3 was successfully installed for x86 windows desktop by running the `list` command.
```
PS D:\src\vcpkg> .\vcpkg list
sqlite3:x86-windows         3.15.0           SQLite is a software library that implements a se...
```

To install for other architectures and platforms such as Universal Windows Platform or x64 Desktop, you can suffix the package name with `:<target>`.
```
PS D:\src\vcpkg> .\vcpkg install sqlite3:x86-uwp zlib:x64-windows
```

See `.\vcpkg help triplet` for all supported targets.

### Step 2: Use
<a name="msbuild"></a>
#### VS/MSBuild Project (User-wide integration)

The recommended and most productive way to use vcpkg is via user-wide integration, making the system available for all projects you build. The user-wide integration will prompt for administrator access the first time it is used on a given machine, but afterwords is no longer required and the integration is configured on a per-user basis.
```
PS D:\src\vcpkg> .\vcpkg integrate install
Applied user-wide integration for this vcpkg root.

All C++ projects can now #include any installed libraries.
Linking will be handled automatically.
Installing new libraries will make them instantly available.
```
*Note: You will need to restart Visual Studio or perform a Build to update intellisense with the changes.* 

You can now simply use File -> New Project in Visual Studio 2015 or Visual Studio 2017 and the library will be automatically available. For Sqlite, you can try out their [C/C++ sample](https://sqlite.org/quickstart.html).

To remove the integration for your user, you can use `.\vcpkg integrate remove`.

<a name="cmake"></a>
#### CMake (Toolchain File)

The best way to use installed libraries with cmake is via the toolchain file `scripts\buildsystems\vcpkg.cmake`. To use this file, you simply need to add it onto your CMake command line as `-DCMAKE_TOOLCHAIN_FILE=D:\src\vcpkg\scripts\buildsystems\vcpkg.cmake`.

If you are using CMake through Open Folder with Visual Studio 2017 you can define `CMAKE_TOOLCHAIN_FILE` by adding a "variables" section to each of your `CMakeSettings.json` configurations:

```json
{
  "configurations": [{
    "name": "x86-Debug",
    "generator": "Visual Studio 15 2017",
    "configurationType" : "Debug",
    "buildRoot":  "${env.LOCALAPPDATA}\\CMakeBuild\\${workspaceHash}\\build\\${name}",
    "cmakeCommandArgs": "",
    "buildCommandArgs": "-m -v:minimal",
    "variables": [{
      "name": "CMAKE_TOOLCHAIN_FILE",
      "value": "D:\\src\\vcpkg\\scripts\\buildsystems\\vcpkg.cmake"
    }]
  }]
}
```

Now let's make a simple CMake project with a main file.
```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.0)
project(test)

find_package(Sqlite3 REQUIRED)

add_executable(main main.cpp)
target_link_libraries(main sqlite3)
```
```cpp
// main.cpp
#include <sqlite3.h>
#include <stdio.h>

int main()
{
    printf("%s\n", sqlite3_libversion());
    return 0;
}
```

Then, we build our project in the normal CMake way:
```
PS D:\src\cmake-test> mkdir build 
PS D:\src\cmake-test> cd build
PS D:\src\cmake-test\build> cmake .. "-DCMAKE_TOOLCHAIN_FILE=D:\src\vcpkg\scripts\buildsystems\vcpkg.cmake"
    // omitted CMake output here //
-- Build files have been written to: D:/src/cmake-test/build
PS D:\src\cmake-test\build> cmake --build .
    // omitted MSBuild output here //
Build succeeded.
    0 Warning(s)
    0 Error(s)

Time Elapsed 00:00:02.38
PS D:\src\cmake-test\build> .\Debug\main.exe
3.15.0
```

*Note: The correct sqlite3.dll is automatically copied to the output folder when building for x86-windows. You will need to distribute this along with your application.*

##### Handling libraries without native cmake support

Unlike other platforms, we do not automatically add the `include\` directory to your compilation line by default. If you're using a library that does not provide CMake integration, you will need to explicitly search for the files and add them yourself using [`find_path()`][1] and [`find_library()`][2].

```cmake
# To find and use catch
find_path(CATCH_INCLUDE_DIR catch.hpp)
include_directories(${CATCH_INCLUDE_DIR})

# To find and use azure-storage-cpp
find_path(WASTORAGE_INCLUDE_DIR was/blob.h)
find_library(WASTORAGE_LIBRARY wastorage)
include_directories(${WASTORAGE_INCLUDE_DIR})
link_libraries(${WASTORAGE_LIBRARY})

# Note that we recommend using the target-specific directives for a cleaner cmake:
#     target_include_directories(main ${LIBRARY})
#     target_link_libraries(main PRIVATE ${LIBRARY})
```

[1]: https://cmake.org/cmake/help/latest/command/find_path.html
[2]: https://cmake.org/cmake/help/latest/command/find_library.html



