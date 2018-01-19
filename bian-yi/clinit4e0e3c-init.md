# &lt;clinit&gt;与&lt;init&gt;

这是由Java编译器自动生成的方法。

# &lt;clinit&gt;

若类包含static成员变量，或者static{}代码块，则编译器自动生成clinit\(\)方法。

```
// 源代码
public class Common {
    private static Integer age = 3;
}
```

class文件

* 编译器自动添加了无参构造函数
* 编译器自动添加了static{}代码块
* 在static{}中调用了

```
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

# &lt;init&gt;

若类定义了构造函数，或者非static成员变量在声明时被赋值了，则编译器自动生成init\(\)方法。

```
// 源代码
public class Common {
    private Integer age = 3;
}
```

class文件

* 编译器自动添加了无参构造函数
* 在构造函数中调用了init

```
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



