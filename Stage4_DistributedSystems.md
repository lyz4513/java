# 第四阶段：分布式系统 - 微服务、缓存、数据库优化

## 学习目标

通过本阶段学习，你将掌握：

1. **微服务架构**：Spring Cloud生态系统、服务注册与发现、负载均衡
2. **分布式缓存**：Redis集群、缓存策略、缓存一致性
3. **数据库优化**：MySQL性能调优、分库分表、读写分离
4. **分布式事务**：两阶段提交、TCC、Saga模式
5. **服务治理**：熔断降级、限流、监控
6. **消息中间件**：Kafka、RocketMQ高级特性

## 知识体系

### 1. 微服务架构基础

#### 1.1 Spring Cloud Eureka服务注册与发现

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.ArrayList;
import java.time.LocalDateTime;

/**
 * Eureka服务注册中心
 * 提供服务注册、发现、健康检查功能
 */
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
        System.out.println("Eureka服务注册中心启动成功！");
        System.out.println("访问地址: http://localhost:8761");
    }
}

#### 3.2 分库分表和读写分离

```java
import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Primary;

import javax.sql.DataSource;
import java.util.*;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

/**
 * 分库分表服务
 * 演示水平分库分表策略
 */
@Service
public class ShardingService {
    
    @Autowired
    @Qualifier("masterDataSource")
    private DataSource masterDataSource;
    
    @Autowired
    @Qualifier("slaveDataSource")
    private DataSource slaveDataSource;
    
    @Autowired
    @Qualifier("masterJdbcTemplate")
    private JdbcTemplate masterJdbcTemplate;
    
    @Autowired
    @Qualifier("slaveJdbcTemplate")
    private JdbcTemplate slaveJdbcTemplate;
    
    // ==================== 分库策略 ====================
    
    /**
     * 根据用户ID进行分库
     * 使用取模算法决定数据存储在哪个数据库
     */
    public String getShardDatabase(Long userId) {
        // 假设有4个数据库分片
        int shardCount = 4;
        int shardIndex = (int) (userId % shardCount);
        return "db_shard_" + shardIndex;
    }
    
    /**
     * 根据订单ID进行分表
     * 使用时间+取模策略决定数据存储在哪个表
     */
    public String getShardTable(Long orderId, LocalDateTime createTime) {
        // 按月分表
        String monthSuffix = createTime.format(DateTimeFormatter.ofPattern("yyyyMM"));
        
        // 同时使用取模进行二次分表
        int tableCount = 8;
        int tableIndex = (int) (orderId % tableCount);
        
        return "orders_" + monthSuffix + "_" + String.format("%02d", tableIndex);
    }
    
    /**
     * 分库分表插入订单
     */
    @Transactional
    public void insertOrderWithSharding(Order order) {
        // 1. 确定分库
        String database = getShardDatabase(order.getUserId());
        
        // 2. 确定分表
        String table = getShardTable(order.getId(), order.getCreateTime());
        
        System.out.println(String.format("插入订单到 %s.%s", database, table));
        
        // 3. 构建动态SQL
        String sql = String.format(
            "INSERT INTO %s (id, user_id, order_number, amount, status, create_time) VALUES (?, ?, ?, ?, ?, ?)",
            table
        );
        
        // 4. 根据分库选择数据源执行
        JdbcTemplate jdbcTemplate = getJdbcTemplateByDatabase(database);
        jdbcTemplate.update(sql,
            order.getId(),
            order.getUserId(),
            order.getOrderNumber(),
            order.getAmount(),
            order.getStatus(),
            order.getCreateTime()
        );
        
        System.out.println("订单插入成功: " + order.getOrderNumber());
    }
    
    /**
     * 分库分表查询订单
     */
    public List<Map<String, Object>> findOrdersByUserId(Long userId, LocalDateTime startTime, LocalDateTime endTime) {
        // 1. 确定分库
        String database = getShardDatabase(userId);
        
        // 2. 确定需要查询的分表（按时间范围）
        List<String> tables = getTablesByTimeRange(startTime, endTime);
        
        List<Map<String, Object>> allOrders = new ArrayList<>();
        
        // 3. 遍历所有相关表进行查询
        for (String table : tables) {
            String sql = String.format(
                "SELECT * FROM %s WHERE user_id = ? AND create_time BETWEEN ? AND ? ORDER BY create_time DESC",
                table
            );
            
            try {
                JdbcTemplate jdbcTemplate = getJdbcTemplateByDatabase(database);
                List<Map<String, Object>> orders = jdbcTemplate.queryForList(sql, userId, startTime, endTime);
                allOrders.addAll(orders);
                
                System.out.println(String.format("从 %s.%s 查询到 %d 条订单", database, table, orders.size()));
            } catch (Exception e) {
                System.err.println(String.format("查询表 %s.%s 失败: %s", database, table, e.getMessage()));
            }
        }
        
        // 4. 合并结果并排序
        allOrders.sort((o1, o2) -> {
            LocalDateTime time1 = (LocalDateTime) o1.get("create_time");
            LocalDateTime time2 = (LocalDateTime) o2.get("create_time");
            return time2.compareTo(time1); // 降序排列
        });
        
        return allOrders;
    }
    
    // ==================== 读写分离 ====================
    
    /**
     * 写操作 - 使用主库
     */
    @Transactional
    public void saveUserToMaster(CacheUser user) {
        String sql = "INSERT INTO users (username, email, age, status, create_time) VALUES (?, ?, ?, ?, ?)";
        
        masterJdbcTemplate.update(sql,
            user.getUsername(),
            user.getEmail(),
            user.getAge(),
            "ACTIVE",
            LocalDateTime.now()
        );
        
        System.out.println("用户数据写入主库: " + user.getUsername());
    }
    
    /**
     * 读操作 - 使用从库
     */
    public List<Map<String, Object>> findUsersFromSlave(String status) {
        String sql = "SELECT id, username, email, age, status, create_time FROM users WHERE status = ? ORDER BY create_time DESC LIMIT 100";
        
        List<Map<String, Object>> users = slaveJdbcTemplate.queryForList(sql, status);
        System.out.println("从从库查询用户数据: " + users.size() + " 条记录");
        
        return users;
    }
    
    /**
     * 强制从主库读取（解决主从延迟问题）
     */
    public Map<String, Object> findUserFromMaster(Long userId) {
        String sql = "SELECT * FROM users WHERE id = ?";
        
        try {
            Map<String, Object> user = masterJdbcTemplate.queryForMap(sql, userId);
            System.out.println("从主库强制读取用户: " + userId);
            return user;
        } catch (Exception e) {
            System.err.println("从主库读取用户失败: " + e.getMessage());
            return null;
        }
    }
    
    // ==================== 辅助方法 ====================
    
    /**
     * 根据数据库名称获取对应的JdbcTemplate
     */
    private JdbcTemplate getJdbcTemplateByDatabase(String database) {
        // 简化实现，实际应该根据分库配置动态选择
        if (database.contains("0") || database.contains("2")) {
            return masterJdbcTemplate;
        } else {
            return slaveJdbcTemplate;
        }
    }
    
    /**
     * 根据时间范围获取需要查询的表列表
     */
    private List<String> getTablesByTimeRange(LocalDateTime startTime, LocalDateTime endTime) {
        List<String> tables = new ArrayList<>();
        
        LocalDateTime current = startTime.withDayOfMonth(1); // 月初
        while (!current.isAfter(endTime)) {
            String monthSuffix = current.format(DateTimeFormatter.ofPattern("yyyyMM"));
            
            // 每个月有8个分表
            for (int i = 0; i < 8; i++) {
                tables.add("orders_" + monthSuffix + "_" + String.format("%02d", i));
            }
            
            current = current.plusMonths(1);
        }
        
        return tables;
    }
}

### 4. 分布式事务

#### 4.1 两阶段提交（2PC）和TCC模式

```java
import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.support.TransactionTemplate;
import org.springframework.transaction.PlatformTransactionManager;

import java.util.*;
import java.time.LocalDateTime;
import java.math.BigDecimal;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.TimeUnit;

/**
 * 分布式事务服务
 * 演示TCC（Try-Confirm-Cancel）模式
 */
@Service
public class DistributedTransactionService {
    
    @Autowired
    private AccountService accountService;
    
    @Autowired
    private InventoryService inventoryService;
    
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private TransactionTemplate transactionTemplate;
    
    // ==================== TCC事务管理器 ====================
    
    /**
     * TCC事务上下文
     */
    public static class TCCContext {
        private String transactionId;
        private Map<String, Object> context;
        private List<TCCParticipant> participants;
        private TCCStatus status;
        private LocalDateTime createTime;
        
        public TCCContext(String transactionId) {
            this.transactionId = transactionId;
            this.context = new HashMap<>();
            this.participants = new ArrayList<>();
            this.status = TCCStatus.TRYING;
            this.createTime = LocalDateTime.now();
        }
        
        // Getters and Setters
        public String getTransactionId() { return transactionId; }
        public Map<String, Object> getContext() { return context; }
        public List<TCCParticipant> getParticipants() { return participants; }
        public TCCStatus getStatus() { return status; }
        public void setStatus(TCCStatus status) { this.status = status; }
        public LocalDateTime getCreateTime() { return createTime; }
        
        public void addParticipant(TCCParticipant participant) {
            this.participants.add(participant);
        }
    }
    
    /**
     * TCC事务状态
     */
    public enum TCCStatus {
        TRYING,     // 尝试阶段
        CONFIRMING, // 确认阶段
        CANCELING,  // 取消阶段
        CONFIRMED,  // 已确认
        CANCELED    // 已取消
    }
    
    /**
     * TCC参与者接口
     */
    public interface TCCParticipant {
        boolean tryExecute(TCCContext context);
        boolean confirmExecute(TCCContext context);
        boolean cancelExecute(TCCContext context);
        String getParticipantId();
    }
    
    /**
     * TCC事务管理器
     */
    @Service
    public static class TCCTransactionManager {
        
        private Map<String, TCCContext> transactions = new HashMap<>();
        
        /**
         * 开始TCC事务
         */
        public TCCContext beginTransaction() {
            String transactionId = "TCC_" + System.currentTimeMillis() + "_" + UUID.randomUUID().toString().substring(0, 8);
            TCCContext context = new TCCContext(transactionId);
            transactions.put(transactionId, context);
            
            System.out.println("开始TCC事务: " + transactionId);
            return context;
        }
        
        /**
         * 提交TCC事务
         */
        public boolean commitTransaction(TCCContext context) {
            System.out.println("提交TCC事务: " + context.getTransactionId());
            
            // 1. Try阶段
            if (!tryPhase(context)) {
                cancelPhase(context);
                return false;
            }
            
            // 2. Confirm阶段
            if (!confirmPhase(context)) {
                cancelPhase(context);
                return false;
            }
            
            context.setStatus(TCCStatus.CONFIRMED);
            System.out.println("TCC事务提交成功: " + context.getTransactionId());
            return true;
        }
        
        /**
         * Try阶段：尝试执行所有参与者的业务逻辑
         */
        private boolean tryPhase(TCCContext context) {
            System.out.println("执行Try阶段: " + context.getTransactionId());
            context.setStatus(TCCStatus.TRYING);
            
            for (TCCParticipant participant : context.getParticipants()) {
                try {
                    if (!participant.tryExecute(context)) {
                        System.err.println("Try阶段失败: " + participant.getParticipantId());
                        return false;
                    }
                    System.out.println("Try阶段成功: " + participant.getParticipantId());
                } catch (Exception e) {
                    System.err.println("Try阶段异常: " + participant.getParticipantId() + ", " + e.getMessage());
                    return false;
                }
            }
            
            return true;
        }
        
        /**
         * Confirm阶段：确认执行所有参与者的业务逻辑
         */
        private boolean confirmPhase(TCCContext context) {
            System.out.println("执行Confirm阶段: " + context.getTransactionId());
            context.setStatus(TCCStatus.CONFIRMING);
            
            boolean allSuccess = true;
            for (TCCParticipant participant : context.getParticipants()) {
                try {
                    if (!participant.confirmExecute(context)) {
                        System.err.println("Confirm阶段失败: " + participant.getParticipantId());
                        allSuccess = false;
                    } else {
                        System.out.println("Confirm阶段成功: " + participant.getParticipantId());
                    }
                } catch (Exception e) {
                    System.err.println("Confirm阶段异常: " + participant.getParticipantId() + ", " + e.getMessage());
                    allSuccess = false;
                }
            }
            
            return allSuccess;
        }
        
        /**
         * Cancel阶段：取消所有参与者的业务逻辑
         */
        private void cancelPhase(TCCContext context) {
            System.out.println("执行Cancel阶段: " + context.getTransactionId());
            context.setStatus(TCCStatus.CANCELING);
            
            for (TCCParticipant participant : context.getParticipants()) {
                try {
                    participant.cancelExecute(context);
                    System.out.println("Cancel阶段成功: " + participant.getParticipantId());
                } catch (Exception e) {
                    System.err.println("Cancel阶段异常: " + participant.getParticipantId() + ", " + e.getMessage());
                }
            }
            
            context.setStatus(TCCStatus.CANCELED);
        }
    }
    
    // ==================== TCC参与者实现 ====================
    
    /**
     * 账户服务TCC参与者
     */
    @Service
    public static class AccountTCCParticipant implements TCCParticipant {
        
        @Override
        public boolean tryExecute(TCCContext context) {
            Long userId = (Long) context.getContext().get("userId");
            BigDecimal amount = (BigDecimal) context.getContext().get("amount");
            
            // 冻结账户余额
            boolean success = freezeAccountBalance(userId, amount);
            if (success) {
                context.getContext().put("account_frozen", true);
                System.out.println("账户余额冻结成功: 用户" + userId + ", 金额" + amount);
            }
            
            return success;
        }
        
        @Override
        public boolean confirmExecute(TCCContext context) {
            Long userId = (Long) context.getContext().get("userId");
            BigDecimal amount = (BigDecimal) context.getContext().get("amount");
            
            // 扣减冻结的余额
            boolean success = deductFrozenBalance(userId, amount);
            if (success) {
                System.out.println("账户余额扣减成功: 用户" + userId + ", 金额" + amount);
            }
            
            return success;
        }
        
        @Override
        public boolean cancelExecute(TCCContext context) {
            Long userId = (Long) context.getContext().get("userId");
            BigDecimal amount = (BigDecimal) context.getContext().get("amount");
            
            // 解冻账户余额
            boolean success = unfreezeAccountBalance(userId, amount);
            if (success) {
                System.out.println("账户余额解冻成功: 用户" + userId + ", 金额" + amount);
            }
            
            return success;
        }
        
        @Override
        public String getParticipantId() {
            return "AccountService";
        }
        
        private boolean freezeAccountBalance(Long userId, BigDecimal amount) {
            // 模拟冻结账户余额
            try {
                Thread.sleep(50); // 模拟数据库操作
                return true;
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return false;
            }
        }
        
        private boolean deductFrozenBalance(Long userId, BigDecimal amount) {
            // 模拟扣减冻结余额
            try {
                Thread.sleep(50);
                return true;
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return false;
            }
        }
        
        private boolean unfreezeAccountBalance(Long userId, BigDecimal amount) {
            // 模拟解冻账户余额
            try {
                Thread.sleep(50);
                return true;
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return false;
            }
        }
    }
    
    /**
     * 库存服务TCC参与者
     */
    @Service
    public static class InventoryTCCParticipant implements TCCParticipant {
        
        @Override
        public boolean tryExecute(TCCContext context) {
            Long productId = (Long) context.getContext().get("productId");
            Integer quantity = (Integer) context.getContext().get("quantity");
            
            // 预扣库存
            boolean success = reserveInventory(productId, quantity);
            if (success) {
                context.getContext().put("inventory_reserved", true);
                System.out.println("库存预扣成功: 商品" + productId + ", 数量" + quantity);
            }
            
            return success;
        }
        
        @Override
        public boolean confirmExecute(TCCContext context) {
            Long productId = (Long) context.getContext().get("productId");
            Integer quantity = (Integer) context.getContext().get("quantity");
            
            // 确认扣减库存
            boolean success = confirmInventoryDeduction(productId, quantity);
            if (success) {
                System.out.println("库存扣减确认成功: 商品" + productId + ", 数量" + quantity);
            }
            
            return success;
        }
        
        @Override
        public boolean cancelExecute(TCCContext context) {
            Long productId = (Long) context.getContext().get("productId");
            Integer quantity = (Integer) context.getContext().get("quantity");
            
            // 释放预扣库存
            boolean success = releaseReservedInventory(productId, quantity);
            if (success) {
                System.out.println("库存释放成功: 商品" + productId + ", 数量" + quantity);
            }
            
            return success;
        }
        
        @Override
        public String getParticipantId() {
            return "InventoryService";
        }
        
        private boolean reserveInventory(Long productId, Integer quantity) {
            // 模拟预扣库存
            try {
                Thread.sleep(50);
                // 模拟库存不足的情况
                return productId != 999L; // 商品999库存不足
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return false;
            }
        }
        
        private boolean confirmInventoryDeduction(Long productId, Integer quantity) {
            // 模拟确认扣减库存
            try {
                Thread.sleep(50);
                return true;
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return false;
            }
        }
        
        private boolean releaseReservedInventory(Long productId, Integer quantity) {
            // 模拟释放预扣库存
            try {
                Thread.sleep(50);
                return true;
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return false;
            }
        }
    }
    
    /**
     * 订单服务TCC参与者
     */
    @Service
    public static class OrderTCCParticipant implements TCCParticipant {
        
        @Override
        public boolean tryExecute(TCCContext context) {
            String orderNumber = (String) context.getContext().get("orderNumber");
            
            // 创建预订单
            boolean success = createPendingOrder(orderNumber);
            if (success) {
                context.getContext().put("order_created", true);
                System.out.println("预订单创建成功: " + orderNumber);
            }
            
            return success;
        }
        
        @Override
        public boolean confirmExecute(TCCContext context) {
            String orderNumber = (String) context.getContext().get("orderNumber");
            
            // 确认订单
            boolean success = confirmOrder(orderNumber);
            if (success) {
                System.out.println("订单确认成功: " + orderNumber);
            }
            
            return success;
        }
        
        @Override
        public boolean cancelExecute(TCCContext context) {
            String orderNumber = (String) context.getContext().get("orderNumber");
            
            // 取消订单
            boolean success = cancelOrder(orderNumber);
            if (success) {
                System.out.println("订单取消成功: " + orderNumber);
            }
            
            return success;
        }
        
        @Override
        public String getParticipantId() {
            return "OrderService";
        }
        
        private boolean createPendingOrder(String orderNumber) {
            // 模拟创建预订单
            try {
                Thread.sleep(50);
                return true;
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return false;
            }
        }
        
        private boolean confirmOrder(String orderNumber) {
            // 模拟确认订单
            try {
                Thread.sleep(50);
                return true;
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return false;
            }
        }
        
        private boolean cancelOrder(String orderNumber) {
            // 模拟取消订单
            try {
                Thread.sleep(50);
                return true;
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return false;
            }
        }
    }
    
    // ==================== 业务服务 ====================
    
    @Autowired
    private TCCTransactionManager tccTransactionManager;
    
    @Autowired
    private AccountTCCParticipant accountParticipant;
    
    @Autowired
    private InventoryTCCParticipant inventoryParticipant;
    
    @Autowired
    private OrderTCCParticipant orderParticipant;
    
    /**
     * 下单业务 - 使用TCC分布式事务
     */
    public boolean placeOrderWithTCC(Long userId, Long productId, Integer quantity, BigDecimal amount) {
        // 1. 开始TCC事务
        TCCContext context = tccTransactionManager.beginTransaction();
        
        // 2. 设置事务上下文
        String orderNumber = "ORDER_" + System.currentTimeMillis();
        context.getContext().put("userId", userId);
        context.getContext().put("productId", productId);
        context.getContext().put("quantity", quantity);
        context.getContext().put("amount", amount);
        context.getContext().put("orderNumber", orderNumber);
        
        // 3. 添加TCC参与者
        context.addParticipant(accountParticipant);
        context.addParticipant(inventoryParticipant);
        context.addParticipant(orderParticipant);
        
        // 4. 提交TCC事务
        boolean success = tccTransactionManager.commitTransaction(context);
        
        if (success) {
            System.out.println("下单成功: " + orderNumber);
        } else {
            System.out.println("下单失败: " + orderNumber);
        }
        
        return success;
    }
}

### 5. 服务治理

#### 5.1 熔断降级和限流

```java
import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import io.github.resilience4j.circuitbreaker.CircuitBreaker;
import io.github.resilience4j.circuitbreaker.CircuitBreakerConfig;
import io.github.resilience4j.ratelimiter.RateLimiter;
import io.github.resilience4j.ratelimiter.RateLimiterConfig;
import io.github.resilience4j.retry.Retry;
import io.github.resilience4j.retry.RetryConfig;
import io.github.resilience4j.bulkhead.Bulkhead;
import io.github.resilience4j.bulkhead.BulkheadConfig;

import java.time.Duration;
import java.util.*;
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.function.Supplier;

/**
 * 服务治理配置
 * 配置熔断器、限流器、重试等组件
 */
@Configuration
public class ResilienceConfig {
    
    /**
     * 配置熔断器
     */
    @Bean
    public CircuitBreaker userServiceCircuitBreaker() {
        CircuitBreakerConfig config = CircuitBreakerConfig.custom()
                .failureRateThreshold(50)                    // 失败率阈值50%
                .waitDurationInOpenState(Duration.ofSeconds(30))  // 熔断器打开后等待30秒
                .slidingWindowSize(10)                       // 滑动窗口大小10
                .minimumNumberOfCalls(5)                     // 最小调用次数5
                .permittedNumberOfCallsInHalfOpenState(3)    // 半开状态允许3次调用
                .automaticTransitionFromOpenToHalfOpenEnabled(true) // 自动从打开到半开
                .build();
        
        return CircuitBreaker.of("userService", config);
    }
    
    /**
     * 配置限流器
     */
    @Bean
    public RateLimiter userServiceRateLimiter() {
        RateLimiterConfig config = RateLimiterConfig.custom()
                .limitForPeriod(10)                          // 每个周期允许10次调用
                .limitRefreshPeriod(Duration.ofSeconds(1))   // 每1秒刷新一次
                .timeoutDuration(Duration.ofMillis(500))     // 获取许可超时500ms
                .build();
        
        return RateLimiter.of("userService", config);
    }
    
    /**
     * 配置重试器
     */
    @Bean
    public Retry userServiceRetry() {
        RetryConfig config = RetryConfig.custom()
                .maxAttempts(3)                              // 最大重试3次
                .waitDuration(Duration.ofMillis(500))        // 重试间隔500ms
                .retryOnException(throwable -> 
                    throwable instanceof RuntimeException)   // 只对RuntimeException重试
                .build();
        
        return Retry.of("userService", config);
    }
    
    /**
     * 配置舱壁隔离
     */
    @Bean
    public Bulkhead userServiceBulkhead() {
        BulkheadConfig config = BulkheadConfig.custom()
                .maxConcurrentCalls(5)                       // 最大并发调用5个
                .maxWaitDuration(Duration.ofMillis(1000))    // 最大等待1秒
                .build();
        
        return Bulkhead.of("userService", config);
    }
}

/**
 * 服务治理服务
 * 演示熔断、限流、重试等功能
 */
@Service
public class ServiceGovernanceService {
    
    @Autowired
    private CircuitBreaker userServiceCircuitBreaker;
    
    @Autowired
    private RateLimiter userServiceRateLimiter;
    
    @Autowired
    private Retry userServiceRetry;
    
    @Autowired
    private Bulkhead userServiceBulkhead;
    
    // 模拟服务调用失败计数器
    private AtomicInteger failureCount = new AtomicInteger(0);
    
    // ==================== 熔断器示例 ====================
    
    /**
     * 使用熔断器保护的服务调用
     */
    public CacheUser getUserWithCircuitBreaker(Long userId) {
        Supplier<CacheUser> decoratedSupplier = CircuitBreaker
                .decorateSupplier(userServiceCircuitBreaker, () -> {
                    return callUserService(userId);
                });
        
        try {
            CacheUser user = decoratedSupplier.get();
            System.out.println("熔断器状态: " + userServiceCircuitBreaker.getState());
            return user;
        } catch (Exception e) {
            System.err.println("熔断器调用失败: " + e.getMessage());
            // 返回降级结果
            return getFallbackUser(userId);
        }
    }
    
    /**
     * 模拟用户服务调用
     */
    private CacheUser callUserService(Long userId) {
        // 模拟服务调用
        try {
            Thread.sleep(100); // 模拟网络延迟
            
            // 模拟服务偶尔失败
            if (failureCount.incrementAndGet() % 3 == 0) {
                throw new RuntimeException("用户服务暂时不可用");
            }
            
            return new CacheUser(userId, "用户" + userId, "user" + userId + "@example.com", 25);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException("服务调用被中断");
        }
    }
    
    /**
     * 降级方法
     */
    private CacheUser getFallbackUser(Long userId) {
        System.out.println("执行降级逻辑，返回默认用户");
        return new CacheUser(userId, "默认用户", "default@example.com", 0);
    }
    
    // ==================== 限流器示例 ====================
    
    /**
     * 使用限流器保护的服务调用
     */
    public List<CacheUser> getUsersWithRateLimit(int count) {
        Supplier<List<CacheUser>> decoratedSupplier = RateLimiter
                .decorateSupplier(userServiceRateLimiter, () -> {
                    return callUserListService(count);
                });
        
        try {
            List<CacheUser> users = decoratedSupplier.get();
            System.out.println("限流器调用成功，获取用户数: " + users.size());
            return users;
        } catch (Exception e) {
            System.err.println("限流器调用失败: " + e.getMessage());
            // 返回空列表或缓存数据
            return new ArrayList<>();
        }
    }
    
    /**
     * 模拟用户列表服务调用
     */
    private List<CacheUser> callUserListService(int count) {
        List<CacheUser> users = new ArrayList<>();
        for (int i = 1; i <= count; i++) {
            users.add(new CacheUser((long) i, "用户" + i, "user" + i + "@example.com", 20 + i));
        }
        return users;
    }
    
    // ==================== 重试器示例 ====================
    
    /**
     * 使用重试器的服务调用
     */
    public CacheUser getUserWithRetry(Long userId) {
        Supplier<CacheUser> decoratedSupplier = Retry
                .decorateSupplier(userServiceRetry, () -> {
                    return callUnstableUserService(userId);
                });
        
        try {
            CacheUser user = decoratedSupplier.get();
            System.out.println("重试器调用成功: " + user.getUsername());
            return user;
        } catch (Exception e) {
            System.err.println("重试器最终失败: " + e.getMessage());
            return getFallbackUser(userId);
        }
    }
    
    /**
     * 模拟不稳定的用户服务
     */
    private CacheUser callUnstableUserService(Long userId) {
        // 模拟70%的失败率
        if (Math.random() < 0.7) {
            System.out.println("服务调用失败，准备重试...");
            throw new RuntimeException("网络超时");
        }
        
        System.out.println("服务调用成功");
        return new CacheUser(userId, "重试用户" + userId, "retry" + userId + "@example.com", 30);
    }
    
    // ==================== 舱壁隔离示例 ====================
    
    /**
     * 使用舱壁隔离的服务调用
     */
    public CompletableFuture<CacheUser> getUserWithBulkhead(Long userId) {
        Supplier<CompletableFuture<CacheUser>> decoratedSupplier = Bulkhead
                .decorateSupplier(userServiceBulkhead, () -> {
                    return CompletableFuture.supplyAsync(() -> callSlowUserService(userId));
                });
        
        try {
            CompletableFuture<CacheUser> future = decoratedSupplier.get();
            System.out.println("舱壁隔离调用提交成功");
            return future;
        } catch (Exception e) {
            System.err.println("舱壁隔离调用失败: " + e.getMessage());
            return CompletableFuture.completedFuture(getFallbackUser(userId));
        }
    }
    
    /**
     * 模拟慢服务调用
     */
    private CacheUser callSlowUserService(Long userId) {
        try {
            // 模拟慢服务
            Thread.sleep(2000);
            return new CacheUser(userId, "慢服务用户" + userId, "slow" + userId + "@example.com", 35);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException("服务调用被中断");
        }
    }
    
    // ==================== 组合使用示例 ====================
    
    /**
     * 组合使用多种治理策略
     */
    public CacheUser getUserWithAllStrategies(Long userId) {
        // 组合限流器、熔断器、重试器
        Supplier<CacheUser> decoratedSupplier = RateLimiter
                .decorateSupplier(userServiceRateLimiter,
                    CircuitBreaker.decorateSupplier(userServiceCircuitBreaker,
                        Retry.decorateSupplier(userServiceRetry, () -> {
                            return callUserService(userId);
                        })
                    )
                );
        
        try {
            CacheUser user = decoratedSupplier.get();
            System.out.println("组合策略调用成功: " + user.getUsername());
            return user;
        } catch (Exception e) {
            System.err.println("组合策略调用失败: " + e.getMessage());
            return getFallbackUser(userId);
        }
    }
    
    // ==================== 监控和指标 ====================
    
    /**
     * 获取熔断器指标
     */
    public Map<String, Object> getCircuitBreakerMetrics() {
        Map<String, Object> metrics = new HashMap<>();
        
        CircuitBreaker.Metrics cbMetrics = userServiceCircuitBreaker.getMetrics();
        metrics.put("state", userServiceCircuitBreaker.getState().toString());
        metrics.put("failure_rate", cbMetrics.getFailureRate());
        metrics.put("number_of_calls", cbMetrics.getNumberOfBufferedCalls());
        metrics.put("number_of_failed_calls", cbMetrics.getNumberOfFailedCalls());
        metrics.put("number_of_successful_calls", cbMetrics.getNumberOfSuccessfulCalls());
        
        return metrics;
    }
    
    /**
     * 获取限流器指标
     */
    public Map<String, Object> getRateLimiterMetrics() {
        Map<String, Object> metrics = new HashMap<>();
        
        RateLimiter.Metrics rlMetrics = userServiceRateLimiter.getMetrics();
        metrics.put("available_permissions", rlMetrics.getAvailablePermissions());
        metrics.put("number_of_waiting_threads", rlMetrics.getNumberOfWaitingThreads());
        
        return metrics;
    }
    
    /**
     * 获取舱壁隔离指标
     */
    public Map<String, Object> getBulkheadMetrics() {
        Map<String, Object> metrics = new HashMap<>();
        
        Bulkhead.Metrics bhMetrics = userServiceBulkhead.getMetrics();
        metrics.put("available_concurrent_calls", bhMetrics.getAvailableConcurrentCalls());
        metrics.put("max_allowed_concurrent_calls", bhMetrics.getMaxAllowedConcurrentCalls());
        
        return metrics;
    }
}

/**
 * 服务治理控制器
 * 提供服务治理功能的REST API
 */
@RestController
@RequestMapping("/governance")
class ServiceGovernanceController {
    
    @Autowired
    private ServiceGovernanceService governanceService;
    
    /**
     * 测试熔断器
     */
    @GetMapping("/circuit-breaker/user/{id}")
    public CacheUser testCircuitBreaker(@PathVariable Long id) {
        return governanceService.getUserWithCircuitBreaker(id);
    }
    
    /**
     * 测试限流器
     */
    @GetMapping("/rate-limiter/users")
    public List<CacheUser> testRateLimiter(@RequestParam(defaultValue = "5") int count) {
        return governanceService.getUsersWithRateLimit(count);
    }
    
    /**
     * 测试重试器
     */
    @GetMapping("/retry/user/{id}")
    public CacheUser testRetry(@PathVariable Long id) {
        return governanceService.getUserWithRetry(id);
    }
    
    /**
     * 测试舱壁隔离
     */
    @GetMapping("/bulkhead/user/{id}")
    public CompletableFuture<CacheUser> testBulkhead(@PathVariable Long id) {
        return governanceService.getUserWithBulkhead(id);
    }
    
    /**
     * 测试组合策略
     */
    @GetMapping("/combined/user/{id}")
    public CacheUser testCombinedStrategies(@PathVariable Long id) {
        return governanceService.getUserWithAllStrategies(id);
    }
    
    /**
     * 获取治理指标
     */
    @GetMapping("/metrics")
    public Map<String, Object> getGovernanceMetrics() {
        Map<String, Object> allMetrics = new HashMap<>();
        allMetrics.put("circuit_breaker", governanceService.getCircuitBreakerMetrics());
        allMetrics.put("rate_limiter", governanceService.getRateLimiterMetrics());
        allMetrics.put("bulkhead", governanceService.getBulkheadMetrics());
        return allMetrics;
    }
}

## 经典面试题

### 1. 微服务架构相关

**Q1: 什么是微服务架构？它有哪些优缺点？**

**答案：**

微服务架构是一种将单一应用程序开发为一组小型服务的方法，每个服务运行在自己的进程中，并使用轻量级机制（通常是HTTP API）进行通信。每个微服务都围绕特定的业务功能构建，可以由小团队独立开发、部署和维护。

**微服务架构特征：**

| 特征 | 描述 | 实现方式 | 示例 |
|------|------|----------|------|
| **服务自治** | 每个服务独立开发、部署、运行 | 独立的代码库、数据库、部署流水线 | 用户服务、订单服务、支付服务 |
| **去中心化** | 没有中央控制器，服务间平等通信 | 服务注册发现、API网关 | Eureka、Consul、Spring Cloud Gateway |
| **故障隔离** | 单个服务故障不影响整个系统 | 熔断器、舱壁模式 | Hystrix、Resilience4j |
| **技术多样性** | 不同服务可使用不同技术栈 | 语言无关的API | Java、Python、Go、Node.js |
| **数据分离** | 每个服务管理自己的数据 | 数据库分离、事件驱动 | 每服务一个数据库 |

**详细优缺点分析：**

**优点：**

```java
// 1. 独立部署和扩展示例

/**
 * 用户服务 - 可独立部署和扩展
 */
@SpringBootApplication
@EnableEurekaClient
public class UserServiceApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
        System.out.println("用户服务启动成功 - 端口: 8081");
    }
}

@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    /**
     * 用户服务可以独立扩展
     * 根据用户访问量单独调整实例数量
     */
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        return ResponseEntity.ok(user);
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User savedUser = userService.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(savedUser);
    }
}

/**
 * 订单服务 - 独立的业务逻辑和数据
 */
@SpringBootApplication
@EnableEurekaClient
public class OrderServiceApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
        System.out.println("订单服务启动成功 - 端口: 8082");
    }
}

@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private UserServiceClient userServiceClient; // 通过Feign调用用户服务
    
    /**
     * 订单服务调用用户服务
     * 展示服务间通信
     */
    @PostMapping
    public ResponseEntity<Order> createOrder(@RequestBody CreateOrderRequest request) {
        
        // 1. 验证用户是否存在（调用用户服务）
        User user = userServiceClient.getUser(request.getUserId());
        if (user == null) {
            return ResponseEntity.badRequest().build();
        }
        
        // 2. 创建订单
        Order order = orderService.createOrder(request);
        
        return ResponseEntity.status(HttpStatus.CREATED).body(order);
    }
}

// 2. 技术栈多样性示例

/**
 * 用户服务使用MySQL + JPA
 */
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String username;
    
    @Column(nullable = false)
    private String email;
    
    // getter、setter
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    Optional<User> findByEmail(String email);
}

/**
 * 商品服务使用MongoDB
 */
@Document(collection = "products")
public class Product {
    @Id
    private String id;
    
    private String name;
    private String description;
    private BigDecimal price;
    private List<String> categories;
    private Map<String, Object> attributes;
    
