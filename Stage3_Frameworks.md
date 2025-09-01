# 第三阶段：框架技术 - Spring全家桶、MyBatis、消息队列

## 🎯 学习目标

本阶段将深入学习Java企业级开发中的核心框架技术，包括Spring生态系统、持久层框架MyBatis、消息队列等。通过本阶段学习，你将能够：

- 掌握Spring框架的核心概念和使用方法
- 熟练使用Spring Boot进行快速开发
- 理解Spring MVC的工作原理和最佳实践
- 掌握Spring Data JPA和MyBatis的使用
- 了解消息队列的原理和应用场景
- 能够构建完整的企业级Web应用

## 📚 知识体系

### 1. Spring框架核心

#### 1.1 IoC容器和依赖注入

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.*;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;
import org.springframework.stereotype.Repository;

/**
 * Spring IoC容器和依赖注入演示
 * 展示Spring的核心特性：控制反转和依赖注入
 */
public class SpringIoCDemo {
    
    public static void main(String[] args) {
        // 创建Spring应用上下文
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        
        // 从容器中获取Bean
        UserService userService = context.getBean(UserService.class);
        userService.createUser("张三", "zhangsan@example.com");
        
        // 演示不同的注入方式
        OrderService orderService = context.getBean(OrderService.class);
        orderService.createOrder("ORD001", 1L);
        
        // 演示Bean的作用域
        demonstrateBeanScope(context);
    }
    
    /**
     * 演示Bean的作用域
     */
    private static void demonstrateBeanScope(ApplicationContext context) {
        System.out.println("\n=== Bean作用域演示 ===");
        
        // Singleton作用域（默认）
        SingletonBean singleton1 = context.getBean(SingletonBean.class);
        SingletonBean singleton2 = context.getBean(SingletonBean.class);
        System.out.println("Singleton Bean相同实例: " + (singleton1 == singleton2));
        
        // Prototype作用域
        PrototypeBean prototype1 = context.getBean(PrototypeBean.class);
        PrototypeBean prototype2 = context.getBean(PrototypeBean.class);
        System.out.println("Prototype Bean不同实例: " + (prototype1 != prototype2));
    }
}

/**
 * Spring配置类
 * 使用@Configuration注解标记配置类
 */
@Configuration
@ComponentScan(basePackages = "com.example")
@EnableAspectJAutoProxy // 启用AOP
public class AppConfig {
    
    /**
     * 使用@Bean注解定义Bean
     * 演示手动配置Bean的方式
     */
    @Bean
    @Primary // 当有多个相同类型的Bean时，优先选择此Bean
    public EmailService primaryEmailService() {
        return new SmtpEmailService("smtp.example.com", 587);
    }
    
    @Bean
    @Qualifier("backup")
    public EmailService backupEmailService() {
        return new SmtpEmailService("backup.example.com", 587);
    }
    
    /**
     * 配置数据源（模拟）
     */
    @Bean
    public DataSource dataSource() {
        // 实际项目中这里会配置真实的数据源
        return new MockDataSource();
    }
}

/**
 * 用户实体类
 */
class User {
    private Long id;
    private String name;
    private String email;
    
    public User() {}
    
    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }
    
    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    @Override
    public String toString() {
        return String.format("User{id=%d, name='%s', email='%s'}", id, name, email);
    }
}

/**
 * 用户数据访问层
 * 使用@Repository注解标记数据访问层组件
 */
@Repository
class UserRepository {
    
    @Autowired
    private DataSource dataSource;
    
    public void save(User user) {
        System.out.println("保存用户到数据库: " + user);
        // 模拟数据库保存操作
        user.setId(System.currentTimeMillis());
    }
    
    public User findById(Long id) {
        System.out.println("从数据库查询用户ID: " + id);
        // 模拟数据库查询
        return new User("用户" + id, "user" + id + "@example.com");
    }
}

/**
 * 邮件服务接口
 */
interface EmailService {
    void sendEmail(String to, String subject, String content);
}

/**
 * SMTP邮件服务实现
 */
class SmtpEmailService implements EmailService {
    private String host;
    private int port;
    
    public SmtpEmailService(String host, int port) {
        this.host = host;
        this.port = port;
    }
    
    @Override
    public void sendEmail(String to, String subject, String content) {
        System.out.println(String.format("通过SMTP(%s:%d)发送邮件到%s: %s", 
                                       host, port, to, subject));
    }
}

/**
 * 用户服务层
 * 使用@Service注解标记业务逻辑层组件
 */
@Service
class UserService {
    
    // 字段注入（不推荐，仅用于演示）
    @Autowired
    private UserRepository userRepository;
    
    // 构造器注入（推荐方式）
    private final EmailService emailService;
    
    @Autowired
    public UserService(EmailService emailService) {
        this.emailService = emailService;
    }
    
    public void createUser(String name, String email) {
        System.out.println("\n=== 创建用户 ===");
        
        User user = new User(name, email);
        userRepository.save(user);
        
        // 发送欢迎邮件
        emailService.sendEmail(email, "欢迎注册", "欢迎您注册我们的服务！");
        
        System.out.println("用户创建成功: " + user);
    }
}

/**
 * 订单服务
 * 演示Setter注入和@Qualifier的使用
 */
@Service
class OrderService {
    
    private UserRepository userRepository;
    private EmailService emailService;
    
    // Setter注入
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    // 使用@Qualifier指定具体的Bean
    @Autowired
    @Qualifier("backup")
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
    
    public void createOrder(String orderNo, Long userId) {
        System.out.println("\n=== 创建订单 ===");
        
        User user = userRepository.findById(userId);
        System.out.println("订单创建成功: " + orderNo + ", 用户: " + user.getName());
        
        // 发送订单确认邮件
        emailService.sendEmail(user.getEmail(), "订单确认", 
                              "您的订单 " + orderNo + " 已创建成功！");
    }
}

/**
 * Singleton作用域Bean（默认）
 */
@Component
class SingletonBean {
    public SingletonBean() {
        System.out.println("创建SingletonBean实例: " + this.hashCode());
    }
}

/**
 * Prototype作用域Bean
 */
@Component
@Scope("prototype")
class PrototypeBean {
    public PrototypeBean() {
        System.out.println("创建PrototypeBean实例: " + this.hashCode());
    }
}

/**
 * 模拟数据源
 */
class MockDataSource {
    public MockDataSource() {
        System.out.println("初始化模拟数据源");
    }
}

/**
 * 数据源接口（简化版）
 */
interface DataSource {
    // 简化的数据源接口
}

/**
 * 模拟数据源实现
 */
class MockDataSource implements DataSource {
    public MockDataSource() {
        System.out.println("初始化模拟数据源");
    }
}
```

#### 1.2 Spring AOP（面向切面编程）

```java
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * Spring AOP演示
 * 展示面向切面编程的核心概念和使用方法
 */
public class SpringAOPDemo {
    
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AopConfig.class);
        
        // 获取被代理的服务
        CalculatorService calculator = context.getBean(CalculatorService.class);
        
        System.out.println("=== AOP演示 ===");
        
        // 执行方法，观察AOP的效果
        int result1 = calculator.add(10, 5);
        System.out.println("加法结果: " + result1);
        
        int result2 = calculator.multiply(4, 3);
        System.out.println("乘法结果: " + result2);
        
        // 演示异常处理切面
        try {
            calculator.divide(10, 0);
        } catch (Exception e) {
            System.out.println("捕获异常: " + e.getMessage());
        }
        
        // 演示自定义注解切面
        UserService userService = context.getBean(UserService.class);
        userService.sensitiveOperation("删除用户数据");
    }
}

/**
 * AOP配置类
 */
@Configuration
@ComponentScan
@EnableAspectJAutoProxy
class AopConfig {
}

/**
 * 计算器服务
 */
@Service
class CalculatorService {
    
    public int add(int a, int b) {
        System.out.println("执行加法运算: " + a + " + " + b);
        return a + b;
    }
    
    public int multiply(int a, int b) {
        System.out.println("执行乘法运算: " + a + " * " + b);
        return a * b;
    }
    
    public int divide(int a, int b) {
        System.out.println("执行除法运算: " + a + " / " + b);
        if (b == 0) {
            throw new IllegalArgumentException("除数不能为零");
        }
        return a / b;
    }
}

/**
 * 用户服务（用于演示自定义注解切面）
 */
@Service
class UserService {
    
    @AuditLog
    public void sensitiveOperation(String operation) {
        System.out.println("执行敏感操作: " + operation);
    }
    
    public void normalOperation() {
        System.out.println("执行普通操作");
    }
}

/**
 * 自定义审计日志注解
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@interface AuditLog {
    String value() default "";
}

/**
 * 日志切面
 * 使用@Aspect注解标记切面类
 */
@Aspect
@Component
class LoggingAspect {
    
    /**
     * 前置通知
     * 在目标方法执行前执行
     */
    @Before("execution(* CalculatorService.*(..))")
    public void beforeMethod(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        Object[] args = joinPoint.getArgs();
        System.out.println("[前置通知] 即将执行方法: " + methodName + ", 参数: " + java.util.Arrays.toString(args));
    }
    
    /**
     * 后置通知
     * 在目标方法执行后执行（无论是否抛出异常）
     */
    @After("execution(* CalculatorService.*(..))")
    public void afterMethod(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("[后置通知] 方法执行完成: " + methodName);
    }
    
    /**
     * 返回通知
     * 在目标方法正常返回后执行
     */
    @AfterReturning(pointcut = "execution(* CalculatorService.*(..))", returning = "result")
    public void afterReturning(JoinPoint joinPoint, Object result) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("[返回通知] 方法 " + methodName + " 返回值: " + result);
    }
    
    /**
     * 异常通知
     * 在目标方法抛出异常后执行
     */
    @AfterThrowing(pointcut = "execution(* CalculatorService.*(..))", throwing = "ex")
    public void afterThrowing(JoinPoint joinPoint, Exception ex) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("[异常通知] 方法 " + methodName + " 抛出异常: " + ex.getMessage());
    }
}

/**
 * 性能监控切面
 */
@Aspect
@Component
class PerformanceAspect {
    
    /**
     * 环绕通知
     * 可以在目标方法执行前后进行处理
     */
    @Around("execution(* CalculatorService.*(..))")
    public Object aroundMethod(ProceedingJoinPoint joinPoint) throws Throwable {
        String methodName = joinPoint.getSignature().getName();
        
        // 方法执行前
        long startTime = System.currentTimeMillis();
        System.out.println("[性能监控] 开始执行方法: " + methodName);
        
        try {
            // 执行目标方法
            Object result = joinPoint.proceed();
            
            // 方法执行后
            long endTime = System.currentTimeMillis();
            System.out.println("[性能监控] 方法 " + methodName + " 执行耗时: " + (endTime - startTime) + "ms");
            
            return result;
        } catch (Throwable throwable) {
            long endTime = System.currentTimeMillis();
            System.out.println("[性能监控] 方法 " + methodName + " 执行失败，耗时: " + (endTime - startTime) + "ms");
            throw throwable;
        }
    }
}

/**
 * 审计日志切面
 * 演示基于注解的切点表达式
 */
@Aspect
@Component
class AuditAspect {
    
    /**
     * 基于注解的切点
     * 只有标记了@AuditLog注解的方法才会被拦截
     */
    @Around("@annotation(auditLog)")
    public Object auditLog(ProceedingJoinPoint joinPoint, AuditLog auditLog) throws Throwable {
        String methodName = joinPoint.getSignature().getName();
        String className = joinPoint.getTarget().getClass().getSimpleName();
        
        System.out.println("[审计日志] 开始执行敏感操作: " + className + "." + methodName);
        System.out.println("[审计日志] 操作时间: " + new java.util.Date());
        System.out.println("[审计日志] 操作用户: 当前用户"); // 实际项目中从安全上下文获取
        
        try {
            Object result = joinPoint.proceed();
            System.out.println("[审计日志] 敏感操作执行成功");
            return result;
        } catch (Throwable throwable) {
            System.out.println("[审计日志] 敏感操作执行失败: " + throwable.getMessage());
            throw throwable;
        }
    }
}

/**
 * 切点表达式演示
 */
@Aspect
@Component
class PointcutExpressionAspect {
    
    /**
     * 定义可重用的切点
     */
    @Pointcut("execution(public * com.example.service.*.*(..))")
    public void serviceLayer() {}
    
    @Pointcut("execution(* *..repository.*.*(..))")
    public void repositoryLayer() {}
    
    @Pointcut("serviceLayer() || repositoryLayer()")
    public void businessLayer() {}
    
    /**
     * 使用定义的切点
     */
    @Before("businessLayer()")
    public void beforeBusinessMethod(JoinPoint joinPoint) {
        System.out.println("[业务层拦截] 执行业务方法: " + joinPoint.getSignature().getName());
    }
}
```

### 2. Spring Boot快速开发

#### 2.1 Spring Boot基础

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.*;

import javax.annotation.PostConstruct;
import java.util.*;

/**
 * Spring Boot应用主类
 * @SpringBootApplication包含了@Configuration、@EnableAutoConfiguration、@ComponentScan
 */
@SpringBootApplication
@EnableConfigurationProperties(AppProperties.class)
public class SpringBootDemoApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(SpringBootDemoApplication.class, args);
        System.out.println("Spring Boot应用启动成功！");
    }
    
    /**
     * 自定义Bean配置
     */
    @Bean
    public CustomService customService() {
        return new CustomService("Spring Boot Service");
    }
}

/**
 * 配置属性类
 * 演示Spring Boot的配置绑定功能
 */
@ConfigurationProperties(prefix = "app")
class AppProperties {
    private String name = "默认应用名称";
    private String version = "1.0.0";
    private Database database = new Database();
    private List<String> features = new ArrayList<>();
    
    // Getters and Setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getVersion() { return version; }
    public void setVersion(String version) { this.version = version; }
    public Database getDatabase() { return database; }
    public void setDatabase(Database database) { this.database = database; }
    public List<String> getFeatures() { return features; }
    public void setFeatures(List<String> features) { this.features = features; }
    
    public static class Database {
        private String url = "jdbc:h2:mem:testdb";
        private String username = "sa";
        private String password = "";
        
        // Getters and Setters
        public String getUrl() { return url; }
        public void setUrl(String url) { this.url = url; }
        public String getUsername() { return username; }
        public void setUsername(String username) { this.username = username; }
        public String getPassword() { return password; }
        public void setPassword(String password) { this.password = password; }
    }
}

/**
 * REST控制器
 * 演示Spring Boot的Web功能
 */
@RestController
@RequestMapping("/api")
class ApiController {
    
    private final AppProperties appProperties;
    private final CustomService customService;
    
    public ApiController(AppProperties appProperties, CustomService customService) {
        this.appProperties = appProperties;
        this.customService = customService;
    }
    
    /**
     * 获取应用信息
     */
    @GetMapping("/info")
    public Map<String, Object> getAppInfo() {
        Map<String, Object> info = new HashMap<>();
        info.put("name", appProperties.getName());
        info.put("version", appProperties.getVersion());
        info.put("features", appProperties.getFeatures());
        info.put("database", appProperties.getDatabase().getUrl());
        return info;
    }
    
    /**
     * 健康检查
     */
    @GetMapping("/health")
    public Map<String, String> health() {
        Map<String, String> status = new HashMap<>();
        status.put("status", "UP");
        status.put("timestamp", new Date().toString());
        status.put("service", customService.getServiceInfo());
        return status;
    }
    
    /**
     * 用户管理API
     */
    @GetMapping("/users")
    public List<User> getUsers() {
        return Arrays.asList(
            new User(1L, "张三", "zhangsan@example.com"),
            new User(2L, "李四", "lisi@example.com")
        );
    }
    
    @PostMapping("/users")
    public User createUser(@RequestBody User user) {
        user.setId(System.currentTimeMillis());
        System.out.println("创建用户: " + user);
        return user;
    }
    
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return new User(id, "用户" + id, "user" + id + "@example.com");
    }
    
    @PutMapping("/users/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody User user) {
        user.setId(id);
        System.out.println("更新用户: " + user);
        return user;
    }
    
    @DeleteMapping("/users/{id}")
    public Map<String, String> deleteUser(@PathVariable Long id) {
        System.out.println("删除用户ID: " + id);
        Map<String, String> result = new HashMap<>();
        result.put("message", "用户删除成功");
        result.put("id", id.toString());
        return result;
    }
}

/**
 * 自定义服务
 */
@Component
class CustomService {
    private final String serviceName;
    
    public CustomService(String serviceName) {
        this.serviceName = serviceName;
    }
    
    @PostConstruct
    public void init() {
        System.out.println("CustomService初始化完成: " + serviceName);
    }
    
    public String getServiceInfo() {
        return serviceName + " - 运行中";
    }
}

/**
 * 用户实体类
 */
class User {
    private Long id;
    private String name;
    private String email;
    
    public User() {}
    
    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }
    
    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    @Override
    public String toString() {
        return String.format("User{id=%d, name='%s', email='%s'}", id, name, email);
    }
}
```

#### 2.2 Spring Boot配置文件示例

**application.yml**
```yaml
# Spring Boot配置文件示例
app:
  name: "Spring Boot演示应用"
  version: "2.0.0"
  database:
    url: "jdbc:mysql://localhost:3306/demo"
    username: "root"
    password: "password"
  features:
    - "用户管理"
    - "订单处理"
    - "支付集成"

spring:
  # 数据源配置
  datasource:
    url: ${app.database.url}
    username: ${app.database.username}
    password: ${app.database.password}
    driver-class-name: com.mysql.cj.jdbc.Driver
    
  # JPA配置
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        
  # Redis配置
  redis:
    host: localhost
    port: 6379
    password: 
    database: 0
    
  # 消息队列配置
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
    
# 服务器配置
server:
  port: 8080
  servlet:
    context-path: /demo
    
# 日志配置
logging:
  level:
    com.example: DEBUG
    org.springframework: INFO
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
    
# 管理端点配置
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
  endpoint:
    health:
      show-details: always
```

**application.properties**
```properties
# Spring Boot配置文件示例（Properties格式）

# 应用配置
app.name=Spring Boot演示应用
app.version=2.0.0
app.database.url=jdbc:mysql://localhost:3306/demo
app.database.username=root
app.database.password=password
app.features[0]=用户管理
app.features[1]=订单处理
app.features[2]=支付集成

# 数据源配置
spring.datasource.url=${app.database.url}
spring.datasource.username=${app.database.username}
spring.datasource.password=${app.database.password}
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA配置
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Redis配置
spring.redis.host=localhost
spring.redis.port=6379
spring.redis.database=0

# 服务器配置
server.port=8080
server.servlet.context-path=/demo

# 日志配置
logging.level.com.example=DEBUG
logging.level.org.springframework=INFO
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n

# 管理端点配置
management.endpoints.web.exposure.include=health,info,metrics
management.endpoint.health.show-details=always
```

### 3. Spring MVC Web开发

#### 3.1 控制器和视图

```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import javax.validation.Valid;
import javax.validation.constraints.Email;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Size;
import java.util.ArrayList;
import java.util.List;

/**
 * Spring MVC控制器演示
 * 展示不同类型的控制器方法和视图处理
 */
@Controller
@RequestMapping("/web")
public class WebController {
    
    private List<User> users = new ArrayList<>();
    
    public WebController() {
        // 初始化一些测试数据
        users.add(new User(1L, "张三", "zhangsan@example.com"));
        users.add(new User(2L, "李四", "lisi@example.com"));
    }
    
    /**
     * 首页
     */
    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("title", "Spring MVC演示");
        model.addAttribute("message", "欢迎使用Spring MVC！");
        return "index"; // 返回视图名称
    }
    
    /**
     * 用户列表页面
     */
    @GetMapping("/users")
    public ModelAndView userList() {
        ModelAndView mav = new ModelAndView("user/list");
        mav.addObject("users", users);
        mav.addObject("totalCount", users.size());
        return mav;
    }
    
    /**
     * 显示用户详情
     */
    @GetMapping("/users/{id}")
    public String userDetail(@PathVariable Long id, Model model) {
        User user = findUserById(id);
        if (user == null) {
            model.addAttribute("error", "用户不存在");
            return "error";
        }
        model.addAttribute("user", user);
        return "user/detail";
    }
    
    /**
     * 显示创建用户表单
     */
    @GetMapping("/users/new")
    public String newUserForm(Model model) {
        model.addAttribute("user", new UserForm());
        return "user/form";
    }
    
    /**
     * 处理用户创建
     */
    @PostMapping("/users")
    public String createUser(@Valid @ModelAttribute UserForm userForm, 
                           BindingResult bindingResult,
                           RedirectAttributes redirectAttributes) {
        
        // 验证失败，返回表单页面
        if (bindingResult.hasErrors()) {
            return "user/form";
        }
        
        // 创建用户
        User user = new User();
        user.setId(System.currentTimeMillis());
        user.setName(userForm.getName());
        user.setEmail(userForm.getEmail());
        users.add(user);
        
        // 添加成功消息
        redirectAttributes.addFlashAttribute("successMessage", "用户创建成功！");
        
        // 重定向到用户列表
        return "redirect:/web/users";
    }
    
    /**
     * 显示编辑用户表单
     */
    @GetMapping("/users/{id}/edit")
    public String editUserForm(@PathVariable Long id, Model model) {
        User user = findUserById(id);
        if (user == null) {
            model.addAttribute("error", "用户不存在");
            return "error";
        }
        
        UserForm userForm = new UserForm();
        userForm.setName(user.getName());
        userForm.setEmail(user.getEmail());
        
        model.addAttribute("user", userForm);
        model.addAttribute("userId", id);
        return "user/edit";
    }
    
    /**
     * 处理用户更新
     */
    @PostMapping("/users/{id}")
    public String updateUser(@PathVariable Long id,
                           @Valid @ModelAttribute UserForm userForm,
                           BindingResult bindingResult,
                           Model model,
                           RedirectAttributes redirectAttributes) {
        
        if (bindingResult.hasErrors()) {
            model.addAttribute("userId", id);
            return "user/edit";
        }
        
        User user = findUserById(id);
        if (user == null) {
            model.addAttribute("error", "用户不存在");
            return "error";
        }
        
        user.setName(userForm.getName());
        user.setEmail(userForm.getEmail());
        
        redirectAttributes.addFlashAttribute("successMessage", "用户更新成功！");
        return "redirect:/web/users";
    }
    
    /**
     * 删除用户
     */
    @PostMapping("/users/{id}/delete")
    public String deleteUser(@PathVariable Long id, RedirectAttributes redirectAttributes) {
        users.removeIf(user -> user.getId().equals(id));
        redirectAttributes.addFlashAttribute("successMessage", "用户删除成功！");
        return "redirect:/web/users";
    }
    
    /**
     * 搜索用户
     */
    @GetMapping("/users/search")
    public String searchUsers(@RequestParam(required = false) String keyword, Model model) {
        List<User> searchResults = new ArrayList<>();
        
        if (keyword != null && !keyword.trim().isEmpty()) {
            for (User user : users) {
                if (user.getName().contains(keyword) || user.getEmail().contains(keyword)) {
                    searchResults.add(user);
                }
            }
        } else {
            searchResults = users;
        }
        
        model.addAttribute("users", searchResults);
        model.addAttribute("keyword", keyword);
        model.addAttribute("resultCount", searchResults.size());
        return "user/search";
    }
    
    /**
     * AJAX请求处理
     */
    @GetMapping("/users/ajax")
    @ResponseBody
    public List<User> getUsersAjax() {
        return users;
    }
    
    /**
     * 异常处理
     */
    @ExceptionHandler(Exception.class)
    public String handleException(Exception e, Model model) {
        model.addAttribute("error", "发生错误: " + e.getMessage());
        return "error";
    }
    
    // 辅助方法
    private User findUserById(Long id) {
        return users.stream()
                   .filter(user -> user.getId().equals(id))
                   .findFirst()
                   .orElse(null);
    }
}

/**
 * 用户表单类
 * 用于表单数据绑定和验证
 */
class UserForm {
    
    @NotBlank(message = "姓名不能为空")
    @Size(min = 2, max = 50, message = "姓名长度必须在2-50个字符之间")
    private String name;
    
    @NotBlank(message = "邮箱不能为空")
    @Email(message = "邮箱格式不正确")
    private String email;
    
    // Getters and Setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}
```

#### 3.2 拦截器和过滤器

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * Web配置类
 * 注册拦截器和其他Web组件
 */
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 注册日志拦截器
        registry.addInterceptor(new LoggingInterceptor())
                .addPathPatterns("/**")
                .excludePathPatterns("/static/**", "/css/**", "/js/**", "/images/**");
        
        // 注册认证拦截器
        registry.addInterceptor(new AuthInterceptor())
                .addPathPatterns("/admin/**")
                .order(1); // 设置执行顺序
    }
}

/**
 * 日志拦截器
 * 记录请求的基本信息
 */
class LoggingInterceptor implements HandlerInterceptor {
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String method = request.getMethod();
        String uri = request.getRequestURI();
        String queryString = request.getQueryString();
        
        System.out.println(String.format("[请求开始] %s %s%s", 
                                       method, uri, 
                                       queryString != null ? "?" + queryString : ""));
        
        // 记录开始时间
        request.setAttribute("startTime", System.currentTimeMillis());
        
        return true; // 继续执行
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        Long startTime = (Long) request.getAttribute("startTime");
        long endTime = System.currentTimeMillis();
        long duration = endTime - startTime;
        
        String method = request.getMethod();
        String uri = request.getRequestURI();
        int status = response.getStatus();
        
        System.out.println(String.format("[请求完成] %s %s - 状态码: %d, 耗时: %dms", 
                                       method, uri, status, duration));
        
        if (ex != null) {
            System.out.println("[请求异常] " + ex.getMessage());
        }
    }
}

/**
 * 认证拦截器
 * 检查用户是否已登录
 */
class AuthInterceptor implements HandlerInterceptor {
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 检查session中是否有用户信息
        Object user = request.getSession().getAttribute("user");
        
        if (user == null) {
            System.out.println("[认证失败] 用户未登录，重定向到登录页面");
            response.sendRedirect("/login");
            return false; // 阻止继续执行
        }
        
        System.out.println("[认证成功] 用户已登录: " + user);
        return true;
    }
}

/**
 * 字符编码过滤器
 * 确保请求和响应使用UTF-8编码
 */
@Component
class CharacterEncodingFilter implements Filter {
    
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("字符编码过滤器初始化");
    }
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
            throws IOException, ServletException {
        
        // 设置请求编码
        request.setCharacterEncoding("UTF-8");
        
        // 设置响应编码
        response.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");
        
        // 继续执行过滤器链
        chain.doFilter(request, response);
    }
    
    @Override
    public void destroy() {
        System.out.println("字符编码过滤器销毁");
    }
}

/**
 * CORS过滤器
 * 处理跨域请求
 */
