# 第一阶段：Java基础强化 (3-6个月)

## 🎯 学习目标
- 掌握Java语法基础和面向对象编程思想
- 熟练使用Java集合框架和常用API
- 理解异常处理机制和最佳实践
- 具备基础算法思维和问题解决能力
- 能够独立完成简单的Java应用开发

## 📚 知识体系

### 1. Java语法基础

#### 1.1 数据类型和变量
```java
/**
 * Java数据类型详解
 * 包含基本数据类型和引用数据类型的使用
 */
public class DataTypeDemo {
    
    public static void main(String[] args) {
        // 基本数据类型 (8种)
        
        // 整数类型
        byte b = 127;           // 8位，范围：-128 ~ 127
        short s = 32767;        // 16位，范围：-32768 ~ 32767
        int i = 2147483647;     // 32位，范围：-2^31 ~ 2^31-1
        long l = 9223372036854775807L; // 64位，注意L后缀
        
        // 浮点类型
        float f = 3.14f;        // 32位，注意f后缀
        double d = 3.141592653589793; // 64位，默认浮点类型
        
        // 字符类型
        char c = 'A';           // 16位Unicode字符
        char unicode = '\u0041'; // Unicode表示法，也是'A'
        
        // 布尔类型
        boolean flag = true;    // 只有true和false两个值
        
        // 引用数据类型
        String str = "Hello World"; // 字符串是引用类型
        int[] array = {1, 2, 3, 4, 5}; // 数组是引用类型
        
        // 类型转换示例
        demonstrateTypeCasting();
        
        // 包装类使用
        demonstrateWrapperClasses();
    }
    
    /**
     * 演示类型转换
     * 包括自动类型转换和强制类型转换
     */
    private static void demonstrateTypeCasting() {
        System.out.println("=== 类型转换演示 ===");
        
        // 自动类型转换（隐式转换）- 小范围到大范围
        int intValue = 100;
        long longValue = intValue;      // int自动转换为long
        double doubleValue = intValue;  // int自动转换为double
        
        System.out.println("自动转换: int " + intValue + " -> long " + longValue);
        System.out.println("自动转换: int " + intValue + " -> double " + doubleValue);
        
        // 强制类型转换（显式转换）- 大范围到小范围
        double d = 3.14;
        int i = (int) d;  // 强制转换，会丢失小数部分
        
        System.out.println("强制转换: double " + d + " -> int " + i);
        
        // 注意：强制转换可能导致数据丢失
        long bigLong = 3000000000L;
        int fromLong = (int) bigLong;  // 数据溢出
        System.out.println("溢出示例: long " + bigLong + " -> int " + fromLong);
    }
    
    /**
     * 演示包装类的使用
     * 包装类提供了基本类型的对象表示和实用方法
     */
    private static void demonstrateWrapperClasses() {
        System.out.println("\n=== 包装类演示 ===");
        
        // 装箱和拆箱
        Integer intObj = 100;           // 自动装箱
        int primitiveInt = intObj;      // 自动拆箱
        
        // 包装类的实用方法
        String numberStr = "123";
        int parsed = Integer.parseInt(numberStr);  // 字符串转整数
        String converted = Integer.toString(456);  // 整数转字符串
        
        System.out.println("字符串转整数: \"" + numberStr + "\" -> " + parsed);
        System.out.println("整数转字符串: 456 -> \"" + converted + "\"");
        
        // 包装类的比较
        Integer a = 127;
        Integer b = 127;
        Integer c = 128;
        Integer d = 128;
        
        System.out.println("a == b (127): " + (a == b));  // true，缓存范围内
        System.out.println("c == d (128): " + (c == d));  // false，超出缓存范围
        System.out.println("c.equals(d): " + c.equals(d)); // true，值相等
    }
}
```

#### 1.2 运算符和表达式
```java
/**
 * Java运算符详解
 * 包含算术、关系、逻辑、位运算等各种运算符
 */
public class OperatorDemo {
    
    public static void main(String[] args) {
        demonstrateArithmeticOperators();
        demonstrateRelationalOperators();
        demonstrateLogicalOperators();
        demonstrateBitwiseOperators();
        demonstrateAssignmentOperators();
        demonstrateTernaryOperator();
    }
    
    /**
     * 算术运算符演示
     */
    private static void demonstrateArithmeticOperators() {
        System.out.println("=== 算术运算符 ===");
        
        int a = 10, b = 3;
        
        System.out.println("a = " + a + ", b = " + b);
        System.out.println("a + b = " + (a + b));  // 加法
        System.out.println("a - b = " + (a - b));  // 减法
        System.out.println("a * b = " + (a * b));  // 乘法
        System.out.println("a / b = " + (a / b));  // 除法（整数除法）
        System.out.println("a % b = " + (a % b));  // 取模
        
        // 浮点除法
        double result = (double) a / b;
        System.out.println("(double)a / b = " + result);
        
        // 自增自减运算符
        int x = 5;
        System.out.println("\n自增自减演示:");
        System.out.println("x = " + x);
        System.out.println("++x = " + (++x));  // 前置自增
        System.out.println("x++ = " + (x++));  // 后置自增
        System.out.println("x = " + x);
    }
    
    /**
     * 关系运算符演示
     */
    private static void demonstrateRelationalOperators() {
        System.out.println("\n=== 关系运算符 ===");
        
        int a = 10, b = 20;
        
        System.out.println("a = " + a + ", b = " + b);
        System.out.println("a > b: " + (a > b));   // 大于
        System.out.println("a < b: " + (a < b));   // 小于
        System.out.println("a >= b: " + (a >= b)); // 大于等于
        System.out.println("a <= b: " + (a <= b)); // 小于等于
        System.out.println("a == b: " + (a == b)); // 等于
        System.out.println("a != b: " + (a != b)); // 不等于
    }
    
    /**
     * 逻辑运算符演示
     */
    private static void demonstrateLogicalOperators() {
        System.out.println("\n=== 逻辑运算符 ===");
        
        boolean p = true, q = false;
        
        System.out.println("p = " + p + ", q = " + q);
        System.out.println("p && q: " + (p && q)); // 逻辑与（短路）
        System.out.println("p || q: " + (p || q)); // 逻辑或（短路）
        System.out.println("!p: " + (!p));         // 逻辑非
        
        // 短路演示
        System.out.println("\n短路运算演示:");
        int x = 0;
        boolean result1 = (x != 0) && (10 / x > 1); // 短路，不会执行除法
        System.out.println("短路与结果: " + result1);
        
        boolean result2 = (x == 0) || (10 / x > 1); // 短路，不会执行除法
        System.out.println("短路或结果: " + result2);
    }
    
    /**
     * 位运算符演示
     */
    private static void demonstrateBitwiseOperators() {
        System.out.println("\n=== 位运算符 ===");
        
        int a = 12;  // 二进制: 1100
        int b = 10;  // 二进制: 1010
        
        System.out.println("a = " + a + " (二进制: " + Integer.toBinaryString(a) + ")");
        System.out.println("b = " + b + " (二进制: " + Integer.toBinaryString(b) + ")");
        
        System.out.println("a & b = " + (a & b) + " (按位与)");
        System.out.println("a | b = " + (a | b) + " (按位或)");
        System.out.println("a ^ b = " + (a ^ b) + " (按位异或)");
        System.out.println("~a = " + (~a) + " (按位取反)");
        
        // 位移运算
        System.out.println("\n位移运算:");
        System.out.println("a << 2 = " + (a << 2) + " (左移2位)");
        System.out.println("a >> 2 = " + (a >> 2) + " (右移2位)");
        System.out.println("a >>> 2 = " + (a >>> 2) + " (无符号右移)");
    }
    
    /**
     * 赋值运算符演示
     */
    private static void demonstrateAssignmentOperators() {
        System.out.println("\n=== 赋值运算符 ===");
        
        int x = 10;
        System.out.println("初始值 x = " + x);
        
        x += 5;  // 等价于 x = x + 5
        System.out.println("x += 5: " + x);
        
        x -= 3;  // 等价于 x = x - 3
        System.out.println("x -= 3: " + x);
        
        x *= 2;  // 等价于 x = x * 2
        System.out.println("x *= 2: " + x);
        
        x /= 4;  // 等价于 x = x / 4
        System.out.println("x /= 4: " + x);
        
        x %= 3;  // 等价于 x = x % 3
        System.out.println("x %= 3: " + x);
    }
    
    /**
     * 三元运算符演示
     */
    private static void demonstrateTernaryOperator() {
        System.out.println("\n=== 三元运算符 ===");
        
        int a = 10, b = 20;
        
        // 语法: 条件 ? 值1 : 值2
        int max = (a > b) ? a : b;
        System.out.println("max(" + a + ", " + b + ") = " + max);
        
        String result = (a % 2 == 0) ? "偶数" : "奇数";
        System.out.println(a + " 是 " + result);
        
        // 嵌套三元运算符（不推荐，影响可读性）
        int score = 85;
        String grade = (score >= 90) ? "A" : 
                      (score >= 80) ? "B" : 
                      (score >= 70) ? "C" : "D";
        System.out.println("分数 " + score + " 对应等级: " + grade);
    }
}
```

