https://zhuanlan.zhihu.com/p/500002865
# step 1: 构建最小项目
最基本的项目是将一个源代码文件生成可执行文件。
```cmake
cmake_minimum_required(VERSION 3.15)

# set the project name
project(Tutorial)

# add the executable
add_executable(Tutorial tutorial.cpp)
```

`cmake_minimum_required`指定使用CMake的最低版本号;
`project`指定项目名称;
`add_executable`用来生成可执行文件，需要指定生成可执行文件的名称和相关源文件。

注意，此实例在CMakeList。txt使用小写命令。
CMake支持大写、小写和混合大小写命令。

## 构建、编译和运行
现在就可以构建和运行我们的项目了，就是先运行cmake命令来构建项目，然后使用你选择的编译工具进行编译。

```
step1/
    build/
    CMakeLists.txt
    tutorial.cpp
```

## 外部构建与内部构建
这里创建了一个build目录存放编译产物，可以避免编译产物与代码文件混在一起，这种叫做外部构建。

还有一种内部构建，即直接在项目根目录下进行构建系统与编译。

内部构建会使得项目文件很混乱，一般直接用外部构建即可。

# step 2: 优化CMakeLists.txt文件
## set与PROJECT_NAME
指定项目名后，后面可能会有多个地方用到这个项目名，如果更改了这个名字，就要改多个地方，比较麻烦，那么可以用`PROJECT_NAME`来表示项目名:

```cmake
add_executable(${PROJECT_NAME} tutorial.cpp)
```

生成可执行文件需要指定相关的源文件，如果有多个，那么就用空格隔开，比如:

```cmake
add_executable(${PROJECT_NAME} a.cpp b.cpp c.cpp)
```

我们也可以用一个变量来表示这多个源文件:

```cmake
set(SRC_LIST a.cpp b.cpp c.cpp)
add_executable(${PROJECT_NAME} ${SRC_LIST})
```
set命令指定SRC_LIST变量来表示多个源文件。

CMakeLists.txt文件就可以变成如下所示:

```cmake
cmake_minimum_required(VERSION 3.15)

# set the project name
project(Tutorial)

SET(SRC_LIST tutorial.cpp)

# add the executable
add_executable(${PROJECT_NAME} ${SRC_LIST})
```

## 添加版本号和配置头文件
我们可以在 CMakeLists.txt 为可执行文件和项目提供一个版本号。首先，修改 CMakeLists.txt 文件，使用 project 命令设置项目名称和版本号。

```cmake
cmake_minimum_required(VERSION 3.15)

# set the project name and version
project(Tutorial VERSION 1.0)

configure_file(TutorialConfig.h.in TutorialConfig.h)
```

然后，配置头文件将版本号传递给源代码：

```cmake
configure_file(TutorialConfig.h.in TutorialConfig.h)
```

由于TutorialConfig.h文件会被自动写入build目录，因此必须将该目录添加到搜索头文件的路径列表中。将以下行添加到CMakeList.txt文件的末尾:

```cmake
target_include_directories(${PROJECT_NAME} PUBLIC
                           ${PROJECT_BINARY_DIR}
                           )
```

`PROJECT_BINARY_DIR `表示当前工程的二进制路径，即编译产物会被存放到该路径，此时`PROJECT_BINARY_DIR`就是build所在路径

然后创建`TutorialConfig.h.in`文件，包含以下内容:

```cmake
// the configured options and settings for Tutorial
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

当使用CMake构建项目后，会在build中生成一个TutorialConfig.h文件，内容如下:

```cmake
// the configured options and settings for Tutorial
#define Tutorial_VERSION_MAJOR 1
#define Tutorial_VERSION_MINOR 0
```

下一步在tutorial.cpp中包含头文件TutorialConfig.h，最后通过以下代码打印出可执行文件的名称和版本号。

## 指定C++标准
接下来将`step1/tutorial.cpp`源码中的`atof`替换为`std::stod`，这是C++11的特性，并删除`#include<cstdlib>`

```c++
const double inputValue = std::stod(argv[1]);
```

在CMake中支持特定C++标准的最简单方法是使用`CMAKE_CXX_STANDARD`标准变量。在CMakeList.txt中设置`CMAKE_CXX_STANDARD`为11,`CMAKE_CXX_STANDARD_REQUIRED`设置为True。确保在`add_executable`命令之前添加`CMAKE_CXX_STANDARD_REQUIRED`命令。

