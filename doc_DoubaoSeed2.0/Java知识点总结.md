
# MyBatis Spring Boot JPetStore 项目知识点详解

## 一、项目概述

### 1.1 项目简介

这是一个基于 MyBatis、Spring Boot 和 Thymeleaf 构建的宠物商店 Web 应用程序。作为一名 C++ 程序员学习 Java 的入门项目，它非常适合，因为：

- 完整展示了现代 Java Web 应用的开发模式
- 包含了企业级应用的关键组件
- 使用了主流的技术栈

### 1.2 项目核心功能

- 用户注册和登录
- 商品浏览和搜索
- 购物车管理
- 订单管理
- 用户账户管理

### 1.3 环境要求

- **Java 8+ (JDK 1.8+)**：最低要求，推荐使用 JDK 8 或更高版本
- **Maven 3.6+**：项目构建工具（项目已包含 Maven Wrapper）
- **IDE 支持**（可选）：IntelliJ IDEA、Eclipse 或 STS
- **IDE 插件**（如使用 IDE 开发）：
  - Lombok 插件：用于支持 Lombok 注解
  - Groovy 插件（Eclipse/STS）：用于支持 Groovy 编写的 Mapper

### 1.4 项目快速启动指南

#### 方式一：使用 Maven Wrapper（推荐，无需安装 Maven）

**Windows 系统：**

```powershell
# 1. 进入项目目录
cd d:\Project\Java\mybatis-spring-boot-jpetstore

# 2. 直接启动项目
mvnw.cmd clean spring-boot:run
```

**Linux/Mac 系统：**

```bash
# 1. 进入项目目录
cd /path/to/mybatis-spring-boot-jpetstore

# 2. 直接启动项目
./mvnw clean spring-boot:run
```

#### 方式二：使用已安装的 Maven

```bash
# 1. 进入项目目录
cd d:\Project\Java\mybatis-spring-boot-jpetstore

# 2. 编译并启动
mvn clean spring-boot:run
```

#### 方式三：先打包再运行 JAR

```bash
# 1. 打包（跳过测试）
mvnw.cmd clean package -DskipTests=true

# 2. 运行 JAR 包
java -jar target/mybatis-spring-boot-jpetstore-2.0.0-SNAPSHOT.jar
```

#### 方式四：在 IDE 中运行

1. **导入项目**：在 IDE 中导入 Maven 项目
2. **安装 Lombok 插件**：确保 IDE 已安装 Lombok 插件
3. **配置 Groovy 支持**（Eclipse/STS）：
   - 安装 Groovy Eclipse 插件
   - 将 `src/main/groovy` 添加为源码目录
4. **运行主类**：找到 `JpetstoreApplication.java`，右键运行

### 1.5 访问应用

启动成功后，在浏览器中访问：

- **首页**：http://localhost:8080/
- **商品目录**：http://localhost:8080/catalog

### 1.6 默认测试账号

项目预置了两个测试账号：

| 用户名 | 密码 | 说明 |
|--------|------|------|
| j2ee | j2ee | 测试账号1 |
| ACID | ACID | 测试账号2 |

### 1.7 数据库说明

项目使用 **HSQLDB 嵌入式数据库**，数据存储在用户主目录下：

```
用户主目录/
  └── db/
      ├── jpetstore.script      # 数据库脚本
      └── jpetstore.properties  # 数据库配置
```

数据库初始化由 **Flyway** 自动完成，SQL 脚本位于：
- `src/main/resources/db/migration/V1.0.0.01__create_schema.sql` - 创建表结构
- `src/main/resources/db/migration/V1.0.0.02__import_data.sql` - 导入初始数据

### 1.8 运行集成测试

```bash
# 运行完整的集成测试
mvnw.cmd clean test
```

### 1.9 常见问题

**问题1：编译时找不到 Lombok 生成的 getter/setter**

解决：确保 IDE 已安装 Lombok 插件并启用注解处理