### 2. 面向对象编程

#### 2.1 类和对象
```java
/**
 * 学生类 - 演示面向对象编程的基本概念
 * 包含封装、构造方法、方法重载等特性
 */
public class Student {
    
    // 实例变量（属性）- 私有化体现封装性
    private String name;        // 姓名
    private int age;           // 年龄
    private String studentId;  // 学号
    private double[] scores;   // 各科成绩
    private static int totalStudents = 0; // 静态变量，记录学生总数
    
    // 常量定义
    private static final int MAX_SUBJECTS = 10;
    private static final double PASS_SCORE = 60.0;
    
    // 默认构造方法
    public Student() {
        this("未知", 0, "000000"); // 调用其他构造方法
    }
    
    // 带参数的构造方法
    public Student(String name, int age, String studentId) {
        this.name = name;
        this.age = age;
        this.studentId = studentId;
        this.scores = new double[MAX_SUBJECTS];
        totalStudents++; // 每创建一个学生对象，总数加1
        
        System.out.println("创建学生对象: " + name + "，当前学生总数: " + totalStudents);
    }
    
    // 完整构造方法
    public Student(String name, int age, String studentId, double[] scores) {
        this(name, age, studentId); // 调用上面的构造方法
        
        // 复制成绩数组，避免外部修改
        if (scores != null && scores.length <= MAX_SUBJECTS) {
            System.arraycopy(scores, 0, this.scores, 0, scores.length);
        }
    }
    
    // Getter方法 - 提供对私有属性的只读访问
    public String getName() {
        return name;
    }
    
    public int getAge() {
        return age;
    }
    
    public String getStudentId() {
        return studentId;
    }
    
    // Setter方法 - 提供对私有属性的受控修改
    public void setName(String name) {
        if (name != null && !name.trim().isEmpty()) {
            this.name = name;
        } else {
            throw new IllegalArgumentException("姓名不能为空");
        }
    }
    
    public void setAge(int age) {
        if (age >= 0 && age <= 150) {
            this.age = age;
        } else {
            throw new IllegalArgumentException("年龄必须在0-150之间");
        }
    }
    
    public void setStudentId(String studentId) {
        if (studentId != null && studentId.matches("\\d{6}")) {
            this.studentId = studentId;
        } else {
            throw new IllegalArgumentException("学号必须是6位数字");
        }
    }
    
    // 方法重载 - 添加成绩的不同方式
    public void addScore(double score) {
        addScore(score, -1); // 添加到第一个空位置
    }
    
    public void addScore(double score, int subject) {
        if (score < 0 || score > 100) {
            throw new IllegalArgumentException("成绩必须在0-100之间");
        }
        
        if (subject >= 0 && subject < MAX_SUBJECTS) {
            scores[subject] = score;
        } else {
            // 找到第一个空位置（值为0的位置）
            for (int i = 0; i < scores.length; i++) {
                if (scores[i] == 0) {
                    scores[i] = score;
                    break;
                }
            }
        }
    }
    
    // 计算平均分
    public double calculateAverage() {
        double sum = 0;
        int count = 0;
        
        for (double score : scores) {
            if (score > 0) { // 只计算有效成绩
                sum += score;
                count++;
            }
        }
        
        return count > 0 ? sum / count : 0;
    }
    
    // 判断是否及格
    public boolean isPassed() {
        return calculateAverage() >= PASS_SCORE;
    }
    
    // 获取等级
    public String getGrade() {
        double avg = calculateAverage();
        if (avg >= 90) return "优秀";
        if (avg >= 80) return "良好";
        if (avg >= 70) return "中等";
        if (avg >= 60) return "及格";
        return "不及格";
    }
    
    // 静态方法 - 获取学生总数
    public static int getTotalStudents() {
        return totalStudents;
    }
    
    // 静态方法 - 比较两个学生的平均分
    public static int compareByAverage(Student s1, Student s2) {
        return Double.compare(s1.calculateAverage(), s2.calculateAverage());
    }
    
    // 重写toString方法
    @Override
    public String toString() {
        return String.format("Student{姓名='%s', 年龄=%d, 学号='%s', 平均分=%.2f, 等级='%s'}",
                name, age, studentId, calculateAverage(), getGrade());
    }
    
    // 重写equals方法
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        
        Student student = (Student) obj;
        return studentId.equals(student.studentId); // 根据学号判断是否为同一学生
    }
    
    // 重写hashCode方法
    @Override
    public int hashCode() {
        return studentId.hashCode();
    }
    
    // 演示方法
    public static void main(String[] args) {
        System.out.println("=== 面向对象编程演示 ===");
        
        // 创建学生对象
        Student student1 = new Student();
        Student student2 = new Student("张三", 20, "202101");
        Student student3 = new Student("李四", 19, "202102", new double[]{85, 92, 78, 88});
        
        // 设置学生信息
        student1.setName("王五");
        student1.setAge(21);
        student1.setStudentId("202103");
        student1.addScore(75);
        student1.addScore(82);
        student1.addScore(90);
        
        // 添加成绩
        student2.addScore(88, 0);
        student2.addScore(76, 1);
        student2.addScore(94, 2);
        
        // 输出学生信息
        System.out.println("\n学生信息:");
        System.out.println(student1);
        System.out.println(student2);
        System.out.println(student3);
        
        // 比较学生
        System.out.println("\n学生比较:");
        int comparison = Student.compareByAverage(student2, student3);
        if (comparison > 0) {
            System.out.println(student2.getName() + " 的平均分更高");
        } else if (comparison < 0) {
            System.out.println(student3.getName() + " 的平均分更高");
        } else {
            System.out.println("两人平均分相同");
        }
        
        // 显示统计信息
        System.out.println("\n统计信息:");
        System.out.println("学生总数: " + Student.getTotalStudents());
    }
}
```

