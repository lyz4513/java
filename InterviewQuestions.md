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

**详细答案：**

微服务架构是一种将单一应用程序开发为一套小服务的方法，每个服务运行在自己的进程中，并使用轻量级机制通信。

**微服务架构详细对比：**

| 对比维度 | 单体架构 | 微服务架构 | 说明 |
|----------|----------|------------|------|
| 部署方式 | 整体部署 | 独立部署 | 微服务可单独发布上线 |
| 技术栈 | 统一技术栈 | 技术栈多样化 | 不同服务可选择最适合的技术 |
| 团队组织 | 功能团队 | 业务团队 | 按业务领域划分团队 |
| 数据管理 | 共享数据库 | 数据库分离 | 每个服务管理自己的数据 |
| 故障影响 | 全局影响 | 局部影响 | 单个服务故障不影响整体 |
| 系统复杂度 | 相对简单 | 分布式复杂性 | 网络、一致性、监控等挑战 |

**微服务架构优点详解：**

**1. 独立部署和发布**
- **快速迭代**：服务可独立开发、测试、部署
- **降低风险**：单个服务的变更不影响其他服务
- **灵活发布**：支持蓝绿部署、金丝雀发布等策略

**2. 技术栈灵活性**
- **技术选型自由**：不同服务可选择最适合的编程语言和框架
- **技术演进**：可逐步引入新技术，不需要全盘重写
- **团队专业化**：团队可专注于特定技术领域

**3. 故障隔离**
- **局部故障**：单个服务故障不会导致整个系统崩溃
- **容错设计**：通过熔断、降级等机制提高系统稳定性
- **快速恢复**：问题定位和修复更加精准

**4. 团队自治**
- **业务对齐**：团队按业务领域组织，职责清晰
- **决策效率**：减少跨团队协调，提高开发效率
- **责任明确**：每个团队对自己的服务负全责

**5. 可扩展性**
- **按需扩展**：可针对性能瓶颈服务进行扩展
- **资源优化**：不同服务可配置不同的资源规格
- **弹性伸缩**：支持自动扩缩容

**微服务架构缺点详解：**

**1. 系统复杂性增加**
- **分布式挑战**：网络分区、时钟同步、分布式锁等问题
- **服务治理**：需要服务注册发现、配置管理、API网关等
- **调试困难**：跨服务调用链路复杂，问题排查困难

**2. 网络延迟和通信开销**
- **网络调用**：服务间通信增加网络延迟
- **序列化开销**：数据序列化和反序列化消耗资源
- **网络故障**：需要处理网络超时、重试等问题

**3. 数据一致性挑战**
- **分布式事务**：跨服务事务处理复杂
- **数据同步**：服务间数据同步和一致性保证困难
- **最终一致性**：需要接受数据的最终一致性模型

**4. 运维复杂性**
- **服务数量**：需要管理大量的服务实例
- **监控告警**：需要完善的监控和日志系统
- **部署协调**：服务间依赖关系复杂，部署需要协调

**5. 测试复杂性**
- **集成测试**：跨服务的集成测试复杂
- **环境管理**：需要维护多套测试环境
- **数据准备**：测试数据准备和清理困难

**代码示例：**
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.web.bind.annotation.*;
import org.springframework.beans.factory.annotation.Autowired;
import feign.FeignClient;
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import io.github.resilience4j.retry.annotation.Retry;
import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;

// 1. 用户服务 - 微服务示例
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}

@RestController
@RequestMapping("/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @Autowired
    private OrderServiceClient orderServiceClient;
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        return ResponseEntity.ok(user);
    }
    
    @GetMapping("/{id}/orders")
    @CircuitBreaker(name = "order-service", fallbackMethod = "getOrdersFallback")
    @Retry(name = "order-service")
    public ResponseEntity<List<Order>> getUserOrders(@PathVariable Long id) {
        // 跨服务调用
        List<Order> orders = orderServiceClient.getOrdersByUserId(id);
        return ResponseEntity.ok(orders);
    }
    
    // 熔断降级方法
    public ResponseEntity<List<Order>> getOrdersFallback(Long id, Exception ex) {
        System.out.println("订单服务不可用，返回默认数据: " + ex.getMessage());
        return ResponseEntity.ok(Collections.emptyList());
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody CreateUserRequest request) {
        User user = userService.createUser(request);
        
        // 发布用户创建事件
        userService.publishUserCreatedEvent(user);
        
        return ResponseEntity.ok(user);
    }
}

// 2. Feign客户端 - 服务间通信
@FeignClient(name = "order-service", fallback = OrderServiceFallback.class)
public interface OrderServiceClient {
    
    @GetMapping("/orders/user/{userId}")
    List<Order> getOrdersByUserId(@PathVariable("userId") Long userId);
    
    @PostMapping("/orders")
    Order createOrder(@RequestBody CreateOrderRequest request);
}

// 3. 熔断降级实现
@Component
public class OrderServiceFallback implements OrderServiceClient {
    
    @Override
    public List<Order> getOrdersByUserId(Long userId) {
        System.out.println("订单服务降级: 返回空订单列表");
        return Collections.emptyList();
    }
    
    @Override
    public Order createOrder(CreateOrderRequest request) {
        System.out.println("订单服务降级: 创建订单失败");
        throw new ServiceUnavailableException("订单服务暂时不可用");
    }
}

// 4. 事件发布 - 服务解耦
@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private EventPublisher eventPublisher;
    
    public User createUser(CreateUserRequest request) {
        User user = new User();
        user.setName(request.getName());
        user.setEmail(request.getEmail());
        user.setCreatedAt(LocalDateTime.now());
        
        user = userRepository.save(user);
        
        return user;
    }
    
    public void publishUserCreatedEvent(User user) {
        UserCreatedEvent event = new UserCreatedEvent();
        event.setUserId(user.getId());
        event.setUserName(user.getName());
        event.setUserEmail(user.getEmail());
        event.setTimestamp(LocalDateTime.now());
        
        eventPublisher.publish("user.created", event);
    }
    
    public User findById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException("用户不存在: " + id));
    }
}

// 5. API网关配置 - 统一入口
@SpringBootApplication
public class GatewayApplication {
    
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            // 用户服务路由
            .route("user-service", r -> r
                .path("/api/users/**")
                .filters(f -> f
                    .stripPrefix(1)
                    .addRequestHeader("X-Gateway", "Spring-Cloud-Gateway")
                    .circuitBreaker(config -> config
                        .setName("user-service-cb")
                        .setFallbackUri("forward:/fallback/users")))
                .uri("lb://user-service"))
            
            // 订单服务路由
            .route("order-service", r -> r
                .path("/api/orders/**")
                .filters(f -> f
                    .stripPrefix(1)
                    .addRequestHeader("X-Gateway", "Spring-Cloud-Gateway")
                    .circuitBreaker(config -> config
                        .setName("order-service-cb")
                        .setFallbackUri("forward:/fallback/orders")))
                .uri("lb://order-service"))
            
            .build();
    }
    
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}

// 6. 配置中心配置
@RestController
public class ConfigController {
    
    @Value("${app.name:default-app}")
    private String appName;
    
    @Value("${app.version:1.0.0}")
    private String appVersion;
    
    @GetMapping("/config")
    public Map<String, String> getConfig() {
        Map<String, String> config = new HashMap<>();
        config.put("appName", appName);
        config.put("appVersion", appVersion);
        config.put("timestamp", LocalDateTime.now().toString());
        return config;
    }
}

// 7. 健康检查
@Component
public class CustomHealthIndicator implements HealthIndicator {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public Health health() {
        try {
            // 检查数据库连接
            long userCount = userRepository.count();
            
            return Health.up()
                .withDetail("database", "available")
                .withDetail("userCount", userCount)
                .withDetail("timestamp", LocalDateTime.now())
                .build();
        } catch (Exception e) {
            return Health.down()
                .withDetail("database", "unavailable")
                .withDetail("error", e.getMessage())
                .build();
        }
    }
}

// 8. 分布式追踪
@RestController
public class TraceController {
    
    private static final Logger logger = LoggerFactory.getLogger(TraceController.class);
    
    @Autowired
    private OrderServiceClient orderServiceClient;
    
    @GetMapping("/trace/{userId}")
    public ResponseEntity<Map<String, Object>> traceUserOrders(@PathVariable Long userId) {
        String traceId = MDC.get("traceId");
        logger.info("开始处理用户订单查询, userId: {}, traceId: {}", userId, traceId);
        
        try {
            // 调用其他服务
            List<Order> orders = orderServiceClient.getOrdersByUserId(userId);
            
            Map<String, Object> result = new HashMap<>();
            result.put("userId", userId);
            result.put("orders", orders);
            result.put("traceId", traceId);
            result.put("timestamp", LocalDateTime.now());
            
            logger.info("用户订单查询完成, userId: {}, orderCount: {}, traceId: {}", 
                       userId, orders.size(), traceId);
            
            return ResponseEntity.ok(result);
        } catch (Exception e) {
            logger.error("用户订单查询失败, userId: {}, traceId: {}, error: {}", 
                        userId, traceId, e.getMessage());
            throw e;
        }
    }
}
```

**微服务架构最佳实践：**

**1. 服务拆分原则**
- **业务边界**：按业务领域拆分，遵循DDD原则
- **数据独立**：每个服务管理自己的数据
- **团队规模**：遵循"两个披萨"团队原则
- **变更频率**：将变更频率相似的功能放在一起

**2. 通信机制**
- **同步通信**：HTTP/REST、gRPC用于实时查询
- **异步通信**：消息队列用于事件通知
- **API设计**：RESTful API，版本管理
- **超时重试**：设置合理的超时和重试策略

**3. 数据管理**
- **数据库分离**：每个服务独立的数据库
- **事务处理**：使用Saga模式处理分布式事务
- **数据同步**：通过事件驱动实现数据同步
- **缓存策略**：合理使用分布式缓存

**4. 运维监控**
- **服务发现**：使用Eureka、Consul等注册中心
- **配置管理**：集中化配置管理
- **日志聚合**：ELK Stack收集和分析日志
- **链路追踪**：Zipkin、Jaeger追踪调用链路
- **监控告警**：Prometheus + Grafana监控

**5. 容错设计**
- **熔断器**：防止故障传播
- **限流降级**：保护系统稳定性
- **超时控制**：避免资源耗尽
- **重试机制**：处理临时故障

**适用场景：**

**适合微服务的场景：**
- 大型复杂系统
- 多团队并行开发
- 不同模块技术栈需求差异大
- 需要独立扩展不同功能模块
- 对系统可用性要求高

**不适合微服务的场景：**
- 小型简单应用
- 团队规模小
- 业务边界不清晰
- 对一致性要求极高
- 运维能力不足

**迁移策略：**

1. **绞杀者模式**：逐步替换单体应用的功能
2. **数据库分离**：先分离数据库，再分离应用
3. **API网关**：统一入口，逐步路由到微服务
4. **事件驱动**：通过事件实现服务间解耦
5. **监控先行**：建立完善的监控体系

**Q26: 服务注册与发现的原理？**

**答案：**
- **服务注册**：服务启动时向注册中心注册自己的信息
- **服务发现**：客户端从注册中心获取服务列表
- **健康检查**：定期检查服务健康状态
- **负载均衡**：在多个服务实例间分发请求

**Q27: 分布式事务的解决方案？**

**详细答案：**

分布式事务是指事务的参与者、支持事务的服务器、资源服务器以及事务管理器分别位于不同的分布式系统的不同节点之上。

**分布式事务解决方案对比：**

| 解决方案 | 一致性 | 可用性 | 性能 | 复杂度 | 适用场景 |
|----------|--------|--------|------|--------|----------|
| 2PC/3PC | 强一致性 | 较低 | 较低 | 中等 | 对一致性要求极高的场景 |
| TCC | 最终一致性 | 高 | 中等 | 高 | 业务逻辑复杂的场景 |
| Saga | 最终一致性 | 高 | 高 | 中等 | 长流程业务场景 |
| 本地消息表 | 最终一致性 | 高 | 高 | 低 | 简单的异步处理场景 |
| MQ事务消息 | 最终一致性 | 高 | 高 | 低 | 消息驱动的场景 |

**1. 两阶段提交（2PC）详解：**

**原理：**
- **准备阶段**：协调者询问所有参与者是否可以提交事务
- **提交阶段**：如果所有参与者都同意，则提交；否则回滚

**优点：**
- 保证强一致性
- 实现相对简单

**缺点：**
- 同步阻塞，性能较差
- 单点故障问题
- 数据不一致风险

**2. 三阶段提交（3PC）详解：**

**改进点：**
- 增加了CanCommit阶段
- 引入超时机制
- 减少阻塞时间

**3. TCC（Try-Confirm-Cancel）详解：**

**三个阶段：**
- **Try**：尝试执行业务，预留资源
- **Confirm**：确认执行业务，使用预留资源
- **Cancel**：取消执行业务，释放预留资源

**特点：**
- 业务侵入性强
- 需要实现三个接口
- 保证最终一致性

**4. Saga模式详解：**

**两种实现方式：**
- **编排式（Orchestration）**：中央协调器控制流程
- **编舞式（Choreography）**：各服务通过事件协调

**补偿机制：**
- 每个事务步骤都有对应的补偿操作
- 失败时执行补偿操作回滚

**代码示例：**
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.scheduling.annotation.Async;
import java.time.LocalDateTime;
import java.util.concurrent.CompletableFuture;
import java.util.UUID;

// 1. TCC模式实现示例
@SpringBootApplication
public class DistributedTransactionApplication {
    public static void main(String[] args) {
        SpringApplication.run(DistributedTransactionApplication.class, args);
    }
}

// TCC接口定义
public interface TccTransaction {
    /**
     * Try阶段：尝试执行业务，预留资源
     */
    boolean tryExecute(String transactionId, Object... params);
    
    /**
     * Confirm阶段：确认执行业务
     */
    boolean confirmExecute(String transactionId);
    
    /**
     * Cancel阶段：取消执行业务，释放资源
     */
    boolean cancelExecute(String transactionId);
}

// 账户服务TCC实现
@Service
public class AccountTccService implements TccTransaction {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    @Override
    public boolean tryExecute(String transactionId, Object... params) {
        Long accountId = (Long) params[0];
        BigDecimal amount = (BigDecimal) params[1];
        
        try {
            // 冻结账户金额
            String sql = "UPDATE account SET frozen_amount = frozen_amount + ?, " +
                        "version = version + 1 WHERE id = ? AND balance >= ?";
            int rows = jdbcTemplate.update(sql, amount, accountId, amount);
            
            if (rows > 0) {
                // 记录TCC事务日志
                recordTccLog(transactionId, "ACCOUNT_FREEZE", "TRY", 
                           "SUCCESS", accountId, amount);
                return true;
            }
            
            recordTccLog(transactionId, "ACCOUNT_FREEZE", "TRY", 
                        "FAILED", accountId, amount);
            return false;
            
        } catch (Exception e) {
            System.err.println("账户冻结失败: " + e.getMessage());
            recordTccLog(transactionId, "ACCOUNT_FREEZE", "TRY", 
                        "ERROR", accountId, amount);
            return false;
        }
    }
    
    @Override
    public boolean confirmExecute(String transactionId) {
        try {
            TccLog tccLog = getTccLog(transactionId, "ACCOUNT_FREEZE");
            if (tccLog == null) {
                return false;
            }
            
            // 扣减账户余额，释放冻结金额
            String sql = "UPDATE account SET balance = balance - ?, " +
                        "frozen_amount = frozen_amount - ? WHERE id = ?";
            jdbcTemplate.update(sql, tccLog.getAmount(), 
                              tccLog.getAmount(), tccLog.getAccountId());
            
            // 更新TCC日志状态
            updateTccLogStatus(transactionId, "ACCOUNT_FREEZE", "CONFIRM", "SUCCESS");
            
            System.out.println("账户扣款确认成功: " + tccLog.getAmount());
            return true;
            
        } catch (Exception e) {
            System.err.println("账户扣款确认失败: " + e.getMessage());
            updateTccLogStatus(transactionId, "ACCOUNT_FREEZE", "CONFIRM", "ERROR");
            return false;
        }
    }
    
    @Override
    public boolean cancelExecute(String transactionId) {
        try {
            TccLog tccLog = getTccLog(transactionId, "ACCOUNT_FREEZE");
            if (tccLog == null) {
                return true; // 幂等性处理
            }
            
            // 释放冻结金额
            String sql = "UPDATE account SET frozen_amount = frozen_amount - ? WHERE id = ?";
            jdbcTemplate.update(sql, tccLog.getAmount(), tccLog.getAccountId());
            
            // 更新TCC日志状态
            updateTccLogStatus(transactionId, "ACCOUNT_FREEZE", "CANCEL", "SUCCESS");
            
            System.out.println("账户冻结取消成功: " + tccLog.getAmount());
            return true;
            
        } catch (Exception e) {
            System.err.println("账户冻结取消失败: " + e.getMessage());
            updateTccLogStatus(transactionId, "ACCOUNT_FREEZE", "CANCEL", "ERROR");
            return false;
        }
    }
    
    private void recordTccLog(String transactionId, String operation, 
                             String phase, String status, 
                             Long accountId, BigDecimal amount) {
        String sql = "INSERT INTO tcc_log (transaction_id, operation, phase, status, " +
                    "account_id, amount, create_time) VALUES (?, ?, ?, ?, ?, ?, ?)";
        jdbcTemplate.update(sql, transactionId, operation, phase, status, 
                          accountId, amount, LocalDateTime.now());
    }
    
    private TccLog getTccLog(String transactionId, String operation) {
        String sql = "SELECT * FROM tcc_log WHERE transaction_id = ? AND operation = ?";
        List<TccLog> logs = jdbcTemplate.query(sql, 
            (rs, rowNum) -> {
                TccLog log = new TccLog();
                log.setTransactionId(rs.getString("transaction_id"));
                log.setOperation(rs.getString("operation"));
                log.setAccountId(rs.getLong("account_id"));
                log.setAmount(rs.getBigDecimal("amount"));
                return log;
            }, transactionId, operation);
        return logs.isEmpty() ? null : logs.get(0);
    }
    
    private void updateTccLogStatus(String transactionId, String operation, 
                                   String phase, String status) {
        String sql = "UPDATE tcc_log SET phase = ?, status = ?, update_time = ? " +
                    "WHERE transaction_id = ? AND operation = ?";
        jdbcTemplate.update(sql, phase, status, LocalDateTime.now(), 
                          transactionId, operation);
    }
}

// 2. Saga模式实现示例
@Service
public class OrderSagaService {
    
    @Autowired
    private AccountTccService accountService;
    
    @Autowired
    private InventoryService inventoryService;
    
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;
    
    /**
     * 编排式Saga：创建订单流程
     */
    public void createOrderSaga(CreateOrderRequest request) {
        String sagaId = UUID.randomUUID().toString();
        
        try {
            // 步骤1：创建订单
            Long orderId = orderService.createOrder(request);
            publishSagaEvent(sagaId, "ORDER_CREATED", orderId);
            
            // 步骤2：扣减库存
            boolean inventoryResult = inventoryService.reduceInventory(
                request.getProductId(), request.getQuantity());
            
            if (!inventoryResult) {
                // 补偿：取消订单
                orderService.cancelOrder(orderId);
                publishSagaEvent(sagaId, "ORDER_CANCELLED", orderId);
                throw new SagaException("库存不足，订单创建失败");
            }
            
            publishSagaEvent(sagaId, "INVENTORY_REDUCED", request.getProductId());
            
            // 步骤3：扣减账户余额
            boolean paymentResult = accountService.tryExecute(sagaId, 
                request.getAccountId(), request.getAmount());
            
            if (!paymentResult) {
                // 补偿：恢复库存
                inventoryService.restoreInventory(
                    request.getProductId(), request.getQuantity());
                publishSagaEvent(sagaId, "INVENTORY_RESTORED", request.getProductId());
                
                // 补偿：取消订单
                orderService.cancelOrder(orderId);
                publishSagaEvent(sagaId, "ORDER_CANCELLED", orderId);
                
                throw new SagaException("账户余额不足，订单创建失败");
            }
            
            // 确认支付
            accountService.confirmExecute(sagaId);
            publishSagaEvent(sagaId, "PAYMENT_CONFIRMED", request.getAccountId());
            
            // 步骤4：完成订单
            orderService.completeOrder(orderId);
            publishSagaEvent(sagaId, "ORDER_COMPLETED", orderId);
            
            System.out.println("订单创建成功: " + orderId);
            
        } catch (Exception e) {
            System.err.println("Saga执行失败: " + e.getMessage());
            publishSagaEvent(sagaId, "SAGA_FAILED", e.getMessage());
        }
    }
    
    private void publishSagaEvent(String sagaId, String eventType, Object data) {
        SagaEvent event = new SagaEvent();
        event.setSagaId(sagaId);
        event.setEventType(eventType);
        event.setData(data);
        event.setTimestamp(LocalDateTime.now());
        
        kafkaTemplate.send("saga-events", event);
        System.out.println("发布Saga事件: " + eventType + ", sagaId: " + sagaId);
    }
}

// 3. 本地消息表实现示例
@Service
public class LocalMessageService {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;
    
    /**
     * 本地事务 + 消息表
     */
    @Transactional
    public void processOrderWithLocalMessage(CreateOrderRequest request) {
        try {
            // 1. 执行本地业务操作
            Long orderId = createOrderInDatabase(request);
            
            // 2. 在同一个本地事务中插入消息表记录
            String messageId = UUID.randomUUID().toString();
            insertLocalMessage(messageId, "ORDER_CREATED", orderId, request);
            
            System.out.println("订单创建成功，消息已记录: " + orderId);
            
        } catch (Exception e) {
            System.err.println("订单创建失败: " + e.getMessage());
            throw e;
        }
    }
    
    private Long createOrderInDatabase(CreateOrderRequest request) {
        String sql = "INSERT INTO orders (user_id, product_id, quantity, amount, status, create_time) " +
                    "VALUES (?, ?, ?, ?, ?, ?)";
        
        KeyHolder keyHolder = new GeneratedKeyHolder();
        jdbcTemplate.update(connection -> {
            PreparedStatement ps = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
            ps.setLong(1, request.getUserId());
            ps.setLong(2, request.getProductId());
            ps.setInt(3, request.getQuantity());
            ps.setBigDecimal(4, request.getAmount());
            ps.setString(5, "CREATED");
            ps.setTimestamp(6, Timestamp.valueOf(LocalDateTime.now()));
            return ps;
        }, keyHolder);
        
        return keyHolder.getKey().longValue();
    }
    
    private void insertLocalMessage(String messageId, String eventType, 
                                   Long orderId, CreateOrderRequest request) {
        String sql = "INSERT INTO local_message (message_id, event_type, order_id, " +
                    "payload, status, create_time, retry_count) VALUES (?, ?, ?, ?, ?, ?, ?)";
        
        String payload = JSON.toJSONString(request);
        jdbcTemplate.update(sql, messageId, eventType, orderId, payload, 
                          "PENDING", LocalDateTime.now(), 0);
    }
    
    /**
     * 定时任务：发送未发送的消息
     */
    @Scheduled(fixedDelay = 5000) // 每5秒执行一次
    public void sendPendingMessages() {
        String sql = "SELECT * FROM local_message WHERE status = 'PENDING' " +
                    "AND retry_count < 3 ORDER BY create_time LIMIT 100";
        
        List<LocalMessage> messages = jdbcTemplate.query(sql, 
            (rs, rowNum) -> {
                LocalMessage msg = new LocalMessage();
                msg.setMessageId(rs.getString("message_id"));
                msg.setEventType(rs.getString("event_type"));
                msg.setOrderId(rs.getLong("order_id"));
                msg.setPayload(rs.getString("payload"));
                msg.setRetryCount(rs.getInt("retry_count"));
                return msg;
            });
        
        for (LocalMessage message : messages) {
            try {
                // 发送消息到MQ
                kafkaTemplate.send("order-events", message);
                
                // 更新消息状态为已发送
                updateMessageStatus(message.getMessageId(), "SENT");
                
                System.out.println("消息发送成功: " + message.getMessageId());
                
            } catch (Exception e) {
                // 增加重试次数
                incrementRetryCount(message.getMessageId());
                System.err.println("消息发送失败: " + message.getMessageId() + 
                                 ", 错误: " + e.getMessage());
            }
        }
    }
    
    private void updateMessageStatus(String messageId, String status) {
        String sql = "UPDATE local_message SET status = ?, update_time = ? WHERE message_id = ?";
        jdbcTemplate.update(sql, status, LocalDateTime.now(), messageId);
    }
    
    private void incrementRetryCount(String messageId) {
        String sql = "UPDATE local_message SET retry_count = retry_count + 1, " +
                    "update_time = ? WHERE message_id = ?";
        jdbcTemplate.update(sql, LocalDateTime.now(), messageId);
    }
}
```

