## Spring Cloud 是如何实现热更新的

---

## TL;DR

* `RefreshBusEndpoint.refresh()` 发布  `RefreshRemoteApplicationEvent` 事件
* `RefreshListener.onApplicationEvent()` 监听到事件, 调用 `ContextRefresher.refresh()` 刷新配置
* `ContextRefresher.refresh()-> addConfigFilesToEnvironment()` 新建一个`SpringApplication`来加载新的配置, 并使用新配置覆盖旧配置
* `ContextRefresher.refresh() -> this.context.publishEvent` 刷新配置后, 发布了 `EnvironmentChangeEvent`事件通知哪些配置发生改变
* `ContextRefresher.refresh() -> this.scope.refreshAll()` 刷新 `@RefreshScope` 注释的类
* `ConfigurationPropertiesRebinder` 和 `LoggingRebinder` 监听了 `EnvironmentChangeEvent`事件

---

## 详细步骤代码解析

1. `RefreshBusEndpoint.refresh()` 发布  `RefreshRemoteApplicationEvent` 事件

```java
public void refresh(@RequestParam(value = "destination", required = false) String destination) {
    publish(new RefreshRemoteApplicationEvent(this, getInstanceId(), destination));
}
```

2. `RefreshListener.onApplicationEvent()` 监听到事件, 调用 `ContextRefresher.refresh()` 刷新配置

```java
private ContextRefresher contextRefresher;

public void onApplicationEvent(RefreshRemoteApplicationEvent event) {
    contextRefresher.refresh();
}
```

3. `refresh()`方法执行刷新操作

```java
public synchronized Set<String> refresh() {
    // extract 方法提取出所有的非标准属性源的属性
    Map<String, Object> before = extract(this.context.getEnvironment().getPropertySources());
    // 核心方法, 相当于重新启动一次, 加载最新的配置源. 然后保留不可更改的部分, 使用新的配置替换就的配置
    addConfigFilesToEnvironment();
    // 哪些属性发生了改变, this.context.getEnvironment().getPropertySources() 的属性已经
    // 被 #addConfigFilesToEnvironment 更新过, 是最新的属性.
    Set<String> keys = changes(before, extract(this.context.getEnvironment().getPropertySources())).keySet();
    // 发布事件, 通知哪些属性是被更改过
    this.context.publishEvent(new EnvironmentChangeEvent(context, keys));
    // 刷新
    this.scope.refreshAll();
    return keys;
}
```

4. `extract()`方法提取出所有的非标准属性源的属性,这些属性是不需要更改的属性

* `StandardEnvironment.SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME = "systemProperties" ` 
* `StandardEnvironment.SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME = "systemEnvironment"`
* `StandardServletEnvironment.JNDI_PROPERTY_SOURCE_NAME = "jndiProperties"` 
* `StandardServletEnvironment.SERVLET_CONFIG_PROPERTY_SOURCE_NAME = "servletConfigInitParams"`
* `StandardServletEnvironment.SERVLET_CONTEXT_PROPERTY_SOURCE_NAME = "servletContextInitParams"`

5. `addConfigFilesToEnvironment` 是核心逻辑, 新建一个`SpringApplication`来加载新的配置, 并使用新配置覆盖旧配置

