## Spring注解

### 使用注解来构造IoC容器

用注解来向Spring容器注册Bean。需要在**applicationContext.xml**中注册**<context:component-scan base-package=”pagkage1[,pagkage2,…,pagkageN]”/>**。

如：在**base-package**指明一个包

```java
 <context:component-scan base-package="cn.gacl.java"/>
```

表明**cn.gacl.java**包及其子包中，如果某个类的头上带有特定的注解

【@Component/@Repository/@Service/@Controller】，就会将这个对象作为Bean注册进Spring容器。也可以在**<context:component-scan base-package=” ”/>**中指定多个包，如：

```java
<context:component-scan base-package="cn.gacl.dao.impl,cn.gacl.service.impl,cn.gacl.action"/>
```

多个包逗号隔开。



### @Component

**@Component**是所有受Spring 管理组件的通用形式，@Component注解可以放在类的头上，@Component不推荐使用。



### @Controller

**@Controller**对应表现层的Bean，也就是Action，例如：

```java
1 @Controller
2 @Scope("prototype")
3 public class UserAction extends BaseAction<User>{
4 ……
5 }
```

使用**@Controller**注解标识**UserAction**之后，就表示要把UserAction交给Spring容器管理，在Spring容器中会存在一个名字为"userAction"的action，这个名字是根据UserAction类名来取的。注意：如果@Controller不指定其value【@Controller】，则**默认的bean名字为这个类的类名首字母小写**，如果指定value【@Controller(value="UserAction")】或者【@Controller("UserAction")】，**则使用value作为bean的名字。**

这里的**UserAction**还使用了@Scope注解，**@Scope("prototype")**表示将Action的范围声明为原型，可以利用容器的scope="prototype"来保证每一个请求有一个单独的Action来处理，避免struts中Action的线程安全问题。**spring 默认scope 是单例模式(scope="singleton")**，这样只会创建一个Action对象，每次访问都是同一Action对象，数据不安全，struts2 是要求每次次访问都对应不同的Action，scope="prototype" 可以保证当有请求的时候都创建一个Action对象

@Controller用于标记在一个类上，使用它标记的类就是一个SpringMvc Controller对象，分发处理器会扫描使用该注解的类的方法，并检测该方法是否使用了@RequestMapping注解。
@Controller只是定义了一个控制器类，而使用@RequestMapping注解的方法才是处理请求的处理器。
@Controller标记在一个类上还不能真正意义上说它就是SpringMvc的控制器，应为这个时候Spring还不认识它，这个时候需要把这个控制器交给Spring来管理。有两种方式可以管理：

```java
<!--方式一-->
<bean class="com.HelloWorld"/>

<!--方式二-->
<!--路径写到controller的上一层-->
<context:component-scan base-package="com"/>

同样
可以使用注解@ComponentScans
```



### @ Service

### @Resource

@Service对应的是业务层Bean，例如：

```java
1 @Service("userService")
2 public class UserServiceImpl implements UserService {
3 ………
4 }
```

**@Service("userService")**注解是告诉Spring，当Spring要创建UserServiceImpl的的实例时，bean的名字必须叫做"userService"，这样当Action需要使用UserServiceImpl的的实例时,就可以由Spring创建好的"userService"，然后注入给Action：在Action只需要声明一个名字叫“userService”的变量来接收由Spring注入的"userService"即可，具体代码如下：

```java
1 // 注入userService
2 @Resource(name = "userService")
3 private UserService userService;
```

注意：在Action声明的“userService”变量的类型必须是“UserServiceImpl”或者是其父类“UserService”，否则由于类型不一致而无法注入，由于Action中的声明的“userService”变量使用了**@Resource**注解去标注，并且指明了其name = "userService"，这就等于告诉Spring，说我Action要实例化一个“userService”，你Spring快点帮我实例化好，然后给我，当Spring看到userService变量上的@Resource的注解时，根据其指明的**name属性**可以知道，Action中需要用到一个UserServiceImpl的实例，此时Spring就会把自己创建好的名字叫做"userService"的UserServiceImpl的实例注入给Action中的“userService”变量，帮助Action完成userService的实例化，这样在Action中就不用通过“UserService userService = new UserServiceImpl();”这种最原始的方式去实例化userService了。