#### 2.2 继承和多态
```java
/**
 * 动物基类 - 演示继承和多态
 */
abstract class Animal {
    protected String name;    // 受保护的属性，子类可以访问
    protected int age;
    protected String species; // 物种
    
    // 构造方法
    public Animal(String name, int age, String species) {
        this.name = name;
        this.age = age;
        this.species = species;
        System.out.println("创建动物: " + name);
    }
    
    // 抽象方法 - 子类必须实现
    public abstract void makeSound();
    public abstract void move();
    
    // 具体方法 - 子类可以继承或重写
    public void eat() {
        System.out.println(name + " 正在吃东西");
    }
    
    public void sleep() {
        System.out.println(name + " 正在睡觉");
    }
    
    // final方法 - 子类不能重写
    public final void breathe() {
        System.out.println(name + " 正在呼吸");
    }
    
    // Getter方法
    public String getName() { return name; }
    public int getAge() { return age; }
    public String getSpecies() { return species; }
    
    // 重写toString方法
    @Override
    public String toString() {
        return String.format("%s{name='%s', age=%d, species='%s'}", 
                getClass().getSimpleName(), name, age, species);
    }
}

/**
 * 狗类 - 继承Animal
 */
class Dog extends Animal {
    private String breed; // 品种
    private boolean isTrained; // 是否训练过
    
    public Dog(String name, int age, String breed) {
        super(name, age, "犬科"); // 调用父类构造方法
        this.breed = breed;
        this.isTrained = false;
    }
    
    // 实现抽象方法
    @Override
    public void makeSound() {
        System.out.println(name + " 汪汪叫");
    }
    
    @Override
    public void move() {
        System.out.println(name + " 在地上跑");
    }
    
    // 重写父类方法
    @Override
    public void eat() {
        System.out.println(name + " 正在吃狗粮");
    }
    
    // 子类特有的方法
    public void wagTail() {
        System.out.println(name + " 摇尾巴表示友好");
    }
    
    public void train() {
        this.isTrained = true;
        System.out.println(name + " 接受了训练");
    }
    
    public void performTrick() {
        if (isTrained) {
            System.out.println(name + " 表演特技：坐下、握手、装死");
        } else {
            System.out.println(name + " 还没有接受训练，不会表演特技");
        }
    }
    
    // Getter方法
    public String getBreed() { return breed; }
    public boolean isTrained() { return isTrained; }
}

/**
 * 猫类 - 继承Animal
 */
class Cat extends Animal {
    private boolean isIndoor; // 是否室内猫
    private int livesLeft;    // 剩余生命数（猫有九条命的传说）
    
    public Cat(String name, int age, boolean isIndoor) {
        super(name, age, "猫科");
        this.isIndoor = isIndoor;
        this.livesLeft = 9; // 传说中猫有九条命
    }
    
    // 实现抽象方法
    @Override
    public void makeSound() {
        System.out.println(name + " 喵喵叫");
    }
    
    @Override
    public void move() {
        System.out.println(name + " 优雅地走动");
    }
    
    // 重写父类方法
    @Override
    public void eat() {
        System.out.println(name + " 正在吃猫粮");
    }
    
    @Override
    public void sleep() {
        System.out.println(name + " 蜷缩成一团睡觉（猫一天要睡12-16小时）");
    }
    
    // 子类特有的方法
    public void purr() {
        System.out.println(name + " 发出呼噜声表示满足");
    }
    
    public void climb() {
        if (!isIndoor) {
            System.out.println(name + " 爬树");
        } else {
            System.out.println(name + " 爬猫爬架");
        }
    }
    
    public void useLives() {
        if (livesLeft > 0) {
            livesLeft--;
            System.out.println(name + " 用掉一条命，还剩 " + livesLeft + " 条命");
        } else {
            System.out.println(name + " 已经没有剩余生命了");
        }
    }
    
    // Getter方法
    public boolean isIndoor() { return isIndoor; }
    public int getLivesLeft() { return livesLeft; }
}

/**
 * 鸟类 - 继承Animal
 */
class Bird extends Animal {
    private double wingSpan; // 翼展
    private boolean canFly;  // 是否会飞
    
    public Bird(String name, int age, double wingSpan, boolean canFly) {
        super(name, age, "鸟类");
        this.wingSpan = wingSpan;
        this.canFly = canFly;
    }
    
    // 实现抽象方法
    @Override
    public void makeSound() {
        System.out.println(name + " 啾啾叫");
    }
    
    @Override
    public void move() {
        if (canFly) {
            System.out.println(name + " 在天空中飞翔");
        } else {
            System.out.println(name + " 在地上走动");
        }
    }
    
    // 重写父类方法
    @Override
    public void eat() {
        System.out.println(name + " 正在啄食");
    }
    
    // 子类特有的方法
    public void buildNest() {
        System.out.println(name + " 正在筑巢");
    }
    
    public void migrate() {
        if (canFly) {
            System.out.println(name + " 开始迁徙");
        } else {
            System.out.println(name + " 不会飞，无法迁徙");
        }
    }
    
    // Getter方法
    public double getWingSpan() { return wingSpan; }
    public boolean canFly() { return canFly; }
}

/**
 * 动物园类 - 演示多态的使用
 */
class Zoo {
    private Animal[] animals;
    private int count;
    
    public Zoo(int capacity) {
        this.animals = new Animal[capacity];
        this.count = 0;
    }
    
    // 添加动物
    public void addAnimal(Animal animal) {
        if (count < animals.length) {
            animals[count++] = animal;
            System.out.println("动物园新增动物: " + animal.getName());
        } else {
            System.out.println("动物园已满，无法添加更多动物");
        }
    }
    
    // 所有动物发出声音 - 体现多态
    public void makeAllAnimalsSound() {
        System.out.println("\n=== 动物园大合唱 ===");
        for (int i = 0; i < count; i++) {
            animals[i].makeSound(); // 多态调用
        }
    }
    
    // 所有动物移动 - 体现多态
    public void makeAllAnimalsMove() {
        System.out.println("\n=== 动物运动时间 ===");
        for (int i = 0; i < count; i++) {
            animals[i].move(); // 多态调用
        }
    }
    
    // 喂食所有动物
    public void feedAllAnimals() {
        System.out.println("\n=== 喂食时间 ===");
        for (int i = 0; i < count; i++) {
            animals[i].eat(); // 多态调用
        }
    }
    
    // 演示instanceof操作符
    public void demonstrateInstanceof() {
        System.out.println("\n=== instanceof 演示 ===");
        for (int i = 0; i < count; i++) {
            Animal animal = animals[i];
            
            System.out.print(animal.getName() + " 是 ");
            
            if (animal instanceof Dog) {
                System.out.println("狗");
                Dog dog = (Dog) animal; // 向下转型
                dog.wagTail();
            } else if (animal instanceof Cat) {
                System.out.println("猫");
                Cat cat = (Cat) animal; // 向下转型
                cat.purr();
            } else if (animal instanceof Bird) {
                System.out.println("鸟");
                Bird bird = (Bird) animal; // 向下转型
                bird.buildNest();
            }
        }
    }
    
    // 获取动物信息
    public void showAllAnimals() {
        System.out.println("\n=== 动物园动物信息 ===");
        for (int i = 0; i < count; i++) {
            System.out.println(animals[i]);
        }
    }
}

/**
 * 多态演示主类
 */
public class PolymorphismDemo {
    
    public static void main(String[] args) {
        System.out.println("=== 继承和多态演示 ===");
        
        // 创建动物园
        Zoo zoo = new Zoo(10);
        
        // 创建不同类型的动物
        Dog dog1 = new Dog("旺财", 3, "金毛");
        Dog dog2 = new Dog("小黑", 2, "拉布拉多");
        Cat cat1 = new Cat("咪咪", 2, true);
        Cat cat2 = new Cat("花花", 4, false);
        Bird bird1 = new Bird("小鸟", 1, 0.3, true);
        Bird bird2 = new Bird("企鹅", 5, 0.8, false);
        
        // 添加到动物园
        zoo.addAnimal(dog1);
        zoo.addAnimal(dog2);
        zoo.addAnimal(cat1);
        zoo.addAnimal(cat2);
        zoo.addAnimal(bird1);
        zoo.addAnimal(bird2);
        
        // 展示多态
        zoo.showAllAnimals();
        zoo.makeAllAnimalsSound();
        zoo.makeAllAnimalsMove();
        zoo.feedAllAnimals();
        zoo.demonstrateInstanceof();
        
        // 演示子类特有方法
        System.out.println("\n=== 子类特有方法演示 ===");
        dog1.train();
        dog1.performTrick();
        
        cat1.purr();
        cat1.climb();
        
        bird1.buildNest();
        bird1.migrate();
        
        // 演示方法重写
        System.out.println("\n=== 方法重写演示 ===");
        Animal animal = new Dog("多态狗", 1, "哈士奇");
        animal.eat(); // 调用Dog重写的eat方法
        animal.makeSound(); // 调用Dog实现的makeSound方法
        
        // 注意：不能调用子类特有的方法
        // animal.wagTail(); // 编译错误
        
        // 需要向下转型才能调用子类特有方法
        if (animal instanceof Dog) {
            ((Dog) animal).wagTail();
        }
    }
}
```

### 3. 集合框架

