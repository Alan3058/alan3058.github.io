---
layout: post
title: [开发C#串口程序一]
categories: [C#]
tags: [c#,串口程序]
id: [18650123730944]
fullview: false
---

# 1.背景


因工作需求，需要实时读取电子称的重量数据。一开始时其实没啥想法，压根就不太理解什么是串口，COM1、COM2等是什么东东（不过对于它们的数据交互心里还是有一个大致的思路![](http://img.baidu.com/hi/jx2/j_0059.gif)![](http://img.baidu.com/hi/jx2/j_0059.gif)![](http://img.baidu.com/hi/jx2/j_0059.gif)）。经过一翻周折，查找了一堆资料，最后才有点似懂非懂。好了，不扯蛋了。。。

C\#开发串口程序需要使用SerialPort类，查看微软官方帮助文档，在文档里描述的非常详细（说实话，真心佩服微软的官方文档，太详细了，估计这也是它强大的原因吧）。该类里有几个比较重要的属性：串口号、波特率、数据位、停止位、校验位、流控，当然write写数据的方法、read读数据的方法是必不可少的。了解了这几个东东，开发一个简单的串口读写程序基本上ok了。

# 2.实操

1.首先看初始化串口属性代码，如下

```java
         /// <summary>
        /// 初始化序列号串口对象
        /// </summary>
        private void initSerailPort(){
            //判断配置文件是否存在
            if(!File.Exists(configFilePath)){
                FileStream fs = File.Create(configFilePath);
                fs.Close();
                initConfigFile();
            }
            string portName,baudRate,dataBits;
            serialPort = new SerialPort();
            //初始化序列号串口对象属性
            if((portName = IniFileUtil.readIniData("install","PortName",configFilePath))!=string.Empty){
                //串口号
                serialPort.PortName = portName.ToUpper();
            }
            if((baudRate = IniFileUtil.readIniData("install","BaudRate",configFilePath))!=string.Empty){
                //波特率
                serialPort.BaudRate = int.Parse(baudRate);
            }
            if((dataBits = IniFileUtil.readIniData("install","DataBits",configFilePath))!=string.Empty){
                //数据位
                serialPort.DataBits = int.Parse(dataBits);
            }
            //停止位
            serialPort.StopBits = convertStopBits(IniFileUtil.readIniData("install","StopBits",configFilePath));
            //校验位
            serialPort.Parity = convertParity(IniFileUtil.readIniData("install","Parity",configFilePath));
            //绑定串口对象接收数据方法
            serialPort.DataReceived += new SerialDataReceivedEventHandler(dataReceivedHandler);
            
        }
```

2.之后是读取串口数据的方法，其实就是使用SerialPort类的Read方法读取。

```java
         /// <summary>
        /// 串口数据接收处理
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void dataReceivedHandler(object sender, SerialDataReceivedEventArgs e)
        {
            SerialPort serialPortObj = (SerialPort)sender;
            byte[] bytes = new byte[serialPortObj.BytesToRead];
            int length = serialPortObj.Read(bytes,0,bytes.Length);
            StringBuilder tempString = new StringBuilder();
            for(int i=0;i<length;i++){
                string str = Convert.ToString(bytes[i],16);
                if(str.Length==1){
                    str = "0" + str;
                }
                tempString.Append(" "+str);
            }
            data.Append(" "+tempString.ToString().Trim());
            log.Info(tempString.ToString().Trim());
        }
```

3.接下来是打开串口

```java
         /// <summary>
        /// 打开串口
        /// </summary>
        public void openSerailPort(){
            if(serialPort.IsOpen){
                MessageBox.Show(serialPort.PortName+"串口已被占用");
                return;
            }else{
                serialPort.Open();
            }
            
        }
```

基本上，有了这三步，一个读取串口数据小程序就大功告成。当然实际情况可能还需要增加一些配置文件处理等功能来丰富该程序，我这里还增加了http服务监听(使用HttpListener类，该类可查询官方文档)，ini文件读写（涉及到调用windows类库），开机自启动（涉及到注册表的使用）。

以下是Http服务器监听的代码片段。其实很简单，和java的socket实现差不了太多，就是将需要监听的url交给HttpListener对象，开启另一个的线程去监听http请求。

```java
         public void start(){
            if (!HttpListener.IsSupported)
            {
                MessageBox.Show ("Windows XP SP2 or Server 2003 is required to use the HttpListener class.");
                return;
            }
            // URI prefixes are required,
            if (this.prefixes == null || this.prefixes.Length == 0){
                throw new ArgumentException("prefixes");
            }

            // Create a listener.
            listener = new HttpListener();
            // Add the prefixes.
            foreach (string s in prefixes)
            {
                listener.Prefixes.Add(s);
            }
            listener.Start();
            log.Info("Listening...");
            Thread thread = new Thread(processRequest);
            thread.Start();
        }

         /// <summary>
        /// 处理Http请求
        /// </summary>
        private void processRequest()
        {
            // Note: The GetContext method blocks while waiting for a request.
            while(true){
                HttpListenerContext context = listener.GetContext();
                HttpListenerRequest request = context.Request;
                // Obtain a response object.
                HttpListenerResponse response = context.Response;
                Uri uri = request.Url;
                string path = uri.AbsolutePath;
                string responseString = "";
                if(path == "/hello"||path == "/hello/"){
                    
                    responseString = "Hello world";
                }
                byte[] buffer = System.Text.Encoding.UTF8.GetBytes(responseString);
                // Get a response stream and write the response to it.
                response.ContentLength64 = buffer.Length;
                Stream output = response.OutputStream;
                output.Write(buffer,0,buffer.Length);
                // You must close the output stream.
                output.Close();
            }
        }
```

待续中。。。 [开发C\#串口程序二](http://ctosb.com/article/18652123456944)

