---


---

<h1 id="编译tensorflow-c-api">编译tensorflow C++ API</h1>
<ol>
<li>下载工具并安装
<ol>
<li><a href="https://cmake.org/download/">Cmake</a></li>
<li><a href="http://www.swig.org/download.html">swig</a></li>
<li><a href="https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/">Anaconda3-5.2.0</a></li>
</ol>
</li>
<li>下载代码
<ol>
<li><a href="https://github.com/tensorflow/tensorflow/tree/v1.6.0-rc0">tensorflow v1.6.0-rc0</a><strong>（tensorflow建议直接下载链接给的这个版本，太新的版本会有错误）</strong></li>
</ol>
</li>
<li>编译
<ol>
<li>打开vs2015 x64本机工具命令提示符，以管理员身份运行 （一定要以管理员身份运行，不然在最后一步会报错）</li>
<li>
<pre><code>cd tensorflow\contrib\cmake
mkdir _build

set SWIG_EXECUTABLE=E:\workspace\103-Whistle\software\swigwin-3.0.12\swig.exe 
set PYTHON_EXECUTABLE=d:\Anaconda3\envs\tensorflow\python.exe 
set PYTHON_LIBRARIES=C:\Anaconda3\envs\tensorflow\libs\python36.lib

cmake -A x64 -DCMAKE_BUILD_TYPE=Release -DSWIG_EXECUTABLE=%SWIG_EXECUTABLE% -DPYTHON_EXECUTABLE=%PYTHON_EXECUTABLE% -DPYTHON_LIBRARIES=%PYTHON_LIBRARIES%  -Dtensorflow_BUILD_PYTHON_BINDINGS=OFF -Dtensorflow_ENABLE_GRPC_SUPPORT=OFF -Dtensorflow_BUILD_SHARED_LIB=ON -DCMAKE_BUILD_TYPE=Release ..

MSBuild /m:1 /p:CL_MPCount=1 /p:Configuration=Release /p:Platform=x64 /p:PreferredToolArchitecture=x64 ALL_BUILD.vcxproj  /filelogger
MSbuild /m:1 /p:CL_MPCount=1 /p:Configuration=Release /p:Platform=x64 /p:PreferredToolArchitecture=x64 INSTALL.vcxproj /filelogger
</code></pre>
</li>
</ol>
</li>
<li>linux环境下（ubuntu16）
<ol>
<li>下载安装 bazel（下载安装最新版） VPN</li>
<li>下载tensorflow（没有版本要求）</li>
<li>cd 到tensorflow文件夹下运行命令 ./configure<br>
运行这条命令后会有一些选项，一些需要地址的输入软件地址（比如python地址），其余的一些配置一律选择no，<br>
如果选择yes会因为一些相应的包没有安装导致报错，也根据自己的实际情况自行选择。一般全部选择no不会报错</li>
<li>运行 bazel build --config=opt //tensorflow:libtensorflow_cc.so</li>
</ol>
</li>
</ol>
<h3 id="c通过上面编译好的dll调用tensorflow生成的.pb文件。">C++通过上面编译好的DLL调用tensorflow生成的.pb文件。</h3>
<ol>
<li>
<p>配置环境</p>
<pre><code>包含目录：
	C:\Program Files\tensorflow\include\tensorflow
	C:\Program Files\tensorflow\include
	C:\Program Files\tensorflow\include\tensorflow\core\protobuf
	C:\Program Files\tensorflow\include\external\nsync\public
	C:\Program Files\tensorflow\include\external\eigen_archive
库目录：
	C:\Program Files\tensorflow\lib
</code></pre>
</li>
<li>
<p>程序源码</p>
<pre><code>#include "stdafx.h"

#include "tensorflow/core/public/session.h"
#include "tensorflow/core/platform/env.h"
#include "tensorflow/core/framework/tensor.h"
#include "tensorflow/cc/ops/standard_ops.h"
#include "tensorflow/cc/client/client_session.h"

#include "stdlib.h"

using namespace tensorflow;