**分布式事务选择指南：**

**1. 强一致性需求**
- 选择2PC/3PC
- 适用于金融、支付等对一致性要求极高的场景
- 需要容忍性能损失

**2. 最终一致性可接受**
- 选择TCC、Saga、本地消息表或MQ事务消息
- 适用于大多数业务场景
- 性能和可用性更好

**3. 业务复杂度考虑**
- 简单场景：本地消息表、MQ事务消息
- 复杂场景：TCC、Saga
- 长流程：Saga模式

**4. 技术栈考虑**
- Spring Cloud：Seata（支持TCC、Saga、AT模式）
- 消息队列：RocketMQ事务消息、Kafka
- 数据库：支持XA事务的数据库

**最佳实践：**

1. **幂等性设计**：所有操作都要支持幂等
2. **超时处理**：设置合理的超时时间
3. **重试机制**：实现指数退避重试
4. **监控告警**：完善的事务状态监控
5. **补偿机制**：设计完善的补偿逻辑
6. **状态机管理**：清晰的事务状态流转
7. **日志记录**：详细的事务执行日志

### 4.2 缓存类

**Q28: Redis的数据类型和使用场景？**

**详细答案：**

Redis支持多种数据类型，每种类型都有其特定的使用场景和优化策略。

**Redis数据类型详细对比：**

| 数据类型 | 底层实现 | 内存占用 | 时间复杂度 | 主要使用场景 |
|----------|----------|----------|------------|-------------|
| String | SDS(Simple Dynamic String) | 较高 | O(1) | 缓存、计数器、分布式锁 |
| Hash | ziplist/hashtable | 中等 | O(1) | 对象存储、购物车 |
| List | ziplist/linkedlist | 中等 | O(1)/O(N) | 消息队列、最新列表 |
| Set | intset/hashtable | 中等 | O(1) | 去重、交集并集 |
| ZSet | ziplist/skiplist+hashtable | 较高 | O(log N) | 排行榜、延迟队列 |
| Stream | radix tree | 较高 | O(1)/O(log N) | 消息流、事件溯源 |
| Bitmap | String | 极低 | O(1) | 用户签到、布隆过滤器 |
| HyperLogLog | String | 极低 | O(1) | 基数统计、UV统计 |

**1. String类型详解：**

**特点：**
- 最基本的数据类型
- 二进制安全，可存储任何数据
- 最大512MB

**常用命令：**
- SET/GET：基本存取
- INCR/DECR：原子性递增递减
- SETEX：设置过期时间
- SETNX：不存在时设置

**使用场景：**
- **缓存**：存储序列化的对象
- **计数器**：网站访问量、点赞数
- **分布式锁**：利用SETNX实现
- **限流**：结合过期时间实现

**2. Hash类型详解：**

**特点：**
- 键值对集合
- 适合存储对象
- 节省内存（小hash使用ziplist）

**常用命令：**
- HSET/HGET：设置/获取字段
- HMSET/HMGET：批量操作
- HINCRBY：字段递增
- HGETALL：获取所有字段

**使用场景：**
- **对象存储**：用户信息、商品信息
- **购物车**：用户ID为key，商品ID为field
- **配置管理**：应用配置信息

**3. List类型详解：**

**特点：**
- 有序列表
- 支持双端操作
- 可重复元素

**常用命令：**
- LPUSH/RPUSH：左/右端插入
- LPOP/RPOP：左/右端弹出
- LRANGE：范围查询
- BLPOP/BRPOP：阻塞弹出

**使用场景：**
- **消息队列**：生产者LPUSH，消费者BRPOP
- **最新列表**：最新文章、最新评论
- **栈和队列**：LIFO和FIFO数据结构

**4. Set类型详解：**

**特点：**
- 无序集合
- 元素唯一
- 支持集合运算

**常用命令：**
- SADD/SREM：添加/删除元素
- SISMEMBER：判断成员存在
- SINTER/SUNION/SDIFF：交集/并集/差集
- SPOP/SRANDMEMBER：随机获取

**使用场景：**
- **去重**：用户访问记录去重
- **标签系统**：用户标签、文章标签
- **抽奖系统**：SPOP随机抽取
- **共同关注**：SINTER计算共同好友

**5. ZSet类型详解：**

**特点：**
- 有序集合
- 每个元素关联一个分数
- 按分数排序

**常用命令：**
- ZADD/ZREM：添加/删除元素
- ZRANGE/ZREVRANGE：范围查询
- ZRANK/ZREVRANK：获取排名
- ZINCRBY：增加分数

**使用场景：**
- **排行榜**：游戏积分榜、销量排行
- **延迟队列**：分数为执行时间
- **限流**：滑动窗口限流
- **自动补全**：搜索建议

**代码示例：**
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ZSetOperations;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import java.time.LocalDateTime;
import java.util.*;
import java.util.concurrent.TimeUnit;

@SpringBootApplication
public class RedisDataTypesApplication {
    public static void main(String[] args) {
        SpringApplication.run(RedisDataTypesApplication.class, args);
    }
}

// 1. String类型应用示例
@Service
public class StringRedisService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 缓存用户信息
     */
    public void cacheUser(Long userId, User user) {
        String key = "user:" + userId;
        // 序列化对象存储，设置1小时过期
        redisTemplate.opsForValue().set(key, user, 1, TimeUnit.HOURS);
        System.out.println("用户信息已缓存: " + userId);
    }
    
    public User getUser(Long userId) {
        String key = "user:" + userId;
        return (User) redisTemplate.opsForValue().get(key);
    }
    
    /**
     * 计数器：网站访问量
     */
    public Long incrementPageView(String page) {
        String key = "page_view:" + page;
        return redisTemplate.opsForValue().increment(key);
    }
    
    public Long getPageView(String page) {
        String key = "page_view:" + page;
        Object value = redisTemplate.opsForValue().get(key);
        return value != null ? Long.valueOf(value.toString()) : 0L;
    }
    
    /**
     * 分布式锁实现
     */
    public boolean tryLock(String lockKey, String requestId, int expireTime) {
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then " +
                       "return redis.call('del', KEYS[1]) " +
                       "else return 0 end";
        
        // 使用SET NX EX命令实现分布式锁
        Boolean result = redisTemplate.opsForValue().setIfAbsent(
            lockKey, requestId, expireTime, TimeUnit.SECONDS);
        
        return Boolean.TRUE.equals(result);
    }
    
    public boolean releaseLock(String lockKey, String requestId) {
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then " +
                       "return redis.call('del', KEYS[1]) " +
                       "else return 0 end";
        
        DefaultRedisScript<Long> redisScript = new DefaultRedisScript<>();
        redisScript.setScriptText(script);
        redisScript.setResultType(Long.class);
        
        Long result = redisTemplate.execute(redisScript, 
            Collections.singletonList(lockKey), requestId);
        
        return Long.valueOf(1).equals(result);
    }
    
    /**
     * 限流器：滑动窗口限流
     */
    public boolean isAllowed(String userId, int limit, int windowSize) {
        String key = "rate_limit:" + userId;
        long now = System.currentTimeMillis();
        
        // 使用Lua脚本保证原子性
        String script = 
            "local key = KEYS[1] " +
            "local window = tonumber(ARGV[1]) " +
            "local limit = tonumber(ARGV[2]) " +
            "local now = tonumber(ARGV[3]) " +
            "local clearBefore = now - window " +
            "redis.call('zremrangebyscore', key, 0, clearBefore) " +
            "local current = redis.call('zcard', key) " +
            "if current < limit then " +
            "  redis.call('zadd', key, now, now) " +
            "  redis.call('expire', key, window) " +
            "  return 1 " +
            "else " +
            "  return 0 " +
            "end";
        
        DefaultRedisScript<Long> redisScript = new DefaultRedisScript<>();
        redisScript.setScriptText(script);
        redisScript.setResultType(Long.class);
        
        Long result = redisTemplate.execute(redisScript, 
            Collections.singletonList(key), 
            String.valueOf(windowSize * 1000), 
            String.valueOf(limit), 
            String.valueOf(now));
        
        return Long.valueOf(1).equals(result);
    }
}

// 2. Hash类型应用示例
@Service
public class HashRedisService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 购物车管理
     */
    public void addToCart(Long userId, Long productId, Integer quantity) {
        String key = "cart:" + userId;
        redisTemplate.opsForHash().put(key, productId.toString(), quantity);
        // 设置购物车过期时间为7天
        redisTemplate.expire(key, 7, TimeUnit.DAYS);
        System.out.println("商品已添加到购物车: " + productId);
    }
    
    public void removeFromCart(Long userId, Long productId) {
        String key = "cart:" + userId;
        redisTemplate.opsForHash().delete(key, productId.toString());
    }
    
    public Map<Object, Object> getCart(Long userId) {
        String key = "cart:" + userId;
        return redisTemplate.opsForHash().entries(key);
    }
    
    public void updateCartQuantity(Long userId, Long productId, Integer quantity) {
        String key = "cart:" + userId;
        if (quantity <= 0) {
            removeFromCart(userId, productId);
        } else {
            redisTemplate.opsForHash().put(key, productId.toString(), quantity);
        }
    }
    
    /**
     * 用户信息存储（避免序列化开销）
     */
    public void saveUserProfile(Long userId, UserProfile profile) {
        String key = "profile:" + userId;
        Map<String, Object> profileMap = new HashMap<>();
        profileMap.put("name", profile.getName());
        profileMap.put("email", profile.getEmail());
        profileMap.put("age", profile.getAge());
        profileMap.put("city", profile.getCity());
        
        redisTemplate.opsForHash().putAll(key, profileMap);
        redisTemplate.expire(key, 1, TimeUnit.HOURS);
    }
    
    public UserProfile getUserProfile(Long userId) {
        String key = "profile:" + userId;
        Map<Object, Object> profileMap = redisTemplate.opsForHash().entries(key);
        
        if (profileMap.isEmpty()) {
            return null;
        }
        
        UserProfile profile = new UserProfile();
        profile.setName((String) profileMap.get("name"));
        profile.setEmail((String) profileMap.get("email"));
        profile.setAge(Integer.valueOf(profileMap.get("age").toString()));
        profile.setCity((String) profileMap.get("city"));
        
        return profile;
    }
}

// 3. List类型应用示例
@Service
public class ListRedisService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 消息队列实现
     */
    public void sendMessage(String queue, Object message) {
        String key = "queue:" + queue;
        redisTemplate.opsForList().leftPush(key, message);
        System.out.println("消息已发送到队列: " + queue);
    }
    
    public Object receiveMessage(String queue, int timeout) {
        String key = "queue:" + queue;
        // 阻塞式获取消息
        List<Object> result = redisTemplate.opsForList().rightPop(key, timeout, TimeUnit.SECONDS);
        return result != null && !result.isEmpty() ? result.get(0) : null;
    }
    
    /**
     * 最新文章列表
     */
    public void addLatestArticle(Long articleId) {
        String key = "latest_articles";
        redisTemplate.opsForList().leftPush(key, articleId);
        
        // 只保留最新的100篇文章
        redisTemplate.opsForList().trim(key, 0, 99);
    }
    
    public List<Object> getLatestArticles(int count) {
        String key = "latest_articles";
        return redisTemplate.opsForList().range(key, 0, count - 1);
    }
    
    /**
     * 用户操作历史
     */
    public void recordUserAction(Long userId, String action) {
        String key = "user_actions:" + userId;
        Map<String, Object> actionData = new HashMap<>();
        actionData.put("action", action);
        actionData.put("timestamp", System.currentTimeMillis());
        
        redisTemplate.opsForList().leftPush(key, actionData);
        
        // 只保留最近1000条操作记录
        redisTemplate.opsForList().trim(key, 0, 999);
        
        // 设置过期时间30天
        redisTemplate.expire(key, 30, TimeUnit.DAYS);
    }
    
    public List<Object> getUserActions(Long userId, int count) {
        String key = "user_actions:" + userId;
        return redisTemplate.opsForList().range(key, 0, count - 1);
    }
}

// 4. Set类型应用示例
@Service
public class SetRedisService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 用户标签系统
     */
    public void addUserTag(Long userId, String tag) {
        String key = "user_tags:" + userId;
        redisTemplate.opsForSet().add(key, tag);
        redisTemplate.expire(key, 7, TimeUnit.DAYS);
    }
    
    public void removeUserTag(Long userId, String tag) {
        String key = "user_tags:" + userId;
        redisTemplate.opsForSet().remove(key, tag);
    }
    
    public Set<Object> getUserTags(Long userId) {
        String key = "user_tags:" + userId;
        return redisTemplate.opsForSet().members(key);
    }
    
    /**
     * 共同关注功能
     */
    public void followUser(Long userId, Long followeeId) {
        String key = "following:" + userId;
        redisTemplate.opsForSet().add(key, followeeId);
        
        String followerKey = "followers:" + followeeId;
        redisTemplate.opsForSet().add(followerKey, userId);
    }
    
    public void unfollowUser(Long userId, Long followeeId) {
        String key = "following:" + userId;
        redisTemplate.opsForSet().remove(key, followeeId);
        
        String followerKey = "followers:" + followeeId;
        redisTemplate.opsForSet().remove(followerKey, userId);
    }
    
    public Set<Object> getCommonFollowing(Long userId1, Long userId2) {
        String key1 = "following:" + userId1;
        String key2 = "following:" + userId2;
        return redisTemplate.opsForSet().intersect(key1, key2);
    }
    
    /**
     * 抽奖系统
     */
    public void joinLottery(String lotteryId, Long userId) {
        String key = "lottery:" + lotteryId;
        redisTemplate.opsForSet().add(key, userId);
        redisTemplate.expire(key, 1, TimeUnit.DAYS);
    }
    
    public Long drawWinner(String lotteryId) {
        String key = "lottery:" + lotteryId;
        Object winner = redisTemplate.opsForSet().pop(key);
        return winner != null ? Long.valueOf(winner.toString()) : null;
    }
    
    public Long getLotteryParticipants(String lotteryId) {
        String key = "lottery:" + lotteryId;
        return redisTemplate.opsForSet().size(key);
    }
}