```cmake
cmake_minimum_required(VERSION 3.15)

# set the project name and version
project(${PROJECT_NAME} VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

需要注意的是，如果你的gcc编译器版本够高，也可以不用指定C++版本为11。从 GCC 6.1 开始，当不指定任何版本 C++ 标准时，默认版本是 C++ 14，如果你想用 C++17 的语言，还是需要指定的。

修改完成后，需要对代码进行编译`cmake ../`,此时可以不用进行项目构建。

此时目录结构为:

```
step2/
    build/
    CMakeLists.txt
    tutorial.cpp
    TutorialConfig.h.in
```

# step 3: 添加库
现在我们将向项目中添加一个库，这个库包含计算数字平方根的实现，可执行文件使用这个库，而不是编译器提供的标准平方根函数。

我们把库放在名为`MathFunctions`的子目录中。此目录包含头文件MathFunctions.h 和源文件 mysqrt.cpp。源文件有一个名为mysqrt的函数，它提供了与编译器sqrt函数类似的功能，MathFunctions.h则是该函数的声明。

在MathFunctions 目录下创建一个CMakeList.txt文件，并添加以下一行:

```cmake
# MathFunctions/CMakeLists.txt
add_library(MathFunctions mysqrt.cpp)
```

表示添加一个叫MathFunctions的库文件。

CMake中的target有可执行文件和库文件，分别使用`add_executable`和`add_library`命令生成，除了指定生成的可执行文件名/库文件名，还需要指定相关的源文件。

此时的文件结构为:
```
step3/
    build/
    MathFunctions/
        CMakeLists.txt
        MathFunctions.h
        mysqrt.cpp
    CMakeLists.txt
    tutorial.cpp
    TutorialConfig.h.in
```

为了使用MathFunctions 这个库，我们将在顶级CMakeLists.txt文件中添加一个`add_subdirectory(MathFunctions)`命令指定库所在子目录，该子目录下应包含CMakeLists.txt文件和代码文件。

可执行文件要使用库文件，需要能够找到库文件和对应的头文件，可以分别通过`target_link_libraries`和`target_include_directories`来指定。

使用`target_link_libraries`将新的库文件添加到可执行文件中;
使用`target_include_directories`将MathFunctions添加为头文件目录，添加到Tutorial目标上，以便mysqrt.h可以被找到。

顶级CMakeLists.txt的最后几行如下所示:

```cmake
# add the MathFunctions library
add_subdirectory(MathFunctions)

# add the executable
add_executable(${PROJECT_NAME} tutorial.cpp)

target_link_libraries(${PROJECT_NAME} PUBLIC MathFunctions)

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(${PROJECT_NAME} PUBLIC
                           ${PROJECT_BINARY_DIR}
                           ${PROJECT_SOURCE_DIR}/MathFunctions
                           )
