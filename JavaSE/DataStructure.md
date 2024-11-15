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
java里太多相似的关键字了，static, final就是典型。困扰了我很久，所以现在