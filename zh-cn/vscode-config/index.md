# Vscode运行配置


<!--more-->
## Ubuntu(Linux) /Windows 配置
- Linux 下编译运行命令
```bash
#切换至构建目录并编译文件
cd build
cmake ..
make
#运行
./demo  #可执行文件名
```
- Windows 下编译运行命令
```bash
cd build
cmake -G "MinGW Makefiles" .. #指定生成编译文件类型为makefile，否则会调用vs编译成vs文件
mingw32-make.exe              #生成可执行文件
```
- launch.json
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
- tasks.json
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
                ".."      #cmake 工程路径
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
       "dependsOrder": "sequence",//按列出的顺序执行任务P
       "dependsOn":[
        "cmake",
        "make"
       ]
        }
    ]
}
```