#### 3.1 List集合详解
```java
import java.util.*;
import java.util.concurrent.CopyOnWriteArrayList;

/**
 * List集合详解
 * 包含ArrayList、LinkedList、Vector的使用和性能对比
 */
public class ListDemo {
    
    public static void main(String[] args) {
        demonstrateArrayList();
        demonstrateLinkedList();
        demonstrateVector();
        compareListPerformance();
        demonstrateListOperations();
    }
    
    /**
     * ArrayList演示
     * 基于动态数组实现，查询快，插入删除慢
     */
    private static void demonstrateArrayList() {
        System.out.println("=== ArrayList 演示 ===");
        
        // 创建ArrayList
        List<String> arrayList = new ArrayList<>();
        
        // 添加元素
        arrayList.add("Apple");
        arrayList.add("Banana");
        arrayList.add("Cherry");
        arrayList.add("Date");
        arrayList.add("Elderberry");
        
        System.out.println("初始列表: " + arrayList);
        System.out.println("列表大小: " + arrayList.size());
        
        // 插入元素
        arrayList.add(2, "Coconut"); // 在索引2处插入
        System.out.println("插入Coconut后: " + arrayList);
        
        // 访问元素
        System.out.println("索引0的元素: " + arrayList.get(0));
        System.out.println("索引3的元素: " + arrayList.get(3));
        
        // 修改元素
        arrayList.set(1, "Blueberry");
        System.out.println("修改索引1后: " + arrayList);
        
        // 查找元素
        int index = arrayList.indexOf("Cherry");
        System.out.println("Cherry的索引: " + index);
        
        boolean contains = arrayList.contains("Date");
        System.out.println("是否包含Date: " + contains);
        
        // 删除元素
        arrayList.remove("Coconut"); // 按值删除
        arrayList.remove(0); // 按索引删除
        System.out.println("删除后: " + arrayList);
        
        // 遍历方式
        System.out.println("\n遍历方式:");
        
        // 1. 传统for循环
        System.out.print("传统for循环: ");
        for (int i = 0; i < arrayList.size(); i++) {
            System.out.print(arrayList.get(i) + " ");
        }
        System.out.println();
        
        // 2. 增强for循环
        System.out.print("增强for循环: ");
        for (String fruit : arrayList) {
            System.out.print(fruit + " ");
        }
        System.out.println();
        
        // 3. 迭代器
        System.out.print("迭代器: ");
        Iterator<String> iterator = arrayList.iterator();
        while (iterator.hasNext()) {
            System.out.print(iterator.next() + " ");
        }
        System.out.println();
        
        // 4. Lambda表达式（Java 8+）
        System.out.print("Lambda表达式: ");
        arrayList.forEach(fruit -> System.out.print(fruit + " "));
        System.out.println();
    }
    
    /**
     * LinkedList演示
     * 基于双向链表实现，插入删除快，查询慢
     */
    private static void demonstrateLinkedList() {
        System.out.println("\n=== LinkedList 演示 ===");
        
        // LinkedList实现了List和Deque接口
        LinkedList<Integer> linkedList = new LinkedList<>();
        
        // 添加元素
        linkedList.add(10);
        linkedList.add(20);
        linkedList.add(30);
        linkedList.add(40);
        
        System.out.println("初始列表: " + linkedList);
        
        // LinkedList特有的方法（作为双端队列使用）
        linkedList.addFirst(5);  // 在头部添加
        linkedList.addLast(50);  // 在尾部添加
        System.out.println("头尾添加后: " + linkedList);
        
        // 获取头尾元素
        System.out.println("第一个元素: " + linkedList.getFirst());
        System.out.println("最后一个元素: " + linkedList.getLast());
        
        // 删除头尾元素
        Integer first = linkedList.removeFirst();
        Integer last = linkedList.removeLast();
        System.out.println("删除的第一个元素: " + first);
        System.out.println("删除的最后一个元素: " + last);
        System.out.println("删除后: " + linkedList);
        
        // 作为栈使用
        System.out.println("\n作为栈使用:");
        linkedList.push(100); // 压栈
        linkedList.push(200);
        System.out.println("压栈后: " + linkedList);
        
        Integer popped = linkedList.pop(); // 出栈
        System.out.println("出栈元素: " + popped);
        System.out.println("出栈后: " + linkedList);
        
        // 作为队列使用
        System.out.println("\n作为队列使用:");
        linkedList.offer(300); // 入队
        linkedList.offer(400);
        System.out.println("入队后: " + linkedList);
        
        Integer polled = linkedList.poll(); // 出队
        System.out.println("出队元素: " + polled);
        System.out.println("出队后: " + linkedList);
    }
    
    /**
     * Vector演示
     * 线程安全的动态数组，性能较ArrayList差
     */
    private static void demonstrateVector() {
        System.out.println("\n=== Vector 演示 ===");
        
        Vector<String> vector = new Vector<>();
        
        // Vector的方法与ArrayList类似
        vector.add("Vector1");
        vector.add("Vector2");
        vector.add("Vector3");
        
        System.out.println("Vector内容: " + vector);
        System.out.println("Vector容量: " + vector.capacity());
        System.out.println("Vector大小: " + vector.size());
        
        // Vector特有的方法
        vector.addElement("Vector4"); // 等价于add
        vector.insertElementAt("Vector0", 0); // 在指定位置插入
        
        System.out.println("添加元素后: " + vector);
        
        // 枚举遍历（Vector特有）
        System.out.print("枚举遍历: ");
        Enumeration<String> enumeration = vector.elements();
        while (enumeration.hasMoreElements()) {
            System.out.print(enumeration.nextElement() + " ");
        }
        System.out.println();
    }
    
    /**
     * List性能对比
     */
    private static void compareListPerformance() {
        System.out.println("\n=== List性能对比 ===");
        
        int size = 100000;
        
        // ArrayList性能测试
        List<Integer> arrayList = new ArrayList<>();
        long startTime = System.currentTimeMillis();
        for (int i = 0; i < size; i++) {
            arrayList.add(i);
        }
        long arrayListAddTime = System.currentTimeMillis() - startTime;
        
        // LinkedList性能测试
        List<Integer> linkedList = new LinkedList<>();
        startTime = System.currentTimeMillis();
        for (int i = 0; i < size; i++) {
            linkedList.add(i);
        }
        long linkedListAddTime = System.currentTimeMillis() - startTime;
        
        System.out.println("添加" + size + "个元素:");
        System.out.println("ArrayList耗时: " + arrayListAddTime + "ms");
        System.out.println("LinkedList耗时: " + linkedListAddTime + "ms");
        
        // 随机访问性能测试
        Random random = new Random();
        
        startTime = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++) {
            arrayList.get(random.nextInt(size));
        }
        long arrayListGetTime = System.currentTimeMillis() - startTime;
        
        startTime = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++) {
            linkedList.get(random.nextInt(size));
        }
        long linkedListGetTime = System.currentTimeMillis() - startTime;
        
        System.out.println("\n随机访问10000次:");
        System.out.println("ArrayList耗时: " + arrayListGetTime + "ms");
        System.out.println("LinkedList耗时: " + linkedListGetTime + "ms");
    }
    
    /**
     * List常用操作演示
     */
    private static void demonstrateListOperations() {
        System.out.println("\n=== List常用操作 ===");
        
        List<String> list1 = new ArrayList<>(Arrays.asList("A", "B", "C", "D"));
        List<String> list2 = new ArrayList<>(Arrays.asList("C", "D", "E", "F"));
        
        System.out.println("list1: " + list1);
        System.out.println("list2: " + list2);
        
        // 子列表
        List<String> subList = list1.subList(1, 3);
        System.out.println("list1的子列表[1,3): " + subList);
        
        // 列表合并
        List<String> merged = new ArrayList<>(list1);
        merged.addAll(list2);
        System.out.println("合并后: " + merged);
        
        // 求交集
        List<String> intersection = new ArrayList<>(list1);
        intersection.retainAll(list2);
        System.out.println("交集: " + intersection);
        
        // 求差集
        List<String> difference = new ArrayList<>(list1);
        difference.removeAll(list2);
        System.out.println("list1-list2差集: " + difference);
        
        // 排序
        List<String> sortList = new ArrayList<>(Arrays.asList("Banana", "Apple", "Cherry", "Date"));
        System.out.println("排序前: " + sortList);
        
        Collections.sort(sortList); // 自然排序
        System.out.println("自然排序后: " + sortList);
        
        Collections.sort(sortList, Collections.reverseOrder()); // 逆序
        System.out.println("逆序排序后: " + sortList);
        
        // 自定义排序
        Collections.sort(sortList, (s1, s2) -> s1.length() - s2.length());
        System.out.println("按长度排序: " + sortList);
        
        // 查找
        Collections.sort(sortList); // 二分查找需要有序列表
        int index = Collections.binarySearch(sortList, "Cherry");
        System.out.println("Cherry的索引: " + index);
        
        // 反转
        Collections.reverse(sortList);
        System.out.println("反转后: " + sortList);
        
        // 打乱
        Collections.shuffle(sortList);
        System.out.println("打乱后: " + sortList);
    }
}
```

## 📝 第一阶段经典面试题

### 基础语法类
1. **Java中的基本数据类型有哪些？它们的取值范围是什么？**
   
   **8种基本数据类型详解：**
   
   - **整型类型**：
     - `byte`：8位，范围-128~127，默认值0
     - `short`：16位，范围-32,768~32,767，默认值0
     - `int`：32位，范围-2,147,483,648~2,147,483,647，默认值0
     - `long`：64位，范围-9,223,372,036,854,775,808~9,223,372,036,854,775,807，默认值0L
   
   - **浮点型类型**：
     - `float`：32位IEEE754单精度，约6-7位有效数字，默认值0.0f
     - `double`：64位IEEE754双精度，约15-16位有效数字，默认值0.0d
   
   - **字符型**：
     - `char`：16位Unicode字符，范围0~65,535，默认值'\u0000'
   
   - **布尔型**：
     - `boolean`：只有true/false两个值，默认值false
   
   **重要特性：**
   ```java
   // 整型字面量
   int decimal = 100;        // 十进制
   int hex = 0x64;          // 十六进制
   int binary = 0b1100100;  // 二进制(JDK7+)
   long bigNum = 100L;      // long类型需要L后缀
   
   // 浮点型精度问题
   float f = 0.1f + 0.2f;   // 结果不等于0.3
   double d = 0.1 + 0.2;    // 结果不等于0.3
   // 解决方案：使用BigDecimal进行精确计算
   
   // 字符型Unicode支持
   char chinese = '中';      // 支持中文字符
   char unicode = '\u4e2d'; // Unicode编码表示
   ```
   
   **内存占用和性能考虑：**
   - 选择合适的数据类型可以节省内存
   - byte和short在数组中能显著减少内存占用
   - float vs double：double精度更高但占用更多内存

