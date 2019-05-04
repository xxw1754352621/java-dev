# spring boot 内嵌容器启动(本文具体容器为ｔｏｍｃａｔ，ｓｐｉｒｎｇ　ｂｏｏｔ　ｖｅｒｓｉｏｎ　2.0.5.RELEASE)

## 内嵌与独立的区别

### 独立部署的tomcat服务器的启动过程
传统意义上一个独立部署和运行的tomcat服务器的启动可以理解成两个阶段 :

1.tomcat 容器本身的启动;

2.tomcat容器中所部署的web app的启动；

完成了以上两个阶段，我们才能访问到我们所开发的业务逻辑。在这种情况下，web app的部署动作，通常是由系统部署人员通过某种方式在启动服务器前完成的。

### spring boot web应用启动过程和独立部署的tomcat服务器启动过程的不同点
相对于一个独立部署和运行的tomcat服务器，一个缺省配置的spring boot web应用，情况有些不同 ：

tomcat 不再是独立存在的，是被内嵌到应用中的；

web app的部署不是系统部署人员部署的，是spring boot 应用按照某种约定运行时组装和部署的；

在启动spring boot web应用之前，开发人员需要按照 spring boot的规范编码，实现特定的类，打包部署该spring boot web应用，然后才能启动该应用并提供相应的服务。


##使用
引入依赖

    <dependency>  
    <groupId>org.apache.tomcat.embed</groupId>  
    <artifactId>tomcat-embed-core</artifactId>  
    <version>8.5.5</version>  
    </dependency>  
    <dependency>  
    <groupId>org.apache.tomcat.embed</groupId>  
    <artifactId>tomcat-embed-el</artifactId>  
    <version>8.5.5</version>  
    </dependency>  
    <dependency>  
    <groupId>org.apache.tomcat.embed</groupId>  
    <artifactId>tomcat-embed-jasper</artifactId>  
    <version>8.5.5</version>  
    </dependency>  

### Http 服务
    /**
    * http server demo
    */
    public class SimpleTomcatServer {
    static final int port = 9080;
    static final String docBase = "/tmp/tomcat";

    public static void main(String[] args) throws Exception {
        Tomcat tomcat = new Tomcat();
        //注册端口
        tomcat.setPort(port);
        tomcat.setBaseDir(docBase);
        tomcat.getHost().setAutoDeploy(false);

        String contextPath = "/";
        StandardContext context = new StandardContext();
        context.setPath(contextPath);
        context.addLifecycleListener(new Tomcat.FixContextListener());
        //配置Context
        tomcat.getHost().addChild(context);

        //配置　映射ｓｅｒｖｌｅｔ
        tomcat.addServlet(contextPath, "homeServlet", new HomeServlet());
        context.addServletMappingDecoded("/home", "homeServlet");
        //启动
        tomcat.start();
        tomcat.getServer().await();
    }


    }
    class HomeServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("request scheme: " + req.getScheme());
        resp.getWriter().print("hello tomcat");
    }
    }
 输入http://127.0.0.1:9080/home
 ### 总结
 项目代码的运行，就是一个配置一个基础的server.xml（即tomcat目录下的 conf/server.xml)，先配置运行端口，关闭监听端口；然后配置运行的host以及添加一个上下文context，最后就开始运行并开始监听（注：本篇仅讲容器启动流程，大概讲一下ｔｏｍｃａｔ，若对ｔｏｍｃａｔ详细内容有兴趣的同学可以新开一个话题分享ｔｏｍｃａｔ）   

## Spring boot 如何启动内嵌容器

### 初始化配置

引入依赖

         <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${spring.boot.version}</version>
        </dependency>

缺省引入

        <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <version>2.0.5.RELEASE</version>
        <scope>compile</scope>
        </dependency>

＠SpringBootApplication注解含有 @EnableAutoConfiguration

        @Target({ElementType.TYPE})
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @Inherited
        @SpringBootConfiguration
        @EnableAutoConfiguration
        @ComponentScan(
        excludeFilters = {@Filter(
        type = FilterType.CUSTOM,
        classes = {TypeExcludeFilter.class}
        ), @Filter(
        type = FilterType.CUSTOM,
        classes = {AutoConfigurationExcludeFilter.class}
        )}
        )
        public @interface SpringBootApplication

