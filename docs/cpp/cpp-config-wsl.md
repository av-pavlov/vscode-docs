---
Order: 1
Area: cpp
TOCTitle: Tutorial
ContentId: 
PageTitle: Get Started with C++ and Windows Subsystem for Linux in Visual Studio Code
DateApproved: 02/20/2019
MetaDescription: Configuring the C++ extension in Visual Studio Code to target g++ and gdb on WSL installation with Ubuntu
MetaSocialImage: images/tutorial/social.png
---
# Configure VS Code for Windows Subsystem for Linux

In this tutorial, you configure Visual Studio Code on Windows to use the g++ compiler and gdb debugger on Ubuntu in the Windows Subsystem for Linux (WSL). The advantage of using WSL over a remote machine or container is that WSL provides direct access to both the Windows file system and the Linux file system. We don't have to bother with setting up an ssh connection. We'll edit the source code in VS Code on Windows, and then compile and debug it in WSL.

The configuration applies to a single project folder hierarchy, but you can easily copy the configuration files to other folders where the same settings are required. The same general steps apply to any Linux distro you might want to use. 

After completing this tutorial, you will be ready to create and configure your own workspace, and to explore the VS Code documentation for further information about its many features. This tutorial does not teach you about GCC or Linux or the C++ language. For those subjects there are many good resources available on the Web.

