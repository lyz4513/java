# 第五阶段：架构师层面 - 系统架构设计与技术决策

## 概述

第五阶段是Java学习的最高层次，专注于架构师所需的核心技能：系统架构设计、技术选型、性能优化、团队管理等。这个阶段需要具备全局视野和深度技术理解。

## 学习目标

- 掌握大型系统架构设计原则和方法
- 具备技术选型和架构决策能力
- 理解性能优化的系统性方法
- 掌握团队技术管理和代码质量保证
- 具备解决复杂技术问题的能力

## 1. 系统架构设计

### 1.1 架构设计原则

```java
import org.springframework.stereotype.Component;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.boot.context.properties.ConfigurationProperties;

import java.util.*;
import java.time.LocalDateTime;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 架构设计原则演示
 * 展示SOLID原则、设计模式在大型系统中的应用
 */
public class ArchitecturalPrinciples {
    
    // ==================== 单一职责原则 (SRP) ====================
    
    /**
     * 用户服务 - 只负责用户相关业务
     * 遵循单一职责原则，每个类只有一个变化的理由
     */
    @Component
    public class UserService {
        
        @Autowired
        private UserRepository userRepository;
        
        @Autowired
        private UserValidator userValidator;
        
        @Autowired
        private UserNotificationService notificationService;
        
        /**
         * 创建用户 - 只关注用户创建逻辑
         */
        public User createUser(CreateUserRequest request) {
            // 1. 验证用户数据
            userValidator.validate(request);
            
            // 2. 创建用户实体
            User user = User.builder()
                .username(request.getUsername())
                .email(request.getEmail())
                .password(request.getPassword())
                .createTime(LocalDateTime.now())
                .status(UserStatus.ACTIVE)
                .build();
            
            // 3. 保存用户
            User savedUser = userRepository.save(user);
            
            // 4. 发送通知（委托给专门的通知服务）
            notificationService.sendWelcomeNotification(savedUser);
            
            return savedUser;
        }
        
        /**
         * 获取用户信息
         */
        public User getUserById(Long userId) {
            return userRepository.findById(userId)
                .orElseThrow(() -> new UserNotFoundException("用户不存在: " + userId));
        }
    }
    
    /**
     * 用户验证器 - 专门负责用户数据验证
     */
    @Component
    public class UserValidator {
        
        public void validate(CreateUserRequest request) {
            validateUsername(request.getUsername());
            validateEmail(request.getEmail());
            validatePassword(request.getPassword());
        }
        
        private void validateUsername(String username) {
            if (username == null || username.trim().isEmpty()) {
                throw new ValidationException("用户名不能为空");
            }
            if (username.length() < 3 || username.length() > 20) {
                throw new ValidationException("用户名长度必须在3-20个字符之间");
            }
        }
        
        private void validateEmail(String email) {
            if (email == null || !email.matches("^[A-Za-z0-9+_.-]+@(.+)$")) {
                throw new ValidationException("邮箱格式不正确");
            }
        }
        
        private void validatePassword(String password) {
            if (password == null || password.length() < 8) {
                throw new ValidationException("密码长度不能少于8位");
            }
        }
    }
    
    // ==================== 开闭原则 (OCP) ====================
    
    /**
     * 支付处理器接口 - 对扩展开放，对修改关闭
     */
    public interface PaymentProcessor {
        PaymentResult process(PaymentRequest request);
        boolean supports(PaymentMethod method);
    }
    
    /**
     * 支付服务 - 通过策略模式实现开闭原则
     */
    @Component
    public class PaymentService {
        
        private final List<PaymentProcessor> processors;
        
        public PaymentService(List<PaymentProcessor> processors) {
            this.processors = processors;
        }
        
        /**
         * 处理支付 - 无需修改现有代码即可支持新的支付方式
         */
        public PaymentResult processPayment(PaymentRequest request) {
            PaymentProcessor processor = processors.stream()
                .filter(p -> p.supports(request.getMethod()))
                .findFirst()
                .orElseThrow(() -> new UnsupportedPaymentMethodException(
                    "不支持的支付方式: " + request.getMethod()));
            
            return processor.process(request);
        }
    }
    
    /**
     * 支付宝支付处理器
     */
    @Component
    public class AlipayProcessor implements PaymentProcessor {
        
        @Override
        public PaymentResult process(PaymentRequest request) {
            // 支付宝支付逻辑
            System.out.println("处理支付宝支付: " + request.getAmount());
            
            // 模拟支付处理
            return PaymentResult.builder()
                .success(true)
                .transactionId("ALIPAY_" + System.currentTimeMillis())
                .message("支付成功")
                .build();
        }
        
        @Override
        public boolean supports(PaymentMethod method) {
            return PaymentMethod.ALIPAY.equals(method);
        }
    }
    
    /**
     * 微信支付处理器
     */
    @Component
    public class WechatPayProcessor implements PaymentProcessor {
        
        @Override
        public PaymentResult process(PaymentRequest request) {
            // 微信支付逻辑
            System.out.println("处理微信支付: " + request.getAmount());
            
            return PaymentResult.builder()
                .success(true)
                .transactionId("WECHAT_" + System.currentTimeMillis())
                .message("支付成功")
                .build();
        }
        
        @Override
        public boolean supports(PaymentMethod method) {
            return PaymentMethod.WECHAT_PAY.equals(method);
        }
    }
    
    // ==================== 里氏替换原则 (LSP) ====================
    
    /**
     * 抽象缓存接口
     */
    public abstract class Cache {
        
        public abstract void put(String key, Object value);
        public abstract Object get(String key);
        public abstract void remove(String key);
        
        /**
         * 模板方法 - 定义缓存操作的通用流程
         */
        public final Object getOrCompute(String key, Supplier<Object> supplier) {
            Object value = get(key);
            if (value == null) {
                value = supplier.get();
                if (value != null) {
                    put(key, value);
                }
            }
            return value;
        }
    }
    
    /**
     * Redis缓存实现 - 可以完全替换父类
     */
    @Component
    public class RedisCache extends Cache {
        
        @Autowired
        private RedisTemplate<String, Object> redisTemplate;
        
        @Override
        public void put(String key, Object value) {
            redisTemplate.opsForValue().set(key, value, Duration.ofHours(1));
            System.out.println("Redis缓存存储: " + key);
        }
        
        @Override
        public Object get(String key) {
            Object value = redisTemplate.opsForValue().get(key);
            System.out.println("Redis缓存获取: " + key + " = " + value);
            return value;
        }
        
        @Override
        public void remove(String key) {
            redisTemplate.delete(key);
            System.out.println("Redis缓存删除: " + key);
        }
    }
    
    /**
     * 本地缓存实现 - 可以完全替换父类
     */
    @Component
    public class LocalCache extends Cache {
        
        private final Map<String, Object> cache = new ConcurrentHashMap<>();
        
        @Override
        public void put(String key, Object value) {
            cache.put(key, value);
            System.out.println("本地缓存存储: " + key);
        }
        
        @Override
        public Object get(String key) {
            Object value = cache.get(key);
            System.out.println("本地缓存获取: " + key + " = " + value);
            return value;
        }
        
        @Override
        public void remove(String key) {
            cache.remove(key);
            System.out.println("本地缓存删除: " + key);
        }
    }
    
    // ==================== 接口隔离原则 (ISP) ====================
    
    /**
     * 可读接口 - 只包含读操作
     */
    public interface Readable {
        Object read(String key);
        boolean exists(String key);
    }
    
    /**
     * 可写接口 - 只包含写操作
     */
    public interface Writable {
        void write(String key, Object value);
        void delete(String key);
    }
    
    /**
     * 可过期接口 - 只包含过期相关操作
     */
    public interface Expirable {
        void setExpire(String key, Duration duration);
        Duration getExpire(String key);
    }
    
    /**
     * 完整的存储服务 - 实现所有接口
     */
    @Component
    public class StorageService implements Readable, Writable, Expirable {
        
        @Autowired
        private RedisTemplate<String, Object> redisTemplate;
        
        // 实现Readable接口
        @Override
        public Object read(String key) {
            return redisTemplate.opsForValue().get(key);
        }
        
        @Override
        public boolean exists(String key) {
            return Boolean.TRUE.equals(redisTemplate.hasKey(key));
        }
        
        // 实现Writable接口
        @Override
        public void write(String key, Object value) {
            redisTemplate.opsForValue().set(key, value);
        }
        
        @Override
        public void delete(String key) {
            redisTemplate.delete(key);
        }
        
        // 实现Expirable接口
        @Override
        public void setExpire(String key, Duration duration) {
            redisTemplate.expire(key, duration);
        }
        
        @Override
        public Duration getExpire(String key) {
            Long expire = redisTemplate.getExpire(key);
            return expire != null ? Duration.ofSeconds(expire) : null;
        }
    }
    
    /**
     * 只读服务 - 只需要实现Readable接口
     */
    @Component
    public class ReadOnlyService {
        
        private final Readable readable;
        
        public ReadOnlyService(Readable readable) {
            this.readable = readable;
        }
        
        public Object getData(String key) {
            if (readable.exists(key)) {
                return readable.read(key);
            }
            return null;
        }
    }
    
    // ==================== 依赖倒置原则 (DIP) ====================
    
    /**
     * 消息发送接口 - 高层模块定义的抽象
     */
    public interface MessageSender {
        void sendMessage(String recipient, String message);
    }
    
    /**
     * 邮件发送实现 - 低层模块实现
     */
    @Component
    public class EmailSender implements MessageSender {
        
        @Override
        public void sendMessage(String recipient, String message) {
            System.out.println("发送邮件到: " + recipient);
            System.out.println("邮件内容: " + message);
            // 实际的邮件发送逻辑
        }
    }
    
    /**
     * 短信发送实现 - 低层模块实现
     */
    @Component
    public class SmsSender implements MessageSender {
        
        @Override
        public void sendMessage(String recipient, String message) {
            System.out.println("发送短信到: " + recipient);
            System.out.println("短信内容: " + message);
            // 实际的短信发送逻辑
        }
    }
    
    /**
     * 通知服务 - 高层模块，依赖抽象而不是具体实现
     */
    @Component
    public class NotificationService {
        
        private final List<MessageSender> messageSenders;
        
        public NotificationService(List<MessageSender> messageSenders) {
            this.messageSenders = messageSenders;
        }
        
        /**
         * 发送通知 - 依赖抽象接口，可以灵活切换实现
         */
        public void sendNotification(String recipient, String message, NotificationType type) {
            MessageSender sender = selectSender(type);
            sender.sendMessage(recipient, message);
        }
        
        private MessageSender selectSender(NotificationType type) {
            // 根据类型选择合适的发送器
            return messageSenders.stream()
                .filter(sender -> isCompatible(sender, type))
                .findFirst()
                .orElse(messageSenders.get(0)); // 默认使用第一个
        }
        
        private boolean isCompatible(MessageSender sender, NotificationType type) {
            if (type == NotificationType.EMAIL) {
                return sender instanceof EmailSender;
            } else if (type == NotificationType.SMS) {
                return sender instanceof SmsSender;
            }
            return false;
        }
    }
}

### 1.2 分层架构设计

```java
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;
import org.springframework.stereotype.Repository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;
import java.util.*;
import java.time.LocalDateTime;

/**
 * 分层架构设计演示
 * 展示经典的四层架构：表现层、业务层、持久层、基础设施层
 */
public class LayeredArchitecture {
    
    // ==================== 表现层 (Presentation Layer) ====================
    
    /**
     * 用户控制器 - 处理HTTP请求，负责数据转换和验证
     */
    @RestController
    @RequestMapping("/api/users")
    public class UserController {
        
        @Autowired
        private UserApplicationService userApplicationService;
        
        /**
         * 创建用户
         */
        @PostMapping
        public ResponseEntity<UserResponse> createUser(@Valid @RequestBody CreateUserRequest request) {
            try {
                UserDto userDto = userApplicationService.createUser(request);
                UserResponse response = UserResponse.fromDto(userDto);
                return ResponseEntity.ok(response);
            } catch (ValidationException e) {
                return ResponseEntity.badRequest().build();
            } catch (Exception e) {
                return ResponseEntity.internalServerError().build();
            }
        }
        
        /**
         * 获取用户详情
         */
        @GetMapping("/{id}")
        public ResponseEntity<UserResponse> getUser(@PathVariable Long id) {
            try {
                UserDto userDto = userApplicationService.getUserById(id);
                UserResponse response = UserResponse.fromDto(userDto);
                return ResponseEntity.ok(response);
            } catch (UserNotFoundException e) {
                return ResponseEntity.notFound().build();
            }
        }
        
        /**
         * 获取用户列表
         */
        @GetMapping
        public ResponseEntity<PageResponse<UserResponse>> getUsers(
                @RequestParam(defaultValue = "0") int page,
                @RequestParam(defaultValue = "20") int size,
                @RequestParam(required = false) String keyword) {
            
            PageRequest pageRequest = PageRequest.of(page, size);
            UserSearchCriteria criteria = UserSearchCriteria.builder()
                .keyword(keyword)
                .build();
            
            PageDto<UserDto> userPage = userApplicationService.searchUsers(criteria, pageRequest);
            PageResponse<UserResponse> response = PageResponse.fromDto(userPage, UserResponse::fromDto);
            
            return ResponseEntity.ok(response);
        }
    }
    
    // ==================== 应用层 (Application Layer) ====================
    
    /**
     * 用户应用服务 - 协调业务流程，处理事务边界
     */
    @Service
    @Transactional
    public class UserApplicationService {
        
        @Autowired
        private UserDomainService userDomainService;
        
        @Autowired
        private UserRepository userRepository;
        
        @Autowired
        private EmailService emailService;
        
        @Autowired
        private AuditService auditService;
        
        /**
         * 创建用户 - 协调多个领域服务完成业务流程
         */
        public UserDto createUser(CreateUserRequest request) {
            // 1. 验证业务规则
            userDomainService.validateUserCreation(request);
            
            // 2. 创建用户实体
            User user = userDomainService.createUser(
                request.getUsername(),
                request.getEmail(),
                request.getPassword()
            );
            
            // 3. 保存用户
            User savedUser = userRepository.save(user);
            
            // 4. 发送欢迎邮件（异步）
            CompletableFuture.runAsync(() -> {
                emailService.sendWelcomeEmail(savedUser.getEmail(), savedUser.getUsername());
            });
            
            // 5. 记录审计日志
            auditService.logUserCreation(savedUser.getId(), getCurrentUserId());
            
            return UserDto.fromEntity(savedUser);
        }
        
        /**
         * 获取用户信息
         */
        @Transactional(readOnly = true)
        public UserDto getUserById(Long userId) {
            User user = userRepository.findById(userId)
                .orElseThrow(() -> new UserNotFoundException("用户不存在: " + userId));
            
            return UserDto.fromEntity(user);
        }
        
        /**
         * 搜索用户
         */
        @Transactional(readOnly = true)
        public PageDto<UserDto> searchUsers(UserSearchCriteria criteria, PageRequest pageRequest) {
            Page<User> userPage = userRepository.findByCriteria(criteria, pageRequest);
            
            List<UserDto> userDtos = userPage.getContent().stream()
                .map(UserDto::fromEntity)
                .collect(Collectors.toList());
            
            return PageDto.<UserDto>builder()
                .content(userDtos)
                .totalElements(userPage.getTotalElements())
                .totalPages(userPage.getTotalPages())
                .currentPage(userPage.getNumber())
                .size(userPage.getSize())
                .build();
        }
        
        private Long getCurrentUserId() {
            // 从安全上下文获取当前用户ID
            return SecurityContextHolder.getContext().getAuthentication().getName();
        }
    }
    
    // ==================== 领域层 (Domain Layer) ====================
    
    /**
     * 用户领域服务 - 包含核心业务逻辑
     */
    @Service
    public class UserDomainService {
        
        @Autowired
        private UserRepository userRepository;
        
        @Autowired
        private PasswordEncoder passwordEncoder;
        
        /**
         * 验证用户创建的业务规则
         */
        public void validateUserCreation(CreateUserRequest request) {
            // 1. 检查用户名是否已存在
            if (userRepository.existsByUsername(request.getUsername())) {
                throw new BusinessException("用户名已存在: " + request.getUsername());
            }
            
            // 2. 检查邮箱是否已存在
            if (userRepository.existsByEmail(request.getEmail())) {
                throw new BusinessException("邮箱已存在: " + request.getEmail());
            }
            
            // 3. 验证密码强度
            validatePasswordStrength(request.getPassword());
        }
        
        /**
         * 创建用户实体
         */
        public User createUser(String username, String email, String password) {
            // 1. 加密密码
            String encodedPassword = passwordEncoder.encode(password);
            
            // 2. 生成用户ID
            String userId = generateUserId();
            
            // 3. 创建用户实体
            return User.builder()
                .userId(userId)
                .username(username)
                .email(email)
                .password(encodedPassword)
                .status(UserStatus.ACTIVE)
                .createTime(LocalDateTime.now())
                .lastLoginTime(null)
                .build();
        }
        
        /**
         * 验证密码强度
         */
        private void validatePasswordStrength(String password) {
            if (password.length() < 8) {
                throw new BusinessException("密码长度不能少于8位");
            }
            
            if (!password.matches(".*[A-Z].*")) {
                throw new BusinessException("密码必须包含大写字母");
            }
            
            if (!password.matches(".*[a-z].*")) {
                throw new BusinessException("密码必须包含小写字母");
            }
            
            if (!password.matches(".*[0-9].*")) {
                throw new BusinessException("密码必须包含数字");
            }
        }
        
        /**
         * 生成用户ID
         */
        private String generateUserId() {
            return "USER_" + System.currentTimeMillis() + "_" + 
                   UUID.randomUUID().toString().substring(0, 8).toUpperCase();
        }
    }
    
    /**
     * 用户实体 - 领域模型
     */
    @Entity
    @Table(name = "users")
    public class User {
        
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
        
        @Column(unique = true, nullable = false)
        private String userId;
        
        @Column(unique = true, nullable = false)
        private String username;
        
        @Column(unique = true, nullable = false)
        private String email;
        
        @Column(nullable = false)
        private String password;
        
        @Enumerated(EnumType.STRING)
        private UserStatus status;
        
        private LocalDateTime createTime;
        private LocalDateTime lastLoginTime;
        
        // 领域行为方法
        
        /**
         * 用户登录
         */
        public void login() {
            if (this.status != UserStatus.ACTIVE) {
                throw new BusinessException("用户状态异常，无法登录");
            }
            this.lastLoginTime = LocalDateTime.now();
        }
        
        /**
         * 禁用用户
         */
        public void disable() {
            this.status = UserStatus.DISABLED;
        }
        
        /**
         * 启用用户
         */
        public void enable() {
            this.status = UserStatus.ACTIVE;
        }
        
        /**
         * 检查是否为活跃用户
         */
        public boolean isActive() {
            return UserStatus.ACTIVE.equals(this.status);
        }
        
        // 构造器、getter、setter等
        public static UserBuilder builder() {
            return new UserBuilder();
        }
        
        // Builder模式实现
        public static class UserBuilder {
            private String userId;
            private String username;
            private String email;
            private String password;
            private UserStatus status;
            private LocalDateTime createTime;
            private LocalDateTime lastLoginTime;
            
            public UserBuilder userId(String userId) {
                this.userId = userId;
                return this;
            }
            
            public UserBuilder username(String username) {
                this.username = username;
                return this;
            }
            
            public UserBuilder email(String email) {
                this.email = email;
                return this;
            }
            
            public UserBuilder password(String password) {
                this.password = password;
                return this;
            }
            
            public UserBuilder status(UserStatus status) {
                this.status = status;
                return this;
            }
            
            public UserBuilder createTime(LocalDateTime createTime) {
                this.createTime = createTime;
                return this;
            }
            
            public UserBuilder lastLoginTime(LocalDateTime lastLoginTime) {
                this.lastLoginTime = lastLoginTime;
                return this;
            }
            
            public User build() {
                User user = new User();
                user.userId = this.userId;
                user.username = this.username;
                user.email = this.email;
                user.password = this.password;
                user.status = this.status;
                user.createTime = this.createTime;
                user.lastLoginTime = this.lastLoginTime;
                return user;
            }
        }
    }
    
    // ==================== 基础设施层 (Infrastructure Layer) ====================
    
    /**
     * 用户仓储实现 - 数据访问层
     */
    @Repository
    public interface UserRepository extends JpaRepository<User, Long> {
        
        Optional<User> findByUsername(String username);
        Optional<User> findByEmail(String email);
        boolean existsByUsername(String username);
        boolean existsByEmail(String email);
        
        @Query("SELECT u FROM User u WHERE " +
               "(:#{#criteria.keyword} IS NULL OR " +
               "u.username LIKE %:#{#criteria.keyword}% OR " +
               "u.email LIKE %:#{#criteria.keyword}%) AND " +
               "(:#{#criteria.status} IS NULL OR u.status = :#{#criteria.status})")
        Page<User> findByCriteria(@Param("criteria") UserSearchCriteria criteria, Pageable pageable);
    }
    
    /**
     * 邮件服务实现
     */
    @Service
    public class EmailService {
        
        @Autowired
        private JavaMailSender mailSender;
        
        @Autowired
        private TemplateEngine templateEngine;
        
        /**
         * 发送欢迎邮件
         */
        public void sendWelcomeEmail(String email, String username) {
            try {
                MimeMessage message = mailSender.createMimeMessage();
                MimeMessageHelper helper = new MimeMessageHelper(message, true, "UTF-8");
                
                helper.setTo(email);
                helper.setSubject("欢迎注册我们的平台");
                
                // 使用模板引擎生成邮件内容
                Context context = new Context();
                context.setVariable("username", username);
                String content = templateEngine.process("welcome-email", context);
                
                helper.setText(content, true);
                
                mailSender.send(message);
                System.out.println("欢迎邮件发送成功: " + email);
            } catch (Exception e) {
                System.err.println("邮件发送失败: " + e.getMessage());
            }
        }
    }
    
    /**
     * 审计服务实现
     */
    @Service
    public class AuditService {
        
        @Autowired
        private AuditLogRepository auditLogRepository;
        
        /**
         * 记录用户创建日志
         */
        public void logUserCreation(Long userId, Long operatorId) {
            AuditLog auditLog = AuditLog.builder()
                .action("USER_CREATED")
                .targetType("USER")
                .targetId(userId.toString())
                .operatorId(operatorId)
                .operateTime(LocalDateTime.now())
                .details("用户创建成功")
                .build();
            
            auditLogRepository.save(auditLog);
        }
    }
}

### 1.3 微服务架构设计

```java
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.cloud.gateway.filter.GatewayFilter;
import org.springframework.cloud.gateway.filter.factory.AbstractGatewayFilterFactory;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.*;

import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import io.github.resilience4j.retry.annotation.Retry;
import io.github.resilience4j.ratelimiter.annotation.RateLimiter;

import java.util.*;
import java.time.LocalDateTime;
import java.util.concurrent.CompletableFuture;

/**
 * 微服务架构设计演示
 * 展示服务拆分、服务间通信、API网关等核心概念
 */
public class MicroserviceArchitecture {
    
    // ==================== 用户服务 ====================
    
    /**
     * 用户服务API
     */
    @RestController
    @RequestMapping("/api/users")
    public class UserServiceController {
        
        @Autowired
        private UserService userService;
        
        @GetMapping("/{id}")
        public ResponseEntity<UserDto> getUser(@PathVariable Long id) {
            UserDto user = userService.getUserById(id);
            return ResponseEntity.ok(user);
        }
        
        @PostMapping
        public ResponseEntity<UserDto> createUser(@RequestBody CreateUserRequest request) {
            UserDto user = userService.createUser(request);
            return ResponseEntity.ok(user);
        }
        
        @GetMapping("/{id}/profile")
        public ResponseEntity<UserProfileDto> getUserProfile(@PathVariable Long id) {
            UserProfileDto profile = userService.getUserProfile(id);
            return ResponseEntity.ok(profile);
        }
    }
    
    // ==================== 订单服务 ====================
    
    /**
     * 订单服务API
     */
    @RestController
    @RequestMapping("/api/orders")
    public class OrderServiceController {
        
        @Autowired
        private OrderService orderService;
        
        @PostMapping
        public ResponseEntity<OrderDto> createOrder(@RequestBody CreateOrderRequest request) {
            OrderDto order = orderService.createOrder(request);
            return ResponseEntity.ok(order);
        }
        
        @GetMapping("/user/{userId}")
        public ResponseEntity<List<OrderDto>> getUserOrders(@PathVariable Long userId) {
            List<OrderDto> orders = orderService.getOrdersByUserId(userId);
            return ResponseEntity.ok(orders);
        }
    }
    
    /**
     * 订单服务实现 - 调用其他微服务
     */
    @Service
    public class OrderService {
        
        @Autowired
        private UserServiceClient userServiceClient;
        
        @Autowired
        private ProductServiceClient productServiceClient;
        
        @Autowired
        private PaymentServiceClient paymentServiceClient;
        
        @Autowired
        private OrderRepository orderRepository;
        
        /**
         * 创建订单 - 编排多个微服务
         */
        @Transactional
        public OrderDto createOrder(CreateOrderRequest request) {
            // 1. 验证用户信息
            UserDto user = userServiceClient.getUser(request.getUserId());
            if (user == null) {
                throw new BusinessException("用户不存在");
            }
            
            // 2. 验证商品信息和库存
            List<OrderItemDto> orderItems = new ArrayList<>();
            BigDecimal totalAmount = BigDecimal.ZERO;
            
            for (CreateOrderItemRequest itemRequest : request.getItems()) {
                ProductDto product = productServiceClient.getProduct(itemRequest.getProductId());
                if (product == null) {
                    throw new BusinessException("商品不存在: " + itemRequest.getProductId());
                }
                
                // 检查库存
                if (product.getStock() < itemRequest.getQuantity()) {
                    throw new BusinessException("库存不足: " + product.getName());
                }
                
                BigDecimal itemAmount = product.getPrice().multiply(BigDecimal.valueOf(itemRequest.getQuantity()));
                totalAmount = totalAmount.add(itemAmount);
                
                OrderItemDto orderItem = OrderItemDto.builder()
                    .productId(product.getId())
                    .productName(product.getName())
                    .price(product.getPrice())
                    .quantity(itemRequest.getQuantity())
                    .amount(itemAmount)
                    .build();
                
                orderItems.add(orderItem);
            }
            
            // 3. 创建订单
            Order order = Order.builder()
                .orderNumber(generateOrderNumber())
                .userId(request.getUserId())
                .totalAmount(totalAmount)
                .status(OrderStatus.PENDING)
                .createTime(LocalDateTime.now())
                .build();
            
            Order savedOrder = orderRepository.save(order);
            
            // 4. 异步处理库存扣减和支付
            CompletableFuture.runAsync(() -> {
                processOrderAsync(savedOrder, orderItems);
            });
            
            return OrderDto.fromEntity(savedOrder);
        }
        
        /**
         * 异步处理订单
         */
        private void processOrderAsync(Order order, List<OrderItemDto> orderItems) {
            try {
                // 1. 扣减库存
                for (OrderItemDto item : orderItems) {
                    productServiceClient.reduceStock(item.getProductId(), item.getQuantity());
                }
                
                // 2. 处理支付
                PaymentRequest paymentRequest = PaymentRequest.builder()
                    .orderId(order.getId())
                    .amount(order.getTotalAmount())
                    .userId(order.getUserId())
                    .build();
                
                PaymentResult paymentResult = paymentServiceClient.processPayment(paymentRequest);
                
                // 3. 更新订单状态
                if (paymentResult.isSuccess()) {
                    order.setStatus(OrderStatus.PAID);
                    order.setPayTime(LocalDateTime.now());
                } else {
                    order.setStatus(OrderStatus.FAILED);
                    // 回滚库存
                    rollbackStock(orderItems);
                }
                
                orderRepository.save(order);
                
            } catch (Exception e) {
                // 处理异常，回滚操作
                order.setStatus(OrderStatus.FAILED);
                orderRepository.save(order);
                rollbackStock(orderItems);
            }
        }
        
        private void rollbackStock(List<OrderItemDto> orderItems) {
            for (OrderItemDto item : orderItems) {
                try {
                    productServiceClient.increaseStock(item.getProductId(), item.getQuantity());
                } catch (Exception e) {
                    System.err.println("库存回滚失败: " + item.getProductId());
                }
            }
        }
        
        private String generateOrderNumber() {
            return "ORDER_" + System.currentTimeMillis();
        }
    }
    
    // ==================== 服务间通信 ====================
    
    /**
     * 用户服务客户端 - 使用Feign进行服务间调用
     */
    @FeignClient(name = "user-service", fallback = UserServiceFallback.class)
    public interface UserServiceClient {
        
        @GetMapping("/api/users/{id}")
        @CircuitBreaker(name = "user-service")
        @Retry(name = "user-service")
        UserDto getUser(@PathVariable("id") Long id);
        
        @GetMapping("/api/users/{id}/profile")
        @CircuitBreaker(name = "user-service")
        UserProfileDto getUserProfile(@PathVariable("id") Long id);
    }
    
    /**
     * 用户服务降级实现
     */
    @Component
    public class UserServiceFallback implements UserServiceClient {
        
        @Override
        public UserDto getUser(Long id) {
            System.out.println("用户服务降级: 返回默认用户信息");
            return UserDto.builder()
                .id(id)
                .username("默认用户")
                .email("default@example.com")
                .build();
        }
        
        @Override
        public UserProfileDto getUserProfile(Long id) {
            System.out.println("用户服务降级: 返回默认用户档案");
            return UserProfileDto.builder()
                .userId(id)
                .nickname("默认用户")
                .avatar("default-avatar.png")
                .build();
        }
    }
    
    /**
     * 商品服务客户端
     */
    @FeignClient(name = "product-service", fallback = ProductServiceFallback.class)
    public interface ProductServiceClient {
        
        @GetMapping("/api/products/{id}")
        @CircuitBreaker(name = "product-service")
        ProductDto getProduct(@PathVariable("id") Long id);
        
        @PostMapping("/api/products/{id}/reduce-stock")
        @CircuitBreaker(name = "product-service")
        void reduceStock(@PathVariable("id") Long productId, @RequestParam Integer quantity);
        
        @PostMapping("/api/products/{id}/increase-stock")
        void increaseStock(@PathVariable("id") Long productId, @RequestParam Integer quantity);
    }
    
    /**
     * 商品服务降级实现
     */
    @Component
    public class ProductServiceFallback implements ProductServiceClient {
        
        @Override
        public ProductDto getProduct(Long id) {
            throw new ServiceUnavailableException("商品服务暂时不可用");
        }
        
        @Override
        public void reduceStock(Long productId, Integer quantity) {
            throw new ServiceUnavailableException("库存服务暂时不可用");
        }
        
        @Override
        public void increaseStock(Long productId, Integer quantity) {
            System.out.println("库存回滚失败，需要人工处理: " + productId);
        }
    }
    
    /**
     * 支付服务客户端
     */
    @FeignClient(name = "payment-service", fallback = PaymentServiceFallback.class)
    public interface PaymentServiceClient {
        
        @PostMapping("/api/payments")
        @CircuitBreaker(name = "payment-service")
        @RateLimiter(name = "payment-service")
        PaymentResult processPayment(@RequestBody PaymentRequest request);
    }
    
    /**
     * 支付服务降级实现
     */
    @Component
    public class PaymentServiceFallback implements PaymentServiceClient {
        
        @Override
        public PaymentResult processPayment(PaymentRequest request) {
            System.out.println("支付服务降级: 支付失败");
            return PaymentResult.builder()
                .success(false)
                .message("支付服务暂时不可用，请稍后重试")
                .build();
        }
    }
    
    // ==================== API网关 ====================
    
    /**
     * 自定义网关过滤器 - 请求日志记录
     */
    @Component
    public class RequestLoggingGatewayFilterFactory 
            extends AbstractGatewayFilterFactory<RequestLoggingGatewayFilterFactory.Config> {
        
        public RequestLoggingGatewayFilterFactory() {
            super(Config.class);
        }
        
        @Override
        public GatewayFilter apply(Config config) {
            return (exchange, chain) -> {
                ServerHttpRequest request = exchange.getRequest();
                
                // 记录请求信息
                System.out.println("请求路径: " + request.getPath());
                System.out.println("请求方法: " + request.getMethod());
                System.out.println("请求头: " + request.getHeaders());
                System.out.println("请求时间: " + LocalDateTime.now());
                
                // 继续执行过滤器链
                return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                    ServerHttpResponse response = exchange.getResponse();
                    System.out.println("响应状态: " + response.getStatusCode());
                    System.out.println("响应时间: " + LocalDateTime.now());
                }));
            };
        }
        
        public static class Config {
            // 配置属性
        }
    }
    
    /**
     * 认证过滤器
     */
    @Component
    public class AuthenticationGatewayFilterFactory 
            extends AbstractGatewayFilterFactory<AuthenticationGatewayFilterFactory.Config> {
        
        @Autowired
        private JwtTokenUtil jwtTokenUtil;
        
        public AuthenticationGatewayFilterFactory() {
            super(Config.class);
        }
        
        @Override
        public GatewayFilter apply(Config config) {
            return (exchange, chain) -> {
                ServerHttpRequest request = exchange.getRequest();
                
                // 检查是否需要认证
                if (isPublicPath(request.getPath().toString())) {
                    return chain.filter(exchange);
                }
                
                // 获取Authorization头
                String authHeader = request.getHeaders().getFirst("Authorization");
                if (authHeader == null || !authHeader.startsWith("Bearer ")) {
                    return handleUnauthorized(exchange);
                }
                
                // 验证JWT令牌
                String token = authHeader.substring(7);
                try {
                    if (!jwtTokenUtil.validateToken(token)) {
                        return handleUnauthorized(exchange);
                    }
                    
                    // 添加用户信息到请求头
                    String userId = jwtTokenUtil.getUserIdFromToken(token);
                    ServerHttpRequest modifiedRequest = request.mutate()
                        .header("X-User-Id", userId)
                        .build();
                    
                    return chain.filter(exchange.mutate().request(modifiedRequest).build());
                    
                } catch (Exception e) {
                    return handleUnauthorized(exchange);
                }
            };
        }
        
        private boolean isPublicPath(String path) {
            List<String> publicPaths = Arrays.asList(
                "/api/auth/login",
                "/api/auth/register",
                "/api/health",
                "/api/docs"
            );
            return publicPaths.stream().anyMatch(path::startsWith);
        }
        
        private Mono<Void> handleUnauthorized(ServerWebExchange exchange) {
            ServerHttpResponse response = exchange.getResponse();
            response.setStatusCode(HttpStatus.UNAUTHORIZED);
            return response.setComplete();
        }
        
        public static class Config {
            // 配置属性
        }
    }
    
    /**
     * 限流过滤器
     */
    @Component
    public class RateLimitingGatewayFilterFactory 
            extends AbstractGatewayFilterFactory<RateLimitingGatewayFilterFactory.Config> {
        
        @Autowired
        private RedisTemplate<String, String> redisTemplate;
        
        public RateLimitingGatewayFilterFactory() {
            super(Config.class);
        }
        
        @Override
        public GatewayFilter apply(Config config) {
            return (exchange, chain) -> {
                ServerHttpRequest request = exchange.getRequest();
                
                // 获取客户端IP
                String clientIp = getClientIp(request);
                String key = "rate_limit:" + clientIp;
                
                // 使用Redis实现滑动窗口限流
                try {
                    String count = redisTemplate.opsForValue().get(key);
                    int currentCount = count != null ? Integer.parseInt(count) : 0;
                    
                    if (currentCount >= config.getLimit()) {
                        return handleRateLimitExceeded(exchange);
                    }
                    
                    // 增加计数
                    redisTemplate.opsForValue().increment(key);
                    redisTemplate.expire(key, Duration.ofSeconds(config.getWindowSize()));
                    
                    return chain.filter(exchange);
                    
                } catch (Exception e) {
                    // 限流失败时允许请求通过
                    return chain.filter(exchange);
                }
            };
        }
        
        private String getClientIp(ServerHttpRequest request) {
            String xForwardedFor = request.getHeaders().getFirst("X-Forwarded-For");
            if (xForwardedFor != null && !xForwardedFor.isEmpty()) {
                return xForwardedFor.split(",")[0].trim();
            }
            
            String xRealIp = request.getHeaders().getFirst("X-Real-IP");
            if (xRealIp != null && !xRealIp.isEmpty()) {
                return xRealIp;
            }
            
            return request.getRemoteAddress().getAddress().getHostAddress();
        }
        
        private Mono<Void> handleRateLimitExceeded(ServerWebExchange exchange) {
            ServerHttpResponse response = exchange.getResponse();
            response.setStatusCode(HttpStatus.TOO_MANY_REQUESTS);
            response.getHeaders().add("Content-Type", "application/json");
            
            String body = "{\"error\":\"Rate limit exceeded\",\"message\":\"请求过于频繁，请稍后重试\"}";
            DataBuffer buffer = response.bufferFactory().wrap(body.getBytes());
            
            return response.writeWith(Mono.just(buffer));
        }
        
        public static class Config {
            private int limit = 100;        // 限制次数
            private int windowSize = 60;    // 时间窗口（秒）
            
            // getters and setters
            public int getLimit() { return limit; }
            public void setLimit(int limit) { this.limit = limit; }
            public int getWindowSize() { return windowSize; }
            public void setWindowSize(int windowSize) { this.windowSize = windowSize; }
        }
    }
}

## 2. 技术选型与架构决策

### 2.1 技术选型框架

```java
import org.springframework.stereotype.Component;
import java.util.*;
import java.time.LocalDateTime;

/**
 * 技术选型决策框架
 * 提供系统化的技术选型方法和评估标准
 */
@Component
public class TechnologySelectionFramework {
    
    // ==================== 技术选型评估模型 ====================
    
    /**
     * 技术选型评估标准
     */
    public enum EvaluationCriteria {
        PERFORMANCE("性能", 0.25),           // 性能表现
        SCALABILITY("可扩展性", 0.20),       // 扩展能力
        MAINTAINABILITY("可维护性", 0.15),   // 维护成本
        COMMUNITY("社区支持", 0.10),         // 社区活跃度
        LEARNING_CURVE("学习成本", 0.10),    // 学习难度
        STABILITY("稳定性", 0.10),          // 技术成熟度
        COST("成本", 0.10);                 // 使用成本
        
