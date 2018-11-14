
# GRPC使用

## 编译安装

1. 安装Perl，Go，YASM，Git，CMake, nasm 
2. 不要使用--recursive来递归clone，因为墙的原因会失败
3. 所以子模块的下载请使用：git submodule update --init
4. 编译，切换到源码目录，在源码目录下创建一个 .build （如果要编译64位，则是 .build_x64）子目录，我们用该目录保存所有生成的工程文件。创建后进入该目录。
   1. 32位：
        1. `md .build & cd .build`
        2. `cmake -G "Visual Studio 14 2015" -DCMAKE_BUILD_TYPE=Release ..\..`
        3. `MSBuild ALL_BUILD.vcxproj /t:Build /p:Configuration=Debug;Platform=Win32`
        4. `MSBuild ALL_BUILD.vcxproj /t:Build /p:Configuration=Release;Platform=Win32`
       
   2. 64位：
        1. `md .build_x64 &cd .build_x64`
        2. `cmake -G "Visual Studio 14 2015 Win64" -DCMAKE_BUILD_TYPE=Release ..\..`
        3. `MSBuild ALL_BUILD.vcxproj /t:Build /p:Configuration=Debug;Platform=x64`
        4. `MSBuild ALL_BUILD.vcxproj /t:Build /p:Configuration=Release;Platform=x64`
5. 安装，从开始菜单以管理员方式打开MSBuild命令号，切换到上一步的文件夹
    1.  32位安装到 `C:\Program Files (x86)\grpc`, 
    > `MSBuild INSTALL.vcxproj /t:Build /p:Configuration=Release;Platform=Win32`

    1.  64位安装到 `C:\Program Files\grpc`, 
    > `MSBuild INSTALL.vcxproj /t:Build /p:Configuration=Release;Platform=x64`
6. 安装后缺失的库
  `ssl.lib` `crypto.lib`


## 生成接口文件和服务代码

1. C++代码
    1. `protoc -I=. inv903.proto --cpp_out=cpp`
   1. `protoc -I . --grpc_out=cpp --plugin=protoc-gen-grpc="c:\\windows\\grpc_cpp_plugin.exe" inv903.proto`
 1. Go代码
     1. `protoc -I=. inv903.proto --go_out=plugins=grpc:.`
2. Java代码
     1. `protoc -I=. inv903.proto --java_out=java`

## 使用

1. 不使用预编译头
2. 在C++预处理中加入_WIN32_WINNT=0x600
3. 关闭SDL检查
4. 增加include
   ```
   E:\033-daspembed\code\grpcte\third_party\protobuf\include
   E:\033-daspembed\code\grpcte\third_party\grpc\include
5. 加lib
   ```text
  ws2_32.lib
  libprotoc.lib
  libprotobuf.lib
  grpc.lib
  gpr.lib
  grpc++.lib
  zlibstatic.lib
  ssl.lib
  crypto.lib
6. 增加lib搜索目录
    ```text
    E:\033-daspembed\code\grpcte\third_party\protobuf\lib
  E:\033-daspembed\code\grpcte\third_party\grpc\Debug
  E:\033-daspembed\code\grpcte\third_party\grpc.dependencies.zlib.1.2.8.10\build\native\lib\v140\Win32\Debug\static\cdecl
  E:\033-daspembed\code\grpcte\third_party\grpc.dependencies.openssl.1.0.204.1\build\native\lib\v140\Win32\Debug\static
  E:\033-daspembed\code\grpcte\third_party\gflags.2.1.2.1\build\native\Win32\v120\static\Lib


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1MDY3MTU4MzVdfQ==
-->