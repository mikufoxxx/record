# 目录
## 数据类型
### 基本数据类型和包装类型
#### 问题背景
在学习[JavaGuide-JavaSE(上)](https://javaguide.cn/java/basis/java-basic-questions-01.html#%E8%87%AA%E5%A2%9E%E8%87%AA%E5%87%8F%E8%BF%90%E7%AE%97%E7%AC%A6)的时候，看到基本类型和包装类型总是理解的不是很全面，所以写下这篇，便于理解。
#### 前置知识
**基本类型**包括$ byte, short, int, long, float, double,char,boolean $
**包装类型**包括$ Byte, Short, Integer, Long, Float, Double, Character, Boolean $
**自动装箱**指将*基本类型*自动转换为对应的*包装类型*
**自动拆箱**指将*包装类型*自动转换为对应的*基本类型*
#### 详细说明
- 基本数据类型
  - 存储在栈中，不需要额外的内存开销
  - 有默认值
- 包装类型
  - 存储在堆中，需要额外的内存来存储对象头和引用
  - 默认值一般为null
  - 可以调用方法
  - List、Set、Map等集合只能存储包装类型
    - 这里提醒一点
    ```java
    //我们平时经常会使用List这种数据结构
    List<Integer> list = new ArrayList<>();
    list.add(10); 
    // 这里我们其实用到了自动装箱，因为泛型里是包装类
    ```
  
---
在自动拆箱的时候，我们可能会遇到空值处理的问题【因为包装类的默认值是null】，这时候我们需要进行异常处理
```java
public class NullHandlingExample {
    public static void main(String[] args) {
        Integer wrapperInt = null;
        int primitiveInt = 0;

        try {
            primitiveInt = wrapperInt; // 自动拆箱，抛出NullPointerException
        } catch (NullPointerException e) {
            System.out.println("NullPointerException caught");
        }

        if (wrapperInt != null) {
            primitiveInt = wrapperInt; // 安全拆箱
        }

        System.out.println("Primitive Int: " + primitiveInt);
    }
}
```
---
>关于值的比较

### 包装类型的缓存机制补充
在学习[包装类型的缓存机制](https://javaguide.cn/java/basis/java-basic-questions-01.html#%E5%8C%85%E8%A3%85%E7%B1%BB%E5%9E%8B%E7%9A%84%E7%BC%93%E5%AD%98%E6%9C%BA%E5%88%B6%E4%BA%86%E8%A7%A3%E4%B9%88)的时候，虽然提到了大部分包装类型都用到了缓存机制来提升性能。但是我真的不知道到底咋提升的。所以下面我来解释一下，以Integer为例

```java
 private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];//声明为final，所以缓存的对象会被放入常量池中；声明为static，所以是在类加载的时候就创建好了
    //创建-128～127的值的包装类对象
    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```
上面的代码的意思就是，包装类首先初始化[-128（固定）, 127（可修改）]的包装类类型的数组【本质是数组】，当程序想通过自动装箱机制/new出来一个包装类对象的时候，会首先判断数值是否在这个范围内，如果满足，那么会从该常量池重寻找指定的数值，然后直接指向这个包装类对象就行【常量池是所有线程共享的】。

如果不在这个范围，再去新建，这样就可以提升性能了。

### static和final的个人理解
java里太多相似的关键字了，static, final就是典型。困扰了我很久，所以现在详细记录一下
#### static
1. static可以修饰成员变量，常量，方法和代码块
2. 静态成员是全局的，归整个类所有，不依赖特定的对象，是被所有类的对象共享的
3. 只要类被java虚拟机加载，就可以根据类名在全局数据域内找到他们

也就是说，静态成员**被类的所有实例对象共享**。
#### 被static修饰的属性(成员变量)称为静态变量，也叫做类变量； 
访问方式不需要再通过对象来访问，而是可以直接通过类来访问
```java
class User{
    public static int userId;
 
    public int getUserId() {
        return userId;
    }
 
    public void setUserId(int userId) {
        this.userId = userId;
    }
}
 
public class Test {
    public void test() {
        // 没有static的话，就是
        // User user = new User();
        // user.userId = 1;
        User.userId = 1;
    }
}
```
#### 被static修饰的方法称为静态方法，也叫做类方法
和成员变量类似，修饰后可以直接通过类来访问

**静态方法中只可以调用静态方法**
```java
class User{
    private static int userId;
 
    public int getUserId() {
        return userId;
    }
 
    public static void setUserId(int userId) {
        User.userId = userId;
    }
}
 
public class Test {
    public void test() {
      // 没有static的话，就是
      // User user = new User();
      // user.setUserId(1);
        User.setUserId(1);
    }
}
```
#### 被static修饰的常量称为静态常量
被static修饰的成员变量、常量、方法都**归整个类所有**，改变都是类似的

和前面修饰成员变量和方法操作类似
```java
class User{
    private static int userId;
    public static String username = "admin"; 
 
    public int getUserId() {
        return userId;
    }
 
    public static void setUserId(int userId) {
        User.userId = userId;
    }
}
 
public class Test {
    public void test() {
        User.username = "SpecialFox";
    }
}
```
#### 被static修饰的代码块叫做静态代码块； 
执行优先级高于非静态的初始化块，它会在**类初始化的时候执行一次，执行完成便销毁**，它仅能初始化类变量，即 static 修饰的数据成员。

而非静态初始化块会在构造函数执行时，在构造函数主体代码执行之前被运行。
> 静态代码块的执行顺序：静态代码块----->非静态代码块-------->构造函数
> 
> 静态代码块被 static 修饰，是属于类的，只在项目运行后类初始化时执行一次
> 
> 而非静态代码块在每个对象生成时都会被执行一次

#### 被static修饰符的内部类，叫做静态内部类
在外部类声明static，程序不会过编译

所以必须在类的内部声明一个内部类

>静态内部类跟静态方法一样，只能访问静态的成员变量和方法，不能访问非静态的方法和属性，但是普通内部类可以访问任意外部类的成员变量和方法
> 
> 静态内部类可以声明普通成员变量和方法，而普通内部类不能声明static成员变量和方法。
> 
> 静态内部类可以单独初始化

```java
public class Outer {
    private String name;
    private int age;
 
    public static class Builder {
        private String name;
        private int age;
 
        public Builder(int age) {
            this.age = age;
        }
 
        public Builder withName(String name) {
            this.name = name;
            return this;
        }
 
        public Builder withAge(int age) {
            this.age = age;
            return this;
        }
 
        public Outer build() {
            return new Outer(this);
        }
    }
 
    private Outer(Builder b) {
        this.age = b.age;
        this.name = b.name;
    }
}
```
静态内部类调用外部类的构造函数，来构造外部类，由于静态内部类可以被单独初始化说有在外部就有以下实现：
```java
public Outer getOuter()
{
    Outer outer = new Outer.Builder(2).withName("SpecialFox").build();
    return outer;
}
```
#### 为什么我们要用静态类？
1. 如果类的构造器或静态工厂中有多个参数，设计这样类时，最好使用Builder模式，特别是当大多数参数都是可选的时候。
2. 如果现在不能确定参数的个数，最好一开始就使用构建器即Builder模式。

#### Final
> final 修饰类，不存在子类，比如String类
> 
> final修饰方法，子类不能重写。
> 
> final进行修饰属性，为常量，需要初始化，并且不可修改 ，常量命名通常用大写字母，每个字母中间用下划线隔开
> 
> final进行修饰属性，子类可以使用，但也不能修改

重点是理解final这个"最终"的意思，理解了自然就可以想到：

- final修饰变量，即表示该变量是常量
- final修饰方法后就定死了，不能被子类重写
- final修饰类后也定死了，不能被继承

> 用final修饰的int型，String型一旦设定进不能改变，但是修饰数组时，可以修改数组的某个位置的值，但是不能修改数组空间。