2. **== 和 equals() 的区别是什么？**
   
   **== 操作符详解：**
   - **基本数据类型**：直接比较值是否相等
   - **引用数据类型**：比较对象的内存地址（引用）是否相同
   
   **equals() 方法详解：**
   - **默认实现**：Object类中equals()等同于==，比较内存地址
   - **重写后**：可以自定义比较逻辑，通常比较对象的内容
   
   **实际应用示例：**
   ```java
   // 基本数据类型
   int a = 100, b = 100;
   System.out.println(a == b);  // true，比较值
   
   // 引用数据类型
   String s1 = new String("hello");
   String s2 = new String("hello");
   String s3 = "hello";
   String s4 = "hello";
   
   System.out.println(s1 == s2);        // false，不同对象
   System.out.println(s1.equals(s2));   // true，内容相同
   System.out.println(s3 == s4);        // true，字符串常量池
   
   // Integer缓存机制
   Integer i1 = 127, i2 = 127;
   Integer i3 = 128, i4 = 128;
   System.out.println(i1 == i2);        // true，缓存范围内
   System.out.println(i3 == i4);        // false，超出缓存范围
   ```
   
   **String特殊情况：**
   - **字符串常量池**：相同字符串字面量指向同一对象
   - **new String()**：总是创建新对象
   - **intern()方法**：将字符串加入常量池
   
   **安全使用equals()：**
   ```java
   // 错误方式：可能抛出NullPointerException
   if (str.equals("target")) { ... }
   
   // 正确方式1：先判空
   if (str != null && str.equals("target")) { ... }
   
   // 正确方式2：常量在前
   if ("target".equals(str)) { ... }
   
   // 正确方式3：使用Objects.equals()
   if (Objects.equals(str, "target")) { ... }
   ```
   
   **重写equals()的规范：**
   - **自反性**：x.equals(x) 必须返回true
   - **对称性**：x.equals(y) 与 y.equals(x) 结果相同
   - **传递性**：如果x.equals(y)且y.equals(z)，则x.equals(z)
   - **一致性**：多次调用结果一致
   - **非空性**：x.equals(null) 必须返回false
   - **重写equals()必须同时重写hashCode()**

3. **String、StringBuffer、StringBuilder的区别？**
   
   **String类特性：**
   - **不可变性**：String对象一旦创建，内容不可修改
   - **线程安全**：由于不可变，天然线程安全
   - **内存机制**：字符串常量池，相同内容共享对象
   - **适用场景**：字符串内容固定，少量字符串操作
   
   **StringBuffer类特性：**
   - **可变性**：内部使用char数组，可动态扩容
   - **线程安全**：所有方法都使用synchronized修饰
   - **性能开销**：同步机制带来性能损耗
   - **适用场景**：多线程环境下的字符串拼接
   
   **StringBuilder类特性：**
   - **可变性**：与StringBuffer相同的内部实现
   - **线程不安全**：没有同步机制，性能最优
   - **JDK版本**：JDK 1.5引入
   - **适用场景**：单线程环境下的大量字符串操作
   
   **性能对比示例：**
   ```java
   // String拼接 - 性能最差
   String str = "";
   for (int i = 0; i < 10000; i++) {
       str += "a";  // 每次都创建新对象
   }
   
   // StringBuilder拼接 - 性能最优
   StringBuilder sb = new StringBuilder();
   for (int i = 0; i < 10000; i++) {
       sb.append("a");  // 在原对象上修改
   }
   String result = sb.toString();
   
   // StringBuffer拼接 - 性能中等
   StringBuffer sbf = new StringBuffer();
   for (int i = 0; i < 10000; i++) {
       sbf.append("a");  // 线程安全但有同步开销
   }
   ```
   
   **内部实现原理：**
   ```java
   // String的不可变性
   public final class String {
       private final char[] value;  // final修饰，不可变
   }
   
   // StringBuilder/StringBuffer的可变性
   abstract class AbstractStringBuilder {
       char[] value;                // 可变数组
       int count;                   // 实际长度
       
       // 扩容机制：通常是原容量的2倍+2
       void expandCapacity(int minimumCapacity) {
           int newCapacity = value.length * 2 + 2;
           // ...
       }
   }
   ```
   
   **选择建议：**
   - **少量操作**：直接使用String
   - **大量拼接(单线程)**：使用StringBuilder
   - **大量拼接(多线程)**：使用StringBuffer
   - **JDK 8+**：可考虑使用String.join()或StringJoiner
   
   **性能数据参考：**
   - 10000次拼接测试：StringBuilder(1ms) < StringBuffer(3ms) < String(300ms+)
   - 内存占用：StringBuilder和StringBuffer会预分配空间，String每次创建新对象

4. **final、finally、finalize的区别？**
   
   **final关键字详解：**
   - **修饰类**：类不能被继承，如String、Integer等
   - **修饰方法**：方法不能被重写，但可以被重载
   - **修饰变量**：变量不能被重新赋值，但对象内容可以修改
   
   ```java
   // final修饰类
   public final class String { ... }  // 不能被继承
   
   // final修饰方法
   class Parent {
       public final void method() { ... }  // 不能被重写
   }
   
   // final修饰变量
   final int x = 10;           // 基本类型：值不可变
   final List<String> list = new ArrayList<>();  // 引用类型：引用不可变
   list.add("item");          // 但对象内容可以修改
   
   // final修饰参数
   public void method(final int param) {
       // param = 20;  // 编译错误
   }
   ```
   
   **finally语句块详解：**
   - **执行时机**：无论try块是否异常都会执行
   - **不执行情况**：System.exit()、JVM崩溃、无限循环
   - **执行顺序**：在return语句之前执行
   
   ```java
   public int testFinally() {
       try {
           return 1;
       } catch (Exception e) {
           return 2;
       } finally {
           // 这里会执行，但不会改变返回值
           System.out.println("finally executed");
           // return 3;  // 如果有return，会覆盖try/catch的返回值
       }
   }
   ```
   
   **finalize()方法详解：**
   - **调用时机**：对象被垃圾回收前调用（不保证一定调用）
   - **性能影响**：会延迟对象回收，影响GC性能
   - **替代方案**：使用try-with-resources或显式close()方法
   
   ```java
   // 不推荐使用finalize()
   @Override
   protected void finalize() throws Throwable {
       try {
           // 清理资源
       } finally {
           super.finalize();
       }
   }
   
   // 推荐使用try-with-resources
   try (FileInputStream fis = new FileInputStream("file.txt")) {
       // 使用资源
   } // 自动关闭资源
   ```

5. **什么是自动装箱和拆箱？**
   
   **自动装箱(Autoboxing)详解：**
   - **定义**：基本数据类型自动转换为对应的包装类对象
   - **实现原理**：编译器自动调用包装类的valueOf()方法
   - **发生场景**：赋值、方法参数传递、集合操作
   
   **自动拆箱(Unboxing)详解：**
   - **定义**：包装类对象自动转换为对应的基本数据类型
   - **实现原理**：编译器自动调用包装类的xxxValue()方法
   - **发生场景**：算术运算、比较操作、赋值给基本类型
   
   **代码示例：**
   ```java
   // 自动装箱示例
   Integer i = 100;           // 等价于 Integer.valueOf(100)
   List<Integer> list = new ArrayList<>();
   list.add(200);             // 等价于 list.add(Integer.valueOf(200))
   
   // 自动拆箱示例
   Integer j = 300;
   int k = j;                 // 等价于 j.intValue()
   int sum = i + j;           // 等价于 i.intValue() + j.intValue()
   
   // 包装类缓存机制
   Integer a = 127, b = 127;
   Integer c = 128, d = 128;
   System.out.println(a == b);  // true，缓存范围内(-128~127)
   System.out.println(c == d);  // false，超出缓存范围
   ```
   
   **重要注意事项：**
   - **NullPointerException风险**：
   ```java
   Integer num = null;
   int value = num;  // 抛出NullPointerException
   ```
   
   - **性能影响**：频繁装箱拆箱会创建大量对象，影响性能
   ```java
   // 性能较差的写法
   Integer sum = 0;
   for (int i = 0; i < 1000; i++) {
       sum += i;  // 每次都装箱拆箱
   }
   
   // 性能较好的写法
   int sum = 0;
   for (int i = 0; i < 1000; i++) {
       sum += i;  // 避免装箱拆箱
   }
   ```
   
   - **包装类缓存范围**：
     - Boolean：true/false
     - Byte：-128~127
     - Character：0~127
     - Short/Integer：-128~127
     - Long：-128~127

