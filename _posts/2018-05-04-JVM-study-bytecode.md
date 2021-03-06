---
layout: post
title:  "JVM study——字节码学习"
categories: [java,jvm]
tags: [java,jvm,bytecode]
fullview: false
---
# 百科
维基上是这样描述的：java字节码是jvm虚拟机可以识别的一套指令集。虽然java程序员不需要知道或者理解java字节码，但是IBM网站还是建议：理解字节码和java编译器可能产生什么样的字节码，这个对java程序员很有帮助。  

java指令集可以分以下几大类：
1. Load（入栈） and store（出栈）(eg:aload_0,istore_1)
2. 算术和逻辑(eg:iadd,fcmpl)
3. 类型转换(eg:i2b,d2i)
4. 对象创建和对象操作(eg:new,putfield)
5. 操作数栈管理(eg:swap,dup2)
6. 控制流程(eg:ifeq,goto)
7. 方法执行和返回(eg:invokespecial,areturn,return)

许多指令都有一个前缀或者后缀去表示操作数的类型，如下

|前后缀|操作数类型|
|-|-|
|i|integer|
|l|long|
|s|short|
|b|byte|
|c|character|
|f|float|
|d|double|
|a|reference|

如iadd，aload，istore。以上信息翻译[维基百科java bytecode](https://en.wikipedia.org/wiki/Java_bytecode)。

> 关于更多字节码信息，可看[维基百科字节码列表](https://en.wikipedia.org/wiki/Java_bytecode_instruction_listings)  
> [字节码指令集](https://www.cnblogs.com/tenghoo/p/jvm_opcodejvm.html)  
> [java字节码指令集](https://blog.csdn.net/github_35983163/article/details/52945845)  

# 例子
JDK版本：jdk8  
java类进行编译成class文件，再通过javap命令查看class字节码信息。如下例子，是一个简单的java测试类。
```java
import org.junit.Test;

/**
 * 字节码测试类
 * @author liliangang-1163
 * @date 2018/5/4 10:10
 * @see
 */
public class BytecodeTest {

	@Test
	public void test() {
		short i = 1234;
		String a = "abcd";
		String b = "efgh";
		String c = "abcdefgh";
		String d = "abcd" + "efgh";
		String e = a + b;
		String f = a + "efgh";
		System.out.println(i);
		System.out.println(c == d);
		System.out.println(e == d);
		System.out.println(f == d);
		System.out.println(f == e);
	}
}
```

# 反编译class成字节码
通过javap后得到如下字节码信息。并且我已经加上了中文注释，方便查看

```java
public class BytecodeTest {
  public BytecodeTest();
    Code:
	   //将this变量入栈
       0: aload_0
	   //执行<init>初始化实例方法
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
	   //void返回
       4: return

  public void test();
    Code:
	   //将2字节内大小范围（short范围）的数字（1234）入栈
       0: sipush        1234
	   //将栈顶的int类型值（即0步骤的数据1234）保存到局部变量1中
       3: istore_1
	   //常量池的常量（abcd）入栈
       4: ldc           #2                  // String abcd
	   //将栈顶引用类型值（当前栈顶值abcd）保存到局部变量2中
       6: astore_2
	   //常量池的常量（efgh）入栈
       7: ldc           #3                  // String efgh
	   //将栈顶引用类型值（当前栈顶值efgh）保存到局部变量3中
       9: astore_3
	   //常量池的常量（abcdefgh）入栈
      10: ldc           #4                  // String abcdefgh
	  //将栈顶引用类型值（当前栈顶值abcdefgh）保存到局部变量4中
      12: astore        4
	  //常量池的常量（abcdefgh）入栈
      14: ldc           #4                  // String abcdefgh
	  //将栈顶引用类型值（当前栈顶值abcdefgh）保存到局部变量5中
      16: astore        5
	  //创建StringBuilder对象
      18: new           #5                  // class java/lang/StringBuilder
	  //复制栈顶的值（不明白为啥要这个操作）
      21: dup
	  //调用特殊处理的实例方法（实例初始化方法）
      22: invokespecial #6                  // Method java/lang/StringBuilder."<init>":()V
	  //将局部变量2的引用类型推送到栈顶
      25: aload_2
	  //调用StringBuilder对象的实例方法append，将栈顶变量当作参数
      26: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
	  //将局部变量3的引用类型推送到栈顶	
      29: aload_3
	  //调用StringBuilder对象的实例方法append，将栈顶变量当作参数
      30: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
	  //调用StringBuilder对象的实例方法toString，返回字符串，并将结果的字符串置入栈顶。
      33: invokevirtual #8                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
	  //将栈顶引用类型值（当前栈顶值abcdefgh）保存到局部变量6中
      36: astore        6
      38: new           #5                  // class java/lang/StringBuilder
      41: dup
      42: invokespecial #6                  // Method java/lang/StringBuilder."<init>":()V
      45: aload_2
      46: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
	  //常量池的常量（efgh）入栈
      49: ldc           #3                  // String efgh
      51: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      54: invokevirtual #8                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
  	  //将栈顶引用类型值（当前栈顶值abcdefgh）保存到局部变量7中
      57: astore        7
	  //获取静态变量out
      59: getstatic     #9                  // Field java/lang/System.out:Ljava/io/PrintStream;
	  //从局部变量1的int值入栈
      62: iload_1
	  //执行实例方法println打印
      63: invokevirtual #10                 // Method java/io/PrintStream.println:(I)V
      66: getstatic     #9                  // Field java/lang/System.out:Ljava/io/PrintStream;
	  //从局部变量4的引用类型值入栈
      69: aload         4
	  //从局部变量5的引用类型值入栈
      71: aload         5
	  //比较栈顶两个引用类型是否相等
      73: if_acmpne     80
	  //相等的话，1入栈
      76: iconst_1
	  //否则跳转下一步
      77: goto          81
	  //否则0入栈
      80: iconst_0
	  //调用out的实例方法println进行打印
      81: invokevirtual #11                 // Method java/io/PrintStream.println:(Z)V
      84: getstatic     #9                  // Field java/lang/System.out:Ljava/io/PrintStream;
      87: aload         6
      89: aload         5
      91: if_acmpne     98
      94: iconst_1
      95: goto          99
      98: iconst_0
      99: invokevirtual #11                 // Method java/io/PrintStream.println:(Z)V
     102: getstatic     #9                  // Field java/lang/System.out:Ljava/io/PrintStream;
     105: aload         7
     107: aload         5
     109: if_acmpne     116
     112: iconst_1
     113: goto          117
     116: iconst_0
     117: invokevirtual #11                 // Method java/io/PrintStream.println:(Z)V
     120: getstatic     #9                  // Field java/lang/System.out:Ljava/io/PrintStream;
     123: aload         7
     125: aload         6
     127: if_acmpne     134
     130: iconst_1
     131: goto          135
     134: iconst_0
     135: invokevirtual #11                 // Method java/io/PrintStream.println:(Z)V
	 //void返回
     138: return
}
```

# 总结

如上字节码所示，我们可以知道为何变量c和变量d是相等的，因为在编译阶段，编译器把"abcd"+"efgh"直接编译成常量"abcdefgh"字符串，并且赋值给d。
当两个字符串常量进行+运算时，编译器会直接计算出它们的结果，并赋值给变量（int、long、byte、float、double类型也类似）。但当计算过程中包含变量，那么编译器会通过StringBuilder对象去对字符串进行append方法累加。   

在非静态方法中aload_0存储的是this变量，之后从aload_1 --> aload_xxx表示是以方法入参和方法局部变量顺序自增1，静态方法中没有this变量，则直接从0开始累加表示方法入参和方法局部变量。