// 5. ZSet类型应用示例
@Service
public class ZSetRedisService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 游戏排行榜
     */
    public void updateScore(Long userId, double score) {
        String key = "game_leaderboard";
        redisTemplate.opsForZSet().add(key, userId, score);
    }
    
    public Set<ZSetOperations.TypedTuple<Object>> getTopPlayers(int count) {
        String key = "game_leaderboard";
        // 获取分数最高的玩家（降序）
        return redisTemplate.opsForZSet().reverseRangeWithScores(key, 0, count - 1);
    }
    
    public Long getUserRank(Long userId) {
        String key = "game_leaderboard";
        // 获取用户排名（降序排名）
        return redisTemplate.opsForZSet().reverseRank(key, userId);
    }
    
    public Double getUserScore(Long userId) {
        String key = "game_leaderboard";
        return redisTemplate.opsForZSet().score(key, userId);
    }
    
    /**
     * 延迟队列实现
     */
    public void addDelayedTask(String taskId, Object taskData, long delaySeconds) {
        String key = "delayed_tasks";
        long executeTime = System.currentTimeMillis() + delaySeconds * 1000;
        
        Map<String, Object> task = new HashMap<>();
        task.put("id", taskId);
        task.put("data", taskData);
        task.put("createTime", System.currentTimeMillis());
        
        redisTemplate.opsForZSet().add(key, task, executeTime);
        System.out.println("延迟任务已添加: " + taskId + ", 延迟: " + delaySeconds + "秒");
    }
    
    public List<Object> getReadyTasks() {
        String key = "delayed_tasks";
        long now = System.currentTimeMillis();
        
        // 获取到期的任务
        Set<Object> tasks = redisTemplate.opsForZSet().rangeByScore(key, 0, now);
        
        if (!tasks.isEmpty()) {
            // 移除已获取的任务
            redisTemplate.opsForZSet().removeRangeByScore(key, 0, now);
        }
        
        return new ArrayList<>(tasks);
    }
    
    /**
     * 热门文章排行（基于点击量）
     */
    public void incrementArticleView(Long articleId) {
        String key = "hot_articles";
        redisTemplate.opsForZSet().incrementScore(key, articleId, 1);
        
        // 设置过期时间，定期清理
        redisTemplate.expire(key, 1, TimeUnit.DAYS);
    }
    
    public Set<ZSetOperations.TypedTuple<Object>> getHotArticles(int count) {
        String key = "hot_articles";
        return redisTemplate.opsForZSet().reverseRangeWithScores(key, 0, count - 1);
    }
    
    /**
     * 滑动窗口限流
     */
    public boolean slidingWindowRateLimit(String userId, int limit, int windowSeconds) {
        String key = "rate_limit:" + userId;
        long now = System.currentTimeMillis();
        long windowStart = now - windowSeconds * 1000L;
        
        // 清理窗口外的记录
        redisTemplate.opsForZSet().removeRangeByScore(key, 0, windowStart);
        
        // 检查当前窗口内的请求数
        Long count = redisTemplate.opsForZSet().count(key, windowStart, now);
        
        if (count < limit) {
            // 添加当前请求
            redisTemplate.opsForZSet().add(key, UUID.randomUUID().toString(), now);
            redisTemplate.expire(key, windowSeconds, TimeUnit.SECONDS);
            return true;
        }
        
        return false;
    }
}

// 6. 高级数据类型示例
@Service
public class AdvancedRedisService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * Bitmap应用：用户签到
     */
    public void userCheckIn(Long userId, int day) {
        String key = "checkin:" + userId + ":" + getCurrentMonth();
        redisTemplate.opsForValue().setBit(key, day - 1, true);
        redisTemplate.expire(key, 32, TimeUnit.DAYS);
    }
    
    public boolean isUserCheckedIn(Long userId, int day) {
        String key = "checkin:" + userId + ":" + getCurrentMonth();
        return Boolean.TRUE.equals(redisTemplate.opsForValue().getBit(key, day - 1));
    }
    
    public long getUserCheckInDays(Long userId) {
        String key = "checkin:" + userId + ":" + getCurrentMonth();
        // 使用BITCOUNT命令统计签到天数
        return redisTemplate.execute((connection) -> {
            return connection.bitCount(key.getBytes());
        });
    }
    
    /**
     * HyperLogLog应用：UV统计
     */
    public void recordUV(String page, String userId) {
        String key = "uv:" + page + ":" + getCurrentDate();
        redisTemplate.opsForHyperLogLog().add(key, userId);
        redisTemplate.expire(key, 7, TimeUnit.DAYS);
    }
    
    public long getUV(String page) {
        String key = "uv:" + page + ":" + getCurrentDate();
        return redisTemplate.opsForHyperLogLog().size(key);
    }
    
    private String getCurrentMonth() {
        return LocalDateTime.now().format(java.time.format.DateTimeFormatter.ofPattern("yyyy-MM"));
    }
    
    private String getCurrentDate() {
        return LocalDateTime.now().format(java.time.format.DateTimeFormatter.ofPattern("yyyy-MM-dd"));
    }
}
```

**性能优化建议：**

**1. 内存优化**
- 使用Hash代替String存储对象（节省内存）
- 合理设置过期时间，避免内存泄漏
- 使用压缩算法存储大对象

**2. 网络优化**
- 使用Pipeline批量操作
- 避免大key操作
- 合理使用连接池

**3. 数据结构选择**
- 小数据量使用ziplist编码
- 大数据量考虑分片存储
- 根据访问模式选择合适的数据类型

**4. 监控告警**
- 监控内存使用率
- 监控慢查询
- 设置合理的告警阈值

**最佳实践：**

1. **键命名规范**：使用统一的命名规范，便于管理
2. **过期时间设置**：避免数据永久存储，合理设置TTL
3. **数据序列化**：选择高效的序列化方式
4. **错误处理**：处理Redis连接异常和数据异常
5. **安全考虑**：设置密码，限制访问IP
6. **备份策略**：定期备份重要数据
7. **版本兼容**：注意Redis版本差异

**Q29: 缓存穿透、缓存击穿、缓存雪崩的解决方案？**

**详细答案：**

缓存是提升系统性能的重要手段，但在高并发场景下会遇到三个经典问题：缓存穿透、缓存击穿和缓存雪崩。

**问题对比分析：**

| 问题类型 | 定义 | 影响 | 发生场景 | 解决难度 |
|----------|------|------|----------|----------|
| 缓存穿透 | 查询不存在的数据 | 数据库压力大 | 恶意攻击、业务bug | 中等 |
| 缓存击穿 | 热点数据过期 | 瞬时数据库压力 | 热点key过期 | 较低 |
| 缓存雪崩 | 大量缓存同时失效 | 系统整体不可用 | 缓存集中过期、宕机 | 较高 |

## 1. 缓存穿透详解

**问题描述：**
查询一个不存在的数据，由于缓存不命中，会去查询数据库，数据库也没有该记录，无法将查询结果写入缓存，导致每次查询都会去数据库，给数据库带来压力。

**典型场景：**
- 恶意攻击：故意查询不存在的用户ID
- 业务bug：程序错误导致查询无效数据
- 数据删除：数据被删除但缓存未及时清理

**解决方案：**

### 1.1 布隆过滤器

**原理：**
使用布隆过滤器在缓存之前进行过滤，如果布隆过滤器判断数据不存在，直接返回，不查询数据库。

**优点：**
- 内存占用极小
- 查询效率高O(k)
- 可以处理海量数据

**缺点：**
- 存在误判（假阳性）
- 不支持删除操作
- 需要预先知道数据集合

### 1.2 空值缓存

**原理：**
当查询数据库返回空结果时，也将空值写入缓存，但设置较短的过期时间。

**优点：**
- 实现简单
- 可以防止短时间内的重复查询

**缺点：**
- 占用缓存空间
- 可能缓存大量无效数据

### 1.3 参数校验

**原理：**
在接口层面对参数进行校验，过滤明显不合法的请求。

## 2. 缓存击穿详解

**问题描述：**
某个热点key在缓存过期的瞬间，有大量并发请求，这些请求都会去查询数据库，导致数据库压力瞬间增大。

**典型场景：**
- 热门商品信息过期
- 热点用户数据过期
- 系统配置信息过期

**解决方案：**

### 2.1 分布式锁

**原理：**
使用分布式锁确保只有一个线程去查询数据库，其他线程等待结果。

### 2.2 热点数据永不过期

**原理：**
对于热点数据，设置永不过期，通过异步任务定期更新。

### 2.3 提前刷新

**原理：**
在缓存即将过期前，异步刷新缓存数据。

## 3. 缓存雪崩详解

**问题描述：**
大量缓存在同一时间过期，或者缓存服务宕机，导致大量请求直接访问数据库，可能导致数据库压力过大而宕机。

**典型场景：**
- 缓存集中过期（如凌晨统一过期）
- 缓存服务器宕机
- 网络故障导致缓存不可用

**解决方案：**

### 3.1 过期时间随机化

**原理：**
在设置缓存过期时间时，加上一个随机值，避免大量缓存同时过期。

### 3.2 多级缓存

**原理：**
建立多级缓存体系，如本地缓存+Redis缓存，提高系统可用性。

### 3.3 熔断降级

**原理：**
当缓存不可用时，启用熔断机制，返回默认值或降级处理。

**代码实现示例：**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import java.util.*;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;
import com.google.common.hash.BloomFilter;
import com.google.common.hash.Funnels;
import java.nio.charset.Charset;

@SpringBootApplication
public class CacheProblemSolutionApplication {
    public static void main(String[] args) {
        SpringApplication.run(CacheProblemSolutionApplication.class, args);
    }
}

// 1. 缓存穿透解决方案
@Service
public class CachePenetrationService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    // 布隆过滤器
    private BloomFilter<String> bloomFilter;
    
    // 本地锁（实际应用中应使用分布式锁）
    private final ReentrantLock lock = new ReentrantLock();
    
    @PostConstruct
    public void initBloomFilter() {
        // 初始化布隆过滤器，预期元素数量100万，误判率0.01%
        bloomFilter = BloomFilter.create(
            Funnels.stringFunnel(Charset.defaultCharset()), 
            1000000, 
            0.0001
        );
        
        // 将所有存在的用户ID加入布隆过滤器
        initBloomFilterData();
    }
    
    private void initBloomFilterData() {
        // 从数据库加载所有用户ID
        List<String> existingUserIds = getAllUserIdsFromDatabase();
        for (String userId : existingUserIds) {
            bloomFilter.put(userId);
        }
        System.out.println("布隆过滤器初始化完成，加载用户数: " + existingUserIds.size());
    }
    
    /**
     * 使用布隆过滤器防止缓存穿透
     */
    public User getUserWithBloomFilter(String userId) {
        // 1. 布隆过滤器检查
        if (!bloomFilter.mightContain(userId)) {
            System.out.println("布隆过滤器判断用户不存在: " + userId);
            return null;
        }
        
        // 2. 查询缓存
        String cacheKey = "user:" + userId;
        User cachedUser = (User) redisTemplate.opsForValue().get(cacheKey);
        if (cachedUser != null) {
            System.out.println("缓存命中: " + userId);
            return cachedUser;
        }
        
        // 3. 查询数据库
        User dbUser = getUserFromDatabase(userId);
        if (dbUser != null) {
            // 4. 写入缓存
            redisTemplate.opsForValue().set(cacheKey, dbUser, 300, TimeUnit.SECONDS);
            System.out.println("数据库查询成功并缓存: " + userId);
        } else {
            // 5. 缓存空值，防止缓存穿透
            redisTemplate.opsForValue().set(cacheKey, "NULL", 60, TimeUnit.SECONDS);
            System.out.println("用户不存在，缓存空值: " + userId);
        }
        
        return dbUser;
    }
    
    /**
     * 空值缓存防止缓存穿透
     */
    public User getUserWithNullCache(String userId) {
        String cacheKey = "user:" + userId;
        
        // 1. 查询缓存
        Object cached = redisTemplate.opsForValue().get(cacheKey);
        if (cached != null) {
            if ("NULL".equals(cached)) {
                System.out.println("缓存空值命中: " + userId);
                return null;
            }
            return (User) cached;
        }
        
        // 2. 查询数据库
        User dbUser = getUserFromDatabase(userId);
        if (dbUser != null) {
            // 3. 缓存真实数据
            redisTemplate.opsForValue().set(cacheKey, dbUser, 300, TimeUnit.SECONDS);
        } else {
            // 4. 缓存空值，设置较短过期时间
            redisTemplate.opsForValue().set(cacheKey, "NULL", 60, TimeUnit.SECONDS);
        }
        
        return dbUser;
    }
    
    /**
     * 参数校验防止缓存穿透
     */
    public User getUserWithValidation(String userId) {
        // 1. 参数校验
        if (!isValidUserId(userId)) {
            System.out.println("无效的用户ID: " + userId);
            return null;
        }
        
        // 2. 正常查询流程
        return getUserWithNullCache(userId);
    }
    
    private boolean isValidUserId(String userId) {
        if (userId == null || userId.trim().isEmpty()) {
            return false;
        }
        
        // 检查用户ID格式（例如：只允许数字，长度在1-20之间）
        return userId.matches("^[1-9]\\d{0,19}$");
    }
    
    private List<String> getAllUserIdsFromDatabase() {
        // 模拟从数据库获取所有用户ID
        List<String> userIds = new ArrayList<>();
        for (int i = 1; i <= 1000; i++) {
            userIds.add(String.valueOf(i));
        }
        return userIds;
    }
    
    private User getUserFromDatabase(String userId) {
        // 模拟数据库查询
        try {
            Thread.sleep(100); // 模拟数据库查询耗时
            
            // 只有ID在1-1000范围内的用户存在
            int id = Integer.parseInt(userId);
            if (id >= 1 && id <= 1000) {
                User user = new User();
                user.setId(userId);
                user.setName("User" + userId);
                user.setEmail(userId + "@example.com");
                return user;
            }
        } catch (Exception e) {
            System.err.println("数据库查询异常: " + e.getMessage());
        }
        return null;
    }
}

// 2. 缓存击穿解决方案
@Service
public class CacheBreakdownService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    private final Map<String, ReentrantLock> lockMap = new ConcurrentHashMap<>();
    
    /**
     * 使用分布式锁防止缓存击穿
     */
    public User getHotUserWithLock(String userId) {
        String cacheKey = "hot_user:" + userId;
        
        // 1. 查询缓存
        User cachedUser = (User) redisTemplate.opsForValue().get(cacheKey);
        if (cachedUser != null) {
            return cachedUser;
        }
        
        // 2. 缓存未命中，使用分布式锁
        String lockKey = "lock:user:" + userId;
        String requestId = UUID.randomUUID().toString();
        
        try {
            // 3. 尝试获取分布式锁
            if (tryLock(lockKey, requestId, 10)) {
                // 4. 获取锁成功，再次检查缓存（双重检查）
                cachedUser = (User) redisTemplate.opsForValue().get(cacheKey);
                if (cachedUser != null) {
                    return cachedUser;
                }
                
                // 5. 查询数据库
                User dbUser = getUserFromDatabase(userId);
                if (dbUser != null) {
                    // 6. 写入缓存，设置较长过期时间
                    redisTemplate.opsForValue().set(cacheKey, dbUser, 1800, TimeUnit.SECONDS);
                    System.out.println("热点数据已缓存: " + userId);
                }
                
                return dbUser;
            } else {
                // 7. 获取锁失败，等待一段时间后重试
                Thread.sleep(50);
                return (User) redisTemplate.opsForValue().get(cacheKey);
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return null;
        } finally {
            // 8. 释放锁
            releaseLock(lockKey, requestId);
        }
    }
    
    /**
     * 热点数据永不过期策略
     */
    public User getHotUserNeverExpire(String userId) {
        String cacheKey = "hot_user_never:" + userId;
        String updateTimeKey = cacheKey + ":update_time";
        
        // 1. 查询缓存
        User cachedUser = (User) redisTemplate.opsForValue().get(cacheKey);
        
        if (cachedUser != null) {
            // 2. 检查更新时间
            Long updateTime = (Long) redisTemplate.opsForValue().get(updateTimeKey);
            long currentTime = System.currentTimeMillis();
            
            // 3. 如果数据超过30分钟未更新，异步刷新
            if (updateTime == null || (currentTime - updateTime) > 30 * 60 * 1000) {
                // 异步更新缓存
                CompletableFuture.runAsync(() -> {
                    refreshHotUserCache(userId);
                });
            }
            
            return cachedUser;
        } else {
            // 4. 缓存不存在，同步加载
            return refreshHotUserCache(userId);
        }
    }
    
    private User refreshHotUserCache(String userId) {
        String cacheKey = "hot_user_never:" + userId;
        String updateTimeKey = cacheKey + ":update_time";
        
        try {
            User dbUser = getUserFromDatabase(userId);
            if (dbUser != null) {
                // 设置永不过期
                redisTemplate.opsForValue().set(cacheKey, dbUser);
                redisTemplate.opsForValue().set(updateTimeKey, System.currentTimeMillis());
                System.out.println("热点数据已刷新: " + userId);
            }
            return dbUser;
        } catch (Exception e) {
            System.err.println("刷新热点数据失败: " + e.getMessage());
            return null;
        }
    }
    
    /**
     * 提前刷新策略
     */
    public User getUserWithPreRefresh(String userId) {
        String cacheKey = "user_pre:" + userId;
        
        // 1. 查询缓存
        User cachedUser = (User) redisTemplate.opsForValue().get(cacheKey);
        
        if (cachedUser != null) {
            // 2. 检查剩余过期时间
            Long ttl = redisTemplate.getExpire(cacheKey, TimeUnit.SECONDS);
            
            // 3. 如果剩余时间少于60秒，异步刷新
            if (ttl != null && ttl < 60) {
                CompletableFuture.runAsync(() -> {
                    User dbUser = getUserFromDatabase(userId);
                    if (dbUser != null) {
                        redisTemplate.opsForValue().set(cacheKey, dbUser, 300, TimeUnit.SECONDS);
                        System.out.println("提前刷新缓存: " + userId);
                    }
                });
            }
            
            return cachedUser;
        } else {
            // 4. 缓存不存在，同步加载
            User dbUser = getUserFromDatabase(userId);
            if (dbUser != null) {
                redisTemplate.opsForValue().set(cacheKey, dbUser, 300, TimeUnit.SECONDS);
            }
            return dbUser;
        }
    }
    
    private boolean tryLock(String lockKey, String requestId, int expireTime) {
        // 使用SET NX EX命令实现分布式锁
        Boolean result = redisTemplate.opsForValue().setIfAbsent(
            lockKey, requestId, expireTime, TimeUnit.SECONDS);
        return Boolean.TRUE.equals(result);
    }
    
    private void releaseLock(String lockKey, String requestId) {
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then " +
                       "return redis.call('del', KEYS[1]) " +
                       "else return 0 end";
        
        DefaultRedisScript<Long> redisScript = new DefaultRedisScript<>();
        redisScript.setScriptText(script);
        redisScript.setResultType(Long.class);
        
        redisTemplate.execute(redisScript, Collections.singletonList(lockKey), requestId);
    }
    
    private User getUserFromDatabase(String userId) {
        // 模拟数据库查询
        try {
            Thread.sleep(200); // 模拟数据库查询耗时
            
            User user = new User();
            user.setId(userId);
            user.setName("HotUser" + userId);
            user.setEmail(userId + "@hotmail.com");
            return user;
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return null;
        }
    }
}

// 3. 缓存雪崩解决方案
@Service
public class CacheAvalancheService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    // 本地缓存作为二级缓存
    private final Map<String, CacheItem> localCache = new ConcurrentHashMap<>();
    
    /**
     * 过期时间随机化防止缓存雪崩
     */
    public User getUserWithRandomExpire(String userId) {
        String cacheKey = "user_random:" + userId;
        
        // 1. 查询缓存
        User cachedUser = (User) redisTemplate.opsForValue().get(cacheKey);
        if (cachedUser != null) {
            return cachedUser;
        }
        
        // 2. 查询数据库
        User dbUser = getUserFromDatabase(userId);
        if (dbUser != null) {
            // 3. 设置随机过期时间（基础时间300秒 + 随机0-60秒）
            int randomExpire = 300 + new Random().nextInt(60);
            redisTemplate.opsForValue().set(cacheKey, dbUser, randomExpire, TimeUnit.SECONDS);
            System.out.println("缓存已设置，过期时间: " + randomExpire + "秒");
        }
        
        return dbUser;
    }
    
    /**
     * 多级缓存防止缓存雪崩
     */
    public User getUserWithMultiLevelCache(String userId) {
        String cacheKey = "user_multi:" + userId;
        
        // 1. 查询本地缓存（一级缓存）
        CacheItem localItem = localCache.get(cacheKey);
        if (localItem != null && !localItem.isExpired()) {
            System.out.println("本地缓存命中: " + userId);
            return (User) localItem.getValue();
        }
        
        // 2. 查询Redis缓存（二级缓存）
        User cachedUser = null;
        try {
            cachedUser = (User) redisTemplate.opsForValue().get(cacheKey);
            if (cachedUser != null) {
                // 更新本地缓存
                localCache.put(cacheKey, new CacheItem(cachedUser, 60)); // 本地缓存60秒
                System.out.println("Redis缓存命中: " + userId);
                return cachedUser;
            }
        } catch (Exception e) {
            System.err.println("Redis查询失败，使用本地缓存: " + e.getMessage());
            // Redis不可用时，使用本地缓存（即使过期）
            if (localItem != null) {
                System.out.println("Redis故障，使用过期本地缓存: " + userId);
                return (User) localItem.getValue();
            }
        }
        
        // 3. 查询数据库
        User dbUser = getUserFromDatabase(userId);
        if (dbUser != null) {
            try {
                // 4. 更新Redis缓存
                int randomExpire = 300 + new Random().nextInt(60);
                redisTemplate.opsForValue().set(cacheKey, dbUser, randomExpire, TimeUnit.SECONDS);
            } catch (Exception e) {
                System.err.println("Redis更新失败: " + e.getMessage());
            }
            
            // 5. 更新本地缓存
            localCache.put(cacheKey, new CacheItem(dbUser, 60));
        }
        
        return dbUser;
    }
    
    /**
     * 熔断降级防止缓存雪崩
     */
    public User getUserWithCircuitBreaker(String userId) {
        String cacheKey = "user_cb:" + userId;
        
        try {
            // 1. 查询缓存
            User cachedUser = (User) redisTemplate.opsForValue().get(cacheKey);
            if (cachedUser != null) {
                return cachedUser;
            }
            
            // 2. 查询数据库
            User dbUser = getUserFromDatabase(userId);
            if (dbUser != null) {
                // 3. 更新缓存
                redisTemplate.opsForValue().set(cacheKey, dbUser, 300, TimeUnit.SECONDS);
            }
            
            return dbUser;
        } catch (Exception e) {
            System.err.println("缓存服务异常，启用降级: " + e.getMessage());
            
            // 4. 降级处理：返回默认用户信息
            return getDefaultUser(userId);
        }
    }
    
    /**
     * 缓存预热防止缓存雪崩
     */
    @PostConstruct
    public void warmUpCache() {
        System.out.println("开始缓存预热...");
        
        // 预热热点数据
        List<String> hotUserIds = getHotUserIds();
        
        for (String userId : hotUserIds) {
            try {
                String cacheKey = "user_warmup:" + userId;
                User user = getUserFromDatabase(userId);
                if (user != null) {
                    // 设置随机过期时间
                    int randomExpire = 1800 + new Random().nextInt(300); // 30-35分钟
                    redisTemplate.opsForValue().set(cacheKey, user, randomExpire, TimeUnit.SECONDS);
                }
                
                // 避免对数据库造成过大压力
                Thread.sleep(10);
            } catch (Exception e) {
                System.err.println("预热缓存失败: " + userId + ", " + e.getMessage());
            }
        }
        
        System.out.println("缓存预热完成");
    }
    
    private User getDefaultUser(String userId) {
        User defaultUser = new User();
        defaultUser.setId(userId);
        defaultUser.setName("DefaultUser");
        defaultUser.setEmail("default@example.com");
        return defaultUser;
    }
    
    private List<String> getHotUserIds() {
        // 返回热点用户ID列表
        List<String> hotIds = new ArrayList<>();
        for (int i = 1; i <= 100; i++) {
            hotIds.add(String.valueOf(i));
        }
        return hotIds;
    }
    
    private User getUserFromDatabase(String userId) {
        // 模拟数据库查询
        try {
            Thread.sleep(100);
            
            User user = new User();
            user.setId(userId);
            user.setName("User" + userId);
            user.setEmail(userId + "@example.com");
            return user;
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return null;
        }
    }
    
    // 本地缓存项
    private static class CacheItem {
        private final Object value;
        private final long expireTime;
        
        public CacheItem(Object value, int ttlSeconds) {
            this.value = value;
            this.expireTime = System.currentTimeMillis() + ttlSeconds * 1000L;
        }
        
        public Object getValue() {
            return value;
        }
        
        public boolean isExpired() {
            return System.currentTimeMillis() > expireTime;
        }
    }
}

// 用户实体类
class User {
    private String id;
    private String name;
    private String email;
    
    // 构造函数、getter和setter方法
    public User() {}
    
    public String getId() { return id; }
    public void setId(String id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    @Override
    public String toString() {
        return String.format("User{id='%s', name='%s', email='%s'}", id, name, email);
    }
}

// 控制器示例
@RestController
@RequestMapping("/api/cache")
public class CacheController {
    
    @Autowired
    private CachePenetrationService penetrationService;
    
    @Autowired
    private CacheBreakdownService breakdownService;
    
    @Autowired
    private CacheAvalancheService avalancheService;
    
    @GetMapping("/penetration/{userId}")
    public User testPenetration(@PathVariable String userId) {
        return penetrationService.getUserWithBloomFilter(userId);
    }
    
    @GetMapping("/breakdown/{userId}")
    public User testBreakdown(@PathVariable String userId) {
        return breakdownService.getHotUserWithLock(userId);
    }
    
    @GetMapping("/avalanche/{userId}")
    public User testAvalanche(@PathVariable String userId) {
        return avalancheService.getUserWithMultiLevelCache(userId);
    }
}
```

