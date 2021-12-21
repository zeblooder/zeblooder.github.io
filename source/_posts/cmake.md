---
title: cmake相关
date: 2021-12-17 00:18:30
tags: 工具
---

## CMake命令

    cmake -B build      # 生成构建目录
    cmake --build build # 执行构建
    ./build/xx			# xx是可执行程序名

## CMakeLists.txt

### 主项目

* cmake_minimum_required(VERSION 3.9)
* project(<PROJECT-NAME>
          [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
          [DESCRIPTION <project-description-string>]
          [HOMEPAGE_URL <url-string>]
          [LANGUAGES <language-name>...])
* * 指定项目名，一般是最终文件名
* add_subdirectory

  * 添加一个lib的文件夹，参数是lib所在文件夹名字
  * 单个参数，多个lib就写多行。
* add_executable

  * 添加可执行文件
  * e.g.`add_executable(${PROJECT_NAME} main.cpp)`
  * 源代码可以是多个，空格分开
* include_directories

  * 将指定目录添加到编译器的头文件搜索路径之下，指定的目录被解释成当前源码路径的相对路径。

  * 一般是工程本身有多个源文件和头文件在include目录或其他目录中，使用这个命令后就能直接引用到需要的文件。

### 库

用户可以自定义一个库，将相关文件放在一个文件夹下，以math为例。

* add_library(<name> [STATIC | SHARED | MODULE]            [EXCLUDE_FROM_ALL]            [source1] [source2 ...])
  * 添加一个库到工程中（库所在的工程），库的类型是`STATIC(静态库)`/`SHARED(动态库)`/`MODULE(模块库)`之一。
  * add_library(libmath STATIC math.cpp math1.cpp)
* target_include_directories(libmath PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
  * 将头文件公开使得外部的工程可以include
* target_link_libraries

  * 链接lib
  * e.g. `target_link_libraries(${PROJECT_NAME} libmath)`
  * 库可以多个，空格分开
* find_package
* if-endif()
* message(SEND_ERROR "error msg!")
* target_compile_definitions(libanswer PRIVATE WOLFRAM_APPID="${WOLFRAM_APPID}")

  * 代码里可以以宏的形式使用
  * cmake里用-DWOLFRAM_APPID 代入
* target_compile_features(libanswer INTERFACE cxx_std_20)
* add_custom_command
  * 添加自定义命令

```cmake
add_custom_command(
  OUTPUT out.c
  COMMAND someTool -i ${CMAKE_CURRENT_SOURCE_DIR}/in.txt
                   -o out.c
  DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/in.txt
  VERBATIM)
add_library(myLib out.c)
```

* add_custom_target
  * 命令行里用cmake命令时可以指定--target来执行命令

例子：https://github.com/zeblooder/ComputerGraphics/blob/master/final/CMakeLists.txt

参考：https://github.com/richardchien/modern-cmake-by-example

