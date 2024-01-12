#### 1. find_package
`find_package`是CMake中的一个命令，用于在系统中查找并加载外部项目或库。它的基本语法如下：

```cmake
find_package(<PackageName> [version] [EXACT] [QUIET] [MODULE] [REQUIRED] [[COMPONENTS] [components...]] [OPTIONAL_COMPONENTS components...])
```

- `<PackageName>`：需要查找的包的名称。

- `version`：需要查找的包的最低版本号。

- `EXACT`：要求找到的包的版本号必须与指定的版本号完全匹配。

- `QUIET`：不显示任何消息，包括错误消息。

- `MODULE`：只使用CMake的模块模式查找包，不使用配置模式。

- `REQUIRED`：如果CMake无法找到包，或者找到的包的版本号低于指定的最低版本号，那么CMake会报错并停止配置过程。

- `COMPONENTS`：需要加载的包的组件列表。

- `OPTIONAL_COMPONENTS`：如果可能，会加载的包的组件列表。

返回值: 如果`find_package(<PackageName> REQUIRED)`成功执行，那么它将定义一个变量`<PackageName>_FOUND`，值为真（true），以表明线程库已经找到。此外，还可以定义其他与线程相关的变量，如`<PackageName>_LIBRARY`、`<PackageName>_INCLUDE_DIR`等。这些变量可供之后的代码使用。

例如，以下命令查找版本号至少为3.1的OpenGL库，并加载GLUT和GLEW组件：

```cmake
find_package(OpenGL 3.1 REQUIRED COMPONENTS GLUT GLEW)
```

在这个命令中，如果CMake无法找到版本号至少为3.1的OpenGL库，或者无法加载GLUT和GLEW组件，那么CMake会报错并停止配置过程。

注意，`find_package`命令的行为会根据包的查找脚本的不同而不同。一些包可能不支持版本号或组件的查找，或者可能有额外的选项。你可以查看包的查找脚本，或者查看包的文档，以获取更多信息。

#### 2.add_library
`add_library`是CMake中的一个命令，用于创建和管理库。它的基本语法如下：

```cmake
add_library(<name> [STATIC | SHARED | MODULE] [EXCLUDE_FROM_ALL] source1 [source2 ...])
```

- `<name>`：库的名称。
- `STATIC | SHARED | MODULE`：指定库的类型。STATIC表示静态库，SHARED表示动态库，MODULE表示作为loadable模块的库。如果没有指定类型，CMake会使用`BUILD_SHARED_LIBS`变量的值来决定。
- `EXCLUDE_FROM_ALL`：这是一个可选参数，如果指定了，那么这个库不会被默认构建目标包含。
- `source1 [source2 ...]`：库的源文件列表。

例如，以下命令创建了一个名为`mylib`的静态库，源文件是`source1.cpp`和`source2.cpp`：

```cmake
add_library(mylib STATIC source1.cpp source2.cpp)
```

你也可以先创建一个库，然后再添加源文件：

```cmake
add_library(mylib STATIC)
target_sources(mylib PRIVATE source1.cpp source2.cpp)
```

在这个例子中，`target_sources`命令被用来给`mylib`库添加源文件。

注意，`add_library`命令必须在`add_executable`命令之前调用，因为库需要在链接可执行文件时已经存在。

在CMake中，`add_library`命令的`INTERFACE`参数用于创建一个接口库。接口库本身并不直接构建任何二进制文件，而是用于定义一组构建规则和依赖，这些规则和依赖可以被其他目标（如库或可执行文件）使用。

例如，以下命令创建了一个名为`simple-web-server`的接口库：

```cmake
add_library(simple-web-server INTERFACE)
```

你可以使用`target_include_directories`，`target_compile_definitions`，`target_compile_options`，`target_link_libraries`等命令来为接口库添加属性。这些属性会被链接到接口库的其他目标自动使用。

例如，以下命令将一个目录添加到`simple-web-server`接口库的公共包含目录中：

```cmake
target_include_directories(simple-web-server INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
```

然后，当你链接到`simple-web-server`库时，这个目录会被添加到编译器的包含路径中：

```cmake
target_link_libraries(my_executable PRIVATE simple-web-server)
```

在这个例子中，`my_executable`目标会自动获取`simple-web-server`接口库的所有属性，包括其包含目录。

#### 3.cmake命令
cmake -H. -Bbuild : 用来生成构建系统的配置文件。其中，-H. 表示将当前目录作为源代码目录，-Bbuild 表示将生成的构建系统文件放在名为 "build" 的目录中。

cmake --build build : 用来执行构建过程。它会在 "build" 目录中查找构建系统文件，并根据配置文件中的指令来编译、链接和构建项目。