@Component
class CorsFilter implements Filter {
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
            throws IOException, ServletException {
        
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        
        // 设置CORS头
        httpResponse.setHeader("Access-Control-Allow-Origin", "*");
        httpResponse.setHeader("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS");
        httpResponse.setHeader("Access-Control-Allow-Headers", "Content-Type, Authorization");
        httpResponse.setHeader("Access-Control-Max-Age", "3600");
        
        // 处理预检请求
        if ("OPTIONS".equalsIgnoreCase(httpRequest.getMethod())) {
            httpResponse.setStatus(HttpServletResponse.SC_OK);
            return;
        }
        
        chain.doFilter(request, response);
    }
}
```

### 4. 数据访问层

#### 4.1 Spring Data JPA

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.*;
import java.time.LocalDateTime;
import java.util.List;
import java.util.Optional;

/**
 * Spring Data JPA演示
 * 展示JPA实体、Repository和服务层的使用
 */

/**
 * 用户实体类
 * 使用JPA注解进行ORM映射
 */
@Entity
@Table(name = "users")
class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "username", nullable = false, unique = true, length = 50)
    private String username;
    
    @Column(name = "email", nullable = false, unique = true, length = 100)
    private String email;
    
    @Column(name = "password", nullable = false, length = 255)
    private String password;
    
    @Column(name = "full_name", length = 100)
    private String fullName;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "status")
    private UserStatus status = UserStatus.ACTIVE;
    
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // 一对多关系：一个用户可以有多个订单
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Order> orders;
    
    // 多对多关系：用户和角色
    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private List<Role> roles;
    
    // JPA生命周期回调
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
    
    // 构造函数
    public User() {}
    
    public User(String username, String email, String password) {
        this.username = username;
        this.email = email;
        this.password = password;
    }
    
    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    public String getPassword() { return password; }
    public void setPassword(String password) { this.password = password; }
    public String getFullName() { return fullName; }
    public void setFullName(String fullName) { this.fullName = fullName; }
    public UserStatus getStatus() { return status; }
    public void setStatus(UserStatus status) { this.status = status; }
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public void setUpdatedAt(LocalDateTime updatedAt) { this.updatedAt = updatedAt; }
    public List<Order> getOrders() { return orders; }
    public void setOrders(List<Order> orders) { this.orders = orders; }
    public List<Role> getRoles() { return roles; }
    public void setRoles(List<Role> roles) { this.roles = roles; }
}

/**
 * 用户状态枚举
 */
enum UserStatus {
    ACTIVE, INACTIVE, SUSPENDED
}

/**
 * 订单实体类
 */
@Entity
@Table(name = "orders")
class Order {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "order_number", nullable = false, unique = true)
    private String orderNumber;
    
    @Column(name = "total_amount", nullable = false)
    private Double totalAmount;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "status")
    private OrderStatus status = OrderStatus.PENDING;
    
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    // 多对一关系：多个订单属于一个用户
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
    }
    
    // 构造函数和Getters/Setters
    public Order() {}
    
    public Order(String orderNumber, Double totalAmount, User user) {
        this.orderNumber = orderNumber;
        this.totalAmount = totalAmount;
        this.user = user;
    }
    
    // Getters and Setters省略...
}

/**
 * 订单状态枚举
 */
enum OrderStatus {
    PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED
}

/**
 * 角色实体类
 */
@Entity
@Table(name = "roles")
class Role {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "name", nullable = false, unique = true)
    private String name;
    
    @Column(name = "description")
    private String description;
    
    // 构造函数和Getters/Setters
    public Role() {}
    
    public Role(String name, String description) {
        this.name = name;
        this.description = description;
    }
    
    // Getters and Setters省略...
}

/**
 * 用户Repository接口
 * 继承JpaRepository获得基本的CRUD操作
 */
@Repository
interface UserRepository extends JpaRepository<User, Long> {
    
    // 方法名查询：Spring Data JPA会根据方法名自动生成查询
    Optional<User> findByUsername(String username);
    Optional<User> findByEmail(String email);
    List<User> findByStatus(UserStatus status);
    List<User> findByFullNameContaining(String keyword);
    
    // 使用@Query注解自定义查询
    @Query("SELECT u FROM User u WHERE u.createdAt >= :startDate")
    List<User> findUsersCreatedAfter(@Param("startDate") LocalDateTime startDate);
    
    // 原生SQL查询
    @Query(value = "SELECT * FROM users WHERE email LIKE %:domain%", nativeQuery = true)
    List<User> findUsersByEmailDomain(@Param("domain") String domain);
    
    // 分页查询
    Page<User> findByStatusOrderByCreatedAtDesc(UserStatus status, Pageable pageable);
    
    // 统计查询
    long countByStatus(UserStatus status);
    
    // 存在性查询
    boolean existsByUsername(String username);
    boolean existsByEmail(String email);
    
    // 删除查询
    void deleteByStatus(UserStatus status);
    
    // 复杂查询：查找有订单的用户
    @Query("SELECT DISTINCT u FROM User u JOIN u.orders o WHERE o.status = :orderStatus")
    List<User> findUsersWithOrderStatus(@Param("orderStatus") OrderStatus orderStatus);
}

/**
 * 订单Repository接口
 */
@Repository
interface OrderRepository extends JpaRepository<Order, Long> {
    
    List<Order> findByUser(User user);
    List<Order> findByUserIdOrderByCreatedAtDesc(Long userId);
    List<Order> findByStatus(OrderStatus status);
    
    @Query("SELECT o FROM Order o WHERE o.totalAmount >= :minAmount")
    List<Order> findHighValueOrders(@Param("minAmount") Double minAmount);
    
    // 聚合查询
    @Query("SELECT SUM(o.totalAmount) FROM Order o WHERE o.user.id = :userId")
    Double getTotalAmountByUserId(@Param("userId") Long userId);
}

/**
 * 用户服务类
 * 演示事务管理和业务逻辑
 */
@Service
@Transactional
class UserService {
    
    private final UserRepository userRepository;
    private final OrderRepository orderRepository;
    
    public UserService(UserRepository userRepository, OrderRepository orderRepository) {
        this.userRepository = userRepository;
        this.orderRepository = orderRepository;
    }
    
    /**
     * 创建用户
     */
    public User createUser(String username, String email, String password, String fullName) {
        // 检查用户名和邮箱是否已存在
        if (userRepository.existsByUsername(username)) {
            throw new IllegalArgumentException("用户名已存在: " + username);
        }
        if (userRepository.existsByEmail(email)) {
            throw new IllegalArgumentException("邮箱已存在: " + email);
        }
        
        User user = new User(username, email, password);
        user.setFullName(fullName);
        
        return userRepository.save(user);
    }
    
    /**
     * 根据用户名查找用户
     */
    @Transactional(readOnly = true)
    public Optional<User> findByUsername(String username) {
        return userRepository.findByUsername(username);
    }
    
    /**
     * 获取活跃用户列表
     */
    @Transactional(readOnly = true)
    public List<User> getActiveUsers() {
        return userRepository.findByStatus(UserStatus.ACTIVE);
    }
    
    /**
     * 分页获取用户
     */
    @Transactional(readOnly = true)
    public Page<User> getUsers(Pageable pageable) {
        return userRepository.findAll(pageable);
    }
    
    /**
     * 更新用户状态
     */
    public void updateUserStatus(Long userId, UserStatus status) {
        User user = userRepository.findById(userId)
                .orElseThrow(() -> new IllegalArgumentException("用户不存在: " + userId));
        
        user.setStatus(status);
        userRepository.save(user);
    }
    
    /**
     * 为用户创建订单
     */
    public Order createOrderForUser(Long userId, String orderNumber, Double totalAmount) {
        User user = userRepository.findById(userId)
                .orElseThrow(() -> new IllegalArgumentException("用户不存在: " + userId));
        
        Order order = new Order(orderNumber, totalAmount, user);
        return orderRepository.save(order);
    }
    
    /**
     * 获取用户的订单历史
     */
    @Transactional(readOnly = true)
    public List<Order> getUserOrders(Long userId) {
        return orderRepository.findByUserIdOrderByCreatedAtDesc(userId);
    }
    
    /**
     * 获取用户的总消费金额
     */
    @Transactional(readOnly = true)
    public Double getUserTotalAmount(Long userId) {
        Double total = orderRepository.getTotalAmountByUserId(userId);
        return total != null ? total : 0.0;
    }
    
    /**
     * 搜索用户
     */
    @Transactional(readOnly = true)
    public List<User> searchUsers(String keyword) {
        return userRepository.findByFullNameContaining(keyword);
    }
    
    /**
     * 批量删除非活跃用户
     */
    public void deleteInactiveUsers() {
        userRepository.deleteByStatus(UserStatus.INACTIVE);
    }
}
```

#### 4.2 MyBatis持久层框架

```java
import org.apache.ibatis.annotations.*;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.util.List;
import java.util.Map;

/**
 * MyBatis配置类
 */
@Configuration
@MapperScan("com.example.mapper")
public class MyBatisConfig {
    // MyBatis配置
}

/**
 * 产品实体类
 */
class Product {
    private Long id;
    private String name;
    private String description;
    private Double price;
    private Integer stock;
    private String category;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    
    // 构造函数
    public Product() {}
    
    public Product(String name, String description, Double price, Integer stock, String category) {
        this.name = name;
        this.description = description;
        this.price = price;
        this.stock = stock;
        this.category = category;
    }
    
    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
    public Double getPrice() { return price; }
    public void setPrice(Double price) { this.price = price; }
    public Integer getStock() { return stock; }
    public void setStock(Integer stock) { this.stock = stock; }
    public String getCategory() { return category; }
    public void setCategory(String category) { this.category = category; }
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public void setUpdatedAt(LocalDateTime updatedAt) { this.updatedAt = updatedAt; }
    
    @Override
    public String toString() {
        return String.format("Product{id=%d, name='%s', price=%.2f, stock=%d}", 
                           id, name, price, stock);
    }
}

/**
 * 产品Mapper接口
 * 使用MyBatis注解进行SQL映射
 */
@Mapper
interface ProductMapper {
    
    /**
     * 插入产品
     */
    @Insert("INSERT INTO products(name, description, price, stock, category, created_at, updated_at) " +
            "VALUES(#{name}, #{description}, #{price}, #{stock}, #{category}, NOW(), NOW())")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    int insert(Product product);
    
    /**
     * 根据ID查询产品
     */
    @Select("SELECT * FROM products WHERE id = #{id}")
    @Results({
        @Result(property = "id", column = "id"),
        @Result(property = "name", column = "name"),
        @Result(property = "description", column = "description"),
        @Result(property = "price", column = "price"),
        @Result(property = "stock", column = "stock"),
        @Result(property = "category", column = "category"),
        @Result(property = "createdAt", column = "created_at"),
        @Result(property = "updatedAt", column = "updated_at")
    })
    Product findById(@Param("id") Long id);
    
    /**
     * 查询所有产品
     */
    @Select("SELECT * FROM products ORDER BY created_at DESC")
    List<Product> findAll();
    
    /**
     * 根据分类查询产品
     */
    @Select("SELECT * FROM products WHERE category = #{category}")
    List<Product> findByCategory(@Param("category") String category);
    
    /**
     * 根据价格范围查询产品
     */
    @Select("SELECT * FROM products WHERE price BETWEEN #{minPrice} AND #{maxPrice}")
    List<Product> findByPriceRange(@Param("minPrice") Double minPrice, 
                                  @Param("maxPrice") Double maxPrice);
    
    /**
     * 更新产品
     */
    @Update("UPDATE products SET name=#{name}, description=#{description}, " +
            "price=#{price}, stock=#{stock}, category=#{category}, updated_at=NOW() " +
            "WHERE id=#{id}")
    int update(Product product);
    
    /**
     * 更新库存
     */
    @Update("UPDATE products SET stock = stock + #{quantity}, updated_at=NOW() WHERE id=#{id}")
    int updateStock(@Param("id") Long id, @Param("quantity") Integer quantity);
    
    /**
     * 删除产品
     */
    @Delete("DELETE FROM products WHERE id = #{id}")
    int deleteById(@Param("id") Long id);
    
    /**
     * 搜索产品（动态SQL）
     */
    @SelectProvider(type = ProductSqlProvider.class, method = "searchProducts")
    List<Product> searchProducts(@Param("keyword") String keyword, 
                               @Param("category") String category,
                               @Param("minPrice") Double minPrice,
                               @Param("maxPrice") Double maxPrice);
    
    /**
     * 获取产品统计信息
     */
    @Select("SELECT category, COUNT(*) as count, AVG(price) as avgPrice, SUM(stock) as totalStock " +
            "FROM products GROUP BY category")
    @Results({
        @Result(property = "category", column = "category"),
        @Result(property = "count", column = "count"),
        @Result(property = "avgPrice", column = "avgPrice"),
        @Result(property = "totalStock", column = "totalStock")
    })
    List<Map<String, Object>> getProductStatistics();
}

/**
 * 动态SQL提供者
 */
class ProductSqlProvider {
    
    public String searchProducts(Map<String, Object> params) {
        StringBuilder sql = new StringBuilder("SELECT * FROM products WHERE 1=1");
        
        if (params.get("keyword") != null) {
            sql.append(" AND (name LIKE CONCAT('%', #{keyword}, '%') OR description LIKE CONCAT('%', #{keyword}, '%'))");
        }
        
        if (params.get("category") != null) {
            sql.append(" AND category = #{category}");
        }
        
        if (params.get("minPrice") != null) {
            sql.append(" AND price >= #{minPrice}");
        }
        
        if (params.get("maxPrice") != null) {
            sql.append(" AND price <= #{maxPrice}");
        }
        
        sql.append(" ORDER BY created_at DESC");
        
        return sql.toString();
    }
}

/**
 * 产品服务类
 */
@Service
class ProductService {
    
    private final ProductMapper productMapper;
    
    public ProductService(ProductMapper productMapper) {
        this.productMapper = productMapper;
    }
    
    /**
     * 创建产品
     */
    public Product createProduct(String name, String description, Double price, 
                               Integer stock, String category) {
        Product product = new Product(name, description, price, stock, category);
        productMapper.insert(product);
        return product;
    }
    
    /**
     * 获取产品详情
     */
    public Product getProduct(Long id) {
        Product product = productMapper.findById(id);
        if (product == null) {
            throw new IllegalArgumentException("产品不存在: " + id);
        }
        return product;
    }
    
    /**
     * 获取所有产品
     */
    public List<Product> getAllProducts() {
        return productMapper.findAll();
    }
    
    /**
     * 根据分类获取产品
     */
    public List<Product> getProductsByCategory(String category) {
        return productMapper.findByCategory(category);
    }
    
    /**
     * 搜索产品
     */
    public List<Product> searchProducts(String keyword, String category, 
                                      Double minPrice, Double maxPrice) {
        return productMapper.searchProducts(keyword, category, minPrice, maxPrice);
    }
    
    /**
     * 更新产品
     */
    public Product updateProduct(Long id, String name, String description, 
                               Double price, Integer stock, String category) {
        Product product = getProduct(id);
        product.setName(name);
        product.setDescription(description);
        product.setPrice(price);
        product.setStock(stock);
        product.setCategory(category);
        
        productMapper.update(product);
        return product;
    }
    
    /**
     * 更新库存
     */
    public void updateStock(Long id, Integer quantity) {
        int affected = productMapper.updateStock(id, quantity);
        if (affected == 0) {
            throw new IllegalArgumentException("产品不存在或库存更新失败: " + id);
        }
    }
    
    /**
     * 删除产品
     */
    public void deleteProduct(Long id) {
        int affected = productMapper.deleteById(id);
        if (affected == 0) {
            throw new IllegalArgumentException("产品不存在: " + id);
        }
    }
    
    /**
     * 获取产品统计信息
     */
    public List<Map<String, Object>> getProductStatistics() {
        return productMapper.getProductStatistics();
    }
}
```

**MyBatis XML配置示例**

```xml
<!-- ProductMapper.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.mapper.ProductMapper">
    
    <!-- 结果映射 -->
    <resultMap id="ProductResultMap" type="com.example.entity.Product">
        <id property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="description" column="description"/>
        <result property="price" column="price"/>
        <result property="stock" column="stock"/>
        <result property="category" column="category"/>
        <result property="createdAt" column="created_at"/>
        <result property="updatedAt" column="updated_at"/>
    </resultMap>
    
    <!-- 动态查询 -->
    <select id="searchProductsXml" resultMap="ProductResultMap">
        SELECT * FROM products
        <where>
            <if test="keyword != null and keyword != ''">
                AND (name LIKE CONCAT('%', #{keyword}, '%') 
                     OR description LIKE CONCAT('%', #{keyword}, '%'))
            </if>
            <if test="category != null and category != ''">
                AND category = #{category}
            </if>
            <if test="minPrice != null">
                AND price >= #{minPrice}
            </if>
            <if test="maxPrice != null">
                AND price <= #{maxPrice}
            </if>
        </where>
        ORDER BY created_at DESC
    </select>
    
    <!-- 批量插入 -->
    <insert id="batchInsert" parameterType="list">
        INSERT INTO products(name, description, price, stock, category, created_at, updated_at)
        VALUES
        <foreach collection="list" item="product" separator=",">
            (#{product.name}, #{product.description}, #{product.price}, 
             #{product.stock}, #{product.category}, NOW(), NOW())
        </foreach>
    </insert>
    
    <!-- 复杂查询：关联查询 -->
    <select id="findProductsWithOrders" resultMap="ProductWithOrdersResultMap">
        SELECT p.*, o.id as order_id, o.order_number, o.quantity as order_quantity
        FROM products p
        LEFT JOIN order_items oi ON p.id = oi.product_id
        LEFT JOIN orders o ON oi.order_id = o.id
        WHERE p.id = #{productId}
    </select>
    
    <resultMap id="ProductWithOrdersResultMap" type="com.example.entity.Product">
        <id property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="price" column="price"/>
        <collection property="orders" ofType="com.example.entity.Order">
            <id property="id" column="order_id"/>
            <result property="orderNumber" column="order_number"/>
            <result property="quantity" column="order_quantity"/>
        </collection>
    </resultMap>
    
</mapper>
```

### 5. 消息队列

#### 5.1 RabbitMQ消息队列

```java
import org.springframework.amqp.core.*;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.Map;

/**
 * RabbitMQ配置类
 * 定义队列、交换机和绑定关系
 */
@Configuration
public class RabbitMQConfig {
    
    // 队列名称常量
    public static final String USER_QUEUE = "user.queue";
    public static final String ORDER_QUEUE = "order.queue";
    public static final String EMAIL_QUEUE = "email.queue";
    public static final String DEAD_LETTER_QUEUE = "dead.letter.queue";
    
    // 交换机名称常量
    public static final String USER_EXCHANGE = "user.exchange";
    public static final String ORDER_EXCHANGE = "order.exchange";
    public static final String TOPIC_EXCHANGE = "topic.exchange";
    
    /**
     * 声明用户队列
     */
    @Bean
    public Queue userQueue() {
        return QueueBuilder.durable(USER_QUEUE)
                .withArgument("x-dead-letter-exchange", "dlx.exchange")
                .withArgument("x-dead-letter-routing-key", "dead.letter")
                .build();
    }
    
    /**
     * 声明订单队列
     */
    @Bean
    public Queue orderQueue() {
        return QueueBuilder.durable(ORDER_QUEUE).build();
    }
    
    /**
     * 声明邮件队列
     */
    @Bean
    public Queue emailQueue() {
        return QueueBuilder.durable(EMAIL_QUEUE).build();
    }
    
    /**
     * 声明死信队列
     */
    @Bean
    public Queue deadLetterQueue() {
        return QueueBuilder.durable(DEAD_LETTER_QUEUE).build();
    }
    
    /**
     * 声明直连交换机
     */
    @Bean
    public DirectExchange userExchange() {
        return new DirectExchange(USER_EXCHANGE);
    }
    
    /**
     * 声明主题交换机
     */
    @Bean
    public TopicExchange topicExchange() {
        return new TopicExchange(TOPIC_EXCHANGE);
    }
    
    /**
     * 声明死信交换机
     */
    @Bean
    public DirectExchange deadLetterExchange() {
        return new DirectExchange("dlx.exchange");
    }
    
    /**
     * 绑定用户队列到用户交换机
     */
    @Bean
    public Binding userBinding() {
        return BindingBuilder.bind(userQueue())
                .to(userExchange())
                .with("user.created");
    }
    
    /**
     * 绑定邮件队列到主题交换机
     */
    @Bean
    public Binding emailBinding() {
        return BindingBuilder.bind(emailQueue())
                .to(topicExchange())
                .with("email.*");
    }
    
    /**
     * 绑定死信队列
     */
    @Bean
    public Binding deadLetterBinding() {
        return BindingBuilder.bind(deadLetterQueue())
                .to(deadLetterExchange())
                .with("dead.letter");
    }
}

/**
 * 消息实体类
 */
class UserMessage {
    private Long userId;
    private String username;
    private String email;
    private String action;
    private LocalDateTime timestamp;
    
    public UserMessage() {
        this.timestamp = LocalDateTime.now();
    }
    
    public UserMessage(Long userId, String username, String email, String action) {
        this();
        this.userId = userId;
        this.username = username;
        this.email = email;
        this.action = action;
    }
    
    // Getters and Setters
    public Long getUserId() { return userId; }
    public void setUserId(Long userId) { this.userId = userId; }
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    public String getAction() { return action; }
    public void setAction(String action) { this.action = action; }
    public LocalDateTime getTimestamp() { return timestamp; }
    public void setTimestamp(LocalDateTime timestamp) { this.timestamp = timestamp; }
    
    @Override
    public String toString() {
        return String.format("UserMessage{userId=%d, username='%s', action='%s', timestamp=%s}", 
                           userId, username, action, timestamp);
    }
}

/**
 * 邮件消息实体类
 */
class EmailMessage {
    private String to;
    private String subject;
    private String content;
    private String type;
    private LocalDateTime timestamp;
    
    public EmailMessage() {
        this.timestamp = LocalDateTime.now();
    }
    
    public EmailMessage(String to, String subject, String content, String type) {
        this();
        this.to = to;
        this.subject = subject;
        this.content = content;
        this.type = type;
    }
    
    // Getters and Setters省略...
    
    @Override
    public String toString() {
        return String.format("EmailMessage{to='%s', subject='%s', type='%s'}", to, subject, type);
    }
}

/**
 * 消息发送服务
 */
@Service
public class MessageProducer {
    
    private final RabbitTemplate rabbitTemplate;
    
    public MessageProducer(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }
    
    /**
     * 发送用户创建消息
     */
    public void sendUserCreatedMessage(Long userId, String username, String email) {
        UserMessage message = new UserMessage(userId, username, email, "CREATED");
        
        System.out.println("发送用户创建消息: " + message);
        rabbitTemplate.convertAndSend(
            RabbitMQConfig.USER_EXCHANGE, 
            "user.created", 
            message
        );
    }
    
    /**
     * 发送邮件消息
     */
    public void sendEmailMessage(String to, String subject, String content, String type) {
        EmailMessage message = new EmailMessage(to, subject, content, type);
        
        System.out.println("发送邮件消息: " + message);
        rabbitTemplate.convertAndSend(
            RabbitMQConfig.TOPIC_EXCHANGE,
            "email." + type.toLowerCase(),
            message
        );
    }
    
    /**
     * 发送延迟消息（使用TTL）
     */
    public void sendDelayedMessage(String message, long delayMillis) {
        Map<String, Object> headers = new HashMap<>();
        headers.put("x-delay", delayMillis);
        
        MessageProperties properties = new MessageProperties();
        properties.setExpiration(String.valueOf(delayMillis));
        
        Message rabbitMessage = new Message(message.getBytes(), properties);
        
        System.out.println("发送延迟消息: " + message + ", 延迟: " + delayMillis + "ms");
        rabbitTemplate.send("delayed.exchange", "delayed.key", rabbitMessage);
    }
}

/**
 * 消息消费者
 */
@Component
public class MessageConsumer {
    
    /**
     * 处理用户创建消息
     */
    @RabbitListener(queues = RabbitMQConfig.USER_QUEUE)
    public void handleUserCreated(UserMessage message) {
        System.out.println("\n=== 处理用户创建消息 ===");
        System.out.println("接收到消息: " + message);
        
        try {
            // 模拟业务处理
            Thread.sleep(1000);
            
            // 处理用户创建后的业务逻辑
            System.out.println("为用户 " + message.getUsername() + " 初始化默认设置");
            System.out.println("发送欢迎邮件到: " + message.getEmail());
            
            // 可以在这里发送其他消息
            // messageProducer.sendEmailMessage(message.getEmail(), "欢迎注册", "欢迎您！", "WELCOME");
            
            System.out.println("用户创建消息处理完成");
            
        } catch (Exception e) {
            System.err.println("处理用户创建消息失败: " + e.getMessage());
            throw new RuntimeException("消息处理失败", e);
        }
    }
    
    /**
     * 处理邮件消息
     */
    @RabbitListener(queues = RabbitMQConfig.EMAIL_QUEUE)
    public void handleEmailMessage(EmailMessage message) {
        System.out.println("\n=== 处理邮件消息 ===");
        System.out.println("接收到邮件消息: " + message);
        
        try {
            // 模拟邮件发送
            Thread.sleep(500);
            
            System.out.println("正在发送邮件...");
            System.out.println("收件人: " + message.getTo());
            System.out.println("主题: " + message.getSubject());
            System.out.println("类型: " + message.getType());
            
            // 模拟邮件发送成功
            System.out.println("邮件发送成功！");
            
        } catch (Exception e) {
            System.err.println("邮件发送失败: " + e.getMessage());
            throw new RuntimeException("邮件发送失败", e);
        }
    }
    
    /**
     * 处理死信消息
     */
    @RabbitListener(queues = RabbitMQConfig.DEAD_LETTER_QUEUE)
    public void handleDeadLetterMessage(Message message) {
        System.out.println("\n=== 处理死信消息 ===");
        System.out.println("接收到死信消息: " + new String(message.getBody()));
        
        // 记录死信消息，进行人工处理或重试
        System.out.println("记录死信消息到数据库，等待人工处理");
    }
}

/**
 * 消息队列演示服务
 */
@Service
public class MessageQueueDemoService {
    
    private final MessageProducer messageProducer;
    
    public MessageQueueDemoService(MessageProducer messageProducer) {
        this.messageProducer = messageProducer;
    }
    
    /**
     * 演示消息队列功能
     */
    public void demonstrateMessageQueue() {
        System.out.println("\n=== RabbitMQ消息队列演示 ===");
        
        // 发送用户创建消息
        messageProducer.sendUserCreatedMessage(1L, "张三", "zhangsan@example.com");
        
        // 发送不同类型的邮件消息
        messageProducer.sendEmailMessage("user@example.com", "欢迎注册", "欢迎您注册我们的服务！", "WELCOME");
        messageProducer.sendEmailMessage("user@example.com", "密码重置", "请点击链接重置密码", "PASSWORD_RESET");
        messageProducer.sendEmailMessage("user@example.com", "订单确认", "您的订单已确认", "ORDER_CONFIRMATION");
        
        System.out.println("消息已发送，等待消费者处理...");
    }
}
```

## 经典面试题

### Spring框架相关

**1. Spring IoC容器的工作原理是什么？**

**答案：**

Spring IoC（Inversion of Control，控制反转）容器是Spring框架的核心，它负责管理对象的创建、配置和生命周期。

**IoC容器工作原理详解：**

```java
// 1. 传统方式 vs IoC方式对比

// 传统方式：对象自己创建依赖
class TraditionalUserService {
    private UserRepository userRepository;
    
    public TraditionalUserService() {
        // 紧耦合：直接创建依赖对象
        this.userRepository = new UserRepositoryImpl();
    }
    
    public User findUser(Long id) {
        return userRepository.findById(id);
    }
}

// IoC方式：容器注入依赖
@Service
class IoCUserService {
    private final UserRepository userRepository;
    
    // 构造器注入：容器负责提供依赖
    public IoCUserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public User findUser(Long id) {
        return userRepository.findById(id);
    }
}

@Repository
class UserRepositoryImpl implements UserRepository {
    @Override
    public User findById(Long id) {
        // 数据库查询逻辑
        return new User(id, "用户" + id, "user" + id + "@example.com");
    }
}
```

**IoC容器的核心组件：**

