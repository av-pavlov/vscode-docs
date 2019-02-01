---
Order: 1
Area: cpp
TOCTitle: Tutorial
ContentId: 7efec972-6556-4526-8aa8-c73b3319d612
PageTitle: Get Started with C++ and Mingw-w64 in Visual Studio Code
DateApproved: 11/26/2018
MetaDescription: Configuring the C++ extension in Visual Studio Code to target g++ and gdb on a Mingw-w64 installation
MetaSocialImage: images/tutorial/social.png
---
# Configure VS Code for Mingw-w64 and GCC

In this tutorial, you configure Visual Studio Code on Windows to use the g++ compiler and gdb debugger in [Mingw-w64 ](http://mingw-w64.org/doku.php/start). The configuration applies to a single project folder hierarchy, but you can easily copy the configuration files to other folders where the same settings are required. After configuring VS Code, you will ready to follow the [Getting Started with VS Code on Windows](cpp-tutorial.md) tutorial.

If you have any problems, feel free to file an issue for this tutorial in the [VS Code documentation repository](https://github.com/Microsoft/vscode-docs/issues).

## Prerequisites

To successfully complete this tutorial, you must do the following:

1. Install [Visual Studio Code for Windows](https://code.visualstudio.com/docs/?dv=win).

1. Install the [C++ extension for VS Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools).

1. Install [Mingw-w64](http://mingw-w64.org/doku.php/download/mingw-builds).

1. Ensure that your Mingw-w64 /bin/ folder is in your path prior to other mingw folders (for example, if you have installed Git with its Bash console, you probably have C:\Program Files\Git\mingw in your path).
    a. You can add the path permanently to your user environment variable by typing `path` in the Windows search box and following the prompts. 
    b. You can set the path temporarily for a single session from a command prompt but then you must start VS Code from that same prompt.
    c. You can set paths manually in the launch.json file for each debug run.
*[Note to reviewers: What is the preferred way to set the path? Maybe just as a session env variable?]

## Start VS Code in a workspace folder

At a command prompt or terminal, create an empty folder called "hello" or whatever you like, navigate into it, and open VS Code (`code`) in that folder (`.`) by entering the following commands:

```bash
mkdir hello
cd hello
code .
```
By starting VS Code in a folder, that folder becomes your "workspace". VS Code stores user settings that are specific to that workspace in `.vscode/settings.json`, which are separate from user settings that are stored globally. In this tutorial, we'll add three additional files to the `.vscode` folder:

- `c_cpp_properties.json` to specify the compiler and include paths.
- `tasks.json` to specify how to build the executable
- `launch.json` to specify debugger settings

## Configure the compiler path

Press **Ctrl+Shift+P** to open the Command Palette. Start typing "C/C++" and then choose **Edit Configurations** from the list of suggestions. VS Code creates a file called `c_cpp_properties.json` and populates it with some default settings. Find the compilerPath setting and paste in the path to g++.exe. In a default Mingw-w64 installation, for example for version 8.1.0, it will be something like this, depending on which specific version you have installed: `C:\Program Files\mingw-w64\x86_64-8.1.0-win32-seh-rt_v6-rev0\mingw64\bin\gcc.exe`.

The extension can now infer the path to the mingw-w64 include folder, which it needs for IntelliSense support. There is no need to specify it explicitly in the `includePath` setting unless you have additional or non-standard paths in your code base. 

Your complete `tasks.json` file should look something like this:

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build hello world",
            "type": "shell",
            "command": "g++",
            "args": [
                "-g",
                "-o",
                "helloworld",
                "main.cpp"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ] 
}
```

## Create a build task

Next, let's edit `tasks.json` to add a build task for our program. The `label` value will be used to identify the task in the VS Code Command Palette; you can name this whatever you like. The `args` array specifies the command line arguments that will be passed to the compiler that was specified in the previous step. These arguments must be specified in the order expected by the compiler. 

The `group` value specifies that this task will be run when you press **Ctrl+Alt+B**.

Your complete `tasks.json` file should look something like this:

```json
"version": "2.0.0",
    "tasks": [
        {
            "label": "build hello world",
            "type": "shell",
            "command": "g++",
            "args": [
                "-g", // compile with debug symbols
                "-o", // name the executable "helloworld.exe"
                "helloworld",
                "main.cpp" // Input file
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
```

## Configure debug settings

Next, we'll configure VS Code to launch gdb when we press *F5* to debug the program. Note that
the program name `helloworld.exe` matches what we specified in `tasks.json`. You will need to adjust your `miDebuggerPath` value to exactly match the path to your Mingw-w64 installation. 

By default, the C++ extension adds a breakpoint to the first line of `main`. The `stopAtEntry` value is set to `true` to cause the debugger to stop on that breakpoint. You can set this to `false` if you prefer tto ignore it.

Your complete `launch.json` file should look something like this:

```json
   "version": "0.2.0",
    "configurations": [
        
        {
            "name": "(gdb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/helloworld.exe",
            "args": [],
            // set to false to ignore the breakpoint set by VS Code:
            "stopAtEntry": true, 
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            // modify to match your specific install location:
            "miDebuggerPath": "C:/Program Files/mingw-w64/x86_64-8.1.0-win32-seh-rt_v6-rev0/mingw64/bin/gdb.exe",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}
```

## Next steps

You are now ready to begin the [Hello World tutorial](cpp-tutorial.md)!