如果没有Spring，那么当Action需要使用UserServiceImpl时，必须通过“UserService userService = new UserServiceImpl();”主动去创建实例对象，但使用了Spring之后，Action要使用UserServiceImpl时，就不用主动去创建UserServiceImpl的实例了，创建UserServiceImpl实例已经交给Spring来做了，Spring把创建好的UserServiceImpl实例给Action，Action拿到就可以直接用了。Action由原来的主动创建UserServiceImpl实例后就可以马上使用，变成了被动等待由Spring创建好UserServiceImpl实例之后再注入给Action，Action才能够使用。

这说明**Action对“UserServiceImpl”类的“控制权”已经被“反转”了**，原来主动权在自己手上，自己要使用“UserServiceImpl”类的实例，自己主动去new一个出来马上就可以使用了，但现在自己不能主动去new“UserServiceImpl”类的实例，new“UserServiceImpl”类的实例的权力已经被Spring拿走了，只有Spring才能够new“UserServiceImpl”类的实例，而Action只能等Spring创建好“UserServiceImpl”类的实例后，再“恳求”Spring把创建好的“UserServiceImpl”类的实例给他，这样他才能够使用“UserServiceImpl”，这就是Spring核心思想“**控制反转**”，也叫“**依赖注入**”，“依赖注入”也很好理解，Action需要使用UserServiceImpl干活，那么就是对UserServiceImpl产生了依赖，Spring把Acion需要依赖的UserServiceImpl注入(也就是“给”)给Action，这就是所谓的“依赖注入”。对Action而言，**Action依赖什么东西，就请求Spring注入给他**，对Spring而言，Action需要什么，Spring就主动注入给他。



### @ Repository

@Repository对应数据访问层Bean ，例如：

```java
1 @Repository(value="userDao")
2 public class UserDaoImpl extends BaseDaoImpl<User> {
3 ………
4 }
```

**@Repository(value="userDao")**注解是告诉Spring，让Spring创建一个名字叫“userDao”的UserDaoImpl实例。

当Service需要使用Spring创建的名字叫“userDao”的UserDaoImpl实例时，就可以使用@Resource(name = "userDao")注解告诉Spring，Spring把创建好的userDao注入给Service即可。

```java
1 // 注入userDao，从数据库中根据用户Id取出指定用户时需要用到
2 @Resource(name = "userDao")
3 private BaseDao<User> userDao;
```



### @Configuration

@Configuration  告诉Spring这是一个配置类，配置类==配置文件。



### @ComponentScans

```java
//注解
@ComponentScans(
      value = {
            @ComponentScan(value="com.atguigu",includeFilters = {
/*                @Filter(type=FilterType.ANNOTATION,classes={Controller.class}),
                  @Filter(type=FilterType.ASSIGNABLE_TYPE,classes={BookService.class}),*/
                  @Filter(type=FilterType.CUSTOM,classes={MyTypeFilter.class})
            },useDefaultFilters = false)   
      }
)
//component-scan默认扫描的注解类型是@Component,不过，在@Component的语义基础之上细化为@Reposity，@Service,@Controller.

//@ComponentScan  value:指定要扫描的包
//excludeFilters = Filter[] ：指定扫描的时候按照什么规则排除那些组件
//includeFilters = Filter[] ：指定扫描的时候只需要包含哪些组件
//FilterType.ANNOTATION：按照注解
//FilterType.ASSIGNABLE_TYPE：按照给定的类型；
//FilterType.ASPECTJ：使用ASPECTJ表达式
//FilterType.REGEX：使用正则指定
//FilterType.CUSTOM：使用自定义规则

//xml
<context:component-scan base-package="com" user-default-filters="false">
    <context:include-filter type="regex" expression="com.tan.*"/>
</context:component-scan>

```



