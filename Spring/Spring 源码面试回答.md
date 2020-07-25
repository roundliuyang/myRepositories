### Spring 源码面试回答

------

### **AOP原理**

无非是通过代理模式为目标对象生产代理对象，并将**横切逻辑**插入到目标方法执行的前后。

### **通知 - Advice**

通知 **Advice** 即我们定义的**横切逻辑**，比如我们可以定义一个用于监控方法性能的通知，也可以定义一个安全检查的通知等。

Spring 中定义了以下几种通知类型：

- 前置通知（Before advice）- 在目标方便调用前执行通知
- 后置通知（After advice）- 在目标方法完成后执行通知
- 返回通知（After returning advice）- 在目标方法执行成功后，调用通知
- 异常通知（After throwing advice）- 在目标方法抛出异常后，执行通知
- 环绕通知（Around advice）- 在目标方法调用前后均可执行自定义逻辑

### **织入 - Weaving**

现在我们有了**连接点**、**切点**、**通知**，以及切面等，但是还差织入。所谓织入就是在切点的引导下，将通知逻辑插入到方法调用上，使得我们的通知逻辑在方法调用时得以执行。

Spring 是通过何种方式将通知织入到**目标方法**上的

先来说说以何种方式进行织入，这个方式就是通过实现后置处理器 **BeanPostProcessor** 接口。该接口是 Spring 提供的一个**拓展接口**，通过实现该接口，用户可在 bean 初始化前后做一些自定义操作。那 Spring 是在何时进行织入操作的呢？答案是在 bean **初始化**完成后，即 bean 执行完初始化方法**（init-method）**。Spring通过切点对 bean 类中的方法进行匹配。若匹配成功，则会为该 bean 生成代理对象，并将代理对象返回给容器。容器向后置处理器输入 bean 对象，得到 bean 对象的代理，这样就完成了**织入**过程。

```java
public interface BeanPostProcessor {
    // bean 初始化前的回调方法
    Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;

    // bean 初始化后的回调方法    
    Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

}
```

**BeanPostProcessor** 是 Spring 框架的一个扩展点，通过实现 **BeanPostProcessor** 接口，我们就可插手 bean 实例化的过程。比如大家熟悉的 **AOP** 就是在 **bean 实例后**期间将切面逻辑织入 bean 实例中的，AOP 也正是通过 **BeanPostProcessor** 和 **IOC** 容器建立起了联系。

****



## **AOP大致流程**

AOP 的实现代码中，主要使用了 JDK 动态代理，在特定场景下（被代理对象没有 implements 的接口）也用到了 CGLIB 生成代理对象。通过 AOP 的源码设计可以看到，其先为目标对象建立了代理对象，这个代理对象的生成可以使用 JDK 动态代理或 CGLIB 完成。然后启动为代理对象配置的拦截器，对横切面（目标方法集合）进行相应的增强，将 AOP 的横切面设计和 Proxy 模式有机地结合起来，实现了在 AOP 中定义好的各种织入方式。

### 1.1 ProxyFactoryBean

这里我们主要以 ProxyFactoryBean 的实现为例，对 AOP 的实现原理进行分析。ProxyFactoryBean 主要持有目标对象 target 的代理对象 aopProxy，和 Advisor 通知器，而 Advisor 持有 Advice 和 Pointcut，这样就可以判断 aopProxy 中的方法 是否是某个指定的切面 Pointcut，然后根据其配置的织入方向（前置增强/后置增强），通过反射为其织入相应的增强行为 Advice。

### 1.2 JDK动态代理 生成 AopProxy代理对象

通过 JdkDynamicAopProxy 的源码可以非常清楚地看到，其使用了 JDK动态代理 的方式生成了 代理对象。JdkDynamicAopProxy 实现了 InvocationHandler 接口，并通过 java.lang.reflect.Proxy 的 newProxyInstance()静态方法 生成代理对象并返回。

### 1.3 Spring AOP 拦截器调用的实现