```java
// 2. BeanFactory - 容器的基础接口
public class BeanFactoryDemo {
    public static void main(String[] args) {
        // 创建Bean工厂
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        
        // 手动注册Bean定义
        BeanDefinition beanDefinition = BeanDefinitionBuilder
            .genericBeanDefinition(UserService.class)
            .addConstructorArgReference("userRepository")
            .getBeanDefinition();
        
        beanFactory.registerBeanDefinition("userService", beanDefinition);
        
        // 注册依赖Bean
        BeanDefinition repoBeanDefinition = BeanDefinitionBuilder
            .genericBeanDefinition(UserRepositoryImpl.class)
            .getBeanDefinition();
        
        beanFactory.registerBeanDefinition("userRepository", repoBeanDefinition);
        
        // 获取Bean实例
        UserService userService = beanFactory.getBean("userService", UserService.class);
        User user = userService.findUser(1L);
        System.out.println("获取用户：" + user);
    }
}

// 3. ApplicationContext - 高级容器
@Configuration
@ComponentScan(basePackages = "com.example")
class AppConfig {
    
    @Bean
    @Primary
    public DataSource primaryDataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl("jdbc:h2:mem:primary");
        dataSource.setUsername("sa");
        dataSource.setPassword("");
        return dataSource;
    }
    
    @Bean("backupDataSource")
    public DataSource backupDataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl("jdbc:h2:mem:backup");
        dataSource.setUsername("sa");
        dataSource.setPassword("");
        return dataSource;
    }
}

public class ApplicationContextDemo {
    public static void main(String[] args) {
        // 创建应用上下文
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        
        // 演示不同的Bean获取方式
        demonstrateBeanRetrieval(context);
        
        // 演示Bean的作用域
        demonstrateBeanScopes(context);
        
        // 演示依赖注入方式
        demonstrateDependencyInjection(context);
    }
    
    private static void demonstrateBeanRetrieval(ApplicationContext context) {
        System.out.println("=== Bean获取方式演示 ===");
        
        // 1. 按类型获取
        UserService userService1 = context.getBean(UserService.class);
        System.out.println("按类型获取：" + userService1.getClass().getSimpleName());
        
        // 2. 按名称获取
        Object userService2 = context.getBean("userService");
        System.out.println("按名称获取：" + userService2.getClass().getSimpleName());
        
        // 3. 按名称和类型获取
        UserService userService3 = context.getBean("userService", UserService.class);
        System.out.println("按名称和类型获取：" + userService3.getClass().getSimpleName());
        
        // 4. 获取所有同类型Bean
        Map<String, DataSource> dataSources = context.getBeansOfType(DataSource.class);
        System.out.println("所有DataSource Bean：" + dataSources.keySet());
    }
    
    private static void demonstrateBeanScopes(ApplicationContext context) {
        System.out.println("\n=== Bean作用域演示 ===");
        
        // Singleton作用域（默认）
        UserService singleton1 = context.getBean(UserService.class);
        UserService singleton2 = context.getBean(UserService.class);
        System.out.println("Singleton Bean相同实例：" + (singleton1 == singleton2));
        System.out.println("Singleton Bean hashCode：" + singleton1.hashCode() + ", " + singleton2.hashCode());
    }
}
```

**依赖注入的三种方式：**

```java
// 4. 依赖注入方式详解

// 构造器注入（推荐）
@Service
class ConstructorInjectionService {
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    // 构造器注入：保证依赖不可变，支持final字段
    public ConstructorInjectionService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
    
    public void processUser(Long userId) {
        User user = userRepository.findById(userId);
        emailService.sendWelcomeEmail(user.getEmail());
    }
}

// Setter注入
@Service
class SetterInjectionService {
    private UserRepository userRepository;
    private EmailService emailService;
    
    // Setter注入：支持可选依赖
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Autowired(required = false) // 可选依赖
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
    
    public void processUser(Long userId) {
        User user = userRepository.findById(userId);
        if (emailService != null) {
            emailService.sendWelcomeEmail(user.getEmail());
        }
    }
}

// 字段注入（不推荐）
@Service
class FieldInjectionService {
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    @Qualifier("smtpEmailService")
    private EmailService emailService;
    
    public void processUser(Long userId) {
        User user = userRepository.findById(userId);
        emailService.sendWelcomeEmail(user.getEmail());
    }
}

// 方法注入
@Service
class MethodInjectionService {
    private UserRepository userRepository;
    private EmailService emailService;
    private NotificationService notificationService;
    
    // 多参数方法注入
    @Autowired
    public void configureServices(UserRepository userRepository, 
                                 EmailService emailService,
                                 @Qualifier("smsNotification") NotificationService notificationService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
        this.notificationService = notificationService;
    }
}
```

**IoC容器启动流程：**

```java
// 5. 容器启动流程演示
public class ContainerStartupDemo {
    
    public static void main(String[] args) {
        System.out.println("=== Spring容器启动流程 ===");
        
        // 1. 创建容器
        System.out.println("1. 创建ApplicationContext");
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        
        // 2. 注册配置类
        System.out.println("2. 注册配置类");
        context.register(AppConfig.class);
        
        // 3. 刷新容器（核心步骤）
        System.out.println("3. 刷新容器 - 开始");
        context.refresh();
        System.out.println("3. 刷新容器 - 完成");
        
        // 4. 使用Bean
        System.out.println("4. 使用Bean");
        UserService userService = context.getBean(UserService.class);
        userService.findUser(1L);
        
        // 5. 关闭容器
        System.out.println("5. 关闭容器");
        context.close();
    }
}

// 自定义BeanPostProcessor演示容器处理过程
@Component
class CustomBeanPostProcessor implements BeanPostProcessor {
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (bean instanceof UserService) {
            System.out.println("BeanPostProcessor: 初始化前处理 - " + beanName);
        }
        return bean;
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (bean instanceof UserService) {
            System.out.println("BeanPostProcessor: 初始化后处理 - " + beanName);
        }
        return bean;
    }
}
```

**IoC容器的优势：**

| 特性 | 传统方式 | IoC容器 |
|------|----------|----------|
| **耦合度** | 高耦合，直接依赖具体实现 | 低耦合，依赖抽象接口 |
| **可测试性** | 难以进行单元测试 | 易于Mock依赖进行测试 |
| **可配置性** | 硬编码，难以更换实现 | 配置驱动，易于切换实现 |
| **生命周期管理** | 手动管理对象生命周期 | 容器自动管理 |
| **AOP支持** | 需要手动实现 | 容器提供AOP支持 |

**2. Spring Bean的生命周期？**

**答案：**

Spring Bean的生命周期是指从Bean的创建到销毁的整个过程，Spring容器负责管理这个完整的生命周期。

**Bean生命周期详细流程：**

```java
// 1. Bean生命周期演示类
@Component
public class LifecycleBean implements BeanNameAware, BeanFactoryAware, 
                                     ApplicationContextAware, InitializingBean, DisposableBean {
    
    private String beanName;
    private BeanFactory beanFactory;
    private ApplicationContext applicationContext;
    
    // 1. 构造器
    public LifecycleBean() {
        System.out.println("1. 构造器：创建Bean实例");
    }
    
    // 2. 属性注入
    @Autowired
    private UserRepository userRepository;
    
    @Value("${app.name:默认应用}")
    private String appName;
    
    // 3. Aware接口回调
    @Override
    public void setBeanName(String name) {
        this.beanName = name;
        System.out.println("3. BeanNameAware.setBeanName(): " + name);
    }
    
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
        System.out.println("3. BeanFactoryAware.setBeanFactory()");
    }
    
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
        System.out.println("3. ApplicationContextAware.setApplicationContext()");
    }
    
    // 4. 初始化前处理（通过BeanPostProcessor）
    // 见CustomBeanPostProcessor.postProcessBeforeInitialization()
    
    // 5. 初始化方法
    @PostConstruct
    public void postConstruct() {
        System.out.println("5. @PostConstruct：初始化方法1");
    }
    
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("5. InitializingBean.afterPropertiesSet()：初始化方法2");
    }
    
    @Bean(initMethod = "customInit")
    public void customInit() {
        System.out.println("5. 自定义初始化方法：customInit()");
    }
    
    // 6. 初始化后处理（通过BeanPostProcessor）
    // 见CustomBeanPostProcessor.postProcessAfterInitialization()
    
    // 7. 业务方法
    public void doSomething() {
        System.out.println("7. 业务方法执行：Bean正在被使用");
        System.out.println("   - Bean名称：" + beanName);
        System.out.println("   - 应用名称：" + appName);
        System.out.println("   - 容器中Bean数量：" + applicationContext.getBeanDefinitionCount());
    }
    
    // 8. 销毁方法
    @PreDestroy
    public void preDestroy() {
        System.out.println("8. @PreDestroy：销毁方法1");
    }
    
    @Override
    public void destroy() throws Exception {
        System.out.println("8. DisposableBean.destroy()：销毁方法2");
    }
    
    public void customDestroy() {
        System.out.println("8. 自定义销毁方法：customDestroy()");
    }
}

// 2. 自定义BeanPostProcessor
@Component
class LifecycleBeanPostProcessor implements BeanPostProcessor {
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (bean instanceof LifecycleBean) {
            System.out.println("4. BeanPostProcessor.postProcessBeforeInitialization(): " + beanName);
        }
        return bean;
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (bean instanceof LifecycleBean) {
            System.out.println("6. BeanPostProcessor.postProcessAfterInitialization(): " + beanName);
        }
        return bean;
    }
}

// 3. 生命周期演示
public class BeanLifecycleDemo {
    public static void main(String[] args) {
        System.out.println("=== Spring Bean生命周期演示 ===");
        
        // 创建容器
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        context.register(AppConfig.class);
        
        System.out.println("\n--- 容器启动，Bean开始创建 ---");
        context.refresh();
        
        System.out.println("\n--- Bean创建完成，开始使用 ---");
        LifecycleBean lifecycleBean = context.getBean(LifecycleBean.class);
        lifecycleBean.doSomething();
        
        System.out.println("\n--- 容器关闭，Bean开始销毁 ---");
        context.close();
        
        System.out.println("\n--- Bean生命周期结束 ---");
    }
}
```

**不同作用域Bean的生命周期：**

```java
// 4. 不同作用域的生命周期

// Singleton作用域（默认）
@Component
@Scope(ConfigurableBeanFactory.SCOPE_SINGLETON)
class SingletonLifecycleBean {
    
    public SingletonLifecycleBean() {
        System.out.println("Singleton Bean创建：" + this.hashCode());
    }
    
    @PostConstruct
    public void init() {
        System.out.println("Singleton Bean初始化：" + this.hashCode());
    }
    
    @PreDestroy
    public void destroy() {
        System.out.println("Singleton Bean销毁：" + this.hashCode());
    }
}

// Prototype作用域
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
class PrototypeLifecycleBean {
    
    public PrototypeLifecycleBean() {
        System.out.println("Prototype Bean创建：" + this.hashCode());
    }
    
    @PostConstruct
    public void init() {
        System.out.println("Prototype Bean初始化：" + this.hashCode());
    }
    
    @PreDestroy
    public void destroy() {
        // 注意：Prototype作用域的Bean，Spring不会调用销毁方法
        System.out.println("Prototype Bean销毁：" + this.hashCode());
    }
}

// Request作用域（Web环境）
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
class RequestScopedBean {
    
    private final String requestId;
    
    public RequestScopedBean() {
        this.requestId = UUID.randomUUID().toString();
        System.out.println("Request Bean创建：" + requestId);
    }
    
    public String getRequestId() {
        return requestId;
    }
    
    @PreDestroy
    public void destroy() {
        System.out.println("Request Bean销毁：" + requestId);
    }
}

// 作用域演示
public class ScopeLifecycleDemo {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        
        System.out.println("=== Singleton作用域 ===");
        SingletonLifecycleBean singleton1 = context.getBean(SingletonLifecycleBean.class);
        SingletonLifecycleBean singleton2 = context.getBean(SingletonLifecycleBean.class);
        System.out.println("相同实例：" + (singleton1 == singleton2));
        
        System.out.println("\n=== Prototype作用域 ===");
        PrototypeLifecycleBean prototype1 = context.getBean(PrototypeLifecycleBean.class);
        PrototypeLifecycleBean prototype2 = context.getBean(PrototypeLifecycleBean.class);
        System.out.println("不同实例：" + (prototype1 != prototype2));
        
        context.close();
    }
}
```

**Bean生命周期总结图：**

```
容器启动
    ↓
1. 实例化Bean（构造器）
    ↓
2. 设置Bean属性（依赖注入）
    ↓
3. Aware接口回调
   - BeanNameAware.setBeanName()
   - BeanFactoryAware.setBeanFactory()
   - ApplicationContextAware.setApplicationContext()
    ↓
4. BeanPostProcessor.postProcessBeforeInitialization()
    ↓
5. 初始化Bean
   - @PostConstruct方法
   - InitializingBean.afterPropertiesSet()
   - 自定义init-method
    ↓
6. BeanPostProcessor.postProcessAfterInitialization()
    ↓
7. Bean可以使用
    ↓
容器关闭
    ↓
8. 销毁Bean
   - @PreDestroy方法
   - DisposableBean.destroy()
   - 自定义destroy-method
    ↓
Bean销毁完成
```

**生命周期管理最佳实践：**

1. **初始化方法选择**：优先使用`@PostConstruct`，其次是`InitializingBean`接口
2. **销毁方法选择**：优先使用`@PreDestroy`，其次是`DisposableBean`接口
3. **资源管理**：在初始化方法中获取资源，在销毁方法中释放资源
4. **异常处理**：初始化和销毁方法中要妥善处理异常
5. **作用域考虑**：Prototype作用域的Bean不会自动调用销毁方法

**3. Spring AOP的实现原理？**

**答案：**

Spring AOP（Aspect-Oriented Programming，面向切面编程）是一种编程范式，用于将横切关注点（如日志、事务、安全等）从业务逻辑中分离出来，提高代码的模块化程度。

**AOP核心概念：**

```java
// 1. AOP核心概念演示

// 切面（Aspect）：横切关注点的模块化
@Aspect
@Component
public class LoggingAspect {
    
    // 切点（Pointcut）：定义在哪些连接点应用通知
    @Pointcut("execution(* com.example.service.*.*(..))") 
    public void serviceLayer() {}
    
    @Pointcut("@annotation(com.example.annotation.Loggable)")
    public void loggableMethod() {}
    
    // 前置通知（Before Advice）：在目标方法执行前执行
    @Before("serviceLayer()")
    public void logBefore(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        String className = joinPoint.getTarget().getClass().getSimpleName();
        Object[] args = joinPoint.getArgs();
        
        System.out.println("[前置通知] 调用方法: " + className + "." + methodName + 
                          "(), 参数: " + Arrays.toString(args));
    }
    
    // 后置通知（After Returning）：在目标方法正常返回后执行
    @AfterReturning(pointcut = "serviceLayer()", returning = "result")
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("[后置通知] 方法: " + methodName + "() 返回值: " + result);
    }
    
    // 异常通知（After Throwing）：在目标方法抛出异常后执行
    @AfterThrowing(pointcut = "serviceLayer()", throwing = "exception")
    public void logAfterThrowing(JoinPoint joinPoint, Exception exception) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("[异常通知] 方法: " + methodName + "() 抛出异常: " + exception.getMessage());
    }
    
    // 最终通知（After）：无论目标方法是否正常执行都会执行
    @After("serviceLayer()")
    public void logAfter(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("[最终通知] 方法: " + methodName + "() 执行完成");
    }
    
    // 环绕通知（Around）：包围目标方法执行，最强大的通知类型
    @Around("loggableMethod()")
    public Object logAround(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        String methodName = proceedingJoinPoint.getSignature().getName();
        long startTime = System.currentTimeMillis();
        
        System.out.println("[环绕通知-前] 开始执行方法: " + methodName + "()");
        
        try {
            // 执行目标方法
            Object result = proceedingJoinPoint.proceed();
            
            long endTime = System.currentTimeMillis();
            System.out.println("[环绕通知-后] 方法: " + methodName + "() 执行成功，耗时: " + 
                              (endTime - startTime) + "ms");
            
            return result;
        } catch (Exception e) {
            System.out.println("[环绕通知-异常] 方法: " + methodName + "() 执行失败: " + e.getMessage());
            throw e;
        }
    }
}

// 自定义注解用于切点
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Loggable {
    String value() default "";
}
```

**Spring AOP实现原理 - 动态代理：**

```java
// 2. JDK动态代理实现

// 业务接口
public interface UserService {
    User findById(Long id);
    void saveUser(User user);
    void deleteUser(Long id);
}

// 业务实现类
@Service
public class UserServiceImpl implements UserService {
    
    @Override
    public User findById(Long id) {
        System.out.println("执行查询用户业务逻辑: " + id);
        return new User(id, "用户" + id, "user" + id + "@example.com");
    }
    
    @Override
    @Loggable("保存用户")
    public void saveUser(User user) {
        System.out.println("执行保存用户业务逻辑: " + user.getName());
        // 模拟业务处理时间
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    @Override
    public void deleteUser(Long id) {
        if (id == 999L) {
            throw new RuntimeException("无法删除系统用户");
        }
        System.out.println("执行删除用户业务逻辑: " + id);
    }
}

// 手动实现JDK动态代理（Spring内部原理）
public class JdkDynamicProxyDemo {
    
    public static void main(String[] args) {
        // 创建目标对象
        UserService target = new UserServiceImpl();
        
        // 创建代理对象
        UserService proxy = (UserService) Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            target.getClass().getInterfaces(),
            new LoggingInvocationHandler(target)
        );
        
        System.out.println("=== JDK动态代理演示 ===");
        System.out.println("代理对象类型: " + proxy.getClass().getName());
        
        // 通过代理调用方法
        proxy.findById(1L);
        proxy.saveUser(new User(2L, "张三", "zhangsan@example.com"));
        
        try {
            proxy.deleteUser(999L);
        } catch (Exception e) {
            System.out.println("捕获异常: " + e.getMessage());
        }
    }
}

// 自定义InvocationHandler
class LoggingInvocationHandler implements InvocationHandler {
    private final Object target;
    
    public LoggingInvocationHandler(Object target) {
        this.target = target;
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String methodName = method.getName();
        long startTime = System.currentTimeMillis();
        
        System.out.println("[代理] 开始执行方法: " + methodName + "(), 参数: " + Arrays.toString(args));
        
        try {
            // 调用目标方法
            Object result = method.invoke(target, args);
            
            long endTime = System.currentTimeMillis();
            System.out.println("[代理] 方法: " + methodName + "() 执行成功，耗时: " + (endTime - startTime) + "ms");
            
            return result;
        } catch (InvocationTargetException e) {
            Throwable cause = e.getCause();
            System.out.println("[代理] 方法: " + methodName + "() 执行失败: " + cause.getMessage());
            throw cause;
        }
    }
}
```

**CGLIB代理实现：**

```java
// 3. CGLIB代理实现（用于没有接口的类）

// 没有接口的业务类
@Service
public class OrderService {
    
    public Order createOrder(String productName, int quantity) {
        System.out.println("执行创建订单业务逻辑: " + productName + ", 数量: " + quantity);
        return new Order(System.currentTimeMillis(), productName, quantity);
    }
    
    @Loggable("处理订单")
    public void processOrder(Long orderId) {
        System.out.println("执行处理订单业务逻辑: " + orderId);
        // 模拟处理时间
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    public final void finalMethod() {
        // final方法无法被代理
        System.out.println("final方法，无法被CGLIB代理");
    }
}

// 手动实现CGLIB代理（Spring内部原理）
public class CglibProxyDemo {
    
    public static void main(String[] args) {
        // 创建CGLIB代理
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(OrderService.class);
        enhancer.setCallback(new LoggingMethodInterceptor());
        
        // 创建代理对象
        OrderService proxy = (OrderService) enhancer.create();
        
        System.out.println("=== CGLIB代理演示 ===");
        System.out.println("代理对象类型: " + proxy.getClass().getName());
        System.out.println("是否为OrderService子类: " + (proxy instanceof OrderService));
        
        // 通过代理调用方法
        proxy.createOrder("笔记本电脑", 1);
        proxy.processOrder(12345L);
        proxy.finalMethod(); // final方法不会被拦截
    }
}

// 自定义MethodInterceptor
class LoggingMethodInterceptor implements MethodInterceptor {
    
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        String methodName = method.getName();
        
        // 跳过Object类的方法
        if (method.getDeclaringClass() == Object.class) {
            return proxy.invokeSuper(obj, args);
        }
        
        long startTime = System.currentTimeMillis();
        System.out.println("[CGLIB代理] 开始执行方法: " + methodName + "(), 参数: " + Arrays.toString(args));
        
        try {
            // 调用父类方法（目标方法）
            Object result = proxy.invokeSuper(obj, args);
            
            long endTime = System.currentTimeMillis();
            System.out.println("[CGLIB代理] 方法: " + methodName + "() 执行成功，耗时: " + (endTime - startTime) + "ms");
            
            return result;
        } catch (Exception e) {
            System.out.println("[CGLIB代理] 方法: " + methodName + "() 执行失败: " + e.getMessage());
            throw e;
        }
    }
}
```

**JDK动态代理 vs CGLIB代理对比：**

| 特性 | JDK动态代理 | CGLIB代理 |
|------|-------------|----------|
| **实现方式** | 基于接口的代理 | 基于继承的代理 |
| **要求** | 目标类必须实现接口 | 目标类不能是final |
| **性能** | 创建快，调用相对慢 | 创建慢，调用快 |
| **代理对象** | 实现相同接口的新对象 | 目标类的子类 |
| **方法限制** | 只能代理接口方法 | 不能代理final/static/private方法 |
| **Spring默认** | 有接口时使用 | 无接口时使用 |

**4. @Autowired和@Resource的区别？**

**答案：**

`@Autowired`和`@Resource`都是用于依赖注入的注解，但它们来源不同、注入策略不同、使用方式也有差异。

**基本区别对比：**

| 特性 | @Autowired | @Resource |
|------|------------|----------|
| **来源** | Spring框架注解 | JDK标准注解（JSR-250） |
| **注入策略** | 默认按类型注入 | 默认按名称注入 |
| **备选策略** | 配合@Qualifier按名称注入 | 找不到名称时按类型注入 |
| **支持位置** | 字段、构造器、方法、参数 | 字段、方法 |
| **required属性** | 支持（默认true） | 不支持 |
| **集合注入** | 支持注入List、Map等集合 | 不支持集合注入 |

**详细代码演示：**

```java
// 1. 基础接口和实现类
public interface NotificationService {
    void sendNotification(String message);
}

@Service("emailNotification")
public class EmailNotificationService implements NotificationService {
    @Override
    public void sendNotification(String message) {
        System.out.println("邮件通知: " + message);
    }
}

@Service("smsNotification")
public class SmsNotificationService implements NotificationService {
    @Override
    public void sendNotification(String message) {
        System.out.println("短信通知: " + message);
    }
}

@Service("pushNotification")
public class PushNotificationService implements NotificationService {
    @Override
    public void sendNotification(String message) {
        System.out.println("推送通知: " + message);
    }
}

// 2. @Autowired注入演示
@Service
public class AutowiredDemoService {
    
    // 字段注入 - 按类型注入（如果有多个实现会报错）
    // @Autowired
    // private NotificationService notificationService; // 会报错：有多个NotificationService实现
    
    // 字段注入 - 配合@Qualifier按名称注入
    @Autowired
    @Qualifier("emailNotification")
    private NotificationService emailService;
    
    // 字段注入 - 配合@Primary注解的Bean
    @Autowired
    private NotificationService primaryService; // 注入标记了@Primary的Bean
    
    // 构造器注入（推荐方式）
    private final NotificationService constructorInjectedService;
    
    @Autowired
    public AutowiredDemoService(@Qualifier("smsNotification") NotificationService service) {
        this.constructorInjectedService = service;
    }
    
    // Setter方法注入
    private NotificationService setterInjectedService;
    
    @Autowired
    @Qualifier("pushNotification")
    public void setNotificationService(NotificationService service) {
        this.setterInjectedService = service;
    }
    
    // 可选依赖注入
    @Autowired(required = false)
    private OptionalService optionalService; // 如果容器中没有OptionalService，不会报错
    
    // 集合注入 - 注入所有NotificationService实现
    @Autowired
    private List<NotificationService> allNotificationServices;
    
    // Map注入 - key为Bean名称，value为Bean实例
    @Autowired
    private Map<String, NotificationService> notificationServiceMap;
    
    public void demonstrateAutowired() {
        System.out.println("=== @Autowired注入演示 ===");
        
        emailService.sendNotification("@Qualifier指定的邮件服务");
        constructorInjectedService.sendNotification("构造器注入的短信服务");
        setterInjectedService.sendNotification("Setter注入的推送服务");
        
        System.out.println("\n所有NotificationService实现（" + allNotificationServices.size() + "个）:");
        allNotificationServices.forEach(service -> 
            service.sendNotification("来自" + service.getClass().getSimpleName()));
        
        System.out.println("\nNotificationService Map:");
        notificationServiceMap.forEach((name, service) -> 
            System.out.println("Bean名称: " + name + ", 类型: " + service.getClass().getSimpleName()));
        
        if (optionalService != null) {
            System.out.println("可选服务已注入");
        } else {
            System.out.println("可选服务未注入（容器中不存在）");
        }
    }
}

// 3. @Resource注入演示
@Service
public class ResourceDemoService {
    
    // 按名称注入 - 指定Bean名称
    @Resource(name = "emailNotification")
    private NotificationService emailService;
    
    // 按名称注入 - 根据字段名匹配Bean名称
    @Resource
    private NotificationService smsNotification; // 自动匹配名为"smsNotification"的Bean
    
    // 按类型注入 - 当指定的名称不存在时
    @Resource(name = "nonExistentService")
    private NotificationService fallbackService; // 名称不存在，会按类型注入（如果只有一个实现）
    
    // Setter方法注入
    private NotificationService setterInjectedService;
    
    @Resource(name = "pushNotification")
    public void setNotificationService(NotificationService service) {
        this.setterInjectedService = service;
    }
    
    // @Resource不支持构造器注入
    // @Resource // 编译错误
    // public ResourceDemoService(NotificationService service) {}
    
    // @Resource不支持集合注入
    // @Resource // 运行时错误
    // private List<NotificationService> services;
    
    public void demonstrateResource() {
        System.out.println("\n=== @Resource注入演示 ===");
        
        emailService.sendNotification("@Resource指定名称的邮件服务");
        smsNotification.sendNotification("@Resource按字段名匹配的短信服务");
        setterInjectedService.sendNotification("@Resource Setter注入的推送服务");
        
        if (fallbackService != null) {
            fallbackService.sendNotification("@Resource按类型回退注入的服务");
        }
    }
}

// 4. 混合使用演示
@Service
public class MixedInjectionService {
    
    // 使用@Autowired注入主要服务
    @Autowired
    @Qualifier("emailNotification")
    private NotificationService primaryNotificationService;
    
    // 使用@Resource注入备用服务
    @Resource(name = "smsNotification")
    private NotificationService backupNotificationService;
    
    // 使用@Autowired注入所有服务用于负载均衡
    @Autowired
    private List<NotificationService> allServices;
    
    private int currentIndex = 0;
    
    public void sendNotificationWithLoadBalance(String message) {
        if (allServices != null && !allServices.isEmpty()) {
            NotificationService service = allServices.get(currentIndex % allServices.size());
            service.sendNotification("[负载均衡] " + message);
            currentIndex++;
        }
    }
    
    public void sendNotificationWithFallback(String message) {
        try {
            primaryNotificationService.sendNotification("[主要服务] " + message);
        } catch (Exception e) {
            System.out.println("主要服务失败，使用备用服务");
            backupNotificationService.sendNotification("[备用服务] " + message);
        }
    }
}

// 5. 配置类演示不同注入策略
@Configuration
public class InjectionConfig {
    
    // 标记主要的NotificationService实现
    @Bean
    @Primary
    public NotificationService primaryNotificationService() {
        return new EmailNotificationService();
    }
    
    // 条件注入
    @Bean
    @ConditionalOnProperty(name = "notification.sms.enabled", havingValue = "true")
    public NotificationService conditionalSmsService() {
        return new SmsNotificationService();
    }
}

// 6. 完整演示类
public class InjectionComparisonDemo {
    
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(InjectionConfig.class);
        
        // 演示@Autowired
        AutowiredDemoService autowiredService = context.getBean(AutowiredDemoService.class);
        autowiredService.demonstrateAutowired();
        
        // 演示@Resource
        ResourceDemoService resourceService = context.getBean(ResourceDemoService.class);
        resourceService.demonstrateResource();
        
        // 演示混合使用
        MixedInjectionService mixedService = context.getBean(MixedInjectionService.class);
        
        System.out.println("\n=== 混合注入演示 ===");
        mixedService.sendNotificationWithFallback("测试消息1");
        
        System.out.println("\n=== 负载均衡演示 ===");
        for (int i = 0; i < 5; i++) {
            mixedService.sendNotificationWithLoadBalance("负载均衡消息 " + (i + 1));
        }
    }
}
```