    // getter、setter
}

@Repository
public interface ProductRepository extends MongoRepository<Product, String> {
    List<Product> findByCategoriesContaining(String category);
    List<Product> findByPriceBetween(BigDecimal minPrice, BigDecimal maxPrice);
}

/**
 * 日志服务使用Elasticsearch
 */
@Document(indexName = "application-logs")
public class ApplicationLog {
    @Id
    private String id;
    
    @Field(type = FieldType.Date)
    private LocalDateTime timestamp;
    
    @Field(type = FieldType.Keyword)
    private String level;
    
    @Field(type = FieldType.Keyword)
    private String service;
    
    @Field(type = FieldType.Text)
    private String message;
    
    // getter、setter
}

// 3. 故障隔离示例

/**
 * 使用熔断器实现故障隔离
 */
@Component
public class UserServiceClient {
    
    @Autowired
    private RestTemplate restTemplate;
    
    /**
     * 调用用户服务，带熔断保护
     */
    @HystrixCommand(
        fallbackMethod = "getUserFallback",
        commandProperties = {
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50"),
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "5000")
        }
    )
    public User getUser(Long userId) {
        String url = "http://user-service/api/users/" + userId;
        return restTemplate.getForObject(url, User.class);
    }
    
    /**
     * 降级方法：当用户服务不可用时返回默认用户
     */
    public User getUserFallback(Long userId) {
        User defaultUser = new User();
        defaultUser.setId(userId);
        defaultUser.setUsername("临时用户");
        defaultUser.setEmail("temp@example.com");
        return defaultUser;
    }
}
```

**缺点：**

```java
// 1. 分布式系统复杂性示例

/**
 * 分布式配置管理
 */
@Configuration
public class DistributedSystemConfig {
    
    /**
     * 服务发现配置
     */
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
    
    /**
     * 分布式追踪配置
     */
    @Bean
    public Sampler alwaysSampler() {
        return Sampler.create(1.0f); // 100%采样
    }
    
    /**
     * 分布式锁配置
     */
    @Bean
    public RedissonClient redissonClient() {
        Config config = new Config();
        config.useSingleServer().setAddress("redis://localhost:6379");
        return Redisson.create(config);
    }
}

// 2. 数据一致性挑战示例

/**
 * 分布式事务处理
 */
@Service
public class OrderTransactionService {
    
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private PaymentServiceClient paymentServiceClient;
    
    @Autowired
    private InventoryServiceClient inventoryServiceClient;
    
    /**
     * 创建订单的分布式事务
     * 需要协调多个服务的数据一致性
     */
    @Transactional
    public OrderResult createOrderWithTransaction(CreateOrderRequest request) {
        
        String transactionId = UUID.randomUUID().toString();
        
        try {
            // 1. 扣减库存（Try阶段）
            InventoryResult inventoryResult = inventoryServiceClient.reserveInventory(
                request.getProductId(), request.getQuantity(), transactionId);
            
            if (!inventoryResult.isSuccess()) {
                throw new BusinessException("库存不足");
            }
            
            // 2. 创建订单
            Order order = orderService.createOrder(request);
            
            // 3. 处理支付（Try阶段）
            PaymentResult paymentResult = paymentServiceClient.processPayment(
                order.getId(), request.getAmount(), transactionId);
            
            if (!paymentResult.isSuccess()) {
                // 补偿：释放库存
                inventoryServiceClient.releaseInventory(
                    request.getProductId(), request.getQuantity(), transactionId);
                throw new BusinessException("支付失败");
            }
            
            // 4. 确认所有操作（Confirm阶段）
            inventoryServiceClient.confirmInventory(transactionId);
            paymentServiceClient.confirmPayment(transactionId);
            
            return OrderResult.success(order);
            
        } catch (Exception e) {
            // 补偿所有操作（Cancel阶段）
            try {
                inventoryServiceClient.cancelInventory(transactionId);
                paymentServiceClient.cancelPayment(transactionId);
            } catch (Exception compensationException) {
                // 记录补偿失败，需要人工介入
                System.err.println("补偿操作失败: " + compensationException.getMessage());
            }
            
            return OrderResult.failure(e.getMessage());
        }
    }
}

// 3. 网络延迟和故障处理

/**
 * 网络调用重试和超时配置
 */
@Component
public class ResilientServiceClient {
    
    @Autowired
    private RestTemplate restTemplate;
    
    /**
     * 带重试机制的服务调用
     */
    @Retryable(
        value = {ConnectException.class, SocketTimeoutException.class},
        maxAttempts = 3,
        backoff = @Backoff(delay = 1000, multiplier = 2)
    )
    public String callExternalService(String serviceUrl, Object request) {
        
        try {
            // 设置超时时间
            HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory();
            factory.setConnectTimeout(3000); // 连接超时3秒
            factory.setReadTimeout(5000);    // 读取超时5秒
            restTemplate.setRequestFactory(factory);
            
            return restTemplate.postForObject(serviceUrl, request, String.class);
            
        } catch (Exception e) {
            System.err.println("服务调用失败: " + serviceUrl + ", 错误: " + e.getMessage());
            throw e;
        }
    }
    
    /**
     * 重试失败后的降级处理
     */
    @Recover
    public String recover(Exception ex, String serviceUrl, Object request) {
        System.err.println("服务调用最终失败，启用降级: " + serviceUrl);
        return "服务暂时不可用，请稍后重试";
    }
}

// 4. 运维复杂度示例

/**
 * 微服务监控和健康检查
 */
@Component
public class MicroserviceHealthIndicator implements HealthIndicator {
    
    @Autowired
    private List<ServiceClient> serviceClients;
    
    @Override
    public Health health() {
        
        Health.Builder builder = Health.up();
        Map<String, Object> details = new HashMap<>();
        
        // 检查所有依赖服务的健康状态
        for (ServiceClient client : serviceClients) {
            try {
                boolean isHealthy = client.healthCheck();
                details.put(client.getServiceName(), isHealthy ? "UP" : "DOWN");
                
                if (!isHealthy) {
                    builder.down();
                }
                
            } catch (Exception e) {
                details.put(client.getServiceName(), "ERROR: " + e.getMessage());
                builder.down();
            }
        }
        
        return builder.withDetails(details).build();
    }
}

/**
 * 分布式日志聚合
 */
@Component
public class DistributedLoggingService {
    
    private static final Logger logger = LoggerFactory.getLogger(DistributedLoggingService.class);
    
    /**
     * 记录分布式调用链日志
     */
    public void logServiceCall(String serviceName, String operation, 
                              String traceId, long duration, boolean success) {
        
        Map<String, Object> logData = new HashMap<>();
        logData.put("service", serviceName);
        logData.put("operation", operation);
        logData.put("traceId", traceId);
        logData.put("duration", duration);
        logData.put("success", success);
        logData.put("timestamp", LocalDateTime.now());
        
        // 使用结构化日志格式，便于日志聚合系统处理
        logger.info("SERVICE_CALL: {}", new ObjectMapper().writeValueAsString(logData));
    }
}
```

**微服务架构适用场景：**

| 场景 | 适用性 | 原因 | 示例 |
|------|--------|------|------|
| **大型企业应用** | 高 | 复杂业务需要拆分，团队规模大 | 电商平台、金融系统 |
| **高并发系统** | 高 | 可独立扩展热点服务 | 社交媒体、视频平台 |
| **快速迭代项目** | 中 | 独立部署加快发布速度 | 互联网产品 |
| **小型项目** | 低 | 增加不必要的复杂性 | 企业内部工具 |
| **单一团队** | 低 | 团队协调成本低，单体更简单 | 初创公司产品 |

**微服务架构演进路径：**

```
单体应用 → 模块化单体 → 微服务架构
    ↓           ↓            ↓
简单快速    逐步解耦    完全分离
```

**总结：**

微服务架构是一把双刃剑，它带来了灵活性和可扩展性，但也增加了系统复杂性。选择微服务架构需要考虑：
1. **团队规模**：是否有足够的团队支撑多个服务
2. **业务复杂度**：业务是否复杂到需要拆分
3. **技术能力**：团队是否具备分布式系统开发能力
4. **运维能力**：是否有完善的DevOps和监控体系
5. **业务发展阶段**：是否到了需要快速扩展的阶段

**Q2: Spring Cloud中的服务注册与发现是如何工作的？**

**答案：**

Spring Cloud使用Eureka作为服务注册中心，实现服务的自动注册和发现。这是微服务架构中的核心组件，解决了服务实例动态变化和服务间通信的问题。

**服务注册与发现架构：**

```
┌─────────────────┐    注册服务    ┌─────────────────┐
│   服务提供者A    │ ──────────→   │  Eureka Server  │
│   (8081)       │               │   (8761)       │
└─────────────────┘               └─────────────────┘
                                          ↑
┌─────────────────┐    注册服务           │
│   服务提供者B    │ ──────────────────────┘
│   (8082)       │               
└─────────────────┘               
                                  
┌─────────────────┐    获取服务列表 ┌─────────────────┐
│   服务消费者     │ ←──────────→  │  Eureka Server  │
│   (8080)       │    服务调用    │   (8761)       │
└─────────────────┘               └─────────────────┘
```

**详细实现步骤：**

**1. Eureka Server配置：**

```java
/**
 * Eureka注册中心服务器
 */
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
        System.out.println("Eureka注册中心启动成功 - 端口: 8761");
    }
}
```

```yaml
# application.yml - Eureka Server配置
server:
  port: 8761

spring:
  application:
    name: eureka-server

eureka:
  instance:
    hostname: localhost
  client:
    # 不向注册中心注册自己
    register-with-eureka: false
    # 不从注册中心获取服务列表
    fetch-registry: false
    service-url:
      defaultZone: http://localhost:8761/eureka/
  server:
    # 关闭自我保护模式（开发环境）
    enable-self-preservation: false
    # 清理间隔（毫秒）
    eviction-interval-timer-in-ms: 5000
```

**2. 服务提供者配置：**

```java
/**
 * 用户服务提供者
 */
@SpringBootApplication
@EnableEurekaClient
public class UserServiceApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}

@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Value("${server.port}")
    private String port;
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = new User();
        user.setId(id);
        user.setUsername("user" + id);
        user.setPort(port); // 标识服务实例
        
        System.out.println("用户服务[" + port + "]处理请求: " + id);
        return ResponseEntity.ok(user);
    }
}
```

```yaml
# application.yml - 用户服务配置
server:
  port: 8081

spring:
  application:
    name: user-service

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
    instance-id: ${spring.application.name}:${server.port}
    lease-renewal-interval-in-seconds: 10
    lease-expiration-duration-in-seconds: 30
```

**3. 服务消费者配置：**

```java
/**
 * 订单服务（服务消费者）
 */
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
public class OrderServiceApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}

/**
 * 负载均衡配置
 */
@Configuration
public class LoadBalanceConfig {
    
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

/**
 * 使用RestTemplate调用服务
 */
@Service
public class UserServiceClient {
    
    @Autowired
    private RestTemplate restTemplate;
    
    public User getUserById(Long userId) {
        String url = "http://user-service/api/users/" + userId;
        return restTemplate.getForObject(url, User.class);
    }
}

/**
 * 使用Feign声明式调用
 */
@FeignClient(name = "user-service")
public interface UserServiceFeign {
    
    @GetMapping("/api/users/{id}")
    User getUser(@PathVariable("id") Long id);
}

@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    @Autowired
    private UserServiceClient userServiceClient;
    
    @Autowired
    private UserServiceFeign userServiceFeign;
    
    @PostMapping("/resttemplate")
    public ResponseEntity<Order> createOrderWithRestTemplate(@RequestBody CreateOrderRequest request) {
        User user = userServiceClient.getUserById(request.getUserId());
        
        Order order = new Order();
        order.setUserId(user.getId());
        order.setUserName(user.getUsername());
        order.setServicePort(user.getPort());
        
        return ResponseEntity.ok(order);
    }
    
    @PostMapping("/feign")
    public ResponseEntity<Order> createOrderWithFeign(@RequestBody CreateOrderRequest request) {
        User user = userServiceFeign.getUser(request.getUserId());
        
        Order order = new Order();
        order.setUserId(user.getId());
        order.setUserName(user.getUsername());
        order.setServicePort(user.getPort());
        
        return ResponseEntity.ok(order);
    }
}
```

**工作流程详解：**

| 步骤 | 操作 | 说明 | 时间间隔 |
|------|------|------|----------|
| **1. 服务注册** | 服务启动时向Eureka注册 | 发送服务元数据（IP、端口、健康检查URL等） | 启动时 |
| **2. 服务续约** | 定期发送心跳 | 证明服务仍然可用 | 30秒（默认） |
| **3. 服务发现** | 客户端获取服务列表 | 从Eureka获取可用服务实例 | 30秒（默认） |
| **4. 服务调用** | 负载均衡选择实例 | 使用Ribbon进行客户端负载均衡 | 每次调用 |
| **5. 服务下线** | 正常关闭时注销服务 | 主动通知Eureka移除服务 | 关闭时 |

**负载均衡策略：**

```java
/**
 * 自定义负载均衡策略
 */
@Configuration
public class RibbonConfig {
    
    @Bean
    public IRule ribbonRule() {
        // 轮询策略（默认）
        return new RoundRobinRule();
        
        // 其他策略：
        // return new RandomRule(); // 随机
        // return new WeightedResponseTimeRule(); // 响应时间加权
        // return new BestAvailableRule(); // 最少并发
        // return new RetryRule(); // 重试
    }
}
```

**服务发现时序图：**

```
服务提供者    Eureka Server    服务消费者
    │              │              │
    │─────注册────→│              │
    │              │              │
    │←────确认─────│              │
    │              │              │
    │──定期心跳───→│              │
    │              │              │
    │              │←──获取服务列表──│
    │              │              │
    │              │─────返回─────→│
    │              │              │
    │←─────────直接调用───────────│
    │              │              │
    │─────响应────────────────────→│
```

**关键特性：**

1. **自我保护机制**：当网络分区时，Eureka会保留服务注册信息
2. **客户端缓存**：客户端本地缓存服务列表，提高可用性
3. **增量同步**：只同步变化的服务信息，提高效率
4. **多级缓存**：ReadOnlyMap和ReadWriteMap提高性能

**最佳实践：**

1. **生产环境配置**：
   - 启用Eureka集群
   - 配置合适的心跳和超时时间
   - 启用安全认证

2. **服务治理**：
   - 实现健康检查端点
   - 配置服务元数据
   - 监控服务注册状态

3. **故障处理**：
   - 实现服务降级
   - 配置重试机制
   - 使用熔断器保护

**总结：**

Spring Cloud的服务注册与发现通过Eureka提供了完整的微服务治理能力，包括自动注册、健康检查、负载均衡和故障转移，是构建可靠微服务架构的基础组件。

**Q3: 什么是API网关？它解决了什么问题？**

**答案：**

API网关是微服务架构中的统一入口点，作为客户端和后端服务之间的中间层，提供路由、安全、监控、限流等功能。它是微服务架构中不可或缺的基础设施组件。

**API网关架构图：**

```
┌─────────────┐    ┌─────────────────────────────────┐    ┌─────────────┐
│  移动客户端  │    │                                │    │  用户服务    │
└─────────────┘    │                                │    │  (8081)    │
       │           │          API Gateway           │    └─────────────┘
┌─────────────┐    │        (Spring Cloud          │           │
│  Web客户端   │────│         Gateway)              │───────────┤
└─────────────┘    │                                │    ┌─────────────┐
       │           │  ┌─────────┐  ┌─────────────┐  │    │  订单服务    │
┌─────────────┐    │  │ 路由转发 │  │ 认证授权     │  │    │  (8082)    │
│ 第三方系统   │────│  │ 负载均衡 │  │ 限流熔断     │  │    └─────────────┘
└─────────────┘    │  │ 协议转换 │  │ 监控日志     │  │           │
                   │  └─────────┘  └─────────────┘  │    ┌─────────────┐
                   │                                │    │  支付服务    │
                   └─────────────────────────────────┘    │  (8083)    │
                                                          └─────────────┘
```

**API网关解决的核心问题：**

| 问题 | 传统方式 | API网关方式 | 优势 |
|------|----------|-------------|------|
| **服务调用复杂** | 客户端直接调用多个服务 | 统一入口点 | 简化客户端逻辑 |
| **安全控制分散** | 每个服务独立认证 | 统一认证授权 | 安全策略一致性 |
| **跨域问题** | 每个服务处理CORS | 网关统一处理 | 简化跨域配置 |
| **协议不统一** | 客户端适配多种协议 | 协议转换 | 客户端协议统一 |
| **监控困难** | 分散的监控点 | 集中监控 | 统一的可观测性 |

**Spring Cloud Gateway详细实现：**

**1. 网关服务配置：**

```java
/**
 * API网关应用
 */
@SpringBootApplication
@EnableEurekaClient
public class GatewayApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
        System.out.println("API网关启动成功 - 端口: 8080");
    }
}

/**
 * 网关路由配置
 */
@Configuration
public class GatewayConfig {
    
    /**
     * 编程式路由配置
     */
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            // 用户服务路由
            .route("user-service", r -> r
                .path("/api/users/**")
                .filters(f -> f
                    .addRequestHeader("X-Gateway", "Spring-Cloud-Gateway")
                    .addResponseHeader("X-Response-Time", String.valueOf(System.currentTimeMillis()))
                    .circuitBreaker(config -> config
                        .setName("user-service-cb")
                        .setFallbackUri("forward:/fallback/user"))
                    .retry(config -> config
                        .setRetries(3)
                        .setMethods(HttpMethod.GET)
                        .setBackoff(Duration.ofMillis(100), Duration.ofMillis(1000), 2, false)))
                .uri("lb://user-service"))
            
            // 订单服务路由
            .route("order-service", r -> r
                .path("/api/orders/**")
                .filters(f -> f
                    .addRequestHeader("X-Gateway", "Spring-Cloud-Gateway")
                    .requestRateLimiter(config -> config
                        .setRateLimiter(redisRateLimiter())
                        .setKeyResolver(ipKeyResolver()))
                    .circuitBreaker(config -> config
                        .setName("order-service-cb")
                        .setFallbackUri("forward:/fallback/order")))
                .uri("lb://order-service"))
            
            // 支付服务路由（需要认证）
            .route("payment-service", r -> r
                .path("/api/payments/**")
                .filters(f -> f
                    .addRequestHeader("X-Gateway", "Spring-Cloud-Gateway")
                    .filter(authenticationFilter())
                    .circuitBreaker(config -> config
                        .setName("payment-service-cb")
                        .setFallbackUri("forward:/fallback/payment")))
                .uri("lb://payment-service"))
            
            .build();
    }
    
    /**
     * Redis限流器
     */
    @Bean
    public RedisRateLimiter redisRateLimiter() {
        return new RedisRateLimiter(10, 20); // 每秒10个请求，突发20个
    }
    
    /**
     * IP限流键解析器
     */
    @Bean
    public KeyResolver ipKeyResolver() {
        return exchange -> Mono.just(exchange.getRequest().getRemoteAddress().getAddress().getHostAddress());
    }
    
    /**
     * 用户限流键解析器
     */
    @Bean
    public KeyResolver userKeyResolver() {
        return exchange -> exchange.getPrincipal()
            .cast(JwtAuthenticationToken.class)
            .map(JwtAuthenticationToken::getToken)
            .map(jwt -> jwt.getClaimAsString("sub"))
            .switchIfEmpty(Mono.just("anonymous"));
    }
    
    /**
     * 自定义认证过滤器
     */
    @Bean
    public GatewayFilter authenticationFilter() {
        return new AuthenticationGatewayFilter();
    }
}

/**
 * 自定义认证过滤器
 */
public class AuthenticationGatewayFilter implements GatewayFilter {
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        
        ServerHttpRequest request = exchange.getRequest();
        
        // 获取Authorization头
        String authHeader = request.getHeaders().getFirst("Authorization");
        
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            return handleUnauthorized(exchange);
        }
        
        String token = authHeader.substring(7);
        
        // 验证JWT Token
        try {
            if (validateToken(token)) {
                // 添加用户信息到请求头
                ServerHttpRequest modifiedRequest = request.mutate()
                    .header("X-User-Id", getUserIdFromToken(token))
                    .header("X-User-Role", getUserRoleFromToken(token))
                    .build();
                
                return chain.filter(exchange.mutate().request(modifiedRequest).build());
            } else {
                return handleUnauthorized(exchange);
            }
        } catch (Exception e) {
            return handleUnauthorized(exchange);
        }
    }
    
    private Mono<Void> handleUnauthorized(ServerWebExchange exchange) {
        ServerHttpResponse response = exchange.getResponse();
        response.setStatusCode(HttpStatus.UNAUTHORIZED);
        response.getHeaders().add("Content-Type", "application/json");
        
        String body = "{\"error\":\"Unauthorized\",\"message\":\"Invalid or missing token\"}";
        DataBuffer buffer = response.bufferFactory().wrap(body.getBytes());
        
        return response.writeWith(Mono.just(buffer));
    }
    
    private boolean validateToken(String token) {
        // JWT token验证逻辑
        try {
            Jwts.parser()
                .setSigningKey("your-secret-key")
                .parseClaimsJws(token);
            return true;
        } catch (Exception e) {
            return false;
        }
    }
    
    private String getUserIdFromToken(String token) {
        Claims claims = Jwts.parser()
            .setSigningKey("your-secret-key")
            .parseClaimsJws(token)
            .getBody();
        return claims.getSubject();
    }
    
    private String getUserRoleFromToken(String token) {
        Claims claims = Jwts.parser()
            .setSigningKey("your-secret-key")
            .parseClaimsJws(token)
            .getBody();
        return claims.get("role", String.class);
    }
}

/**
 * 全局异常处理
 */
@RestController
public class FallbackController {
    
    @RequestMapping("/fallback/user")
    public Mono<ResponseEntity<Map<String, Object>>> userFallback() {
        Map<String, Object> response = new HashMap<>();
        response.put("error", "用户服务暂时不可用");
        response.put("message", "请稍后重试");
        response.put("timestamp", LocalDateTime.now());
        
        return Mono.just(ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE).body(response));
    }
    
    @RequestMapping("/fallback/order")
    public Mono<ResponseEntity<Map<String, Object>>> orderFallback() {
        Map<String, Object> response = new HashMap<>();
        response.put("error", "订单服务暂时不可用");
        response.put("message", "请稍后重试");
        response.put("timestamp", LocalDateTime.now());
        
        return Mono.just(ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE).body(response));
    }
    
    @RequestMapping("/fallback/payment")
    public Mono<ResponseEntity<Map<String, Object>>> paymentFallback() {
        Map<String, Object> response = new HashMap<>();
        response.put("error", "支付服务暂时不可用");
        response.put("message", "请稍后重试");
        response.put("timestamp", LocalDateTime.now());
        
        return Mono.just(ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE).body(response));
    }
}

/**
 * 全局过滤器 - 请求日志
 */
@Component
public class RequestLoggingFilter implements GlobalFilter, Ordered {
    
    private static final Logger logger = LoggerFactory.getLogger(RequestLoggingFilter.class);
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        
        ServerHttpRequest request = exchange.getRequest();
        String requestId = UUID.randomUUID().toString();
        
        // 记录请求开始
        long startTime = System.currentTimeMillis();
        
        logger.info("请求开始 - ID: {}, Method: {}, URI: {}, IP: {}", 
            requestId, 
            request.getMethod(), 
            request.getURI(), 
            request.getRemoteAddress());
        
        // 添加请求ID到响应头
        exchange.getResponse().getHeaders().add("X-Request-Id", requestId);
        
        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            long endTime = System.currentTimeMillis();
            long duration = endTime - startTime;
            
            logger.info("请求结束 - ID: {}, Duration: {}ms, Status: {}", 
                requestId, 
                duration, 
                exchange.getResponse().getStatusCode());
        }));
    }
    
    @Override
    public int getOrder() {
        return -1; // 最高优先级
    }
}
```

```yaml
# application.yml - 网关配置
server:
  port: 8080

spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      # 发现服务路由
      discovery:
        locator:
          enabled: true
          lower-case-service-id: true
      # 全局CORS配置
      globalcors:
        cors-configurations:
          '[/**]':
            allowed-origins: "*"
            allowed-methods:
              - GET
              - POST
              - PUT
              - DELETE
              - OPTIONS
            allowed-headers: "*"
            allow-credentials: true
      # 默认过滤器
      default-filters:
        - AddResponseHeader=X-Gateway, Spring-Cloud-Gateway
        - AddResponseHeader=X-Response-Default, Default-Value
  redis:
    host: localhost
    port: 6379
    database: 0

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true

# 日志配置
logging:
  level:
    org.springframework.cloud.gateway: DEBUG
    org.springframework.web.reactive: DEBUG
```

**API网关的核心功能：**

**1. 路由转发：**
```java
// 基于路径的路由
.route("user-service", r -> r.path("/api/users/**").uri("lb://user-service"))

// 基于请求头的路由
.route("mobile-api", r -> r.header("X-Client-Type", "mobile").uri("lb://mobile-service"))

// 基于查询参数的路由
.route("version-api", r -> r.query("version", "v2").uri("lb://api-v2-service"))
```

**2. 负载均衡：**
```java
// 使用Ribbon负载均衡
.uri("lb://service-name")

// 自定义负载均衡策略
@Bean
public ReactorLoadBalancer<ServiceInstance> reactorServiceInstanceLoadBalancer(
        Environment environment, LoadBalancerClientFactory loadBalancerClientFactory) {
    String name = environment.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);
    return new RoundRobinLoadBalancer(
        loadBalancerClientFactory.getLazyProvider(name, ServiceInstanceListSupplier.class), name);
}
```

**3. 限流控制：**
```java
// Redis限流
.requestRateLimiter(config -> config
    .setRateLimiter(redisRateLimiter())
    .setKeyResolver(ipKeyResolver()))

// 自定义限流策略
@Component
public class CustomRateLimitFilter implements GatewayFilter {
    
    private final Map<String, AtomicInteger> requestCounts = new ConcurrentHashMap<>();
    private final int maxRequests = 100;
    private final Duration timeWindow = Duration.ofMinutes(1);
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String clientId = getClientId(exchange.getRequest());
        
        if (isRateLimited(clientId)) {
            return handleRateLimited(exchange);
        }
        
        return chain.filter(exchange);
    }
}
```

**API网关对比：**

| 特性 | Spring Cloud Gateway | Zuul | Kong | Nginx |
|------|---------------------|------|------|-------|
| **性能** | 高（异步非阻塞） | 中（同步阻塞） | 高 | 极高 |
| **扩展性** | 强（Java生态） | 强（Java生态） | 强（插件） | 中 |
| **配置方式** | 代码+配置文件 | 代码+配置文件 | 配置文件 | 配置文件 |
| **监控** | 丰富 | 丰富 | 丰富 | 基础 |
| **学习成本** | 中 | 低 | 中 | 低 |

**最佳实践：**

1. **性能优化**：
   - 使用连接池
   - 启用响应缓存
   - 合理配置超时时间

2. **安全加固**：
   - 实现统一认证
   - 配置HTTPS
   - 防止SQL注入和XSS攻击

3. **监控告警**：
   - 集成APM工具
   - 配置健康检查
   - 设置关键指标告警

4. **高可用部署**：
   - 多实例部署
   - 配置负载均衡
   - 实现故障转移

**总结：**

API网关是微服务架构的重要组件，它解决了服务调用复杂性、安全控制、协议转换等关键问题，提供了统一的服务入口和管理能力，是构建可靠微服务系统的基础设施。

### 2. 分布式缓存相关

**Q4: Redis的数据类型有哪些？分别适用于什么场景？**

**答案：**

Redis支持丰富的数据类型，每种类型都有其特定的应用场景和优化的操作命令。合理选择数据类型是Redis性能优化的关键。

**Redis数据类型全览：**

```
┌─────────────────────────────────────────────────────────────┐
│                    Redis 数据类型                           │
├─────────────┬─────────────┬─────────────┬─────────────────┤
│   基础类型   │   集合类型   │   特殊类型   │     流类型       │
├─────────────┼─────────────┼─────────────┼─────────────────┤
│   String    │    Hash     │   Bitmap    │     Stream      │
│             │    List     │ HyperLogLog │                 │
│             │    Set      │  Geospatial │                 │
│             │    ZSet     │             │                 │
└─────────────┴─────────────┴─────────────┴─────────────────┘
```

**详细数据类型分析：**

### **1. String（字符串）**

**特点：** 最基础的数据类型，可以存储字符串、数字、二进制数据

**应用场景与实现：**

```java
/**
 * String类型应用示例
 */
@Service
public class StringRedisService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 1. 缓存应用 - 用户信息缓存
     */
    public void cacheUserInfo(Long userId, User user) {
        String key = "user:info:" + userId;
        String userJson = JSON.toJSONString(user);
        redisTemplate.opsForValue().set(key, userJson, Duration.ofHours(1));
        System.out.println("用户信息已缓存: " + key);
    }
    
    /**
     * 2. 计数器应用 - 文章浏览量
     */
    public Long incrementViewCount(Long articleId) {
        String key = "article:views:" + articleId;
        return redisTemplate.opsForValue().increment(key);
    }
    
    /**
     * 3. 分布式锁应用
     */
    public boolean tryLock(String lockKey, String lockValue, Duration expireTime) {
        Boolean result = redisTemplate.opsForValue().setIfAbsent(lockKey, lockValue, expireTime);
        return Boolean.TRUE.equals(result);
    }
    
    public void releaseLock(String lockKey, String lockValue) {
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then " +
                       "return redis.call('del', KEYS[1]) " +
                       "else return 0 end";
        
        redisTemplate.execute(new DefaultRedisScript<>(script, Long.class), 
                            Collections.singletonList(lockKey), lockValue);
    }
}
```

### **2. Hash（哈希）**

**特点：** 键值对集合，适合存储对象

```java
/**
 * Hash类型应用示例
 */
@Service
public class HashRedisService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 1. 用户信息存储 - 避免序列化开销
     */
    public void saveUserProfile(Long userId, UserProfile profile) {
        String key = "user:profile:" + userId;
        
        Map<String, Object> profileMap = new HashMap<>();
        profileMap.put("nickname", profile.getNickname());
        profileMap.put("avatar", profile.getAvatar());
        profileMap.put("age", profile.getAge());
        profileMap.put("gender", profile.getGender());
        
        redisTemplate.opsForHash().putAll(key, profileMap);
        redisTemplate.expire(key, Duration.ofDays(7));
    }
    
    /**
     * 2. 购物车实现
     */
    public void addToCart(Long userId, Long productId, Integer quantity) {
        String key = "cart:" + userId;
        redisTemplate.opsForHash().put(key, productId.toString(), quantity);
        redisTemplate.expire(key, Duration.ofDays(30));
    }
    
    public Map<String, Integer> getCartItems(Long userId) {
        String key = "cart:" + userId;
        Map<Object, Object> cartMap = redisTemplate.opsForHash().entries(key);
        
        Map<String, Integer> result = new HashMap<>();
        cartMap.forEach((k, v) -> result.put(k.toString(), Integer.valueOf(v.toString())));
        return result;
    }
}
```

### **3. List（列表）**

**特点：** 有序的字符串列表，支持双端操作

```java
/**
 * List类型应用示例
 */
@Service
public class ListRedisService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 1. 消息队列实现
     */
    public void sendMessage(String queueName, Object message) {
        String messageJson = JSON.toJSONString(message);
        redisTemplate.opsForList().leftPush(queueName, messageJson);
        System.out.println("消息已发送到队列: " + queueName);
    }
    
    public <T> T receiveMessage(String queueName, Class<T> messageType, Duration timeout) {
        List<Object> result = redisTemplate.opsForList().rightPop(queueName, timeout);
        
        if (result != null && !result.isEmpty()) {
            String messageJson = result.get(0).toString();
            return JSON.parseObject(messageJson, messageType);
        }
        return null;
    }
    
    /**
     * 2. 最新动态列表
     */
    public void addUserActivity(Long userId, UserActivity activity) {
        String key = "user:activities:" + userId;
        String activityJson = JSON.toJSONString(activity);
        
        redisTemplate.opsForList().leftPush(key, activityJson);
        redisTemplate.opsForList().trim(key, 0, 99); // 保持列表长度不超过100
        redisTemplate.expire(key, Duration.ofDays(30));
    }
}
```

### **4. Set（集合）**

**特点：** 无序的唯一字符串集合

```java
/**
 * Set类型应用示例
 */
@Service
public class SetRedisService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 1. 标签系统
     */
    public void addUserTags(Long userId, Set<String> tags) {
        String key = "user:tags:" + userId;
        redisTemplate.opsForSet().add(key, tags.toArray());
        redisTemplate.expire(key, Duration.ofDays(30));
    }
    
    public Set<String> getCommonTags(Long userId1, Long userId2) {
        String key1 = "user:tags:" + userId1;
        String key2 = "user:tags:" + userId2;
        
        Set<Object> commonTags = redisTemplate.opsForSet().intersect(key1, key2);
        return commonTags.stream().map(Object::toString).collect(Collectors.toSet());
    }
    
    /**
     * 2. 去重应用 - 今日访问用户
     */
    public void recordDailyVisitor(String date, Long userId) {
        String key = "daily:visitors:" + date;
        redisTemplate.opsForSet().add(key, userId.toString());
        redisTemplate.expire(key, Duration.ofDays(7));
    }
    
    public Long getDailyVisitorCount(String date) {
        String key = "daily:visitors:" + date;
        return redisTemplate.opsForSet().size(key);
    }
}
```

### **5. ZSet（有序集合）**

**特点：** 有序的唯一字符串集合，每个元素关联一个分数

```java
/**
 * ZSet类型应用示例
 */
@Service
public class ZSetRedisService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 1. 排行榜系统
     */
    public void updateUserScore(Long userId, double score) {
        String key = "leaderboard:global";
        redisTemplate.opsForZSet().add(key, userId.toString(), score);
    }
    
    public List<LeaderboardEntry> getTopUsers(int limit) {
        String key = "leaderboard:global";
        Set<ZSetOperations.TypedTuple<Object>> topUsers = 
            redisTemplate.opsForZSet().reverseRangeWithScores(key, 0, limit - 1);
        
        List<LeaderboardEntry> result = new ArrayList<>();
        int rank = 1;
        
        for (ZSetOperations.TypedTuple<Object> tuple : topUsers) {
            LeaderboardEntry entry = new LeaderboardEntry();
            entry.setUserId(Long.valueOf(tuple.getValue().toString()));
            entry.setScore(tuple.getScore());
            entry.setRank(rank++);
            result.add(entry);
        }
        return result;
    }
    
    /**
     * 2. 延时队列实现
     */
    public void addDelayedTask(String taskId, Object taskData, long delaySeconds) {
        String key = "delayed:tasks";
        long executeTime = System.currentTimeMillis() + (delaySeconds * 1000);
        
        DelayedTask task = new DelayedTask(taskId, taskData, executeTime);
        String taskJson = JSON.toJSONString(task);
        
        redisTemplate.opsForZSet().add(key, taskJson, executeTime);
    }
    
    public List<DelayedTask> getReadyTasks(int limit) {
        String key = "delayed:tasks";
        long currentTime = System.currentTimeMillis();
        
        Set<Object> readyTasks = redisTemplate.opsForZSet()
            .rangeByScore(key, 0, currentTime, 0, limit);
        
        List<DelayedTask> result = new ArrayList<>();
        for (Object taskObj : readyTasks) {
            DelayedTask task = JSON.parseObject(taskObj.toString(), DelayedTask.class);
            result.add(task);
            redisTemplate.opsForZSet().remove(key, taskObj);
        }
        return result;
    }
}
```

### **6. 特殊数据类型**

**Bitmap（位图）：**
```java
// 用户签到统计
public void userSignIn(Long userId, int day) {
    String key = "signin:" + userId + ":" + LocalDate.now().getYear();
    redisTemplate.opsForValue().setBit(key, day, true);
}