**问题2：Groovy 文件报错**

解决（Eclipse/STS）：安装 Groovy Eclipse 插件，将 `src/main/groovy` 添加为源码目录

**问题3：端口 8080 被占用**

解决：修改 `application.properties` 添加 `server.port=8081` 或其他端口

---

## 二、技术栈详解

### 2.1 Spring Boot 2.7

#### 什么是 Spring Boot？

Spring Boot 是一个基于 Spring 框架的快速开发框架，它：

- 简化了 Spring 应用的初始搭建和开发过程
- 提供了自动化配置（auto-configuration）
- 内置了 Tomcat 等 Web 容器
- 提供了统一的依赖管理

#### 核心概念

**@SpringBootApplication 注解**

```java
@SpringBootApplication
public class JpetstoreApplication {
    public static void main(String[] args) {
        SpringApplication.run(JpetstoreApplication.class, args);
    }
}
```

这是项目的入口类。`@SpringBootApplication` 是一个组合注解，包含：

- `@Configuration`：标记为配置类
- `@EnableAutoConfiguration`：启用自动配置
- `@ComponentScan`：自动扫描组件

#### C++ 程序员视角理解

可以把 Spring Boot 看作是一个**应用框架引擎**，类似于 C++ 中的 Qt 或 Boost，但更专注于 Web 应用。它处理了大量的底层配置，让你专注于业务逻辑。

---

### 2.2 MyBatis 3.5

#### 什么是 MyBatis？

MyBatis 是一个优秀的持久层框架，它：

- 支持自定义 SQL、存储过程
- 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集的工作
- 可以使用简单的 XML 或注解进行配置和原始映射

#### Mapper 接口（数据访问层）

```groovy
@Mapper
@CacheNamespace
interface AccountMapper {
    @Select('''
        SELECT
            SIGNON.USERNAME,
            SIGNON.PASSWORD,
            ACCOUNT.EMAIL,
            ...
        FROM
            ACCOUNT, PROFILE, SIGNON, BANNERDATA
        WHERE
            ACCOUNT.USERID = #{username}
            AND SIGNON.USERNAME = ACCOUNT.USERID
            ...
    ''')
    Account getAccountByUsername(String username)
}
```

**关键注解：**

- `@Mapper`：标记这是一个 MyBatis Mapper 接口
- `@CacheNamespace`：启用二级缓存
- `@Select`、`@Insert`、`@Update`、`@Delete`：定义 SQL 操作

#### C++ 程序员视角理解

MyBatis 类似于 C++ 中的 ORM 库（如 ODB），但更灵活。它把数据库操作封装成 Java 方法调用，SQL 语句和 Java 代码分离但又紧密结合。

---

### 2.3 Spring MVC

#### 什么是 Spring MVC？

Spring MVC 是 Spring 框架的 Web 模块，基于 MVC（Model-View-Controller）设计模式。

#### Controller（控制器层）

```java
@RequestMapping("/accounts")
@Controller
@RequiredArgsConstructor
public class AccountController {
    private final AccountService accountService;

    @GetMapping(path = "/create", params = "form")
    public String createForm() {
        return "account/createForm";
    }

    @PostMapping("/create")
    public String create(@Validated AccountForm form, BindingResult result,
                        Model model, RedirectAttributes redirectAttributes) {
        if (result.hasErrors()) {
            model.addAttribute(new Messages().error("Input values are invalid."));
            return createForm();
        }
        accountService.createAccount(form.toAccount());
        redirectAttributes.addFlashAttribute(new Messages().success("Account created!"));
        return "redirect:/login";
    }
}
```

**关键注解：**

- `@Controller`：标记为控制器组件
- `@RequestMapping`：定义 URL 映射
- `@GetMapping`、`@PostMapping`：定义 HTTP 方法映射
- `@Validated`：启用参数验证

#### C++ 程序员视角理解