        private final String description;
        private final double weight;
        
        EvaluationCriteria(String description, double weight) {
            this.description = description;
            this.weight = weight;
        }
        
        public String getDescription() { return description; }
        public double getWeight() { return weight; }
    }
    
    /**
     * 技术选项评估
     */
    public static class TechnologyOption {
        private String name;
        private String description;
        private Map<EvaluationCriteria, Integer> scores; // 1-10分
        private List<String> pros;
        private List<String> cons;
        private double totalScore;
        
        public TechnologyOption(String name, String description) {
            this.name = name;
            this.description = description;
            this.scores = new HashMap<>();
            this.pros = new ArrayList<>();
            this.cons = new ArrayList<>();
        }
        
        public TechnologyOption score(EvaluationCriteria criteria, int score) {
            if (score < 1 || score > 10) {
                throw new IllegalArgumentException("评分必须在1-10之间");
            }
            this.scores.put(criteria, score);
            return this;
        }
        
        public TechnologyOption addPro(String pro) {
            this.pros.add(pro);
            return this;
        }
        
        public TechnologyOption addCon(String con) {
            this.cons.add(con);
            return this;
        }
        
        public void calculateTotalScore() {
            this.totalScore = scores.entrySet().stream()
                .mapToDouble(entry -> entry.getKey().getWeight() * entry.getValue())
                .sum();
        }
        
        // Getters
        public String getName() { return name; }
        public String getDescription() { return description; }
        public Map<EvaluationCriteria, Integer> getScores() { return scores; }
        public List<String> getPros() { return pros; }
        public List<String> getCons() { return cons; }
        public double getTotalScore() { return totalScore; }
    }
    
    /**
     * 技术选型决策器
     */
    public static class TechnologySelector {
        
        private String category;
        private List<TechnologyOption> options;
        private String selectedTechnology;
        private String decisionReason;
        
        public TechnologySelector(String category) {
            this.category = category;
            this.options = new ArrayList<>();
        }
        
        public TechnologySelector addOption(TechnologyOption option) {
            this.options.add(option);
            return this;
        }
        
        /**
         * 执行技术选型决策
         */
        public TechnologySelectionResult makeDecision() {
            if (options.isEmpty()) {
                throw new IllegalStateException("没有可选的技术方案");
            }
            
            // 计算每个选项的总分
            options.forEach(TechnologyOption::calculateTotalScore);
            
            // 按总分排序
            options.sort((o1, o2) -> Double.compare(o2.getTotalScore(), o1.getTotalScore()));
            
            // 选择最高分的技术
            TechnologyOption selected = options.get(0);
            this.selectedTechnology = selected.getName();
            this.decisionReason = generateDecisionReason(selected);
            
            return new TechnologySelectionResult(
                category,
                selectedTechnology,
                decisionReason,
                options,
                LocalDateTime.now()
            );
        }
        
        private String generateDecisionReason(TechnologyOption selected) {
            StringBuilder reason = new StringBuilder();
            reason.append(String.format("选择 %s 的主要原因：\n", selected.getName()));
            reason.append(String.format("总分: %.2f\n", selected.getTotalScore()));
            reason.append("主要优势:\n");
            selected.getPros().forEach(pro -> reason.append("- ").append(pro).append("\n"));
            
            if (!selected.getCons().isEmpty()) {
                reason.append("需要注意的问题:\n");
                selected.getCons().forEach(con -> reason.append("- ").append(con).append("\n"));
            }
            
            return reason.toString();
        }
    }
    
    /**
     * 技术选型结果
     */
    public static class TechnologySelectionResult {
        private String category;
        private String selectedTechnology;
        private String decisionReason;
        private List<TechnologyOption> allOptions;
        private LocalDateTime decisionTime;
        
        public TechnologySelectionResult(String category, String selectedTechnology, 
                                       String decisionReason, List<TechnologyOption> allOptions,
                                       LocalDateTime decisionTime) {
            this.category = category;
            this.selectedTechnology = selectedTechnology;
            this.decisionReason = decisionReason;
            this.allOptions = new ArrayList<>(allOptions);
            this.decisionTime = decisionTime;
        }
        
        public void printReport() {
            System.out.println("=== 技术选型决策报告 ===");
            System.out.println("类别: " + category);
            System.out.println("选择结果: " + selectedTechnology);
            System.out.println("决策时间: " + decisionTime);
            System.out.println();
            System.out.println("决策理由:");
            System.out.println(decisionReason);
            System.out.println();
            System.out.println("所有选项评分:");
            allOptions.forEach(option -> {
                System.out.printf("- %s: %.2f分\n", option.getName(), option.getTotalScore());
            });
        }
        
        // Getters
        public String getCategory() { return category; }
        public String getSelectedTechnology() { return selectedTechnology; }
        public String getDecisionReason() { return decisionReason; }
        public List<TechnologyOption> getAllOptions() { return allOptions; }
        public LocalDateTime getDecisionTime() { return decisionTime; }
    }
    
    // ==================== 具体技术选型示例 ====================
    
    /**
     * 数据库技术选型示例
     */
    public TechnologySelectionResult selectDatabase() {
        TechnologySelector selector = new TechnologySelector("数据库");
        
        // MySQL选项
        TechnologyOption mysql = new TechnologyOption("MySQL", "关系型数据库，广泛使用")
            .score(EvaluationCriteria.PERFORMANCE, 7)
            .score(EvaluationCriteria.SCALABILITY, 6)
            .score(EvaluationCriteria.MAINTAINABILITY, 8)
            .score(EvaluationCriteria.COMMUNITY, 9)
            .score(EvaluationCriteria.LEARNING_CURVE, 8)
            .score(EvaluationCriteria.STABILITY, 9)
            .score(EvaluationCriteria.COST, 9)
            .addPro("成熟稳定，社区支持好")
            .addPro("学习成本低，人才储备充足")
            .addPro("开源免费")
            .addCon("单机性能有限")
            .addCon("水平扩展复杂");
        
        // PostgreSQL选项
        TechnologyOption postgresql = new TechnologyOption("PostgreSQL", "功能强大的关系型数据库")
            .score(EvaluationCriteria.PERFORMANCE, 8)
            .score(EvaluationCriteria.SCALABILITY, 7)
            .score(EvaluationCriteria.MAINTAINABILITY, 8)
            .score(EvaluationCriteria.COMMUNITY, 8)
            .score(EvaluationCriteria.LEARNING_CURVE, 6)
            .score(EvaluationCriteria.STABILITY, 9)
            .score(EvaluationCriteria.COST, 9)
            .addPro("功能丰富，支持复杂查询")
            .addPro("ACID特性完整")
            .addPro("扩展性好")
            .addCon("学习成本相对较高")
            .addCon("配置复杂");
        
        // MongoDB选项
        TechnologyOption mongodb = new TechnologyOption("MongoDB", "文档型NoSQL数据库")
            .score(EvaluationCriteria.PERFORMANCE, 8)
            .score(EvaluationCriteria.SCALABILITY, 9)
            .score(EvaluationCriteria.MAINTAINABILITY, 7)
            .score(EvaluationCriteria.COMMUNITY, 8)
            .score(EvaluationCriteria.LEARNING_CURVE, 7)
            .score(EvaluationCriteria.STABILITY, 7)
            .score(EvaluationCriteria.COST, 8)
            .addPro("水平扩展能力强")
            .addPro("灵活的文档结构")
            .addPro("适合大数据场景")
            .addCon("事务支持有限")
            .addCon("数据一致性相对较弱");
        
        return selector.addOption(mysql)
                      .addOption(postgresql)
                      .addOption(mongodb)
                      .makeDecision();
    }
    
    /**
     * 缓存技术选型示例
     */
    public TechnologySelectionResult selectCache() {
        TechnologySelector selector = new TechnologySelector("缓存");
        
        // Redis选项
        TechnologyOption redis = new TechnologyOption("Redis", "内存数据结构存储")
            .score(EvaluationCriteria.PERFORMANCE, 9)
            .score(EvaluationCriteria.SCALABILITY, 8)
            .score(EvaluationCriteria.MAINTAINABILITY, 8)
            .score(EvaluationCriteria.COMMUNITY, 9)
            .score(EvaluationCriteria.LEARNING_CURVE, 7)
            .score(EvaluationCriteria.STABILITY, 8)
            .score(EvaluationCriteria.COST, 8)
            .addPro("性能优异")
            .addPro("数据结构丰富")
            .addPro("支持持久化")
            .addCon("内存消耗大")
            .addCon("单线程模型");
        
        // Memcached选项
        TechnologyOption memcached = new TechnologyOption("Memcached", "高性能分布式内存缓存")
            .score(EvaluationCriteria.PERFORMANCE, 9)
            .score(EvaluationCriteria.SCALABILITY, 7)
            .score(EvaluationCriteria.MAINTAINABILITY, 7)
            .score(EvaluationCriteria.COMMUNITY, 7)
            .score(EvaluationCriteria.LEARNING_CURVE, 8)
            .score(EvaluationCriteria.STABILITY, 8)
            .score(EvaluationCriteria.COST, 9)
            .addPro("简单高效")
            .addPro("多线程支持")
            .addPro("内存使用效率高")
            .addCon("功能相对简单")
            .addCon("不支持持久化");
        
        return selector.addOption(redis)
                      .addOption(memcached)
                      .makeDecision();
    }
    
    /**
     * 消息队列技术选型示例
     */
    public TechnologySelectionResult selectMessageQueue() {
        TechnologySelector selector = new TechnologySelector("消息队列");
        
        // RabbitMQ选项
        TechnologyOption rabbitmq = new TechnologyOption("RabbitMQ", "功能丰富的消息代理")
            .score(EvaluationCriteria.PERFORMANCE, 7)
            .score(EvaluationCriteria.SCALABILITY, 7)
            .score(EvaluationCriteria.MAINTAINABILITY, 8)
            .score(EvaluationCriteria.COMMUNITY, 8)
            .score(EvaluationCriteria.LEARNING_CURVE, 6)
            .score(EvaluationCriteria.STABILITY, 9)
            .score(EvaluationCriteria.COST, 8)
            .addPro("功能完整，支持多种消息模式")
            .addPro("管理界面友好")
            .addPro("可靠性高")
            .addCon("性能相对较低")
            .addCon("配置复杂");
        
        // Apache Kafka选项
        TechnologyOption kafka = new TechnologyOption("Apache Kafka", "高吞吐量分布式流处理平台")
            .score(EvaluationCriteria.PERFORMANCE, 9)
            .score(EvaluationCriteria.SCALABILITY, 9)
            .score(EvaluationCriteria.MAINTAINABILITY, 6)
            .score(EvaluationCriteria.COMMUNITY, 8)
            .score(EvaluationCriteria.LEARNING_CURVE, 5)
            .score(EvaluationCriteria.STABILITY, 8)
            .score(EvaluationCriteria.COST, 7)
            .addPro("超高吞吐量")
            .addPro("水平扩展能力强")
            .addPro("支持流处理")
            .addCon("学习成本高")
            .addCon("运维复杂");
        
        // RocketMQ选项
        TechnologyOption rocketmq = new TechnologyOption("Apache RocketMQ", "阿里巴巴开源的消息中间件")
            .score(EvaluationCriteria.PERFORMANCE, 8)
            .score(EvaluationCriteria.SCALABILITY, 8)
            .score(EvaluationCriteria.MAINTAINABILITY, 7)
            .score(EvaluationCriteria.COMMUNITY, 7)
            .score(EvaluationCriteria.LEARNING_CURVE, 6)
            .score(EvaluationCriteria.STABILITY, 8)
            .score(EvaluationCriteria.COST, 8)
            .addPro("性能优秀")
            .addPro("支持事务消息")
            .addPro("运维相对简单")
            .addCon("社区相对较小")
            .addCon("文档不够完善");
        
        return selector.addOption(rabbitmq)
                      .addOption(kafka)
                      .addOption(rocketmq)
                      .makeDecision();
    }
}

### 2.2 架构决策记录 (ADR)

```java
import java.time.LocalDateTime;
import java.util.*;
import java.util.stream.Collectors;

/**
 * 架构决策记录 (Architecture Decision Record)
 * 用于记录和追踪重要的架构决策
 */
public class ArchitectureDecisionRecord {
    
    // ==================== ADR核心结构 ====================
    
    /**
     * 决策状态
     */
    public enum DecisionStatus {
        PROPOSED("提议中"),
        ACCEPTED("已接受"),
        DEPRECATED("已废弃"),
        SUPERSEDED("已被替代");
        
        private final String description;
        
        DecisionStatus(String description) {
            this.description = description;
        }
        
        public String getDescription() { return description; }
    }
    
    /**
     * 架构决策记录实体
     */
    public static class ADR {
        private String id;
        private String title;
        private DecisionStatus status;
        private LocalDateTime date;
        private String context;
        private String decision;
        private String rationale;
        private List<String> consequences;
        private List<String> alternatives;
        private String author;
        private List<String> reviewers;
        private String supersededBy;
        
        public ADR(String id, String title, String author) {
            this.id = id;
            this.title = title;
            this.author = author;
            this.status = DecisionStatus.PROPOSED;
            this.date = LocalDateTime.now();
            this.consequences = new ArrayList<>();
            this.alternatives = new ArrayList<>();
            this.reviewers = new ArrayList<>();
        }
        
        // Builder模式
        public ADR context(String context) {
            this.context = context;
            return this;
        }
        
        public ADR decision(String decision) {
            this.decision = decision;
            return this;
        }
        
        public ADR rationale(String rationale) {
            this.rationale = rationale;
            return this;
        }
        
        public ADR addConsequence(String consequence) {
            this.consequences.add(consequence);
            return this;
        }
        
        public ADR addAlternative(String alternative) {
            this.alternatives.add(alternative);
            return this;
        }
        
        public ADR addReviewer(String reviewer) {
            this.reviewers.add(reviewer);
            return this;
        }
        
        public ADR accept() {
            this.status = DecisionStatus.ACCEPTED;
            return this;
        }
        
        public ADR deprecate() {
            this.status = DecisionStatus.DEPRECATED;
            return this;
        }
        
        public ADR supersede(String newAdrId) {
            this.status = DecisionStatus.SUPERSEDED;
            this.supersededBy = newAdrId;
            return this;
        }
        
        /**
         * 生成ADR文档
         */
        public String generateDocument() {
            StringBuilder doc = new StringBuilder();
            
            doc.append("# ADR-").append(id).append(": ").append(title).append("\n\n");
            doc.append("**状态**: ").append(status.getDescription()).append("\n");
            doc.append("**日期**: ").append(date.toLocalDate()).append("\n");
            doc.append("**作者**: ").append(author).append("\n");
            
            if (!reviewers.isEmpty()) {
                doc.append("**评审者**: ").append(String.join(", ", reviewers)).append("\n");
            }
            
            if (supersededBy != null) {
                doc.append("**被替代**: ADR-").append(supersededBy).append("\n");
            }
            
            doc.append("\n## 背景\n\n").append(context).append("\n\n");
            doc.append("## 决策\n\n").append(decision).append("\n\n");
            doc.append("## 理由\n\n").append(rationale).append("\n\n");
            
            if (!alternatives.isEmpty()) {
                doc.append("## 备选方案\n\n");
                alternatives.forEach(alt -> doc.append("- ").append(alt).append("\n"));
                doc.append("\n");
            }
            
            if (!consequences.isEmpty()) {
                doc.append("## 后果\n\n");
                consequences.forEach(cons -> doc.append("- ").append(cons).append("\n"));
            }
            
            return doc.toString();
        }
        
        // Getters
        public String getId() { return id; }
        public String getTitle() { return title; }
        public DecisionStatus getStatus() { return status; }
        public LocalDateTime getDate() { return date; }
        public String getContext() { return context; }
        public String getDecision() { return decision; }
        public String getRationale() { return rationale; }
        public List<String> getConsequences() { return consequences; }
        public List<String> getAlternatives() { return alternatives; }
        public String getAuthor() { return author; }
        public List<String> getReviewers() { return reviewers; }
        public String getSupersededBy() { return supersededBy; }
    }
    
    /**
     * ADR管理器
     */
    public static class ADRManager {
        private Map<String, ADR> adrs;
        private int nextId;
        
        public ADRManager() {
            this.adrs = new HashMap<>();
            this.nextId = 1;
        }
        
        /**
         * 创建新的ADR
         */
        public ADR createADR(String title, String author) {
            String id = String.format("%03d", nextId++);
            ADR adr = new ADR(id, title, author);
            adrs.put(id, adr);
            return adr;
        }
        
        /**
         * 获取ADR
         */
        public ADR getADR(String id) {
            return adrs.get(id);
        }
        
        /**
         * 获取所有有效的ADR
         */
        public List<ADR> getActiveADRs() {
            return adrs.values().stream()
                .filter(adr -> adr.getStatus() == DecisionStatus.ACCEPTED)
                .sorted(Comparator.comparing(ADR::getDate))
                .collect(Collectors.toList());
        }
        
        /**
         * 生成ADR索引
         */
        public String generateIndex() {
            StringBuilder index = new StringBuilder();
            index.append("# 架构决策记录索引\n\n");
            
            Map<DecisionStatus, List<ADR>> groupedAdrs = adrs.values().stream()
                .collect(Collectors.groupingBy(ADR::getStatus));
            
            for (DecisionStatus status : DecisionStatus.values()) {
                List<ADR> statusAdrs = groupedAdrs.get(status);
                if (statusAdrs != null && !statusAdrs.isEmpty()) {
                    index.append("## ").append(status.getDescription()).append("\n\n");
                    statusAdrs.stream()
                        .sorted(Comparator.comparing(ADR::getDate))
                        .forEach(adr -> {
                            index.append("- [ADR-").append(adr.getId())
                                 .append("](./adr-").append(adr.getId())
                                 .append(".md): ").append(adr.getTitle())
                                 .append(" (").append(adr.getDate().toLocalDate())
                                 .append(")\n");
                        });
                    index.append("\n");
                }
            }
            
            return index.toString();
        }
    }
    
    // ==================== ADR示例 ====================
    
    /**
     * 创建具体的ADR示例
     */
    public static void createSampleADRs() {
        ADRManager manager = new ADRManager();
        
        // ADR-001: 选择微服务架构
        ADR microserviceADR = manager.createADR("选择微服务架构", "架构师")
            .context("随着业务快速发展，单体应用已经无法满足快速迭代和团队扩展的需求。" +
                    "当前系统面临以下问题：\n" +
                    "1. 代码库过大，开发效率低下\n" +
                    "2. 部署风险高，一个模块的问题影响整个系统\n" +
                    "3. 技术栈固化，难以引入新技术\n" +
                    "4. 团队协作困难，代码冲突频繁")
            .decision("采用微服务架构，将单体应用拆分为多个独立的服务")
            .rationale("微服务架构能够解决当前面临的主要问题：\n" +
                      "1. 服务独立部署，降低部署风险\n" +
                      "2. 技术栈多样化，可以为不同服务选择最适合的技术\n" +
                      "3. 团队独立开发，提高开发效率\n" +
                      "4. 更好的可扩展性和容错性")
            .addAlternative("继续使用单体架构，通过模块化改进")
            .addAlternative("采用模块化单体架构")
            .addConsequence("正面影响：提高开发效率，降低部署风险，技术栈灵活")
            .addConsequence("负面影响：增加系统复杂性，需要处理分布式系统问题")
            .addConsequence("运维成本增加，需要服务治理和监控")
            .addReviewer("技术总监")
            .addReviewer("开发团队负责人")
            .accept();
        
        // ADR-002: 选择Spring Cloud作为微服务框架
        ADR springCloudADR = manager.createADR("选择Spring Cloud作为微服务框架", "架构师")
            .context("在决定采用微服务架构后，需要选择合适的微服务框架。" +
                    "团队主要使用Java技术栈，需要考虑学习成本和生态完整性。")
            .decision("选择Spring Cloud作为微服务开发框架")
            .rationale("Spring Cloud的优势：\n" +
                      "1. 与Spring Boot无缝集成，学习成本低\n" +
                      "2. 生态完整，包含服务发现、配置管理、熔断器等组件\n" +
                      "3. 社区活跃，文档完善\n" +
                      "4. 团队已有Spring经验，技术栈一致")
            .addAlternative("Dubbo + Zookeeper")
            .addAlternative("Service Mesh (Istio)")
            .addConsequence("正面影响：快速上手，生态完整，开发效率高")
            .addConsequence("负面影响：Netflix组件停止更新，需要迁移到Spring Cloud Alibaba")
            .addReviewer("技术总监")
            .accept();
        
        // ADR-003: 数据库选择
        ADR databaseADR = manager.createADR("选择MySQL作为主数据库", "DBA")
            .context("微服务架构下，每个服务需要独立的数据库。" +
                    "需要选择稳定、性能好、运维成本低的数据库。")
            .decision("选择MySQL 8.0作为主要的关系型数据库")
            .rationale("MySQL的优势：\n" +
                      "1. 成熟稳定，生产环境验证充分\n" +
                      "2. 性能优秀，满足当前业务需求\n" +
                      "3. 运维经验丰富，人才储备充足\n" +
                      "4. 开源免费，成本可控")
            .addAlternative("PostgreSQL")
            .addAlternative("Oracle")
            .addConsequence("正面影响：稳定可靠，运维成本低")
            .addConsequence("负面影响：单机扩展能力有限，需要考虑分库分表")
            .addReviewer("架构师")
            .accept();
        
        // 打印ADR索引
        System.out.println(manager.generateIndex());
        
        // 打印具体ADR文档
        System.out.println(microserviceADR.generateDocument());
    }
 }
 ```

### 3.3 缓存优化策略

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Service;
import java.util.*;
import java.util.concurrent.*;

/**
 * 缓存优化策略和工具
 */
@Service
public class CacheOptimizationStrategy {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    // ==================== 缓存预热策略 ====================
    
    /**
     * 缓存预热管理器
     */
    public static class CacheWarmupManager {
        private final ExecutorService executor;
        private final Map<String, WarmupTask> warmupTasks;
        
        public CacheWarmupManager() {
            this.executor = Executors.newFixedThreadPool(5);
            this.warmupTasks = new ConcurrentHashMap<>();
        }
        
        /**
         * 注册预热任务
         */
        public void registerWarmupTask(String taskName, WarmupTask task) {
            warmupTasks.put(taskName, task);
        }
        
        /**
         * 执行所有预热任务
         */
        public CompletableFuture<Void> executeAllWarmupTasks() {
            List<CompletableFuture<Void>> futures = new ArrayList<>();
            
            for (Map.Entry<String, WarmupTask> entry : warmupTasks.entrySet()) {
                CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
                    try {
                        System.out.println("开始预热任务: " + entry.getKey());
                        entry.getValue().warmup();
                        System.out.println("完成预热任务: " + entry.getKey());
                    } catch (Exception e) {
                        System.err.println("预热任务失败: " + entry.getKey() + ", 错误: " + e.getMessage());
                    }
                }, executor);
                futures.add(future);
            }
            
            return CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]));
        }
        
        /**
         * 执行特定预热任务
         */
        public CompletableFuture<Void> executeWarmupTask(String taskName) {
            WarmupTask task = warmupTasks.get(taskName);
            if (task == null) {
                return CompletableFuture.completedFuture(null);
            }
            
            return CompletableFuture.runAsync(() -> {
                try {
                    task.warmup();
                } catch (Exception e) {
                    throw new RuntimeException("预热任务执行失败: " + taskName, e);
                }
            }, executor);
        }
    }
    
    @FunctionalInterface
    public interface WarmupTask {
        void warmup() throws Exception;
    }
    
    // ==================== 缓存监控 ====================
    
    /**
     * 缓存性能监控器
     */
    public class CacheMonitor {
        private final Map<String, CacheMetrics> metricsMap = new ConcurrentHashMap<>();
        
        /**
         * 记录缓存命中
         */
        public void recordHit(String cacheName) {
            metricsMap.computeIfAbsent(cacheName, k -> new CacheMetrics()).incrementHit();
        }
        
        /**
         * 记录缓存未命中
         */
        public void recordMiss(String cacheName) {
            metricsMap.computeIfAbsent(cacheName, k -> new CacheMetrics()).incrementMiss();
        }
        
        /**
         * 记录缓存加载时间
         */
        public void recordLoadTime(String cacheName, long loadTimeMs) {
            metricsMap.computeIfAbsent(cacheName, k -> new CacheMetrics()).addLoadTime(loadTimeMs);
        }
        
        /**
         * 获取缓存统计信息
         */
        public CacheMetrics getMetrics(String cacheName) {
            return metricsMap.get(cacheName);
        }
        
        /**
         * 生成缓存报告
         */
        public String generateCacheReport() {
            StringBuilder report = new StringBuilder();
            report.append("=== 缓存性能报告 ===\n");
            
            for (Map.Entry<String, CacheMetrics> entry : metricsMap.entrySet()) {
                CacheMetrics metrics = entry.getValue();
                report.append("缓存: ").append(entry.getKey()).append("\n");
                report.append("  命中率: ").append(String.format("%.2f%%", metrics.getHitRate() * 100)).append("\n");
                report.append("  总请求: ").append(metrics.getTotalRequests()).append("\n");
                report.append("  命中次数: ").append(metrics.getHitCount()).append("\n");
                report.append("  未命中次数: ").append(metrics.getMissCount()).append("\n");
                report.append("  平均加载时间: ").append(String.format("%.2fms", metrics.getAverageLoadTime())).append("\n\n");
            }
            
            return report.toString();
        }
    }
    
    public static class CacheMetrics {
        private long hitCount = 0;
        private long missCount = 0;
        private long totalLoadTime = 0;
        private long loadCount = 0;
        
        public synchronized void incrementHit() {
            hitCount++;
        }
        
        public synchronized void incrementMiss() {
            missCount++;
        }
        
        public synchronized void addLoadTime(long loadTimeMs) {
            totalLoadTime += loadTimeMs;
            loadCount++;
        }
        
        public long getHitCount() { return hitCount; }
        public long getMissCount() { return missCount; }
        public long getTotalRequests() { return hitCount + missCount; }
        
        public double getHitRate() {
            long total = getTotalRequests();
            return total > 0 ? (double) hitCount / total : 0.0;
        }
        
        public double getAverageLoadTime() {
            return loadCount > 0 ? (double) totalLoadTime / loadCount : 0.0;
        }
    }
    
    // ==================== 缓存优化建议 ====================
    
    /**
     * 缓存优化建议生成器
     */
    public class CacheOptimizationAdvisor {
        
        /**
         * 分析缓存性能并生成优化建议
         */
        public List<String> generateOptimizationAdvice(String cacheName, CacheMetrics metrics) {
            List<String> advice = new ArrayList<>();
            
            // 命中率分析
            double hitRate = metrics.getHitRate();
            if (hitRate < 0.8) {
                advice.add("缓存命中率较低(" + String.format("%.2f%%", hitRate * 100) + 
                          ")，建议检查缓存策略和过期时间设置");
            }
            
            // 加载时间分析
            double avgLoadTime = metrics.getAverageLoadTime();
            if (avgLoadTime > 100) {
                advice.add("缓存加载时间过长(" + String.format("%.2fms", avgLoadTime) + 
                          ")，建议优化数据源查询或增加缓存预热");
            }
            
            // 请求量分析
            long totalRequests = metrics.getTotalRequests();
            if (totalRequests > 10000 && hitRate < 0.9) {
                advice.add("高频访问缓存命中率不足，建议增加缓存容量或调整淘汰策略");
            }
            
            // 通用优化建议
            advice.add("建议定期监控缓存性能指标");
            advice.add("考虑使用多级缓存架构提高性能");
            advice.add("实施缓存预热策略减少冷启动影响");
            
            return advice;
        }
        
        /**
         * 推荐缓存配置
         */
        public CacheConfiguration recommendConfiguration(CacheMetrics metrics) {
            CacheConfiguration config = new CacheConfiguration();
            
            // 基于命中率推荐TTL
            double hitRate = metrics.getHitRate();
            if (hitRate > 0.9) {
                config.ttlSeconds = 3600; // 1小时
            } else if (hitRate > 0.8) {
                config.ttlSeconds = 1800; // 30分钟
            } else {
                config.ttlSeconds = 900; // 15分钟
            }
            
            // 基于请求量推荐最大大小
            long totalRequests = metrics.getTotalRequests();
            if (totalRequests > 100000) {
                config.maxSize = 10000;
            } else if (totalRequests > 10000) {
                config.maxSize = 5000;
            } else {
                config.maxSize = 1000;
            }
            
            return config;
        }
    }
    
    public static class CacheConfiguration {
        public int ttlSeconds;
        public int maxSize;
        public String evictionPolicy = "LRU";
        
        @Override
        public String toString() {
            return String.format("CacheConfiguration{ttl=%ds, maxSize=%d, eviction=%s}",
                ttlSeconds, maxSize, evictionPolicy);
        }
    }
}
```

## 4. 经典面试题

### 4.1 架构设计类问题

**1. 如何设计一个高并发的秒杀系统？**

秒杀系统是典型的高并发、低延迟场景，需要从多个层面进行优化设计。

**核心挑战：**
- 瞬时流量巨大（正常流量的10-100倍）
- 库存数量有限，需要精确控制
- 用户体验要求高，响应时间要短
- 防止恶意刷单和超卖

**系统架构设计：**

| 层次 | 组件 | 作用 | 关键技术 |
|------|------|------|----------|
| 前端层 | CDN + 静态化 | 减少服务器压力 | 页面静态化、资源缓存 |
| 接入层 | 负载均衡 + 限流 | 流量控制和分发 | Nginx、网关限流 |
| 应用层 | 微服务 + 缓存 | 业务处理 | Spring Cloud、Redis |
| 数据层 | 分库分表 + 读写分离 | 数据存储 | MySQL集群、Redis集群 |
| 消息层 | 消息队列 | 异步处理 | RocketMQ、Kafka |

**详细实现方案：**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.stereotype.Service;
import org.springframework.stereotype.Component;

import java.util.*;
import java.util.concurrent.*;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

/**
 * 秒杀系统核心实现
 */
@SpringBootApplication
@EnableFeignClients
@EnableAsync
@EnableTransactionManagement
public class SeckillSystemApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(SeckillSystemApplication.class, args);
    }
    
    // ==================== 1. 前端优化层 ====================
    
    /**
     * 秒杀页面控制器 - 静态化处理
     */
    @RestController
    @RequestMapping("/seckill")
    public class SeckillPageController {
        
        @Autowired
        private SeckillService seckillService;
        
        /**
         * 获取秒杀商品详情（静态数据）
         * 通过CDN缓存，减少服务器压力
         */
        @GetMapping("/goods/{goodsId}")
        public SeckillGoodsVO getGoodsDetail(@PathVariable Long goodsId) {
            // 返回静态商品信息，不包含库存等动态数据
            return seckillService.getStaticGoodsInfo(goodsId);
        }
        
        /**
         * 获取动态数据（库存、状态等）
         * 通过AJAX异步获取，避免页面刷新
         */
        @GetMapping("/status/{goodsId}")
        public SeckillStatusVO getGoodsStatus(@PathVariable Long goodsId) {
            return seckillService.getGoodsStatus(goodsId);
        }
        
        /**
         * 防重复提交令牌获取
         */
        @GetMapping("/token/{goodsId}")
        public String getSeckillToken(@PathVariable Long goodsId, 
                                    @RequestHeader("User-Id") Long userId) {
            return seckillService.generateSeckillToken(goodsId, userId);
        }
    }
    
    // ==================== 2. 接入层限流 ====================
    
    /**
     * 限流组件 - 基于Redis + Lua脚本
     */
    @Component
    public class RateLimiter {
        
        @Autowired
        private RedisTemplate<String, Object> redisTemplate;
        
        // 滑动窗口限流Lua脚本
        private static final String RATE_LIMIT_SCRIPT = 
            "local key = KEYS[1]\n" +
            "local window = tonumber(ARGV[1])\n" +
            "local limit = tonumber(ARGV[2])\n" +
            "local current = tonumber(ARGV[3])\n" +
            "\n" +
            "-- 清理过期数据\n" +
            "redis.call('zremrangebyscore', key, 0, current - window * 1000)\n" +
            "\n" +
            "-- 获取当前窗口内的请求数\n" +
            "local currentCount = redis.call('zcard', key)\n" +
            "\n" +
            "-- 检查是否超过限制\n" +
            "if currentCount < limit then\n" +
            "    -- 添加当前请求\n" +
            "    redis.call('zadd', key, current, current)\n" +
            "    redis.call('expire', key, window + 1)\n" +
            "    return 1\n" +
            "else\n" +
            "    return 0\n" +
            "end";
        
        /**
         * 滑动窗口限流
         * @param key 限流key（可以是用户ID、IP等）
         * @param windowSeconds 时间窗口（秒）
         * @param limit 限制次数
         * @return true-允许通过，false-被限流
         */
        public boolean isAllowed(String key, int windowSeconds, int limit) {
            DefaultRedisScript<Long> script = new DefaultRedisScript<>();
            script.setScriptText(RATE_LIMIT_SCRIPT);
            script.setResultType(Long.class);
            
            Long result = redisTemplate.execute(script, 
                Collections.singletonList("rate_limit:" + key),
                windowSeconds, limit, System.currentTimeMillis());
            
            return result != null && result == 1;
        }
    }
    
    // ==================== 3. 业务层核心逻辑 ====================
    
    /**
     * 秒杀服务核心实现
     */
    @Service
    public class SeckillService {
        
        @Autowired
        private RedisTemplate<String, Object> redisTemplate;
        
        @Autowired
        private RateLimiter rateLimiter;
        
        @Autowired
        private SeckillOrderProducer orderProducer;
        
        @Autowired
        private SeckillGoodsMapper goodsMapper;
        
        // Redis中库存的key前缀
        private static final String STOCK_KEY_PREFIX = "seckill:stock:";
        // 秒杀令牌key前缀
        private static final String TOKEN_KEY_PREFIX = "seckill:token:";
        
        /**
         * 秒杀核心方法
         */
        public SeckillResult doSeckill(Long goodsId, Long userId, String token) {
            // 1. 验证令牌，防止重复提交
            if (!validateToken(goodsId, userId, token)) {
                return SeckillResult.fail("请勿重复提交");
            }
            
            // 2. 用户限流检查
            if (!rateLimiter.isAllowed("user:" + userId, 60, 5)) {
                return SeckillResult.fail("请求过于频繁，请稍后再试");
            }
            
            // 3. 商品限流检查
            if (!rateLimiter.isAllowed("goods:" + goodsId, 1, 1000)) {
                return SeckillResult.fail("商品访问过于频繁");
            }
            
            // 4. 预扣库存（Redis原子操作）
            if (!decreaseStock(goodsId)) {
                return SeckillResult.fail("商品已售罄");
            }
            
            // 5. 异步创建订单
            SeckillOrderMessage orderMessage = new SeckillOrderMessage();
            orderMessage.setGoodsId(goodsId);
            orderMessage.setUserId(userId);
            orderMessage.setCreateTime(LocalDateTime.now());
            
            orderProducer.sendOrderMessage(orderMessage);
            
            return SeckillResult.success("秒杀成功，正在生成订单");
        }
        
        /**
         * Redis原子扣减库存
         */
        private boolean decreaseStock(Long goodsId) {
            String stockKey = STOCK_KEY_PREFIX + goodsId;
            
            // Lua脚本保证原子性
            String luaScript = 
                "local stock = redis.call('get', KEYS[1])\n" +
                "if stock and tonumber(stock) > 0 then\n" +
                "    return redis.call('decr', KEYS[1])\n" +
                "else\n" +
                "    return -1\n" +
                "end";
            
            DefaultRedisScript<Long> script = new DefaultRedisScript<>();
            script.setScriptText(luaScript);
            script.setResultType(Long.class);
            
            Long result = redisTemplate.execute(script, 
                Collections.singletonList(stockKey));
            
            return result != null && result >= 0;
        }
        
        /**
         * 验证秒杀令牌
         */
        private boolean validateToken(Long goodsId, Long userId, String token) {
            String tokenKey = TOKEN_KEY_PREFIX + goodsId + ":" + userId;
            String storedToken = (String) redisTemplate.opsForValue().get(tokenKey);
            
            if (storedToken != null && storedToken.equals(token)) {
                // 令牌使用后立即删除，防止重复使用
                redisTemplate.delete(tokenKey);
                return true;
            }
            return false;
        }
        
        /**
         * 生成秒杀令牌
         */
        public String generateSeckillToken(Long goodsId, Long userId) {
            // 检查用户是否已经参与过该商品的秒杀
            String userSeckillKey = "seckill:user:" + goodsId + ":" + userId;
            if (redisTemplate.hasKey(userSeckillKey)) {
                return null; // 已经参与过
            }
            
            // 生成令牌
            String token = UUID.randomUUID().toString().replace("-", "");
            String tokenKey = TOKEN_KEY_PREFIX + goodsId + ":" + userId;
            
            // 令牌5分钟有效
            redisTemplate.opsForValue().set(tokenKey, token, 300, TimeUnit.SECONDS);
            // 标记用户已参与
            redisTemplate.opsForValue().set(userSeckillKey, "1", 3600, TimeUnit.SECONDS);
            
            return token;
        }
        
        /**
         * 初始化商品库存到Redis
         */
        public void initGoodsStock(Long goodsId, Integer stock) {
            String stockKey = STOCK_KEY_PREFIX + goodsId;
            redisTemplate.opsForValue().set(stockKey, stock);
        }
        
        /**
         * 获取商品状态
         */
        public SeckillStatusVO getGoodsStatus(Long goodsId) {
            String stockKey = STOCK_KEY_PREFIX + goodsId;
            Integer stock = (Integer) redisTemplate.opsForValue().get(stockKey);
            
            SeckillStatusVO status = new SeckillStatusVO();
            status.setGoodsId(goodsId);
            status.setStock(stock != null ? stock : 0);
            status.setStatus(stock != null && stock > 0 ? "AVAILABLE" : "SOLD_OUT");
            status.setUpdateTime(LocalDateTime.now());
            
            return status;
        }
        
        /**
         * 获取静态商品信息
         */
        public SeckillGoodsVO getStaticGoodsInfo(Long goodsId) {
            // 从缓存或数据库获取商品基本信息
            SeckillGoods goods = goodsMapper.selectById(goodsId);
            
            SeckillGoodsVO vo = new SeckillGoodsVO();
            vo.setGoodsId(goods.getId());
            vo.setGoodsName(goods.getGoodsName());
            vo.setGoodsImg(goods.getGoodsImg());
            vo.setSeckillPrice(goods.getSeckillPrice());
            vo.setOriginalPrice(goods.getOriginalPrice());
            vo.setStartTime(goods.getStartTime());
            vo.setEndTime(goods.getEndTime());
            
            return vo;
        }
    }
    
    // ==================== 4. 异步订单处理 ====================
    
    /**
     * 秒杀订单消息生产者
     */
    @Component
    public class SeckillOrderProducer {
        
        @Autowired
        private RocketMQTemplate rocketMQTemplate;
        
        /**
         * 发送订单创建消息
         */
        public void sendOrderMessage(SeckillOrderMessage message) {
            // 使用事务消息保证一致性
            rocketMQTemplate.sendMessageInTransaction(
                "seckill-order-topic", 
                MessageBuilder.withPayload(message).build(),
                message
            );
        }
    }
    
    /**
     * 秒杀订单消息消费者
     */
    @Component
    @RocketMQMessageListener(
        topic = "seckill-order-topic",
        consumerGroup = "seckill-order-consumer"
    )
    public class SeckillOrderConsumer implements RocketMQListener<SeckillOrderMessage> {
        
        @Autowired
        private OrderService orderService;
        
        @Override
        public void onMessage(SeckillOrderMessage message) {
            try {
                // 创建订单
                orderService.createSeckillOrder(message);
            } catch (Exception e) {
                // 订单创建失败，回滚库存
                rollbackStock(message.getGoodsId());
                throw e;
            }
        }
        
        /**
         * 回滚库存
         */
        private void rollbackStock(Long goodsId) {
            String stockKey = "seckill:stock:" + goodsId;
            redisTemplate.opsForValue().increment(stockKey);
        }
    }
    
    // ==================== 5. 数据模型 ====================
    
    /**
     * 秒杀结果
     */
    public static class SeckillResult {
        private boolean success;
        private String message;
        private Object data;
        
        public static SeckillResult success(String message) {
            SeckillResult result = new SeckillResult();
            result.success = true;
            result.message = message;
            return result;
        }
        
        public static SeckillResult fail(String message) {
            SeckillResult result = new SeckillResult();
            result.success = false;
            result.message = message;
            return result;
        }
        
        // getters and setters
        public boolean isSuccess() { return success; }
        public void setSuccess(boolean success) { this.success = success; }
        public String getMessage() { return message; }
        public void setMessage(String message) { this.message = message; }
        public Object getData() { return data; }
        public void setData(Object data) { this.data = data; }
    }
    
    /**
     * 秒杀商品VO
     */
    public static class SeckillGoodsVO {
        private Long goodsId;
        private String goodsName;
        private String goodsImg;
        private BigDecimal seckillPrice;
        private BigDecimal originalPrice;
        private LocalDateTime startTime;
        private LocalDateTime endTime;
        
        // getters and setters
        public Long getGoodsId() { return goodsId; }
        public void setGoodsId(Long goodsId) { this.goodsId = goodsId; }
        public String getGoodsName() { return goodsName; }
        public void setGoodsName(String goodsName) { this.goodsName = goodsName; }
        public String getGoodsImg() { return goodsImg; }
        public void setGoodsImg(String goodsImg) { this.goodsImg = goodsImg; }
        public BigDecimal getSeckillPrice() { return seckillPrice; }
        public void setSeckillPrice(BigDecimal seckillPrice) { this.seckillPrice = seckillPrice; }
        public BigDecimal getOriginalPrice() { return originalPrice; }
        public void setOriginalPrice(BigDecimal originalPrice) { this.originalPrice = originalPrice; }
        public LocalDateTime getStartTime() { return startTime; }
        public void setStartTime(LocalDateTime startTime) { this.startTime = startTime; }
        public LocalDateTime getEndTime() { return endTime; }
        public void setEndTime(LocalDateTime endTime) { this.endTime = endTime; }
    }
    
    /**
     * 秒杀状态VO
     */
    public static class SeckillStatusVO {
        private Long goodsId;
        private Integer stock;
        private String status;
        private LocalDateTime updateTime;
        
        // getters and setters
        public Long getGoodsId() { return goodsId; }
        public void setGoodsId(Long goodsId) { this.goodsId = goodsId; }
        public Integer getStock() { return stock; }
        public void setStock(Integer stock) { this.stock = stock; }
        public String getStatus() { return status; }
        public void setStatus(String status) { this.status = status; }
        public LocalDateTime getUpdateTime() { return updateTime; }
        public void setUpdateTime(LocalDateTime updateTime) { this.updateTime = updateTime; }
    }
    
    /**
     * 秒杀订单消息
     */
    public static class SeckillOrderMessage {
        private Long goodsId;
        private Long userId;
        private LocalDateTime createTime;
        
        // getters and setters
        public Long getGoodsId() { return goodsId; }
        public void setGoodsId(Long goodsId) { this.goodsId = goodsId; }
        public Long getUserId() { return userId; }
        public void setUserId(Long userId) { this.userId = userId; }
        public LocalDateTime getCreateTime() { return createTime; }
        public void setCreateTime(LocalDateTime createTime) { this.createTime = createTime; }
    }
}
```

