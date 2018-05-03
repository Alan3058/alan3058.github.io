---
layout: post
title: [Maven——Maven依赖范围和传递依赖]
categories: [开发工具]
tags: [maven,依赖范围,传递依赖]
id: [18734009810944]
fullview: false
---
# 依赖范围

Maven添加依赖包时，有一个scope的选项，该选项是范围的意思，在Maven中标识依赖范围。Maven有如下几种依赖范围。  
1. compile：编译依赖范围，默认使用该依赖范围。即对编译、测试、运行都有效。  
2. test：测试依赖范围。只对测试有效，在进行编译和运行是将无法使用此依赖包。  
3. provided：已提供依赖范围。只对编译和测试有效，在运行时将无法使用此依赖包。  
4. runtime：运行时依赖范围。只对测试和运行有效，在编译时将无法使用该依赖包。  
5. system：系统依赖范围。其范围和provided一样，不同的是它必须指定systemPath元素显示指定依赖文件的路径。即与本地系统有关，与仓库无关。一般不使用。  

# 传递依赖

传递依赖，顾名思义，依赖带有传递性。即A依赖B，B依赖C，那么A可能依赖C。然而传递依赖和依赖范围有关。以下是一个传递表格（其中左边第一列代表A依赖B，顶部首行代表B依赖C，中间部分就是B对C的依赖范围）

| |compile|test|provided|runtime|
|-|-|-|-|-|
|compile|compile|-|-|runtime|
|test|test|-|-|test|
|provided|provided|-|provided|provided|
|runtime|runtime|-|-|runtime|

当在同一个项目中出现依赖同一软件不同版本时，这时Maven会自动帮我选择。Maven有如下两个选择原则。

1. 路径最短优先。  
2. 第一声明优先。  
如A->B->C(1.0) D->F->G->C(2.0),这时项目会选择最短路径C(1.0)版本包。  
如A->B->C(1.0) D->H->C(2.0),这时会选择优先声明的依赖包C(1.0)。  

# 查看依赖信息

mvn dependency:list 查看当前项目已解析的依赖。  
mvn dependency:tree 查看当前项目的依赖树。  
mvn dependency:analyze 分析当前依赖包信息。需要关注Used undeclared dependencies，指的是项目中已使用但未声明的依赖。Unused declared dependencies，指的是项目中未使用但已声明的依赖。可以从这两个点去查看当前依赖包情况。