**注入失败场景对比：**

```java
// 7. 注入失败场景演示
@Service
public class InjectionFailureDemo {
    
    // @Autowired注入失败场景
    
    // 场景1：多个相同类型Bean，没有@Qualifier
    // @Autowired
    // private NotificationService service; // 失败：NoUniqueBeanDefinitionException
    
    // 场景2：必需依赖不存在
    // @Autowired
    // private NonExistentService service; // 失败：NoSuchBeanDefinitionException
    
    // 场景3：可选依赖不存在（不会失败）
    @Autowired(required = false)
    private NonExistentService optionalService; // 成功：注入null
    
    // @Resource注入失败场景
    
    // 场景1：指定名称不存在，且有多个相同类型Bean
    // @Resource(name = "nonExistent")
    // private NotificationService service; // 失败：按类型回退时发现多个Bean
    
    // 场景2：字段名不匹配任何Bean名称，且有多个相同类型Bean
    // @Resource
    // private NotificationService unknownService; // 失败：名称匹配失败，类型匹配发现多个Bean
    
    // 场景3：完全找不到匹配的Bean
    // @Resource
    // private NonExistentService service; // 失败：NoSuchBeanDefinitionException
}
```

**最佳实践建议：**

1. **优先使用@Autowired**：Spring生态中更常用，功能更强大
2. **构造器注入优于字段注入**：保证依赖不可变，便于测试
3. **明确指定Bean名称**：使用@Qualifier避免歧义
4. **合理使用@Primary**：为常用的实现标记主要Bean
5. **谨慎使用required=false**：确保业务逻辑能处理null值
6. **集合注入的妙用**：利用List、Map注入实现策略模式

**5. Spring Boot自动配置原理？**

**答案：**

Spring Boot的自动配置是其核心特性之一，通过"约定优于配置"的理念，自动为应用程序配置所需的Bean，大大简化了Spring应用的开发和配置工作。

**自动配置核心机制：**

1. **@EnableAutoConfiguration注解**：启用自动配置
2. **spring.factories文件**：定义自动配置类列表
3. **条件注解**：控制配置类是否生效
4. **配置属性**：通过application.properties/yml自定义配置

**详细实现原理：**

```java
// 1. Spring Boot启动类
@SpringBootApplication // 包含@EnableAutoConfiguration
public class AutoConfigDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(AutoConfigDemoApplication.class, args);
    }
}

// @SpringBootApplication注解的组成
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration // 等同于@Configuration
@EnableAutoConfiguration // 启用自动配置
@ComponentScan(excludeFilters = { // 组件扫描
    @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
    @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class)
})
public @interface SpringBootApplication {
    // ...
}

// 2. @EnableAutoConfiguration注解分析
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class) // 关键：导入自动配置选择器
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
    
    Class<?>[] exclude() default {};
    String[] excludeName() default {};
}

// 3. 自动配置导入选择器（简化版）
public class AutoConfigurationImportSelector implements DeferredImportSelector {
    
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        if (!isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        }
        
        // 获取自动配置条目
        AutoConfigurationEntry autoConfigurationEntry = 
            getAutoConfigurationEntry(annotationMetadata);
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }
    
    protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
        if (!isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        }
        
        AnnotationAttributes attributes = getAttributes(annotationMetadata);
        
        // 从spring.factories文件加载候选配置类
        List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
        
        // 去重
        configurations = removeDuplicates(configurations);
        
        // 排除指定的配置类
        Set<String> exclusions = getExclusions(annotationMetadata, attributes);
        checkExcludedClasses(configurations, exclusions);
        configurations.removeAll(exclusions);
        
        // 过滤：根据条件注解判断是否应该加载
        configurations = getConfigurationClassFilter().filter(configurations);
        
        // 触发自动配置导入事件
        fireAutoConfigurationImportEvents(configurations, exclusions);
        
        return new AutoConfigurationEntry(configurations, exclusions);
    }
    
    protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, 
                                                      AnnotationAttributes attributes) {
        // 从META-INF/spring.factories文件加载
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
            getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
        
        Assert.notEmpty(configurations, 
            "No auto configuration classes found in META-INF/spring.factories.");
        return configurations;
    }
}

// 4. spring.factories文件示例（部分内容）
/*
# META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration
*/

// 5. 条件注解详解
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ DataSource.class, JdbcTemplate.class }) // classpath中存在指定类时生效
@ConditionalOnSingleCandidate(DataSource.class) // 容器中只有一个DataSource Bean时生效
@AutoConfigureAfter(DataSourceAutoConfiguration.class) // 在DataSourceAutoConfiguration之后配置
@EnableConfigurationProperties(JdbcProperties.class) // 启用配置属性
public class JdbcTemplateAutoConfiguration {
    
    @Configuration(proxyBeanMethods = false)
    @ConditionalOnMissingBean(JdbcOperations.class) // 容器中不存在JdbcOperations Bean时生效
    static class JdbcTemplateConfiguration {
        
        @Bean
        @Primary
        JdbcTemplate jdbcTemplate(DataSource dataSource, JdbcProperties properties) {
            JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
            JdbcProperties.Template template = properties.getTemplate();
            jdbcTemplate.setFetchSize(template.getFetchSize());
            jdbcTemplate.setMaxRows(template.getMaxRows());
            if (template.getQueryTimeout() != null) {
                jdbcTemplate.setQueryTimeout((int) template.getQueryTimeout().getSeconds());
            }
            return jdbcTemplate;
        }
    }
}

// 6. 常用条件注解示例
@Configuration
public class ConditionalAnnotationExamples {
    
    // 类路径条件
    @Bean
    @ConditionalOnClass(name = "com.mysql.cj.jdbc.Driver")
    public DataSource mysqlDataSource() {
        return new HikariDataSource();
    }
    
    // 缺少类条件
    @Bean
    @ConditionalOnMissingClass("com.mysql.cj.jdbc.Driver")
    public DataSource h2DataSource() {
        return new org.h2.jdbcx.JdbcDataSource();
    }
    
    // Bean存在条件
    @Bean
    @ConditionalOnBean(DataSource.class)
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
    
    // Bean不存在条件
    @Bean
    @ConditionalOnMissingBean(name = "customJdbcTemplate")
    public JdbcTemplate defaultJdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
    
    // 属性条件
    @Bean
    @ConditionalOnProperty(
        prefix = "app.feature",
        name = "cache",
        havingValue = "true",
        matchIfMissing = false
    )
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager();
    }
    
    // 表达式条件
    @Bean
    @ConditionalOnExpression("${app.feature.advanced:false} and ${app.environment} == 'production'")
    public AdvancedFeatureService advancedFeatureService() {
        return new AdvancedFeatureService();
    }
    
    // Web应用条件
    @Bean
    @ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
    public FilterRegistrationBean<CustomFilter> customFilter() {
        FilterRegistrationBean<CustomFilter> registration = new FilterRegistrationBean<>();
        registration.setFilter(new CustomFilter());
        registration.addUrlPatterns("/*");
        return registration;
    }
}

// 7. 自定义自动配置类示例
@Configuration
@ConditionalOnClass(CustomService.class)
@ConditionalOnProperty(
    prefix = "custom.service",
    name = "enabled",
    havingValue = "true",
    matchIfMissing = true
)
@EnableConfigurationProperties(CustomServiceProperties.class)
public class CustomServiceAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public CustomService customService(CustomServiceProperties properties) {
        CustomService service = new CustomService();
        service.setMaxConnections(properties.getMaxConnections());
        service.setTimeout(properties.getTimeout());
        service.setRetryCount(properties.getRetryCount());
        return service;
    }
    
    @Bean
    @ConditionalOnBean(CustomService.class)
    @ConditionalOnProperty(prefix = "custom.service", name = "monitoring.enabled", havingValue = "true")
    public CustomServiceMonitor customServiceMonitor(CustomService customService) {
        return new CustomServiceMonitor(customService);
    }
}

// 8. 配置属性类
@ConfigurationProperties(prefix = "custom.service")
public class CustomServiceProperties {
    
    private boolean enabled = true;
    private int maxConnections = 10;
    private Duration timeout = Duration.ofSeconds(30);
    private int retryCount = 3;
    
    // 监控配置
    private Monitoring monitoring = new Monitoring();
    
    public static class Monitoring {
        private boolean enabled = false;
        private Duration interval = Duration.ofMinutes(5);
        
        // getters and setters
        public boolean isEnabled() { return enabled; }
        public void setEnabled(boolean enabled) { this.enabled = enabled; }
        public Duration getInterval() { return interval; }
        public void setInterval(Duration interval) { this.interval = interval; }
    }
    
    // getters and setters
    public boolean isEnabled() { return enabled; }
    public void setEnabled(boolean enabled) { this.enabled = enabled; }
    public int getMaxConnections() { return maxConnections; }
    public void setMaxConnections(int maxConnections) { this.maxConnections = maxConnections; }
    public Duration getTimeout() { return timeout; }
    public void setTimeout(Duration timeout) { this.timeout = timeout; }
    public int getRetryCount() { return retryCount; }
    public void setRetryCount(int retryCount) { this.retryCount = retryCount; }
    public Monitoring getMonitoring() { return monitoring; }
    public void setMonitoring(Monitoring monitoring) { this.monitoring = monitoring; }
}
```

**自动配置执行流程图：**

```
启动Spring Boot应用
        ↓
@EnableAutoConfiguration注解
        ↓
AutoConfigurationImportSelector
        ↓
从spring.factories加载候选配置类
        ↓
应用过滤器（条件注解判断）
        ↓
创建符合条件的配置类实例
        ↓
注册Bean到Spring容器
        ↓
应用程序启动完成
```

**条件注解汇总表：**

| 条件注解 | 作用 | 示例 |
|---------|------|------|
| @ConditionalOnClass | 类路径存在指定类 | @ConditionalOnClass(DataSource.class) |
| @ConditionalOnMissingClass | 类路径不存在指定类 | @ConditionalOnMissingClass("com.mysql.Driver") |
| @ConditionalOnBean | 容器中存在指定Bean | @ConditionalOnBean(DataSource.class) |
| @ConditionalOnMissingBean | 容器中不存在指定Bean | @ConditionalOnMissingBean(JdbcTemplate.class) |
| @ConditionalOnProperty | 配置属性满足条件 | @ConditionalOnProperty(name="app.enabled", havingValue="true") |
| @ConditionalOnResource | 类路径存在指定资源 | @ConditionalOnResource(resources="classpath:config.xml") |
| @ConditionalOnWebApplication | 是Web应用 | @ConditionalOnWebApplication |
| @ConditionalOnNotWebApplication | 不是Web应用 | @ConditionalOnNotWebApplication |
| @ConditionalOnExpression | SpEL表达式为true | @ConditionalOnExpression("${app.feature:false}") |
| @ConditionalOnJava | Java版本满足条件 | @ConditionalOnJava(JavaVersion.EIGHT) |
| @ConditionalOnSingleCandidate | 容器中只有一个候选Bean | @ConditionalOnSingleCandidate(DataSource.class) |

**自动配置最佳实践：**

1. **合理使用条件注解**：确保配置类只在需要时生效
2. **提供配置属性**：允许用户自定义配置
3. **设置合理默认值**：遵循"约定优于配置"原则
4. **考虑配置顺序**：使用@AutoConfigureBefore/@AutoConfigureAfter
5. **提供排除机制**：允许用户排除不需要的自动配置
6. **编写测试**：确保自动配置在各种条件下正常工作

### Spring MVC相关

**6. Spring MVC的执行流程？**

**答案：**

Spring MVC是基于Model-View-Controller设计模式的Web框架，其核心是DispatcherServlet前端控制器，负责协调各个组件完成请求处理。

**Spring MVC核心组件：**

1. **DispatcherServlet**：前端控制器，统一处理请求
2. **HandlerMapping**：处理器映射器，找到请求对应的处理器
3. **HandlerAdapter**：处理器适配器，执行具体的处理器方法
4. **Controller**：控制器，处理业务逻辑
5. **ViewResolver**：视图解析器，解析视图名称
6. **View**：视图，渲染模型数据

**详细执行流程：**

```java
// 1. 控制器示例
@Controller
@RequestMapping("/user")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    // 处理GET请求，返回视图名称
    @GetMapping("/list")
    public String listUsers(Model model, 
                           @RequestParam(defaultValue = "1") int page,
                           @RequestParam(defaultValue = "10") int size) {
        
        System.out.println("1. 控制器方法开始执行");
        
        // 业务逻辑处理
        PageResult<User> users = userService.findUsers(page, size);
        
        // 添加模型数据
        model.addAttribute("users", users.getData());
        model.addAttribute("totalPages", users.getTotalPages());
        model.addAttribute("currentPage", page);
        
        System.out.println("2. 控制器方法执行完成，返回视图名称: user/list");
        
        // 返回视图名称
        return "user/list"; // 将被ViewResolver解析为具体视图
    }
    
    // 处理POST请求，返回ModelAndView
    @PostMapping("/create")
    public ModelAndView createUser(@Valid @ModelAttribute User user, 
                                  BindingResult bindingResult) {
        
        ModelAndView mav = new ModelAndView();
        
        if (bindingResult.hasErrors()) {
            mav.setViewName("user/form");
            mav.addObject("user", user);
            mav.addObject("errors", bindingResult.getAllErrors());
            return mav;
        }
        
        User savedUser = userService.saveUser(user);
        mav.setViewName("redirect:/user/list");
        mav.addObject("message", "用户创建成功");
        
        return mav;
    }
    
    // RESTful API，返回JSON数据
    @GetMapping("/api/{id}")
    @ResponseBody
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        User user = userService.findById(id);
        if (user != null) {
            return ResponseEntity.ok(user);
        } else {
            return ResponseEntity.notFound().build();
        }
    }
    
    // 异常处理
    @ExceptionHandler(UserNotFoundException.class)
    public ModelAndView handleUserNotFound(UserNotFoundException ex) {
        ModelAndView mav = new ModelAndView("error/404");
        mav.addObject("message", ex.getMessage());
        return mav;
    }
}

// 2. 自定义HandlerMapping示例
@Component
public class CustomHandlerMapping implements HandlerMapping {
    
    private final Map<String, Object> handlerMap = new HashMap<>();
    
    @PostConstruct
    public void initHandlerMap() {
        // 注册自定义处理器
        handlerMap.put("/custom/hello", new CustomController());
    }
    
    @Override
    public HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
        String requestURI = request.getRequestURI();
        Object handler = handlerMap.get(requestURI);
        
        if (handler != null) {
            // 创建处理器执行链，可以添加拦截器
            HandlerExecutionChain chain = new HandlerExecutionChain(handler);
            chain.addInterceptor(new LoggingInterceptor());
            return chain;
        }
        
        return null;
    }
}

// 3. 自定义HandlerAdapter示例
@Component
public class CustomHandlerAdapter implements HandlerAdapter {
    
    @Override
    public boolean supports(Object handler) {
        return handler instanceof CustomController;
    }
    
    @Override
    public ModelAndView handle(HttpServletRequest request, 
                              HttpServletResponse response, 
                              Object handler) throws Exception {
        
        CustomController controller = (CustomController) handler;
        
        // 执行控制器方法
        String result = controller.handleRequest(request, response);
        
        // 创建ModelAndView
        ModelAndView mav = new ModelAndView();
        mav.setViewName(result);
        mav.addObject("timestamp", System.currentTimeMillis());
        
        return mav;
    }
    
    @Override
    public long getLastModified(HttpServletRequest request, Object handler) {
        return -1;
    }
}

// 4. 自定义ViewResolver示例
@Component
public class CustomViewResolver implements ViewResolver {
    
    private String prefix = "/WEB-INF/views/";
    private String suffix = ".jsp";
    
    @Override
    public View resolveViewName(String viewName, Locale locale) throws Exception {
        
        // 处理重定向
        if (viewName.startsWith("redirect:")) {
            String redirectUrl = viewName.substring(9);
            return new RedirectView(redirectUrl);
        }
        
        // 处理转发
        if (viewName.startsWith("forward:")) {
            String forwardUrl = viewName.substring(8);
            return new InternalResourceView(forwardUrl);
        }
        
        // 普通视图解析
        String viewPath = prefix + viewName + suffix;
        InternalResourceView view = new InternalResourceView(viewPath);
        view.setContentType("text/html;charset=UTF-8");
        
        return view;
    }
}

// 5. 拦截器示例
@Component
public class RequestProcessingInterceptor implements HandlerInterceptor {
    
    private static final Logger logger = LoggerFactory.getLogger(RequestProcessingInterceptor.class);
    
    @Override
    public boolean preHandle(HttpServletRequest request, 
                           HttpServletResponse response, 
                           Object handler) throws Exception {
        
        long startTime = System.currentTimeMillis();
        request.setAttribute("startTime", startTime);
        
        logger.info("请求开始: {} {}, 处理器: {}", 
                   request.getMethod(), 
                   request.getRequestURI(), 
                   handler.getClass().getSimpleName());
        
        return true; // 继续执行
    }
    
    @Override
    public void postHandle(HttpServletRequest request, 
                          HttpServletResponse response, 
                          Object handler, 
                          ModelAndView modelAndView) throws Exception {
        
        if (modelAndView != null) {
            logger.info("控制器执行完成，视图名称: {}, 模型数据: {}", 
                       modelAndView.getViewName(), 
                       modelAndView.getModel().keySet());
        }
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, 
                               HttpServletResponse response, 
                               Object handler, 
                               Exception ex) throws Exception {
        
        Long startTime = (Long) request.getAttribute("startTime");
        if (startTime != null) {
            long endTime = System.currentTimeMillis();
            long processingTime = endTime - startTime;
            
            logger.info("请求完成: {} {}, 处理时间: {}ms, 状态码: {}", 
                       request.getMethod(), 
                       request.getRequestURI(), 
                       processingTime, 
                       response.getStatus());
        }
        
        if (ex != null) {
            logger.error("请求处理异常: ", ex);
        }
    }
}

// 6. 完整的请求处理流程演示
@Component
public class SpringMvcFlowDemo {
    
    public void demonstrateRequestFlow() {
        System.out.println("=== Spring MVC请求处理流程演示 ===");
        
        // 模拟请求流程
        simulateRequestFlow("/user/list?page=1&size=10");
    }
    
    private void simulateRequestFlow(String requestUrl) {
        System.out.println("\n请求URL: " + requestUrl);
        
        // 步骤1：请求到达DispatcherServlet
        System.out.println("1. 请求到达DispatcherServlet");
        
        // 步骤2：DispatcherServlet查询HandlerMapping
        System.out.println("2. DispatcherServlet查询HandlerMapping找到处理器");
        System.out.println("   - RequestMappingHandlerMapping找到UserController.listUsers方法");
        
        // 步骤3：获取HandlerAdapter
        System.out.println("3. 获取HandlerAdapter");
        System.out.println("   - RequestMappingHandlerAdapter适配@RequestMapping注解的方法");
        
        // 步骤4：执行拦截器preHandle
        System.out.println("4. 执行拦截器preHandle方法");
        
        // 步骤5：HandlerAdapter执行处理器方法
        System.out.println("5. HandlerAdapter执行控制器方法");
        System.out.println("   - 参数解析：@RequestParam page=1, size=10");
        System.out.println("   - 执行业务逻辑");
        System.out.println("   - 返回视图名称: user/list");
        
        // 步骤6：执行拦截器postHandle
        System.out.println("6. 执行拦截器postHandle方法");
        
        // 步骤7：ViewResolver解析视图
        System.out.println("7. ViewResolver解析视图名称");
        System.out.println("   - InternalResourceViewResolver: user/list -> /WEB-INF/views/user/list.jsp");
        
        // 步骤8：渲染视图
        System.out.println("8. 渲染视图");
        System.out.println("   - JSP引擎渲染页面");
        System.out.println("   - 填充模型数据");
        
        // 步骤9：执行拦截器afterCompletion
        System.out.println("9. 执行拦截器afterCompletion方法");
        
        // 步骤10：返回响应
        System.out.println("10. 返回HTTP响应给客户端");
    }
}
```

**Spring MVC执行流程图：**

```
客户端请求
    ↓
DispatcherServlet（前端控制器）
    ↓
HandlerMapping（处理器映射器）
    ↓
HandlerExecutionChain（处理器执行链）
    ↓
拦截器preHandle()
    ↓
HandlerAdapter（处理器适配器）
    ↓
Controller（控制器）
    ↓
ModelAndView
    ↓
拦截器postHandle()
    ↓
ViewResolver（视图解析器）
    ↓
View（视图）
    ↓
视图渲染
    ↓
拦截器afterCompletion()
    ↓
响应返回客户端
```

**核心组件详细说明：**

| 组件 | 作用 | 常见实现 |
|------|------|----------|
| **DispatcherServlet** | 前端控制器，统一处理请求 | DispatcherServlet |
| **HandlerMapping** | 根据请求找到对应的处理器 | RequestMappingHandlerMapping<br>BeanNameUrlHandlerMapping |
| **HandlerAdapter** | 执行具体的处理器方法 | RequestMappingHandlerAdapter<br>HttpRequestHandlerAdapter |
| **Controller** | 处理业务逻辑 | @Controller注解的类 |
| **ViewResolver** | 解析视图名称为具体视图 | InternalResourceViewResolver<br>ThymeleafViewResolver |
| **View** | 渲染模型数据 | JstlView<br>RedirectView<br>JsonView |
| **HandlerInterceptor** | 拦截器，在处理器执行前后进行处理 | 自定义拦截器实现 |

**请求处理的关键步骤：**

1. **请求接收**：DispatcherServlet接收HTTP请求
2. **处理器查找**：通过HandlerMapping找到对应的处理器
3. **适配器获取**：根据处理器类型获取对应的HandlerAdapter
4. **拦截器前置处理**：执行拦截器的preHandle方法
5. **处理器执行**：HandlerAdapter调用具体的处理器方法
6. **拦截器后置处理**：执行拦截器的postHandle方法
7. **视图解析**：ViewResolver将视图名称解析为具体的View对象
8. **视图渲染**：View对象渲染模型数据生成响应内容
9. **拦截器完成处理**：执行拦截器的afterCompletion方法
10. **响应返回**：将渲染后的内容返回给客户端

**7. Spring MVC中的拦截器和过滤器的区别？**

**答案：**

拦截器（Interceptor）和过滤器（Filter）都是Web开发中常用的组件，但它们在实现机制、执行时机和功能范围上有显著差异。

**核心区别对比：**

| 特性 | 过滤器（Filter） | 拦截器（Interceptor） |
|------|------------------|----------------------|
| **规范归属** | Servlet规范 | Spring框架 |
| **执行时机** | DispatcherServlet之前 | DispatcherServlet之后，Controller前后 |
| **配置方式** | web.xml或@WebFilter | Spring配置或@Component |
| **依赖注入** | 不支持Spring依赖注入 | 支持Spring依赖注入 |
| **访问范围** | 所有请求（包括静态资源） | 仅Spring MVC处理的请求 |
| **异常处理** | 无法处理Controller异常 | 可以处理Controller异常 |
| **执行粒度** | 粗粒度（请求级别） | 细粒度（方法级别） |
| **性能影响** | 较小 | 相对较大 |

**详细代码示例：**

```java
// 1. 过滤器示例
@WebFilter(urlPatterns = "/*", filterName = "requestLoggingFilter")
public class RequestLoggingFilter implements Filter {
    
    private static final Logger logger = LoggerFactory.getLogger(RequestLoggingFilter.class);
    
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        logger.info("RequestLoggingFilter初始化");
    }
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                        FilterChain chain) throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        
        // 请求开始时间
        long startTime = System.currentTimeMillis();
        
        // 请求信息记录
        String requestURI = httpRequest.getRequestURI();
        String method = httpRequest.getMethod();
        String queryString = httpRequest.getQueryString();
        String clientIP = getClientIP(httpRequest);
        
        logger.info("[FILTER] 请求开始 - {} {} {} from {}", 
                   method, requestURI, 
                   queryString != null ? "?" + queryString : "", 
                   clientIP);
        
        try {
            // 继续执行过滤器链
            chain.doFilter(request, response);
            
        } catch (Exception e) {
            logger.error("[FILTER] 请求处理异常: ", e);
            throw e;
            
        } finally {
            // 请求结束处理
            long endTime = System.currentTimeMillis();
            long processingTime = endTime - startTime;
            
            logger.info("[FILTER] 请求结束 - {} {} 状态码: {} 耗时: {}ms", 
                       method, requestURI, 
                       httpResponse.getStatus(), 
                       processingTime);
        }
    }
    
    @Override
    public void destroy() {
        logger.info("RequestLoggingFilter销毁");
    }
    
    private String getClientIP(HttpServletRequest request) {
        String xForwardedFor = request.getHeader("X-Forwarded-For");
        if (xForwardedFor != null && !xForwardedFor.isEmpty()) {
            return xForwardedFor.split(",")[0].trim();
        }
        
        String xRealIP = request.getHeader("X-Real-IP");
        if (xRealIP != null && !xRealIP.isEmpty()) {
            return xRealIP;
        }
        
        return request.getRemoteAddr();
    }
}

// 2. 字符编码过滤器
@WebFilter(urlPatterns = "/*", filterName = "characterEncodingFilter")
public class CharacterEncodingFilter implements Filter {
    
    private String encoding = "UTF-8";
    private boolean forceEncoding = true;
    
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        String encodingParam = filterConfig.getInitParameter("encoding");
        if (encodingParam != null) {
            this.encoding = encodingParam;
        }
        
        String forceEncodingParam = filterConfig.getInitParameter("forceEncoding");
        if (forceEncodingParam != null) {
            this.forceEncoding = Boolean.parseBoolean(forceEncodingParam);
        }
    }
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                        FilterChain chain) throws IOException, ServletException {
        
        // 设置请求编码
        if (forceEncoding || request.getCharacterEncoding() == null) {
            request.setCharacterEncoding(encoding);
        }
        
        // 设置响应编码
        if (forceEncoding || response.getCharacterEncoding() == null) {
            response.setCharacterEncoding(encoding);
        }
        
        chain.doFilter(request, response);
    }
}

// 3. 拦截器示例
@Component
public class AuthenticationInterceptor implements HandlerInterceptor {
    
    private static final Logger logger = LoggerFactory.getLogger(AuthenticationInterceptor.class);
    
    @Autowired
    private UserService userService; // 可以注入Spring Bean
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Override
    public boolean preHandle(HttpServletRequest request, 
                           HttpServletResponse response, 
                           Object handler) throws Exception {
        
        logger.info("[INTERCEPTOR] preHandle - 开始身份验证");
        
        // 检查是否是HandlerMethod
        if (!(handler instanceof HandlerMethod)) {
            return true;
        }
        
        HandlerMethod handlerMethod = (HandlerMethod) handler;
        
        // 检查方法是否需要认证
        RequireAuth requireAuth = handlerMethod.getMethodAnnotation(RequireAuth.class);
        if (requireAuth == null) {
            requireAuth = handlerMethod.getBeanType().getAnnotation(RequireAuth.class);
        }
        
        if (requireAuth == null) {
            return true; // 不需要认证
        }
        
        // 获取token
        String token = extractToken(request);
        if (token == null) {
            logger.warn("[INTERCEPTOR] 缺少认证token");
            sendUnauthorizedResponse(response, "缺少认证token");
            return false;
        }
        
        // 验证token
        try {
            UserInfo userInfo = validateToken(token);
            if (userInfo == null) {
                logger.warn("[INTERCEPTOR] 无效的token: {}", token);
                sendUnauthorizedResponse(response, "无效的token");
                return false;
            }
            
            // 检查用户权限
            if (requireAuth.role().length > 0) {
                boolean hasRequiredRole = Arrays.stream(requireAuth.role())
                    .anyMatch(role -> userInfo.getRoles().contains(role));
                
                if (!hasRequiredRole) {
                    logger.warn("[INTERCEPTOR] 用户权限不足: {} 需要: {}", 
                               userInfo.getRoles(), Arrays.toString(requireAuth.role()));
                    sendForbiddenResponse(response, "权限不足");
                    return false;
                }
            }
            
            // 将用户信息存储到请求属性中
            request.setAttribute("currentUser", userInfo);
            
            logger.info("[INTERCEPTOR] 用户认证成功: {}", userInfo.getUsername());
            return true;
            
        } catch (Exception e) {
            logger.error("[INTERCEPTOR] token验证异常: ", e);
            sendUnauthorizedResponse(response, "认证失败");
            return false;
        }
    }
    
    @Override
    public void postHandle(HttpServletRequest request, 
                          HttpServletResponse response, 
                          Object handler, 
                          ModelAndView modelAndView) throws Exception {
        
        logger.info("[INTERCEPTOR] postHandle - 控制器执行完成");
        
        // 可以修改ModelAndView
        if (modelAndView != null) {
            UserInfo currentUser = (UserInfo) request.getAttribute("currentUser");
            if (currentUser != null) {
                modelAndView.addObject("currentUser", currentUser);
            }
        }
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, 
                               HttpServletResponse response, 
                               Object handler, 
                               Exception ex) throws Exception {
        
        logger.info("[INTERCEPTOR] afterCompletion - 请求处理完成");
        
        // 清理资源
        request.removeAttribute("currentUser");
        
        // 记录异常
        if (ex != null) {
            logger.error("[INTERCEPTOR] 控制器执行异常: ", ex);
        }
    }
    
    private String extractToken(HttpServletRequest request) {
        // 从Header中获取token
        String authHeader = request.getHeader("Authorization");
        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            return authHeader.substring(7);
        }
        
        // 从参数中获取token
        return request.getParameter("token");
    }
    
    private UserInfo validateToken(String token) {
        // 从Redis中获取token信息
        String userInfoJson = (String) redisTemplate.opsForValue().get("token:" + token);
        if (userInfoJson != null) {
            return JSON.parseObject(userInfoJson, UserInfo.class);
        }
        
        return null;
    }
    
    private void sendUnauthorizedResponse(HttpServletResponse response, String message) 
            throws IOException {
        response.setStatus(HttpStatus.UNAUTHORIZED.value());
        response.setContentType("application/json;charset=UTF-8");
        
        Map<String, Object> result = new HashMap<>();
        result.put("code", 401);
        result.put("message", message);
        result.put("timestamp", System.currentTimeMillis());
        
        response.getWriter().write(JSON.toJSONString(result));
    }
    
    private void sendForbiddenResponse(HttpServletResponse response, String message) 
            throws IOException {
        response.setStatus(HttpStatus.FORBIDDEN.value());
        response.setContentType("application/json;charset=UTF-8");
        
        Map<String, Object> result = new HashMap<>();
        result.put("code", 403);
        result.put("message", message);
        result.put("timestamp", System.currentTimeMillis());
        
        response.getWriter().write(JSON.toJSONString(result));
    }
}

// 4. 自定义认证注解
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequireAuth {
    String[] role() default {};
    boolean required() default true;
}

// 5. 拦截器配置
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    
    @Autowired
    private AuthenticationInterceptor authenticationInterceptor;
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        
        // 认证拦截器 - 排除登录和静态资源
        registry.addInterceptor(authenticationInterceptor)
                .addPathPatterns("/**")
                .excludePathPatterns(
                    "/login", 
                    "/register", 
                    "/public/**", 
                    "/static/**", 
                    "/css/**", 
                    "/js/**", 
                    "/images/**"
                )
                .order(1); // 设置执行顺序
    }
}
```

