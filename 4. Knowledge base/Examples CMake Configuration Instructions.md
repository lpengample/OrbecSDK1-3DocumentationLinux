# CMake Configuration Instructions
## Examples CMake project structure
```
Examples
│
├─c
│  ├─CMakeLists.txt
│  ├─Sample-ColorViewer
│  │     └── CMakeLists.txt
│  ├─Sample-DepthViewer
│  │     └── CMakeLists.txt
│  ├─Sample-HelloOrbbec
│  │     └── CMakeLists.txt
│  ├─Sample-HotPlugin
│  │     └── CMakeLists.txt
│  ├─Sample-InfraredViewer
│  │     └── CMakeLists.txt
│  ├─Sample-PointCloud
│  │     └── CMakeLists.txt
│  └─Sample-SensorControl
│        └── CMakeLists.txt
└─cpp
    ├─CMakeLists.txt
    ├─Sample-ColorViewer
    │     └── CMakeLists.txt
    ├─Sample-DepthViewer
    │     └── CMakeLists.txt
    ├─Sample-HelloOrbbec
    │     └── CMakeLists.txt
    ├─Sample-HotPlugin
    │     └── CMakeLists.txt
    ├─Sample-ImuReader
    │     └── CMakeLists.txt
    ├─Sample-InfraredViewer
    │     └── CMakeLists.txt
    ├─Sample-MultiDevice
    │     └── CMakeLists.txt
    ├─Sample-PointCloud
    │     └── CMakeLists.txt
    ├─Sample-SaveToDisk
    │     └── CMakeLists.txt
    ├─Sample-SensorControl
    │     └── CMakeLists.txt
    ├─Sample-SyncAlignViewer
    |     └── CMakeLists.txt 
    ├─Sample-Recorder
    |     └── CMakeLists.txt 
    └─Sample-Playback
          └── CMakeLists.txt 
          
```
As the above tree structure, the entire CMake project structure can be divided into three layers, which is the same as the program directory structure. Each layer will have a CMakeLists.txt file, and CMakeLists.txt in the root directory of the first layer is the main CMakeLists.txt. The CMakeLists.txt of the second layer is placed in two different directories, corresponding to the C version and the C++ version of Example. The third layer of CMakeLists.txt is placed in the respective Example directory.
## Main CMakeLists.txt description

The following is from the CMakeLists.txt in the root directory:<br />1. The minimum version of CMake required for setting is 3.1.15
```
cmake_minimum_required(VERSION 3.1.15)
```
2. Generate a project named OrbbecSDK-Examples, using C++ and C language
```
project(OrbbecSDK-Examples LANGUAGES CXX C)
```
3. Compile with standard C++11
```
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```
4. Set SDK header file and library path
```
set(OBSensor_DIR ${CMAKE_SOURCE_DIR}/../SDK)
set(OBSensor_LIBRARY_DIRS ${OBSensor_ROOT_DIR}/lib/${CMAKE_BUILD_TYPE})
set(OBSensor_INCLUDE_DIR ${OBSensor_ROOT_DIR}/include)
include_directories(${OBSensor_INCLUDE_DIR})
link_directories(${OBSensor_LIBRARY_DIRS})
```
5. Find the opencv library
```
# set(OpenCV_DIR "your/path/to/opencv")  # Opencv can be specified with this command_ Dir module lookup path
find_package(OpenCV QUIET)
if(${OpenCV_FOUND})
   	include_directories(${OpenCV_INCLUDE_DIR})
 		link_directories(${OpenCV_LIBRARY_DIRS})
ENDIF(${OpenCV_FOUND})
```
6. Set the output directory of ARCHIVE, LIBRARY, RUNTIME,
```
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
```
Introduction of these three variables:

|  | Windows | Unix |
| --- | --- | --- |
| RUNTIME | .exe、.dll | Executable program |
| LIBRARY |  | .so |
| ARCHIVE | .a、.lib | .a |

7. Add subproject directory
```
add_subdirectory(c)
add_subdirectory(cpp)
```
## C language version Examples-Main CMakeLists description
```
cmake_minimum_required(VERSION 3.1.15)

add_subdirectory(Sample-HelloOrbbec)
add_subdirectory(Sample-SensorControl)
add_subdirectory(Sample-HotPlugin)

# opencv required
IF(${OpenCV_FOUND})
    add_subdirectory(Sample-ColorViewer)
    add_subdirectory(Sample-DepthViewer)
    add_subdirectory(Sample-InfraredViewer)
    add_subdirectory(Sample-PointCloud)
ENDIF(${OpenCV_FOUND})
```
This file is relatively simple, here is a brief description. It is mainly based on whether the opencv library is found and whether each example depends on the opencv library to determine whether to add the example to the project.
## C language version Examples-Sub CMakeLists instructions
In each Example subdirectory contains their own CMakeLists.txt, taking Sample-HelloOrbbec as an example:

```
cmake_minimum_required(VERSION 3.1.15)
set(CMAKE_CXX_STANDARD 11)

project(Sample-HelloOrbbec)

add_executable(hello_orbbec HelloOrbbec.c)
target_link_libraries(hello_orbbec obsensor)
```
The content is to specify the source file HelloOrbbec.c for producing the executable file hello_orbbec (windows platform generation file to be .exe suffix), and specify the connection to the obsensor library.

## C++ version Examples-Main CMakeLists description
```
cmake_minimum_required(VERSION 3.1.15)

add_subdirectory(Sample-HelloOrbbec)
add_subdirectory(Sample-SensorControl)
add_subdirectory(Sample-ImuReader)

# opencv required
IF(${OpenCV_FOUND})
    add_subdirectory(Sample-ColorViewer)
    add_subdirectory(Sample-DepthViewer)
    add_subdirectory(Sample-InfraredViewer)
    add_subdirectory(Sample-SyncAlignViewer)
    add_subdirectory(Sample-HotPlugin)
    add_subdirectory(Sample-PointCloud)
    add_subdirectory(Sample-MultiDevice)
    add_subdirectory(Sample-SaveToDisk)
    add_subdirectory(Sample-Recorder)
    add_subdirectory(Sample-Playback)
ENDIF(${OpenCV_FOUND})
```
The target and content of the main CMakeLists.txt of the C++ version Examples are basically the same as the C language version, so we will not repeat them here. The difference is that the C++ version provides richer routines.
## C++ version Examples-Sub CMakeLists description
It is basically the same as the C language version, so we will not repeat it here.
