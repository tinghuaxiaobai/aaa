#season介绍
 1. SeasonApplication

 

		SeasonApplication为Spring Boot的启动基类，用于对整个应用的配置，Season项目必须要继承该类。
	    SeasonApplication的子类可以定义Bean、Listener、Filter等，具体参考Spring Boot官方文档。除了Spring Boot的配置，SeasonApplication的子类还可以覆盖父类的相关方法进行其他配置：
	
	    /**
	     * 配置一些常量
	     */
	    public void configConstant(Constants me){}
	    /**
	     * 配置插件
	     */
	    public void configPlugin(Plugins me){}
	    /**
	     * 配置全局拦截器
	     */
	    public void configInterceptor(Interceptors me){}
	    /**
	     * 配置Handler
	     */
	    public void configHandler(Handlers me){}
	    /**
	     * 应用启动后回调
	     */
	    public void afterStart(){}
	    /**
	     * 应用停止前回调
	     */
	    public void beforeStop(){}
	    /**
	     * 指定SlaverDao数据源
	     */
	    protected void setSlaverDao(SlaverDao slaverDao){}
		 
2.1 configConstant
-
	  public void configConstant(Constants me){
        //是否输出Action日志
        me.setShowAction(constantsProperties.getShowAction());
        //是否输出SQL
        me.setShowSql(constantsProperties.getShowSql());
        me.setBaseViewPath(constantsProperties.getBaseViewPath());
        me.setEncoding(constantsProperties.getEncoding());
        me.setError401View(constantsProperties.getError401View());
        me.setError403View(constantsProperties.getError403View());
        me.setError404View(constantsProperties.getError404View());
        me.setError500View(constantsProperties.getError500View());
        me.setMaxPostSize(constantsProperties.getMaxPostSize());
        me.setBeetlViewExtension(constantsProperties.getBeetlViewExtension());
        if(StrKit.isNotEmpty(constantsProperties.getFileRenderPath())){
            me.setFileRenderPath(constantsProperties.getFileRenderPath());
        }
        if(StrKit.isNotEmpty(constantsProperties.getUploadedFileSaveDirectory())){
            me.setUploadedFileSaveDirectory(constantsProperties.getUploadedFileSaveDirectory());
        }
        me.setUrlParaSeparator(constantsProperties.getUrlParaSeparator());

    }
	这是父类中的configConstant，可以看见父类中已经对很多常量已经进行了默认设置，设置的值都可在application.properties中配置。
如果子类中要配置configConstant，最好调用下父类的configConstant再进行自己的配置，可以进行一些默认配置，而且只有在调用了父类的configConstant，在application.properties中配置才能生效。

    public void configConstant(Constants me) {
        super.configConstant(me);
        //再进行自己的设置 me.SetXXXX
    }
2.2 configPlugin
-
插件配置，Season中的插件都需要手动添加到配置中才能使用，比如如下代码，添加TRSServer插件和Redis插件：

    @Override
    public void configPlugin(Plugins me) {
        TRSServerPlugin trsServerPlugin = new TRSServerPlugin();
        me.add(trsServerPlugin);//添加后，可使用TSKit工具类操作TRSServer
        RedisPlugin redisPlugin = new RedisPlugin();
        me.add(redisPlugin);
    }
2.3 configInterceptor
-
全局拦截器配置，全局拦截器将拦截所有 action 请求,除非使用 @Clear 在 Controller 中清除,如下代码配置了名为 IDSInterceptor 的拦截器。

    @Autowired
    IDSInterceptor idsInterceptor;//spring注入

    public void configInterceptor(Interceptors me){
         me.addGlobalActionInterceptor(idsInterceptor);
     }
如果IDSInterceptor中使用了Spring相关的注入，那么添加IDSInterceptor时也得使用Spring注入的方式添加。
Interceptor 配置粒度分为 Global、Class、Method 三个层次,其中以上代码配置粒度为全局。Class 与 Method 级的 Interceptor 配置将在后续章节中详细介绍。

