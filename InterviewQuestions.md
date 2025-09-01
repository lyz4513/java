# Java架构师面试题大全

## 目录

1. [Java基础面试题](#1-java基础面试题)
2. [Java进阶面试题](#2-java进阶面试题)
3. [框架技术面试题](#3-框架技术面试题)
4. [分布式系统面试题](#4-分布式系统面试题)
5. [架构设计面试题](#5-架构设计面试题)
6. [综合实战面试题](#6-综合实战面试题)
7. [面试技巧和准备建议](#7-面试技巧和准备建议)

---

## 1. Java基础面试题

### 1.1 语法基础类

**Q1: Java中的基本数据类型有哪些？它们的默认值和取值范围是什么？**

**详细答案：**

Java中有8种基本数据类型，分为4类：

| 类型 | 数据类型 | 大小(字节) | 默认值 | 取值范围 | 包装类 |
|------|----------|------------|--------|----------|--------|
| 整型 | byte | 1 | 0 | -128 ~ 127 | Byte |
| 整型 | short | 2 | 0 | -32,768 ~ 32,767 | Short |
| 整型 | int | 4 | 0 | -2,147,483,648 ~ 2,147,483,647 | Integer |
| 整型 | long | 8 | 0L | -9,223,372,036,854,775,808 ~ 9,223,372,036,854,775,807 | Long |
| 浮点型 | float | 4 | 0.0f | ±3.4E38 (6-7位有效数字) | Float |
| 浮点型 | double | 8 | 0.0d | ±1.8E308 (15位有效数字) | Double |
| 字符型 | char | 2 | '\u0000' | 0 ~ 65,535 (Unicode字符) | Character |
| 布尔型 | boolean | 1 | false | true/false | Boolean |

**代码示例：**
```java
public class PrimitiveTypesDemo {
    public static void main(String[] args) {
        // 整型演示
        byte byteVar = 127;
        short shortVar = 32767;
        int intVar = 2147483647;
        long longVar = 9223372036854775807L; // 注意L后缀
        
        // 浮点型演示
        float floatVar = 3.14f; // 注意f后缀
        double doubleVar = 3.141592653589793;
        
        // 字符型演示
        char charVar1 = 'A';
        char charVar2 = 65; // ASCII码
        char charVar3 = '\u0041'; // Unicode
        
        // 布尔型演示
        boolean boolVar = true;
        
        // 类型转换演示
        demonstrateTypeCasting();
        
        // 包装类演示
        demonstrateWrapperClasses();
    }
    
    private static void demonstrateTypeCasting() {
        // 自动类型提升（隐式转换）
        byte b = 10;
        int i = b; // byte -> int 自动提升
        
        // 强制类型转换（显式转换）
        int largeInt = 300;
        byte smallByte = (byte) largeInt; // 数据丢失：44
        
        // 浮点数转换
        double d = 3.14;
        int truncated = (int) d; // 截断小数部分：3
        
        System.out.println("强制转换结果: " + smallByte);
        System.out.println("截断结果: " + truncated);
    }
    
    private static void demonstrateWrapperClasses() {
        // 自动装箱和拆箱
        Integer intObj = 100; // 自动装箱
        int primitiveInt = intObj; // 自动拆箱
        
        // 包装类的缓存机制
        Integer a = 127;
        Integer b = 127;
        Integer c = 128;
        Integer d = 128;
        
        System.out.println("a == b: " + (a == b)); // true (缓存)
        System.out.println("c == d: " + (c == d)); // false (超出缓存范围)
        
        // 包装类的实用方法
        String numberStr = "123";
        int parsed = Integer.parseInt(numberStr);
        String binary = Integer.toBinaryString(15); // "1111"
        
        System.out.println("解析结果: " + parsed);
        System.out.println("二进制表示: " + binary);
    }
}
```

**重要知识点：**

1. **内存占用**：基本类型直接存储值，包装类存储对象引用
2. **性能差异**：基本类型性能更好，包装类提供更多功能
3. **缓存机制**：Integer缓存-128到127的对象
4. **类型转换**：小范围到大范围自动转换，反之需要强制转换
5. **精度问题**：浮点数计算可能存在精度丢失

**实际应用场景：**
- **byte**：网络传输、文件IO、图像处理
- **short**：音频采样、小范围计数
- **int**：最常用的整数类型，循环计数、数组索引
- **long**：时间戳、大数计算、唯一ID
- **float**：3D图形、科学计算（精度要求不高）
- **double**：金融计算、科学计算（高精度）
- **char**：字符处理、Unicode支持
- **boolean**：条件判断、状态标识

**Q2: String、StringBuilder、StringBuffer的区别？**

**详细答案：**

| 特性 | String | StringBuilder | StringBuffer |
|------|--------|---------------|-------------|
| 可变性 | 不可变(Immutable) | 可变(Mutable) | 可变(Mutable) |
| 线程安全 | 安全 | 不安全 | 安全(synchronized) |
| 性能 | 慢(频繁创建对象) | 快 | 较快(有同步开销) |
| 内存使用 | 高(创建多个对象) | 低(复用缓冲区) | 低(复用缓冲区) |
| 适用场景 | 少量字符串操作 | 单线程大量操作 | 多线程大量操作 |
| JDK版本 | 1.0+ | 1.5+ | 1.0+ |

**原理分析：**

1. **String不可变性原理**：
   - String内部使用final char[]数组存储
   - 所有修改操作都会创建新的String对象
   - 字符串常量池优化相同字符串的存储

2. **StringBuilder/StringBuffer可变性原理**：
   - 内部使用可扩展的char[]数组
   - 修改操作直接在原数组上进行
   - 容量不足时自动扩容(通常是原容量的2倍+2)

**代码示例：**
```java
public class StringComparisonDemo {
    public static void main(String[] args) {
        // 性能对比测试
        performanceComparison();
        
        // 内存使用分析
        memoryUsageAnalysis();
        
        // 线程安全测试
        threadSafetyTest();
        
        // 实际应用场景
        practicalUseCases();
    }
    
    private static void performanceComparison() {
        int iterations = 10000;
        
        // String拼接性能测试
        long startTime = System.currentTimeMillis();
        String str = "";
        for (int i = 0; i < iterations; i++) {
            str += "a"; // 每次都创建新对象
        }
        long stringTime = System.currentTimeMillis() - startTime;
        
        // StringBuilder性能测试
        startTime = System.currentTimeMillis();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < iterations; i++) {
            sb.append("a"); // 在原对象上修改
        }
        String sbResult = sb.toString();
        long sbTime = System.currentTimeMillis() - startTime;
        
        // StringBuffer性能测试
        startTime = System.currentTimeMillis();
        StringBuffer sbf = new StringBuffer();
        for (int i = 0; i < iterations; i++) {
            sbf.append("a"); // 同步方法调用
        }
        String sbfResult = sbf.toString();
        long sbfTime = System.currentTimeMillis() - startTime;
        
        System.out.println("性能对比(" + iterations + "次拼接):");
        System.out.println("String: " + stringTime + "ms");
        System.out.println("StringBuilder: " + sbTime + "ms");
        System.out.println("StringBuffer: " + sbfTime + "ms");
    }
    
    private static void memoryUsageAnalysis() {
        // String的内存问题演示
        System.out.println("\n=== String内存分析 ===");
        
        // 字符串常量池
        String s1 = "Hello";
        String s2 = "Hello";
        String s3 = new String("Hello");
        
        System.out.println("s1 == s2: " + (s1 == s2)); // true (常量池)
        System.out.println("s1 == s3: " + (s1 == s3)); // false (堆中新对象)
        System.out.println("s1.equals(s3): " + s1.equals(s3)); // true (内容相同)
        
        // StringBuilder容量管理
        StringBuilder sb = new StringBuilder();
        System.out.println("初始容量: " + sb.capacity()); // 16
        
        sb.append("This is a long string that exceeds initial capacity");
        System.out.println("扩容后容量: " + sb.capacity()); // 自动扩容
        
        // 预设容量优化
        StringBuilder optimizedSb = new StringBuilder(100);
        System.out.println("预设容量: " + optimizedSb.capacity()); // 100
    }
    
    private static void threadSafetyTest() {
        System.out.println("\n=== 线程安全测试 ===");
        
        StringBuilder sb = new StringBuilder();
        StringBuffer sbf = new StringBuffer();
        
        // 多线程并发测试
        Thread[] threads = new Thread[10];
        
        // StringBuilder并发测试(不安全)
        for (int i = 0; i < threads.length; i++) {
            final int threadId = i;
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 100; j++) {
                    sb.append(threadId);
                }
            });
        }
        
        for (Thread thread : threads) {
            thread.start();
        }
        
        try {
            for (Thread thread : threads) {
                thread.join();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        System.out.println("StringBuilder长度: " + sb.length()); // 可能小于1000
        
        // StringBuffer并发测试(安全)
        Thread[] safeThreads = new Thread[10];
        for (int i = 0; i < safeThreads.length; i++) {
            final int threadId = i;
            safeThreads[i] = new Thread(() -> {
                for (int j = 0; j < 100; j++) {
                    sbf.append(threadId);
                }
            });
        }
        
        for (Thread thread : safeThreads) {
            thread.start();
        }
        
        try {
            for (Thread thread : safeThreads) {
                thread.join();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        System.out.println("StringBuffer长度: " + sbf.length()); // 总是1000
    }
    
    private static void practicalUseCases() {
        System.out.println("\n=== 实际应用场景 ===");
        
        // 1. 配置信息拼接(String适用)
        String config = "server.port=8080" + 
                       "&server.host=localhost" + 
                       "&server.timeout=30";
        
        // 2. 日志信息构建(StringBuilder适用)
        StringBuilder logBuilder = new StringBuilder()
            .append("[")
            .append(java.time.LocalDateTime.now())
            .append("] ")
            .append("INFO: ")
            .append("User login successful");
        
        // 3. SQL语句动态构建(StringBuilder适用)
        StringBuilder sqlBuilder = new StringBuilder("SELECT * FROM users WHERE 1=1");
        String name = "John";
        Integer age = 25;
        
        if (name != null) {
            sqlBuilder.append(" AND name = '").append(name).append("'");
        }
        if (age != null) {
            sqlBuilder.append(" AND age = ").append(age);
        }
        
        // 4. 多线程日志收集(StringBuffer适用)
        StringBuffer threadSafeLog = new StringBuffer();
        // 在多线程环境中安全地添加日志
        
        System.out.println("配置: " + config);
        System.out.println("日志: " + logBuilder.toString());
        System.out.println("SQL: " + sqlBuilder.toString());
    }
}
```

**选择建议：**

1. **使用String的场景**：
   - 字符串内容不会改变
   - 少量的字符串拼接操作
   - 需要利用字符串常量池优化内存

2. **使用StringBuilder的场景**：
   - 单线程环境下的大量字符串拼接
   - 动态构建SQL、JSON、XML等
   - 性能要求较高的字符串操作

3. **使用StringBuffer的场景**：
   - 多线程环境下的字符串拼接
   - 需要线程安全保证的场景
   - 遗留代码维护(现在更推荐StringBuilder+外部同步)

**性能优化技巧：**

1. **预估容量**：创建StringBuilder时指定合适的初始容量
2. **避免toString()频繁调用**：在构建完成后再调用toString()
3. **复用对象**：通过setLength(0)清空StringBuilder重复使用
4. **选择合适的方法**：append()比+操作符效率更高

**Q3: == 和 equals() 的区别？**

**详细答案：**

| 比较方式 | == | equals() |
|----------|----|---------|
| 基本类型 | 比较值 | 不适用(基本类型没有方法) |
| 引用类型 | 比较内存地址 | 比较对象内容(需正确重写) |
| 性能 | 快(直接比较) | 慢(方法调用+逻辑判断) |
| 可重写 | 不可重写 | 可以重写 |
| null处理 | 可以比较null | 调用者为null会抛NPE |

**原理分析：**

1. **== 操作符原理**：
   - 基本类型：直接比较栈中的值
   - 引用类型：比较堆中对象的内存地址
   - 编译时确定，运行时直接比较

2. **equals() 方法原理**：
   - Object类默认实现就是用==比较
   - 子类可以重写来实现自定义比较逻辑
   - 运行时动态调用，涉及方法查找和执行

**代码示例：**
```java
import java.util.*;

public class EqualsComparisonDemo {
    public static void main(String[] args) {
        // 基本类型比较
        primitiveComparison();
        
        // 字符串比较
        stringComparison();
        
        // 对象比较
        objectComparison();
        
        // 集合中的应用
        collectionUsage();
        
        // 常见陷阱
        commonPitfalls();
    }
    
    private static void primitiveComparison() {
        System.out.println("=== 基本类型比较 ===");
        
        int a = 100;
        int b = 100;
        System.out.println("int: a == b = " + (a == b)); // true
        
        Integer x = 100;
        Integer y = 100;
        Integer z = 200;
        Integer w = 200;
        
        System.out.println("Integer缓存范围内: x == y = " + (x == y)); // true
        System.out.println("Integer缓存范围外: z == w = " + (z == w)); // false
        System.out.println("Integer equals: z.equals(w) = " + z.equals(w)); // true
    }
    
    private static void stringComparison() {
        System.out.println("\n=== 字符串比较 ===");
        
        // 字符串常量池
        String s1 = "Hello";
        String s2 = "Hello";
        String s3 = new String("Hello");
        String s4 = new String("Hello");
        
        System.out.println("常量池: s1 == s2 = " + (s1 == s2)); // true
        System.out.println("new创建: s3 == s4 = " + (s3 == s4)); // false
        System.out.println("内容比较: s1.equals(s3) = " + s1.equals(s3)); // true
        
        // intern()方法
        String s5 = s3.intern();
        System.out.println("intern后: s1 == s5 = " + (s1 == s5)); // true
    }
    
    private static void objectComparison() {
        System.out.println("\n=== 对象比较 ===");
        
        // 使用自定义Person类
        Person p1 = new Person("张三", 25);
        Person p2 = new Person("张三", 25);
        Person p3 = p1;
        
        System.out.println("不同对象: p1 == p2 = " + (p1 == p2)); // false
        System.out.println("相同引用: p1 == p3 = " + (p1 == p3)); // true
        System.out.println("内容相同: p1.equals(p2) = " + p1.equals(p2)); // true
        
        // hashCode一致性
        System.out.println("p1.hashCode() = " + p1.hashCode());
        System.out.println("p2.hashCode() = " + p2.hashCode());
        System.out.println("hashCode相等: " + (p1.hashCode() == p2.hashCode())); // true
    }
    
    private static void collectionUsage() {
        System.out.println("\n=== 集合中的应用 ===");
        
        Set<Person> personSet = new HashSet<>();
        Person p1 = new Person("李四", 30);
        Person p2 = new Person("李四", 30);
        
        personSet.add(p1);
        personSet.add(p2); // 由于equals()返回true，不会重复添加
        
        System.out.println("Set大小: " + personSet.size()); // 1
        System.out.println("包含p2: " + personSet.contains(p2)); // true
        
        // Map中的应用
        Map<Person, String> personMap = new HashMap<>();
        personMap.put(p1, "员工1");
        
        System.out.println("通过p2获取值: " + personMap.get(p2)); // "员工1"
    }
    
    private static void commonPitfalls() {
        System.out.println("\n=== 常见陷阱 ===");
        
        // 1. null比较
        String nullStr = null;
        String normalStr = "test";
        
        System.out.println("null == null: " + (nullStr == null)); // true
        // System.out.println(nullStr.equals(normalStr)); // 会抛出NPE
        System.out.println("安全比较: " + Objects.equals(nullStr, normalStr)); // false
        
        // 2. 浮点数比较
        double d1 = 0.1 + 0.2;
        double d2 = 0.3;
        System.out.println("浮点数==: " + (d1 == d2)); // false (精度问题)
        System.out.println("浮点数差值: " + Math.abs(d1 - d2)); // 很小的差值
        System.out.println("精度比较: " + (Math.abs(d1 - d2) < 1e-10)); // true
        
        // 3. 数组比较
        int[] arr1 = {1, 2, 3};
        int[] arr2 = {1, 2, 3};
        System.out.println("数组==: " + (arr1 == arr2)); // false
        System.out.println("数组equals: " + arr1.equals(arr2)); // false
        System.out.println("数组内容比较: " + Arrays.equals(arr1, arr2)); // true
    }
}

// 正确实现equals和hashCode的示例类
class Person {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    @Override
    public boolean equals(Object obj) {
        // 1. 检查是否为同一个对象
        if (this == obj) return true;
        
        // 2. 检查是否为null或不同类型
        if (obj == null || getClass() != obj.getClass()) return false;
        
        // 3. 类型转换并比较字段
        Person person = (Person) obj;
        return age == person.age && Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        // 使用Objects.hash()生成hashCode
        return Objects.hash(name, age);
    }
    
    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + "}";
    }
    
    // getter和setter方法
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
}
```

**equals()和hashCode()契约：**

1. **自反性**：x.equals(x) 必须返回true
2. **对称性**：x.equals(y) 和 y.equals(x) 必须返回相同结果
3. **传递性**：如果x.equals(y)和y.equals(z)都为true，则x.equals(z)也必须为true
4. **一致性**：多次调用equals()必须返回相同结果
5. **非空性**：x.equals(null) 必须返回false
6. **hashCode一致性**：如果equals()返回true，则hashCode()必须相等

**最佳实践：**

1. **重写equals()时必须重写hashCode()**
2. **使用Objects.equals()进行null安全比较**
3. **使用Objects.hash()生成hashCode**
4. **考虑使用IDE自动生成或Lombok注解**
5. **浮点数比较使用精度范围**
6. **数组比较使用Arrays.equals()**

**性能考虑：**

1. **==比较性能最高**：直接内存地址比较
2. **equals()性能取决于实现**：避免复杂计算
3. **hashCode()要高效**：用于HashMap等集合的性能
4. **缓存hashCode**：对于不可变对象可以缓存hashCode值

### 1.2 面向对象类

**Q4: Java中的继承、封装、多态分别是什么？**

**答案：**
- **继承**：子类继承父类的属性和方法，实现代码复用
- **封装**：隐藏内部实现细节，通过访问修饰符控制访问权限
- **多态**：同一接口的不同实现，运行时动态绑定

**Q5: 抽象类和接口的区别？**

**答案：**
- **抽象类**：可以有构造方法、成员变量、具体方法；单继承
- **接口**：只能有常量、抽象方法（Java 8+可有默认方法）；多实现
- **使用场景**：抽象类用于"is-a"关系，接口用于"can-do"关系

### 1.3 集合框架类

**Q6: ArrayList和LinkedList的区别？**

**答案：**
- **ArrayList**：基于数组，随机访问快O(1)，插入删除慢O(n)
- **LinkedList**：基于链表，随机访问慢O(n)，插入删除快O(1)
- **选择原则**：频繁查询用ArrayList，频繁增删用LinkedList

**Q7: HashMap的实现原理？**

**详细答案：**

HashMap是Java中最常用的Map实现，基于哈希表数据结构，提供O(1)的平均时间复杂度。

**核心数据结构：**

| JDK版本 | 数据结构 | 冲突解决 | 性能特点 |
|---------|----------|----------|----------|
| JDK 7及以前 | 数组+链表 | 链地址法 | 冲突多时性能退化严重 |
| JDK 8+ | 数组+链表+红黑树 | 链地址法+树化 | 最坏情况O(log n) |

**实现原理详解：**

1. **底层存储结构**：
   ```java
   // JDK 8+ HashMap内部结构
   transient Node<K,V>[] table; // 哈希桶数组
   transient int size;           // 实际存储的键值对数量
   int threshold;                // 扩容阈值 = capacity * loadFactor
   final float loadFactor;       // 负载因子，默认0.75
   ```

2. **Node节点结构**：
   ```java
   static class Node<K,V> implements Map.Entry<K,V> {
       final int hash;    // 哈希值
       final K key;       // 键
       V value;           // 值
       Node<K,V> next;    // 链表下一个节点
   }
   ```

**代码示例：**
```java
import java.util.*;
import java.lang.reflect.Field;

public class HashMapInternalsDemo {
    public static void main(String[] args) {
        // HashMap基本操作演示
        basicOperations();
        
        // 哈希冲突演示
        hashCollisionDemo();
        
        // 扩容机制演示
        resizingDemo();
        
        // 红黑树转换演示
        treeificationDemo();
        
        // 性能分析
        performanceAnalysis();
    }
    
    private static void basicOperations() {
        System.out.println("=== HashMap基本操作 ===");
        
        Map<String, Integer> map = new HashMap<>();
        
        // put操作的内部流程
        System.out.println("1. 计算hash值");
        String key = "Java";
        int hash = key.hashCode();
        System.out.println("key=" + key + ", hashCode=" + hash);
        
        // 模拟HashMap的hash()方法
        int finalHash = hash ^ (hash >>> 16);
        System.out.println("HashMap内部hash=" + finalHash);
        
        // 计算数组索引
        int capacity = 16; // 默认初始容量
        int index = finalHash & (capacity - 1);
        System.out.println("数组索引=" + index);
        
        map.put(key, 100);
        System.out.println("存储完成: " + map);
    }
    
    private static void hashCollisionDemo() {
        System.out.println("\n=== 哈希冲突演示 ===");
        
        // 创建会产生哈希冲突的键
        Map<HashCollisionKey, String> map = new HashMap<>();
        
        HashCollisionKey key1 = new HashCollisionKey("A", 1);
        HashCollisionKey key2 = new HashCollisionKey("B", 1); // 相同hashCode
        HashCollisionKey key3 = new HashCollisionKey("C", 1); // 相同hashCode
        
        map.put(key1, "Value1");
        map.put(key2, "Value2");
        map.put(key3, "Value3");
        
        System.out.println("冲突键的hashCode: " + key1.hashCode());
        System.out.println("Map大小: " + map.size());
        System.out.println("所有值都能正确获取:");
        System.out.println("key1: " + map.get(key1));
        System.out.println("key2: " + map.get(key2));
        System.out.println("key3: " + map.get(key3));
    }
    
    private static void resizingDemo() {
        System.out.println("\n=== 扩容机制演示 ===");
        
        Map<Integer, String> map = new HashMap<>(4); // 初始容量4
        
        System.out.println("初始容量: 4, 扩容阈值: " + (4 * 0.75));
        
        // 添加元素触发扩容
        for (int i = 1; i <= 10; i++) {
            map.put(i, "Value" + i);
            
            if (i == 3) {
                System.out.println("添加第3个元素后，即将触发扩容");
            } else if (i == 4) {
                System.out.println("添加第4个元素，触发扩容，新容量: 8");
            } else if (i == 6) {
                System.out.println("扩容阈值现在是: " + (8 * 0.75));
            } else if (i == 7) {
                System.out.println("添加第7个元素，即将再次扩容");
            } else if (i == 8) {
                System.out.println("触发第二次扩容，新容量: 16");
            }
        }
        
        System.out.println("最终Map大小: " + map.size());
    }
    
    private static void treeificationDemo() {
        System.out.println("\n=== 红黑树转换演示 ===");
        
        // 创建大量哈希冲突来触发树化
        Map<TreeificationKey, String> map = new HashMap<>();
        
        System.out.println("添加9个哈希冲突的键值对...");
        for (int i = 0; i < 9; i++) {
            TreeificationKey key = new TreeificationKey("Key" + i);
            map.put(key, "Value" + i);
        }
        
        System.out.println("当链表长度达到8时，会转换为红黑树");
        System.out.println("Map大小: " + map.size());
        
        // 验证所有值都能正确获取
        boolean allFound = true;
        for (int i = 0; i < 9; i++) {
            TreeificationKey key = new TreeificationKey("Key" + i);
            String value = map.get(key);
            if (!value.equals("Value" + i)) {
                allFound = false;
                break;
            }
        }
        
        System.out.println("所有值都能正确获取: " + allFound);
    }
    
    private static void performanceAnalysis() {
        System.out.println("\n=== 性能分析 ===");
        
        int size = 100000;
        
        // HashMap性能测试
        Map<Integer, String> hashMap = new HashMap<>();
        long startTime = System.currentTimeMillis();
        
        for (int i = 0; i < size; i++) {
            hashMap.put(i, "Value" + i);
        }
        
        long putTime = System.currentTimeMillis() - startTime;
        
        startTime = System.currentTimeMillis();
        for (int i = 0; i < size; i++) {
            hashMap.get(i);
        }
        long getTime = System.currentTimeMillis() - startTime;
        
        System.out.println("HashMap (" + size + "个元素):");
        System.out.println("Put操作耗时: " + putTime + "ms");
        System.out.println("Get操作耗时: " + getTime + "ms");
        
        // 与TreeMap对比
        Map<Integer, String> treeMap = new TreeMap<>();
        startTime = System.currentTimeMillis();
        
        for (int i = 0; i < size; i++) {
            treeMap.put(i, "Value" + i);
        }
        
        long treeMapPutTime = System.currentTimeMillis() - startTime;
        
        startTime = System.currentTimeMillis();
        for (int i = 0; i < size; i++) {
            treeMap.get(i);
        }
        long treeMapGetTime = System.currentTimeMillis() - startTime;
        
        System.out.println("\nTreeMap (" + size + "个元素):");
        System.out.println("Put操作耗时: " + treeMapPutTime + "ms");
        System.out.println("Get操作耗时: " + treeMapGetTime + "ms");
        
        System.out.println("\n性能对比:");
        System.out.println("HashMap Put速度是TreeMap的 " + 
                          (double)treeMapPutTime / putTime + " 倍");
        System.out.println("HashMap Get速度是TreeMap的 " + 
                          (double)treeMapGetTime / getTime + " 倍");
    }
}

// 用于演示哈希冲突的类
class HashCollisionKey {
    private String name;
    private int fixedHash;
    
    public HashCollisionKey(String name, int fixedHash) {
        this.name = name;
        this.fixedHash = fixedHash;
    }
    
    @Override
    public int hashCode() {
        return fixedHash; // 固定返回相同的hash值
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        HashCollisionKey that = (HashCollisionKey) obj;
        return Objects.equals(name, that.name);
    }
    
    @Override
    public String toString() {
        return "HashCollisionKey{name='" + name + "'}";
    }
}

// 用于演示树化的类
class TreeificationKey implements Comparable<TreeificationKey> {
    private String name;
    
    public TreeificationKey(String name) {
        this.name = name;
    }
    
    @Override
    public int hashCode() {
        return 1; // 所有对象返回相同hash值，强制冲突
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        TreeificationKey that = (TreeificationKey) obj;
        return Objects.equals(name, that.name);
    }
    
    @Override
    public int compareTo(TreeificationKey other) {
        return this.name.compareTo(other.name);
    }
    
    @Override
    public String toString() {
        return "TreeificationKey{name='" + name + "'}";
    }
}
```

**关键机制详解：**

1. **哈希函数优化**：
   ```java
   static final int hash(Object key) {
       int h;
       return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
   }
   ```
   - 高16位与低16位异或，减少冲突
   - null键的hash值为0，存储在索引0位置

2. **索引计算**：
   ```java
   int index = (n - 1) & hash; // n为数组长度，必须是2的幂
   ```
   - 使用位运算替代取模，提高性能
   - 要求数组长度必须是2的幂次方

3. **扩容条件**：
   - 当前元素数量 > 阈值(capacity * loadFactor)
   - 链表长度达到8且数组长度小于64时，优先扩容而非树化

4. **树化条件**：
   - 链表长度 ≥ 8
   - 数组长度 ≥ 64
   - 树化后如果节点数量 ≤ 6，会退化回链表

**性能特点：**

| 操作 | 平均时间复杂度 | 最坏时间复杂度 | 说明 |
|------|----------------|----------------|------|
| get | O(1) | O(log n) | JDK8+红黑树优化 |
| put | O(1) | O(log n) | 包含扩容时间 |
| remove | O(1) | O(log n) | 同get操作 |
| containsKey | O(1) | O(log n) | 同get操作 |

**使用建议：**

1. **初始容量设置**：预估数据量，避免频繁扩容
2. **负载因子选择**：默认0.75平衡了时间和空间
3. **键对象要求**：正确实现hashCode()和equals()
4. **线程安全**：多线程环境使用ConcurrentHashMap
5. **内存优化**：及时清理不需要的键值对

**Q8: ConcurrentHashMap的实现原理？**

**答案：**
- **JDK 7**：分段锁（Segment），减少锁竞争
- **JDK 8**：CAS+synchronized，锁粒度更细
- **优势**：高并发性能好，读操作无锁

---

## 2. Java进阶面试题

### 2.1 多线程类

**Q9: 线程的生命周期有哪些状态？**

**答案：**
1. **NEW**：线程创建但未启动
2. **RUNNABLE**：可运行状态（包括运行和就绪）
3. **BLOCKED**：阻塞等待监视器锁
4. **WAITING**：无限期等待
5. **TIMED_WAITING**：有时限等待
6. **TERMINATED**：线程终止

**Q10: synchronized和ReentrantLock的区别？**

**答案：**
- **synchronized**：JVM层面，自动释放锁，不可中断
- **ReentrantLock**：API层面，手动释放锁，可中断，支持公平锁
- **性能**：JDK 6+后synchronized性能优化，差距不大

**Q11: 线程池的核心参数和工作原理？**

**详细答案：**

线程池是Java并发编程中的核心组件，通过复用线程来提高性能和资源利用率。

**核心参数详解：**

| 参数 | 类型 | 作用 | 建议值 |
|------|------|------|--------|
| corePoolSize | int | 核心线程数，即使空闲也保持存活 | CPU密集型：CPU核数+1<br>IO密集型：2*CPU核数 |
| maximumPoolSize | int | 最大线程数，包括核心和非核心线程 | 根据系统负载能力设定 |
| keepAliveTime | long | 非核心线程空闲存活时间 | 30-60秒 |
| unit | TimeUnit | keepAliveTime的时间单位 | SECONDS或MILLISECONDS |
| workQueue | BlockingQueue | 任务等待队列 | 根据场景选择合适类型 |
| threadFactory | ThreadFactory | 线程创建工厂 | 自定义线程名称和属性 |
| handler | RejectedExecutionHandler | 拒绝策略 | 根据业务需求选择 |

**工作队列类型：**

| 队列类型 | 特点 | 适用场景 | 容量 |
|----------|------|----------|------|
| ArrayBlockingQueue | 有界队列，FIFO | 资源有限，需要控制内存 | 固定大小 |
| LinkedBlockingQueue | 可选有界，FIFO | 一般场景，默认选择 | 可设置或无界 |
| SynchronousQueue | 不存储元素，直接传递 | 高并发，快速响应 | 0 |
| PriorityBlockingQueue | 优先级队列 | 任务有优先级要求 | 无界 |
| DelayQueue | 延迟队列 | 定时任务 | 无界 |

**拒绝策略：**

| 策略 | 行为 | 适用场景 |
|------|------|----------|
| AbortPolicy | 抛出异常(默认) | 需要感知任务丢失 |
| CallerRunsPolicy | 调用者线程执行 | 重要任务不能丢失 |
| DiscardPolicy | 静默丢弃 | 允许任务丢失 |
| DiscardOldestPolicy | 丢弃最老任务 | 新任务优先级更高 |

**代码示例：**
```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

public class ThreadPoolDemo {
    public static void main(String[] args) {
        // 线程池工作原理演示
        demonstrateWorkingPrinciple();
        
        // 不同队列类型演示
        demonstrateQueueTypes();
        
        // 拒绝策略演示
        demonstrateRejectionPolicies();
        
        // 线程池监控
        demonstrateMonitoring();
        
        // 最佳实践
        bestPractices();
    }
    
    private static void demonstrateWorkingPrinciple() {
        System.out.println("=== 线程池工作原理演示 ===");
        
        // 创建自定义线程池
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            2,                          // corePoolSize
            4,                          // maximumPoolSize
            60L,                        // keepAliveTime
            TimeUnit.SECONDS,           // unit
            new ArrayBlockingQueue<>(2), // workQueue，容量为2
            new CustomThreadFactory("Demo"), // threadFactory
            new ThreadPoolExecutor.AbortPolicy() // handler
        );
        
        System.out.println("线程池配置:");
        System.out.println("核心线程数: " + executor.getCorePoolSize());
        System.out.println("最大线程数: " + executor.getMaximumPoolSize());
        System.out.println("队列容量: 2");
        
        // 提交任务演示工作流程
        for (int i = 1; i <= 8; i++) {
            final int taskId = i;
            try {
                executor.submit(() -> {
                    System.out.println("任务" + taskId + " 开始执行，线程: " + 
                                     Thread.currentThread().getName());
                    try {
                        Thread.sleep(3000); // 模拟任务执行
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                    System.out.println("任务" + taskId + " 执行完成");
                });
                
                System.out.println("提交任务" + taskId + 
                                 ", 当前线程数: " + executor.getPoolSize() +
                                 ", 队列大小: " + executor.getQueue().size());
                
                Thread.sleep(500); // 间隔提交
            } catch (RejectedExecutionException e) {
                System.out.println("任务" + taskId + " 被拒绝: " + e.getMessage());
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
        
        executor.shutdown();
    }
    
    private static void demonstrateQueueTypes() {
        System.out.println("\n=== 不同队列类型演示 ===");
        
        // 1. ArrayBlockingQueue - 有界队列
        System.out.println("1. ArrayBlockingQueue (有界队列):");
        ThreadPoolExecutor arrayQueuePool = new ThreadPoolExecutor(
            1, 1, 0L, TimeUnit.MILLISECONDS,
            new ArrayBlockingQueue<>(2)
        );
        
        testQueueBehavior(arrayQueuePool, "ArrayBlockingQueue");
        
        // 2. LinkedBlockingQueue - 无界队列
        System.out.println("\n2. LinkedBlockingQueue (无界队列):");
        ThreadPoolExecutor linkedQueuePool = new ThreadPoolExecutor(
            1, 3, 0L, TimeUnit.MILLISECONDS,
            new LinkedBlockingQueue<>() // 无界
        );
        
        testQueueBehavior(linkedQueuePool, "LinkedBlockingQueue");
        
        // 3. SynchronousQueue - 直接传递
        System.out.println("\n3. SynchronousQueue (直接传递):");
        ThreadPoolExecutor syncQueuePool = new ThreadPoolExecutor(
            0, 3, 60L, TimeUnit.SECONDS,
            new SynchronousQueue<>()
        );
        
        testQueueBehavior(syncQueuePool, "SynchronousQueue");
    }
    
    private static void testQueueBehavior(ThreadPoolExecutor executor, String queueType) {
        for (int i = 1; i <= 5; i++) {
            final int taskId = i;
            try {
                executor.submit(() -> {
                    try {
                        Thread.sleep(1000);
                        System.out.println(queueType + " - 任务" + taskId + "完成");
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                });
                System.out.println(queueType + " - 提交任务" + taskId + 
                                 ", 线程数: " + executor.getPoolSize() +
                                 ", 队列大小: " + executor.getQueue().size());
            } catch (RejectedExecutionException e) {
                System.out.println(queueType + " - 任务" + taskId + "被拒绝");
            }
        }
        
        executor.shutdown();
        try {
            executor.awaitTermination(5, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    private static void demonstrateRejectionPolicies() {
        System.out.println("\n=== 拒绝策略演示 ===");
        
        // 1. AbortPolicy - 抛出异常
        System.out.println("1. AbortPolicy (抛出异常):");
        testRejectionPolicy(new ThreadPoolExecutor.AbortPolicy(), "AbortPolicy");
        
        // 2. CallerRunsPolicy - 调用者执行
        System.out.println("\n2. CallerRunsPolicy (调用者执行):");
        testRejectionPolicy(new ThreadPoolExecutor.CallerRunsPolicy(), "CallerRunsPolicy");
        
        // 3. DiscardPolicy - 静默丢弃
        System.out.println("\n3. DiscardPolicy (静默丢弃):");
        testRejectionPolicy(new ThreadPoolExecutor.DiscardPolicy(), "DiscardPolicy");
        
        // 4. DiscardOldestPolicy - 丢弃最老任务
        System.out.println("\n4. DiscardOldestPolicy (丢弃最老任务):");
        testRejectionPolicy(new ThreadPoolExecutor.DiscardOldestPolicy(), "DiscardOldestPolicy");
    }
    
    private static void testRejectionPolicy(RejectedExecutionHandler handler, String policyName) {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            1, 1, 0L, TimeUnit.MILLISECONDS,
            new ArrayBlockingQueue<>(1),
            handler
        );
        
        for (int i = 1; i <= 4; i++) {
            final int taskId = i;
            try {
                executor.submit(() -> {
                    try {
                        Thread.sleep(2000);
                        System.out.println(policyName + " - 任务" + taskId + "完成");
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                });
                System.out.println(policyName + " - 提交任务" + taskId);
            } catch (RejectedExecutionException e) {
                System.out.println(policyName + " - 任务" + taskId + "被拒绝: " + e.getClass().getSimpleName());
            }
        }
        
        executor.shutdown();
    }
    
    private static void demonstrateMonitoring() {
        System.out.println("\n=== 线程池监控演示 ===");
        
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            2, 4, 60L, TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(10),
            new CustomThreadFactory("Monitor")
        );
        
        // 提交一些任务
        for (int i = 1; i <= 8; i++) {
            final int taskId = i;
            executor.submit(() -> {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        
        // 监控线程池状态
        ScheduledExecutorService monitor = Executors.newScheduledThreadPool(1);
        monitor.scheduleAtFixedRate(() -> {
            System.out.println("线程池状态监控:");
            System.out.println("  当前线程数: " + executor.getPoolSize());
            System.out.println("  活跃线程数: " + executor.getActiveCount());
            System.out.println("  队列大小: " + executor.getQueue().size());
            System.out.println("  已完成任务数: " + executor.getCompletedTaskCount());
            System.out.println("  总任务数: " + executor.getTaskCount());
            System.out.println("  是否关闭: " + executor.isShutdown());
            System.out.println("  是否终止: " + executor.isTerminated());
            System.out.println("---");
        }, 0, 1, TimeUnit.SECONDS);
        
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        executor.shutdown();
        monitor.shutdown();
    }
    
    private static void bestPractices() {
        System.out.println("\n=== 最佳实践演示 ===");
        
        // 1. 自定义线程工厂
        ThreadFactory customFactory = new ThreadFactory() {
            private final AtomicInteger threadNumber = new AtomicInteger(1);
            
            @Override
            public Thread newThread(Runnable r) {
                Thread t = new Thread(r, "CustomPool-" + threadNumber.getAndIncrement());
                t.setDaemon(false); // 非守护线程
                t.setPriority(Thread.NORM_PRIORITY);
                return t;
            }
        };
        
        // 2. 自定义拒绝策略
        RejectedExecutionHandler customHandler = (r, executor) -> {
            System.out.println("任务被拒绝，尝试重新提交: " + r.toString());
            try {
                // 等待一段时间后重试
                Thread.sleep(100);
                if (!executor.isShutdown()) {
                    executor.getQueue().offer(r, 1, TimeUnit.SECONDS);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };
        
        // 3. 创建优化的线程池
        ThreadPoolExecutor optimizedPool = new ThreadPoolExecutor(
            Runtime.getRuntime().availableProcessors(),
            Runtime.getRuntime().availableProcessors() * 2,
            60L, TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(100),
            customFactory,
            customHandler
        );
        
        // 4. 预启动核心线程
        optimizedPool.prestartAllCoreThreads();
        
        System.out.println("优化后的线程池配置:");
        System.out.println("核心线程数: " + optimizedPool.getCorePoolSize());
        System.out.println("最大线程数: " + optimizedPool.getMaximumPoolSize());
        System.out.println("预启动的核心线程数: " + optimizedPool.getPoolSize());
        
        optimizedPool.shutdown();
    }
}

// 自定义线程工厂
class CustomThreadFactory implements ThreadFactory {
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;
    
    public CustomThreadFactory(String namePrefix) {
        this.namePrefix = namePrefix + "-thread-";
    }
    
    @Override
    public Thread newThread(Runnable r) {
        Thread t = new Thread(r, namePrefix + threadNumber.getAndIncrement());
        if (t.isDaemon()) {
            t.setDaemon(false);
        }
        if (t.getPriority() != Thread.NORM_PRIORITY) {
            t.setPriority(Thread.NORM_PRIORITY);
        }
        return t;
    }
}
```

**线程池执行流程图：**

```
提交任务
    ↓
核心线程数是否已满？
    ↓ 否                    ↓ 是
创建新的核心线程    →    队列是否已满？
    ↓                        ↓ 否              ↓ 是
执行任务            →    任务入队    →    最大线程数是否已满？
                                            ↓ 否              ↓ 是
                                        创建非核心线程    →    执行拒绝策略
                                            ↓
                                        执行任务
```

**常见线程池类型：**

| 线程池类型 | 核心线程数 | 最大线程数 | 队列 | 适用场景 |
|------------|------------|------------|------|----------|
| FixedThreadPool | n | n | LinkedBlockingQueue | 固定负载 |
| CachedThreadPool | 0 | Integer.MAX_VALUE | SynchronousQueue | 短期异步任务 |
| SingleThreadExecutor | 1 | 1 | LinkedBlockingQueue | 顺序执行 |
| ScheduledThreadPool | n | Integer.MAX_VALUE | DelayedWorkQueue | 定时任务 |

**性能调优建议：**

1. **CPU密集型任务**：线程数 = CPU核数 + 1
2. **IO密集型任务**：线程数 = CPU核数 × (1 + IO等待时间/CPU计算时间)
3. **混合型任务**：分离CPU密集和IO密集任务，使用不同线程池
4. **队列选择**：有界队列控制内存，无界队列避免任务丢失
5. **监控指标**：线程数、队列大小、任务完成率、拒绝率

**注意事项：**

1. **避免使用Executors创建线程池**：容易导致OOM
2. **合理设置队列大小**：防止内存溢出
3. **自定义线程名称**：便于问题排查
4. **优雅关闭**：使用shutdown()而非shutdownNow()
5. **异常处理**：提交任务时处理RejectedExecutionException

**Q12: volatile关键字的作用？**

**答案：**
- **可见性**：保证变量修改对所有线程立即可见
- **禁止重排序**：防止指令重排序优化
- **不保证原子性**：复合操作仍需要同步
- **使用场景**：状态标志、双重检查锁定

### 2.2 JVM类

**Q13: JVM内存结构包括哪些区域？**

**答案：**
- **程序计数器**：当前线程执行字节码的行号指示器
- **虚拟机栈**：存储局部变量、操作数栈、方法出口
- **本地方法栈**：为Native方法服务
- **堆**：存储对象实例，分为新生代和老年代
- **方法区**：存储类信息、常量、静态变量

**Q14: 垃圾回收算法有哪些？**

**答案：**
- **标记-清除**：标记垃圾对象后清除，产生内存碎片
- **复制算法**：将存活对象复制到另一块内存，适合新生代
- **标记-整理**：标记后移动存活对象，适合老年代
- **分代收集**：根据对象生命周期采用不同算法

**Q15: 常见的垃圾收集器有哪些？**

**答案：**
- **Serial**：单线程，适合小应用
- **Parallel**：多线程，适合吞吐量优先
- **CMS**：并发收集，适合响应时间优先
- **G1**：低延迟，适合大堆内存
- **ZGC/Shenandoah**：超低延迟收集器

### 2.3 IO类

**Q16: BIO、NIO、AIO的区别？**

**答案：**
- **BIO**：同步阻塞，一个连接一个线程
- **NIO**：同步非阻塞，多路复用，一个线程处理多个连接
- **AIO**：异步非阻塞，操作系统完成后通知应用程序

**Q17: NIO的核心组件有哪些？**

**答案：**
- **Channel**：数据传输通道
- **Buffer**：数据缓冲区
- **Selector**：多路复用器，监控多个Channel

---

## 3. 框架技术面试题

### 3.1 Spring框架类

**Q18: Spring的核心特性有哪些？**

**详细答案：**

Spring框架是Java企业级应用开发的核心框架，提供了全面的编程和配置模型。

**Spring核心特性详解：**

| 核心特性 | 描述 | 主要组件 | 优势 |
|----------|------|----------|------|
| IoC容器 | 控制反转，对象创建和依赖管理 | BeanFactory, ApplicationContext | 降低耦合，提高可测试性 |
| AOP | 面向切面编程，横切关注点分离 | AspectJ, Spring AOP | 代码复用，关注点分离 |
| 事务管理 | 声明式和编程式事务支持 | PlatformTransactionManager | 简化事务处理，支持多数据源 |
| MVC框架 | Web应用开发框架 | DispatcherServlet, Controller | 分层架构，灵活配置 |
| 数据访问 | 统一的数据访问抽象 | JdbcTemplate, ORM集成 | 简化数据库操作 |
| 测试支持 | 集成测试框架 | Spring Test Context | 便于单元测试和集成测试 |

**1. IoC容器（控制反转）**

**核心概念：**
- **依赖注入（DI）**：对象不再主动创建依赖，而是被动接收
- **容器管理**：Spring容器负责对象的创建、配置和生命周期管理
- **配置方式**：XML配置、注解配置、Java配置

**容器类型：**

| 容器类型 | 特点 | 适用场景 | 主要接口 |
|----------|------|----------|----------|
| BeanFactory | 基础容器，延迟加载 | 资源受限环境 | BeanFactory |
| ApplicationContext | 高级容器，立即加载 | 企业级应用 | ApplicationContext |

**2. AOP（面向切面编程）**

**核心概念：**
- **切面（Aspect）**：横切关注点的模块化
- **连接点（Join Point）**：程序执行的特定点
- **切入点（Pointcut）**：连接点的集合
- **通知（Advice）**：在切入点执行的代码
- **织入（Weaving）**：将切面应用到目标对象的过程

**通知类型：**

| 通知类型 | 执行时机 | 注解 | 用途 |
|----------|----------|------|------|
| Before | 方法执行前 | @Before | 参数验证，权限检查 |
| After | 方法执行后 | @After | 资源清理 |
| AfterReturning | 方法正常返回后 | @AfterReturning | 结果处理 |
| AfterThrowing | 方法抛异常后 | @AfterThrowing | 异常处理 |
| Around | 方法执行前后 | @Around | 性能监控，事务管理 |

**3. 事务管理**

**事务管理方式：**
- **声明式事务**：通过注解或XML配置
- **编程式事务**：通过代码控制事务

**事务传播行为：**

| 传播行为 | 描述 | 使用场景 |
|----------|------|----------|
| REQUIRED | 需要事务，没有则创建 | 默认行为 |
| REQUIRES_NEW | 总是创建新事务 | 独立事务操作 |
| SUPPORTS | 支持事务，没有也可以 | 查询操作 |
| NOT_SUPPORTED | 不支持事务 | 非事务操作 |
| MANDATORY | 必须在事务中运行 | 严格事务要求 |
| NEVER | 不能在事务中运行 | 特殊业务需求 |
| NESTED | 嵌套事务 | 部分回滚场景 |

**代码示例：**
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.*;
import org.springframework.stereotype.*;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.aop.framework.AopContext;
import org.aspectj.lang.annotation.*;
import org.aspectj.lang.ProceedingJoinPoint;
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

// 1. IoC容器演示
@Configuration
@EnableAspectJAutoProxy
@ComponentScan("com.example")
public class SpringCoreDemo {
    
    public static void main(String[] args) {
        // 创建Spring容器
        ApplicationContext context = new AnnotationConfigApplicationContext(SpringCoreDemo.class);
        
        // IoC容器演示
        demonstrateIoC(context);
        
        // AOP演示
        demonstrateAOP(context);
        
        // 事务管理演示
        demonstrateTransaction(context);
    }
    
    private static void demonstrateIoC(ApplicationContext context) {
        System.out.println("=== IoC容器演示 ===");
        
        // 获取Bean
        UserService userService = context.getBean(UserService.class);
        userService.createUser("张三", "zhangsan@example.com");
        
        // 显示依赖注入
        System.out.println("UserService依赖: " + userService.getUserRepository().getClass().getSimpleName());
        System.out.println("UserService配置: " + userService.getAppConfig().getAppName());
    }
    
    private static void demonstrateAOP(ApplicationContext context) {
        System.out.println("\n=== AOP演示 ===");
        
        UserService userService = context.getBean(UserService.class);
        
        // 触发AOP切面
        userService.findUser(1L);
        userService.updateUser(1L, "李四");
        
        try {
            userService.deleteUser(999L); // 触发异常处理切面
        } catch (Exception e) {
            System.out.println("捕获异常: " + e.getMessage());
        }
    }
    
    private static void demonstrateTransaction(ApplicationContext context) {
        System.out.println("\n=== 事务管理演示 ===");
        
        UserService userService = context.getBean(UserService.class);
        
        // 事务方法调用
        userService.transferUser(1L, 2L);
        
        // 事务回滚演示
        try {
            userService.createUserWithError("错误用户", "error@example.com");
        } catch (Exception e) {
            System.out.println("事务回滚: " + e.getMessage());
        }
    }
}

// 2. 业务服务类 - 演示IoC和事务
@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private AppConfig appConfig;
    
    @PostConstruct
    public void init() {
        System.out.println("UserService初始化完成");
    }
    
    @PreDestroy
    public void destroy() {
        System.out.println("UserService销毁");
    }
    
    // 普通方法 - 会被AOP拦截
    public void createUser(String name, String email) {
        System.out.println("创建用户: " + name + ", " + email);
        userRepository.save(name, email);
    }
    
    public User findUser(Long id) {
        System.out.println("查找用户: " + id);
        return userRepository.findById(id);
    }
    
    public void updateUser(Long id, String name) {
        System.out.println("更新用户: " + id + " -> " + name);
        userRepository.update(id, name);
    }
    
    public void deleteUser(Long id) {
        if (id == 999L) {
            throw new RuntimeException("无法删除用户: " + id);
        }
        System.out.println("删除用户: " + id);
        userRepository.delete(id);
    }
    
    // 事务方法
    @Transactional
    public void transferUser(Long fromId, Long toId) {
        System.out.println("开始用户转移事务: " + fromId + " -> " + toId);
        
        User fromUser = userRepository.findById(fromId);
        User toUser = userRepository.findById(toId);
        
        // 模拟业务操作
        userRepository.update(fromId, fromUser.getName() + "_transferred");
        userRepository.update(toId, toUser.getName() + "_received");
        
        System.out.println("用户转移事务完成");
    }
    
    @Transactional(rollbackFor = Exception.class)
    public void createUserWithError(String name, String email) {
        System.out.println("创建用户(会出错): " + name);
        userRepository.save(name, email);
        
        // 模拟业务异常，触发事务回滚
        throw new RuntimeException("业务处理失败");
    }
    
    // Getter方法用于演示
    public UserRepository getUserRepository() {
        return userRepository;
    }
    
    public AppConfig getAppConfig() {
        return appConfig;
    }
}

// 3. 数据访问层 - 演示依赖注入
@Repository
public class UserRepository {
    
    public void save(String name, String email) {
        System.out.println("保存用户到数据库: " + name + ", " + email);
    }
    
    public User findById(Long id) {
        System.out.println("从数据库查询用户: " + id);
        return new User(id, "用户" + id, "user" + id + "@example.com");
    }
    
    public void update(Long id, String name) {
        System.out.println("更新数据库用户: " + id + " -> " + name);
    }
    
    public void delete(Long id) {
        System.out.println("从数据库删除用户: " + id);
    }
}

// 4. 配置类 - 演示配置管理
@Component
public class AppConfig {
    
    private String appName = "Spring核心特性演示";
    private String version = "1.0.0";
    
    public String getAppName() {
        return appName;
    }
    
    public String getVersion() {
        return version;
    }
}

// 5. AOP切面类 - 演示面向切面编程
@Aspect
@Component
public class UserServiceAspect {
    
    // 定义切入点
    @Pointcut("execution(* com.example.UserService.*(..))")
    public void userServiceMethods() {}
    
    // 前置通知
    @Before("userServiceMethods()")
    public void beforeMethod(org.aspectj.lang.JoinPoint joinPoint) {
        System.out.println("[AOP-Before] 执行方法: " + joinPoint.getSignature().getName());
        Object[] args = joinPoint.getArgs();
        if (args.length > 0) {
            System.out.println("[AOP-Before] 方法参数: " + java.util.Arrays.toString(args));
        }
    }
    
    // 后置通知
    @After("userServiceMethods()")
    public void afterMethod(org.aspectj.lang.JoinPoint joinPoint) {
        System.out.println("[AOP-After] 方法执行完成: " + joinPoint.getSignature().getName());
    }
    
    // 返回通知
    @AfterReturning(pointcut = "userServiceMethods()", returning = "result")
    public void afterReturning(org.aspectj.lang.JoinPoint joinPoint, Object result) {
        System.out.println("[AOP-AfterReturning] 方法返回: " + joinPoint.getSignature().getName());
        if (result != null) {
            System.out.println("[AOP-AfterReturning] 返回值: " + result);
        }
    }
    
    // 异常通知
    @AfterThrowing(pointcut = "userServiceMethods()", throwing = "ex")
    public void afterThrowing(org.aspectj.lang.JoinPoint joinPoint, Exception ex) {
        System.out.println("[AOP-AfterThrowing] 方法异常: " + joinPoint.getSignature().getName());
        System.out.println("[AOP-AfterThrowing] 异常信息: " + ex.getMessage());
    }
    
    // 环绕通知 - 性能监控
    @Around("execution(* com.example.UserService.find*(..))")
    public Object aroundFindMethods(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();
        
        System.out.println("[AOP-Around] 开始执行查询方法: " + joinPoint.getSignature().getName());
        
        try {
            Object result = joinPoint.proceed();
            long endTime = System.currentTimeMillis();
            System.out.println("[AOP-Around] 查询方法执行时间: " + (endTime - startTime) + "ms");
            return result;
        } catch (Exception e) {
            System.out.println("[AOP-Around] 查询方法执行异常: " + e.getMessage());
            throw e;
        }
    }
}

// 6. 用户实体类
class User {
    private Long id;
    private String name;
    private String email;
    
    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }
    
    // Getter和Setter方法
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    @Override
    public String toString() {
        return "User{id=" + id + ", name='" + name + "', email='" + email + "'}";
    }
}
```

**Spring框架架构图：**

```
Spring Framework
├── Core Container (核心容器)
│   ├── Core (核心)
│   ├── Beans (Bean管理)
│   ├── Context (上下文)
│   └── Expression Language (表达式语言)
├── AOP (面向切面编程)
│   ├── AOP (Spring AOP)
│   ├── Aspects (AspectJ集成)
│   └── Instrumentation (字节码增强)
├── Data Access/Integration (数据访问/集成)
│   ├── JDBC (JDBC抽象)
│   ├── ORM (ORM集成)
│   ├── OXM (对象XML映射)
│   ├── JMS (消息服务)
│   └── Transaction (事务管理)
├── Web (Web层)
│   ├── Web (Web基础)
│   ├── Web-MVC (MVC框架)
│   ├── Web-Socket (WebSocket)
│   └── Web-Portlet (Portlet)
└── Test (测试)
    └── Test (测试支持)
```

**Spring核心优势：**

1. **轻量级**：非侵入式框架，最小化依赖
2. **松耦合**：通过IoC实现组件间的松耦合
3. **声明式**：通过配置而非编程实现功能
4. **可测试**：便于单元测试和集成测试
5. **模块化**：可选择性使用框架的不同部分
6. **集成性**：与其他框架良好集成

**最佳实践：**

1. **合理使用注解**：@Component、@Service、@Repository、@Controller
2. **配置分离**：将配置信息外部化
3. **接口编程**：面向接口而非实现编程
4. **单一职责**：每个Bean承担单一职责
5. **生命周期管理**：合理使用@PostConstruct和@PreDestroy
6. **事务边界**：在服务层定义事务边界
7. **AOP使用**：将横切关注点模块化

**常见问题和解决方案：**

1. **循环依赖**：使用@Lazy或重构设计
2. **Bean作用域**：理解singleton、prototype等作用域
3. **事务失效**：注意自调用和代理机制
4. **AOP不生效**：检查切入点表达式和代理配置
5. **性能优化**：合理使用Bean缓存和延迟加载

**Q19: Spring Bean的生命周期？**

**答案：**
1. 实例化Bean
2. 设置属性值
3. 调用BeanNameAware、BeanFactoryAware等接口方法
4. 调用BeanPostProcessor的前置处理方法
5. 调用InitializingBean的afterPropertiesSet方法
6. 调用自定义的init方法
7. 调用BeanPostProcessor的后置处理方法
8. Bean可以使用
9. 调用DisposableBean的destroy方法
10. 调用自定义的destroy方法

**Q20: Spring的事务传播机制有哪些？**

**答案：**
- **REQUIRED**：默认，有事务加入，无事务创建
- **REQUIRES_NEW**：总是创建新事务
- **SUPPORTS**：有事务加入，无事务以非事务运行
- **NOT_SUPPORTED**：以非事务方式运行
- **MANDATORY**：必须在事务中运行
- **NEVER**：不能在事务中运行
- **NESTED**：嵌套事务

### 3.2 Spring Boot类

**Q21: Spring Boot的自动配置原理？**

**答案：**
- **@EnableAutoConfiguration**：启用自动配置
- **spring.factories**：配置自动配置类列表
- **条件注解**：@ConditionalOnClass、@ConditionalOnProperty等
- **配置属性**：@ConfigurationProperties绑定配置

**Q22: Spring Boot Starter的作用？**

**答案：**
- **依赖管理**：自动引入相关依赖
- **自动配置**：提供默认配置
- **简化开发**：减少配置工作
- **版本兼容**：保证依赖版本兼容性

### 3.3 MyBatis类

**Q23: MyBatis的核心组件有哪些？**

**答案：**
- **SqlSessionFactory**：会话工厂
- **SqlSession**：执行SQL的会话
- **Mapper**：映射器接口
- **Configuration**：配置信息

**Q24: MyBatis的缓存机制？**

**答案：**
- **一级缓存**：SqlSession级别，默认开启
- **二级缓存**：Mapper级别，需要配置开启
- **缓存失效**：增删改操作会清空缓存
- **自定义缓存**：可以集成Redis等外部缓存

---

## 4. 分布式系统面试题

### 4.1 微服务类

**Q25: 微服务架构的优缺点？**

**答案：**
- **优点**：独立部署、技术栈灵活、故障隔离、团队自治
- **缺点**：系统复杂性增加、网络延迟、数据一致性、运维复杂

**Q26: 服务注册与发现的原理？**

**答案：**
- **服务注册**：服务启动时向注册中心注册自己的信息
- **服务发现**：客户端从注册中心获取服务列表
- **健康检查**：定期检查服务健康状态
- **负载均衡**：在多个服务实例间分发请求

**Q27: 分布式事务的解决方案？**

**答案：**
- **2PC**：两阶段提交，强一致性但性能差
- **TCC**：Try-Confirm-Cancel，补偿型事务
- **Saga**：长事务拆分，事件驱动
- **最终一致性**：通过消息队列实现

### 4.2 缓存类

**Q28: Redis的数据类型和使用场景？**

**答案：**
- **String**：缓存、计数器、分布式锁
- **Hash**：对象存储、购物车
- **List**：消息队列、最新列表
- **Set**：去重、交集并集
- **ZSet**：排行榜、延迟队列

**Q29: 缓存穿透、缓存击穿、缓存雪崩的解决方案？**

**答案：**
- **缓存穿透**：布隆过滤器、空值缓存
- **缓存击穿**：分布式锁、热点数据永不过期
- **缓存雪崩**：过期时间随机化、多级缓存

### 4.3 消息队列类

**Q30: 消息队列的作用和选型考虑？**

**答案：**
- **作用**：解耦、异步、削峰填谷
- **选型因素**：性能、可靠性、功能特性、运维复杂度
- **常见产品**：RabbitMQ（功能丰富）、Kafka（高吞吐）、RocketMQ（事务消息）

---

## 5. 架构设计面试题

### 5.1 系统设计类

**Q31: 如何设计一个高并发的秒杀系统？**

**答案要点：**
- **前端**：静态化、CDN、防重复提交
- **网关**：限流、防刷、负载均衡
- **业务**：库存预扣、异步处理、消息队列
- **存储**：Redis缓存、数据库优化、读写分离

**Q32: 如何设计一个分布式ID生成器？**

**答案要点：**
- **UUID**：简单但无序，不适合数据库主键
- **数据库自增**：简单但性能瓶颈
- **雪花算法**：有序、高性能，需要时钟同步
- **号段模式**：批量获取，减少数据库访问

**Q33: 如何保证系统的高可用性？**

**答案要点：**
- **冗余设计**：多副本、多机房
- **故障隔离**：熔断器、舱壁模式
- **快速恢复**：自动故障转移、健康检查
- **降级策略**：核心功能保障

### 5.2 性能优化类

**Q34: 系统性能优化的思路和方法？**

**答案要点：**
- **前端优化**：CDN、压缩、缓存
- **应用优化**：代码优化、JVM调优、连接池
- **数据库优化**：索引、SQL优化、分库分表
- **架构优化**：缓存、异步、负载均衡

**Q35: 数据库查询慢的排查和优化方法？**

**答案要点：**
- **排查方法**：慢查询日志、执行计划分析
- **索引优化**：创建合适索引、避免索引失效
- **SQL优化**：避免全表扫描、优化JOIN
- **架构优化**：读写分离、分库分表

---

## 6. 综合实战面试题

### 6.1 场景设计类

**Q36: 设计一个类似微信的即时通讯系统**

**答案要点：**
- **架构设计**：网关、用户服务、消息服务、推送服务
- **技术选型**：WebSocket、Netty、Redis、消息队列
- **数据存储**：用户信息、好友关系、聊天记录
- **性能考虑**：连接管理、消息路由、离线消息

**Q37: 设计一个电商系统的架构**

**答案要点：**
- **服务拆分**：用户、商品、订单、支付、库存
- **数据一致性**：分布式事务、最终一致性
- **高并发处理**：缓存、限流、异步处理
- **搜索推荐**：Elasticsearch、推荐算法

### 6.2 故障处理类

**Q38: 线上系统突然出现大量超时，如何排查？**

**答案要点：**
1. **监控检查**：CPU、内存、网络、数据库连接
2. **日志分析**：错误日志、慢查询日志
3. **链路追踪**：定位具体的慢服务
4. **应急处理**：限流、降级、扩容
5. **根因分析**：代码问题、资源不足、外部依赖

**Q39: 如何处理数据库连接池耗尽的问题？**

**答案要点：**
- **立即处理**：重启应用、增加连接数
- **排查原因**：慢SQL、连接泄漏、并发过高
- **长期优化**：连接池配置、SQL优化、读写分离
- **监控预警**：连接数监控、自动告警

---

## 7. 面试技巧和准备建议

### 7.1 面试准备策略

**技术准备：**
1. **基础扎实**：Java核心知识点要熟练掌握
2. **项目经验**：准备2-3个有代表性的项目
3. **技术广度**：了解主流技术栈的优缺点
4. **深度挖掘**：对使用过的技术要深入理解

**简历优化：**
1. **突出亮点**：技术栈、项目成果、解决的问题
2. **量化指标**：性能提升、用户量、并发量等
3. **技术匹配**：根据职位要求调整技术重点
4. **逻辑清晰**：时间线清楚、职责明确

### 7.2 面试回答技巧

**回答结构：**
1. **总分总**：先总述，再分点，最后总结
2. **举例说明**：理论结合实际项目经验
3. **对比分析**：说明不同方案的优缺点
4. **扩展思考**：从问题延伸到相关知识点

**常见陷阱：**
1. **避免绝对化**：技术选择要考虑场景
2. **承认不足**：不懂的问题要诚实回答
3. **保持谦逊**：展示学习能力和成长潜力
4. **逻辑清晰**：回答要有条理，不要跳跃

### 7.3 不同级别面试重点

**初级工程师（1-3年）：**
- 重点：Java基础、常用框架、基本算法
- 项目：能独立完成模块开发
- 能力：代码规范、基本调试能力

**中级工程师（3-5年）：**
- 重点：框架原理、性能优化、系统设计
- 项目：能设计模块架构、解决复杂问题
- 能力：技术选型、团队协作

**高级工程师（5-8年）：**
- 重点：架构设计、技术决策、团队管理
- 项目：能设计系统架构、技术攻关
- 能力：技术规划、人才培养

**架构师（8年+）：**
- 重点：业务理解、技术战略、团队建设
- 项目：能规划技术路线、推动技术变革
- 能力：跨部门协作、技术影响力

### 7.4 面试后的总结

**及时复盘：**
1. **记录问题**：面试中遇到的技术问题
2. **查漏补缺**：不会的知识点要及时学习
3. **总结经验**：回答好的和不好的地方
4. **持续改进**：根据反馈调整准备策略

**持续学习：**
1. **技术跟进**：关注新技术发展趋势
2. **项目实践**：在工作中应用新学到的知识
3. **社区参与**：参加技术会议、开源项目
4. **知识分享**：通过分享加深理解

---

## 总结

这份面试题集合涵盖了从Java基础到架构设计的各个层面，每个问题都提供了详细的答案要点。在准备面试时，建议：

1. **系统学习**：按照学习路线图逐步深入
2. **实践验证**：通过项目实践验证理论知识
3. **持续总结**：定期回顾和总结学习成果
4. **模拟练习**：多进行模拟面试练习

记住，面试不仅是技术能力的展示，更是综合素质的体现。保持学习的热情，持续提升自己的技术能力和解决问题的能力，相信你一定能够成功！