### @Bean 

```java
//给容器中注册一个Bean;类型为返回值的类型，id默认是用方法名作为id
@Bean("person")
public Person person01(){
   return new Person("lisi", 20);
}
```

Spring的**@Bean**注解用于告诉方法，产生一个Bean对象，然后这个Bean对象交给Spring管理。产生这个Bean对象的方法Spring只会调用一次，随后这个Spring将会将这个Bean对象放在自己的IOC容器中。

**SpringIOC** 容器管理一个或者多个bean，这些bean都需要在@Configuration注解下进行创建，在一个方法上使用@Bean注解就表明这个方法需要交给Spring进行管理。



### @Scope

```java
@Scope:调整作用域
prototype：多实例的：ioc容器启动并不会去调用方法创建对象放在容器中。
           每次获取的时候才会调用方法创建对象；
singleton：单实例的（默认值）：ioc容器启动会调用方法创建对象放到ioc容器中。
           以后每次获取就是直接从容器（map.get()）中拿，
request：同一次请求创建一个实例
session：同一个session创建一个实例
```



### @Lazy

懒加载：
	 单实例bean：默认在容器启动的时候创建对象；
	 懒加载：容器启动不创建对象。第一次使用(获取)Bean创建对象，并初始化



### @Conditional

**@Conditional**({Condition}) ： 按照一定的条件进行判断，满足条件给容器中注册bean

```java
//类中组件统一设置。满足当前条件，这个类中配置的所有bean注册才能生效；
@Conditional({WindowsCondition.class})
public class MainConfig2 {
    
}
```



### @Import

```
//@Import导入组件，id默认是组件的全类名
给容器中注册组件；
1）、包扫描+组件标注注解（@Controller/@Service/@Repository/@Component）[自己写的类]
2）、@Bean[导入的第三方包里面的组件]
3）、@Import[快速给容器中导入一个组件]
    1）、@Import(要导入到容器中的组件)；容器中就会自动注册这个组件，id默认是全类名
    2）、ImportSelector:返回需要导入的组件的全类名数组；
    3）、ImportBeanDefinitionRegistrar:手动注册bean到容器中
4）、使用Spring提供的 FactoryBean（工厂Bean）;
    1）、默认获取到的是工厂bean调用getObject创建的对象
    2）、要获取工厂Bean本身，我们需要给id前面加一个& 
    &colorFactoryBean
     
```



### 源码待看

```java
/**
 * bean的生命周期：
 *        bean创建---初始化----销毁的过程
 * 容器管理bean的生命周期；
 * 我们可以自定义初始化和销毁方法；容器在bean进行到当前生命周期的时候来调用我们自定义的初始化和销毁方法
 * 
 * 构造（对象创建）
 *        单实例：在容器启动的时候创建对象
 *        多实例：在每次获取的时候创建对象\
 * 
 * BeanPostProcessor.postProcessBeforeInitialization
 * 初始化：
 *        对象创建完成，并赋值好，调用初始化方法。。。
 * BeanPostProcessor.postProcessAfterInitialization
 * 销毁：
 *        单实例：容器关闭的时候
 *        多实例：容器不会管理这个bean；容器不会调用销毁方法；
 * 
 * 
 * 遍历得到容器中所有的BeanPostProcessor；挨个执行beforeInitialization，
 * 一但返回null，跳出for循环，不会执行后面的BeanPostProcessor.postProcessorsBeforeInitialization
 * 
 * BeanPostProcessor原理
 * populateBean(beanName, mbd, instanceWrapper);给bean进行属性赋值
 * initializeBean
 * {
 * applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
 * invokeInitMethods(beanName, wrappedBean, mbd);执行自定义初始化
 * applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
 *}
 * 
 * 
 * 
 * 1）、指定初始化和销毁方法；
 *        通过@Bean指定init-method和destroy-method；
 * 2）、通过让Bean实现InitializingBean（定义初始化逻辑），
 *              DisposableBean（定义销毁逻辑）;
 * 3）、可以使用JSR250；
 *        @PostConstruct：在bean创建完成并且属性赋值完成；来执行初始化方法
 *        @PreDestroy：在容器销毁bean之前通知我们进行清理工作
 * 4）、BeanPostProcessor【interface】：bean的后置处理器；
 *        在bean初始化前后进行一些处理工作；
 *        postProcessBeforeInitialization:在初始化之前工作
 *        postProcessAfterInitialization:在初始化之后工作
 * 
 * Spring底层对 BeanPostProcessor 的使用；
 *        bean赋值，注入其他组件，@Autowired，生命周期注解功能，@Async,xxx BeanPostProcessor;
 * 
 * @author lfy
 *
 */
```



