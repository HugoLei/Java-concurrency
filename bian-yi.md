# 静态成员 VS. 普通成员

源代码

```
public class Common {
	private Integer age = 3;
	private static Integer money = 4;
}
```

反编译javap -c -private Common.class

```
Compiled from "Common.java"
public class testjava.loadclass.Common {
  private java.lang.Integer age; // 普通成员

  private static java.lang.Integer money; // 静态成员

  static {}; // 自动添加static block，将静态成员的赋值操作包含进来
    Code:
       0: iconst_4
       1: invokestatic  #11                 // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       4: putstatic     #17                 // Field money:Ljava/lang/Integer;
       7: return

  public testjava.loadclass.Common(); // 自动构建无参构造函数，将成员变量的赋值操作包含进来
    Code:
       0: aload_0
       1: invokespecial #22                 // Method java/lang/Object."<init>":()V
       4: aload_0
       5: iconst_3
       6: invokestatic  #11                 // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       9: putfield      #24                 // Field age:Ljava/lang/Integer;
      12: return
}
```



