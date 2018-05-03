---
layout: post
title: [Java IO和NIO]
categories: [Java]
tags: [java io,java nio]
fullview: false
---
很久以前就听说过Java NIO，但似乎在工作中很少使用，我们大部分还是在使用Java IO。这两天突发奇想，变想起了NIO这东东。相对来讲，IO是每次传输字还是移动一个字节，而NIO每次传输是一个数据块，并且IO是阻塞型的而NIO是非阻塞型。这样导致NIO处理效率较IO高等特点。

以下是Java IO和NIO测试类
package com.ctosb.sample; import java.io.BufferedReader; import java.io.Closeable; import java.io.File; import java.io.FileInputStream; import java.io.FileOutputStream; import java.io.FileReader; import java.io.FileWriter; import java.io.IOException; import java.nio.ByteBuffer; import java.nio.channels.FileChannel; public class FileUtil { //*/* /* 创建文件 /* /* @param fileName /* @return /* @throws IOException /* @author Alan /* @date 2016年7月4日 下午7:35:20 /*/ public File createFile(String fileName) throws IOException { File file = new File(fileName); if (file.isFile()) { createFile(file.getParent()); } if (!file.exists()) { if (file.isDirectory()) { file.mkdirs(); } else { file.createNewFile(); } } return file; } //*/* /* 关闭 closeable /* /* @param closeable /* @author Alan /* @date 2016年7月4日 下午7:38:58 /*/ public void close(Closeable closeable) { if (closeable != null) { try { closeable.close(); } catch (IOException e) { e.printStackTrace(); } } } //*/* /* 写文件 /* /* @param fileName /* @param content /* @throws IOException /* @author Alan /* @date 2016年7月4日 下午7:35:46 /*/ public void writeFile(String fileName, String content) throws IOException { FileWriter fileWriter = new FileWriter(createFile(fileName), false); fileWriter.append(content); close(fileWriter); } //*/* /* 读取文件 /* /* @param fileName /* @return /* @throws IOException /* @author Alan /* @date 2016年7月4日 下午7:35:55 /*/ public String readFile(String fileName) throws IOException { FileReader fileReader = new FileReader(createFile(fileName)); StringBuffer stringBuffer = new StringBuffer(); String content; BufferedReader bufferedReader = new BufferedReader(fileReader); while ((content = bufferedReader.readLine()) != null) { stringBuffer.append(content); } close(fileReader); return stringBuffer.toString(); } //*/* /* 以nio方式写 /* /* @param fileName /* @param content /* @throws IOException /* @author Alan /* @date 2016年7月4日 下午7:36:02 /*/ public void writeFileByNio(String fileName, String content) throws IOException { FileOutputStream fileOutputStream = new FileOutputStream(createFile(fileName), false); FileChannel fileChannel = fileOutputStream.getChannel(); ByteBuffer byteBuffer = ByteBuffer.allocate(content.getBytes().length); byteBuffer.put(content.getBytes()); byteBuffer.flip();// 每次写之前需要flip fileChannel.write(byteBuffer); fileOutputStream.flush(); close(fileChannel); close(fileOutputStream); } //*/* /* 以nio方式读取 /* /* @param fileName /* @return /* @throws IOException /* @author Alan /* @date 2016年7月4日 下午7:36:16 /*/ public String readFileByNio(String fileName) throws IOException { FileInputStream fileInputStream = new FileInputStream(createFile(fileName)); FileChannel fileChannel = fileInputStream.getChannel(); StringBuffer stringBuffer = new StringBuffer(); ByteBuffer byteBuffer = ByteBuffer.allocate(1024); while ((fileChannel.read(byteBuffer)) != -1) { byteBuffer.clear();// 每次读取之前都需要clear stringBuffer.append(new String(byteBuffer.array())); } close(fileChannel); close(fileInputStream); return stringBuffer.toString(); } public static void main(String[] args) { FileUtil fileUtil = new FileUtil(); try { StringBuffer content = new StringBuffer(); for (int i = 0; i < 10000000; i++) { content.append(new Random().nextInt(10000000) + "\r\n"); } long fmtime = System.currentTimeMillis(); fileUtil.writeFile("d:\\test.txt", content.toString()); long totime = System.currentTimeMillis(); System.out.println("fileutil-write:" + (totime - fmtime) /* 1.0 / 1000); fileUtil.readFile("d:\\test.txt"); long totime1 = System.currentTimeMillis(); System.out.println("fileutil-read:" + (totime1 - totime) /* 1.0 / 1000); long nfmtime = System.currentTimeMillis(); fileUtil.writeFileByNio("d:\\test.txt", content.toString()); long ntotime = System.currentTimeMillis(); System.out.println("fileutilnio-write:" + (ntotime - nfmtime) /* 1.0 / 1000); fileUtil.readFileByNio("d:\\test.txt"); long ntotime1 = System.currentTimeMillis(); System.out.println("fileutilnio-read:" + (ntotime1 - ntotime) /* 1.0 / 1000); } catch (IOException e) { e.printStackTrace(); } } }

在处理NIO时与IO不一样，NIO主要使用Channel和Buffer这两个类（其实还有个Selector类）。并且在读取前要执行Buffer的clear方法，在写前要执行Buffer的flip方法。

在这里我测试了一千万行数据，然而似乎并没有得到传说中说的那么快。如下结果
fileutil-write:0.469 fileutil-read:1.389 fileutilnio-write:0.795 fileutilnio-read:0.635

基本上nio读取会比io快1倍多，然而nio的写会比io慢到快1倍。

之后翻查资料才发现IBM官方中如下一段话
在 JDK 1.4 中原来的 I/O 包和 NIO 已经很好地集成了。 java.io./* 已经以 NIO 为基础重新实现了，所以现在它可以利用 NIO 的一些特性。例如， java.io./* 包中的一些类包含以块的形式读写数据的方法，这使得即使在更面向流的系统中，处理速度也会更快。

也就是说，1.4后的Java IO是以NIO重新实现。所以NIO的速度并不会比超过IO太多。使用NIO主要是为了考虑其他的特性，比如

1、非阻塞性I/O

2、文件锁定功能

具体功能待续。。。![](http://img.baidu.com/hi/jx2/j_0028.gif)![](http://img.baidu.com/hi/jx2/j_0028.gif)![](http://img.baidu.com/hi/jx2/j_0028.gif)