### @Value

```java
public class Person {
   
   //使用@Value赋值；
   //1、基本数值
   //2、可以写SpEL； #{}
   //3、可以写${}；取出配置文件【properties】中的值（在运行环境变量里面的值）
   
   @Value("张三")
   private String name;
   @Value("#{20-2}")
   private Integer age;
   
   @Value("${person.nickName}")
   private String nickName;
    
}
```



### @PropertySource

```java
//使用@PropertySource读取外部配置文件中的k/v保存到运行的环境变量中;加载完外部的配置文件以后使用${}取出配置文件的值
@PropertySource(value={"classpath:/person.properties"})
@Configuration
public class MainConfigOfPropertyValues {
   
   @Bean
   public Person person(){
      return new Person();
   }

}
```



### @Autowired

### @Resource

### @Inject

```java
/**
 * 自动装配;
 *        Spring利用依赖注入（DI），完成对IOC容器中中各个组件的依赖关系赋值；
 * 
 * 1）、@Autowired：自动注入：
 *        1）、默认优先按照类型去容器中找对应的组件:applicationContext.getBean(BookDao.class);找到就赋值
 *        2）、如果找到多个相同类型的组件，再将属性的名称作为组件的id去容器中查找
 *                       applicationContext.getBean("bookDao")
 *        3）、@Qualifier("bookDao")：使用@Qualifier指定需要装配的组件的id，而不是使用属性名
 *        4）、自动装配默认一定要将属性赋值好，没有就会报错；
 *           可以使用@Autowired(required=false);
 *        5）、@Primary：让Spring进行自动装配的时候，默认使用首选的bean；
 *              也可以继续使用@Qualifier指定需要装配的bean的名字
 *        BookService{
 *           @Autowired
 *           BookDao  bookDao;
 *        }
 * 
 * 2）、Spring还支持使用@Resource(JSR250)和@Inject(JSR330)[java规范的注解]
 *        @Resource:
 *           可以和@Autowired一样实现自动装配功能；默认是按照组件名称进行装配的；
 *           没有能支持@Primary功能没有支持@Autowired（reqiured=false）;
 *        @Inject:
 *           需要导入javax.inject的包，和Autowired的功能一样。没有required=false的功能；
 *  @Autowired:Spring定义的； @Resource、@Inject都是java规范
 *     
 * AutowiredAnnotationBeanPostProcessor:解析完成自动装配功能；       
 * 
 * 3）、 @Autowired:构造器，参数，方法，属性；都是从容器中获取参数组件的值
 *        1）、[标注在方法位置]：@Bean+方法参数；参数从容器中获取;默认不写@Autowired效果是一样的；都能自动装配
 *        2）、[标在构造器上]：如果组件只有一个有参构造器，这个有参构造器的@Autowired可以省略，参数位置的组件还是可以自动从容器中获取
 *        3）、放在参数位置：
 * 
 * 4）、自定义组件想要使用Spring容器底层的一些组件（ApplicationContext，BeanFactory，xxx）；
 *        自定义组件实现xxxAware；在创建对象的时候，会调用接口规定的方法注入相关组件；Aware；
 *        把Spring底层一些组件注入到自定义的Bean中；
 *        xxxAware：功能使用xxxProcessor；
 *           ApplicationContextAware==》ApplicationContextAwareProcessor；
 *     
 *        
 * @author lfy
 *
 */
```