@EnableAutoConfiguration会启动EmbeddedWebServerFactoryCustomizerAutoConfiguration，根据引入的依赖生成不同的ｗｅｂ　ｆａｃｔｏｒｙ

@Configuration
@ConditionalOnWebApplication
@EnableConfigurationProperties(ServerProperties.class)
public class EmbeddedWebServerFactoryCustomizerAutoConfiguration {

	/**
	 * Nested configuration if Tomcat is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Tomcat.class, UpgradeProtocol.class })
	public static class TomcatWebServerFactoryCustomizerConfiguration {

		@Bean
		public TomcatWebServerFactoryCustomizer tomcatWebServerFactoryCustomizer(
				Environment environment, ServerProperties serverProperties) {
			return new TomcatWebServerFactoryCustomizer(environment, serverProperties);
		}

	}

	/**
	 * Nested configuration if Jetty is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Server.class, Loader.class, WebAppContext.class })
	public static class JettyWebServerFactoryCustomizerConfiguration {

		@Bean
		public JettyWebServerFactoryCustomizer jettyWebServerFactoryCustomizer(
				Environment environment, ServerProperties serverProperties) {
			return new JettyWebServerFactoryCustomizer(environment, serverProperties);
		}

	}

	/**
	 * Nested configuration if Undertow is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Undertow.class, SslClientAuthMode.class })
	public static class UndertowWebServerFactoryCustomizerConfiguration {

		@Bean
		public UndertowWebServerFactoryCustomizer undertowWebServerFactoryCustomizer(
				Environment environment, ServerProperties serverProperties) {
			return new UndertowWebServerFactoryCustomizer(environment, serverProperties);
		}

	}

    }

### 启动

SpringApplication.run(ＸＸＸ.class, args)　会调用

ConfigurableApplicationContext run(String... args)方法
        
        public ConfigurableApplicationContext run(String... args) {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        ConfigurableApplicationContext context = null;
        Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList();
        this.configureHeadlessProperty();
        SpringApplicationRunListeners listeners = this.getRunListeners(args);
        listeners.starting();

        Collection exceptionReporters;
        try {
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            ConfigurableEnvironment environment = this.prepareEnvironment(listeners, applicationArguments);
            this.configureIgnoreBeanInfo(environment);
            Banner printedBanner = this.printBanner(environment);
            context = this.createApplicationContext();
            exceptionReporters = this.getSpringFactoriesInstances(SpringBootExceptionReporter.class, new Class[]{ConfigurableApplicationContext.class}, context);
            this.prepareContext(context, environment, listeners, applicationArguments, printedBanner);
            //启动容器
            this.refreshContext(context);
            this.afterRefresh(context, applicationArguments);
            stopWatch.stop();
            if (this.logStartupInfo) {
                (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), stopWatch);
            }

            listeners.started(context);
            this.callRunners(context, applicationArguments);
        } catch (Throwable var10) {
            this.handleRunFailure(context, var10, exceptionReporters, listeners);
            throw new IllegalStateException(var10);
        }

        try {
            listeners.running(context);
            return context;
        } catch (Throwable var9) {
            this.handleRunFailure(context, var9, exceptionReporters, (SpringApplicationRunListeners)null);
            throw new IllegalStateException(var9);
        }
    }


	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
                //初始化容器
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
                //最终启动
				finishRefresh();
			}


ｔｏｍｃａｔ　就是在　refreshContext(context)　方法启动

调用ServletWebServerApplicationContext．onRefresh（）
调用createWebServer()　容器
    
     private void createWebServer() {
        WebServer webServer = this.webServer;
        ServletContext servletContext = this.getServletContext();
        if (webServer == null && servletContext == null) {
             // 获取对应ｂｅａｎＦａｃｔｏｒｙ，ｔｏｍｃａｔ为例，返回ＴｏｍｃａｔＳｅｒｖｌｅｔＷｅｂＳｅｒｖｅｒＦａｃｔｏｒｙ
            ServletWebServerFactory factory = this.getWebServerFactory();
            
            this.webServer = factory.getWebServer(new ServletContextInitializer[]{this.getSelfInitializer()});
        } else if (servletContext != null) {
            try {
                this.getSelfInitializer().onStartup(servletContext);
            } catch (ServletException var4) {
                throw new ApplicationContextException("Cannot initialize servlet context", var4);
            }
        }

        this.initPropertySources();
    }
    
调用ServletWebServerApplicationContext．getWebServerFactory（）生成ＴomcatServletServiceFactory
    
    protected ServletWebServerFactory getWebServerFactory() {
    String[] beanNames = this.getBeanFactory().getBeanNamesForType(ServletWebServerFactory.class);
    if (beanNames.length == 0) {
    throw new ApplicationContextException("Unable to start ServletWebServerApplicationContext due to missing ServletWebServerFactory bean.");
    } else if (beanNames.length > 1) {
    throw new ApplicationContextException("Unable to start ServletWebServerApplicationContext due to multiple ServletWebServerFactory beans : " + StringUtils.arrayToCommaDelimitedString(beanNames));
    } else {
    return (ServletWebServerFactory)this.getBeanFactory().getBean(beanNames[0], ServletWebServerFactory.class);
    }
    }

调用TomcatServletWebServerFactory.getWebServer(ServletContextInitializer... initializers)
启动内嵌ｔｍｏｃａｔ

	public WebServer getWebServer(ServletContextInitializer... initializers) {
        // 这个Tomcat启动器缺省会 ： 
		// 1. 创建一个Tomcat Server，
		// 2. 创建并关联到这个 Tomcat Server上一个 Tomcat Service: 
		//     1 Tomcat Service = 1 Tomcat Engine + N Tomcat Connector
		// 每个Service只能包含一个Servlet引擎（Engine）, 表示一个特定Service的请求处理流水线。
		// 做为一个Service可以有多个连接器，引擎从连接器接收和处理所有的请求，将响应返回给
		// 适合的连接器，通过连接器传输给用户。
		// 用户可以通过实现Engine接口提供自定义引擎，但通常不需要这么做。
		// 3. 在所创建的那一个 Tomcat Engine 上创建了一个 Tomcat Host 。
		// 每个 Virtual Host 虚拟主机和某个网络域名Domain Name相匹配,每个虚拟主机下都可以部署(deploy)一个
		// 或者多个Web App，每个Web App对应于一个Context，有一个Context path。当Host获得一个请求时，
		// 将把该请求匹配到某个Context上。 一个 Tomcat Engine 上面可以有多个 Tomcat Host。
        
		Tomcat tomcat = new Tomcat();
         // 设置ｂａｓｅＤＩｒ，可配置指定路径或者临时目录，缺省临时目录
		File baseDir = (this.baseDirectory != null) ? this.baseDirectory
				: createTempDir("tomcat");
         // 设置tomcat的基础工作目录
		tomcat.setBaseDir(baseDir.getAbsolutePath());
        // 缺省　org.apache.coyote.http11.Http11NioProtocol
		Connector connector = new Connector(this.protocol);
		tomcat.getService().addConnector(connector);
		customizeConnector(connector);
		tomcat.setConnector(connector);
		tomcat.getHost().setAutoDeploy(false);
        // 根据配置参数定制 connector ：端口，uri encoding字符集，是否启用SSL, 是否使用压缩等
		// 缺省情况下端口是 8080, uri encoding 是 utf-8
		configureEngine(tomcat.getEngine());
        // 添加ｃｏｎｎｅｃｔｏｒ，缺省没有
		for (Connector additionalConnector : this.additionalTomcatConnectors) {
			tomcat.getService().addConnector(additionalConnector);
		}
         // 准备 tomcat host 的 StandardContext，对应一个webapp,并将其通过host关联到tomcat
		prepareContext(tomcat.getHost(), initializers);
        // 创建 TomcatEmbeddedServletContainer 并初始化
		// 其中包括调用 tomcat.start()
		return getTomcatWebServer(tomcat);
	}

    protected void prepareContext(Host host, ServletContextInitializer[] initializers) {
            File documentRoot = getValidDocumentRoot();
             // 创建ｗｅｂ应用，一个 Context 对应一个 Web 工程
            TomcatEmbeddedContext context = new TomcatEmbeddedContext();
            if (documentRoot != null) {
                context.setResources(new LoaderHidingResourceRoot(context));
            }
            context.setName(getContextPath());
            context.setDisplayName(getDisplayName());
            context.setPath(getContextPath());
            File docBase = (documentRoot != null) ? documentRoot
                    : createTempDir("tomcat-docbase");
            context.setDocBase(docBase.getAbsolutePath());
            context.addLifecycleListener(new FixContextListener());
            context.setParentClassLoader(
                    (this.resourceLoader != null) ? this.resourceLoader.getClassLoader()
                            : ClassUtils.getDefaultClassLoader());
            resetDefaultLocaleMapping(context);
            addLocaleMappings(context);
            context.setUseRelativeRedirects(false);
            configureTldSkipPatterns(context);
            WebappLoader loader = new WebappLoader(context.getParentClassLoader());
            loader.setLoaderClass(TomcatEmbeddedWebappClassLoader.class.getName());
            loader.setDelegate(true);
            context.setLoader(loader);
            if (isRegisterDefaultServlet()) {
            // 缺省情况下，会注册 Tomcat 的 DefaultServlet,
			// DefaultServlet是Tomcat缺省的资源服务Servlet,用来服务HTML,图片等静态资源
			// 配置信息 :
			// servletClass : org.apache.catalina.servlets.DefaultServlet
			// name : default
			// overridable : true
			// initParameter : debug, 0
			// initParameter : listings, false
			// loadOnStartup : 1
			// servletMapping : / , default

                addDefaultServlet(context);
            }
            if (shouldRegisterJspServlet()) {
                // Spring boot 提供了一个工具类 org.springframework.boot.context.embedded.JspServlet
			// 检测类 org.apache.jasper.servlet.JspServlet 是否存在于 classpath 中，如果存在，
			// 则认为应该注册JSP Servlet。
			// 缺省情况下,不注册(换句话讲,Springboot web应用缺省不支持JSP)
			// 注意 !!! 这一点和使用Tomcat充当外部容器的情况是不一样的，                                  
			// 使用Tomcat作为外部容器的时候，JSP Servlet 缺省是被注册的。
			// 如果想在 Spring boot中支持JSP，则需要将 tomcat-embed-jasper 包加入 classpath 中。
			// 配置信息 :
			// servletClass : org.apache.jasper.servlet.JspServlet
			// name : jsp
			// initParameter : fork, false
			// initParameter : development, false
			// loadOnStartup : 3
			// servletMapping : *.jsp , jsp
			// servletMapping : *.jspx , jsp
                addJspServlet(context);
                // Jasper 把JSP文件解析成java文件，然后编译成JVM可以使用的class文件。
			// 有很多的JSP解析引擎，Tomcat中使用的是Jasper。
			// 下面添加的 Jasper initializer 用于初始化 jasper。
                addJasperInitializer(context);
            }
            context.addLifecycleListener(new StaticResourceConfigurer(context));
            // 合并参数提供的Spring SCI : EmbeddedWebApplicationContext$1,
		// 这是一个匿名内部类，封装的逻辑来自方法 selfInitialize()
		// 和当前servlet容器在bean创建时通过EmbeddedServletContainerCustomizer
		// ServerProperties添加进来的两个Spring SCI :
		// ServerProperties$SessionConfiguringInitializer
		// InitParameterConfiguringServletContextInitializer
		// 注意这里的SCI接口由spring定义，tomcat jar中也包含了一个servlet API规范
		// 定义的SCI接口,这是定义相同的两个接口而非同一个,最终实现了Spring SCI接口的
		// 类的逻辑必须通过某种方式封装成实现了servlet API规范定义的SCI的逻辑才能被
		// 执行
            ServletContextInitializer[] initializersToUse = mergeInitializers(initializers);
            host.addChild(context);
            // 配置context，
		// 1.将Spring提供的SCI封装成Servlet API标准SCI配置到context中去，
		// 通过一个实现了Servlet API标准SCI接口的spring类 TomcatStarter
		// 2.将spring领域的MIME映射配置设置到context中去，
		// 3.将spring领域的session配置设置到context中去，比如 sessionTimeout
            configureContext(context, initializersToUse);
            postProcessContext(context);
        }


创建TomcatEmbeddedServletContainer并初始化


	protected TomcatWebServer getTomcatWebServer(Tomcat tomcat) {
		return new TomcatWebServer(tomcat, getPort() >= 0);
	}

    public TomcatWebServer(Tomcat tomcat, boolean autoStart) {
		Assert.notNull(tomcat, "Tomcat Server must not be null");
		this.tomcat = tomcat;
		this.autoStart = autoStart;
		initialize();
	}

    private void initialize() throws WebServerException {
		TomcatWebServer.logger
				.info("Tomcat initialized with port(s): " + getPortsDescription(false));
		synchronized (this.monitor) {
			try {
				addInstanceIdToEngineName();

				Context context = findContext();
				context.addLifecycleListener((event) -> {
					if (context.equals(event.getSource())
							&& Lifecycle.START_EVENT.equals(event.getType())) {
						// Remove service connectors so that protocol binding doesn't
						// happen when the service is started.
						removeServiceConnectors();
					}
				});

				// Start the server to trigger initialization listeners
				this.tomcat.start();

				// We can re-throw failure exception directly in the main thread
				rethrowDeferredStartupExceptions();

				try {
					ContextBindings.bindClassLoader(context, context.getNamingToken(),
							getClass().getClassLoader());
				}
				catch (NamingException ex) {
					// Naming is not enabled. Continue
				}

                 // 该线程每10秒检查一次tomcat是否接收到结束信号，如果接收到，则结束执行。
                //设置该线程的原因 : 所有的tomcat线程都是daemon线程，如果没有这样一个非daemon线程，
                 // 整个程序会马上退出。
				startDaemonAwaitThread();
			}
			catch (Exception ex) {
				stopSilently();
				throw new WebServerException("Unable to start embedded Tomcat", ex);
			}
		}
	}

     public void start() throws LifecycleException {
        getServer();
        getConnector();
        // 启动  tomcat StandardServer。这里会对相应的 server,service,engine,
		// 分别进行初始化和启动，也就是调用他们的init()方法和 start()方法。
		// 而 connector只执行了初始化 init(),然后被从 service 中删掉。
		// 该被删除的 connector 随后会被添加回来，然后再调用其启动start()方法。
        server.start();
    }

在整个上面的启动过程中，虽然调用 Tomcat 实例的启动方法，但是整个容器tomcatEmbeddedServletContainer只能算是启动了一部分，也就是其中除了Tomcat Connector 之外的其他部分。

此时tomcatEmbeddedServletContainer实例的属性 started仍然为false，表示整个容器处于尚未启动的状态。所以上的逻辑更像是容器创建和初始化的过程。在finishRefresh()方法启动
Tomcat Connector是用来接收来自网络的请求的关键组件。这一部分组件启动之后，整个容器tomcatEmbeddedServletContainer才能接受和处理来自网络的请求从而为用户提供服务，所以所有这些的启动都结束，才能算是整个容器的启动完成，此时started属性才能设置为true。

    protected void finishRefresh() {
        super.finishRefresh();
        WebServer webServer = startWebServer();
        if (webServer != null) {
            publishEvent(new ServletWebServerInitializedEvent(webServer, this));
        }
    }

调用TomcatWebServer．start()启动ｃｏｎｎｅｃｔｉｏｎ， 创建服务器套接字,绑定端口等操作


	public void start() throws WebServerException {
		synchronized (this.monitor) {
			if (this.started) {
				return;
			}
			try {
				addPreviouslyRemovedConnectors();
				Connector connector = this.tomcat.getConnector();
				if (connector != null && this.autoStart) {
					performDeferredLoadOnStartup();
				}
				checkThatConnectorsHaveStarted();
				this.started = true;
				TomcatWebServer.logger
						.info("Tomcat started on port(s): " + getPortsDescription(true)
								+ " with context path '" + getContextPath() + "'");
			}
			catch (ConnectorStartFailedException ex) {
				stopSilently();
				throw ex;
			}
			catch (Exception ex) {
				throw new WebServerException("Unable to start embedded Tomcat server",
						ex);
			}
			finally {
				Context context = findContext();
				ContextBindings.unbindClassLoader(context, context.getNamingToken(),
						getClass().getClassLoader());
			}
		}
	}

其他容器启动流程同理


若对ｔｏｍｃａｔ容器有兴趣或者有了解，可以单独分享，并添加相关链接！