```java
ConfigurableApplicationContext addConfigFilesToEnvironment() {
    ConfigurableApplicationContext capture = null;
    // 1. 实例化了一个StandardEnvironment, 用于获取 systemProperties 和 systemEnvironment
    // 2. 从旧的配置中获取 DEFAULT_PROPERTY_SOURCES, 并替换或新增到新的环境中
    // 3. 设置 REFRESH_ARGS_PROPERTY_SOURCE
    StandardEnvironment environment = copyEnvironment(this.context.getEnvironment());

    // 启动一个新的Application, 用于加载其余配置
    SpringApplicationBuilder builder = new SpringApplicationBuilder(Empty.class)
        .bannerMode(Mode.OFF).web(false).environment(environment);
    builder.application()
        .setListeners(Arrays.asList(new BootstrapApplicationListener(),
                                    new ConfigFileApplicationListener()));
    capture = builder.run(); 

    // 移除 copyEnvironment 方法中添加的 REFRESH_ARGS_PROPERTY_SOURCE 配置
    // @TODO: 为啥要先添加在移除?
    if (environment.getPropertySources().contains(REFRESH_ARGS_PROPERTY_SOURCE)) {
        environment.getPropertySources().remove(REFRESH_ARGS_PROPERTY_SOURCE);
    }
    // 获取到旧的的属性源
    MutablePropertySources target = this.context.getEnvironment().getPropertySources();
    String targetName = null;
    // 比较新旧两个配置源, 更新非标准数据源, 忽略 standardSources
    // standardSources 包括 StandardEnvironment 和 StandardServletEnvironment
    // StandardEnvironment: systemProperties, systemEnvironment
    // StandardServletEnvironment: servletContextInitParams, servletConfigInitParams,
    // jndiProperties 这些配置源都是不可改变的
    for (PropertySource<?> source : environment.getPropertySources()) {
        String name = source.getName();
        // 旧配置源中存在新配置中的配置源
        if (target.contains(name)) {
            targetName = name;
        }
        // 处理非标准属性源 StandardEnvironment 和 StandardServletEnvironment
        if (!this.standardSources.contains(name)) {
            if (target.contains(name)) {
                // 旧配置中已经存在, 则使用新的配置替换
                target.replace(name, source);
            }
            else {
                // @TODO: 处理旧配置中不存在的配置源,有点奇怪
                // 需要研究 addAfter,addFirst 等的具体实现
                if (targetName != null) {
                    target.addAfter(targetName, source);
                }
                else {
                    // targetName was null so we are at the start of the list
                    target.addFirst(source);
                    targetName = name;
                }
            }
        }
    }
    return capture;
}
```

```java
private StandardEnvironment copyEnvironment(ConfigurableEnvironment input) {
    // 获取系统变量(System Properties)和环境变量(System Environment)
    StandardEnvironment environment = new StandardEnvironment();
    // 只有systemProperties 和 systemEnvironment
    MutablePropertySources capturedPropertySources = environment.getPropertySources();
    // DEFAULT_PROPERTY_SOURCES: defaultProperties,commandLineArgs 这两个不需要改变
    for (String name : DEFAULT_PROPERTY_SOURCES) {
        // 判断旧配置是否包含 DEFAULT_PROPERTY_SOURCES
        if (input.getPropertySources().contains(name)) {	
            // 包含则进行替换,不包含则新增,所以 DEFAULT_PROPERTY_SOURCES 是无法进行更新的
            if (capturedPropertySources.contains(name)) {
                capturedPropertySources.replace(name,input.getPropertySources().get(name));
            }
            else {
                capturedPropertySources.addLast(input.getPropertySources().get(name));
            }
        }
    }
    environment.setActiveProfiles(input.getActiveProfiles());
    environment.setDefaultProfiles(input.getDefaultProfiles());
    // 设置 REFRESH_ARGS_PROPERTY_SOURCE
    Map<String, Object> map = new HashMap<String, Object>();
    map.put("spring.jmx.enabled", false);
    map.put("spring.main.sources", "");
    capturedPropertySources.addFirst(new MapPropertySource(REFRESH_ARGS_PROPERTY_SOURCE, map));
    return environment;
}
```

```java
private Map<String, Object> changes(Map<String, Object> before, Map<String, Object> after) {
    Map<String, Object> result = new HashMap<String, Object>();
    for (String key : before.keySet()) {
        if (!after.containsKey(key)) {
            // 旧配置中有, 新配置中没有, 把值设为 null
            result.put(key, null);
        }
        else if (!equal(before.get(key), after.get(key))) {
            // 新旧配置中都有,但是值不一样
            result.put(key, after.get(key));
        }
    }
    for (String key : after.keySet()) {
        if (!before.containsKey(key)) {
            // 新配置中有, 旧配置中没有
            result.put(key, after.get(key));
        }
    }
    return result;
}
```



The application listens for an `EnvironmentChangeEvent` and reacts to the change in a couple of standard ways (additional `ApplicationListeners` can be added as `@Beans` by the user in the normal way). When an `EnvironmentChangeEvent` is observed, it has a list of key values that have changed, and the application uses those to:

- Re-bind any `@ConfigurationProperties` beans in the context
- Set the logger levels for any properties in `logging.level.*`

`ConfigurationPropertiesRebinder` 由 `ConfigurationPropertiesRebinderAutoConfiguration`注册生成bean

`ConfigurationPropertiesBeans` 需要 `ConfigurationBeanFactoryMetaData` 这个类逻辑很简单，它是一个 `BeanFactoryPostProcessor` 的实现，将所有的 Bean 都存在了内部的一个 Map 中 

