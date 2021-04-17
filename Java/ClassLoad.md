## Class Object

1. 获取类对象

   ```java
   // getClass()
   Class<?> myCl = getClass();
   Class<?> c = o.getClass();
   
   // class literals as name.class
   c = int.class; // same as Integer.TYPE
   c = String.class;
   c = byte[].class;
   
   // For primitive types and void --> class literals
   c = Void.TYPE;
   c = Byte.TYPE;
   c = Integer.TYPE;
   //...
   ```

   > class literals: Class objects for known types
   >
   > 类字面量：已知类型的类对象

2. 类对象

   类对象包含元数据：该类上定义的方法、构造器、域等；在加载时该类一无所知，仍可以通过类对象访问类

   两个例子：

   - 获取两个类的公共父类
   - 获取一个类文件中全部deprecated方法

   class文件要能被JVM合法加载，需要符合特定的结构：

   - 魔法数字 CA FE BA BE
   - 类文件版本
   - 常量池
   - 类名
   - 继承信息（父类名称等）
   - 实现的接口
   - 域
   - 方法
   - 属性

   > class文件是二进制文件，可使用javap理解其内容

   常量池：包含类涉及的所有（在该类中或其他类中）方法、类、域、常数。字节码可以通过索引获取常量池对象

### Phase of Classloading

类加载：一个新的类添加到JVM进程的过程，新代码进入系统的唯一方式

1. Loading 加载

   加载字节数组（从文件系统、URL或其他位置中读取）

   `Classloader::defineClass()`：将类文件转换为类对象，是一个`protected`方法，不继承无法访问

   `defineClass()`创建类对象的基本框架，并进行一些基础的检查（如确认常量池中的常量是self-consistent）

2. 类链接

   Loading并不会创建完整的类对象，类仍不可用。在Loading以后，类必须被链接Linked，这一步分为三个阶段

   - Verification 验证
   - Preparation and resolution 准备和解析
   - Initialization 初始化

3. 验证

   Verification确认类文件符合预期，不违反JVM的安全模型

   JVM被设计为可以静态检查（大多时候），这会增加类加载时间，减少运行时时间（运行时不检查）

   Verification可以防止执行可能引起JVM crash或处于undefine/untest状态的字节码（阻止恶意Java字节码或非信息编译器）

   > default 方法加载机制：
   >
   > 在一个接口实现类加载时，会检查class文件是否有默认方法的实现，有则正常加载，没有则使用默认实现

4. 准备和解析

   内存分配，静态变量==准备==初始化

   在这一阶段，变量没有初始化，类的字节码没有执行；JVM检查在运行该类涉及任何类型都已知，如果有未知类型，JVM会加载其他类型；加载和发现过程迭代进行，直到达到一个稳定的类型集（这被称为是原加载类的==传递闭包==）

5. 初始化

   静态变量被初始化、静态初始化块执行

   这是JVM第一次从新加载的类中执行字节码；静态块完成后，类被全部加载

### 安全和类加载

Java程序可以从多种源（包含不可信源）动态加载类，需要一个安全架构来允许非可信代码安全运行

Java很多的安全特性是在类加载子系统中实现，其核心观念是：只有一种方式可以让新的可执行代码进入进程（即：类）