**解决方案对比总结：**

| 问题 | 解决方案 | 优点 | 缺点 | 适用场景 |
|------|----------|------|------|----------|
| **缓存穿透** | 布隆过滤器 | 内存占用小，效率高 | 存在误判，不支持删除 | 海量数据过滤 |
| | 空值缓存 | 实现简单 | 占用缓存空间 | 小规模应用 |
| | 参数校验 | 从源头解决 | 需要业务配合 | 所有应用 |
| **缓存击穿** | 分布式锁 | 精确控制 | 性能开销 | 热点数据 |
| | 永不过期 | 性能好 | 数据可能不一致 | 准实时数据 |
| | 提前刷新 | 用户体验好 | 实现复杂 | 高要求场景 |
| **缓存雪崩** | 随机过期 | 实现简单 | 治标不治本 | 基础防护 |
| | 多级缓存 | 可用性高 | 架构复杂 | 高可用要求 |
| | 熔断降级 | 保证服务可用 | 功能受限 | 容错要求高 |

**最佳实践建议：**

1. **预防为主**：合理设计缓存策略，避免问题发生
2. **多重防护**：结合多种解决方案，提高系统健壮性
3. **监控告警**：实时监控缓存命中率、异常情况
4. **容量规划**：合理评估缓存容量和数据库承载能力
5. **降级预案**：准备完善的降级和恢复方案
6. **定期演练**：定期进行故障演练，验证解决方案有效性

### 4.3 消息队列类

**Q30: 消息队列的作用和选型考虑？**

**详细答案：**

消息队列（Message Queue，MQ）是分布式系统中重要的中间件，用于实现系统间的异步通信和解耦。

**消息队列核心作用：**

| 作用 | 描述 | 应用场景 | 优势 |
|------|------|----------|------|
| 异步处理 | 将耗时操作异步化 | 邮件发送、图片处理 | 提升响应速度 |
| 系统解耦 | 降低系统间依赖 | 微服务通信 | 提高系统灵活性 |
| 流量削峰 | 平滑处理突发流量 | 秒杀、促销活动 | 保护下游系统 |
| 数据分发 | 一对多数据传输 | 订单状态通知 | 简化数据同步 |
| 最终一致性 | 保证数据最终一致 | 分布式事务 | 提高系统可用性 |

## 1. 消息队列作用详解

### 1.1 异步处理

**场景描述：**
用户注册后需要发送邮件、短信通知，如果同步处理会影响用户体验。

**传统同步方式：**
```
用户注册 → 写入数据库 → 发送邮件 → 发送短信 → 返回结果
总耗时：100ms + 2000ms + 1000ms = 3100ms
```

**异步处理方式：**
```
用户注册 → 写入数据库 → 发送消息到MQ → 返回结果
总耗时：100ms + 10ms = 110ms
后台异步：消费消息 → 发送邮件 → 发送短信
```

### 1.2 系统解耦

**问题：**
订单系统直接调用库存、支付、物流等系统，系统间耦合度高。

**解决方案：**
通过消息队列实现系统解耦，订单系统只需发送消息，不关心具体的处理逻辑。

### 1.3 流量削峰

**场景：**
秒杀活动瞬间产生大量请求，直接访问数据库可能导致系统崩溃。

**解决方案：**
将请求先放入消息队列，后端按照处理能力逐步消费，保护数据库。

### 1.4 数据分发

**场景：**
订单状态变更需要通知多个系统（库存、财务、CRM等）。

**解决方案：**
使用发布-订阅模式，一条消息可以被多个消费者处理。

## 2. 主流消息队列对比

| 特性 | RabbitMQ | RocketMQ | Kafka | ActiveMQ | Redis |
|------|----------|----------|-------|----------|-------|
| **性能** | 中等 | 高 | 极高 | 中等 | 高 |
| **可靠性** | 高 | 高 | 高 | 高 | 中等 |
| **功能丰富度** | 高 | 高 | 中等 | 高 | 低 |
| **运维复杂度** | 中等 | 中等 | 高 | 低 | 低 |
| **社区活跃度** | 高 | 高 | 极高 | 中等 | 极高 |
| **适用场景** | 企业级应用 | 大规模分布式 | 大数据处理 | 传统企业 | 轻量级场景 |

## 3. 消息队列选型考虑因素

### 3.1 性能要求

**吞吐量：**
- Kafka：百万级/秒
- RocketMQ：十万级/秒
- RabbitMQ：万级/秒

**延迟：**
- Redis：微秒级
- RabbitMQ：毫秒级
- Kafka：毫秒级

### 3.2 可靠性要求

**消息持久化：**
- 是否需要消息持久化到磁盘
- 消息丢失的容忍度

**消息确认机制：**
- 生产者确认
- 消费者确认
- 事务支持

### 3.3 功能特性

**消息模式：**
- 点对点（Queue）
- 发布订阅（Topic）
- 请求响应（RPC）

**高级特性：**
- 延迟消息
- 消息重试
- 死信队列
- 消息过滤
- 消息顺序

### 3.4 运维成本

**部署复杂度：**
- 集群搭建难度
- 配置复杂程度
- 监控和运维工具

**学习成本：**
- 团队技术栈匹配度
- 文档和社区支持
- 人才储备情况

## 4. 代码实现示例

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.core.KafkaTemplate;
import java.util.concurrent.CompletableFuture;
import java.time.LocalDateTime;

@SpringBootApplication
public class MessageQueueApplication {
    public static void main(String[] args) {
        SpringApplication.run(MessageQueueApplication.class, args);
    }
}

// 1. RabbitMQ 实现示例
@Service
public class RabbitMQService {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    /**
     * 异步处理示例：用户注册
     */
    public void registerUser(UserRegistrationRequest request) {
        try {
            // 1. 保存用户信息到数据库
            User user = saveUserToDatabase(request);
            
            // 2. 发送异步消息
            UserRegistrationEvent event = new UserRegistrationEvent();
            event.setUserId(user.getId());
            event.setEmail(user.getEmail());
            event.setPhone(user.getPhone());
            event.setRegistrationTime(LocalDateTime.now());
            
            // 发送到不同队列处理不同任务
            rabbitTemplate.convertAndSend("user.registration.email", event);
            rabbitTemplate.convertAndSend("user.registration.sms", event);
            rabbitTemplate.convertAndSend("user.registration.analytics", event);
            
            System.out.println("用户注册成功，异步任务已提交: " + user.getId());
        } catch (Exception e) {
            System.err.println("用户注册失败: " + e.getMessage());
            throw new RuntimeException("注册失败", e);
        }
    }
    
    /**
     * 系统解耦示例：订单处理
     */
    public void processOrder(OrderRequest orderRequest) {
        try {
            // 1. 创建订单
            Order order = createOrder(orderRequest);
            
            // 2. 发布订单事件，让各个系统自行处理
            OrderCreatedEvent event = new OrderCreatedEvent();
            event.setOrderId(order.getId());
            event.setUserId(order.getUserId());
            event.setProductId(order.getProductId());
            event.setQuantity(order.getQuantity());
            event.setAmount(order.getAmount());
            
            // 使用Topic Exchange实现发布-订阅
            rabbitTemplate.convertAndSend("order.events", "order.created", event);
            
            System.out.println("订单创建成功: " + order.getId());
        } catch (Exception e) {
            System.err.println("订单处理失败: " + e.getMessage());
            throw new RuntimeException("订单处理失败", e);
        }
    }
    
    /**
     * 流量削峰示例：秒杀处理
     */
    public void handleSeckill(SeckillRequest request) {
        try {
            // 1. 基本校验
            if (!validateSeckillRequest(request)) {
                throw new IllegalArgumentException("请求参数无效");
            }
            
            // 2. 将请求放入队列，避免直接冲击数据库
            SeckillEvent event = new SeckillEvent();
            event.setUserId(request.getUserId());
            event.setProductId(request.getProductId());
            event.setRequestTime(LocalDateTime.now());
            
            rabbitTemplate.convertAndSend("seckill.queue", event);
            
            System.out.println("秒杀请求已提交: " + request.getUserId());
        } catch (Exception e) {
            System.err.println("秒杀请求处理失败: " + e.getMessage());
            throw new RuntimeException("秒杀失败", e);
        }
    }
    
    private User saveUserToDatabase(UserRegistrationRequest request) {
        // 模拟数据库保存
        User user = new User();
        user.setId("USER_" + System.currentTimeMillis());
        user.setEmail(request.getEmail());
        user.setPhone(request.getPhone());
        user.setName(request.getName());
        
        // 模拟数据库操作耗时
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        return user;
    }
    
    private Order createOrder(OrderRequest request) {
        Order order = new Order();
        order.setId("ORDER_" + System.currentTimeMillis());
        order.setUserId(request.getUserId());
        order.setProductId(request.getProductId());
        order.setQuantity(request.getQuantity());
        order.setAmount(request.getAmount());
        order.setStatus("CREATED");
        
        return order;
    }
    
    private boolean validateSeckillRequest(SeckillRequest request) {
        return request != null && 
               request.getUserId() != null && 
               request.getProductId() != null;
    }
}

// 2. RabbitMQ 消费者
@Component
public class RabbitMQConsumer {
    
    /**
     * 邮件发送消费者
     */
    @RabbitListener(queues = "user.registration.email")
    public void handleEmailNotification(UserRegistrationEvent event) {
        try {
            System.out.println("开始发送注册邮件: " + event.getEmail());
            
            // 模拟邮件发送
            Thread.sleep(2000);
            
            System.out.println("注册邮件发送成功: " + event.getEmail());
        } catch (Exception e) {
            System.err.println("邮件发送失败: " + e.getMessage());
            // 可以实现重试机制或发送到死信队列
        }
    }
    
    /**
     * 短信发送消费者
     */
    @RabbitListener(queues = "user.registration.sms")
    public void handleSmsNotification(UserRegistrationEvent event) {
        try {
            System.out.println("开始发送注册短信: " + event.getPhone());
            
            // 模拟短信发送
            Thread.sleep(1000);
            
            System.out.println("注册短信发送成功: " + event.getPhone());
        } catch (Exception e) {
            System.err.println("短信发送失败: " + e.getMessage());
        }
    }
    
    /**
     * 库存扣减消费者
     */
    @RabbitListener(queues = "inventory.update")
    public void handleInventoryUpdate(OrderCreatedEvent event) {
        try {
            System.out.println("开始扣减库存: " + event.getProductId());
            
            // 模拟库存扣减
            boolean success = updateInventory(event.getProductId(), event.getQuantity());
            
            if (success) {
                System.out.println("库存扣减成功: " + event.getProductId());
            } else {
                System.err.println("库存不足: " + event.getProductId());
                // 发送库存不足消息
                handleInventoryShortage(event);
            }
        } catch (Exception e) {
            System.err.println("库存更新失败: " + e.getMessage());
        }
    }
    
    /**
     * 秒杀处理消费者
     */
    @RabbitListener(queues = "seckill.queue")
    public void handleSeckillProcess(SeckillEvent event) {
        try {
            System.out.println("开始处理秒杀: " + event.getUserId());
            
            // 1. 检查库存
            if (!checkSeckillInventory(event.getProductId())) {
                System.out.println("秒杀商品已售罄: " + event.getProductId());
                return;
            }
            
            // 2. 检查用户是否已参与
            if (hasSeckillRecord(event.getUserId(), event.getProductId())) {
                System.out.println("用户已参与秒杀: " + event.getUserId());
                return;
            }
            
            // 3. 创建秒杀订单
            boolean success = createSeckillOrder(event);
            
            if (success) {
                System.out.println("秒杀成功: " + event.getUserId());
            } else {
                System.out.println("秒杀失败: " + event.getUserId());
            }
        } catch (Exception e) {
            System.err.println("秒杀处理异常: " + e.getMessage());
        }
    }
    
    private boolean updateInventory(String productId, int quantity) {
        // 模拟库存更新
        return Math.random() > 0.1; // 90%成功率
    }
    
    private void handleInventoryShortage(OrderCreatedEvent event) {
        // 处理库存不足情况
        System.out.println("处理库存不足: " + event.getOrderId());
    }
    
    private boolean checkSeckillInventory(String productId) {
        // 模拟库存检查
        return Math.random() > 0.3; // 70%有库存
    }
    
    private boolean hasSeckillRecord(String userId, String productId) {
        // 模拟重复参与检查
        return Math.random() > 0.8; // 20%重复参与
    }
    
    private boolean createSeckillOrder(SeckillEvent event) {
        // 模拟订单创建
        try {
            Thread.sleep(100);
            return Math.random() > 0.2; // 80%成功率
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return false;
        }
    }
}

// 3. Kafka 实现示例
@Service
public class KafkaService {
    
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;
    
    /**
     * 大数据处理示例：用户行为日志
     */
    public void logUserBehavior(UserBehaviorLog log) {
        try {
            // 发送到Kafka进行大数据分析
            kafkaTemplate.send("user-behavior-logs", log.getUserId(), log);
            
            System.out.println("用户行为日志已发送: " + log.getUserId());
        } catch (Exception e) {
            System.err.println("日志发送失败: " + e.getMessage());
        }
    }
    
    /**
     * 实时数据流处理
     */
    public void processRealTimeData(RealTimeData data) {
        try {
            // 根据数据类型发送到不同分区
            String topic = "realtime-data";
            String key = data.getDataType() + "_" + data.getTimestamp();
            
            kafkaTemplate.send(topic, key, data);
            
            System.out.println("实时数据已发送: " + data.getDataType());
        } catch (Exception e) {
            System.err.println("实时数据发送失败: " + e.getMessage());
        }
    }
}

@Component
public class KafkaConsumer {
    
    /**
     * 用户行为分析消费者
     */
    @KafkaListener(topics = "user-behavior-logs", groupId = "behavior-analysis-group")
    public void analyzeBehavior(UserBehaviorLog log) {
        try {
            System.out.println("分析用户行为: " + log.getUserId());
            
            // 进行用户行为分析
            analyzeBehaviorPattern(log);
            
            System.out.println("用户行为分析完成: " + log.getUserId());
        } catch (Exception e) {
            System.err.println("行为分析失败: " + e.getMessage());
        }
    }
    
    /**
     * 实时推荐消费者
     */
    @KafkaListener(topics = "user-behavior-logs", groupId = "recommendation-group")
    public void generateRecommendation(UserBehaviorLog log) {
        try {
            System.out.println("生成实时推荐: " + log.getUserId());
            
            // 生成个性化推荐
            generatePersonalizedRecommendation(log);
            
            System.out.println("推荐生成完成: " + log.getUserId());
        } catch (Exception e) {
            System.err.println("推荐生成失败: " + e.getMessage());
        }
    }
    
