## Build & Install

My enviroment:

- Architecture: x86_64
- OS: Ubuntu 22.04 LTS
- Editor: VS Code 1.71.0
- Compiler: gcc 11.2.0
- Build Tool: CMake 3.22.1
    

Simply follow the steps in [OpenCV Tutorials](https://docs.opencv.org/4.x/d7/d9f/tutorial_linux_install.html) to install OpenCV on Linux.

I use VS Code as my editor while CMake and gcc to build and compile the program. VSCode support [IntelliSense for cross-compiling](https://code.visualstudio.com/docs/cpp/configure-intellisense-crosscompilation), but if you need code completions for opencv (and no `include path error` when you use opencv!) in your VS Code, some configuration is required.

`opencv4` was installed to _/usr/local/_ by default, and the headers are in _/usr/local/include/opencv4_ from [OpenCV Tutorials](https://docs.opencv.org/4.x/d7/d9f/tutorial_linux_install.html#tutorial_linux_install_detailed_basic_verify). Use `ls /usr/local/include/opencv4` to see here is only one directory _opencv2_ in it.

Press `Ctrl+Shift+P` to open the search box, then enter and choose **`C/C++: Edit Configuration (JSON)`**, the editor will open _.vscode/c_cpp_properties.json_ in current directory. Add the include path of opencv4 to your "includePath":

```JSON
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**",
                "/usr/local/include/opencv4"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/gcc",
            "cStandard": "gnu17",
            "cppStandard": "gnu++17",
            "intelliSenseMode": "linux-gcc-x64",
            "configurationProvider": "ms-vscode.makefile-tools"
        }
    ],
    "version": 4
}
```

Now you can use `#include <opecv2/...>` in your code.

````ad-help
**\[Problem\]** ❓ When try to run example in [Using OpenCV with gcc and CMake](https://docs.opencv.org/4.x/db/df5/tutorial_linux_gcc_cmake.html), I received the following error message when executing `./DisplayImage lena.jpg`:

```
terminate called after throwing an instance of 'cv::Exception'
  what():  
  OpenCV(4.6.0-dev) 
  /home/xyu/Libs/opencv/opencv-4.x/modules/highgui/src/window.cpp:1253: 
  error: (-2:Unspecified error) The function is not implemented. 
  Rebuild the library with Windows, GTK+ 2.x or Cocoa support. 
  If you are on Ubuntu or Debian, install libgtk2.0-dev and pkg-config, 
  then re-run cmake or configure script in function 'cvNamedWindow'

Aborted (core dumped)
```

**[Solution]** ✅ As the hint says:

```shell
sudo apt install libgtk2.0-dev pkg-config
```

Then re-run `cmake` solved this problem.
````