`ConfigurationPropertiesBeans` 实现了 `BeanPostProcessor` 接口, 会调用`postProcessBeforeInitialization()`来获取全部带有 `@ConfigurationProperties` 注解的类

```java
@Bean
@ConditionalOnMissingBean(search = SearchStrategy.CURRENT)
public ConfigurationPropertiesBeans configurationPropertiesBeans() {
    // Since this is a BeanPostProcessor we have to be super careful not to
    // cause a cascade of bean instantiation. Knowing the *name* of the beans we
    // need is super optimal, but a little brittle (unfortunately we have no
    // choice).
    ConfigurationBeanFactoryMetaData metaData = this.context.getBean(
        ConfigurationPropertiesBindingPostProcessorRegistrar.BINDER_BEAN_NAME + ".store",
        ConfigurationBeanFactoryMetaData.class);
    ConfigurationPropertiesBeans beans = new ConfigurationPropertiesBeans();
    beans.setBeanMetaDataStore(metaData);
    return beans;
}

@Bean
@ConditionalOnMissingBean(search = SearchStrategy.CURRENT)
public ConfigurationPropertiesRebinder configurationPropertiesRebinder(ConfigurationPropertiesBeans beans) {
    return new ConfigurationPropertiesRebinder(beans);
}
```

```java
@Override
public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
    if (isRefreshScoped(beanName))  return bean;
    ConfigurationProperties annotation = AnnotationUtils.findAnnotation(
        bean.getClass(), ConfigurationProperties.class);
    if (annotation != null) {
        this.beans.put(beanName, bean);
    }
    else if (this.metaData != null) {
        annotation = this.metaData.findFactoryAnnotation(beanName, ConfigurationProperties.class);
        if (annotation != null)  this.beans.put(beanName, bean);
    }
    return bean;
}
```

`ConfigurationPropertiesRebinder`是一个事件监听器,会监听 `EnvironmentChangeEvent`事件

```java
@Override
public void onApplicationEvent(EnvironmentChangeEvent event) {
    if (this.applicationContext.equals(event.getSource())
        // Backwards compatible
        || event.getKeys().equals(event.getSource())) {
        rebind(); 
    }
}
```

`rebind()`方法会重新绑定属性

```java

// 自动注入,包含全部有@ConfigurationProperties注释的类
private ConfigurationPropertiesBeans beans;

public void rebind() {
    this.errors.clear();
    for (String name : this.beans.getBeanNames()) {
        rebind(name);
    }
}

public boolean rebind(String name) {
    if (!this.beans.getBeanNames().contains(name)) {
        return false;
    }
    if (this.applicationContext != null) {
        Object bean = this.applicationContext.getBean(name);
        if (AopUtils.isAopProxy(bean)) {
            bean = getTargetObject(bean);
        }
        this.applicationContext.getAutowireCapableBeanFactory().destroyBean(bean);
        this.applicationContext.getAutowireCapableBeanFactory().initializeBean(bean, name);
        return true;
    }
    return false;
}
```



```java
public boolean rebind(String name) {
    if (!this.beans.getBeanNames().contains(name)) {
        return false;
    }
    if (this.applicationContext != null) {
        Object bean = this.applicationContext.getBean(name);
        if (AopUtils.isAopProxy(bean)) {
            bean = getTargetObject(bean);
        }
        this.applicationContext.getAutowireCapableBeanFactory().destroyBean(bean);
        this.applicationContext.getAutowireCapableBeanFactory().initializeBean(bean, name);
        return true;
    }
    return false;
}
```

其中主要做了三件事：

1. `applyBeanPostProcessorsBeforeInitialization`：调用所有 `BeanPostProcessor` 的 `postProcessBeforeInitialization` 方法。
2. `invokeInitMethods`：如果 Bean 继承了 `InitializingBean`，执行 `afterPropertiesSet` 方法，或是如果 Bean 指定了 `init-method` 属性，如果有则调用对应方法
3. `applyBeanPostProcessorsAfterInitialization`：调用所有 `BeanPostProcessor` 的 `postProcessAfterInitialization` 方法。

之后 `ConfigurationPropertiesRebinder` 就完成整个重新绑定流程了

```java
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
    if (System.getSecurityManager() != null) {
        AccessController.doPrivileged(() -> {
                invokeAwareMethods(beanName, bean);
                return null;
            }, getAccessControlContext());
    }
    else {
        invokeAwareMethods(beanName, bean);
    }

    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }
    invokeInitMethods(beanName, wrappedBean, mbd);
    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }
    return wrappedBean;
}
```