**执行顺序和时机：**

```
客户端请求
    ↓
过滤器1.doFilter() - 前置处理
    ↓
过滤器2.doFilter() - 前置处理
    ↓
DispatcherServlet
    ↓
拦截器1.preHandle()
    ↓
拦截器2.preHandle()
    ↓
Controller方法执行
    ↓
拦截器2.postHandle()
    ↓
拦截器1.postHandle()
    ↓
视图渲染
    ↓
拦截器2.afterCompletion()
    ↓
拦截器1.afterCompletion()
    ↓
过滤器2.doFilter() - 后置处理
    ↓
过滤器1.doFilter() - 后置处理
    ↓
响应返回客户端
```

**使用场景对比：**

| 使用场景 | 推荐组件 | 原因 |
|----------|----------|------|
| **字符编码设置** | 过滤器 | 需要在所有处理之前设置 |
| **请求日志记录** | 过滤器 | 记录所有请求，包括静态资源 |
| **身份认证** | 拦截器 | 需要访问Spring容器，细粒度控制 |
| **权限检查** | 拦截器 | 基于注解的权限控制 |
| **性能监控** | 拦截器 | 需要访问具体的处理器方法信息 |
| **跨域处理** | 过滤器 | 需要在DispatcherServlet之前处理 |
| **数据压缩** | 过滤器 | 对响应内容进行压缩 |
| **缓存控制** | 拦截器 | 基于业务逻辑的缓存策略 |

**最佳实践：**

1. **过滤器适用于**：
   - 通用的请求预处理（编码、日志、跨域）
   - 不依赖Spring容器的功能
   - 需要处理所有请求的场景

2. **拦截器适用于**：
   - 需要访问Spring Bean的场景
   - 基于注解的细粒度控制
   - 业务相关的横切关注点

3. **性能考虑**：
   - 过滤器执行更早，性能开销更小
   - 拦截器功能更强大，但开销相对较大
   - 合理选择执行顺序，避免不必要的处理

### MyBatis相关

**8. MyBatis的工作原理？**

**答案：**

MyBatis是一个优秀的持久层框架，它支持自定义SQL、存储过程以及高级映射。MyBatis免除了几乎所有的JDBC代码以及设置参数和获取结果集的工作。

**MyBatis核心组件：**

1. **SqlSessionFactory**：会话工厂，用于创建SqlSession
2. **SqlSession**：会话，用于执行SQL语句
3. **Executor**：执行器，负责SQL语句的执行和缓存维护
4. **StatementHandler**：语句处理器，负责SQL语句的预编译和参数设置
5. **ParameterHandler**：参数处理器，负责参数映射
6. **ResultSetHandler**：结果集处理器，负责结果映射
7. **TypeHandler**：类型处理器，负责Java类型和JDBC类型的转换

**详细工作流程：**

```java
// 1. MyBatis配置文件 - mybatis-config.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" 
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 环境配置 -->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/test"/>
                <property name="username" value="root"/>
                <property name="password" value="password"/>
            </dataSource>
        </environment>
    </environments>
    
    <!-- 映射器配置 -->
    <mappers>
        <mapper resource="mapper/UserMapper.xml"/>
        <mapper class="com.example.mapper.UserMapper"/>
    </mappers>
</configuration>

// 2. 实体类
public class User {
    private Long id;
    private String username;
    private String email;
    private Date createTime;
    private Integer status;
    
    // 构造方法、getter、setter省略
    
    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", email='" + email + '\'' +
                ", createTime=" + createTime +
                ", status=" + status +
                '}';
    }
}

// 3. Mapper接口
public interface UserMapper {
    
    // 根据ID查询用户
    User selectById(Long id);
    
    // 根据条件查询用户列表
    List<User> selectByCondition(@Param("username") String username, 
                                @Param("status") Integer status);
    
    // 插入用户
    int insertUser(User user);
    
    // 更新用户
    int updateUser(User user);
    
    // 删除用户
    int deleteById(Long id);
    
    // 分页查询
    List<User> selectByPage(@Param("offset") int offset, 
                           @Param("limit") int limit);
    
    // 复杂查询
    List<User> selectByComplexCondition(UserQueryCondition condition);
}

// 4. Mapper XML文件 - UserMapper.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.UserMapper">
    
    <!-- 结果映射 -->
    <resultMap id="userResultMap" type="com.example.entity.User">
        <id property="id" column="id" jdbcType="BIGINT"/>
        <result property="username" column="username" jdbcType="VARCHAR"/>
        <result property="email" column="email" jdbcType="VARCHAR"/>
        <result property="createTime" column="create_time" jdbcType="TIMESTAMP"/>
        <result property="status" column="status" jdbcType="INTEGER"/>
    </resultMap>
    
    <!-- SQL片段 -->
    <sql id="userColumns">
        id, username, email, create_time, status
    </sql>
    
    <!-- 根据ID查询 -->
    <select id="selectById" parameterType="long" resultMap="userResultMap">
        SELECT <include refid="userColumns"/>
        FROM user 
        WHERE id = #{id}
    </select>
    
    <!-- 条件查询 -->
    <select id="selectByCondition" resultMap="userResultMap">
        SELECT <include refid="userColumns"/>
        FROM user 
        <where>
            <if test="username != null and username != ''">
                AND username LIKE CONCAT('%', #{username}, '%')
            </if>
            <if test="status != null">
                AND status = #{status}
            </if>
        </where>
        ORDER BY create_time DESC
    </select>
    
    <!-- 插入用户 -->
    <insert id="insertUser" parameterType="com.example.entity.User" 
            useGeneratedKeys="true" keyProperty="id">
        INSERT INTO user (username, email, create_time, status)
        VALUES (#{username}, #{email}, #{createTime}, #{status})
    </insert>
    
    <!-- 更新用户 -->
    <update id="updateUser" parameterType="com.example.entity.User">
        UPDATE user 
        <set>
            <if test="username != null and username != ''">
                username = #{username},
            </if>
            <if test="email != null and email != ''">
                email = #{email},
            </if>
            <if test="status != null">
                status = #{status},
            </if>
        </set>
        WHERE id = #{id}
    </update>
    
    <!-- 删除用户 -->
    <delete id="deleteById" parameterType="long">
        DELETE FROM user WHERE id = #{id}
    </delete>
    
    <!-- 分页查询 -->
    <select id="selectByPage" resultMap="userResultMap">
        SELECT <include refid="userColumns"/>
        FROM user 
        ORDER BY create_time DESC
        LIMIT #{offset}, #{limit}
    </select>
    
    <!-- 复杂条件查询 -->
    <select id="selectByComplexCondition" 
            parameterType="com.example.dto.UserQueryCondition" 
            resultMap="userResultMap">
        SELECT <include refid="userColumns"/>
        FROM user 
        <where>
            <choose>
                <when test="keyword != null and keyword != ''">
                    AND (username LIKE CONCAT('%', #{keyword}, '%') 
                         OR email LIKE CONCAT('%', #{keyword}, '%'))
                </when>
                <otherwise>
                    <if test="username != null and username != ''">
                        AND username = #{username}
                    </if>
                    <if test="email != null and email != ''">
                        AND email = #{email}
                    </if>
                </otherwise>
            </choose>
            <if test="status != null">
                AND status = #{status}
            </if>
            <if test="startDate != null">
                AND create_time >= #{startDate}
            </if>
            <if test="endDate != null">
                AND create_time <= #{endDate}
            </if>
            <if test="ids != null and ids.size() > 0">
                AND id IN
                <foreach collection="ids" item="id" open="(" close=")" separator=",">
                    #{id}
                </foreach>
            </if>
        </where>
        ORDER BY 
        <choose>
            <when test="orderBy != null and orderBy != ''">
                ${orderBy}
            </when>
            <otherwise>
                create_time DESC
            </otherwise>
        </choose>
    </select>
</mapper>

// 5. MyBatis工作原理演示
public class MyBatisWorkflowDemo {
    
    public static void main(String[] args) throws IOException {
        
        // 步骤1：读取配置文件，构建SqlSessionFactory
        System.out.println("=== MyBatis工作流程演示 ===");
        
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        
        // 使用SqlSessionFactoryBuilder构建SqlSessionFactory
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        System.out.println("1. SqlSessionFactory创建完成");
        
        // 步骤2：通过SqlSessionFactory创建SqlSession
        try (SqlSession sqlSession = sqlSessionFactory.openSession()) {
            System.out.println("2. SqlSession创建完成");
            
            // 步骤3：获取Mapper代理对象
            UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
            System.out.println("3. Mapper代理对象创建完成");
            
            // 步骤4：执行SQL语句
            System.out.println("4. 开始执行SQL语句");
            
            // 查询操作
            User user = userMapper.selectById(1L);
            System.out.println("   查询结果: " + user);
            
            // 插入操作
            User newUser = new User();
            newUser.setUsername("testuser");
            newUser.setEmail("test@example.com");
            newUser.setCreateTime(new Date());
            newUser.setStatus(1);
            
            int insertResult = userMapper.insertUser(newUser);
            System.out.println("   插入结果: " + insertResult + ", 生成的ID: " + newUser.getId());
            
            // 提交事务
            sqlSession.commit();
            System.out.println("5. 事务提交完成");
            
        } catch (Exception e) {
            System.err.println("执行过程中发生异常: " + e.getMessage());
        }
    }
}

// 6. 自定义TypeHandler示例
@MappedJdbcTypes(JdbcType.VARCHAR)
@MappedTypes(UserStatus.class)
public class UserStatusTypeHandler extends BaseTypeHandler<UserStatus> {
    
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, 
                                   UserStatus parameter, JdbcType jdbcType) throws SQLException {
        ps.setString(i, parameter.getCode());
    }
    
    @Override
    public UserStatus getNullableResult(ResultSet rs, String columnName) throws SQLException {
        String code = rs.getString(columnName);
        return UserStatus.fromCode(code);
    }
    
    @Override
    public UserStatus getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        String code = rs.getString(columnIndex);
        return UserStatus.fromCode(code);
    }
    
    @Override
    public UserStatus getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        String code = cs.getString(columnIndex);
        return UserStatus.fromCode(code);
    }
}

// 7. 自定义插件示例
@Intercepts({
    @Signature(type = Executor.class, method = "update", args = {MappedStatement.class, Object.class}),
    @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class})
})
public class SqlLogInterceptor implements Interceptor {
    
    private static final Logger logger = LoggerFactory.getLogger(SqlLogInterceptor.class);
    
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        
        MappedStatement mappedStatement = (MappedStatement) invocation.getArgs()[0];
        Object parameter = invocation.getArgs()[1];
        
        String sqlId = mappedStatement.getId();
        BoundSql boundSql = mappedStatement.getBoundSql(parameter);
        String sql = boundSql.getSql();
        
        long startTime = System.currentTimeMillis();
        
        try {
            // 执行原方法
            Object result = invocation.proceed();
            
            long endTime = System.currentTimeMillis();
            long executionTime = endTime - startTime;
            
            logger.info("SQL执行完成 - ID: {}, 耗时: {}ms, SQL: {}", 
                       sqlId, executionTime, sql.replaceAll("\\s+", " "));
            
            return result;
            
        } catch (Exception e) {
            long endTime = System.currentTimeMillis();
            long executionTime = endTime - startTime;
            
            logger.error("SQL执行异常 - ID: {}, 耗时: {}ms, SQL: {}, 异常: {}", 
                        sqlId, executionTime, sql.replaceAll("\\s+", " "), e.getMessage());
            
            throw e;
        }
    }
    
    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }
    
    @Override
    public void setProperties(Properties properties) {
        // 可以设置插件属性
    }
}
```

**MyBatis工作流程图：**

```
应用程序
    ↓
1. 读取配置文件
    ↓
2. SqlSessionFactoryBuilder
    ↓
3. SqlSessionFactory
    ↓
4. SqlSession
    ↓
5. Mapper代理对象
    ↓
6. Executor（执行器）
    ↓
7. StatementHandler（语句处理器）
    ↓
8. ParameterHandler（参数处理器）
    ↓
9. 执行SQL语句
    ↓
10. ResultSetHandler（结果处理器）
    ↓
11. TypeHandler（类型处理器）
    ↓
12. 返回结果
```

**核心组件详细说明：**

| 组件 | 作用 | 主要实现 |
|------|------|----------|
| **SqlSessionFactory** | 会话工厂，创建SqlSession | DefaultSqlSessionFactory |
| **SqlSession** | 会话，执行SQL的主要接口 | DefaultSqlSession |
| **Executor** | 执行器，负责SQL执行和缓存 | SimpleExecutor<br>ReuseExecutor<br>BatchExecutor |
| **StatementHandler** | 语句处理器，处理SQL语句 | PreparedStatementHandler<br>CallableStatementHandler |
| **ParameterHandler** | 参数处理器，设置SQL参数 | DefaultParameterHandler |
| **ResultSetHandler** | 结果处理器，处理结果集 | DefaultResultSetHandler |
| **TypeHandler** | 类型处理器，类型转换 | 各种具体类型处理器 |

**执行器类型对比：**

| 执行器类型 | 特点 | 适用场景 |
|------------|------|----------|
| **SimpleExecutor** | 每次执行都创建新的Statement | 默认执行器，适用于大多数场景 |
| **ReuseExecutor** | 重用Statement对象 | 频繁执行相同SQL的场景 |
| **BatchExecutor** | 批量执行SQL语句 | 大量数据插入/更新场景 |

**MyBatis的优势：**

1. **灵活的SQL控制**：支持动态SQL，可以根据条件生成不同的SQL
2. **强大的映射功能**：支持复杂的对象关系映射
3. **缓存机制**：提供一级和二级缓存，提高查询性能
4. **插件机制**：支持自定义插件，可以拦截SQL执行过程
5. **类型处理**：自动处理Java类型和JDBC类型的转换
6. **事务管理**：与Spring集成，支持声明式事务管理

**9. MyBatis中#{}和${}的区别？**

**答案：**

在MyBatis中，`#{}`和`${}`是两种不同的参数占位符，它们在SQL处理方式、安全性和使用场景上有重要区别。

**核心区别对比：**

| 特性 | #{} | ${} |
|------|-----|-----|
| **处理方式** | 预编译处理（PreparedStatement） | 字符串替换 |
| **SQL注入** | 防止SQL注入 | 存在SQL注入风险 |
| **引号处理** | 自动添加单引号 | 不添加引号 |
| **类型转换** | 自动类型转换 | 按字符串处理 |
| **性能** | 高（预编译缓存） | 低（每次重新编译） |
| **使用场景** | 参数值 | 动态表名、列名、排序 |

**详细代码示例：**

```java
// 1. 实体类和查询条件
public class User {
    private Long id;
    private String username;
    private String email;
    private Integer status;
    private Date createTime;
    
    // 构造方法、getter、setter省略
}

public class UserQueryCondition {
    private String username;
    private Integer status;
    private String orderBy;
    private String tableName;
    private List<String> columns;
    private String keyword;
    
    // 构造方法、getter、setter省略
}

// 2. Mapper接口
public interface UserMapper {
    
    // 使用#{}的安全查询
    List<User> selectByConditionSafe(@Param("username") String username, 
                                     @Param("status") Integer status);
    
    // 使用${}的动态查询（需要谨慎使用）
    List<User> selectByConditionDynamic(@Param("tableName") String tableName,
                                       @Param("orderBy") String orderBy);
    
    // 混合使用示例
    List<User> selectByMixedCondition(@Param("condition") UserQueryCondition condition);
    
    // 危险的${}使用示例（演示SQL注入风险）
    List<User> selectByUnsafeCondition(@Param("username") String username);
    
    // 安全的#{}使用示例
    List<User> selectBySafeCondition(@Param("username") String username);
    
    // 动态列查询
    List<Map<String, Object>> selectDynamicColumns(@Param("columns") String columns,
                                                   @Param("tableName") String tableName);
    
    // 分页查询（展示不同用法）
    List<User> selectByPage(@Param("offset") Integer offset,
                           @Param("limit") Integer limit,
                           @Param("orderBy") String orderBy);
}

// 3. Mapper XML配置
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.UserMapper">
    
    <!-- 使用#{}的安全查询 -->
    <select id="selectByConditionSafe" resultType="com.example.entity.User">
        SELECT id, username, email, status, create_time
        FROM user 
        WHERE 1=1
        <if test="username != null and username != ''">
            AND username = #{username}  <!-- 预编译，安全 -->
        </if>
        <if test="status != null">
            AND status = #{status}      <!-- 预编译，安全 -->
        </if>
        ORDER BY create_time DESC
    </select>
    
    <!-- 使用${}的动态查询（谨慎使用） -->
    <select id="selectByConditionDynamic" resultType="com.example.entity.User">
        SELECT id, username, email, status, create_time
        FROM ${tableName}              <!-- 字符串替换，用于动态表名 -->
        WHERE status = 1
        ORDER BY ${orderBy}            <!-- 字符串替换，用于动态排序 -->
    </select>
    
    <!-- 混合使用示例 -->
    <select id="selectByMixedCondition" 
            parameterType="com.example.dto.UserQueryCondition"
            resultType="com.example.entity.User">
        SELECT 
        <choose>
            <when test="condition.columns != null and condition.columns.size() > 0">
                <foreach collection="condition.columns" item="column" separator=",">
                    ${column}          <!-- 动态列名，使用${} -->
                </foreach>
            </when>
            <otherwise>
                id, username, email, status, create_time
            </otherwise>
        </choose>
        FROM ${condition.tableName}   <!-- 动态表名，使用${} -->
        <where>
            <if test="condition.username != null and condition.username != ''">
                AND username LIKE CONCAT('%', #{condition.username}, '%')  <!-- 参数值，使用#{} -->
            </if>
            <if test="condition.status != null">
                AND status = #{condition.status}  <!-- 参数值，使用#{} -->
            </if>
            <if test="condition.keyword != null and condition.keyword != ''">
                AND (username LIKE CONCAT('%', #{condition.keyword}, '%') 
                     OR email LIKE CONCAT('%', #{condition.keyword}, '%'))  <!-- 参数值，使用#{} -->
            </if>
        </where>
        <if test="condition.orderBy != null and condition.orderBy != ''">
            ORDER BY ${condition.orderBy}  <!-- 动态排序，使用${} -->
        </if>
    </select>
    
    <!-- 危险的${}使用示例（演示SQL注入风险） -->
    <select id="selectByUnsafeCondition" resultType="com.example.entity.User">
        SELECT id, username, email, status, create_time
        FROM user 
        WHERE username = '${username}'  <!-- 危险！存在SQL注入风险 -->
    </select>
    
    <!-- 安全的#{}使用示例 -->
    <select id="selectBySafeCondition" resultType="com.example.entity.User">
        SELECT id, username, email, status, create_time
        FROM user 
        WHERE username = #{username}    <!-- 安全！预编译处理 -->
    </select>
    
    <!-- 动态列查询 -->
    <select id="selectDynamicColumns" resultType="java.util.Map">
        SELECT ${columns}               <!-- 动态列名，必须使用${} -->
        FROM ${tableName}              <!-- 动态表名，必须使用${} -->
        WHERE status = 1
    </select>
    
    <!-- 分页查询（展示不同用法） -->
    <select id="selectByPage" resultType="com.example.entity.User">
        SELECT id, username, email, status, create_time
        FROM user 
        WHERE status = 1
        ORDER BY ${orderBy}            <!-- 动态排序字段，使用${} -->
        LIMIT #{offset}, #{limit}      <!-- 分页参数，使用#{} -->
    </select>
</mapper>

// 4. SQL注入风险演示
public class SqlInjectionDemo {
    
    public static void main(String[] args) {
        
        System.out.println("=== SQL注入风险演示 ===");
        
        // 正常查询
        String normalUsername = "john";
        System.out.println("正常查询用户名: " + normalUsername);
        
        // 使用#{}的安全查询（推荐）
        // 生成的SQL: SELECT * FROM user WHERE username = ?
        // 参数: ["john"]
        System.out.println("安全查询（#{}）: 使用预编译，参数单独传递");
        
        // 使用${}的危险查询
        // 生成的SQL: SELECT * FROM user WHERE username = 'john'
        System.out.println("字符串替换（${}）: 直接拼接到SQL中");
        
        System.out.println("\n=== SQL注入攻击示例 ===");
        
        // 恶意输入
        String maliciousInput = "admin' OR '1'='1";
        System.out.println("恶意输入: " + maliciousInput);
        
        // 使用#{}的安全处理
        // 生成的SQL: SELECT * FROM user WHERE username = ?
        // 参数: ["admin' OR '1'='1"]
        // 结果: 查询用户名为 "admin' OR '1'='1" 的用户（不存在）
        System.out.println("安全处理（#{}）: 恶意输入被当作普通字符串参数");
        
        // 使用${}的危险处理
        // 生成的SQL: SELECT * FROM user WHERE username = 'admin' OR '1'='1'
        // 结果: 返回所有用户（SQL注入成功）
        System.out.println("危险处理（${}）: 恶意输入被拼接到SQL中，导致SQL注入");
        
        System.out.println("\n=== 合法的${}使用场景 ===");
        
        // 动态表名
        String tableName = "user_2023";
        System.out.println("动态表名: " + tableName);
        System.out.println("必须使用${}: SELECT * FROM ${tableName}");
        
        // 动态排序
        String orderBy = "create_time DESC";
        System.out.println("动态排序: " + orderBy);
        System.out.println("必须使用${}: ORDER BY ${orderBy}");
        
        // 动态列名
        String columns = "id, username, email";
        System.out.println("动态列名: " + columns);
        System.out.println("必须使用${}: SELECT ${columns} FROM user");
    }
}
```

**安全性对比示例：**

```sql
-- 使用#{}（安全）
-- 原始SQL: SELECT * FROM user WHERE username = #{username}
-- 用户输入: admin' OR '1'='1
-- 实际执行: SELECT * FROM user WHERE username = ?
-- 参数: ["admin' OR '1'='1"]
-- 结果: 查找用户名为 "admin' OR '1'='1" 的用户（安全）

-- 使用${}（危险）
-- 原始SQL: SELECT * FROM user WHERE username = '${username}'
-- 用户输入: admin' OR '1'='1
-- 实际执行: SELECT * FROM user WHERE username = 'admin' OR '1'='1'
-- 结果: 返回所有用户（SQL注入成功）
```

**使用场景总结：**

**使用#{}的场景：**
- 所有的参数值（字符串、数字、日期等）
- WHERE条件中的参数
- INSERT、UPDATE语句中的值
- LIMIT分页参数

**使用${}的场景：**
- 动态表名：`FROM ${tableName}`
- 动态列名：`SELECT ${columns} FROM table`
- 动态排序：`ORDER BY ${orderBy}`
- 动态SQL片段（需要极其谨慎）

**安全建议：**

1. **默认使用#{}**：除非确实需要动态SQL结构，否则总是使用`#{}`
2. **输入验证**：使用`${}`时必须进行严格的输入验证
3. **白名单机制**：对于动态表名、列名，使用白名单验证
4. **避免用户输入**：永远不要将未验证的用户输入直接用于`${}`
5. **代码审查**：重点审查所有使用`${}`的地方

**10. MyBatis的缓存机制？**

**答案：**

MyBatis提供了两级缓存机制来提高查询性能，减少数据库访问次数。缓存机制是MyBatis性能优化的重要特性。

**缓存级别对比：**

| 特性 | 一级缓存 | 二级缓存 |
|------|----------|----------|
| **作用域** | SqlSession级别 | Mapper级别 |
| **默认状态** | 默认开启 | 需要手动开启 |
| **生命周期** | SqlSession生命周期 | 应用程序生命周期 |
| **共享范围** | 同一个SqlSession | 多个SqlSession |
| **存储位置** | 内存中 | 内存或磁盘 |
| **配置复杂度** | 无需配置 | 需要配置 |

**详细代码示例：**