    private void analyzeBehaviorPattern(UserBehaviorLog log) {
        // 模拟行为分析
        try {
            Thread.sleep(50);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    private void generatePersonalizedRecommendation(UserBehaviorLog log) {
        // 模拟推荐生成
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}

// 4. 事件和实体类
class UserRegistrationEvent {
    private String userId;
    private String email;
    private String phone;
    private LocalDateTime registrationTime;
    
    // getter和setter方法
    public String getUserId() { return userId; }
    public void setUserId(String userId) { this.userId = userId; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public String getPhone() { return phone; }
    public void setPhone(String phone) { this.phone = phone; }
    
    public LocalDateTime getRegistrationTime() { return registrationTime; }
    public void setRegistrationTime(LocalDateTime registrationTime) { this.registrationTime = registrationTime; }
}

class OrderCreatedEvent {
    private String orderId;
    private String userId;
    private String productId;
    private int quantity;
    private double amount;
    
    // getter和setter方法
    public String getOrderId() { return orderId; }
    public void setOrderId(String orderId) { this.orderId = orderId; }
    
    public String getUserId() { return userId; }
    public void setUserId(String userId) { this.userId = userId; }
    
    public String getProductId() { return productId; }
    public void setProductId(String productId) { this.productId = productId; }
    
    public int getQuantity() { return quantity; }
    public void setQuantity(int quantity) { this.quantity = quantity; }
    
    public double getAmount() { return amount; }
    public void setAmount(double amount) { this.amount = amount; }
}

class SeckillEvent {
    private String userId;
    private String productId;
    private LocalDateTime requestTime;
    
    // getter和setter方法
    public String getUserId() { return userId; }
    public void setUserId(String userId) { this.userId = userId; }
    
    public String getProductId() { return productId; }
    public void setProductId(String productId) { this.productId = productId; }
    
    public LocalDateTime getRequestTime() { return requestTime; }
    public void setRequestTime(LocalDateTime requestTime) { this.requestTime = requestTime; }
}

class UserBehaviorLog {
    private String userId;
    private String action;
    private String page;
    private String productId;
    private long timestamp;
    private String dataType;
    
    // getter和setter方法
    public String getUserId() { return userId; }
    public void setUserId(String userId) { this.userId = userId; }
    
    public String getAction() { return action; }
    public void setAction(String action) { this.action = action; }
    
    public String getPage() { return page; }
    public void setPage(String page) { this.page = page; }
    
    public String getProductId() { return productId; }
    public void setProductId(String productId) { this.productId = productId; }
    
    public long getTimestamp() { return timestamp; }
    public void setTimestamp(long timestamp) { this.timestamp = timestamp; }
    
    public String getDataType() { return dataType; }
    public void setDataType(String dataType) { this.dataType = dataType; }
}

// 控制器示例
@RestController
@RequestMapping("/api/mq")
public class MessageQueueController {
    
    @Autowired
    private RabbitMQService rabbitMQService;
    
    @Autowired
    private KafkaService kafkaService;
    
    @PostMapping("/register")
    public String registerUser(@RequestBody UserRegistrationRequest request) {
        rabbitMQService.registerUser(request);
        return "注册成功";
    }
    
    @PostMapping("/order")
    public String createOrder(@RequestBody OrderRequest request) {
        rabbitMQService.processOrder(request);
        return "订单创建成功";
    }
    
    @PostMapping("/seckill")
    public String seckill(@RequestBody SeckillRequest request) {
        rabbitMQService.handleSeckill(request);
        return "秒杀请求已提交";
    }
    
    @PostMapping("/log")
    public String logBehavior(@RequestBody UserBehaviorLog log) {
        kafkaService.logUserBehavior(log);
        return "日志已记录";
    }
}
```

## 5. 选型决策树

```
消息队列选型
├── 性能要求
│   ├── 极高吞吐量 → Kafka
│   ├── 高吞吐量 → RocketMQ
│   └── 中等吞吐量 → RabbitMQ
├── 功能需求
│   ├── 复杂路由 → RabbitMQ
│   ├── 事务消息 → RocketMQ
│   └── 流处理 → Kafka
├── 运维要求
│   ├── 简单运维 → Redis/ActiveMQ
│   ├── 中等复杂度 → RabbitMQ/RocketMQ
│   └── 可接受复杂 → Kafka
└── 团队技术栈
    ├── Java生态 → RocketMQ
    ├── 通用性 → RabbitMQ
    └── 大数据 → Kafka
```

**最佳实践建议：**

1. **合理选型**：根据业务需求和技术栈选择合适的消息队列
2. **消息设计**：设计合理的消息格式和路由策略
3. **可靠性保证**：实现消息确认、重试和死信处理机制
4. **性能优化**：合理设置批量大小、并发度等参数
5. **监控告警**：建立完善的监控和告警体系
6. **容量规划**：根据业务量合理规划集群容量
7. **灾备方案**：制定完善的备份和恢复策略

---

## 5. 架构设计面试题

### 5.1 系统设计类

**Q31: 如何设计一个高并发的秒杀系统？**

**详细答案：**

秒杀系统是典型的高并发、低延迟场景，需要从多个维度进行系统设计和优化。

**秒杀系统核心挑战：**

| 挑战 | 描述 | 影响 | 解决思路 |
|------|------|------|----------|
| 瞬时高并发 | 短时间内大量请求涌入 | 系统崩溃 | 限流、削峰 |
| 库存超卖 | 并发导致库存计算错误 | 业务损失 | 原子操作、预扣库存 |
| 数据库压力 | 大量读写请求冲击数据库 | 性能瓶颈 | 缓存、读写分离 |
| 用户体验 | 响应慢、页面卡顿 | 用户流失 | 静态化、CDN |
| 恶意刷单 | 机器人恶意请求 | 资源浪费 | 防刷、验证码 |

## 1. 整体架构设计

### 1.1 分层架构

```
用户层 → CDN层 → 网关层 → 应用层 → 缓存层 → 数据库层
```

**各层职责：**
- **CDN层**：静态资源缓存，就近访问
- **网关层**：限流、防刷、负载均衡
- **应用层**：业务逻辑处理、库存管理
- **缓存层**：热点数据缓存、库存缓存
- **数据库层**：持久化存储、最终一致性

### 1.2 核心流程设计

```
1. 用户访问 → 静态页面（CDN）
2. 点击秒杀 → 网关限流验证
3. 通过验证 → 库存预检查（Redis）
4. 库存充足 → 异步下单（MQ）
5. 后台处理 → 真实扣库存
6. 结果通知 → 用户获得反馈
```

## 2. 前端优化策略

### 2.1 页面静态化

**目标：**
减少服务器压力，提高页面加载速度。

**实现方案：**
- 商品详情页静态化
- 倒计时前端实现
- 按钮状态前端控制

### 2.2 CDN加速

**配置策略：**
- 静态资源（图片、CSS、JS）全部CDN
- 多级缓存策略
- 就近访问原则

### 2.3 防重复提交

**前端控制：**
- 按钮点击后立即禁用
- 请求去重（Token机制）
- 客户端限流

## 3. 网关层设计

### 3.1 限流策略

**多级限流：**
- **用户级限流**：每个用户每秒最多1次请求
- **IP级限流**：每个IP每秒最多10次请求
- **全局限流**：系统总QPS限制

**限流算法：**
- 令牌桶算法（平滑限流）
- 滑动窗口算法（精确控制）
- 漏桶算法（恒定速率）

### 3.2 防刷机制

**验证策略：**
- 图形验证码
- 短信验证码
- 行为验证（滑块、点击）
- 风控系统（设备指纹、行为分析）

### 3.3 负载均衡

**策略选择：**
- 加权轮询（考虑服务器性能）
- 最少连接数（动态负载）
- 一致性哈希（会话保持）

## 4. 业务层设计

### 4.1 库存管理

**预扣库存机制：**
```
1. Redis预扣库存（原子操作）
2. 异步真实扣库存
3. 定时补偿机制
4. 库存回滚处理
```

**库存同步策略：**
- 定时同步（每秒同步一次）
- 阈值同步（库存变化超过阈值）
- 实时同步（关键操作）

### 4.2 异步处理

**消息队列应用：**
- 订单创建异步化
- 库存扣减异步化
- 支付处理异步化
- 通知发送异步化

### 4.3 数据一致性

**最终一致性保证：**
- 补偿机制
- 重试机制
- 幂等性设计
- 分布式事务（TCC/Saga）

## 5. 存储层优化

### 5.1 Redis缓存设计

**缓存策略：**
- 商品信息缓存（长期）
- 库存信息缓存（实时）
- 用户状态缓存（临时）
- 黑名单缓存（防刷）

**数据结构选择：**
- String：商品基本信息
- Hash：用户购买记录
- Set：参与用户集合
- ZSet：排队队列

### 5.2 数据库优化

**读写分离：**
- 主库：写操作（订单、库存）
- 从库：读操作（查询、统计）
- 延迟处理：异步同步

**分库分表：**
- 按商品ID分表
- 按用户ID分库
- 按时间分表

## 6. 代码实现示例

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.web.bind.annotation.*;
import java.util.concurrent.TimeUnit;
import java.util.Arrays;
import java.time.LocalDateTime;
import java.util.concurrent.CompletableFuture;

@SpringBootApplication
public class SeckillSystemApplication {
    public static void main(String[] args) {
        SpringApplication.run(SeckillSystemApplication.class, args);
    }
}

// 1. 秒杀服务核心实现
@Service
public class SeckillService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    @Autowired
    private SeckillMapper seckillMapper;
    
    // Lua脚本：原子性库存扣减
    private static final String STOCK_LUA_SCRIPT = 
        "local key = KEYS[1]\n" +
        "local quantity = tonumber(ARGV[1])\n" +
        "local stock = tonumber(redis.call('get', key) or 0)\n" +
        "if stock >= quantity then\n" +
        "    redis.call('decrby', key, quantity)\n" +
        "    return stock - quantity\n" +
        "else\n" +
        "    return -1\n" +
        "end";
    
    /**
     * 秒杀核心逻辑
     */
    public SeckillResult seckill(SeckillRequest request) {
        try {
            // 1. 参数校验
            if (!validateSeckillRequest(request)) {
                return SeckillResult.fail("参数无效");
            }
            
            // 2. 用户资格检查
            if (!checkUserEligibility(request.getUserId(), request.getProductId())) {
                return SeckillResult.fail("用户不符合参与条件");
            }
            
            // 3. 防重复提交检查
            if (!checkDuplicateSubmission(request.getUserId(), request.getProductId())) {
                return SeckillResult.fail("请勿重复提交");
            }
            
            // 4. 库存预扣减（Redis原子操作）
            int remainingStock = preDeductStock(request.getProductId(), 1);
            if (remainingStock < 0) {
                return SeckillResult.fail("商品已售罄");
            }
            
            // 5. 生成预订单
            String preOrderId = generatePreOrder(request);
            
            // 6. 异步处理真实订单
            SeckillOrderEvent event = new SeckillOrderEvent();
            event.setPreOrderId(preOrderId);
            event.setUserId(request.getUserId());
            event.setProductId(request.getProductId());
            event.setQuantity(1);
            event.setCreateTime(LocalDateTime.now());
            
            rabbitTemplate.convertAndSend("seckill.order.queue", event);
            
            // 7. 返回成功结果
            return SeckillResult.success(preOrderId, "秒杀成功，正在处理订单");
            
        } catch (Exception e) {
            log.error("秒杀处理异常: {}", e.getMessage(), e);
            return SeckillResult.fail("系统繁忙，请稍后重试");
        }
    }
    
    /**
     * Redis原子性库存扣减
     */
    private int preDeductStock(String productId, int quantity) {
        String stockKey = "seckill:stock:" + productId;
        
        DefaultRedisScript<Long> script = new DefaultRedisScript<>();
        script.setScriptText(STOCK_LUA_SCRIPT);
        script.setResultType(Long.class);
        
        Long result = redisTemplate.execute(script, 
            Arrays.asList(stockKey), 
            String.valueOf(quantity));
        
        return result != null ? result.intValue() : -1;
    }
    
    /**
     * 用户资格检查
     */
    private boolean checkUserEligibility(String userId, String productId) {
        try {
            // 1. 检查用户是否在黑名单
            String blacklistKey = "seckill:blacklist:" + userId;
            if (redisTemplate.hasKey(blacklistKey)) {
                return false;
            }
            
            // 2. 检查用户是否已参与过该商品秒杀
            String participateKey = "seckill:participate:" + productId;
            if (redisTemplate.opsForSet().isMember(participateKey, userId)) {
                return false;
            }
            
            // 3. 检查用户等级限制
            UserInfo userInfo = getUserInfo(userId);
            if (userInfo == null || userInfo.getLevel() < 1) {
                return false;
            }
            
            return true;
        } catch (Exception e) {
            log.error("用户资格检查异常: {}", e.getMessage(), e);
            return false;
        }
    }
    
    /**
     * 防重复提交检查
     */
    private boolean checkDuplicateSubmission(String userId, String productId) {
        String lockKey = "seckill:lock:" + userId + ":" + productId;
        
        // 使用Redis分布式锁防止重复提交
        Boolean lockAcquired = redisTemplate.opsForValue()
            .setIfAbsent(lockKey, "1", 5, TimeUnit.SECONDS);
        
        return lockAcquired != null && lockAcquired;
    }
    
    /**
     * 生成预订单
     */
    private String generatePreOrder(SeckillRequest request) {
        String preOrderId = "PRE_" + System.currentTimeMillis() + "_" + request.getUserId();
        
        // 预订单信息存储到Redis，设置过期时间
        String preOrderKey = "seckill:preorder:" + preOrderId;
        PreOrder preOrder = new PreOrder();
        preOrder.setPreOrderId(preOrderId);
        preOrder.setUserId(request.getUserId());
        preOrder.setProductId(request.getProductId());
        preOrder.setQuantity(1);
        preOrder.setStatus("PENDING");
        preOrder.setCreateTime(LocalDateTime.now());
        
        redisTemplate.opsForValue().set(preOrderKey, preOrder, 10, TimeUnit.MINUTES);
        
        return preOrderId;
    }
    
    /**
     * 库存初始化
     */
    public void initStock(String productId, int stock) {
        String stockKey = "seckill:stock:" + productId;
        redisTemplate.opsForValue().set(stockKey, stock);
        
        log.info("商品库存初始化完成: {} -> {}", productId, stock);
    }
    
    /**
     * 获取当前库存
     */
    public int getCurrentStock(String productId) {
        String stockKey = "seckill:stock:" + productId;
        Object stock = redisTemplate.opsForValue().get(stockKey);
        return stock != null ? Integer.parseInt(stock.toString()) : 0;
    }
    
    private boolean validateSeckillRequest(SeckillRequest request) {
        return request != null && 
               request.getUserId() != null && 
               request.getProductId() != null;
    }
    
    private UserInfo getUserInfo(String userId) {
        // 模拟用户信息获取
        UserInfo userInfo = new UserInfo();
        userInfo.setUserId(userId);
        userInfo.setLevel(2);
        return userInfo;
    }
}

// 2. 异步订单处理
@Component
public class SeckillOrderProcessor {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private InventoryService inventoryService;
    
    /**
     * 处理秒杀订单
     */
    @RabbitListener(queues = "seckill.order.queue")
    public void processSeckillOrder(SeckillOrderEvent event) {
        try {
            log.info("开始处理秒杀订单: {}", event.getPreOrderId());
            
            // 1. 获取预订单信息
            PreOrder preOrder = getPreOrder(event.getPreOrderId());
            if (preOrder == null) {
                log.warn("预订单不存在或已过期: {}", event.getPreOrderId());
                return;
            }
            
            // 2. 真实库存扣减
            boolean stockDeducted = inventoryService.deductStock(
                event.getProductId(), event.getQuantity());
            
            if (!stockDeducted) {
                log.warn("库存扣减失败，回滚Redis库存: {}", event.getProductId());
                rollbackRedisStock(event.getProductId(), event.getQuantity());
                updatePreOrderStatus(event.getPreOrderId(), "FAILED", "库存不足");
                return;
            }
            
            // 3. 创建正式订单
            Order order = createOrder(event, preOrder);
            if (order == null) {
                log.error("订单创建失败，回滚库存: {}", event.getPreOrderId());
                inventoryService.rollbackStock(event.getProductId(), event.getQuantity());
                rollbackRedisStock(event.getProductId(), event.getQuantity());
                updatePreOrderStatus(event.getPreOrderId(), "FAILED", "订单创建失败");
                return;
            }
            
            // 4. 更新预订单状态
            updatePreOrderStatus(event.getPreOrderId(), "SUCCESS", "订单创建成功");
            
            // 5. 记录用户参与状态
            recordUserParticipation(event.getUserId(), event.getProductId());
            
            // 6. 发送成功通知
            sendSuccessNotification(event.getUserId(), order.getOrderId());
            
            log.info("秒杀订单处理完成: {} -> {}", event.getPreOrderId(), order.getOrderId());
            
        } catch (Exception e) {
            log.error("秒杀订单处理异常: {}", e.getMessage(), e);
            
            // 异常情况下的补偿处理
            handleProcessException(event);
        }
    }
    
    private PreOrder getPreOrder(String preOrderId) {
        String preOrderKey = "seckill:preorder:" + preOrderId;
        return (PreOrder) redisTemplate.opsForValue().get(preOrderKey);
    }
    
    private void rollbackRedisStock(String productId, int quantity) {
        String stockKey = "seckill:stock:" + productId;
        redisTemplate.opsForValue().increment(stockKey, quantity);
    }
    
    private void updatePreOrderStatus(String preOrderId, String status, String message) {
        String preOrderKey = "seckill:preorder:" + preOrderId;
        PreOrder preOrder = getPreOrder(preOrderId);
        if (preOrder != null) {
            preOrder.setStatus(status);
            preOrder.setMessage(message);
            preOrder.setUpdateTime(LocalDateTime.now());
            redisTemplate.opsForValue().set(preOrderKey, preOrder, 10, TimeUnit.MINUTES);
        }
    }
    
    private Order createOrder(SeckillOrderEvent event, PreOrder preOrder) {
        try {
            Order order = new Order();
            order.setOrderId("ORDER_" + System.currentTimeMillis());
            order.setUserId(event.getUserId());
            order.setProductId(event.getProductId());
            order.setQuantity(event.getQuantity());
            order.setOrderType("SECKILL");
            order.setStatus("PAID"); // 秒杀订单直接标记为已支付
            order.setCreateTime(LocalDateTime.now());
            
            return orderService.createOrder(order);
        } catch (Exception e) {
            log.error("创建订单异常: {}", e.getMessage(), e);
            return null;
        }
    }
    
    private void recordUserParticipation(String userId, String productId) {
        String participateKey = "seckill:participate:" + productId;
        redisTemplate.opsForSet().add(participateKey, userId);
        redisTemplate.expire(participateKey, 24, TimeUnit.HOURS);
    }
    
    private void sendSuccessNotification(String userId, String orderId) {
        // 发送成功通知（短信、邮件、推送等）
        CompletableFuture.runAsync(() -> {
            try {
                // 模拟通知发送
                Thread.sleep(100);
                log.info("成功通知已发送: {} -> {}", userId, orderId);
            } catch (Exception e) {
                log.error("通知发送失败: {}", e.getMessage(), e);
            }
        });
    }
    
    private void handleProcessException(SeckillOrderEvent event) {
        // 异常补偿处理
        rollbackRedisStock(event.getProductId(), event.getQuantity());
        updatePreOrderStatus(event.getPreOrderId(), "FAILED", "系统异常");
    }
}

// 3. 限流组件
@Component
public class RateLimiter {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    // 令牌桶算法限流
    private static final String TOKEN_BUCKET_LUA = 
        "local key = KEYS[1]\n" +
        "local capacity = tonumber(ARGV[1])\n" +
        "local tokens = tonumber(ARGV[2])\n" +
        "local interval = tonumber(ARGV[3])\n" +
        "local current = redis.call('hmget', key, 'tokens', 'timestamp')\n" +
        "local currentTokens = tonumber(current[1]) or capacity\n" +
        "local lastTime = tonumber(current[2]) or 0\n" +
        "local now = tonumber(redis.call('time')[1])\n" +
        "local elapsed = now - lastTime\n" +
        "local newTokens = math.min(capacity, currentTokens + elapsed * tokens / interval)\n" +
        "if newTokens >= 1 then\n" +
        "    redis.call('hmset', key, 'tokens', newTokens - 1, 'timestamp', now)\n" +
        "    redis.call('expire', key, interval * 2)\n" +
        "    return 1\n" +
        "else\n" +
        "    redis.call('hmset', key, 'tokens', newTokens, 'timestamp', now)\n" +
        "    redis.call('expire', key, interval * 2)\n" +
        "    return 0\n" +
        "end";
    
    /**
     * 用户级限流
     */
    public boolean isUserAllowed(String userId) {
        String key = "rate_limit:user:" + userId;
        return executeTokenBucket(key, 5, 1, 1); // 每秒1个令牌，桶容量5
    }
    
    /**
     * IP级限流
     */
    public boolean isIpAllowed(String ip) {
        String key = "rate_limit:ip:" + ip;
        return executeTokenBucket(key, 50, 10, 1); // 每秒10个令牌，桶容量50
    }
    
    /**
     * 全局限流
     */
    public boolean isGlobalAllowed() {
        String key = "rate_limit:global";
        return executeTokenBucket(key, 10000, 1000, 1); // 每秒1000个令牌
    }
    
    private boolean executeTokenBucket(String key, int capacity, int tokens, int interval) {
        DefaultRedisScript<Long> script = new DefaultRedisScript<>();
        script.setScriptText(TOKEN_BUCKET_LUA);
        script.setResultType(Long.class);
        
        Long result = redisTemplate.execute(script, 
            Arrays.asList(key), 
            String.valueOf(capacity),
            String.valueOf(tokens),
            String.valueOf(interval));
        
        return result != null && result == 1;
    }
}

// 4. 控制器实现
@RestController
@RequestMapping("/api/seckill")
public class SeckillController {
    
    @Autowired
    private SeckillService seckillService;
    
    @Autowired
    private RateLimiter rateLimiter;
    
    /**
     * 秒杀接口
     */
    @PostMapping("/kill")
    public ResponseEntity<SeckillResult> seckill(
            @RequestBody SeckillRequest request,
            HttpServletRequest httpRequest) {
        
        try {
            // 1. 获取用户IP
            String userIp = getClientIp(httpRequest);
            
            // 2. 多级限流检查
            if (!rateLimiter.isGlobalAllowed()) {
                return ResponseEntity.ok(SeckillResult.fail("系统繁忙，请稍后重试"));
            }
            
            if (!rateLimiter.isIpAllowed(userIp)) {
                return ResponseEntity.ok(SeckillResult.fail("请求过于频繁，请稍后重试"));
            }
            
            if (!rateLimiter.isUserAllowed(request.getUserId())) {
                return ResponseEntity.ok(SeckillResult.fail("操作过于频繁，请稍后重试"));
            }
            
            // 3. 执行秒杀逻辑
            SeckillResult result = seckillService.seckill(request);
            
            return ResponseEntity.ok(result);
            
        } catch (Exception e) {
            log.error("秒杀接口异常: {}", e.getMessage(), e);
            return ResponseEntity.ok(SeckillResult.fail("系统异常，请稍后重试"));
        }
    }
    
    /**
     * 查询库存
     */
    @GetMapping("/stock/{productId}")
    public ResponseEntity<Integer> getStock(@PathVariable String productId) {
        int stock = seckillService.getCurrentStock(productId);
        return ResponseEntity.ok(stock);
    }
    
    /**
     * 初始化库存（管理接口）
     */
    @PostMapping("/init-stock")
    public ResponseEntity<String> initStock(
            @RequestParam String productId,
            @RequestParam int stock) {
        
        seckillService.initStock(productId, stock);
        return ResponseEntity.ok("库存初始化成功");
    }
    
    private String getClientIp(HttpServletRequest request) {
        String ip = request.getHeader("X-Forwarded-For");
        if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("Proxy-Client-IP");
        }
        if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("WL-Proxy-Client-IP");
        }
        if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getRemoteAddr();
        }
        return ip;
    }
}

// 5. 实体类定义
class SeckillRequest {
    private String userId;
    private String productId;
    private String token; // 防重复提交token
    
    // getter和setter方法
    public String getUserId() { return userId; }
    public void setUserId(String userId) { this.userId = userId; }
    
    public String getProductId() { return productId; }
    public void setProductId(String productId) { this.productId = productId; }
    
    public String getToken() { return token; }
    public void setToken(String token) { this.token = token; }
}

class SeckillResult {
    private boolean success;
    private String message;
    private String preOrderId;
    private Object data;
    
    public static SeckillResult success(String preOrderId, String message) {
        SeckillResult result = new SeckillResult();
        result.success = true;
        result.preOrderId = preOrderId;
        result.message = message;
        return result;
    }
    
    public static SeckillResult fail(String message) {
        SeckillResult result = new SeckillResult();
        result.success = false;
        result.message = message;
        return result;
    }
    
    // getter和setter方法
    public boolean isSuccess() { return success; }
    public void setSuccess(boolean success) { this.success = success; }
    
    public String getMessage() { return message; }
    public void setMessage(String message) { this.message = message; }
    
    public String getPreOrderId() { return preOrderId; }
    public void setPreOrderId(String preOrderId) { this.preOrderId = preOrderId; }
    
    public Object getData() { return data; }
    public void setData(Object data) { this.data = data; }
}
```

## 7. 性能优化策略

### 7.1 缓存优化

**多级缓存：**
- L1：本地缓存（Caffeine）
- L2：分布式缓存（Redis）
- L3：CDN缓存

**缓存策略：**
- 预热：提前加载热点数据
- 更新：异步更新，避免缓存雪崩
- 降级：缓存失效时的降级策略

### 7.2 数据库优化

**索引优化：**
- 主键索引：订单ID、用户ID
- 复合索引：(product_id, create_time)
- 覆盖索引：减少回表查询

**SQL优化：**
- 批量操作：减少数据库连接
- 分页查询：避免大结果集
- 读写分离：分散数据库压力

### 7.3 系统监控

**关键指标：**
- QPS/TPS：系统吞吐量
- 响应时间：接口性能
- 错误率：系统稳定性
- 库存准确性：业务正确性

**告警机制：**
- 实时监控：关键指标异常告警
- 预警机制：趋势分析预警
- 自动恢复：故障自动处理

**最佳实践总结：**

1. **架构设计**：分层设计，职责清晰
2. **性能优化**：多级缓存，异步处理
3. **数据一致性**：最终一致性，补偿机制
4. **安全防护**：多级限流，防刷机制
5. **监控运维**：全链路监控，快速响应
6. **容量规划**：压力测试，弹性扩容
7. **降级策略**：服务降级，保证核心功能

**Q32: 如何设计一个分布式ID生成器？**

**详细答案：**

分布式ID生成器是分布式系统中的基础组件，需要保证全局唯一性、高性能、高可用性。

**分布式ID核心要求：**

| 要求 | 描述 | 重要性 | 实现难度 |
|------|------|--------|----------|
| 全局唯一 | 在分布式环境下保证ID不重复 | 必须 | 中等 |
| 高性能 | 支持高并发ID生成 | 高 | 中等 |
| 有序性 | ID按时间递增，便于排序和索引 | 高 | 高 |
| 高可用 | 服务不能单点故障 | 高 | 高 |
| 信息安全 | 不能泄露业务信息 | 中等 | 低 |

## 1. 常见方案对比

### 1.1 方案对比表

| 方案 | 优点 | 缺点 | 适用场景 | 性能 |
|------|------|------|----------|------|
| UUID | 简单、本地生成、无依赖 | 无序、长度长、不适合主键 | 非主键场景 | 高 |
| 数据库自增 | 简单、有序、短小 | 性能瓶颈、单点故障 | 小规模系统 | 低 |
| 数据库号段 | 减少数据库访问、有序 | 仍依赖数据库、可能不连续 | 中等规模系统 | 中等 |
| 雪花算法 | 高性能、有序、信息丰富 | 时钟依赖、机器ID管理 | 大规模分布式系统 | 高 |
| Redis | 高性能、简单 | 依赖Redis、可能丢失 | 对连续性要求不高 | 高 |
| Leaf | 双buffer、高可用 | 复杂度高 | 企业级应用 | 高 |

## 2. 雪花算法详解

### 2.1 算法原理

**64位ID结构：**
```
0 - 0000000000 0000000000 0000000000 0000000000 0 - 00000 - 00000 - 000000000000
|   |                                                 |   |       |       |
|   |<-------------- 41位时间戳 ---------------->|   |<-5位->|<-5位->|<-12位->
|   |                                                 |   |       |       |
|   |                                                 |   |       |       +-> 序列号
|   |                                                 |   |       +---------> 机器ID
|   |                                                 |   +-----------------> 数据中心ID
|   |                                                 +---------------------> 保留位
+---+-> 符号位(固定为0)
```

**各部分说明：**
- **1位符号位**：固定为0，保证生成的ID为正数
- **41位时间戳**：毫秒级时间戳，可使用69年
- **5位数据中心ID**：支持32个数据中心
- **5位机器ID**：每个数据中心支持32台机器
- **12位序列号**：同一毫秒内支持4096个ID

### 2.2 性能特点

**理论性能：**
- 单机QPS：409.6万/秒（4096 × 1000）
- 集群QPS：1.3亿/秒（32 × 32 × 409.6万）
- ID长度：19位数字
- 时间范围：69年

## 3. 代码实现示例

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.*;
import java.util.concurrent.atomic.AtomicLong;
import java.time.LocalDateTime;
import java.util.concurrent.ConcurrentHashMap;
import java.util.Map;

@SpringBootApplication
public class DistributedIdApplication {
    public static void main(String[] args) {
        SpringApplication.run(DistributedIdApplication.class, args);
    }
}

// 1. 雪花算法实现
@Component
public class SnowflakeIdGenerator {
    
    // 起始时间戳 (2020-01-01)
    private static final long START_TIMESTAMP = 1577836800000L;
    
    // 各部分位数
    private static final long SEQUENCE_BITS = 12;
    private static final long MACHINE_BITS = 5;
    private static final long DATACENTER_BITS = 5;
    
    // 最大值
    private static final long MAX_SEQUENCE = ~(-1L << SEQUENCE_BITS);
    private static final long MAX_MACHINE_NUM = ~(-1L << MACHINE_BITS);
    private static final long MAX_DATACENTER_NUM = ~(-1L << DATACENTER_BITS);
    
    // 位移
    private static final long MACHINE_LEFT = SEQUENCE_BITS;
    private static final long DATACENTER_LEFT = SEQUENCE_BITS + MACHINE_BITS;
    private static final long TIMESTAMP_LEFT = DATACENTER_LEFT + DATACENTER_BITS;
    
    private final long datacenterId;
    private final long machineId;
    private long sequence = 0L;
    private long lastTimestamp = -1L;
    
    // 同步锁
    private final Object lock = new Object();
    
    public SnowflakeIdGenerator(long datacenterId, long machineId) {
        if (datacenterId > MAX_DATACENTER_NUM || datacenterId < 0) {
            throw new IllegalArgumentException("datacenterId can't be greater than " + MAX_DATACENTER_NUM + " or less than 0");
        }
        if (machineId > MAX_MACHINE_NUM || machineId < 0) {
            throw new IllegalArgumentException("machineId can't be greater than " + MAX_MACHINE_NUM + " or less than 0");
        }
        this.datacenterId = datacenterId;
        this.machineId = machineId;
    }
    
    /**
     * 生成下一个ID
     */
    public long nextId() {
        synchronized (lock) {
            long currentTimestamp = getCurrentTimestamp();
            
            // 时钟回拨检查
            if (currentTimestamp < lastTimestamp) {
                throw new RuntimeException("Clock moved backwards. Refusing to generate id for " + 
                    (lastTimestamp - currentTimestamp) + " milliseconds");
            }
            
            if (currentTimestamp == lastTimestamp) {
                // 同一毫秒内，序列号自增
                sequence = (sequence + 1) & MAX_SEQUENCE;
                if (sequence == 0) {
                    // 序列号溢出，等待下一毫秒
                    currentTimestamp = getNextMillis();
                }
            } else {
                // 不同毫秒内，序列号置为0
                sequence = 0L;
            }
            
            lastTimestamp = currentTimestamp;
            
            // 组装ID
            return ((currentTimestamp - START_TIMESTAMP) << TIMESTAMP_LEFT)
                    | (datacenterId << DATACENTER_LEFT)
                    | (machineId << MACHINE_LEFT)
                    | sequence;
        }
    }
    
    /**
     * 获取下一毫秒
     */
    private long getNextMillis() {
        long mill = getCurrentTimestamp();
        while (mill <= lastTimestamp) {
            mill = getCurrentTimestamp();
        }
        return mill;
    }
    
    /**
     * 获取当前时间戳
     */
    private long getCurrentTimestamp() {
        return System.currentTimeMillis();
    }
    
    /**
     * 解析ID信息
     */
    public IdInfo parseId(long id) {
        long timestamp = (id >> TIMESTAMP_LEFT) + START_TIMESTAMP;
        long datacenterId = (id >> DATACENTER_LEFT) & MAX_DATACENTER_NUM;
        long machineId = (id >> MACHINE_LEFT) & MAX_MACHINE_NUM;
        long sequence = id & MAX_SEQUENCE;
        
        IdInfo info = new IdInfo();
        info.setId(id);
        info.setTimestamp(timestamp);
        info.setDatacenterId(datacenterId);
        info.setMachineId(machineId);
        info.setSequence(sequence);
        info.setDateTime(new java.util.Date(timestamp));
        
        return info;
    }
}

// 2. 号段模式实现
@Component
public class SegmentIdGenerator {
    
    @Autowired
    private IdSegmentMapper segmentMapper;
    
    // 缓存当前号段
    private final Map<String, Segment> segmentCache = new ConcurrentHashMap<>();
    
    // 双buffer机制
    private final Map<String, SegmentBuffer> bufferCache = new ConcurrentHashMap<>();
    
    /**
     * 获取下一个ID
     */
    public long nextId(String bizType) {
        SegmentBuffer buffer = bufferCache.computeIfAbsent(bizType, k -> new SegmentBuffer());
        
        if (!buffer.isReady()) {
            synchronized (buffer) {
                if (!buffer.isReady()) {
                    loadSegment(bizType, buffer);
                }
            }
        }
        
        return buffer.nextId();
    }
    
    /**
     * 加载号段
     */
    private void loadSegment(String bizType, SegmentBuffer buffer) {
        try {
            // 从数据库获取号段
            IdSegment segment = segmentMapper.getAndUpdateSegment(bizType);
            
            if (segment != null) {
                Segment newSegment = new Segment();
                newSegment.setValue(new AtomicLong(segment.getMaxId() - segment.getStep()));
                newSegment.setMax(segment.getMaxId());
                newSegment.setStep(segment.getStep());
                
                buffer.setSegment(newSegment);
                buffer.setReady(true);
                
                log.info("加载号段成功: {} -> [{}, {}]", bizType, 
                    segment.getMaxId() - segment.getStep(), segment.getMaxId());
            }
        } catch (Exception e) {
            log.error("加载号段失败: {}", bizType, e);
            throw new RuntimeException("Failed to load segment for " + bizType, e);
        }
    }
    
    /**
     * 预加载下一个号段
     */
    private void preloadNextSegment(String bizType, SegmentBuffer buffer) {
        CompletableFuture.runAsync(() -> {
            try {
                loadSegment(bizType, buffer);
            } catch (Exception e) {
                log.error("预加载号段失败: {}", bizType, e);
            }
        });
    }
}

// 3. Redis实现
@Component
public class RedisIdGenerator {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    // Lua脚本：原子性获取ID
    private static final String REDIS_LUA_SCRIPT = 
        "local key = KEYS[1]\n" +
        "local step = tonumber(ARGV[1])\n" +
        "local current = redis.call('get', key)\n" +
        "if current == false then\n" +
        "    redis.call('set', key, step)\n" +
        "    return step\n" +
        "else\n" +
        "    return redis.call('incrby', key, step)\n" +
        "end";
    
    /**
     * 获取下一个ID
     */
    public long nextId(String bizType) {
        return nextId(bizType, 1);
    }
    
    /**
     * 批量获取ID
     */
    public long nextId(String bizType, int step) {
        String key = "id_generator:" + bizType;
        
        DefaultRedisScript<Long> script = new DefaultRedisScript<>();
        script.setScriptText(REDIS_LUA_SCRIPT);
        script.setResultType(Long.class);
        
        Long result = redisTemplate.execute(script, 
            Arrays.asList(key), 
            String.valueOf(step));
        
        return result != null ? result : 0L;
    }
    
    /**
     * 重置ID计数器
     */
    public void resetId(String bizType, long value) {
        String key = "id_generator:" + bizType;
        redisTemplate.opsForValue().set(key, value);
    }
    
    /**
     * 获取当前ID值
     */
    public long getCurrentId(String bizType) {
        String key = "id_generator:" + bizType;
        Object value = redisTemplate.opsForValue().get(key);
        return value != null ? Long.parseLong(value.toString()) : 0L;
    }
}

// 4. 统一ID生成服务
@Service
public class IdGeneratorService {
    
    @Autowired
    private SnowflakeIdGenerator snowflakeGenerator;
    
    @Autowired
    private SegmentIdGenerator segmentGenerator;
    
    @Autowired
    private RedisIdGenerator redisGenerator;
    
    /**
     * 根据业务类型选择合适的ID生成策略
     */
    public long generateId(String bizType, IdType idType) {
        switch (idType) {
            case SNOWFLAKE:
                return snowflakeGenerator.nextId();
            case SEGMENT:
                return segmentGenerator.nextId(bizType);
            case REDIS:
                return redisGenerator.nextId(bizType);
            default:
                throw new IllegalArgumentException("Unsupported ID type: " + idType);
        }
    }
    
    /**
     * 批量生成ID
     */
    public List<Long> generateIds(String bizType, IdType idType, int count) {
        List<Long> ids = new ArrayList<>();
        
        if (idType == IdType.REDIS) {
            // Redis支持批量获取
            long endId = redisGenerator.nextId(bizType, count);
            for (int i = 0; i < count; i++) {
                ids.add(endId - count + 1 + i);
            }
        } else {
            // 其他方式逐个生成
            for (int i = 0; i < count; i++) {
                ids.add(generateId(bizType, idType));
            }
        }
        
        return ids;
    }
    
    /**
     * 解析雪花算法ID
     */
    public IdInfo parseSnowflakeId(long id) {
        return snowflakeGenerator.parseId(id);
    }
}

// 5. 控制器实现
@RestController
@RequestMapping("/api/id")
public class IdGeneratorController {
    
    @Autowired
    private IdGeneratorService idGeneratorService;
    
    /**
     * 生成单个ID
     */
    @GetMapping("/generate")
    public ResponseEntity<Long> generateId(
            @RequestParam String bizType,
            @RequestParam(defaultValue = "SNOWFLAKE") IdType idType) {
        
        long id = idGeneratorService.generateId(bizType, idType);
        return ResponseEntity.ok(id);
    }
    
    /**
     * 批量生成ID
     */
    @GetMapping("/batch")
    public ResponseEntity<List<Long>> generateIds(
            @RequestParam String bizType,
            @RequestParam(defaultValue = "SNOWFLAKE") IdType idType,
            @RequestParam(defaultValue = "10") int count) {
        
        if (count > 1000) {
            return ResponseEntity.badRequest().build();
        }
        
        List<Long> ids = idGeneratorService.generateIds(bizType, idType, count);
        return ResponseEntity.ok(ids);
    }
    
    /**
     * 解析雪花算法ID
     */
    @GetMapping("/parse/{id}")
    public ResponseEntity<IdInfo> parseId(@PathVariable long id) {
        IdInfo info = idGeneratorService.parseSnowflakeId(id);
        return ResponseEntity.ok(info);
    }
    
    /**
     * 性能测试
     */
    @GetMapping("/benchmark")
    public ResponseEntity<BenchmarkResult> benchmark(
            @RequestParam(defaultValue = "SNOWFLAKE") IdType idType,
            @RequestParam(defaultValue = "10000") int count) {
        
        long startTime = System.currentTimeMillis();
        
        for (int i = 0; i < count; i++) {
            idGeneratorService.generateId("benchmark", idType);
        }
        
        long endTime = System.currentTimeMillis();
        long duration = endTime - startTime;
        
        BenchmarkResult result = new BenchmarkResult();
        result.setIdType(idType);
        result.setCount(count);
        result.setDuration(duration);
        result.setQps(count * 1000.0 / duration);
        
        return ResponseEntity.ok(result);
    }
}

// 6. 实体类定义
class IdInfo {
    private long id;
    private long timestamp;
    private long datacenterId;
    private long machineId;
    private long sequence;
    private Date dateTime;
    
    // getter和setter方法
    public long getId() { return id; }
    public void setId(long id) { this.id = id; }
    
    public long getTimestamp() { return timestamp; }
    public void setTimestamp(long timestamp) { this.timestamp = timestamp; }
    
    public long getDatacenterId() { return datacenterId; }
    public void setDatacenterId(long datacenterId) { this.datacenterId = datacenterId; }
    
    public long getMachineId() { return machineId; }
    public void setMachineId(long machineId) { this.machineId = machineId; }
    
    public long getSequence() { return sequence; }
    public void setSequence(long sequence) { this.sequence = sequence; }
    
    public Date getDateTime() { return dateTime; }
    public void setDateTime(Date dateTime) { this.dateTime = dateTime; }
}

class Segment {
    private AtomicLong value;
    private long max;
    private int step;
    
    public long nextId() {
        long currentValue = value.incrementAndGet();
        if (currentValue > max) {
            throw new RuntimeException("Segment exhausted");
        }
        return currentValue;
    }
    
    public boolean isExhausted() {
        return value.get() >= max;
    }
    
    // getter和setter方法
    public AtomicLong getValue() { return value; }
    public void setValue(AtomicLong value) { this.value = value; }
    
    public long getMax() { return max; }
    public void setMax(long max) { this.max = max; }
    
    public int getStep() { return step; }
    public void setStep(int step) { this.step = step; }
}

class SegmentBuffer {
    private Segment segment;
    private volatile boolean ready = false;
    
    public long nextId() {
        if (!ready || segment == null) {
            throw new RuntimeException("Segment not ready");
        }
        return segment.nextId();
    }
    
    // getter和setter方法
    public Segment getSegment() { return segment; }
    public void setSegment(Segment segment) { this.segment = segment; }
    
    public boolean isReady() { return ready; }
    public void setReady(boolean ready) { this.ready = ready; }
}

enum IdType {
    SNOWFLAKE, SEGMENT, REDIS, UUID
}

class BenchmarkResult {
    private IdType idType;
    private int count;
    private long duration;
    private double qps;
    
    // getter和setter方法
    public IdType getIdType() { return idType; }
    public void setIdType(IdType idType) { this.idType = idType; }
    
    public int getCount() { return count; }
    public void setCount(int count) { this.count = count; }
    
    public long getDuration() { return duration; }
    public void setDuration(long duration) { this.duration = duration; }
    
    public double getQps() { return qps; }
    public void setQps(double qps) { this.qps = qps; }
}
```

## 4. 高可用设计

### 4.1 机器ID管理

**自动分配策略：**
- ZooKeeper临时节点
- 数据库记录
- 配置文件管理
- 环境变量注入

**冲突检测：**
- 启动时检查
- 运行时监控
- 异常告警

### 4.2 时钟同步

**时钟回拨处理：**
- 拒绝服务（抛异常）
- 等待时钟追上
- 使用备用方案

**NTP同步：**
- 定期同步
- 监控时钟偏移
- 告警机制

### 4.3 容灾设计

**多机房部署：**
- 不同机房使用不同数据中心ID
- 跨机房时钟同步
- 故障切换机制

**降级策略：**
- UUID降级
- 本地时间戳
- 缓存预生成ID

## 5. 性能优化

### 5.1 批量生成

**批量策略：**
- 一次生成多个ID
- 本地缓存
- 异步补充

### 5.2 内存优化

**对象池：**
- 复用ID对象
- 减少GC压力
- 提高性能

### 5.3 并发优化

**无锁设计：**
- CAS操作
- 线程本地存储
- 分段锁

**最佳实践总结：**

1. **方案选择**：根据业务需求选择合适方案
2. **高可用**：避免单点故障，支持故障切换
3. **性能优化**：批量生成，减少锁竞争
4. **监控告警**：关键指标监控，异常及时处理
5. **容灾设计**：多机房部署，降级策略
6. **时钟管理**：NTP同步，时钟回拨处理
7. **扩展性**：支持水平扩展，动态调整

**Q33: 如何保证系统的高可用性？**

**答案要点：**
- **冗余设计**：多副本、多机房
- **故障隔离**：熔断器、舱壁模式
- **快速恢复**：自动故障转移、健康检查
- **降级策略**：核心功能保障

**详细解答：**

#### 1. 高可用性核心指标

| 可用性等级 | 年停机时间 | 月停机时间 | 周停机时间 | 日停机时间 |
|------------|------------|------------|------------|------------|
| 99%        | 3.65天     | 7.31小时   | 1.68小时   | 14.40分钟  |
| 99.9%      | 8.77小时   | 43.83分钟  | 10.08分钟  | 1.44分钟   |
| 99.99%     | 52.60分钟  | 4.38分钟   | 1.01分钟   | 8.64秒     |
| 99.999%    | 5.26分钟   | 26.30秒    | 6.05秒     | 0.86秒     |

#### 2. 高可用架构设计

**2.1 冗余设计**
- **应用层冗余**：多实例部署，无状态设计
- **数据层冗余**：主从复制，多副本存储
- **网络冗余**：多网卡，多链路
- **机房冗余**：多机房部署，异地容灾

**2.2 故障隔离**
- **服务隔离**：微服务架构，独立部署
- **资源隔离**：容器化，资源限制
- **故障域隔离**：机架、机房级别隔离
- **数据隔离**：分库分表，读写分离

**2.3 故障检测与恢复**
- **健康检查**：应用、数据库、中间件监控
- **自动故障转移**：主备切换，负载均衡
- **快速恢复**：预热机制，快速启动
- **回滚机制**：版本管理，快速回退

#### 3. 关键技术实现

**3.1 负载均衡与故障转移**

```java
@Component
public class HighAvailabilityService {
    
    @Autowired
    private List<DataSource> dataSources;
    
    @Autowired
    private CircuitBreakerRegistry circuitBreakerRegistry;
    
    /**
     * 多数据源故障转移
     */
    public <T> T executeWithFailover(String operation, 
                                   Function<DataSource, T> function) {
        for (DataSource dataSource : dataSources) {
            try {
                CircuitBreaker circuitBreaker = circuitBreakerRegistry
                    .circuitBreaker("datasource-" + dataSource.hashCode());
                
                return circuitBreaker.executeSupplier(() -> {
                    return function.apply(dataSource);
                });
            } catch (Exception e) {
                log.warn("DataSource {} failed, trying next", dataSource, e);
                continue;
            }
        }
        throw new RuntimeException("All data sources failed");
    }
    
    /**
     * 健康检查
     */
    @Scheduled(fixedDelay = 30000)
    public void healthCheck() {
        dataSources.parallelStream().forEach(dataSource -> {
            try (Connection conn = dataSource.getConnection()) {
                conn.createStatement().execute("SELECT 1");
                markHealthy(dataSource);
            } catch (Exception e) {
                markUnhealthy(dataSource);
                log.error("Health check failed for {}", dataSource, e);
            }
        });
    }
}
```

**3.2 熔断器模式**

```java
@Component
public class CircuitBreakerService {
    
    private final CircuitBreaker circuitBreaker;
    
    public CircuitBreakerService() {
        this.circuitBreaker = CircuitBreaker.ofDefaults("externalService");
        
        // 配置熔断器
        circuitBreaker.getEventPublisher()
            .onStateTransition(event -> 
                log.info("Circuit breaker state transition: {}", event));
    }
    
    /**
     * 带熔断的服务调用
     */
    public String callExternalService(String request) {
        Supplier<String> decoratedSupplier = CircuitBreaker
            .decorateSupplier(circuitBreaker, () -> {
                // 模拟外部服务调用
                return externalServiceClient.call(request);
            });
        
        try {
            return decoratedSupplier.get();
        } catch (CallNotPermittedException e) {
            // 熔断器开启时的降级处理
            return getFallbackResponse(request);
        }
    }
    
    private String getFallbackResponse(String request) {
        // 降级逻辑：返回缓存数据或默认值
        return cacheService.get(request).orElse("Default response");
    }
}
```

**3.3 服务降级**

```java
@Component
public class DegradationService {
    
    @Value("${system.degradation.enabled:false}")
    private boolean degradationEnabled;
    
    /**
     * 分级降级策略
     */
    public enum DegradationLevel {
        NORMAL(0),      // 正常服务
        LEVEL_1(1),     // 关闭非核心功能
        LEVEL_2(2),     // 只保留核心功能
        LEVEL_3(3);     // 只读模式
        
        private final int level;
        
        DegradationLevel(int level) {
            this.level = level;
        }
    }
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 根据系统负载动态降级
     */
    @Scheduled(fixedDelay = 10000)
    public void checkSystemLoad() {
        SystemMetrics metrics = getSystemMetrics();
        DegradationLevel currentLevel = getCurrentDegradationLevel();
        
        if (metrics.getCpuUsage() > 80 || metrics.getMemoryUsage() > 85) {
            if (currentLevel.level < DegradationLevel.LEVEL_2.level) {
                setDegradationLevel(DegradationLevel.LEVEL_2);
                log.warn("System degraded to level 2 due to high resource usage");
            }
        } else if (metrics.getCpuUsage() < 50 && metrics.getMemoryUsage() < 60) {
            if (currentLevel.level > DegradationLevel.NORMAL.level) {
                setDegradationLevel(DegradationLevel.NORMAL);
                log.info("System restored to normal level");
            }
        }
    }
    
    /**
     * 功能开关
     */
    public boolean isFeatureEnabled(String featureName) {
        DegradationLevel level = getCurrentDegradationLevel();
        
        switch (level) {
            case LEVEL_1:
                return !NON_CORE_FEATURES.contains(featureName);
            case LEVEL_2:
                return CORE_FEATURES.contains(featureName);
            case LEVEL_3:
                return READ_ONLY_FEATURES.contains(featureName);
            default:
                return true;
        }
    }
}
```

**3.4 数据一致性保障**

```java
@Component
public class DataConsistencyService {
    
    /**
     * 最终一致性实现
     */
    @Transactional
    public void updateWithEventualConsistency(Long id, Object data) {
        try {
            // 1. 更新主数据库
            primaryDataSource.update(id, data);
            
            // 2. 发送同步消息
            DataSyncEvent event = new DataSyncEvent(id, data, System.currentTimeMillis());
            messageProducer.send("data-sync-topic", event);
            
        } catch (Exception e) {
            // 3. 失败重试机制
            retryTemplate.execute(context -> {
                compensateDataSync(id, data);
                return null;
            });
        }
    }
    
    /**
     * 数据补偿机制
     */
    @RabbitListener(queues = "data-sync-queue")
    public void handleDataSync(DataSyncEvent event) {
        try {
            // 同步到从数据库
            replicaDataSource.update(event.getId(), event.getData());
            
            // 更新缓存
            cacheService.put(event.getId(), event.getData());
            
        } catch (Exception e) {
            // 重试或记录失败日志
            if (event.getRetryCount() < MAX_RETRY_COUNT) {
                event.incrementRetryCount();
                messageProducer.send("data-sync-retry-topic", event);
            } else {
                log.error("Data sync failed after max retries: {}", event, e);
            }
        }
    }
}
```

#### 4. 监控与告警

**4.1 关键指标监控**

```java
@Component
public class AvailabilityMonitor {
    
    private final MeterRegistry meterRegistry;
    private final Counter requestCounter;
    private final Timer responseTimer;
    private final Gauge systemLoadGauge;
    
    public AvailabilityMonitor(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        this.requestCounter = Counter.builder("requests.total")
            .description("Total requests")
            .register(meterRegistry);
        this.responseTimer = Timer.builder("response.time")
            .description("Response time")
            .register(meterRegistry);
        this.systemLoadGauge = Gauge.builder("system.load")
            .description("System load")
            .register(meterRegistry, this, AvailabilityMonitor::getSystemLoad);
    }
    
    /**
     * SLA计算
     */
    @Scheduled(fixedDelay = 60000)
    public void calculateSLA() {
        long totalRequests = requestCounter.count();
        long successRequests = getSuccessRequestCount();
        
        double availability = (double) successRequests / totalRequests * 100;
        
        meterRegistry.gauge("sla.availability", availability);
        
        if (availability < SLA_THRESHOLD) {
            alertService.sendAlert("SLA breach detected: " + availability + "%");
        }
    }
}
```

#### 5. 最佳实践

**5.1 设计原则**
- **无单点故障**：任何组件都不应成为单点
- **故障隔离**：故障不应传播到其他组件
- **快速恢复**：故障检测和恢复时间最小化
- **优雅降级**：在部分功能不可用时保持核心服务

**5.2 运维实践**
- **自动化部署**：减少人为错误
- **灰度发布**：降低发布风险
- **容量规划**：预留足够的冗余容量
- **演练测试**：定期进行故障演练

**5.3 监控告警**
- **多层次监控**：基础设施、应用、业务层面
- **智能告警**：避免告警风暴，精准定位问题
- **可观测性**：日志、指标、链路追踪三位一体
- **自动恢复**：能自动处理的故障不依赖人工干预

### 5.2 性能优化类

**Q34: 系统性能优化的思路和方法？**

**答案要点：**
- **前端优化**：CDN、压缩、缓存
- **应用优化**：代码优化、JVM调优、连接池
- **数据库优化**：索引、SQL优化、分库分表
- **架构优化**：缓存、异步、负载均衡

**详细解答：**

#### 1. 性能优化层次结构

| 优化层次 | 优化重点 | 性能提升 | 实施难度 | 成本投入 |
|----------|----------|----------|----------|----------|
| 前端优化 | 用户体验 | 20-50% | 低 | 低 |
| 应用优化 | 代码效率 | 30-80% | 中 | 中 |
| 数据库优化 | 查询性能 | 50-200% | 中高 | 中 |
| 架构优化 | 系统设计 | 100-500% | 高 | 高 |
| 基础设施 | 硬件资源 | 50-100% | 低 | 高 |

#### 2. 前端性能优化

**2.1 资源优化**
- **文件压缩**：Gzip、Brotli压缩
- **图片优化**：WebP格式、懒加载、响应式图片
- **代码分割**：按需加载、Tree Shaking
- **缓存策略**：浏览器缓存、Service Worker

**2.2 网络优化**
- **CDN加速**：就近访问、边缘计算
- **HTTP/2**：多路复用、服务器推送
- **DNS优化**：DNS预解析、DNS缓存
- **连接优化**：Keep-Alive、连接复用

#### 3. 应用层性能优化

**3.1 代码优化**

```java
@Component
public class PerformanceOptimizationService {
    
    /**
     * 批量处理优化
     */
    public void batchProcessOptimization() {
        // 错误做法：逐条处理
        // for (User user : users) {
        //     userService.updateUser(user);
        // }
        
        // 正确做法：批量处理
        List<User> users = getUsersToUpdate();
        userService.batchUpdateUsers(users);
    }
    
    /**
     * 缓存优化
     */
    @Cacheable(value = "userCache", key = "#userId")
    public User getUserById(Long userId) {
        return userRepository.findById(userId);
    }
    
    /**
     * 异步处理优化
     */
    @Async("taskExecutor")
    public CompletableFuture<Void> processAsyncTask(String data) {
        // 耗时操作异步处理
        heavyProcessing(data);
        return CompletableFuture.completedFuture(null);
    }
    
    /**
     * 对象池优化
     */
    @Autowired
    private ObjectPool<ExpensiveObject> objectPool;
    
    public void useObjectPool() {
        ExpensiveObject obj = null;
        try {
            obj = objectPool.borrowObject();
            // 使用对象
            obj.doSomething();
        } finally {
            if (obj != null) {
                objectPool.returnObject(obj);
            }
        }
    }
}
```

**3.2 JVM调优**

```java
@Component
public class JVMTuningService {
    
    /**
     * 内存使用监控
     */
    @EventListener
    public void monitorMemoryUsage(ApplicationReadyEvent event) {
        MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
        
        // 堆内存使用情况
        MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
        log.info("Heap Memory - Used: {}MB, Max: {}MB, Usage: {}%",
            heapUsage.getUsed() / 1024 / 1024,
            heapUsage.getMax() / 1024 / 1024,
            (double) heapUsage.getUsed() / heapUsage.getMax() * 100);
        
        // GC信息
        List<GarbageCollectorMXBean> gcBeans = ManagementFactory.getGarbageCollectorMXBeans();
        for (GarbageCollectorMXBean gcBean : gcBeans) {
            log.info("GC {} - Collections: {}, Time: {}ms",
                gcBean.getName(),
                gcBean.getCollectionCount(),
                gcBean.getCollectionTime());
        }
    }
    
    /**
     * 线程池优化配置
     */
    @Bean("optimizedTaskExecutor")
    public TaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        
        // 核心线程数 = CPU核心数
        executor.setCorePoolSize(Runtime.getRuntime().availableProcessors());
        // 最大线程数 = CPU核心数 * 2
        executor.setMaxPoolSize(Runtime.getRuntime().availableProcessors() * 2);
        // 队列容量
        executor.setQueueCapacity(1000);
        // 线程名前缀
        executor.setThreadNamePrefix("Optimized-");
        // 拒绝策略
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        
        executor.initialize();
        return executor;
    }
}
```

**3.3 连接池优化**

```java
@Configuration
public class ConnectionPoolConfig {
    
    /**
     * 数据库连接池优化
     */
    @Bean
    @Primary
    public DataSource optimizedDataSource() {
        HikariConfig config = new HikariConfig();
        
        // 基本配置
        config.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        config.setUsername("root");
        config.setPassword("password");
        
        // 连接池优化配置
        config.setMinimumIdle(10);                    // 最小空闲连接
        config.setMaximumPoolSize(50);                // 最大连接数
        config.setConnectionTimeout(30000);           // 连接超时时间
        config.setIdleTimeout(600000);                // 空闲超时时间
        config.setMaxLifetime(1800000);               // 连接最大生命周期
        config.setLeakDetectionThreshold(60000);      // 连接泄漏检测
        
        // 性能优化配置
        config.addDataSourceProperty("cachePrepStmts", "true");
        config.addDataSourceProperty("prepStmtCacheSize", "250");
        config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");
        config.addDataSourceProperty("useServerPrepStmts", "true");
        
        return new HikariDataSource(config);
    }
    
    /**
     * Redis连接池优化
     */
    @Bean
    public JedisConnectionFactory jedisConnectionFactory() {
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        
        // 连接池配置
        poolConfig.setMaxTotal(200);                  // 最大连接数
        poolConfig.setMaxIdle(50);                    // 最大空闲连接
        poolConfig.setMinIdle(10);                    // 最小空闲连接
        poolConfig.setMaxWaitMillis(3000);            // 最大等待时间
        poolConfig.setTestOnBorrow(true);             // 借用时测试
        poolConfig.setTestOnReturn(true);             // 归还时测试
        poolConfig.setTestWhileIdle(true);            // 空闲时测试
        
        RedisStandaloneConfiguration config = new RedisStandaloneConfiguration();
        config.setHostName("localhost");
        config.setPort(6379);
        
        JedisConnectionFactory factory = new JedisConnectionFactory(config);
        factory.setPoolConfig(poolConfig);
        
        return factory;
    }
}
```

#### 4. 数据库性能优化

**4.1 索引优化**

```java
@Repository
public class OptimizedUserRepository {
    
    /**
     * 复合索引优化查询
     */
    @Query("SELECT u FROM User u WHERE u.status = :status " +
           "AND u.createTime >= :startTime " +
           "ORDER BY u.createTime DESC")
    List<User> findActiveUsersWithIndex(@Param("status") String status,
                                       @Param("startTime") LocalDateTime startTime,
                                       Pageable pageable);
    
    /**
     * 避免SELECT *，只查询需要的字段
     */
    @Query("SELECT new com.example.dto.UserSummary(u.id, u.name, u.email) " +
           "FROM User u WHERE u.status = 'ACTIVE'")
    List<UserSummary> findActiveUserSummaries();
    
    /**
     * 批量查询优化
     */
    @Query("SELECT u FROM User u WHERE u.id IN :ids")
    List<User> findByIds(@Param("ids") List<Long> ids);
}
```

**4.2 SQL优化**

```sql
-- 创建优化索引
CREATE INDEX idx_user_status_time ON user(status, create_time);
CREATE INDEX idx_order_user_status ON order_info(user_id, status);

-- 优化前：关联查询
SELECT u.*, o.* 
FROM user u 
LEFT JOIN order_info o ON u.id = o.user_id 
WHERE u.status = 'ACTIVE';

-- 优化后：分步查询
SELECT id, name, email FROM user WHERE status = 'ACTIVE';
SELECT order_no, amount FROM order_info WHERE user_id IN (1,2,3,...);

-- 分页优化
-- 优化前：OFFSET性能差
SELECT * FROM user ORDER BY id LIMIT 10000, 20;

-- 优化后：基于ID的分页
SELECT * FROM user WHERE id > 10000 ORDER BY id LIMIT 20;
```

**4.3 读写分离**

```java
@Component
public class ReadWriteSplitService {
    
    @Autowired
    @Qualifier("masterDataSource")
    private DataSource masterDataSource;
    
    @Autowired
    @Qualifier("slaveDataSource")
    private DataSource slaveDataSource;
    
    /**
     * 动态数据源路由
     */
    public class DynamicDataSource extends AbstractRoutingDataSource {
        
        @Override
        protected Object determineCurrentLookupKey() {
            return DataSourceContextHolder.getDataSourceType();
        }
    }
    
    /**
     * 读操作使用从库
     */
    @Transactional(readOnly = true)
    public List<User> findUsers() {
        DataSourceContextHolder.setDataSourceType(DataSourceType.SLAVE);
        try {
            return userRepository.findAll();
        } finally {
            DataSourceContextHolder.clearDataSourceType();
        }
    }
    
    /**
     * 写操作使用主库
     */
    @Transactional
    public User saveUser(User user) {
        DataSourceContextHolder.setDataSourceType(DataSourceType.MASTER);
        try {
            return userRepository.save(user);
        } finally {
            DataSourceContextHolder.clearDataSourceType();
        }
    }
}
```

#### 5. 架构层性能优化

**5.1 缓存策略**

```java
@Service
public class CacheOptimizationService {
    
    /**
     * 多级缓存
     */
    public User getUserWithMultiLevelCache(Long userId) {
        // L1: 本地缓存
        User user = localCache.get(userId);
        if (user != null) {
            return user;
        }
        
        // L2: Redis缓存
        user = redisTemplate.opsForValue().get("user:" + userId);
        if (user != null) {
            localCache.put(userId, user);
            return user;
        }
        
        // L3: 数据库
        user = userRepository.findById(userId);
        if (user != null) {
            redisTemplate.opsForValue().set("user:" + userId, user, Duration.ofMinutes(30));
            localCache.put(userId, user);
        }
        
        return user;
    }
    
    /**
     * 缓存预热
     */
    @EventListener(ApplicationReadyEvent.class)
    public void warmUpCache() {
        List<User> hotUsers = userRepository.findHotUsers();
        for (User user : hotUsers) {
            redisTemplate.opsForValue().set("user:" + user.getId(), user, Duration.ofHours(1));
        }
    }
}
```

**5.2 异步处理**

```java
@Service
public class AsyncProcessingService {
    
    /**
     * 消息队列异步处理
     */
    @RabbitListener(queues = "heavy-task-queue")
    public void processHeavyTask(HeavyTaskMessage message) {
        // 异步处理耗时任务
        heavyTaskProcessor.process(message);
    }
    
    /**
     * 事件驱动异步处理
     */
    @EventListener
    @Async
    public void handleUserRegistration(UserRegistrationEvent event) {
        // 发送欢迎邮件
        emailService.sendWelcomeEmail(event.getUser());
        
        // 初始化用户数据
        userDataService.initializeUserData(event.getUser());
        
        // 记录用户行为
        analyticsService.trackUserRegistration(event.getUser());
    }
}
```

#### 6. 性能监控与分析

**6.1 性能指标监控**

```java
@Component
public class PerformanceMonitor {
    
    private final MeterRegistry meterRegistry;
    
    /**
     * 方法执行时间监控
     */
    @Around("@annotation(Monitored)")
    public Object monitorPerformance(ProceedingJoinPoint joinPoint) throws Throwable {
        Timer.Sample sample = Timer.start(meterRegistry);
        String methodName = joinPoint.getSignature().getName();
        
        try {
            Object result = joinPoint.proceed();
            sample.stop(Timer.builder("method.execution.time")
                .tag("method", methodName)
                .tag("status", "success")
                .register(meterRegistry));
            return result;
        } catch (Exception e) {
            sample.stop(Timer.builder("method.execution.time")
                .tag("method", methodName)
                .tag("status", "error")
                .register(meterRegistry));
            throw e;
        }
    }
    
    /**
     * 系统资源监控
     */
    @Scheduled(fixedDelay = 30000)
    public void monitorSystemResources() {
        // CPU使用率
        double cpuUsage = systemMetrics.getCpuUsage();
        meterRegistry.gauge("system.cpu.usage", cpuUsage);
        
        // 内存使用率
        double memoryUsage = systemMetrics.getMemoryUsage();
        meterRegistry.gauge("system.memory.usage", memoryUsage);
        
        // 磁盘IO
        double diskIO = systemMetrics.getDiskIO();
        meterRegistry.gauge("system.disk.io", diskIO);
    }
}
```

#### 7. 性能优化最佳实践

**7.1 优化原则**
- **测量优先**：先测量再优化，避免过早优化
- **瓶颈定位**：找到真正的性能瓶颈
- **渐进优化**：逐步优化，避免大幅改动
- **效果验证**：优化后验证效果

**7.2 常见性能陷阱**
- **N+1查询问题**：使用JOIN或批量查询解决
- **大事务**：拆分事务，减少锁定时间
- **频繁GC**：优化对象创建，调整堆大小
- **连接池耗尽**：合理配置连接池参数

**7.3 性能测试**
- **压力测试**：模拟高并发场景
- **基准测试**：建立性能基线
- **容量规划**：预估系统容量需求
- **持续监控**：生产环境性能监控

**Q35: 数据库查询慢的排查和优化方法？**

**答案要点：**
- **排查方法**：慢查询日志、执行计划分析
- **索引优化**：创建合适索引、避免索引失效
- **SQL优化**：避免全表扫描、优化JOIN
- **架构优化**：读写分离、分库分表

**详细解答：**

#### 1. 慢查询排查流程

| 排查步骤 | 工具/方法 | 关注指标 | 优化方向 |
|----------|-----------|----------|----------|
| 1. 发现问题 | 监控告警 | 响应时间、QPS | 定位慢查询 |
| 2. 定位SQL | 慢查询日志 | 执行时间、频次 | 找出问题SQL |
| 3. 分析执行计划 | EXPLAIN | 扫描行数、索引使用 | 优化执行路径 |
| 4. 检查索引 | SHOW INDEX | 索引覆盖、选择性 | 优化索引设计 |
| 5. 优化SQL | 重写查询 | 逻辑优化 | 提升执行效率 |
| 6. 验证效果 | 性能测试 | 执行时间对比 | 确认优化效果 |

#### 2. 慢查询排查工具

**2.1 MySQL慢查询日志配置**

```sql
-- 开启慢查询日志
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow.log';
SET GLOBAL long_query_time = 1; -- 超过1秒的查询记录
SET GLOBAL log_queries_not_using_indexes = 'ON'; -- 记录未使用索引的查询

-- 查看慢查询配置
SHOW VARIABLES LIKE '%slow%';
SHOW VARIABLES LIKE '%long_query_time%';
```

**2.2 执行计划分析**

```java
@Repository
public class SlowQueryAnalyzer {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    /**
     * 分析SQL执行计划
     */
    public void analyzeQueryPlan(String sql) {
        String explainSql = "EXPLAIN FORMAT=JSON " + sql;
        
        List<Map<String, Object>> result = jdbcTemplate.queryForList(explainSql);
        
        for (Map<String, Object> row : result) {
            log.info("Execution Plan: {}", row);
            
            // 分析关键指标
            analyzeExecutionPlan(row);
        }
    }
    
    private void analyzeExecutionPlan(Map<String, Object> plan) {
        // 检查是否使用索引
        String type = (String) plan.get("type");
        if ("ALL".equals(type)) {
            log.warn("Full table scan detected!");
        }
        
        // 检查扫描行数
        Long rows = (Long) plan.get("rows");
        if (rows != null && rows > 10000) {
            log.warn("Large number of rows scanned: {}", rows);
        }
        
        // 检查是否使用临时表
        String extra = (String) plan.get("Extra");
        if (extra != null && extra.contains("Using temporary")) {
            log.warn("Using temporary table detected!");
        }
    }
}
```

**2.3 性能监控**

```java
@Component
public class DatabasePerformanceMonitor {
    
    @Autowired
    private DataSource dataSource;
    
    /**
     * 监控数据库性能指标
     */
    @Scheduled(fixedDelay = 60000)
    public void monitorDatabasePerformance() {
        try (Connection conn = dataSource.getConnection();
             Statement stmt = conn.createStatement()) {
            
            // 查询当前运行的慢查询
            ResultSet rs = stmt.executeQuery(
                "SELECT * FROM information_schema.processlist " +
                "WHERE time > 5 AND command = 'Query'");
            
            while (rs.next()) {
                log.warn("Long running query detected: ID={}, Time={}s, Query={}",
                    rs.getLong("id"),
                    rs.getInt("time"),
                    rs.getString("info"));
            }
            
            // 查询锁等待情况
            rs = stmt.executeQuery(
                "SELECT * FROM information_schema.innodb_locks");
            
            while (rs.next()) {
                log.warn("Lock detected: {}", rs.getString("lock_id"));
            }
            
        } catch (SQLException e) {
            log.error("Database monitoring failed", e);
        }
    }
    
    /**
     * 分析表统计信息
     */
    public void analyzeTableStats(String tableName) {
        String sql = "SELECT " +
                    "table_name, " +
                    "table_rows, " +
                    "avg_row_length, " +
                    "data_length, " +
                    "index_length " +
                    "FROM information_schema.tables " +
                    "WHERE table_name = ?";
        
        jdbcTemplate.query(sql, rs -> {
            log.info("Table Stats - Name: {}, Rows: {}, Avg Row Length: {}, " +
                    "Data Size: {}MB, Index Size: {}MB",
                rs.getString("table_name"),
                rs.getLong("table_rows"),
                rs.getLong("avg_row_length"),
                rs.getLong("data_length") / 1024 / 1024,
                rs.getLong("index_length") / 1024 / 1024);
        }, tableName);
    }
}
```

#### 3. 索引优化策略

**3.1 索引设计原则**

```java
@Entity
@Table(name = "user_order", indexes = {
    // 单列索引
    @Index(name = "idx_user_id", columnList = "user_id"),
    @Index(name = "idx_status", columnList = "status"),
    
    // 复合索引（注意字段顺序）
    @Index(name = "idx_user_status_time", columnList = "user_id, status, create_time"),
    
    // 覆盖索引
    @Index(name = "idx_cover_query", columnList = "user_id, status, order_amount")
})
public class UserOrder {
    
    @Id
    private Long id;
    
    @Column(name = "user_id")
    private Long userId;
    
    @Column(name = "status")
    private String status;
    
    @Column(name = "order_amount")
    private BigDecimal orderAmount;
    
    @Column(name = "create_time")
    private LocalDateTime createTime;
}
```

**3.2 索引优化实践**

```sql
-- 分析索引使用情况
SELECT 
    table_name,
    index_name,
    cardinality,
    sub_part,
    packed,
    nullable,
    index_type
FROM information_schema.statistics 
WHERE table_schema = 'your_database'
ORDER BY table_name, seq_in_index;

-- 查找未使用的索引
SELECT 
    object_schema,
    object_name,
    index_name
FROM performance_schema.table_io_waits_summary_by_index_usage 
WHERE index_name IS NOT NULL 
AND count_star = 0 
AND object_schema != 'mysql'
ORDER BY object_schema, object_name;

-- 创建优化索引
CREATE INDEX idx_user_order_optimized 
ON user_order(user_id, status, create_time DESC);

-- 删除重复索引
DROP INDEX idx_duplicate ON user_order;
```

**3.3 索引失效场景**

```java
@Repository
public class IndexOptimizedRepository {
    
    /**
     * 正确的索引使用
     */
    @Query("SELECT u FROM UserOrder u WHERE u.userId = :userId " +
           "AND u.status = :status " +
           "AND u.createTime >= :startTime " +
           "ORDER BY u.createTime DESC")
    List<UserOrder> findOrdersOptimized(@Param("userId") Long userId,
                                       @Param("status") String status,
                                       @Param("startTime") LocalDateTime startTime);
    
    /**
     * 避免索引失效的查询
     */
    @Query("SELECT u FROM UserOrder u WHERE u.userId = :userId " +
           "AND u.orderAmount >= :minAmount")  // 避免函数操作
    List<UserOrder> findOrdersByAmount(@Param("userId") Long userId,
                                      @Param("minAmount") BigDecimal minAmount);
    
    /**
     * 使用IN替代OR
     */
    @Query("SELECT u FROM UserOrder u WHERE u.status IN :statuses")
    List<UserOrder> findOrdersByStatuses(@Param("statuses") List<String> statuses);
}
```

#### 4. SQL优化技巧

**4.1 查询重写**

```sql
-- 优化前：子查询
SELECT * FROM user u 
WHERE u.id IN (
    SELECT DISTINCT o.user_id 
    FROM order_info o 
    WHERE o.status = 'COMPLETED'
);

-- 优化后：JOIN
SELECT DISTINCT u.* 
FROM user u 
INNER JOIN order_info o ON u.id = o.user_id 
WHERE o.status = 'COMPLETED';

-- 优化前：复杂WHERE条件
SELECT * FROM order_info 
WHERE DATE(create_time) = '2024-01-01';

-- 优化后：范围查询
SELECT * FROM order_info 
WHERE create_time >= '2024-01-01 00:00:00' 
AND create_time < '2024-01-02 00:00:00';

-- 优化前：LIKE前缀通配符
SELECT * FROM user WHERE name LIKE '%张%';

-- 优化后：全文索引
SELECT * FROM user WHERE MATCH(name) AGAINST('张' IN NATURAL LANGUAGE MODE);
```

**4.2 分页优化**

```java
@Repository
public class PaginationOptimizedRepository {
    
    /**
     * 传统分页（性能差）
     */
    @Query("SELECT u FROM User u ORDER BY u.id LIMIT :offset, :size")
    List<User> findUsersWithOffset(@Param("offset") int offset, 
                                  @Param("size") int size);
    
    /**
     * 游标分页（性能好）
     */
    @Query("SELECT u FROM User u WHERE u.id > :lastId ORDER BY u.id LIMIT :size")
    List<User> findUsersWithCursor(@Param("lastId") Long lastId, 
                                  @Param("size") int size);
    
    /**
     * 延迟关联优化大偏移量分页
     */
    @Query(value = "SELECT u.* FROM user u " +
                  "INNER JOIN (" +
                  "  SELECT id FROM user ORDER BY id LIMIT :offset, :size" +
                  ") t ON u.id = t.id", nativeQuery = true)
    List<User> findUsersWithDelayedJoin(@Param("offset") int offset, 
                                       @Param("size") int size);
}
```

**4.3 批量操作优化**

```java
@Service
public class BatchOptimizationService {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    /**
     * 批量插入优化
     */
    public void batchInsertUsers(List<User> users) {
        String sql = "INSERT INTO user (name, email, phone) VALUES (?, ?, ?)";
        
        jdbcTemplate.batchUpdate(sql, new BatchPreparedStatementSetter() {
            @Override
            public void setValues(PreparedStatement ps, int i) throws SQLException {
                User user = users.get(i);
                ps.setString(1, user.getName());
                ps.setString(2, user.getEmail());
                ps.setString(3, user.getPhone());
            }
            
            @Override
            public int getBatchSize() {
                return users.size();
            }
        });
    }
    
    /**
     * 批量更新优化
     */
    public void batchUpdateUserStatus(List<Long> userIds, String status) {
        // 使用CASE WHEN进行批量更新
        StringBuilder sql = new StringBuilder("UPDATE user SET status = CASE id ");
        
        for (Long userId : userIds) {
            sql.append("WHEN ").append(userId).append(" THEN '").append(status).append("' ");
        }
        
        sql.append("END WHERE id IN (");
        sql.append(userIds.stream().map(String::valueOf).collect(Collectors.joining(",")));
        sql.append(")");
        
        jdbcTemplate.update(sql.toString());
    }
}
```

#### 5. 架构层优化

**5.1 读写分离**

```java
@Configuration
public class ReadWriteSplitConfig {
    
    @Bean
    @Primary
    public DataSource routingDataSource() {
        Map<Object, Object> dataSourceMap = new HashMap<>();
        dataSourceMap.put(DataSourceType.MASTER, masterDataSource());
        dataSourceMap.put(DataSourceType.SLAVE, slaveDataSource());
        
        DynamicDataSource routingDataSource = new DynamicDataSource();
        routingDataSource.setDefaultTargetDataSource(masterDataSource());
        routingDataSource.setTargetDataSources(dataSourceMap);
        
        return routingDataSource;
    }
    
    @Bean
    public DataSource masterDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://master:3306/db")
            .username("root")
            .password("password")
            .build();
    }
    
    @Bean
    public DataSource slaveDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://slave:3306/db")
            .username("root")
            .password("password")
            .build();
    }
}

@Aspect
@Component
public class DataSourceAspect {
    
