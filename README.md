# 清华校园网环境下-电脑的远程控制与无线网自动登录认证方法

---
## 1. 系统要求
&ensp;&ensp;&ensp;&ensp;只有专业版、教育版、服务器版win系统可以被用户远程桌面控制（服务器版可以实现多用户的同时远程桌面控制同一台电脑），而电脑通常自带的系统是&ensp; &ensp;只有专业版、教育版、服务器版win系统可以被用户远程桌面控制（服务器版可以实现多用户的同时远程桌面控制），而电脑通常自带的系统是家庭学生版win系统，是无法被windows自带的远程桌面进行控制的。因此，需要先升级为win11专业版（学校有正版教育版win系统，推荐专业版更面向个人）。    

&ensp;&ensp;&ensp;&ensp;[（win1家庭版系统升级为专业版系统参考教程）](https://www.bilibili.com/video/BV1se411g78q/?spm_id_from=333.337.search-card.all.click&vd_source=d7a9d2057ba27dae12daba18a11cf783)

---
## 2.无线网选取  
&ensp;&ensp;&ensp;&ensp;由于windows的安全机制限制，使得电脑在远程连接时，会自动断开802.1x类型的无线网（**Tsinghua-secure**）。因此，当台式机连接的是**Tsinghua-secure**无线网时，一旦进行远程连接，电脑就会自动断网，此时在远程端首先会显示黑屏，然后再显示远程桌面链接失败。  
&ensp;&ensp;&ensp;&ensp;可以使用（[临时解决方法](https://www.zhihu.com/question/419880471/answer/1845083292)）来处理，但当电脑重启之后就会失效，再次出现上述的问题。因此，推荐选择连接清华校园内的**Tsinghua-IPv4**无线网，由于此网不是802.1x类型，因此在远程控制时，系统不会自动断开网络。但是，**Tsinghua-IPv4**无线网在连接后需要首先登陆认证网页进行认证。因此，一旦电脑重启后，就需要手动进行联网认证登陆。为此，需要配置远程电脑的校园网<font face='黑体' color=#ff0000 size=4>**自动登陆认证**</font>，实现电脑远程重启后，依然能自动连接网络。

---
## 3.清华大学校园无线网自动登陆认证教程（实现电脑远程开机、重启后能自动连上无线网）  

[（教程参考链接）](https://zhuanlan.zhihu.com/p/339779399)  
  
**准备工具：** 
+ **auto-thu-wifi.bat：** 用于后续生成auto-thu-wifi.win64.exe软件
+ **auto-thu-wifi.win64.exe：** 无线校园网自动登陆认证软件  
+ **Bat_To_Exe_Converter_(x64).exe：** 批处理文件（.bat）转.exe工具  
+ **Instsrv.exe：** 系统服务注册程序
+ **srvany.exe：** 用于链接（相当于外壳），将任何.exe程序作为windows的系统服务运行
+ **conf.json：** 登陆配置文件（无线网的账号、密码）

**具体流程：**  
（注意配置文件中路径要与实际路径相同；路径名中不能有<font face='黑体' color=#ff0000 size=4>**中文**</font>；若路径名中含有<font face='黑体' color=#ff0000 size=4>**空格**</font>，则在.bat文件中要把对应的路径整体加上<font face='黑体' color=#ff0000 size=4>**单或双引号！！！**</font>）  
  
### 3.1 **创建一个auto-thu-wifi的文件夹：**  
后续软件和程序都放在该文件夹中。（注意：为避免被杀毒软件误删，需要将该文件夹添加到Defender的排除项中）  

<center>   

![Alt text](.\img\image1.png)
![Alt text](.\img\image2.png)  

</center>  

### 3.2 **设置认证登陆程序auto-thu-wifi.win64.exe的配置文件conf.json：**  
&ensp;&ensp;&ensp;&ensp;先将conf.json文件改为.txt后打开，再将其中"username"、"password"修改为自己的信息(需要带双引号)。具体内容如下：  
    {  
    "username": "username",  
    "password": "password"  
    }

### 3.3 **设置电脑在开机后，能自动调用校园无线网的登陆认证程序auto-thu-wifi.win64.exe**  
#### 3.3.1 **首先编辑批处理文件auto-thu-wifi.bat：**  

&ensp;&ensp;&ensp;&ensp;在文件夹中的auto-thu-wifi.txt文件中输入以下内容（注意根据自己的实际情况替换掉对应的路径），最后将后缀改为.bat即可得到对应的批处理程序auto-thu-wifi.bat。  

---
    ::-----------------------------------------------------------------
    ::--------------------------编辑部分--------------------------------

    :start
    ::测试是否不能正常访问百度网站(用于测试电脑是否能正常上网)
    ping www.baidu.com -n 1 > nul
    if %errorlevel% equ 0 (
    ::能正常上网就直接结束进程，避免重复联网以及中途的断网操作
        ::echo Website %website% is reachable. Not executing the program.
        Exit ::退出批处理程序
    ) else (
    ::不能正常上网时，就调用自动登录认证程序进行自动联网认证
        TIMEOUT /T 15 /NOBREAK ::第一次自动登录前先延时15s
        "D:\Program Files\some_tool\auto-thu-wifi\auto-thu-wifi.win64.exe" --config-file "D:\Program Files\some_tool\auto-thu-wifi\conf.json" auto-thu-wifi --keep-online 
        ::调用的auto-thu-wifi.win64.exe是自动登录认证软件，config-file是对应的配置文件，包含有认证的账号和密码
        TIMEOUT /T 50 /NOBREAK
        "D:\Program Files\some_tool\auto-thu-wifi\auto-thu-wifi.win64.exe" --config-file "D:\Program Files\some_tool\auto-thu-wifi\conf.json" deauth
        Exit ::退出批处理程序
    )
    goto start


    ::--------------------------编辑部分--------------------------------
    ::-----------------------------------------------------------------
---
**测试：** 在清华网络登录界面中选择“断开连接”，此时在浏览器中无法再正常上网。然后确保电脑依然是连接着Tsinghua-IPV4无线网的，此时双击运行auto-thu-wifi.bat，等待一段时间（大约20s左右），若系统自动连上网了，说明此时的批处理文件是正确的。  

#### 3.3.2 生成auto-thu-wifi.exe插件
&ensp;&ensp;&ensp;&ensp;利用**Bat_To_Exe_Converter_(x64).exe**将auto-thu-wifi.bat转换为.exe文件，便于后续通过srvany.exe将其作为win11的系统服务来运行。  
&ensp;&ensp;&ensp;&ensp;软件具体设置如下（**注意：** 打开Bat_To_Exe_Converter_(x64).exe后，可能过一段时间会报错为：invalid memory access，因此可以尽量快速的完成软件的导出）  
  
<center>  

![Alt text](.\img\image3.png)

</center>

**测试：** 在清华网络登录界面中选择断开连接，此时在浏览器中无法再正常上网。然后确保电脑依然是连接着Tsinghua-IPV4无线网的，此时双击运行auto-thu-wifi.exe，等待一段时间（大约20s左右），若系统自动连上网了，说明此时的转换后的.exe文件是正确的。


### 3.4 添加win11的系统服务（实现开机后，系统自动运行该自动登陆认证程序）：  
&ensp;&ensp;&ensp;&ensp;首先，要将**Tsinghua-IPV4**无线网设置为<font face='黑体' color=#ff0000 size=4>**“在信号范围内时自动连接”**</font>， 同时取消启动无线网的自动连接。从而实现电脑开机或重启后能首先自动连接上**Tsinghua-IPV4**无线网。
![Alt text](.\img\image7.png) 

&ensp;&ensp;&ensp;&ensp;接下来开始设置windows的系统服务。原理是使用srvany.exe来链接（相当于外壳），才能最终将我们自己的auto-thu-wifi.exe程序作为windows的系统服务运行。所以该过程可分为两步（都在cmd上执行）：  
&ensp;&ensp;&ensp;&ensp;第一步是利用instsev.exe将srvany.exe自身注册为系统服务，
需要在cmd中输入（要用管理员身份运行cmd，同时cmd的目录要和instsev.exe所在的目录一样）：

    instsrv.exe AutoThuWifi(自定义的系统服务名称) “D:\Program Files\some_tool\auto-thu-wifi\srvany.exe”  

&ensp;&ensp;&ensp;&ensp;特别注意：如果电脑是64位的系统，可能会报错Unable to find the file at the given path，这可能是由于instsrv.exe与srvany.exe都是32位系统，由于兼容问题造成的具体的原因与解决方法参考链接[（原因与解决方法）](https://www.cnblogs.com/soundcode/p/4027859.html)。简单操作就是把instsrv.exe与srvany.exe两个软件都复制到windows/system32中，然后执行（**要用管理员身份运行cmd，同时cmd的目录要和instsev.exe所在的目录一样**）：

    instsrv.exe AutoThuWifi(自定义的系统服务名称) “D:\Program Files\some_tool\auto-thu-wifi\srvany.exe”  
    
&ensp;&ensp;&ensp;&ensp;第二步是将真正需要启动运行的程序（auto-thu-wifi.exe）与：srvany.exe提供的外壳配置起来（设计注册标准对应参数的设置）。
    
    REG ADD HKLM\SYSTEM\CurrentControlSet\Services\AutoThuWifi\Parameters
    REG ADD HKLM\SYSTEM\CurrentControlSet\Services\AutoThuWifi\Parameters /v Application /t REG_SZ /d “D:\\Program Files\\some_tool\\AutoThuWifi\\auto-thu-wifi.exe”


（**注意：**如果报错，也可以打开注册表，手动在“HKLM\SYSTEM\CurrentControlSet\Services\”处创建AutoThuWifi\Parameter\文件夹，然后在Parameter\文件下创建值：
![Alt text](.\img\image4.png)

&ensp;&ensp;&ensp;&ensp;第三步是设置开机自动启动服务（实现开机后自动登录认证连上网）:
 &ensp;&ensp;&ensp;&ensp;**win系统设置路径** 搜索 > 服务 > AutoThuWifi，将其设置为 **“自动”**  

 ![Alt text](.\img\image5.png)

![Alt text](.\img\image6.png)  

**测试：** 在清华校园网网络登录界面选择断开连接，此时在浏览器中无法再正常上网。然后确保电脑的WiFi设置中是发现**Tsinghua-IPV4**无线网时能自动连接的，此时关机。然后再开机，等待1~2分钟后，查看“服务”程序，如果显示了**AutoThuWifi**服务是正在运行的，并且打开浏览器后电脑也能正常上网，说明该服务已创建成功。  
  
### 3.5设置电脑定时检测网络连接情况（若意外掉线，可自动重新登录认证校园无线网）  
&ensp;&ensp;&ensp;&ensp;设置电脑每1小时定时启动检测一次（**实用场景：** 当远程电脑突然断网后，此时无法再远程控制该电脑进行重启联网，利用这一定时检测方法可实现电脑自动的重新连上无线网。）
搜索 > 任务计划程序   

#### 3.5.1 在任务计划的windows文件夹下创建AutoThuWifi文件夹：  
<center>  

![Alt text](.\img\image8.png)   

</center> 
  
#### 3.5.2 在AutoThuWifi文件夹下创建定时启动任务：

<center>  

![Alt text](.\img\image9.png)  

</center>

#### 3.5.3 任务的具体设置如下：  
**“常规”栏：**
<center>  

![Alt text](.\img\image10.png)  
  
</center>  

**“触发器”栏：**  
<center>  

![Alt text](.\img\image11.png)

</center>  
  
**“操作”栏：**  
<center>  

![Alt text](.\img\image12.png)  

</center>  
  
**“设置”栏：**    
下面这个步骤中，要将“如果此任务已经运行，以下规则适用(IN):”选择为“停止现有实例”，否则上一次自启动的计划任务可能不会自动停止，此时再次到时间后的自启动将无法进行。  
  
<center>  

![Alt text](.\img\image13.png) 

</center>  
  
<font face='黑体' color=#ff0000 size=4>**注意：**</font>  
该定时启动计划同样可能被Defender报毒误删，所以先试定在几分钟后自启动运行这个计划。然后，可能发现在“任务计划程序”中，这个计划已经被删除消失了，此时需要去查Defender的保护记录，将该计划的对应的被删除记录取消执行，后续在“任务计划程序”中，这个计划将重新出现，并且能避免Defender后续再将这个计划自动给删除掉。

**测试：** 在清华网络登录界面中选择断开连接（为模拟远程控制时的突然断网场景，推荐用“远程桌面连接”来操作这一步，操作完成后，由于远程电脑断网，此时远程界面将显示断连无法再连接），此时在浏览器中无法再正常上网。然后确保电脑的WiFi设置中是发现**Tsinghua-IPV4**无线网时能自动连接的。
然后等待一段时间（到任务计划中服务下一次开始自动执行的时间之后再过1~2分钟）  
![Alt text](.\img\image14.png)  
查看“任务计划”程序，如果显示了**AutoThuWifi**服务上次的运行结果是“操作完成”或者“正在运行”并且打开浏览器后电脑也能正常上网，说明该计划服务已创建成功。  
  
---
## 4. 远程断网时的关机、开机与重启：  
  
&ensp;&ensp;&ensp;&ensp;当远程使用电脑时可能由于系统死机、网络问题等造成远程电脑断网，此时将无法远程控制电脑进行关机再开机、重启并重新联网。同时，在一些特殊场景下，可能需要使处于关机状态的电脑被远程开机。  
  
**具体解决方法（2种）：**
1. 方法一：在电脑的bios中将其设置为通电开机（此时电脑机箱的开机按钮将失效），然后将电脑的电源插到具有远程控制电源通断的插座上（如小米的智能插座3）,并将智能插座设置为功耗低于2w持续5min后自动断电。使用时，可先在远程的电脑系统中进行软件关机，需要开机时直接通过米家远程开启智能插座供电即可智能开机。
问题：这种方案的问题时，当远程电脑突然断网或死机后，将不能通过系统进行软件关机，只能直接关闭插座电源。这种直接断电的关机方式（相当于电脑经历了意外断电）对电脑系统的损害较大，尤其是对固态硬盘的伤害。

2. 方法二：使用可以接入米家等平台的开机卡，通过将电脑机箱的开机按钮跳接到开机卡然后再接回主板，从而实现模拟人按开机按钮进行开机、关机和重启的操作，避免了在意外情况下需要直接断电进行关机从而对电脑系统以及硬件造成损害的风险。
问题：这种方法和第一种方法有一个共同的缺点，不论是开机卡还是智能插座都无法连接进入需要认证的无线网或者免密的开放无线网，所以在校园网环境中需要注意；此外，虽然随着ACPI规范的推出，使得通过关键按钮进行关机的操作不会对电脑硬件造成损害了，但依然会对软件系统造成损害（可能不会保存任何已打开的文档并且可能会丢失数据）。
