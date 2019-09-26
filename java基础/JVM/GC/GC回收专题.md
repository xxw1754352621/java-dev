# GC回收专题

## GC Roots 集合（**引用类型**）-根节点

- 虚拟机栈中的引用对象

- 方法区中的类静态属性引用的对象

- 方法区中的常量引用的对象

- 本地方法栈中JNI（native方法）引用的对象

  ```properties
  Local variables 本地变量
  Static variables 静态变量
  JNI References JNI引用等
  ```

  

## 垃圾回收器选者

| 参数 | 新生代 | 老年代 |
| ---- | ------ | ------ |
|      |        |        |
|      |        |        |
|      |        |        |

## JVM参数查看

- java -XX:+PrintFlagsFinal 默认参数
- -XX:+PrintCommandLineFlags  修改的参数