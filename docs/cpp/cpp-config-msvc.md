---
Order: 1
Area: cpp
TOCTitle: Tutorial
ContentId: c8b779d6-79e2-49d6-acfc-430d7ac3a299
PageTitle: Configure Visual Studio Code for Microsoft C++ 
DateApproved: 11/26/2018
MetaDescription: Configure the C++ extension in Visual Studio Code to target Microsoft C++ on Windows.
MetaSocialImage: images/tutorial/social.png
---
# Configure VS Code for Microsoft C++

In this tutorial, you configure Visual Studio Code on Windows to use the Microsoft C++ compiler and debugger. The configuration applies to a single project folder hierarchy, but you can easily copy the configuration files to other folders where the same settings are required. After configuring VS Code, you will ready to follow the [Getting Started with VS Code on Windows](cpp-tutorial.md) tutorial.

If you have any problems, feel free to file an issue for this tutorial in the [VS Code documentation repository](https://github.com/Microsoft/vscode-docs/issues).

## Prerequisites

To successfully complete this tutorial, you must do the following:

1. Install [Visual Studio Code for Windows](https://code.visualstudio.com/docs/?dv=win).
1. Install the [C++ extension for VS Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools).

1. Install the Microsoft C++ (MSVC) compiler toolset. 
    a. If you have a recent version of Visual Studio, open the Visual Studio Installer from the Windows Start menu and verify that the C++ workload is checked. 
    b. Download the standalone toolset by clicking the Visual Studio Build Tools link on the [Downloads page](https://visualstudio.microsoft.com/downloads/#other) and follow the prompts.
2. 

## Start VS Code in a workspace folder

At a command prompt or terminal, create an empty folder called "hello" or whatever you like, navigate into it, and open VS Code (`code`) in that folder (`.`) by entering the following commands:

```bash
mkdir hello
cd hello
code .
```
By starting VS Code in a folder, that folder becomes your "workspace". VS Code stores user settings that are specific to that workspace in `.vscode/settings.json`, which are separate from user settings that are stored globally. In this tutorial, we'll add three additional files to the `.vscode` folder to configure the workspace to target MSVC:

- `c_cpp_properties.json` to specify the compiler and include paths.
- `tasks.json` to specify how to build the executable
- `launch.json` to specify debugger settings

## Configure the compiler path

1. Press **Ctrl+Shift+P** to open the **Command Palette**. Start typing "C/C++" and then choose **Edit Configurations** from the list of suggestions. VS Code creates a file called `c_cpp_properties.json` and populates it with some default settings. Find the `compilerPath` setting and paste in the path to cl.exe. In a default Visual Studio 2017 Community installation, it will be something like this, depending on which specific version you have installed: `C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/bin/Hostx64/x64/cl.exe`.

1. Change the `intelliSenseMode` value to `"msvc-x64"`.
 
The extension can now infer the path to the MSVC include folder, which it needs for IntelliSense support. There is no need to specify the `includePath` value explicitly unless you have additional paths to header files in your code base. 

Your complete `c_cpp_properties.json` file should like something like this:

```json
{
    "configurations": [
        {
            "name": "Win32",
            "defines": [ 
                "_DEBUG",
                "UNICODE"
            ],
            "compilerPath": "C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/bin/Hostx64/x64/cl.exe",
            "intelliSenseMode": "msvc-x64",
            "browse": {
                "path": [
                    "${workspaceFolder}"
                ],
                "limitSymbolsToIncludedHeaders": true,
                "databaseFilename": ""
            }
        }
    ],
    "version": 4
}
```

## Create a build task

Next, let's edit `tasks.json` to add a build task for our program. The `label` value will be used in the VS Code Command Palette and can be whatever name you like. The `command` value says that we are using `cl.exe`, the MSVC compiler. The `args` array specifies the command line arguments that will be passed to the compiler that was specified in the previous step. They must appear in the order expected by the compiler. Note that currently you must use a dash (for example, -EHsc) before any [MSVC compiler options](https://docs.microsoft.com/en-us/cpp/build/reference/compiler-options), even though the compiler itself also allows slashes (for example, /EHsc). In this example, we are specifying the exception handling mode (EHsc) and telling the compiler to produce a debug build with symbols (Zi). The `-o` argument tells the compiler to name the executable "helloworld.exe". 

The `group` value specifies that this task will be run when you press **Ctrl+Alt+B**.

Your complete `tasks.json` file should look something like this:

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "msvc build",
            "type": "shell",
            "command": "cl.exe",
            "args": [
                "-EHsc",
                "-Zi",               
                "main.cpp",
                "-o",
                "helloworld.exe"
            ],
            "group": "build",
            "presentation": {
                // Reveal the output only if unrecognized errors occur.
                "reveal":"always"
            },
            // Use the standard MS compiler pattern to detect errors, warnings and infos
            "problemMatcher": "$msCompile"
        }
    ]
}

```

## Configure debug settings

Finally, we'll configure VS Code to launch the Visual Studio debugger when we press *F5* to debug the program. Note that
the program name `helloworld.exe` matches what we specified in `tasks.json`. 
*[Note to reviewers: does cppvsdbg actually mean the VS debugger and if so, what is cppdbg and does it come with the standalone build tools?]*

By default, the C++ extension adds a breakpoint to the first line of `main`. The `stopAtEntry` value is set to `true` to cause the debugger to stop on that breakpoint. You can set this to `false` if you prefer tto ignore it.


```json
   "version": "0.2.0",
    "configurations": [
        
        {
            "name": "(msvc) Launch",
            "type": "cppvsdbg",
            "request": "launch",
            "program": "${workspaceFolder}/helloworld.exe",
            "args": [],
            // set to false to ignore the breakpoint set by VS Code:
            "stopAtEntry": true, 
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false
        }
    ]
}
```

## Next steps

You are now ready to begin the [Hello World tutorial](cpp-tutorial.md)!