Controller 类似于 C++ Web 框架中的请求处理器。Spring MVC 处理了 HTTP 请求的解析、路由、参数绑定等繁琐工作。

---

### 2.4 Spring Security

#### 什么是 Spring Security？

Spring Security 是一个强大且高度可定制的认证和访问控制框架。

#### 安全配置

```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
                .loginPage("/login")
                .defaultSuccessUrl("/catalog")
                .permitAll();
        http.logout()
                .logoutRequestMatcher(new AntPathRequestMatcher("/logout"))
                .logoutSuccessUrl("/catalog")
                .deleteCookies("JSESSIONID")
                .permitAll();
        http.authorizeRequests()
                .mvcMatchers("/my/**").authenticated()
                .anyRequest().permitAll();
    }
}
```

**核心功能：**

- 认证（Authentication）：验证用户身份
- 授权（Authorization）：控制用户访问权限
- 密码加密

#### 密码加密

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new Pbkdf2PasswordEncoder();
}
```

#### C++ 程序员视角理解

Spring Security 提供了完整的安全基础设施，类似于 C++ 中的安全库，但集成度更高，开箱即用。

---

### 2.5 Thymeleaf

#### 什么是 Thymeleaf？

Thymeleaf 是一个现代的服务器端 Java 模板引擎，用于 Web 和独立环境。

#### 模板示例

```html
&lt;!DOCTYPE html&gt;
&lt;html xmlns:th="http://www.thymeleaf.org"&gt;
&lt;head&gt;
    &lt;title&gt;Login&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;form th:action="@{/login}" method="post"&gt;
        &lt;div&gt;
            &lt;label&gt;Username:&lt;/label&gt;
            &lt;input type="text" name="username"/&gt;
        &lt;/div&gt;
        &lt;div&gt;
            &lt;label&gt;Password:&lt;/label&gt;
            &lt;input type="password" name="password"/&gt;
        &lt;/div&gt;
        &lt;button type="submit"&gt;Login&lt;/button&gt;
    &lt;/form&gt;
&lt;/body&gt;
&lt;/html&gt;
```

#### C++ 程序员视角理解

Thymeleaf 类似于 C++ Web 开发中的模板引擎（如 CTemplate），但它与 Spring 集成得更好，支持复杂的表达式和国际化。

---

### 2.6 Maven

#### 什么是 Maven？

Maven 是一个项目管理和构建自动化工具。

#### pom.xml（项目对象模型）

```xml
&lt;project xmlns="http://maven.apache.org/POM/4.0.0"&gt;
    &lt;modelVersion&gt;4.0.0&lt;/modelVersion&gt;
    &lt;groupId&gt;com.kazuki43zoo.examples&lt;/groupId&gt;
    &lt;artifactId&gt;mybatis-spring-boot-jpetstore&lt;/artifactId&gt;
    &lt;version&gt;2.0.0-SNAPSHOT&lt;/version&gt;
    &lt;packaging&gt;jar&lt;/packaging&gt;

    &lt;parent&gt;
        &lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
        &lt;artifactId&gt;spring-boot-starter-parent&lt;/artifactId&gt;
        &lt;version&gt;2.7.0&lt;/version&gt;
    &lt;/parent&gt;

    &lt;dependencies&gt;
        &lt;dependency&gt;
            &lt;groupId&gt;org.mybatis.spring.boot&lt;/groupId&gt;
            &lt;artifactId&gt;mybatis-spring-boot-starter&lt;/artifactId&gt;
            &lt;version&gt;2.2.2&lt;/version&gt;
        &lt;/dependency&gt;
        &lt;!-- 更多依赖... --&gt;
    &lt;/dependencies&gt;