If you have any problems, feel free to file an issue for this tutorial in the [VS Code documentation repository](https://github.com/Microsoft/vscode-docs/issues).

## Prerequisites

To successfully complete this tutorial, you must do the following:

1. Install [Visual Studio Code for Windows](https://code.visualstudio.com/docs/?dv=win).

1. Install the [C++ extension for VS Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools).

1. Install [Windows Subsytem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10) and then use the links on that same page to install your Linux distro of choice. This tutorial uses Ubuntu.

1. Open the Bash shell for WSL. For an Ubuntu distro, type "Ubuntu" in the Windows search box. For Debian, type "Debian", and so on.

1. From the command prompt, install the GNU compiler tools and the GDB debugger by typing:

```cmd
sudo apt-get install build-essential gdb
```

1. Under your $HOME directory in WSL, create a new folder called `projects` and a new folder under that called `helloworld'. This is where we will tell g++ to place the output file that it will produce from our source file.

```cmd
cd $HOME
mkdir projects
cd projects 
mkdir helloworld
```

## Start VS Code in a workspace folder

At a Windows command prompt, create an empty folder called "helloworld" or whatever you like, navigate into it, and open VS Code (`code`) in that folder (`.`) by entering the following commands:

```bash
mkdir hello
cd hello
code .
```
By starting VS Code in a folder, that folder becomes your "workspace". VS Code stores user settings that are specific to that workspace in `.vscode/settings.json`, which are separate from user settings that are stored globally. In this tutorial, we'll add three additional files to the `.vscode` folder:

- `c_cpp_properties.json` to specify the compiler path
- `tasks.json` to specify how to build the executable
- `launch.json` to specify debugger settings

## Configure the compiler path

Press **Ctrl+Shift+P** to open the Command Palette. Start typing "C/C++" and then choose **Edit Configurations** from the list of suggestions. VS Code creates a file called `c_cpp_properties.json` and populates it with some default settings. Find the `compilerPath` setting and paste in the path to the `bin` folder. In a Mingw-w64 installation, for example for version 8.1.0, it will be something like this, depending on which version you have installed: `C:\mingw-w64\x86_64-8.1.0-win32-seh-rt_v6-rev0\mingw64\bin\g++.exe`. (Note that this is not the default location, which is problematic due to the space in "Program Files".)

The `compilerPath` setting is the most important setting in yor configuration. The extension uses it to infer the path to system header files, which it needs for IntelliSense support. There is no need to specify it explicitly in the `includePath` setting unless you have additional or non-standard paths in your code base. In fact, we recommend that you delete the setting entirely if you don't need it. The only other change is to set `intelliSenseMode` to `gcc-x64"`. Your complete `c_cpp_properties.json` file should look something like this:

```json
{
    "configurations": [
        {
            "name": "Win32",
            "defines": [
                "_DEBUG",
                "UNICODE",
                "_UNICODE"
            ],
            "compilerPath": "/usr/bin/g++",
            "cStandard": "c11",
            "cppStandard": "c++17",
            "intelliSenseMode": "gcc-x64",
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

Next, let's edit `tasks.json` to add a build task for our program. Pay attention to the args array; we are telling g++ on WSL to grab the source file in our current workspace directory on Windows, compile it, then place the executable file in our `helloworld` folder under the $HOME/projects/helloworld folder in WSL. 

The `label` value will be used to identify the task in the VS Code Command Palette; you can name this whatever you like. The `args` array specifies the command line arguments that will be passed to the compiler that was specified in the previous step. These arguments must be specified in the order expected by the compiler. 

The `"isDefault": true` value in the `group` object specifies that this task will be run when you press **Ctrl+Alt+B**. This property is for convenience only; if you set it to false you'll have to run it from the Command Pallette menu under "Run Build Task".

Your complete `tasks.json` file should look something like this:

```json
 // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build hello world on WSL",
            "type": "shell",
            "command": "g++",
            "args": [
                "-g",
                "-o",
                "$HOME/projects/helloworld/helloworld.out",
                "helloworld.cpp"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ] 
}
```

## Configure debug settings

Next, we'll configure VS Code to launch gdb on WSL when we press *F5* to debug the program. Note that the program name `helloworld.out` matches what we specified in `tasks.json`. You might need to adjust your `miDebuggerPath` value to exactly match the path to your WSL installation.

Note that the path to the executable must be an absolute path. You will have to substitute your actual Linux user name in the `program` and `cwd` properties. Under `sourceFileMap` we need to tell gdb where to find the header files. Currently, this value must point to the actual location in the Windows file system. After you add the source code file to this project, you'll be able to easily get that path. For now you can paste this JSON into your `launch.json` file; we'll come back later to fix the file mapping.

Your complete `launch.json` file should look something like this:

```json
"version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "/home/<your Linux user name>/projects/helloworld/helloworld.out",
            "args": ["-fThreading"],
            "stopAtEntry": true,
            "cwd": "/home/<your Linux user name>/projects/helloworld/",
            "environment": [],
            "externalConsole": true,
            "windows": {
                "MIMode": "gdb",
                "setupCommands": [
                    {
                        "description": "Enable pretty-printing for gdb",
                        "text": "-enable-pretty-printing",
                        "ignoreFailures": true
                    }
                ]
            },
            "pipeTransport": {
                "pipeCwd": "",
                "pipeProgram": "c:\\windows\\sysnative\\bash.exe",
                "pipeArgs": ["-c"],
                "debuggerPath": "/usr/bin/gdb"
            },
            "sourceFileMap": {
                "/mnt/c": "c:\\",
                "/usr": "C:/Users/<your Windows user name>/AppData/Local/Packages/CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc/LocalState/rootfs/usr/"
            }
        }
    ]
}
```

By default, the C++ extension adds a breakpoint to the first line of `main`. The `stopAtEntry` value is set to `true` to cause the debugger to stop on that breakpoint. You can set this to `false` if you prefer to ignore it.

## Tell VS Code which Bash console to use

By default, the VS Code integrated Terminal first looks for a WSL Bash console. This is located in either "C:/Windows/System32/bash.exe" (for 64-bit VS Code) or "C:/Windows/sysnative/bash.exe" (for 32-bit VS Code). Since it's possible that you have other bash consoles on your system, for example in your Git for Windows folder, let's make sure that VS Code knows unambiguously which console we intend to use for this project. This will save potential headaches if we

1. From the main menu, open **View > Command Palette** and start typing "Settings". From the suggestion list, choose **Preferences: Open Settings (JSON)**. If you are running 64-bit VS code, add this line:

```json
    "terminal.integrated.shell.windows": "C:/Windows/System32/bash.exe"
```

If you are running 32-bit VS Code add this line:

```json
    "terminal.integrated.shell.windows": "C:/Windows/sysnative/bash.exe"
