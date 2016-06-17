## 前言

作为一个IT技术人员，自然离不开技术的积累，而技术的积累则通过文档或代码的形式呈现出来。好记性不如烂笔头，程序员应该乐于并擅于记录总结工作中遇到的各种问题、工作成果、奇思异想和感悟等。

程序员的世界是孤独的，但也是充满激情和阳光的。正是开源精神点亮了这一切。生命中最美丽的报偿之一便是帮助他人的同时，也帮助了自己 ─ 罗夫‧瓦尔多‧爱默生 


因此，将个人的经验和感悟与他人分享，成就的不仅仅是个人，更可以帮助其他人少走弯路。那么就开始分享的旅程吧！

# 传统的笔记或者blog平台
在本地可以用各种文档格式保存自己的技术积累，也可以采用一些市面上的blog平台作为载体来存储自己的技术文章，但这类方式存在一些弊端，这点你可以打开你的脑洞，想想有哪些，本人不是来挑起舌战的。

# Github pages+Hexo+Nodejs搭建个人blog
好处这里就暂不提了，请自行google。

步骤如下：
1. 安装git：[git下载](https://git-scm.com "git下载")，注意：Windows平台需要下载对应的安装exe,安装之后后续的命令行操作需要在GitShell中打开而非Windows默认的命令行。
2. 安装nodejs：[node.js官网](https://nodejs.org "nodejs官网")，注意：请根据自己主机的平台下载对应版本。
3. 安装hexo及部署：[hexo官网](https://hexo.io/ "hexo官网")，注意：安装 Hexo 相当简单。然而在安装前，您必须确保先完成步骤1和2。Hexo网站可以选择语言为简体中文，方便使用。查看其中的文档可以看到hexo的详细使用说明，so easy！

重点阐述下步骤3的过程(hexo的详细信息可参加[hexo说明文档](https://hexo.io/zh-cn/docs/index.html "hexo说明文档"))：
- 安装hexo，在Gitshell输入：
	
  > $ npm install -g hexo-cli
- 建站

	安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹(自行选定的文件夹folder)中新建所需要的文件。
	
  > $ hexo init <folder>
  > $ cd <folder>
  > $ npm install

	新建完成后，指定文件夹的目录如下：

	.
	├── _config.yml
	├── package.json
	├── scaffolds
	├── source
	|   ├── _drafts
	|   └── _posts
	└── themes

	**_config.yml**
	网站的 配置 信息，您可以在此配置大部分的参数。

	**package.json**
	应用程序的信息。EJS, Stylus 和 Markdown renderer 已默认安装，您可以自由移除。

		package.json
		{
		  "name": "hexo-site",
		  "version": "0.0.0",
		  "private": true,
		  "hexo": {
		    "version": ""
		  },
		  "dependencies": {
		    "hexo": "^3.0.0",
		    "hexo-generator-archive": "^0.1.0",
		    "hexo-generator-category": "^0.1.0",
		    "hexo-generator-index": "^0.1.0",
		    "hexo-generator-tag": "^0.1.0",
		    "hexo-renderer-ejs": "^0.1.0",
		    "hexo-renderer-stylus": "^0.2.0",
		    "hexo-renderer-marked": "^0.2.4",
		    "hexo-server": "^0.1.2"
		  }
		}
	**scaffolds**
	模版 文件夹。当您新建文章时，Hexo 会根据 scaffold 来建立文件。

	**source**
	资源文件夹是存放用户资源的地方。除 _posts 文件夹之外，开头命名为 _ (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 public 文件夹，而其他文件会被拷贝过去。

	**themes**
	[主题](https://hexo.io/docs/themes.html "hexo主题") 文件夹。Hexo 会根据主题来生成静态页面。

- 本地启动server，可以检验安装是否ok。如果能成功，那么恭喜你离成功更近一步了。

	在Gitshell中进入前面指定的folder(建立的站点的根目录)，输入：

	
  > $ hexo server

	可以观察命令行的输出，然后通过浏览器打开[http://localhost:4000/](http://localhost:4000/)，成功的情况下则会看到默认的欢迎页面。

- 部署到github pages([点击了解Github Pages](https://help.github.com/categories/github-pages-basics/ "github pages介绍"))

	Hexo 提供了快速方便的一键部署功能，让您只需一条命令就能将网站部署到服务器上。

	
	> $ hexo deploy

	在开始之前，您必须先在站点的配置文件_config.yml(前面建立的folder目录下)中修改参数，一个正确的部署配置中至少要有 type 参数，例如：
	
		deploy:
  			type: git

	您可同时使用多个 deployer，Hexo 会依照顺序执行每个 deployer。

		deploy:
		- type: git
		  repo:
		- type: heroku
		  repo:


	这里重点提及部署到Github pages的方法。
	**a.首先在Github上面创建一个新的Repository，命名格式为yourname.github.io，注意：格式一定需要符合这个样式。**
	**b.然后，配置文件_config.yml**

		deploy:
		  type: git
		  repository: git@github.com:your_github_account/your_github_account.github.io.git
		  branch: master
		  message: [message]

	说明：

	|参数  |	描述|
	|--------------|   ----------------------------------|
	|repo|	库（Repository）地址|
	|branch|	分支名称。如果您使用的是 GitHub 或 GitCafe 的话，程序会尝试自动检测。|
	|message|	自定义提交信息 (默认为 Site updated: {{ now('YYYY-MM-DD HH:mm:ss') }})|



	eg:

		deploy:
		  type: git
		  repository: git@github.com:cstsinghua/cstsinghua.github.io.git
		  branch: master

	注意：如果是建立项目网站，则branch需要修改为gh-pages，详细情况请参见[Github pages中User, Organization, and Project Pages的差异](https://help.github.com/articles/user-organization-and-project-pages/)

	**c.安装hexo的git插件[hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git "hexo的git插件")**
    
   > $ npm install hexo-deployer-git --save

	**d.部署前面建立的站点(folder下面的内容)：**
	
	执行完，开始部署，即先hexo generate，然后hexo deploy。也可以一步到位：hexo d -g

	
	> $ hexo clean
	$ hexo generate
	$ hexo deploy

	或者：
	
   > $ hexo d -g


注意：在这一步可能报错，已知的一些错误可以参见[hexo git部署常见问题](https://hexo.io/zh-cn/docs/troubleshooting.html#Git-部署问题)

另外，可能遇到SSH publickey接入问题，可以参见[创建github SSH key](https://help.github.com/articles/generating-an-ssh-key/ "创建github ssh key")和[github SSH key管理](https://github.com/settings/keys "github SSH key管理")

这里关于生成ssh-key: [生成github ssh-key](http://blog.csdn.net/hustpzb/article/details/8230454/ "生成github ssh-key")，其需要通过 gitbash来执行命令ssh-keygen,此外，配置key详情见[github配置ssh-key](http://jingyan.baidu.com/album/a65957f4e91ccf24e77f9b11.html?picindex=3 "http://jingyan.baidu.com/album/a65957f4e91ccf24e77f9b11.html?picindex=3")


- (optional)Custom domain redirects for GitHub Pages sites(将独立(个性)域名与GitHub Pages的空间域名绑定)

	yourname.github.io的域名格式比较固定，那么是否可以设置一个个性化的域名呢，另外需要注意的是Github pages的容量受限于github的要求，目前是1GB(请参见[https://help.github.com/articles/what-are-github-pages/](https://help.github.com/articles/what-are-github-pages/))。因此，建立一个独立的个性化(blog)网站(域名是个性化独立的，容量也可以调整)，在某些情况下还是有必要的(请参见[github pages域名重定向](https://help.github.com/articles/custom-domain-redirects-for-github-pages-sites/ "github pages域名重定向"))。

	> Github pages 设置：在Repository的根目录下面，新建一个名为CNAME的文本文件，里面写入你要绑定的域名，如cstsinghua.me。

	> DNS设置：注册DNSpod，添加域名，不是必要的步骤，但是据说可以提高解析效率。本人没有测试。

    > 在域名服务商，如net.cn中修改增加两条A记录，指向github pages 提供的 ip
	        192.30.252.153
	        192.30.252.154


## 具体部署使用

参考 [搭建hexo博客](https://yq.aliyun.com/articles/33519 "https://yq.aliyun.com/articles/33519")

![poor](http://i.imgur.com/9ZNXK65.jpg)