2.4 configHandler
-
此方法用来配置 Season 的 Handler,如下代码配置了名为 PathHandler 的处理器,Handler 可以接管所有 web 请求,并对应用拥有完全的控制权,可以很方便地实现更高层的功能性扩展。

    @Autowired
    PathHandler pathHandler;

    public void configHandler(Handlers me) {
        me.add(pathHandler);
    }
2.5 afterStart和beforeStop
-
Season 会在系统启动完成后回调 afterStart()方法,会在系统关闭前回调 beforeStop()方法。这两个方法可以很方便地在项目启动后与关闭前让开发者有机会进行 额外操作,如在系统启动后创建调度线程或在系统关闭前写回缓存。

2.6 setSlaverDao
-
Season默认提供Dao和SlaverDao两个操作数据库的Dao，如果不进行任何配置他们默认操作的是同一个数据源。SlaverDao因为是为查询从库设计的，所以只提供数据库查询相关的方法。setSlaverDao可以指定SlaverDao的数据源，这样调用的时候比较简单，不用再使用SlaverDao.use()指定数据源。

    //设置slaverDao的数据源
    @Override
    protected void setSlaverDao(SlaverDao slaverDao){
        slaverDao.setDsName("slaver");
    }
3. Controller
-
Controller是Season核心类之一,该类作为MVC模式中的控制器。基于Season的Web应用的控制器需要继承该类。Controller是定义 Action方法的地点,是组织Action的一种方式,一个Controller可以包含多个Action。Controller是线程安全的。

3.1 Action
-
Controller以及在其中定义的public无参方法称为一个Action。Action是请求的最小单位。Action 方法必须在 Controller 中声明,该方法必须是 public 可见性且没有形参。Controller上必须添加ControllerKey注解，用于指定nameSpace和viewPath，nameSpace必填，如果没有指定viewPath，则viewPath就是nameSpace。同时Spring Bean的名称也为nameSpace。

    @ControllerKey("hello")
    public class HelloController extends Controller { 
        public void index() {
            renderText("此方法是一个action"); 
        }
        @ActionKey("test2")
        public void test() { 
            renderText("此方法是一个action");
        } 
    }
以上代码中定义了两个 Action:HelloController.index()、HelloController.test()。 @ActionKey用于指定Action的名称，如果没有添加@ActionKey注解，默认的Action名称为方法名称，默认的如果方法名为index对应的Action名为“/”。比如上述例子，访问URL就分别为：/contextPath/hello/、/contextPath/hello/test2。 在 Controller 中提供了getPara系列方法setAttr方法以及render系列方法供Action使用。

3.2 getPara系列方法
-
Controller提供了getPara系列方法用来从请求中获取参数。getPara系列方法分为两种类型。
第一种类型为第一个形参为String的getPara系列方法。该系列方法是对HttpServletRequest.getParameter(Stringname)的封装,这类方法都是转调了HttpServletRequest.getParameter(Stringname)。
第二种类型为第一个形参为int或无形参的getPara系列方法。该系列方法是去获取urlPara中所带的参数值。getParaMap与getParaNames分别对应HttpServletRequest的getParameterMap与getParameterNames。

记忆技巧:第一个参数为 String 类型的将获取表单或者 url 中问号挂参的域值。第一个参数为 int 或无参数的将获取 urlPara 中的参数值。

getPara 使用例子:
-
方法调用	-----------------------------------返回值  

getPara("title")-----------------------返回页面表单域名为"title"参数值
getParaToInt("age")	-------------------返回页面表单域名为"age"的参数值并转为 int 型

getPara(0)	---------------返回 url 请求中的 urlPara 参数的第一个值,如 http://localhost/controllerKey/method/v0-v1-v2 这个请求将 返回"v0"

