# 参考文章

- [深入学习spring源码，应该买什么书？](https://www.zhihu.com/question/400486014) 

![截图](E:\Coding\article\spring\images\3-1.png)

![Spring IoC 容器类图](E:\Coding\article\spring\images\3-2.webp)

其中 BeanFactory 作为最顶层的一个接口类，它定义了 IOC 容器的基本功能规范BeanFactory 有三 个重要的子类：ListableBeanFactory、HierarchicalBeanFactory 和 AutowireCapableBeanFactory。

但是从类图中我们可以发现最终的默认实现类是 DefaultListableBeanFactory，它实现了所有的接口。

> 那为何要定义这么多层次的接口呢？

查阅这些接口的源码和说明发现，每个接口都有它使用的场合，它主要是为了区分在 Spring 内部在操作过程中对象的传递和转化过程时，对对象的数据访问所做的限制。

例如 ListableBeanFactory 接口表示这些 Bean 是可列表化的，而 HierarchicalBeanFactory 表示的是这些 Bean 是有继承关系的，也就是每个 Bean 有可能有父 Bean。

AutowireCapableBeanFactory 接口定义 Bean 的自动装配规则。这三个接口共同定义了 Bean 的集合、Bean 之间的关系、以及 Bean 行为。





**关于 FactoryBean 和 BeanFactory**

在 Spring 中，有两个很容易混淆的类：**BeanFactory** 和 **FactoryBean**。

**BeanFactory**：Bean 工厂，是一个工厂(Factory)，我们 Spring IOC 容器的最顶层接口就是这个BeanFactory，它的作用是管理 Bean，即实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。

**FactoryBean**：工厂 Bean，是一个 Bean，作用是产生其他 bean 实例。通常情况下，这种 Bean 没有什么特别的要求，仅需要提供一个工厂方法，该方法用来返回其他 Bean 实例。通常情况下，Bean 无须自己实现工厂模式，Spring 容器担任工厂角色；但少数情况下，容器中的 Bean 本身就是工厂，其作用是产生其它 Bean 实例。

当用户使用容器本身时，可以使用转义字符”&”来得到 FactoryBean 本身，以区别通过 FactoryBean产生的实例对象和 FactoryBean 对象本身。在 BeanFactory 中通过如下代码定义了该转义字符：

String FACTORY_BEAN_PREFIX = "&";

如果 myJndiObject 是一个 FactoryBean，则使用&myJndiObject 得到的是 myJndiObject 对象，而不是 myJndiObject 产生出来的对象。



SpringIOC 容器对 Bean 配置资源的载入是从 refresh()函数开始的，refresh()是一个模板方法，规定了IOC 容 器 的 启 动 流 程 ， 有 些 逻 辑 要 交 给 其 子 类 去 实 现 。

它 对 Bean 配 置 资 源 进 行 载 入 ClassPathXmlApplicationContext 通过调用其父类 AbstractApplicationContext 的 refresh()函数启动整个 IOC 容器对 Bean 定义的载入过程，现在我们来详细看看 refresh()中的逻辑处理：

```
@Override public void refresh() throws BeansException, IllegalStateException { 
    synchronized (this.startupShutdownMonitor) { 
    // Prepare this context for refreshing. 
    //1、调用容器准备刷新的方法，获取容器的当时时间，同时给容器设置同步标识 
    prepareRefresh(); 
    // Tell the subclass to refresh the internal bean factory. 
    //2、告诉子类启动 refreshBeanFactory()方法，Bean 定义资源文件的载入从 
    //子类的 refreshBeanFactory()方法启动 
    ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory(); 
    // Prepare the bean factory for use in this context. 
    //3、为 BeanFactory 配置容器特性，例如类加载器、事件处理器等 
    prepareBeanFactory(beanFactory); 
    try { 
        // Allows post-processing of the bean factory in context subclasses. 
        //4、为容器的某些子类指定特殊的 BeanPost 事件处理器 
        postProcessBeanFactory(beanFactory); 
        // Invoke factory processors registered as beans in the context. 
        //5、调用所有注册的 BeanFactoryPostProcessor 的 Bean 
        invokeBeanFactoryPostProcessors(beanFactory); 
        // Register bean processors that intercept bean creation. 
        //6、为 BeanFactory 注册 BeanPost 事件处理器. 
        //BeanPostProcessor 是 Bean 后置处理器，用于监听容器触发的事件 
        registerBeanPostProcessors(beanFactory); 
        // Initialize message source for this context. 
        //7、初始化信息源，和国际化相关. 
        initMessageSource(); 
        // Initialize event multicaster for this context. 
        //8、初始化容器事件传播器. 
        initApplicationEventMulticaster(); 
        // Initialize other special beans in specific context subclasses. 
        //9、调用子类的某些特殊 Bean 初始化方法 
        onRefresh(); 
        // Check for listener beans and register them. 
        //10、为事件传播器注册事件监听器.
        registerListeners(); 
        // Instantiate all remaining (non-lazy-init) singletons. 
        //11、初始化所有剩余的单例 Bean 
        finishBeanFactoryInitialization(beanFactory); 
        // Last step: publish corresponding event. 
        //12、初始化容器的生命周期事件处理器，并发布容器的生命周期事件 
        finishRefresh(); 
    }catch (BeansException ex) { 
        if (logger.isWarnEnabled()) { 
        logger.warn("Exception encountered during context initialization - " + 
        "cancelling refresh attempt: " + ex); 
        }
    // Destroy already created singletons to avoid dangling resources. 
    //13、销毁已创建的 Bean 
    destroyBeans(); 
    // Reset 'active' flag. 
    //14、取消 refresh 操作，重置容器的同步标识. 
    cancelRefresh(ex); 
    // Propagate exception to caller. 
    throw ex; 
    }finally { 
        // Reset common introspection caches in Spring's core, since we 
        // might not ever need metadata for singleton beans anymore... 
        //15、重设公共缓存 
        resetCommonCaches(); 
        } 
    } }
```