int main() {

	
	std::vector&lt;float&gt; vec= {};

	//新建session
	Session* session;
	Status status = NewSession(SessionOptions(), &amp;session);
	if (!status.ok()) {
		std::cout &lt;&lt; status.ToString() &lt;&lt; "\n";
		return 1;
	}

	//读取创建的pb文件中的graph
	GraphDef graph_def;
	status = ReadBinaryProto(Env::Default(), "model1_pb", &amp;graph_def);
	if (!status.ok()) {
		std::cout &lt;&lt; status.ToString() &lt;&lt; "\n";
		return 1;
	}

	//将读取的graph添加到session中
	status = session-&gt;Create(graph_def);
	if (!status.ok()) {
		std::cout &lt;&lt; status.ToString() &lt;&lt; "\n";
		return 1;
	}

	//设置输入输出
	Tensor x_t(DT_FLOAT, TensorShape({1,1066}));
	int ndim = vec.size();
	Tensor X(DT_FLOAT, TensorShape({ 1,ndim }));
	auto x_map = X.tensor&lt;float, 2&gt;();
	for (int j = 0; j &lt; ndim; j++) {
		x_map(0, j) = vec[j];
	}

	Tensor keep_prob(DT_FLOAT, TensorShape());
	keep_prob.scalar&lt;float&gt;()() = 1.0;

	Tensor batch_norm(DT_BOOL, TensorShape());
	batch_norm.scalar&lt;bool&gt;()() = false;

	std::vector&lt;std::string&gt;labels;
	labels.emplace_back("output/label");
	std::vector&lt;Tensor&gt;label;

	std::vector&lt;std::pair&lt;string, Tensor&gt;&gt;inputs = { {"input",X },
	{"keep_prob",keep_prob },{ "batch_norm",batch_norm } };

	//Run the session ， evaluation operation from the graph
	status = session-&gt;Run(inputs, labels, {}, &amp;label);
	if (!status.ok()) {
		std::cout &lt;&lt; status.ToString() &lt;&lt; "\n";
		return 1;
	}

	打印结果
	auto output_c = label[0].scalar&lt;int&gt;();
	std::cout &lt;&lt; label[0].DebugString() &lt;&lt; "\n";
	std::cout &lt;&lt; output_c() &lt;&lt; "\n";

	//关闭会话
	session-&gt;Close();
	return 0;

	// 如果程序编译没有出错，运行时出错。一般是输入数据的名字与模型placehold不一样导致的，输出名字一定要和模型输出数据结点一致，
	// 还有一种情况是输入数据的维度和pb文件定义的输入维度不一致。
}
</code></pre>
</li>
</ol>
<h3 id="编译好的动态库使用">编译好的动态库使用</h3>
<p>1、初始化<br>
1.1、函数原型<br>
int *WhistleIdentificationInit(char *szPbFile)<br>
1.2、使用说明<br>
输入：pb文件<br>
输出：无<br>
返回：无<br>
说明：此函数主要是用于新建会话，以及加载计算图结构，以及一些初始化。<br>
2、直接对文件推理<br>
2.1、函数原型<br>
int WhistleIdentificationRunFile(char *wave)<br>
2.2、使用说明<br>
输入：wav文件<br>
输出：无<br>
返回：标签<br>
说明：此函数主要是用于wav文件直接进行推理，输入一个wav文件，返回一个标签。<br>
3、直接对内存中的wav波形进行推理<br>
3.1、函数原型<br>
int WhistleIdentificationRunWave(float *wave, int size)<br>
3.2、使用说明<br>
输入：wav波形，wav波形占内存大小<br>
输出：无<br>
返回：标签<br>
说明：此函数用于直接对内存中波形进行推理，输入一个wav波形，返回一个标签。</p>
<p>四、例程<br>
int inference()<br>
{<br>
//加载模型以及进行初始化<br>
char* filePath = “C:\Users\ym\Desktop\model1_pb”;<br>
WhistleIdentificationInit(filePath);</p>
<pre><code>int label;

//用wav文件进行推理
char* wave = "C:\\Users\\ym\\Desktop\\声音文件\\新建文件夹\\20181005_221911_tsl.wav";
label = WhistleIdentificationRunFile(wave);

//用wav波形进行推理
float wave[512];
label = WhistleIdentificationRunWave(wave, sizeof(wave)/sizeof(float));

//关闭会话
WhistleIdentificationClose();	
</code></pre>
<p>}</p>
<h3 id="参考链接">参考链接</h3>
<ol>
<li><a href="https://medium.com/jim-fleming/loading-a-tensorflow-graph-with-the-c-api-4caaff88463f">https://medium.com/jim-fleming/loading-a-tensorflow-graph-with-the-c-api-4caaff88463f</a></li>
<li><a href="https://medium.com/@shiweili/building-tensorflow-c-shared-library-on-windows-e79c90e23e6e">https://medium.com/@shiweili/building-tensorflow-c-shared-library-on-windows-e79c90e23e6e</a></li>
</ol>