getParaToInt(1)	-------------返回 url 请求中的 urlPara 参数的第二个值并转换成 int 型,如 http://localhost/controllerKey/method/2-5-9 这个请求将返回 5

getParaToInt(2)	---------------如http://localhost/controllerKey/method/2-5-N8 这个 请求将返回 -8。注意:约定字母 N 与 n 可以表示负 号,这对 urlParaSeparator 为 "-" 时非常有用。

getPara()------------------------	返回 url 请求中的 urlPara 参数的整体值,如 http://localhost/controllerKey/method/v0-v1-v2 这个 请求将返回"v0-v1-v2"

3.3 setAttr方法
-
setAttr(String, Object)转调了HttpServletRequest.setAttribute(String, Object),该方法可以将各种数据传递给 View 并在 View 中显示出来。

3.4 getFile系列文件上传方法
-
Controller提供了getFile系列方法支持文件上传。

特别注意:如果客户端请求为multipart request(form表单使用了enctype="multipart/form-data"),那么必须先调用getFile系列方法才能使getPara系列方法正常工作,因为multipartrequest需要通过getFile系列方法解析请求体中的数据,包括参数。

文件默认上传至项目根路径下的upload子路径之下,该路径称为文件上传基础路径。
可以在configConstant(Constantsme me)方法中通过me.setUploadedFileSaveDirectory(uploadedFileSaveDirectory)设置文件上传基础路径，或在application.properties中设置season.constants.uploadedFileSaveDirectory的值， 该路径参数接受以”/”打头或者以windows磁盘盘符打头的绝对路径,即可将基础路径指向项目根径之外,方便单机多实例部署。当该路径参数设置为相对路径时,则是以项目根为基础的相对路径。
当然Controller中提供了多个getFile方法，也可以直接指定文件的保存路径。

    //前台表单的文件上传的input的name为asyncFile，保存路径为sFileSavePath
    UploadFile sUploadFile = getFile("asyncFile",sFileSavePath);
3.5 renderFile文件下载
-
Controller提供了renderFile系列方法支持文件下载。
文件默认下载路径为项目根路径下的download子路径之下,该路径称为文件下载基础路 径。可以在configConstant(Constantsme me)方法中通过me.setFileRenderPath(fileRenderPath)设置文件下载基础路径,该路径参数接受以”/”打头或者以windows磁盘盘符打头的绝对路径,即可将基础路径指向项目根径之外,方便单机多实例部署。当该路径参数设置为相对路径时,则是以项目根为基础的相对路径。

3.6 session操作方法
-
通过 setSessionAttr(key, value)可以向 session 中存放数据,getSessionAttr(key)可以从 session中读取数据。还可以通过getSession()得到session对象从而使用全面的session API。

3.7 render系列方法
-
render系列方法将渲染不同类型的视图并返回给客户端。
Season目前支持的视图类型有:Httl、Beetl、JSON、File、Text、Html等等。除了Season支持的视图型以外,还可以通过继承Render抽象类来无限扩展视图类型。

通常情况下使用Controller.render(String)方法来渲染视图,使用Controller.render(String)时的视图类型由configConstant(Constantsconstants me)配置中的constants.setViewType(ViewType)来决定,该设置方法支持的ViewType有:Httl、Beetl,不进行配置时的缺省配置为Beetl。

此外,还可以通过constants.setMainRenderFactory(IMainRenderFactory)来设置 Controller.render(String)所使用的视图,IMainRenderFactory专门用来对Controller.render(String)方法扩展除了Httl、Beetl之外的视图。

视图模板的路径可以通过 configConstant(Constantsconstants me)配置中的constants.setBaseViewPath(baseViewPath)设置基础路径，当然也可在application.properties中设置season.constants.baseViewPath的值。

假设在上面例子的HelloController中，viewPath为hello，baseViewPath设置为/templates，render(Stringview)使用例子:

方法调用	-------------------------------描述

render(”test.html”)	----------------渲染名为 test.html 的视图,该视图的全路径 为”/templates/hello/test.html”