**关键技术点：**

1. **前端优化**：
   - 页面静态化，商品信息通过CDN缓存
   - 动态数据（库存、状态）通过AJAX异步获取
   - 防重复提交令牌机制

2. **限流策略**：
   - 基于Redis + Lua脚本的滑动窗口限流
   - 用户维度和商品维度的双重限流
   - 网关层面的总体流量控制

3. **库存控制**：
   - Redis预扣库存，保证原子性
   - Lua脚本避免超卖问题
   - 异步订单处理，失败时回滚库存

4. **异步处理**：
   - 消息队列削峰填谷
   - 事务消息保证数据一致性
   - 订单创建与库存扣减分离

5. **监控告警**：
   - 实时监控系统QPS、响应时间
   - 库存预警和自动补货
   - 异常流量检测和自动限流

**性能优化建议：**

- **数据库优化**：读写分离、分库分表、索引优化
- **缓存策略**：多级缓存、热点数据预加载
- **服务器优化**：水平扩容、负载均衡
- **网络优化**：CDN加速、HTTP/2、压缩传输

**2. 微服务架构中如何保证数据一致性？**

微服务架构中的数据一致性是一个复杂的挑战，需要在性能、可用性和一致性之间做出权衡。

**一致性级别对比：**

| 一致性类型 | 特点 | 适用场景 | 实现复杂度 | 性能影响 |
|------------|------|----------|------------|----------|
| 强一致性 | 数据实时同步 | 金融交易、库存管理 | 高 | 高 |
| 最终一致性 | 数据最终同步 | 用户信息、商品详情 | 中 | 低 |
| 弱一致性 | 允许数据不一致 | 日志记录、统计分析 | 低 | 极低 |

**详细实现方案：**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.*;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.*;
import java.util.concurrent.*;
import java.time.LocalDateTime;
import java.math.BigDecimal;

/**
 * 微服务数据一致性解决方案
 */
@SpringBootApplication
@EnableFeignClients
@EnableTransactionManagement
public class DataConsistencyApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(DataConsistencyApplication.class, args);
    }
    
    // ==================== 1. 分布式事务 - 2PC模式 ====================
    
    /**
     * 分布式事务协调器
     */
    @Service
    public class DistributedTransactionCoordinator {
        
        @Autowired
        private List<TransactionParticipant> participants;
        
        /**
         * 执行分布式事务
         */
        public boolean executeDistributedTransaction(TransactionContext context) {
            // Phase 1: Prepare阶段
            List<String> preparedParticipants = new ArrayList<>();
            
            try {
                for (TransactionParticipant participant : participants) {
                    if (participant.prepare(context)) {
                        preparedParticipants.add(participant.getName());
                    } else {
                        // 有参与者准备失败，回滚所有已准备的参与者
                        rollbackParticipants(preparedParticipants, context);
                        return false;
                    }
                }
                
                // Phase 2: Commit阶段
                for (TransactionParticipant participant : participants) {
                    participant.commit(context);
                }
                
                return true;
                
            } catch (Exception e) {
                // 异常情况下回滚
                rollbackParticipants(preparedParticipants, context);
                return false;
            }
        }
        
        /**
         * 回滚参与者
         */
        private void rollbackParticipants(List<String> participantNames, 
                                         TransactionContext context) {
            for (String participantName : participantNames) {
                TransactionParticipant participant = findParticipant(participantName);
                if (participant != null) {
                    participant.rollback(context);
                }
            }
        }
        
        private TransactionParticipant findParticipant(String name) {
            return participants.stream()
                .filter(p -> p.getName().equals(name))
                .findFirst()
                .orElse(null);
        }
    }
    
    /**
     * 事务参与者接口
     */
    public interface TransactionParticipant {
        String getName();
        boolean prepare(TransactionContext context);
        void commit(TransactionContext context);
        void rollback(TransactionContext context);
    }
    
    /**
     * 订单服务事务参与者
     */
    @Component
    public class OrderTransactionParticipant implements TransactionParticipant {
        
        @Autowired
        private OrderService orderService;
        
        @Override
        public String getName() {
            return "OrderService";
        }
        
        @Override
        public boolean prepare(TransactionContext context) {
            try {
                // 预创建订单，但不提交
                return orderService.prepareCreateOrder(context.getOrderInfo());
            } catch (Exception e) {
                return false;
            }
        }
        
        @Override
        public void commit(TransactionContext context) {
            orderService.commitCreateOrder(context.getOrderInfo());
        }
        
        @Override
        public void rollback(TransactionContext context) {
            orderService.rollbackCreateOrder(context.getOrderInfo());
        }
    }
    
    /**
     * 库存服务事务参与者
     */
    @Component
    public class InventoryTransactionParticipant implements TransactionParticipant {
        
        @Autowired
        private InventoryService inventoryService;
        
        @Override
        public String getName() {
            return "InventoryService";
        }
        
        @Override
        public boolean prepare(TransactionContext context) {
            try {
                // 预扣库存
                return inventoryService.prepareDeductStock(
                    context.getOrderInfo().getProductId(),
                    context.getOrderInfo().getQuantity()
                );
            } catch (Exception e) {
                return false;
            }
        }
        
        @Override
        public void commit(TransactionContext context) {
            inventoryService.commitDeductStock(
                context.getOrderInfo().getProductId(),
                context.getOrderInfo().getQuantity()
            );
        }
        
        @Override
        public void rollback(TransactionContext context) {
            inventoryService.rollbackDeductStock(
                context.getOrderInfo().getProductId(),
                context.getOrderInfo().getQuantity()
            );
        }
    }
    
    // ==================== 2. Saga模式 ====================
    
    /**
     * Saga事务管理器
     */
    @Service
    public class SagaTransactionManager {
        
        @Autowired
        private KafkaTemplate<String, Object> kafkaTemplate;
        
        /**
         * 执行Saga事务
         */
        public void executeSagaTransaction(SagaTransaction sagaTransaction) {
            SagaContext context = new SagaContext(sagaTransaction.getId());
            
            try {
                // 按顺序执行所有步骤
                for (SagaStep step : sagaTransaction.getSteps()) {
                    step.execute(context);
                    context.addCompletedStep(step);
                }
                
                // 所有步骤执行成功
                publishSagaCompletedEvent(sagaTransaction.getId());
                
            } catch (Exception e) {
                // 执行补偿操作
                compensate(context);
                publishSagaFailedEvent(sagaTransaction.getId(), e.getMessage());
            }
        }
        
        /**
         * 执行补偿操作
         */
        private void compensate(SagaContext context) {
            // 逆序执行补偿操作
            List<SagaStep> completedSteps = context.getCompletedSteps();
            Collections.reverse(completedSteps);
            
            for (SagaStep step : completedSteps) {
                try {
                    step.compensate(context);
                } catch (Exception e) {
                    // 补偿失败，记录日志，可能需要人工介入
                    logCompensationFailure(step, e);
                }
            }
        }
        
        private void publishSagaCompletedEvent(String sagaId) {
            SagaEvent event = new SagaEvent(sagaId, "COMPLETED", LocalDateTime.now());
            kafkaTemplate.send("saga-events", event);
        }
        
        private void publishSagaFailedEvent(String sagaId, String reason) {
            SagaEvent event = new SagaEvent(sagaId, "FAILED", LocalDateTime.now());
            event.setReason(reason);
            kafkaTemplate.send("saga-events", event);
        }
        
        private void logCompensationFailure(SagaStep step, Exception e) {
            // 记录补偿失败日志，触发告警
            System.err.println("Compensation failed for step: " + step.getName() + 
                             ", error: " + e.getMessage());
        }
    }
    
    /**
     * Saga步骤接口
     */
    public interface SagaStep {
        String getName();
        void execute(SagaContext context) throws Exception;
        void compensate(SagaContext context) throws Exception;
    }
    
    /**
     * 创建订单步骤
     */
    @Component
    public class CreateOrderStep implements SagaStep {
        
        @Autowired
        private OrderService orderService;
        
        @Override
        public String getName() {
            return "CreateOrder";
        }
        
        @Override
        public void execute(SagaContext context) throws Exception {
            OrderInfo orderInfo = context.getOrderInfo();
            Order order = orderService.createOrder(orderInfo);
            context.setOrderId(order.getId());
        }
        
        @Override
        public void compensate(SagaContext context) throws Exception {
            if (context.getOrderId() != null) {
                orderService.cancelOrder(context.getOrderId());
            }
        }
    }
    
    /**
     * 扣减库存步骤
     */
    @Component
    public class DeductInventoryStep implements SagaStep {
        
        @Autowired
        private InventoryService inventoryService;
        
        @Override
        public String getName() {
            return "DeductInventory";
        }
        
        @Override
        public void execute(SagaContext context) throws Exception {
            OrderInfo orderInfo = context.getOrderInfo();
            inventoryService.deductStock(
                orderInfo.getProductId(), 
                orderInfo.getQuantity()
            );
        }
        
        @Override
        public void compensate(SagaContext context) throws Exception {
            OrderInfo orderInfo = context.getOrderInfo();
            inventoryService.restoreStock(
                orderInfo.getProductId(), 
                orderInfo.getQuantity()
            );
        }
    }
    
    /**
     * 处理支付步骤
     */
    @Component
    public class ProcessPaymentStep implements SagaStep {
        
        @Autowired
        private PaymentService paymentService;
        
        @Override
        public String getName() {
            return "ProcessPayment";
        }
        
        @Override
        public void execute(SagaContext context) throws Exception {
            OrderInfo orderInfo = context.getOrderInfo();
            PaymentResult result = paymentService.processPayment(
                context.getOrderId(),
                orderInfo.getAmount(),
                orderInfo.getPaymentMethod()
            );
            context.setPaymentId(result.getPaymentId());
        }
        
        @Override
        public void compensate(SagaContext context) throws Exception {
            if (context.getPaymentId() != null) {
                paymentService.refund(context.getPaymentId());
            }
        }
    }
    
    // ==================== 3. 事件驱动架构 ====================
    
    /**
     * 事件发布器
     */
    @Component
    public class DomainEventPublisher {
        
        @Autowired
        private KafkaTemplate<String, Object> kafkaTemplate;
        
        /**
         * 发布领域事件
         */
        public void publishEvent(DomainEvent event) {
            kafkaTemplate.send(event.getEventType(), event);
        }
        
        /**
         * 发布事件并等待确认
         */
        public void publishEventWithConfirmation(DomainEvent event) {
            try {
                kafkaTemplate.send(event.getEventType(), event).get(5, TimeUnit.SECONDS);
            } catch (Exception e) {
                throw new RuntimeException("Failed to publish event: " + event, e);
            }
        }
    }
    
    /**
     * 订单事件处理器
     */
    @Component
    public class OrderEventHandler {
        
        @Autowired
        private InventoryService inventoryService;
        
        @Autowired
        private NotificationService notificationService;
        
        /**
         * 处理订单创建事件
         */
        @KafkaListener(topics = "order-created")
        public void handleOrderCreated(OrderCreatedEvent event) {
            try {
                // 扣减库存
                inventoryService.deductStock(
                    event.getProductId(), 
                    event.getQuantity()
                );
                
                // 发送通知
                notificationService.sendOrderConfirmation(
                    event.getUserId(), 
                    event.getOrderId()
                );
                
            } catch (Exception e) {
                // 处理失败，发布补偿事件
                publishCompensationEvent(event);
            }
        }
        
        /**
         * 处理订单取消事件
         */
        @KafkaListener(topics = "order-cancelled")
        public void handleOrderCancelled(OrderCancelledEvent event) {
            // 恢复库存
            inventoryService.restoreStock(
                event.getProductId(), 
                event.getQuantity()
            );
        }
        
        private void publishCompensationEvent(OrderCreatedEvent originalEvent) {
            OrderCreationFailedEvent compensationEvent = new OrderCreationFailedEvent(
                originalEvent.getOrderId(),
                originalEvent.getUserId(),
                "Inventory deduction failed"
            );
            
            // 发布补偿事件
            kafkaTemplate.send("order-creation-failed", compensationEvent);
        }
    }
    
    // ==================== 4. 最终一致性实现 ====================
    
    /**
     * 最终一致性管理器
     */
    @Service
    public class EventualConsistencyManager {
        
        @Autowired
        private ConsistencyCheckService consistencyCheckService;
        
        @Autowired
        private CompensationService compensationService;
        
        /**
         * 定期检查数据一致性
         */
        @Scheduled(fixedDelay = 60000) // 每分钟检查一次
        public void checkDataConsistency() {
            List<InconsistencyRecord> inconsistencies = 
                consistencyCheckService.findInconsistencies();
            
            for (InconsistencyRecord record : inconsistencies) {
                try {
                    compensationService.compensate(record);
                } catch (Exception e) {
                    // 补偿失败，记录并告警
                    logInconsistencyFailure(record, e);
                }
            }
        }
        
        private void logInconsistencyFailure(InconsistencyRecord record, Exception e) {
            System.err.println("Failed to compensate inconsistency: " + record + 
                             ", error: " + e.getMessage());
        }
    }
    
    /**
     * 一致性检查服务
     */
    @Service
    public class ConsistencyCheckService {
        
        @Autowired
        private OrderRepository orderRepository;
        
        @Autowired
        private InventoryRepository inventoryRepository;
        
        /**
         * 查找数据不一致记录
         */
        public List<InconsistencyRecord> findInconsistencies() {
            List<InconsistencyRecord> inconsistencies = new ArrayList<>();
            
            // 检查订单和库存的一致性
            List<Order> orders = orderRepository.findRecentOrders();
            for (Order order : orders) {
                if (!isOrderInventoryConsistent(order)) {
                    inconsistencies.add(new InconsistencyRecord(
                        "ORDER_INVENTORY_MISMATCH",
                        order.getId(),
                        "Order exists but inventory not deducted"
                    ));
                }
            }
            
            return inconsistencies;
        }
        
        private boolean isOrderInventoryConsistent(Order order) {
            // 检查订单对应的库存是否已正确扣减
            // 这里是简化的检查逻辑
            return true;
        }
    }
    
    // ==================== 5. 数据模型 ====================
    
    /**
     * 事务上下文
     */
    public static class TransactionContext {
        private String transactionId;
        private OrderInfo orderInfo;
        private Map<String, Object> attributes;
        
        public TransactionContext(String transactionId, OrderInfo orderInfo) {
            this.transactionId = transactionId;
            this.orderInfo = orderInfo;
            this.attributes = new HashMap<>();
        }
        
        // getters and setters
        public String getTransactionId() { return transactionId; }
        public OrderInfo getOrderInfo() { return orderInfo; }
        public Map<String, Object> getAttributes() { return attributes; }
    }
    
    /**
     * Saga上下文
     */
    public static class SagaContext {
        private String sagaId;
        private OrderInfo orderInfo;
        private Long orderId;
        private String paymentId;
        private List<SagaStep> completedSteps;
        
        public SagaContext(String sagaId) {
            this.sagaId = sagaId;
            this.completedSteps = new ArrayList<>();
        }
        
        public void addCompletedStep(SagaStep step) {
            completedSteps.add(step);
        }
        
        // getters and setters
        public String getSagaId() { return sagaId; }
        public OrderInfo getOrderInfo() { return orderInfo; }
        public void setOrderInfo(OrderInfo orderInfo) { this.orderInfo = orderInfo; }
        public Long getOrderId() { return orderId; }
        public void setOrderId(Long orderId) { this.orderId = orderId; }
        public String getPaymentId() { return paymentId; }
        public void setPaymentId(String paymentId) { this.paymentId = paymentId; }
        public List<SagaStep> getCompletedSteps() { return completedSteps; }
    }
    
    /**
     * 领域事件基类
     */
    public abstract static class DomainEvent {
        private String eventId;
        private String eventType;
        private LocalDateTime occurredOn;
        
        public DomainEvent(String eventType) {
            this.eventId = UUID.randomUUID().toString();
            this.eventType = eventType;
            this.occurredOn = LocalDateTime.now();
        }
        
        // getters
        public String getEventId() { return eventId; }
        public String getEventType() { return eventType; }
        public LocalDateTime getOccurredOn() { return occurredOn; }
    }
    
    /**
     * 订单创建事件
     */
    public static class OrderCreatedEvent extends DomainEvent {
        private Long orderId;
        private Long userId;
        private Long productId;
        private Integer quantity;
        
        public OrderCreatedEvent(Long orderId, Long userId, Long productId, Integer quantity) {
            super("order-created");
            this.orderId = orderId;
            this.userId = userId;
            this.productId = productId;
            this.quantity = quantity;
        }
        
        // getters
        public Long getOrderId() { return orderId; }
        public Long getUserId() { return userId; }
        public Long getProductId() { return productId; }
        public Integer getQuantity() { return quantity; }
    }
}
```

**关键技术点：**

1. **分布式事务（2PC）**：
   - 适用于强一致性要求的场景
   - 性能开销大，可用性较低
   - 需要所有参与者支持事务

2. **Saga模式**：
   - 适用于长事务和复杂业务流程
   - 通过补偿操作保证最终一致性
   - 实现相对复杂，需要设计补偿逻辑

3. **事件驱动架构**：
   - 松耦合，高可用性
   - 异步处理，性能好
   - 需要处理消息重复和顺序问题

4. **最终一致性**：
   - 性能最好，可用性最高
   - 需要业务能容忍短暂的不一致
   - 需要补偿和对账机制

**选择建议：**

- **金融支付**：使用2PC或TCC保证强一致性
- **电商订单**：使用Saga模式处理复杂流程
- **用户信息**：使用事件驱动实现最终一致性
- **日志统计**：可以接受弱一致性

**3. 如何设计一个分布式缓存系统？**

分布式缓存系统需要解决数据分布、一致性、高可用性和性能等核心问题。

**系统架构设计：**

| 组件 | 功能 | 实现方式 | 关键技术 |
|------|------|----------|----------|
| 客户端 | 请求路由、负载均衡 | 智能客户端 | 一致性哈希、故障检测 |
| 代理层 | 请求转发、协议转换 | Proxy模式 | 连接池、请求聚合 |
| 缓存节点 | 数据存储、处理 | 分布式节点 | 内存管理、持久化 |
| 配置中心 | 集群管理、元数据 | 注册中心 | 服务发现、配置同步 |

**详细实现方案：**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.stereotype.Service;
import org.springframework.stereotype.Component;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.data.redis.core.RedisTemplate;

import java.util.*;
import java.util.concurrent.*;
import java.security.MessageDigest;
import java.time.LocalDateTime;
import java.nio.charset.StandardCharsets;

/**
 * 分布式缓存系统实现
 */
@SpringBootApplication
public class DistributedCacheApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(DistributedCacheApplication.class, args);
    }
    
    // ==================== 1. 一致性哈希实现 ====================
    
    /**
     * 一致性哈希环
     */
    @Component
    public class ConsistentHashRing {
        
        private final TreeMap<Long, CacheNode> ring = new TreeMap<>();
        private final int virtualNodes = 150; // 每个物理节点对应的虚拟节点数
        private final List<CacheNode> physicalNodes = new ArrayList<>();
        
        /**
         * 添加缓存节点
         */
        public synchronized void addNode(CacheNode node) {
            physicalNodes.add(node);
            
            // 为每个物理节点创建虚拟节点
            for (int i = 0; i < virtualNodes; i++) {
                String virtualNodeKey = node.getNodeId() + "#" + i;
                long hash = hash(virtualNodeKey);
                ring.put(hash, node);
            }
            
            System.out.println("Added node: " + node.getNodeId() + 
                             ", total virtual nodes: " + ring.size());
        }
        
        /**
         * 移除缓存节点
         */
        public synchronized void removeNode(CacheNode node) {
            physicalNodes.remove(node);
            
            // 移除所有虚拟节点
            for (int i = 0; i < virtualNodes; i++) {
                String virtualNodeKey = node.getNodeId() + "#" + i;
                long hash = hash(virtualNodeKey);
                ring.remove(hash);
            }
            
            System.out.println("Removed node: " + node.getNodeId());
        }
        
        /**
         * 根据key获取对应的缓存节点
         */
        public CacheNode getNode(String key) {
            if (ring.isEmpty()) {
                return null;
            }
            
            long hash = hash(key);
            
            // 查找第一个大于等于hash值的节点
            Map.Entry<Long, CacheNode> entry = ring.ceilingEntry(hash);
            
            // 如果没找到，说明应该路由到第一个节点（环形结构）
            if (entry == null) {
                entry = ring.firstEntry();
            }
            
            return entry.getValue();
        }
        
        /**
         * 获取key的副本节点列表
         */
        public List<CacheNode> getReplicaNodes(String key, int replicaCount) {
            List<CacheNode> replicas = new ArrayList<>();
            Set<String> addedNodes = new HashSet<>();
            
            long hash = hash(key);
            
            // 从hash位置开始，顺时针查找不同的物理节点
            NavigableMap<Long, CacheNode> tailMap = ring.tailMap(hash, true);
            addUniqueNodes(tailMap.values(), replicas, addedNodes, replicaCount);
            
            // 如果还没找够，从头开始找
            if (replicas.size() < replicaCount) {
                addUniqueNodes(ring.values(), replicas, addedNodes, replicaCount);
            }
            
            return replicas;
        }
        
        private void addUniqueNodes(Collection<CacheNode> nodes, 
                                   List<CacheNode> replicas, 
                                   Set<String> addedNodes, 
                                   int replicaCount) {
            for (CacheNode node : nodes) {
                if (replicas.size() >= replicaCount) {
                    break;
                }
                
                if (!addedNodes.contains(node.getNodeId())) {
                    replicas.add(node);
                    addedNodes.add(node.getNodeId());
                }
            }
        }
        
        /**
         * 计算hash值
         */
        private long hash(String key) {
            try {
                MessageDigest md = MessageDigest.getInstance("MD5");
                byte[] digest = md.digest(key.getBytes(StandardCharsets.UTF_8));
                
                long hash = 0;
                for (int i = 0; i < 4; i++) {
                    hash <<= 8;
                    hash |= ((int) digest[i]) & 0xFF;
                }
                
                return hash;
            } catch (Exception e) {
                throw new RuntimeException("Hash calculation failed", e);
            }
        }
        
        /**
         * 获取所有物理节点
         */
        public List<CacheNode> getPhysicalNodes() {
            return new ArrayList<>(physicalNodes);
        }
    }
    
    // ==================== 2. 缓存节点实现 ====================
    
    /**
     * 缓存节点
     */
    public static class CacheNode {
        private final String nodeId;
        private final String host;
        private final int port;
        private final ConcurrentHashMap<String, CacheEntry> cache;
        private final ScheduledExecutorService cleanupExecutor;
        private volatile boolean healthy = true;
        
        public CacheNode(String nodeId, String host, int port) {
            this.nodeId = nodeId;
            this.host = host;
            this.port = port;
            this.cache = new ConcurrentHashMap<>();
            this.cleanupExecutor = Executors.newSingleThreadScheduledExecutor();
            
            // 启动过期数据清理任务
            startCleanupTask();
        }
        
        /**
         * 存储数据
         */
        public void put(String key, Object value, long ttlMillis) {
            long expireTime = ttlMillis > 0 ? 
                System.currentTimeMillis() + ttlMillis : Long.MAX_VALUE;
            
            CacheEntry entry = new CacheEntry(value, expireTime, System.currentTimeMillis());
            cache.put(key, entry);
        }
        
        /**
         * 获取数据
         */
        public Object get(String key) {
            CacheEntry entry = cache.get(key);
            
            if (entry == null) {
                return null;
            }
            
            // 检查是否过期
            if (entry.isExpired()) {
                cache.remove(key);
                return null;
            }
            
            // 更新访问时间（用于LRU）
            entry.updateAccessTime();
            return entry.getValue();
        }
        
        /**
         * 删除数据
         */
        public void delete(String key) {
            cache.remove(key);
        }
        
        /**
         * 检查key是否存在
         */
        public boolean exists(String key) {
            CacheEntry entry = cache.get(key);
            return entry != null && !entry.isExpired();
        }
        
        /**
         * 获取缓存统计信息
         */
        public CacheStats getStats() {
            int totalEntries = cache.size();
            int expiredEntries = 0;
            long totalMemory = 0;
            
            for (CacheEntry entry : cache.values()) {
                if (entry.isExpired()) {
                    expiredEntries++;
                } else {
                    totalMemory += entry.getEstimatedSize();
                }
            }
            
            return new CacheStats(totalEntries, expiredEntries, totalMemory);
        }
        
        /**
         * 启动清理任务
         */
        private void startCleanupTask() {
            cleanupExecutor.scheduleAtFixedRate(() -> {
                try {
                    cleanupExpiredEntries();
                } catch (Exception e) {
                    System.err.println("Cleanup task failed for node " + nodeId + ": " + e.getMessage());
                }
            }, 60, 60, TimeUnit.SECONDS); // 每分钟清理一次
        }
        
        /**
         * 清理过期数据
         */
        private void cleanupExpiredEntries() {
            int removedCount = 0;
            Iterator<Map.Entry<String, CacheEntry>> iterator = cache.entrySet().iterator();
            
            while (iterator.hasNext()) {
                Map.Entry<String, CacheEntry> entry = iterator.next();
                if (entry.getValue().isExpired()) {
                    iterator.remove();
                    removedCount++;
                }
            }
            
            if (removedCount > 0) {
                System.out.println("Node " + nodeId + " cleaned up " + removedCount + " expired entries");
            }
        }
        
        // getters
        public String getNodeId() { return nodeId; }
        public String getHost() { return host; }
        public int getPort() { return port; }
        public boolean isHealthy() { return healthy; }
        public void setHealthy(boolean healthy) { this.healthy = healthy; }
        
        @Override
        public boolean equals(Object obj) {
            if (this == obj) return true;
            if (obj == null || getClass() != obj.getClass()) return false;
            CacheNode cacheNode = (CacheNode) obj;
            return Objects.equals(nodeId, cacheNode.nodeId);
        }
        
        @Override
        public int hashCode() {
            return Objects.hash(nodeId);
        }
    }
    
    /**
     * 缓存条目
     */
    public static class CacheEntry {
        private final Object value;
        private final long expireTime;
        private volatile long lastAccessTime;
        
        public CacheEntry(Object value, long expireTime, long createTime) {
            this.value = value;
            this.expireTime = expireTime;
            this.lastAccessTime = createTime;
        }
        
        public boolean isExpired() {
            return expireTime != Long.MAX_VALUE && System.currentTimeMillis() > expireTime;
        }
        
        public void updateAccessTime() {
            this.lastAccessTime = System.currentTimeMillis();
        }
        
        public long getEstimatedSize() {
            // 简化的内存估算
            if (value instanceof String) {
                return ((String) value).length() * 2; // 假设每个字符2字节
            }
            return 100; // 其他对象假设100字节
        }
        
        // getters
        public Object getValue() { return value; }
        public long getExpireTime() { return expireTime; }
        public long getLastAccessTime() { return lastAccessTime; }
    }
    
    // ==================== 3. 分布式缓存客户端 ====================
    
    /**
     * 分布式缓存客户端
     */
    @Service
    public class DistributedCacheClient {
        
        @Autowired
        private ConsistentHashRing hashRing;
        
        @Autowired
        private CacheReplicationManager replicationManager;
        
        private final int replicaCount = 2; // 副本数量
        
        /**
         * 存储数据
         */
        public void put(String key, Object value, long ttlMillis) {
            List<CacheNode> nodes = hashRing.getReplicaNodes(key, replicaCount);
            
            if (nodes.isEmpty()) {
                throw new RuntimeException("No available cache nodes");
            }
            
            // 并行写入所有副本
            CompletableFuture<Void>[] futures = nodes.stream()
                .map(node -> CompletableFuture.runAsync(() -> {
                    try {
                        node.put(key, value, ttlMillis);
                    } catch (Exception e) {
                        System.err.println("Failed to put to node " + node.getNodeId() + ": " + e.getMessage());
                    }
                }))
                .toArray(CompletableFuture[]::new);
            
            try {
                // 等待至少一个副本写入成功
                CompletableFuture.anyOf(futures).get(1, TimeUnit.SECONDS);
            } catch (Exception e) {
                throw new RuntimeException("Failed to put cache entry", e);
            }
        }
        
        /**
         * 获取数据
         */
        public Object get(String key) {
            List<CacheNode> nodes = hashRing.getReplicaNodes(key, replicaCount);
            
            // 按优先级尝试读取
            for (CacheNode node : nodes) {
                if (!node.isHealthy()) {
                    continue;
                }
                
                try {
                    Object value = node.get(key);
                    if (value != null) {
                        return value;
                    }
                } catch (Exception e) {
                    System.err.println("Failed to get from node " + node.getNodeId() + ": " + e.getMessage());
                    // 标记节点不健康
                    node.setHealthy(false);
                }
            }
            
            return null;
        }
        
        /**
         * 删除数据
         */
        public void delete(String key) {
            List<CacheNode> nodes = hashRing.getReplicaNodes(key, replicaCount);
            
            // 并行删除所有副本
            nodes.parallelStream().forEach(node -> {
                try {
                    node.delete(key);
                } catch (Exception e) {
                    System.err.println("Failed to delete from node " + node.getNodeId() + ": " + e.getMessage());
                }
            });
        }
        
        /**
         * 检查key是否存在
         */
        public boolean exists(String key) {
            List<CacheNode> nodes = hashRing.getReplicaNodes(key, replicaCount);
            
            for (CacheNode node : nodes) {
                if (node.isHealthy()) {
                    try {
                        if (node.exists(key)) {
                            return true;
                        }
                    } catch (Exception e) {
                        System.err.println("Failed to check existence on node " + node.getNodeId() + ": " + e.getMessage());
                    }
                }
            }
            
            return false;
        }
    }
    
    // ==================== 4. 副本管理 ====================
    
    /**
     * 缓存副本管理器
     */
    @Service
    public class CacheReplicationManager {
        
        @Autowired
        private ConsistentHashRing hashRing;
        
        /**
         * 数据迁移（节点加入时）
         */
        public void migrateDataForNewNode(CacheNode newNode) {
            List<CacheNode> allNodes = hashRing.getPhysicalNodes();
            
            // 为新节点分配数据
            for (CacheNode sourceNode : allNodes) {
                if (sourceNode.equals(newNode)) {
                    continue;
                }
                
                migrateDataBetweenNodes(sourceNode, newNode);
            }
        }
        
        /**
         * 数据迁移（节点离开时）
         */
        public void migrateDataForRemovedNode(CacheNode removedNode) {
            List<CacheNode> remainingNodes = hashRing.getPhysicalNodes();
            
            if (remainingNodes.isEmpty()) {
                System.err.println("No remaining nodes for data migration");
                return;
            }
            
            // 将移除节点的数据迁移到其他节点
            for (CacheNode targetNode : remainingNodes) {
                migrateDataBetweenNodes(removedNode, targetNode);
            }
        }
        
        /**
         * 节点间数据迁移
         */
        private void migrateDataBetweenNodes(CacheNode sourceNode, CacheNode targetNode) {
            try {
                // 这里是简化的迁移逻辑
                // 实际实现中需要考虑数据一致性、迁移进度等
                System.out.println("Migrating data from " + sourceNode.getNodeId() + 
                                 " to " + targetNode.getNodeId());
                
                // 实际的数据迁移逻辑
                // ...
                
            } catch (Exception e) {
                System.err.println("Data migration failed: " + e.getMessage());
            }
        }
        
        /**
         * 副本同步
         */
        @Scheduled(fixedDelay = 300000) // 每5分钟同步一次
        public void syncReplicas() {
            List<CacheNode> nodes = hashRing.getPhysicalNodes();
            
            for (CacheNode node : nodes) {
                if (node.isHealthy()) {
                    try {
                        syncNodeReplicas(node);
                    } catch (Exception e) {
                        System.err.println("Replica sync failed for node " + node.getNodeId() + ": " + e.getMessage());
                    }
                }
            }
        }
        
        private void syncNodeReplicas(CacheNode node) {
            // 副本同步逻辑
            // 检查数据一致性，修复不一致的数据
        }
    }
    
    // ==================== 5. 健康检查和故障恢复 ====================
    
    /**
     * 节点健康检查器
     */
    @Component
    public class NodeHealthChecker {
        
        @Autowired
        private ConsistentHashRing hashRing;
        
        @Autowired
        private CacheReplicationManager replicationManager;
        
        /**
         * 定期健康检查
         */
        @Scheduled(fixedDelay = 30000) // 每30秒检查一次
        public void checkNodeHealth() {
            List<CacheNode> nodes = hashRing.getPhysicalNodes();
            
            for (CacheNode node : nodes) {
                boolean isHealthy = performHealthCheck(node);
                
                if (node.isHealthy() && !isHealthy) {
                    // 节点变为不健康
                    node.setHealthy(false);
                    System.err.println("Node " + node.getNodeId() + " marked as unhealthy");
                    
                    // 触发故障转移
                    handleNodeFailure(node);
                    
                } else if (!node.isHealthy() && isHealthy) {
                    // 节点恢复健康
                    node.setHealthy(true);
                    System.out.println("Node " + node.getNodeId() + " recovered");
                    
                    // 触发数据恢复
                    handleNodeRecovery(node);
                }
            }
        }
        
        /**
         * 执行健康检查
         */
        private boolean performHealthCheck(CacheNode node) {
            try {
                // 简单的健康检查：尝试存储和读取一个测试值
                String testKey = "__health_check__" + System.currentTimeMillis();
                String testValue = "test";
                
                node.put(testKey, testValue, 5000); // 5秒TTL
                Object result = node.get(testKey);
                node.delete(testKey);
                
                return testValue.equals(result);
                
            } catch (Exception e) {
                return false;
            }
        }
        
        /**
         * 处理节点故障
         */
        private void handleNodeFailure(CacheNode failedNode) {
            // 数据迁移到其他健康节点
            replicationManager.migrateDataForRemovedNode(failedNode);
        }
        
        /**
         * 处理节点恢复
         */
        private void handleNodeRecovery(CacheNode recoveredNode) {
            // 恢复节点的数据
            replicationManager.migrateDataForNewNode(recoveredNode);
        }
    }
    
    // ==================== 6. 缓存统计和监控 ====================
    
    /**
     * 缓存统计信息
     */
    public static class CacheStats {
        private final int totalEntries;
        private final int expiredEntries;
        private final long totalMemoryBytes;
        
        public CacheStats(int totalEntries, int expiredEntries, long totalMemoryBytes) {
            this.totalEntries = totalEntries;
            this.expiredEntries = expiredEntries;
            this.totalMemoryBytes = totalMemoryBytes;
        }
        
        // getters
        public int getTotalEntries() { return totalEntries; }
        public int getExpiredEntries() { return expiredEntries; }
        public long getTotalMemoryBytes() { return totalMemoryBytes; }
        public int getActiveEntries() { return totalEntries - expiredEntries; }
    }
    
    /**
     * 缓存监控服务
     */
    @Service
    public class CacheMonitoringService {
        
        @Autowired
        private ConsistentHashRing hashRing;
        
        /**
         * 获取集群统计信息
         */
        public ClusterStats getClusterStats() {
            List<CacheNode> nodes = hashRing.getPhysicalNodes();
            
            int totalNodes = nodes.size();
            int healthyNodes = 0;
            int totalEntries = 0;
            long totalMemory = 0;
            
            for (CacheNode node : nodes) {
                if (node.isHealthy()) {
                    healthyNodes++;
                    CacheStats stats = node.getStats();
                    totalEntries += stats.getActiveEntries();
                    totalMemory += stats.getTotalMemoryBytes();
                }
            }
            
            return new ClusterStats(totalNodes, healthyNodes, totalEntries, totalMemory);
        }
        
        /**
         * 获取节点详细信息
         */
        public List<NodeInfo> getNodeInfos() {
            List<CacheNode> nodes = hashRing.getPhysicalNodes();
            List<NodeInfo> nodeInfos = new ArrayList<>();
            
            for (CacheNode node : nodes) {
                CacheStats stats = node.getStats();
                NodeInfo info = new NodeInfo(
                    node.getNodeId(),
                    node.getHost(),
                    node.getPort(),
                    node.isHealthy(),
                    stats
                );
                nodeInfos.add(info);
            }
            
            return nodeInfos;
        }
    }
    
    /**
     * 集群统计信息
     */
    public static class ClusterStats {
        private final int totalNodes;
        private final int healthyNodes;
        private final int totalEntries;
        private final long totalMemoryBytes;
        
        public ClusterStats(int totalNodes, int healthyNodes, int totalEntries, long totalMemoryBytes) {
            this.totalNodes = totalNodes;
            this.healthyNodes = healthyNodes;
            this.totalEntries = totalEntries;
            this.totalMemoryBytes = totalMemoryBytes;
        }
        
        // getters
        public int getTotalNodes() { return totalNodes; }
        public int getHealthyNodes() { return healthyNodes; }
        public int getTotalEntries() { return totalEntries; }
        public long getTotalMemoryBytes() { return totalMemoryBytes; }
    }
    
    /**
     * 节点信息
     */
    public static class NodeInfo {
        private final String nodeId;
        private final String host;
        private final int port;
        private final boolean healthy;
        private final CacheStats stats;
        
        public NodeInfo(String nodeId, String host, int port, boolean healthy, CacheStats stats) {
            this.nodeId = nodeId;
            this.host = host;
            this.port = port;
            this.healthy = healthy;
            this.stats = stats;
        }
        
        // getters
        public String getNodeId() { return nodeId; }
        public String getHost() { return host; }
        public int getPort() { return port; }
        public boolean isHealthy() { return healthy; }
        public CacheStats getStats() { return stats; }
    }
}
```

