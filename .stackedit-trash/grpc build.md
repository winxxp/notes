---


---

<h1 id="grpc使用">GRPC使用</h1>
<h2 id="编译安装">编译安装</h2>
<ol>
<li>
<p>安装Perl，Go，YASM，Git，CMake, nasm</p>
</li>
<li>
<p>不要使用–recursive来递归clone，因为墙的原因会失败</p>
</li>
<li>
<p>所以子模块的下载请使用：git submodule update --init</p>
</li>
<li>
<p>编译，切换到源码目录，在源码目录下创建一个 .build （如果要编译64位，则是 .build_x64）子目录，我们用该目录保存所有生成的工程文件。创建后进入该目录。</p>
<ol>
<li>
<p>32位：</p>
<ol>
<li><code>md .build &amp; cd .build</code></li>
<li><code>cmake -G "Visual Studio 14 2015" -DCMAKE_BUILD_TYPE=Release ..\..</code></li>
<li><code>MSBuild ALL_BUILD.vcxproj /t:Build /p:Configuration=Debug;Platform=Win32</code></li>
<li><code>MSBuild ALL_BUILD.vcxproj /t:Build /p:Configuration=Release;Platform=Win32</code></li>
</ol>
</li>
<li>
<p>64位：</p>
<ol>
<li><code>md .build_x64 &amp;cd .build_x64</code></li>
<li><code>cmake -G "Visual Studio 14 2015 Win64" -DCMAKE_BUILD_TYPE=Release ..\..</code></li>
<li><code>MSBuild ALL_BUILD.vcxproj /t:Build /p:Configuration=Debug;Platform=x64</code></li>
<li><code>MSBuild ALL_BUILD.vcxproj /t:Build /p:Configuration=Release;Platform=x64</code></li>
</ol>
</li>
</ol>
</li>
<li>
<p>安装，从开始菜单以管理员方式打开MSBuild命令号，切换到上一步的文件夹</p>
<ol>
<li>32位安装到 <code>C:\Program Files (x86)\grpc</code>,</li>
</ol>
<blockquote>
<p><code>MSBuild INSTALL.vcxproj /t:Build /p:Configuration=Release;Platform=Win32</code></p>
</blockquote>
<ol>
<li>64位安装到 <code>C:\Program Files\grpc</code>,</li>
</ol>
<blockquote>
<p><code>MSBuild INSTALL.vcxproj /t:Build /p:Configuration=Release;Platform=x64</code></p>
</blockquote>
</li>
<li>
<p>安装后缺失的库<br>
<code>ssl.lib</code> <code>crypto.lib</code></p>
</li>
</ol>
<h2 id="生成接口文件和服务代码">生成接口文件和服务代码</h2>
<ol>
<li>C++代码
<ol>
<li><code>protoc -I=. inv903.proto --cpp_out=cpp</code></li>
<li><code>protoc -I . --grpc_out=cpp --plugin=protoc-gen-grpc="c:\\windows\\grpc_cpp_plugin.exe" inv903.proto</code></li>
</ol>
</li>
<li>Go代码
<ol>
<li><code>protoc -I=. inv903.proto --go_out=plugins=grpc:.</code></li>
</ol>
</li>
<li>Java代码
<ol>
<li><code>protoc -I=. inv903.proto --java_out=java</code></li>
</ol>
</li>
</ol>
<h2 id="使用">使用</h2>
<ol>
<li>不使用预编译头</li>
<li>在C++预处理中加入_WIN32_WINNT=0x600</li>
<li>关闭SDL检查</li>
<li>增加include<pre><code>E:\033-daspembed\code\grpcte\third_party\protobuf\include
E:\033-daspembed\code\grpcte\third_party\grpc\include
</code></pre>
</li>
<li>加lib<pre class=" language-text"><code class="prism  language-text"></code></pre>
</li>
</ol>
<p>ws2_32.lib<br>
libprotoc.lib<br>
libprotobuf.lib<br>
grpc.lib<br>
gpr.lib<br>
grpc++.lib<br>
zlibstatic.lib<br>
ssl.lib<br>
crypto.lib<br>
6. 增加lib搜索目录<br>
```text<br>
E:\033-daspembed\code\grpcte\third_party\protobuf\lib<br>
E:\033-daspembed\code\grpcte\third_party\grpc\Debug<br>
E:\033-daspembed\code\grpcte\third_party\grpc.dependencies.zlib.1.2.8.10\build\native\lib\v140\Win32\Debug\static\cdecl<br>
E:\033-daspembed\code\grpcte\third_party\grpc.dependencies.openssl.1.0.204.1\build\native\lib\v140\Win32\Debug\static<br>
E:\033-daspembed\code\grpcte\third_party\gflags.2.1.2.1\build\native\Win32\v120\static\Lib</p>

<!--stackedit_data:
eyJoaXN0b3J5IjpbNjk1Nzg2MDAxXX0=
-->