render(”/other_path/test.html”)---	渲染名为 test.html 的视图,该视图的全路径 为”/other_path/test.html”,即当参数以”/”开头时将 采用绝对路径。
其它 render 方法使用例子:

方法调用--------------------------------	描述

renderBeetl(”test.html”)--------------	渲染名为 test.html 的视图,且视图类型为 Beetl

renderJson()------------------------	将所有通过 Controller.setAttr(String, Object)设置 的变量转换成 json 数据并渲染。


renderJson(“users”, userList)	---------以”users”为根,仅将 userList 中的数据转换成json 数据并渲染。

renderJson(user)	-------------------------将 user 对象转换成 json 数据并渲染。

renderJson(“{\”age\”:18}” )	--------------直接渲染 json 字符串。

renderJson(new String[]{“user”, “blog”})---	仅将 setAttr(“user”, user)与 setAttr(“blog”, blog)设 置的属性转换成 json 并渲染。使用 setAttr 设置的 其它属性并不转换为 json。

renderFile(“test.zip”);-----------------------	渲染名为 test.zip 的文件,一般用于文件下载

renderText(“Hello Season”)	----------------渲染纯文本内容”Hello Season”。

renderHtml(“Hello Html”)---------------------	渲染 Html 内容”Hello Html”。

renderError (404 , “test.html”)	-------------渲染名为 test.html 的文件,且状态为 404。

renderError (500 , “test.html”)---------------	渲染名为 test.html 的文件,且状态为 500。

renderNull()	-----------------------------------不渲染,即不向客户端返回数据。

除 renderError 方法以外,在调用 render 系列的方法后程序并不会立即返回,如果需要立即 返回需要使用 return 语句。在一个 action 中多次调用 render 方法只有最后一次有效。
4.AOP

传统AOP实现需要引入大量繁杂而多余的概念,例如:Aspect、Advice、Joinpoint、Poincut、Introduction、Weaving、Around等等,并且需要引入IOC容器并配合大量的XML或者annotation来进行组件装配。
传统AOP不但学习成本极高,开发效率极低,开发体验极差,而且还影响系统性能,尤其是在开发阶段造成项目启动缓慢,极大影响开发效率。
Season采用极速化的AOP设计,专注AOP最核心的目标,将概念减少到极致,仅有三个概念:Interceptor、Around、Clear,并且无需引入IOC也无需使用繁杂的XML。

4.1 Interceptor

Interceptor 可以对方法进行拦截,并供机会在方法的前后添加切面代码,实现 AOP 的 核心目标。Interceptor 接口仅仅定了一个方法 void intercept(Invocation inv)。以下是简单的示例:

        public class DemoInterceptor implements Interceptor { 
            public void intercept(Invocation inv) {
                System.out.println("Before method invoking"); 
                inv.invoke();
                System.out.println("After method invoking");
            } 
        }
以上代码中的 DemoInterceptor 将拦截目标方法,并且在目标方法调用前后向控制台输出 文本。inv.invoke()这一行代码是对目标方法的调用,在这一行代码的前后插入切面代码可以很 方便地实现 AOP。

Invocation 作为 Interceptor 接口 intercept 方法中的唯一参数, 供了很多便利的方法在拦 截器中使用。以下为 Invocation 中的方法:

方法	描述
void invoke()	传递本次调用,调用剩下的拦截器与目标方法
Controller getController()	获取 Action 调用的 Controller 对象(仅用于控制层拦截)
String getActionKey()	获取 Action 调用的 action key 值(仅用于控制层拦截)
String getControllerKey()	获取 Action 调用的 controller key 值(仅用于控制层拦截)
String getViewPath()	获取 Action 调用的视图路径(仅用于控制层拦截)
T getTarget()	获取被拦截方法所属的对象
Method getMethod()	获取被拦截方法的 Method 对象
String getMethodName()	获取被拦截方法的方法名
Object[] getArgs()	获取被拦截方法的所有参数值
Object getArg(int)	获取被拦截方法指定序号的参数值
T getReturnValue()	获取被拦截方法的返回值
void setArg(int)	设置被拦截方法指定序号的参数值
void setReturnValue(Object)	设置被拦截方法的返回值
boolean isActionInvocation()	判断是否为 Action 调用,也即是否为控制层拦截
4.2 @Around