```java
// 1. 一级缓存演示
public class FirstLevelCacheDemo {
    
    public static void main(String[] args) throws IOException {
        
        System.out.println("=== MyBatis一级缓存演示 ===");
        
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        
        // 同一个SqlSession内的缓存测试
        try (SqlSession sqlSession = sqlSessionFactory.openSession()) {
            
            UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
            
            System.out.println("第一次查询用户ID=1:");
            User user1 = userMapper.selectById(1L);
            System.out.println("查询结果: " + user1);
            System.out.println("执行了SQL查询");
            
            System.out.println("\n第二次查询用户ID=1:");
            User user2 = userMapper.selectById(1L);
            System.out.println("查询结果: " + user2);
            System.out.println("从一级缓存获取，未执行SQL");
            
            System.out.println("\n对象引用比较: " + (user1 == user2)); // true
            
            System.out.println("\n执行更新操作:");
            User updateUser = new User();
            updateUser.setId(1L);
            updateUser.setUsername("updated_user");
            userMapper.updateUser(updateUser);
            System.out.println("一级缓存被清空");
            
            System.out.println("\n第三次查询用户ID=1:");
            User user3 = userMapper.selectById(1L);
            System.out.println("查询结果: " + user3);
            System.out.println("重新执行了SQL查询");
        }
        
        // 不同SqlSession之间的缓存测试
        System.out.println("\n=== 不同SqlSession测试 ===");
        
        try (SqlSession sqlSession1 = sqlSessionFactory.openSession()) {
            UserMapper userMapper1 = sqlSession1.getMapper(UserMapper.class);
            System.out.println("SqlSession1查询用户ID=1:");
            User user1 = userMapper1.selectById(1L);
            System.out.println("查询结果: " + user1);
        }
        
        try (SqlSession sqlSession2 = sqlSessionFactory.openSession()) {
            UserMapper userMapper2 = sqlSession2.getMapper(UserMapper.class);
            System.out.println("\nSqlSession2查询用户ID=1:");
            User user2 = userMapper2.selectById(1L);
            System.out.println("查询结果: " + user2);
            System.out.println("不同SqlSession，重新执行SQL查询");
        }
    }
}

// 2. 二级缓存配置和演示
// mybatis-config.xml配置
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" 
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    
    <!-- 开启二级缓存 -->
    <settings>
        <setting name="cacheEnabled" value="true"/>
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="multipleResultSetsEnabled" value="true"/>
        <setting name="useColumnLabel" value="true"/>
        <setting name="useGeneratedKeys" value="false"/>
        <setting name="autoMappingBehavior" value="PARTIAL"/>
        <setting name="defaultExecutorType" value="SIMPLE"/>
        <setting name="defaultStatementTimeout" value="25"/>
        <setting name="defaultFetchSize" value="100"/>
        <setting name="safeRowBoundsEnabled" value="false"/>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <setting name="localCacheScope" value="SESSION"/>
        <setting name="jdbcTypeForNull" value="OTHER"/>
        <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
    </settings>
    
    <!-- 数据源配置省略 -->
    
</configuration>

// 3. Mapper XML中的二级缓存配置
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.UserMapper">
    
    <!-- 开启二级缓存 -->
    <cache 
        eviction="LRU"           <!-- 缓存回收策略：LRU、FIFO、SOFT、WEAK -->
        flushInterval="60000"    <!-- 刷新间隔：60秒 -->
        size="512"               <!-- 缓存对象数量 -->
        readOnly="false"         <!-- 是否只读 -->
        type="org.apache.ibatis.cache.impl.PerpetualCache"/>  <!-- 缓存实现类 -->
    
    <!-- 或者使用自定义缓存 -->
    <!--
    <cache type="com.example.cache.RedisCache">
        <property name="host" value="localhost"/>
        <property name="port" value="6379"/>
    </cache>
    -->
    
    <!-- 查询语句，使用二级缓存 -->
    <select id="selectById" parameterType="long" 
            resultType="com.example.entity.User" 
            useCache="true">  <!-- 明确使用缓存 -->
        SELECT id, username, email, status, create_time
        FROM user 
        WHERE id = #{id}
    </select>
    
    <!-- 查询语句，不使用二级缓存 -->
    <select id="selectByIdNoCache" parameterType="long" 
            resultType="com.example.entity.User" 
            useCache="false">  <!-- 不使用缓存 -->
        SELECT id, username, email, status, create_time
        FROM user 
        WHERE id = #{id}
    </select>
    
    <!-- 更新语句，会清空二级缓存 -->
    <update id="updateUser" parameterType="com.example.entity.User" 
            flushCache="true">  <!-- 清空缓存 -->
        UPDATE user 
        SET username = #{username}, email = #{email}
        WHERE id = #{id}
    </update>
    
    <!-- 插入语句，不清空缓存 -->
    <insert id="insertUser" parameterType="com.example.entity.User" 
            flushCache="false">  <!-- 不清空缓存 -->
        INSERT INTO user (username, email, status, create_time)
        VALUES (#{username}, #{email}, #{status}, #{createTime})
    </insert>
    
</mapper>

// 4. 实体类需要实现Serializable（二级缓存要求）
public class User implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    private Long id;
    private String username;
    private String email;
    private Integer status;
    private Date createTime;
    
    // 构造方法、getter、setter省略
    
    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", email='" + email + '\'' +
                ", status=" + status +
                ", createTime=" + createTime +
                '}';
    }
}

// 5. 二级缓存演示
public class SecondLevelCacheDemo {
    
    public static void main(String[] args) throws IOException {
        
        System.out.println("=== MyBatis二级缓存演示 ===");
        
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        
        // 第一个SqlSession查询
        try (SqlSession sqlSession1 = sqlSessionFactory.openSession()) {
            UserMapper userMapper1 = sqlSession1.getMapper(UserMapper.class);
            
            System.out.println("SqlSession1第一次查询用户ID=1:");
            User user1 = userMapper1.selectById(1L);
            System.out.println("查询结果: " + user1);
            System.out.println("执行了SQL查询，结果存入二级缓存");
            
            // 必须提交或关闭SqlSession，数据才会进入二级缓存
            sqlSession1.commit();
        }
        
        // 第二个SqlSession查询
        try (SqlSession sqlSession2 = sqlSessionFactory.openSession()) {
            UserMapper userMapper2 = sqlSession2.getMapper(UserMapper.class);
            
            System.out.println("\nSqlSession2查询用户ID=1:");
            User user2 = userMapper2.selectById(1L);
            System.out.println("查询结果: " + user2);
            System.out.println("从二级缓存获取，未执行SQL");
        }
        
        // 第三个SqlSession执行更新操作
        try (SqlSession sqlSession3 = sqlSessionFactory.openSession()) {
            UserMapper userMapper3 = sqlSession3.getMapper(UserMapper.class);
            
            System.out.println("\nSqlSession3执行更新操作:");
            User updateUser = new User();
            updateUser.setId(1L);
            updateUser.setUsername("updated_user");
            userMapper3.updateUser(updateUser);
            sqlSession3.commit();
            System.out.println("二级缓存被清空");
        }
        
        // 第四个SqlSession再次查询
        try (SqlSession sqlSession4 = sqlSessionFactory.openSession()) {
            UserMapper userMapper4 = sqlSession4.getMapper(UserMapper.class);
            
            System.out.println("\nSqlSession4查询用户ID=1:");
            User user4 = userMapper4.selectById(1L);
            System.out.println("查询结果: " + user4);
            System.out.println("重新执行了SQL查询");
        }
    }
}

// 6. 自定义Redis缓存实现
public class RedisCache implements Cache {
    
    private final String id;
    private RedisTemplate<String, Object> redisTemplate;
    
    public RedisCache(String id) {
        this.id = id;
        // 初始化Redis连接
        this.redisTemplate = SpringContextUtil.getBean(RedisTemplate.class);
    }
    
    @Override
    public String getId() {
        return id;
    }
    
    @Override
    public void putObject(Object key, Object value) {
        String redisKey = generateKey(key);
        redisTemplate.opsForValue().set(redisKey, value, 30, TimeUnit.MINUTES);
    }
    
    @Override
    public Object getObject(Object key) {
        String redisKey = generateKey(key);
        return redisTemplate.opsForValue().get(redisKey);
    }
    
    @Override
    public Object removeObject(Object key) {
        String redisKey = generateKey(key);
        Object value = redisTemplate.opsForValue().get(redisKey);
        redisTemplate.delete(redisKey);
        return value;
    }
    
    @Override
    public void clear() {
        Set<String> keys = redisTemplate.keys(id + ":*");
        if (keys != null && !keys.isEmpty()) {
            redisTemplate.delete(keys);
        }
    }
    
    @Override
    public int getSize() {
        Set<String> keys = redisTemplate.keys(id + ":*");
        return keys != null ? keys.size() : 0;
    }
    
    private String generateKey(Object key) {
        return id + ":" + key.toString();
    }
}

// 7. 缓存配置最佳实践
public class CacheConfigurationDemo {
    
    public static void main(String[] args) {
        
        System.out.println("=== 缓存配置最佳实践 ===");
        
        System.out.println("1. 一级缓存配置:");
        System.out.println("   - 默认开启，无需配置");
        System.out.println("   - 可通过localCacheScope设置作用域");
        System.out.println("   - SESSION: SqlSession级别（默认）");
        System.out.println("   - STATEMENT: 语句级别（禁用缓存）");
        
        System.out.println("\n2. 二级缓存配置:");
        System.out.println("   - 全局开关: cacheEnabled=true");
        System.out.println("   - Mapper级别: <cache/>标签");
        System.out.println("   - 语句级别: useCache属性");
        
        System.out.println("\n3. 缓存回收策略:");
        System.out.println("   - LRU: 最近最少使用（推荐）");
        System.out.println("   - FIFO: 先进先出");
        System.out.println("   - SOFT: 软引用");
        System.out.println("   - WEAK: 弱引用");
        
        System.out.println("\n4. 缓存失效策略:");
        System.out.println("   - 时间失效: flushInterval");
        System.out.println("   - 操作失效: flushCache=true");
        System.out.println("   - 手动清除: sqlSession.clearCache()");
    }
}
```

**缓存工作流程图：**

```
查询请求
    ↓
检查一级缓存
    ↓
命中？ → 是 → 返回结果
    ↓ 否
检查二级缓存
    ↓
命中？ → 是 → 存入一级缓存 → 返回结果
    ↓ 否
执行SQL查询
    ↓
存入一级缓存
    ↓
存入二级缓存
    ↓
返回结果
```

**缓存失效场景：**

| 操作类型 | 一级缓存 | 二级缓存 |
|----------|----------|----------|
| **SELECT** | 不影响 | 不影响 |
| **INSERT** | 清空 | 根据flushCache配置 |
| **UPDATE** | 清空 | 根据flushCache配置 |
| **DELETE** | 清空 | 根据flushCache配置 |
| **commit()** | 不影响 | 数据进入二级缓存 |
| **rollback()** | 清空 | 不影响 |
| **close()** | 清空 | 数据进入二级缓存 |

**使用建议：**

1. **一级缓存**：默认开启，适用于所有场景
2. **二级缓存**：适用于读多写少的场景
3. **分布式环境**：使用Redis等分布式缓存
4. **缓存穿透**：对于不存在的数据也要缓存
5. **缓存雪崩**：设置不同的过期时间
6. **缓存一致性**：及时清理过期数据

### 消息队列相关

**11. 消息队列的作用？**

**答案：**

消息队列（Message Queue）是一种应用程序间的通信方法，它在分布式系统中扮演着重要角色，提供异步通信机制。

**核心作用对比：**

| 作用 | 说明 | 应用场景 | 优势 |
|------|------|----------|------|
| **异步处理** | 发送方无需等待接收方处理完成 | 邮件发送、短信通知 | 提高响应速度 |
| **系统解耦** | 降低系统间的直接依赖关系 | 微服务架构 | 提高系统灵活性 |
| **削峰填谷** | 平滑处理突发流量 | 秒杀活动、促销 | 保护下游系统 |
| **可靠性保证** | 确保消息不丢失 | 订单处理、支付 | 数据一致性 |
| **负载均衡** | 分散处理压力 | 任务分发 | 提高处理能力 |
| **数据分发** | 一对多消息传递 | 事件通知 | 广播能力 |

**详细代码示例：**

```java
// 1. 异步处理示例 - 用户注册场景

// 传统同步处理方式
@RestController
public class UserControllerSync {
    
    @Autowired
    private UserService userService;
    
    @Autowired
    private EmailService emailService;
    
    @Autowired
    private SmsService smsService;
    
    @PostMapping("/register/sync")
    public ResponseEntity<String> registerSync(@RequestBody UserRegisterRequest request) {
        
        long startTime = System.currentTimeMillis();
        
        try {
            // 1. 保存用户信息（100ms）
            User user = userService.saveUser(request);
            System.out.println("用户保存完成: " + user.getId());
            
            // 2. 发送欢迎邮件（500ms）
            emailService.sendWelcomeEmail(user.getEmail());
            System.out.println("欢迎邮件发送完成");
            
            // 3. 发送短信验证码（300ms）
            smsService.sendVerificationCode(user.getPhone());
            System.out.println("短信验证码发送完成");
            
            // 4. 初始化用户积分（200ms）
            userService.initUserPoints(user.getId());
            System.out.println("用户积分初始化完成");
            
            long endTime = System.currentTimeMillis();
            System.out.println("总耗时: " + (endTime - startTime) + "ms"); // 约1100ms
            
            return ResponseEntity.ok("注册成功");
            
        } catch (Exception e) {
            return ResponseEntity.status(500).body("注册失败: " + e.getMessage());
        }
    }
}

// 使用消息队列的异步处理方式
@RestController
public class UserControllerAsync {
    
    @Autowired
    private UserService userService;
    
    @Autowired
    private MessageProducer messageProducer;
    
    @PostMapping("/register/async")
    public ResponseEntity<String> registerAsync(@RequestBody UserRegisterRequest request) {
        
        long startTime = System.currentTimeMillis();
        
        try {
            // 1. 保存用户信息（100ms）
            User user = userService.saveUser(request);
            System.out.println("用户保存完成: " + user.getId());
            
            // 2. 发送异步消息到队列
            UserRegisteredEvent event = new UserRegisteredEvent(
                user.getId(), user.getEmail(), user.getPhone()
            );
            messageProducer.sendUserRegisteredEvent(event);
            System.out.println("注册事件已发送到消息队列");
            
            long endTime = System.currentTimeMillis();
            System.out.println("总耗时: " + (endTime - startTime) + "ms"); // 约110ms
            
            return ResponseEntity.ok("注册成功");
            
        } catch (Exception e) {
            return ResponseEntity.status(500).body("注册失败: " + e.getMessage());
        }
    }
}

// 消息生产者
@Component
public class MessageProducer {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public void sendUserRegisteredEvent(UserRegisteredEvent event) {
        rabbitTemplate.convertAndSend(
            "user.exchange", 
            "user.registered", 
            event
        );
    }
}

// 消息消费者
@Component
public class UserEventConsumer {
    
    @Autowired
    private EmailService emailService;
    
    @Autowired
    private SmsService smsService;
    
    @Autowired
    private UserService userService;
    
    @RabbitListener(queues = "user.email.queue")
    public void handleEmailNotification(UserRegisteredEvent event) {
        try {
            emailService.sendWelcomeEmail(event.getEmail());
            System.out.println("欢迎邮件发送完成: " + event.getEmail());
        } catch (Exception e) {
            System.err.println("邮件发送失败: " + e.getMessage());
            // 可以重试或记录错误日志
        }
    }
    
    @RabbitListener(queues = "user.sms.queue")
    public void handleSmsNotification(UserRegisteredEvent event) {
        try {
            smsService.sendVerificationCode(event.getPhone());
            System.out.println("短信验证码发送完成: " + event.getPhone());
        } catch (Exception e) {
            System.err.println("短信发送失败: " + e.getMessage());
        }
    }
    
    @RabbitListener(queues = "user.points.queue")
    public void handlePointsInitialization(UserRegisteredEvent event) {
        try {
            userService.initUserPoints(event.getUserId());
            System.out.println("用户积分初始化完成: " + event.getUserId());
        } catch (Exception e) {
            System.err.println("积分初始化失败: " + e.getMessage());
        }
    }
}

// 2. 系统解耦示例 - 订单处理场景

// 紧耦合的订单处理
@Service
public class OrderServiceCoupled {
    
    @Autowired
    private InventoryService inventoryService;
    
    @Autowired
    private PaymentService paymentService;
    
    @Autowired
    private LogisticsService logisticsService;
    
    @Autowired
    private NotificationService notificationService;
    
    public void processOrder(Order order) {
        
        try {
            // 1. 扣减库存
            inventoryService.decreaseStock(order.getProductId(), order.getQuantity());
            
            // 2. 处理支付
            paymentService.processPayment(order.getPaymentInfo());
            
            // 3. 安排物流
            logisticsService.arrangeShipping(order);
            
            // 4. 发送通知
            notificationService.sendOrderConfirmation(order.getUserId());
            
            System.out.println("订单处理完成: " + order.getId());
            
        } catch (Exception e) {
            // 需要手动回滚所有操作
            System.err.println("订单处理失败，需要回滚: " + e.getMessage());
            rollbackOrder(order);
        }
    }
    
    private void rollbackOrder(Order order) {
        // 复杂的回滚逻辑
    }
}

// 使用消息队列解耦的订单处理
@Service
public class OrderServiceDecoupled {
    
    @Autowired
    private MessageProducer messageProducer;
    
    public void processOrder(Order order) {
        
        try {
            // 1. 保存订单
            order.setStatus(OrderStatus.CREATED);
            // 保存到数据库
            
            // 2. 发布订单创建事件
            OrderCreatedEvent event = new OrderCreatedEvent(
                order.getId(),
                order.getUserId(),
                order.getProductId(),
                order.getQuantity(),
                order.getPaymentInfo()
            );
            
            messageProducer.sendOrderCreatedEvent(event);
            
            System.out.println("订单创建事件已发布: " + order.getId());
            
        } catch (Exception e) {
            System.err.println("订单创建失败: " + e.getMessage());
        }
    }
}

// 3. 削峰填谷示例 - 秒杀场景

@RestController
public class SeckillController {
    
    @Autowired
    private MessageProducer messageProducer;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @PostMapping("/seckill/{productId}")
    public ResponseEntity<String> seckill(
            @PathVariable Long productId,
            @RequestParam Long userId) {
        
        try {
            // 1. 预检查（Redis）
            String stockKey = "seckill:stock:" + productId;
            Long remainingStock = redisTemplate.opsForValue().decrement(stockKey);
            
            if (remainingStock < 0) {
                // 恢复库存
                redisTemplate.opsForValue().increment(stockKey);
                return ResponseEntity.ok("商品已售罄");
            }
            
            // 2. 发送秒杀请求到消息队列
            SeckillRequest request = new SeckillRequest(productId, userId, System.currentTimeMillis());
            messageProducer.sendSeckillRequest(request);
            
            return ResponseEntity.ok("秒杀请求已提交，请稍后查看结果");
            
        } catch (Exception e) {
            return ResponseEntity.status(500).body("系统繁忙，请稍后重试");
        }
    }
}

@Component
public class SeckillConsumer {
    
    @Autowired
    private SeckillService seckillService;
    
    // 使用多个消费者并发处理，但控制处理速度
    @RabbitListener(queues = "seckill.queue", concurrency = "5-10")
    public void processSeckillRequest(SeckillRequest request) {
        
        try {
            // 限流处理，避免数据库压力过大
            Thread.sleep(100); // 模拟处理时间
            
            boolean success = seckillService.processSeckill(
                request.getProductId(), 
                request.getUserId()
            );
            
            if (success) {
                System.out.println("秒杀成功: 用户" + request.getUserId() + 
                                 "，商品" + request.getProductId());
            } else {
                System.out.println("秒杀失败: 用户" + request.getUserId() + 
                                 "，商品" + request.getProductId());
            }
            
        } catch (Exception e) {
            System.err.println("秒杀处理异常: " + e.getMessage());
        }
    }
}
```

**消息队列架构图：**

```
生产者应用
    ↓
消息队列中间件
    ↓
消费者应用

详细流程：
[生产者] → [Exchange] → [Queue] → [消费者]
    ↓           ↓         ↓         ↓
  发送消息    路由规则   存储消息   处理消息
```

**使用场景对比：**

| 场景 | 不使用消息队列 | 使用消息队列 |
|------|----------------|-------------|
| **用户注册** | 响应时间1100ms | 响应时间110ms |
| **订单处理** | 强耦合，难维护 | 松耦合，易扩展 |
| **秒杀活动** | 数据库压力大 | 流量平滑处理 |
| **系统故障** | 数据可能丢失 | 消息持久化保证 |
| **扩展性** | 修改影响全局 | 独立扩展服务 |

**消息队列优势总结：**

1. **性能提升**：异步处理提高响应速度
2. **系统解耦**：降低服务间依赖关系
3. **流量控制**：削峰填谷保护系统
4. **可靠性**：消息持久化和确认机制
5. **扩展性**：水平扩展消费者
6. **容错性**：单点故障不影响整体

**12. RabbitMQ的交换机类型？**

**答案：**

RabbitMQ中的交换机（Exchange）是消息路由的核心组件，负责接收生产者发送的消息并根据路由规则将消息分发到相应的队列。

**交换机类型对比：**

| 交换机类型 | 路由方式 | 使用场景 | 特点 |
|------------|----------|----------|------|
| **Direct** | 精确匹配routing key | 点对点消息传递 | 简单直接，性能高 |
| **Topic** | 通配符匹配routing key | 复杂路由规则 | 灵活性强，支持模式匹配 |
| **Fanout** | 广播到所有绑定队列 | 发布/订阅模式 | 无需routing key，速度最快 |
| **Headers** | 匹配消息头属性 | 复杂条件路由 | 功能强大，性能较低 |

**详细代码示例：**

```java
// 1. Direct Exchange（直连交换机）示例

@Configuration
public class DirectExchangeConfig {
    
    // 声明Direct交换机
    @Bean
    public DirectExchange directExchange() {
        return new DirectExchange("direct.exchange", true, false);
    }
    
    // 声明队列
    @Bean
    public Queue orderQueue() {
        return QueueBuilder.durable("order.queue").build();
    }
    
    @Bean
    public Queue paymentQueue() {
        return QueueBuilder.durable("payment.queue").build();
    }
    
    @Bean
    public Queue logisticsQueue() {
        return QueueBuilder.durable("logistics.queue").build();
    }
    
    // 绑定队列到交换机
    @Bean
    public Binding orderBinding() {
        return BindingBuilder
                .bind(orderQueue())
                .to(directExchange())
                .with("order.created");
    }
    
    @Bean
    public Binding paymentBinding() {
        return BindingBuilder
                .bind(paymentQueue())
                .to(directExchange())
                .with("payment.processed");
    }
    
    @Bean
    public Binding logisticsBinding() {
        return BindingBuilder
                .bind(logisticsQueue())
                .to(directExchange())
                .with("logistics.arranged");
    }
}

// Direct Exchange生产者
@Component
public class DirectExchangeProducer {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public void sendOrderMessage(String message) {
        rabbitTemplate.convertAndSend(
            "direct.exchange", 
            "order.created",  // 精确匹配routing key
            message
        );
        System.out.println("发送订单消息: " + message);
    }
    
    public void sendPaymentMessage(String message) {
        rabbitTemplate.convertAndSend(
            "direct.exchange", 
            "payment.processed", 
            message
        );
        System.out.println("发送支付消息: " + message);
    }
    
    public void sendLogisticsMessage(String message) {
        rabbitTemplate.convertAndSend(
            "direct.exchange", 
            "logistics.arranged", 
            message
        );
        System.out.println("发送物流消息: " + message);
    }
}

// Direct Exchange消费者
@Component
public class DirectExchangeConsumer {
    
    @RabbitListener(queues = "order.queue")
    public void handleOrderMessage(String message) {
        System.out.println("订单队列接收到消息: " + message);
    }
    
    @RabbitListener(queues = "payment.queue")
    public void handlePaymentMessage(String message) {
        System.out.println("支付队列接收到消息: " + message);
    }
    
    @RabbitListener(queues = "logistics.queue")
    public void handleLogisticsMessage(String message) {
        System.out.println("物流队列接收到消息: " + message);
    }
}

// 2. Topic Exchange（主题交换机）示例

@Configuration
public class TopicExchangeConfig {
    
    // 声明Topic交换机
    @Bean
    public TopicExchange topicExchange() {
        return new TopicExchange("topic.exchange", true, false);
    }
    
    // 声明队列
    @Bean
    public Queue userAllQueue() {
        return QueueBuilder.durable("user.all.queue").build();
    }
    
    @Bean
    public Queue userEmailQueue() {
        return QueueBuilder.durable("user.email.queue").build();
    }
    
    @Bean
    public Queue orderAllQueue() {
        return QueueBuilder.durable("order.all.queue").build();
    }
    
    @Bean
    public Queue criticalQueue() {
        return QueueBuilder.durable("critical.queue").build();
    }
    
    // 绑定队列到交换机（使用通配符）
    @Bean
    public Binding userAllBinding() {
        return BindingBuilder
                .bind(userAllQueue())
                .to(topicExchange())
                .with("user.*");  // 匹配user.开头的所有routing key
    }
    
    @Bean
    public Binding userEmailBinding() {
        return BindingBuilder
                .bind(userEmailQueue())
                .to(topicExchange())
                .with("user.email.*");  // 匹配user.email.开头的routing key
    }
    
    @Bean
    public Binding orderAllBinding() {
        return BindingBuilder
                .bind(orderAllQueue())
                .to(topicExchange())
                .with("order.#");  // 匹配order.开头的所有routing key（包括多级）
    }
    
    @Bean
    public Binding criticalBinding() {
        return BindingBuilder
                .bind(criticalQueue())
                .to(topicExchange())
                .with("*.critical");  // 匹配以.critical结尾的routing key
    }
}

// Topic Exchange生产者
@Component
public class TopicExchangeProducer {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public void sendMessage(String routingKey, String message) {
        rabbitTemplate.convertAndSend(
            "topic.exchange", 
            routingKey, 
            message
        );
        System.out.println("发送消息 [" + routingKey + "]: " + message);
    }
    
    // 演示不同的routing key
    public void demonstrateTopicRouting() {
        
        // 这些消息会被路由到不同的队列
        sendMessage("user.registered", "用户注册消息");           // → user.all.queue
        sendMessage("user.email.sent", "邮件发送消息");           // → user.all.queue, user.email.queue
        sendMessage("user.email.failed", "邮件发送失败");         // → user.all.queue, user.email.queue
        sendMessage("order.created", "订单创建消息");             // → order.all.queue
        sendMessage("order.payment.completed", "订单支付完成");   // → order.all.queue
        sendMessage("user.critical", "用户关键消息");            // → user.all.queue, critical.queue
        sendMessage("system.critical", "系统关键消息");          // → critical.queue
    }
}

// 3. Fanout Exchange（扇形交换机）示例

@Configuration
public class FanoutExchangeConfig {
    
    // 声明Fanout交换机
    @Bean
    public FanoutExchange fanoutExchange() {
        return new FanoutExchange("fanout.exchange", true, false);
    }
    
    // 声明队列
    @Bean
    public Queue emailNotificationQueue() {
        return QueueBuilder.durable("email.notification.queue").build();
    }
    
    @Bean
    public Queue smsNotificationQueue() {
        return QueueBuilder.durable("sms.notification.queue").build();
    }
    
    @Bean
    public Queue pushNotificationQueue() {
        return QueueBuilder.durable("push.notification.queue").build();
    }
    
    @Bean
    public Queue auditLogQueue() {
        return QueueBuilder.durable("audit.log.queue").build();
    }
    
    // 绑定队列到交换机（无需routing key）
    @Bean
    public Binding emailNotificationBinding() {
        return BindingBuilder
                .bind(emailNotificationQueue())
                .to(fanoutExchange());
    }
    
    @Bean
    public Binding smsNotificationBinding() {
        return BindingBuilder
                .bind(smsNotificationQueue())
                .to(fanoutExchange());
    }
    
    @Bean
    public Binding pushNotificationBinding() {
        return BindingBuilder
                .bind(pushNotificationQueue())
                .to(fanoutExchange());
    }
    
    @Bean
    public Binding auditLogBinding() {
        return BindingBuilder
                .bind(auditLogQueue())
                .to(fanoutExchange());
    }
}

// Fanout Exchange生产者
@Component
public class FanoutExchangeProducer {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public void broadcastNotification(String message) {
        // Fanout交换机会忽略routing key，广播到所有绑定的队列
        rabbitTemplate.convertAndSend(
            "fanout.exchange", 
            "",  // routing key被忽略
            message
        );
        System.out.println("广播通知消息: " + message);
    }
}

// Fanout Exchange消费者
@Component
public class FanoutExchangeConsumer {
    
    @RabbitListener(queues = "email.notification.queue")
    public void handleEmailNotification(String message) {
        System.out.println("邮件通知服务接收到: " + message);
        // 发送邮件通知
    }
    
    @RabbitListener(queues = "sms.notification.queue")
    public void handleSmsNotification(String message) {
        System.out.println("短信通知服务接收到: " + message);
        // 发送短信通知
    }
    
    @RabbitListener(queues = "push.notification.queue")
    public void handlePushNotification(String message) {
        System.out.println("推送通知服务接收到: " + message);
        // 发送推送通知
    }
    
    @RabbitListener(queues = "audit.log.queue")
    public void handleAuditLog(String message) {
        System.out.println("审计日志服务接收到: " + message);
        // 记录审计日志
    }
}

// 4. Headers Exchange（头交换机）示例

@Configuration
public class HeadersExchangeConfig {
    
    // 声明Headers交换机
    @Bean
    public HeadersExchange headersExchange() {
        return new HeadersExchange("headers.exchange", true, false);
    }
    
    // 声明队列
    @Bean
    public Queue highPriorityQueue() {
        return QueueBuilder.durable("high.priority.queue").build();
    }
    
    @Bean
    public Queue vipUserQueue() {
        return QueueBuilder.durable("vip.user.queue").build();
    }
    
    @Bean
    public Queue errorHandlingQueue() {
        return QueueBuilder.durable("error.handling.queue").build();
    }
    
    // 绑定队列到交换机（基于消息头）
    @Bean
    public Binding highPriorityBinding() {
        return BindingBuilder
                .bind(highPriorityQueue())
                .to(headersExchange())
                .where("priority").matches("high");
    }
    
    @Bean
    public Binding vipUserBinding() {
        Map<String, Object> headers = new HashMap<>();
        headers.put("userType", "VIP");
        headers.put("region", "CN");
        
        return BindingBuilder
                .bind(vipUserQueue())
                .to(headersExchange())
                .whereAll(headers).match();  // 所有头属性都必须匹配
    }
    
    @Bean
    public Binding errorHandlingBinding() {
        Map<String, Object> headers = new HashMap<>();
        headers.put("messageType", "error");
        headers.put("severity", "critical");
        
        return BindingBuilder
                .bind(errorHandlingQueue())
                .to(headersExchange())
                .whereAny(headers).match();  // 任一头属性匹配即可
    }
}

// Headers Exchange生产者
@Component
public class HeadersExchangeProducer {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public void sendMessageWithHeaders(String message, Map<String, Object> headers) {
        
        MessageProperties properties = new MessageProperties();
        headers.forEach(properties::setHeader);
        
        Message rabbitMessage = new Message(message.getBytes(), properties);
        
        rabbitTemplate.send("headers.exchange", "", rabbitMessage);
        
        System.out.println("发送消息: " + message + ", 头信息: " + headers);
    }
    
    public void demonstrateHeadersRouting() {
        
        // 高优先级消息
        Map<String, Object> highPriorityHeaders = new HashMap<>();
        highPriorityHeaders.put("priority", "high");
        sendMessageWithHeaders("高优先级任务", highPriorityHeaders);
        
        // VIP用户消息
        Map<String, Object> vipHeaders = new HashMap<>();
        vipHeaders.put("userType", "VIP");
        vipHeaders.put("region", "CN");
        sendMessageWithHeaders("VIP用户专属消息", vipHeaders);
        
        // 错误处理消息
        Map<String, Object> errorHeaders = new HashMap<>();
        errorHeaders.put("messageType", "error");
        errorHeaders.put("severity", "critical");
        errorHeaders.put("source", "payment-service");
        sendMessageWithHeaders("支付服务错误", errorHeaders);
    }
}
```

