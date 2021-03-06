---
layout: post
title: [开发C#串口程序二]
categories: [Csharp]
tags: [Csharp,串口程序]
id: [18652123456944]
fullview: false
---

第一篇[开发C\#串口程序一](/160222/Csharp-serail-port1)的代码都是零零碎碎的，现将完整类CommPortHelper代码整理贴出来。

```java
/*
 * 串口工具类
 * 用户： Alan
 * 日期: 2016-02-06
 * 时间: 10:45
 * 
 */
using System;
using System.IO;
using System.IO.Ports;
using System.Reflection;
using System.Text;
using System.Windows.Forms;

using log4net;

namespace CommPort
{
    /// <summary>
    /// Description of CommPortHelper.
    /// </summary>
    public class CommPortHelper
    {
        private ILog log = LogManager.GetLogger(MethodBase.GetCurrentMethod().DeclaringType);
        private SerialPort serialPort;
        private StringBuilder data = new StringBuilder();
        private static String configFilePath = Application.StartupPath+"/config.ini";
        
        public CommPortHelper()
        {
            initSerailPort();
            openSerailPort();
        }
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
        
        /// <summary>
        /// 转换奇偶校验位
        /// </summary>
        /// <param name="parity"></param>
        /// <returns></returns>
        public Parity convertParity(String parity)
        {
            switch(parity.ToLower())
            {
                case "even":
                    return Parity.Even;
                case "odd":
                    return Parity.Odd;
                case "mark":
                    return Parity.Mark;
                case "space":
                    return Parity.Space;
                default:
                    // 其余默认为none
                    return Parity.None;
            }
            
        }
        /// <summary>
        /// 转换停止位
        /// </summary>
        /// <param name="stopBits"></param>
        /// <returns></returns>
        public StopBits convertStopBits(String stopBits)
        {
            switch(stopBits)
            {
                case "1":
                    return StopBits.One;
                case "1.5":
                    return StopBits.OnePointFive;
                case "2":
                    return StopBits.Two;
                default:
                    // 其余默认为none
                    return StopBits.One;
            }
        }
        
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
        /// <summary>
        /// 初始化配置文件
        /// </summary>
        public static void initConfigFile(){
            IniFileUtil.writeIniData("install","PortName","COM1",configFilePath);
            IniFileUtil.writeIniData("install","BaudRate","9600",configFilePath);
            IniFileUtil.writeIniData("install","DataBits","8",configFilePath);
            IniFileUtil.writeIniData("install","StopBits","1",configFilePath);
            IniFileUtil.writeIniData("install","Parity","none",configFilePath);
        }
        
        /// <summary>
        /// 串口写入16进制字符
        /// </summary>
        /// <param name="str"></param>
        public void writeHexString(string str){
            byte[] bytes = HexUtil.strToHexBytes(str);
            serialPort.Write(bytes,0,bytes.Length);
        }
        /// <summary>
        /// 串口写入16进制数组
        /// </summary>
        /// <param name="bytes"></param>
        public void writeHexArray(byte[] bytes){
            serialPort.Write(bytes,0,bytes.Length);
        }
        /// <summary>
        /// 串口写入字符串
        /// </summary>
        /// <param name="str"></param>
        public void write(string str){
            serialPort.Write(str);
        }
        /// <summary>
        /// 串口写字符串，带换行符
        /// </summary>
        /// <param name="str"></param>
        public void writeLine(string str){
            serialPort.WriteLine(str);
        }
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
        /// <summary>
        /// 获取串口接收的数据，并清理数据
        /// </summary>
        /// <returns></returns>
        public string getData(){
            String rs = data.ToString();
            data.Length=0;
            return rs;
        }
        
    }
}
```