@Around 注解用来对拦截器进行配置,该注解可配置 Class、Method 级别的拦截器,以下是 代码示例:

     // 配置一个Class级别的拦截器,她将拦截本类中的所有方法 
    @Around(AaaInter.class)
    public class BlogController extends Controller {
        // 配置多个Method级别的拦截器,仅拦截本方法 
        @Around({BbbInter.class, CccInter.class}) 
        public void index() {
        }
        // 未配置Method级别拦截器,但会被Class级别拦截器AaaInter所拦截 
        public void show() {
    }
如上代码所示,@Around 可以将拦截器配置为 Class 级别与 Method 级别,前者将拦截本类 中所有方法,后者仅拦截本方法。此外 @Around 可以同时配置多个拦截器,只需用在大括号内用逗号将多个拦截器进行分隔即可。

除了 Class 与 Method 级别的拦截器以外,Season 还支持全局拦截器,全局拦截器分为控制层全局拦截器与业务层全局拦截器,前者拦截控制 层所有 Action 方法,后者拦截业务层所有方法。 全局拦截器需要在 YourSeasonApplication 进行配置,以下是配置示例:

    public class App extends SeasonApplication { 
        public void configInterceptor(Interceptors me) {
            // 添加控制层全局拦截器
            me.addGlobalActionInterceptor(new GlobalActionInterceptor());
            // 添加业务层全局拦截器
            me.addGlobalServiceInterceptor(new GlobalServiceInterceptor());
        }
    }
当某个 Method 被多个级别的拦截器所拦截,拦截器各级别执行的次序依次为:Global、Class、Method,如果同级中有多个拦截器,那么同级中的执行次序是:配置在前面的 先执行。

4.3 @Clear

拦截器从上到下依次分为 Global、Class、Method 三个层次,@Clear 用于清除自身 所处层次以上层的拦截器。

@Clear 声明在 Method 层时将针对 Global、Class 进行清除。@Clear 声明在 Class 层时 将针对 Global 进行清除。@Clear 注解携带参数时清除目标层中指定的拦截器。

@Clear 用法记忆技巧:

共有 Global、Class、Method 三层拦截器
清除只针对 @Clear 本身所处层的向上所有层,本层与下层不清除
不带参数时清除所有拦截器,带参时清除参数指定的拦截器
在某些应用场景之下,需要移除 Global 或 Class 拦截器。例如某个后台管理系统,配置了 一个全局的权限拦截器,但是其登录 action 就必须清除掉她,否则无法完成登录操作,以下是 代码示例:

    // login方法需要移除该权限拦截器才能正常登录 
    @Around(AuthInterceptor.class)
    public class UserController extends Controller {
        // AuthInterceptor 已被Clear清除掉,不会被其拦截 
        @Clear
        public void login() {
        }
        // 此方法将被AuthInterceptor拦截 
        public void show() {
        }
    }
@Clear 注解带有参数时,能清除指定的拦截器,以下是一个更加全面的示例:

    @Around(AAA.class)
    public class UserController extends Controller {
        @Clear 
        @Around(BBB.class) 
        public void login() {
            // Global、Class级别的拦截器将被清除,但本方法上声明的BBB不受影响 
        }
        @Clear({AAA.class, CCC.class})// 清除指定的拦截器AAA与CCC 
        @Around(CCC.class)
        public void show() {
            // 虽然Clear注解中指定清除CCC,但她无法被清除,因为Method级的拦截器无法被清除 }
        }
    }