&lt;/project&gt;
```

**核心概念：**

- `groupId`、`artifactId`、`version`：项目坐标
- `dependencies`：项目依赖
- `parent`：父 POM，继承配置

#### C++ 程序员视角理解

Maven 类似于 C++ 中的 CMake 或 Conan，它处理依赖管理、构建流程、项目结构等。

---

### 2.7 Lombok

#### 什么是 Lombok？

Lombok 是一个 Java 库，通过注解自动生成样板代码。

#### 使用示例

```java
@Getter
@Setter
@EqualsAndHashCode
public class Product implements Serializable {
    private String productId;
    private String categoryId;
    private String name;
    private String description;
}
```

**常用注解：**

- `@Getter`、`@Setter`：自动生成 getter/setter
- `@EqualsAndHashCode`：自动生成 equals 和 hashCode
- `@RequiredArgsConstructor`：自动生成构造函数
- `@Data`：包含以上所有功能

#### C++ 程序员视角理解

Lombok 类似于 C++ 中的代码生成工具或宏，它在编译时自动生成代码，减少了大量的样板代码。

---

### 2.8 Groovy

#### 什么是 Groovy？

Groovy 是一种基于 JVM 的动态语言，与 Java 无缝集成。

#### 在项目中的应用

项目使用 Groovy 编写 Mapper 接口，主要利用其多行字符串特性：

```groovy
@Select('''
    SELECT
        PRODUCTID,
        NAME,
        DESCN as description,
        CATEGORY as categoryId
    FROM
        PRODUCT
    WHERE
        CATEGORY = #{categoryId}
''')
List&lt;Product&gt; getProductListByCategory(String categoryId);
```

---

### 2.9 Flyway

#### 什么是 Flyway？

Flyway 是一个数据库版本控制工具。

#### 数据库迁移

SQL 文件放在 `src/main/resources/db/migration` 目录：

- `V1.0.0.01__create_schema.sql`：创建表结构
- `V1.0.0.02__import_data.sql`：导入初始数据

---

### 2.10 HSQLDB

#### 什么是 HSQLDB？

HSQLDB（HyperSQL Database）是一个纯 Java 编写的关系型数据库，可以作为嵌入式数据库运行。

#### 配置

```properties
spring.datasource.url=jdbc:hsqldb:file:~/db/jpetstore
```

---

## 三、项目架构详解

### 3.1 分层架构

项目采用经典的**分层架构**：

```
┌─────────────────────────────────────────┐
│           UI 层 (Controller)             │  ← 处理HTTP请求
├─────────────────────────────────────────┤
│         Service 层 (Service)            │  ← 业务逻辑
├─────────────────────────────────────────┤
│        Mapper 层 (Mapper)               │  ← 数据访问
├─────────────────────────────────────────┤
│         Domain 层 (Entity)              │  ← 数据模型
├─────────────────────────────────────────┤
│         Database (HSQLDB)               │  ← 数据存储
└─────────────────────────────────────────┘
```

### 3.2 目录结构

```
src/main/
├── groovy/                    # Groovy 源代码
│   └── com/kazuki43zoo/jpetstore/
│       └── mapper/            # MyBatis Mapper 接口
├── java/                      # Java 源代码
│   └── com/kazuki43zoo/jpetstore/
│       ├── component/         # 通用组件
│       │   ├── event/         # 事件相关
│       │   ├── exception/     # 异常类
│       │   ├── message/       # 消息组件
│       │   └── validation/    # 验证注解
│       ├── config/            # 配置类
│       ├── domain/            # 领域模型
│       ├── service/           # 服务层
│       └── ui/                # UI 层
│           └── controller/    # 控制器
└── resources/                 # 资源文件
    ├── db/migration/          # Flyway 数据库迁移
    ├── static/                # 静态资源
    └── templates/             # Thymeleaf 模板
