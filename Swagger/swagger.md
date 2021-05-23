



收起目录-

- [项目结构](https://www.kuangstudy.com/bbs/1375417006890790914#header1)
- [Swagger简介](https://www.kuangstudy.com/bbs/1375417006890790914#header2)
- swagger集成
  - [导入依赖](https://www.kuangstudy.com/bbs/1375417006890790914#header3)
  - [写一个简单的controller，测试环境](https://www.kuangstudy.com/bbs/1375417006890790914#header4)
  - [测试](https://www.kuangstudy.com/bbs/1375417006890790914#header5)
- Swagger基本配置
  - [写一个Swagger的配置类](https://www.kuangstudy.com/bbs/1375417006890790914#header7)
  - [测试](https://www.kuangstudy.com/bbs/1375417006890790914#header8)
  - [swagger分组](https://www.kuangstudy.com/bbs/1375417006890790914#header9)
  - [为Model添加swagger](https://www.kuangstudy.com/bbs/1375417006890790914#header10)
  - [切换生产环境](https://www.kuangstudy.com/bbs/1375417006890790914#header11)
  - [在线测试](https://www.kuangstudy.com/bbs/1375417006890790914#header12)
- Swagger的常用注解
  - [示例：](https://www.kuangstudy.com/bbs/1375417006890790914#header14)

> 本文介绍Swagger的相关知识

## 项目结构

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/26/kuangstudyaa30e0b1-c736-4082-b8db-036b4287472b.png)

## Swagger简介

> 官网：<https://swagger.io/>
> swagger是一个文档在线自动生成器，它可以在线测试ApI,支持多种语言。之所以会会产生这个框架，是因为在实际的项目开发过程中，前后端是分离的，他们之间通过API进行交互，但是由于两者之间无法做到实时的同步，可能会导致问题的发生。而swagger解决了上述问题，可以使得API实时在线更新，还可以在线测试。

## swagger集成

### 导入依赖

**swagger3.0只需要导入这一个坐标**

```xml
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-boot-starter</artifactId>
            <version>3.0.0</version>
        </dependency>
```

**如果是swagger2.x,需要导入下面的坐标**

```xml
<dependency>
   <groupId>io.springfox</groupId>
   <artifactId>springfox-swagger2</artifactId>
   <version>2.9.2</version>
</dependency>
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
<dependency>
   <groupId>io.springfox</groupId>
   <artifactId>springfox-swagger-ui</artifactId>
   <version>2.9.2</version>
</dependency>
```

### 写一个简单的controller，测试环境

```java
@RestController
public class SwaggerController {
    @PostMapping("/hello")
    public String hello() {
        return "hello";
    }
```

### 测试

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/26/kuangstudya0258ccd-d04e-4aaf-bba2-6ae0640ebba1.png)

## Swagger基本配置

### 写一个Swagger的配置类

```java
@Configuration //声明该类是一个配置类
@EnableSwagger2 //开启swagger
public class SwaggerConfig {
    /**
     * 创建Docket的bean实例对象
     * @return
     */
    @Bean
    public Docket getDocket(){
        return new Docket
                (new DocumentationType("swagger", "2.0")).groupName("hello")
                .apiInfo(getApiInfo())
                //RequestHandlerSelectors.basePackage指定要扫描的包
                //RequestHandlerSelectors.any()扫描所有包
                //RequestHandlerSelectors.none()任何包都不扫描
                //RequestHandlerSelectors.withMethodAnnotation()扫描方法上的指定注解
                //RequestHandlerSelectors.withMethodAnnotation()扫描类上的指定注解
                .select().apis(RequestHandlerSelectors.basePackage("com.swagger.controller"))
                //配置要拦截的路径，any:拦截所有，ant:拦截指定路径，none,都不拦截
                //.paths(PathSelectors.any())
                .build();
    }
    public ApiInfo getApiInfo(){
        //作者信息
        Contact DEFAULT_CONTACT = new Contact("hello", "https://www.baidu.com/", "213233945@qq.com");
        return  new ApiInfo(
                "项目文档", //标题
                "项目文档接口", //描述
                "1.0", //版本
                "https://www.baidu.com/", //项目地址
                DEFAULT_CONTACT, //作者信息
                "Apache 2.0", //许可
                "http://www.apache.org/licenses/LICENSE-2.0", //许可证网址
                new ArrayList<>());
    }
}
```

### 测试

在网址栏中输入(本文使用的是swagger3)：<http://localhost:8080/swagger-ui/index.html> ,如果是swagger2，则网址输入：localhost:8080/swagger-ui.html
![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/26/kuangstudyff176f2e-2465-4bbb-b004-d569a98eed3d.png)

### swagger分组

只需要在上述controller中加入对应的Docket实例的，并将其注入到spring中，在方法中加入groupName即可

```java
 @Bean
    public Docket getDocket1(){
        return new Docket(DocumentationType.SWAGGER_2).groupName("a");
    }
    @Bean
    public Docket getDocket2(){
        return new Docket(DocumentationType.SWAGGER_2).groupName("b");
    }
    @Bean
    public Docket getDocket3(){
        return new Docket(DocumentationType.SWAGGER_2).groupName("c");
    }
```

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/26/kuangstudycf3294b7-d630-4681-8e84-26818f3b054c.png)

### 为Model添加swagger

#### 新建User类

该类的注解会在下面详解，如果我们想要为某个pojo类添加swagger信息，只需要书写对应的类，然后在controller接口中返回该对象就可以。
**注意：User类中一定要有属性的setter，getter方法，否则，在后面的ui界面中不会显示User的属性，而且后面测试带参数的方法时，可能不能接收我们输入的值**

```java
@ApiModel("User实体类")
public class User {
    @ApiModelProperty("用户名")
    private String username;
    @ApiModelProperty("密码")
    private String password;
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
    @Override
    public String toString() {
        return "User{" +
                "username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```

#### 在controller中加入下面的代码

```java
  @ApiOperation("获取用户的controller")
    @GetMapping("/user")
    public User User() {
        return new User();
    }
```

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/26/kuangstudy586f7cf3-73fa-43ce-81f3-43b9a4454aa3.png)

### 切换生产环境

#### 新建application-dev.properties,application-pro.properties

在dev中指定端口号为：server.port=8081，pro端口号为：server.port=8082，并且在application.properites中指定dev环境

#### 修改config

```java
@Bean
    public Docket getDocket(Environment environment){
        //指定swagger扫描指定的配置文件
        Profiles profiles = Profiles.of("dev");
        boolean flag = environment.acceptsProfiles(profiles);
        return new Docket
                (new DocumentationType("swagger", "2.0")).groupName("hello")
                .apiInfo(getApiInfo())
                //是否开启swagger
                .enable(flag)
                //RequestHandlerSelectors.basePackage指定要扫描的包
                //RequestHandlerSelectors.any()扫描所有包
                //RequestHandlerSelectors.none()任何包都不扫描
                //RequestHandlerSelectors.withMethodAnnotation()扫描方法上的指定注解
                //RequestHandlerSelectors.withMethodAnnotation()扫描类上的指定注解
                .select().apis(RequestHandlerSelectors.basePackage("com.swagger.controller"))
                //配置要拦截的路径，any:拦截所有，ant:拦截指定路径，none,都不拦截
                //.paths(PathSelectors.any())
                .build();
    }
    public ApiInfo getApiInfo(){
        //作者信息
        Contact DEFAULT_CONTACT = new Contact("hello", "https://www.baidu.com/", "213233945@qq.com");
        return  new ApiInfo(
                "项目文档",
                "项目文档接口",
                "1.0",
                "https://www.baidu.com/",
                DEFAULT_CONTACT,
                "Apache 2.0",
                "http://www.apache.org/licenses/LICENSE-2.0",
                new ArrayList<>());
    }
}
```

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/26/kuangstudycdcacfa0-19ea-4cb0-815f-25d6f16d6aca.png)

### 在线测试

```java
 @ApiOperation("获取用户的controller2")
    @GetMapping("/user2")
    public User user(User user) {
        return user;
    }
```

**在UI界面中点击Try it Out**
![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/26/kuangstudyd859e903-1556-4a74-a62e-b469c47ae30b.png)
**输入参数**
![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/26/kuangstudyc0b126f5-9d44-4740-909b-86ed734c7c23.png)
**点击excute,测试**
![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/26/kuangstudy299d08f4-5b1b-4b56-bd5c-448fb8928b7b.png)

## Swagger的常用注解

在上述代码中或多或少都已经接触了swagger的常用注解，其实这些注解都是一些注释，他们的存在只是为了方便阅读与调试，如果嫌麻烦，甚至可以完全不用，但是在实际的项目中我们常常会使用他们，因此，掌握他们是非常有必要的，下面进行详解。

> [@Api](https://github.com/Api): 修饰整个类，用于controller类上
> [@ApiOperation](https://github.com/ApiOperation): 描述一个接口，作用于controller方法上
> [@ApiParam](https://github.com/ApiParam): 单个参数描述
> [@ApiModel](https://github.com/ApiModel): 用来对象接收参数,即返回对象
> [@ApiModelProperty](https://github.com/ApiModelProperty): 对象接收参数时，描述对象的字段，用于entity类中的字段
> [@ApiResponse](https://github.com/ApiResponse): Http响应其中的描述，在ApiResonse中
> [@ApiResponses](https://github.com/ApiResponses): Http响应所有的描述，用在
> [@ApiIgnore](https://github.com/ApiIgnore): 忽略这个API
> [@ApiError](https://github.com/ApiError): 发生错误的返回信息
> [@ApiImplicitParam](https://github.com/ApiImplicitParam): 一个请求参数
> [@ApiImplicitParams](https://github.com/ApiImplicitParams): 多个请求参数

### 示例：

`@ApiModel("User实体类") public class User {}`
![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/26/kuangstudy8697884a-4a5f-49dc-b9e2-2eba195133bb.png)

`@ApiModelProperty("用户名") private String username; @ApiModelProperty("密码") private String password;`
![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/26/kuangstudya0825500-9b76-4cb4-b78f-319435475b7e.png)

`@ApiOperation("获取用户的controller") @GetMapping("/user") public User User() { return new User(); }`
![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/26/kuangstudy1f1610df-9ecd-40d4-8af8-64ae1e547449.png)









转载自狂神

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------



版权声明：本文为博主原创文章，遵循[CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/)版权协议,转载请附上原文出处链接和本声明，KuangStudy,以学为伴，一生相伴！

[本文链接：https://www.kuangstudy.com/bbs/1375417006890790914](https://www.kuangstudy.com/bbs/1375417006890790914)