```

MathFunctions库就算添加完成了，接下来就是在主函数使用该库中的函数，现在tutorial.cpp文件中添加头文件:

```c++
#include "MathFunctions.h"
```

然后使用`mysqrt`函数即可:

```c++
const double outputValue = mysqrt(inputValue);
```

# step 4: 将库设置为可选项
现在将 MathFunctions 库设为可选的，虽然对于本教程来说，没有必要这样做，但对于较大的项目来说，这种情况很常见。

第一步是向顶级CMakeList.txt文件添加一个选项。
```cmake
option(USE_MYMATH "Use tutorial provided math implementation" ON)
```

`option`表示用户可以选择的选项。命令格式为:`option(<variable> "description [initial value])`。

`USE_MYMATH`这个选项缺省值为ON，用户可以更改这个值。此设置将存储在缓存中，以便用户不需要在每次构建项目时设置该值。

下一个更改是使MathFunctions库的构建和链接成为条件。于是创建一个if语句，该语句检查选项`USE_MYMATH`的值。

```cmkae
if(USE_MYMATH)
    add_subdirectory(MathFunctions)
    list(APPEND EXTRA_LIBS MathFunctions)
    list(APPEND EXTRA_INCLUDES ${PROJECT_SOURCE_DIR}/MathFunctions)
endif()

# add the executable
add_executable(${PROJECT_NAME} tutorial.cpp)

target_link_libraries(${PROJECT_NAME} PUBLIC ${EXTRA_LIBS})

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h

target_include_directories(${PROJECT_NAME} PUBLIC
                           ${PROJECT_BINARY_DIR}
                           ${EXTRA_INCLUDES}
                           )
```

在if块中，有`add_subdirectory`命令和`list`命令，`APPEND`表示将元素`MathFunctions`追加到列表`EXTRA_LIBS`中，将元素`${PROJECT_SOURCE_DIR}/MathFunctions`追加到列表`EXTRA_INCLUDES`中。`EXTRA_LIBS`存储MathFunctions库，`EXTRA_INCLUDES`存储MathFunctions头文件。

变量EXTRA_LIBS用来保存需要链接到可执行程序的可选库，变量EXTRA_INCLUDES用来保存可选的头文件搜索路径。这是处理可选组件的经典方法，将在下一步介绍现代方法。

接下来对源代码进行修改。首先，在tutorial.cpp中包含MathFunctions.h头文件:

```c++
#ifdef USE_MYMATH
    #include "MathFunctions.h"
#endif
```

然后，还在tutorial.cpp中，使用`USE_MYMATH`选择使用哪个平方根函数:

```c++
#ifdef USE_MYMATH
    const double outputValue = mysqrt(inputValue);
#else
    const double outputValue = sqrt(intputValue);
#endif
```

因为源代码使用USE_MYMATH宏，可以用下面的行添加到tutorialconfig.h.in文档中:

```cmake
// TutorialConfig.h.in
# cmakedefind USE_MYMATH
```

现在使用cmake命令构建项目，并运行生成的Tutorial可执行文件。

默认调用 mysqrt 函数，也可以在构建项目时指定 USE_MYMATH 的值为 OFF：

```
> cmake -DUSE_MYMATH=OFF ..
> cmake --build .
```

# step 5: 添加库的使用要求
使用要求会对库或可执行程序的链接、头文件包含命令行提供了更好的控制，也是CMake中目标的传递目标属性更加可控。利用使用要求的主要命令是:
> + target_compile_definitions()
> + target_compile_options()
> + target_include_directories()
> + target_link_libraries()

现在重构一下step4中的代码，使用更加现代的CMake方法来包含MathFuncitons库的头文件。

首先声明，链接MathFunctions库的任何可执行文件/库文件都需要包含MathFunctions目录作为头文件路径，而MathFunctions本身不需要包含，这被称为`INTERFACE`使用要求

`INTERFACE`是指用户需要，开发者不需要的那些东西。在MathFunctions/CMakeLists.txt最后添加:

```cmake
# MathFunctions/CMakeLists.txt
target_include_directories(MathFunctions
            INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
            )
```

`CMAKE_CURRENT_SOURCE_DIR`表示MathFunctions库所在目录。

现在我们已经为MathFunctions指定了使用要求`INTERFACE`，那么可以从顶级CMakeList.txt中删除`EXTRA_INCLUDES`变量的相关使用:

```cmake
if(USE_MYMATH)
    add_subdirectory(MathFunctions)
    list(APPEND EXTRA_LIBS MathFunctions)
    list(APPEND EXTRA_INCLUDES ${PROJECT_SOURCE_DIR}/MathFunctions) # 删除此行
endif

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(${PROJECT_NAME} PUBLIC
                           ${PROJECT_BINARY_DIR}
                           ${EXTRA_INCLUDES}   # 删除此行
                           )
```

现在只要是链接了MathFunctions库，就会自动包含MathFunctions所在目录的头文件，简介而优雅。

这里补充两个知识点:

1、使用要求除了`INTERFACE`,还有`PRIVATE`和`PUBLIC`。`INTERFACE`表示用户需要，开发者不需要，`PRIVATE`表示用户不需要，开发者需要，`PUBLIC`表示用户和开发者都需要。

2、 这里使用add_library命令生成的MathFunctions库其实是静态链接库。动态库和静态库的区别是:静态库在连接阶段会被链接到最终目标中(比如可执行程序)，缺点是同一个静态库如果被不同的程序引用，那么内存中会存在这个静态库函数的多份拷贝。动态库在链接阶段不会被拷贝到最终目标中，程序在运行阶段才会加载这个动态库。所以多个程序就算引用了同一个动态库，内存中也只存在一份动态库函数的拷贝。

# step 6: build目录介绍
在文本中，都是创建了一个build用来存放cmake构建和编译的产物，这里简单说下里面有些什么东西。

```
build/
    CMakeCache.txt
    CMakeFiles/
    cmake_install.cmake
    Makefile
    Tutorial.exe
    TutorialConfig.h
    MathFunctions/
```

其中`Makfile`是根据顶级CMakeLists.txt生成的构建文件，通过该文件可以对整个项目进行编译。

`Tutorial.exe`就是生成的可执行文件，通过该文件运行程序。

`TutorialConfig.h`是用于配置信息的头文件，是cmake根据TutorialConfig.h.in文件自动生成的。

还有个MathFunctions文件夹:
```
MathFunctions/
    CMakeFiles/
    cmake_install.cmake
    Makefile
    libMathFunctions.a
```

其中`Makefile`是cmake根据MathFunctions目录下的CMakeLists.txt生成的构建文件。

`libMathFunctions.a`则是MathFunctions静态链接库，可执行文件会通过这个库调用mysqrt函数。