```

### 3.3 请求处理流程

以创建用户账户为例：

1. **用户提交表单** → POST `/accounts/create`
2. **Controller 接收** → `AccountController.create()`
3. **参数验证** → `@Validated` 验证 `AccountForm`
4. **Service 处理** → `AccountService.createAccount()`
5. **事务处理** → `@Transactional` 确保数据一致性
6. **密码加密** → `PasswordEncoder` 加密密码
7. **数据持久化** → `AccountMapper` 执行 SQL
8. **返回响应** → 重定向到登录页面

---

## 四、核心组件详解

### 4.1 领域模型（Domain）

#### Account 类

```java
@Getter
@Setter
public class Account implements Serializable {
    private String username;
    private String password;
    private String email;
    private String firstName;
    private String lastName;
    // ... 更多字段
}
```

**关键点：**

- 实现 `Serializable` 接口：支持序列化
- 使用 Lombok 注解：自动生成 getter/setter

#### Product 类

```java
@Getter
@Setter
@EqualsAndHashCode
public class Product implements Serializable {
    private String productId;
    private String categoryId;
    private String name;
    private String description;
}
```

**C++ 程序员视角理解**：

这些类类似于 C++ 中的结构体或简单类，用于封装数据。`Serializable` 接口类似于 C++ 中需要实现序列化功能的基类。

---

### 4.2 服务层（Service）

#### AccountService

```java
@Service
@RequiredArgsConstructor
public class AccountService {
    private final AccountMapper accountMapper;
    private final PasswordEncoder passwordEncoder;

    @Transactional
    public void createAccount(Account account) {
        account.setPassword(passwordEncoder.encode(account.getPassword()));
        accountMapper.insertAccount(account);
        accountMapper.insertProfile(account);
        accountMapper.insertSignon(account);
    }

    @Transactional
    public void updateAccount(Account account, String newPassword) {
        accountMapper.updateAccount(account);
        accountMapper.updateProfile(account);
        Optional.ofNullable(newPassword).ifPresent(x -&gt; {
            account.setPassword(passwordEncoder.encode(x));
            accountMapper.updateSignon(account);
        });
    }
}
```

**关键注解：**

- `@Service`：标记为服务层组件
- `@RequiredArgsConstructor`：Lombok 注解，生成包含 final 字段的构造函数
- `@Transactional`：声明式事务管理

**关键特性：**

- **依赖注入**：通过构造函数注入依赖
- **事务管理**：`@Transactional` 确保多个数据库操作的原子性
- **密码加密**：使用 `PasswordEncoder` 安全存储密码
- **Optional**：优雅处理可能为 null 的值

**C++ 程序员视角理解**：

- `@RequiredArgsConstructor` 类似于 C++ 中自动生成构造函数
- `@Transactional` 类似于 C++ 中的 RAII 或事务管理器
- `Optional` 类似于 C++17 中的 `std::optional`

---

### 4.3 数据访问层（Mapper）

#### AccountMapper

```groovy
@Mapper
@CacheNamespace
interface AccountMapper {
    @Select('''
        SELECT
            SIGNON.USERNAME,
            SIGNON.PASSWORD,
            ACCOUNT.EMAIL,
            ACCOUNT.FIRSTNAME,
            ACCOUNT.LASTNAME,
            ...
        FROM
            ACCOUNT, PROFILE, SIGNON, BANNERDATA
        WHERE
            ACCOUNT.USERID = #{username}
            AND SIGNON.USERNAME = ACCOUNT.USERID
            AND PROFILE.USERID = ACCOUNT.USERID
            AND PROFILE.FAVCATEGORY = BANNERDATA.FAVCATEGORY
    ''')
    Account getAccountByUsername(String username);

    @Insert('''
        INSERT INTO ACCOUNT (EMAIL, FIRSTNAME, LASTNAME, ...)
        VALUES (#{email}, #{firstName}, #{lastName}, ...)
    ''')
    void insertAccount(Account account);
}
```

**关键点：**

- 使用注解定义 SQL
- `#{}` 语法：参数绑定，防止 SQL 注入
- `@CacheNamespace`：启用二级缓存

---

### 4.4 配置类

#### ApplicationConfig

