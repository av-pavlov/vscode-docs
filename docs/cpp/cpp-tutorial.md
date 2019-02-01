---
Order: 1
Area: cpp
TOCTitle: Tutorial
ContentId: 7efec972-6556-4526-8aa8-c73b3319d612
PageTitle: Get Started with C++ and Mingw-w64 in Visual Studio Code
DateApproved: 11/26/2018
MetaDescription: A C++ hello world tutorial using the C++ extension in Visual Studio Code to target g++ and gdb on a Mingw-w64 installation
MetaSocialImage: images/tutorial/social.png
---
# Getting Started with C++ in VS Code

In this tutorial, you create a simple "Hello World" C++ application in Visual Studio Code on Windows. The tutorial shows how to write and navigate code in the editor, and how to build and debug your program. This tutorial is not intended to teach you about C++ itself. Once you are familiar with the basics of VS Code, you can then follow any tutorial on web, for example, the [C++ Tutorials and Courses](https://hackr.io/tutorials/learn-c-plus-plus) listed on hackr.io.

If you have any problems, feel free to file an issue for this tutorial in the [VS Code documentation repository](https://github.com/Microsoft/vscode-docs/issues).

## Prerequisites

To successfully complete this tutorial, you must first do the following:

1. Install [Visual Studio Code for Windows](https://code.visualstudio.com/docs/?dv=win).

1. Install the [C++ extension for VS Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools).

1. Configure VS code for Windows to target one of these environments:
    a. [the Microsoft C++ toolset (MSVC)](cpp-config-msvc.md)
    b. [gcc on Mingw-w64](cpp-config-mingw.md)
    c. [gcc on Windows Subsystem for Linux]()

## Navigate to your workspace

If you followed the configuration steps in one of the tutorials in the Prerequisites section, you now have a folder called "hello" on your local machine. That folder contains a .vscode subfolder which contains your configuration files. This folder is your *workspace*. Navigate to it now in a command prompt and type `code .` to open the workspace in VS Code. 

## Create a source file

We need some source code to compile and run. You can supply your own program or use the following example. In either case, name the file `main.cpp` to correspond to the values that you set in the configuration files.

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
}
```

## Explore IntelliSense

In your new `main.cpp` file, hover over `vector` or `string` to see type information. After the declaration of the `msg` variable, start typing `msg.` as you would when calling a member function. You should immediately see a completion list that shows all the member functions, and a window that shows the type information for the `msg` object:

![Statement completion IntelliSense](images/cpp-intellisense-vector.png)

## Run the build task

If you configured your workspace according to the instructions in the Prerequisites, you have already defined a default build task that will compiler main.cpp. To run the task, type **Ctrl+Shift+B**. 

*[Note to reviewers: my msvc tasks.json doesn't seem to like the "isDefault" setting. am i doing something wrong there?]* 

You should see the Terminal window appear with output from the compiler that indicates whether the build succeeded or failed. 