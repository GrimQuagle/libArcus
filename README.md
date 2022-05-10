# Arcus

<p align="center">
    <a href="https://github.com/Ultimaker/libArcus/actions/workflows/conan-package.yml" alt="Conan Package">
        <img src="https://github.com/Ultimaker/libarcus/actions/workflows/conan-package.yml/badge.svg" /></a>
    <a href="https://github.com/Ultimaker/libArcus/issues" alt="Open Issues">
        <img src="https://img.shields.io/github/issues/ultimaker/libarcus" /></a>
    <a href="https://github.com/Ultimaker/libArcus/issues?q=is%3Aissue+is%3Aclosed" alt="Closed Issues">
        <img src="https://img.shields.io/github/issues-closed/ultimaker/libarcus?color=g" /></a>
    <a href="https://github.com/Ultimaker/libArcus/pulls" alt="Pull Requests">
        <img src="https://img.shields.io/github/issues-pr/ultimaker/libarcus" /></a>
    <a href="https://github.com/Ultimaker/libArcus/graphs/contributors" alt="Contributors">
        <img src="https://img.shields.io/github/contributors/ultimaker/libarcus" /></a>
    <a href="https://github.com/Ultimaker/libArcus" alt="Repo Size">
        <img src="https://img.shields.io/github/repo-size/ultimaker/libarcus?style=flat" /></a>
    <a href="https://github.com/Ultimaker/libArcus/blob/master/LICENSE" alt="License">
        <img src="https://img.shields.io/github/license/ultimaker/libarcus?style=flat" /></a>
</p>

This library contains C++ code and Python bindings for creating a socket in a thread and using this socket to send and receive messages
based on the Protocol Buffers library. It is designed to facilitate the communication between Cura and its backend and similar code.

## License