```java
@Configuration
public class ApplicationConfig {
    @Bean
    Clock clock() {
        return Clock.systemDefaultZone();
    }

    @Bean
    PasswordEncoder passwordEncoder() {
        return new Pbkdf2PasswordEncoder();
    }

    @Bean
    List&lt;String&gt; clCreditCardTypes() {
        return Arrays.asList("Visa", "MasterCard", "American Express");
    }
}
```

**关键注解：**

- `@Configuration`：标记为配置类
- `@Bean`：定义一个 Bean，由 Spring 容器管理

---

## 五、关键 Java 概念详解

### 5.1 注解（Annotation）

#### 什么是注解？

注解是一种元数据，可以添加到 Java 代码中，提供额外的信息但不直接影响代码执行。

#### 常见注解

```java
// 组件注解
@Component          // 通用组件
@Service            // 服务层组件
@Controller         // 控制器组件
@Repository         // 数据访问层组件
@Configuration      // 配置类

// Web 注解
@RequestMapping     // URL 映射
@GetMapping         // GET 请求
@PostMapping        // POST 请求
@RequestParam       // 请求参数
@PathVariable       // URL 路径变量

// 事务注解
@Transactional      // 事务管理

// Lombok 注解
@Getter             // 生成 getter
@Setter             // 生成 setter
@Data               // 生成 getter/setter/toString/equals/hashCode
@RequiredArgsConstructor  // 生成构造函数
```

#### C++ 程序员视角理解

注解类似于 C++11 中的属性（attributes），但功能更强大，被广泛用于框架中实现各种功能。

---

### 5.2 依赖注入（Dependency Injection, DI）

#### 什么是依赖注入？

依赖注入是一种设计模式，让对象的依赖由外部提供，而不是对象自己创建。

#### 构造函数注入（推荐）

```java
@Service
@RequiredArgsConstructor
public class AccountService {
    private final AccountMapper accountMapper;
    private final PasswordEncoder passwordEncoder;
    // Lombok 自动生成构造函数
}
```

#### 字段注入（不推荐）

```java
@Service
public class AccountService {
    @Autowired
    private AccountMapper accountMapper;
}
```

#### C++ 程序员视角理解

依赖注入类似于 C++ 中通过构造函数或 setter 传递依赖对象，Spring 容器自动完成这个过程。

---

### 5.3 控制反转（Inversion of Control, IoC）

#### 什么是控制反转？

控制反转是一种设计原则，将对象的创建和管理交给容器（如 Spring 容器）。

#### Spring IoC 容器

Spring 容器负责：

- 创建 Bean
- 管理 Bean 的生命周期
- 注入 Bean 的依赖

---

### 5.4 Bean

#### 什么是 Bean？

Bean 是由 Spring IoC 容器管理的对象。

#### Bean 的生命周期

1. 实例化
2. 属性赋值
3. 初始化
4. 使用
5. 销毁

---

### 5.5 事务管理

#### 声明式事务

```java
@Transactional
public void createAccount(Account account) {
    // 多个数据库操作
    accountMapper.insertAccount(account);
    accountMapper.insertProfile(account);
    accountMapper.insertSignon(account);
}
```

**事务特性（ACID）：**

- **原子性（Atomicity）**：全部成功或全部失败
- **一致性（Consistency）**：从一个一致状态到另一个
- **隔离性（Isolation）**：事务之间互不干扰
- **持久性（Durability）**：一旦提交，永久保存

---

### 5.6 异常处理

#### 自定义异常

```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

---

### 5.7 数据验证

#### Bean Validation

```java
public class AccountForm {
    @NotBlank
    @Size(max = 80)
    private String username;

    @NotBlank
    @Size(min = 8, max = 80)
    private String password;