### 面向对象类
6. **面向对象的三大特性是什么？请详细解释。**
   
   **封装(Encapsulation)详解：**
   - **核心思想**：将数据和操作数据的方法绑定在一起，隐藏内部实现细节
   - **实现方式**：使用访问修饰符控制成员的可见性
   - **优点**：提高安全性、可维护性、代码复用性
   
   ```java
   public class Student {
       // 私有属性，外部无法直接访问
       private String name;
       private int age;
       
       // 公共方法提供受控访问
       public String getName() {
           return name;
       }
       
       public void setAge(int age) {
           // 数据验证，确保数据有效性
           if (age > 0 && age < 150) {
               this.age = age;
           } else {
               throw new IllegalArgumentException("年龄无效");
           }
       }
       
       // 内部方法，封装复杂逻辑
       private boolean isValidName(String name) {
           return name != null && !name.trim().isEmpty();
       }
   }
   ```
   
   **继承(Inheritance)详解：**
   - **核心思想**：子类继承父类的属性和方法，实现代码复用
   - **关键字**：extends(类继承)、implements(接口实现)
   - **特点**：单继承、传递性、子类可以扩展父类功能
   
   ```java
   // 父类
   public class Animal {
       protected String name;
       protected int age;
       
       public void eat() {
           System.out.println(name + " is eating");
       }
       
       public void sleep() {
           System.out.println(name + " is sleeping");
       }
   }
   
   // 子类继承父类
   public class Dog extends Animal {
       private String breed;
       
       // 子类特有方法
       public void bark() {
           System.out.println(name + " is barking");
       }
       
       // 重写父类方法
       @Override
       public void eat() {
           System.out.println("Dog " + name + " is eating dog food");
       }
   }
   ```
   
   **多态(Polymorphism)详解：**
   - **核心思想**：同一个接口，不同的实现；同一个方法调用，不同的执行结果
   - **实现方式**：方法重载(编译时多态)、方法重写(运行时多态)
   - **必要条件**：继承关系、方法重写、父类引用指向子类对象
   
   ```java
   // 多态示例
   public class PolymorphismDemo {
       public static void main(String[] args) {
           // 父类引用指向不同子类对象
           Animal animal1 = new Dog();
           Animal animal2 = new Cat();
           
           // 同一个方法调用，不同的执行结果
           animal1.makeSound();  // 输出：Dog barks
           animal2.makeSound();  // 输出：Cat meows
           
           // 运行时类型判断
           if (animal1 instanceof Dog) {
               Dog dog = (Dog) animal1;  // 向下转型
               dog.wagTail();            // 调用子类特有方法
           }
       }
   }
   
   abstract class Animal {
       public abstract void makeSound();  // 抽象方法
   }
   
   class Dog extends Animal {
       @Override
       public void makeSound() {
           System.out.println("Dog barks");
       }
       
       public void wagTail() {
           System.out.println("Dog wags tail");
       }
   }
   
   class Cat extends Animal {
       @Override
       public void makeSound() {
           System.out.println("Cat meows");
       }
   }
   ```
   
   **三大特性的关系：**
   - **封装**是基础，保证了类的内部实现的安全性
   - **继承**是手段，实现了代码的复用和扩展
   - **多态**是目标，提供了灵活的程序设计方式
   
   **实际应用价值：**
   - **可维护性**：修改实现不影响接口
   - **可扩展性**：容易添加新的子类
   - **可复用性**：避免重复代码
   - **灵活性**：运行时动态绑定

7. **重载和重写的区别？**
   
   **方法重载(Overload)详解：**
   - **定义**：同一个类中，方法名相同但参数列表不同
   - **判断依据**：方法名 + 参数列表（参数个数、类型、顺序）
   - **多态类型**：编译时多态（静态多态）
   - **返回值**：可以相同也可以不同，但不能仅凭返回值区分
   
   ```java
   public class Calculator {
       // 方法重载示例
       public int add(int a, int b) {
           return a + b;
       }
       
       public double add(double a, double b) {
           return a + b;
       }
       
       public int add(int a, int b, int c) {
           return a + b + c;
       }
       
       // 错误：仅返回值不同不能构成重载
       // public double add(int a, int b) { return a + b; }
   }
   ```
   
   **方法重写(Override)详解：**
   - **定义**：子类重新定义父类的方法，提供不同的实现
   - **判断依据**：方法签名必须完全相同（方法名、参数列表、返回值类型）
   - **多态类型**：运行时多态（动态多态）
   - **注解**：建议使用@Override注解，编译器会检查
   
   ```java
   class Animal {
       public void makeSound() {
           System.out.println("Animal makes sound");
       }
       
       public Animal getAnimal() {
           return this;
       }
   }
   
   class Dog extends Animal {
       @Override
       public void makeSound() {
           System.out.println("Dog barks");  // 重写父类方法
       }
       
       // 协变返回类型：返回值可以是父类返回值的子类
       @Override
       public Dog getAnimal() {
           return this;
       }
   }
   ```
   
   **重载vs重写对比表：**
   | 特性 | 重载(Overload) | 重写(Override) |
   |------|----------------|----------------|
   | 关系 | 水平关系(同一类) | 垂直关系(继承) |
   | 参数 | 必须不同 | 必须相同 |
   | 返回值 | 可以不同 | 必须相同(或协变) |
   | 访问修饰符 | 可以不同 | 不能更严格 |
   | 异常 | 可以不同 | 不能抛出新的检查异常 |
   | 绑定时机 | 编译时 | 运行时 |

8. **抽象类和接口的区别？**
   
   **抽象类(Abstract Class)详解：**
   - **定义**：使用abstract关键字修饰的类，不能被实例化
   - **成员特性**：可以有构造方法、实例变量、具体方法、抽象方法
   - **继承关系**：使用extends关键字，只能单继承
   - **访问修饰符**：成员可以有任意访问修饰符
   
   ```java
   public abstract class Shape {
       protected String color;          // 实例变量
       private static int count = 0;   // 静态变量
       
       // 构造方法
       public Shape(String color) {
           this.color = color;
           count++;
       }
       
       // 具体方法
       public String getColor() {
           return color;
       }
       
       // 抽象方法
       public abstract double getArea();
       public abstract double getPerimeter();
       
       // 静态方法
       public static int getCount() {
           return count;
       }
   }
   
   class Circle extends Shape {
       private double radius;
       
       public Circle(String color, double radius) {
           super(color);  // 调用父类构造方法
           this.radius = radius;
       }
       
       @Override
       public double getArea() {
           return Math.PI * radius * radius;
       }
       
       @Override
       public double getPerimeter() {
           return 2 * Math.PI * radius;
       }
   }
   ```
   
   **接口(Interface)详解：**
   - **定义**：使用interface关键字定义的契约
   - **成员特性**：常量(public static final)、抽象方法、默认方法(JDK8+)、静态方法(JDK8+)
   - **实现关系**：使用implements关键字，可以多实现
   - **访问修饰符**：所有成员都是public的
   
   ```java
   public interface Drawable {
       // 常量（默认public static final）
       String TYPE = "GRAPHIC";
       
       // 抽象方法（默认public abstract）
       void draw();
       
       // 默认方法（JDK 8+）
       default void print() {
           System.out.println("Printing " + TYPE);
       }
       
       // 静态方法（JDK 8+）
       static void info() {
           System.out.println("This is a drawable interface");
       }
       
       // 私有方法（JDK 9+）
       private void helper() {
           System.out.println("Helper method");
       }
   }
   
   public interface Colorable {
       void setColor(String color);
   }
   
   // 类可以实现多个接口
   class Rectangle implements Drawable, Colorable {
       private String color;
       
       @Override
       public void draw() {
           System.out.println("Drawing a " + color + " rectangle");
       }
       
       @Override
       public void setColor(String color) {
           this.color = color;
       }
   }
   ```
   
   **抽象类vs接口对比表：**
   | 特性 | 抽象类 | 接口 |
   |------|--------|------|
   | 关键字 | abstract class | interface |
   | 继承/实现 | extends(单继承) | implements(多实现) |
   | 构造方法 | 可以有 | 不能有 |
   | 实例变量 | 可以有 | 只能有常量 |
   | 方法类型 | 抽象+具体方法 | 抽象+默认+静态方法 |
   | 访问修饰符 | 任意 | public |
   | 使用场景 | is-a关系 | can-do关系 |
   
   **选择建议：**
   - **抽象类**：当多个类有共同的属性和部分共同行为时
   - **接口**：当需要定义一组规范，不关心实现细节时
   - **JDK 8+**：接口功能增强，选择更加灵活

9. **什么是多态？多态的实现方式有哪些？**
   
   **多态的定义：**
   - **概念**：同一个接口，不同的实现；一个对象在不同情况下表现出不同的形态
   - **本质**：允许不同类的对象对同一消息做出响应，但具体的响应行为由对象的实际类型决定
   - **核心思想**："一个接口，多种实现"
   
   **多态的分类：**
   
   **1. 编译时多态（静态多态）：**
   - **实现方式**：方法重载(Overload)
   - **绑定时机**：编译时确定调用哪个方法
   - **判断依据**：方法签名（方法名+参数列表）
   
   ```java
   public class MathUtils {
       // 编译时多态示例
       public int add(int a, int b) {
           return a + b;
       }
       
       public double add(double a, double b) {
           return a + b;
       }
       
       public String add(String a, String b) {
           return a + b;
       }
   }
   ```
   
   **2. 运行时多态（动态多态）：**
   - **实现方式**：继承+方法重写、接口实现
   - **绑定时机**：运行时根据对象的实际类型确定调用哪个方法
   - **核心机制**：动态绑定(Dynamic Binding)
   
   ```java
   // 通过继承实现多态
   abstract class Animal {
       public abstract void makeSound();
       
       public void sleep() {
           System.out.println("Animal is sleeping");
       }
   }
   
   class Dog extends Animal {
       @Override
       public void makeSound() {
           System.out.println("Dog barks: Woof!");
       }
   }
   
   class Cat extends Animal {
       @Override
       public void makeSound() {
           System.out.println("Cat meows: Meow!");
       }
   }
   
   // 多态的使用
   public class PolymorphismDemo {
       public static void main(String[] args) {
           // 父类引用指向子类对象
           Animal[] animals = {
               new Dog(),
               new Cat()
           };
           
           // 运行时多态：根据对象实际类型调用相应方法
           for (Animal animal : animals) {
               animal.makeSound();  // 动态绑定
           }
       }
   }
   ```
   
   **多态的三个必要条件：**
   1. **继承**：子类继承父类或实现接口
   2. **重写**：子类重写父类的方法
   3. **向上转型**：父类引用指向子类对象
   
   **多态的优点：**
   - **可扩展性**：添加新的子类不需要修改现有代码
   - **可维护性**：统一的接口，降低代码耦合度
   - **灵活性**：同一份代码可以处理多种类型的对象