    @Before("@annotation(readOnly)")
    public void setReadDataSource(ReadOnly readOnly) {
        DataSourceContextHolder.setDataSourceType(DataSourceType.SLAVE);
    }
    
    @Before("@annotation(Transactional) && !@annotation(ReadOnly)")
    public void setWriteDataSource() {
        DataSourceContextHolder.setDataSourceType(DataSourceType.MASTER);
    }
    
    @After("@annotation(ReadOnly) || @annotation(Transactional)")
    public void clearDataSource() {
        DataSourceContextHolder.clearDataSourceType();
    }
}
```

**5.2 分库分表**

```java
@Component
public class ShardingStrategy {
    
    /**
     * 用户表分片策略
     */
    public String determineUserTableName(Long userId) {
        int shardIndex = (int) (userId % 4);
        return "user_" + shardIndex;
    }
    
    /**
     * 订单表分片策略（按时间分片）
     */
    public String determineOrderTableName(LocalDateTime createTime) {
        String yearMonth = createTime.format(DateTimeFormatter.ofPattern("yyyyMM"));
        return "order_" + yearMonth;
    }
    
    /**
     * 数据库分片策略
     */
    public String determineDatabaseName(Long userId) {
        int dbIndex = (int) (userId % 2);
        return "db_" + dbIndex;
    }
}

@Repository
public class ShardingUserRepository {
    
