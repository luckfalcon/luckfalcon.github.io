# Vscode运行配置


<!--more-->
## Ubuntu(Linux) /Windows 配置Vscode的Cmake环境
### 命令编译
1. Linux 下编译运行命令
```bash
#切换至构建目录并编译文件
cd build
cmake ..
make
#运行
./demo  #可执行文件名
```
2. Windows 下编译运行命令
```bash
cd build
cmake -G "MinGW Makefiles" .. #指定生成编译文件类型为makefile，否则会调用vs编译成vs文件
mingw32-make.exe              #生成可执行文件
```
### Vscode环境编译

- `launch.json` 文件配置
```bash
{
    "configurations": [
    {
        "name": "(gdb) Launch",
        "type": "cppdbg",
        "request": "launch",
        "program": "${workspaceFolder}/build/demo", #调试可执行文件所在路径及名字
        "args": [],
        "stopAtEntry": false,
        "cwd": "${fileDirname}",
        "environment": [],
        "externalConsole": false,
        "MIMode": "gdb",
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
        "preLaunchTask": "Build", #调试前预处理生产可执行文件
        "miDebuggerPath": "/usr/bin/gdb" #调试器运行路径
    }
    ]
}
```
- `tasks.json`文件配置
```bash
{

    "version": "2.0.0",
    "options": {
        "cwd": "${workspaceFolder}/build" #项目构建路径
    },
    "tasks": [
        {
            "type": "shell",
            "label": "cmake",
            "command": "cmake",
            "args": [
                ".."      #cmake 工程路径,即顶层CmakeList.txt文件所在路径
                #如果是windows下配置cmake,需要在".."前增加两个参数用于指定生成makefile而不是msvc,即
                # "-G",
                # "MinGW Makefiles",
            ]
        },
        {
            "label": "make",
          "group": {
            "kind": "build",
            "isDefault": true
          },
          #"command":"mingw32-make.exe",
          "command":"make",  #linux 下为 make, windows 下为 mingw32-make.exe
          "args": [

          ]
        },
        {
            "label": "Build",
       "dependsOrder": "sequence",  #按列出的顺序执行任务P
       "dependsOn":[
        "cmake",
        "make"
       ]
        }
    ]
}
```
## VScode `task.json` 部分环境变量说明
```bash
${workspaceFolder}          #表示当前 workspace 文件夹路径, 也即/home/Demo/Test
${workspaceRootFolderName}  #表示workspace的文件夹名, 也即Test
${file}                     #文件自身的绝对路径, 也即/home/Demo/Test/.vscode/tasks.json
${relativeFile}             #文件在workspace中的路径, 也即.vscode/tasks.ison
${fileBasenameNoExtension}  #当前文件的文件名, 不带后缀, 也即tasks
${fileBasename}             #当前文件的文件名, tasks.json
${fileDirname}              #文件所在的文件夹路径, 也即/home/Demo/Test/.vscode
${fileExtname}              #当前文件的后缀，也即.json
${lineNumber}               #当前文件光标所在的行号
${envPATH}                  #系统中的环境变量
```