**交换机路由规则详解：**

```
Direct Exchange:
  routing key = "order.created" → 只路由到绑定了"order.created"的队列

Topic Exchange:
  routing key = "user.email.sent"
  - "user.*" → 匹配 ✓
  - "user.email.*" → 匹配 ✓
  - "*.email.*" → 匹配 ✓
  - "user.#" → 匹配 ✓
  - "order.*" → 不匹配 ✗

Fanout Exchange:
  忽略routing key，广播到所有绑定的队列

Headers Exchange:
  基于消息头属性匹配：
  - whereAll(): 所有指定的头属性都必须匹配
  - whereAny(): 任一指定的头属性匹配即可
  - where(key).matches(value): 指定头属性精确匹配
```

**性能对比：**

| 交换机类型 | 路由性能 | 内存占用 | 适用场景 |
|------------|----------|----------|----------|
| **Fanout** | 最高 | 最低 | 简单广播 |
| **Direct** | 高 | 低 | 精确路由 |
| **Topic** | 中等 | 中等 | 模式匹配 |
| **Headers** | 最低 | 最高 | 复杂条件 |

**选择建议：**

1. **Direct Exchange**：适用于简单的点对点消息传递
2. **Topic Exchange**：适用于需要灵活路由规则的场景
3. **Fanout Exchange**：适用于发布/订阅模式，需要广播消息
4. **Headers Exchange**：适用于复杂的路由条件，但性能较低

**13. 如何保证消息不丢失？**

**答案：**

消息丢失可能发生在三个阶段：生产者发送消息、消息在队列中存储、消费者处理消息。需要在每个阶段都采取相应的保障措施。

**消息丢失场景分析：**

| 丢失阶段 | 丢失原因 | 解决方案 | 实现方式 |
|----------|----------|----------|----------|
| **生产者** | 网络异常、服务器宕机 | 生产者确认机制 | Publisher Confirms |
| **队列存储** | 服务器宕机、磁盘故障 | 消息持久化 | Durable Queue + Persistent Message |
| **消费者** | 处理异常、服务宕机 | 消费者确认机制 | Manual ACK |
| **系统级** | 整体服务不可用 | 高可用部署 | 集群 + 镜像队列 |

**详细实现方案：**

```java
// 1. 生产者确认机制（Publisher Confirms）

@Configuration
public class RabbitMQReliabilityConfig {
    
    @Bean
    public RabbitTemplate reliableRabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate template = new RabbitTemplate(connectionFactory);
        
        // 启用生产者确认模式
        template.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
            @Override
            public void confirm(CorrelationData correlationData, boolean ack, String cause) {
                if (ack) {
                    System.out.println("消息发送成功: " + correlationData.getId());
                } else {
                    System.err.println("消息发送失败: " + correlationData.getId() + ", 原因: " + cause);
                    // 重试逻辑或记录失败日志
                    handleSendFailure(correlationData, cause);
                }
            }
        });
        
        // 启用消息返回机制（当消息无法路由到队列时）
        template.setReturnsCallback(new RabbitTemplate.ReturnsCallback() {
            @Override
            public void returnedMessage(ReturnedMessage returned) {
                System.err.println("消息路由失败: " + returned.getMessage());
                System.err.println("返回码: " + returned.getReplyCode());
                System.err.println("返回文本: " + returned.getReplyText());
                System.err.println("交换机: " + returned.getExchange());
                System.err.println("路由键: " + returned.getRoutingKey());
                
                // 处理路由失败的消息
                handleRoutingFailure(returned);
            }
        });
        
        // 设置为强制返回
        template.setMandatory(true);
        
        return template;
    }
    
    private void handleSendFailure(CorrelationData correlationData, String cause) {
        // 实现重试逻辑
        System.out.println("处理发送失败的消息: " + correlationData.getId());
    }
    
    private void handleRoutingFailure(ReturnedMessage returned) {
        // 处理路由失败的消息
        System.out.println("处理路由失败的消息");
    }
}

// 可靠的消息生产者
@Component
public class ReliableMessageProducer {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    private final AtomicLong messageIdGenerator = new AtomicLong(0);
    
    public void sendReliableMessage(String exchange, String routingKey, Object message) {
        
        // 生成唯一的消息ID
        String messageId = "msg_" + messageIdGenerator.incrementAndGet() + "_" + System.currentTimeMillis();
        CorrelationData correlationData = new CorrelationData(messageId);
        
        try {
            // 发送消息
            rabbitTemplate.convertAndSend(exchange, routingKey, message, correlationData);
            System.out.println("发送消息: " + messageId + ", 内容: " + message);
            
        } catch (Exception e) {
            System.err.println("发送消息异常: " + messageId + ", 错误: " + e.getMessage());
            // 可以实现重试机制
            retryMessage(exchange, routingKey, message, correlationData, 1);
        }
    }
    
    private void retryMessage(String exchange, String routingKey, Object message, 
                             CorrelationData correlationData, int retryCount) {
        
        if (retryCount > 3) {
            System.err.println("消息重试次数超限，放弃发送: " + correlationData.getId());
            return;
        }
        
        try {
            Thread.sleep(1000 * retryCount); // 延迟重试
            rabbitTemplate.convertAndSend(exchange, routingKey, message, correlationData);
            System.out.println("重试发送消息: " + correlationData.getId() + ", 第" + retryCount + "次");
            
        } catch (Exception e) {
            System.err.println("重试发送失败: " + correlationData.getId());
            retryMessage(exchange, routingKey, message, correlationData, retryCount + 1);
        }
    }
}

// 2. 消息和队列持久化配置

@Configuration
public class DurableQueueConfig {
    
    // 声明持久化交换机
    @Bean
    public DirectExchange durableExchange() {
        return new DirectExchange("durable.exchange", 
                                 true,    // durable: 持久化
                                 false);  // autoDelete: 不自动删除
    }
    
    // 声明持久化队列
    @Bean
    public Queue durableQueue() {
        return QueueBuilder
                .durable("durable.queue")  // 队列持久化
                .withArgument("x-message-ttl", 60000)  // 消息TTL
                .withArgument("x-max-length", 10000)   // 队列最大长度
                .build();
    }
    
    // 绑定
    @Bean
    public Binding durableBinding() {
        return BindingBuilder
                .bind(durableQueue())
                .to(durableExchange())
                .with("durable.routing.key");
    }
}

// 发送持久化消息
@Component
public class DurableMessageProducer {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public void sendDurableMessage(String message) {
        
        // 设置消息属性为持久化
        rabbitTemplate.convertAndSend(
            "durable.exchange", 
            "durable.routing.key", 
            message,
            messagePostProcessor -> {
                // 设置消息持久化
                messagePostProcessor.getMessageProperties().setDeliveryMode(MessageDeliveryMode.PERSISTENT);
                // 设置消息ID
                messagePostProcessor.getMessageProperties().setMessageId(UUID.randomUUID().toString());
                // 设置时间戳
                messagePostProcessor.getMessageProperties().setTimestamp(new Date());
                return messagePostProcessor;
            }
        );
        
        System.out.println("发送持久化消息: " + message);
    }
}

// 3. 消费者手动确认机制

@Component
public class ReliableMessageConsumer {
    
    @RabbitListener(queues = "durable.queue", ackMode = "MANUAL")
    public void handleMessage(String message, Channel channel, 
                             @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag) {
        
        try {
            // 模拟业务处理
            processBusinessLogic(message);
            
            // 业务处理成功，手动确认消息
            channel.basicAck(deliveryTag, false);
            System.out.println("消息处理成功并确认: " + message);
            
        } catch (BusinessException e) {
            // 业务异常，拒绝消息并重新入队
            try {
                System.err.println("业务处理失败，消息重新入队: " + message + ", 错误: " + e.getMessage());
                channel.basicNack(deliveryTag, false, true);
            } catch (IOException ioException) {
                System.err.println("消息确认失败: " + ioException.getMessage());
            }
            
        } catch (Exception e) {
            // 其他异常，拒绝消息且不重新入队
            try {
                System.err.println("消息处理异常，拒绝消息: " + message + ", 错误: " + e.getMessage());
                channel.basicNack(deliveryTag, false, false);
            } catch (IOException ioException) {
                System.err.println("消息拒绝失败: " + ioException.getMessage());
            }
        }
    }
    
    private void processBusinessLogic(String message) throws BusinessException {
        // 模拟业务处理逻辑
        if (message.contains("error")) {
            throw new BusinessException("业务处理失败");
        }
        
        // 模拟处理时间
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("业务处理完成: " + message);
    }
    
    // 自定义业务异常
    public static class BusinessException extends Exception {
        public BusinessException(String message) {
            super(message);
        }
    }
}

// 4. 集群和镜像队列配置

@Configuration
public class HighAvailabilityConfig {
    
    // 配置镜像队列
    @Bean
    public Queue mirrorQueue() {
        return QueueBuilder
                .durable("mirror.queue")
                // 设置队列为镜像队列，在所有节点上复制
                .withArgument("x-ha-policy", "all")
                // 设置镜像同步模式为自动
                .withArgument("x-ha-sync-mode", "automatic")
                // 设置镜像队列提升策略
                .withArgument("x-ha-promote-on-shutdown", "always")
                .build();
    }
    
    // 配置连接工厂的高可用性
    @Bean
    public CachingConnectionFactory connectionFactory() {
        CachingConnectionFactory factory = new CachingConnectionFactory();
        
        // 设置集群地址
        factory.setAddresses("192.168.1.100:5672,192.168.1.101:5672,192.168.1.102:5672");
        factory.setUsername("admin");
        factory.setPassword("admin123");
        factory.setVirtualHost("/");
        
        // 启用生产者确认
        factory.setPublisherConfirmType(CachingConnectionFactory.ConfirmType.CORRELATED);
        factory.setPublisherReturns(true);
        
        // 设置连接超时
        factory.setConnectionTimeout(30000);
        
        // 设置心跳间隔
        factory.setRequestedHeartBeat(60);
        
        // 启用自动恢复
        factory.getRabbitConnectionFactory().setAutomaticRecoveryEnabled(true);
        factory.getRabbitConnectionFactory().setNetworkRecoveryInterval(5000);
        
        return factory;
    }
}
```

**消息可靠性保障流程图：**

```
生产者 → [确认机制] → 交换机 → [持久化] → 队列 → [手动ACK] → 消费者
   ↓           ↓              ↓           ↓            ↓
重试机制    返回机制      镜像复制    死信队列    异常处理
   ↓           ↓              ↓           ↓            ↓
失败记录    备用路由      高可用      重试机制    业务回滚
```

**最佳实践总结：**

| 保障措施 | 实现方式 | 适用场景 | 性能影响 |
|----------|----------|----------|----------|
| **生产者确认** | Publisher Confirms | 所有场景 | 轻微 |
| **消息持久化** | Durable + Persistent | 重要消息 | 中等 |
| **手动确认** | Manual ACK | 业务处理 | 轻微 |
| **事务机制** | Channel Transaction | 强一致性 | 较高 |
| **集群部署** | Mirror Queue | 高可用 | 中等 |
| **死信队列** | DLX + DLQ | 异常处理 | 轻微 |

**配置建议：**

1. **开发环境**：启用基本的确认机制
2. **测试环境**：启用持久化和手动确认
3. **生产环境**：启用所有保障措施，包括集群和监控

**14. 如何处理消息重复消费？**

**答案：**

消息重复消费是分布式系统中常见的问题，可能由网络异常、消费者重启、消息重发等原因导致。解决方案的核心是实现幂等性处理。

**重复消费产生原因：**

| 原因类别 | 具体场景 | 解决方案 | 实现方式 |
|----------|----------|----------|----------|
| **网络异常** | ACK丢失、网络超时 | 消息去重 | 唯一ID + 状态记录 |
| **消费者异常** | 处理中断、服务重启 | 幂等设计 | 业务逻辑幂等化 |
| **消息重发** | 生产者重试、系统故障恢复 | 分布式锁 | Redis/数据库锁 |
| **并发处理** | 多实例同时消费 | 顺序消费 | 单线程/分区消费 |

**详细实现方案：**

```java
// 1. 基于唯一ID的消息去重

@Entity
@Table(name = "message_consume_record")
public class MessageConsumeRecord {
    
    @Id
    private String messageId;
    
    @Column(name = "consumer_group")
    private String consumerGroup;
    
    @Column(name = "consume_time")
    private LocalDateTime consumeTime;
    
    @Column(name = "status")
    @Enumerated(EnumType.STRING)
    private ConsumeStatus status;
    
    @Column(name = "retry_count")
    private Integer retryCount = 0;
    
    // 构造函数、getter、setter
    public MessageConsumeRecord() {}
    
    public MessageConsumeRecord(String messageId, String consumerGroup) {
        this.messageId = messageId;
        this.consumerGroup = consumerGroup;
        this.consumeTime = LocalDateTime.now();
        this.status = ConsumeStatus.PROCESSING;
    }
    
    public enum ConsumeStatus {
        PROCESSING,  // 处理中
        SUCCESS,     // 成功
        FAILED       // 失败
    }
    
    // getter和setter方法
    public String getMessageId() { return messageId; }
    public void setMessageId(String messageId) { this.messageId = messageId; }
    
    public String getConsumerGroup() { return consumerGroup; }
    public void setConsumerGroup(String consumerGroup) { this.consumerGroup = consumerGroup; }
    
    public LocalDateTime getConsumeTime() { return consumeTime; }
    public void setConsumeTime(LocalDateTime consumeTime) { this.consumeTime = consumeTime; }
    
    public ConsumeStatus getStatus() { return status; }
    public void setStatus(ConsumeStatus status) { this.status = status; }
    
    public Integer getRetryCount() { return retryCount; }
    public void setRetryCount(Integer retryCount) { this.retryCount = retryCount; }
}

@Repository
public interface MessageConsumeRecordRepository extends JpaRepository<MessageConsumeRecord, String> {
    
    Optional<MessageConsumeRecord> findByMessageIdAndConsumerGroup(String messageId, String consumerGroup);
    
    @Modifying
    @Query("UPDATE MessageConsumeRecord m SET m.status = :status, m.consumeTime = :consumeTime WHERE m.messageId = :messageId AND m.consumerGroup = :consumerGroup")
    int updateStatus(@Param("messageId") String messageId, 
                    @Param("consumerGroup") String consumerGroup,
                    @Param("status") MessageConsumeRecord.ConsumeStatus status,
                    @Param("consumeTime") LocalDateTime consumeTime);
}

// 消息去重服务
@Service
public class MessageDeduplicationService {
    
    @Autowired
    private MessageConsumeRecordRepository recordRepository;
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    private static final String REDIS_KEY_PREFIX = "msg:consume:";
    private static final int REDIS_EXPIRE_SECONDS = 3600; // 1小时过期
    
    /**
     * 检查消息是否已被消费
     */
    public boolean isMessageConsumed(String messageId, String consumerGroup) {
        
        // 1. 先检查Redis缓存（快速检查）
        String redisKey = REDIS_KEY_PREFIX + consumerGroup + ":" + messageId;
        String cached = redisTemplate.opsForValue().get(redisKey);
        if ("SUCCESS".equals(cached)) {
            return true;
        }
        
        // 2. 检查数据库记录
        Optional<MessageConsumeRecord> record = recordRepository
                .findByMessageIdAndConsumerGroup(messageId, consumerGroup);
        
        if (record.isPresent()) {
            MessageConsumeRecord.ConsumeStatus status = record.get().getStatus();
            if (status == MessageConsumeRecord.ConsumeStatus.SUCCESS) {
                // 更新Redis缓存
                redisTemplate.opsForValue().set(redisKey, "SUCCESS", REDIS_EXPIRE_SECONDS, TimeUnit.SECONDS);
                return true;
            } else if (status == MessageConsumeRecord.ConsumeStatus.PROCESSING) {
                // 检查是否超时（可能是僵尸记录）
                LocalDateTime consumeTime = record.get().getConsumeTime();
                if (consumeTime.isBefore(LocalDateTime.now().minusMinutes(10))) {
                    // 超时记录，允许重新处理
                    return false;
                }
                return true; // 正在处理中
            }
        }
        
        return false;
    }
    
    /**
     * 标记消息开始处理
     */
    @Transactional
    public boolean markMessageProcessing(String messageId, String consumerGroup) {
        try {
            MessageConsumeRecord record = new MessageConsumeRecord(messageId, consumerGroup);
            recordRepository.save(record);
            
            // 在Redis中标记为处理中
            String redisKey = REDIS_KEY_PREFIX + consumerGroup + ":" + messageId;
            redisTemplate.opsForValue().set(redisKey, "PROCESSING", REDIS_EXPIRE_SECONDS, TimeUnit.SECONDS);
            
            return true;
        } catch (DataIntegrityViolationException e) {
            // 主键冲突，说明已有记录
            return false;
        }
    }
    
    /**
     * 标记消息处理成功
     */
    @Transactional
    public void markMessageSuccess(String messageId, String consumerGroup) {
        recordRepository.updateStatus(messageId, consumerGroup, 
                MessageConsumeRecord.ConsumeStatus.SUCCESS, LocalDateTime.now());
        
        // 更新Redis缓存
        String redisKey = REDIS_KEY_PREFIX + consumerGroup + ":" + messageId;
        redisTemplate.opsForValue().set(redisKey, "SUCCESS", REDIS_EXPIRE_SECONDS, TimeUnit.SECONDS);
    }
    
    /**
     * 标记消息处理失败
     */
    @Transactional
    public void markMessageFailed(String messageId, String consumerGroup) {
        recordRepository.updateStatus(messageId, consumerGroup, 
                MessageConsumeRecord.ConsumeStatus.FAILED, LocalDateTime.now());
        
        // 删除Redis缓存，允许重试
        String redisKey = REDIS_KEY_PREFIX + consumerGroup + ":" + messageId;
        redisTemplate.delete(redisKey);
    }
}

// 2. 幂等性消息消费者

@Component
public class IdempotentMessageConsumer {
    
    @Autowired
    private MessageDeduplicationService deduplicationService;
    
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private PaymentService paymentService;
    
    private static final String CONSUMER_GROUP = "order-consumer-group";
    
    @RabbitListener(queues = "order.queue", ackMode = "MANUAL")
    public void handleOrderMessage(String messageBody, Channel channel, 
                                  @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag,
                                  @Header("messageId") String messageId) {
        
        try {
            // 1. 检查消息是否已被消费
            if (deduplicationService.isMessageConsumed(messageId, CONSUMER_GROUP)) {
                System.out.println("消息已被消费，跳过处理: " + messageId);
                channel.basicAck(deliveryTag, false);
                return;
            }
            
            // 2. 标记消息开始处理
            if (!deduplicationService.markMessageProcessing(messageId, CONSUMER_GROUP)) {
                System.out.println("消息正在被其他实例处理: " + messageId);
                channel.basicAck(deliveryTag, false);
                return;
            }
            
            // 3. 解析消息
            OrderMessage orderMessage = parseOrderMessage(messageBody);
            
            // 4. 幂等性业务处理
            processOrderIdempotently(orderMessage);
            
            // 5. 标记消息处理成功
            deduplicationService.markMessageSuccess(messageId, CONSUMER_GROUP);
            
            // 6. 确认消息
            channel.basicAck(deliveryTag, false);
            
            System.out.println("订单消息处理成功: " + messageId);
            
        } catch (BusinessException e) {
            // 业务异常，标记失败但不重试
            handleBusinessException(messageId, e, channel, deliveryTag);
            
        } catch (Exception e) {
            // 系统异常，可以重试
            handleSystemException(messageId, e, channel, deliveryTag);
        }
    }
    
    private void processOrderIdempotently(OrderMessage orderMessage) throws BusinessException {
        
        String orderId = orderMessage.getOrderId();
        
        // 检查订单是否已存在
        if (orderService.existsById(orderId)) {
            System.out.println("订单已存在，跳过创建: " + orderId);
            return;
        }
        
        // 创建订单（幂等操作）
        Order order = new Order();
        order.setOrderId(orderId);
        order.setUserId(orderMessage.getUserId());
        order.setProductId(orderMessage.getProductId());
        order.setQuantity(orderMessage.getQuantity());
        order.setAmount(orderMessage.getAmount());
        order.setStatus(OrderStatus.CREATED);
        
        orderService.createOrder(order);
        
        // 处理支付（幂等操作）
        if (orderMessage.getPaymentInfo() != null) {
            processPaymentIdempotently(orderMessage.getPaymentInfo());
        }
    }
    
    private void processPaymentIdempotently(PaymentInfo paymentInfo) throws BusinessException {
        
        String paymentId = paymentInfo.getPaymentId();
        
        // 检查支付记录是否已存在
        if (paymentService.existsById(paymentId)) {
            System.out.println("支付记录已存在，跳过处理: " + paymentId);
            return;
        }
        
        // 创建支付记录
        Payment payment = new Payment();
        payment.setPaymentId(paymentId);
        payment.setOrderId(paymentInfo.getOrderId());
        payment.setAmount(paymentInfo.getAmount());
        payment.setPaymentMethod(paymentInfo.getPaymentMethod());
        payment.setStatus(PaymentStatus.PENDING);
        
        paymentService.createPayment(payment);
    }
    
    private void handleBusinessException(String messageId, BusinessException e, 
                                       Channel channel, long deliveryTag) {
        try {
            System.err.println("业务异常，消息处理失败: " + messageId + ", 错误: " + e.getMessage());
            deduplicationService.markMessageFailed(messageId, CONSUMER_GROUP);
            
            // 业务异常不重试，直接确认消息
            channel.basicAck(deliveryTag, false);
            
        } catch (IOException ioException) {
            System.err.println("消息确认失败: " + ioException.getMessage());
        }
    }
    
    private void handleSystemException(String messageId, Exception e, 
                                     Channel channel, long deliveryTag) {
        try {
            System.err.println("系统异常，消息重新入队: " + messageId + ", 错误: " + e.getMessage());
            deduplicationService.markMessageFailed(messageId, CONSUMER_GROUP);
            
            // 系统异常，拒绝消息并重新入队
            channel.basicNack(deliveryTag, false, true);
            
        } catch (IOException ioException) {
            System.err.println("消息拒绝失败: " + ioException.getMessage());
        }
    }
    
    private OrderMessage parseOrderMessage(String messageBody) {
        // 解析JSON消息
        try {
            ObjectMapper mapper = new ObjectMapper();
            return mapper.readValue(messageBody, OrderMessage.class);
        } catch (Exception e) {
            throw new RuntimeException("消息解析失败: " + e.getMessage(), e);
        }
    }
}

// 3. 基于分布式锁的消息处理

@Component
public class DistributedLockMessageConsumer {
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    @Autowired
    private OrderService orderService;
    
    private static final String LOCK_KEY_PREFIX = "msg:lock:";
    private static final int LOCK_EXPIRE_SECONDS = 300; // 5分钟锁过期
    
    @RabbitListener(queues = "payment.queue", ackMode = "MANUAL")
    public void handlePaymentMessage(String messageBody, Channel channel, 
                                   @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag,
                                   @Header("messageId") String messageId) {
        
        String lockKey = LOCK_KEY_PREFIX + messageId;
        String lockValue = UUID.randomUUID().toString();
        
        try {
            // 尝试获取分布式锁
            Boolean lockAcquired = redisTemplate.opsForValue()
                    .setIfAbsent(lockKey, lockValue, LOCK_EXPIRE_SECONDS, TimeUnit.SECONDS);
            
            if (!lockAcquired) {
                System.out.println("获取锁失败，消息可能正在被处理: " + messageId);
                channel.basicAck(deliveryTag, false);
                return;
            }
            
            try {
                // 处理业务逻辑
                PaymentMessage paymentMessage = parsePaymentMessage(messageBody);
                processPayment(paymentMessage);
                
                // 处理成功，确认消息
                channel.basicAck(deliveryTag, false);
                System.out.println("支付消息处理成功: " + messageId);
                
            } finally {
                // 释放锁
                releaseLock(lockKey, lockValue);
            }
            
        } catch (Exception e) {
            System.err.println("支付消息处理异常: " + messageId + ", 错误: " + e.getMessage());
            try {
                // 异常情况下拒绝消息并重新入队
                channel.basicNack(deliveryTag, false, true);
                // 释放锁
                releaseLock(lockKey, lockValue);
            } catch (IOException ioException) {
                System.err.println("消息拒绝失败: " + ioException.getMessage());
            }
        }
    }
    
    private void releaseLock(String lockKey, String lockValue) {
        // 使用Lua脚本确保原子性释放锁
        String luaScript = 
            "if redis.call('get', KEYS[1]) == ARGV[1] then " +
            "    return redis.call('del', KEYS[1]) " +
            "else " +
            "    return 0 " +
            "end";
        
        redisTemplate.execute(new DefaultRedisScript<>(luaScript, Long.class), 
                            Collections.singletonList(lockKey), lockValue);
    }
    
    private PaymentMessage parsePaymentMessage(String messageBody) {
        // 解析支付消息
        try {
            ObjectMapper mapper = new ObjectMapper();
            return mapper.readValue(messageBody, PaymentMessage.class);
        } catch (Exception e) {
            throw new RuntimeException("支付消息解析失败: " + e.getMessage(), e);
        }
    }
    
    private void processPayment(PaymentMessage paymentMessage) {
        // 处理支付逻辑
        System.out.println("处理支付: " + paymentMessage.getPaymentId());
    }
}

// 4. 消息实体类

public class OrderMessage {
    private String orderId;
    private String userId;
    private String productId;
    private Integer quantity;
    private BigDecimal amount;
    private PaymentInfo paymentInfo;
    
    // 构造函数、getter、setter
    public OrderMessage() {}
    
    public String getOrderId() { return orderId; }
    public void setOrderId(String orderId) { this.orderId = orderId; }
    
    public String getUserId() { return userId; }
    public void setUserId(String userId) { this.userId = userId; }
    
    public String getProductId() { return productId; }
    public void setProductId(String productId) { this.productId = productId; }
    
    public Integer getQuantity() { return quantity; }
    public void setQuantity(Integer quantity) { this.quantity = quantity; }
    
    public BigDecimal getAmount() { return amount; }
    public void setAmount(BigDecimal amount) { this.amount = amount; }
    
    public PaymentInfo getPaymentInfo() { return paymentInfo; }
    public void setPaymentInfo(PaymentInfo paymentInfo) { this.paymentInfo = paymentInfo; }
}

public class PaymentInfo {
    private String paymentId;
    private String orderId;
    private BigDecimal amount;
    private String paymentMethod;
    
    // 构造函数、getter、setter
    public PaymentInfo() {}
    
    public String getPaymentId() { return paymentId; }
    public void setPaymentId(String paymentId) { this.paymentId = paymentId; }
    
    public String getOrderId() { return orderId; }
    public void setOrderId(String orderId) { this.orderId = orderId; }
    
    public BigDecimal getAmount() { return amount; }
    public void setAmount(BigDecimal amount) { this.amount = amount; }
    
    public String getPaymentMethod() { return paymentMethod; }
    public void setPaymentMethod(String paymentMethod) { this.paymentMethod = paymentMethod; }
}

public class PaymentMessage {
    private String paymentId;
    private String orderId;
    private BigDecimal amount;
    private String paymentMethod;
    private String status;
    
    // 构造函数、getter、setter
    public PaymentMessage() {}
    
    public String getPaymentId() { return paymentId; }
    public void setPaymentId(String paymentId) { this.paymentId = paymentId; }
    
    public String getOrderId() { return orderId; }
    public void setOrderId(String orderId) { this.orderId = orderId; }
    
    public BigDecimal getAmount() { return amount; }
    public void setAmount(BigDecimal amount) { this.amount = amount; }
    
    public String getPaymentMethod() { return paymentMethod; }
    public void setPaymentMethod(String paymentMethod) { this.paymentMethod = paymentMethod; }
    
    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }
}

// 自定义异常
public class BusinessException extends Exception {
    public BusinessException(String message) {
        super(message);
    }
    
    public BusinessException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

**防重复消费策略对比：**

| 策略 | 实现方式 | 优点 | 缺点 | 适用场景 |
|------|----------|------|------|----------|
| **唯一ID去重** | 数据库+Redis | 可靠性高，支持持久化 | 需要额外存储 | 对可靠性要求高 |
| **幂等性设计** | 业务逻辑改造 | 性能好，无额外开销 | 改造成本高 | 业务天然支持幂等 |
| **分布式锁** | Redis锁 | 实现简单，并发安全 | 可能死锁，性能一般 | 并发量不高的场景 |
| **状态机** | 状态流转控制 | 业务清晰，易维护 | 复杂度高 | 复杂业务流程 |

**最佳实践建议：**

1. **消息设计**：每条消息必须包含唯一ID
2. **存储选择**：Redis做缓存，数据库做持久化
3. **超时处理**：设置合理的锁超时和记录清理机制
4. **监控告警**：监控重复消费率和处理异常
5. **业务幂等**：从业务层面设计幂等操作
6. **分层防护**：结合多种策略，提供多层保障

**15. 死信队列的作用？**

**答案：**

死信队列（Dead Letter Queue，DLQ）是消息队列系统中用于处理无法正常消费的消息的特殊队列。当消息因为各种原因无法被正常处理时，会被转发到死信队列中，避免消息丢失并便于后续处理。

**死信产生的原因：**

| 原因类别 | 具体场景 | 触发条件 | 处理策略 |
|----------|----------|----------|----------|
| **消息过期** | TTL超时 | 消息在队列中停留时间超过设定值 | 设置合理TTL，及时处理 |
| **队列满载** | 队列长度超限 | 队列达到最大长度限制 | 扩容队列，优化消费速度 |
| **消费失败** | 业务异常 | 消费者处理失败且达到重试上限 | 业务逻辑优化，异常处理 |
| **消息拒绝** | 主动拒绝 | 消费者明确拒绝消息且不重新入队 | 消息格式校验，业务规则检查 |

**详细实现方案：**

```java
// 1. RabbitMQ死信队列配置