![License](https://img.shields.io/github/license/ultimaker/libarcus?style=flat)  
Arcus is released under terms of the AGPLv3 License. Terms of the license can be found in the LICENSE file. Or at
http://www.gnu.org/licenses/agpl.html

> But in general it boils down to:  
> **You need to share the source of any Arcus modifications if you make an application with Arcus.**

## How to build

> **Note:**  
> We are currently in the process of switch our builds and pipelines to an approach which uses [Conan](https://conan.io/)
> and pip to manage our dependencies, which are stored on our JFrog Artifactory server and in the pypi.org.
> At the moment not everything is fully ported yet, so bare with us.

Follow the instructions below setup your development environment.  
_requirements: Python 3.6+ (3.10.4 is recommended since Cura 5.+ uses that one), gcc 9+ (Linux), VS2019+ (Windows),
apple-clang 11+ (MacOS), CMake 3.20+ (optional), Ninja 1.10+ (optional), make (Linux, MacOS), nmake (Windows), sip
(optional)_

If you have never used [Conan](https://conan.io/) read their [documentation](https://docs.conan.io/en/latest/index.html)
which is quite extensive and well maintained. Conan is a Python program and can be installed using pip

```shell
pip install conan
```

> **IMPORTANT**  
> Ultimaker has its own Conan config repository; which ensures that the correct compiler
> settings and Conan remotes are used. Although you could use your own Conan configuration and profiles it might be
> easier to use the Conan configuration which Ultimaker uses. This can be installed with the following command:
> ```shell
> conan profile new default --detect  # This should create a default profile with the system detected standard compiler
> conan config install https://github.com/ultimaker/conan-config.git  # This will install our profiles, settinsg and remotes
> ```

As mentioned in the notes above we use our own JFrog Artifactory server, so make sure you add this server tou your Conan
remotes. **This step is not necessary if you use our supplied Conan configuration.**

```shell
conan remote add ultimaker https://peer23peer.jfrog.io/artifactory/api/conan/cura-conan True 
```

### pyArcus python module (optional)

This repository also contains a Python module named pyArcus. To build it [sip](https://pypi.org/project/sip/)==6.5.1
needs to be installed in the current `site-packages`. This can be done with:

```shell
python -m pip install sip==6.5.1
```

#### usage

```python
import pyArcus
socket = pyArcus.Socket()
```

### Building Arcus and pyArcus

The steps above should be enough to get your system in such a state you can start development on Arcus. If you want
to use your own system provided CMake and CMake generators, such as: Ninja, Make, NMake use the following steps to
install the dependencies for Arcus. Executed in the root directory of Arcus.

#### Release build type

```shell
conan install . -pr:b cura_build.jinja -pr:h cura_release.jinja --build=missing
cd cmake-build-release
cmake --toolchain=generators/conan_toolchain.cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build .
```

Conan can also provide the build tools, such as CMake and Ninja. This can handy if your current system package manager
doesn't have the minimum required versions available. We can use these if we use the
[VirtualBuildEnv](https://docs.conan.io/en/latest/reference/conanfile/tools/env/virtualbuildenv.html) generator and
activate that build environement during building

```shell
# for Linux/MacOS
conan install . -pr:b cura_build.jinja -pr:h cura_release.jinja --build=missing -g VirtualBuildEnv
cd cmake-build-release
. generators/conanbuild.sh
cmake --toolchain=generators/conan_toolchain.cmake -DCMAKE_BUILD_TYPE=Release -G "Ninja" ..
cmake --build .
```

```shell
# for Windows
conan install . -pr:b cura_build.jinja -pr:h cura_release.jinja --build=missing -g VirtualBuildEnv
cd cmake-build-release
generators\conanbuild.bat
cmake --toolchain=build\generators\conan_toolchain.cmake -DCMAKE_BUILD_TYPE=Release -G "Ninja" ..
cmake --build .
```

#### Debug build type

Use the same instructions as above, but replace the host profile with cura_release.jinja for the conan install command
`-pr:h cura_debug.jinja`

```shell
conan install . -pr:b cura_build.jinja -pr:h cura_debug.jinja --build=missing
cd cmake-build-debug
cmake --toolchain=generators/conan_toolchain.cmake -DCMAKE_BUILD_TYPE=Debug ..
cmake --build .
```

Conan can also provide the build tools, such as CMake and Ninja. This can handy if your current system package manager
doesn't have the minimum required versions available. We can use these if we use the
[VirtualBuildEnv](https://docs.conan.io/en/latest/reference/conanfile/tools/env/virtualbuildenv.html) generator and
activate that build environement during building

```shell
# for Linux/MacOS
conan install . -pr:b cura_build.jinja -pr:h cura_debug.jinja --build=missing -g VirtualBuildEnv
cd cmake-build-debug
. generators/conanbuild.sh
cmake --toolchain=generators/conan_toolchain.cmake -DCMAKE_BUILD_TYPE=Debug -G "Ninja" ..
cmake --build .
```

```shell
# for Windows
conan install . -pr:b cura_build.jinja -pr:h cura_debug.jinja --build=missing -g VirtualBuildEnv
cd cmake-build-debug
generators\conanbuild.bat
cmake --toolchain=build\generators\conan_toolchain.cmake -DCMAKE_BUILD_TYPE=Debug -G "Ninja" ..
cmake --build .
```

## Dependencies

![Dependency graph](docs/assets/deps.png)

### Runtime dependencies
- [protobuf](docs/development/protobuf.md)
- [zlib](docs/development/zlib.md)

### Build dependencies
- [Python](https://www.python.org/)
- [Cmake](https://cmake.org/)
- [Ninja (optional)](https://ninja-build.org/)
- [GNU-GCC](https://gcc.gnu.org/)
- [Visual Studio](https://visualstudio.microsoft.com/vs/)
- [xcode command line tools](https://developer.apple.com/xcode/)
- [sip](https://pypi.org/project/sip/)

### IDE

If you're using [CLion](https://www.jetbrains.com/clion/) as an IDE be sure to checkout the Conan plugin
[Conan CLion plugin](https://docs.conan.io/en/latest/integrations/ide/clion.html)

## Using arcus with CMake

<br>

### [Conan CMake generators](https://docs.conan.io/en/latest/reference/conanfile/tools/cmake.html)

<br>

* [CMakeDeps](https://docs.conan.io/en/latest/reference/conanfile/tools/cmake/cmakedeps.html): generates information about where the **arcus** library and its dependencies  ( [protobuf](https://conan.io/center/protobuf),  [zlib](https://conan.io/center/zlib)) are installed together with other information like version, flags, and directory data or configuration. CMake will use this files when you invoke ``find_package()`` in your *CMakeLists.txt*.

* [CMakeToolchain](https://docs.conan.io/en/latest/reference/conanfile/tools/cmake/cmaketoolchain.html): generates a CMake toolchain file the you can later invoke with CMake in the command line using `-DCMAKE_TOOLCHAIN_FILE=conantoolchain.cmake`.

Declare these generators in your **conanfile.txt** along with your **arcus** dependency like:

```ini
[requires]
arcus/latest6@ultimaker/stable

[generators]
CMakeDeps
CMakeToolchain
```

<br>

To use **arcus** in a simple CMake project with this structure:

```shell
.
|-- CMakeLists.txt
|-- conanfile.txt
`-- src
    `-- main..cpp
```

<br>

Your **CMakeLists.txt** could look similar to this, using the global **arcus::arcus** CMake's target:

```cmake
cmake_minimum_required(VERSION 3.15)
project(arcus_project CXX)

find_package(arcus)

add_executable(${PROJECT_NAME} src/main.cpp)

# Use the global target
target_link_libraries(${PROJECT_NAME} arcus::arcus)
```

<br>

To install **arcus/latest@ultimaker/stable**, its dependencies and build your project, you just have to do:

```shell
# for Linux/macOS
$ conan install . --install-folder cmake-build-release --build=missing
$ cmake . -DCMAKE_TOOLCHAIN_FILE=cmake-build-release/conan_toolchain.cmake
$ cmake --build .

# for Windows and Visual Studio 2017
$ conan install . --output-folder cmake-build --build=missing
$ cmake . -G "Visual Studio 15 2017" -DCMAKE_TOOLCHAIN_FILE=cmake-build/conan_toolchain.cmake
$ cmake --build . --config Release
```



<br>



As the arcus Conan package defines components you can link only that desired part of the library in your project. For example, linking only with the arcus **libarcus** component, through the **arcus::libarcus** target.

```cmake
...
# Link just to arcus libarcus component
target_link_libraries(${PROJECT_NAME} arcus::libarcus)
```

<br>

To check all the available components for **arcus** Conan package, please check the dedicated section at the end of this document.


## Using the Socket


The socket assumes a very simple and strict wire protocol: one 32-bit integer with
a header, one 32-bit integer with the message size, one 32-bit integer with a type id
then a byte array containing the message as serialized by Protobuf. The receiving side
checks for these fields and will deserialize the message, after which it can be processed 
by the application.

To send or receive messages, the message first needs to be registered on both sides with 
a call to `registerMessageType()`. You can also register all messages from a Protobuf 
 .proto file with a call to `registerAllMessageTypes()`. For the Python bindings, this 
is the only supported way of registering since there are no Python classses for 
individual message types.

The Python bindings expose the same API as the Public C++ API, except for the missing
`registerMessageType()` and the individual messages. The Python bindings wrap the
messages in a class that exposes the message's properties as Python properties, and
can thus be set the same way you would set any other Python property. 

The exception is repeated fields. Currently, only repeated messages are supported, which
can be created through the `addRepeatedMessage()` method. `repeatedMessageCount()` will
return the number of repeated messages on an object and `getRepeatedMessage()` will get
a certain instance of a repeated message. See python/PythonMessage.h for more details.

Origin of the Name
==================

The name Arcus is from the Roman god Arcus. This god is the roman equivalent of
the goddess Iris, who is the personification of the rainbow and the messenger
of the gods.

Java
====
There is a Java port of libArcus, which can be found [here](https://github.com/Ocarthon/libArcus-Java).
