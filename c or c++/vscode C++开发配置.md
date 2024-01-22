# 一、vscode配置
## 1. tasks.json文件
**作用**：tasks.json 文件用于配置和自定义项目的构建任务。它可以帮助您在 VSCode 中运行常见的任务，例如编译、测试和格式化代码等。这些任务可以通过按下 Ctrl+Shift+B（Windows/Linux）或 Cmd+Shift+B（Mac）或在 VSCode 的“终端”菜单中选择“运行任务”来执行。

**配置解析**：
```json
{
  // tasks，是一个包含一个或多个任务的数组。
  "tasks": [
    {
      "type": "cppbuild", // 指定任务类型为 C++ 构建任务。
      "label": "C/C++: clang++ 生成活动文件", // 可以理解为任务的名称，为任务指定一个名为 "C/C++: clang++ 生成活动文件" 的标签，以便在 VSCode 中识别和引用此任务。
      "command": "/usr/bin/clang++", // 指定用于执行此任务的命令，这里使用的是 clang++ 编译器。
      // args 配置 command 指定的命令的参数。这里是在配置clang++编译指令的参数。
      "args": [
        "-fcolor-diagnostics", // 启用彩色诊断输出。
        "-fansi-escape-codes", // 启用 ANSI 转义码。
        "-g", // 生成调试信息，以便在调试时查看源代码。
        "${fileDirname}/*.cpp", // 编译当前文件所在目录下的所有 .cpp 文件。
        "-I","${fileDirname}/*.h",  // 将当前文件所在目录下的所有 .h 文件包含到编译过程中。
        "-o", // 指定输出文件名。
        "${fileDirname}/${fileBasenameNoExtension}" // 将输出文件保存到当前文件所在目录，并使用不带扩展名的当前文件名作为输出文件名。
      ],
      // 指定任务选项。
      "options": {
        "cwd": "${fileDirname}" // 设置任务的当前工作目录为当前文件所在的目录。
      },
      // 是一个包含一个或多个问题匹配器的数组，用于从任务输出中识别错误和警告。在这个例子中，使用了 $gcc 问题匹配器，它可以识别 GCC 和 Clang 编译器的错误和警告。
      "problemMatcher": [
        "$gcc"
      ],
      // 将任务分组。
      "group": {
        "kind": "build",
        "isDefault": true // 将此任务设置为默认构建任务。当在 VSCode 中按下 Ctrl+Shift+B（Windows/Linux）或 Cmd+Shift+B（Mac）时，将执行此任务。
      },
      "detail": "调试器生成的任务。" // 提供有关任务的详细信息。
    }
  ],
  "version": "2.0.0" // 指定 tasks.json 文件的版本
}
```
上面的配置文件定义了一个名为“C/C++: clang++ 生成活动文件”的任务，它使用 "/usr/bin/clang++" 命令编译名当前目录下的所有文件（${fileDirname}/*.cpp），并将其输出为名为 ${fileDirname} 的可执行文件。

## 2. launch.json文件
**作用**：launch.json 文件用于配置和自定义项目的调试任务。它可以帮助您在 VSCode 中设置断点、单步执行和查看变量等调试功能。这些调试任务可以通过按下 F5 或在 VSCode 的“运行”菜单中选择“启动调试”来执行。

**配置解析**：
```json
{
  // 是一个包含一个或多个调试配置的数组。
  "configurations": [
    {
      "name": "(lldb) 启动", // 为调试配置指定一个名为 "(lldb) 启动" 的名称，以便在 VSCode 中识别和引用此配置。
      "type": "cppdbg", // 指定调试器类型为 C++ 调试器。
      "request": "launch", // 指定调试请求类型为 "launch"，表示启动并调试程序。
      "program": "${fileDirname}/${fileBasenameNoExtension}", // 指定要调试的可执行程序。在这个例子中，它使用当前文件所在目录下的不带扩展名的当前文件名作为可执行程序。
      "args": [], // 指定传递给可执行程序的命令行参数。在这个例子中，没有传递任何参数。
      "stopAtEntry": false, // 指定是否在程序入口点（如 main 函数）处停止调试器。在这个例子中，设置为 false，表示调试器不会在程序入口点处停止。
      "cwd": "${workspaceFolder}", // 设置调试过程的当前工作目录为当前工作区文件夹。
      "environment": [], // 指定在调试过程中设置的环境变量。在这个例子中，没有设置任何环境变量。
      "externalConsole": true, // 指定是否在外部控制台中运行可执行程序。在这个例子中，设置为 true，表示在外部控制台中运行程序。
      "MIMode": "lldb", // 指定调试器后端模式为 LLDB。
      "preLaunchTask": "C/C++: clang++ 生成活动文件" // 在启动调试之前，自动执行名为 "C/C++: clang++ 生成活动文件" 的任务（在 tasks.json 中定义）来构建项目。
    }
  ],
  "version": "2.0.0" // 指定 launch.json 文件的版本。
}
```

# 二、VSCode下c++多文件夹项目编译调试（非makefile）
## 1. 项目目录划分
![Alt text](../pics/vscode%20c++%20非makefile%20目录划分.png)

## 2. 配置
修改tasks.json文件中的配置：
```json
"args": [
  "-fcolor-diagnostics", // 启用彩色诊断输出。
  "-fansi-escape-codes", // 启用 ANSI 转义码。
  "-g", // 生成调试信息，以便在调试时查看源代码。
  "${workspaceFolder}/source/*.cpp", // 编译当前文件所在目录下的所有 .cpp 文件。
  "-I","${workspaceFolder}/include/*.h",  // 将当前文件所在目录下的所有 .h 文件包含到编译过程中。
  "-o", // 指定输出文件名。
  "${workspaceFolder}/out/${fileBasenameNoExtension}" // 将输出文件保存到当前文件所在目录，并使用不带扩展名的当前文件名作为输出文件名。
],
```