在 **Spring AOP** 通过 **JDK** 的 Proxy类 生成代理对象时，相关的**拦截器**已经配置到了代理对象持有的 InvocationHandler(即，**ProxyBeanFactory)** 的 invoke() 方法中，**拦截器**最后起作用，是通过**调用代理对象**的**目标方法**时，在**代理类中**触发了 **InvocationHandler** 的 **invoke() 回调**。

### 1.4JDK动态代理 生成 AopProxy代理对象

```Java
/**
 * 可以看到，其实现了 InvocationHandler 接口，所以肯定也定义了一个 使用 java.lang.reflect.Proxy
 * 动态生成代理对象的方法，并在实现的 invoke() 方法中为代理对象织入增强方法
 */
final class JdkDynamicAopProxy implements AopProxy, InvocationHandler, Serializable {

    /** AdvisedSupport 持有一个 List<Advisor>属性 */
    private final AdvisedSupport advised;

    public JdkDynamicAopProxy(AdvisedSupport config) throws AopConfigException {
        Assert.notNull(config, "AdvisedSupport must not be null");
        if (config.getAdvisors().length == 0 && config.getTargetSource() == AdvisedSupport.EMPTY_TARGET_SOURCE) {
            throw new AopConfigException("No advisors and no TargetSource specified");
        }
        // 这个 advised 是一个 AdvisedSupport 对象，可以通过它获取被代理对象 target
        // 这样，当 invoke()方法 被 代理对象aopProxy 调用时，就可以调用 target 的目标方法了
        this.advised = config;
    }
    
    public Object getProxy() {
        return getProxy(ClassUtils.getDefaultClassLoader());
    }
    
    public Object getProxy(ClassLoader classLoader) {
        if (logger.isDebugEnabled()) {
            logger.debug("Creating JDK dynamic proxy: target source is " + this.advised.getTargetSource());
        }
        
        // 获取代理类要实现的接口
        Class[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised);
        findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
        
        // 通过 java.lang.reflect.Proxy 生成代理对象并返回
        return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
    }
}
```

通过 **JdkDynamicAopProxy** 的源码可以非常清楚地看到，其使用了 JDK动态代理 的方式生成了 代理对象。**JdkDynamicAopProxy** 实现了 **InvocationHandler** 接口，并通过 **java.lang.reflect.Proxy** 的 newProxyInstance()静态方法 生成**代理对象**并返回。

### 1.5 AOP 拦截器链的调用

JdkDynamicAopProxy 和 CglibAopProxy 虽然使用了不同的代理对象，但对 AOP 拦截的处理却是相同的，都是通过 ReflectiveMethodInvocation 的 proceed() 方法实现的。

