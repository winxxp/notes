编译tensorflow C++ API
=================
1. 下载工具并安装 
   1.  [Cmake](https://cmake.org/download/)
   1.  [swig](http://www.swig.org/download.html)
   1.  [Anaconda3-5.2.0](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/) 
1. 下载代码
   1. [tensorflow v1.6.0-rc0](https://github.com/tensorflow/tensorflow/tree/v1.6.0-rc0)**（tensorflow建议直接下载链接给的这个版本，太新的版本会有错误）**
 1. 编译
	1.  打开vs2015 x64本机工具命令提示符，以管理员身份运行 （一定要以管理员身份运行，不然在最后一步会报错）
	2. `cd tensorflow\contrib\cmake`
	3. `mkdir _build`
	4. `cmake -A x64 -DCMAKE_BUILD_TYPE=Release -DSWIG_EXECUTABLE=%SWIG_EXECUTABLE% -DPYTHON_EXECUTABLE=%PYTHON_EXECUTABLE% -DPYTHON_LIBRARIES=%PYTHON_LIBRARIES%  -Dtensorflow_BUILD_PYTHON_BINDINGS=OFF -Dtensorflow_ENABLE_GRPC_SUPPORT=OFF -Dtensorflow_BUILD_SHARED_LIB=ON -DCMAKE_BUILD_TYPE=Release ..` （命令中的三个路径根据自己第一步安装的路径进行相应的修改）
		7.1、运行命令 `MSbuild m:1 p:CL_MPCount=1 p:Configuration=Release  p:Platform=x64 p:PreferredToolArchitecture=x64 ALL_BUILD.vcxproj filelogger`
		   （运行这一步时一定要有稳定的VPN，因为要到google下载一些库，否则会编译不成功）
		8.1、运行命令 MSbuild m:1 p:CL_MPCount=1 p:Configuration=Release  p:Platform=x64 p:PreferredToolArchitecture=x64 INSTALL.vcxproj filelogger
		   （运行完这一步后就会在c:/program下生成一个tensorflow文件夹。）
	
	2、linux环境下（ubuntu16）
		1、下载安装 bazel（下载安装最新版） VPN
		2、下载tensorflow（没有版本要求）
		3、cd 到tensorflow文件夹下运行命令 ./configure
			运行这条命令后会有一些选项，一些需要地址的输入软件地址（比如python地址），其余的一些配置一律选择no，
			如果选择yes会因为一些相应的包没有安装导致报错，也根据自己的实际情况自行选择。一般全部选择no不会报错
		4、运行 bazel build --config=opt //tensorflow:libtensorflow_cc.so

	
  
二、 C++通过上面编译好的DLL调用tensorflow生成的.pb文件。

1、配置环境 
包含目录：
	C:\Program Files\tensorflow\include\tensorflow
	C:\Program Files\tensorflow\include
	C:\Program Files\tensorflow\include\tensorflow\core\protobuf
	C:\Program Files\tensorflow\include\external\nsync\public
	C:\Program Files\tensorflow\include\external\eigen_archive
库目录：
	C:\Program Files\tensorflow\lib

2、程序源码

#include "stdafx.h"

#include "tensorflow/core/public/session.h"
#include "tensorflow/core/platform/env.h"
#include "tensorflow/core/framework/tensor.h"
#include "tensorflow/cc/ops/standard_ops.h"
#include "tensorflow/cc/client/client_session.h"

#include "stdlib.h"

using namespace tensorflow;

int main() {

	
	std::vector<float> vec= {};

	//新建session
	Session* session;
	Status status = NewSession(SessionOptions(), &session);
	if (!status.ok()) {
		std::cout << status.ToString() << "\n";
		return 1;
	}

	//读取创建的pb文件中的graph
	GraphDef graph_def;
	status = ReadBinaryProto(Env::Default(), "model1_pb", &graph_def);
	if (!status.ok()) {
		std::cout << status.ToString() << "\n";
		return 1;
	}

	//将读取的graph添加到session中
	status = session->Create(graph_def);
	if (!status.ok()) {
		std::cout << status.ToString() << "\n";
		return 1;
	}

	//设置输入输出
	Tensor x_t(DT_FLOAT, TensorShape({1,1066}));
	int ndim = vec.size();
	Tensor X(DT_FLOAT, TensorShape({ 1,ndim }));
	auto x_map = X.tensor<float, 2>();
	for (int j = 0; j < ndim; j++) {
		x_map(0, j) = vec[j];
	}

	Tensor keep_prob(DT_FLOAT, TensorShape());
	keep_prob.scalar<float>()() = 1.0;

	Tensor batch_norm(DT_BOOL, TensorShape());
	batch_norm.scalar<bool>()() = false;

	std::vector<std::string>labels;
	labels.emplace_back("output/label");
	std::vector<Tensor>label;

	std::vector<std::pair<string, Tensor>>inputs = { {"input",X },
	{"keep_prob",keep_prob },{ "batch_norm",batch_norm } };

	//Run the session ， evaluation operation from the graph
	status = session->Run(inputs, labels, {}, &label);
	if (!status.ok()) {
		std::cout << status.ToString() << "\n";
		return 1;
	}

	打印结果
	auto output_c = label[0].scalar<int>();
	std::cout << label[0].DebugString() << "\n";
	std::cout << output_c() << "\n";

	//关闭会话
	session->Close();
	return 0;

	// 如果程序编译没有出错，运行时出错。一般是输入数据的名字与模型placehold不一样导致的，输出名字一定要和模型输出数据结点一致，
	// 还有一种情况是输入数据的维度和pb文件定义的输入维度不一致。
}



三、编译好的动态库使用
1、初始化
	1.1、函数原型
		int *WhistleIdentificationInit(char *szPbFile)
	1.2、使用说明
		输入：pb文件
		输出：无
		返回：无
		说明：此函数主要是用于新建会话，以及加载计算图结构，以及一些初始化。
2、直接对文件推理
	2.1、函数原型
		int WhistleIdentificationRunFile(char *wave)
	2.2、使用说明
		输入：wav文件
		输出：无
		返回：标签
		说明：此函数主要是用于wav文件直接进行推理，输入一个wav文件，返回一个标签。
3、直接对内存中的wav波形进行推理
	3.1、函数原型
		int WhistleIdentificationRunWave(float *wave, int size)
	3.2、使用说明
		输入：wav波形，wav波形占内存大小
		输出：无
		返回：标签
		说明：此函数用于直接对内存中波形进行推理，输入一个wav波形，返回一个标签。
		
四、例程
int inference()
{
	//加载模型以及进行初始化
	char* filePath = "C:\\Users\\ym\\Desktop\\model1_pb";
	WhistleIdentificationInit(filePath);
	
	int label;
	
	//用wav文件进行推理
	char* wave = "C:\\Users\\ym\\Desktop\\声音文件\\新建文件夹\\20181005_221911_tsl.wav";
	label = WhistleIdentificationRunFile(wave);

	//用wav波形进行推理
	float wave[512];
	label = WhistleIdentificationRunWave(wave, sizeof(wave)/sizeof(float));

	//关闭会话
	WhistleIdentificationClose();	

}


参考链接
https://medium.com/jim-fleming/loading-a-tensorflow-graph-with-the-c-api-4caaff88463f
https://medium.com/@shiweili/building-tensorflow-c-shared-library-on-windows-e79c90e23e6e一、编译tensorflow C++ API
	1、window10环境下
		1.1、下载安装 ：Cmake（https://cmake.org/download/） 
					  swig（http://www.swig.org/download.html）
					  Anaconda3-5.2.0（https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/）			  
		2.1、下载tensorflow （https://github.com/tensorflow/tensorflow/tree/v1.6.0-rc0）（tensorflow建议直接下载链接给的这个版本，太新的版本会有错误）
		3.1、打开vs2015 x64本机工具命令提示符，以管理员身份运行 （一定要以管理员身份运行，不然在最后一步会报错）
		4.1、输入命令powershell
		5.1、cd到刚刚下载的tensorflow文件夹tensorflow\contrib\cmake新建一个build文件夹
		6.1、运行命令cmake .. -A x64 DCMAKE_BUILD_TYPE=Release DSWIG_EXECUTABLE=C:\swigwin-3.0.12\swig.exe DPYTHON_EXECUTABLE=C:\Anaconda3\envs\tensorflow\python.exe DPYTHON_LIBRARIES=C:\Anaconda3\envs\tensorflow\libs\python35.lib Dtensorflow_BUILD_PYTHON_BINDINGS=OFF Dtensorflow_ENABLE_GRPC_SUPPORT=OFF Dtensorflow_BUILD_SHARED_LIB=ON  
		-A x64 DCMAKE_BUILD_TYPE=Release （命令中的三个路径根据自己第一步安装的路径进行相应的修改）
		7.1、编译` MSBuild /m:1 /p:CL_MPCount=1 /p:Configuration=Release /p:Platform=x64 /p:PreferredToolArchitecture=x64 ALL_BUILD.vcxproj  /filelogger`（运行这一步时一定要有稳定的VPN，因为要到google下载一些库，否则会编译不成功）
		8.1、安装`MSbuild /m:1 /p:CL_MPCount=1 /p:Configuration=Release /p:Platform=x64 /p:PreferredToolArchitecture=x64 INSTALL.vcxproj /filelogger`（运行完这一步后就会在c:/program下生成一个tensorflow文件夹。）
	
	2、linux环境下（ubuntu16）
		1、下载安装 bazel（下载安装最新版） VPN
		2、下载tensorflow（没有版本要求）
		3、cd 到tensorflow文件夹下运行命令 ./configure
			运行这条命令后会有一些选项，一些需要地址的输入软件地址（比如python地址），其余的一些配置一律选择no，
			如果选择yes会因为一些相应的包没有安装导致报错，也根据自己的实际情况自行选择。一般全部选择no不会报错
		4、运行 bazel build --config=opt //tensorflow:libtensorflow_cc.so

	
  
二、 C++通过上面编译好的DLL调用tensorflow生成的.pb文件。

1、配置环境 
包含目录：
	C:\Program Files\tensorflow\include\tensorflow
	C:\Program Files\tensorflow\include
	C:\Program Files\tensorflow\include\tensorflow\core\protobuf
	C:\Program Files\tensorflow\include\external\nsync\public
	C:\Program Files\tensorflow\include\external\eigen_archive
库目录：
	C:\Program Files\tensorflow\lib

2、程序源码

#include "stdafx.h"

#include "tensorflow/core/public/session.h"
#include "tensorflow/core/platform/env.h"
#include "tensorflow/core/framework/tensor.h"
#include "tensorflow/cc/ops/standard_ops.h"
#include "tensorflow/cc/client/client_session.h"

#include "stdlib.h"

using namespace tensorflow;

int main() {

	
	std::vector<float> vec= {};

	//新建session
	Session* session;
	Status status = NewSession(SessionOptions(), &session);
	if (!status.ok()) {
		std::cout << status.ToString() << "\n";
		return 1;
	}

	//读取创建的pb文件中的graph
	GraphDef graph_def;
	status = ReadBinaryProto(Env::Default(), "model1_pb", &graph_def);
	if (!status.ok()) {
		std::cout << status.ToString() << "\n";
		return 1;
	}

	//将读取的graph添加到session中
	status = session->Create(graph_def);
	if (!status.ok()) {
		std::cout << status.ToString() << "\n";
		return 1;
	}

	//设置输入输出
	Tensor x_t(DT_FLOAT, TensorShape({1,1066}));
	int ndim = vec.size();
	Tensor X(DT_FLOAT, TensorShape({ 1,ndim }));
	auto x_map = X.tensor<float, 2>();
	for (int j = 0; j < ndim; j++) {
		x_map(0, j) = vec[j];
	}

	Tensor keep_prob(DT_FLOAT, TensorShape());
	keep_prob.scalar<float>()() = 1.0;

	Tensor batch_norm(DT_BOOL, TensorShape());
	batch_norm.scalar<bool>()() = false;

	std::vector<std::string>labels;
	labels.emplace_back("output/label");
	std::vector<Tensor>label;

	std::vector<std::pair<string, Tensor>>inputs = { {"input",X },
	{"keep_prob",keep_prob },{ "batch_norm",batch_norm } };

	//Run the session ， evaluation operation from the graph
	status = session->Run(inputs, labels, {}, &label);
	if (!status.ok()) {
		std::cout << status.ToString() << "\n";
		return 1;
	}

	打印结果
	auto output_c = label[0].scalar<int>();
	std::cout << label[0].DebugString() << "\n";
	std::cout << output_c() << "\n";

	//关闭会话
	session->Close();
	return 0;

	// 如果程序编译没有出错，运行时出错。一般是输入数据的名字与模型placehold不一样导致的，输出名字一定要和模型输出数据结点一致，
	// 还有一种情况是输入数据的维度和pb文件定义的输入维度不一致。
}



三、编译好的动态库使用
1、初始化
	1.1、函数原型
		int *WhistleIdentificationInit(char *szPbFile)
	1.2、使用说明
		输入：pb文件
		输出：无
		返回：无
		说明：此函数主要是用于新建会话，以及加载计算图结构，以及一些初始化。
2、直接对文件推理
	2.1、函数原型
		int WhistleIdentificationRunFile(char *wave)
	2.2、使用说明
		输入：wav文件
		输出：无
		返回：标签
		说明：此函数主要是用于wav文件直接进行推理，输入一个wav文件，返回一个标签。
3、直接对内存中的wav波形进行推理
	3.1、函数原型
		int WhistleIdentificationRunWave(float *wave, int size)
	3.2、使用说明
		输入：wav波形，wav波形占内存大小
		输出：无
		返回：标签
		说明：此函数用于直接对内存中波形进行推理，输入一个wav波形，返回一个标签。
		
四、例程
int inference()
{
	//加载模型以及进行初始化
	char* filePath = "C:\\Users\\ym\\Desktop\\model1_pb";
	WhistleIdentificationInit(filePath);
	
	int label;
	
	//用wav文件进行推理
	char* wave = "C:\\Users\\ym\\Desktop\\声音文件\\新建文件夹\\20181005_221911_tsl.wav";
	label = WhistleIdentificationRunFile(wave);

	//用wav波形进行推理
	float wave[512];
	label = WhistleIdentificationRunWave(wave, sizeof(wave)/sizeof(float));

	//关闭会话
	WhistleIdentificationClose();	

}


参考链接
https://medium.com/jim-fleming/loading-a-tensorflow-graph-with-the-c-api-4caaff88463f
https://medium.com/@shiweili/building-tensorflow-c-shared-library-on-windows-e79c90e23e6e
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwODU5MjE2NjYsODA4NTI4OTMwLDEwMz
g5NDMzMzZdfQ==
-->