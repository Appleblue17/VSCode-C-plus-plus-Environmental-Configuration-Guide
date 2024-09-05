

## 前言

本指南旨在让 VSCode 新玩家通过 VSCode 自带配置项与拓展的功能完成 C/C++ 的配置，并理解这些配置项的含义与工作方式。

当然如果你想要以最快的方式完成配置，也可以直接复制文末的模板，在工作区根目录下创建 `.vscode` 文件夹并放进去。（不保证一定成功哦）

本文中默认 VSCode 语言为英文，你也可以下载汉化拓展，但是我并不推荐。

限于时间与篇幅，本文主要以文字形式呈现，几乎没有图片。如果你不理解或者卡在某一步，可以参考一篇 csdn 教程：[全网最详细保姆级VSCode配置C/C++教程（基于官方文档）- Mr_Luka
](https://blog.csdn.net/qq_50813669/article/details/124461328)。

如果你有任何问题或者发现了本文的错误，欢迎给我发邮件或在 cnblogs 或在 github 中留言。

## 准备工作

1. 下载 VSCode 并安装。链接：[VSCode](https://code.visualstudio.com/)

2. 下载 MinGW（编译器）。对于 Windows 64 位系统：

    - 直接下载。链接：[MinGW64](https://www.mingw-w64.org/)

    - 或者下载 Dev-C++，里面自带 MinGW。链接：[Dev-C++](https://sourceforge.net/projects/orwelldevcpp/)

   对于 Linux 系统，详见下文。

## 配置流程（Windows 64 位）

### 方法一

使用 VSCode 自带的编译并调试功能。注意：是调试而不是只运行（虽然你也可以用调试代替运行，但调试会比运行更慢）。快捷键为 `F5`。

调试：点击行号左侧的红点以设置断点，程序运行时会执行到下一个断点并停下。

缺点：如果程序结束，窗口会直接关闭。解决办法是在程序末尾设置断点，或者写 `system("pause");` 中断。

下面是具体配置流程。

1. 下载 C/C++ 拓展

   左侧边栏中找到 Extensions，搜索 `C/C++` 拓展并下载。

2. 设置环境变量

   找到下载的 MinGW 文件目录，将 `...\MinGW64\bin` 加入环境变量。

   完成后，打开 cmd（或者 Powershell），输入 `gcc -v`，如果正常输出版本信息则配置成功。

3. 创建工作区。
   
   创建准备作为工作区的文件夹，在顶栏中选择 "File - Open Folder" 并选中它。

   正确完成后会进入工作区界面。

4. 创建测试文件

   在左侧文件区顶栏点击 `New File` 按钮，创建 `.cpp` 文件，并写入 `c++` 程序，用于测试。

   下面是一个测试程序示例：

```cpp
#include <iostream>
​using namespace std;
int main(){
    cout << "Hello World" << endl;
    int a,b;
    cin>>a>>b;
    cout<<a+b<<endl;
    return 0;
}
```

5. 配置 `tasks.json`

   在顶栏中选择 `Terminal - Configure Tasks`，弹出选项中选择 `C/C++: g++.exe build active file`。

   此时 VSCode 会在工作区根目录中创建 `.vscode` 文件夹并在其中创建 `tasks.json` 文件。

   `tasks.json` 文件用于配置程序的编译选项（即生成 `.exe` 可执行文件的过程）。下面是几个重要参数的解释：

   - `label`：标签，可以理解为任务的名字，需要与 `launch.json` 中 `preLaunchTask` 的参数相同。

   - `command`：编译器，需要修改为 `...\MinGW64\bin\g++.exe`。

   - `args`：执行命令，可以在这里配置编译选项。



6. 配置 `launch.json`

   在顶栏中选择 `Run - Add Configuration`，弹出选项中选择 `C++ (GDB/LLDB)`。

   此时 VSCode 会自动创建 `launch.json` 文件。窗口下方会出现 `Add Configuration` 蓝色按钮，点击并选中 `C/C++: gdb (Launch)`。

   在 `setupCommands` 下面的中括号后添加 `,`，再在下一行插入 `"preLaunchTask": "C/C++: g++.exe build active file"`，这里的参数要与 `tasks.json` 中的 `label` 参数相同。（可见文末示例）

   解释：`preLaunchTask` 参数会在执行 运行/调试 前先执行编译，如果配置错误就不会进行编译，只进行 运行/调试。

   `launch.json` 文件用于配置程序的运行/调试选项（即执行 `.exe` 可执行文件的过程）。下面是几个重要参数的解释：

   - `program`：用于 运行/调试 的可执行文件的路径，建议设为 `"${fileDirname}/${fileBasenameNoExtension}.exe"`（意为当前路径）。

   - `externalConsole`：是否启用外部终端。貌似新版本 VSCode 用不了内部终端，建议改为 `True`。

   - `miDebuggerPath`：调试器路径，建议 `"gdb.exe"`。

7. 尝试运行

   点击右上角的按钮（`Debug C/C++ File`），或者点击左侧边栏中的 `Run and Debug`，或者按 `F5` 快捷键进行调试。

### 方法二

使用 Code Runner 拓展进行编译和运行。快捷键为 `Ctrl+Fn+N`。

缺点：只能运行而不能调试。

Code Runner 实际上就是在终端中输入一行命令，因此只需要配置命令即可。

以下是配置流程：

1. 下载并配置 consolePauser。

   consolePauser 是一个小巧的版本控制台暂停程序，可以解决程序结束后窗口直接关闭的问题。

   Dev-c++ 中自带 consolePauser，你也可以在网上下载。

   将 consolePauser 所在的文件夹加入环境变量中。

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

## 配置流程（Linux）

与 Windows 基本相同，只是编译器安装方式不同。VSCode 会给你指示：

![](pic.png)

后续步骤基本相同，除了注意编译出的文件名应当去掉 `.exe` 后缀（虽然不去掉也能运行，后缀名不影响文件内容）。

## 如何 Debug

1. 测试能否编译

   打开程序所在的文件夹，在文件夹中打开终端（或者在 VSCode 内置终端中）。假设程序名为 `A.cpp`，输入 `g++ A.cpp -o A` 命令，观察是否能够成功编译。

2. 测试 VSCode 中能否编译

   在界面右上角的三角符号旁打开下拉菜单，选择 `Run C/C++ File`，这样 VSCode 只会执行编译而不会运行/调试。

3. 仔细查看错误信息

   如果报错，VSCode 会很人性化地给出报错信息。一般来说仔细看看就能知道哪里出错了。

   如果出现 `xxx does not exist`，很可能是文件路径写错了。

   VSCode 下方的终端中会显示运行的指令，可以看看与预期是否相符，再去相应的 `.json` 文件里修改。

## 模板

- `tasks.json` 文件

```json
{
	"version": "2.0.0",
	"tasks": [
		{
			"type": "cppbuild",
			"label": "C/C++: g++.exe build active file", //要和 settings.json 的 preLaunchTask 相对应
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
			"detail": "compiler: ...\\MinGW64\\bin\\g++.exe" //并不重要
		}
	]
}
```

- `settings.json` 文件

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
            "program": "${fileDirname}/${fileBasenameNoExtension}.exe", // 将要进行调试的程序的路径
            "args": [], // 程序调试时传递给程序的命令行参数，一般设为空
            "stopAtEntry": false, // 设为true时程序将暂停在程序入口处，相当于在main上打断点
            "cwd": "${fileDirname}", // 调试程序时的工作目录
            "environment": [], // 环境变量
            "externalConsole": true, // 使用单独的cmd窗口，与其它IDE一致；为false时使用内置终端
            "MIMode": "gdb", // 指定连接的调试器，可以为gdb或lldb
            "miDebuggerPath": "gdb.exe", // 调试器路径，Windows下后缀不能省略，Linux下则不要
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

## 参考资料

1. [全网最详细保姆级VSCode配置C/C++教程（基于官方文档）- Mr_Luka
](https://blog.csdn.net/qq_50813669/article/details/124461328)。