10. **静态变量和实例变量的区别？**
    
    **静态变量（类变量）详解：**
    - **定义**：使用static关键字修饰的变量，属于类而不是实例
    - **内存位置**：方法区（JDK8前）或堆的元空间（JDK8+）
    - **生命周期**：类加载时创建，类卸载时销毁
    - **访问方式**：类名.变量名 或 对象.变量名（不推荐）
    - **共享性**：所有实例共享同一份静态变量
    
    **实例变量（成员变量）详解：**
    - **定义**：没有static修饰的变量，属于具体的对象实例
    - **内存位置**：堆内存中的对象实例内
    - **生命周期**：对象创建时分配，对象销毁时释放
    - **访问方式**：对象.变量名
    - **独立性**：每个对象实例都有自己独立的实例变量副本
    
    ```java
    public class VariableDemo {
        private static int staticCount = 0;    // 静态变量
        private int instanceId;                // 实例变量
        private String name;                   // 实例变量
        
        // 静态代码块：类加载时执行一次
        static {
            System.out.println("类加载，静态变量初始化");
            staticCount = 100;
        }
        
        // 构造方法
        public VariableDemo(String name) {
            staticCount++;              // 修改静态变量
            this.instanceId = staticCount;  // 实例变量赋值
            this.name = name;           // 实例变量赋值
        }
        
        // 静态方法：只能访问静态变量
        public static int getStaticCount() {
            return staticCount;
            // return instanceId;  // 编译错误：不能访问实例变量
        }
        
        // 实例方法：可以访问静态变量和实例变量
        public void displayInfo() {
            System.out.println("静态变量: " + staticCount);
            System.out.println("实例ID: " + instanceId);
            System.out.println("姓名: " + name);
        }
        
        public static void main(String[] args) {
            System.out.println("初始静态变量: " + VariableDemo.getStaticCount());
            
            VariableDemo obj1 = new VariableDemo("张三");
            VariableDemo obj2 = new VariableDemo("李四");
            
            obj1.displayInfo();
            obj2.displayInfo();
            
            System.out.println("最终静态变量: " + VariableDemo.getStaticCount());
        }
    }
    ```
    
    **详细对比表：**
    | 特性 | 静态变量 | 实例变量 |
    |------|----------|----------|
    | **关键字** | static | 无 |
    | **所属** | 类 | 对象实例 |
    | **内存位置** | 方法区/元空间 | 堆内存 |
    | **生命周期** | 类加载到类卸载 | 对象创建到对象销毁 |
    | **初始化时机** | 类首次加载时 | 对象创建时 |
    | **访问方式** | 类名.变量名（推荐） | 对象.变量名 |
    | **共享性** | 所有实例共享 | 每个实例独有 |
    | **默认值** | 有（与实例变量相同） | 有 |
    | **线程安全** | 需要考虑同步 | 对象级别隔离 |
    
    **使用场景：**
    - **静态变量**：计数器、配置信息、常量、缓存等类级别的数据
    - **实例变量**：对象的属性、状态信息等实例级别的数据
    
    **注意事项：**
    - 静态变量在多线程环境下需要考虑线程安全问题
    - 过度使用静态变量可能导致内存泄漏
    - 静态变量会影响类的可测试性

### 集合框架类
11. **ArrayList和LinkedList的区别？**
    
    **ArrayList详解：**
    - **底层结构**：基于动态数组（Object[]）实现
    - **默认容量**：10，扩容时增长50%（newCapacity = oldCapacity + (oldCapacity >> 1)）
    - **随机访问**：支持通过索引快速访问，时间复杂度O(1)
    - **内存特点**：元素在内存中连续存储，缓存友好
    
    **LinkedList详解：**
    - **底层结构**：基于双向链表实现
    - **节点结构**：每个节点包含数据、前驱指针、后继指针
    - **顺序访问**：必须从头或尾开始遍历，时间复杂度O(n)
    - **内存特点**：节点分散存储，需要额外的指针空间
    
    ```java
    // ArrayList核心源码结构
    public class ArrayList<E> extends AbstractList<E> {
        private static final int DEFAULT_CAPACITY = 10;
        private Object[] elementData;  // 存储元素的数组
        private int size;              // 实际元素个数
        
        // 扩容机制
        private void grow(int minCapacity) {
            int oldCapacity = elementData.length;
            int newCapacity = oldCapacity + (oldCapacity >> 1);  // 增长50%
            elementData = Arrays.copyOf(elementData, newCapacity);
        }
    }
    
    // LinkedList核心源码结构
    public class LinkedList<E> extends AbstractSequentialList<E> {
        private Node<E> first;  // 头节点
        private Node<E> last;   // 尾节点
        private int size;
        
        // 节点结构
        private static class Node<E> {
            E item;        // 数据
            Node<E> next;  // 后继指针
            Node<E> prev;  // 前驱指针
        }
    }
    ```
    
    **性能对比详表：**
    | 操作 | ArrayList | LinkedList | 说明 |
    |------|-----------|------------|------|
    | **随机访问** | O(1) | O(n) | ArrayList支持索引直接访问 |
    | **头部插入** | O(n) | O(1) | ArrayList需要移动所有元素 |
    | **尾部插入** | O(1)* | O(1) | ArrayList可能触发扩容 |
    | **中间插入** | O(n) | O(n) | ArrayList移动元素，LinkedList需要定位 |
    | **头部删除** | O(n) | O(1) | ArrayList需要移动所有元素 |
    | **尾部删除** | O(1) | O(1) | 都是常数时间 |
    | **中间删除** | O(n) | O(n) | ArrayList移动元素，LinkedList需要定位 |
    | **内存占用** | 较少 | 较多 | LinkedList每个节点额外存储两个指针 |
    
    **使用场景建议：**
    - **ArrayList**：频繁随机访问、遍历操作多、内存敏感的场景
    - **LinkedList**：频繁在头尾插入删除、不需要随机访问的场景

12. **HashMap的底层实现原理？**
    
    **HashMap核心结构：**
    - **JDK 1.7**：数组 + 链表
    - **JDK 1.8+**：数组 + 链表 + 红黑树
    
    **详细实现原理：**
    
    **1. 数据结构演进：**
    ```java
    // JDK 1.8+ HashMap核心结构
    public class HashMap<K,V> {
        static final int DEFAULT_INITIAL_CAPACITY = 16;  // 默认容量
        static final float DEFAULT_LOAD_FACTOR = 0.75f; // 负载因子
        static final int TREEIFY_THRESHOLD = 8;          // 链表转红黑树阈值
        static final int UNTREEIFY_THRESHOLD = 6;        // 红黑树转链表阈值
        
        Node<K,V>[] table;  // 哈希桶数组
        int size;           // 键值对数量
        int threshold;      // 扩容阈值
        
        // 链表节点
        static class Node<K,V> {
            final int hash;
            final K key;
            V value;
            Node<K,V> next;
        }
        
        // 红黑树节点
        static final class TreeNode<K,V> extends Node<K,V> {
            TreeNode<K,V> parent;
            TreeNode<K,V> left;
            TreeNode<K,V> right;
            TreeNode<K,V> prev;
            boolean red;
        }
    }
    ```
    
    **2. Hash计算过程：**
    ```java
    // 计算hash值
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
    
    // 计算数组索引
    int index = (table.length - 1) & hash;
    ```
    
    **3. Put操作流程：**
    ```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
        // 1. 如果table为空，进行初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
            
        // 2. 计算索引，如果该位置为空，直接插入
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            // 3. 该位置有元素，处理冲突
            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;  // key相同，覆盖
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);  // 红黑树插入
            else {
                // 链表插入
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1)  // 链表长度>=8，转红黑树
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
        }
        
        // 4. 检查是否需要扩容
        if (++size > threshold)
            resize();
    }
    ```
    
    **4. 扩容机制：**
    - **触发条件**：size > threshold（capacity * loadFactor）
    - **扩容大小**：容量翻倍
    - **重新hash**：所有元素重新计算位置
    
    **5. 红黑树优化（JDK 1.8+）：**
    - **转换条件**：链表长度 ≥ 8 且 数组长度 ≥ 64
    - **退化条件**：红黑树节点数 ≤ 6
    - **性能提升**：最坏情况下查找时间从O(n)降到O(log n)

