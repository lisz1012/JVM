# Runtime Data Area and Instruction Set

jvms 2.4 2.5

## 指令集分类

1. 基于寄存器的指令集
2. 基于栈的指令集
   Hotspot中的Local Variable Table = JVM中的寄存器

## Runtime Data Area

PC 程序计数器

> 存放指令位置
>
> 虚拟机的运行，类似于这样的循环：
>
> while( not end ) {
>
> ​	取PC中的位置，找到对应位置的指令；
>
> ​	执行该指令；
>
> ​	PC ++;
>
> }

JVM Stack

1. Frame - 整个JVM 的线程stack包括一个个的栈帧，每个方法对应一个栈帧，每个栈帧都有自己的一个操作数栈和Dynamic linking
   1. Local Variable Table 参数也是局部变量，栈帧一弹出就没了，这就是为什么除了这个大括号就没人人是局部变量了。
   2. Operand Stack，也就是说，栈帧里面有一个小栈
      对于long的处理（store and load），多数虚拟机的实现都是原子的
      jls 17.7，没必要加volatile
   3. Dynamic Linking
       https://blog.csdn.net/qq_41813060/article/details/88379473 
      jvms 2.6.3
      Dynamic Linking指向我们运行时常量池，就是Class文件里面那个constant pool里面的那个符号链接：这个方法叫什么名字、类型是什么等，
      看看他有没有解析，如果还没有就进行动态解析，如果已经解析了就直接拿过来用，非动态解析应该是Resolution那一步做的事情。比如：
      A方法调用了B方法，找B的时候就去常量池里面找，这个link就叫Dynamic Linking
   4. return address
      a() -> b()，方法a调用了方法b, b方法的返回值放在什么地方（a方法栈的栈顶），以及b执行完了之后应该回到那里继续执行。

TestPlusPlus.java 的执行：先提一下，这里局部变量有两个下标为0的是args，下标为1的是i
代码：
```java
public static void main(String[] args) {
        int i = 8;
        i = i++;
        System.out.println(i);
    }
```
故事的情节是这样的：
```
 0 bipush 8                  把8压进操作数栈 
 2 istore_1                  栈顶的那个int数字出栈，并放到局部变量表中下标值为1的那个局部变量i中，至此，int i = 8 完成
 3 iload_1                   从局部变量表下标为1的那个变量里，往外面取值，放到栈里面
 4 iinc 1 by 1               局部变量表下标为1的那个变量自增1，得到8+1=9
 7 istore_1                  重复操作2，栈顶的那个int数字出栈，并放到局部变量表中下标值为1的那个局部变量i中，8覆盖了9
 8 getstatic #2 <java/lang/System.out>  开始打印
11 iload_1
12 invokevirtual #3 <java/io/PrintStream.println>
15 return
```
换成++i：
```
public static void main(String[] args) {
        int i = 8;
        i = ++i;
        System.out.println(i);
    }
```
编译之后：
```
 0 bipush 8
 2 istore_1
 3 iinc 1 by 1
 6 iload_1
 7 istore_1
 8 getstatic #2 <java/lang/System.out>
11 iload_1
12 invokevirtual #3 <java/io/PrintStream.println>
15 return
```
指令集有两大类：
1 基于栈的指令集。特点是简单  
2 基于寄存器的指令集。相对复杂，但是速度快  
Hotspot的local variable那张表，就非常类似于寄存器。虚拟机无论怎么设计，到了硬件层面都是基于寄存器的

Heap

Method Area 有以下两个实现：

1. Perm Space (<1.8)
   字符串常量位于PermSpace
   FGC不会清理
   大小启动的时候指定，不能变
2. Meta Space (>=1.8)
   字符串常量位于堆
   会触发FGC清理
   不设定的话，最大就是物理内存

Runtime Constant Pool

Native Method Stack

Direct Memory

每个线程有自己的PC，为了线程间切换

> JVM可以直接访问的内核空间的内存 (OS 管理的内存)
>
> NIO ， 提高效率，实现zero copy

思考：

> 如何证明1.7字符串常量位于Perm，而1.8位于Heap？
>
> 提示：结合GC， 一直创建字符串常量，观察堆，和Metaspace



## 常用指令

store

load

pop

mul

sub

invoke

1. InvokeStatic
2. InvokeVirtual
3. InvokeInterface
4. InovkeSpecial
   可以直接定位，不需要多态的方法
   private 方法 ， 构造方法
5. InvokeDynamic
   JVM最难的指令
   lambda表达式或者反射或者其他动态语言scala kotlin，或者CGLib ASM，动态产生的class，会用到的指令

