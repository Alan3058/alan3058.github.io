---
layout: post
title: [Java IO总结]
categories: [Java]
tags: [java io,javaio,总结]
id: [18792730066944]
fullview: false
---

Java IO是Java最基本、最主要的框架之一，在工作中，其使用率也是非常的频繁。故而再次梳理总结下Java IO框架知识，增强理解。

总得来说，IO流可分为字符流和字节流，而每种流又可分为输入和输出流，故而可将IO流分为四类：InputStream、OutputStream、Reader、Writer。字节流可通过InputStreamReader，OutputStreamWriter类转换成字符流。

File文件流：FileInputStream、FileOutStream、FileReader、FileWriter。

Buffered缓冲流：BufferedInputStream、BufferedOutputStream、BufferedReader、BufferedWriter。

字节字符缓冲区流：ByteArrayInputStrean、ByteArrayOutputStream、CharArrayReader、CharArrayWriter。

针对对象序列化，java专门提供了对象序列化输入输出字节流：ObjectInputStream、ObjectOutputStream。

String字符流：StringReader、StringWriter。

基本java数据类型流：DataInputStream、DataOutputStream。

通道流：PipedInputStream、PipedOutputStream、PipedReader、PipedWriter。


对行号输入流：LineNumberInputStream（已过时，操作字符流建议使用后者）、LineNumberReader。

其他：PrintStream、PrintWriter。


