---
title: Protocol Buffer 运用
date: 2016-07-22 17:03:39
tags: [Android,Google Protocol Buffer,数据序列化]
categories: [编程,Android] # 配置categories
---


# Protocol Buffer

## Protocol Buffer简介

Protocol buffer是Google推出来的一个序列化结构化数据的库。其提供了一种在各个平台、各个编程语言之间传递结构序列化数据。假使，Android客户端（使用java语言）需要封装一些特定的数据给服务端（使用php或者go），此时通过protocol buffer 就可以将要传递的数据结构化，例如，在java平台就以对象形式，通过 Build模式封装数据等。

>* Protocol buffers are a language-neutral, platform-neutral extensible mechanism for serializing structured data.
>
>* Protocol buffers are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data – think XML, but smaller, faster, and simpler. You define how you want your data to be structured once, then you can use special generated source code to easily write and read your structured data to and from a variety of data streams and using a variety of languages.

官方地址: [https://developers.google.com/protocol-buffers/](https://developers.google.com/protocol-buffers/ "https://developers.google.com/protocol-buffers/")

GitHub地址：[https://github.com/google/protobuf](https://github.com/google/protobuf "https://github.com/google/protobuf")

### 举例说明

例如，现在要在客户端和服务端传递 Person 数据，其包含 id、name、email三个属性。

protocol 协议（各个平台共用）:

	message Person {
	  required string name = 1;
	  required int32 id = 2;
	  optional string email = 3;
	}

然后通过 proto.ext编译，java平台：

	Person john = Person.newBuilder()
	    .setId(1234)
	    .setName("John Doe")
	    .setEmail("jdoe@example.com")
	    .build();
	output = new FileOutputStream(args[0]);
	john.writeTo(output);

ios平台：

	Person john;
	fstream input(argv[1],
	    ios::in | ios::binary);
	john.ParseFromIstream(&input);
	id = john.id();
	name = john.name();
	email = john.email();


## Protocol Buffer运用

本篇文章旨在记录Protocol 库的使用 以及proto.ext即相关jar包的编译。

使用protobuf，需要两个部分：

* install the protocol compiler ，例如安装proto.exe（以windows平台为例，如下若不特殊说明，都以windows平台作说明），其用于编译 .proto文件；
* install the protobuf runtime for your chosen programming language,即安装所选择语言对应的支持库。例如，正对java平台，你需要 依赖 protobuf-java-3.0.0-beta-2.jar

>To install protobuf, you need to install the protocol compiler (used to compile .proto files) and the protobuf runtime for your chosen programming language.

注意：protobuf的compiler和runtime library版本要对应起来。


Protocol Buffer使用一般包含以下几个步骤。

### 1. proto.exe可执行文件的获取(Protocol Compiler Installation)

proto compiler是干什么的？

其就是用来 对 .proto文件进行编译的。

maven仓库获取地址：[protocol compiler protocol可执行文件](http://repo1.maven.org/maven2/com/google/protobuf/protoc/ "http://repo1.maven.org/maven2/com/google/protobuf/protoc/")

当前 protocol compiler使用C++来写的，如果当前非C++的使用者，最简单的办法就是从官网上下载对应的文件
[protobuf releases 下载](https://github.com/google/protobuf/releases "https://github.com/google/protobuf/releases")。

当然你也可以自己编译 protobuf，具体源码也可以在 [protobuf 源码 下载](https://github.com/google/protobuf/releases "https://github.com/google/protobuf/releases")。

### 2. 生成 对应语言 proto 运行时依赖库(Protobuf Runtime Installation)

以java平台为例，生成对应的依赖库，也就是 支持 通过 proto.exe编译 用户 .proto文件生成的java文件的library jar包。

具体的 runntime library源码，可在 [protobuf releases版本目录](https://github.com/google/protobuf/releases "https://github.com/google/protobuf/releases")进行下载。

具体语言对应的 结构化序列化过程都在这个 依赖库中。

下载java平台相关的 protocol runtime library 具体可参考 [Protobuf Runtime Installation for java](https://github.com/google/protobuf/tree/master/java "https://github.com/google/protobuf/tree/master/java")


因为，现在google针对java平台的proto runntime是通过maven进行编译构建的，所以，建议以maven的形式完成对proto。

* step1: 下载protobuf runtime library 源码（注意要和之前编译好的 proto.exe版本一致）

	具体的 runntime library源码，可在 [protobuf releases版本目录](https://github.com/google/protobuf/releases "https://github.com/google/protobuf/releases")进行下载。

	![download proto](http://i.imgur.com/pH5QVAC.png)
	
	
*  step2:解压，并且进入java目录，然后通过命令行 进入对应java目录，并将之前proto.exe放置在 ../src下。
	
	![proto directory](http://i.imgur.com/EAVVcp9.png)

	然后通过命令行进入java目录，此时需要注意几点：

	* 检查proto compiler 二进制文件版本

			D:\UserProfiles\Haiolv\Downloads\protobuf-3.0.0-beta-4\src>protoc --version
	
			libprotoc 3.0.0

	* 将之前已编译好或者下载好的对应版本的proto.exe放置在 ../src下，如下查询记录

			D:\UserProfiles\Haiolv\Downloads\protobuf-3.0.0-beta-4\java>
			D:\UserProfiles\Haiolv\Downloads\protobuf-3.0.0-beta-4\java>cd ../src
			
			D:\UserProfiles\Haiolv\Downloads\protobuf-3.0.0-beta-4\src>dir
			 驱动器 D 中的卷是 Document
			 卷的序列号是 D4A2-99E4
			
			 D:\UserProfiles\Haiolv\Downloads\protobuf-3.0.0-beta-4\src 的目录
			
			2016/07/22  11:13    <DIR>          .
			2016/07/22  11:13    <DIR>          ..
			2016/07/22  11:04    <DIR>          google
			2016/07/20  08:23            50,260 Makefile.am
			2016/07/20  08:24           819,873 Makefile.in
			2016/07/19  05:39         4,060,160 protoc.exe
			2016/07/02  04:43             7,195 README.md
			2016/07/22  11:04    <DIR>          solaris
			               4 个文件      4,937,488 字节
			               4 个目录 100,164,812,800 可用字节
			
			D:\UserProfiles\Haiolv\Downloads\protobuf-3.0.0-beta-4\src>

	* 需要将java目录下maven的构建脚本pom.xml中添加编码格式

		pom.xml类似Ant的build.xml或者gradle的build.gradle配置脚本文件。

		maven构建项目


	* ，否则构建会采用操作系统默认的编码，Windows平台中文的话是GBK，而我们在Android平台采用的是UTF_8。添加部分代码如下：

			<properties> 
	      		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding> 
  			</properties>

		![](http://i.imgur.com/q5WtQgd.png)

	reference：[Maven POM Reference](http://maven.apache.org/pom.html "http://maven.apache.org/pom.html")



* step3:检查当前是否已安装maven，如未安装，请下载安装(注意需要安装JDK1.7及其以上版本)

	maven下载地址(目前最新版本3.3.9)，[http://maven.apache.org/](http://maven.apache.org/ "maven下载地址")。下载之后解压，然后将解压后得到的目录路径添加到系统path中，这样使得可以直接在命令行使用mvn命令。

	可以通过 
		
		mvn --version

	来检查 maven是否正确安装以及其版本。

* step4:Run the tests（运行测试）

Notice:

* step5:构建生成jar包

	通过命令行，生成对应的jar

		mvn package

	[[jar 图]]

	The .jar will be placed in the "target" directory.

	>mvn package:The typical invocation for building a Maven project uses a Maven life cycle phase
	>reference: [http://maven.apache.org/run.html](http://maven.apache.org/run.html "http://maven.apache.org/run.html")

* step6:将生成的jar包安装到本地maven仓库中（可选）

	通过命令行，生成对应的jar

		mvn install

	>mvn install:Just creating the package and installing it in the local repository for re-use from other projects can be done with


### 3. 使用

经过上述两个步骤，我们拿到了 protoc.exe以及 对应版本的 protobuf的支持库 protobuf-java-3.0.0-beta-4.jar。

* step1 : Defining Your Protocol Format(定义要传递的数据机构的协议格式)

	定义业务要传递的数据协议格式，类似定义xml文件格式一样，其相当于定义要传递的数据的结构。但是相比xml定义，proto格式更简单易懂，并且会提供出对数据操作的对应接口。简单来说，其相当于一个数据结构，对应用程序中要交换的数据做一个定义，包括各种字段的声明等等。其文件扩展名为.proto。对于.proto文件如何定义和其语法规范，请详细阅读[Protocol Buffer Language Guide](https://developers.google.com/protocol-buffers/docs/proto3 "Protocol Buffer Language Guide")。这里是proto3版本(最新版本，建议采用)，proto2的也可以参阅[proto2 Language Guide](https://developers.google.com/protocol-buffers/docs/proto)
	
	.proto示例(addressbook.proto)：


		syntax = "proto3";
		package tutorial;
		
		option java_package = "com.example.tutorial";
		option java_outer_classname = "AddressBookProtos";
		// [END java_declaration]
		
		message Person {
		  string name = 1;
		  int32 id = 2;  // Unique ID number for this person.
		  string email = 3;
		
		  enum PhoneType {
		    MOBILE = 0;
		    HOME = 1;
		    WORK = 2;
		  }
		
		  message PhoneNumber {
		    string number = 1;
		    PhoneType type = 2;
		  }
		
		  repeated PhoneNumber phones = 4;
		}
		
		message AddressBook {
		  repeated Person people = 1;
		}


* step2: 编译生成对应语言的api

	在命令行执行：

		protoc -I=$SRC_DIR --java_out=$DST_DIR $SRC_DIR/addressbook.proto

	SRC_DIR：where your application's source code lives – the current directory is used if you don't provide a value；

	DST_DIR：where you want the generated code to go; often the same as $SRC_DIR；

	java_out：Because you want Java classes, you use the --java_out option – similar options are provided for other supported languages.


	例如，在f盘将上述的proto写入一个名叫test.proto文件，然后执行：

		F:\>D:\UserProfiles\Haiolv\Downloads\protobuf-3.0.0-beta-4\src\protoc.exe -I . --java_out=. .\test.proto

	
	在F盘下生成文件：F:\com\example\tutorial\AddressBookProtos.java



		// Generated by the protocol buffer compiler.  DO NOT EDIT!
		// source: addres.proto
		
		package com.example.tutorial;
		
		public final class AddressBookProtos {
		  private AddressBookProtos() {}
		  public static void registerAllExtensions(
		      com.google.protobuf.ExtensionRegistryLite registry) {
		  }
		
		  public static void registerAllExtensions(
		      com.google.protobuf.ExtensionRegistry registry) {
		    registerAllExtensions(
		        (com.google.protobuf.ExtensionRegistryLite) registry);
		  }
		
		  public static com.google.protobuf.Descriptors.FileDescriptor
		      getDescriptor() {
		    return descriptor;
		  }
		  private static  com.google.protobuf.Descriptors.FileDescriptor
		      descriptor;
		  static {
		    java.lang.String[] descriptorData = {
		      "\n\014addres.proto\022\010tutorialB)\n\024com.example." +
		      "tutorialB\021AddressBookProtosb\006proto3"
		    };
		    com.google.protobuf.Descriptors.FileDescriptor.InternalDescriptorAssigner assigner =
		        new com.google.protobuf.Descriptors.FileDescriptor.    InternalDescriptorAssigner() {
		          public com.google.protobuf.ExtensionRegistry assignDescriptors(
		              com.google.protobuf.Descriptors.FileDescriptor root) {
		            descriptor = root;
		            return null;
		          }
		        };
		    com.google.protobuf.Descriptors.FileDescriptor
		      .internalBuildGeneratedFileFrom(descriptorData,
		        new com.google.protobuf.Descriptors.FileDescriptor[] {
		        }, assigner);
		  }
		
		  // @@protoc_insertion_point(outer_class_scope)
		}

然后对数据进行操作，具体的 proto语法可以请详细阅读[Protocol Buffer Language Guide](https://developers.google.com/protocol-buffers/docs/proto3 "Protocol Buffer Language Guide")。这里是proto3版本(最新版本，建议采用)，proto2的也可以参阅[proto2 Language Guide](https://developers.google.com/protocol-buffers/docs/proto)


