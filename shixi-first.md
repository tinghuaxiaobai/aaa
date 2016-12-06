#git与maven的配置
	git：下载安装包安装即可，当然，首先你需要注册一个github账户。
	maven：安装之后，需要在本地配置环境变量
#git的基本用法
	1，创建本地库在cmd或者终端下输入以下命令行：
	git config --global user.name trigkit4
	git config --global user.email 345823102@qq.com
	当然，这是我的账户信息，你需要将他们换成你自己的。
	创建本地ssh
	这是一种传输代码的方法，速度快又安全。SSH 是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。
	在终端或cmd输入以下命令行：
	ssh-keygen -t rsa -C "345823102@qq.com"
	当然，邮箱依然换成你注册github时所用的邮箱。如下图所示
	
	路径选择 : 使用该命令之后, 会出现提示选择ssh-key生成路径, 这里直接点回车默认即可, 生成的ssh-key在默认路径中;
	密码确认 : 这里我们不使用密码进行登录, 用密码太麻烦;就一路回车下去
	将ssh配置到GitHub中
	在mac os X 下前往文件夹，/Users/自己电脑用户名/.ssh。
	windows应该是（C:\Documents and Settings\Administrator\.ssh （或者 C:\Users\自己电脑用户名\.ssh）中）。
		然后用记事本打开id_rsa.pub，将里面的全部代码复制到github的SSH中。
	id_rsa.pub 文件内容 :
	ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDS0qLtpontavr43AQntX4oBOsg2R3QlWubMYvfgJsIDX6NWd5RaIDLBLEMwIFLDcpvpQKvk5S/bTy4vTuWqeY6fkQ/tZBKksQn1WuYDcSfjLF8BuPMfdkboTh9NaKESKnsiWdranEVbmB5vOAHm8T+HFGdzG7Tz4ygzCnTwvdpBYrd/4jgeazws2d7CuMeuyb+FxdDTQy9YmJJm+82ypfR//bLyzRJo3SYadnPO3VdOAZCO1Isky+p/0nNN/obC4t2y2+oHBAqJV9h37f9S8UShgDmZoVLicRi4poni0i70xj+t/hnOsT8fmEc+vM9USyN+ndawz2oWjikKgln1jOB 345823102@qq.com
	登陆github网站，点击Settings——SSH keys——点击右侧的Add SSH key ，接下去你懂得。
	
	验证是否配置成功 :
	复制如下代码:
	ssh -T git@github.com
	然后出现如下信息：
	Warning: Permanently added the RSA host key for IP address '192.30.252.131' to the list of known hosts.
	
	Hi hawx1993! You've successfully authenticated, but GitHub does not provide shell access.
	验证时可能让你输入YES，当出现以上信息时，说明配置成功，可以连接上GitHub;
		第一步，在本地创建一个版本库，代码如下：
	$ mkdir test   #test是你的文件夹的名字
	$ cd test     #进入test所在目录
	$ pwd        #pwd命令用于显示当前目录    
	/Users/trigkit4/test #这是在我的Mac上的目录
	第二步，通过git init命令把这个目录变成Git可以管理的仓库：
	$ git init
	然后会输出以下信息：
	Initialized empty Git repository in /Users/trigkit4/banner/.git/
	这里的.git目录是Git用来跟踪管理版本库的，默认是隐藏的。
	第三部，接着，在github上创建一个你自己的new repository，然后下一步，
	mkdir test  
	cd test   
	git init    
	# initialize your git repository  
	touch README  
	# create a file named README  
	git add README    
	# add README to cache  
	git commit -m 'first commit'  
	# commit your files to local repository  
	然后我们将本地的文件传送至github中，使用如下命令：
	git remote add origin https://github.com/yourname/test.git  
	fatal: remote origin already exists.若出现这个信息，则$ git remote rm origin
	
	git push -u origin master  
	从现有仓库克隆
	git clone git://github.com/yourname/test.git
	git clone git://github.com/yourname/grit.git mypro#克隆到自定义文件夹
#intellij idea
	下载连接 https://www.jetbrains.com/idea/#chooseYourEdition
##intellij idea项目创建
	1. Hello World
	
	1.1 创建项目
	
	推荐使用Intellij Idea15进行开发，直接官网下载最新版。 Intellij idea的开源版是免费的，收费版功能插件多些。如果支持正版可以选择购买正版，免费版也能完全满足我们的开发需求。 可以为自己的工具更换下主题，获取主题地址：http://www.riaway.com，所有安装都默认就行。
	
	新建maven工程
	
	 new_project
	
	jdk选择1.7及以上
	
	填写maven信息
	
	 new_project1
	
	GroupId填写为tws.com.cn，ArtifactId填写为你的应用名称，版本号默认即可。
	
	填写项目信息
	
	 new_project2
	1.2 配置pom.xml
	1.3 创建启动类
		创建一个类，继承com.season.core.spring.SeasonApplication，添加main方法

	    public static void main(String[] args){
	        SeasonRunner.run(App.class);
	    }
		启动main方法，即可启动应用。当然现在什么功能也没有，只是启动了应用，应用默认的容器是Tomcat8。
	1.4 创建Controller
		新建HelloController，继承com.season.core.Controller，添加@ControllerKey注解。新建类所属的包名必须是com开始，因 Season 默认配置的 Spring 扫描包为com。

	    @ControllerKey("hello")
	    public class HelloController extends Controller{
	        public void season(){
	            renderText("hi season!");
	        }
	    }
	    重新启动main方法，访问http://localhost:8080/hello/season，即可看见效果。
