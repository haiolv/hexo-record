---
title: Groovy learner.md
date: 2016-07-22 17:03:39
tags: [Android，Groovy, gradle插件]
categories: [编程,Android] # 配置categories
---

# Groovy learner

## gradle插件开发入门

### 开发步骤

* **step1**: 新建一个Android project，之后，新建一个Android Module项目，类型选择Android Library，然后将新建的Module中除了build.gradle文件外的其余文件全都删除，并删除build.gradle文件中的所有内容。
* **step2**: 在刚新建的Module中新建文件夹 **src**，接着在src文件目录下新建main文件夹，在main目录下新建 **groovy** 目录。除此在main目录下新建groovy目录外，你还需要在main目录下新建 **resources** 目录。这时候groovy文件夹会被Android识别为groovy源码目录，同理resources目录会被自动识别为资源文件夹。
* **step3**: 在groovy目录下新建项目包名，如同新建Java包名那样。resources目录下新建文件夹 **META-INF**，META-INF文件夹下新建 **gradle-plugins** 文件夹。

按照上述步骤操作完，你会看到如下目录：

![gradle_directory](http://i.imgur.com/R8P4HIv.png)

从上面开发步骤中，我们得知，在 src/main 下新建groovy目录，代表我们将通过groovy来开发工程，resources代表资源目录。

接下来，我们引用开发gradle插件的apk，并将其在build.gradle中进行配置。

#### 配置 插件Module的build.gradle

打开Module的build.gradle

	apply plugin: 'groovy'
	apply plugin: 'maven'
	
	dependencies {
	    compile gradleApi()
	    compile localGroovy()
	}
	
	repositories {
	    mavenCentral()
	}

从maven中心仓库中引入 gradleApi()以及localGroovy()。

#### 撰写对应的插件入口

在 src/main/groovy 目录下新建 groovy类的包名，如上图 com/haio/groovy,然后新建 PluginImpl.groovy groovy类文件。

	package com.haio.groovy
	
	import org.gradle.api.Plugin
	import org.gradle.api.Project
	
	public class PluginImpl implements Plugin<Project>{
	
	    @Override
	    void apply(Project project) {
	        project.task("testPlugin") << {
	            println "hello gradle plugin"
	        }
	    }
	}

注意：如果需要自己修改，将包名替换成自己写的。

#### 映射入口文件

在 resources/META-INF/gradle-plugins 目录下新建一个properties文件，并将其命名为你到时候想在 其他项目中引用的名称，例如，这里命名成 plugin.test.properties，并在其输入：

	implementation-class=com.haio.groovy.PluginImpl

注意： 
	
* 该文件的命名就是你只有使用插件的名字，这里命名为plugin.test.properties
* implementation-class中引用的类的路径要和 groovy中写的文件一一对应起来，注意包名需要替换为你自己的包名。

此去经年，就完成了一简单的gradle插件，这里面创建了一个 叫 testPlugin的task,执行该 task 会输出 “hello gradle plugin”

### 发布该插件到 本地 仓库

接着，我们需要将插件发布到maven中央仓库，当然此时，我们先将插件发布到本地仓库进行测试，在module项目下的buidl.gradle文件中新加入发布的代码。

	group='cn.haio.gradle.plugin'
	version='1.0.0'
	
	uploadArchives {
	    repositories {
	        mavenDeployer {
	            repository(url: uri('../repo'))
	        }
	    }
	}

完整的 gradleLibarar/build.gradle中的内容如下：

	apply plugin: 'groovy'
	apply plugin: 'maven'
	
	dependencies {
	    compile gradleApi()
	    compile localGroovy()
	}
	
	repositories {
	    mavenCentral()
	}
	
	group='cn.haio.gradle.plugin'
	version='1.0.0'
	
	uploadArchives {
	    repositories {
	        mavenDeployer {
	            repository(url: uri('../repo'))
	        }
	    }
	}

上面中定义的 **group** 和 **version** 的定义会被使用，作为在maven仓库的定位坐标的一部分，group会被作为坐标的groupId，version会被作为坐标的version，而坐标的artifactId组成即当前 **module** 名，我们让其取一个别名moduleName。然后maven本地仓库的目录就是当前项目目录下的repo目录。

此后，在其他Module中引用遵从如下格式

	group:artifactId:version

这时候，同步更新工程，右侧的gradle Toolbar就会在module下多出一个task。

[[upload]]
![upload](http://i.imgur.com/CPEYKXy.png)

然后，点击uploadArchives这个Task，就会在项目下多出一个repo目录，里面存着这个gradle插件。

【【repo】】
![repo](http://i.imgur.com/HbIkeEn.png)

### 使用发布到本地仓库的gradle插件

* **step1:** 在当前工程的app (即准本引用该插件的Module)的android项目下的gradle.build的文件中加入 dependencies。
 
		buildscript {
		    repositories {
		        maven {
		            url uri('../repo')
		        }
		    }
		    dependencies {
		        classpath 'cn.haio.gradle.plugin:gradlelibrary:1.0.0'
		    }
		}
* **step2:** 使用该task,在gradle.build 中apply

		apply plugin: 'plugin.test'

apply plugin后面引号内的名字就是前文plugin.test.properties文件的文件名。而class path后面引号里的内容，就是上面grade中定义的group，version以及moduleName所共同决定的，和maven是一样的。

完整的使用例子如下：

	apply plugin: 'com.android.application'
	
	android {
	    compileSdkVersion 23
	    buildToolsVersion "23.0.2"
	
	    defaultConfig {
	        applicationId "com.haio.gradle"
	        minSdkVersion 15
	        targetSdkVersion 23
	        versionCode 1
	        versionName "1.0"
	    }
	    buildTypes {
	        release {
	            minifyEnabled false
	            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
	        }
	    }
	}
	
	dependencies {
	    compile fileTree(dir: 'libs', include: ['*.jar'])
	    testCompile 'junit:junit:4.12'
	    compile 'com.android.support:appcompat-v7:23.1.1'
	}
	
	buildscript {
	    repositories {
	        maven {
	            url uri('../repo')
	        }
	    }
	    dependencies {
	        classpath 'cn.haio.gradle.plugin:gradlelibrary:1.0.0'
	    }
	}
	apply plugin: 'plugin.test'

同步一下gradle，右侧app下other分类下就会多出一个testTask，双击执行这个Task，控制台就会输出刚才我们输入的字符串。
[[task]]

![task](http://i.imgur.com/m7cvXwg.png)

	....
	Incremental java compilation is an incubating feature.
	:app:testPlugin
	hello gradle plugin
	
	BUILD SUCCESSFUL
	
	Total time: 4.049 secs
	15:38:39: External task execution finished 'testPlugin'.



