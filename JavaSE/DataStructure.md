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