```java
/**
 * Profile：
 *        Spring为我们提供的可以根据当前环境，动态的激活和切换一系列组件的功能；
 * 
 * 开发环境、测试环境、生产环境；
 * 数据源：(/A)(/B)(/C)；
 * 
 * 
 * @Profile：指定组件在哪个环境的情况下才能被注册到容器中，不指定，任何环境下都能注册这个组件
 * 
 * 1）、加了环境标识的bean，只有这个环境被激活的时候才能注册到容器中。默认是default环境
 * 2）、写在配置类上，只有是指定的环境的时候，整个配置类里面的所有配置才能开始生效
 * 3）、没有标注环境标识的bean在，任何环境下都是加载的；
 */

@PropertySource("classpath:/dbconfig.properties")
@Configuration
public class MainConfigOfProfile implements EmbeddedValueResolverAware{
   
   @Value("${db.user}")
   private String user;
   
   private StringValueResolver valueResolver;
   
   private String  driverClass;
   
   
   @Bean
   public Yellow yellow(){
      return new Yellow();
   }
   
   @Profile("test")
   @Bean("testDataSource")
   public DataSource dataSourceTest(@Value("${db.password}")String pwd) throws Exception{
      ComboPooledDataSource dataSource = new ComboPooledDataSource();
      dataSource.setUser(user);
      dataSource.setPassword(pwd);
      dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
      dataSource.setDriverClass(driverClass);
      return dataSource;
   }
   
   
   @Profile("dev")
   @Bean("devDataSource")
   public DataSource dataSourceDev(@Value("${db.password}")String pwd) throws Exception{
      ComboPooledDataSource dataSource = new ComboPooledDataSource();
      dataSource.setUser(user);
      dataSource.setPassword(pwd);
      dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/ssm_crud");
      dataSource.setDriverClass(driverClass);
      return dataSource;
   }
   
   @Profile("prod")
   @Bean("prodDataSource")
   public DataSource dataSourceProd(@Value("${db.password}")String pwd) throws Exception{
      ComboPooledDataSource dataSource = new ComboPooledDataSource();
      dataSource.setUser(user);
      dataSource.setPassword(pwd);
      dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/scw_0515");
      
      dataSource.setDriverClass(driverClass);
      return dataSource;
   }

   @Override
   public void setEmbeddedValueResolver(StringValueResolver resolver) {
      // TODO Auto-generated method stub
      this.valueResolver = resolver;
      driverClass = valueResolver.resolveStringValue("${db.driverClass}");
   }

}


//测试
public class IOCTest_Profile {
	
	//1、使用命令行动态参数: 在虚拟机参数位置加载 -Dspring.profiles.active=test
	//2、代码的方式激活某种环境；
	@Test
	public void test01(){
		AnnotationConfigApplicationContext applicationContext = 
				new AnnotationConfigApplicationContext();
		//1、创建一个applicationContext
		//2、设置需要激活的环境
		applicationContext.getEnvironment().setActiveProfiles("dev");
		//3、注册主配置类
		applicationContext.register(MainConfigOfProfile.class);
		//4、启动刷新容器
		applicationContext.refresh();
		
		
		String[] namesForType = applicationContext.getBeanNamesForType(DataSource.class);
		for (String string : namesForType) {
			System.out.println(string);
		}
		
		Yellow bean = applicationContext.getBean(Yellow.class);
		System.out.println(bean);
		applicationContext.close();
	}

}

```