**关键技术点：**

1. **一致性哈希**：
   - 解决节点增减时的数据迁移问题
   - 虚拟节点提高数据分布均匀性
   - 支持节点权重和故障转移

2. **数据分片策略**：
   - 基于key的哈希分片
   - 支持范围分片和目录分片
   - 动态分片调整

3. **副本机制**：
   - 多副本保证高可用性
   - 异步复制提高性能
   - 副本一致性检查和修复

4. **缓存淘汰算法**：
   - LRU（最近最少使用）
   - LFU（最少使用频率）
   - TTL（生存时间）
   - 内存压力触发清理

5. **故障处理**：
   - 节点健康检查
   - 自动故障转移
   - 数据迁移和恢复

**性能优化建议：**

- **客户端优化**：连接池、批量操作、本地缓存
- **网络优化**：协议压缩、连接复用、异步IO
- **内存优化**：对象池、序列化优化、内存映射
- **监控告警**：性能指标、错误率、容量规划

### 4.2 性能优化类问题

**4. JVM调优的主要参数和策略是什么？**

**答案要点：**
- **内存参数**：-Xms、-Xmx、-XX:NewRatio
- **GC参数**：-XX:+UseG1GC、-XX:MaxGCPauseMillis
- **监控参数**：-XX:+PrintGCDetails、-XX:+HeapDumpOnOutOfMemoryError
- **调优策略**：根据应用特点选择合适的GC算法

**5. 数据库查询性能优化有哪些方法？**

数据库查询性能优化是系统性能提升的关键环节，需要从多个维度进行综合优化。

**优化策略对比：**

| 优化类型 | 适用场景 | 优化效果 | 实施难度 | 成本 |
|----------|----------|----------|----------|---------|
| 索引优化 | 查询频繁的字段 | 显著提升 | 低 | 低 |
| SQL优化 | 复杂查询语句 | 中等提升 | 中 | 低 |
| 分库分表 | 大数据量场景 | 显著提升 | 高 | 高 |
| 读写分离 | 读多写少场景 | 中等提升 | 中 | 中 |
| 缓存策略 | 热点数据访问 | 显著提升 | 中 | 中 |

**详细实现方案：**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.stereotype.Service;
import org.springframework.stereotype.Repository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

import javax.sql.DataSource;
import javax.persistence.*;
import java.util.*;
import java.time.LocalDateTime;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ConcurrentHashMap;

/**
 * 数据库查询性能优化实现
 */
@SpringBootApplication
public class DatabaseOptimizationApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(DatabaseOptimizationApplication.class, args);
    }
    
    // ==================== 1. 索引优化策略 ====================
    
    /**
     * 用户实体 - 演示索引优化
     */
    @Entity
    @Table(name = "users", indexes = {
        @Index(name = "idx_email", columnList = "email", unique = true),
        @Index(name = "idx_status_created", columnList = "status, created_time"),
        @Index(name = "idx_department_salary", columnList = "department_id, salary"),
        @Index(name = "idx_name_prefix", columnList = "name(10)") // 前缀索引
    })
    public class User {
        
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
        
        @Column(nullable = false, length = 100)
        private String name;
        
        @Column(unique = true, nullable = false)
        private String email;
        
        @Column(nullable = false)
        private Integer status;
        
        @Column(name = "department_id")
        private Long departmentId;
        
        @Column(precision = 10, scale = 2)
        private Double salary;
        
        @Column(name = "created_time")
        private LocalDateTime createdTime;
        
        @Column(name = "updated_time")
        private LocalDateTime updatedTime;
        
        // getters and setters
        public Long getId() { return id; }
        public void setId(Long id) { this.id = id; }
        public String getName() { return name; }
        public void setName(String name) { this.name = name; }
        public String getEmail() { return email; }
        public void setEmail(String email) { this.email = email; }
        public Integer getStatus() { return status; }
        public void setStatus(Integer status) { this.status = status; }
        public Long getDepartmentId() { return departmentId; }
        public void setDepartmentId(Long departmentId) { this.departmentId = departmentId; }
        public Double getSalary() { return salary; }
        public void setSalary(Double salary) { this.salary = salary; }
        public LocalDateTime getCreatedTime() { return createdTime; }
        public void setCreatedTime(LocalDateTime createdTime) { this.createdTime = createdTime; }
        public LocalDateTime getUpdatedTime() { return updatedTime; }
        public void setUpdatedTime(LocalDateTime updatedTime) { this.updatedTime = updatedTime; }
    }
    
    /**
     * 索引优化服务
     */
    @Service
    public class IndexOptimizationService {
        
        @Autowired
        private JdbcTemplate jdbcTemplate;
        
        /**
         * 分析查询执行计划
         */
        public QueryExecutionPlan analyzeQuery(String sql) {
            String explainSql = "EXPLAIN ANALYZE " + sql;
            
            List<Map<String, Object>> result = jdbcTemplate.queryForList(explainSql);
            
            QueryExecutionPlan plan = new QueryExecutionPlan();
            
            for (Map<String, Object> row : result) {
                String selectType = (String) row.get("select_type");
                String table = (String) row.get("table");
                String type = (String) row.get("type");
                String possibleKeys = (String) row.get("possible_keys");
                String key = (String) row.get("key");
                Long rows = (Long) row.get("rows");
                String extra = (String) row.get("Extra");
                
                ExecutionStep step = new ExecutionStep(
                    selectType, table, type, possibleKeys, key, rows, extra
                );
                plan.addStep(step);
                
                // 检查性能问题
                if ("ALL".equals(type)) {
                    plan.addWarning("全表扫描: " + table);
                }
                
                if (rows != null && rows > 10000) {
                    plan.addWarning("扫描行数过多: " + table + " (" + rows + " rows)");
                }
                
                if (extra != null && extra.contains("Using filesort")) {
                    plan.addWarning("使用文件排序: " + table);
                }
                
                if (extra != null && extra.contains("Using temporary")) {
                    plan.addWarning("使用临时表: " + table);
                }
            }
            
            return plan;
        }
        
        /**
         * 推荐索引创建
         */
        public List<IndexRecommendation> recommendIndexes(String tableName) {
            List<IndexRecommendation> recommendations = new ArrayList<>();
            
            // 分析慢查询日志
            String slowQuerySql = 
                "SELECT sql_text, execution_count, avg_timer_wait/1000000000 as avg_time_sec " +
                "FROM performance_schema.events_statements_summary_by_digest " +
                "WHERE schema_name = DATABASE() AND table_names LIKE '%" + tableName + "%' " +
                "ORDER BY avg_timer_wait DESC LIMIT 10";
            
            List<Map<String, Object>> slowQueries = jdbcTemplate.queryForList(slowQuerySql);
            
            for (Map<String, Object> query : slowQueries) {
                String sqlText = (String) query.get("sql_text");
                Double avgTime = (Double) query.get("avg_time_sec");
                
                if (avgTime > 1.0) { // 超过1秒的查询
                    List<String> suggestedIndexes = analyzeSqlForIndexes(sqlText);
                    
                    for (String indexSuggestion : suggestedIndexes) {
                        recommendations.add(new IndexRecommendation(
                            tableName, indexSuggestion, avgTime, sqlText
                        ));
                    }
                }
            }
            
            return recommendations;
        }
        
        /**
         * 分析SQL语句推荐索引
         */
        private List<String> analyzeSqlForIndexes(String sql) {
            List<String> suggestions = new ArrayList<>();
            
            // 简化的SQL分析逻辑
            if (sql.contains("WHERE")) {
                // 提取WHERE条件中的字段
                String whereClause = sql.substring(sql.indexOf("WHERE") + 5);
                
                if (whereClause.contains("status =")) {
                    suggestions.add("CREATE INDEX idx_status ON users(status)");
                }
                
                if (whereClause.contains("created_time")) {
                    suggestions.add("CREATE INDEX idx_created_time ON users(created_time)");
                }
                
                if (whereClause.contains("department_id") && whereClause.contains("salary")) {
                    suggestions.add("CREATE INDEX idx_dept_salary ON users(department_id, salary)");
                }
            }
            
            if (sql.contains("ORDER BY")) {
                String orderClause = sql.substring(sql.indexOf("ORDER BY") + 8);
                suggestions.add("CREATE INDEX idx_order ON users(" + orderClause.trim() + ")");
            }
            
            return suggestions;
        }
    }
    
    // ==================== 2. SQL优化实现 ====================
    
    /**
     * SQL优化服务
     */
    @Service
    public class SqlOptimizationService {
        
        @Autowired
        private JdbcTemplate jdbcTemplate;
        
        /**
         * 优化JOIN查询
         */
        public List<UserDepartmentDto> getOptimizedUserDepartments(List<Long> userIds) {
            // 原始低效查询（避免）
            // SELECT u.*, d.name as dept_name FROM users u, departments d WHERE u.department_id = d.id
            
            // 优化后的查询
            String optimizedSql = 
                "SELECT u.id, u.name, u.email, u.salary, d.name as dept_name " +
                "FROM users u " +
                "INNER JOIN departments d ON u.department_id = d.id " +
                "WHERE u.id IN (" + String.join(",", Collections.nCopies(userIds.size(), "?")) + ") " +
                "AND u.status = 1 " +
                "ORDER BY u.salary DESC";
            
            return jdbcTemplate.query(optimizedSql, userIds.toArray(), (rs, rowNum) -> {
                UserDepartmentDto dto = new UserDepartmentDto();
                dto.setId(rs.getLong("id"));
                dto.setName(rs.getString("name"));
                dto.setEmail(rs.getString("email"));
                dto.setSalary(rs.getDouble("salary"));
                dto.setDeptName(rs.getString("dept_name"));
                return dto;
            });
        }
        
        /**
         * 批量操作优化
         */
        @Transactional
        public void batchUpdateUserSalary(List<UserSalaryUpdate> updates) {
            // 避免逐条更新
            // for (UserSalaryUpdate update : updates) {
            //     jdbcTemplate.update("UPDATE users SET salary = ? WHERE id = ?", 
            //                        update.getSalary(), update.getUserId());
            // }
            
            // 使用批量更新
            String sql = "UPDATE users SET salary = ? WHERE id = ?";
            
            List<Object[]> batchArgs = updates.stream()
                .map(update -> new Object[]{update.getSalary(), update.getUserId()})
                .collect(java.util.stream.Collectors.toList());
            
            jdbcTemplate.batchUpdate(sql, batchArgs);
        }
        
        /**
         * 分页查询优化
         */
        public PageResult<User> getOptimizedUserPage(int page, int size, String department) {
            // 避免使用OFFSET进行深分页
            // SELECT * FROM users WHERE department = ? ORDER BY id LIMIT ? OFFSET ?
            
            // 使用游标分页
            Long lastId = getLastIdFromPreviousPage(page, size);
            
            String sql;
            Object[] params;
            
            if (lastId == null) {
                // 第一页
                sql = "SELECT * FROM users WHERE department_id = ? ORDER BY id LIMIT ?";
                params = new Object[]{department, size};
            } else {
                // 后续页面
                sql = "SELECT * FROM users WHERE department_id = ? AND id > ? ORDER BY id LIMIT ?";
                params = new Object[]{department, lastId, size};
            }
            
            List<User> users = jdbcTemplate.query(sql, params, (rs, rowNum) -> {
                User user = new User();
                user.setId(rs.getLong("id"));
                user.setName(rs.getString("name"));
                user.setEmail(rs.getString("email"));
                user.setStatus(rs.getInt("status"));
                user.setDepartmentId(rs.getLong("department_id"));
                user.setSalary(rs.getDouble("salary"));
                return user;
            });
            
            // 获取总数（仅在需要时计算）
            String countSql = "SELECT COUNT(*) FROM users WHERE department_id = ?";
            Long total = jdbcTemplate.queryForObject(countSql, Long.class, department);
            
            return new PageResult<>(users, total, page, size);
        }
        
        private Long getLastIdFromPreviousPage(int page, int size) {
            if (page <= 1) return null;
            
            // 这里可以从缓存或其他方式获取上一页的最后一个ID
            // 简化实现
            return (long) ((page - 1) * size);
        }
        
        /**
         * 子查询优化
         */
        public List<User> getHighSalaryUsers(String department) {
            // 避免相关子查询
            // SELECT * FROM users u1 WHERE salary > 
            //   (SELECT AVG(salary) FROM users u2 WHERE u2.department_id = u1.department_id)
            
            // 优化为JOIN查询
            String optimizedSql = 
                "SELECT u.* FROM users u " +
                "INNER JOIN ( " +
                "  SELECT department_id, AVG(salary) as avg_salary " +
                "  FROM users " +
                "  GROUP BY department_id " +
                ") dept_avg ON u.department_id = dept_avg.department_id " +
                "WHERE u.salary > dept_avg.avg_salary " +
                "AND u.department_id = ?";
            
            return jdbcTemplate.query(optimizedSql, new Object[]{department}, (rs, rowNum) -> {
                User user = new User();
                user.setId(rs.getLong("id"));
                user.setName(rs.getString("name"));
                user.setEmail(rs.getString("email"));
                user.setSalary(rs.getDouble("salary"));
                return user;
            });
        }
    }
    
    // ==================== 3. 读写分离实现 ====================
    
    /**
     * 数据源配置
     */
    @Configuration
    public class DataSourceConfig {
        
        @Bean
        @Primary
        public DataSource routingDataSource() {
            DynamicDataSource dynamicDataSource = new DynamicDataSource();
            
            Map<Object, Object> dataSourceMap = new HashMap<>();
            dataSourceMap.put("master", masterDataSource());
            dataSourceMap.put("slave1", slave1DataSource());
            dataSourceMap.put("slave2", slave2DataSource());
            
            dynamicDataSource.setTargetDataSources(dataSourceMap);
            dynamicDataSource.setDefaultTargetDataSource(masterDataSource());
            
            return dynamicDataSource;
        }
        
        private DataSource masterDataSource() {
            // 主库配置
            org.springframework.boot.jdbc.DataSourceBuilder<?> builder = 
                org.springframework.boot.jdbc.DataSourceBuilder.create();
            return builder
                .url("jdbc:mysql://master-db:3306/testdb")
                .username("root")
                .password("password")
                .driverClassName("com.mysql.cj.jdbc.Driver")
                .build();
        }
        
        private DataSource slave1DataSource() {
            // 从库1配置
            org.springframework.boot.jdbc.DataSourceBuilder<?> builder = 
                org.springframework.boot.jdbc.DataSourceBuilder.create();
            return builder
                .url("jdbc:mysql://slave1-db:3306/testdb")
                .username("root")
                .password("password")
                .driverClassName("com.mysql.cj.jdbc.Driver")
                .build();
        }
        
        private DataSource slave2DataSource() {
            // 从库2配置
            org.springframework.boot.jdbc.DataSourceBuilder<?> builder = 
                org.springframework.boot.jdbc.DataSourceBuilder.create();
            return builder
                .url("jdbc:mysql://slave2-db:3306/testdb")
                .username("root")
                .password("password")
                .driverClassName("com.mysql.cj.jdbc.Driver")
                .build();
        }
    }
    
    /**
     * 动态数据源
     */
    public class DynamicDataSource extends org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource {
        
        @Override
        protected Object determineCurrentLookupKey() {
            return DataSourceContextHolder.getDataSourceType();
        }
    }
    
    /**
     * 数据源上下文持有者
     */
    public class DataSourceContextHolder {
        
        private static final ThreadLocal<String> contextHolder = new ThreadLocal<>();
        
        public static void setDataSourceType(String dataSourceType) {
            contextHolder.set(dataSourceType);
        }
        
        public static String getDataSourceType() {
            return contextHolder.get();
        }
        
        public static void clearDataSourceType() {
            contextHolder.remove();
        }
    }
    
    /**
     * 读写分离服务
     */
    @Service
    public class ReadWriteSplitService {
        
        @Autowired
        private UserRepository userRepository;
        
        /**
         * 写操作 - 路由到主库
         */
        @Transactional
        public User saveUser(User user) {
            DataSourceContextHolder.setDataSourceType("master");
            try {
                user.setCreatedTime(LocalDateTime.now());
                return userRepository.save(user);
            } finally {
                DataSourceContextHolder.clearDataSourceType();
            }
        }
        
        /**
         * 读操作 - 路由到从库
         */
        @Transactional(readOnly = true)
        public List<User> findUsersByDepartment(Long departmentId) {
            // 负载均衡选择从库
            String slaveDataSource = selectSlaveDataSource();
            DataSourceContextHolder.setDataSourceType(slaveDataSource);
            
            try {
                return userRepository.findByDepartmentIdAndStatus(departmentId, 1);
            } finally {
                DataSourceContextHolder.clearDataSourceType();
            }
        }
        
        /**
         * 从库负载均衡选择
         */
        private String selectSlaveDataSource() {
            // 简单的轮询策略
            long currentTime = System.currentTimeMillis();
            int slaveIndex = (int) (currentTime % 2) + 1;
            return "slave" + slaveIndex;
        }
        
        /**
         * 强制读主库（解决主从延迟问题）
         */
        @Transactional(readOnly = true)
        public User findUserByIdFromMaster(Long userId) {
            DataSourceContextHolder.setDataSourceType("master");
            try {
                return userRepository.findById(userId).orElse(null);
            } finally {
                DataSourceContextHolder.clearDataSourceType();
            }
        }
    }
    
    // ==================== 4. 缓存策略实现 ====================
    
    /**
     * 多级缓存服务
     */
    @Service
    public class MultiLevelCacheService {
        
        // L1缓存：本地缓存
        private final Map<String, CacheEntry> localCache = new ConcurrentHashMap<>();
        
        @Autowired
        private org.springframework.data.redis.core.RedisTemplate<String, Object> redisTemplate;
        
        @Autowired
        private UserRepository userRepository;
        
        /**
         * 多级缓存查询
         */
        public User getUserWithCache(Long userId) {
            String cacheKey = "user:" + userId;
            
            // L1缓存查询
            CacheEntry localEntry = localCache.get(cacheKey);
            if (localEntry != null && !localEntry.isExpired()) {
                return (User) localEntry.getValue();
            }
            
            // L2缓存查询（Redis）
            User user = (User) redisTemplate.opsForValue().get(cacheKey);
            if (user != null) {
                // 回填L1缓存
                localCache.put(cacheKey, new CacheEntry(user, System.currentTimeMillis() + 300000)); // 5分钟
                return user;
            }
            
            // 数据库查询
            user = userRepository.findById(userId).orElse(null);
            if (user != null) {
                // 回填缓存
                redisTemplate.opsForValue().set(cacheKey, user, java.time.Duration.ofMinutes(30));
                localCache.put(cacheKey, new CacheEntry(user, System.currentTimeMillis() + 300000));
            }
            
            return user;
        }
        
        /**
         * 缓存更新策略
         */
        @CacheEvict(value = "users", key = "#user.id")
        public User updateUserWithCache(User user) {
            // 先更新数据库
            User updatedUser = userRepository.save(user);
            
            // 删除缓存（Cache-Aside模式）
            String cacheKey = "user:" + user.getId();
            localCache.remove(cacheKey);
            redisTemplate.delete(cacheKey);
            
            return updatedUser;
        }
        
        /**
         * 预热缓存
         */
        public void warmUpCache() {
            // 预热热点数据
            List<Long> hotUserIds = getHotUserIds();
            
            CompletableFuture.runAsync(() -> {
                for (Long userId : hotUserIds) {
                    try {
                        getUserWithCache(userId);
                        Thread.sleep(10); // 避免数据库压力过大
                    } catch (Exception e) {
                        System.err.println("Cache warm up failed for user: " + userId);
                    }
                }
            });
        }
        
        private List<Long> getHotUserIds() {
            // 从统计数据或日志中获取热点用户ID
            return Arrays.asList(1L, 2L, 3L, 4L, 5L);
        }
    }
    
    // ==================== 5. 辅助类定义 ====================
    
    /**
     * 查询执行计划
     */
    public static class QueryExecutionPlan {
        private List<ExecutionStep> steps = new ArrayList<>();
        private List<String> warnings = new ArrayList<>();
        
        public void addStep(ExecutionStep step) {
            steps.add(step);
        }
        
        public void addWarning(String warning) {
            warnings.add(warning);
        }
        
        // getters
        public List<ExecutionStep> getSteps() { return steps; }
        public List<String> getWarnings() { return warnings; }
    }
    
    /**
     * 执行步骤
     */
    public static class ExecutionStep {
        private String selectType;
        private String table;
        private String type;
        private String possibleKeys;
        private String key;
        private Long rows;
        private String extra;
        
        public ExecutionStep(String selectType, String table, String type, 
                           String possibleKeys, String key, Long rows, String extra) {
            this.selectType = selectType;
            this.table = table;
            this.type = type;
            this.possibleKeys = possibleKeys;
            this.key = key;
            this.rows = rows;
            this.extra = extra;
        }
        
        // getters
        public String getSelectType() { return selectType; }
        public String getTable() { return table; }
        public String getType() { return type; }
        public String getPossibleKeys() { return possibleKeys; }
        public String getKey() { return key; }
        public Long getRows() { return rows; }
        public String getExtra() { return extra; }
    }
    
    /**
     * 索引推荐
     */
    public static class IndexRecommendation {
        private String tableName;
        private String indexSql;
        private Double avgExecutionTime;
        private String sampleQuery;
        
        public IndexRecommendation(String tableName, String indexSql, 
                                 Double avgExecutionTime, String sampleQuery) {
            this.tableName = tableName;
            this.indexSql = indexSql;
            this.avgExecutionTime = avgExecutionTime;
            this.sampleQuery = sampleQuery;
        }
        
        // getters
        public String getTableName() { return tableName; }
        public String getIndexSql() { return indexSql; }
        public Double getAvgExecutionTime() { return avgExecutionTime; }
        public String getSampleQuery() { return sampleQuery; }
    }
    
    /**
     * 用户部门DTO
     */
    public static class UserDepartmentDto {
        private Long id;
        private String name;
        private String email;
        private Double salary;
        private String deptName;
        
        // getters and setters
        public Long getId() { return id; }
        public void setId(Long id) { this.id = id; }
        public String getName() { return name; }
        public void setName(String name) { this.name = name; }
        public String getEmail() { return email; }
        public void setEmail(String email) { this.email = email; }
        public Double getSalary() { return salary; }
        public void setSalary(Double salary) { this.salary = salary; }
        public String getDeptName() { return deptName; }
        public void setDeptName(String deptName) { this.deptName = deptName; }
    }
    
    /**
     * 用户薪资更新
     */
    public static class UserSalaryUpdate {
        private Long userId;
        private Double salary;
        
        public UserSalaryUpdate(Long userId, Double salary) {
            this.userId = userId;
            this.salary = salary;
        }
        
        // getters
        public Long getUserId() { return userId; }
        public Double getSalary() { return salary; }
    }
    
    /**
     * 分页结果
     */
    public static class PageResult<T> {
        private List<T> data;
        private Long total;
        private Integer page;
        private Integer size;
        
        public PageResult(List<T> data, Long total, Integer page, Integer size) {
            this.data = data;
            this.total = total;
            this.page = page;
            this.size = size;
        }
        
        // getters
        public List<T> getData() { return data; }
        public Long getTotal() { return total; }
        public Integer getPage() { return page; }
        public Integer getSize() { return size; }
    }
    
    /**
     * 缓存条目
     */
    public static class CacheEntry {
        private Object value;
        private long expireTime;
        
        public CacheEntry(Object value, long expireTime) {
            this.value = value;
            this.expireTime = expireTime;
        }
        
        public boolean isExpired() {
            return System.currentTimeMillis() > expireTime;
        }
        
        public Object getValue() { return value; }
    }
    
    /**
     * 用户Repository
     */
    @Repository
    public interface UserRepository extends JpaRepository<User, Long> {
        
        @Query("SELECT u FROM User u WHERE u.departmentId = ?1 AND u.status = ?2")
        List<User> findByDepartmentIdAndStatus(Long departmentId, Integer status);
        
        @Query(value = "SELECT * FROM users WHERE salary > ?1 ORDER BY salary DESC LIMIT ?2", 
               nativeQuery = true)
        List<User> findTopUsersBySalary(Double minSalary, Integer limit);
        
        @Modifying
        @Query("UPDATE User u SET u.salary = u.salary * 1.1 WHERE u.departmentId = ?1")
        int increaseSalaryByDepartment(Long departmentId);
    }
}
```

**关键技术点：**

1. **索引优化策略**：
   - 单列索引：高选择性字段
   - 复合索引：多条件查询
   - 前缀索引：长字符串字段
   - 覆盖索引：避免回表查询

2. **SQL优化技巧**：
   - 避免SELECT *，只查询需要的字段
   - 使用INNER JOIN替代子查询
   - 批量操作替代逐条操作
   - 游标分页替代OFFSET深分页

3. **读写分离架构**：
   - 主库处理写操作
   - 从库处理读操作
   - 动态数据源路由
   - 主从延迟处理

4. **多级缓存策略**：
   - L1缓存：本地内存缓存
   - L2缓存：分布式缓存（Redis）
   - 缓存更新策略：Cache-Aside
   - 缓存预热和失效处理

5. **分库分表策略**：
   - 垂直拆分：按业务模块
   - 水平拆分：按数据量
   - 分片键选择：均匀分布
   - 跨分片查询处理

**性能优化建议：**

- **监控指标**：慢查询日志、执行计划分析、缓存命中率
- **容量规划**：数据增长预估、硬件资源配置
- **定期维护**：索引重建、统计信息更新、数据归档
- **压力测试**：模拟高并发场景、性能瓶颈识别

### 4.3 技术选型类问题

**6. 如何进行技术选型？需要考虑哪些因素？**

技术选型是架构设计中的关键决策，需要综合考虑多个维度的因素，确保选择最适合项目的技术方案。

**技术选型评估框架：**

| 评估维度 | 权重 | 评估标准 | 评分方法 | 风险等级 |
|----------|------|----------|----------|---------|
| 业务匹配度 | 30% | 功能覆盖、性能满足、扩展性 | 1-10分 | 高 |
| 技术成熟度 | 25% | 版本稳定、社区活跃、文档质量 | 1-10分 | 中 |
| 团队适配度 | 20% | 学习成本、技能匹配、维护难度 | 1-10分 | 中 |
| 成本效益 | 15% | 开发成本、运维成本、许可费用 | 1-10分 | 低 |
| 生态兼容性 | 10% | 技术栈兼容、工具链支持 | 1-10分 | 低 |

**详细实现方案：**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.stereotype.Service;
import org.springframework.stereotype.Component;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.bind.annotation.*;

import java.util.*;
import java.time.LocalDateTime;
import java.util.stream.Collectors;
import java.math.BigDecimal;
import java.math.RoundingMode;

/**
 * 技术选型决策支持系统
 */
@SpringBootApplication
public class TechSelectionApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(TechSelectionApplication.class, args);
    }
    
    // ==================== 1. 技术选型评估框架 ====================
    
    /**
     * 技术选型评估服务
     */
    @Service
    public class TechSelectionService {
        
        @Autowired
        private List<EvaluationCriteria> evaluationCriteriaList;
        
        /**
         * 综合评估技术方案
         */
        public TechEvaluationResult evaluateTechnology(TechOption techOption, ProjectContext context) {
            List<CriteriaScore> scores = new ArrayList<>();
            BigDecimal totalScore = BigDecimal.ZERO;
            BigDecimal totalWeight = BigDecimal.ZERO;
            
            for (EvaluationCriteria criteria : evaluationCriteriaList) {
                CriteriaScore score = criteria.evaluate(techOption, context);
                scores.add(score);
                
                BigDecimal weightedScore = score.getScore().multiply(criteria.getWeight());
                totalScore = totalScore.add(weightedScore);
                totalWeight = totalWeight.add(criteria.getWeight());
            }
            
            BigDecimal finalScore = totalScore.divide(totalWeight, 2, RoundingMode.HALF_UP);
            
            return new TechEvaluationResult(
                techOption.getName(),
                finalScore,
                scores,
                generateRecommendation(finalScore, scores),
                assessRisk(scores)
            );
        }
        
        /**
         * 比较多个技术方案
         */
        public TechComparisonResult compareTechnologies(List<TechOption> options, ProjectContext context) {
            List<TechEvaluationResult> results = options.stream()
                .map(option -> evaluateTechnology(option, context))
                .sorted((r1, r2) -> r2.getFinalScore().compareTo(r1.getFinalScore()))
                .collect(Collectors.toList());
            
            return new TechComparisonResult(
                results,
                generateComparisonMatrix(results),
                recommendBestOption(results),
                identifyKeyDifferences(results)
            );
        }
        
        /**
         * 生成选型建议
         */
        private String generateRecommendation(BigDecimal score, List<CriteriaScore> scores) {
            if (score.compareTo(new BigDecimal("8.0")) >= 0) {
                return "强烈推荐：该技术在各方面表现优秀，非常适合当前项目";
            } else if (score.compareTo(new BigDecimal("6.0")) >= 0) {
                return "推荐：该技术整体表现良好，建议进一步评估具体实施方案";
            } else if (score.compareTo(new BigDecimal("4.0")) >= 0) {
                return "谨慎考虑：该技术存在一些问题，需要制定风险缓解措施";
            } else {
                return "不推荐：该技术不适合当前项目，建议寻找替代方案";
            }
        }
        
        /**
         * 风险评估
         */
        private RiskAssessment assessRisk(List<CriteriaScore> scores) {
            List<String> highRisks = new ArrayList<>();
            List<String> mediumRisks = new ArrayList<>();
            List<String> lowRisks = new ArrayList<>();
            
            for (CriteriaScore score : scores) {
                if (score.getScore().compareTo(new BigDecimal("4.0")) < 0) {
                    highRisks.add(score.getCriteriaName() + ": " + score.getComment());
                } else if (score.getScore().compareTo(new BigDecimal("6.0")) < 0) {
                    mediumRisks.add(score.getCriteriaName() + ": " + score.getComment());
                } else {
                    lowRisks.add(score.getCriteriaName() + ": " + score.getComment());
                }
            }
            
            return new RiskAssessment(highRisks, mediumRisks, lowRisks);
        }
        
        private ComparisonMatrix generateComparisonMatrix(List<TechEvaluationResult> results) {
            // 生成技术对比矩阵
            return new ComparisonMatrix(results);
        }
        
        private String recommendBestOption(List<TechEvaluationResult> results) {
            if (results.isEmpty()) return "无可用选项";
            
            TechEvaluationResult best = results.get(0);
            return String.format("推荐选择 %s，综合评分: %.2f", 
                best.getTechName(), best.getFinalScore());
        }
        
        private List<String> identifyKeyDifferences(List<TechEvaluationResult> results) {
            List<String> differences = new ArrayList<>();
            
            if (results.size() >= 2) {
                TechEvaluationResult first = results.get(0);
                TechEvaluationResult second = results.get(1);
                
                BigDecimal scoreDiff = first.getFinalScore().subtract(second.getFinalScore());
                differences.add(String.format("%s 比 %s 综合评分高 %.2f 分", 
                    first.getTechName(), second.getTechName(), scoreDiff));
                
                // 分析具体差异
                for (int i = 0; i < first.getScores().size(); i++) {
                    CriteriaScore firstScore = first.getScores().get(i);
                    CriteriaScore secondScore = second.getScores().get(i);
                    
                    BigDecimal diff = firstScore.getScore().subtract(secondScore.getScore());
                    if (diff.abs().compareTo(new BigDecimal("2.0")) >= 0) {
                        differences.add(String.format("%s 在 %s 方面差异显著 (%.1f分)", 
                            diff.compareTo(BigDecimal.ZERO) > 0 ? first.getTechName() : second.getTechName(),
                            firstScore.getCriteriaName(), diff.abs()));
                    }
                }
            }
            
            return differences;
        }
    }
    
    // ==================== 2. 具体评估标准实现 ====================
    
    /**
     * 业务匹配度评估
     */
    @Component
    public class BusinessFitCriteria implements EvaluationCriteria {
        
        @Override
        public String getName() {
            return "业务匹配度";
        }
        
        @Override
        public BigDecimal getWeight() {
            return new BigDecimal("0.30");
        }
        
        @Override
        public CriteriaScore evaluate(TechOption techOption, ProjectContext context) {
            BigDecimal score = BigDecimal.ZERO;
            List<String> comments = new ArrayList<>();
            
            // 功能覆盖度评估
            int functionalCoverage = evaluateFunctionalCoverage(techOption, context.getRequiredFeatures());
            score = score.add(new BigDecimal(functionalCoverage * 0.4));
            comments.add("功能覆盖度: " + functionalCoverage + "/10");
            
            // 性能要求匹配度
            int performanceMatch = evaluatePerformanceMatch(techOption, context.getPerformanceRequirements());
            score = score.add(new BigDecimal(performanceMatch * 0.4));
            comments.add("性能匹配度: " + performanceMatch + "/10");
            
            // 扩展性评估
            int scalability = evaluateScalability(techOption, context.getScalabilityNeeds());
            score = score.add(new BigDecimal(scalability * 0.2));
            comments.add("扩展性: " + scalability + "/10");
            
            return new CriteriaScore(getName(), score, String.join(", ", comments));
        }
        
        private int evaluateFunctionalCoverage(TechOption techOption, List<String> requiredFeatures) {
            int coveredFeatures = 0;
            for (String feature : requiredFeatures) {
                if (techOption.getSupportedFeatures().contains(feature)) {
                    coveredFeatures++;
                }
            }
            return Math.min(10, (coveredFeatures * 10) / requiredFeatures.size());
        }
        
        private int evaluatePerformanceMatch(TechOption techOption, PerformanceRequirements requirements) {
            int score = 10;
            
            if (techOption.getMaxThroughput() < requirements.getMinThroughput()) {
                score -= 3;
            }
            
            if (techOption.getAverageLatency() > requirements.getMaxLatency()) {
                score -= 3;
            }
            
            if (techOption.getMemoryUsage() > requirements.getMaxMemoryUsage()) {
                score -= 2;
            }
            
            return Math.max(0, score);
        }
        
        private int evaluateScalability(TechOption techOption, ScalabilityNeeds needs) {
            int score = 5; // 基础分
            
            if (techOption.supportsHorizontalScaling() && needs.requiresHorizontalScaling()) {
                score += 3;
            }
            
            if (techOption.supportsVerticalScaling() && needs.requiresVerticalScaling()) {
                score += 2;
            }
            
            return Math.min(10, score);
        }
    }
    
    /**
     * 技术成熟度评估
     */
    @Component
    public class TechMaturityCriteria implements EvaluationCriteria {
        
        @Override
        public String getName() {
            return "技术成熟度";
        }
        
        @Override
        public BigDecimal getWeight() {
            return new BigDecimal("0.25");
        }
        
        @Override
        public CriteriaScore evaluate(TechOption techOption, ProjectContext context) {
            BigDecimal score = BigDecimal.ZERO;
            List<String> comments = new ArrayList<>();
            
            // 版本稳定性
            int versionStability = evaluateVersionStability(techOption);
            score = score.add(new BigDecimal(versionStability * 0.4));
            comments.add("版本稳定性: " + versionStability + "/10");
            
            // 社区活跃度
            int communityActivity = evaluateCommunityActivity(techOption);
            score = score.add(new BigDecimal(communityActivity * 0.3));
            comments.add("社区活跃度: " + communityActivity + "/10");
            
            // 文档质量
            int documentationQuality = evaluateDocumentationQuality(techOption);
            score = score.add(new BigDecimal(documentationQuality * 0.3));
            comments.add("文档质量: " + documentationQuality + "/10");
            
            return new CriteriaScore(getName(), score, String.join(", ", comments));
        }
        
        private int evaluateVersionStability(TechOption techOption) {
            // 基于版本发布历史、bug修复频率等评估
            if (techOption.getVersionHistory().size() < 5) return 3;
            if (techOption.getMajorVersionChanges() > 3) return 5;
            if (techOption.getCriticalBugCount() > 10) return 6;
            return 9;
        }
        
        private int evaluateCommunityActivity(TechOption techOption) {
            // 基于GitHub stars、contributors、issues等评估
            int score = 5;
            if (techOption.getGithubStars() > 10000) score += 2;
            if (techOption.getActiveContributors() > 100) score += 2;
            if (techOption.getRecentCommits() > 50) score += 1;
            return Math.min(10, score);
        }
        
        private int evaluateDocumentationQuality(TechOption techOption) {
            // 基于文档完整性、示例质量、更新频率等评估
            int score = 5;
            if (techOption.hasComprehensiveDocumentation()) score += 2;
            if (techOption.hasGoodExamples()) score += 2;
            if (techOption.isDocumentationUpToDate()) score += 1;
            return Math.min(10, score);
        }
    }
    
    /**
     * 团队适配度评估
     */
    @Component
    public class TeamFitCriteria implements EvaluationCriteria {
        
        @Override
        public String getName() {
            return "团队适配度";
        }
        
        @Override
        public BigDecimal getWeight() {
            return new BigDecimal("0.20");
        }
        
        @Override
        public CriteriaScore evaluate(TechOption techOption, ProjectContext context) {
            BigDecimal score = BigDecimal.ZERO;
            List<String> comments = new ArrayList<>();
            
            // 学习成本
            int learningCost = evaluateLearningCost(techOption, context.getTeamSkills());
            score = score.add(new BigDecimal(learningCost * 0.4));
            comments.add("学习成本: " + (10 - learningCost) + "/10 (越低越好)");
            
            // 技能匹配度
            int skillMatch = evaluateSkillMatch(techOption, context.getTeamSkills());
            score = score.add(new BigDecimal(skillMatch * 0.4));
            comments.add("技能匹配: " + skillMatch + "/10");
            
            // 维护难度
            int maintenanceDifficulty = evaluateMaintenanceDifficulty(techOption);
            score = score.add(new BigDecimal(maintenanceDifficulty * 0.2));
            comments.add("维护难度: " + (10 - maintenanceDifficulty) + "/10 (越低越好)");
            
            return new CriteriaScore(getName(), score, String.join(", ", comments));
        }
        
        private int evaluateLearningCost(TechOption techOption, TeamSkills teamSkills) {
            // 评估团队学习该技术的成本（分数越高表示成本越低）
            if (teamSkills.hasExperienceWith(techOption.getTechnologyStack())) {
                return 9;
            }
            if (teamSkills.hasSimilarExperience(techOption.getTechnologyStack())) {
                return 6;
            }
            return 3;
        }
        
        private int evaluateSkillMatch(TechOption techOption, TeamSkills teamSkills) {
            // 评估团队技能与技术要求的匹配度
            int matchCount = 0;
            List<String> requiredSkills = techOption.getRequiredSkills();
            
            for (String skill : requiredSkills) {
                if (teamSkills.hasSkill(skill)) {
                    matchCount++;
                }
            }
            
            return (matchCount * 10) / requiredSkills.size();
        }
        
        private int evaluateMaintenanceDifficulty(TechOption techOption) {
            // 评估维护难度（分数越高表示越容易维护）
            int score = 5;
            if (techOption.hasGoodToolingSupport()) score += 2;
            if (techOption.hasActiveSupport()) score += 2;
            if (techOption.isWellDocumented()) score += 1;
            return Math.min(10, score);
        }
    }
    
    /**
     * 成本效益评估
     */
    @Component
    public class CostBenefitCriteria implements EvaluationCriteria {
        
        @Override
        public String getName() {
            return "成本效益";
        }
        
        @Override
        public BigDecimal getWeight() {
            return new BigDecimal("0.15");
        }
        
        @Override
        public CriteriaScore evaluate(TechOption techOption, ProjectContext context) {
            BigDecimal score = BigDecimal.ZERO;
            List<String> comments = new ArrayList<>();
            
            // 开发成本
            int developmentCost = evaluateDevelopmentCost(techOption, context);
            score = score.add(new BigDecimal(developmentCost * 0.4));
            comments.add("开发成本: " + (10 - developmentCost) + "/10");
            
            // 运维成本
            int operationalCost = evaluateOperationalCost(techOption);
            score = score.add(new BigDecimal(operationalCost * 0.4));
            comments.add("运维成本: " + (10 - operationalCost) + "/10");
            
            // 许可证成本
            int licenseCost = evaluateLicenseCost(techOption);
            score = score.add(new BigDecimal(licenseCost * 0.2));
            comments.add("许可证成本: " + (10 - licenseCost) + "/10");
            
            return new CriteriaScore(getName(), score, String.join(", ", comments));
        }
        
        private int evaluateDevelopmentCost(TechOption techOption, ProjectContext context) {
            // 评估开发成本（分数越高表示成本越低）
            int baseCost = 5;
            
            if (techOption.hasRichEcosystem()) baseCost += 2;
            if (techOption.hasGoodDeveloperTools()) baseCost += 2;
            if (context.getTeamSkills().hasExperienceWith(techOption.getTechnologyStack())) {
                baseCost += 1;
            }
            
            return Math.min(10, baseCost);
        }
        
        private int evaluateOperationalCost(TechOption techOption) {
            // 评估运维成本
            int score = 5;
            
            if (techOption.hasLowResourceRequirements()) score += 2;
            if (techOption.hasGoodMonitoringSupport()) score += 2;
            if (techOption.isEasyToScale()) score += 1;
            
            return Math.min(10, score);
        }
        
        private int evaluateLicenseCost(TechOption techOption) {
            // 评估许可证成本
            if (techOption.isOpenSource()) return 10;
            if (techOption.hasFreeTier()) return 7;
            if (techOption.hasReasonablePricing()) return 5;
            return 2;
        }
    }
    
    /**
     * 生态兼容性评估
     */
    @Component
    public class EcosystemCompatibilityCriteria implements EvaluationCriteria {
        
        @Override
        public String getName() {
            return "生态兼容性";
        }
        
        @Override
        public BigDecimal getWeight() {
            return new BigDecimal("0.10");
        }
        
        @Override
        public CriteriaScore evaluate(TechOption techOption, ProjectContext context) {
            BigDecimal score = BigDecimal.ZERO;
            List<String> comments = new ArrayList<>();
            
            // 技术栈兼容性
            int stackCompatibility = evaluateStackCompatibility(techOption, context.getCurrentTechStack());
            score = score.add(new BigDecimal(stackCompatibility * 0.6));
            comments.add("技术栈兼容: " + stackCompatibility + "/10");
            
            // 工具链支持
            int toolchainSupport = evaluateToolchainSupport(techOption, context.getCurrentTools());
            score = score.add(new BigDecimal(toolchainSupport * 0.4));
            comments.add("工具链支持: " + toolchainSupport + "/10");
            
            return new CriteriaScore(getName(), score, String.join(", ", comments));
        }
        
        private int evaluateStackCompatibility(TechOption techOption, List<String> currentStack) {
            int compatibilityScore = 5;
            
            for (String tech : currentStack) {
                if (techOption.isCompatibleWith(tech)) {
                    compatibilityScore += 1;
                } else if (techOption.conflictsWith(tech)) {
                    compatibilityScore -= 2;
                }
            }
            
            return Math.max(0, Math.min(10, compatibilityScore));
        }
        
        private int evaluateToolchainSupport(TechOption techOption, List<String> currentTools) {
            int supportScore = 5;
            
            for (String tool : currentTools) {
                if (techOption.supportsIntegrationWith(tool)) {
                    supportScore += 1;
                }
            }
            
            return Math.min(10, supportScore);
        }
    }
    
    // ==================== 3. 数据模型定义 ====================
    
    /**
     * 技术选项
     */
    public static class TechOption {
        private String name;
        private String version;
        private List<String> supportedFeatures;
        private List<String> requiredSkills;
        private String technologyStack;
        private PerformanceMetrics performanceMetrics;
        private CommunityMetrics communityMetrics;
        private CostMetrics costMetrics;
        private List<String> compatibleTechnologies;
        private List<String> conflictingTechnologies;
        
        // Constructor
        public TechOption(String name, String version) {
            this.name = name;
            this.version = version;
            this.supportedFeatures = new ArrayList<>();
            this.requiredSkills = new ArrayList<>();
            this.compatibleTechnologies = new ArrayList<>();
            this.conflictingTechnologies = new ArrayList<>();
        }
        
        // Getters and utility methods
        public String getName() { return name; }
        public String getVersion() { return version; }
        public List<String> getSupportedFeatures() { return supportedFeatures; }
        public List<String> getRequiredSkills() { return requiredSkills; }
        public String getTechnologyStack() { return technologyStack; }
        
        public int getMaxThroughput() {
            return performanceMetrics != null ? performanceMetrics.getMaxThroughput() : 0;
        }
        
        public int getAverageLatency() {
            return performanceMetrics != null ? performanceMetrics.getAverageLatency() : Integer.MAX_VALUE;
        }
        
        public int getMemoryUsage() {
            return performanceMetrics != null ? performanceMetrics.getMemoryUsage() : Integer.MAX_VALUE;
        }
        
        public boolean supportsHorizontalScaling() {
            return supportedFeatures.contains("horizontal_scaling");
        }
        
        public boolean supportsVerticalScaling() {
            return supportedFeatures.contains("vertical_scaling");
        }
        
        public List<String> getVersionHistory() {
            return communityMetrics != null ? communityMetrics.getVersionHistory() : new ArrayList<>();
        }
        
        public int getMajorVersionChanges() {
            return communityMetrics != null ? communityMetrics.getMajorVersionChanges() : 0;
        }
        
        public int getCriticalBugCount() {
            return communityMetrics != null ? communityMetrics.getCriticalBugCount() : 0;
        }
        
        public int getGithubStars() {
            return communityMetrics != null ? communityMetrics.getGithubStars() : 0;
        }
        
        public int getActiveContributors() {
            return communityMetrics != null ? communityMetrics.getActiveContributors() : 0;
        }
        
        public int getRecentCommits() {
            return communityMetrics != null ? communityMetrics.getRecentCommits() : 0;
        }
        
        public boolean hasComprehensiveDocumentation() {
            return communityMetrics != null && communityMetrics.hasComprehensiveDocumentation();
        }
        
        public boolean hasGoodExamples() {
            return communityMetrics != null && communityMetrics.hasGoodExamples();
        }
        
        public boolean isDocumentationUpToDate() {
            return communityMetrics != null && communityMetrics.isDocumentationUpToDate();
        }
        
        public boolean hasGoodToolingSupport() {
            return supportedFeatures.contains("good_tooling");
        }
        
        public boolean hasActiveSupport() {
            return communityMetrics != null && communityMetrics.hasActiveSupport();
        }
        
        public boolean isWellDocumented() {
            return hasComprehensiveDocumentation();
        }
        
        public boolean hasRichEcosystem() {
            return supportedFeatures.contains("rich_ecosystem");
        }
        
        public boolean hasGoodDeveloperTools() {
            return supportedFeatures.contains("good_dev_tools");
        }
        
        public boolean hasLowResourceRequirements() {
            return performanceMetrics != null && performanceMetrics.hasLowResourceRequirements();
        }
        
        public boolean hasGoodMonitoringSupport() {
            return supportedFeatures.contains("monitoring_support");
        }
        
        public boolean isEasyToScale() {
            return supportsHorizontalScaling() || supportsVerticalScaling();
        }
        
        public boolean isOpenSource() {
            return costMetrics != null && costMetrics.isOpenSource();
        }
        
        public boolean hasFreeTier() {
            return costMetrics != null && costMetrics.hasFreeTier();
        }
        
        public boolean hasReasonablePricing() {
            return costMetrics != null && costMetrics.hasReasonablePricing();
        }
        
        public boolean isCompatibleWith(String technology) {
            return compatibleTechnologies.contains(technology);
        }
        
        public boolean conflictsWith(String technology) {
            return conflictingTechnologies.contains(technology);
        }
        
        public boolean supportsIntegrationWith(String tool) {
            return supportedFeatures.contains("integration_" + tool.toLowerCase());
        }
        
        // Setters
        public void setTechnologyStack(String technologyStack) { this.technologyStack = technologyStack; }
        public void setPerformanceMetrics(PerformanceMetrics performanceMetrics) { this.performanceMetrics = performanceMetrics; }
        public void setCommunityMetrics(CommunityMetrics communityMetrics) { this.communityMetrics = communityMetrics; }
        public void setCostMetrics(CostMetrics costMetrics) { this.costMetrics = costMetrics; }
    }
    
    // ==================== 4. 辅助类定义 ====================
    
    public interface EvaluationCriteria {
        String getName();
        BigDecimal getWeight();
        CriteriaScore evaluate(TechOption techOption, ProjectContext context);
    }
    
    public static class CriteriaScore {
        private String criteriaName;
        private BigDecimal score;
        private String comment;
        
        public CriteriaScore(String criteriaName, BigDecimal score, String comment) {
            this.criteriaName = criteriaName;
            this.score = score;
            this.comment = comment;
        }
        
        // Getters
        public String getCriteriaName() { return criteriaName; }
        public BigDecimal getScore() { return score; }
        public String getComment() { return comment; }
    }
    
    public static class TechEvaluationResult {
        private String techName;
        private BigDecimal finalScore;
        private List<CriteriaScore> scores;
        private String recommendation;
        private RiskAssessment riskAssessment;
        
        public TechEvaluationResult(String techName, BigDecimal finalScore, 
                                  List<CriteriaScore> scores, String recommendation, 
                                  RiskAssessment riskAssessment) {
            this.techName = techName;
            this.finalScore = finalScore;
            this.scores = scores;
            this.recommendation = recommendation;
            this.riskAssessment = riskAssessment;
        }
        
        // Getters
        public String getTechName() { return techName; }
        public BigDecimal getFinalScore() { return finalScore; }
        public List<CriteriaScore> getScores() { return scores; }
        public String getRecommendation() { return recommendation; }
        public RiskAssessment getRiskAssessment() { return riskAssessment; }
    }
    
    public static class TechComparisonResult {
        private List<TechEvaluationResult> results;
        private ComparisonMatrix comparisonMatrix;
        private String bestOption;
        private List<String> keyDifferences;
        
        public TechComparisonResult(List<TechEvaluationResult> results, 
                                  ComparisonMatrix comparisonMatrix, 
                                  String bestOption, List<String> keyDifferences) {
            this.results = results;
            this.comparisonMatrix = comparisonMatrix;
            this.bestOption = bestOption;
            this.keyDifferences = keyDifferences;
        }
        
        // Getters
        public List<TechEvaluationResult> getResults() { return results; }
        public ComparisonMatrix getComparisonMatrix() { return comparisonMatrix; }
        public String getBestOption() { return bestOption; }
        public List<String> getKeyDifferences() { return keyDifferences; }
    }
    
    public static class ProjectContext {
        private List<String> requiredFeatures;
        private PerformanceRequirements performanceRequirements;
        private ScalabilityNeeds scalabilityNeeds;
        private TeamSkills teamSkills;
        private List<String> currentTechStack;
        private List<String> currentTools;
        
        // Constructor and getters
        public ProjectContext() {
            this.requiredFeatures = new ArrayList<>();
            this.currentTechStack = new ArrayList<>();
            this.currentTools = new ArrayList<>();
        }
        
        public List<String> getRequiredFeatures() { return requiredFeatures; }
        public PerformanceRequirements getPerformanceRequirements() { return performanceRequirements; }
        public ScalabilityNeeds getScalabilityNeeds() { return scalabilityNeeds; }
        public TeamSkills getTeamSkills() { return teamSkills; }
        public List<String> getCurrentTechStack() { return currentTechStack; }
        public List<String> getCurrentTools() { return currentTools; }
        
        // Setters
        public void setPerformanceRequirements(PerformanceRequirements performanceRequirements) {
            this.performanceRequirements = performanceRequirements;
        }
        public void setScalabilityNeeds(ScalabilityNeeds scalabilityNeeds) {
            this.scalabilityNeeds = scalabilityNeeds;
        }
        public void setTeamSkills(TeamSkills teamSkills) {
            this.teamSkills = teamSkills;
        }
    }
    
    // 其他辅助类的简化定义
    public static class PerformanceRequirements {
        private int minThroughput;
        private int maxLatency;
        private int maxMemoryUsage;
        
        public int getMinThroughput() { return minThroughput; }
        public int getMaxLatency() { return maxLatency; }
        public int getMaxMemoryUsage() { return maxMemoryUsage; }
    }
    
    public static class ScalabilityNeeds {
        private boolean requiresHorizontalScaling;
        private boolean requiresVerticalScaling;
        
        public boolean requiresHorizontalScaling() { return requiresHorizontalScaling; }
        public boolean requiresVerticalScaling() { return requiresVerticalScaling; }
    }
    
    public static class TeamSkills {
        private List<String> skills;
        private Map<String, Integer> experienceLevel;
        
        public TeamSkills() {
            this.skills = new ArrayList<>();
            this.experienceLevel = new HashMap<>();
        }
        
        public boolean hasSkill(String skill) {
            return skills.contains(skill);
        }
        
        public boolean hasExperienceWith(String technology) {
            return experienceLevel.getOrDefault(technology, 0) >= 3;
        }
        
        public boolean hasSimilarExperience(String technology) {
            // 简化实现：检查相关技术经验
            return experienceLevel.values().stream().anyMatch(level -> level >= 2);
        }
    }
    
    public static class PerformanceMetrics {
        private int maxThroughput;
        private int averageLatency;
        private int memoryUsage;
        private boolean lowResourceRequirements;
        
        public int getMaxThroughput() { return maxThroughput; }
        public int getAverageLatency() { return averageLatency; }
        public int getMemoryUsage() { return memoryUsage; }
        public boolean hasLowResourceRequirements() { return lowResourceRequirements; }
    }
    
    public static class CommunityMetrics {
        private List<String> versionHistory;
        private int majorVersionChanges;
        private int criticalBugCount;
        private int githubStars;
        private int activeContributors;
        private int recentCommits;
        private boolean comprehensiveDocumentation;
        private boolean goodExamples;
        private boolean documentationUpToDate;
        private boolean activeSupport;
        
        public List<String> getVersionHistory() { return versionHistory; }
        public int getMajorVersionChanges() { return majorVersionChanges; }
        public int getCriticalBugCount() { return criticalBugCount; }
        public int getGithubStars() { return githubStars; }
        public int getActiveContributors() { return activeContributors; }
        public int getRecentCommits() { return recentCommits; }
        public boolean hasComprehensiveDocumentation() { return comprehensiveDocumentation; }
        public boolean hasGoodExamples() { return goodExamples; }
        public boolean isDocumentationUpToDate() { return documentationUpToDate; }
        public boolean hasActiveSupport() { return activeSupport; }
    }
    
    public static class CostMetrics {
        private boolean openSource;
        private boolean freeTier;
        private boolean reasonablePricing;
        
        public boolean isOpenSource() { return openSource; }
        public boolean hasFreeTier() { return freeTier; }
        public boolean hasReasonablePricing() { return reasonablePricing; }
    }
    
    public static class RiskAssessment {
        private List<String> highRisks;
        private List<String> mediumRisks;
        private List<String> lowRisks;
        
        public RiskAssessment(List<String> highRisks, List<String> mediumRisks, List<String> lowRisks) {
            this.highRisks = highRisks;
            this.mediumRisks = mediumRisks;
            this.lowRisks = lowRisks;
        }
        
        public List<String> getHighRisks() { return highRisks; }
        public List<String> getMediumRisks() { return mediumRisks; }
        public List<String> getLowRisks() { return lowRisks; }
    }
    
    public static class ComparisonMatrix {
        private List<TechEvaluationResult> results;
        
        public ComparisonMatrix(List<TechEvaluationResult> results) {
            this.results = results;
        }
        
        public List<TechEvaluationResult> getResults() { return results; }
    }
}
```