```

## Add a source code file

1. In the main VS Code menu, click on **File > New File** and name it `helloworld.cpp`. 
1. Paste in this source code:

    ```cpp
    #include <iostream>
    #include <vector>
    #include <string>
    
    using namespace std;
    
    int main()
    {
    
        vector<string> msg {"Hello", "C++", "World", "from", "VS Code!"};
    
        for (const string& word : msg)
        {
            cout << word << " ";
        }
        cout << endl;
    }
    ```

1. Now press **Ctrl+S** to save the file. Notice how all the files we have just edited appear in the **File Explorer** view in the left panel of VS Code:

![File Explorer](images/file-explorer-mingw.png)

This same panel is also used for source control, debugging, searching and replacing text, and managing extensions. The buttons on the left control those views. We'll look at the **Debug View** later in this tutorial. You can find out more about the other views in the VS Code documentation.

## Get the path to the system headers

Now that we have a source file, we can use it to easily get the Windows path to the system headers. We need to specify this value in the file mappings in `launch.json` file so that gdb can step into system headers if you press **F11** during debugging.

1. In helloworld.cpp hover your mouse over the `string` in this statement: `vector<string> msg...`
1. Right-click and choose **Go to definition** to open `stringfwd.h` in the editor.
1. Right click on the tab with the file name and choose **Copy path**.
1. Navigate back to `launch.json` and replace the path in this value with the path you just copied:

```json
"/usr": "C:/Users/mblome/AppData/Local/Packages/CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc/LocalState/rootfs/usr/"
```

1. Delete everything in your new path back to `usr/` as shown in the example above.

## Build the program

1. To run the build task that you defined in tasks.json, press **Ctrl+Shift+B** or from the main menu choose **View > Command Palette** and start typing "Tasks: Run Build Task". The option will appear before you finish typing. 
1. When the task starts, you should see the integrated Terminal window appear below the code editor. After the task completes, the terminal shows output from the compiler that indicates whether the build succeeded or failed. For a successful g++ build, the output looks something like this:

![G++ build output in terminal](images/wsl-task-in-terminal.png)

1. As the message instructs, press any key to close the build message; the terminal now returns to the shell command prompt.

## Start a debugging session

1. You are now ready to run the program. Press **F5** or from the main menu choose **Debug > Start Debugging**. Before we start stepping through the code, let's take a moment to notice several changes in the user interface: 

- The integrated terminal appears at the bottom of the code editor. In the **Debug Output** tab, you see output that indicates the debugger is up and running.
- The code editor highlights the first statement in the `main` method. This is a breakpoint that the C++ extension automatically sets for you: 

![Initial breakpoint](images/wsl-breakpoint-default.png)

- The workspace pane on the left now shows debugging information: 

![Debugging windows](images/debugging-windows.png)

- At the top of the code editor, a debugging control panel appears. You can move this around the screen by grabbing the dots on the left side.

![Debugging controls](images/debug-controls.png)

## Step through the code

Now we're ready to start stepping through the code. 

1. Click or press the "Step over" icon in the debugging control panel. 

    ![Step over button](images/step-over-button.png)

    This will advance program execution to the first line of the for loop, and skip over all the internal function calls within the `vector` and `string` classes that are invoked when the `msg` variable is created and initialized. Notice the change in the **Variables** window on the left. In this case, the errors are expected because, although the variable names for the loop are now visible to the debugger, the statement has not executed yet, so there is nothing to read at this point. The contents of `msg` are visible, however, because that statement has completed. 
1. Press **Step over** again to advance to the next statement in this program (skipping over all the internal code that is executed to initialize the loop). Now, the **Variables** window shows information about the loop variables. 
1. Press **Step over** again to execute the `cout` statement. In the **Terminal** window, you now see "Hello".
1. If you like, you can keep pressing **Step over** until all the words in the vector have been printed to the console. But if you are curious, try pressing the **Step Into** button to step through source code in the C++ standard library! 

    ![Breakpoint in gcc standard library header](images/gcc-system-header-stepping.png)

    To return to your own code, one way is keep pressing **Step over**. Another way is to set a breakpoint in your code by switching to the `helloworld.cpp` tab in the code editor, putting the insertion point somewhere on the `cout` statement inside the loop, and pressing **F9**. A red dot appears in the gutter on the left to indicate that a breakpoint has been set on this line. 

    ![Breakpoint in main](images/breakpoint-in-main.png)

    Then press **F5** to start execution from the current line in the standard library header. Execution will break on `cout`. If you like, you can press **F9** again to toggle the breakpoint off.

## Set a watch

Sometimes you might want to keep track of the value of a variable as your program executes. You can do this by setting a *watch* on the variable. 

1. Place the insertion point inside the loop. In the **Watch** window, click the plus sign and in the text box, type `word`, which is the name of the loop variable. Now view the Watch window as you step through the loop.
1. Add another watch by adding this statement before the loop: `int i = 0;`. Then, inside the loop, add this statement: `++i;`. Now add a watch for `i` as you did in the previous step. 

## Next steps


