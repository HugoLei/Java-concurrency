# class文件

## class文件

查看class文件的命令，javap -verbose Common.class

### Constant pool

```text
Constant pool:
   #1 = Class              #2             // testjava/loadclass/Common
   #2 = Utf8               testjava/loadclass/Common
   #3 = Class              #4             // java/lang/Object
   #4 = Utf8               java/lang/Object
   #5 = Utf8               <init>
   #6 = Utf8               ()V
   #7 = Utf8               Code
   #8 = Methodref          #3.#9          // java/lang/Object."<init>":()V
   #9 = NameAndType        #5:#6          // "<init>":()V
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               Ltestjava/loadclass/Common;
  #14 = Utf8               SourceFile
  #15 = Utf8               Common.java
```

常量池显示的解读：

一共两列

* 第一列：索引值 = 元素类型，例如`#1 = Class`，表示的是常量池的第一个元素，该元素的类型是`Class`
* 第二列：该元素中保存的值，例如`#1 = Class #2`，表示该元素存的值在常量池索引是2的位置里，该例中存的是字符串`testjava/loadclass/Common`

## clinit\(\) 和 init\(\)

这是由Java编译器自动生成的方法。

## &lt;clinit&gt;

若类包含static成员变量，或者static{}代码块，则编译器自动生成clinit\(\)方法。

```text
// 源代码
public class Common {
    private static Integer age = 3;
}
```

class文件

* 编译器自动添加了无参构造函数
* 编译器自动添加了static{}代码块
* 在static{}中调用了

```text
public class testjava.loadclass.Common {
  private static java.lang.Integer age;

  static {};
    Code:
       0: iconst_3
       1: invokestatic  #10                 // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       4: putstatic     #16                 // Field age:Ljava/lang/Integer;
       7: return

  public testjava.loadclass.Common();
    Code:
       0: aload_0
       1: invokespecial #21                 // Method java/lang/Object."<init>":()V
       4: return
}
```

## &lt;init&gt;

若类定义了构造函数，或者非static成员变量在声明时被赋值了，则编译器自动生成init\(\)方法。

```text
// 源代码
public class Common {
    private Integer age = 3;
}
```

class文件

* 编译器自动添加了无参构造函数
* 在构造函数中调用了init

```text
public class testjava.loadclass.Common {
  private java.lang.Integer age;

  public testjava.loadclass.Common(); // 编译器自动添加了无参构造函数
    Code:
       0: aload_0
       1: invokespecial #10                 // Method java/lang/Object."<init>":()V
       4: aload_0
       5: iconst_3
       6: invokestatic  #12                 // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       9: putfield      #18                 // Field age:Ljava/lang/Integer;
      12: return
}
```