**关键技术点：**

1. **多维度评估体系**：
   - 业务匹配度：功能覆盖、性能匹配、扩展性
   - 技术成熟度：版本稳定性、社区活跃度、文档质量
   - 团队适配度：学习成本、技能匹配、维护难度
   - 成本效益：开发成本、运维成本、许可费用
   - 生态兼容性：技术栈兼容、工具链支持

2. **量化评分机制**：
   - 权重分配：根据项目重要性设置各维度权重
   - 评分标准：1-10分制，标准化评估流程
   - 综合计算：加权平均得出最终评分

3. **风险识别与评估**：
   - 高风险项：评分低于4分的关键因素
   - 中风险项：评分4-6分的注意事项
   - 低风险项：评分高于6分的优势项

4. **对比分析功能**：
   - 多方案对比：并行评估多个技术选项
   - 差异分析：识别关键差异点
   - 推荐排序：基于综合评分排序

5. **决策支持工具**：
   - 评估报告：详细的评估结果和建议
   - 风险缓解：针对识别的风险提供解决方案
   - 实施路径：技术选型后的实施建议

**选型决策流程：**

1. **需求分析**：明确项目需求和约束条件
2. **候选技术调研**：收集潜在技术方案信息
3. **多维度评估**：使用评估框架进行量化分析
4. **风险评估**：识别和评估潜在风险
5. **方案对比**：比较不同技术方案的优劣
6. **决策制定**：基于评估结果做出最终选择
7. **实施规划**：制定技术实施和迁移计划

**7. 消息队列的选型考虑因素有哪些？**

消息队列选型是分布式系统架构中的重要决策，需要综合考虑性能、可靠性、功能特性、运维成本等多个维度。

**消息队列选型对比分析：**

| 选型维度 | RabbitMQ | Apache Kafka | RocketMQ | ActiveMQ | Redis Streams |
|----------|----------|--------------|----------|----------|---------------|
| 吞吐量 | 中等(万级/秒) | 极高(百万级/秒) | 高(十万级/秒) | 中等(万级/秒) | 高(十万级/秒) |
| 延迟 | 低(<1ms) | 中等(2-4ms) | 低(<1ms) | 中等(1-5ms) | 极低(<0.1ms) |
| 可靠性 | 高 | 极高 | 极高 | 高 | 中等 |
| 扩展性 | 中等 | 极高 | 高 | 低 | 中等 |
| 功能丰富度 | 极高 | 中等 | 高 | 高 | 低 |
| 运维复杂度 | 中等 | 高 | 中等 | 低 | 低 |
| 社区活跃度 | 高 | 极高 | 高 | 中等 | 高 |

**详细实现方案：**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.stereotype.Service;
import org.springframework.stereotype.Component;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.bind.annotation.*;

import java.util.*;
import java.time.LocalDateTime;
import java.util.stream.Collectors;
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.TimeUnit;

/**
 * 消息队列选型决策支持系统
 */