```java
public class ReflectiveMethodInvocation implements ProxyMethodInvocation, Cloneable {

    protected final Object proxy;
    
    protected final Object target;
    
    protected final Method method;
    
    protected Object[] arguments;
    
    private final Class targetClass;
    
    /** MethodInterceptor和InterceptorAndDynamicMethodMatcher的集合 */
    protected final List interceptorsAndDynamicMethodMatchers;
    
    private int currentInterceptorIndex = -1;
    
    protected ReflectiveMethodInvocation(Object proxy, Object target, Method method,
            Object[] arguments, Class targetClass,
            List<Object> interceptorsAndDynamicMethodMatchers) {
    
        this.proxy = proxy;
        this.target = target;
        this.targetClass = targetClass;
        this.method = BridgeMethodResolver.findBridgedMethod(method);
        this.arguments = arguments;
        this.interceptorsAndDynamicMethodMatchers = interceptorsAndDynamicMethodMatchers;
    }
    
    public Object proceed() throws Throwable {
        // 从拦截器链中按顺序依次调用拦截器，直到所有的拦截器调用完毕，开始调用目标方法，对目标方法的调用
        // 是在 invokeJoinpoint() 中通过 AopUtils 的 invokeJoinpointUsingReflection() 方法完成的
        if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
            // invokeJoinpoint() 直接通过 AopUtils 进行目标方法的调用
            return invokeJoinpoint();
        }
    
        // 这里沿着定义好的 interceptorsAndDynamicMethodMatchers拦截器链 进行处理，
        // 它是一个 List，也没有定义泛型，interceptorOrInterceptionAdvice 是其中的一个元素
        Object interceptorOrInterceptionAdvice =
                this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
        if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
            // 这里通过拦截器的 方法匹配器methodMatcher 进行方法匹配，
            // 如果 目标类 的 目标方法 和配置的 Pointcut 匹配，那么这个 增强行为advice 将会被执行，
            // Pointcut 定义了切面方法（要进行增强的方法），advice 定义了增强的行为
            InterceptorAndDynamicMethodMatcher dm = (InterceptorAndDynamicMethodMatcher) interceptorOrInterceptionAdvice;
            // 目标类的目标方法是否为 Pointcut 所定义的切面
            if (dm.methodMatcher.matches(this.method, this.targetClass, this.arguments)) {
                // 执行当前这个 拦截器interceptor 的 增强方法
                return dm.interceptor.invoke(this);
            }
            else {
                // 如果不匹配，那么 process()方法 会被递归调用，直到所有的拦截器都被运行过为止
                return proceed();
            }
        }
        else {
            // 如果 interceptorOrInterceptionAdvice 是一个 MethodInterceptor
            // 则直接调用其对应的方法
            return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
        }
    }
}
```

- **这里通过拦截器的 方法匹配器methodMatcher 进行方法匹配，**
- **如果 目标类 的 目标方法 和配置的 Pointcut 匹配，那么这个 增强行为advice 将会被执行，**
- **Pointcut 定义了切面方法（要进行增强的方法），advice 定义了增强的行为**

------------------------------------------------------------------------------------------------
**面试回答总结版**
**ProxyFactoryBean 主要持有目标对象 target 的代理对象 aopProxy，和 Advisor 通知器，而 Advisor 持有 Advice 和 Pointcut，这样就可以判断 aopProxy 中的方法 是否是某个指定的切面 Pointcut，然后根据其配置的织入方向（前置增强/后置增强），通过反射为其织入相应的增强行为 Advice。
通过动态代理（JDK或CgLIB）为 目标对象target 生成代理对象 之后，在调用 代理对象 的目标方法时，目标方法会进行 invoke()回调（JDK动态代理） 或 callbacks()回调（CGLIB），然后就可以在回调方法中对目标对象的目标方法进行拦截和增强处理了。这里通过拦截器的 方法匹配器methodMatcher 进行方法匹配，如果 目标类的目标方法 和配置的 Pointcut 匹配，那么这个 增强行为advice 将会被执行，Pointcut 定义了切面方法（要进行增强的方法），advice 定义了增强的行为**
- **JdkDynamicAopProxy 实现了 InvocationHandler 接口，并通过 java.lang.reflect.Proxy 的 newProxyInstance()静态方法 生成代理对象并返回。
- **JdkDynamicAopProxy 和 CglibAopProxy 虽然使用了不同的代理对象，但对 AOP 拦截的处理却是相同的，都是通过 ReflectiveMethodInvocation 的 proceed() 方法实现的。
- **在JdkDynamicAopProxy类的invoke方法中，如果没有配置拦截器，就直接通过反射调用目标对象 target 的 method对象，并获取返回值，如果有拦截器链，则需要先调用拦截器链中的拦截器，再调用目标的对应方法，这里通过构造 ReflectiveMethodInvocation 来实现
- **AOP 拦截器链的调用-- ReflectiveMethodInvocation  
**这里通过拦截器的 方法匹配器methodMatcher 进行方法匹配，
如果目标类的目标方法 和配置的 Pointcut 匹配，那么这个 增强行为advice 将会被执行，
Pointcut 定义了切面方法（要进行增强的方法），advice 定义了增强的行为
