---
title: visual studio code配置C/C++环境
date: 2018-08-28 22:51:56
categories: mome
tags:
- visual studio code
- c++
- c
---

目的是构建了简单 C/C++ 环境，用于小项目 (比如算法题) 。

## 为什么用 visual studio code 写 C/C++

因为我是 visual studio code 的粉丝。这是我用过最快乐的编辑器了。

因为平时写 C/C++ 代码都是为了写算法题而写的，之前在 Windows 上用的 vs，感觉用了牛刀。。不过 vs 的调试功能给以前弱智的我有很大的帮助。。

后来在 Ubuntu 上都是用的 codeblocks 或者直接用 vim 开撸。。不知道为什么我真的对 vim 爱不起来，尝试放弃尝试放弃无数次，感觉只能是大概知道 vim 怎么用，但是作为日常编辑器感觉还是喜欢 vscode。。

第一次用 vscode 的时候，尝试过用 vscode 写 C/C++，但是那时候感觉体验有点糟糕。。后来貌似微软团队对 C/C++ 对了改进的，现在感觉还行，实在不行，就继续之前码完就 `g++ -Wall -std=c++11 main.cpp`。。。

## Prerequisite

下 C/C++ 这个扩展，Microsoft 官方出品的这个。

## Configuration

下面所有的 vscode 命令用 `Ctrl + Alt + P` 打开 vscode 的工具窗口输入。

- `C/CPP: Edit Configuration` => 生成一个 `c_cpp_properities.json` 。
- `Task: Configure Task Runner` => 选择 Other  (当然如果要用 MSBuild 也可以，不过我自己用的是 g++，所以选了 Other => 生成 `task.json`。
    
    这里的 `task.json` 需要自己配置一下，我直接用的 vscode 的 snippet 功能的，贴一下我的 snippet (我的 json.json) : 

    ```json
    {
        // Place your snippets for JSON here. Each snippet is defined under a snippet name and has a prefix, body and 
        // description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
        // $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the 
        // same ids are connected.
        // Example:
        "cpp tasks": {
            "prefix": "cpptask",
            "body": [
            "{",
            "   \"label\": \"build\",",
            "   \"command\": \"g++\",",
            "   \"args\": [",
            "       \"-std=c++11\",",
            "      \"$1\",",
            "      \"-o a.out\"",
            "   ],",
            "   \"type\": \"shell\"",
            "},",
            "{",
            "   \"label\": \"build-debug\",",
            "   \"command\": \"g++\",",
            "   \"args\": [",
            "      \"-g\",",
            "      \"-std=c++11\",",
            "      \"$2\",",
            "      \"-o a.out\"",
            "   ],",
            "   \"type\": \"shell\"",
            "}"
            ],
            "description": "Log output to console"
        }
    }
    ```

    用的 snippet 就减少重复的工作量了，直接输入 `cpptask` 就能生成一个 customize 的 snippet。然后自己写一下需要编译的 cpp 文件名就可以了，最后生成的 task.json 大概是这样的: 

    ```json
    {
        // See https://go.microsoft.com/fwlink/?LinkId=733558
        // for the documentation about the tasks.json format
        "version": "2.0.0",
        "tasks": [
            {
                "label": "echo",
                "type": "shell",
                "command": "echo Hello"
            },
            {
            "label": "build",
            "command": "g++",
            "args": [
                "-std=c++11",
                "./LRU/lru.cpp",
                "-o a.out"
            ],
            "type": "shell"
            },
            {
            "label": "build-debug",
            "command": "g++",
            "args": [
                "-g",
                "-std=c++11",
                "./LRU/lru.cpp",
                "-o a.out"
            ],
            "type": "shell"
            }
        ]
    }
    ```
- 然后就可以编译了，输入 `Tasks: Run Tasks` => `build` 然后编译完成，看到目标文件，执行就好了。
- 调试的功能 (因为可视化的调试比终端里调试舒服多了，这也是我不想用 vim 的理由。。) 
    - `Ctrl + Shift + D` 然后点那个齿轮，就自动生成了一个 `launch.json` 文件 
    - 修改 `program` 里文件名
    - 加上一个 `preLaunchTask` 的字段，值为 `build-debug`，表示调试前先编译。
    - 最后的 `launch.json` 文件大概是这样的: 
    ```json
    {
        "version": "0.2.0",
        "configurations": [
            {
                "name": "(gdb) Launch",
                "type": "cppdbg",
                "request": "launch",
                "program": "${workspaceFolder}/a.out",
                "args": [],
                "stopAtEntry": false,
                "cwd": "${workspaceFolder}",
                "environment": [],
                "externalConsole": true,
                "MIMode": "gdb",
                "setupCommands": [
                    {
                        "description": "Enable pretty-printing for gdb",
                        "text": "-enable-pretty-printing",
                        "ignoreFailures": true
                    }
                ],
                "preLaunchTask": "build-debug"
            }
        ]
    }
    ```
    - 然后打断点，调试，就完事了。

----

具体可以看 [C/C++ for VS Code (Preview)](https://code.visualstudio.com/docs/languages/cpp)，官方的文档更详细。

## References

- [C/C++ for VS Code (Preview)](https://code.visualstudio.com/docs/languages/cpp)
- [VS Code 配置 C/C++ 环境](https://blog.csdn.net/wzxlovesy/article/details/76708151)