@SpringBootApplication
public class MessageQueueSelectionApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(MessageQueueSelectionApplication.class, args);
    }
    
    // ==================== 1. 消息队列选型评估框架 ====================
    
    /**
     * 消息队列选型评估服务
     */
    @Service
    public class MQSelectionService {
        
        @Autowired
        private List<MQEvaluationCriteria> evaluationCriteriaList;
        
        /**
         * 综合评估消息队列方案
         */
        public MQEvaluationResult evaluateMessageQueue(MQOption mqOption, BusinessRequirements requirements) {
            List<MQCriteriaScore> scores = new ArrayList<>();
            BigDecimal totalScore = BigDecimal.ZERO;
            BigDecimal totalWeight = BigDecimal.ZERO;
            
            for (MQEvaluationCriteria criteria : evaluationCriteriaList) {
                MQCriteriaScore score = criteria.evaluate(mqOption, requirements);
                scores.add(score);
                
                BigDecimal weightedScore = score.getScore().multiply(criteria.getWeight());
                totalScore = totalScore.add(weightedScore);
                totalWeight = totalWeight.add(criteria.getWeight());
            }
            
            BigDecimal finalScore = totalScore.divide(totalWeight, 2, RoundingMode.HALF_UP);
            
            return new MQEvaluationResult(
                mqOption.getName(),
                finalScore,
                scores,
                generateMQRecommendation(finalScore, scores, requirements),
                assessMQRisk(scores, requirements),
                generateDeploymentPlan(mqOption, requirements)
            );
        }
        
        /**
         * 比较多个消息队列方案
         */
        public MQComparisonResult compareMessageQueues(List<MQOption> options, BusinessRequirements requirements) {
            List<MQEvaluationResult> results = options.stream()
                .map(option -> evaluateMessageQueue(option, requirements))
                .sorted((r1, r2) -> r2.getFinalScore().compareTo(r1.getFinalScore()))
                .collect(Collectors.toList());
            
            return new MQComparisonResult(
                results,
                generateMQComparisonMatrix(results),
                recommendBestMQOption(results, requirements),
                identifyMQKeyDifferences(results),
                generateMigrationStrategy(results, requirements)
            );
        }
        
        /**
         * 生成选型建议
         */
        private String generateMQRecommendation(BigDecimal score, List<MQCriteriaScore> scores, BusinessRequirements requirements) {
            StringBuilder recommendation = new StringBuilder();
            
            if (score.compareTo(new BigDecimal("8.5")) >= 0) {
                recommendation.append("强烈推荐：该消息队列完全满足业务需求，各方面表现优秀。");
            } else if (score.compareTo(new BigDecimal("7.0")) >= 0) {
                recommendation.append("推荐：该消息队列基本满足需求，建议进行POC验证。");
            } else if (score.compareTo(new BigDecimal("5.0")) >= 0) {
                recommendation.append("谨慎考虑：该消息队列存在一些限制，需要评估风险。");
            } else {
                recommendation.append("不推荐：该消息队列不适合当前场景。");
            }
            
            // 添加具体建议
            if (requirements.getExpectedThroughput() > 100000) {
                recommendation.append(" 高吞吐量场景建议优先考虑Kafka或RocketMQ。");
            }
            
            if (requirements.getMaxLatency() < 10) {
                recommendation.append(" 低延迟要求建议考虑Redis Streams或RabbitMQ。");
            }
            
            if (requirements.requiresComplexRouting()) {
                recommendation.append(" 复杂路由需求建议选择RabbitMQ。");
            }
            
            return recommendation.toString();
        }
        
        /**
         * 风险评估
         */
        private MQRiskAssessment assessMQRisk(List<MQCriteriaScore> scores, BusinessRequirements requirements) {
            List<String> highRisks = new ArrayList<>();
            List<String> mediumRisks = new ArrayList<>();
            List<String> lowRisks = new ArrayList<>();
            List<String> mitigationStrategies = new ArrayList<>();
            
            for (MQCriteriaScore score : scores) {
                if (score.getScore().compareTo(new BigDecimal("4.0")) < 0) {
                    highRisks.add(score.getCriteriaName() + ": " + score.getComment());
                    mitigationStrategies.add(generateMitigationStrategy(score.getCriteriaName(), requirements));
                } else if (score.getScore().compareTo(new BigDecimal("6.0")) < 0) {
                    mediumRisks.add(score.getCriteriaName() + ": " + score.getComment());
                } else {
                    lowRisks.add(score.getCriteriaName() + ": " + score.getComment());
                }
            }
            
            return new MQRiskAssessment(highRisks, mediumRisks, lowRisks, mitigationStrategies);
        }
        
        /**
         * 生成风险缓解策略
         */
        private String generateMitigationStrategy(String criteriaName, BusinessRequirements requirements) {
            switch (criteriaName) {
                case "性能表现":
                    return "考虑集群部署、分区优化、批量处理等性能优化策略";
                case "可靠性保障":
                    return "实施多副本、定期备份、故障转移等可靠性措施";
                case "扩展能力":
                    return "设计水平扩展方案、实施分片策略、预留扩容空间";
                case "功能完整性":
                    return "通过中间件或自定义开发补充缺失功能";
                case "运维便利性":
                    return "加强监控告警、自动化运维、团队培训";
                default:
                    return "制定针对性的风险应对措施";
            }
        }
        
        /**
         * 生成部署方案
         */
        private DeploymentPlan generateDeploymentPlan(MQOption mqOption, BusinessRequirements requirements) {
            DeploymentPlan plan = new DeploymentPlan();
            
            // 集群规模建议
            int recommendedNodes = calculateRecommendedNodes(requirements.getExpectedThroughput(), 
                                                           requirements.getExpectedMessageSize());
            plan.setRecommendedNodes(recommendedNodes);
            
            // 资源配置建议
            ResourceConfiguration resourceConfig = calculateResourceRequirements(mqOption, requirements);
            plan.setResourceConfiguration(resourceConfig);
            
            // 网络配置建议
            NetworkConfiguration networkConfig = generateNetworkConfiguration(mqOption, requirements);
            plan.setNetworkConfiguration(networkConfig);
            
            // 监控配置建议
            MonitoringConfiguration monitoringConfig = generateMonitoringConfiguration(mqOption);
            plan.setMonitoringConfiguration(monitoringConfig);
            
            return plan;
        }
        
        private int calculateRecommendedNodes(long throughput, int messageSize) {
            // 基于吞吐量和消息大小计算推荐节点数
            long totalDataPerSecond = throughput * messageSize;
            
            if (totalDataPerSecond < 100 * 1024 * 1024) { // < 100MB/s
                return 3; // 最小集群
            } else if (totalDataPerSecond < 1024 * 1024 * 1024) { // < 1GB/s
                return 5;
            } else {
                return Math.max(7, (int) (totalDataPerSecond / (200 * 1024 * 1024))); // 每200MB/s一个节点
            }
        }
        
        private ResourceConfiguration calculateResourceRequirements(MQOption mqOption, BusinessRequirements requirements) {
            ResourceConfiguration config = new ResourceConfiguration();
            
            // CPU配置
            int cpuCores = Math.max(4, (int) (requirements.getExpectedThroughput() / 10000));
            config.setCpuCores(cpuCores);
            
            // 内存配置
            int memoryGB = Math.max(8, (int) (requirements.getExpectedThroughput() * requirements.getExpectedMessageSize() / (1024 * 1024 * 100)));
            config.setMemoryGB(memoryGB);
            
            // 存储配置
            long storageGB = calculateStorageRequirements(requirements);
            config.setStorageGB(storageGB);
            
            return config;
        }
        
        private long calculateStorageRequirements(BusinessRequirements requirements) {
            // 基于消息保留时间、吞吐量、消息大小计算存储需求
            long dailyData = requirements.getExpectedThroughput() * requirements.getExpectedMessageSize() * 86400;
            long totalData = dailyData * requirements.getMessageRetentionDays();
            
            // 添加30%的缓冲空间
            return (long) (totalData * 1.3 / (1024 * 1024 * 1024));
        }
        
        private NetworkConfiguration generateNetworkConfiguration(MQOption mqOption, BusinessRequirements requirements) {
            NetworkConfiguration config = new NetworkConfiguration();
            
            // 带宽要求
            long bandwidthMbps = requirements.getExpectedThroughput() * requirements.getExpectedMessageSize() * 8 / (1024 * 1024);
            config.setRequiredBandwidthMbps(Math.max(100, bandwidthMbps * 2)); // 2倍冗余
            
            // 网络延迟要求
            config.setMaxNetworkLatencyMs(Math.min(10, requirements.getMaxLatency() / 2));
            
            // 端口配置
            config.setPorts(mqOption.getDefaultPorts());
            
            return config;
        }
        
        private MonitoringConfiguration generateMonitoringConfiguration(MQOption mqOption) {
            MonitoringConfiguration config = new MonitoringConfiguration();
            
            // 关键指标
            List<String> keyMetrics = Arrays.asList(
                "message_throughput", "message_latency", "queue_depth",
                "consumer_lag", "error_rate", "connection_count"
            );
            config.setKeyMetrics(keyMetrics);
            
            // 告警阈值
            Map<String, String> alertThresholds = new HashMap<>();
            alertThresholds.put("queue_depth", "> 10000");
            alertThresholds.put("consumer_lag", "> 1000");
            alertThresholds.put("error_rate", "> 1%");
            config.setAlertThresholds(alertThresholds);
            
            return config;
        }
        
        // 其他辅助方法的实现...
        private MQComparisonMatrix generateMQComparisonMatrix(List<MQEvaluationResult> results) {
            return new MQComparisonMatrix(results);
        }
        
        private String recommendBestMQOption(List<MQEvaluationResult> results, BusinessRequirements requirements) {
            if (results.isEmpty()) return "无可用选项";
            
            MQEvaluationResult best = results.get(0);
            return String.format("推荐选择 %s，综合评分: %.2f，最适合当前业务场景", 
                best.getMqName(), best.getFinalScore());
        }
        
        private List<String> identifyMQKeyDifferences(List<MQEvaluationResult> results) {
            List<String> differences = new ArrayList<>();
            
            if (results.size() >= 2) {
                MQEvaluationResult first = results.get(0);
                MQEvaluationResult second = results.get(1);
                
                BigDecimal scoreDiff = first.getFinalScore().subtract(second.getFinalScore());
                differences.add(String.format("%s 比 %s 综合评分高 %.2f 分", 
                    first.getMqName(), second.getMqName(), scoreDiff));
                
                // 分析具体差异
                for (int i = 0; i < first.getScores().size(); i++) {
                    MQCriteriaScore firstScore = first.getScores().get(i);
                    MQCriteriaScore secondScore = second.getScores().get(i);
                    
                    BigDecimal diff = firstScore.getScore().subtract(secondScore.getScore());
                    if (diff.abs().compareTo(new BigDecimal("2.0")) >= 0) {
                        differences.add(String.format("%s 在 %s 方面明显优于 %s (差距%.1f分)", 
                            diff.compareTo(BigDecimal.ZERO) > 0 ? first.getMqName() : second.getMqName(),
                            firstScore.getCriteriaName(),
                            diff.compareTo(BigDecimal.ZERO) > 0 ? second.getMqName() : first.getMqName(),
                            diff.abs()));
                    }
                }
            }
            
            return differences;
        }
        
        private MigrationStrategy generateMigrationStrategy(List<MQEvaluationResult> results, BusinessRequirements requirements) {
            if (results.isEmpty()) return new MigrationStrategy();
            
            MQEvaluationResult bestOption = results.get(0);
            MigrationStrategy strategy = new MigrationStrategy();
            
            // 迁移阶段
            List<String> phases = Arrays.asList(
                "1. 环境准备和集群搭建",
                "2. 数据迁移工具开发",
                "3. 灰度迁移和测试",
                "4. 全量迁移和切换",
                "5. 监控和优化"
            );
            strategy.setMigrationPhases(phases);
            
            // 风险控制
            List<String> riskControls = Arrays.asList(
                "双写方案确保数据一致性",
                "回滚方案应对异常情况",
                "监控告警及时发现问题",
                "分批迁移降低影响范围"
            );
            strategy.setRiskControls(riskControls);
            
            // 预估时间
            int estimatedDays = calculateMigrationTime(requirements);
            strategy.setEstimatedDays(estimatedDays);
            
            return strategy;
        }
        
        private int calculateMigrationTime(BusinessRequirements requirements) {
            int baseDays = 30; // 基础迁移时间
            
            // 根据数据量调整
            if (requirements.getExpectedThroughput() > 100000) {
                baseDays += 15;
            }
            
            // 根据复杂度调整
            if (requirements.requiresComplexRouting()) {
                baseDays += 10;
            }
            
            return baseDays;
        }
    }
    
    // ==================== 2. 具体评估标准实现 ====================
    
    /**
     * 性能表现评估
     */
    @Component
    public class PerformanceCriteria implements MQEvaluationCriteria {
        
        @Override
        public String getName() {
            return "性能表现";
        }
        
        @Override
        public BigDecimal getWeight() {
            return new BigDecimal("0.30");
        }
        
        @Override
        public MQCriteriaScore evaluate(MQOption mqOption, BusinessRequirements requirements) {
            BigDecimal score = BigDecimal.ZERO;
            List<String> comments = new ArrayList<>();
            
            // 吞吐量评估
            int throughputScore = evaluateThroughput(mqOption, requirements.getExpectedThroughput());
            score = score.add(new BigDecimal(throughputScore * 0.5));
            comments.add("吞吐量: " + throughputScore + "/10");
            
            // 延迟评估
            int latencyScore = evaluateLatency(mqOption, requirements.getMaxLatency());
            score = score.add(new BigDecimal(latencyScore * 0.3));
            comments.add("延迟: " + latencyScore + "/10");
            
            // 并发处理能力评估
            int concurrencyScore = evaluateConcurrency(mqOption, requirements.getExpectedConcurrency());
            score = score.add(new BigDecimal(concurrencyScore * 0.2));
            comments.add("并发能力: " + concurrencyScore + "/10");
            
            return new MQCriteriaScore(getName(), score, String.join(", ", comments));
        }
        
        private int evaluateThroughput(MQOption mqOption, long expectedThroughput) {
            long maxThroughput = mqOption.getMaxThroughput();
            
            if (maxThroughput >= expectedThroughput * 2) {
                return 10; // 性能充足
            } else if (maxThroughput >= expectedThroughput * 1.5) {
                return 8; // 性能良好
            } else if (maxThroughput >= expectedThroughput) {
                return 6; // 刚好满足
            } else if (maxThroughput >= expectedThroughput * 0.8) {
                return 4; // 略有不足
            } else {
                return 2; // 明显不足
            }
        }
        
        private int evaluateLatency(MQOption mqOption, int maxLatency) {
            int avgLatency = mqOption.getAverageLatency();
            
            if (avgLatency <= maxLatency * 0.5) {
                return 10;
            } else if (avgLatency <= maxLatency * 0.7) {
                return 8;
            } else if (avgLatency <= maxLatency) {
                return 6;
            } else if (avgLatency <= maxLatency * 1.5) {
                return 4;
            } else {
                return 2;
            }
        }
        
        private int evaluateConcurrency(MQOption mqOption, int expectedConcurrency) {
            int maxConcurrency = mqOption.getMaxConcurrentConnections();
            
            if (maxConcurrency >= expectedConcurrency * 2) {
                return 10;
            } else if (maxConcurrency >= expectedConcurrency) {
                return 8;
            } else {
                return Math.max(2, (maxConcurrency * 10) / expectedConcurrency);
            }
        }
    }
    
    /**
     * 可靠性保障评估
     */
    @Component
    public class ReliabilityCriteria implements MQEvaluationCriteria {
        
        @Override
        public String getName() {
            return "可靠性保障";
        }
        
        @Override
        public BigDecimal getWeight() {
            return new BigDecimal("0.25");
        }
        
        @Override
        public MQCriteriaScore evaluate(MQOption mqOption, BusinessRequirements requirements) {
            BigDecimal score = BigDecimal.ZERO;
            List<String> comments = new ArrayList<>();
            
            // 消息持久化
            int persistenceScore = evaluatePersistence(mqOption, requirements.requiresPersistence());
            score = score.add(new BigDecimal(persistenceScore * 0.4));
            comments.add("持久化: " + persistenceScore + "/10");
            
            // 故障恢复
            int recoveryScore = evaluateRecovery(mqOption);
            score = score.add(new BigDecimal(recoveryScore * 0.3));
            comments.add("故障恢复: " + recoveryScore + "/10");
            
            // 数据一致性
            int consistencyScore = evaluateConsistency(mqOption, requirements.getConsistencyLevel());
            score = score.add(new BigDecimal(consistencyScore * 0.3));
            comments.add("数据一致性: " + consistencyScore + "/10");
            
            return new MQCriteriaScore(getName(), score, String.join(", ", comments));
        }
        
        private int evaluatePersistence(MQOption mqOption, boolean requiresPersistence) {
            if (!requiresPersistence) return 10; // 不需要持久化
            
            if (mqOption.supportsPersistence()) {
                if (mqOption.hasReplication()) {
                    return 10; // 支持持久化和副本
                } else {
                    return 7; // 仅支持持久化
                }
            } else {
                return 2; // 不支持持久化
            }
        }
        
        private int evaluateRecovery(MQOption mqOption) {
            int score = 5;
            
            if (mqOption.supportsAutoFailover()) score += 3;
            if (mqOption.hasBackupMechanism()) score += 2;
            
            return Math.min(10, score);
        }
        
        private int evaluateConsistency(MQOption mqOption, String consistencyLevel) {
            switch (consistencyLevel.toLowerCase()) {
                case "strong":
                    return mqOption.supportsStrongConsistency() ? 10 : 4;
                case "eventual":
                    return mqOption.supportsEventualConsistency() ? 8 : 6;
                case "weak":
                    return 8; // 大多数MQ都能满足弱一致性
                default:
                    return 6;
            }
        }
    }
    
    /**
     * 扩展能力评估
     */
    @Component
    public class ScalabilityCriteria implements MQEvaluationCriteria {
        
        @Override
        public String getName() {
            return "扩展能力";
        }
        
        @Override
        public BigDecimal getWeight() {
            return new BigDecimal("0.20");
        }
        
        @Override
        public MQCriteriaScore evaluate(MQOption mqOption, BusinessRequirements requirements) {
            BigDecimal score = BigDecimal.ZERO;
            List<String> comments = new ArrayList<>();
            
            // 水平扩展
            int horizontalScore = evaluateHorizontalScaling(mqOption);
            score = score.add(new BigDecimal(horizontalScore * 0.6));
            comments.add("水平扩展: " + horizontalScore + "/10");
            
            // 分区支持
            int partitionScore = evaluatePartitioning(mqOption, requirements.requiresPartitioning());
            score = score.add(new BigDecimal(partitionScore * 0.4));
            comments.add("分区支持: " + partitionScore + "/10");
            
            return new MQCriteriaScore(getName(), score, String.join(", ", comments));
        }
        
        private int evaluateHorizontalScaling(MQOption mqOption) {
            if (mqOption.supportsHorizontalScaling()) {
                if (mqOption.hasAutoScaling()) {
                    return 10;
                } else {
                    return 7;
                }
            } else {
                return 3;
            }
        }
        
        private int evaluatePartitioning(MQOption mqOption, boolean requiresPartitioning) {
            if (!requiresPartitioning) return 8;
            
            if (mqOption.supportsPartitioning()) {
                if (mqOption.hasAdvancedPartitioning()) {
                    return 10;
                } else {
                    return 7;
                }
            } else {
                return 3;
            }
        }
    }
    
    /**
     * 功能完整性评估
     */
    @Component
    public class FeatureCompletenessCriteria implements MQEvaluationCriteria {
        
        @Override
        public String getName() {
            return "功能完整性";
        }
        
        @Override
        public BigDecimal getWeight() {
            return new BigDecimal("0.15");
        }
        
        @Override
        public MQCriteriaScore evaluate(MQOption mqOption, BusinessRequirements requirements) {
            BigDecimal score = BigDecimal.ZERO;
            List<String> comments = new ArrayList<>();
            
            // 消息路由
            int routingScore = evaluateRouting(mqOption, requirements.requiresComplexRouting());
            score = score.add(new BigDecimal(routingScore * 0.3));
            comments.add("消息路由: " + routingScore + "/10");
            
            // 死信队列
            int dlqScore = evaluateDeadLetterQueue(mqOption, requirements.requiresDeadLetterQueue());
            score = score.add(new BigDecimal(dlqScore * 0.2));
            comments.add("死信队列: " + dlqScore + "/10");
            
            // 延迟消息
            int delayScore = evaluateDelayedMessage(mqOption, requirements.requiresDelayedMessage());
            score = score.add(new BigDecimal(delayScore * 0.2));
            comments.add("延迟消息: " + delayScore + "/10");
            
            // 事务支持
            int transactionScore = evaluateTransaction(mqOption, requirements.requiresTransaction());
            score = score.add(new BigDecimal(transactionScore * 0.3));
            comments.add("事务支持: " + transactionScore + "/10");
            
            return new MQCriteriaScore(getName(), score, String.join(", ", comments));
        }
        
        private int evaluateRouting(MQOption mqOption, boolean requiresComplexRouting) {
            if (!requiresComplexRouting) return 8;
            
            if (mqOption.supportsComplexRouting()) {
                return 10;
            } else if (mqOption.supportsBasicRouting()) {
                return 6;
            } else {
                return 3;
            }
        }
        
        private int evaluateDeadLetterQueue(MQOption mqOption, boolean requiresDLQ) {
            if (!requiresDLQ) return 8;
            
            return mqOption.supportsDeadLetterQueue() ? 10 : 4;
        }
        
        private int evaluateDelayedMessage(MQOption mqOption, boolean requiresDelayed) {
            if (!requiresDelayed) return 8;
            
            return mqOption.supportsDelayedMessage() ? 10 : 4;
        }
        
        private int evaluateTransaction(MQOption mqOption, boolean requiresTransaction) {
            if (!requiresTransaction) return 8;
            
            if (mqOption.supportsDistributedTransaction()) {
                return 10;
            } else if (mqOption.supportsLocalTransaction()) {
                return 6;
            } else {
                return 3;
            }
        }
    }
    
    /**
     * 运维便利性评估
     */
    @Component
    public class OperationalEaseCriteria implements MQEvaluationCriteria {
        
        @Override
        public String getName() {
            return "运维便利性";
        }
        
        @Override
        public BigDecimal getWeight() {
            return new BigDecimal("0.10");
        }
        
        @Override
        public MQCriteriaScore evaluate(MQOption mqOption, BusinessRequirements requirements) {
            BigDecimal score = BigDecimal.ZERO;
            List<String> comments = new ArrayList<>();
            
            // 部署复杂度
            int deploymentScore = evaluateDeploymentComplexity(mqOption);
            score = score.add(new BigDecimal(deploymentScore * 0.3));
            comments.add("部署复杂度: " + (10 - deploymentScore) + "/10");
            
            // 监控工具
            int monitoringScore = evaluateMonitoringTools(mqOption);
            score = score.add(new BigDecimal(monitoringScore * 0.4));
            comments.add("监控工具: " + monitoringScore + "/10");
            
            // 社区支持
            int communityScore = evaluateCommunitySupport(mqOption);
            score = score.add(new BigDecimal(communityScore * 0.3));
            comments.add("社区支持: " + communityScore + "/10");
            
            return new MQCriteriaScore(getName(), score, String.join(", ", comments));
        }
        
        private int evaluateDeploymentComplexity(MQOption mqOption) {
            // 分数越高表示部署越简单
            if (mqOption.hasDockerSupport() && mqOption.hasKubernetesSupport()) {
                return 9;
            } else if (mqOption.hasDockerSupport()) {
                return 7;
            } else {
                return 5;
            }
        }
        
        private int evaluateMonitoringTools(MQOption mqOption) {
            int score = 5;
            
            if (mqOption.hasBuiltInMonitoring()) score += 2;
            if (mqOption.supportsPrometheusMetrics()) score += 2;
            if (mqOption.hasWebUI()) score += 1;
            
            return Math.min(10, score);
        }
        
        private int evaluateCommunitySupport(MQOption mqOption) {
            int score = 5;
            
            if (mqOption.hasActiveCommunity()) score += 2;
            if (mqOption.hasGoodDocumentation()) score += 2;
            if (mqOption.hasCommercialSupport()) score += 1;
            
            return Math.min(10, score);
        }
    }
    
    // ==================== 3. 数据模型定义 ====================
    
    /**
     * 消息队列选项
     */
    public static class MQOption {
        private String name;
        private String version;
        private long maxThroughput;
        private int averageLatency;
        private int maxConcurrentConnections;
        private boolean supportsPersistence;
        private boolean hasReplication;
        private boolean supportsAutoFailover;
        private boolean hasBackupMechanism;
        private boolean supportsStrongConsistency;
        private boolean supportsEventualConsistency;
        private boolean supportsHorizontalScaling;
        private boolean hasAutoScaling;
        private boolean supportsPartitioning;
        private boolean hasAdvancedPartitioning;
        private boolean supportsComplexRouting;
        private boolean supportsBasicRouting;
        private boolean supportsDeadLetterQueue;
        private boolean supportsDelayedMessage;
        private boolean supportsDistributedTransaction;
        private boolean supportsLocalTransaction;
        private boolean hasDockerSupport;
        private boolean hasKubernetesSupport;
        private boolean hasBuiltInMonitoring;
        private boolean supportsPrometheusMetrics;
        private boolean hasWebUI;
        private boolean hasActiveCommunity;
        private boolean hasGoodDocumentation;
        private boolean hasCommercialSupport;
        private List<Integer> defaultPorts;
        
        // Constructor
        public MQOption(String name, String version) {
            this.name = name;
            this.version = version;
            this.defaultPorts = new ArrayList<>();
        }
        
        // Getters and Setters
        public String getName() { return name; }
        public String getVersion() { return version; }
        public long getMaxThroughput() { return maxThroughput; }
        public int getAverageLatency() { return averageLatency; }
        public int getMaxConcurrentConnections() { return maxConcurrentConnections; }
        public boolean supportsPersistence() { return supportsPersistence; }
        public boolean hasReplication() { return hasReplication; }
        public boolean supportsAutoFailover() { return supportsAutoFailover; }
        public boolean hasBackupMechanism() { return hasBackupMechanism; }
        public boolean supportsStrongConsistency() { return supportsStrongConsistency; }
        public boolean supportsEventualConsistency() { return supportsEventualConsistency; }
        public boolean supportsHorizontalScaling() { return supportsHorizontalScaling; }
        public boolean hasAutoScaling() { return hasAutoScaling; }
        public boolean supportsPartitioning() { return supportsPartitioning; }
        public boolean hasAdvancedPartitioning() { return hasAdvancedPartitioning; }
        public boolean supportsComplexRouting() { return supportsComplexRouting; }
        public boolean supportsBasicRouting() { return supportsBasicRouting; }
        public boolean supportsDeadLetterQueue() { return supportsDeadLetterQueue; }
        public boolean supportsDelayedMessage() { return supportsDelayedMessage; }
        public boolean supportsDistributedTransaction() { return supportsDistributedTransaction; }
        public boolean supportsLocalTransaction() { return supportsLocalTransaction; }
        public boolean hasDockerSupport() { return hasDockerSupport; }
        public boolean hasKubernetesSupport() { return hasKubernetesSupport; }
        public boolean hasBuiltInMonitoring() { return hasBuiltInMonitoring; }
        public boolean supportsPrometheusMetrics() { return supportsPrometheusMetrics; }
        public boolean hasWebUI() { return hasWebUI; }
        public boolean hasActiveCommunity() { return hasActiveCommunity; }
        public boolean hasGoodDocumentation() { return hasGoodDocumentation; }
        public boolean hasCommercialSupport() { return hasCommercialSupport; }
        public List<Integer> getDefaultPorts() { return defaultPorts; }
        
        // Setters
        public void setMaxThroughput(long maxThroughput) { this.maxThroughput = maxThroughput; }
        public void setAverageLatency(int averageLatency) { this.averageLatency = averageLatency; }
        public void setMaxConcurrentConnections(int maxConcurrentConnections) { this.maxConcurrentConnections = maxConcurrentConnections; }
        public void setSupportsPersistence(boolean supportsPersistence) { this.supportsPersistence = supportsPersistence; }
        public void setHasReplication(boolean hasReplication) { this.hasReplication = hasReplication; }
        public void setSupportsAutoFailover(boolean supportsAutoFailover) { this.supportsAutoFailover = supportsAutoFailover; }
        public void setHasBackupMechanism(boolean hasBackupMechanism) { this.hasBackupMechanism = hasBackupMechanism; }
        public void setSupportsStrongConsistency(boolean supportsStrongConsistency) { this.supportsStrongConsistency = supportsStrongConsistency; }
        public void setSupportsEventualConsistency(boolean supportsEventualConsistency) { this.supportsEventualConsistency = supportsEventualConsistency; }
        public void setSupportsHorizontalScaling(boolean supportsHorizontalScaling) { this.supportsHorizontalScaling = supportsHorizontalScaling; }
        public void setHasAutoScaling(boolean hasAutoScaling) { this.hasAutoScaling = hasAutoScaling; }
        public void setSupportsPartitioning(boolean supportsPartitioning) { this.supportsPartitioning = supportsPartitioning; }
        public void setHasAdvancedPartitioning(boolean hasAdvancedPartitioning) { this.hasAdvancedPartitioning = hasAdvancedPartitioning; }
        public void setSupportsComplexRouting(boolean supportsComplexRouting) { this.supportsComplexRouting = supportsComplexRouting; }
        public void setSupportsBasicRouting(boolean supportsBasicRouting) { this.supportsBasicRouting = supportsBasicRouting; }
        public void setSupportsDeadLetterQueue(boolean supportsDeadLetterQueue) { this.supportsDeadLetterQueue = supportsDeadLetterQueue; }
        public void setSupportsDelayedMessage(boolean supportsDelayedMessage) { this.supportsDelayedMessage = supportsDelayedMessage; }
        public void setSupportsDistributedTransaction(boolean supportsDistributedTransaction) { this.supportsDistributedTransaction = supportsDistributedTransaction; }
        public void setSupportsLocalTransaction(boolean supportsLocalTransaction) { this.supportsLocalTransaction = supportsLocalTransaction; }
        public void setHasDockerSupport(boolean hasDockerSupport) { this.hasDockerSupport = hasDockerSupport; }
        public void setHasKubernetesSupport(boolean hasKubernetesSupport) { this.hasKubernetesSupport = hasKubernetesSupport; }
        public void setHasBuiltInMonitoring(boolean hasBuiltInMonitoring) { this.hasBuiltInMonitoring = hasBuiltInMonitoring; }
        public void setSupportsPrometheusMetrics(boolean supportsPrometheusMetrics) { this.supportsPrometheusMetrics = supportsPrometheusMetrics; }
        public void setHasWebUI(boolean hasWebUI) { this.hasWebUI = hasWebUI; }
        public void setHasActiveCommunity(boolean hasActiveCommunity) { this.hasActiveCommunity = hasActiveCommunity; }
        public void setHasGoodDocumentation(boolean hasGoodDocumentation) { this.hasGoodDocumentation = hasGoodDocumentation; }
        public void setHasCommercialSupport(boolean hasCommercialSupport) { this.hasCommercialSupport = hasCommercialSupport; }
    }
    
    // ==================== 4. 辅助类定义 ====================
    
    public interface MQEvaluationCriteria {
        String getName();
        BigDecimal getWeight();
        MQCriteriaScore evaluate(MQOption mqOption, BusinessRequirements requirements);
    }
    
    public static class MQCriteriaScore {
        private String criteriaName;
        private BigDecimal score;
        private String comment;
        
        public MQCriteriaScore(String criteriaName, BigDecimal score, String comment) {
            this.criteriaName = criteriaName;
            this.score = score;
            this.comment = comment;
        }
        
        public String getCriteriaName() { return criteriaName; }
        public BigDecimal getScore() { return score; }
        public String getComment() { return comment; }
    }
    
    public static class MQEvaluationResult {
        private String mqName;
        private BigDecimal finalScore;
        private List<MQCriteriaScore> scores;
        private String recommendation;
        private MQRiskAssessment riskAssessment;
        private DeploymentPlan deploymentPlan;
        
        public MQEvaluationResult(String mqName, BigDecimal finalScore, 
                                List<MQCriteriaScore> scores, String recommendation, 
                                MQRiskAssessment riskAssessment, DeploymentPlan deploymentPlan) {
            this.mqName = mqName;
            this.finalScore = finalScore;
            this.scores = scores;
            this.recommendation = recommendation;
            this.riskAssessment = riskAssessment;
            this.deploymentPlan = deploymentPlan;
        }
        
        public String getMqName() { return mqName; }
        public BigDecimal getFinalScore() { return finalScore; }
        public List<MQCriteriaScore> getScores() { return scores; }
        public String getRecommendation() { return recommendation; }
        public MQRiskAssessment getRiskAssessment() { return riskAssessment; }
        public DeploymentPlan getDeploymentPlan() { return deploymentPlan; }
    }
    
    public static class BusinessRequirements {
        private long expectedThroughput;
        private int maxLatency;
        private int expectedConcurrency;
        private int expectedMessageSize;
        private int messageRetentionDays;
        private boolean requiresPersistence;
        private String consistencyLevel;
        private boolean requiresPartitioning;
        private boolean requiresComplexRouting;
        private boolean requiresDeadLetterQueue;
        private boolean requiresDelayedMessage;
        private boolean requiresTransaction;
        
        // Getters and Setters
        public long getExpectedThroughput() { return expectedThroughput; }
        public int getMaxLatency() { return maxLatency; }
        public int getExpectedConcurrency() { return expectedConcurrency; }
        public int getExpectedMessageSize() { return expectedMessageSize; }
        public int getMessageRetentionDays() { return messageRetentionDays; }
        public boolean requiresPersistence() { return requiresPersistence; }
        public String getConsistencyLevel() { return consistencyLevel; }
        public boolean requiresPartitioning() { return requiresPartitioning; }
        public boolean requiresComplexRouting() { return requiresComplexRouting; }
        public boolean requiresDeadLetterQueue() { return requiresDeadLetterQueue; }
        public boolean requiresDelayedMessage() { return requiresDelayedMessage; }
        public boolean requiresTransaction() { return requiresTransaction; }
        
        public void setExpectedThroughput(long expectedThroughput) { this.expectedThroughput = expectedThroughput; }
        public void setMaxLatency(int maxLatency) { this.maxLatency = maxLatency; }
        public void setExpectedConcurrency(int expectedConcurrency) { this.expectedConcurrency = expectedConcurrency; }
        public void setExpectedMessageSize(int expectedMessageSize) { this.expectedMessageSize = expectedMessageSize; }
        public void setMessageRetentionDays(int messageRetentionDays) { this.messageRetentionDays = messageRetentionDays; }
        public void setRequiresPersistence(boolean requiresPersistence) { this.requiresPersistence = requiresPersistence; }
        public void setConsistencyLevel(String consistencyLevel) { this.consistencyLevel = consistencyLevel; }
        public void setRequiresPartitioning(boolean requiresPartitioning) { this.requiresPartitioning = requiresPartitioning; }
        public void setRequiresComplexRouting(boolean requiresComplexRouting) { this.requiresComplexRouting = requiresComplexRouting; }
        public void setRequiresDeadLetterQueue(boolean requiresDeadLetterQueue) { this.requiresDeadLetterQueue = requiresDeadLetterQueue; }
        public void setRequiresDelayedMessage(boolean requiresDelayedMessage) { this.requiresDelayedMessage = requiresDelayedMessage; }
        public void setRequiresTransaction(boolean requiresTransaction) { this.requiresTransaction = requiresTransaction; }
    }
    
    // 其他辅助类的简化定义
    public static class MQRiskAssessment {
        private List<String> highRisks;
        private List<String> mediumRisks;
        private List<String> lowRisks;
        private List<String> mitigationStrategies;
        
        public MQRiskAssessment(List<String> highRisks, List<String> mediumRisks, 
                              List<String> lowRisks, List<String> mitigationStrategies) {
            this.highRisks = highRisks;
            this.mediumRisks = mediumRisks;
            this.lowRisks = lowRisks;
            this.mitigationStrategies = mitigationStrategies;
        }
        
        public List<String> getHighRisks() { return highRisks; }
        public List<String> getMediumRisks() { return mediumRisks; }
        public List<String> getLowRisks() { return lowRisks; }
        public List<String> getMitigationStrategies() { return mitigationStrategies; }
    }
    
    public static class DeploymentPlan {
        private int recommendedNodes;
        private ResourceConfiguration resourceConfiguration;
        private NetworkConfiguration networkConfiguration;
        private MonitoringConfiguration monitoringConfiguration;
        
        public int getRecommendedNodes() { return recommendedNodes; }
        public ResourceConfiguration getResourceConfiguration() { return resourceConfiguration; }
        public NetworkConfiguration getNetworkConfiguration() { return networkConfiguration; }
        public MonitoringConfiguration getMonitoringConfiguration() { return monitoringConfiguration; }
        
        public void setRecommendedNodes(int recommendedNodes) { this.recommendedNodes = recommendedNodes; }
        public void setResourceConfiguration(ResourceConfiguration resourceConfiguration) { this.resourceConfiguration = resourceConfiguration; }
        public void setNetworkConfiguration(NetworkConfiguration networkConfiguration) { this.networkConfiguration = networkConfiguration; }
        public void setMonitoringConfiguration(MonitoringConfiguration monitoringConfiguration) { this.monitoringConfiguration = monitoringConfiguration; }
    }
    
    public static class ResourceConfiguration {
        private int cpuCores;
        private int memoryGB;
        private long storageGB;
        
        public int getCpuCores() { return cpuCores; }
        public int getMemoryGB() { return memoryGB; }
        public long getStorageGB() { return storageGB; }
        
        public void setCpuCores(int cpuCores) { this.cpuCores = cpuCores; }
        public void setMemoryGB(int memoryGB) { this.memoryGB = memoryGB; }
        public void setStorageGB(long storageGB) { this.storageGB = storageGB; }
    }
    
    public static class NetworkConfiguration {
        private long requiredBandwidthMbps;
        private int maxNetworkLatencyMs;
        private List<Integer> ports;
        
        public long getRequiredBandwidthMbps() { return requiredBandwidthMbps; }
        public int getMaxNetworkLatencyMs() { return maxNetworkLatencyMs; }
        public List<Integer> getPorts() { return ports; }
        
        public void setRequiredBandwidthMbps(long requiredBandwidthMbps) { this.requiredBandwidthMbps = requiredBandwidthMbps; }
        public void setMaxNetworkLatencyMs(int maxNetworkLatencyMs) { this.maxNetworkLatencyMs = maxNetworkLatencyMs; }
        public void setPorts(List<Integer> ports) { this.ports = ports; }
    }
    
    public static class MonitoringConfiguration {
        private List<String> keyMetrics;
        private Map<String, String> alertThresholds;
        
        public List<String> getKeyMetrics() { return keyMetrics; }
        public Map<String, String> getAlertThresholds() { return alertThresholds; }
        
        public void setKeyMetrics(List<String> keyMetrics) { this.keyMetrics = keyMetrics; }
        public void setAlertThresholds(Map<String, String> alertThresholds) { this.alertThresholds = alertThresholds; }
    }
    
    public static class MQComparisonResult {
        private List<MQEvaluationResult> results;
        private MQComparisonMatrix comparisonMatrix;
        private String bestOption;
        private List<String> keyDifferences;
        private MigrationStrategy migrationStrategy;
        
        public MQComparisonResult(List<MQEvaluationResult> results, MQComparisonMatrix comparisonMatrix, 
                                String bestOption, List<String> keyDifferences, MigrationStrategy migrationStrategy) {
            this.results = results;
            this.comparisonMatrix = comparisonMatrix;
            this.bestOption = bestOption;
            this.keyDifferences = keyDifferences;
            this.migrationStrategy = migrationStrategy;
        }
        
        public List<MQEvaluationResult> getResults() { return results; }
        public MQComparisonMatrix getComparisonMatrix() { return comparisonMatrix; }
        public String getBestOption() { return bestOption; }
        public List<String> getKeyDifferences() { return keyDifferences; }
        public MigrationStrategy getMigrationStrategy() { return migrationStrategy; }
    }
    
    public static class MQComparisonMatrix {
        private List<MQEvaluationResult> results;
        
        public MQComparisonMatrix(List<MQEvaluationResult> results) {
            this.results = results;
        }
        
        public List<MQEvaluationResult> getResults() { return results; }
    }
    
    public static class MigrationStrategy {
        private List<String> migrationPhases;
        private List<String> riskControls;
        private int estimatedDays;
        
        public List<String> getMigrationPhases() { return migrationPhases; }
        public List<String> getRiskControls() { return riskControls; }
        public int getEstimatedDays() { return estimatedDays; }
        
        public void setMigrationPhases(List<String> migrationPhases) { this.migrationPhases = migrationPhases; }
        public void setRiskControls(List<String> riskControls) { this.riskControls = riskControls; }
        public void setEstimatedDays(int estimatedDays) { this.estimatedDays = estimatedDays; }
    }
}
```

**关键技术点：**

1. **多维度评估体系**：
   - 性能表现：吞吐量、延迟、并发处理能力
   - 可靠性保障：消息持久化、故障恢复、数据一致性
   - 扩展能力：水平扩展、分区支持
   - 功能完整性：消息路由、死信队列、延迟消息、事务支持
   - 运维便利性：部署复杂度、监控工具、社区支持

2. **量化评分机制**：
   - 权重分配：性能30%、可靠性25%、扩展性20%、功能15%、运维10%
   - 评分标准：1-10分制，基于具体指标量化评估
   - 综合计算：加权平均得出最终评分

3. **风险识别与缓解**：
   - 高风险项：评分低于4分的关键因素
   - 缓解策略：针对每个风险项提供具体解决方案
   - 实施建议：制定详细的风险控制措施

4. **部署方案生成**：
   - 集群规模：基于吞吐量和消息大小计算节点数
   - 资源配置：CPU、内存、存储的详细规格建议
   - 网络配置：带宽、延迟、端口等网络要求
   - 监控配置：关键指标和告警阈值设置

5. **迁移策略制定**：
   - 迁移阶段：环境准备、数据迁移、灰度测试、全量切换
   - 风险控制：双写方案、回滚机制、监控告警
   - 时间估算：基于业务复杂度和数据量评估

**选型决策建议：**

1. **高吞吐量场景**：优先考虑Kafka或RocketMQ
2. **低延迟要求**：推荐Redis Streams或RabbitMQ
3. **复杂路由需求**：选择RabbitMQ
4. **强一致性要求**：考虑RocketMQ或Kafka
5. **简单场景**：ActiveMQ或Redis Streams
6. **云原生环境**：优先支持Kubernetes的方案

### 4.4 架构演进类问题

**8. 单体架构向微服务架构演进的策略是什么？**

**答案要点：**
- **渐进式拆分**：按业务边界逐步拆分
- **数据库拆分**：先拆应用再拆数据库
- **API网关**：统一入口、路由转发
- **服务治理**：注册发现、配置管理、监控
- **团队组织**：康威定律、DevOps文化

**详细解答：**

**架构演进策略对比表：**

| 演进阶段 | 策略 | 技术手段 | 风险控制 | 团队要求 |
|---------|------|----------|----------|----------|
| 准备阶段 | 代码重构、模块化 | 领域建模、边界识别 | 单元测试、集成测试 | 架构师、业务专家 |
| 初期拆分 | 垂直拆分、边缘服务 | API网关、服务注册 | 灰度发布、回滚机制 | DevOps团队 |
| 核心拆分 | 水平拆分、数据分离 | 分布式事务、数据同步 | 监控告警、容错处理 | 全栈团队 |
| 深度优化 | 服务治理、性能调优 | 链路追踪、自动扩缩容 | 混沌工程、故障演练 | SRE团队 |

**Java代码示例：**

```java
/**
 * 单体架构向微服务架构演进应用
 * 演示渐进式拆分、数据库分离、服务治理等策略
 */
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker
public class MigrationStrategyApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(MigrationStrategyApplication.class, args);
    }
    
    // 1. 领域边界识别和服务拆分策略
    @Component
    public static class DomainBoundaryAnalyzer {
        
        @Data
        @AllArgsConstructor
        public static class ServiceBoundary {
            private String serviceName;
            private Set<String> entities;
            private Set<String> operations;
            private int couplingScore;
            private int cohesionScore;
        }
        
        public List<ServiceBoundary> analyzeBoundaries(MonolithApplication monolith) {
            List<ServiceBoundary> boundaries = new ArrayList<>();
            
            // 用户服务边界
            boundaries.add(new ServiceBoundary(
                "user-service",
                Set.of("User", "UserProfile", "UserPreference"),
                Set.of("register", "login", "updateProfile", "getUser"),
                2, // 低耦合
                9  // 高内聚
            ));
            
            // 订单服务边界
            boundaries.add(new ServiceBoundary(
                "order-service",
                Set.of("Order", "OrderItem", "OrderStatus"),
                Set.of("createOrder", "updateOrder", "cancelOrder", "getOrder"),
                3,
                8
            ));
            
            // 商品服务边界
            boundaries.add(new ServiceBoundary(
                "product-service",
                Set.of("Product", "Category", "Inventory"),
                Set.of("addProduct", "updateProduct", "searchProduct", "checkStock"),
                2,
                9
            ));
            
            return boundaries;
        }
        
        public ServiceBoundary findOptimalSplitPoint(List<ServiceBoundary> boundaries) {
            return boundaries.stream()
                .filter(b -> b.getCouplingScore() <= 3 && b.getCohesionScore() >= 7)
                .min(Comparator.comparing(ServiceBoundary::getCouplingScore))
                .orElse(boundaries.get(0));
        }
    }
    
    // 2. 渐进式拆分执行器
    @Service
    public static class ProgressiveSplitExecutor {
        
        @Autowired
        private DomainBoundaryAnalyzer boundaryAnalyzer;
        
        @Data
        @AllArgsConstructor
        public static class SplitPlan {
            private String phase;
            private List<String> services;
            private Map<String, String> migrationSteps;
            private List<String> rollbackSteps;
        }
        
        public List<SplitPlan> createMigrationPlan() {
            List<SplitPlan> plans = new ArrayList<>();
            
            // 阶段1：边缘服务拆分
            plans.add(new SplitPlan(
                "Phase1-EdgeServices",
                List.of("notification-service", "file-service"),
                Map.of(
                    "step1", "创建独立服务项目",
                    "step2", "迁移相关代码",
                    "step3", "配置API网关路由",
                    "step4", "灰度发布验证"
                ),
                List.of("关闭新服务", "恢复单体路由", "回滚数据")
            ));
            
            // 阶段2：核心业务服务拆分
            plans.add(new SplitPlan(
                "Phase2-CoreServices",
                List.of("user-service", "product-service"),
                Map.of(
                    "step1", "数据库读写分离",
                    "step2", "实现数据同步机制",
                    "step3", "拆分服务代码",
                    "step4", "配置分布式事务"
                ),
                List.of("停止数据同步", "合并数据库", "回滚服务")
            ));
            
            // 阶段3：复杂业务服务拆分
            plans.add(new SplitPlan(
                "Phase3-ComplexServices",
                List.of("order-service", "payment-service"),
                Map.of(
                    "step1", "实现Saga模式",
                    "step2", "配置事件总线",
                    "step3", "拆分复杂事务",
                    "step4", "优化服务间通信"
                ),
                List.of("恢复本地事务", "移除事件总线", "合并服务")
            ));
            
            return plans;
        }
        
        @Async
        public CompletableFuture<Boolean> executeSplitPlan(SplitPlan plan) {
            try {
                log.info("开始执行拆分计划: {}", plan.getPhase());
                
                for (Map.Entry<String, String> step : plan.getMigrationSteps().entrySet()) {
                    log.info("执行步骤 {}: {}", step.getKey(), step.getValue());
                    
                    // 模拟执行步骤
                    Thread.sleep(1000);
                    
                    // 健康检查
                    if (!performHealthCheck(plan.getServices())) {
                        log.error("健康检查失败，开始回滚");
                        rollback(plan);
                        return CompletableFuture.completedFuture(false);
                    }
                }
                
                log.info("拆分计划 {} 执行成功", plan.getPhase());
                return CompletableFuture.completedFuture(true);
                
            } catch (Exception e) {
                log.error("拆分计划执行失败", e);
                rollback(plan);
                return CompletableFuture.completedFuture(false);
            }
        }
        
        private boolean performHealthCheck(List<String> services) {
            return services.stream().allMatch(service -> {
                // 模拟健康检查
                return Math.random() > 0.1; // 90%成功率
            });
        }
        
        private void rollback(SplitPlan plan) {
            plan.getRollbackSteps().forEach(step -> {
                log.info("执行回滚步骤: {}", step);
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
    }
    
    // 3. 数据库拆分策略
    @Service
    public static class DatabaseSplitStrategy {
        
        @Data
        @AllArgsConstructor
        public static class DatabaseSplitPlan {
            private String strategy;
            private Map<String, String> tableMapping;
            private List<String> syncRules;
            private String rollbackStrategy;
        }
        
        public DatabaseSplitPlan createSplitPlan(String serviceName) {
            switch (serviceName) {
                case "user-service":
                    return new DatabaseSplitPlan(
                        "垂直拆分",
                        Map.of(
                            "users", "user_db.users",
                            "user_profiles", "user_db.user_profiles",
                            "user_preferences", "user_db.user_preferences"
                        ),
                        List.of(
                            "实时同步用户基础信息",
                            "异步同步用户偏好数据",
                            "定期校验数据一致性"
                        ),
                        "双写模式回滚到单一数据库"
                    );
                    
                case "order-service":
                    return new DatabaseSplitPlan(
                        "水平拆分",
                        Map.of(
                            "orders", "order_db_shard_{user_id % 4}",
                            "order_items", "order_db_shard_{order_id % 4}"
                        ),
                        List.of(
                            "按用户ID分片同步订单数据",
                            "跨分片查询聚合处理",
                            "分布式事务协调"
                        ),
                        "合并分片数据到单一数据库"
                    );
                    
                default:
                    return new DatabaseSplitPlan(
                        "简单拆分",
                        Map.of(),
                        List.of(),
                        "直接回滚"
                    );
            }
        }
        
        @Transactional
        public void executeDatabaseSplit(DatabaseSplitPlan plan) {
            log.info("开始执行数据库拆分: {}", plan.getStrategy());
            
            // 1. 创建新数据库结构
            createNewDatabaseSchema(plan.getTableMapping());
            
            // 2. 设置数据同步
            setupDataSync(plan.getSyncRules());
            
            // 3. 验证数据一致性
            validateDataConsistency(plan.getTableMapping());
            
            log.info("数据库拆分完成");
        }
        
        private void createNewDatabaseSchema(Map<String, String> tableMapping) {
            tableMapping.forEach((oldTable, newTable) -> {
                log.info("创建新表结构: {} -> {}", oldTable, newTable);
                // 实际的DDL执行逻辑
            });
        }
        
        private void setupDataSync(List<String> syncRules) {
            syncRules.forEach(rule -> {
                log.info("设置同步规则: {}", rule);
                // 实际的同步配置逻辑
            });
        }
        
        private void validateDataConsistency(Map<String, String> tableMapping) {
            tableMapping.forEach((oldTable, newTable) -> {
                log.info("验证数据一致性: {} <-> {}", oldTable, newTable);
                // 实际的数据校验逻辑
            });
        }
    }
    
    // 4. API网关配置管理
    @Configuration
    public static class ApiGatewayConfig {
        
        @Bean
        public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
            return builder.routes()
                // 用户服务路由
                .route("user-service", r -> r.path("/api/users/**")
                    .filters(f -> f
                        .circuitBreaker(c -> c.setName("user-service-cb"))
                        .retry(retryConfig -> retryConfig.setRetries(3))
                        .requestRateLimiter(rl -> rl
                            .setRateLimiter(redisRateLimiter())
                            .setKeyResolver(userKeyResolver()))
                    )
                    .uri("lb://user-service"))
                
                // 订单服务路由
                .route("order-service", r -> r.path("/api/orders/**")
                    .filters(f -> f
                        .circuitBreaker(c -> c.setName("order-service-cb"))
                        .retry(retryConfig -> retryConfig.setRetries(2))
                    )
                    .uri("lb://order-service"))
                
                // 商品服务路由
                .route("product-service", r -> r.path("/api/products/**")
                    .filters(f -> f
                        .circuitBreaker(c -> c.setName("product-service-cb"))
                        .addRequestHeader("X-Service-Version", "v2")
                    )
                    .uri("lb://product-service"))
                
                .build();
        }
        
        @Bean
        public RedisRateLimiter redisRateLimiter() {
            return new RedisRateLimiter(100, 200, 1);
        }
        
        @Bean
        public KeyResolver userKeyResolver() {
            return exchange -> Mono.just("user-" + 
                exchange.getRequest().getHeaders().getFirst("X-User-ID"));
        }
    }
    
    // 5. 服务治理和监控
    @Component
    public static class ServiceGovernance {
        
        @Autowired
        private MeterRegistry meterRegistry;
        
        @Data
        @AllArgsConstructor
        public static class ServiceMetrics {
            private String serviceName;
            private double responseTime;
            private double errorRate;
            private int activeConnections;
            private double cpuUsage;
            private double memoryUsage;
        }
        
        @Scheduled(fixedRate = 30000)
        public void collectServiceMetrics() {
            List<String> services = List.of(
                "user-service", "order-service", "product-service"
            );
            
            services.forEach(service -> {
                ServiceMetrics metrics = gatherMetrics(service);
                recordMetrics(metrics);
                
                // 异常检测
                if (metrics.getErrorRate() > 0.05) {
                    triggerAlert(service, "高错误率告警", metrics.getErrorRate());
                }
                
                if (metrics.getResponseTime() > 2000) {
                    triggerAlert(service, "响应时间告警", metrics.getResponseTime());
                }
            });
        }
        
        private ServiceMetrics gatherMetrics(String serviceName) {
            // 模拟指标收集
            return new ServiceMetrics(
                serviceName,
                Math.random() * 1000 + 100, // 响应时间
                Math.random() * 0.1,         // 错误率
                (int)(Math.random() * 100),   // 活跃连接数
                Math.random() * 80 + 10,      // CPU使用率
                Math.random() * 70 + 20       // 内存使用率
            );
        }
        
        private void recordMetrics(ServiceMetrics metrics) {
            Timer.Sample sample = Timer.start(meterRegistry);
            sample.stop(Timer.builder("service.response.time")
                .tag("service", metrics.getServiceName())
                .register(meterRegistry));
            
            Gauge.builder("service.error.rate")
                .tag("service", metrics.getServiceName())
                .register(meterRegistry, metrics, ServiceMetrics::getErrorRate);
            
            Gauge.builder("service.cpu.usage")
                .tag("service", metrics.getServiceName())
                .register(meterRegistry, metrics, ServiceMetrics::getCpuUsage);
        }
        
        private void triggerAlert(String service, String alertType, double value) {
            log.warn("服务告警 - 服务: {}, 类型: {}, 值: {}", service, alertType, value);
            
            // 发送告警通知
            AlertNotification notification = new AlertNotification(
                service, alertType, value, Instant.now()
            );
            
            // 这里可以集成钉钉、邮件、短信等告警渠道
            sendAlert(notification);
        }
        
        @Data
        @AllArgsConstructor
        public static class AlertNotification {
            private String service;
            private String alertType;
            private double value;
            private Instant timestamp;
        }
        
        private void sendAlert(AlertNotification notification) {
            // 实际的告警发送逻辑
            log.info("发送告警通知: {}", notification);
        }
    }
    
    // 6. 团队组织和DevOps支持
    @Service
    public static class TeamOrganizationSupport {
        
        @Data
        @AllArgsConstructor
        public static class TeamStructure {
            private String teamName;
            private List<String> services;
            private List<String> responsibilities;
            private Map<String, String> skillMatrix;
        }
        
        public List<TeamStructure> designTeamStructure() {
            List<TeamStructure> teams = new ArrayList<>();
            
            // 用户域团队
            teams.add(new TeamStructure(
                "用户域团队",
                List.of("user-service", "auth-service", "profile-service"),
                List.of("用户注册登录", "权限管理", "个人信息管理"),
                Map.of(
                    "Java开发", "高级",
                    "Spring Cloud", "中级",
                    "数据库设计", "中级",
                    "前端开发", "初级"
                )
            ));
            
            // 交易域团队
            teams.add(new TeamStructure(
                "交易域团队",
                List.of("order-service", "payment-service", "inventory-service"),
                List.of("订单管理", "支付处理", "库存管理"),
                Map.of(
                    "Java开发", "高级",
                    "分布式事务", "高级",
                    "消息队列", "中级",
                    "性能优化", "中级"
                )
            ));
            
            // 平台团队
            teams.add(new TeamStructure(
                "平台团队",
                List.of("gateway-service", "config-service", "monitor-service"),
                List.of("基础设施", "服务治理", "监控运维"),
                Map.of(
                    "DevOps", "高级",
                    "Kubernetes", "高级",
                    "监控告警", "高级",
                    "CI/CD", "高级"
                )
            ));
            
            return teams;
        }
        
        @Data
        @AllArgsConstructor
        public static class DevOpsPipeline {
            private String serviceName;
            private List<String> buildSteps;
            private List<String> testSteps;
            private List<String> deploySteps;
            private Map<String, String> environmentConfig;
        }
        
        public DevOpsPipeline createDevOpsPipeline(String serviceName) {
            return new DevOpsPipeline(
                serviceName,
                List.of(
                    "代码检出",
                    "依赖安装",
                    "代码编译",
                    "静态代码分析",
                    "安全扫描"
                ),
                List.of(
                    "单元测试",
                    "集成测试",
                    "契约测试",
                    "性能测试",
                    "端到端测试"
                ),
                List.of(
                    "构建Docker镜像",
                    "推送镜像仓库",
                    "更新Kubernetes配置",
                    "滚动部署",
                    "健康检查",
                    "流量切换"
                ),
                Map.of(
                    "dev", "开发环境配置",
                    "test", "测试环境配置",
                    "staging", "预发布环境配置",
                    "prod", "生产环境配置"
                )
            );
        }
    }
}
```

**关键技术要点：**

1. **领域驱动设计**：通过DDD识别服务边界，确保高内聚低耦合
2. **渐进式演进**：分阶段拆分，降低风险，支持快速回滚
3. **数据库演进**：先拆应用后拆数据，保证数据一致性
4. **API网关治理**：统一入口、路由管理、限流熔断
5. **服务监控**：全链路监控、指标收集、异常告警
6. **团队协作**：按康威定律组织团队，建立DevOps文化

**性能优化建议：**

- 使用异步消息传递减少服务间耦合
- 实施缓存策略提升系统响应速度
- 采用事件溯源模式处理复杂业务流程
- 建立完善的监控和告警体系
- 制定详细的灾难恢复计划

**9. 如何设计一个可扩展的系统架构？**

**答案要点：**
- **水平扩展**：无状态设计、负载均衡
- **垂直扩展**：硬件升级、性能优化
- **模块化设计**：松耦合、高内聚
- **异步处理**：消息队列、事件驱动
- **缓存策略**：多级缓存、分布式缓存

**详细解答：**

**可扩展架构设计对比表：**

| 扩展维度 | 策略 | 技术实现 | 适用场景 | 成本考虑 |
|---------|------|----------|----------|----------|
| 水平扩展 | 无状态设计、负载均衡 | 容器化、微服务、CDN | 高并发、分布式场景 | 线性成本增长 |
| 垂直扩展 | 硬件升级、性能优化 | 更强CPU/内存、SSD | 计算密集型应用 | 指数成本增长 |
| 功能扩展 | 模块化、插件化 | 微内核、SPI机制 | 业务快速变化 | 开发成本较高 |
| 数据扩展 | 分库分表、读写分离 | 分片中间件、主从复制 | 海量数据处理 | 运维复杂度高 |
| 地域扩展 | 多地部署、就近访问 | 多活架构、边缘计算 | 全球化业务 | 网络和运维成本 |

**Java代码示例：**

```java
/**
 * 可扩展系统架构设计应用
 * 演示水平扩展、垂直扩展、模块化设计、异步处理等策略
 */
@SpringBootApplication
@EnableAsync
@EnableScheduling
public class ScalableArchitectureApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(ScalableArchitectureApplication.class, args);
    }
    
    // 1. 水平扩展 - 无状态设计和负载均衡
    @RestController
    @RequestMapping("/api/scalable")
    public static class StatelessController {
        
        @Autowired
        private StatelessService statelessService;
        
        @Autowired
        private LoadBalancer loadBalancer;
        
        // 无状态API设计
        @GetMapping("/users/{userId}")
        public ResponseEntity<UserInfo> getUser(@PathVariable String userId,
                                               HttpServletRequest request) {
            // 从请求中获取所有必要信息，不依赖服务器状态
            String requestId = request.getHeader("X-Request-ID");
            String clientVersion = request.getHeader("X-Client-Version");
            
            UserInfo userInfo = statelessService.getUserInfo(userId, requestId);
            
            return ResponseEntity.ok()
                .header("X-Server-Instance", getServerInstanceId())
                .header("X-Response-Time", String.valueOf(System.currentTimeMillis()))
                .body(userInfo);
        }
        
        // 支持水平扩展的批量处理
        @PostMapping("/batch/process")
        public ResponseEntity<BatchResult> processBatch(@RequestBody BatchRequest request) {
            // 将批量请求分发到多个实例
            List<CompletableFuture<ProcessResult>> futures = request.getItems().stream()
                .map(item -> loadBalancer.executeOnOptimalInstance(() -> 
                    statelessService.processItem(item)))
                .collect(Collectors.toList());
            
            // 等待所有任务完成
            List<ProcessResult> results = futures.stream()
                .map(CompletableFuture::join)
                .collect(Collectors.toList());
            
            return ResponseEntity.ok(new BatchResult(results));
        }
        
        private String getServerInstanceId() {
            return InetAddress.getLocalHost().getHostName() + "-" + 
                   ManagementFactory.getRuntimeMXBean().getName();
        }
    }
    
    @Service
    public static class StatelessService {
        
        @Autowired
        private RedisTemplate<String, Object> redisTemplate;
        
        @Autowired
        private DatabaseService databaseService;
        
        // 无状态服务方法
        public UserInfo getUserInfo(String userId, String requestId) {
            // 使用外部存储（Redis/数据库）而非本地状态
            String cacheKey = "user:" + userId;
            
            UserInfo userInfo = (UserInfo) redisTemplate.opsForValue().get(cacheKey);
            if (userInfo == null) {
                userInfo = databaseService.findUserById(userId);
                redisTemplate.opsForValue().set(cacheKey, userInfo, Duration.ofMinutes(30));
            }
            
            // 记录请求日志，便于分布式追踪
            log.info("处理用户请求 - UserId: {}, RequestId: {}, Instance: {}", 
                    userId, requestId, getServerInstanceId());
            
            return userInfo;
        }
        
        public ProcessResult processItem(ProcessItem item) {
            // 无状态处理逻辑
            return new ProcessResult(item.getId(), "processed", Instant.now());
        }
    }
    
    // 2. 智能负载均衡器
    @Component
    public static class LoadBalancer {
        
        private final List<ServerInstance> instances = new CopyOnWriteArrayList<>();
        private final AtomicInteger roundRobinCounter = new AtomicInteger(0);
        
        @Data
        @AllArgsConstructor
        public static class ServerInstance {
            private String instanceId;
            private String host;
            private int port;
            private double cpuUsage;
            private double memoryUsage;
            private int activeConnections;
            private boolean healthy;
        }
        
        @PostConstruct
        public void initializeInstances() {
            // 模拟多个服务实例
            instances.add(new ServerInstance("instance-1", "192.168.1.10", 8080, 0.3, 0.4, 10, true));
            instances.add(new ServerInstance("instance-2", "192.168.1.11", 8080, 0.5, 0.6, 15, true));
            instances.add(new ServerInstance("instance-3", "192.168.1.12", 8080, 0.2, 0.3, 8, true));
        }
        
        // 基于负载的智能路由
        public ServerInstance selectOptimalInstance() {
            List<ServerInstance> healthyInstances = instances.stream()
                .filter(ServerInstance::isHealthy)
                .collect(Collectors.toList());
            
            if (healthyInstances.isEmpty()) {
                throw new RuntimeException("没有可用的健康实例");
            }
            
            // 加权最少连接算法
            return healthyInstances.stream()
                .min(Comparator.comparingDouble(this::calculateInstanceLoad))
                .orElse(healthyInstances.get(0));
        }
        
        private double calculateInstanceLoad(ServerInstance instance) {
            // 综合考虑CPU、内存和连接数
            return instance.getCpuUsage() * 0.4 + 
                   instance.getMemoryUsage() * 0.3 + 
                   (instance.getActiveConnections() / 100.0) * 0.3;
        }
        
        // 异步执行任务到最优实例
        public <T> CompletableFuture<T> executeOnOptimalInstance(Supplier<T> task) {
            ServerInstance instance = selectOptimalInstance();
            
            return CompletableFuture.supplyAsync(() -> {
                try {
                    // 模拟在指定实例上执行任务
                    log.info("在实例 {} 上执行任务", instance.getInstanceId());
                    return task.get();
                } catch (Exception e) {
                    log.error("实例 {} 执行任务失败", instance.getInstanceId(), e);
                    throw new RuntimeException(e);
                }
            });
        }
        
        // 健康检查和实例管理
        @Scheduled(fixedRate = 30000)
        public void performHealthCheck() {
            instances.parallelStream().forEach(instance -> {
                boolean healthy = checkInstanceHealth(instance);
                instance.setHealthy(healthy);
                
                if (!healthy) {
                    log.warn("实例 {} 健康检查失败", instance.getInstanceId());
                }
            });
        }
        
        private boolean checkInstanceHealth(ServerInstance instance) {
            // 模拟健康检查
            return Math.random() > 0.05; // 95%健康率
        }
    }
    
    // 3. 垂直扩展 - 性能优化和资源管理
    @Service
    public static class PerformanceOptimizationService {
        
        @Autowired
        private MeterRegistry meterRegistry;
        
        private final ExecutorService optimizedExecutor;
        
        public PerformanceOptimizationService() {
            // 根据系统资源动态配置线程池
            int coreCount = Runtime.getRuntime().availableProcessors();
            this.optimizedExecutor = new ThreadPoolExecutor(
                coreCount,
                coreCount * 2,
                60L, TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(1000),
                new ThreadFactoryBuilder()
                    .setNameFormat("optimized-pool-%d")
                    .setDaemon(true)
                    .build(),
                new ThreadPoolExecutor.CallerRunsPolicy()
            );
        }
        
        // CPU密集型任务优化
        @Async("optimizedExecutor")
        public CompletableFuture<ComputationResult> performCpuIntensiveTask(ComputationRequest request) {
            Timer.Sample sample = Timer.start(meterRegistry);
            
            try {
                // 使用并行流优化计算
                List<Double> results = request.getData().parallelStream()
                    .mapToDouble(this::complexCalculation)
                    .boxed()
                    .collect(Collectors.toList());
                
                return CompletableFuture.completedFuture(
                    new ComputationResult(request.getId(), results)
                );
                
            } finally {
                sample.stop(Timer.builder("computation.time")
                    .tag("type", "cpu-intensive")
                    .register(meterRegistry));
            }
        }
        
        private double complexCalculation(double input) {
            // 模拟复杂计算
            return Math.sqrt(Math.pow(input, 3) + Math.sin(input) * Math.cos(input));
        }
        
        // 内存优化策略
        @EventListener
        public void handleMemoryPressure(MemoryPressureEvent event) {
            if (event.getUsagePercentage() > 80) {
                log.warn("内存使用率过高: {}%，开始优化", event.getUsagePercentage());
                
                // 触发垃圾回收
                System.gc();
                
                // 清理缓存
                clearNonEssentialCaches();
                
                // 降低线程池大小
                if (optimizedExecutor instanceof ThreadPoolExecutor) {
                    ThreadPoolExecutor tpe = (ThreadPoolExecutor) optimizedExecutor;
                    tpe.setCorePoolSize(Math.max(1, tpe.getCorePoolSize() - 1));
                }
            }
        }
        
        private void clearNonEssentialCaches() {
            // 清理非关键缓存
            log.info("清理非关键缓存以释放内存");
        }
        
        // 资源监控
        @Scheduled(fixedRate = 10000)
        public void monitorSystemResources() {
            MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
            MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
            
            double memoryUsagePercentage = (double) heapUsage.getUsed() / heapUsage.getMax() * 100;
            
            Gauge.builder("system.memory.usage")
                .register(meterRegistry, memoryUsagePercentage, value -> value);
            
            if (memoryUsagePercentage > 80) {
                applicationEventPublisher.publishEvent(
                    new MemoryPressureEvent(memoryUsagePercentage)
                );
            }
        }
    }
    
    // 4. 模块化设计 - 插件化架构
    @Component
    public static class ModularArchitecture {
        
        private final Map<String, PluginInterface> plugins = new ConcurrentHashMap<>();
        
        public interface PluginInterface {
            String getName();
            String getVersion();
            boolean isEnabled();
            Object execute(Map<String, Object> parameters);
            void initialize();
            void destroy();
        }
        
        @PostConstruct
        public void loadPlugins() {
            // 动态加载插件
            loadPlugin(new PaymentPlugin());
            loadPlugin(new NotificationPlugin());
            loadPlugin(new AnalyticsPlugin());
        }
        
        public void loadPlugin(PluginInterface plugin) {
            try {
                plugin.initialize();
                plugins.put(plugin.getName(), plugin);
                log.info("插件加载成功: {} v{}", plugin.getName(), plugin.getVersion());
            } catch (Exception e) {
                log.error("插件加载失败: {}", plugin.getName(), e);
            }
        }
        
        public Object executePlugin(String pluginName, Map<String, Object> parameters) {
            PluginInterface plugin = plugins.get(pluginName);
            if (plugin == null || !plugin.isEnabled()) {
                throw new RuntimeException("插件不可用: " + pluginName);
            }
            
            return plugin.execute(parameters);
        }
        
        // 示例插件实现
        public static class PaymentPlugin implements PluginInterface {
            private boolean enabled = true;
            
            @Override
            public String getName() { return "payment-plugin"; }
            
            @Override
            public String getVersion() { return "1.0.0"; }
            
            @Override
            public boolean isEnabled() { return enabled; }
            
            @Override
            public Object execute(Map<String, Object> parameters) {
                // 支付处理逻辑
                String amount = (String) parameters.get("amount");
                String currency = (String) parameters.get("currency");
                
                return Map.of(
                    "status", "success",
                    "transactionId", UUID.randomUUID().toString(),
                    "amount", amount,
                    "currency", currency
                );
            }
            
            @Override
            public void initialize() {
                log.info("支付插件初始化完成");
            }
            
            @Override
            public void destroy() {
                log.info("支付插件销毁完成");
            }
        }
    }
    
    // 5. 异步处理 - 事件驱动架构
    @Component
    public static class AsyncEventProcessor {
        
        @Autowired
        private ApplicationEventPublisher eventPublisher;
        
        @Autowired
        private RabbitTemplate rabbitTemplate;
        
        // 异步事件处理
        @EventListener
        @Async
        public void handleUserRegistrationEvent(UserRegistrationEvent event) {
            log.info("处理用户注册事件: {}", event.getUserId());
            
            // 发送欢迎邮件
            sendWelcomeEmail(event.getUserId(), event.getEmail());
            
            // 创建用户档案
            createUserProfile(event.getUserId());
            
            // 发送到消息队列进行进一步处理
            rabbitTemplate.convertAndSend("user.registration", event);
        }
        
        @EventListener
        @Async
        public void handleOrderCreatedEvent(OrderCreatedEvent event) {
            log.info("处理订单创建事件: {}", event.getOrderId());
            
            // 库存扣减
            processInventoryDeduction(event.getOrderId(), event.getItems());
            
            // 发送订单确认
            sendOrderConfirmation(event.getUserId(), event.getOrderId());
            
            // 触发物流处理
            triggerLogisticsProcess(event.getOrderId());
        }
        
        // 消息队列异步处理
        @RabbitListener(queues = "user.registration")
        public void processUserRegistration(UserRegistrationEvent event) {
            // 异步处理用户注册后续流程
            log.info("异步处理用户注册: {}", event.getUserId());
            
            // 数据分析
            analyzeUserBehavior(event.getUserId());
            
            // 推荐系统初始化
            initializeRecommendationSystem(event.getUserId());
        }
        
        @RabbitListener(queues = "order.processing")
        public void processOrderAsync(OrderCreatedEvent event) {
            // 异步订单处理
            log.info("异步处理订单: {}", event.getOrderId());
            
            // 风控检查
            performRiskAssessment(event.getOrderId());
            
            // 供应商通知
            notifySuppliers(event.getItems());
        }
        
        // 辅助方法
        private void sendWelcomeEmail(String userId, String email) {
            log.info("发送欢迎邮件到: {}", email);
        }
        
        private void createUserProfile(String userId) {
            log.info("创建用户档案: {}", userId);
        }
        
        private void processInventoryDeduction(String orderId, List<OrderItem> items) {
            log.info("处理订单 {} 的库存扣减", orderId);
        }
        
        private void sendOrderConfirmation(String userId, String orderId) {
            log.info("发送订单确认给用户 {}: {}", userId, orderId);
        }
        
        private void triggerLogisticsProcess(String orderId) {
            log.info("触发订单 {} 的物流处理", orderId);
        }
        
        private void analyzeUserBehavior(String userId) {
            log.info("分析用户 {} 的行为数据", userId);
        }
        
        private void initializeRecommendationSystem(String userId) {
            log.info("为用户 {} 初始化推荐系统", userId);
        }
        
        private void performRiskAssessment(String orderId) {
            log.info("对订单 {} 进行风控检查", orderId);
        }
        
        private void notifySuppliers(List<OrderItem> items) {
            log.info("通知供应商处理 {} 个商品", items.size());
        }
    }
    
    // 6. 多级缓存策略
    @Service
    public static class MultiLevelCacheService {
        
        @Autowired
        private RedisTemplate<String, Object> redisTemplate;
        
        private final Cache<String, Object> localCache;
        
        public MultiLevelCacheService() {
            this.localCache = Caffeine.newBuilder()
                .maximumSize(10000)
                .expireAfterWrite(Duration.ofMinutes(10))
                .recordStats()
                .build();
        }
        
        // 多级缓存获取
        public <T> T get(String key, Class<T> type, Supplier<T> dataLoader) {
            // L1: 本地缓存
            T value = (T) localCache.getIfPresent(key);
            if (value != null) {
                log.debug("L1缓存命中: {}", key);
                return value;
            }
            
            // L2: Redis缓存
            value = (T) redisTemplate.opsForValue().get(key);
            if (value != null) {
                log.debug("L2缓存命中: {}", key);
                localCache.put(key, value);
                return value;
            }
            
            // L3: 数据源
            value = dataLoader.get();
            if (value != null) {
                log.debug("从数据源加载: {}", key);
                
                // 写入各级缓存
                redisTemplate.opsForValue().set(key, value, Duration.ofHours(1));
                localCache.put(key, value);
            }
            
            return value;
        }
        
        // 缓存更新策略
        public void evict(String key) {
            localCache.invalidate(key);
            redisTemplate.delete(key);
            log.info("清除缓存: {}", key);
        }
        
        // 缓存预热
        @PostConstruct
        public void warmUpCache() {
            log.info("开始缓存预热");
            
            // 预加载热点数据
            List<String> hotKeys = getHotDataKeys();
            hotKeys.parallelStream().forEach(key -> {
                try {
                    // 模拟加载热点数据
                    Object data = loadHotData(key);
                    redisTemplate.opsForValue().set(key, data, Duration.ofHours(2));
                    localCache.put(key, data);
                } catch (Exception e) {
                    log.error("预热缓存失败: {}", key, e);
                }
            });
            
            log.info("缓存预热完成");
        }
        
        private List<String> getHotDataKeys() {
            // 返回热点数据键列表
            return List.of("hot:user:1", "hot:product:popular", "hot:config:system");
        }
        
        private Object loadHotData(String key) {
            // 模拟加载热点数据
            return "hot-data-" + key;
        }
        
        // 缓存统计
        @Scheduled(fixedRate = 60000)
        public void reportCacheStats() {
            CacheStats stats = localCache.stats();
            log.info("本地缓存统计 - 命中率: {:.2f}%, 请求数: {}, 命中数: {}, 未命中数: {}",
                stats.hitRate() * 100,
                stats.requestCount(),
                stats.hitCount(),
                stats.missCount());
        }
    }
    
    // 数据模型定义
    @Data
    @AllArgsConstructor
    public static class UserInfo {
        private String userId;
        private String username;
        private String email;
        private Instant createdAt;
    }
    
    @Data
    @AllArgsConstructor
    public static class BatchRequest {
        private List<ProcessItem> items;
    }
    
    @Data
    @AllArgsConstructor
    public static class ProcessItem {
        private String id;
        private String data;
    }
    
    @Data
    @AllArgsConstructor
    public static class ProcessResult {
        private String id;
        private String status;
        private Instant processedAt;
    }
    
    @Data
    @AllArgsConstructor
    public static class BatchResult {
        private List<ProcessResult> results;
    }
    
    // 事件定义
    @Data
    @AllArgsConstructor
    public static class UserRegistrationEvent {
        private String userId;
        private String email;
        private Instant timestamp;
    }
    
    @Data
    @AllArgsConstructor
    public static class OrderCreatedEvent {
        private String orderId;
        private String userId;
        private List<OrderItem> items;
        private Instant timestamp;
    }
    
    @Data
    @AllArgsConstructor
    public static class OrderItem {
        private String productId;
        private int quantity;
        private BigDecimal price;
    }
    
    @Data
    @AllArgsConstructor
    public static class MemoryPressureEvent {
        private double usagePercentage;
    }
}
```

**关键技术要点：**

1. **无状态设计**：服务实例间可互相替换，支持水平扩展
2. **智能负载均衡**：基于实例负载动态路由请求
3. **性能优化**：CPU和内存使用优化，资源监控
4. **模块化架构**：插件化设计，支持功能动态扩展
5. **异步处理**：事件驱动架构，提升系统吞吐量
6. **多级缓存**：本地+分布式缓存，优化数据访问性能

**性能优化建议：**

- 采用容器化部署支持快速扩缩容
- 实施数据库读写分离和分库分表
- 使用CDN加速静态资源访问
- 建立完善的监控和自动扩缩容机制
- 优化数据库查询和索引设计

### 4.5 故障处理类问题

**10. 如何设计一个高可用的系统？**

**答案要点：**
- **冗余设计**：多副本、多机房部署
- **故障隔离**：熔断器、舱壁模式
- **快速恢复**：自动故障转移、健康检查
- **降级策略**：核心功能保障、非核心功能降级
- **监控告警**：实时监控、及时响应

**详细解答：**

**高可用系统设计对比表：**

| 可用性等级 | 年停机时间 | 月停机时间 | 技术要求 | 成本投入 | 适用场景 |
|-----------|-----------|-----------|----------|----------|----------|
| 99% | 3.65天 | 7.2小时 | 基础监控、简单备份 | 低 | 内部系统 |
| 99.9% | 8.76小时 | 43.2分钟 | 负载均衡、故障转移 | 中 | 一般业务系统 |
| 99.99% | 52.56分钟 | 4.32分钟 | 多活架构、自动恢复 | 高 | 重要业务系统 |
| 99.999% | 5.26分钟 | 25.9秒 | 分布式架构、实时监控 | 很高 | 金融、医疗系统 |
| 99.9999% | 31.5秒 | 2.59秒 | 容灾备份、零停机部署 | 极高 | 核心基础设施 |

**Java代码示例：**

```java
/**
 * 高可用系统设计应用
 * 演示冗余设计、故障隔离、快速恢复、降级策略等高可用技术
 */