    @NotBlank
    @Email
    private String email;
}
```

#### 验证器

```java
@PostMapping("/create")
public String create(@Validated AccountForm form, BindingResult result, ...) {
    if (result.hasErrors()) {
        // 处理验证错误
    }
    // ...
}
```

---

### 5.8 Optional

#### 什么是 Optional？

`Optional` 是 Java 8 引入的类，用于优雅处理可能为 null 的值。

#### 使用示例

```java
Optional.ofNullable(newPassword).ifPresent(x -&gt; {
    account.setPassword(passwordEncoder.encode(x));
    accountMapper.updateSignon(account);
});
```

#### 常用方法

- `ofNullable()`：创建可能为 null 的 Optional
- `ifPresent()`：如果值存在，执行操作
- `orElse()`：如果值不存在，返回默认值
- `map()`：转换值

---

### 5.9 Lambda 表达式

#### 什么是 Lambda？

Lambda 是 Java 8 引入的函数式编程特性。

#### 使用示例

```java
Optional.ofNullable(newPassword).ifPresent(x -&gt; {
    account.setPassword(passwordEncoder.encode(x));
    accountMapper.updateSignon(account);
});
```

#### C++ 程序员视角理解

Lambda 表达式类似于 C++11 中的 lambda 表达式，语法略有不同但概念相似。

---

### 5.10 Stream API

#### 什么是 Stream？

Stream 是 Java 8 引入的用于处理集合的 API。

虽然本项目中没有大量使用，但它是 Java 8+ 的重要特性。

---

## 六、Spring Boot 自动配置

### 6.1 什么是自动配置？

Spring Boot 自动配置根据项目依赖自动配置 Spring 应用。

### 6.2 工作原理

1. 检查 classpath 中的依赖
2. 根据依赖推断需要的配置
3. 自动配置相应的 Bean

### 6.3 配置文件

#### application.properties

```properties
spring.profiles.include=common
spring.datasource.url=jdbc:hsqldb:file:~/db/jpetstore
```

#### 配置属性注入

```java
@Configuration
public class MyConfig {
    @Value("${my.property}")
    private String myProperty;
}
```

---

## 七、数据库操作

### 7.1 MyBatis 映射

#### 结果映射

MyBatis 自动将查询结果映射到 Java 对象：

- 列名 → 字段名（自动转换下划线命名到驼峰命名）
- 使用 `AS` 关键字手动映射

```groovy
@Select('''
    SELECT
        PRODUCTID,
        NAME,
        DESCN as description,
        CATEGORY as categoryId
    FROM
        PRODUCT
''')
List&lt;Product&gt; getProductList();
```

### 7.2 参数绑定

使用 `#{}` 语法绑定参数，防止 SQL 注入：

```groovy
@Select('''
    SELECT * FROM PRODUCT WHERE PRODUCTID = #{productId}
''')
Product getProduct(String productId);
```

---

## 八、Web 开发

### 8.1 请求映射

```java
@Controller
@RequestMapping("/products")
public class ProductController {
    @GetMapping("/{productId}")
    public String getProduct(@PathVariable String productId, Model model) {
        // ...
    }

    @GetMapping
    public String listProducts(@RequestParam String categoryId, Model model) {
        // ...
    }
}
```

### 8.2 表单处理

```java
@PostMapping("/create")
public String create(@Validated AccountForm form, BindingResult result,
                    Model model, RedirectAttributes redirectAttributes) {
    if (result.hasErrors()) {
        return "account/createForm";
    }
    accountService.createAccount(form.toAccount());
    redirectAttributes.addFlashAttribute("message", "Account created!");
    return "redirect:/login";
}
```

### 8.3 数据模型

```java
@GetMapping("/products")
public String listProducts(Model model) {
    List&lt;Product&gt; products = productService.getProducts();
    model.addAttribute("products", products);
    return "product/list";
}
```

---

## 九、安全性

### 9.1 认证

```java
http.formLogin()
    .loginPage("/login")
    .defaultSuccessUrl("/catalog")
    .permitAll();
```

### 9.2 授权