    @Autowired
    private ShardingStrategy shardingStrategy;
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    public User findById(Long userId) {
        String tableName = shardingStrategy.determineUserTableName(userId);
        String sql = "SELECT * FROM " + tableName + " WHERE id = ?";
        
        return jdbcTemplate.queryForObject(sql, 
            (rs, rowNum) -> mapRowToUser(rs), userId);
    }
    
    public List<User> findByIds(List<Long> userIds) {
        // 按分片分组
        Map<String, List<Long>> shardGroups = userIds.stream()
            .collect(Collectors.groupingBy(shardingStrategy::determineUserTableName));
        
        List<User> result = new ArrayList<>();
        
        // 并行查询各分片
        shardGroups.entrySet().parallelStream().forEach(entry -> {
            String tableName = entry.getKey();
            List<Long> ids = entry.getValue();
            
            String sql = "SELECT * FROM " + tableName + " WHERE id IN (" +
                        ids.stream().map(String::valueOf).collect(Collectors.joining(",")) + ")";
            
            List<User> users = jdbcTemplate.query(sql, 
                (rs, rowNum) -> mapRowToUser(rs));
            
            synchronized (result) {
                result.addAll(users);
            }
        });
        
        return result;
    }
}
```

#### 6. 最佳实践总结

**6.1 排查优先级**
1. **应急处理**：先解决正在发生的慢查询
2. **根因分析**：分析慢查询的根本原因
3. **系统优化**：从架构层面解决性能问题
4. **预防措施**：建立监控和预警机制

**6.2 优化策略**
- **索引优先**：合理的索引设计是性能优化的基础
- **SQL重写**：优化查询逻辑，减少不必要的计算
- **架构调整**：读写分离、分库分表解决容量问题
- **缓存应用**：减少数据库访问压力

**6.3 监控告警**
- **实时监控**：监控慢查询、锁等待、连接数
- **定期分析**：分析慢查询日志，优化高频慢查询
- **容量规划**：根据业务增长预估数据库容量需求
- **性能基线**：建立性能基线，及时发现性能退化

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