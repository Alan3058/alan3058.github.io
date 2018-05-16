---
layout: post
title: [Gradle下的Task和Action]
categories: [Gradle]
tags: [gradle,task,action,doLast]
id: [17384222897471488]
fullview: false
---

Task是Gradle一个重要的组件，官方解释：Task代表构建下的一个单一原子工作块，比如编译类或者生成javadoc。它是寄生在Project下，并且可以通过TaskContainer类的方法去创建和查询task。

一个Task是由多个Action组成，当执行一个task时，它下面的所有Action都会被执行（通过调用Action的execute()方法）。可通过doLast或者doFirst方法给Task添加Action实例。如下

```bash
task test1 << {
    println "---test---"
}
task test2 {
    doLast{
        println "test last"
    }
}
task test3 {
    doFirst{
        println "test first"
    }
}
```

其中<<是doLast的简写，由于使用时有歧义，所以在gradle4中不建议使用<<符号简写，并申明在gradle5中可能会移除。

注意如下写法，表示该任务在创建时就已经执行了。直接输入gradle，如下语句就会被执行。

```bash
task test{
    println "test"
}
```