```java
http.authorizeRequests()
    .mvcMatchers("/my/**").authenticated()
    .anyRequest().permitAll();
```

### 9.3 密码加密

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new Pbkdf2PasswordEncoder();
}
```

---

## 十、测试

### 10.1 单元测试

```java
@SpringBootTest
class JpetstoreApplicationTests {
    @Test
    void contextLoads() {
    }
}
```

---

## 十一、构建和运行

### 11.1 Maven 常用命令

```bash
# 编译项目
mvnw.cmd clean compile

# 运行测试
mvnw.cmd test

# 打包项目（包含依赖）
mvnw.cmd clean package

# 打包项目（跳过测试）
mvnw.cmd clean package -DskipTests=true

# 直接运行项目
mvnw.cmd spring-boot:run

# 清理项目
mvnw.cmd clean
```

### 11.2 运行打包后的 JAR

```bash
# 先打包
mvnw.cmd clean package -DskipTests=true

# 再运行 JAR 包
java -jar target/mybatis-spring-boot-jpetstore-2.0.0-SNAPSHOT.jar
```

### 11.3 Maven Wrapper 说明

项目包含 **Maven Wrapper**（`mvnw` 和 `mvnw.cmd`），好处是：
- 无需在系统中安装 Maven
- 确保所有开发者使用相同版本的 Maven
- 便于 CI/CD 流水线构建

### 11.4 项目启动日志

正常启动时，你会看到类似这样的日志：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::       (v2.7.0)

...
Started JpetstoreApplication in X.XXX seconds
```

看到 `Started JpetstoreApplication` 表示启动成功！

---

## 十二、C++ 程序员学习建议

### 12.1 思维转换

1. **垃圾回收**：Java 有 GC，不需要手动管理内存
2. **异常处理**：Java 有受检异常和非受检异常
3. **面向接口编程**：Java 更强调接口和抽象
4. **依赖注入**：Spring 的核心思想
5. **注解驱动**：大量使用注解配置

### 12.2 学习路径

1. **Java 基础**：语法、OOP、集合、异常
2. **Spring 核心**：IoC、DI、AOP
3. **Spring Boot**：自动配置、起步依赖
4. **持久层**：MyBatis/JPA
5. **Web 开发**：Spring MVC
6. **安全**：Spring Security
7. **测试**：JUnit、Mockito

### 12.3 关键差异

| 特性 | Java | C++ |
|------|------|-----|
| 内存管理 | GC 自动管理 | 手动管理（new/delete） |
| 指针 | 无（引用） | 有指针 |
| 多继承 | 接口多继承 | 类多继承 |
| 模板 | 泛型（类型擦除） | 模板（编译时） |
| 异常 | 受检/非受检异常 | 无受检异常 |

---

## 十三、扩展学习资源

### 13.1 官方文档

- [Spring Boot 官方文档](https://spring.io/projects/spring-boot)
- [MyBatis 官方文档](https://mybatis.org/mybatis-3/)
- [Spring Security 官方文档](https://spring.io/projects/spring-security)
- [Thymeleaf 官方文档](https://www.thymeleaf.org/)

### 13.2 推荐书籍

- 《Spring Boot 实战》
- 《MyBatis 从入门到精通》
- 《Java 编程思想》
- 《Effective Java》

---

## 十四、总结

这个项目是一个非常好的 Java Web 开发入门示例，它涵盖了：

1. **现代 Java 技术栈**：Spring Boot、MyBatis、Spring Security、Thymeleaf
2. **经典架构模式**：分层架构、MVC
3. **最佳实践**：依赖注入、事务管理、密码加密、数据验证
4. **工具链**：Maven、Lombok、Flyway

作为一名 C++ 程序员，学习 Java 需要：

- 理解 Java 的面向对象特性
- 适应垃圾回收机制
- 掌握 Spring 生态系统
- 学会使用注解和配置

希望这份总结能帮助你更好地理解和学习这个项目！

