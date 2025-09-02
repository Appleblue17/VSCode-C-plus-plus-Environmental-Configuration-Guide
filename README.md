在博客中浏览：[**GitHub**](https://github.com/Appleblue17/VSCode-C-plus-plus-Environmental-Configuration-Guide)，[**cnblogs**](https://www.cnblogs.com/Appleblue17/p/18399462)。

## Change Log

- 2024-09-06: 初始版本发布
- 2025-09-02: 修订文档，并重构了部分内容和排布

## Table of Contents

- [前言](#前言)
- [准备工作](#准备工作)
- [命令行编译](#命令行编译)
- [配置 VS Code](#配置-vs-code)
  - [方案一：VS Code Tasks + Launch](#方案一vs-code-tasks--launch)
  - [方案二：使用 Code Runner](#方案二使用-code-runner)
  - [拓展：Console Pauser](#拓展console-pauser)
- [如何 Debug](#如何-debug)
- [示例模板](#示例模板)
- [参考资料](#参考资料)
- [Contact & Contribution](#contact--contribution)

## 前言

本指南旨在让 VS Code 新玩家通过 VS Code 自带配置项与拓展的功能完成 C/C++ 的配置，并理解这些配置项的含义与工作方式。

本文中默认 VS Code 语言为英文，你也可以下载汉化拓展。

限于时间与篇幅，本文主要以文字形式呈现，几乎没有图片。如果你不理解或者卡在某一步，可以参考文末 [参考资料](#参考资料) 中的博客和官方文档，或者自行搜索其他资料。

***

## 准备工作

1. 下载 VS Code 并安装。链接：[VS Code](https://code.visualstudio.com/)

2. 下载并安装编译器。

对于 Windows 64 位系统，需要安装 MinGW。

   - 直接下载。链接：[MinGW64](https://www.mingw-w64.org/)

   - 或者下载 Dev-C++，里面自带 MinGW。链接：[Dev-C++](https://sourceforge.net/projects/orwelldevcpp/)

对于 Linux 系统，可直接通过包管理器安装，例如使用 `apt`：

```
sudo apt update
sudo apt install g++
```

一般而言，安装后会自动配置环境变量，无需手动设置。

3. 下载 C/C++ 拓展

   左侧边栏中找到 Extensions，搜索 `C/C++` 拓展并下载。

4. 设置环境变量（对于 Windows 系统）

   找到下载的 MinGW 文件目录，将 `...\MinGW64\bin` 加入环境变量。

   完成后，打开 shell（cmd 或 Powershell），输入 `g++ -v`，如果正常输出版本信息则配置成功。

***

### 命令行编译

在开始配置 VS Code 之前，最好先尝试用命令行编译程序，确保编译器功能正常。

新建一个文件（例如叫 `test.cpp`），写入测试程序。下面是一个测试程序示例：

```cpp
#include <iostream>
​using namespace std;
int main(){
    cout << "Hello World" << endl;
    int a,b;
    cout << "Please enter two numbers: " << endl;
    cin >> a >> b;
    cout << a+b << endl;
    return 0;
}
```

打开 shell，进入 `test.cpp` 所在目录。输入：

```sh
g++ test.cpp -o test
./test
```

如果成功，你应该能够看到程序的输出结果。

***

## 配置 VS Code

### 方案一：VS Code Tasks + Launch

使用 VS Code 自带的编译与调试功能。设置 `tasks.json` 来配置编译任务，设置 `launch.json` 来配置调试任务。

**优点**
  - VS Code 支持丰富，集成度高
  - 不同工作区间配置独立

**缺点**
  - 配置相对复杂，与直接设置编译命令相比不直观
  - 每个工作区都需要手动设置，不方便记忆

下面是具体配置流程。

***

1. **创建工作区**

   创建准备作为工作区的文件夹，在顶栏中选择 "File - Open Folder" 并选中它。

   正确完成后会进入工作区界面。

2. **创建测试文件**

   在左侧文件区顶栏点击 `New File` 按钮，创建 `.cpp` 文件，并写入 `c++` 程序，用于测试。

3. **配置** `tasks.json`

   `tasks.json` 本质上是一个脚本，用于自动化手动执行的 `g++ -g main.cpp -o main` 这样的命令。

   在顶栏中选择 `Terminal - Configure Tasks`；或者按下 `Ctrl + shift + P` 唤出命令面板，搜索 Task: `Configure Task` 并选择；或者直接按下 `F5` 或在右上角按钮选择运行/调试，VS Code 会提示你选择编译器。

   弹出选项中选择 `C/C++: g++.exe build active file`。此时 VS Code 会在工作区根目录中创建 `.vscode` 文件夹并在其中自动生成 `tasks.json` 文件。具体设置可参考文末的 [示例模板](#示例模板)。

   `tasks.json` 文件用于配置程序的编译选项（即生成可执行文件的过程）。下面是几个重要参数的解释：

   - `type`：指定任务的类型。`cppbuild` 是 VS Code 为 C/C++ 扩展定义的特定任务类型，它提供了更好的集成支持（如问题匹配）。
   - `label`：标签，可以理解为任务的名字，需要与 `launch.json` 中 `preLaunchTask` 的参数相同。
   - `command`：要执行的命令或脚本的路径。这里需要设置为编译器的路径，如 `...\MinGW64\bin\g++.exe` 或 `/usr/bin/g++` 等。
   - `args`：传递给编译器的参数，可以在这里配置编译选项。
   - `problemMatcher`: 问题匹配器，它会解析命令的输出（即 g++ 编译时产生的错误和警告），然后 VS Code 会将这些信息捕获并转换，直接显示在 Problems 面板中。你可以直接点击面板中的错误信息，光标就会跳转到源代码对应的行。
   - `group`：任务分组，`build` 代表这是一个构建任务，可以通过快捷键 `Ctrl+Shift+B` 来运行。还可以在此将其设置为**默认任务**，这样按下快捷键 `F5` 时就会自动编译，无需再选择执行哪个任务。
   - `details`: 任务的描述与注释，不会影响实际功能。
   
   这个配置相当于执行了以下脚本：
   
   ```sh
   cd /path/to/your/current/file # 对应 "cwd": "${fileDirname}"
   /usr/bin/g++ -fdiagnostics-color=always -g main.cpp -o main # 对应 "command" 和 "args"
   ```

4. **测试编译与运行**

在配置好 `tasks.json` 后，你应该已经可以编译并运行程序。

打开测试程序，在右上角的按钮选择 `Run C/C++ File` 或者按下 `F5` 快捷键，程序应当在下方的终端输出。

5. **配置** `launch.json`

   `launch.json` 的功能是告诉 VS Code 的调试器如何启动你的程序。这是启用断点、单步执行、查看变量等高级调试功能的核心。

   在顶栏中选择 `Run - Add Configuration`；或者唤出命令面板后再搜索 `Run: Add Configuration`，弹出选项中选择 `C++ (GDB/LLDB)`。此时 VSCode 会自动创建 `launch.json` 文件。窗口下方会出现 `Add Configuration` 蓝色按钮，点击并选中 `C/C++: gdb (Launch)`。

   在 `setupCommands` 下面的中括号后添加 `,`，再在下一行插入 `"preLaunchTask": "C/C++: g++.exe build active file"`，这里的参数要与 `tasks.json` 中的 `label` 参数相同。（可见文末[示例模板](#示例模板)）

   解释：`preLaunchTask` 参数会在执行 运行/调试 前先执行编译，

   `launch.json` 文件用于配置程序的调试选项（即执行可执行文件的过程）。下面是几个重要参数的解释：

   - `type`：配置类型，对于 C/C++ 是 `cppdbg`，由 cpptools 提供。
   - `request`：可以为 `launch`（启动程序）或 `attach`（附加到已运行的程序）。
   - `program`：用于 运行/调试 的可执行文件的路径，建议设为 `"${fileDirname}/${fileBasenameNoExtension}.exe"` 或 `"${fileDirname}/${fileBasenameNoExtension}"`（意为当前路径）。
   - `externalConsole`：是否启用外部终端。新版本 VS Code 可能用不了内部终端，建议改为 `True`。
   - `miDebuggerPath`：调试器（如 GDB）的路径，设为 `"gdb.exe"` 或 `"gdb"`。
   - `"preLaunchTask"`: 在**启动调试之前**，自动执行 `tasks.json` 中哪个 `"label"` 的任务，这确保了你在调试前总是拥有最新的可执行文件。如果配置错误就无法进行编译，只用已有的可执行文件进行运行/调试。

7. **测试调试功能**

   为了调试，首先需要设置断点。点击行号左侧的空白区域即可设置断点，再次点击可取消。

   点击右上角的按钮（`Debug C/C++ File`），或者点击左侧边栏中的 `Run and Debug`，或者按 `F5` 快捷键进行调试。

> [!NOTE]
> 默认配置下，程序结束时窗口会直接关闭。解决办法是在程序末尾设置断点，或者写 `system("pause");` 中断。

***

### 方案二：使用 Code Runner

使用 Code Runner 拓展进行编译和运行。快捷键为 `Ctrl+Alt+N`。

**优点**
- 灵活直观，配置简单，极致轻量化
- 支持多种语言，配置一目了然
- 全局应用，不需要重复配置

**缺点**
- 只能运行而不支持调试
- 不支持对每个工作区单独配置
- Code Runner 似乎是通过 Output 面板来显示运行结果的，不支持输入

Code Runner 实际上就是在终端中输入一行命令，因此只需要配置命令即可。

我现在用的 Linux 配置为：

```sh
"cpp": "cd $dir && g++ $fileName -o $fileNameWithoutExt -std=c++14 && time ./$fileNameWithoutExt < in.txt"
```

***

### 拓展：Console Pauser

前面提到程序结束后窗口会直接关闭。在 Windows 系统下，可以使用 consolePauser 解决这个问题。consolePauser 是一个小巧的版本控制台暂停程序，可以在程序结束后保持窗口打开。

1. 下载并配置 consolePauser。

   Dev-c++ 中自带 consolePauser，也可以在网上下载。记得将 consolePauser 所在的文件夹加入环境变量中。

2. 找到配置文件

   下载 Code Runner 拓展，在商店界面找到 `Extension Settings`，找到 `Executor Map` 设置，并进入 `settings.json` 文件。

3. 修改配置文件

   找到 `cpp`，并把那一行修改为：

```json
"cpp": "cd $dir && g++ $fileName -o $fileNameWithoutExt -std=c++14 && start consolePauser $fileNameWithoutExt",
```

   以下是详细解释：

   - `cd $dir`：打开当前目录。
   - `g++ $fileName -o $fileNameWithoutExt -std=c++14`：编译，可以在这里添加编译选项。
   - `start consolePauser $fileNameWithoutExt`：在外部终端中运行 consolePauser。

***

## 如何 Debug

1. 测试能否编译

   打开程序所在的文件夹，在文件夹中打开终端（或者在 VS Code 内置终端中）。假设程序名为 `A.cpp`，输入 `g++ A.cpp -o A` 命令，观察是否能够成功编译。

2. 测试 VS Code 中能否编译

   在界面右上角的三角符号旁打开下拉菜单，选择 `Run C/C++ File`，这样 VSCode 只会执行编译和运行，而不进行调试。

3. 仔细查看错误信息

   如果报错，VS Code 会很人性化地给出报错信息。一般来说仔细看看就能知道哪里出错了。

   如果出现 `xxx does not exist`，很可能是文件路径写错了。

   VS Code 下方的终端中会显示运行的指令，可以看看与预期是否相符，再去相应的 `.json` 文件里修改。

***

## 示例模板

- `tasks.json` 文件

```json
{
   "version": "2.0.0",
   "tasks": [
      {
         "type": "cppbuild",
         "label": "C/C++: g++.exe build active file", //要和 launch.json 的 preLaunchTask 相对应
         "command": "g++",
         "args": [
            "-fdiagnostics-color=always",
            "-g",
            "${file}",
            "-o",
            "${fileDirname}\\${fileBasenameNoExtension}.exe"
         ],
         "options": {
            "cwd": "${fileDirname}"
         },
         "problemMatcher": [
            "$gcc"
         ],
         "group": "build",
         "detail": "Build C++ file"
      }
   ]
}
```

- `launch.json` 文件

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch", // 配置名称，将会在启动配置的下拉菜单中显示
            "type": "cppdbg", // 配置类型，对于C/C++可认为此处只能是cppdbg，由cpptools提供；不同编程语言不同
            "request": "launch", // 可以为launch（启动）或attach（附加）
            "program": "${fileDirname}/${fileBasenameNoExtension}.exe", // 将要进行调试的程序的路径；Linux 下去掉 .exe
            "args": [], // 程序调试时传递给程序的命令行参数，一般设为空
            "stopAtEntry": false, // 设为true时程序将暂停在程序入口处，相当于在main上打断点
            "cwd": "${fileDirname}", // 调试程序时的工作目录
            "environment": [], // 环境变量
            "externalConsole": true, // 使用单独的cmd窗口，与其它IDE一致；为false时使用内置终端
            "MIMode": "gdb", // 指定连接的调试器，可以为gdb或lldb
            "miDebuggerPath": "gdb.exe", // 调试器路径，Linux 下改为 gdb
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "Set Disassembly Flavor to Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C/C++: g++.exe build active file" // 调试前执行的任务，一般为编译程序。与tasks.json的label相对应
        }
    ]
}
```

***

## 参考资料

1. [全网最详细保姆级VSCode配置C/C++教程（基于官方文档）- Mr_Luka
](https://blog.csdn.net/qq_50813669/article/details/124461328)。

2. [Integrate with External Tools via Tasks - VS Code](https://code.visualstudio.com/docs/debugtest/tasks#_custom-tasks)

3. [Using C++ on Linux in VS Code - VS Code](https://code.visualstudio.com/docs/cpp/config-linux?originUrl=%2Fdocs%2Fcpp%2Fconfig-linux)

4. [Using GCC with MinGW - VS Code](https://code.visualstudio.com/docs/cpp/config-mingw?originUrl=%2Fdocs%2Fcpp%2Fconfig-linux)

## Contact & Contribution

如果你有任何问题或建议，或是发现了文中的错误，欢迎在此仓库下提 Issues，或是提交 Pull Request 以贡献此文档！❤