public boolean hasSignedIn(Long userId, int day) {
    String key = "signin:" + userId + ":" + LocalDate.now().getYear();
    return redisTemplate.opsForValue().getBit(key, day);
}
```

**HyperLogLog（基数统计）：**
```java
// UV统计
public void recordUV(String page, String userId) {
    String key = "uv:" + page + ":" + LocalDate.now();
    redisTemplate.opsForHyperLogLog().add(key, userId);
}

public Long getUV(String page) {
    String key = "uv:" + page + ":" + LocalDate.now();
    return redisTemplate.opsForHyperLogLog().size(key);
}
```

**数据类型选择指南：**

| 应用场景 | 推荐类型 | 原因 | 示例 |
|----------|----------|------|------|
| **简单缓存** | String | 操作简单，性能高 | 用户信息、配置数据 |
| **对象存储** | Hash | 避免序列化，支持部分更新 | 用户资料、商品信息 |
| **消息队列** | List | 支持FIFO，阻塞操作 | 任务队列、通知消息 |
| **去重统计** | Set | 自动去重，集合运算 | 访问统计、标签系统 |
| **排行榜** | ZSet | 自动排序，范围查询 | 积分排行、热门内容 |
| **计数器** | String | 原子操作，高性能 | 点赞数、浏览量 |
| **签到统计** | Bitmap | 内存效率高 | 用户签到、活跃统计 |
| **UV统计** | HyperLogLog | 内存占用小 | 页面访问量统计 |

**性能优化建议：**

1. **合理选择数据类型**：根据业务场景选择最适合的数据类型
2. **控制数据大小**：避免单个key存储过大的数据
3. **设置过期时间**：防止内存泄漏
4. **使用批量操作**：减少网络往返次数
5. **避免大key操作**：可能阻塞Redis服务

**总结：**

Redis的丰富数据类型为不同的应用场景提供了优化的解决方案。正确选择和使用数据类型，能够显著提升应用性能和开发效率。

**Q5: 什么是缓存穿透、缓存击穿、缓存雪崩？如何解决？**

**答案：**

这三种情况是Redis缓存使用中最常见的问题，会严重影响系统性能和稳定性。理解其原理和解决方案对于构建高可用缓存系统至关重要。

**问题对比分析：**

| 问题类型 | 定义 | 影响 | 根本原因 | 解决难度 |
|----------|------|------|----------|----------|
| **缓存穿透** | 查询不存在的数据 | 数据库压力大 | 恶意攻击或业务逻辑问题 | 中等 |
| **缓存击穿** | 热点数据过期 | 瞬时数据库压力 | 并发访问热点数据 | 较难 |
| **缓存雪崩** | 大量缓存同时失效 | 系统整体崩溃 | 缓存设计不合理 | 困难 |

### **1. 缓存穿透（Cache Penetration）**

**问题描述：** 查询一个不存在的数据，缓存和数据库都没有，每次请求都会打到数据库

**详细解决方案：**

```java
/**
 * 缓存穿透解决方案
 */
@Service
public class CachePenetrationSolution {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Autowired
    private UserRepository userRepository;
    
    // 布隆过滤器
    private BloomFilter<Long> bloomFilter;
    
    @PostConstruct
    public void initBloomFilter() {
        // 初始化布隆过滤器，预期插入100万个元素，误判率0.01%
        bloomFilter = BloomFilter.create(
            Funnels.longFunnel(), 
            1000000, 
            0.0001
        );
        
        // 将所有存在的用户ID加入布隆过滤器
        List<Long> existingUserIds = userRepository.findAllUserIds();
        existingUserIds.forEach(bloomFilter::put);
        
        System.out.println("布隆过滤器初始化完成，已加载 " + existingUserIds.size() + " 个用户ID");
    }
    
    /**
     * 方案1：布隆过滤器 + 缓存空值
     */
    public User getUserById(Long userId) {
        // 1. 布隆过滤器预检查
        if (!bloomFilter.mightContain(userId)) {
            System.out.println("布隆过滤器判断用户不存在: " + userId);
            return null;
        }
        
        String cacheKey = "user:" + userId;
        
        // 2. 查询缓存
        String userJson = (String) redisTemplate.opsForValue().get(cacheKey);
        if (userJson != null) {
            if ("NULL".equals(userJson)) {
                // 缓存的空值
                return null;
            }
            return JSON.parseObject(userJson, User.class);
        }
        
        // 3. 查询数据库
        User user = userRepository.findById(userId);
        
        if (user != null) {
            // 缓存真实数据
            redisTemplate.opsForValue().set(cacheKey, JSON.toJSONString(user), Duration.ofHours(1));
        } else {
            // 缓存空值，设置较短的过期时间
            redisTemplate.opsForValue().set(cacheKey, "NULL", Duration.ofMinutes(5));
        }
        
        return user;
    }
    
    /**
     * 方案2：参数校验 + 白名单
     */
    public User getUserByIdWithValidation(Long userId) {
        // 参数校验
        if (userId == null || userId <= 0) {
            throw new IllegalArgumentException("用户ID不能为空或小于等于0");
        }
        
        // ID范围校验（假设用户ID在合理范围内）
        if (userId > 10000000L) {
            System.out.println("用户ID超出合理范围: " + userId);
            return null;
        }
        
        return getUserById(userId);
    }
    
    /**
     * 方案3：限流 + 监控
     */
    @RateLimiter(name = "getUserById", fallbackMethod = "getUserByIdFallback")
    public User getUserByIdWithRateLimit(Long userId) {
        return getUserById(userId);
    }
    
    public User getUserByIdFallback(Long userId, Exception ex) {
        System.out.println("触发限流，返回默认用户信息: " + userId);
        return new User(userId, "默认用户", "default@example.com");
    }
}
```

### **2. 缓存击穿（Cache Breakdown）**

**问题描述：** 热点数据过期的瞬间，大量并发请求同时访问数据库

```java
/**
 * 缓存击穿解决方案
 */
@Service
public class CacheBreakdownSolution {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Autowired
    private ProductRepository productRepository;
    
    // 本地锁映射
    private final ConcurrentHashMap<String, Object> localLocks = new ConcurrentHashMap<>();
    
    /**
     * 方案1：分布式锁
     */
    public Product getHotProduct(Long productId) {
        String cacheKey = "hot:product:" + productId;
        String lockKey = "lock:product:" + productId;
        
        // 1. 查询缓存
        String productJson = (String) redisTemplate.opsForValue().get(cacheKey);
        if (productJson != null) {
            return JSON.parseObject(productJson, Product.class);
        }
        
        // 2. 尝试获取分布式锁
        String lockValue = UUID.randomUUID().toString();
        Boolean lockAcquired = redisTemplate.opsForValue()
            .setIfAbsent(lockKey, lockValue, Duration.ofSeconds(10));
        
        if (Boolean.TRUE.equals(lockAcquired)) {
            try {
                // 双重检查
                productJson = (String) redisTemplate.opsForValue().get(cacheKey);
                if (productJson != null) {
                    return JSON.parseObject(productJson, Product.class);
                }
                
                // 查询数据库
                Product product = productRepository.findById(productId);
                if (product != null) {
                    // 设置缓存，热点数据设置较长过期时间
                    redisTemplate.opsForValue().set(cacheKey, 
                        JSON.toJSONString(product), Duration.ofHours(2));
                }
                
                return product;
                
            } finally {
                // 释放锁
                releaseLock(lockKey, lockValue);
            }
        } else {
            // 未获取到锁，等待一段时间后重试
            try {
                Thread.sleep(50);
                return getHotProduct(productId);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return null;
            }
        }
    }
    
    /**
     * 方案2：本地锁 + 双重检查
     */
    public Product getHotProductWithLocalLock(Long productId) {
        String cacheKey = "hot:product:" + productId;
        
        // 1. 查询缓存
        String productJson = (String) redisTemplate.opsForValue().get(cacheKey);
        if (productJson != null) {
            return JSON.parseObject(productJson, Product.class);
        }
        
        // 2. 获取本地锁
        Object lock = localLocks.computeIfAbsent(cacheKey, k -> new Object());
        
        synchronized (lock) {
            try {
                // 双重检查
                productJson = (String) redisTemplate.opsForValue().get(cacheKey);
                if (productJson != null) {
                    return JSON.parseObject(productJson, Product.class);
                }
                
                // 查询数据库
                Product product = productRepository.findById(productId);
                if (product != null) {
                    redisTemplate.opsForValue().set(cacheKey, 
                        JSON.toJSONString(product), Duration.ofHours(2));
                }
                
                return product;
                
            } finally {
                // 清理本地锁
                localLocks.remove(cacheKey);
            }
        }
    }
    
    /**
     * 方案3：逻辑过期 + 异步刷新
     */
    public Product getHotProductWithLogicalExpire(Long productId) {
        String cacheKey = "hot:product:" + productId;
        
        // 查询缓存（永不过期）
        String cacheValue = (String) redisTemplate.opsForValue().get(cacheKey);
        
        if (cacheValue != null) {
            CacheData<Product> cacheData = JSON.parseObject(cacheValue, 
                new TypeReference<CacheData<Product>>() {});
            
            // 检查逻辑过期时间
            if (cacheData.getExpireTime().isAfter(LocalDateTime.now())) {
                // 未过期，直接返回
                return cacheData.getData();
            } else {
                // 已过期，异步刷新
                CompletableFuture.runAsync(() -> refreshCache(productId));
                // 返回过期数据
                return cacheData.getData();
            }
        }
        
        // 缓存不存在，同步加载
        return refreshCacheSync(productId);
    }
    
    private void refreshCache(Long productId) {
        String cacheKey = "hot:product:" + productId;
        String lockKey = "refresh:lock:" + productId;
        
        // 尝试获取刷新锁
        Boolean lockAcquired = redisTemplate.opsForValue()
            .setIfAbsent(lockKey, "1", Duration.ofSeconds(30));
        
        if (Boolean.TRUE.equals(lockAcquired)) {
            try {
                Product product = productRepository.findById(productId);
                if (product != null) {
                    CacheData<Product> cacheData = new CacheData<>();
                    cacheData.setData(product);
                    cacheData.setExpireTime(LocalDateTime.now().plusHours(2));
                    
                    redisTemplate.opsForValue().set(cacheKey, JSON.toJSONString(cacheData));
                }
            } finally {
                redisTemplate.delete(lockKey);
            }
        }
    }
    
    private Product refreshCacheSync(Long productId) {
        Product product = productRepository.findById(productId);
        if (product != null) {
            String cacheKey = "hot:product:" + productId;
            CacheData<Product> cacheData = new CacheData<>();
            cacheData.setData(product);
            cacheData.setExpireTime(LocalDateTime.now().plusHours(2));
            
            redisTemplate.opsForValue().set(cacheKey, JSON.toJSONString(cacheData));
        }
        return product;
    }
    
    private void releaseLock(String lockKey, String lockValue) {
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then " +
                       "return redis.call('del', KEYS[1]) " +
                       "else return 0 end";
        
        redisTemplate.execute(new DefaultRedisScript<>(script, Long.class), 
                            Collections.singletonList(lockKey), lockValue);
    }
}

/**
 * 缓存数据包装类
 */
class CacheData<T> {
    private T data;
    private LocalDateTime expireTime;
    
    // getters and setters
    public T getData() { return data; }
    public void setData(T data) { this.data = data; }
    public LocalDateTime getExpireTime() { return expireTime; }
    public void setExpireTime(LocalDateTime expireTime) { this.expireTime = expireTime; }
}
```

### **3. 缓存雪崩（Cache Avalanche）**

**问题描述：** 大量缓存在同一时间失效，导致请求全部打到数据库

```java
/**
 * 缓存雪崩解决方案
 */
@Service
public class CacheAvalancheSolution {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Autowired
    private UserRepository userRepository;
    
    private final Random random = new Random();
    
    /**
     * 方案1：过期时间随机化
     */
    public void cacheUserWithRandomExpire(Long userId, User user) {
        String cacheKey = "user:" + userId;
        
        // 基础过期时间1小时，随机增加0-30分钟
        long baseExpireMinutes = 60;
        long randomMinutes = random.nextInt(31); // 0-30分钟
        Duration expireTime = Duration.ofMinutes(baseExpireMinutes + randomMinutes);
        
        redisTemplate.opsForValue().set(cacheKey, JSON.toJSONString(user), expireTime);
        
        System.out.println("用户缓存已设置，过期时间: " + (baseExpireMinutes + randomMinutes) + "分钟");
    }
    
    /**
     * 方案2：多级缓存
     */
    @Cacheable(value = "userL1", key = "#userId") // 本地缓存
    public User getUserWithMultiLevelCache(Long userId) {
        // L2缓存：Redis
        String cacheKey = "user:l2:" + userId;
        String userJson = (String) redisTemplate.opsForValue().get(cacheKey);
        
        if (userJson != null) {
            User user = JSON.parseObject(userJson, User.class);
            System.out.println("从L2缓存获取用户: " + userId);
            return user;
        }
        
        // L3缓存：数据库
        User user = userRepository.findById(userId);
        if (user != null) {
            // 设置L2缓存
            cacheUserWithRandomExpire(userId, user);
            System.out.println("从数据库获取用户并缓存: " + userId);
        }
        
        return user;
    }
    
    /**
     * 方案3：缓存预热
     */
    @EventListener(ApplicationReadyEvent.class)
    public void warmUpCache() {
        System.out.println("开始缓存预热...");
        
        // 预热热点数据
        List<Long> hotUserIds = userRepository.findHotUserIds(1000);
        
        for (Long userId : hotUserIds) {
            User user = userRepository.findById(userId);
            if (user != null) {
                cacheUserWithRandomExpire(userId, user);
            }
        }
        
        System.out.println("缓存预热完成，预热了 " + hotUserIds.size() + " 个热点用户");
    }
    
    /**
     * 方案4：熔断降级
     */
    @CircuitBreaker(name = "userService", fallbackMethod = "getUserFallback")
    public User getUserWithCircuitBreaker(Long userId) {
        return getUserWithMultiLevelCache(userId);
    }
    
    public User getUserFallback(Long userId, Exception ex) {
        System.out.println("触发熔断，返回默认用户: " + userId);
        
        // 返回默认用户或从备用数据源获取
        return new User(userId, "默认用户", "default@example.com");
    }
    
    /**
     * 方案5：限流保护
     */
    public User getUserWithRateLimit(Long userId) {
        String rateLimitKey = "rate_limit:user:" + LocalDateTime.now().format(
            DateTimeFormatter.ofPattern("yyyy-MM-dd:HH:mm"));
        
        // 每分钟最多1000次请求
        Long currentCount = redisTemplate.opsForValue().increment(rateLimitKey);
        redisTemplate.expire(rateLimitKey, Duration.ofMinutes(1));
        
        if (currentCount > 1000) {
            throw new RuntimeException("请求过于频繁，请稍后重试");
        }
        
        return getUserWithMultiLevelCache(userId);
    }
    
    /**
     * 方案6：异步刷新
     */
    @Async
    public CompletableFuture<Void> asyncRefreshCache(List<Long> userIds) {
        System.out.println("异步刷新缓存开始，用户数量: " + userIds.size());
        
        for (Long userId : userIds) {
            try {
                User user = userRepository.findById(userId);
                if (user != null) {
                    cacheUserWithRandomExpire(userId, user);
                }
                
                // 避免数据库压力过大
                Thread.sleep(10);
                
            } catch (Exception e) {
                System.err.println("刷新用户缓存失败: " + userId + ", 错误: " + e.getMessage());
            }
        }
        
        System.out.println("异步刷新缓存完成");
        return CompletableFuture.completedFuture(null);
    }
}
```

**解决方案对比：**

| 解决方案 | 缓存穿透 | 缓存击穿 | 缓存雪崩 | 实现复杂度 | 性能影响 |
|----------|----------|----------|----------|------------|----------|
| **布隆过滤器** | ✅ | ❌ | ❌ | 中 | 低 |
| **缓存空值** | ✅ | ❌ | ❌ | 低 | 低 |
| **分布式锁** | ❌ | ✅ | ❌ | 高 | 中 |
| **逻辑过期** | ❌ | ✅ | ❌ | 高 | 低 |
| **随机过期** | ❌ | ❌ | ✅ | 低 | 低 |
| **多级缓存** | ✅ | ✅ | ✅ | 高 | 低 |
| **熔断降级** | ✅ | ✅ | ✅ | 中 | 中 |

**最佳实践建议：**

1. **预防为主**：合理设计缓存策略，避免问题发生
2. **多重防护**：结合多种解决方案，构建完整的防护体系
3. **监控告警**：实时监控缓存命中率、数据库压力等关键指标
4. **容量规划**：合理评估缓存容量和数据库承载能力
5. **定期演练**：定期进行故障演练，验证解决方案的有效性

**总结：**

缓存三大问题的解决需要综合考虑业务场景、系统架构和性能要求。通过合理的设计和多重防护机制，可以有效避免这些问题对系统造成的影响。

**Q6: 分布式锁的实现方式有哪些？**

**答案：**

分布式锁是分布式系统中确保多个节点对共享资源互斥访问的重要机制。不同的实现方式各有优缺点，需要根据具体场景选择合适的方案。

**实现方式对比：**

| 实现方式 | 性能 | 可靠性 | 复杂度 | 适用场景 | 主要缺点 |
|----------|------|--------|--------|----------|----------|
| **Redis** | 高 | 中 | 低 | 高并发、低延迟 | 单点故障风险 |
| **Zookeeper** | 中 | 高 | 中 | 强一致性要求 | 性能相对较低 |
| **数据库** | 低 | 高 | 低 | 简单场景 | 性能瓶颈 |
| **Etcd** | 中 | 高 | 中 | 云原生环境 | 学习成本高 |

### **1. Redis分布式锁**

**核心原理：** 利用Redis的原子性操作和过期时间机制

```java
/**
 * Redis分布式锁实现
 */
@Component
public class RedisDistributedLock {
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    private static final String LOCK_PREFIX = "distributed_lock:";
    private static final String UNLOCK_SCRIPT = 
        "if redis.call('get', KEYS[1]) == ARGV[1] then " +
        "return redis.call('del', KEYS[1]) " +
        "else return 0 end";
    
    /**
     * 基础版本：简单的SET NX实现
     */
    public boolean tryLock(String lockKey, String lockValue, long expireTime) {
        String key = LOCK_PREFIX + lockKey;
        
        // SET key value NX EX timeout
        Boolean result = redisTemplate.opsForValue()
            .setIfAbsent(key, lockValue, Duration.ofSeconds(expireTime));
        
        return Boolean.TRUE.equals(result);
    }
    
    /**
     * 释放锁：使用Lua脚本保证原子性
     */
    public boolean unlock(String lockKey, String lockValue) {
        String key = LOCK_PREFIX + lockKey;
        
        Long result = redisTemplate.execute(
            new DefaultRedisScript<>(UNLOCK_SCRIPT, Long.class),
            Collections.singletonList(key),
            lockValue
        );
        
        return Long.valueOf(1).equals(result);
    }
    
    /**
     * 增强版本：支持重试和看门狗机制
     */
    public boolean lockWithRetry(String lockKey, long expireTime, 
                                long retryInterval, int maxRetries) {
        String lockValue = UUID.randomUUID().toString();
        String key = LOCK_PREFIX + lockKey;
        
        for (int i = 0; i < maxRetries; i++) {
            // 尝试获取锁
            Boolean acquired = redisTemplate.opsForValue()
                .setIfAbsent(key, lockValue, Duration.ofSeconds(expireTime));
            
            if (Boolean.TRUE.equals(acquired)) {
                // 启动看门狗线程
                startWatchdog(key, lockValue, expireTime);
                return true;
            }
            
            // 等待重试
            try {
                Thread.sleep(retryInterval);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return false;
            }
        }
        
        return false;
    }
    
    /**
     * 看门狗机制：自动续期
     */
    private void startWatchdog(String key, String lockValue, long expireTime) {
        ScheduledExecutorService scheduler = Executors.newSingleThreadScheduledExecutor();
        
        scheduler.scheduleAtFixedRate(() -> {
            try {
                // 检查锁是否还存在且是当前线程持有
                String currentValue = redisTemplate.opsForValue().get(key);
                if (lockValue.equals(currentValue)) {
                    // 续期
                    redisTemplate.expire(key, Duration.ofSeconds(expireTime));
                    System.out.println("锁续期成功: " + key);
                } else {
                    // 锁已被释放或被其他线程持有，停止看门狗
                    scheduler.shutdown();
                }
            } catch (Exception e) {
                System.err.println("看门狗续期失败: " + e.getMessage());
                scheduler.shutdown();
            }
        }, expireTime / 3, expireTime / 3, TimeUnit.SECONDS);
    }
    
    /**
     * 可重入锁实现
     */
    private final ThreadLocal<Map<String, Integer>> lockCounts = new ThreadLocal<>();
    
    public boolean reentrantLock(String lockKey, long expireTime) {
        String threadId = Thread.currentThread().getName();
        String lockValue = threadId + ":" + System.currentTimeMillis();
        String key = LOCK_PREFIX + lockKey;
        
        // 检查当前线程是否已持有锁
        Map<String, Integer> counts = lockCounts.get();
        if (counts == null) {
            counts = new HashMap<>();
            lockCounts.set(counts);
        }
        
        Integer count = counts.get(lockKey);
        if (count != null && count > 0) {
            // 重入
            counts.put(lockKey, count + 1);
            return true;
        }
        
        // 尝试获取锁
        Boolean acquired = redisTemplate.opsForValue()
            .setIfAbsent(key, lockValue, Duration.ofSeconds(expireTime));
        
        if (Boolean.TRUE.equals(acquired)) {
            counts.put(lockKey, 1);
            return true;
        }
        
        return false;
    }
    
    public boolean reentrantUnlock(String lockKey) {
        Map<String, Integer> counts = lockCounts.get();
        if (counts == null) {
            return false;
        }
        
        Integer count = counts.get(lockKey);
        if (count == null || count <= 0) {
            return false;
        }
        
        if (count > 1) {
            // 减少重入计数
            counts.put(lockKey, count - 1);
            return true;
        } else {
            // 释放锁
            counts.remove(lockKey);
            String threadId = Thread.currentThread().getName();
            String lockValue = threadId + ":" + System.currentTimeMillis();
            return unlock(lockKey, lockValue);
        }
    }
}
```

### **2. Zookeeper分布式锁**

**核心原理：** 利用Zookeeper的临时顺序节点和监听机制

```java
/**
 * Zookeeper分布式锁实现
 */
@Component
public class ZookeeperDistributedLock {
    
    private CuratorFramework client;
    private static final String LOCK_ROOT = "/distributed_locks";
    
    @PostConstruct
    public void init() {
        client = CuratorFrameworkFactory.builder()
            .connectString("localhost:2181")
            .sessionTimeoutMs(30000)
            .connectionTimeoutMs(5000)
            .retryPolicy(new ExponentialBackoffRetry(1000, 3))
            .build();
        
        client.start();
        
        try {
            // 确保根节点存在
            if (client.checkExists().forPath(LOCK_ROOT) == null) {
                client.create()
                    .creatingParentsIfNeeded()
                    .forPath(LOCK_ROOT);
            }
        } catch (Exception e) {
            throw new RuntimeException("初始化Zookeeper锁失败", e);
        }
    }
    
    /**
     * 获取分布式锁
     */
    public InterProcessMutex createLock(String lockKey) {
        String lockPath = LOCK_ROOT + "/" + lockKey;
        return new InterProcessMutex(client, lockPath);
    }
    
    /**
     * 自定义实现：基于临时顺序节点
     */
    public class CustomZkLock {
        private final String lockPath;
        private String currentPath;
        private String waitPath;
        
        public CustomZkLock(String lockKey) {
            this.lockPath = LOCK_ROOT + "/" + lockKey;
        }
        
        public boolean tryLock(long timeout, TimeUnit unit) throws Exception {
            // 创建锁目录
            if (client.checkExists().forPath(lockPath) == null) {
                client.create()
                    .creatingParentsIfNeeded()
                    .forPath(lockPath);
            }
            
            // 创建临时顺序节点
            currentPath = client.create()
                .withMode(CreateMode.EPHEMERAL_SEQUENTIAL)
                .forPath(lockPath + "/lock_");
            
            return waitForLock(timeout, unit);
        }
        
        private boolean waitForLock(long timeout, TimeUnit unit) throws Exception {
            long startTime = System.currentTimeMillis();
            long timeoutMs = unit.toMillis(timeout);
            
            while (true) {
                // 获取所有子节点并排序
                List<String> children = client.getChildren().forPath(lockPath);
                Collections.sort(children);
                
                String currentNode = currentPath.substring(currentPath.lastIndexOf('/') + 1);
                int index = children.indexOf(currentNode);
                
                if (index == 0) {
                    // 当前节点是最小的，获得锁
                    return true;
                }
                
                // 监听前一个节点
                String prevNode = children.get(index - 1);
                waitPath = lockPath + "/" + prevNode;
                
                CountDownLatch latch = new CountDownLatch(1);
                
                Watcher watcher = new Watcher() {
                    @Override
                    public void process(WatchedEvent event) {
                        if (event.getType() == Event.EventType.NodeDeleted) {
                            latch.countDown();
                        }
                    }
                };
                
                // 检查前一个节点是否存在
                if (client.checkExists().usingWatcher(watcher).forPath(waitPath) == null) {
                    // 前一个节点已删除，继续循环
                    continue;
                }
                
                // 等待前一个节点删除或超时
                long remainingTime = timeoutMs - (System.currentTimeMillis() - startTime);
                if (remainingTime <= 0) {
                    return false;
                }
                
                boolean acquired = latch.await(remainingTime, TimeUnit.MILLISECONDS);
                if (!acquired) {
                    return false;
                }
            }
        }
        
        public void unlock() throws Exception {
            if (currentPath != null) {
                client.delete().guaranteed().forPath(currentPath);
                currentPath = null;
            }
        }
    }
    
    /**
     * 使用示例
     */
    public void lockExample() {
        InterProcessMutex lock = createLock("my_lock");
        
        try {
            // 尝试获取锁，最多等待10秒
            if (lock.acquire(10, TimeUnit.SECONDS)) {
                try {
                    // 执行业务逻辑
                    System.out.println("获得锁，执行业务逻辑");
                    Thread.sleep(5000);
                } finally {
                    // 释放锁
                    lock.release();
                }
            } else {
                System.out.println("获取锁超时");
            }
        } catch (Exception e) {
            System.err.println("锁操作异常: " + e.getMessage());
        }
    }
}
```

### **3. 数据库分布式锁**

**核心原理：** 利用数据库的唯一索引约束和事务机制

```java
/**
 * 数据库分布式锁实现
 */
@Component
public class DatabaseDistributedLock {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    /**
     * 锁表结构
     * CREATE TABLE distributed_lock (
     *   lock_key VARCHAR(255) PRIMARY KEY,
     *   lock_value VARCHAR(255) NOT NULL,
     *   expire_time TIMESTAMP NOT NULL,
     *   create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
     * );
     */
    
    /**
     * 方案1：基于唯一索引
     */
    public boolean tryLock(String lockKey, String lockValue, long expireSeconds) {
        try {
            Timestamp expireTime = new Timestamp(
                System.currentTimeMillis() + expireSeconds * 1000);
            
            int result = jdbcTemplate.update(
                "INSERT INTO distributed_lock (lock_key, lock_value, expire_time) VALUES (?, ?, ?)",
                lockKey, lockValue, expireTime
            );
            
            return result > 0;
            
        } catch (DuplicateKeyException e) {
            // 锁已存在，检查是否过期
            return tryAcquireExpiredLock(lockKey, lockValue, expireSeconds);
        }
    }
    
    private boolean tryAcquireExpiredLock(String lockKey, String lockValue, long expireSeconds) {
        try {
            Timestamp now = new Timestamp(System.currentTimeMillis());
            Timestamp newExpireTime = new Timestamp(
                System.currentTimeMillis() + expireSeconds * 1000);
            
            int result = jdbcTemplate.update(
                "UPDATE distributed_lock SET lock_value = ?, expire_time = ? " +
                "WHERE lock_key = ? AND expire_time < ?",
                lockValue, newExpireTime, lockKey, now
            );
            
            return result > 0;
            
        } catch (Exception e) {
            return false;
        }
    }
    
    public boolean unlock(String lockKey, String lockValue) {
        try {
            int result = jdbcTemplate.update(
                "DELETE FROM distributed_lock WHERE lock_key = ? AND lock_value = ?",
                lockKey, lockValue
            );
            
            return result > 0;
            
        } catch (Exception e) {
            return false;
        }
    }
    
    /**
     * 方案2：基于SELECT FOR UPDATE
     */
    @Transactional
    public boolean lockWithSelectForUpdate(String lockKey, String lockValue) {
        try {
            // 查询并锁定记录
            String currentValue = jdbcTemplate.queryForObject(
                "SELECT lock_value FROM distributed_lock WHERE lock_key = ? FOR UPDATE",
                String.class, lockKey
            );
            
            if (currentValue == null) {
                // 记录不存在，插入新记录
                jdbcTemplate.update(
                    "INSERT INTO distributed_lock (lock_key, lock_value, expire_time) VALUES (?, ?, ?)",
                    lockKey, lockValue, new Timestamp(System.currentTimeMillis() + 30000)
                );
                return true;
            } else {
                // 记录存在，检查是否可以获取
                return lockValue.equals(currentValue);
            }
            
        } catch (EmptyResultDataAccessException e) {
            // 记录不存在，尝试插入
            try {
                jdbcTemplate.update(
                    "INSERT INTO distributed_lock (lock_key, lock_value, expire_time) VALUES (?, ?, ?)",
                    lockKey, lockValue, new Timestamp(System.currentTimeMillis() + 30000)
                );
                return true;
            } catch (Exception ex) {
                return false;
            }
        }
    }
    
    /**
     * 清理过期锁
     */
    @Scheduled(fixedRate = 60000) // 每分钟执行一次
    public void cleanExpiredLocks() {
        try {
            Timestamp now = new Timestamp(System.currentTimeMillis());
            int deletedCount = jdbcTemplate.update(
                "DELETE FROM distributed_lock WHERE expire_time < ?", now
            );
            
            if (deletedCount > 0) {
                System.out.println("清理过期锁: " + deletedCount + " 个");
            }
        } catch (Exception e) {
            System.err.println("清理过期锁失败: " + e.getMessage());
        }
    }
}
```

### **4. Redisson分布式锁**

**核心原理：** 基于Redis的高级分布式锁实现

```java
/**
 * Redisson分布式锁实现
 */
@Component
public class RedissonDistributedLock {
    
    @Autowired
    private RedissonClient redissonClient;
    