13. **HashSet如何保证元素不重复？**
    
    **HashSet底层实现：**
    ```java
    public class HashSet<E> extends AbstractSet<E> {
        private transient HashMap<E,Object> map;  // 底层使用HashMap
        private static final Object PRESENT = new Object();  // 固定value值
        
        public HashSet() {
            map = new HashMap<>();
        }
        
        public boolean add(E e) {
            return map.put(e, PRESENT) == null;  // 元素作为key，PRESENT作为value
        }
        
        public boolean contains(Object o) {
            return map.containsKey(o);
        }
    }
    ```
    
    **去重机制详解：**
    
    **1. 两步判断过程：**
    ```java
    // 添加元素的完整过程
    public boolean add(E element) {
        // 第一步：计算hashCode
        int hash = element.hashCode();
        int index = hash & (table.length - 1);
        
        // 第二步：在对应桶中查找
        Node<E> current = table[index];
        while (current != null) {
            // 先比较hash值（性能优化）
            if (current.hash == hash && 
                // 再比较equals（确保逻辑正确）
                (current.key == element || element.equals(current.key))) {
                return false;  // 元素已存在，添加失败
            }
            current = current.next;
        }
        
        // 元素不存在，添加成功
        addNode(hash, element);
        return true;
    }
    ```
    
    **2. hashCode()和equals()的重要性：**
    ```java
    public class Person {
        private String name;
        private int age;
        
        // 必须重写hashCode()和equals()
        @Override
        public int hashCode() {
            return Objects.hash(name, age);
        }
        
        @Override
        public boolean equals(Object obj) {
            if (this == obj) return true;
            if (obj == null || getClass() != obj.getClass()) return false;
            Person person = (Person) obj;
            return age == person.age && Objects.equals(name, person.name);
        }
    }
    
    // 使用示例
    HashSet<Person> set = new HashSet<>();
    set.add(new Person("张三", 25));
    set.add(new Person("张三", 25));  // 重复元素，不会被添加
    System.out.println(set.size());  // 输出：1
    ```
    
    **3. 性能分析：**
    - **平均时间复杂度**：O(1)
    - **最坏时间复杂度**：O(n)（所有元素hash冲突）
    - **JDK 1.8优化**：链表长度≥8时转红黑树，最坏情况O(log n)

14. **Iterator和ListIterator的区别？**
    
    **Iterator接口详解：**
    ```java
    public interface Iterator<E> {
        boolean hasNext();     // 是否有下一个元素
        E next();             // 获取下一个元素
        void remove();        // 删除当前元素（可选操作）
    }
    
    // 使用示例
    List<String> list = Arrays.asList("A", "B", "C");
    Iterator<String> it = list.iterator();
    while (it.hasNext()) {
        String element = it.next();
        System.out.println(element);
        // it.remove();  // 可以安全删除当前元素
    }
    ```
    
    **ListIterator接口详解：**
    ```java
    public interface ListIterator<E> extends Iterator<E> {
        // Iterator的方法
        boolean hasNext();
        E next();
        void remove();
        
        // ListIterator特有的方法
        boolean hasPrevious();     // 是否有前一个元素
        E previous();             // 获取前一个元素
        int nextIndex();          // 下一个元素的索引
        int previousIndex();      // 前一个元素的索引
        void set(E e);           // 替换当前元素
        void add(E e);           // 在当前位置插入元素
    }
    
    // 使用示例
    List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
    ListIterator<String> lit = list.listIterator();
    
    // 正向遍历
    while (lit.hasNext()) {
        String element = lit.next();
        if ("B".equals(element)) {
            lit.set("B_Modified");  // 修改元素
            lit.add("B_Added");     // 插入新元素
        }
    }
    
    // 反向遍历
    while (lit.hasPrevious()) {
        String element = lit.previous();
        System.out.println(element);
    }
    ```
    
    **详细对比表：**
    | 特性 | Iterator | ListIterator |
    |------|----------|-------------|
    | **适用集合** | 所有Collection | 仅List |
    | **遍历方向** | 单向（向前） | 双向（前后） |
    | **删除操作** | remove() | remove() |
    | **修改操作** | 不支持 | set() |
    | **插入操作** | 不支持 | add() |
    | **索引操作** | 不支持 | nextIndex(), previousIndex() |
    | **起始位置** | 集合开头 | 任意位置（通过listIterator(index)） |
    
    **使用场景：**
    - **Iterator**：简单的单向遍历和删除操作
    - **ListIterator**：需要双向遍历、修改、插入操作的场景

15. **Collections和Collection的区别？**
    
    **Collection接口详解：**
    ```java
    // Collection是集合框架的根接口
    public interface Collection<E> extends Iterable<E> {
        // 基本操作
        int size();
        boolean isEmpty();
        boolean contains(Object o);
        Iterator<E> iterator();
        Object[] toArray();
        
        // 修改操作
        boolean add(E e);
        boolean remove(Object o);
        
        // 批量操作
        boolean containsAll(Collection<?> c);
        boolean addAll(Collection<? extends E> c);
        boolean removeAll(Collection<?> c);
        boolean retainAll(Collection<?> c);
        void clear();
    }
    
    // Collection的主要实现类
    // List: ArrayList, LinkedList, Vector
    // Set: HashSet, LinkedHashSet, TreeSet
    // Queue: PriorityQueue, ArrayDeque
    ```
    
    **Collections工具类详解：**
    ```java
    // Collections是操作集合的工具类
    public class Collections {
        
        // 排序操作
        public static <T extends Comparable<? super T>> void sort(List<T> list);
        public static <T> void sort(List<T> list, Comparator<? super T> c);
        
        // 查找操作
        public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key);
        public static <T> T max(Collection<? extends T> coll);
        public static <T> T min(Collection<? extends T> coll);
        
        // 修改操作
        public static void reverse(List<?> list);
        public static void shuffle(List<?> list);
        public static <T> void fill(List<? super T> list, T obj);
        
        // 创建不可变集合
        public static <T> Set<T> singleton(T o);
        public static <T> List<T> singletonList(T o);
        public static <K,V> Map<K,V> singletonMap(K key, V value);
        
        // 创建同步集合
        public static <T> Collection<T> synchronizedCollection(Collection<T> c);
        public static <T> List<T> synchronizedList(List<T> list);
        public static <T> Set<T> synchronizedSet(Set<T> s);
    }
    ```
    
    **常用方法示例：**
    ```java
    List<Integer> numbers = new ArrayList<>(Arrays.asList(3, 1, 4, 1, 5, 9));
    
    // 排序
    Collections.sort(numbers);
    System.out.println(numbers);  // [1, 1, 3, 4, 5, 9]
    
    // 反转
    Collections.reverse(numbers);
    System.out.println(numbers);  // [9, 5, 4, 3, 1, 1]
    
    // 打乱
    Collections.shuffle(numbers);
    System.out.println(numbers);  // 随机顺序
    
    // 查找最大最小值
    int max = Collections.max(numbers);
    int min = Collections.min(numbers);
    
    // 二分查找（需要先排序）
    Collections.sort(numbers);
    int index = Collections.binarySearch(numbers, 4);
    
    // 创建不可变集合
    List<String> immutableList = Collections.singletonList("Hello");
    Set<String> immutableSet = Collections.singleton("World");
    
    // 创建线程安全集合
    List<String> syncList = Collections.synchronizedList(new ArrayList<>());
    ```
    
    **核心区别总结：**
    | 特性 | Collection | Collections |
    |------|------------|-------------|
    | **类型** | 接口 | 工具类 |
    | **作用** | 定义集合规范 | 提供集合操作方法 |
    | **实例化** | 不能直接实例化 | 不能实例化（私有构造器） |
    | **继承关系** | 被List、Set等继承 | 继承Object |
    | **方法类型** | 抽象方法 | 静态方法 |
    | **使用方式** | 通过实现类使用 | 直接调用静态方法 |

## 🎯 第一阶段学习建议

### 学习重点
1. **扎实掌握Java语法基础**
2. **深入理解面向对象编程思想**
3. **熟练使用集合框架**
4. **培养良好的编程习惯**

### 实践项目
- 实现一个简单的学生管理系统
- 编写各种排序算法
- 实现简单的数据结构（栈、队列、链表）

### 时间安排
- **每日学习时间：** 2-3小时
- **理论学习：** 1-1.5小时
- **编程实践：** 1-1.5小时
- **周末项目：** 4-6小时

---

**下一阶段预告：** 第二阶段将深入学习多线程编程、IO操作、反射机制等Java进阶技术。