@Configuration
public class DeadLetterQueueConfig {
    
    // 业务队列
    public static final String BUSINESS_QUEUE = "business.queue";
    public static final String BUSINESS_EXCHANGE = "business.exchange";
    public static final String BUSINESS_ROUTING_KEY = "business.routing.key";
    
    // 死信队列
    public static final String DEAD_LETTER_QUEUE = "dead.letter.queue";
    public static final String DEAD_LETTER_EXCHANGE = "dead.letter.exchange";
    public static final String DEAD_LETTER_ROUTING_KEY = "dead.letter.routing.key";
    
    // 重试队列
    public static final String RETRY_QUEUE = "retry.queue";
    public static final String RETRY_EXCHANGE = "retry.exchange";
    public static final String RETRY_ROUTING_KEY = "retry.routing.key";
    
    /**
     * 业务交换机
     */
    @Bean
    public DirectExchange businessExchange() {
        return new DirectExchange(BUSINESS_EXCHANGE, true, false);
    }
    
    /**
     * 业务队列（配置死信交换机）
     */
    @Bean
    public Queue businessQueue() {
        return QueueBuilder.durable(BUSINESS_QUEUE)
                .withArgument("x-dead-letter-exchange", DEAD_LETTER_EXCHANGE)
                .withArgument("x-dead-letter-routing-key", DEAD_LETTER_ROUTING_KEY)
                .withArgument("x-message-ttl", 300000) // 5分钟TTL
                .withArgument("x-max-length", 10000)   // 最大队列长度
                .build();
    }
    
    /**
     * 业务队列绑定
     */
    @Bean
    public Binding businessBinding() {
        return BindingBuilder.bind(businessQueue())
                .to(businessExchange())
                .with(BUSINESS_ROUTING_KEY);
    }
    
    /**
     * 死信交换机
     */
    @Bean
    public DirectExchange deadLetterExchange() {
        return new DirectExchange(DEAD_LETTER_EXCHANGE, true, false);
    }
    
    /**
     * 死信队列
     */
    @Bean
    public Queue deadLetterQueue() {
        return QueueBuilder.durable(DEAD_LETTER_QUEUE).build();
    }
    
    /**
     * 死信队列绑定
     */
    @Bean
    public Binding deadLetterBinding() {
        return BindingBuilder.bind(deadLetterQueue())
                .to(deadLetterExchange())
                .with(DEAD_LETTER_ROUTING_KEY);
    }
    
    /**
     * 重试交换机
     */
    @Bean
    public DirectExchange retryExchange() {
        return new DirectExchange(RETRY_EXCHANGE, true, false);
    }
    
    /**
     * 重试队列（延迟重新投递到业务队列）
     */
    @Bean
    public Queue retryQueue() {
        return QueueBuilder.durable(RETRY_QUEUE)
                .withArgument("x-dead-letter-exchange", BUSINESS_EXCHANGE)
                .withArgument("x-dead-letter-routing-key", BUSINESS_ROUTING_KEY)
                .withArgument("x-message-ttl", 60000) // 1分钟后重新投递
                .build();
    }
    
    /**
     * 重试队列绑定
     */
    @Bean
    public Binding retryBinding() {
        return BindingBuilder.bind(retryQueue())
                .to(retryExchange())
                .with(RETRY_ROUTING_KEY);
    }
}

// 2. 业务消息消费者

@Component
public class BusinessMessageConsumer {
    
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    private static final int MAX_RETRY_COUNT = 3;
    
    @RabbitListener(queues = DeadLetterQueueConfig.BUSINESS_QUEUE, ackMode = "MANUAL")
    public void handleBusinessMessage(String messageBody, Channel channel,
                                    @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag,
                                    @Header(value = "x-retry-count", required = false) Integer retryCount,
                                    @Header("messageId") String messageId) {
        
        // 初始化重试次数
        if (retryCount == null) {
            retryCount = 0;
        }
        
        try {
            System.out.println("处理业务消息: " + messageId + ", 重试次数: " + retryCount);
            
            // 解析消息
            BusinessMessage businessMessage = parseBusinessMessage(messageBody);
            
            // 处理业务逻辑
            processBusinessMessage(businessMessage);
            
            // 处理成功，确认消息
            channel.basicAck(deliveryTag, false);
            System.out.println("业务消息处理成功: " + messageId);
            
        } catch (BusinessException e) {
            // 业务异常，不重试，直接进入死信队列
            handleBusinessException(messageId, e, channel, deliveryTag);
            
        } catch (SystemException e) {
            // 系统异常，可以重试
            handleSystemException(messageId, messageBody, retryCount, e, channel, deliveryTag);
            
        } catch (Exception e) {
            // 未知异常，进行重试
            handleUnknownException(messageId, messageBody, retryCount, e, channel, deliveryTag);
        }
    }
    
    private void processBusinessMessage(BusinessMessage message) throws BusinessException, SystemException {
        
        // 模拟业务处理
        if ("INVALID_ORDER".equals(message.getOrderId())) {
            throw new BusinessException("无效的订单ID: " + message.getOrderId());
        }
        
        if ("SYSTEM_ERROR".equals(message.getOrderId())) {
            throw new SystemException("系统暂时不可用");
        }
        
        // 正常业务处理
        orderService.processOrder(message.getOrderId(), message.getAmount());
    }
    
    private void handleBusinessException(String messageId, BusinessException e, 
                                       Channel channel, long deliveryTag) {
        try {
            System.err.println("业务异常，消息直接进入死信队列: " + messageId + ", 错误: " + e.getMessage());
            
            // 拒绝消息，不重新入队，直接进入死信队列
            channel.basicNack(deliveryTag, false, false);
            
        } catch (IOException ioException) {
            System.err.println("消息拒绝失败: " + ioException.getMessage());
        }
    }
    
    private void handleSystemException(String messageId, String messageBody, int retryCount,
                                     SystemException e, Channel channel, long deliveryTag) {
        try {
            if (retryCount < MAX_RETRY_COUNT) {
                System.err.println("系统异常，消息进入重试队列: " + messageId + ", 重试次数: " + (retryCount + 1));
                
                // 发送到重试队列
                sendToRetryQueue(messageBody, messageId, retryCount + 1);
                
                // 确认原消息
                channel.basicAck(deliveryTag, false);
                
            } else {
                System.err.println("系统异常，重试次数已达上限，消息进入死信队列: " + messageId);
                
                // 拒绝消息，进入死信队列
                channel.basicNack(deliveryTag, false, false);
            }
            
        } catch (IOException ioException) {
            System.err.println("消息处理失败: " + ioException.getMessage());
        }
    }
    
    private void handleUnknownException(String messageId, String messageBody, int retryCount,
                                      Exception e, Channel channel, long deliveryTag) {
        try {
            if (retryCount < MAX_RETRY_COUNT) {
                System.err.println("未知异常，消息进入重试队列: " + messageId + ", 重试次数: " + (retryCount + 1));
                
                // 发送到重试队列
                sendToRetryQueue(messageBody, messageId, retryCount + 1);
                
                // 确认原消息
                channel.basicAck(deliveryTag, false);
                
            } else {
                System.err.println("未知异常，重试次数已达上限，消息进入死信队列: " + messageId);
                
                // 拒绝消息，进入死信队列
                channel.basicNack(deliveryTag, false, false);
            }
            
        } catch (IOException ioException) {
            System.err.println("消息处理失败: " + ioException.getMessage());
        }
    }
    
    private void sendToRetryQueue(String messageBody, String messageId, int retryCount) {
        
        MessageProperties properties = new MessageProperties();
        properties.setHeader("messageId", messageId);
        properties.setHeader("x-retry-count", retryCount);
        properties.setDeliveryMode(MessageDeliveryMode.PERSISTENT);
        
        Message message = new Message(messageBody.getBytes(), properties);
        
        rabbitTemplate.send(DeadLetterQueueConfig.RETRY_EXCHANGE, 
                          DeadLetterQueueConfig.RETRY_ROUTING_KEY, message);
    }
    
    private BusinessMessage parseBusinessMessage(String messageBody) {
        try {
            ObjectMapper mapper = new ObjectMapper();
            return mapper.readValue(messageBody, BusinessMessage.class);
        } catch (Exception e) {
            throw new RuntimeException("消息解析失败: " + e.getMessage(), e);
        }
    }
}

// 3. 死信队列消费者

@Component
public class DeadLetterQueueConsumer {
    
    @Autowired
    private DeadLetterService deadLetterService;
    
    @Autowired
    private NotificationService notificationService;
    
    @RabbitListener(queues = DeadLetterQueueConfig.DEAD_LETTER_QUEUE, ackMode = "MANUAL")
    public void handleDeadLetterMessage(String messageBody, Channel channel,
                                      @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag,
                                      @Header("messageId") String messageId,
                                      @Header(value = "x-first-death-reason", required = false) String deathReason,
                                      @Header(value = "x-first-death-queue", required = false) String originalQueue) {
        
        try {
            System.out.println("处理死信消息: " + messageId + ", 原因: " + deathReason + ", 原队列: " + originalQueue);
            
            // 记录死信消息
            DeadLetterRecord record = new DeadLetterRecord();
            record.setMessageId(messageId);
            record.setMessageBody(messageBody);
            record.setDeathReason(deathReason);
            record.setOriginalQueue(originalQueue);
            record.setCreateTime(LocalDateTime.now());
            record.setStatus(DeadLetterStatus.PENDING);
            
            deadLetterService.saveDeadLetterRecord(record);
            
            // 分析死信原因并处理
            analyzeAndHandleDeadLetter(record);
            
            // 发送告警通知
            sendAlertNotification(record);
            
            // 确认消息
            channel.basicAck(deliveryTag, false);
            
            System.out.println("死信消息处理完成: " + messageId);
            
        } catch (Exception e) {
            System.err.println("死信消息处理异常: " + messageId + ", 错误: " + e.getMessage());
            try {
                // 死信处理失败，拒绝消息（避免无限循环）
                channel.basicNack(deliveryTag, false, false);
            } catch (IOException ioException) {
                System.err.println("死信消息拒绝失败: " + ioException.getMessage());
            }
        }
    }
    
    private void analyzeAndHandleDeadLetter(DeadLetterRecord record) {
        
        String deathReason = record.getDeathReason();
        
        switch (deathReason) {
            case "expired":
                // 消息过期
                handleExpiredMessage(record);
                break;
                
            case "maxlen":
                // 队列长度超限
                handleMaxLengthMessage(record);
                break;
                
            case "rejected":
                // 消息被拒绝
                handleRejectedMessage(record);
                break;
                
            default:
                // 其他原因
                handleOtherReasons(record);
                break;
        }
    }
    
    private void handleExpiredMessage(DeadLetterRecord record) {
        System.out.println("处理过期消息: " + record.getMessageId());
        
        // 可以选择重新发送到业务队列或标记为已处理
        record.setStatus(DeadLetterStatus.EXPIRED);
        record.setHandleTime(LocalDateTime.now());
        record.setHandleResult("消息已过期，不再处理");
        
        deadLetterService.updateDeadLetterRecord(record);
    }
    
    private void handleMaxLengthMessage(DeadLetterRecord record) {
        System.out.println("处理队列满载消息: " + record.getMessageId());
        
        // 队列满载，可以考虑重新发送
        record.setStatus(DeadLetterStatus.RETRY_PENDING);
        record.setHandleTime(LocalDateTime.now());
        record.setHandleResult("队列满载，等待重试");
        
        deadLetterService.updateDeadLetterRecord(record);
    }
    
    private void handleRejectedMessage(DeadLetterRecord record) {
        System.out.println("处理被拒绝消息: " + record.getMessageId());
        
        // 分析拒绝原因，可能需要人工介入
        record.setStatus(DeadLetterStatus.MANUAL_REVIEW);
        record.setHandleTime(LocalDateTime.now());
        record.setHandleResult("消息被拒绝，需要人工审核");
        
        deadLetterService.updateDeadLetterRecord(record);
    }
    
    private void handleOtherReasons(DeadLetterRecord record) {
        System.out.println("处理其他原因死信: " + record.getMessageId());
        
        record.setStatus(DeadLetterStatus.UNKNOWN);
        record.setHandleTime(LocalDateTime.now());
        record.setHandleResult("未知原因，需要进一步分析");
        
        deadLetterService.updateDeadLetterRecord(record);
    }
    
    private void sendAlertNotification(DeadLetterRecord record) {
        
        AlertMessage alert = new AlertMessage();
        alert.setTitle("死信队列告警");
        alert.setContent(String.format(
            "检测到死信消息:\n" +
            "消息ID: %s\n" +
            "死信原因: %s\n" +
            "原始队列: %s\n" +
            "时间: %s",
            record.getMessageId(),
            record.getDeathReason(),
            record.getOriginalQueue(),
            record.getCreateTime()
        ));
        alert.setLevel(AlertLevel.WARNING);
        
        notificationService.sendAlert(alert);
    }
}

// 4. 死信记录实体和服务

@Entity
@Table(name = "dead_letter_record")
public class DeadLetterRecord {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "message_id", unique = true)
    private String messageId;
    
    @Column(name = "message_body", columnDefinition = "TEXT")
    private String messageBody;
    
    @Column(name = "death_reason")
    private String deathReason;
    
    @Column(name = "original_queue")
    private String originalQueue;
    
    @Column(name = "create_time")
    private LocalDateTime createTime;
    
    @Column(name = "handle_time")
    private LocalDateTime handleTime;
    
    @Column(name = "status")
    @Enumerated(EnumType.STRING)
    private DeadLetterStatus status;
    
    @Column(name = "handle_result")
    private String handleResult;
    
    // 构造函数、getter、setter
    public DeadLetterRecord() {}
    
    public enum DeadLetterStatus {
        PENDING,        // 待处理
        EXPIRED,        // 已过期
        RETRY_PENDING,  // 等待重试
        MANUAL_REVIEW,  // 人工审核
        RESOLVED,       // 已解决
        UNKNOWN         // 未知状态
    }
    
    // getter和setter方法
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getMessageId() { return messageId; }
    public void setMessageId(String messageId) { this.messageId = messageId; }
    
    public String getMessageBody() { return messageBody; }
    public void setMessageBody(String messageBody) { this.messageBody = messageBody; }
    
    public String getDeathReason() { return deathReason; }
    public void setDeathReason(String deathReason) { this.deathReason = deathReason; }
    
    public String getOriginalQueue() { return originalQueue; }
    public void setOriginalQueue(String originalQueue) { this.originalQueue = originalQueue; }
    
    public LocalDateTime getCreateTime() { return createTime; }
    public void setCreateTime(LocalDateTime createTime) { this.createTime = createTime; }
    
    public LocalDateTime getHandleTime() { return handleTime; }
    public void setHandleTime(LocalDateTime handleTime) { this.handleTime = handleTime; }
    
    public DeadLetterStatus getStatus() { return status; }
    public void setStatus(DeadLetterStatus status) { this.status = status; }
    
    public String getHandleResult() { return handleResult; }
    public void setHandleResult(String handleResult) { this.handleResult = handleResult; }
}

@Service
public class DeadLetterService {
    
    @Autowired
    private DeadLetterRecordRepository deadLetterRepository;
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public void saveDeadLetterRecord(DeadLetterRecord record) {
        deadLetterRepository.save(record);
    }
    
    public void updateDeadLetterRecord(DeadLetterRecord record) {
        deadLetterRepository.save(record);
    }
    
    public List<DeadLetterRecord> findPendingRecords() {
        return deadLetterRepository.findByStatus(DeadLetterRecord.DeadLetterStatus.PENDING);
    }
    
    public void retryDeadLetterMessage(Long recordId) {
        
        Optional<DeadLetterRecord> recordOpt = deadLetterRepository.findById(recordId);
        if (!recordOpt.isPresent()) {
            throw new RuntimeException("死信记录不存在: " + recordId);
        }
        
        DeadLetterRecord record = recordOpt.get();
        
        try {
            // 重新发送到业务队列
            MessageProperties properties = new MessageProperties();
            properties.setHeader("messageId", record.getMessageId());
            properties.setHeader("x-retry-count", 0);
            properties.setDeliveryMode(MessageDeliveryMode.PERSISTENT);
            
            Message message = new Message(record.getMessageBody().getBytes(), properties);
            
            rabbitTemplate.send(DeadLetterQueueConfig.BUSINESS_EXCHANGE,
                              DeadLetterQueueConfig.BUSINESS_ROUTING_KEY, message);
            
            // 更新记录状态
            record.setStatus(DeadLetterRecord.DeadLetterStatus.RESOLVED);
            record.setHandleTime(LocalDateTime.now());
            record.setHandleResult("手动重试成功");
            
            deadLetterRepository.save(record);
            
            System.out.println("死信消息重试成功: " + record.getMessageId());
            
        } catch (Exception e) {
            System.err.println("死信消息重试失败: " + record.getMessageId() + ", 错误: " + e.getMessage());
            throw new RuntimeException("死信消息重试失败", e);
        }
    }
}

// 5. 消息实体类和异常

public class BusinessMessage {
    private String orderId;
    private String userId;
    private BigDecimal amount;
    private String productId;
    private LocalDateTime createTime;
    
    // 构造函数、getter、setter
    public BusinessMessage() {}
    
    public String getOrderId() { return orderId; }
    public void setOrderId(String orderId) { this.orderId = orderId; }
    
    public String getUserId() { return userId; }
    public void setUserId(String userId) { this.userId = userId; }
    
    public BigDecimal getAmount() { return amount; }
    public void setAmount(BigDecimal amount) { this.amount = amount; }
    
    public String getProductId() { return productId; }
    public void setProductId(String productId) { this.productId = productId; }
    
    public LocalDateTime getCreateTime() { return createTime; }
    public void setCreateTime(LocalDateTime createTime) { this.createTime = createTime; }
}

public class SystemException extends Exception {
    public SystemException(String message) {
        super(message);
    }
    
    public SystemException(String message, Throwable cause) {
        super(message, cause);
    }
}

public class AlertMessage {
    private String title;
    private String content;
    private AlertLevel level;
    private LocalDateTime createTime;
    
    public enum AlertLevel {
        INFO, WARNING, ERROR, CRITICAL
    }
    
    // 构造函数、getter、setter
    public AlertMessage() {
        this.createTime = LocalDateTime.now();
    }
    
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    
    public String getContent() { return content; }
    public void setContent(String content) { this.content = content; }
    
    public AlertLevel getLevel() { return level; }
    public void setLevel(AlertLevel level) { this.level = level; }
    
    public LocalDateTime getCreateTime() { return createTime; }
    public void setCreateTime(LocalDateTime createTime) { this.createTime = createTime; }
}
```

**死信队列处理流程图：**

```
[生产者] → [业务交换机] → [业务队列] → [消费者]
                                ↓ (失败/过期/拒绝)
                         [死信交换机] → [死信队列] → [死信处理器]
                                                        ↓
                                                  [记录+分析+告警]
                                                        ↓
                                              [人工处理/自动重试]
```

**死信队列最佳实践：**

| 实践项 | 建议 | 原因 | 实现方式 |
|--------|------|------|----------|
| **监控告警** | 实时监控死信数量 | 及时发现系统问题 | 监控系统+告警机制 |
| **分类处理** | 按死信原因分类处理 | 提高处理效率 | 不同处理策略 |
| **记录保存** | 持久化死信记录 | 便于问题追踪和分析 | 数据库存储 |
| **重试机制** | 支持手动/自动重试 | 恢复可处理的消息 | 重试队列+管理界面 |
| **容量规划** | 合理设置队列容量 | 避免死信队列过载 | 监控+扩容策略 |
| **定期清理** | 清理过期死信记录 | 节省存储空间 | 定时任务 |

**总结：**

死信队列是消息队列系统的重要组成部分，它提供了：
1. **消息保护**：防止消息丢失
2. **问题诊断**：便于分析系统问题
3. **恢复机制**：支持消息重新处理
4. **系统稳定**：避免问题消息影响正常业务
5. **监控告警**：及时发现和处理异常

## 学习建议

### 1. 学习路径
- **第一周**：Spring IoC和DI基础概念和实践
- **第二周**：Spring AOP和事务管理
- **第三周**：Spring Boot基础和自动配置
- **第四周**：Spring MVC和Web开发
- **第五周**：Spring Data JPA数据访问
- **第六周**：MyBatis持久层框架
- **第七周**：消息队列基础和RabbitMQ
- **第八周**：综合项目实践

### 2. 实践项目
- **博客系统**：使用Spring Boot + Spring MVC + MyBatis
- **电商系统**：集成消息队列处理订单
- **用户管理系统**：使用Spring Data JPA
- **微服务项目**：Spring Cloud基础应用

### 3. 学习资源

**官方文档**
- Spring Framework Reference Documentation
- Spring Boot Reference Guide
- MyBatis官方文档
- RabbitMQ官方教程

**推荐书籍**
- 《Spring实战》（第5版）
- 《Spring Boot实战》
- 《MyBatis从入门到精通》
- 《RabbitMQ实战指南》

**在线课程**
- 尚硅谷Spring全套教程
- 黑马程序员Spring Boot教程
- 慕课网Spring Cloud微服务

### 4. 学习重点
- **Spring核心**：IoC容器、AOP、事务管理
- **Spring Boot**：自动配置、starter机制、监控
- **数据访问**：JPA、MyBatis、事务处理
- **Web开发**：MVC模式、RESTful API、异常处理
- **消息队列**：异步处理、可靠性保证、性能优化

## 总结

第三阶段主要学习企业级Java开发中最重要的框架技术：

1. **Spring生态系统**：掌握IoC、AOP、事务等核心概念
2. **Spring Boot**：理解自动配置和快速开发
3. **数据访问层**：熟练使用JPA和MyBatis
4. **Web开发**：掌握Spring MVC和RESTful API开发
5. **消息队列**：理解异步处理和系统解耦

这些技术是Java后端开发的基础，也是面试的重点。通过大量的代码实践和项目经验，可以深入理解这些框架的设计思想和最佳实践。

**下一阶段预告**：第四阶段将学习分布式系统相关技术，包括微服务架构、缓存技术、数据库优化等高级主题。