@SpringBootApplication
@EnableCircuitBreaker
@EnableScheduling
public class HighAvailabilityApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(HighAvailabilityApplication.class, args);
    }
    
    // 1. 冗余设计 - 多副本和多机房部署
    @Service
    public static class RedundantService {
        
        @Autowired
        private List<DataCenter> dataCenters;
        
        @Autowired
        private LoadBalancer loadBalancer;
        
        @Data
        @AllArgsConstructor
        public static class DataCenter {
            private String id;
            private String region;
            private String zone;
            private boolean active;
            private double healthScore;
            private List<ServiceInstance> instances;
        }
        
        @Data
        @AllArgsConstructor
        public static class ServiceInstance {
            private String instanceId;
            private String host;
            private int port;
            private boolean healthy;
            private double responseTime;
            private int activeConnections;
        }
        
        // 多副本数据访问
        public <T> T accessDataWithRedundancy(String dataKey, Class<T> type) {
            // 按优先级尝试不同数据中心
            List<DataCenter> sortedCenters = dataCenters.stream()
                .filter(DataCenter::isActive)
                .sorted(Comparator.comparingDouble(DataCenter::getHealthScore).reversed())
                .collect(Collectors.toList());
            
            for (DataCenter center : sortedCenters) {
                try {
                    T result = accessDataFromCenter(center, dataKey, type);
                    if (result != null) {
                        log.info("从数据中心 {} 成功获取数据: {}", center.getId(), dataKey);
                        return result;
                    }
                } catch (Exception e) {
                    log.warn("数据中心 {} 访问失败，尝试下一个: {}", center.getId(), e.getMessage());
                    // 标记数据中心健康状况
                    updateDataCenterHealth(center.getId(), false);
                }
            }
            
            throw new RuntimeException("所有数据中心都不可用");
        }
        
        private <T> T accessDataFromCenter(DataCenter center, String dataKey, Class<T> type) {
            // 从指定数据中心的健康实例获取数据
            List<ServiceInstance> healthyInstances = center.getInstances().stream()
                .filter(ServiceInstance::isHealthy)
                .sorted(Comparator.comparingDouble(ServiceInstance::getResponseTime))
                .collect(Collectors.toList());
            
            if (healthyInstances.isEmpty()) {
                throw new RuntimeException("数据中心 " + center.getId() + " 没有健康实例");
            }
            
            ServiceInstance instance = healthyInstances.get(0);
            // 模拟数据访问
            return (T) ("data-from-" + center.getId() + "-" + instance.getInstanceId());
        }
        
        private void updateDataCenterHealth(String centerId, boolean healthy) {
            dataCenters.stream()
                .filter(dc -> dc.getId().equals(centerId))
                .findFirst()
                .ifPresent(dc -> {
                    double newScore = healthy ? Math.min(1.0, dc.getHealthScore() + 0.1) 
                                             : Math.max(0.0, dc.getHealthScore() - 0.2);
                    dc.setHealthScore(newScore);
                });
        }
        
        // 跨机房数据同步
        @Scheduled(fixedRate = 30000)
        public void synchronizeDataAcrossCenters() {
            log.info("开始跨机房数据同步");
            
            List<DataCenter> activeCenters = dataCenters.stream()
                .filter(DataCenter::isActive)
                .collect(Collectors.toList());
            
            // 并行同步数据
            activeCenters.parallelStream().forEach(center -> {
                try {
                    syncDataToCenter(center);
                    log.debug("数据同步到 {} 完成", center.getId());
                } catch (Exception e) {
                    log.error("数据同步到 {} 失败", center.getId(), e);
                }
            });
        }
        
        private void syncDataToCenter(DataCenter center) {
            // 模拟数据同步逻辑
            Thread.sleep(100); // 模拟网络延迟
        }
    }
    
    // 2. 故障隔离 - 熔断器和舱壁模式
    @Component
    public static class FaultIsolationService {
        
        private final Map<String, CircuitBreaker> circuitBreakers = new ConcurrentHashMap<>();
        private final Map<String, Semaphore> bulkheads = new ConcurrentHashMap<>();
        
        @PostConstruct
        public void initializeFaultIsolation() {
            // 初始化不同服务的熔断器
            circuitBreakers.put("payment-service", createCircuitBreaker("payment", 5, 10000));
            circuitBreakers.put("user-service", createCircuitBreaker("user", 3, 5000));
            circuitBreakers.put("order-service", createCircuitBreaker("order", 8, 15000));
            
            // 初始化舱壁模式的资源隔离
            bulkheads.put("payment-service", new Semaphore(10)); // 支付服务最多10个并发
            bulkheads.put("user-service", new Semaphore(20));    // 用户服务最多20个并发
            bulkheads.put("order-service", new Semaphore(15));   // 订单服务最多15个并发
        }
        
        private CircuitBreaker createCircuitBreaker(String name, int failureThreshold, long timeoutMs) {
            return CircuitBreaker.ofDefaults(name)
                .toBuilder()
                .failureRateThreshold(50.0f) // 失败率阈值50%
                .waitDurationInOpenState(Duration.ofMillis(timeoutMs))
                .slidingWindowSize(failureThreshold * 2)
                .minimumNumberOfCalls(failureThreshold)
                .build();
        }
        
        // 带熔断器的服务调用
        public <T> T callServiceWithCircuitBreaker(String serviceName, Supplier<T> serviceCall, Supplier<T> fallback) {
            CircuitBreaker circuitBreaker = circuitBreakers.get(serviceName);
            if (circuitBreaker == null) {
                throw new IllegalArgumentException("未知服务: " + serviceName);
            }
            
            Supplier<T> decoratedSupplier = CircuitBreaker.decorateSupplier(circuitBreaker, serviceCall);
            
            try {
                return decoratedSupplier.get();
            } catch (Exception e) {
                log.warn("服务 {} 调用失败，执行降级: {}", serviceName, e.getMessage());
                return fallback.get();
            }
        }
        
        // 带舱壁隔离的服务调用
        public <T> CompletableFuture<T> callServiceWithBulkhead(String serviceName, Supplier<T> serviceCall) {
            Semaphore bulkhead = bulkheads.get(serviceName);
            if (bulkhead == null) {
                throw new IllegalArgumentException("未知服务: " + serviceName);
            }
            
            return CompletableFuture.supplyAsync(() -> {
                try {
                    // 获取许可证，实现资源隔离
                    if (bulkhead.tryAcquire(1000, TimeUnit.MILLISECONDS)) {
                        try {
                            log.debug("获取 {} 服务资源许可，开始执行", serviceName);
                            return serviceCall.get();
                        } finally {
                            bulkhead.release();
                        }
                    } else {
                        throw new RuntimeException("服务 " + serviceName + " 资源繁忙，请稍后重试");
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    throw new RuntimeException("服务调用被中断", e);
                }
            });
        }
        
        // 熔断器状态监控
        @Scheduled(fixedRate = 10000)
        public void monitorCircuitBreakers() {
            circuitBreakers.forEach((serviceName, circuitBreaker) -> {
                CircuitBreaker.State state = circuitBreaker.getState();
                CircuitBreaker.Metrics metrics = circuitBreaker.getMetrics();
                
                log.info("服务 {} 熔断器状态: {}, 失败率: {:.2f}%, 调用数: {}",
                    serviceName, state,
                    metrics.getFailureRate(),
                    metrics.getNumberOfCalls());
                
                if (state == CircuitBreaker.State.OPEN) {
                    log.warn("服务 {} 熔断器已打开，正在降级处理", serviceName);
                }
            });
        }
    }
    
    // 3. 快速恢复 - 自动故障转移和健康检查
    @Service
    public static class AutoRecoveryService {
        
        @Autowired
        private RedundantService redundantService;
        
        private final Map<String, HealthStatus> serviceHealthMap = new ConcurrentHashMap<>();
        
        @Data
        @AllArgsConstructor
        public static class HealthStatus {
            private boolean healthy;
            private Instant lastCheckTime;
            private String errorMessage;
            private int consecutiveFailures;
        }
        
        // 健康检查
        @Scheduled(fixedRate = 5000)
        public void performHealthChecks() {
            List<String> services = List.of("payment-service", "user-service", "order-service", "notification-service");
            
            services.parallelStream().forEach(serviceName -> {
                try {
                    boolean healthy = checkServiceHealth(serviceName);
                    updateHealthStatus(serviceName, healthy, null);
                    
                    if (!healthy) {
                        triggerFailover(serviceName);
                    }
                } catch (Exception e) {
                    updateHealthStatus(serviceName, false, e.getMessage());
                    log.error("健康检查失败: {}", serviceName, e);
                }
            });
        }
        
        private boolean checkServiceHealth(String serviceName) {
            // 模拟健康检查
            try {
                // 发送健康检查请求
                Thread.sleep(100); // 模拟网络延迟
                
                // 模拟随机故障
                if (Math.random() < 0.05) { // 5%故障率
                    throw new RuntimeException("服务不可达");
                }
                
                return true;
            } catch (Exception e) {
                return false;
            }
        }
        
        private void updateHealthStatus(String serviceName, boolean healthy, String errorMessage) {
            serviceHealthMap.compute(serviceName, (key, oldStatus) -> {
                if (oldStatus == null) {
                    return new HealthStatus(healthy, Instant.now(), errorMessage, healthy ? 0 : 1);
                }
                
                int consecutiveFailures = healthy ? 0 : oldStatus.getConsecutiveFailures() + 1;
                return new HealthStatus(healthy, Instant.now(), errorMessage, consecutiveFailures);
            });
        }
        
        // 自动故障转移
        private void triggerFailover(String serviceName) {
            HealthStatus status = serviceHealthMap.get(serviceName);
            if (status != null && status.getConsecutiveFailures() >= 3) {
                log.warn("服务 {} 连续失败 {} 次，触发故障转移", serviceName, status.getConsecutiveFailures());
                
                // 执行故障转移逻辑
                performFailover(serviceName);
                
                // 发送告警
                sendFailoverAlert(serviceName, status);
            }
        }
        
        private void performFailover(String serviceName) {
            try {
                // 1. 将流量切换到备用实例
                switchTrafficToBackup(serviceName);
                
                // 2. 重启故障实例
                restartFailedInstance(serviceName);
                
                // 3. 更新负载均衡配置
                updateLoadBalancerConfig(serviceName);
                
                log.info("服务 {} 故障转移完成", serviceName);
                
            } catch (Exception e) {
                log.error("服务 {} 故障转移失败", serviceName, e);
            }
        }
        
        private void switchTrafficToBackup(String serviceName) {
            log.info("将 {} 流量切换到备用实例", serviceName);
        }
        
        private void restartFailedInstance(String serviceName) {
            log.info("重启 {} 故障实例", serviceName);
        }
        
        private void updateLoadBalancerConfig(String serviceName) {
            log.info("更新 {} 负载均衡配置", serviceName);
        }
        
        private void sendFailoverAlert(String serviceName, HealthStatus status) {
            String message = String.format("服务故障转移告警 - 服务: %s, 连续失败: %d次, 错误: %s",
                serviceName, status.getConsecutiveFailures(), status.getErrorMessage());
            
            log.error(message);
            // 发送到告警系统
        }
        
        // 自动恢复检测
        @Scheduled(fixedRate = 30000)
        public void detectAutoRecovery() {
            serviceHealthMap.forEach((serviceName, status) -> {
                if (!status.isHealthy() && status.getConsecutiveFailures() > 0) {
                    // 尝试恢复检测
                    if (checkServiceHealth(serviceName)) {
                        log.info("服务 {} 已自动恢复", serviceName);
                        updateHealthStatus(serviceName, true, null);
                        
                        // 恢复流量
                        restoreTraffic(serviceName);
                    }
                }
            });
        }
        
        private void restoreTraffic(String serviceName) {
            log.info("恢复服务 {} 的流量", serviceName);
        }
    }
    
    // 4. 降级策略 - 核心功能保障
    @Service
    public static class DegradationService {
        
        @Autowired
        private FaultIsolationService faultIsolationService;
        
        private final AtomicBoolean systemOverloaded = new AtomicBoolean(false);
        private final Map<String, Integer> featurePriorities = new ConcurrentHashMap<>();
        
        @PostConstruct
        public void initializePriorities() {
            // 定义功能优先级（数字越小优先级越高）
            featurePriorities.put("user-login", 1);           // 核心功能
            featurePriorities.put("order-payment", 1);        // 核心功能
            featurePriorities.put("product-search", 2);       // 重要功能
            featurePriorities.put("user-profile", 3);         // 一般功能
            featurePriorities.put("recommendation", 4);       // 非核心功能
            featurePriorities.put("analytics", 5);            // 非核心功能
        }
        
        // 智能降级决策
        public <T> T executeWithDegradation(String featureName, Supplier<T> primaryLogic, Supplier<T> degradedLogic) {
            // 检查系统负载
            if (shouldDegrade(featureName)) {
                log.info("功能 {} 执行降级逻辑", featureName);
                return degradedLogic.get();
            }
            
            // 尝试执行主逻辑
            try {
                return faultIsolationService.callServiceWithCircuitBreaker(
                    featureName, primaryLogic, degradedLogic
                );
            } catch (Exception e) {
                log.warn("功能 {} 主逻辑执行失败，降级处理: {}", featureName, e.getMessage());
                return degradedLogic.get();
            }
        }
        
        private boolean shouldDegrade(String featureName) {
            // 系统过载时降级非核心功能
            if (systemOverloaded.get()) {
                int priority = featurePriorities.getOrDefault(featureName, 5);
                return priority > 2; // 优先级低于2的功能降级
            }
            
            // 检查特定功能的资源使用情况
            return checkFeatureResourceUsage(featureName);
        }
        
        private boolean checkFeatureResourceUsage(String featureName) {
            // 模拟资源使用检查
            double cpuUsage = getCurrentCpuUsage();
            double memoryUsage = getCurrentMemoryUsage();
            
            // 资源使用率过高时降级低优先级功能
            if (cpuUsage > 80 || memoryUsage > 85) {
                int priority = featurePriorities.getOrDefault(featureName, 5);
                return priority > 3;
            }
            
            return false;
        }
        
        // 系统负载监控
        @Scheduled(fixedRate = 5000)
        public void monitorSystemLoad() {
            double cpuUsage = getCurrentCpuUsage();
            double memoryUsage = getCurrentMemoryUsage();
            int activeConnections = getCurrentActiveConnections();
            
            // 判断系统是否过载
            boolean overloaded = cpuUsage > 85 || memoryUsage > 90 || activeConnections > 1000;
            
            if (overloaded != systemOverloaded.get()) {
                systemOverloaded.set(overloaded);
                log.warn("系统负载状态变更: {} - CPU: {:.1f}%, Memory: {:.1f}%, Connections: {}",
                    overloaded ? "过载" : "正常", cpuUsage, memoryUsage, activeConnections);
                
                if (overloaded) {
                    triggerEmergencyDegradation();
                } else {
                    restoreNormalOperation();
                }
            }
        }
        
        private void triggerEmergencyDegradation() {
            log.warn("触发紧急降级模式");
            // 关闭非核心功能
            // 限制并发请求
            // 启用缓存模式
        }
        
        private void restoreNormalOperation() {
            log.info("恢复正常运行模式");
            // 恢复所有功能
        }
        
        private double getCurrentCpuUsage() {
            // 获取CPU使用率
            return Math.random() * 100;
        }
        
        private double getCurrentMemoryUsage() {
            // 获取内存使用率
            Runtime runtime = Runtime.getRuntime();
            long totalMemory = runtime.totalMemory();
            long freeMemory = runtime.freeMemory();
            return ((double) (totalMemory - freeMemory) / totalMemory) * 100;
        }
        
        private int getCurrentActiveConnections() {
            // 获取当前活跃连接数
            return (int) (Math.random() * 1200);
        }
    }
    
    // 5. 监控告警 - 实时监控和及时响应
    @Component
    public static class MonitoringAndAlertingService {
        
        @Autowired
        private MeterRegistry meterRegistry;
        
        private final Map<String, AlertRule> alertRules = new ConcurrentHashMap<>();
        private final Queue<AlertEvent> alertQueue = new ConcurrentLinkedQueue<>();
        
        @Data
        @AllArgsConstructor
        public static class AlertRule {
            private String metricName;
            private double threshold;
            private String operator; // >, <, >=, <=, ==
            private Duration duration;
            private AlertLevel level;
            private String description;
        }
        
        public enum AlertLevel {
            INFO, WARNING, ERROR, CRITICAL
        }
        
        @Data
        @AllArgsConstructor
        public static class AlertEvent {
            private String ruleName;
            private AlertLevel level;
            private String message;
            private double currentValue;
            private double threshold;
            private Instant timestamp;
        }
        
        @PostConstruct
        public void initializeAlertRules() {
            // 定义告警规则
            alertRules.put("high-cpu", new AlertRule(
                "system.cpu.usage", 80.0, ">", Duration.ofMinutes(2),
                AlertLevel.WARNING, "CPU使用率过高"
            ));
            
            alertRules.put("critical-cpu", new AlertRule(
                "system.cpu.usage", 95.0, ">", Duration.ofSeconds(30),
                AlertLevel.CRITICAL, "CPU使用率严重过高"
            ));
            
            alertRules.put("high-memory", new AlertRule(
                "system.memory.usage", 85.0, ">", Duration.ofMinutes(3),
                AlertLevel.WARNING, "内存使用率过高"
            ));
            
            alertRules.put("high-error-rate", new AlertRule(
                "http.requests.error.rate", 5.0, ">", Duration.ofMinutes(1),
                AlertLevel.ERROR, "HTTP错误率过高"
            ));
            
            alertRules.put("slow-response", new AlertRule(
                "http.requests.duration.avg", 2000.0, ">", Duration.ofMinutes(2),
                AlertLevel.WARNING, "响应时间过慢"
            ));
        }
        
        // 实时监控指标收集
        @Scheduled(fixedRate = 10000)
        public void collectMetrics() {
            // 收集系统指标
            double cpuUsage = getCurrentCpuUsage();
            double memoryUsage = getCurrentMemoryUsage();
            double diskUsage = getCurrentDiskUsage();
            
            // 注册指标
            Gauge.builder("system.cpu.usage")
                .register(meterRegistry, cpuUsage, value -> value);
            
            Gauge.builder("system.memory.usage")
                .register(meterRegistry, memoryUsage, value -> value);
            
            Gauge.builder("system.disk.usage")
                .register(meterRegistry, diskUsage, value -> value);
            
            // 收集应用指标
            collectApplicationMetrics();
            
            // 检查告警规则
            checkAlertRules();
        }
        
        private void collectApplicationMetrics() {
            // 收集HTTP请求指标
            double errorRate = calculateErrorRate();
            double avgResponseTime = calculateAvgResponseTime();
            int activeConnections = getCurrentActiveConnections();
            
            Gauge.builder("http.requests.error.rate")
                .register(meterRegistry, errorRate, value -> value);
            
            Gauge.builder("http.requests.duration.avg")
                .register(meterRegistry, avgResponseTime, value -> value);
            
            Gauge.builder("http.connections.active")
                .register(meterRegistry, activeConnections, value -> value);
        }
        
        private void checkAlertRules() {
            alertRules.forEach((ruleName, rule) -> {
                try {
                    Gauge gauge = meterRegistry.find(rule.getMetricName()).gauge();
                    if (gauge != null) {
                        double currentValue = gauge.value();
                        
                        if (shouldTriggerAlert(currentValue, rule)) {
                            triggerAlert(ruleName, rule, currentValue);
                        }
                    }
                } catch (Exception e) {
                    log.error("检查告警规则失败: {}", ruleName, e);
                }
            });
        }
        
        private boolean shouldTriggerAlert(double currentValue, AlertRule rule) {
            switch (rule.getOperator()) {
                case ">":
                    return currentValue > rule.getThreshold();
                case "<":
                    return currentValue < rule.getThreshold();
                case ">=":
                    return currentValue >= rule.getThreshold();
                case "<=":
                    return currentValue <= rule.getThreshold();
                case "==":
                    return Math.abs(currentValue - rule.getThreshold()) < 0.001;
                default:
                    return false;
            }
        }
        
        private void triggerAlert(String ruleName, AlertRule rule, double currentValue) {
            AlertEvent event = new AlertEvent(
                ruleName, rule.getLevel(),
                String.format("%s: 当前值 %.2f %s 阈值 %.2f",
                    rule.getDescription(), currentValue, rule.getOperator(), rule.getThreshold()),
                currentValue, rule.getThreshold(), Instant.now()
            );
            
            alertQueue.offer(event);
            
            // 根据告警级别采取不同行动
            handleAlert(event);
        }
        
        private void handleAlert(AlertEvent event) {
            switch (event.getLevel()) {
                case INFO:
                    log.info("信息告警: {}", event.getMessage());
                    break;
                case WARNING:
                    log.warn("警告告警: {}", event.getMessage());
                    sendNotification(event);
                    break;
                case ERROR:
                    log.error("错误告警: {}", event.getMessage());
                    sendNotification(event);
                    triggerAutoRemediation(event);
                    break;
                case CRITICAL:
                    log.error("严重告警: {}", event.getMessage());
                    sendUrgentNotification(event);
                    triggerEmergencyResponse(event);
                    break;
            }
        }
        
        private void sendNotification(AlertEvent event) {
            // 发送邮件、短信、钉钉等通知
            log.info("发送告警通知: {}", event.getMessage());
        }
        
        private void sendUrgentNotification(AlertEvent event) {
            // 发送紧急通知（电话、紧急邮件等）
            log.error("发送紧急告警通知: {}", event.getMessage());
        }
        
        private void triggerAutoRemediation(AlertEvent event) {
            // 触发自动修复措施
            log.info("触发自动修复: {}", event.getRuleName());
        }
        
        private void triggerEmergencyResponse(AlertEvent event) {
            // 触发紧急响应流程
            log.error("触发紧急响应: {}", event.getRuleName());
        }
        
        // 告警统计和报告
        @Scheduled(fixedRate = 300000) // 每5分钟
        public void generateAlertReport() {
            if (alertQueue.isEmpty()) {
                return;
            }
            
            Map<AlertLevel, Long> alertCounts = alertQueue.stream()
                .collect(Collectors.groupingBy(AlertEvent::getLevel, Collectors.counting()));
            
            log.info("告警统计报告: {}", alertCounts);
            
            // 清理旧告警
            Instant cutoff = Instant.now().minus(Duration.ofHours(1));
            alertQueue.removeIf(event -> event.getTimestamp().isBefore(cutoff));
        }
        
        // 辅助方法
        private double getCurrentCpuUsage() {
            return Math.random() * 100;
        }
        
        private double getCurrentMemoryUsage() {
            Runtime runtime = Runtime.getRuntime();
            long totalMemory = runtime.totalMemory();
            long freeMemory = runtime.freeMemory();
            return ((double) (totalMemory - freeMemory) / totalMemory) * 100;
        }
        
        private double getCurrentDiskUsage() {
            return Math.random() * 100;
        }
        
        private double calculateErrorRate() {
            return Math.random() * 10; // 0-10%错误率
        }
        
        private double calculateAvgResponseTime() {
            return 500 + Math.random() * 2000; // 500-2500ms响应时间
        }
        
        private int getCurrentActiveConnections() {
            return (int) (Math.random() * 1200);
        }
    }
    
    // 配置类
    @Configuration
    public static class HighAvailabilityConfig {
        
        @Bean
        public List<RedundantService.DataCenter> dataCenters() {
            return List.of(
                new RedundantService.DataCenter("dc-east-1", "us-east", "zone-a", true, 0.95,
                    List.of(
                        new RedundantService.ServiceInstance("inst-1", "10.0.1.10", 8080, true, 150.0, 10),
                        new RedundantService.ServiceInstance("inst-2", "10.0.1.11", 8080, true, 200.0, 15)
                    )),
                new RedundantService.DataCenter("dc-west-1", "us-west", "zone-b", true, 0.90,
                    List.of(
                        new RedundantService.ServiceInstance("inst-3", "10.0.2.10", 8080, true, 180.0, 12),
                        new RedundantService.ServiceInstance("inst-4", "10.0.2.11", 8080, false, 0.0, 0)
                    )),
                new RedundantService.DataCenter("dc-backup", "backup", "zone-c", false, 0.80,
                    List.of(
                        new RedundantService.ServiceInstance("inst-5", "10.0.3.10", 8080, true, 300.0, 5)
                    ))
            );
        }
        
        @Bean
        public MeterRegistry meterRegistry() {
            return new SimpleMeterRegistry();
        }
    }
}
```

**关键技术要点：**

1. **冗余设计**：多数据中心部署，自动故障转移
2. **故障隔离**：熔断器模式，舱壁模式资源隔离
3. **快速恢复**：健康检查，自动故障检测和恢复
4. **降级策略**：基于优先级的智能降级
5. **监控告警**：实时指标收集，多级告警机制

**高可用设计原则：**

- **无单点故障**：所有组件都有备份
- **快速故障检测**：秒级健康检查
- **自动故障恢复**：无人工干预的自愈能力
- **优雅降级**：保证核心功能可用
- **全面监控**：覆盖所有关键指标和业务流程

## 5. 学习建议

### 5.1 学习路径

1. **理论基础**
   - 深入理解架构设计原则
   - 学习经典的架构模式
   - 掌握系统设计方法论

2. **实践项目**
   - 参与大型项目的架构设计
   - 从单体到微服务的重构实践
   - 性能优化和故障处理经验

3. **技术广度**
   - 了解各种技术栈的优缺点
   - 关注新技术的发展趋势
   - 培养技术选型能力

### 5.2 实践项目建议

1. **电商系统架构设计**
   - 用户服务、商品服务、订单服务
   - 支付系统、库存系统、推荐系统
   - 秒杀系统、搜索系统

2. **社交平台架构**
   - 用户关系、动态发布、消息推送
   - 实时聊天、内容审核、数据分析

3. **金融系统架构**
   - 账户系统、交易系统、风控系统
   - 数据一致性、安全性要求

### 5.3 推荐资源

**书籍推荐：**
- 《软件架构师的12项修炼》
- 《微服务设计》
- 《高性能MySQL》
- 《Redis设计与实现》
- 《分布式系统概念与设计》

**在线资源：**
- 极客时间架构课程
- InfoQ架构文章
- 各大公司技术博客
- GitHub开源项目

### 5.4 学习时间规划

**第1-2个月：架构设计基础**
- 学习架构设计原则和模式
- 掌握技术选型方法
- 完成小型项目架构设计

**第3-4个月：性能优化实践**
- JVM调优实战
- 数据库优化实践
- 缓存策略应用

**第5-6个月：综合项目实战**
- 设计并实现完整的分布式系统
- 处理高并发、高可用场景
- 总结架构设计经验

## 6. 总结

第五阶段是Java架构师学习的最高层次，需要具备：

1. **系统性思维**：能够从全局角度设计系统架构
2. **技术判断力**：具备准确的技术选型能力
3. **性能优化**：深入理解各层次的性能优化方法
4. **实战经验**：通过大量实践积累架构设计经验
5. **持续学习**：保持对新技术的敏感度和学习能力

通过系统学习和实践，你将具备独立设计和实施大型分布式系统的能力，成为真正的Java架构师。

## 3. 性能优化

### 3.1 JVM性能调优

```java
import java.lang.management.*;
import java.util.*;
import java.util.concurrent.*;

