# class文件

查看class文件的命令，javap -verbose Common.class

## Constant pool

```
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