    /**
     * 可重入锁
     */
    public void reentrantLockExample() {
        RLock lock = redissonClient.getLock("my_lock");
        
        try {
            // 尝试获取锁，最多等待10秒，锁定30秒后自动释放
            boolean acquired = lock.tryLock(10, 30, TimeUnit.SECONDS);
            
            if (acquired) {
                try {
                    // 执行业务逻辑
                    System.out.println("获得锁，执行业务逻辑");
                    Thread.sleep(5000);
                } finally {
                    // 释放锁
                    if (lock.isHeldByCurrentThread()) {
                        lock.unlock();
                    }
                }
            } else {
                System.out.println("获取锁超时");
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    /**
     * 公平锁
     */
    public void fairLockExample() {
        RLock fairLock = redissonClient.getFairLock("fair_lock");
        
        try {
            if (fairLock.tryLock(10, 30, TimeUnit.SECONDS)) {
                try {
                    // 执行业务逻辑
                    System.out.println("获得公平锁");
                } finally {
                    fairLock.unlock();
                }
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    /**
     * 读写锁
     */
    public void readWriteLockExample() {
        RReadWriteLock rwLock = redissonClient.getReadWriteLock("rw_lock");
        
        // 读锁
        RLock readLock = rwLock.readLock();
        try {
            if (readLock.tryLock(10, 30, TimeUnit.SECONDS)) {
                try {
                    // 执行读操作
                    System.out.println("获得读锁");
                } finally {
                    readLock.unlock();
                }
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        // 写锁
        RLock writeLock = rwLock.writeLock();
        try {
            if (writeLock.tryLock(10, 30, TimeUnit.SECONDS)) {
                try {
                    // 执行写操作
                    System.out.println("获得写锁");
                } finally {
                    writeLock.unlock();
                }
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    /**
     * 多重锁
     */
    public void multiLockExample() {
        RLock lock1 = redissonClient.getLock("lock1");
        RLock lock2 = redissonClient.getLock("lock2");
        RLock lock3 = redissonClient.getLock("lock3");
        
        RLock multiLock = redissonClient.getMultiLock(lock1, lock2, lock3);
        
        try {
            if (multiLock.tryLock(10, 30, TimeUnit.SECONDS)) {
                try {
                    // 同时获得多个锁
                    System.out.println("获得多重锁");
                } finally {
                    multiLock.unlock();
                }
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

**选择建议：**

1. **高性能场景**：选择Redis或Redisson，配合集群模式提高可用性
2. **强一致性要求**：选择Zookeeper
3. **简单场景**：选择数据库实现，易于理解和维护
4. **功能丰富**：选择Redisson，提供多种锁类型

**最佳实践：**

1. **设置合理的超时时间**：避免死锁
2. **实现锁续期机制**：防止业务执行时间超过锁过期时间
3. **添加监控和告警**：及时发现锁异常
4. **考虑锁的粒度**：平衡并发性能和数据一致性
5. **处理异常情况**：网络分区、节点故障等

### 3. 数据库优化相关

**Q7: MySQL索引的类型有哪些？什么时候会失效？**

**答案：**

MySQL索引是提高查询性能的关键技术，了解不同索引类型和失效场景对数据库优化至关重要。

### **索引类型详解**

**按数据结构分类：**

| 索引类型 | 数据结构 | 特点 | 适用场景 | 存储引擎支持 |
|----------|----------|------|----------|-------------|
| **B+Tree索引** | B+树 | 有序、范围查询高效 | 大部分查询场景 | InnoDB、MyISAM |
| **Hash索引** | 哈希表 | 等值查询快速 | 精确匹配查询 | Memory、NDB |
| **Full-text索引** | 倒排索引 | 文本搜索 | 全文检索 | InnoDB、MyISAM |
| **R-tree索引** | R树 | 空间数据 | 地理位置查询 | MyISAM |

**按功能分类：**

```sql
-- 1. 主键索引（Primary Key Index）
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,  -- 自动创建主键索引
    username VARCHAR(50),
    email VARCHAR(100)
);

-- 2. 唯一索引（Unique Index）
CREATE UNIQUE INDEX idx_username ON users(username);
CREATE UNIQUE INDEX idx_email ON users(email);

-- 3. 普通索引（Normal Index）
CREATE INDEX idx_create_time ON users(create_time);

-- 4. 复合索引（Composite Index）
CREATE INDEX idx_name_age ON users(name, age, city);

-- 5. 前缀索引（Prefix Index）
CREATE INDEX idx_email_prefix ON users(email(10));

-- 6. 函数索引（MySQL 8.0+）
CREATE INDEX idx_upper_name ON users((UPPER(name)));

-- 7. 部分索引（Partial Index）
CREATE INDEX idx_active_users ON users(status) WHERE status = 'active';
```

### **索引失效场景分析**

**1. 函数或表达式操作**

```sql
-- ❌ 索引失效：对索引列使用函数
SELECT * FROM users WHERE UPPER(name) = 'JOHN';
SELECT * FROM users WHERE YEAR(create_time) = 2024;
SELECT * FROM users WHERE age + 1 = 26;

-- ✅ 索引生效：避免对索引列使用函数
SELECT * FROM users WHERE name = 'john';
SELECT * FROM users WHERE create_time >= '2024-01-01' AND create_time < '2025-01-01';
SELECT * FROM users WHERE age = 25;

-- ✅ MySQL 8.0+ 可以使用函数索引
CREATE INDEX idx_upper_name ON users((UPPER(name)));
SELECT * FROM users WHERE UPPER(name) = 'JOHN';  -- 现在可以使用索引
```

**2. 隐式类型转换**

```sql
-- 表结构
CREATE TABLE orders (
    id INT PRIMARY KEY,
    order_no VARCHAR(20),  -- 字符串类型
    amount DECIMAL(10,2)
);

CREATE INDEX idx_order_no ON orders(order_no);

-- ❌ 索引失效：字符串列与数字比较
SELECT * FROM orders WHERE order_no = 12345;  -- 隐式转换为 CAST(order_no AS SIGNED) = 12345

-- ✅ 索引生效：类型匹配
SELECT * FROM orders WHERE order_no = '12345';

-- 验证是否使用索引
EXPLAIN SELECT * FROM orders WHERE order_no = 12345;
EXPLAIN SELECT * FROM orders WHERE order_no = '12345';
```

**3. 模糊查询场景**

```sql
-- ❌ 索引失效：以通配符开头
SELECT * FROM users WHERE name LIKE '%john%';
SELECT * FROM users WHERE name LIKE '%john';

-- ✅ 索引生效：通配符在后面
SELECT * FROM users WHERE name LIKE 'john%';

-- ✅ 全文索引解决方案
CREATE FULLTEXT INDEX idx_name_fulltext ON users(name);
SELECT * FROM users WHERE MATCH(name) AGAINST('john' IN NATURAL LANGUAGE MODE);
```

**4. 复合索引最左前缀原则**

```sql
-- 创建复合索引
CREATE INDEX idx_name_age_city ON users(name, age, city);

-- ✅ 索引生效：遵循最左前缀
SELECT * FROM users WHERE name = 'john';                    -- 使用 name
SELECT * FROM users WHERE name = 'john' AND age = 25;       -- 使用 name, age
SELECT * FROM users WHERE name = 'john' AND age = 25 AND city = 'beijing';  -- 使用 name, age, city
SELECT * FROM users WHERE name = 'john' AND city = 'beijing';  -- 使用 name

-- ❌ 索引失效：不遵循最左前缀
SELECT * FROM users WHERE age = 25;                         -- 跳过 name
SELECT * FROM users WHERE city = 'beijing';                 -- 跳过 name, age
SELECT * FROM users WHERE age = 25 AND city = 'beijing';    -- 跳过 name
```

**5. OR条件使用**

```sql
-- ❌ 索引可能失效：OR连接不同列
SELECT * FROM users WHERE name = 'john' OR email = 'john@example.com';

-- ✅ 索引生效：使用UNION
SELECT * FROM users WHERE name = 'john'
UNION
SELECT * FROM users WHERE email = 'john@example.com';

-- ✅ 索引生效：OR连接同一列
SELECT * FROM users WHERE name = 'john' OR name = 'jane';
-- 等价于
SELECT * FROM users WHERE name IN ('john', 'jane');
```

**6. 范围查询后的列**

```sql
-- 复合索引
CREATE INDEX idx_name_age_city ON users(name, age, city);

-- ❌ city列索引失效：age使用了范围查询
SELECT * FROM users WHERE name = 'john' AND age > 20 AND city = 'beijing';

-- ✅ 优化：调整索引顺序
CREATE INDEX idx_name_city_age ON users(name, city, age);
SELECT * FROM users WHERE name = 'john' AND city = 'beijing' AND age > 20;
```

**7. 不等于操作符**

```sql
-- ❌ 索引效率低：不等于操作
SELECT * FROM users WHERE status != 'deleted';
SELECT * FROM users WHERE status <> 'deleted';

-- ✅ 索引生效：使用IN或EXISTS
SELECT * FROM users WHERE status IN ('active', 'inactive', 'pending');
```

### **索引优化实践**

```java
/**
 * 索引使用监控和优化
 */
@Component
public class IndexOptimizer {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    /**
     * 分析慢查询
     */
    public void analyzeSlowQueries() {
        // 开启慢查询日志
        String enableSlowLog = "SET GLOBAL slow_query_log = 'ON'";
        String setSlowTime = "SET GLOBAL long_query_time = 2";
        
        jdbcTemplate.execute(enableSlowLog);
        jdbcTemplate.execute(setSlowTime);
        
        // 查询慢查询统计
        String slowQueryStats = 
            "SELECT sql_text, exec_count, avg_timer_wait/1000000000 as avg_time_sec " +
            "FROM performance_schema.events_statements_summary_by_digest " +
            "WHERE avg_timer_wait > 2000000000 " +
            "ORDER BY avg_timer_wait DESC LIMIT 10";
        
        List<Map<String, Object>> slowQueries = jdbcTemplate.queryForList(slowQueryStats);
        
        slowQueries.forEach(query -> {
            System.out.println("慢查询: " + query.get("sql_text"));
            System.out.println("平均执行时间: " + query.get("avg_time_sec") + "秒");
            System.out.println("执行次数: " + query.get("exec_count"));
            System.out.println("---");
        });
    }
    
    /**
     * 检查索引使用情况
     */
    public void checkIndexUsage(String tableName) {
        // 查看表的索引信息
        String showIndexes = "SHOW INDEX FROM " + tableName;
        List<Map<String, Object>> indexes = jdbcTemplate.queryForList(showIndexes);
        
        System.out.println("表 " + tableName + " 的索引信息:");
        indexes.forEach(index -> {
            System.out.println("索引名: " + index.get("Key_name"));
            System.out.println("列名: " + index.get("Column_name"));
            System.out.println("基数: " + index.get("Cardinality"));
            System.out.println("---");
        });
        
        // 查看索引统计信息
        String indexStats = 
            "SELECT table_name, index_name, " +
            "rows_read, rows_read_avg, rows_read_per_insert " +
            "FROM performance_schema.table_io_waits_summary_by_index_usage " +
            "WHERE table_name = ?";
        
        List<Map<String, Object>> stats = jdbcTemplate.queryForList(indexStats, tableName);
        
        System.out.println("索引使用统计:");
        stats.forEach(stat -> {
            System.out.println("索引: " + stat.get("index_name"));
            System.out.println("读取行数: " + stat.get("rows_read"));
            System.out.println("平均读取: " + stat.get("rows_read_avg"));
            System.out.println("---");
        });
    }
    
    /**
     * 执行计划分析
     */
    public void explainQuery(String sql) {
        String explainSql = "EXPLAIN FORMAT=JSON " + sql;
        
        try {
            String result = jdbcTemplate.queryForObject(explainSql, String.class);
            System.out.println("执行计划:");
            System.out.println(result);
            
            // 简化分析
            if (result.contains("index")) {
                System.out.println("✅ 查询使用了索引");
            } else {
                System.out.println("❌ 查询未使用索引，可能需要优化");
            }
            
        } catch (Exception e) {
            System.err.println("执行计划分析失败: " + e.getMessage());
        }
    }
    
    /**
     * 索引建议
     */
    public void suggestIndexes(String tableName) {
        // 查找缺失索引的查询
        String missingIndexes = 
            "SELECT DISTINCT " +
            "CONCAT('CREATE INDEX idx_', " +
            "REPLACE(REPLACE(GROUP_CONCAT(column_name), ',', '_'), ' ', ''), " +
            "' ON ', table_name, '(', GROUP_CONCAT(column_name), ');') AS suggested_index " +
            "FROM information_schema.statistics " +
            "WHERE table_name = ? " +
            "GROUP BY table_name, index_name";
        
        try {
            List<String> suggestions = jdbcTemplate.queryForList(missingIndexes, String.class, tableName);
            
            System.out.println("索引建议:");
            suggestions.forEach(suggestion -> {
                System.out.println(suggestion);
            });
            
        } catch (Exception e) {
            System.out.println("无法生成索引建议: " + e.getMessage());
        }
    }
}
```

### **索引设计最佳实践**

**1. 选择性原则**
```sql
-- 计算列的选择性
SELECT 
    COUNT(DISTINCT column_name) / COUNT(*) AS selectivity,
    COUNT(DISTINCT column_name) AS distinct_values,
    COUNT(*) AS total_rows
FROM table_name;

-- 选择性高的列适合建索引（接近1.0）
-- 选择性低的列不适合建索引（接近0.0）
```

**2. 索引维护**
```sql
-- 重建索引
ALTER TABLE users DROP INDEX idx_name, ADD INDEX idx_name(name);

-- 分析表统计信息
ANALYZE TABLE users;

-- 优化表
OPTIMIZE TABLE users;
```

**3. 监控指标**
- 索引命中率
- 查询响应时间
- 索引大小
- 更新开销

**总结：**

索引是数据库性能优化的重要手段，但需要在查询性能和维护成本之间找到平衡。合理设计索引、避免索引失效场景、定期监控和优化是确保数据库高性能的关键。

**Q8: 什么是分库分表？有哪些策略？**

**答案：**

分库分表是应对大数据量和高并发场景的重要数据库架构设计模式，通过将数据分散存储来提升系统性能和可扩展性。

### **分库分表概述**

**核心目标：**
- 解决单表数据量过大问题
- 提升查询性能
- 增强系统并发能力
- 实现水平扩展

**适用场景：**
- 单表数据量超过1000万
- 数据库连接数不足
- 单库磁盘空间不够
- 查询响应时间过长

### **分库分表策略详解**

**策略对比表：**

| 策略类型 | 分割维度 | 优点 | 缺点 | 适用场景 |
|----------|----------|------|------|----------|
| **垂直分库** | 业务模块 | 业务隔离、专业化 | 跨库事务复杂 | 微服务架构 |
| **水平分库** | 数据特征 | 负载均衡、扩展性好 | 路由复杂、数据迁移难 | 大数据量场景 |
| **垂直分表** | 字段维度 | 减少IO、提升缓存命中 | 关联查询复杂 | 宽表优化 |
| **水平分表** | 数据量 | 查询性能提升 | 跨表查询复杂 | 单表数据量大 |

### **1. 垂直分库**

**核心思想：** 按业务模块将不同的表分布到不同的数据库中

```java
/**
 * 垂直分库配置
 */
@Configuration
public class VerticalShardingConfig {
    
    // 用户服务数据库
    @Bean
    @Primary
    @ConfigurationProperties("spring.datasource.user")
    public DataSource userDataSource() {
        return DataSourceBuilder.create().build();
    }
    
    // 订单服务数据库
    @Bean
    @ConfigurationProperties("spring.datasource.order")
    public DataSource orderDataSource() {
        return DataSourceBuilder.create().build();
    }
    
    // 商品服务数据库
    @Bean
    @ConfigurationProperties("spring.datasource.product")
    public DataSource productDataSource() {
        return DataSourceBuilder.create().build();
    }
    
    // 用户服务JdbcTemplate
    @Bean
    public JdbcTemplate userJdbcTemplate(@Qualifier("userDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
    
    // 订单服务JdbcTemplate
    @Bean
    public JdbcTemplate orderJdbcTemplate(@Qualifier("orderDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}

/**
 * 数据库路由器
 */
@Component
public class DatabaseRouter {
    
    @Autowired
    @Qualifier("userJdbcTemplate")
    private JdbcTemplate userJdbcTemplate;
    
    @Autowired
    @Qualifier("orderJdbcTemplate")
    private JdbcTemplate orderJdbcTemplate;
    
    public JdbcTemplate getJdbcTemplate(String businessType) {
        switch (businessType.toLowerCase()) {
            case "user":
                return userJdbcTemplate;
            case "order":
                return orderJdbcTemplate;
            default:
                throw new IllegalArgumentException("未知的业务类型: " + businessType);
        }
    }
}
```

### **2. 水平分库**

**核心思想：** 按照某种规则将同一个表的数据分散到多个数据库中

```java
/**
 * 水平分库实现
 */
@Component
public class HorizontalShardingManager {
    
    private final Map<String, DataSource> dataSourceMap = new HashMap<>();
    private final int dbCount = 4; // 分库数量
    
    @PostConstruct
    public void initDataSources() {
        for (int i = 0; i < dbCount; i++) {
            HikariConfig config = new HikariConfig();
            config.setJdbcUrl("jdbc:mysql://localhost:3306/db_" + i);
            config.setUsername("root");
            config.setPassword("password");
            config.setMaximumPoolSize(10);
            
            dataSourceMap.put("db_" + i, new HikariDataSource(config));
        }
    }
    
    /**
     * 根据用户ID路由到对应数据库
     */
    public DataSource getDataSource(Long userId) {
        int dbIndex = (int) (userId % dbCount);
        return dataSourceMap.get("db_" + dbIndex);
    }
    
    /**
     * 根据哈希值路由
     */
    public DataSource getDataSourceByHash(String shardingKey) {
        int hash = Math.abs(shardingKey.hashCode());
        int dbIndex = hash % dbCount;
        return dataSourceMap.get("db_" + dbIndex);
    }
    
    /**
     * 范围分片
     */
    public DataSource getDataSourceByRange(Long id) {
        if (id <= 1000000) {
            return dataSourceMap.get("db_0");
        } else if (id <= 2000000) {
            return dataSourceMap.get("db_1");
        } else if (id <= 3000000) {
            return dataSourceMap.get("db_2");
        } else {
            return dataSourceMap.get("db_3");
        }
    }
}
```

### **3. 垂直分表**

**核心思想：** 将一个表按字段拆分成多个表

```java
/**
 * 垂直分表示例
 */
@Entity
@Table(name = "user_basic")
public class UserBasic {
    @Id
    private Long id;
    private String username;
    private String email;
    private String phone;
    private Date createTime;
    
    // getters and setters
}

@Entity
@Table(name = "user_profile")
public class UserProfile {
    @Id
    private Long userId;
    private String avatar;
    private String bio;
    private String address;
    private String preferences; // JSON格式存储
    
    // getters and setters
}

@Entity
@Table(name = "user_statistics")
public class UserStatistics {
    @Id
    private Long userId;
    private Integer loginCount;
    private Date lastLoginTime;
    private Long totalSpent;
    private Integer orderCount;
    
    // getters and setters
}

/**
 * 垂直分表服务
 */
@Service
public class UserVerticalService {
    
    @Autowired
    private UserBasicRepository userBasicRepository;
    
    @Autowired
    private UserProfileRepository userProfileRepository;
    
    @Autowired
    private UserStatisticsRepository userStatisticsRepository;
    
    /**
     * 获取用户完整信息
     */
    public UserCompleteInfo getUserCompleteInfo(Long userId) {
        UserBasic basic = userBasicRepository.findById(userId).orElse(null);
        UserProfile profile = userProfileRepository.findByUserId(userId);
        UserStatistics statistics = userStatisticsRepository.findByUserId(userId);
        
        return UserCompleteInfo.builder()
            .basic(basic)
            .profile(profile)
            .statistics(statistics)
            .build();
    }
    
    /**
     * 更新用户基本信息
     */
    @Transactional
    public void updateUserBasic(Long userId, UserBasic userBasic) {
        userBasic.setId(userId);
        userBasicRepository.save(userBasic);
    }
}
```

### **4. 水平分表**

**核心思想：** 将一个表按数据量拆分成多个结构相同的表

```java
/**
 * 水平分表实现
 */
@Component
public class HorizontalTableSharding {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    private final int tableCount = 8; // 分表数量
    
    /**
     * 根据用户ID获取表名
     */
    public String getTableName(String baseTableName, Long userId) {
        int tableIndex = (int) (userId % tableCount);
        return baseTableName + "_" + tableIndex;
    }
    
    /**
     * 根据时间分表
     */
    public String getTableNameByDate(String baseTableName, Date date) {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyyMM");
        return baseTableName + "_" + sdf.format(date);
    }
    
    /**
     * 插入数据到分表
     */
    public void insertOrder(Order order) {
        String tableName = getTableName("t_order", order.getUserId());
        
        String sql = "INSERT INTO " + tableName + 
                    " (id, user_id, product_id, amount, create_time) VALUES (?, ?, ?, ?, ?)";
        
        jdbcTemplate.update(sql, 
            order.getId(),
            order.getUserId(),
            order.getProductId(),
            order.getAmount(),
            order.getCreateTime()
        );
    }
    
    /**
     * 查询用户订单
     */
    public List<Order> findOrdersByUserId(Long userId) {
        String tableName = getTableName("t_order", userId);
        
        String sql = "SELECT * FROM " + tableName + " WHERE user_id = ?";
        
        return jdbcTemplate.query(sql, new Object[]{userId}, (rs, rowNum) -> {
            Order order = new Order();
            order.setId(rs.getLong("id"));
            order.setUserId(rs.getLong("user_id"));
            order.setProductId(rs.getLong("product_id"));
            order.setAmount(rs.getBigDecimal("amount"));
            order.setCreateTime(rs.getTimestamp("create_time"));
            return order;
        });
    }
    
    /**
     * 跨表查询（需要查询所有分表）
     */
    public List<Order> findOrdersByDateRange(Date startDate, Date endDate) {
        List<Order> allOrders = new ArrayList<>();
        
        // 遍历所有分表
        for (int i = 0; i < tableCount; i++) {
            String tableName = "t_order_" + i;
            
            String sql = "SELECT * FROM " + tableName + 
                        " WHERE create_time BETWEEN ? AND ?";
            
            List<Order> orders = jdbcTemplate.query(sql, 
                new Object[]{startDate, endDate}, 
                (rs, rowNum) -> {
                    Order order = new Order();
                    order.setId(rs.getLong("id"));
                    order.setUserId(rs.getLong("user_id"));
                    order.setProductId(rs.getLong("product_id"));
                    order.setAmount(rs.getBigDecimal("amount"));
                    order.setCreateTime(rs.getTimestamp("create_time"));
                    return order;
                }
            );
            
            allOrders.addAll(orders);
        }
        
        // 排序
        allOrders.sort(Comparator.comparing(Order::getCreateTime));
        
        return allOrders;
    }
}
```

### **分片算法详解**

```java
/**
 * 分片算法实现
 */
public class ShardingAlgorithms {
    
    /**
     * 1. 取模算法
     */
    public static class ModuloSharding {
        private final int shardCount;
        
        public ModuloSharding(int shardCount) {
            this.shardCount = shardCount;
        }
        
        public int getShard(Long id) {
            return (int) (id % shardCount);
        }
        
        public String getTableName(String baseTable, Long id) {
            return baseTable + "_" + getShard(id);
        }
    }
    
    /**
     * 2. 范围分片算法
     */
    public static class RangeSharding {
        private final List<RangeRule> rules;
        
        public RangeSharding(List<RangeRule> rules) {
            this.rules = rules;
        }
        
        public String getTableName(String baseTable, Long id) {
            for (RangeRule rule : rules) {
                if (id >= rule.getMinValue() && id <= rule.getMaxValue()) {
                    return baseTable + "_" + rule.getSuffix();
                }
            }
            throw new IllegalArgumentException("无法找到匹配的分片规则: " + id);
        }
        
        public static class RangeRule {
            private Long minValue;
            private Long maxValue;
            private String suffix;
            
            // constructors, getters and setters
        }
    }
    
    /**
     * 3. 一致性哈希算法
     */
    public static class ConsistentHashSharding {
        private final TreeMap<Long, String> ring = new TreeMap<>();
        private final int virtualNodes = 150;
        
        public ConsistentHashSharding(List<String> nodes) {
            for (String node : nodes) {
                addNode(node);
            }
        }
        
        private void addNode(String node) {
            for (int i = 0; i < virtualNodes; i++) {
                String virtualNode = node + "#" + i;
                long hash = hash(virtualNode);
                ring.put(hash, node);
            }
        }
        
        public String getNode(String key) {
            if (ring.isEmpty()) {
                return null;
            }
            
            long hash = hash(key);
            Map.Entry<Long, String> entry = ring.ceilingEntry(hash);
            
            if (entry == null) {
                entry = ring.firstEntry();
            }
            
            return entry.getValue();
        }
        
        private long hash(String key) {
            // 使用CRC32哈希算法
            CRC32 crc32 = new CRC32();
            crc32.update(key.getBytes());
            return crc32.getValue();
        }
    }
    
    /**
     * 4. 时间分片算法
     */
    public static class TimeBasedSharding {
        
        public String getTableName(String baseTable, Date date, String pattern) {
            SimpleDateFormat sdf = new SimpleDateFormat(pattern);
            return baseTable + "_" + sdf.format(date);
        }
        
        // 按月分表
        public String getMonthlyTable(String baseTable, Date date) {
            return getTableName(baseTable, date, "yyyyMM");
        }
        
        // 按日分表
        public String getDailyTable(String baseTable, Date date) {
            return getTableName(baseTable, date, "yyyyMMdd");
        }
        
        // 按年分表
        public String getYearlyTable(String baseTable, Date date) {
            return getTableName(baseTable, date, "yyyy");
        }
    }
}
```

### **分库分表中间件**

```java
/**
 * ShardingSphere配置示例
 */
@Configuration
public class ShardingSphereConfig {
    
    @Bean
    public DataSource shardingDataSource() throws SQLException {
        // 配置数据源
        Map<String, DataSource> dataSourceMap = new HashMap<>();
        dataSourceMap.put("ds0", createDataSource("db0"));
        dataSourceMap.put("ds1", createDataSource("db1"));
        
        // 分库规则
        ShardingRuleConfiguration shardingRuleConfig = new ShardingRuleConfiguration();
        
        // 配置t_order表的分片规则
        TableRuleConfiguration orderTableRuleConfig = new TableRuleConfiguration("t_order", "ds${0..1}.t_order_${0..3}");
        
        // 分库策略
        orderTableRuleConfig.setDatabaseShardingStrategyConfig(
            new InlineShardingStrategyConfiguration("user_id", "ds${user_id % 2}")
        );
        
        // 分表策略
        orderTableRuleConfig.setTableShardingStrategyConfig(
            new InlineShardingStrategyConfiguration("order_id", "t_order_${order_id % 4}")
        );
        
        shardingRuleConfig.getTableRuleConfigs().add(orderTableRuleConfig);
        
        // 创建ShardingDataSource
        return ShardingDataSourceFactory.createDataSource(dataSourceMap, shardingRuleConfig, new Properties());
    }
    
    private DataSource createDataSource(String dbName) {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/" + dbName);
        config.setUsername("root");
        config.setPassword("password");
        return new HikariDataSource(config);
    }
}
```

### **分库分表最佳实践**

**1. 选择合适的分片键**
- 查询频率高的字段
- 数据分布均匀的字段
- 业务相关性强的字段

**2. 避免跨库事务**
- 设计时考虑事务边界
- 使用分布式事务方案
- 采用最终一致性

**3. 处理数据迁移**
- 制定详细的迁移计划
- 使用双写方案
- 验证数据一致性

**4. 监控和运维**
- 监控各分片负载
- 定期数据备份
- 制定扩容方案

**总结：**

分库分表是解决大数据量问题的有效方案，但会增加系统复杂度。选择合适的分片策略、处理好跨库查询和事务问题、做好监控和运维是成功实施分库分表的关键。

**Q9: 读写分离的原理和注意事项？**

**答案：**

读写分离是一种常见的数据库架构模式，通过将读操作和写操作分离到不同的数据库实例上，来提升系统的并发处理能力和查询性能。

### **读写分离原理**

**核心架构：**
```
应用程序
    |
    v
读写分离中间件
    |
    +-- 写操作 --> 主库(Master)
    |                 |
    +-- 读操作 --> 从库(Slave1, Slave2, ...)
                      ^
                      |
                   主从复制
```

**工作流程：**
1. **写操作**：所有的INSERT、UPDATE、DELETE操作都路由到主库
2. **读操作**：所有的SELECT操作都路由到从库
3. **数据同步**：主库通过binlog将数据变更同步到从库
4. **负载均衡**：多个从库之间进行读请求的负载分配

### **读写分离实现方案**

**方案对比表：**

| 实现方式 | 优点 | 缺点 | 适用场景 |
|----------|------|------|----------|
| **应用层实现** | 灵活、可控性强 | 代码侵入性大 | 定制化需求高 |
| **中间件代理** | 透明、易维护 | 增加网络延迟 | 通用场景 |
| **数据库中间件** | 功能丰富、性能好 | 学习成本高 | 大型系统 |
| **云服务方案** | 免运维、高可用 | 厂商绑定 | 云原生应用 |

### **1. 应用层读写分离实现**

```java
/**
 * 数据源配置
 */
@Configuration
public class ReadWriteDataSourceConfig {
    
    @Bean
    @Primary
    @ConfigurationProperties("spring.datasource.master")
    public DataSource masterDataSource() {
        return DataSourceBuilder.create().build();
    }
    
    @Bean
    @ConfigurationProperties("spring.datasource.slave1")
    public DataSource slave1DataSource() {
        return DataSourceBuilder.create().build();
    }
    
    @Bean
    @ConfigurationProperties("spring.datasource.slave2")
    public DataSource slave2DataSource() {
        return DataSourceBuilder.create().build();
    }
    
    @Bean
    public DataSource routingDataSource() {
        ReadWriteRoutingDataSource routingDataSource = new ReadWriteRoutingDataSource();
        
        Map<Object, Object> dataSourceMap = new HashMap<>();
        dataSourceMap.put("master", masterDataSource());
        dataSourceMap.put("slave1", slave1DataSource());
        dataSourceMap.put("slave2", slave2DataSource());
        
        routingDataSource.setTargetDataSources(dataSourceMap);
        routingDataSource.setDefaultTargetDataSource(masterDataSource());
        
        return routingDataSource;
    }
    
    @Bean
    public JdbcTemplate jdbcTemplate(@Qualifier("routingDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}

/**
 * 动态数据源路由
 */
public class ReadWriteRoutingDataSource extends AbstractRoutingDataSource {
    
    @Override
    protected Object determineCurrentLookupKey() {
        return DataSourceContextHolder.getDataSourceType();
    }
}

/**
 * 数据源上下文持有者
 */
public class DataSourceContextHolder {
    
    private static final ThreadLocal<String> CONTEXT_HOLDER = new ThreadLocal<>();
    
    private static final List<String> SLAVE_DATA_SOURCES = Arrays.asList("slave1", "slave2");
    
    private static final AtomicInteger COUNTER = new AtomicInteger(0);
    
    public static void setMaster() {
        CONTEXT_HOLDER.set("master");
    }
    
    public static void setSlave() {
        // 轮询选择从库
        int index = COUNTER.getAndIncrement() % SLAVE_DATA_SOURCES.size();
        CONTEXT_HOLDER.set(SLAVE_DATA_SOURCES.get(index));
    }
    
    public static String getDataSourceType() {
        return CONTEXT_HOLDER.get();
    }
    
    public static void clear() {
        CONTEXT_HOLDER.remove();
    }
}

/**
 * 读写分离注解
 */
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface ReadOnly {
    boolean value() default true;
}

/**
 * 读写分离切面
 */
@Aspect
@Component
@Order(1) // 确保在事务切面之前执行
public class ReadWriteAspect {
    
    @Pointcut("@annotation(com.example.annotation.ReadOnly) || @within(com.example.annotation.ReadOnly)")
    public void readOnlyPointcut() {}
    
    @Before("readOnlyPointcut()")
    public void setReadDataSource(JoinPoint joinPoint) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        
        ReadOnly readOnly = method.getAnnotation(ReadOnly.class);
        if (readOnly == null) {
            readOnly = method.getDeclaringClass().getAnnotation(ReadOnly.class);
        }
        
        if (readOnly != null && readOnly.value()) {
            DataSourceContextHolder.setSlave();
        } else {
            DataSourceContextHolder.setMaster();
        }
    }
    
    @After("readOnlyPointcut()")
    public void clearDataSource() {
        DataSourceContextHolder.clear();
    }
}
```

### **2. 基于MyBatis的读写分离**

```java
/**
 * MyBatis读写分离插件
 */
@Intercepts({
    @Signature(type = Executor.class, method = "update", args = {MappedStatement.class, Object.class}),
    @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class})
})
public class ReadWriteSeparationInterceptor implements Interceptor {
    
    private static final String SELECT = "select";
    
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        Object[] args = invocation.getArgs();
        MappedStatement ms = (MappedStatement) args[0];
        
        // 判断是否为读操作
        if (ms.getSqlCommandType() == SqlCommandType.SELECT) {
            // 检查是否在事务中
            if (!TransactionSynchronizationManager.isActualTransactionActive()) {
                DataSourceContextHolder.setSlave();
            } else {
                // 事务中的读操作也走主库，保证一致性
                DataSourceContextHolder.setMaster();
            }
        } else {
            DataSourceContextHolder.setMaster();
        }
        
        try {
            return invocation.proceed();
        } finally {
            DataSourceContextHolder.clear();
        }
    }
    
    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }
    
    @Override
    public void setProperties(Properties properties) {
        // 可以设置一些配置属性
    }
}

/**
 * 服务层实现示例
 */
@Service
public class UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    /**
     * 读操作 - 自动路由到从库
     */
    @ReadOnly
    public User findById(Long id) {
        return userMapper.selectById(id);
    }
    
    /**
     * 读操作 - 强制走主库（保证一致性）
     */
    public User findByIdFromMaster(Long id) {
        DataSourceContextHolder.setMaster();
        try {
            return userMapper.selectById(id);
        } finally {
            DataSourceContextHolder.clear();
        }
    }
    
    /**
     * 写操作 - 自动路由到主库
     */
    @Transactional
    public void updateUser(User user) {
        userMapper.updateById(user);
        
        // 写操作后立即读取，走主库保证一致性
        User updated = userMapper.selectById(user.getId());
        log.info("Updated user: {}", updated);
    }
    
    /**
     * 复杂业务场景 - 混合读写操作
     */
    @Transactional
    public void processOrder(Order order) {
        // 1. 写操作：创建订单
        orderMapper.insert(order);
        
        // 2. 读操作：查询用户信息（事务中，走主库）
        User user = userMapper.selectById(order.getUserId());
        
        // 3. 写操作：更新用户统计
        user.setOrderCount(user.getOrderCount() + 1);
        userMapper.updateById(user);
        
        // 4. 写操作：扣减库存
        productMapper.decreaseStock(order.getProductId(), order.getQuantity());
    }
}
```

### **3. 数据一致性处理**

```java
/**
 * 主从延迟处理策略
 */
@Component
public class ConsistencyHandler {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 强一致性读取 - 写后读走主库
     */
    public <T> T readAfterWrite(String key, Supplier<T> readFunction, Duration delay) {
        // 检查是否刚刚有写操作
        String writeFlag = "write_flag:" + key;
        if (redisTemplate.hasKey(writeFlag)) {
            // 强制走主库
            DataSourceContextHolder.setMaster();
            try {
                return readFunction.get();
            } finally {
                DataSourceContextHolder.clear();
            }
        } else {
            // 正常走从库
            return readFunction.get();
        }
    }
    
    /**
     * 标记写操作
     */
    public void markWrite(String key, Duration delay) {
        String writeFlag = "write_flag:" + key;
        redisTemplate.opsForValue().set(writeFlag, "1", delay);
    }
    
    /**
     * 延迟双删策略
     */
    @Async
    public void delayedDoubleDelete(String cacheKey, Duration delay) {
        try {
            // 第一次删除缓存
            redisTemplate.delete(cacheKey);
            
            // 等待主从同步
            Thread.sleep(delay.toMillis());
            
            // 第二次删除缓存
            redisTemplate.delete(cacheKey);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            log.error("延迟双删被中断", e);
        }
    }
}
```

### **4. 故障处理和监控**

```java
/**
 * 数据源健康检查
 */
@Component
public class DataSourceHealthChecker {
    
    @Autowired
    private DataSource masterDataSource;
    
    @Autowired
    private List<DataSource> slaveDataSources;
    
    private final Map<DataSource, Boolean> healthStatus = new ConcurrentHashMap<>();
    
    @Scheduled(fixedDelay = 30000) // 每30秒检查一次
    public void checkHealth() {
        // 检查主库
        healthStatus.put(masterDataSource, isHealthy(masterDataSource));
        
        // 检查从库
        for (DataSource slave : slaveDataSources) {
            healthStatus.put(slave, isHealthy(slave));
        }
        
        // 记录健康状态
        logHealthStatus();
    }
    
    private boolean isHealthy(DataSource dataSource) {
        try (Connection conn = dataSource.getConnection()) {
            return conn.isValid(5); // 5秒超时
        } catch (SQLException e) {
            log.error("数据源健康检查失败", e);
            return false;
        }
    }
    
    public List<DataSource> getHealthySlaves() {
        return slaveDataSources.stream()
            .filter(ds -> healthStatus.getOrDefault(ds, false))
            .collect(Collectors.toList());
    }
    
    private void logHealthStatus() {
        healthStatus.forEach((ds, healthy) -> {
            String status = healthy ? "健康" : "异常";
            log.info("数据源 {} 状态: {}", ds.toString(), status);
        });
    }
}
```

### **读写分离注意事项**

**1. 数据一致性问题**
- **主从延迟**：通常在毫秒到秒级别
- **解决方案**：
  - 写后读走主库
  - 使用缓存标记
  - 延迟双删策略
  - 强制路由注解

**2. 事务处理**
- **事务中的读操作**：必须走主库
- **分布式事务**：需要特殊处理
- **事务隔离级别**：影响一致性

**3. 连接池管理**
- **连接数配置**：主库写多读少，从库读多
- **连接超时**：设置合理的超时时间
- **连接泄漏**：确保连接正确释放

**4. 故障处理**
- **主库故障**：需要主从切换机制
- **从库故障**：自动剔除故障节点
- **网络分区**：处理脑裂问题

**5. 监控指标**
- **主从延迟时间**
- **读写QPS分布**
- **连接池使用率**
- **故障切换次数**

**最佳实践：**

1. **合理设计读写比例**：确保从库能承担大部分读请求
2. **选择合适的复制方式**：异步复制vs半同步复制
3. **做好监控告警**：及时发现主从延迟和故障
4. **制定故障预案**：主从切换、数据恢复流程
5. **定期演练**：验证故障处理流程的有效性

**总结：**

读写分离是提升数据库性能的重要手段，但需要处理好数据一致性、故障转移、监控告警等问题。选择合适的实现方案，做好架构设计和运维管理，才能发挥读写分离的最大价值。

### 4. 分布式事务相关

**Q10: 分布式事务的解决方案有哪些？**

**答案：**

分布式事务是分布式系统中的核心问题，需要在多个服务或数据库之间保证数据的一致性。不同的解决方案适用于不同的业务场景和一致性要求。

### **分布式事务解决方案对比**

| 方案 | 一致性 | 性能 | 复杂度 | 适用场景 | 优点 | 缺点 |
|------|--------|------|--------|----------|------|------|
| **2PC** | 强一致性 | 低 | 中等 | 金融交易 | 数据强一致 | 阻塞、单点故障 |
| **3PC** | 强一致性 | 低 | 高 | 关键业务 | 减少阻塞 | 复杂度高 |
| **TCC** | 最终一致性 | 高 | 高 | 电商订单 | 性能好、灵活 | 业务侵入性大 |
| **Saga** | 最终一致性 | 高 | 中等 | 长流程业务 | 支持长事务 | 补偿复杂 |
| **本地消息表** | 最终一致性 | 高 | 低 | 异步处理 | 简单可靠 | 需要定时任务 |
| **MQ事务消息** | 最终一致性 | 高 | 低 | 解耦场景 | 高性能 | 依赖MQ |
| **最大努力通知** | 最终一致性 | 高 | 低 | 通知类业务 | 实现简单 | 可能丢失 |

### **1. 2PC（两阶段提交）**

**核心思想：** 通过协调者统一管理所有参与者的提交或回滚

```java
/**
 * 2PC协调者实现
 */
@Component
public class TwoPhaseCommitCoordinator {
    
    private final List<TransactionParticipant> participants = new ArrayList<>();
    
    /**
     * 执行分布式事务
     */
    public boolean executeTransaction(DistributedTransaction transaction) {
        String transactionId = UUID.randomUUID().toString();
        
        try {
            // 第一阶段：准备阶段
            if (!preparePhase(transactionId, transaction)) {
                // 准备失败，回滚所有参与者
                rollbackAll(transactionId);
                return false;
            }
            
            // 第二阶段：提交阶段
            commitAll(transactionId);
            return true;
            
        } catch (Exception e) {
            log.error("分布式事务执行失败: {}", transactionId, e);
            rollbackAll(transactionId);
            return false;
        }
    }
    
    /**
     * 第一阶段：准备阶段
     */
    private boolean preparePhase(String transactionId, DistributedTransaction transaction) {
        log.info("开始准备阶段: {}", transactionId);
        
        for (TransactionParticipant participant : participants) {
            try {
                boolean prepared = participant.prepare(transactionId, transaction);
                if (!prepared) {
                    log.warn("参与者准备失败: {} - {}", participant.getName(), transactionId);
                    return false;
                }
            } catch (Exception e) {
                log.error("参与者准备异常: {} - {}", participant.getName(), transactionId, e);
                return false;
            }
        }
        
        log.info("准备阶段完成: {}", transactionId);
        return true;
    }
    
    /**
     * 第二阶段：提交阶段
     */
    private void commitAll(String transactionId) {
        log.info("开始提交阶段: {}", transactionId);
        
        for (TransactionParticipant participant : participants) {
            try {
                participant.commit(transactionId);
                log.info("参与者提交成功: {} - {}", participant.getName(), transactionId);
            } catch (Exception e) {
                log.error("参与者提交失败: {} - {}", participant.getName(), transactionId, e);
                // 提交阶段失败需要重试或人工介入
            }
        }
    }
    
    /**
     * 回滚所有参与者
     */
    private void rollbackAll(String transactionId) {
        log.info("开始回滚阶段: {}", transactionId);
        
        for (TransactionParticipant participant : participants) {
            try {
                participant.rollback(transactionId);
                log.info("参与者回滚成功: {} - {}", participant.getName(), transactionId);
            } catch (Exception e) {
                log.error("参与者回滚失败: {} - {}", participant.getName(), transactionId, e);
            }
        }
    }
}

/**
 * 事务参与者接口
 */
public interface TransactionParticipant {
    String getName();
    boolean prepare(String transactionId, DistributedTransaction transaction);
    void commit(String transactionId);
    void rollback(String transactionId);
}

/**
 * 数据库事务参与者
 */
@Component
public class DatabaseParticipant implements TransactionParticipant {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    private final Map<String, Connection> transactionConnections = new ConcurrentHashMap<>();
    
    @Override
    public String getName() {
        return "DatabaseParticipant";
    }
    
    @Override
    public boolean prepare(String transactionId, DistributedTransaction transaction) {
        try {
            Connection conn = jdbcTemplate.getDataSource().getConnection();
            conn.setAutoCommit(false);
            
            // 执行业务SQL
            PreparedStatement stmt = conn.prepareStatement(transaction.getSql());
            stmt.executeUpdate();
            
            // 保存连接，等待提交或回滚
            transactionConnections.put(transactionId, conn);
            
            return true;
        } catch (SQLException e) {
            log.error("数据库准备阶段失败: {}", transactionId, e);
            return false;
        }
    }
    
    @Override
    public void commit(String transactionId) {
        Connection conn = transactionConnections.remove(transactionId);
        if (conn != null) {
            try {
                conn.commit();
                conn.close();
            } catch (SQLException e) {
                log.error("数据库提交失败: {}", transactionId, e);
            }
        }
    }
    
    @Override
    public void rollback(String transactionId) {
        Connection conn = transactionConnections.remove(transactionId);
        if (conn != null) {
            try {
                conn.rollback();
                conn.close();
            } catch (SQLException e) {
                log.error("数据库回滚失败: {}", transactionId, e);
            }
        }
    }
}
```

### **2. TCC（Try-Confirm-Cancel）**

**核心思想：** 将业务逻辑分为三个阶段，通过业务补偿实现最终一致性

```java
/**
 * TCC事务管理器
 */
@Component
public class TccTransactionManager {
    
    @Autowired
    private List<TccParticipant> participants;
    
    @Autowired
    private TransactionLogService transactionLogService;
    
    /**
     * 执行TCC事务
     */
    public boolean executeTccTransaction(TccTransactionContext context) {
        String transactionId = context.getTransactionId();
        
        // 记录事务开始
        transactionLogService.logTransactionStart(transactionId, participants);
        
        try {
            // Try阶段
            if (!tryPhase(context)) {
                // Try失败，执行Cancel
                cancelPhase(context);
                return false;
            }
            
            // Confirm阶段
            confirmPhase(context);
            
            // 记录事务成功
            transactionLogService.logTransactionSuccess(transactionId);
            return true;
            
        } catch (Exception e) {
            log.error("TCC事务执行失败: {}", transactionId, e);
            
            // 异常时执行Cancel
            cancelPhase(context);
            
            // 记录事务失败
            transactionLogService.logTransactionFailure(transactionId, e.getMessage());
            return false;
        }
    }
    
    /**
     * Try阶段：尝试执行业务，预留资源
     */
    private boolean tryPhase(TccTransactionContext context) {
        log.info("开始Try阶段: {}", context.getTransactionId());
        
        for (TccParticipant participant : participants) {
            try {
                boolean result = participant.tryExecute(context);
                if (!result) {
                    log.warn("Try阶段失败: {} - {}", participant.getClass().getSimpleName(), 
                            context.getTransactionId());
                    return false;
                }
                
                // 记录Try成功
                transactionLogService.logParticipantTry(context.getTransactionId(), 
                        participant.getClass().getSimpleName(), true);
                        
            } catch (Exception e) {
                log.error("Try阶段异常: {} - {}", participant.getClass().getSimpleName(), 
                        context.getTransactionId(), e);
                return false;
            }
        }
        
        return true;
    }
    
    /**
     * Confirm阶段：确认执行，提交资源
     */
    private void confirmPhase(TccTransactionContext context) {
        log.info("开始Confirm阶段: {}", context.getTransactionId());
        
        for (TccParticipant participant : participants) {
            try {
                participant.confirmExecute(context);
                
                // 记录Confirm成功
                transactionLogService.logParticipantConfirm(context.getTransactionId(), 
                        participant.getClass().getSimpleName(), true);
                        
            } catch (Exception e) {
                log.error("Confirm阶段异常: {} - {}", participant.getClass().getSimpleName(), 
                        context.getTransactionId(), e);
                
                // Confirm失败需要重试
                scheduleRetry(context, participant, "confirm");
            }
        }
    }
    
    /**
     * Cancel阶段：取消执行，释放资源
     */
    private void cancelPhase(TccTransactionContext context) {
        log.info("开始Cancel阶段: {}", context.getTransactionId());
        
        for (TccParticipant participant : participants) {
            try {
                participant.cancelExecute(context);
                
                // 记录Cancel成功
                transactionLogService.logParticipantCancel(context.getTransactionId(), 
                        participant.getClass().getSimpleName(), true);
                        
            } catch (Exception e) {
                log.error("Cancel阶段异常: {} - {}", participant.getClass().getSimpleName(), 
                        context.getTransactionId(), e);
                
                // Cancel失败需要重试
                scheduleRetry(context, participant, "cancel");
            }
        }
    }
    
    /**
     * 调度重试
     */
    private void scheduleRetry(TccTransactionContext context, TccParticipant participant, String phase) {
        // 实现重试逻辑
    }
}

/**
 * TCC参与者接口
 */
public interface TccParticipant {
    boolean tryExecute(TccTransactionContext context);
    void confirmExecute(TccTransactionContext context);
    void cancelExecute(TccTransactionContext context);
}

/**
 * 账户服务TCC实现
 */
@Component
public class AccountTccParticipant implements TccParticipant {
    
    @Autowired
    private AccountService accountService;
    
    @Override
    public boolean tryExecute(TccTransactionContext context) {
        TransferRequest request = context.getTransferRequest();
        
        try {
            // 冻结转出账户金额
            boolean frozen = accountService.freezeAmount(
                request.getFromAccountId(), 
                request.getAmount(),
                context.getTransactionId()
            );
            
            if (!frozen) {
                log.warn("账户余额不足，冻结失败: {} - {}", 
                        request.getFromAccountId(), request.getAmount());
                return false;
            }
            
            log.info("账户金额冻结成功: {} - {} - {}", 
                    request.getFromAccountId(), request.getAmount(), context.getTransactionId());
            return true;
            
        } catch (Exception e) {
            log.error("账户Try阶段失败", e);
            return false;
        }
    }
    
    @Override
    public void confirmExecute(TccTransactionContext context) {
        TransferRequest request = context.getTransferRequest();
        
        try {
            // 扣减转出账户金额（从冻结金额中扣减）
            accountService.deductFrozenAmount(
                request.getFromAccountId(), 
                request.getAmount(),
                context.getTransactionId()
            );
            
            // 增加转入账户金额
            accountService.addAmount(
                request.getToAccountId(), 
                request.getAmount()
            );
            
            log.info("账户转账确认成功: {} -> {} - {}", 
                    request.getFromAccountId(), request.getToAccountId(), request.getAmount());
                    
        } catch (Exception e) {
            log.error("账户Confirm阶段失败", e);
            throw new RuntimeException("账户确认失败", e);
        }
    }
    
    @Override
    public void cancelExecute(TccTransactionContext context) {
        TransferRequest request = context.getTransferRequest();
        
        try {
            // 解冻转出账户金额
            accountService.unfreezeAmount(
                request.getFromAccountId(), 
                request.getAmount(),
                context.getTransactionId()
            );
            
            log.info("账户金额解冻成功: {} - {} - {}", 
                    request.getFromAccountId(), request.getAmount(), context.getTransactionId());
                    
        } catch (Exception e) {
            log.error("账户Cancel阶段失败", e);
            throw new RuntimeException("账户取消失败", e);
        }
    }
}
```

### **3. Saga模式**

**核心思想：** 将长事务拆分为多个本地事务，通过补偿操作实现最终一致性

```java
/**
 * Saga事务管理器
 */
@Component
public class SagaTransactionManager {
    
    /**
     * 执行Saga事务
     */
    public boolean executeSaga(SagaDefinition sagaDefinition) {
        String sagaId = UUID.randomUUID().toString();
        List<SagaStep> executedSteps = new ArrayList<>();
        
        try {
            // 顺序执行所有步骤
            for (SagaStep step : sagaDefinition.getSteps()) {
                log.info("执行Saga步骤: {} - {}", step.getName(), sagaId);
                
                boolean success = step.execute(sagaId);
                if (success) {
                    executedSteps.add(step);
                } else {
                    log.warn("Saga步骤执行失败: {} - {}", step.getName(), sagaId);
                    
                    // 执行补偿操作
                    compensate(executedSteps, sagaId);
                    return false;
                }
            }
            
            log.info("Saga事务执行成功: {}", sagaId);
            return true;
            
        } catch (Exception e) {
            log.error("Saga事务执行异常: {}", sagaId, e);
            
            // 执行补偿操作
            compensate(executedSteps, sagaId);
            return false;
        }
    }
    
    /**
     * 执行补偿操作
     */
    private void compensate(List<SagaStep> executedSteps, String sagaId) {
        log.info("开始Saga补偿: {}", sagaId);
        
        // 逆序执行补偿操作
        Collections.reverse(executedSteps);
        
        for (SagaStep step : executedSteps) {
            try {
                step.compensate(sagaId);
                log.info("Saga补偿成功: {} - {}", step.getName(), sagaId);
            } catch (Exception e) {
                log.error("Saga补偿失败: {} - {}", step.getName(), sagaId, e);
                // 补偿失败需要重试或人工介入
            }
        }
    }
}

/**
 * Saga步骤接口
 */
public interface SagaStep {
    String getName();
    boolean execute(String sagaId);
    void compensate(String sagaId);
}

/**
 * 订单创建步骤
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
    public boolean execute(String sagaId) {
        try {
            Order order = orderService.createOrder(sagaId);
            log.info("订单创建成功: {} - {}", order.getId(), sagaId);
            return true;
        } catch (Exception e) {
            log.error("订单创建失败: {}", sagaId, e);
            return false;
        }
    }
    
    @Override
    public void compensate(String sagaId) {
        try {
            orderService.cancelOrder(sagaId);
            log.info("订单取消成功: {}", sagaId);
        } catch (Exception e) {
            log.error("订单取消失败: {}", sagaId, e);
            throw new RuntimeException("订单补偿失败", e);
        }
    }
}
```

### **4. 本地消息表**

```java
/**
 * 本地消息表实现
 */
@Service
public class LocalMessageTransactionService {
    
    @Autowired
    private LocalMessageMapper messageMapper;
    
    @Autowired
    private OrderMapper orderMapper;
    
    @Autowired
    private MessageProducer messageProducer;
    
    /**
     * 创建订单并发送消息（本地事务）
     */
    @Transactional
    public void createOrderWithMessage(Order order) {
        // 1. 创建订单
        orderMapper.insert(order);
        
        // 2. 插入本地消息表
        LocalMessage message = LocalMessage.builder()
            .id(UUID.randomUUID().toString())
            .topic("order_created")
            .content(JSON.toJSONString(order))
            .status(MessageStatus.PENDING)
            .createTime(new Date())
            .retryCount(0)
            .build();
            
        messageMapper.insert(message);
        
        log.info("订单和消息创建成功: {}", order.getId());
    }
    
    /**
     * 定时发送待发送的消息
     */
    @Scheduled(fixedDelay = 5000)
    public void sendPendingMessages() {
        List<LocalMessage> pendingMessages = messageMapper.findPendingMessages(100);
        
        for (LocalMessage message : pendingMessages) {
            try {
                // 发送消息
                messageProducer.send(message.getTopic(), message.getContent());
                
                // 更新消息状态为已发送
                message.setStatus(MessageStatus.SENT);
                message.setSentTime(new Date());
                messageMapper.updateById(message);
                
                log.info("消息发送成功: {}", message.getId());
                
            } catch (Exception e) {
                log.error("消息发送失败: {}", message.getId(), e);
                
                // 增加重试次数
                message.setRetryCount(message.getRetryCount() + 1);
                
                if (message.getRetryCount() >= 3) {
                    // 超过重试次数，标记为失败
                    message.setStatus(MessageStatus.FAILED);
                }
                
                messageMapper.updateById(message);
            }
        }
    }
}
```

### **5. MQ事务消息**

```java
/**
 * RocketMQ事务消息实现
 */
@Component
public class TransactionMessageProducer {
    
    private TransactionMQProducer producer;
    
    @PostConstruct
    public void init() {
        producer = new TransactionMQProducer("transaction_producer_group");
        producer.setNamesrvAddr("localhost:9876");
        
        // 设置事务监听器
        producer.setTransactionListener(new OrderTransactionListener());
        
        try {
            producer.start();
        } catch (MQClientException e) {
            log.error("事务消息生产者启动失败", e);
        }
    }
    
    /**
     * 发送事务消息
     */
    public void sendTransactionMessage(Order order) {
        Message message = new Message("order_topic", JSON.toJSONString(order).getBytes());
        
        try {
            TransactionSendResult result = producer.sendMessageInTransaction(message, order);
            log.info("事务消息发送结果: {}", result.getSendStatus());
        } catch (MQClientException e) {
            log.error("事务消息发送失败", e);
        }
    }
    
    /**
     * 事务监听器
     */
    private class OrderTransactionListener implements TransactionListener {
        
        @Override
        public LocalTransactionState executeLocalTransaction(Message msg, Object arg) {
            Order order = (Order) arg;
            
            try {
                // 执行本地事务
                orderService.createOrder(order);
                
                log.info("本地事务执行成功: {}", order.getId());
                return LocalTransactionState.COMMIT_MESSAGE;
                
            } catch (Exception e) {
                log.error("本地事务执行失败: {}", order.getId(), e);
                return LocalTransactionState.ROLLBACK_MESSAGE;
            }
        }
        
        @Override
        public LocalTransactionState checkLocalTransaction(MessageExt msg) {
            // 检查本地事务状态
            String orderId = extractOrderId(msg);
            
            Order order = orderService.findById(orderId);
            if (order != null) {
                return LocalTransactionState.COMMIT_MESSAGE;
            } else {
                return LocalTransactionState.ROLLBACK_MESSAGE;
            }
        }
    }
}
```

### **最佳实践和选择建议**

**1. 选择原则：**
- **强一致性要求**：选择2PC/3PC
- **性能要求高**：选择TCC/Saga
- **业务解耦**：选择消息事务
- **实现简单**：选择本地消息表

**2. 实施建议：**
- 优先考虑业务拆分，避免分布式事务
- 根据业务特点选择合适的方案
- 做好监控和补偿机制
- 考虑幂等性和重试机制

**总结：**

分布式事务没有银弹，需要根据具体的业务场景、一致性要求、性能要求来选择合适的解决方案。在实际应用中，往往需要组合使用多种方案来满足不同的业务需求。

**Q11: TCC模式的三个阶段分别做什么？**

**答案：**

TCC（Try-Confirm-Cancel）是一种分布式事务解决方案，通过将业务逻辑分为三个阶段来实现最终一致性。每个阶段都有明确的职责和实现要求。

### **TCC三个阶段详解**

| 阶段 | 目标 | 操作类型 | 失败处理 | 幂等性要求 | 超时处理 |
|------|------|----------|----------|------------|----------|
| **Try** | 资源预留 | 预留/锁定 | 直接返回失败 | 必须幂等 | 快速失败 |
| **Confirm** | 资源确认 | 提交/释放 | 重试直到成功 | 必须幂等 | 异步重试 |
| **Cancel** | 资源释放 | 回滚/释放 | 重试直到成功 | 必须幂等 | 异步重试 |

### **1. Try阶段（尝试阶段）**

**核心职责：**
- 检查业务条件是否满足
- 预留必要的业务资源
- 不做实际的业务提交
- 为后续的Confirm或Cancel做准备

**实现要点：**
- 必须具备幂等性
- 需要快速响应，避免长时间阻塞
- 失败时直接返回，不需要补偿
- 预留的资源要有超时机制

```java
/**
 * 账户服务Try阶段实现
 */
@Service
public class AccountTryService {
    
    @Autowired
    private AccountMapper accountMapper;
    
    @Autowired
    private FreezeRecordMapper freezeRecordMapper;
    
    /**
     * Try阶段：冻结账户金额
     */
    public boolean tryFreeze(String accountId, BigDecimal amount, String transactionId) {
        log.info("Try阶段开始 - 账户: {}, 金额: {}, 事务ID: {}", accountId, amount, transactionId);
        
        try {
            // 1. 幂等性检查
            FreezeRecord existingRecord = freezeRecordMapper.findByTransactionId(transactionId);
            if (existingRecord != null) {
                log.info("Try阶段已执行，返回之前结果: {}", existingRecord.getStatus());
                return "SUCCESS".equals(existingRecord.getStatus());
            }
            
            // 2. 检查账户余额
            Account account = accountMapper.findById(accountId);
            if (account == null) {
                log.warn("账户不存在: {}", accountId);
                recordFreezeResult(transactionId, accountId, amount, "FAILED", "账户不存在");
                return false;
            }
            
            if (account.getBalance().compareTo(amount) < 0) {
                log.warn("账户余额不足: {} < {}", account.getBalance(), amount);
                recordFreezeResult(transactionId, accountId, amount, "FAILED", "余额不足");
                return false;
            }
            
            // 3. 冻结金额（使用数据库行锁确保并发安全）
            int updated = accountMapper.freezeAmount(accountId, amount);
            if (updated == 0) {
                log.warn("冻结金额失败，可能余额不足: {}", accountId);
                recordFreezeResult(transactionId, accountId, amount, "FAILED", "冻结失败");
                return false;
            }
            
            // 4. 记录冻结记录
            recordFreezeResult(transactionId, accountId, amount, "SUCCESS", "冻结成功");
            
            log.info("Try阶段成功 - 账户: {}, 冻结金额: {}", accountId, amount);
            return true;
            
        } catch (Exception e) {
            log.error("Try阶段异常 - 账户: {}, 金额: {}", accountId, amount, e);
            recordFreezeResult(transactionId, accountId, amount, "FAILED", e.getMessage());
            return false;
        }
    }
    
    /**
     * 记录冻结结果
     */
    private void recordFreezeResult(String transactionId, String accountId, 
                                   BigDecimal amount, String status, String message) {
        FreezeRecord record = FreezeRecord.builder()
            .transactionId(transactionId)
            .accountId(accountId)
            .amount(amount)
            .status(status)
            .message(message)
            .createTime(new Date())
            .expireTime(new Date(System.currentTimeMillis() + 30 * 60 * 1000)) // 30分钟超时
            .build();
            
        freezeRecordMapper.insert(record);
    }
}
```

### **2. Confirm阶段（确认阶段）**

**核心职责：**
- 使用Try阶段预留的资源
- 执行实际的业务操作
- 完成最终的业务提交
- 清理Try阶段的临时数据

**实现要点：**
- 必须具备幂等性
- 失败时需要重试，直到成功
- 不能再失败，因为Try阶段已经成功
- 需要处理超时和异常情况

```java
/**
 * 账户服务Confirm阶段实现
 */
@Service
public class AccountConfirmService {
    
    @Autowired
    private AccountMapper accountMapper;
    
    @Autowired
    private FreezeRecordMapper freezeRecordMapper;
    
    /**
     * Confirm阶段：确认转账，扣减冻结金额
     */
    public void confirmTransfer(String fromAccountId, String toAccountId, 
                               BigDecimal amount, String transactionId) {
        log.info("Confirm阶段开始 - 从账户: {}, 到账户: {}, 金额: {}", 
                fromAccountId, toAccountId, amount);
        
        try {
            // 1. 幂等性检查
            if (isConfirmed(transactionId)) {
                log.info("Confirm阶段已执行: {}", transactionId);
                return;
            }
            
            // 2. 扣减转出账户的冻结金额
            int deductResult = accountMapper.deductFrozenAmount(fromAccountId, amount);
            if (deductResult == 0) {
                throw new RuntimeException("扣减冻结金额失败");
            }
            
            // 3. 增加转入账户金额
            accountMapper.addAmount(toAccountId, amount);
            
            // 4. 清理冻结记录
            freezeRecordMapper.deleteByTransactionId(transactionId);
            
            // 5. 标记Confirm完成
            markConfirmed(transactionId);
            
            log.info("Confirm阶段成功 - 转账完成: {} -> {}", fromAccountId, toAccountId);
                    
        } catch (Exception e) {
            log.error("Confirm阶段异常: {}", transactionId, e);
            throw new RuntimeException("Confirm阶段失败", e);
        }
    }
}
```

### **3. Cancel阶段（取消阶段）**

**核心职责：**
- 释放Try阶段预留的资源
- 回滚Try阶段的操作
- 清理临时数据
- 恢复业务状态

**实现要点：**
- 必须具备幂等性
- 失败时需要重试，直到成功
- 需要处理Try阶段可能未执行的情况
- 要考虑部分成功的场景

```java
/**
 * 账户服务Cancel阶段实现
 */
@Service
public class AccountCancelService {
    
    @Autowired
    private AccountMapper accountMapper;
    
    @Autowired
    private FreezeRecordMapper freezeRecordMapper;
    
    /**
     * Cancel阶段：取消转账，释放冻结金额
     */
    public void cancelTransfer(String accountId, BigDecimal amount, String transactionId) {
        log.info("Cancel阶段开始 - 账户: {}, 金额: {}", accountId, amount);
        
        try {
            // 1. 幂等性检查
            if (isCancelled(transactionId)) {
                log.info("Cancel阶段已执行: {}", transactionId);
                return;
            }
            
            // 2. 检查Try阶段的执行情况
            FreezeRecord freezeRecord = freezeRecordMapper.findByTransactionId(transactionId);
            
            if (freezeRecord != null && "SUCCESS".equals(freezeRecord.getStatus())) {
                // Try阶段成功，需要释放冻结金额
                accountMapper.unfreezeAmount(accountId, amount);
                freezeRecordMapper.deleteByTransactionId(transactionId);
            }
            
            // 3. 标记Cancel完成
            markCancelled(transactionId);
            
            log.info("Cancel阶段成功 - 冻结金额已释放: {}", accountId);
                    
        } catch (Exception e) {
            log.error("Cancel阶段异常: {}", transactionId, e);
            throw new RuntimeException("Cancel阶段失败", e);
        }
    }
}
```

### **TCC实现的关键技术点**

**1. 幂等性保证：**
```java
@Component
public class TccIdempotentHelper {
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    public boolean isExecuted(String transactionId, String phase) {
        String key = String.format("tcc:%s:%s", phase, transactionId);
        return redisTemplate.hasKey(key);
    }
    
    public void markExecuted(String transactionId, String phase) {
        String key = String.format("tcc:%s:%s", phase, transactionId);
        redisTemplate.opsForValue().set(key, "1", Duration.ofHours(24));
    }
}
```

**2. 超时处理：**
```java
@Component
public class TccTimeoutHandler {
    
    @Scheduled(fixedDelay = 60000)
    public void handleTimeoutTransactions() {
        List<FreezeRecord> timeoutRecords = freezeRecordMapper.findTimeoutRecords();
        
        for (FreezeRecord record : timeoutRecords) {
            try {
                accountCancelService.cancelTransfer(
                    record.getAccountId(), 
                    record.getAmount(), 
                    record.getTransactionId()
                );
            } catch (Exception e) {
                log.error("超时事务取消失败: {}", record.getTransactionId(), e);
            }
        }
    }
}
```

**总结：**

TCC模式通过三个阶段的精心设计，在保证数据最终一致性的同时，提供了较好的性能表现。关键在于正确理解每个阶段的职责，实现好幂等性、超时处理和重试机制。

**Q12: 如何保证分布式系统的数据一致性？**

**答案：**

分布式系统的数据一致性是一个复杂的问题，需要在一致性、可用性和分区容错性之间做出权衡。根据CAP定理，我们无法同时满足所有三个特性，因此需要根据业务需求选择合适的一致性策略。

### **数据一致性级别对比**

| 一致性级别 | 特点 | 实现方式 | 适用场景 | 优点 | 缺点 |
|------------|------|----------|----------|------|------|
| **强一致性** | 所有节点同时看到相同数据 | 分布式锁、2PC | 金融交易 | 数据准确 | 性能差、可用性低 |
| **弱一致性** | 允许短期不一致 | 异步复制 | 社交媒体 | 高性能 | 数据可能不准确 |
| **最终一致性** | 最终会达到一致 | 补偿机制、事件驱动 | 电商系统 | 平衡性能和一致性 | 实现复杂 |
| **因果一致性** | 保证因果关系 | 向量时钟 | 协作系统 | 符合直觉 | 实现复杂 |
| **会话一致性** | 单个会话内一致 | 会话绑定 | 用户系统 | 用户体验好 | 局部一致性 |

### **1. 强一致性实现**

**核心思想：** 确保所有节点在同一时刻看到相同的数据

```java
/**
 * 基于分布式锁的强一致性实现
 */
@Service
public class StrongConsistencyService {
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    @Autowired
    private List<DataNode> dataNodes;
    
    /**
     * 强一致性写操作
     */
    public boolean writeWithStrongConsistency(String key, String value) {
        String lockKey = "lock:" + key;
        String lockValue = UUID.randomUUID().toString();
        
        try {
            // 1. 获取分布式锁
            boolean lockAcquired = acquireDistributedLock(lockKey, lockValue, 30);
            if (!lockAcquired) {
                log.warn("获取分布式锁失败: {}", key);
                return false;
            }
            
            // 2. 向所有节点写入数据
            List<CompletableFuture<Boolean>> futures = new ArrayList<>();
            for (DataNode node : dataNodes) {
                CompletableFuture<Boolean> future = CompletableFuture.supplyAsync(() -> {
                    try {
                        return node.write(key, value);
                    } catch (Exception e) {
                        log.error("节点写入失败: {} - {}", node.getId(), key, e);
                        return false;
                    }
                });
                futures.add(future);
            }
            
            // 3. 等待所有节点写入完成
            boolean allSuccess = true;
            for (CompletableFuture<Boolean> future : futures) {
                try {
                    Boolean result = future.get(5, TimeUnit.SECONDS);
                    if (!result) {
                        allSuccess = false;
                        break;
                    }
                } catch (Exception e) {
                    log.error("等待写入结果超时: {}", key, e);
                    allSuccess = false;
                    break;
                }
            }
            
            if (!allSuccess) {
                // 4. 如果有节点失败，回滚所有节点
                rollbackAllNodes(key);
                return false;
            }
            
            log.info("强一致性写入成功: {} = {}", key, value);
            return true;
            
        } finally {
            // 5. 释放分布式锁
            releaseDistributedLock(lockKey, lockValue);
        }
    }
    
    /**
     * 强一致性读操作
     */
    public String readWithStrongConsistency(String key) {
        // 从主节点读取，确保强一致性
        DataNode primaryNode = getPrimaryNode();
        return primaryNode.read(key);
    }
    
    /**
     * 获取分布式锁
     */
    private boolean acquireDistributedLock(String lockKey, String lockValue, int expireSeconds) {
        String script = 
            "if redis.call('get', KEYS[1]) == false then " +
            "    return redis.call('setex', KEYS[1], ARGV[2], ARGV[1]) " +
            "else " +
            "    return false " +
            "end";
            
        Object result = redisTemplate.execute(new DefaultRedisScript<>(script, Boolean.class), 
                Arrays.asList(lockKey), lockValue, String.valueOf(expireSeconds));
        return Boolean.TRUE.equals(result);
    }
    
    /**
     * 释放分布式锁
     */
    private void releaseDistributedLock(String lockKey, String lockValue) {
        String script = 
            "if redis.call('get', KEYS[1]) == ARGV[1] then " +
            "    return redis.call('del', KEYS[1]) " +
            "else " +
            "    return 0 " +
            "end";
            
        redisTemplate.execute(new DefaultRedisScript<>(script, Long.class), 
                Arrays.asList(lockKey), lockValue);
    }
    
    /**
     * 回滚所有节点
     */
    private void rollbackAllNodes(String key) {
        for (DataNode node : dataNodes) {
            try {
                node.delete(key);
            } catch (Exception e) {
                log.error("节点回滚失败: {} - {}", node.getId(), key, e);
            }
        }
    }
}
```

### **2. 最终一致性实现**

**核心思想：** 通过异步复制和补偿机制，最终达到数据一致性

```java
/**
 * 基于事件驱动的最终一致性实现
 */
@Service
public class EventualConsistencyService {
    
    @Autowired
    private EventPublisher eventPublisher;
    
    @Autowired
    private ConsistencyChecker consistencyChecker;
    
    @Autowired
    private CompensationHandler compensationHandler;
    
    /**
     * 最终一致性写操作
     */
    @Transactional
    public void writeWithEventualConsistency(String key, String value) {
        try {
            // 1. 先写入主节点
            DataNode primaryNode = getPrimaryNode();
            primaryNode.write(key, value);
            
            // 2. 发布数据变更事件
            DataChangeEvent event = DataChangeEvent.builder()
                .eventId(UUID.randomUUID().toString())
                .key(key)
                .value(value)
                .timestamp(System.currentTimeMillis())
                .operation("WRITE")
                .build();
                
            eventPublisher.publish(event);
            
            log.info("主节点写入成功，事件已发布: {} = {}", key, value);
            
        } catch (Exception e) {
            log.error("最终一致性写入失败: {} = {}", key, value, e);
            throw new RuntimeException("写入失败", e);
        }
    }
    
    /**
     * 处理数据变更事件
     */
    @EventListener
    @Async
    public void handleDataChangeEvent(DataChangeEvent event) {
        log.info("处理数据变更事件: {}", event.getEventId());
        
        List<DataNode> secondaryNodes = getSecondaryNodes();
        List<CompletableFuture<Void>> futures = new ArrayList<>();
        
        // 异步复制到所有从节点
        for (DataNode node : secondaryNodes) {
            CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
                try {
                    replicateToNode(node, event);
                } catch (Exception e) {
                    log.error("复制到节点失败: {} - {}", node.getId(), event.getKey(), e);
                    
                    // 记录失败，稍后重试
                    recordReplicationFailure(node.getId(), event);
                }
            });
            futures.add(future);
        }
        
        // 等待所有复制完成（不阻塞主流程）
        CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]))
            .thenRun(() -> {
                log.info("数据复制完成: {}", event.getEventId());
                
                // 启动一致性检查
                scheduleConsistencyCheck(event.getKey());
            })
            .exceptionally(throwable -> {
                log.error("数据复制异常: {}", event.getEventId(), throwable);
                return null;
            });
    }
    
    /**
     * 复制数据到指定节点
     */
    private void replicateToNode(DataNode node, DataChangeEvent event) {
        int maxRetries = 3;
        int retryCount = 0;
        
        while (retryCount < maxRetries) {
            try {
                switch (event.getOperation()) {
                    case "WRITE":
                        node.write(event.getKey(), event.getValue());
                        break;
                    case "DELETE":
                        node.delete(event.getKey());
                        break;
                    default:
                        log.warn("未知操作类型: {}", event.getOperation());
                        return;
                }
                
                log.info("节点复制成功: {} - {}", node.getId(), event.getKey());
                return;
                
            } catch (Exception e) {
                retryCount++;
                log.warn("节点复制失败，重试 {}/{}: {} - {}", 
                        retryCount, maxRetries, node.getId(), event.getKey(), e);
                
                if (retryCount < maxRetries) {
                    try {
                        Thread.sleep(1000 * retryCount); // 指数退避
                    } catch (InterruptedException ie) {
                        Thread.currentThread().interrupt();
                        break;
                    }
                }
            }
        }
        
        throw new RuntimeException("节点复制失败，已达最大重试次数");
    }
    
    /**
     * 一致性检查
     */
    @Scheduled(fixedDelay = 60000) // 每分钟检查一次
    public void performConsistencyCheck() {
        List<String> keysToCheck = getKeysForConsistencyCheck();
        
        for (String key : keysToCheck) {
            try {
                boolean isConsistent = consistencyChecker.checkConsistency(key);
                if (!isConsistent) {
                    log.warn("发现数据不一致: {}", key);
                    
                    // 触发补偿机制
                    compensationHandler.compensate(key);
                }
            } catch (Exception e) {
                log.error("一致性检查失败: {}", key, e);
            }
        }
    }
}

/**
 * 一致性检查器
 */
@Component
public class ConsistencyChecker {
    
    @Autowired
    private List<DataNode> dataNodes;
    
    /**
     * 检查指定key的数据一致性
     */
    public boolean checkConsistency(String key) {
        Map<String, Integer> valueCount = new HashMap<>();
        
        // 从所有节点读取数据
        for (DataNode node : dataNodes) {
            try {
                String value = node.read(key);
                valueCount.merge(value, 1, Integer::sum);
            } catch (Exception e) {
                log.error("从节点读取数据失败: {} - {}", node.getId(), key, e);
            }
        }
        
        // 检查是否所有节点数据一致
        if (valueCount.size() <= 1) {
            return true; // 所有节点数据一致或key不存在
        }
        
        log.warn("数据不一致: {} - {}", key, valueCount);
        return false;
    }
}

/**
 * 补偿处理器
 */
@Component
public class CompensationHandler {
    
    @Autowired
    private List<DataNode> dataNodes;
    
    /**
     * 补偿不一致的数据
     */
    public void compensate(String key) {
        log.info("开始补偿数据: {}", key);
        
        try {
            // 1. 从主节点获取正确的数据
            DataNode primaryNode = getPrimaryNode();
            String correctValue = primaryNode.read(key);
            
            // 2. 将正确数据同步到所有从节点
            List<DataNode> secondaryNodes = getSecondaryNodes();
            for (DataNode node : secondaryNodes) {
                try {
                    String currentValue = node.read(key);
                    if (!Objects.equals(correctValue, currentValue)) {
                        node.write(key, correctValue);
                        log.info("补偿节点数据: {} - {} = {}", node.getId(), key, correctValue);
                    }
                } catch (Exception e) {
                    log.error("补偿节点失败: {} - {}", node.getId(), key, e);
                }
            }
            
            log.info("数据补偿完成: {}", key);
            
        } catch (Exception e) {
            log.error("数据补偿失败: {}", key, e);
        }
    }
}
```

### **3. 会话一致性实现**

```java
/**
 * 会话一致性实现
 */
@Service
public class SessionConsistencyService {
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    /**
     * 会话绑定写操作
     */
    public void writeWithSessionConsistency(String sessionId, String key, String value) {
        // 1. 选择会话绑定的节点
        DataNode sessionNode = getSessionBoundNode(sessionId);
        
        // 2. 写入会话绑定节点
        sessionNode.write(key, value);
        
        // 3. 记录会话数据版本
        String versionKey = String.format("session:%s:version:%s", sessionId, key);
        long version = System.currentTimeMillis();
        redisTemplate.opsForValue().set(versionKey, String.valueOf(version), Duration.ofHours(1));
        
        // 4. 异步复制到其他节点
        asyncReplicateToOtherNodes(sessionNode, key, value, version);
        
        log.info("会话一致性写入: session={}, key={}, value={}", sessionId, key, value);
    }
    
    /**
     * 会话绑定读操作
     */
    public String readWithSessionConsistency(String sessionId, String key) {
        // 从会话绑定的节点读取
        DataNode sessionNode = getSessionBoundNode(sessionId);
        return sessionNode.read(key);
    }
    
    /**
     * 获取会话绑定的节点
     */
    private DataNode getSessionBoundNode(String sessionId) {
        // 使用一致性哈希选择节点
        int hash = sessionId.hashCode();
        int nodeIndex = Math.abs(hash) % dataNodes.size();
        return dataNodes.get(nodeIndex);
    }
}
```

### **4. 数据一致性监控**

```java
/**
 * 数据一致性监控
 */
@Component
public class ConsistencyMonitor {
    
    private final MeterRegistry meterRegistry;
    private final Counter inconsistencyCounter;
    private final Timer consistencyCheckTimer;
    
    public ConsistencyMonitor(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        this.inconsistencyCounter = Counter.builder("data.inconsistency")
            .description("数据不一致次数")
            .register(meterRegistry);
        this.consistencyCheckTimer = Timer.builder("consistency.check.duration")
            .description("一致性检查耗时")
            .register(meterRegistry);
    }
    
    /**
     * 记录数据不一致
     */
    public void recordInconsistency(String key, String reason) {
        inconsistencyCounter.increment(
            Tags.of(
                "key", key,
                "reason", reason
            )
        );
        
        log.warn("数据不一致告警: key={}, reason={}", key, reason);
    }
    
    /**
     * 监控一致性检查
     */
    public <T> T monitorConsistencyCheck(String operation, Supplier<T> supplier) {
        return Timer.Sample.start(meterRegistry)
            .stop(consistencyCheckTimer.tag("operation", operation))
            .recordCallable(supplier::get);
    }
}
```

### **最佳实践和选择建议**

**1. 选择原则：**
- **金融支付**：强一致性（2PC、分布式锁）
- **电商库存**：最终一致性（补偿机制）
- **用户会话**：会话一致性（会话绑定）
- **社交内容**：弱一致性（异步复制）

**2. 实施建议：**
- 根据业务特点选择合适的一致性级别
- 实现完善的监控和告警机制
- 设计好补偿和恢复策略
- 考虑网络分区和节点故障场景

**总结：**

分布式系统的数据一致性需要在性能、可用性和一致性之间做出权衡。关键是理解不同一致性级别的特点，根据业务需求选择合适的实现方案，并建立完善的监控和补偿机制。

### 5. 服务治理相关

**Q13: 什么是熔断器？熔断器的状态有哪些？**

**答案：**

熔断器（Circuit Breaker）是一种保护机制，用于防止分布式系统中的故障传播和雪崩效应。它的设计灵感来源于电路中的断路器，当检测到下游服务异常时，会自动切断对该服务的调用，避免无效请求堆积，保护系统整体稳定性。

### **熔断器状态转换图**

```
     失败率超过阈值
  CLOSED ---------> OPEN
     ↑                ↓
     |                | 超时后
     |                ↓
  成功调用      HALF_OPEN
     ↑                ↓
     |                | 测试失败
     └----------------┘
```

### **熔断器状态详解**

| 状态 | 描述 | 行为 | 转换条件 |
|------|------|------|----------|
| **CLOSED** | 关闭状态（正常） | 允许所有请求通过，统计成功/失败率 | 失败率超过阈值 → OPEN |
| **OPEN** | 开启状态（熔断） | 拒绝所有请求，直接返回错误或降级响应 | 超时时间到达 → HALF_OPEN |
| **HALF_OPEN** | 半开状态（试探） | 允许少量请求通过进行测试 | 测试成功 → CLOSED，测试失败 → OPEN |

### **熔断器核心实现**

```java
/**
 * 自定义熔断器实现
 */
public class CircuitBreaker {
    
    public enum State {
        CLOSED,    // 关闭状态
        OPEN,      // 开启状态
        HALF_OPEN  // 半开状态
    }
    
    private volatile State state = State.CLOSED;
    private final int failureThreshold;           // 失败阈值
    private final int successThreshold;           // 成功阈值（半开状态）
    private final long timeout;                   // 超时时间
    private final AtomicInteger failureCount = new AtomicInteger(0);
    private final AtomicInteger successCount = new AtomicInteger(0);
    private final AtomicInteger requestCount = new AtomicInteger(0);
    private volatile long lastFailureTime;
    
    public CircuitBreaker(int failureThreshold, int successThreshold, long timeout) {
        this.failureThreshold = failureThreshold;
        this.successThreshold = successThreshold;
        this.timeout = timeout;
    }
    
    /**
     * 执行受保护的调用
     */
    public <T> T execute(Supplier<T> supplier, Supplier<T> fallback) throws Exception {
        if (!allowRequest()) {
            log.warn("熔断器开启，执行降级逻辑");
            return fallback.get();
        }
        
        try {
            T result = supplier.get();
            onSuccess();
            return result;
        } catch (Exception e) {
            onFailure();
            throw e;
        }
    }
    
    /**
     * 判断是否允许请求通过
     */
    private boolean allowRequest() {
        if (state == State.CLOSED) {
            return true;
        }
        
        if (state == State.OPEN) {
            // 检查是否超过超时时间
            if (System.currentTimeMillis() - lastFailureTime >= timeout) {
                state = State.HALF_OPEN;
                resetCounts();
                log.info("熔断器状态转换: OPEN -> HALF_OPEN");
                return true;
            }
            return false;
        }
        
        // HALF_OPEN状态，允许少量请求通过
        return requestCount.get() < successThreshold;
    }
    
    /**
     * 处理成功调用
     */
    private void onSuccess() {
        resetFailureCount();
        
        if (state == State.HALF_OPEN) {
            int currentSuccessCount = successCount.incrementAndGet();
            if (currentSuccessCount >= successThreshold) {
                state = State.CLOSED;
                resetCounts();
                log.info("熔断器状态转换: HALF_OPEN -> CLOSED");
            }
        }
    }
    
    /**
     * 处理失败调用
     */
    private void onFailure() {
        lastFailureTime = System.currentTimeMillis();
        int currentFailureCount = failureCount.incrementAndGet();
        
        if (state == State.CLOSED && currentFailureCount >= failureThreshold) {
            state = State.OPEN;
            log.warn("熔断器状态转换: CLOSED -> OPEN，失败次数: {}", currentFailureCount);
        } else if (state == State.HALF_OPEN) {
            state = State.OPEN;
            log.warn("熔断器状态转换: HALF_OPEN -> OPEN");
        }
    }
    
    /**
     * 重置计数器
     */
    private void resetCounts() {
        failureCount.set(0);
        successCount.set(0);
        requestCount.set(0);
    }
    
    /**
     * 重置失败计数
     */
    private void resetFailureCount() {
        failureCount.set(0);
    }
    
    /**
     * 获取当前状态
     */
    public State getState() {
        return state;
    }
    
    /**
     * 获取统计信息
     */
    public CircuitBreakerStats getStats() {
        return CircuitBreakerStats.builder()
            .state(state)
            .failureCount(failureCount.get())
            .successCount(successCount.get())
            .requestCount(requestCount.get())
            .lastFailureTime(lastFailureTime)
            .build();
    }
}
```

### **Spring Cloud Hystrix 实现**

```java
/**
 * 使用Hystrix实现熔断器
 */
@Component
public class UserService {
    
    @Autowired
    private UserServiceClient userServiceClient;
    
    /**
     * 带熔断器的用户查询
     */
    @HystrixCommand(
        commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50"),
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "5000"),
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000")
        },
        fallbackMethod = "getUserFallback"
    )
    public User getUser(Long userId) {
        log.info("调用用户服务获取用户信息: {}", userId);
        return userServiceClient.getUser(userId);
    }
    
    /**
     * 降级方法
     */
    public User getUserFallback(Long userId) {
        log.warn("用户服务熔断，返回默认用户信息: {}", userId);
        return User.builder()
            .id(userId)
            .name("默认用户")
            .email("default@example.com")
            .build();
    }
    
    /**
     * 批量查询用户（带熔断器）
     */
    @HystrixCommand(
        commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "20"),
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60")
        },
        fallbackMethod = "getUsersFallback"
    )
    public List<User> getUsers(List<Long> userIds) {
        log.info("批量查询用户信息: {}", userIds);
        return userServiceClient.getUsers(userIds);
    }
    
    /**
     * 批量查询降级方法
     */
    public List<User> getUsersFallback(List<Long> userIds) {
        log.warn("批量用户查询熔断，返回空列表");
        return Collections.emptyList();
    }
}
```

### **Resilience4j 实现**

```java
/**
 * 使用Resilience4j实现熔断器
 */
@Service
public class OrderService {
    
    private final CircuitBreaker circuitBreaker;
    private final OrderServiceClient orderServiceClient;
    
    public OrderService(OrderServiceClient orderServiceClient) {
        this.orderServiceClient = orderServiceClient;
        
        // 配置熔断器
        this.circuitBreaker = CircuitBreaker.ofDefaults("orderService");
        
        // 自定义配置
        CircuitBreakerConfig config = CircuitBreakerConfig.custom()
            .failureRateThreshold(50)                    // 失败率阈值50%
            .waitDurationInOpenState(Duration.ofSeconds(30))  // 开启状态等待30秒
            .slidingWindowSize(10)                       // 滑动窗口大小
            .minimumNumberOfCalls(5)                     // 最小调用次数
            .permittedNumberOfCallsInHalfOpenState(3)    // 半开状态允许调用次数
            .build();
            
        this.circuitBreaker = CircuitBreaker.of("orderService", config);
        
        // 注册事件监听器
        circuitBreaker.getEventPublisher()
            .onStateTransition(event -> 
                log.info("熔断器状态转换: {} -> {}", 
                    event.getStateTransition().getFromState(),
                    event.getStateTransition().getToState())
            )
            .onFailureRateExceeded(event -> 
                log.warn("失败率超过阈值: {}%", event.getFailureRate())
            );
    }
    
    /**
     * 创建订单（带熔断器）
     */
    public Order createOrder(CreateOrderRequest request) {
        Supplier<Order> decoratedSupplier = CircuitBreaker
            .decorateSupplier(circuitBreaker, () -> {
                log.info("调用订单服务创建订单: {}", request.getProductId());
                return orderServiceClient.createOrder(request);
            });
            
        try {
            return decoratedSupplier.get();
        } catch (Exception e) {
            log.error("创建订单失败，执行降级逻辑: {}", request.getProductId(), e);
            return createOrderFallback(request);
        }
    }
    
    /**
     * 降级方法
     */
    private Order createOrderFallback(CreateOrderRequest request) {
        // 将订单请求存储到消息队列，稍后处理
        orderQueue.offer(request);
        
        return Order.builder()
            .id(generateTempOrderId())
            .status("PENDING")
            .message("订单已提交，正在处理中")
            .build();
    }
    
    /**
     * 获取熔断器状态
     */
    public CircuitBreakerStatus getCircuitBreakerStatus() {
        CircuitBreaker.State state = circuitBreaker.getState();
        CircuitBreaker.Metrics metrics = circuitBreaker.getMetrics();
        
        return CircuitBreakerStatus.builder()
            .state(state.name())
            .failureRate(metrics.getFailureRate())
            .numberOfCalls(metrics.getNumberOfCalls())
            .numberOfFailedCalls(metrics.getNumberOfFailedCalls())
            .numberOfSuccessfulCalls(metrics.getNumberOfSuccessfulCalls())
            .build();
    }
}
```

### **熔断器监控和管理**

```java
/**
 * 熔断器监控服务
 */
@Service
public class CircuitBreakerMonitorService {
    
    private final MeterRegistry meterRegistry;
    private final Map<String, CircuitBreaker> circuitBreakers = new ConcurrentHashMap<>();
    
    public CircuitBreakerMonitorService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }
    
    /**
     * 注册熔断器
     */
    public void registerCircuitBreaker(String name, CircuitBreaker circuitBreaker) {
        circuitBreakers.put(name, circuitBreaker);
        
        // 注册监控指标
        Gauge.builder("circuit_breaker_state")
            .description("熔断器状态")
            .tag("name", name)
            .register(meterRegistry, circuitBreaker, cb -> {
                switch (cb.getState()) {
                    case CLOSED: return 0;
                    case OPEN: return 1;
                    case HALF_OPEN: return 0.5;
                    default: return -1;
                }
            });
            
        Gauge.builder("circuit_breaker_failure_rate")
            .description("熔断器失败率")
            .tag("name", name)
            .register(meterRegistry, circuitBreaker, 
                cb -> cb.getMetrics().getFailureRate());
    }
    
    /**
     * 获取所有熔断器状态
     */
    public Map<String, CircuitBreakerInfo> getAllCircuitBreakerInfo() {
        Map<String, CircuitBreakerInfo> result = new HashMap<>();
        
        circuitBreakers.forEach((name, cb) -> {
            CircuitBreaker.Metrics metrics = cb.getMetrics();
            
            CircuitBreakerInfo info = CircuitBreakerInfo.builder()
                .name(name)
                .state(cb.getState().name())
                .failureRate(metrics.getFailureRate())
                .numberOfCalls(metrics.getNumberOfCalls())
                .numberOfFailedCalls(metrics.getNumberOfFailedCalls())
                .numberOfSuccessfulCalls(metrics.getNumberOfSuccessfulCalls())
                .build();
                
            result.put(name, info);
        });
        
        return result;
    }
    
    /**
     * 手动触发熔断器状态转换
     */
    public void transitionToOpenState(String name) {
        CircuitBreaker circuitBreaker = circuitBreakers.get(name);
        if (circuitBreaker != null) {
            circuitBreaker.transitionToOpenState();
            log.info("手动触发熔断器开启: {}", name);
        }
    }
    
    /**
     * 手动关闭熔断器
     */
    public void transitionToClosedState(String name) {
        CircuitBreaker circuitBreaker = circuitBreakers.get(name);
        if (circuitBreaker != null) {
            circuitBreaker.transitionToClosedState();
            log.info("手动关闭熔断器: {}", name);
        }
    }
}

/**
 * 熔断器管理控制器
 */
@RestController
@RequestMapping("/api/circuit-breaker")
public class CircuitBreakerController {
    
    @Autowired
    private CircuitBreakerMonitorService monitorService;
    
    /**
     * 获取所有熔断器状态
     */
    @GetMapping("/status")
    public ResponseEntity<Map<String, CircuitBreakerInfo>> getCircuitBreakerStatus() {
        Map<String, CircuitBreakerInfo> status = monitorService.getAllCircuitBreakerInfo();
        return ResponseEntity.ok(status);
    }
    
    /**
     * 手动开启熔断器
     */
    @PostMapping("/{name}/open")
    public ResponseEntity<String> openCircuitBreaker(@PathVariable String name) {
        monitorService.transitionToOpenState(name);
        return ResponseEntity.ok("熔断器已开启: " + name);
    }
    
    /**
     * 手动关闭熔断器
     */
    @PostMapping("/{name}/close")
    public ResponseEntity<String> closeCircuitBreaker(@PathVariable String name) {
        monitorService.transitionToClosedState(name);
        return ResponseEntity.ok("熔断器已关闭: " + name);
    }
}
```

### **最佳实践**

**1. 参数配置建议：**
- **失败率阈值**：50%-70%（根据业务容忍度）
- **最小调用次数**：10-20次（避免样本过小）
- **超时时间**：30-60秒（给下游服务恢复时间）
- **半开状态测试次数**：3-5次

**2. 降级策略：**
- 返回缓存数据
- 返回默认值
- 异步处理（消息队列）
- 调用备用服务

**3. 监控告警：**
- 熔断器状态变化告警
- 失败率超过阈值告警
- 降级调用次数统计
- 服务恢复时间监控

**总结：**

熔断器是分布式系统中重要的容错机制，通过三种状态的转换来保护系统免受故障传播。正确配置熔断器参数、实现合理的降级策略、建立完善的监控体系是确保系统高可用性的关键。

**Q14: 限流算法有哪些？**

**答案：**

限流（Rate Limiting）是分布式系统中保护服务免受过载的重要手段。通过控制请求的速率，可以确保系统在高并发场景下保持稳定性和可用性。常见的限流算法各有特点，适用于不同的业务场景。

### **限流算法对比**

| 算法 | 特点 | 优点 | 缺点 | 适用场景 |
|------|------|------|------|----------|
| **固定窗口** | 固定时间窗口内限制请求数 | 实现简单，内存占用少 | 边界突刺问题 | 粗粒度限流 |
| **滑动窗口** | 动态时间窗口，平滑限流 | 避免突刺，更精确 | 实现复杂，内存占用大 | 精确限流 |
| **令牌桶** | 以固定速率生成令牌 | 允许突发流量，灵活性好 | 实现相对复杂 | API网关、突发场景 |
| **漏桶** | 以固定速率处理请求 | 平滑输出，削峰填谷 | 不允许突发 | 流量整形 |

### **1. 固定窗口算法**

**核心思想：** 在固定的时间窗口内限制请求数量

```java
/**
 * 固定窗口限流器
 */
public class FixedWindowRateLimiter {
    
    private final int maxRequests;           // 窗口内最大请求数
    private final long windowSizeInMillis;   // 窗口大小（毫秒）
    private final AtomicInteger requestCount = new AtomicInteger(0);
    private volatile long windowStartTime;
    
    public FixedWindowRateLimiter(int maxRequests, long windowSizeInMillis) {
        this.maxRequests = maxRequests;
        this.windowSizeInMillis = windowSizeInMillis;
        this.windowStartTime = System.currentTimeMillis();
    }
    
    /**
     * 尝试获取许可
     */
    public synchronized boolean tryAcquire() {
        long currentTime = System.currentTimeMillis();
        
        // 检查是否需要重置窗口
        if (currentTime - windowStartTime >= windowSizeInMillis) {
            requestCount.set(0);
            windowStartTime = currentTime;
        }
        
        // 检查是否超过限制
        if (requestCount.get() < maxRequests) {
            requestCount.incrementAndGet();
            return true;
        }
        
        return false;
    }
    
    /**
     * 获取当前窗口统计信息
     */
    public WindowStats getStats() {
        long currentTime = System.currentTimeMillis();
        return WindowStats.builder()
            .currentRequests(requestCount.get())
            .maxRequests(maxRequests)
            .windowStartTime(windowStartTime)
            .remainingTime(windowSizeInMillis - (currentTime - windowStartTime))
            .build();
    }
}

/**
 * 分布式固定窗口限流器（基于Redis）
 */
@Component
public class DistributedFixedWindowRateLimiter {
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    /**
     * 分布式限流检查
     */
    public boolean tryAcquire(String key, int maxRequests, int windowSizeInSeconds) {
        String script = 
            "local current = redis.call('GET', KEYS[1]) " +
            "if current == false then " +
            "    redis.call('SET', KEYS[1], 1) " +
            "    redis.call('EXPIRE', KEYS[1], ARGV[2]) " +
            "    return 1 " +
            "else " +
            "    local count = tonumber(current) " +
            "    if count < tonumber(ARGV[1]) then " +
            "        redis.call('INCR', KEYS[1]) " +
            "        return 1 " +
            "    else " +
            "        return 0 " +
            "    end " +
            "end";
            
        Object result = redisTemplate.execute(
            new DefaultRedisScript<>(script, Long.class),
            Arrays.asList(key),
            String.valueOf(maxRequests),
            String.valueOf(windowSizeInSeconds)
        );
        
        return Long.valueOf(1).equals(result);
    }
}
```

### **2. 滑动窗口算法**

**核心思想：** 使用滑动时间窗口，更精确地控制请求速率

```java
/**
 * 滑动窗口限流器
 */
public class SlidingWindowRateLimiter {
    
    private final int maxRequests;
    private final long windowSizeInMillis;
    private final Queue<Long> requestTimestamps = new ConcurrentLinkedQueue<>();
    
    public SlidingWindowRateLimiter(int maxRequests, long windowSizeInMillis) {
        this.maxRequests = maxRequests;
        this.windowSizeInMillis = windowSizeInMillis;
    }
    
    /**
     * 尝试获取许可
     */
    public synchronized boolean tryAcquire() {
        long currentTime = System.currentTimeMillis();
        
        // 清理过期的请求记录
        cleanupExpiredRequests(currentTime);
        
        // 检查是否超过限制
        if (requestTimestamps.size() < maxRequests) {
            requestTimestamps.offer(currentTime);
            return true;
        }
        
        return false;
    }
    
    /**
     * 清理过期的请求记录
     */
    private void cleanupExpiredRequests(long currentTime) {
        while (!requestTimestamps.isEmpty()) {
            Long timestamp = requestTimestamps.peek();
            if (currentTime - timestamp > windowSizeInMillis) {
                requestTimestamps.poll();
            } else {
                break;
            }
        }
    }
    
    /**
     * 获取当前统计信息
     */
    public SlidingWindowStats getStats() {
        cleanupExpiredRequests(System.currentTimeMillis());
        return SlidingWindowStats.builder()
            .currentRequests(requestTimestamps.size())
            .maxRequests(maxRequests)
            .windowSizeInMillis(windowSizeInMillis)
            .build();
    }
}

/**
 * 基于Redis的滑动窗口限流器
 */
@Component
public class RedisSlidingWindowRateLimiter {
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    /**
     * 滑动窗口限流
     */
    public boolean tryAcquire(String key, int maxRequests, int windowSizeInSeconds) {
        String script = 
            "local key = KEYS[1] " +
            "local window = tonumber(ARGV[1]) " +
            "local limit = tonumber(ARGV[2]) " +
            "local current = tonumber(ARGV[3]) " +
            "" +
            "-- 清理过期数据 " +
            "redis.call('ZREMRANGEBYSCORE', key, 0, current - window * 1000) " +
            "" +
            "-- 获取当前窗口内的请求数 " +
            "local currentCount = redis.call('ZCARD', key) " +
            "" +
            "-- 检查是否超过限制 " +
            "if currentCount < limit then " +
            "    -- 添加当前请求 " +
            "    redis.call('ZADD', key, current, current) " +
            "    -- 设置过期时间 " +
            "    redis.call('EXPIRE', key, window) " +
            "    return 1 " +
            "else " +
            "    return 0 " +
            "end";
            
        Object result = redisTemplate.execute(
            new DefaultRedisScript<>(script, Long.class),
            Arrays.asList(key),
            String.valueOf(windowSizeInSeconds),
            String.valueOf(maxRequests),
            String.valueOf(System.currentTimeMillis())
        );
        
        return Long.valueOf(1).equals(result);
    }
}
```

### **3. 令牌桶算法**

**核心思想：** 以固定速率生成令牌，请求需要消耗令牌

```java
/**
 * 令牌桶限流器
 */
public class TokenBucketRateLimiter {
    
    private final int capacity;              // 桶容量
    private final double refillRate;         // 令牌生成速率（每秒）
    private final AtomicInteger tokens;      // 当前令牌数
    private volatile long lastRefillTime;    // 上次补充令牌时间
    
    public TokenBucketRateLimiter(int capacity, double refillRate) {
        this.capacity = capacity;
        this.refillRate = refillRate;
        this.tokens = new AtomicInteger(capacity);
        this.lastRefillTime = System.currentTimeMillis();
    }
    
    /**
     * 尝试获取指定数量的令牌
     */
    public synchronized boolean tryAcquire(int tokensRequested) {
        refillTokens();
        
        if (tokens.get() >= tokensRequested) {
            tokens.addAndGet(-tokensRequested);
            return true;
        }
        
        return false;
    }
    
    /**
     * 尝试获取单个令牌
     */
    public boolean tryAcquire() {
        return tryAcquire(1);
    }
    
    /**
     * 补充令牌
     */
    private void refillTokens() {
        long currentTime = System.currentTimeMillis();
        long timePassed = currentTime - lastRefillTime;
        
        if (timePassed > 0) {
            // 计算应该生成的令牌数
            int tokensToAdd = (int) (timePassed * refillRate / 1000);
            
            if (tokensToAdd > 0) {
                // 更新令牌数，不超过容量
                int currentTokens = tokens.get();
                int newTokens = Math.min(capacity, currentTokens + tokensToAdd);
                tokens.set(newTokens);
                lastRefillTime = currentTime;
            }
        }
    }
    
    /**
     * 获取当前状态
     */
    public TokenBucketStats getStats() {
        refillTokens();
        return TokenBucketStats.builder()
            .currentTokens(tokens.get())
            .capacity(capacity)
            .refillRate(refillRate)
            .build();
    }
}

/**
 * 基于Guava的令牌桶实现
 */
@Component
public class GuavaRateLimiterService {
    
    private final Map<String, RateLimiter> rateLimiters = new ConcurrentHashMap<>();
    
    /**
     * 获取或创建限流器
     */
    public RateLimiter getRateLimiter(String key, double permitsPerSecond) {
        return rateLimiters.computeIfAbsent(key, k -> 
            RateLimiter.create(permitsPerSecond));
    }
    
    /**
     * 尝试获取许可
     */
    public boolean tryAcquire(String key, double permitsPerSecond) {
        RateLimiter rateLimiter = getRateLimiter(key, permitsPerSecond);
        return rateLimiter.tryAcquire();
    }
    
    /**
     * 尝试获取指定数量的许可
     */
    public boolean tryAcquire(String key, double permitsPerSecond, int permits) {
        RateLimiter rateLimiter = getRateLimiter(key, permitsPerSecond);
        return rateLimiter.tryAcquire(permits);
    }
    
    /**
     * 阻塞获取许可
     */
    public void acquire(String key, double permitsPerSecond) {
        RateLimiter rateLimiter = getRateLimiter(key, permitsPerSecond);
        rateLimiter.acquire();
    }
}
```

### **4. 漏桶算法**

**核心思想：** 以固定速率处理请求，平滑输出

```java
/**
 * 漏桶限流器
 */
public class LeakyBucketRateLimiter {
    
    private final int capacity;              // 桶容量
    private final double leakRate;           // 漏出速率（每秒）
    private final Queue<Long> requests;      // 请求队列
    private volatile long lastLeakTime;      // 上次漏出时间
    
    public LeakyBucketRateLimiter(int capacity, double leakRate) {
        this.capacity = capacity;
        this.leakRate = leakRate;
        this.requests = new ConcurrentLinkedQueue<>();
        this.lastLeakTime = System.currentTimeMillis();
    }
    
    /**
     * 尝试添加请求到桶中
     */
    public synchronized boolean tryAcquire() {
        leak();
        
        if (requests.size() < capacity) {
            requests.offer(System.currentTimeMillis());
            return true;
        }
        
        return false;
    }
    
    /**
     * 漏出请求
     */
    private void leak() {
        long currentTime = System.currentTimeMillis();
        long timePassed = currentTime - lastLeakTime;
        
        if (timePassed > 0) {
            // 计算应该漏出的请求数
            int requestsToLeak = (int) (timePassed * leakRate / 1000);
            
            // 漏出请求
            for (int i = 0; i < requestsToLeak && !requests.isEmpty(); i++) {
                requests.poll();
            }
            
            lastLeakTime = currentTime;
        }
    }
    
    /**
     * 获取当前状态
     */
    public LeakyBucketStats getStats() {
        leak();
        return LeakyBucketStats.builder()
            .currentRequests(requests.size())
            .capacity(capacity)
            .leakRate(leakRate)
            .build();
    }
}
```

### **5. 限流器的实际应用**

```java
/**
 * 限流注解
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RateLimit {
    
    /**
     * 限流key，支持SpEL表达式
     */
    String key() default "";
    
    /**
     * 限流算法类型
     */
    RateLimitType type() default RateLimitType.TOKEN_BUCKET;
    
    /**
     * 限流速率
     */
    double rate();
    
    /**
     * 容量/窗口大小
     */
    int capacity() default 100;
    
    /**
     * 时间单位
     */
    TimeUnit timeUnit() default TimeUnit.SECONDS;
    
    /**
     * 限流失败时的处理方式
     */
    RateLimitFallback fallback() default RateLimitFallback.EXCEPTION;
}

/**
 * 限流切面
 */
@Aspect
@Component
public class RateLimitAspect {
    
    @Autowired
    private RateLimiterFactory rateLimiterFactory;
    
    @Around("@annotation(rateLimit)")
    public Object around(ProceedingJoinPoint joinPoint, RateLimit rateLimit) throws Throwable {
        // 构建限流key
        String key = buildRateLimitKey(joinPoint, rateLimit);
        
        // 获取限流器
        RateLimiter rateLimiter = rateLimiterFactory.getRateLimiter(
            key, rateLimit.type(), rateLimit.rate(), rateLimit.capacity());
        
        // 尝试获取许可
        if (rateLimiter.tryAcquire()) {
            return joinPoint.proceed();
        } else {
            // 限流处理
            return handleRateLimit(joinPoint, rateLimit);
        }
    }
    
    /**
     * 构建限流key
     */
    private String buildRateLimitKey(ProceedingJoinPoint joinPoint, RateLimit rateLimit) {
        String key = rateLimit.key();
        if (StringUtils.isEmpty(key)) {
            // 默认使用方法签名作为key
            key = joinPoint.getSignature().toShortString();
        } else {
            // 解析SpEL表达式
            key = parseSpEL(key, joinPoint);
        }
        return "rate_limit:" + key;
    }
    
    /**
     * 处理限流
     */
    private Object handleRateLimit(ProceedingJoinPoint joinPoint, RateLimit rateLimit) {
        switch (rateLimit.fallback()) {
            case EXCEPTION:
                throw new RateLimitException("请求过于频繁，请稍后重试");
            case NULL:
                return null;
            case EMPTY:
                return getEmptyResult(joinPoint.getSignature().getReturnType());
            default:
                throw new RateLimitException("请求过于频繁，请稍后重试");
        }
    }
}

/**
 * 限流器工厂
 */
@Component
public class RateLimiterFactory {
    
    private final Map<String, RateLimiter> rateLimiters = new ConcurrentHashMap<>();
    
    public RateLimiter getRateLimiter(String key, RateLimitType type, 
                                     double rate, int capacity) {
        return rateLimiters.computeIfAbsent(key, k -> createRateLimiter(type, rate, capacity));
    }
    
    private RateLimiter createRateLimiter(RateLimitType type, double rate, int capacity) {
        switch (type) {
            case FIXED_WINDOW:
                return new FixedWindowRateLimiter(capacity, (long)(1000 / rate));
            case SLIDING_WINDOW:
                return new SlidingWindowRateLimiter(capacity, (long)(1000 / rate));
            case TOKEN_BUCKET:
                return new TokenBucketRateLimiter(capacity, rate);
            case LEAKY_BUCKET:
                return new LeakyBucketRateLimiter(capacity, rate);
            default:
                throw new IllegalArgumentException("不支持的限流算法: " + type);
        }
    }
}

/**
 * 使用示例
 */
@RestController
public class ApiController {
    
    /**
     * 用户级别限流
     */
    @RateLimit(key = "#userId", type = RateLimitType.TOKEN_BUCKET, rate = 10, capacity = 20)
    @GetMapping("/api/user/{userId}/profile")
    public UserProfile getUserProfile(@PathVariable Long userId) {
        return userService.getUserProfile(userId);
    }
    
    /**
     * IP级别限流
     */
    @RateLimit(key = "#{request.remoteAddr}", type = RateLimitType.SLIDING_WINDOW, 
               rate = 100, capacity = 200)
    @PostMapping("/api/search")
    public SearchResult search(@RequestBody SearchRequest request, 
                              HttpServletRequest httpRequest) {
        return searchService.search(request);
    }
    
    /**
     * 全局限流
     */
    @RateLimit(key = "global", type = RateLimitType.LEAKY_BUCKET, rate = 1000)
    @GetMapping("/api/public/news")
    public List<News> getNews() {
        return newsService.getLatestNews();
    }
}
```

### **最佳实践和选择建议**

**1. 算法选择：**
- **固定窗口**：简单场景，对精度要求不高
- **滑动窗口**：需要精确限流，避免突刺
- **令牌桶**：允许突发流量，API网关场景
- **漏桶**：需要平滑输出，流量整形

**2. 实施建议：**
- 根据业务特点选择合适的算法
- 合理设置限流参数（速率、容量）
- 实现优雅的降级策略
- 建立监控和告警机制
- 考虑分布式场景下的一致性

**3. 监控指标：**
- 限流触发次数
- 请求通过率
- 响应时间分布
- 系统资源使用率

**总结：**

限流算法是保护系统稳定性的重要手段。选择合适的算法、正确配置参数、实现完善的监控是成功实施限流的关键。在实际应用中，往往需要结合多种算法来应对不同的业务场景。

**Q15: 如何设计一个高可用的分布式系统？**

**答案：**

高可用（High Availability, HA）分布式系统设计是确保系统在面对各种故障时仍能持续提供服务的关键。一个优秀的高可用系统需要从架构设计、技术选型、运维监控等多个维度进行综合考虑。

### **高可用设计原则**

| 原则 | 目标 | 实现方式 | 关键指标 |
|------|------|----------|----------|
| **冗余设计** | 消除单点故障 | 多实例部署、多机房部署 | 可用性99.9%+ |
| **故障隔离** | 限制故障影响范围 | 舱壁模式、熔断器 | 故障隔离率 |
| **快速恢复** | 缩短故障恢复时间 | 健康检查、自动重启 | MTTR < 5分钟 |
| **降级策略** | 保障核心功能 | 服务降级、功能开关 | 核心服务可用性 |
| **监控告警** | 及时发现问题 | 全链路监控、智能告警 | 故障发现时间 |
| **容灾备份** | 应对灾难性故障 | 异地多活、数据备份 | RTO/RPO指标 |

### **1. 架构设计层面**

#### **1.1 微服务架构**

```java
/**
 * 服务注册与发现
 */
@SpringBootApplication
@EnableEurekaServer
public class ServiceRegistryApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceRegistryApplication.class, args);
    }
}

/**
 * 高可用服务提供者
 */
@RestController
@RefreshScope
public class UserServiceController {
    
    @Autowired
    private UserService userService;
    
    @Autowired
    private CircuitBreakerFactory circuitBreakerFactory;
    
    /**
     * 带熔断保护的用户查询
     */
    @GetMapping("/users/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        CircuitBreaker circuitBreaker = circuitBreakerFactory.create("userService");
        
        return circuitBreaker.executeSupplier(() -> {
            User user = userService.findById(id);
            return ResponseEntity.ok(user);
        });
    }
    
    /**
     * 降级处理
     */
    @GetMapping("/users/{id}/fallback")
    public ResponseEntity<User> getUserFallback(@PathVariable Long id) {
        User defaultUser = User.builder()
            .id(id)
            .name("默认用户")
            .status("服务暂时不可用")
            .build();
        return ResponseEntity.ok(defaultUser);
    }
}

/**
 * 服务健康检查
 */
@Component
public class UserServiceHealthIndicator implements HealthIndicator {
    
    @Autowired
    private DataSource dataSource;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Override
    public Health health() {
        Health.Builder builder = new Health.Builder();
        
        try {
            // 检查数据库连接
            checkDatabase(builder);
            
            // 检查Redis连接
            checkRedis(builder);
            
            // 检查外部依赖
            checkExternalDependencies(builder);
            
            return builder.up().build();
        } catch (Exception e) {
            return builder.down(e).build();
        }
    }
    
    private void checkDatabase(Health.Builder builder) {
        try (Connection connection = dataSource.getConnection()) {
            if (connection.isValid(3)) {
                builder.withDetail("database", "UP");
            } else {
                builder.withDetail("database", "DOWN");
            }
        } catch (Exception e) {
            builder.withDetail("database", "DOWN: " + e.getMessage());
        }
    }
    
    private void checkRedis(Health.Builder builder) {
        try {
            redisTemplate.opsForValue().set("health:check", "ok", 10, TimeUnit.SECONDS);
            String result = (String) redisTemplate.opsForValue().get("health:check");
            if ("ok".equals(result)) {
                builder.withDetail("redis", "UP");
            } else {
                builder.withDetail("redis", "DOWN");
            }
        } catch (Exception e) {
            builder.withDetail("redis", "DOWN: " + e.getMessage());
        }
    }
    
    private void checkExternalDependencies(Health.Builder builder) {
        // 检查外部API、消息队列等
        Map<String, String> dependencies = new HashMap<>();
        dependencies.put("payment-service", checkPaymentService());
        dependencies.put("notification-service", checkNotificationService());
        builder.withDetail("external-dependencies", dependencies);
    }
}
```

#### **1.2 负载均衡与故障转移**

```java
/**
 * 自定义负载均衡策略
 */
@Component
public class HealthAwareLoadBalancer implements ReactorServiceInstanceLoadBalancer {
    
    private final String serviceId;
    private final ObjectProvider<ServiceInstanceListSupplier> serviceInstanceListSupplierProvider;
    
    public HealthAwareLoadBalancer(ObjectProvider<ServiceInstanceListSupplier> serviceInstanceListSupplierProvider,
                                  String serviceId) {
        this.serviceId = serviceId;
        this.serviceInstanceListSupplierProvider = serviceInstanceListSupplierProvider;
    }
    
    @Override
    public Mono<Response<ServiceInstance>> choose(Request request) {
        ServiceInstanceListSupplier supplier = serviceInstanceListSupplierProvider
            .getIfAvailable(NoopServiceInstanceListSupplier::new);
            
        return supplier.get(request)
            .next()
            .map(serviceInstances -> processInstanceResponse(serviceInstances));
    }
    
    private Response<ServiceInstance> processInstanceResponse(List<ServiceInstance> serviceInstances) {
        if (serviceInstances.isEmpty()) {
            return new EmptyResponse();
        }
        
        // 过滤健康的实例
        List<ServiceInstance> healthyInstances = serviceInstances.stream()
            .filter(this::isHealthy)
            .collect(Collectors.toList());
            
        if (healthyInstances.isEmpty()) {
            // 如果没有健康实例，返回原始列表中的第一个
            return new DefaultResponse(serviceInstances.get(0));
        }
        
        // 基于权重和响应时间选择实例
        ServiceInstance selectedInstance = selectBestInstance(healthyInstances);
        return new DefaultResponse(selectedInstance);
    }
    
    private boolean isHealthy(ServiceInstance instance) {
        // 检查实例健康状态
        String healthUrl = "http://" + instance.getHost() + ":" + instance.getPort() + "/actuator/health";
        try {
            RestTemplate restTemplate = new RestTemplate();
            ResponseEntity<Map> response = restTemplate.getForEntity(healthUrl, Map.class);
            return response.getStatusCode().is2xxSuccessful() && 
                   "UP".equals(((Map) response.getBody().get("status")));
        } catch (Exception e) {
            return false;
        }
    }
    
    private ServiceInstance selectBestInstance(List<ServiceInstance> instances) {
        // 基于响应时间和负载选择最佳实例
        return instances.stream()
            .min(Comparator.comparingLong(this::getResponseTime))
            .orElse(instances.get(0));
    }
    
    private long getResponseTime(ServiceInstance instance) {
        // 获取实例响应时间（可以从监控系统获取）
        return System.currentTimeMillis() % 100; // 示例实现
    }
}

/**
 * 故障转移配置
 */
@Configuration
public class FailoverConfiguration {
    
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        RestTemplate restTemplate = new RestTemplate();
        
        // 配置重试策略
        restTemplate.setRequestFactory(new HttpComponentsClientHttpRequestFactory() {{
            setConnectTimeout(5000);
            setReadTimeout(10000);
        }});
        
        return restTemplate;
    }
    
    @Bean
    public RetryTemplate retryTemplate() {
        RetryTemplate retryTemplate = new RetryTemplate();
        
        // 重试策略
        FixedBackOffPolicy backOffPolicy = new FixedBackOffPolicy();
        backOffPolicy.setBackOffPeriod(1000);
        retryTemplate.setBackOffPolicy(backOffPolicy);
        
        // 重试条件
        SimpleRetryPolicy retryPolicy = new SimpleRetryPolicy();
        retryPolicy.setMaxAttempts(3);
        retryTemplate.setRetryPolicy(retryPolicy);
        
        return retryTemplate;
    }
}
```

### **2. 数据层高可用**

#### **2.1 数据库高可用**

```java
/**
 * 读写分离配置
 */
@Configuration
public class DatabaseHighAvailabilityConfig {
    
    @Bean
    @Primary
    public DataSource dataSource() {
        HikariConfig masterConfig = new HikariConfig();
        masterConfig.setJdbcUrl("jdbc:mysql://master-db:3306/app");
        masterConfig.setUsername("app_user");
        masterConfig.setPassword("password");
        masterConfig.setMaximumPoolSize(20);
        masterConfig.setConnectionTimeout(30000);
        masterConfig.setIdleTimeout(600000);
        masterConfig.setMaxLifetime(1800000);
        
        HikariDataSource masterDataSource = new HikariDataSource(masterConfig);
        
        HikariConfig slaveConfig = new HikariConfig();
        slaveConfig.setJdbcUrl("jdbc:mysql://slave-db:3306/app");
        slaveConfig.setUsername("app_user");
        slaveConfig.setPassword("password");
        slaveConfig.setMaximumPoolSize(20);
        
        HikariDataSource slaveDataSource = new HikariDataSource(slaveConfig);
        
        // 创建读写分离数据源
        Map<Object, Object> dataSourceMap = new HashMap<>();
        dataSourceMap.put("master", masterDataSource);
        dataSourceMap.put("slave", slaveDataSource);
        
        DynamicDataSource dynamicDataSource = new DynamicDataSource();
        dynamicDataSource.setTargetDataSources(dataSourceMap);
        dynamicDataSource.setDefaultTargetDataSource(masterDataSource);
        
        return dynamicDataSource;
    }
    
    @Bean
    public DataSourceTransactionManager transactionManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}

/**
 * 动态数据源
 */
public class DynamicDataSource extends AbstractRoutingDataSource {
    
    @Override
    protected Object determineCurrentLookupKey() {
        return DataSourceContextHolder.getDataSourceType();
    }
}

/**
 * 数据源上下文
 */
public class DataSourceContextHolder {
    
    private static final ThreadLocal<String> CONTEXT_HOLDER = new ThreadLocal<>();
    
    public static void setDataSourceType(String dataSourceType) {
        CONTEXT_HOLDER.set(dataSourceType);
    }
    
    public static String getDataSourceType() {
        return CONTEXT_HOLDER.get();
    }
    
    public static void clearDataSourceType() {
        CONTEXT_HOLDER.remove();
    }
}

/**
 * 数据库故障检测和切换
 */
@Component
public class DatabaseFailoverManager {
    
    @Autowired
    private DataSource dataSource;
    
    private final AtomicBoolean masterAvailable = new AtomicBoolean(true);
    
    @Scheduled(fixedDelay = 30000) // 每30秒检查一次
    public void checkDatabaseHealth() {
        boolean masterHealthy = checkMasterDatabase();
        
        if (!masterHealthy && masterAvailable.get()) {
            // 主库故障，切换到从库
            masterAvailable.set(false);
            notifyDatabaseFailover("Master database is down, switching to slave");
        } else if (masterHealthy && !masterAvailable.get()) {
            // 主库恢复，切换回主库
            masterAvailable.set(true);
            notifyDatabaseFailover("Master database is back online");
        }
    }
    
    private boolean checkMasterDatabase() {
        try {
            DataSourceContextHolder.setDataSourceType("master");
            try (Connection connection = dataSource.getConnection()) {
                return connection.isValid(5);
            }
        } catch (Exception e) {
            return false;
        } finally {
            DataSourceContextHolder.clearDataSourceType();
        }
    }
    
    private void notifyDatabaseFailover(String message) {
        // 发送告警通知
        log.warn("Database failover: {}", message);
        // 可以集成钉钉、邮件等通知方式
    }
    
    public boolean isMasterAvailable() {
        return masterAvailable.get();
    }
}
```

#### **2.2 缓存高可用**

```java
/**
 * Redis高可用配置
 */
@Configuration
public class RedisHighAvailabilityConfig {
    
    @Bean
    public LettuceConnectionFactory redisConnectionFactory() {
        // Redis Sentinel配置
        RedisSentinelConfiguration sentinelConfig = new RedisSentinelConfiguration()
            .master("mymaster")
            .sentinel("sentinel1", 26379)
            .sentinel("sentinel2", 26379)
            .sentinel("sentinel3", 26379);
            
        // 连接池配置
        GenericObjectPoolConfig<Object> poolConfig = new GenericObjectPoolConfig<>();
        poolConfig.setMaxTotal(20);
        poolConfig.setMaxIdle(10);
        poolConfig.setMinIdle(5);
        poolConfig.setTestOnBorrow(true);
        poolConfig.setTestOnReturn(true);
        poolConfig.setTestWhileIdle(true);
        
        LettucePoolingClientConfiguration clientConfig = LettucePoolingClientConfiguration.builder()
            .poolConfig(poolConfig)
            .commandTimeout(Duration.ofSeconds(5))
            .build();
            
        return new LettuceConnectionFactory(sentinelConfig, clientConfig);
    }
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(LettuceConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        
        // 序列化配置
        Jackson2JsonRedisSerializer<Object> serializer = new Jackson2JsonRedisSerializer<>(Object.class);
        template.setDefaultSerializer(serializer);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(serializer);
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(serializer);
        
        template.afterPropertiesSet();
        return template;
    }
}

/**
 * 多级缓存实现
 */
@Service
public class MultiLevelCacheService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    private final Cache<String, Object> localCache = Caffeine.newBuilder()
        .maximumSize(1000)
        .expireAfterWrite(5, TimeUnit.MINUTES)
        .build();
    
    /**
     * 获取缓存数据
     */
    public <T> T get(String key, Class<T> type) {
        // 1. 先查本地缓存
        Object value = localCache.getIfPresent(key);
        if (value != null) {
            return type.cast(value);
        }
        
        // 2. 查Redis缓存
        try {
            value = redisTemplate.opsForValue().get(key);
            if (value != null) {
                // 更新本地缓存
                localCache.put(key, value);
                return type.cast(value);
            }
        } catch (Exception e) {
            log.warn("Redis cache access failed: {}", e.getMessage());
        }
        
        return null;
    }
    
    /**
     * 设置缓存数据
     */
    public void set(String key, Object value, Duration expiration) {
        // 1. 更新本地缓存
        localCache.put(key, value);
        
        // 2. 更新Redis缓存
        try {
            redisTemplate.opsForValue().set(key, value, expiration);
        } catch (Exception e) {
            log.warn("Redis cache update failed: {}", e.getMessage());
        }
    }
    
    /**
     * 删除缓存
     */
    public void delete(String key) {
        // 1. 删除本地缓存
        localCache.invalidate(key);
        
        // 2. 删除Redis缓存
        try {
            redisTemplate.delete(key);
        } catch (Exception e) {
            log.warn("Redis cache deletion failed: {}", e.getMessage());
        }
    }
}
```

### **3. 监控和告警**

```java
/**
 * 系统监控服务
 */
@Service
public class SystemMonitoringService {
    
    @Autowired
    private MeterRegistry meterRegistry;
    
    @Autowired
    private AlertService alertService;
    
    private final Timer.Sample requestTimer = Timer.start(meterRegistry);
    
    /**
     * 记录请求指标
     */
    public void recordRequest(String endpoint, String method, int statusCode, long duration) {
        Counter.builder("http_requests_total")
            .tag("endpoint", endpoint)
            .tag("method", method)
            .tag("status", String.valueOf(statusCode))
            .register(meterRegistry)
            .increment();
            
        Timer.builder("http_request_duration")
            .tag("endpoint", endpoint)
            .register(meterRegistry)
            .record(duration, TimeUnit.MILLISECONDS);
    }
    
    /**
     * 记录业务指标
     */
    public void recordBusinessMetric(String metricName, double value, String... tags) {
        Gauge.builder(metricName)
            .tags(tags)
            .register(meterRegistry, value);
    }
    
    /**
     * 检查系统健康状态
     */
    @Scheduled(fixedDelay = 60000) // 每分钟检查一次
    public void checkSystemHealth() {
        // 检查CPU使用率
        double cpuUsage = getCpuUsage();
        if (cpuUsage > 80) {
            alertService.sendAlert("High CPU Usage", 
                String.format("CPU usage is %.2f%%", cpuUsage));
        }
        
        // 检查内存使用率
        double memoryUsage = getMemoryUsage();
        if (memoryUsage > 85) {
            alertService.sendAlert("High Memory Usage", 
                String.format("Memory usage is %.2f%%", memoryUsage));
        }
        
        // 检查错误率
        double errorRate = getErrorRate();
        if (errorRate > 5) {
            alertService.sendAlert("High Error Rate", 
                String.format("Error rate is %.2f%%", errorRate));
        }
    }
    
    private double getCpuUsage() {
        OperatingSystemMXBean osBean = ManagementFactory.getOperatingSystemMXBean();
        if (osBean instanceof com.sun.management.OperatingSystemMXBean) {
            return ((com.sun.management.OperatingSystemMXBean) osBean).getProcessCpuLoad() * 100;
        }
        return 0;
    }
    
    private double getMemoryUsage() {
        MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
        MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
        return (double) heapUsage.getUsed() / heapUsage.getMax() * 100;
    }
    
    private double getErrorRate() {
        // 从监控指标中计算错误率
        Counter totalRequests = meterRegistry.find("http_requests_total").counter();
        Counter errorRequests = meterRegistry.find("http_requests_total")
            .tag("status", "5xx")
            .counter();
            
        if (totalRequests != null && errorRequests != null) {
            double total = totalRequests.count();
            double errors = errorRequests.count();
            return total > 0 ? (errors / total) * 100 : 0;
        }
        
        return 0;
    }
}

/**
 * 告警服务
 */
@Service
public class AlertService {
    
    @Value("${alert.webhook.url}")
    private String webhookUrl;
    
    @Autowired
    private RestTemplate restTemplate;
    
    /**
     * 发送告警
     */
    public void sendAlert(String title, String message) {
        AlertMessage alert = AlertMessage.builder()
            .title(title)
            .message(message)
            .timestamp(Instant.now())
            .severity("HIGH")
            .build();
            
        try {
            restTemplate.postForEntity(webhookUrl, alert, String.class);
        } catch (Exception e) {
            log.error("Failed to send alert: {}", e.getMessage());
        }
    }
}
```

### **4. 容灾和备份**

```java
/**
 * 异地多活配置
 */
@Configuration
public class MultiRegionConfiguration {
    
    @Value("${app.region}")
    private String currentRegion;
    
    @Bean
    public RegionAwareLoadBalancer regionAwareLoadBalancer() {
        return new RegionAwareLoadBalancer(currentRegion);
    }
}

/**
 * 数据备份服务
 */
@Service
public class DataBackupService {
    
    @Autowired
    private DataSource dataSource;
    
    @Value("${backup.storage.path}")
    private String backupPath;
    
    /**
     * 定时备份
     */
    @Scheduled(cron = "0 0 2 * * ?") // 每天凌晨2点执行
    public void performBackup() {
        String backupFileName = String.format("backup_%s.sql", 
            LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMdd_HHmmss")));
        String backupFilePath = Paths.get(backupPath, backupFileName).toString();
        
        try {
            // 执行数据库备份
            ProcessBuilder pb = new ProcessBuilder(
                "mysqldump",
                "-h", "localhost",
                "-u", "backup_user",
                "-p", "backup_password",
                "--single-transaction",
                "--routines",
                "--triggers",
                "app_database"
            );
            
            Process process = pb.start();
            
            // 将输出写入文件
            try (FileOutputStream fos = new FileOutputStream(backupFilePath);
                 InputStream is = process.getInputStream()) {
                is.transferTo(fos);
            }
            
            int exitCode = process.waitFor();
            if (exitCode == 0) {
                log.info("Database backup completed: {}", backupFilePath);
                
                // 上传到云存储
                uploadToCloudStorage(backupFilePath);
                
                // 清理旧备份
                cleanupOldBackups();
            } else {
                log.error("Database backup failed with exit code: {}", exitCode);
            }
        } catch (Exception e) {
            log.error("Database backup failed: {}", e.getMessage(), e);
        }
    }
    
    private void uploadToCloudStorage(String filePath) {
        // 上传到云存储（如AWS S3、阿里云OSS等）
        // 实现略
    }
    
    private void cleanupOldBackups() {
        // 清理超过30天的备份文件
        try {
            Files.walk(Paths.get(backupPath))
                .filter(Files::isRegularFile)
                .filter(path -> path.toString().endsWith(".sql"))
                .filter(path -> {
                    try {
                        FileTime fileTime = Files.getLastModifiedTime(path);
                        return fileTime.toInstant().isBefore(
                            Instant.now().minus(30, ChronoUnit.DAYS));
                    } catch (IOException e) {
                        return false;
                    }
                })
                .forEach(path -> {
                    try {
                        Files.delete(path);
                        log.info("Deleted old backup: {}", path);
                    } catch (IOException e) {
                        log.warn("Failed to delete old backup: {}", path);
                    }
                });
        } catch (IOException e) {
            log.error("Failed to cleanup old backups: {}", e.getMessage());
        }
    }
}
```

### **最佳实践总结**

**1. 设计原则：**
- 无单点故障（SPOF）
- 故障隔离和快速恢复
- 优雅降级和熔断保护
- 数据一致性和完整性

**2. 技术选型：**
- 微服务架构 + 容器化部署
- 负载均衡 + 自动扩缩容
- 多级缓存 + 数据库集群
- 消息队列 + 异步处理

**3. 运维保障：**
- 全链路监控和告警
- 自动化部署和回滚
- 定期备份和容灾演练
- 性能调优和容量规划

**4. 关键指标：**
- 可用性：99.9%+（年停机时间 < 8.76小时）
- 恢复时间目标（RTO）：< 5分钟
- 恢复点目标（RPO）：< 1小时
- 错误率：< 0.1%

**总结：**

高可用分布式系统设计是一个系统工程，需要从架构、技术、运维等多个维度进行综合考虑。关键是要建立完善的监控体系、实现自动化的故障检测和恢复机制，并通过持续的优化和演练来提升系统的可靠性。

## 学习建议

### 实践项目

1. **电商微服务系统**
   - 用户服务、商品服务、订单服务、支付服务
   - 使用Spring Cloud全家桶
   - 实现服务注册发现、配置中心、网关路由

2. **分布式缓存系统**
   - 实现多级缓存
   - 缓存一致性保证
   - 缓存预热和更新策略

3. **高并发秒杀系统**
   - 限流、熔断、降级
   - 分布式锁
   - 异步处理

### 推荐资源

**官方文档**
- [Spring Cloud官方文档](https://spring.io/projects/spring-cloud)
- [Redis官方文档](https://redis.io/documentation)
- [MySQL官方文档](https://dev.mysql.com/doc/)

**推荐书籍**
- 《微服务架构设计模式》
- 《分布式系统概念与设计》
- 《高性能MySQL》
- 《Redis设计与实现》

**在线课程**
- 极客时间《微服务架构核心20讲》
- 慕课网《Spring Cloud微服务实战》
- B站《尚硅谷分布式系统教程》

### 学习时间规划

- **第1-2周**：微服务基础和Spring Cloud
- **第3-4周**：分布式缓存和Redis
- **第5-6周**：数据库优化和分库分表
- **第7-8周**：分布式事务
- **第9-10周**：服务治理和监控
- **第11-12周**：综合项目实战

## 总结

第四阶段主要学习分布式系统的核心技术，包括：

1. **微服务架构**：掌握服务拆分、注册发现、网关路由
2. **分布式缓存**：掌握Redis使用、缓存策略、一致性保证
3. **数据库优化**：掌握SQL优化、索引设计、分库分表
4. **分布式事务**：掌握TCC、Saga等事务模式
5. **服务治理**：掌握熔断、限流、监控等治理手段

这些技术是构建大型分布式系统的基础，需要通过大量实践来掌握。建议结合实际项目，逐步深入理解分布式系统的设计原理和最佳实践。

/**
 * Redis缓存控制器
 * 提供缓存操作的REST API
 */
@RestController
@RequestMapping("/cache")
class RedisCacheController {
    
    @Autowired
    private RedisCacheService cacheService;
    
    /**
     * 演示字符串操作
     */
    @PostMapping("/string")
    public Map<String, Object> stringDemo() {
        Map<String, Object> result = new HashMap<>();
        
        // 设置字符串
        cacheService.setString("demo:string", "Hello Redis", 60, TimeUnit.SECONDS);
        
        // 获取字符串
        String value = cacheService.getString("demo:string");
        
        // 计数器操作
        cacheService.increment("demo:counter", 1);
        cacheService.increment("demo:counter", 5);
        String counter = cacheService.getString("demo:counter");
        
        result.put("string_value", value);
        result.put("counter_value", counter);
        result.put("message", "字符串操作演示完成");
        
        return result;
    }
    
    /**
     * 演示用户缓存
     */
    @PostMapping("/user/{id}")
    public CacheUser userCacheDemo(@PathVariable Long id) {
        // 创建测试用户
        CacheUser user = new CacheUser(id, "用户" + id, "user" + id + "@example.com", 25 + id.intValue());
        user.getRoles().add("USER");
        user.getRoles().add("CUSTOMER");
        user.getProfile().put("city", "北京");
        user.getProfile().put("hobby", "编程");
        
        // 缓存用户
        cacheService.cacheUser(user, 300, TimeUnit.SECONDS);
        
        // 获取缓存用户
        return cacheService.getCachedUser(id);
    }
    
    /**
     * 演示列表操作
     */
    @PostMapping("/list")
    public Map<String, Object> listDemo() {
        Map<String, Object> result = new HashMap<>();
        String key = "demo:list";
        
        // 清空列表
        cacheService.deleteCache(key);
        
        // 添加元素
        cacheService.leftPushList(key, "任务1", "任务2", "任务3");
        
        // 获取列表
        List<Object> list = cacheService.getListRange(key, 0, -1);
        
        // 弹出元素
        Object popped = cacheService.rightPopList(key);
        
        result.put("list", list);
        result.put("popped", popped);
        result.put("message", "列表操作演示完成");
        
        return result;
    }
    
    /**
     * 演示集合操作
     */
    @PostMapping("/set")
    public Map<String, Object> setDemo() {
        Map<String, Object> result = new HashMap<>();
        
        // 创建两个集合
        cacheService.addToSet("demo:set1", "Java", "Python", "JavaScript", "Go");
        cacheService.addToSet("demo:set2", "Java", "C++", "JavaScript", "Rust");
        
        // 获取集合成员
        Set<Object> set1 = cacheService.getSetMembers("demo:set1");
        Set<Object> set2 = cacheService.getSetMembers("demo:set2");
        
        // 计算交集
        Set<Object> intersection = cacheService.intersectSets("demo:set1", "demo:set2");
        
        result.put("set1", set1);
        result.put("set2", set2);
        result.put("intersection", intersection);
        result.put("message", "集合操作演示完成");
        
        return result;
    }
    
    /**
     * 演示有序集合操作
     */
    @PostMapping("/zset")
    public Map<String, Object> zsetDemo() {
        Map<String, Object> result = new HashMap<>();
        String key = "demo:leaderboard";
        
        // 清空有序集合
        cacheService.deleteCache(key);
        
        // 添加排行榜数据
        cacheService.addToZSet(key, "张三", 95.5);
        cacheService.addToZSet(key, "李四", 87.2);
        cacheService.addToZSet(key, "王五", 92.8);
        cacheService.addToZSet(key, "赵六", 89.1);
        
        // 获取排行榜（分数90以上）
        Set<Object> topScores = cacheService.getZSetRangeByScore(key, 90.0, 100.0);
        
        // 获取张三的排名
        Long zhangSanRank = cacheService.getZSetRank(key, "张三");
        
        result.put("top_scores", topScores);
        result.put("zhangsan_rank", zhangSanRank);
        result.put("message", "有序集合操作演示完成");
        
        return result;
    }
}

#### 2.2 缓存策略和一致性

```java
import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.data.redis.core.script.RedisScript;

import java.util.*;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 缓存策略服务
 * 演示各种缓存模式和一致性保证
 */
@Service
public class CacheStrategyService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Autowired
    private UserRepository userRepository; // 假设的数据库仓库
    
    private final Lock lock = new ReentrantLock();
    
    // ==================== Cache-Aside模式 ====================
    
    /**
     * Cache-Aside模式：读取数据
     * 先查缓存，缓存不存在则查数据库并更新缓存
     */
    public CacheUser getUserCacheAside(Long userId) {
        String cacheKey = "user:" + userId;
        
        // 1. 先查缓存
        CacheUser cachedUser = (CacheUser) redisTemplate.opsForValue().get(cacheKey);
        if (cachedUser != null) {
            System.out.println("缓存命中: " + cacheKey);
            return cachedUser;
        }
        
        System.out.println("缓存未命中，查询数据库: " + cacheKey);
        
        // 2. 查数据库
        CacheUser dbUser = queryUserFromDatabase(userId);
        if (dbUser != null) {
            // 3. 更新缓存
            redisTemplate.opsForValue().set(cacheKey, dbUser, 300, TimeUnit.SECONDS);
            System.out.println("更新缓存: " + cacheKey);
        }
        
        return dbUser;
    }
    
    /**
     * Cache-Aside模式：更新数据
     * 先更新数据库，再删除缓存
     */
    @Transactional
    public CacheUser updateUserCacheAside(CacheUser user) {
        String cacheKey = "user:" + user.getId();
        
        try {
            // 1. 更新数据库
            CacheUser updatedUser = updateUserInDatabase(user);
            System.out.println("数据库更新成功: " + user.getId());
            
            // 2. 删除缓存（延迟双删策略）
            redisTemplate.delete(cacheKey);
            System.out.println("删除缓存: " + cacheKey);
            
            // 延迟删除，防止脏数据
            new Thread(() -> {
                try {
                    Thread.sleep(1000); // 延迟1秒
                    redisTemplate.delete(cacheKey);
                    System.out.println("延迟删除缓存: " + cacheKey);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }).start();
            
            return updatedUser;
        } catch (Exception e) {
            System.err.println("更新用户失败: " + e.getMessage());
            throw e;
        }
    }
    
    // ==================== Write-Through模式 ====================
    
    /**
     * Write-Through模式：写入数据
     * 同时更新缓存和数据库
     */
    @Transactional
    public CacheUser saveUserWriteThrough(CacheUser user) {
        String cacheKey = "user:" + user.getId();
        
        try {
            // 1. 更新数据库
            CacheUser savedUser = saveUserToDatabase(user);
            System.out.println("数据库保存成功: " + user.getId());
            
            // 2. 同时更新缓存
            redisTemplate.opsForValue().set(cacheKey, savedUser, 300, TimeUnit.SECONDS);
            System.out.println("缓存更新成功: " + cacheKey);
            
            return savedUser;
        } catch (Exception e) {
            System.err.println("Write-Through保存失败: " + e.getMessage());
            // 如果数据库失败，确保缓存也不更新
            redisTemplate.delete(cacheKey);
            throw e;
        }
    }
    
    // ==================== Write-Behind模式 ====================
    
    /**
     * Write-Behind模式：异步写入
     * 先更新缓存，异步更新数据库
     */
    public CacheUser saveUserWriteBehind(CacheUser user) {
        String cacheKey = "user:" + user.getId();
        
        // 1. 立即更新缓存
        redisTemplate.opsForValue().set(cacheKey, user, 300, TimeUnit.SECONDS);
        System.out.println("缓存立即更新: " + cacheKey);
        
        // 2. 异步更新数据库
        new Thread(() -> {
            try {
                Thread.sleep(100); // 模拟异步延迟
                saveUserToDatabase(user);
                System.out.println("数据库异步更新成功: " + user.getId());
            } catch (Exception e) {
                System.err.println("数据库异步更新失败: " + e.getMessage());
                // 可以实现重试机制或者回滚缓存
            }
        }).start();
        
        return user;
    }
    
    // ==================== 分布式锁防止缓存击穿 ====================
    
    /**
     * 使用分布式锁防止缓存击穿
     * 当缓存失效时，只允许一个线程查询数据库
     */
    public CacheUser getUserWithDistributedLock(Long userId) {
        String cacheKey = "user:" + userId;
        String lockKey = "lock:user:" + userId;
        
        // 1. 先查缓存
        CacheUser cachedUser = (CacheUser) redisTemplate.opsForValue().get(cacheKey);
        if (cachedUser != null) {
            return cachedUser;
        }
        
        // 2. 获取分布式锁
        String lockValue = UUID.randomUUID().toString();
        Boolean lockAcquired = redisTemplate.opsForValue().setIfAbsent(lockKey, lockValue, 10, TimeUnit.SECONDS);
        
        if (lockAcquired != null && lockAcquired) {
            try {
                System.out.println("获取分布式锁成功: " + lockKey);
                
                // 3. 双重检查缓存
                cachedUser = (CacheUser) redisTemplate.opsForValue().get(cacheKey);
                if (cachedUser != null) {
                    return cachedUser;
                }
                
                // 4. 查询数据库
                CacheUser dbUser = queryUserFromDatabase(userId);
                if (dbUser != null) {
                    // 5. 更新缓存
                    redisTemplate.opsForValue().set(cacheKey, dbUser, 300, TimeUnit.SECONDS);
                }
                
                return dbUser;
            } finally {
                // 6. 释放锁（使用Lua脚本保证原子性）
                releaseLock(lockKey, lockValue);
            }
        } else {
            // 获取锁失败，等待一段时间后重试
            try {
                Thread.sleep(50);
                return getUserWithDistributedLock(userId); // 递归重试
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return null;
            }
        }
    }
    
    /**
     * 使用Lua脚本释放分布式锁
     * 保证只有锁的持有者才能释放锁
     */
    private void releaseLock(String lockKey, String lockValue) {
        String luaScript = 
            "if redis.call('get', KEYS[1]) == ARGV[1] then " +
            "    return redis.call('del', KEYS[1]) " +
            "else " +
            "    return 0 " +
            "end";
        
        RedisScript<Long> script = new DefaultRedisScript<>(luaScript, Long.class);
        Long result = redisTemplate.execute(script, Collections.singletonList(lockKey), lockValue);
        
        if (result != null && result == 1) {
            System.out.println("释放分布式锁成功: " + lockKey);
        } else {
            System.out.println("释放分布式锁失败: " + lockKey);
        }
    }
    
    // ==================== 缓存预热 ====================
    
    /**
     * 缓存预热
     * 系统启动时预加载热点数据
     */
    public void warmUpCache() {
        System.out.println("开始缓存预热...");
        
        // 预加载热点用户数据
        List<Long> hotUserIds = Arrays.asList(1L, 2L, 3L, 4L, 5L);
        
        for (Long userId : hotUserIds) {
            try {
                CacheUser user = queryUserFromDatabase(userId);
                if (user != null) {
                    String cacheKey = "user:" + userId;
                    redisTemplate.opsForValue().set(cacheKey, user, 600, TimeUnit.SECONDS);
                    System.out.println("预热缓存: " + cacheKey);
                }
                
                // 避免对数据库造成过大压力
                Thread.sleep(10);
            } catch (Exception e) {
                System.err.println("预热缓存失败: " + userId + ", " + e.getMessage());
            }
        }
        
        System.out.println("缓存预热完成");
    }
    
    // ==================== 缓存穿透防护 ====================
    
    /**
     * 布隆过滤器防止缓存穿透
     * 使用Redis的bitmap实现简单布隆过滤器
     */
    public CacheUser getUserWithBloomFilter(Long userId) {
        String bloomKey = "bloom:users";
        
        // 1. 检查布隆过滤器
        if (!checkBloomFilter(bloomKey, userId.toString())) {
            System.out.println("布隆过滤器判断用户不存在: " + userId);
            return null;
        }
        
        // 2. 查缓存
        String cacheKey = "user:" + userId;
        CacheUser cachedUser = (CacheUser) redisTemplate.opsForValue().get(cacheKey);
        if (cachedUser != null) {
            return cachedUser;
        }
        
        // 3. 查数据库
        CacheUser dbUser = queryUserFromDatabase(userId);
        if (dbUser != null) {
            // 4. 更新缓存
            redisTemplate.opsForValue().set(cacheKey, dbUser, 300, TimeUnit.SECONDS);
        } else {
            // 5. 缓存空值，防止缓存穿透
            redisTemplate.opsForValue().set(cacheKey, "NULL", 60, TimeUnit.SECONDS);
            System.out.println("缓存空值防止穿透: " + cacheKey);
        }
        
        return dbUser;
    }
    
    /**
     * 简单布隆过滤器检查
     */
    private boolean checkBloomFilter(String bloomKey, String value) {
        // 使用多个哈希函数
        int[] hashes = {
            Math.abs(value.hashCode() % 10000),
            Math.abs((value + "salt1").hashCode() % 10000),
            Math.abs((value + "salt2").hashCode() % 10000)
        };
        
        for (int hash : hashes) {
            Boolean bit = redisTemplate.opsForValue().getBit(bloomKey, hash);
            if (bit == null || !bit) {
                return false;
            }
        }
        
        return true;
    }
    
    /**
     * 添加到布隆过滤器
     */
    public void addToBloomFilter(String bloomKey, String value) {
        int[] hashes = {
            Math.abs(value.hashCode() % 10000),
            Math.abs((value + "salt1").hashCode() % 10000),
            Math.abs((value + "salt2").hashCode() % 10000)
        };
        
        for (int hash : hashes) {
            redisTemplate.opsForValue().setBit(bloomKey, hash, true);
        }
        
        System.out.println("添加到布隆过滤器: " + value);
    }
    
    // ==================== 模拟数据库操作 ====================
    
    private CacheUser queryUserFromDatabase(Long userId) {
        // 模拟数据库查询延迟
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        // 模拟数据库数据
        if (userId <= 10) {
            CacheUser user = new CacheUser(userId, "数据库用户" + userId, 
                                         "dbuser" + userId + "@example.com", 20 + userId.intValue());
            user.getRoles().add("USER");
            return user;
        }
        
        return null; // 模拟用户不存在
    }
    
    private CacheUser updateUserInDatabase(CacheUser user) {
        // 模拟数据库更新
        try {
            Thread.sleep(50);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        user.setLastLoginTime(LocalDateTime.now());
        return user;
    }
    
    private CacheUser saveUserToDatabase(CacheUser user) {
        // 模拟数据库保存
        try {
            Thread.sleep(50);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        if (user.getId() == null) {
            user.setId(System.currentTimeMillis() % 10000);
        }
        
        return user;
    }
}

// 假设的数据库仓库接口
interface UserRepository {
    CacheUser findById(Long id);
    CacheUser save(CacheUser user);
    void deleteById(Long id);
}

### 3. 数据库优化

#### 3.1 MySQL性能调优

```java
import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;

import javax.sql.DataSource;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;
import java.util.Map;
import java.util.HashMap;
import java.time.LocalDateTime;

/**
 * 数据库性能优化服务
 * 演示SQL优化、索引使用、批量操作等
 */
@Service
public class DatabaseOptimizationService {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    // ==================== 索引优化 ====================
    
    /**
     * 创建索引的SQL示例
     * 在实际应用中，这些应该在数据库迁移脚本中执行
     */
    public void createIndexes() {
        System.out.println("创建数据库索引...");
        
        try {
            // 1. 单列索引
            jdbcTemplate.execute("CREATE INDEX idx_user_email ON users(email)");
            jdbcTemplate.execute("CREATE INDEX idx_user_status ON users(status)");
            
            // 2. 复合索引
            jdbcTemplate.execute("CREATE INDEX idx_user_status_create_time ON users(status, create_time)");
            
            // 3. 唯一索引
            jdbcTemplate.execute("CREATE UNIQUE INDEX idx_user_username ON users(username)");
            
            // 4. 前缀索引（适用于长字符串字段）
            jdbcTemplate.execute("CREATE INDEX idx_user_description ON users(description(50))");
            
            System.out.println("索引创建完成");
        } catch (Exception e) {
            System.err.println("索引创建失败: " + e.getMessage());
        }
    }
    
    /**
     * 优化的用户查询
     * 使用索引提高查询性能
     */
    public List<Map<String, Object>> findActiveUsersByEmail(String emailDomain) {
        // 优化前的SQL（全表扫描）
        // SELECT * FROM users WHERE email LIKE '%@example.com' AND status = 'ACTIVE'
        
        // 优化后的SQL（使用索引）
        String sql = """
            SELECT u.id, u.username, u.email, u.create_time
            FROM users u
            WHERE u.status = 'ACTIVE'
              AND u.email LIKE ?
            ORDER BY u.create_time DESC
            LIMIT 100
            """;
        
        System.out.println("执行优化查询: " + emailDomain);
        return jdbcTemplate.queryForList(sql, "%" + emailDomain);
    }
    
    /**
     * 分页查询优化
     * 使用覆盖索引和延迟关联
     */
    public List<Map<String, Object>> findUsersWithPagination(int page, int size, String status) {
        int offset = page * size;
        
        // 优化前的SQL（深分页性能差）
        // SELECT * FROM users WHERE status = ? ORDER BY id LIMIT ?, ?
        
        // 优化后的SQL（使用覆盖索引 + 延迟关联）
        String sql = """
            SELECT u.id, u.username, u.email, u.status, u.create_time
            FROM users u
            INNER JOIN (
                SELECT id
                FROM users
                WHERE status = ?
                ORDER BY id
                LIMIT ?, ?
            ) t ON u.id = t.id
            ORDER BY u.id
            """;
        
        System.out.println(String.format("分页查询: page=%d, size=%d, status=%s", page, size, status));
        return jdbcTemplate.queryForList(sql, status, offset, size);
    }
    
    // ==================== 批量操作优化 ====================
    
    /**
     * 批量插入优化
     * 使用批处理提高插入性能
     */
    @Transactional
    public void batchInsertUsers(List<CacheUser> users) {
        System.out.println("开始批量插入用户: " + users.size());
        
        String sql = """
            INSERT INTO users (username, email, age, status, create_time)
            VALUES (?, ?, ?, ?, ?)
            """;
        
        // 使用批处理
        jdbcTemplate.batchUpdate(sql, users, 1000, (ps, user) -> {
            ps.setString(1, user.getUsername());
            ps.setString(2, user.getEmail());
            ps.setInt(3, user.getAge());
            ps.setString(4, "ACTIVE");
            ps.setObject(5, LocalDateTime.now());
        });
        
        System.out.println("批量插入完成");
    }
    
    /**
     * 批量更新优化
     * 使用CASE WHEN语句进行批量更新
     */
    @Transactional
    public void batchUpdateUserStatus(Map<Long, String> userStatusMap) {
        if (userStatusMap.isEmpty()) {
            return;
        }
        
        System.out.println("开始批量更新用户状态: " + userStatusMap.size());
        
        // 构建CASE WHEN语句
        StringBuilder sql = new StringBuilder("""
            UPDATE users SET status = CASE id
            """);
        
        List<Object> params = new ArrayList<>();
        for (Map.Entry<Long, String> entry : userStatusMap.entrySet()) {
            sql.append(" WHEN ? THEN ?");
            params.add(entry.getKey());
            params.add(entry.getValue());
        }
        
        sql.append(" END WHERE id IN (");
        for (int i = 0; i < userStatusMap.size(); i++) {
            if (i > 0) sql.append(", ");
            sql.append("?");
        }
        sql.append(")");
        
        params.addAll(userStatusMap.keySet());
        
        int updated = jdbcTemplate.update(sql.toString(), params.toArray());
        System.out.println("批量更新完成，影响行数: " + updated);
    }
    
    // ==================== 查询优化 ====================
    
    /**
     * 使用EXISTS替代IN优化子查询
     */
    public List<Map<String, Object>> findUsersWithOrders() {
        // 优化前的SQL（性能较差）
        // SELECT * FROM users WHERE id IN (SELECT DISTINCT user_id FROM orders)
        
        // 优化后的SQL（使用EXISTS）
        String sql = """
            SELECT u.id, u.username, u.email
            FROM users u
            WHERE EXISTS (
                SELECT 1 FROM orders o WHERE o.user_id = u.id
            )
            AND u.status = 'ACTIVE'
            """;
        
        System.out.println("查询有订单的用户");
        return jdbcTemplate.queryForList(sql);
    }
    
    /**
     * 使用JOIN替代子查询
     */
    public List<Map<String, Object>> findUserOrderSummary() {
        // 优化前的SQL（多次子查询）
        /*
        SELECT u.id, u.username,
               (SELECT COUNT(*) FROM orders WHERE user_id = u.id) as order_count,
               (SELECT SUM(amount) FROM orders WHERE user_id = u.id) as total_amount
        FROM users u
        */
        
        // 优化后的SQL（使用JOIN）
        String sql = """
            SELECT u.id, u.username, u.email,
                   COALESCE(o.order_count, 0) as order_count,
                   COALESCE(o.total_amount, 0) as total_amount
            FROM users u
            LEFT JOIN (
                SELECT user_id,
                       COUNT(*) as order_count,
                       SUM(amount) as total_amount
                FROM orders
                GROUP BY user_id
            ) o ON u.id = o.user_id
            WHERE u.status = 'ACTIVE'
            ORDER BY o.total_amount DESC
            """;
        
        System.out.println("查询用户订单汇总");
        return jdbcTemplate.queryForList(sql);
    }
    
    // ==================== 慢查询分析 ====================
    
    /**
     * 分析慢查询
     * 获取执行时间较长的SQL语句
     */
    public List<Map<String, Object>> analyzeSlowQueries() {
        String sql = """
            SELECT sql_text,
                   exec_count,
                   avg_timer_wait/1000000000 as avg_time_seconds,
                   sum_timer_wait/1000000000 as total_time_seconds
            FROM performance_schema.events_statements_summary_by_digest
            WHERE avg_timer_wait > 1000000000  -- 大于1秒的查询
            ORDER BY avg_timer_wait DESC
            LIMIT 10
            """;
        
        try {
            System.out.println("分析慢查询...");
            return jdbcTemplate.queryForList(sql);
        } catch (Exception e) {
            System.err.println("慢查询分析失败: " + e.getMessage());
            return new ArrayList<>();
        }
    }
    
    /**
     * 查看表的索引使用情况
     */
    public List<Map<String, Object>> analyzeIndexUsage(String tableName) {
        String sql = """
            SELECT DISTINCT
                   s.table_name,
                   s.index_name,
                   s.column_name,
                   s.seq_in_index,
                   s.cardinality,
                   t.rows as table_rows
            FROM information_schema.statistics s
            JOIN information_schema.tables t ON s.table_name = t.table_name
            WHERE s.table_schema = DATABASE()
              AND s.table_name = ?
            ORDER BY s.table_name, s.index_name, s.seq_in_index
            """;
        
        System.out.println("分析表索引使用情况: " + tableName);
        return jdbcTemplate.queryForList(sql, tableName);
    }
    
    // ==================== 连接池优化 ====================
    
    /**
     * 监控数据库连接池状态
     */
    public Map<String, Object> getConnectionPoolStatus() {
        Map<String, Object> status = new HashMap<>();
        
        try {
            // 获取连接池信息（这里以HikariCP为例）
            DataSource dataSource = jdbcTemplate.getDataSource();
            if (dataSource instanceof com.zaxxer.hikari.HikariDataSource) {
                com.zaxxer.hikari.HikariDataSource hikariDS = (com.zaxxer.hikari.HikariDataSource) dataSource;
                com.zaxxer.hikari.HikariPoolMXBean poolBean = hikariDS.getHikariPoolMXBean();
                
                status.put("active_connections", poolBean.getActiveConnections());
                status.put("idle_connections", poolBean.getIdleConnections());
                status.put("total_connections", poolBean.getTotalConnections());
                status.put("threads_awaiting_connection", poolBean.getThreadsAwaitingConnection());
            }
            
            // 获取数据库状态
            List<Map<String, Object>> processlist = jdbcTemplate.queryForList("SHOW PROCESSLIST");
            status.put("active_queries", processlist.size());
            
            Map<String, Object> globalStatus = jdbcTemplate.queryForMap(
                "SHOW GLOBAL STATUS WHERE Variable_name IN ('Threads_connected', 'Threads_running', 'Max_used_connections')"
            );
            status.put("database_status", globalStatus);
            
        } catch (Exception e) {
            System.err.println("获取连接池状态失败: " + e.getMessage());
            status.put("error", e.getMessage());
        }
        
        return status;
    }
    
    /**
     * 数据库健康检查
     */
    public boolean isDatabaseHealthy() {
        try {
            jdbcTemplate.queryForObject("SELECT 1", Integer.class);
            return true;
        } catch (Exception e) {
            System.err.println("数据库健康检查失败: " + e.getMessage());
            return false;
        }
    }
}

/**
 * 用户服务 - 服务提供者
 * 注册到Eureka并提供用户相关API
 */
@SpringBootApplication
@EnableEurekaClient
public class UserServiceApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
        System.out.println("用户服务启动成功！");
    }
}

/**
 * 用户实体类
 */
class User {
    private Long id;
    private String username;
    private String email;
    private String phone;
    private LocalDateTime createTime;
    private String status;
    
    // 构造函数
    public User() {}
    
    public User(Long id, String username, String email, String phone) {
        this.id = id;
        this.username = username;
        this.email = email;
        this.phone = phone;
        this.createTime = LocalDateTime.now();
        this.status = "ACTIVE";
    }
    
    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    public String getPhone() { return phone; }
    public void setPhone(String phone) { this.phone = phone; }
    public LocalDateTime getCreateTime() { return createTime; }
    public void setCreateTime(LocalDateTime createTime) { this.createTime = createTime; }
    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }
    
    @Override
    public String toString() {
        return String.format("User{id=%d, username='%s', email='%s', status='%s'}", 
                           id, username, email, status);
    }
}

/**
 * 用户控制器
 * 提供用户相关的REST API
 */
@RestController
@RequestMapping("/users")
class UserController {
    
    @Autowired
    private UserService userService;
    
    /**
     * 获取所有用户
     */
    @GetMapping
    public List<User> getAllUsers() {
        System.out.println("获取所有用户请求");
        return userService.getAllUsers();
    }
    
    /**
     * 根据ID获取用户
     */
    @GetMapping("/{id}")
    public User getUserById(@PathVariable Long id) {
        System.out.println("获取用户详情，ID: " + id);
        return userService.getUserById(id);
    }
    
    /**
     * 创建用户
     */
    @PostMapping
    public User createUser(@RequestBody User user) {
        System.out.println("创建用户: " + user.getUsername());
        return userService.createUser(user);
    }
    
    /**
     * 更新用户
     */
    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody User user) {
        System.out.println("更新用户，ID: " + id);
        user.setId(id);
        return userService.updateUser(user);
    }
    
    /**
     * 删除用户
     */
    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable Long id) {
        System.out.println("删除用户，ID: " + id);
        userService.deleteUser(id);
    }
    
    /**
     * 根据用户名搜索用户
     */
    @GetMapping("/search")
    public List<User> searchUsers(@RequestParam String keyword) {
        System.out.println("搜索用户，关键词: " + keyword);
        return userService.searchUsers(keyword);
    }
}

/**
 * 用户服务类
 * 处理用户相关的业务逻辑
 */
@Service
class UserService {
    
    // 模拟数据库存储
    private List<User> users = new ArrayList<>();
    private Long nextId = 1L;
    
    public UserService() {
        // 初始化测试数据
        users.add(new User(nextId++, "张三", "zhangsan@example.com", "13800138001"));
        users.add(new User(nextId++, "李四", "lisi@example.com", "13800138002"));
        users.add(new User(nextId++, "王五", "wangwu@example.com", "13800138003"));
    }
    
    /**
     * 获取所有用户
     */
    public List<User> getAllUsers() {
        return new ArrayList<>(users);
    }
    
    /**
     * 根据ID获取用户
     */
    public User getUserById(Long id) {
        return users.stream()
                .filter(user -> user.getId().equals(id))
                .findFirst()
                .orElseThrow(() -> new RuntimeException("用户不存在: " + id));
    }
    
    /**
     * 创建用户
     */
    public User createUser(User user) {
        user.setId(nextId++);
        user.setCreateTime(LocalDateTime.now());
        user.setStatus("ACTIVE");
        users.add(user);
        return user;
    }
    
    /**
     * 更新用户
     */
    public User updateUser(User user) {
        User existingUser = getUserById(user.getId());
        existingUser.setUsername(user.getUsername());
        existingUser.setEmail(user.getEmail());
        existingUser.setPhone(user.getPhone());
        return existingUser;
    }
    
    /**
     * 删除用户
     */
    public void deleteUser(Long id) {
        users.removeIf(user -> user.getId().equals(id));
    }
    
    /**
     * 搜索用户
     */
    public List<User> searchUsers(String keyword) {
        return users.stream()
                .filter(user -> user.getUsername().contains(keyword) || 
                              user.getEmail().contains(keyword))
                .collect(ArrayList::new, ArrayList::add, ArrayList::addAll);
    }
}

/**
 * 订单服务 - 服务消费者
 * 通过Feign客户端调用用户服务
 */
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
public class OrderServiceApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
        System.out.println("订单服务启动成功！");
    }
}

/**
 * 用户服务Feign客户端
 * 声明式HTTP客户端，用于调用用户服务
 */
@FeignClient(name = "user-service", fallback = UserServiceFallback.class)
interface UserServiceClient {
    
    @GetMapping("/users/{id}")
    User getUserById(@PathVariable("id") Long id);
    
    @GetMapping("/users")
    List<User> getAllUsers();
    
    @PostMapping("/users")
    User createUser(@RequestBody User user);
}

/**
 * 用户服务降级处理
 * 当用户服务不可用时的备用方案
 */
@Component
class UserServiceFallback implements UserServiceClient {
    
    @Override
    public User getUserById(Long id) {
        System.out.println("用户服务不可用，返回默认用户信息");
        return new User(id, "默认用户", "default@example.com", "00000000000");
    }
    
    @Override
    public List<User> getAllUsers() {
        System.out.println("用户服务不可用，返回空列表");
        return new ArrayList<>();
    }
    
    @Override
    public User createUser(User user) {
        System.out.println("用户服务不可用，无法创建用户");
        throw new RuntimeException("用户服务暂时不可用");
    }
}

/**
 * 订单实体类
 */
class Order {
    private Long id;
    private Long userId;
    private String orderNumber;
    private Double amount;
    private String status;
    private LocalDateTime createTime;
    private User user; // 关联的用户信息
    
    // 构造函数
    public Order() {}
    
    public Order(Long userId, String orderNumber, Double amount) {
        this.userId = userId;
        this.orderNumber = orderNumber;
        this.amount = amount;
        this.status = "PENDING";
        this.createTime = LocalDateTime.now();
    }
    
    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public Long getUserId() { return userId; }
    public void setUserId(Long userId) { this.userId = userId; }
    public String getOrderNumber() { return orderNumber; }
    public void setOrderNumber(String orderNumber) { this.orderNumber = orderNumber; }
    public Double getAmount() { return amount; }
    public void setAmount(Double amount) { this.amount = amount; }
    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }
    public LocalDateTime getCreateTime() { return createTime; }
    public void setCreateTime(LocalDateTime createTime) { this.createTime = createTime; }
    public User getUser() { return user; }
    public void setUser(User user) { this.user = user; }
    
    @Override
    public String toString() {
        return String.format("Order{id=%d, orderNumber='%s', amount=%.2f, status='%s'}", 
                           id, orderNumber, amount, status);
    }
}

/**
 * 订单控制器
 * 提供订单相关的REST API
 */
@RestController
@RequestMapping("/orders")
class OrderController {
    
    @Autowired
    private OrderService orderService;
    
    /**
     * 获取所有订单
     */
    @GetMapping
    public List<Order> getAllOrders() {
        System.out.println("获取所有订单请求");
        return orderService.getAllOrders();
    }
    
    /**
     * 根据ID获取订单
     */
    @GetMapping("/{id}")
    public Order getOrderById(@PathVariable Long id) {
        System.out.println("获取订单详情，ID: " + id);
        return orderService.getOrderById(id);
    }
    
    /**
     * 创建订单
     */
    @PostMapping
    public Order createOrder(@RequestBody Order order) {
        System.out.println("创建订单: " + order.getOrderNumber());
        return orderService.createOrder(order);
    }
    
    /**
     * 根据用户ID获取订单
     */
    @GetMapping("/user/{userId}")
    public List<Order> getOrdersByUserId(@PathVariable Long userId) {
        System.out.println("获取用户订单，用户ID: " + userId);
        return orderService.getOrdersByUserId(userId);
    }
}

/**
 * 订单服务类
 * 处理订单相关的业务逻辑，调用用户服务获取用户信息
 */
@Service
class OrderService {
    
    @Autowired
    private UserServiceClient userServiceClient;
    
    // 模拟数据库存储
    private List<Order> orders = new ArrayList<>();
    private Long nextId = 1L;
    
    public OrderService() {
        // 初始化测试数据
        orders.add(createTestOrder(1L, "ORD001", 299.99));
        orders.add(createTestOrder(2L, "ORD002", 599.99));
        orders.add(createTestOrder(1L, "ORD003", 199.99));
    }
    
    private Order createTestOrder(Long userId, String orderNumber, Double amount) {
        Order order = new Order(userId, orderNumber, amount);
        order.setId(nextId++);
        return order;
    }
    
    /**
     * 获取所有订单
     */
    public List<Order> getAllOrders() {
        // 为每个订单填充用户信息
        return orders.stream()
                .map(this::fillUserInfo)
                .collect(ArrayList::new, ArrayList::add, ArrayList::addAll);
    }
    
    /**
     * 根据ID获取订单
     */
    public Order getOrderById(Long id) {
        Order order = orders.stream()
                .filter(o -> o.getId().equals(id))
                .findFirst()
                .orElseThrow(() -> new RuntimeException("订单不存在: " + id));
        
        return fillUserInfo(order);
    }
    
    /**
     * 创建订单
     */
    public Order createOrder(Order order) {
        // 验证用户是否存在
        try {
            User user = userServiceClient.getUserById(order.getUserId());
            System.out.println("验证用户成功: " + user.getUsername());
        } catch (Exception e) {
            throw new RuntimeException("用户不存在或用户服务不可用: " + order.getUserId());
        }
        
        order.setId(nextId++);
        order.setCreateTime(LocalDateTime.now());
        order.setStatus("PENDING");
        
        // 生成订单号
        if (order.getOrderNumber() == null) {
            order.setOrderNumber("ORD" + String.format("%06d", order.getId()));
        }
        
        orders.add(order);
        return fillUserInfo(order);
    }
    
    /**
     * 根据用户ID获取订单
     */
    public List<Order> getOrdersByUserId(Long userId) {
        return orders.stream()
                .filter(order -> order.getUserId().equals(userId))
                .map(this::fillUserInfo)
                .collect(ArrayList::new, ArrayList::add, ArrayList::addAll);
    }
    
    /**
     * 为订单填充用户信息
     */
    private Order fillUserInfo(Order order) {
        try {
            User user = userServiceClient.getUserById(order.getUserId());
            order.setUser(user);
        } catch (Exception e) {
            System.err.println("获取用户信息失败: " + e.getMessage());
            // 设置默认用户信息
            User defaultUser = new User();
            defaultUser.setId(order.getUserId());
            defaultUser.setUsername("未知用户");
            order.setUser(defaultUser);
        }
        return order;
    }
}
```

**application.yml配置文件**

```yaml
# Eureka Server配置
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

---
# User Service配置
server:
  port: 8081

spring:
  application:
    name: user-service

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true

---
# Order Service配置
server:
  port: 8082

spring:
  application:
    name: order-service

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true

# Feign配置
feign:
  hystrix:
    enabled: true
  client:
    config:
      default:
        connect-timeout: 5000
        read-timeout: 5000
```

#### 1.2 Spring Cloud Gateway网关

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.http.ResponseEntity;
import org.springframework.http.HttpStatus;

import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.Map;

/**
 * Spring Cloud Gateway网关应用
 * 提供统一的API入口，路由转发、负载均衡、限流等功能
 */
@SpringBootApplication
@EnableEurekaClient
public class GatewayApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
        System.out.println("API网关启动成功！");
        System.out.println("网关地址: http://localhost:8080");
    }
    
    /**
     * 配置路由规则
     * 定义请求如何转发到后端服务
     */
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                // 用户服务路由
                .route("user-service", r -> r
                        .path("/api/users/**")
                        .filters(f -> f
                                .stripPrefix(1) // 去掉/api前缀
                                .addRequestHeader("X-Gateway", "Spring-Cloud-Gateway")
                                .addResponseHeader("X-Response-Time", String.valueOf(System.currentTimeMillis()))
                        )
                        .uri("lb://user-service") // 负载均衡到user-service
                )
                // 订单服务路由
                .route("order-service", r -> r
                        .path("/api/orders/**")
                        .filters(f -> f
                                .stripPrefix(1)
                                .addRequestHeader("X-Gateway", "Spring-Cloud-Gateway")
                                .addResponseHeader("X-Response-Time", String.valueOf(System.currentTimeMillis()))
                        )
                        .uri("lb://order-service")
                )
                // 健康检查路由
                .route("health-check", r -> r
                        .path("/health/**")
                        .uri("http://localhost:8080")
                )
                .build();
    }
}

/**
 * 网关健康检查控制器
 * 提供网关状态和服务健康检查接口
 */
@RestController
class GatewayHealthController {
    
    /**
     * 网关健康检查
     */
    @RequestMapping("/health")
    public ResponseEntity<Map<String, Object>> health() {
        Map<String, Object> health = new HashMap<>();
        health.put("status", "UP");
        health.put("timestamp", LocalDateTime.now());
        health.put("gateway", "Spring Cloud Gateway");
        health.put("version", "1.0.0");
        
        return ResponseEntity.ok(health);
    }
    
    /**
     * 网关信息
     */
    @RequestMapping("/health/info")
    public ResponseEntity<Map<String, Object>> info() {
        Map<String, Object> info = new HashMap<>();
        info.put("name", "API Gateway");
        info.put("description", "Spring Cloud Gateway for Microservices");
        info.put("version", "1.0.0");
        info.put("uptime", System.currentTimeMillis());
        
        Map<String, String> routes = new HashMap<>();
        routes.put("/api/users/**", "user-service");
        routes.put("/api/orders/**", "order-service");
        info.put("routes", routes);
        
        return ResponseEntity.ok(info);
    }
    
    /**
     * 服务状态检查
     */
    @RequestMapping("/health/services")
    public ResponseEntity<Map<String, Object>> services() {
        Map<String, Object> services = new HashMap<>();
        
        // 模拟检查各个服务的状态
        Map<String, String> serviceStatus = new HashMap<>();
        serviceStatus.put("user-service", "UP");
        serviceStatus.put("order-service", "UP");
        serviceStatus.put("eureka-server", "UP");
        
        services.put("services", serviceStatus);
        services.put("timestamp", LocalDateTime.now());
        services.put("total", serviceStatus.size());
        services.put("healthy", serviceStatus.values().stream().mapToInt(s -> "UP".equals(s) ? 1 : 0).sum());
        
        return ResponseEntity.ok(services);
    }
}
```

**Gateway配置文件**

```yaml
# Gateway配置
server:
  port: 8080

spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # 启用服务发现
          lower-case-service-id: true # 服务名小写
      routes:
        - id: user-service-route
          uri: lb://user-service
          predicates:
            - Path=/api/users/**
          filters:
            - StripPrefix=1
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10 # 令牌桶填充速率
                redis-rate-limiter.burstCapacity: 20 # 令牌桶容量
                key-resolver: "#{@ipKeyResolver}" # 限流key解析器
        - id: order-service-route
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
          filters:
            - StripPrefix=1
            - name: CircuitBreaker
              args:
                name: order-service-cb
                fallbackUri: forward:/fallback/orders

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true

# 日志配置
logging:
  level:
    org.springframework.cloud.gateway: DEBUG
    reactor.netty.http.client: DEBUG
```

### 2. 分布式缓存 - Redis

#### 2.1 Redis基础操作和Spring Boot集成

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.*;

import com.fasterxml.jackson.annotation.JsonFormat;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;

import java.time.LocalDateTime;
import java.util.*;
import java.util.concurrent.TimeUnit;

/**
 * Redis缓存应用
 * 演示Redis的各种数据类型和缓存策略
 */
@SpringBootApplication
public class RedisCacheApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(RedisCacheApplication.class, args);
        System.out.println("Redis缓存服务启动成功！");
    }
    
    /**
     * 配置RedisTemplate
     * 设置序列化方式，支持对象存储
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        
        // 设置key序列化方式
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        
        // 设置value序列化方式
        GenericJackson2JsonRedisSerializer jsonSerializer = new GenericJackson2JsonRedisSerializer();
        template.setValueSerializer(jsonSerializer);
        template.setHashValueSerializer(jsonSerializer);
        
        template.afterPropertiesSet();
        return template;
    }
}

/**
 * 缓存用户实体类
 */
class CacheUser {
    private Long id;
    private String username;
    private String email;
    private Integer age;
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime lastLoginTime;
    private List<String> roles;
    private Map<String, Object> profile;
    
    // 构造函数
    public CacheUser() {}
    
    public CacheUser(Long id, String username, String email, Integer age) {
        this.id = id;
        this.username = username;
        this.email = email;
        this.age = age;
        this.lastLoginTime = LocalDateTime.now();
        this.roles = new ArrayList<>();
        this.profile = new HashMap<>();
    }
    
    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    public Integer getAge() { return age; }
    public void setAge(Integer age) { this.age = age; }
    public LocalDateTime getLastLoginTime() { return lastLoginTime; }
    public void setLastLoginTime(LocalDateTime lastLoginTime) { this.lastLoginTime = lastLoginTime; }
    public List<String> getRoles() { return roles; }
    public void setRoles(List<String> roles) { this.roles = roles; }
    public Map<String, Object> getProfile() { return profile; }
    public void setProfile(Map<String, Object> profile) { this.profile = profile; }
    
    @Override
    public String toString() {
        return String.format("CacheUser{id=%d, username='%s', email='%s', age=%d}", 
                           id, username, email, age);
    }
}

/**
 * Redis缓存服务
 * 演示Redis的五种数据类型操作
 */
@Service
public class RedisCacheService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    
    // ==================== String类型操作 ====================
    
    /**
     * 设置字符串值
     */
    public void setString(String key, String value, long timeout, TimeUnit unit) {
        stringRedisTemplate.opsForValue().set(key, value, timeout, unit);
        System.out.println("设置字符串: " + key + " = " + value);
    }
    
    /**
     * 获取字符串值
     */
    public String getString(String key) {
        String value = stringRedisTemplate.opsForValue().get(key);
        System.out.println("获取字符串: " + key + " = " + value);
        return value;
    }
    
    /**
     * 原子递增
     */
    public Long increment(String key, long delta) {
        Long result = stringRedisTemplate.opsForValue().increment(key, delta);
        System.out.println("递增操作: " + key + " += " + delta + " = " + result);
        return result;
    }
    
    // ==================== Hash类型操作 ====================
    
    /**
     * 设置Hash字段
     */
    public void setHashField(String key, String field, Object value) {
        redisTemplate.opsForHash().put(key, field, value);
        System.out.println("设置Hash字段: " + key + "." + field + " = " + value);
    }
    
    /**
     * 获取Hash字段
     */
    public Object getHashField(String key, String field) {
        Object value = redisTemplate.opsForHash().get(key, field);
        System.out.println("获取Hash字段: " + key + "." + field + " = " + value);
        return value;
    }
    
    /**
     * 获取Hash所有字段
     */
    public Map<Object, Object> getHashAll(String key) {
        Map<Object, Object> hash = redisTemplate.opsForHash().entries(key);
        System.out.println("获取Hash所有字段: " + key + " = " + hash);
        return hash;
    }
    
    // ==================== List类型操作 ====================
    
    /**
     * 左侧推入列表
     */
    public Long leftPushList(String key, Object... values) {
        Long size = redisTemplate.opsForList().leftPushAll(key, values);
        System.out.println("左侧推入列表: " + key + " <- " + Arrays.toString(values) + ", 大小: " + size);
        return size;
    }
    
    /**
     * 右侧弹出列表
     */
    public Object rightPopList(String key) {
        Object value = redisTemplate.opsForList().rightPop(key);
        System.out.println("右侧弹出列表: " + key + " -> " + value);
        return value;
    }
    
    /**
     * 获取列表范围
     */
    public List<Object> getListRange(String key, long start, long end) {
        List<Object> list = redisTemplate.opsForList().range(key, start, end);
        System.out.println("获取列表范围: " + key + "[" + start + "," + end + "] = " + list);
        return list;
    }
    
    // ==================== Set类型操作 ====================
    
    /**
     * 添加到集合
     */
    public Long addToSet(String key, Object... values) {
        Long count = redisTemplate.opsForSet().add(key, values);
        System.out.println("添加到集合: " + key + " += " + Arrays.toString(values) + ", 新增: " + count);
        return count;
    }
    
    /**
     * 获取集合所有成员
     */
    public Set<Object> getSetMembers(String key) {
        Set<Object> members = redisTemplate.opsForSet().members(key);
        System.out.println("获取集合成员: " + key + " = " + members);
        return members;
    }
    
    /**
     * 集合交集
     */
    public Set<Object> intersectSets(String key1, String key2) {
        Set<Object> intersection = redisTemplate.opsForSet().intersect(key1, key2);
        System.out.println("集合交集: " + key1 + " ∩ " + key2 + " = " + intersection);
        return intersection;
    }
    
    // ==================== ZSet类型操作 ====================
    
    /**
     * 添加到有序集合
     */
    public Boolean addToZSet(String key, Object value, double score) {
        Boolean result = redisTemplate.opsForZSet().add(key, value, score);
        System.out.println("添加到有序集合: " + key + " += (" + value + ", " + score + "), 结果: " + result);
        return result;
    }
    
    /**
     * 获取有序集合范围（按分数）
     */
    public Set<Object> getZSetRangeByScore(String key, double min, double max) {
        Set<Object> range = redisTemplate.opsForZSet().rangeByScore(key, min, max);
        System.out.println("获取有序集合范围: " + key + "[" + min + "," + max + "] = " + range);
        return range;
    }
    
    /**
     * 获取有序集合排名
     */
    public Long getZSetRank(String key, Object value) {
        Long rank = redisTemplate.opsForZSet().rank(key, value);
        System.out.println("获取有序集合排名: " + key + "." + value + " = " + rank);
        return rank;
    }
    
    // ==================== 对象缓存操作 ====================
    
    /**
     * 缓存用户对象
     */
    public void cacheUser(CacheUser user, long timeout, TimeUnit unit) {
        String key = "user:" + user.getId();
        redisTemplate.opsForValue().set(key, user, timeout, unit);
        System.out.println("缓存用户对象: " + key + " = " + user);
    }
    
    /**
     * 获取缓存用户
     */
    public CacheUser getCachedUser(Long userId) {
        String key = "user:" + userId;
        CacheUser user = (CacheUser) redisTemplate.opsForValue().get(key);
        System.out.println("获取缓存用户: " + key + " = " + user);
        return user;
    }
    
    /**
     * 删除缓存
     */
    public Boolean deleteCache(String key) {
        Boolean result = redisTemplate.delete(key);
        System.out.println("删除缓存: " + key + ", 结果: " + result);
        return result;
    }
    
    /**
     * 检查key是否存在
     */
    public Boolean hasKey(String key) {
        Boolean exists = redisTemplate.hasKey(key);
        System.out.println("检查key存在: " + key + " = " + exists);
        return exists;
    }
    
    /**
     * 设置key过期时间
     */
    public Boolean expire(String key, long timeout, TimeUnit unit) {
        Boolean result = redisTemplate.expire(key, timeout, unit);
        System.out.println("设置过期时间: " + key + " = " + timeout + " " + unit + ", 结果: " + result);
        return result;
    }
    
    /**
     * 获取key剩余过期时间
     */
    public Long getExpire(String key, TimeUnit unit) {
        Long expire = redisTemplate.getExpire(key, unit);
        System.out.println("获取剩余过期时间: " + key + " = " + expire + " " + unit);
        return expire;
    }
}
```