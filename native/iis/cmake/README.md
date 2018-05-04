# ISAPI redirector CMake project

[![AppVeyor CI Build Status](https://ci.appveyor.com/api/projects/status/pghcj500s1a14k9a?svg=true)](https://ci.appveyor.com/project/mabrarov/tomcat-connectors)
[![Coverity Scan Build Status](https://scan.coverity.com/projects/15566/badge.svg)](https://scan.coverity.com/projects/mabrarov-tomcat-connectors)

## Prerequisites

1. [CMake](https://cmake.org/)

    This CMake project was tested with [CMake 3.11.1](https://cmake.org/files/v3.11/cmake-3.11.1-win64-x64.zip).
    
1. [PCRE](https://www.pcre.org/)

    This CMake project was tested with [PCRE 8.42](https://ftp.pcre.org/pub/pcre/pcre-8.42.zip). While there is a copy of PCRE 8.41 in "[native/iis/pcre](../pcre)" directory, it wasn't tested. Please note that according to PCRE documentation there is no really working way to combine CMake project of PCRE with other CMake projects because PCRE requires installation (when some header files are collected and directory for PCRE includes is created) before it can be used.

    Use `PCRE_STATIC_RUNTIME` option of CMake project provided by PCRE to build static PCRE libraries with static C/C++ runtime. It is required if ISAPI redirector is planned to be built with **static** C/C++ runtime. 
    
    Do not use `PCRE_STATIC_RUNTIME` option (or explicitly set it to `OFF`) when building PCRE if ISAPI redirector is planned to be built with **shared** C/C++ runtime (refer to "[The latest supported Visual C++ downloads](https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads)" for information about Microsoft Visual C++ runtime redistributable packages).
      
    Below commands can be used to build 64-bit static PCRE libraries with static C/C++ runtime using Visual Studio 2015 (it is assumed that current directory is some temporary location used for generated Visual Studio project):

    ```cmd
    cmake.exe -D PCRE_STATIC_RUNTIME=ON -D CMAKE_INSTALL_PREFIX=path-to-directory-for-the-built-PCRE -G "Visual Studio 14 2015 Win64" path-to-directory-with-unpacked-sources-of-PCRE
    cmake --build . --config Debug --target install
    cmake --build . --config Release --target install
    ```
    
    Below commands can be used to build 64-bit static PCRE libraries with shared C/C++ runtime using Visual Studio 2015 (it is assumed that current directory is some temporary location used for generated Visual Studio project):
    
    ```cmd
    cmake.exe -D PCRE_STATIC_RUNTIME=OFF -D CMAKE_INSTALL_PREFIX=path-to-directory-for-the-built-PCRE -G "Visual Studio 14 2015 Win64" path-to-directory-with-unpacked-sources-of-PCRE
    cmake --build . --config Debug --target install
    cmake --build . --config Release --target install
    ```

## Building

It's assumed that `TOMCAT_JK_ROOT` is the directory where this git repository is cloned into.

1. `CMAKE_USER_MAKE_RULES_OVERRIDE` option should point to `TOMCAT_JK_ROOT/native/iis/cmake/cmake/static_c_runtime_overrides.cmake` if building with static C/C++ runtime.
1. `CMAKE_USER_MAKE_RULES_OVERRIDE_CXX` option should point to `TOMCAT_JK_ROOT/native/iis/cmake/cmake/static_cxx_runtime_overrides.cmake` if building with static C/C++ runtime.
1. `PCRE_ROOT` can be used as a hint for searching for PCRE. It should point to the directory where built PCRE is installed (refer to `path-to-directory-for-the-built-PCRE` in "[Prerequisites](#prerequisites)" section). 

## Example for generation of project

Example of command line for generation of Visual Studio 2015 project with below parameters:

1. `D:\Users\Marat\Documents\work\cpp\tomcat-connectors-1.2.43-src\native\iis\cmake` is the same directory where this `README.md` is located, i.e. this git repository is cloned into `D:\Users\Marat\Documents\work\cpp\tomcat-connectors-1.2.43-src` directory. 
1. `CMAKE_USER_MAKE_RULES_OVERRIDE` and `CMAKE_USER_MAKE_RULES_OVERRIDE_CXX` are required to use static C/C++ runtime, i.e ISAPI redirector will be built with static C/C++ runtime.
1. `D:\Users\Marat\Documents\work\cpp\pcre-8.42-x64-vs2015` is the directory where PCRE is installed. PCRE is built with static C/C++ runtime. 
1. `Visual Studio 14 2015 Win64` is [CMake generator for Visual Studio 2015 project](https://cmake.org/cmake/help/v3.1/generator/Visual%20Studio%2014%202015.html) building 64-bit binaries.

Current directory is the directory where generated Visual Studio 2015 project will be placed. 

```cmd
cmake -D CMAKE_USER_MAKE_RULES_OVERRIDE=D:\Users\Marat\Documents\work\cpp\tomcat-connectors-1.2.43-src\native\iis\cmake\cmake\static_c_runtime_overrides.cmake -D CMAKE_USER_MAKE_RULES_OVERRIDE_CXX=D:\Users\Marat\Documents\work\cpp\tomcat-connectors-1.2.43-src\native\iis\cmake\cmake\static_cxx_runtime_overrides.cmake -D PCRE_ROOT=D:\Users\Marat\Documents\work\cpp\pcre-8.42-x64-vs2015 -G "Visual Studio 14 2015 Win64" D:\Users\Marat\Documents\work\cpp\tomcat-connectors-1.2.43-src\native\iis\cmake
```

## Example for building of generated Visual Studio 2015 project

Example of command line for building release version of generated Visual Studio 2015 project (current directory is the directory where generated Visual Studio 2015 project is located):

```cmd
cmake --build . --config Release
```
 
`Release/isapi_redirect.dll` file is created if build is successful.