## `Android.mk` 和 `CMakeLists.txt` 文件 
在Android开发中，`Android.mk` 和 `CMakeLists.txt` 文件分别是两种不同构建系统的配置文件。`Android.mk` 用于Android的原生Make系统——NDK build，而`CMakeLists.txt` 用于CMake构建系统。这两种系统用于编译和链接C/C++的原生代码（即NDK开发），它们都可以与Android Studio 配合使用。

下面简要介绍这两种文件的区别：

### Android.mk

1. **格式**：`Android.mk` 文件使用由Android NDK提供的专有语法，该语法基于GNU Make的格式进行了扩展。
2. **构建系统**：`Android.mk` 文件被用于NDK build系统，这是早期Android NDK提供的原生构建工具。
3. **特性**：NDK build通过编写`Android.mk`文件指导构建过程，它对于Android特有的构建步骤（如自动处理`.so` 文件的拷贝到正确的目录）提供了较好的支持。
4. **使用场景**：随着Android的发展，`Android.mk` 文件用得较少，而被Google推荐的CMake所取代，尤其是在新项目中。老的项目可能仍使用`Android.mk`，因为迁移到CMake可能会涉及大量工作。

### CMakeLists.txt

1. **格式**：`CMakeLists.txt` 文件根据CMake的语法编写，CMake是一个跨平台的构建系统。
2. **构建系统**：`CMakeLists.txt` 文件用于CMake构建系统，它是目前Android官方推荐用于构建原生库的工具。
3. **特性**：CMake相对于NDK build来说更现代，支持更复杂的构建场景，并且易于维护和扩展。CMake也更易于集成到其他构建系统中，在非Android环境下也得到了广泛的支持。
4. **使用场景**：对于新的Android NDK项目，CMake是首选的构建系统。它可以与Android Studio更好地集成，并提供更丰富的特性。

在Android Studio中，有两种支持的原生构建系统：NDK build和CMake。从Android Studio 2.2版开始，Google推荐使用CMake，原因是它更加灵活和强大。尽管Google不再积极推广NDK build，但它仍然可以在Android项目中使用，特别是对于一些遗留项目来说，仍然是必须的。

简而言之，`Android.mk` 文件和 `CMakeLists.txt` 文件都用于配置如何构建Android应用中的C/C++代码部分，但它们属于不同的构建系统，并拥有不同的语法和特性。随着Android官方对CMake的支持越来越好，`CMakeLists.txt` 得到了更广泛的使用。

## Android ndk-build 和 Make 构建系统的区别
用于Android的原生Make系统和普通的Make构建系统在基本原理上是一致的，它们都使用Make工具来解析构建脚本（Makefiles）并生成目标文件。但它们是为不同的开发环境和需求设计的，并包含了一些针对特定场景的扩展和工具。

### Android的原生Make系统（ndk-build）

1. **针对性**：它特别为Android平台上的原生代码（C和C++）构建而设计，适用于使用Android NDK（Native Development Kit）的项目。
2. **构建脚本**：使用自定义的`Android.mk` 文件来描述构建规则，其中包含了Android特定的宏和变量。
3. **工具链**：集成了Android特有的工具链，如`aapt`（Android Asset Packaging Tool）和`adb`（Android Debug Bridge），并为Android应用的打包、安装、调试等提供了支持。
4. **应用**：主要被用于构建Native活动和服务，以及生成可在Android设备上运行的动态/静态库（`.so` 和 `.a` 文件）。
5. **易用性**：为了支持跨平台构建和简化Android NDK的使用，特别强化了对Android环境的集成和在Android Studio中的易用性。

### 普通的Make构建系统

1. **通用性**：Make是一个通用的构建工具，它可以用于多种编程语言和平台，不仅限于Android。
2. **构建脚本**：使用通用的Makefile文件来定义构建规则，这些规则通常包括编译器选项、目标文件和依赖关系等。
3. **工具链**：可以配置使用任何编译器和工具链，例如GCC、Clang等。
4. **应用**：被广泛使用于各种软件项目中，无论是小型项目还是大型、复杂的系统。
5. **跨平台**：虽然Make能够被用于跨平台构建，但通常需要对Makefile进行手动修改以确保在不同的操作系统和环境中能够正确工作。

### 区别与联系

- **区别**：主要区别在于它们的用途和集成度。Android的原生Make系统是专门为Android应用的原生部分定制的，提供了特殊的工具和便利性。而通用Make系统更灵活，适用于多种编程项目和平台，但可能需要手工配置更多的细节。
- **联系**：两者的核心原理相同，都是通过解析Makefiles来执行构建任务，完成源代码到可执行文件或库的转换过程。

随着Android开发工具的演进，Google开始推荐使用CMake而不是原生Make系统来构建NDK项目，因为CMake提供了更高的灵活性和易用性，以及更好的IDE集成。不过，Android.mk和ndk-build在一些老项目和特定的用例中仍然有其用武之地。