/**
 * JVM性能监控和调优工具
 */
public class JVMPerformanceOptimization {
    
    // ==================== JVM监控 ====================
    
    /**
     * JVM性能监控器
     */
    public static class JVMMonitor {
        private final MemoryMXBean memoryBean;
        private final List<GarbageCollectorMXBean> gcBeans;
        private final ThreadMXBean threadBean;
        private final RuntimeMXBean runtimeBean;
        
        public JVMMonitor() {
            this.memoryBean = ManagementFactory.getMemoryMXBean();
            this.gcBeans = ManagementFactory.getGarbageCollectorMXBeans();
            this.threadBean = ManagementFactory.getThreadMXBean();
            this.runtimeBean = ManagementFactory.getRuntimeMXBean();
        }
        
        /**
         * 获取内存使用情况
         */
        public MemoryInfo getMemoryInfo() {
            MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
            MemoryUsage nonHeapUsage = memoryBean.getNonHeapMemoryUsage();
            
            return new MemoryInfo(
                heapUsage.getUsed(),
                heapUsage.getMax(),
                nonHeapUsage.getUsed(),
                nonHeapUsage.getMax()
            );
        }
        
        /**
         * 获取GC信息
         */
        public List<GCInfo> getGCInfo() {
            List<GCInfo> gcInfos = new ArrayList<>();
            
            for (GarbageCollectorMXBean gcBean : gcBeans) {
                gcInfos.add(new GCInfo(
                    gcBean.getName(),
                    gcBean.getCollectionCount(),
                    gcBean.getCollectionTime()
                ));
            }
            
            return gcInfos;
        }
        
        /**
         * 获取线程信息
         */
        public ThreadInfo getThreadInfo() {
            return new ThreadInfo(
                threadBean.getThreadCount(),
                threadBean.getDaemonThreadCount(),
                threadBean.getPeakThreadCount(),
                threadBean.getTotalStartedThreadCount()
            );
        }
        
        /**
         * 生成性能报告
         */
        public String generatePerformanceReport() {
            StringBuilder report = new StringBuilder();
            
            // JVM基本信息
            report.append("=== JVM性能报告 ===\n");
            report.append("JVM运行时间: ").append(runtimeBean.getUptime()).append("ms\n");
            report.append("JVM版本: ").append(runtimeBean.getVmVersion()).append("\n\n");
            
            // 内存信息
            MemoryInfo memInfo = getMemoryInfo();
            report.append("=== 内存使用情况 ===\n");
            report.append("堆内存使用: ").append(formatBytes(memInfo.heapUsed))
                  .append(" / ").append(formatBytes(memInfo.heapMax))
                  .append(" (").append(String.format("%.2f%%", 
                      (double) memInfo.heapUsed / memInfo.heapMax * 100)).append(")\n");
            report.append("非堆内存使用: ").append(formatBytes(memInfo.nonHeapUsed))
                  .append(" / ").append(formatBytes(memInfo.nonHeapMax)).append("\n\n");
            
            // GC信息
            report.append("=== 垃圾回收情况 ===\n");
            for (GCInfo gcInfo : getGCInfo()) {
                report.append(gcInfo.name).append(": ")
                      .append("次数=").append(gcInfo.collectionCount)
                      .append(", 时间=").append(gcInfo.collectionTime).append("ms\n");
            }
            report.append("\n");
            
            // 线程信息
            ThreadInfo threadInfo = getThreadInfo();
            report.append("=== 线程情况 ===\n");
            report.append("当前线程数: ").append(threadInfo.currentThreadCount).append("\n");
            report.append("守护线程数: ").append(threadInfo.daemonThreadCount).append("\n");
            report.append("峰值线程数: ").append(threadInfo.peakThreadCount).append("\n");
            report.append("总启动线程数: ").append(threadInfo.totalStartedThreadCount).append("\n");
            
            return report.toString();
        }
        
        private String formatBytes(long bytes) {
            if (bytes < 1024) return bytes + " B";
            if (bytes < 1024 * 1024) return String.format("%.2f KB", bytes / 1024.0);
            if (bytes < 1024 * 1024 * 1024) return String.format("%.2f MB", bytes / (1024.0 * 1024));
            return String.format("%.2f GB", bytes / (1024.0 * 1024 * 1024));
        }
    }
    
    // ==================== 数据类 ====================
    
    public static class MemoryInfo {
        public final long heapUsed;
        public final long heapMax;
        public final long nonHeapUsed;
        public final long nonHeapMax;
        
        public MemoryInfo(long heapUsed, long heapMax, long nonHeapUsed, long nonHeapMax) {
            this.heapUsed = heapUsed;
            this.heapMax = heapMax;
            this.nonHeapUsed = nonHeapUsed;
            this.nonHeapMax = nonHeapMax;
        }
    }
    
    public static class GCInfo {
        public final String name;
        public final long collectionCount;
        public final long collectionTime;
        
        public GCInfo(String name, long collectionCount, long collectionTime) {
            this.name = name;
            this.collectionCount = collectionCount;
            this.collectionTime = collectionTime;
        }
    }
    
    public static class ThreadInfo {
        public final int currentThreadCount;
        public final int daemonThreadCount;
        public final int peakThreadCount;
        public final long totalStartedThreadCount;
        
        public ThreadInfo(int currentThreadCount, int daemonThreadCount, 
                         int peakThreadCount, long totalStartedThreadCount) {
            this.currentThreadCount = currentThreadCount;
            this.daemonThreadCount = daemonThreadCount;
            this.peakThreadCount = peakThreadCount;
            this.totalStartedThreadCount = totalStartedThreadCount;
        }
    }
    
    // ==================== JVM调优建议 ====================
    
    /**
     * JVM调优建议生成器
     */
    public static class JVMTuningAdvisor {
        
        /**
         * 分析并生成调优建议
         */
        public List<String> generateTuningAdvice(JVMMonitor monitor) {
            List<String> advice = new ArrayList<>();
            
            MemoryInfo memInfo = monitor.getMemoryInfo();
            List<GCInfo> gcInfos = monitor.getGCInfo();
            ThreadInfo threadInfo = monitor.getThreadInfo();
            
            // 内存使用分析
            double heapUsagePercent = (double) memInfo.heapUsed / memInfo.heapMax * 100;
            if (heapUsagePercent > 80) {
                advice.add("堆内存使用率过高(" + String.format("%.2f%%", heapUsagePercent) + 
                          ")，建议增加堆内存大小: -Xmx");
            }
            
            // GC分析
            long totalGCTime = gcInfos.stream().mapToLong(gc -> gc.collectionTime).sum();
            long totalGCCount = gcInfos.stream().mapToLong(gc -> gc.collectionCount).sum();
            
            if (totalGCTime > 5000) { // 5秒
                advice.add("GC总时间过长(" + totalGCTime + "ms)，建议优化GC参数或增加内存");
            }
            
            if (totalGCCount > 1000) {
                advice.add("GC次数过多(" + totalGCCount + "次)，建议检查内存泄漏或调整新生代大小");
            }
            
            // 线程分析
            if (threadInfo.currentThreadCount > 500) {
                advice.add("线程数过多(" + threadInfo.currentThreadCount + 
                          ")，建议使用线程池管理线程");
            }
            
            // 通用建议
            advice.add("建议的JVM参数配置:");
            advice.add("-Xms" + formatMemorySize(memInfo.heapMax / 2) + " -Xmx" + 
                      formatMemorySize(memInfo.heapMax));
            advice.add("-XX:+UseG1GC -XX:MaxGCPauseMillis=200");
            advice.add("-XX:+PrintGCDetails -XX:+PrintGCTimeStamps");
            advice.add("-XX:+HeapDumpOnOutOfMemoryError");
            
            return advice;
        }
        
        private String formatMemorySize(long bytes) {
            if (bytes >= 1024 * 1024 * 1024) {
                return (bytes / (1024 * 1024 * 1024)) + "g";
            } else if (bytes >= 1024 * 1024) {
                return (bytes / (1024 * 1024)) + "m";
            } else {
                return (bytes / 1024) + "k";
            }
        }
    }
    
    // ==================== 性能测试工具 ====================
    
    /**
     * 简单的性能测试工具
     */
    public static class PerformanceTester {
        
        /**
         * 测试方法执行时间
         */
        public static <T> TestResult<T> timeMethod(String methodName, Callable<T> method) {
            long startTime = System.nanoTime();
            long startMemory = getUsedMemory();
            
            T result = null;
            Exception exception = null;
            
            try {
                result = method.call();
            } catch (Exception e) {
                exception = e;
            }
            
            long endTime = System.nanoTime();
            long endMemory = getUsedMemory();
            
            return new TestResult<>(
                methodName,
                (endTime - startTime) / 1_000_000, // 转换为毫秒
                endMemory - startMemory,
                result,
                exception
            );
        }
        
        /**
         * 批量性能测试
         */
        public static void batchTest(String testName, Runnable test, int iterations) {
            System.out.println("开始批量测试: " + testName);
            
            long totalTime = 0;
            long minTime = Long.MAX_VALUE;
            long maxTime = Long.MIN_VALUE;
            
            for (int i = 0; i < iterations; i++) {
                long startTime = System.nanoTime();
                test.run();
                long endTime = System.nanoTime();
                
                long duration = (endTime - startTime) / 1_000_000; // 毫秒
                totalTime += duration;
                minTime = Math.min(minTime, duration);
                maxTime = Math.max(maxTime, duration);
            }
            
            System.out.println("测试结果:");
            System.out.println("  总次数: " + iterations);
            System.out.println("  总时间: " + totalTime + "ms");
            System.out.println("  平均时间: " + (totalTime / iterations) + "ms");
            System.out.println("  最短时间: " + minTime + "ms");
            System.out.println("  最长时间: " + maxTime + "ms");
        }
        
        private static long getUsedMemory() {
            Runtime runtime = Runtime.getRuntime();
            return runtime.totalMemory() - runtime.freeMemory();
        }
    }
    
    public static class TestResult<T> {
        public final String methodName;
        public final long executionTimeMs;
        public final long memoryUsedBytes;
        public final T result;
        public final Exception exception;
        
        public TestResult(String methodName, long executionTimeMs, long memoryUsedBytes, 
                         T result, Exception exception) {
            this.methodName = methodName;
            this.executionTimeMs = executionTimeMs;
            this.memoryUsedBytes = memoryUsedBytes;
            this.result = result;
            this.exception = exception;
        }
        
        @Override
        public String toString() {
            return String.format("TestResult{method='%s', time=%dms, memory=%d bytes, success=%s}",
                methodName, executionTimeMs, memoryUsedBytes, exception == null);
        }
    }
}
```

### 3.2 数据库性能优化

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;
import java.util.*;
import java.util.concurrent.CompletableFuture;

/**
 * 数据库性能优化工具和策略
 */
@Service
public class DatabasePerformanceOptimization {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    // ==================== 慢查询分析 ====================
    
    /**
     * 慢查询分析器
     */
    public static class SlowQueryAnalyzer {
        
        /**
         * 分析慢查询日志
         */
        public List<SlowQuery> analyzeSlowQueries(String logContent) {
            List<SlowQuery> slowQueries = new ArrayList<>();
            
            // 解析慢查询日志的简化实现
            String[] lines = logContent.split("\n");
            SlowQuery currentQuery = null;
            
            for (String line : lines) {
                if (line.contains("Query_time:")) {
                    // 解析查询时间
                    String[] parts = line.split("\\s+");
                    double queryTime = Double.parseDouble(parts[2]);
                    long lockTime = (long) (Double.parseDouble(parts[4]) * 1000);
                    int rowsSent = Integer.parseInt(parts[6]);
                    int rowsExamined = Integer.parseInt(parts[8]);
                    
                    currentQuery = new SlowQuery(queryTime, lockTime, rowsSent, rowsExamined);
                } else if (line.startsWith("use ") && currentQuery != null) {
                    currentQuery.database = line.substring(4).replace(";", "");
                } else if (line.startsWith("SET timestamp") && currentQuery != null) {
                    // 解析时间戳
                    String timestamp = line.substring(line.indexOf("=") + 1).replace(";", "");
                    currentQuery.timestamp = Long.parseLong(timestamp);
                } else if (!line.startsWith("#") && !line.trim().isEmpty() && currentQuery != null) {
                    // SQL语句
                    currentQuery.sql = line.trim();
                    slowQueries.add(currentQuery);
                    currentQuery = null;
                }
            }
            
            return slowQueries;
        }
        
        /**
         * 生成优化建议
         */
        public List<String> generateOptimizationAdvice(SlowQuery query) {
            List<String> advice = new ArrayList<>();
            
            // 基于查询时间的建议
            if (query.queryTime > 10) {
                advice.add("查询时间过长(" + query.queryTime + "s)，建议添加索引或重写查询");
            }
            
            // 基于扫描行数的建议
            if (query.rowsExamined > query.rowsSent * 100) {
                advice.add("扫描行数过多，建议优化WHERE条件或添加合适的索引");
            }
            
            // 基于SQL模式的建议
            String sql = query.sql.toLowerCase();
            if (sql.contains("select *")) {
                advice.add("避免使用SELECT *，只查询需要的字段");
            }
            
            if (sql.contains("order by") && !sql.contains("limit")) {
                advice.add("ORDER BY查询建议添加LIMIT限制结果集大小");
            }
            
            if (sql.contains("like '%")) {
                advice.add("避免使用前缀通配符LIKE '%xxx'，考虑全文索引");
            }
            
            return advice;
        }
    }
    
    public static class SlowQuery {
        public double queryTime;
        public long lockTime;
        public int rowsSent;
        public int rowsExamined;
        public String database;
        public long timestamp;
        public String sql;
        
        public SlowQuery(double queryTime, long lockTime, int rowsSent, int rowsExamined) {
            this.queryTime = queryTime;
            this.lockTime = lockTime;
            this.rowsSent = rowsSent;
            this.rowsExamined = rowsExamined;
        }
    }
    
    // ==================== 索引优化 ====================
    
    /**
     * 索引分析和优化
     */
    public class IndexOptimizer {
        
        /**
         * 分析表的索引使用情况
         */
        public IndexAnalysisResult analyzeTableIndexes(String tableName) {
            // 获取表的索引信息
            String indexQuery = "SHOW INDEX FROM " + tableName;
            List<Map<String, Object>> indexes = jdbcTemplate.queryForList(indexQuery);
            
            // 获取索引使用统计
            String statsQuery = "SELECT * FROM information_schema.STATISTICS WHERE TABLE_NAME = ?";
            List<Map<String, Object>> stats = jdbcTemplate.queryForList(statsQuery, tableName);
            
            return new IndexAnalysisResult(tableName, indexes, stats);
        }
        
        /**
         * 建议创建的索引
         */
        public List<String> suggestIndexes(String tableName, List<String> frequentQueries) {
            List<String> suggestions = new ArrayList<>();
            
            for (String query : frequentQueries) {
                // 简化的查询分析
                if (query.toLowerCase().contains("where")) {
                    String whereClause = query.substring(query.toLowerCase().indexOf("where") + 5);
                    // 提取WHERE条件中的字段
                    String[] conditions = whereClause.split("and|or");
                    
                    for (String condition : conditions) {
                        String field = extractFieldFromCondition(condition.trim());
                        if (field != null) {
                            suggestions.add("CREATE INDEX idx_" + tableName + "_" + field + 
                                          " ON " + tableName + "(" + field + ")");
                        }
                    }
                }
            }
            
            return suggestions;
        }
        
        private String extractFieldFromCondition(String condition) {
            // 简化的字段提取逻辑
            String[] operators = {"=", ">", "<", "!=", "LIKE", "IN"};
            for (String op : operators) {
                if (condition.contains(op)) {
                    return condition.substring(0, condition.indexOf(op)).trim();
                }
            }
            return null;
        }
    }
    
    public static class IndexAnalysisResult {
        public final String tableName;
        public final List<Map<String, Object>> indexes;
        public final List<Map<String, Object>> statistics;
        
        public IndexAnalysisResult(String tableName, List<Map<String, Object>> indexes, 
                                 List<Map<String, Object>> statistics) {
            this.tableName = tableName;
            this.indexes = indexes;
            this.statistics = statistics;
        }
    }
    
    // ==================== 查询优化 ====================
    
    /**
     * 查询优化器
     */
    public class QueryOptimizer {
        
        /**
         * 分析查询执行计划
         */
        public ExecutionPlan analyzeQuery(String sql) {
            String explainQuery = "EXPLAIN " + sql;
            List<Map<String, Object>> plan = jdbcTemplate.queryForList(explainQuery);
            
            return new ExecutionPlan(sql, plan);
        }
        
        /**
         * 优化查询建议
         */
        public List<String> optimizeQuery(String sql) {
            List<String> optimizations = new ArrayList<>();
            String lowerSql = sql.toLowerCase();
            
            // 检查常见的优化点
            if (lowerSql.contains("select *")) {
                optimizations.add("使用具体字段名替代SELECT *");
            }
            
            if (lowerSql.contains("order by") && !lowerSql.contains("limit")) {
                optimizations.add("为ORDER BY查询添加LIMIT子句");
            }
            
            if (lowerSql.contains("in (") && lowerSql.contains("select")) {
                optimizations.add("考虑使用EXISTS替代IN子查询");
            }
            
            if (lowerSql.contains("like '%")) {
                optimizations.add("避免前缀通配符，考虑全文索引");
            }
            
            return optimizations;
        }
        
        /**
         * 重写查询
         */
        public String rewriteQuery(String originalSql) {
            String optimizedSql = originalSql;
            
            // 示例：将IN子查询改为EXISTS
            if (optimizedSql.toLowerCase().contains("in (select")) {
                // 这里是简化的重写逻辑
                optimizedSql = optimizedSql.replaceAll(
                    "(?i)in \\(select", "EXISTS (SELECT 1 FROM");
            }
            
            return optimizedSql;
        }
    }
    
    public static class ExecutionPlan {
        public final String sql;
        public final List<Map<String, Object>> plan;
        
        public ExecutionPlan(String sql, List<Map<String, Object>> plan) {
            this.sql = sql;
            this.plan = plan;
        }
        
        public boolean hasFullTableScan() {
            return plan.stream().anyMatch(row -> 
                "ALL".equals(row.get("type")) || "index".equals(row.get("type")));
        }
        
        public long getEstimatedRows() {
            return plan.stream()
                .mapToLong(row -> ((Number) row.get("rows")).longValue())
                .sum();
        }
    }
    
    // ==================== 连接池优化 ====================
    
    /**
     * 连接池监控和优化
     */
    public class ConnectionPoolOptimizer {
        
        /**
         * 监控连接池状态
         */
        public ConnectionPoolStatus getConnectionPoolStatus() {
            // 获取连接池状态的SQL查询
            String statusQuery = "SHOW STATUS LIKE 'Threads_%'";
            List<Map<String, Object>> status = jdbcTemplate.queryForList(statusQuery);
            
            Map<String, Object> statusMap = new HashMap<>();
            for (Map<String, Object> row : status) {
                statusMap.put((String) row.get("Variable_name"), row.get("Value"));
            }
            
            return new ConnectionPoolStatus(
                getIntValue(statusMap, "Threads_connected"),
                getIntValue(statusMap, "Threads_running"),
                getIntValue(statusMap, "Threads_created"),
                getIntValue(statusMap, "Threads_cached")
            );
        }
        
        /**
         * 生成连接池优化建议
         */
        public List<String> generatePoolOptimizationAdvice(ConnectionPoolStatus status) {
            List<String> advice = new ArrayList<>();
            
            if (status.threadsConnected > 100) {
                advice.add("连接数过多(" + status.threadsConnected + 
                          ")，建议检查连接泄漏或增加连接池大小");
            }
            
            if (status.threadsRunning > status.threadsConnected * 0.8) {
                advice.add("活跃连接比例过高，可能存在慢查询阻塞");
            }
            
            if (status.threadsCreated > status.threadsConnected * 2) {
                advice.add("连接创建次数过多，建议增加连接池缓存大小");
            }
            
            return advice;
        }
        
        private int getIntValue(Map<String, Object> map, String key) {
            Object value = map.get(key);
            return value != null ? Integer.parseInt(value.toString()) : 0;
        }
    }
    
    public static class ConnectionPoolStatus {
        public final int threadsConnected;
        public final int threadsRunning;
        public final int threadsCreated;
        public final int threadsCached;
        
        public ConnectionPoolStatus(int threadsConnected, int threadsRunning, 
                                  int threadsCreated, int threadsCached) {
            this.threadsConnected = threadsConnected;
            this.threadsRunning = threadsRunning;
            this.threadsCreated = threadsCreated;
            this.threadsCached = threadsCached;
        }
    }
}
```