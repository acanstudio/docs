入门
======

给新手的建议
-------------

Nagios是一个非常强大且柔性化的软件，但可能需要不少心血来学习如何配置使之符合你所需，一旦掌握了它如何工作并怎样来工作时，你会觉得再也离不开它！ 对于初次使用Nagios的新手这有几个建议需要遵从：

    * 放松点 － 这会花些时间。不要指望它事情可以在转瞬间就搞掟，没有那么容易。设置好Nagios是一个费点事的工作，部分是由于对Nagios设置并不清楚，而还可能是由于并不清楚如何来监控现有网络（或者说如何监控会更好）。
    * 使用快速上手指南。本帮助给出了快速安装指南是给那些新手尽快地将Nagios安装到位并运行起来而写就的。在不到二十分钟之内可以安装并监控本地的系统，一旦完成了，就可以继续学习配置Nagios了。
    * 阅读文档。如果掌握Nagios运行机制，可以高效地配置它并且使之无所不能。确信已经阅读了这些文档(是"配置Nagios"和"基本操作"两章)。在更好地理解基础性配置之前可以对那些高级内容暂时不管。
    * 获得他人协助。如果已经阅读文档并检测了样本配置文件但仍然有问题，写一个EMail给nagios-users邮件列表并写清楚问题。由于在这个项目上我有不少事情要做，直接给我的邮件我可能无法回复，所以最好是求助于邮件列表，如果有较好的背景并且可以将问题描述清楚，或许有人可以指出如何正确来做。更多地信息请在这个链接http://www.nagios.org/support/下寻找。

快速安装指南
-------------

### 介绍

这些指南试图让你在二十分钟内用简单地指令操作下从源程序安装Nagios并监控你的本地机器。这里并不讨论那些高级指令对于95%以上的想起步的用户而言这是基础。

安装后该做的
-------------

一旦你正确地安装并使Nagios运行起来后，毫无疑问你不仅要监控你的主机，你需要审视一下更多的文档来做更多的事情... 

    * 监控Windows主机
    * 监控Linux/Unix主机
    * 监控Netware服务器
    * 监控路由器和交换机
    * 监控网络打印机
    * 监控公众服务平台

监控Windows主机
----------------

### 介绍

本文用来说明如何监控Windows主机的本地服务和特性，包括：

    * 内存占用率
    * CPU负载
    * Disk利用率
    * 服务状态
    * 运行进程
    * 在Windows主机上的公众化服务(如HTTP、FTP、POP3等)可以查阅监控公众化服务这篇文档

注意 如下的内容是假定你已经按照快速安装指南安装好了Nagios系统之后做的，下面所使用的样例配置文件(如commands.cfg、templates.cfg等)已经在安装过程中安装到位。

### 概览

对Windows机器的监控私有服务需要在机器上安装代理程序。代理将会在检测插件与Nagios服务之间起网关代理作用。如果没有在机器上安装代理的话，Nagios将无法对Windows私有服务或属性等进行监控。

在下面例子中，将在Windows机器上安装NSClient++外部构件并使用check_nt插件检测和与NSClient++构件进行通讯。如果你按照指南来安装的话，check_nt插件已经安装到了Nagios服务器上。

如果愿意，可以用其他的Windows代理(象NC_Net)替代NSClient++构件所起的作用－只是要稍稍改一下对应的命令和服务定义等。下面将只是讨论安装了NSClient++外部构件的情况。

### 步骤

为完成对Windows机器的检测，有几个步骤要做，它们是：
    * 确认一下首要条件；
    * 在Windows机器上安装代理(在本例中是安装NSClient++构件)；
    * 给Windows机器创建新的主机和服务对象定义；
    * 重启动Nagios守护进程。

### 已经做了什么？

为使过程简单，已经完成了少量配置文件的工作：
    * 已经把check_nt命令加入到了commands.cfg文件中，就可以直接使用check_nt插件来监控Windows服务；
    * 一个Windows机器的主机对象模板(命名为windows-server)已经在templates.cfg文件里创建好了，可以更容易地加入一个新的Windows主机对象定义。

常用的配置文件可以在/usr/local/nagios/etc/objects/目录里找到。如果愿意可以对里面的对象进行修改以适应你的要求。但是，如果你没有熟悉配置Nagios之前劝你不要这么做。开始时可以只是按照下面的指令操作来快速完成监控Windows机器。

### 首要条件

首次监控一台Winodws机器时需要对Nagios做点额外的工作，记住，仅仅是监控第一台Windows机器时需要做这些工作。

<pre>
vi /usr/local/nagios/etc/nagios.cfg

// 把下面这行最前面的#号去掉：
#cfg_file=/usr/local/nagios/etc/objects/windows.cfg
</pre>

刚才做的是什么呢？是让Nagios起用/usr/local/nagios/etc/objects/windows.cfg这个配置文件里的对象定义。在这个配置文件里可以加些Windows的主机与服务对象定义。该配置文件里已经包含有几个样例主机、主机组及服务对象定义。对于第一台Windows机器，可以只是简单地修改里面已经有的主机与服务对象定义而不要新创建一个。

### 安装Windows代理程序

在用Nagios监控Windows机器的私有服务之前，需要先在机器上安装代理程序。推荐使用NSClient++外部构件，它可以在http://sourceforge.net/projects/nscplus找到。如下指令可以安装一个基本的NSClient++外部构件，同时也配置好Nagios来监控这台Windows机器。
    * 从http://sourceforge.net/projects/nscplus站点下载最新稳定版的NSClient++软件包；
    * 展开软件包到一个目录下，如C:\NSClient++；
    * 打开一个命令行窗口并切换到C:\NSClient++目录下；
    * 用命令将NSClient++系统服务注册到系统里： nsclient++ /install
    * 用命令安装NSClient++系统托盘程序('SysTray'是大小写敏感的)：nsclient++ SysTray
    * 打开服务管理器并确认NSClientpp服务可以在桌面交互(看一下服务管理器里的'Log On'选项页)，如果没有允许桌面交互，点一下里面的选择项打开它。
    * 编辑NSC.INI文件(位于C:\NSClient++目录)并做如下修改：

     去掉在[modules]段里的列出模块程序的注释，除了CheckWMI.dll和RemoteConfiguration.dll；
     最好是修改一下在[Settings]段里的'password'选项；
     去掉在[Settings]段里的'allowed_hosts'选项注释，把Nagios服务所在主机的IP加到这一行里，或是置为空，让全部主机都可以联入；
     确认一下在[NSClient]段里的'port'选项里已经去掉注释并设置成'12489'(默认端口)；

    * 用命令启动NSClient++服务： nsclient++ /start
    * 如果安装正确，一个新的图标会出现在系统托盘里，是个黄圈里面有个黑色的'M'；
    * 完成了。这台Windows机器可以加到Nagios监控配置里了...

### 配置Nagios

为监控Windows机器下面要在Nagios配置文件里加几个对象定义。

编辑方式打开windows.cfg文件。
<pre>
vi /usr/local/nagios/etc/objects/windows.cfg

// 给Windows机器加一个新的主机对象定义以便监控。
// 如果是被监控的第一台Windows机器，可以只是修改windows.cfg文件里的对象定义。
// 修改host_name、alias和address域以符合那台Windows机器。
define host{
	use		windows-server	; Inherit default values from a Windows server template (make sure you keep this line!)
	host_name		winserver
	alias		My Windows Server
	address		192.168.1.2
	}
</pre>        

好了。下面可以加几个服务定义(在同一个配置文件里)以使Nagios监控Windows机器上的不同属性内容。如果是第一台Windows机器，可以只是修改windows.cfg里的服务对象定义。

用你刚刚加好的主机对象定义里的host_name来替换例子里的"winserver"。
加入下面的服务定义以监控运行于Windows机器上的NSClient++外部构件的版本。当到时间要升级Windows机器上的外部构件时这信息会很用有，因为它可以告知这台Windows机器上的NSClient++需要升级到最新版本。

<pre>
define service{
	use			generic-service
	host_name			winserver
	service_description	NSClient++ Version
	check_command		check_nt!CLIENTVERSION
	}

// 加入下面的服务定义以监控Windows机器的启动后运行时间。
define service{
	use			generic-service
	host_name			winserver
	service_description	Uptime
	check_command		check_nt!UPTIME
	}

// 加入下面的服务定义可监控Windows机器的CPU利用率，并在5分钟CPU负荷高于90%时给出一个紧急警报或是高于80%时给出一个告警警报。
define service{
	use			generic-service
	host_name			winserver
	service_description	CPU Load
	check_command		check_nt!CPULOAD!-l 5,80,90
	}

// 加入下面的服务定义可监控Windows机器的内存占用率，
// 并在5分钟内存占用率高于90%时给出一个紧急警报或是高于80%时给出一个告警警报。
define service{
	use			generic-service
	host_name			winserver
	service_description	Memory Usage
	check_command		check_nt!MEMUSE!-w 80 -c 90
	}

// 加入下面的服务定义可监控Windows机器的C:盘的磁盘利用率，
// 并在磁盘利用率高于90%时给出一个紧急警报或是高于80%时给出一个告警警报。
define service{
	use			generic-service
	host_name			winserver
	service_description	C:\ Drive Space
	check_command		check_nt!USEDDISKSPACE!-l c -w 80 -c 90
	}

// 加入下面的服务定义可监控Windows机器上的W3SVC服务状态，并在W3SVC服务停止时给出一个紧急警报。
define service{
	use			generic-service
	host_name			winserver
	service_description	W3SVC
	check_command		check_nt!SERVICESTATE!-d SHOWALL -l W3SVC
	}

// 加入下面的服务定义可监控Windows机器上的Explorer.exe进程，并在进程没有运行时给出一个紧急警报。
define service{
	use			generic-service
	host_name			winserver
	service_description	Explorer
	check_command		check_nt!PROCSTATE!-d SHOWALL -l Explorer.exe
	}
</pre>

都好了，已经加好了基础服务定义，可以监控Windows机器了，保存一下配置文件。

### 口令保护

如果想指定保存在Windows机器上NSClient++配置文件里的口令，可以修改check_nt命令定义，让它带着口令。编辑方式打开commands.cfg文件。

<pre>
vi /usr/local/nagios/etc/commands.cfg

// 修改check_nt命令的定义，带上"-s <PASSWORD>"命令参数(这里的PASSWORD 要换成你Windows机器的真正口令)，象这样：
define command{
	command_name	check_nt
	command_line	$USER1$/check_nt -H $HOSTADDRESS$ -p 12489 -s PASSWORD -v $ARG1$ $ARG2$
	}
</pre>

### 重启动Nagios

如果修改好Nagios配置文件，需要验证你的配置文件并重启动Nagios。
如果验证配置文件过程中有什么错误信息，在做下一步前一定要修正好配置文件。一定要保证验证过程中不再有出错信息后再启动或重启动Nagios！

监控Linux/Unix主机
-------------------

### 介绍

本文档描述了如果监控Linux/UNIX的"私有"服务和属性，如：

    * CPU负荷
    * 内存占用率
    * 磁盘利用率
    * 登录用户
    * 运行进程
    * 由Linux系统上的公众服务(HTTP、FTP、SSH、SMTP等)可以按照这篇监控公众服务文档。

注意 如下内容是假定已经按照快速安装指南安装并设置好Nagios。如下例子参考了样例配置文件(commands.cfg、templates.cfg等)里的对象定义，样例配置文件已经在安装过程中安装就位。

### 概览

[注意：本文档没有结束。推荐阅读文档NRPE外部构件里如何监控远程Linux/Unix服务器中的指令] 

有几种不同方式来监控远程Linux/UNIX服务器的服务与属性。一个是应用共享式SSH密钥运行check_by_ssh插件来执行对远程主机的检测。这种方法本文档不讨论，但它会导致安装有Nagios的监控服务器很高的系统负荷，尤其是你要监控成百个主机中的上千个服务时，这是因为要建立/毁构SSH联接的总开销很高。

另一种方法是使用NRPE外部构件监控远程主机。NRPE外部构件可以在远程的Linux/Unix主机上执行插件程序。如果是要象监控本地主机一样对远程主机的磁盘利用率、CPU负荷和内存占用率等情况下，NRPE外部构件非常有用。

监控路由器和交换机
-------------------

### 介绍

本文档将介绍如何来监控路由器和交换机的状态。一些便宜的"无网管"功能的交换机与集线器不能配置IP地址而且对于网络是不可见的组成构件，因而没办法来监控这种东西。稍贵些的交换机和路由器可以配置IP地址可以用PING检测或是通过SNMP来查询状态信息。

下面将描述如果来监控这些有网管功能的交换机、集线器和路由器：
    * 包丢弃率，平均回包周期RTA
    * SNMP状态信息
    * 带宽与流量

注意 如下指令是假定你已经按快速安装指南安装好Nagios。参考的样例配置是在已经按指南安装就位的配置文件(commands.cfg、templates.cfg等)。

### 概览

监控交换机与路由器可简可繁－主要是看拥有什么样设备与想监控什么内容。做为极为重要的网络组成构件，毫无疑问至少要监控一些基本状态。

交换机与路由器可以简单地用PING来监控丢包率、RTA等数据。如果交换机支持SNMP，就可以监控端口状态等，用check_snmp插件，也可以监控带宽(如果用了MRTG)，用check_mrtgtraf插件。

check_snmp插件只有当系统里安装了net-snmp和net-snmp-utils包后才编译。先确定插件已经在/usr/local/nagios/libexec目录里再继续做，如果没有这个文件，安装net-snmp和net-snmp-utils包并且重编译并重新安装Nagios插件包。

### 步骤

要监控交换机与路由器要有几步工作：

    * 第一时间执行些必备工作；
    * 给设备创建要监控的主机与服务对象定义；
    * 重启动Nagios守护进程。

### 已经做了什么？

为了让工作轻松点，几个配置任务已经做好了：
    * 两个命令定义(check_snmp和check_local_mrtgtraf)已经加到了commands.cfg文件中。可以用check_snmp和check_mrtgtraf插件来监控网络打印机。
    * 一个交换机模板(命名为generic-switch)已经创建在templates.cfg文件里。可以在对象定义里更容易地加一个新的交换机与路由器设备。

以上的监控配置文件可以在/usr/local/nagios/etc/objects/目录里找到。如果愿意可以修改这些定义或是加入其他适合需要的更好的定义。但推荐你最好是等到你熟练地掌握了Nagios配置之后再这么做。开始的时候，只要按上述的配置来监控网络里的路由器和交换机就可以了。

### 必备工作

要配置Nagios用于监控网络里的交换机之前，有必要做点额外工作。记住，这是首先要做的工作才能监控。
编辑Nagios的主配置文件

<pre>
vi /usr/local/nagios/etc/nagios.cfg

// 移除文件里下面这行的最前面的(#)符号
#cfg_file=/usr/local/nagios/etc/objects/switch.cfg
</pre>

为何要这么做？这是要让Nagios检查/usr/local/nagios/etc/objects/switch.cfg配置文件来找些额外的对象定义。在文件里可以增加有关路由器和交换机设备的主机与服务定义。配置文件已经包含了几个样本主机、主机组和服务定义。做为监控路由器与交换机的第一步工作是最好在样例的主机与服务对象定义之上修改而不是重建一个。

### 配置Nagios

需要做些对象定义以监控新的交换机与路由器设备。
打开switch.cfg文件进行编辑。

<pre>
vi /usr/local/nagios/etc/objects/switch.cfg

// 给要监控的交换机加一个新的主机对象定义。
// 如果这是第一台要监控的交换机设备，可以简单地修改switch.cfg里的样例配置。
// 修改主机对象里的host_name、alias和address域值来适用于监控。
define host{
	use		generic-switch		; Inherit default values from a template
	host_name	linksys-srw224p		; The name we're giving to this switch
	alias		Linksys SRW224P Switch	; A longer name associated with the switch
	address		192.168.1.253		; IP address of the switch
	hostgroups	allhosts,switches	; Host groups this switch is associated with
}
</pre>

### 监控服务

现在可以加些针对监控交换机的服务对象定义(在同一个配置文件)。如果是第一台要监控的交换机设备，可以简单地修改switch.cfg里的样例配置。

注意 替换样例定义里的"linksys-srw224p"主机名为你刚才定义的名字，是修改在host_name域。

### 监控丢包率和RTA

<pre>
// 增加如下的服务定义以监控自Nagios监控主机到交换机的丢包率和平均回包周期RTA，在一般情况下每5分钟检测一次。
define service{
	use			generic-service	; Inherit values from a template
	host_name		linksys-srw224p	; The name of the host the service is associated with
	service_description	PING		; The service description
	check_command		check_ping!200.0,20%!600.0,60%	; The command used to monitor the service
	normal_check_interval	5	; Check the service every 5 minutes under normal conditions
	retry_check_interval	1	; Re-check the service every minute until its final/hard state is determined
}
</pre>

这个服务的状态将会处于：
    * 紧急(CRITICAL)－条件是RTA大于600ms或丢包率大于等于60%；
    * 告警(WARNING)－条件是RTA大于200ms或是丢包率大于等于20%；
    * 正常(OK)－条件是RTA小于200ms或丢包率小于20%

### 监控SNMP状态信息

如果交换机与路由器支持SNMP接口，可以用check_snmp插件来监控更丰富的信息。如果不支持SNMP，跳过此节。

<pre>
// 加入如下服务定义到你刚才修改的交换机对象定义之中
define service{
	use			generic-service	; Inherit values from a template
	host_name		linksys-srw224p
	service_description	Uptime	
	check_command		check_snmp!-C public -o sysUpTime.0
}

// 在上述服务定义中的check_command域里，用"-C public"来指定SNMP共同体名称为"public"，
// 用"-o sysUpTime.0"指明要检测的OID(译者注－MIB节点值)。
// 如果要确保交换机上某个指定端口或接口的状态处于运行状态，可以在对象定义里加入一段定义：
define service{
	use			generic-service	; Inherit values from a template
	host_name		linksys-srw224p
	service_description	Port 1 Link Status
	check_command		check_snmp!-C public -o ifOperStatus.1 -r 1 -m RFC1213-MIB
}
</pre>

在上例中，"-o ifOperStatus.1"指出取出交换机的端口编号为1的OID状态。"-r 1"选项是让check_snmp插件检查返回一个正常(OK)状态，如果是在SNMP查询结果中存在"1"(1说明交换机端口处于运行状态)如果没找到1就是紧急(CRITICAL)状态。"-m RFC1213-MIB"是可选的，它告诉check_snmp插件只加载"RFC1213-MIB"库而不是加载每个在系统里的MIB库，这可以加快插件运行速度。

这就是给SNMP库的例子。有成百上千种信息可以通过SNMP来监控，这完全取决于你需要做什么和如果来做监控。祝你好运！

提示 通常可以用如下命令来寻找你想用于监控的OID节点(用你的交换机IP替换192.168.1.253)：snmpwalk -v1 -c public 192.168.1.253 -m ALL .1

### 监控带宽和流量

可以监控交换机或路由器的带宽利用率，用MRTG绘图并让Nagios在流量超出指定门限时报警。check_mrtgtraf插件(它已经包含在Nagios插件软件发行包中)可以实现。

<pre>
// 需要让check_mrtgtraf插件知道如何来保存MRTG数据并存入文件，以及门限等。
// 在例子中，监控了一个Linksys交换机。MRTG日志保存于/var/lib/mrtg/192.168.1.253_1.log文件中。
// 这就是我用于监控的服务定义，它可以用于监控带宽数据到日志文件之中...
define service{
	use			generic-service	; Inherit values from a template
	host_name		linksys-srw224p
	service_description	Port 1 Bandwidth Usage
	check_command		check_local_mrtgtraf!/var/lib/mrtg/192.168.1.253_1.log!AVG!1000000,2000000!5000000,5000000!10
}
</pre>

在上例中，"/var/lib/mrtg/192.168.1.253_1.log"参数传给check_local_mrtgtraf命令意思是插件的MRTG日志文件在这个文件里读写，"AVG"参数的意思是取带宽的统计平均值，"1000000,200000"参数是指流入的告警门限(以字节为单位)，"5000000,5000000"是输出流量紧急状态门限(以字节为单位)，"10"是指如果MRTG日志如果超过10分钟没有数据返回一个紧急状态(应该每5分钟更新一次)。

保存该配置文件

### 重启动Nagios

一旦给switch.cfg文件里加好新的主机与服务对象定义，就可以开始对路由器与交换机进行监控。为了开始监控，需要先验证配置文件再重新启动Nagios。

如果验证过程有有任何错误信息，修改配置文件再继续。一定要保证配置验证过程中没有错误信息再启动Nagios！

监控网络打印机
----------------

### 介绍

本文件描述了如何监控网络打印机。特别是有内置或外置JetDirect卡的HP惠普打印机设备，或是其他(象Troy PocketPro 100S或Netgear PS101)支持JetDirect协议的打印机。

check_hpjd插件(该命令是Nagios插件软件发行包的标准组成部分)可以用SNMP使能的方式来监控JetDirect兼容型打印机。该插件可以检查如下打印机状态：
    * 卡纸
    * 无纸
    * 打印机离线
    * 需要人工干预
    * 墨盒墨粉低
    * 内存不足
    * 开外壳
    * 输出托盘已满
    * 和其他...

注意 如下指令假定你已经按照快速安装指南安装好Nagios。可以参考安装好的样本配置文件(commands.cfg、templates.cfg等)。

### 概览

监控网络打印机的状态很简单。有JetDirect功能的打印机一般提供SNMP功能，可以用check_hpjd插件来检测状态。

check_hpjd插件只是当当前系统中安装有net-snmp和net-snmp-utils软件包时才会被编译和安装。要保证在/usr/local/nagios/libexec目录下有check_hpjd文件再继承，否则，要安装好net-snmp和net-snmp-utils软件包再重新编译安装Nagios插件包。

### 步骤

监控打印机需要做如下几步：
    * 做些事先准备工作；
    * 创建一个用于监控打印机的主机与服务对象定义；
    * 重启动Nagios守护进程。

### 已经做了什么？

为使这项工作更轻松，几个配置工作已经做好：
    * check_hpjd的命令定义已经加到了commands.cfg配置文件中，可以用check_hpjd插件来监控网络打印机；
    * 一个网络打印机模板(命名为generic-printer)已经在templates.cfg配置文件里创建好，可以更方便地加入一个新打印机设备的主机对象。

上面的监控配置文件可以在/usr/local/nagios/etc/objects/目录里找到。如果想做，可以修改里面的定义以更好地适用于你的情况。但是在此之前，推荐你要熟悉Nagios的配置之后再做。起初，最好只是按下面的大概修改一下以实现对网络打印机的监控。

### 事先准备工作

在配置Nagios用于监控网络打印机之前，有些额外工作，记住这是要对第一台打印机设备进行监控。
编辑Nagios的主配置文件。

<pre>
vi /usr/local/nagios/etc/nagios.cfg

// 移除下面这行最前面的(#)号：
#cfg_file=/usr/local/nagios/etc/objects/printer.cfg
</pre>

为何要这样？告诉Nagios查找/usr/local/nagios/etc/objects/printer.cfg文件以取得额外对象定义。该文件中将加入网络打印机设备的主机与服务对象定义。这个配置文件里已经包含有一个样本主机、主机组和服务定义。给第一台打印机设备做监控，可以简单地修改这个文件而不需重生成一个。

### 配置Nagios

需要创建几个对象定义以进行网络打印机的监控。

打开printer.cfg文件并编辑它。

<pre>
vi /usr/local/nagios/etc/objects/printer.cfg

// 增加一个你要监控的网络打印机设备的主机对象定义。
// 如果这是第一台打印机设备，可以简单地修改printer.cfg文件里的样本主机定义。
// 将合理的值赋在host_name、alias和address域里。
define host{
	use		generic-printer	; Inherit default values from a template
	host_name	hplj2605dn	; The name we're giving to this printer
	alias		HP LaserJet 2605dn	; A longer name associated with the printer
	address		192.168.1.30	; IP address of the printer
	hostgroups	allhosts	; Host groups this printer is associated with
}
</pre>

现在可以给监控的打印机加些服务定义(在同一个配置文件里)。如果是第一台被监控的网络打印机，可以简单地修改printer.cfg里的服务配置。

注意 要用你要刚刚加上的被监控打印机主机名替换样例对象"hplj2605dn"里的host_name域值。
按如下方式加好对打印机状态检测的服务定义。服务用check_hpjd插件来检测打印机状态，默认情况下每10分钟检测一次。SNMP共同体串是"public"。

<pre>
define service{
	use			generic-service		; Inherit values from a template
	host_name		hplj2605dn		; The name of the host the service is associated with
	service_description	Printer Status		; The service description
	check_command		check_hpjd!-C public	; The command used to monitor the service
	normal_check_interval	10	; Check the service every 10 minutes under normal conditions
	retry_check_interval	1	; Re-check the service every minute until its final/hard state is determined
}
</pre>

// 加入一个默认每10分钟进行一次的PING检测服务。用于检测RTA、丢包率和网络联接状态。
define service{
        use                     generic-service
        host_name               hplj2605dn
        service_description     PING
        check_command           check_ping!3000.0,80%!5000.0,100%
        normal_check_interval   10
        retry_check_interval    1
}
</pre>

### 重启动Nagios

一旦在printer.cfg文件里加好新的主机和服务对象定义就可以监控网络打印机。为了开始，应该先验证配置文件并重启动Nagios。
如果在验证配置过程中有任何错误信息，修改好配置文件再继续。保证验证过程完成且没有任何错误的情况下再重启动Nagios！

监控Netware服务器
------------------

### 介绍

本文档描述了如何对Netware服务器的"私有"服务和属性进行监控，象这些：
    * 内存占用率
    * 处理器利用率
    * 缓冲区使用情况
    * 活动的联接
    * 磁盘卷使用率
    * 由Netware服务器提供的公众服务(HTTP、FTP等)的监控可以按文档监控公众服务来做。

### 概览

TODO... 

注意 我在找一个志愿者来写就HOWTO文档。我只能接触到一台旧的Netware 4.11服务器，所以无法跟上形势需要。如果可以更新这个文档，请把它张贴到NagiosCommunity wiki里。

### 其他资源

Novell有一些有关Nagios如何来做Netware监控的文档，在网站的Cool Solutions栏目里，包括：
    * MRTGEXT: NLM module for MRTG and Nagios
    * Nagios: Host and Service Monitoring Tool
    * Nagios and NetWare: SNMP-based Monitoring
    * Monitor DirXML/IDM Driver States with Nagios
    * Check NDS Login ability with Nagios
    * NDPS/iPrint Print Queue Monitoring by Nagios
    * check_gwiaRL Plugin for Nagios 2.0

监控公众服务平台
-----------------

### 介绍

本文档介绍如何来监控一些公众化的服务、应用及协议。所谓的"公众"服务是指在网络里常见的服务－不管是本地网络还是因特网上都是这样。公众服务包括象－HTTP服务(WEB)、POP3、IMAP/SMTP、FTP和SSH。其实在日常使用中还有更多的基础服务。这些服务与应用，包括所依托的协议，可以被Nagios直接监控而不需要额外的支持。

与之相对，私有服务如果没有某些中间件做代理Nagios是无法监控的。主机上的私有服务比如说CPU负荷、内存占用率、磁盘利用率、当前登录用户、进程信息等。这些私有服务或属性不能暴露给外部客户。这样就要在这些被监控的主机上安装一些中间件做代理来取得此类信息。更多有关在不同类型主机上的私有服务的信息可以查阅如下文档：

    * 监控Windows主机
    * 监控Netware服务器
    * 监控Linux/Unix机器

提示 偶尔发现有些私有服务和应用的信息可以通过SNMP来做监控。SNMP代理可以让远程监控系统提取一些有用信息。更多有关SNMP来监控服务信息的内容可以查阅一下监控交换机和路由器。

注意 如下内容假定你已经按照快速安装指南安装好Nagios系统。如下的配置样例对象可以参考安装包里的commands.cfg和localhost.cfg两个配置文件。

### 监控服务的插件

当需要对一些应用、服务或协议进行监控时，可能已经有一个用于监控的插件了。Nagios的官方插件包带有插件可以用于监控很多种服务与协议，在插件包的contrib/目录下有很多可用的插件。而且在NagiosExchange.org网站上有许多额外的其他人写的插件可以试试看！
如果不巧没有找到要监控所需的插件，可以自己写一个。插件写起来很容易，不要怕它，读一读插件的开发这篇文档吧。
下面将会领略一下你迟早会用到的一些基础服务，每个服务都可以在Nagios插件包里找到对应的检测插件。下面就开始了...

### 创建一个主机对象定义

在监控一个服务之前需要先定义一个服务要绑定的主机对象。可以把主机对象定义放在cfg_file域所指向的任何一个对象配置文件里或是放在由cfg_dir域所指向的任何一个目录下的文件里。如果已经定义过一个主机对象了，可以跳过这一步。

在本例中，假定你想在一台主机上监控许多种服务，把这台主机命名为remotehost。可以把主机对象定义放在它自己的配置文件里或加到一个已有的对象定义文件里。这里有个remotehost对象的定义象是这样：
<pre>
define host{
	use		generic-host		; Inherit default values from a template
	host_name		remotehost		; The name we're giving to this host
	alias		Some Remote Host	; A longer name associated with the host
	address		192.168.1.50		; IP address of the host
	hostgroups		allhosts		; Host groups this host is associated with
	}
</pre>

现在可以有了一个可被监控的主机对象了，之后就可以定义所要监控的服务。有了主机对象定义，服务对象定义可以加在任何一个对象配置文件的任何一个位置里。

### 创建服务对象定义

在Nagios里每个要监控的服务都必须给出一个绑定在刚才定义出的主机上的一个服务对象。可以把服务对象放在任何一个由cfg_file域指向的对象配置文件里或是放在cfg_dir域所指向的目录下。

下面会给出一些要监控的样例服务(HTTP、FTP等)的对象定义。

### 监控HTTP

有时不管是你或是其他人需要监控一个Web服务器。check_http插件就是做这件事的。插件可以透过HTTP协议并监控响应时间、错误代码、返回页面里的字符串、服务器证书和更多的东西。

<pre>
// 在commands.cfg文件里包含有一个使用check_http插件的命令定义。象是这样的：
define command{
	name		check_http
	command_name	check_http
	command_line	$USER1$/check_http -I $HOSTADDRESS$ $ARG1$
	}

// 对于监控在remotehost机器上的HTTP服务的简单的服务对象定义会象是这样的：
define service{
	use		generic-service		; Inherit default values from a template
	host_name		remotehost
	service_description	HTTP
	check_command	check_http
	}
</pre>

这个服务对象定义将会监控位于remotehost机器上的HTTP服务。如果WEB服务在10秒内没有响应将产生警报或是返回了一个HTTP的错误码(403、404等)也会产生警报。这是最基本的监控要求，很简单，不是么？

提示 对于更高级的监控，可以在命令行下手工运行check_http插件，带着--help参数，看看这个插件有什么选项。--help语法对于本文档所涉及的插件都适用。

下面看看对HTTP服务更高级的监控。这个服务定义将会检测如下内容：在URI(/download/index.php)里是否包含有"latest-version.tar.gz"字符串。如果没有指定字符串，或是URI非法也或许是在5秒内没有响应的话，它将产生一个错误。
<pre>
define service{
	use		generic-service		; Inherit default values from a template
	host_name		remotehost
	service_description	Product Download Link
	check_command	check_http!-u /download/index.php -t 5 -s "latest-version.tar.gz"
	}
</pre>

### 监控FTP

监控FTP服务可以用check_ftp插件。在commands.cfg文件里包含有一个使用check_ftp插件的命令样例，象是这样：
<pre>
define command{
	command_name	check_ftp
	command_line	$USER1$/check_ftp -H $HOSTADDRESS$ $ARG1$
	}

// 对remotehost主机上的FTP服务的简单监控的例子象是这样：
define service{
	use		generic-service		; Inherit default values from a template
	host_name		remotehost
	service_description	FTP
	check_command	check_ftp
	}
</pre>

这个服务对象定义将会在FTP服务器10秒内不响应时给出一个警报。

下面对FTP做些高级监控。下面这个服务将检测FTP服务器绑定在remotehost主机上的1023端口的FTP服务。如果在5秒内没有响应或是服务里没有回应字符串"Pure-FTPd [TLS]"时将会产生警报。
<pre>
define service{
	use		generic-service		; Inherit default values from a template
	host_name		remotehost
	service_description	Special FTP 
	check_command	check_ftp!-p 1023 -t 5 -e "Pure-FTPd [TLS]"
	}
</pre>

### 监控SSH

监控SSH服务可以用check_ssh插件。在commands.cfg文件里包含在使用check_ssh插件有样例命令，象是这样：
<pre>
define command{
	command_name	check_ssh
	command_line	$USER1$/check_ssh $ARG1$ $HOSTADDRESS$
	}

// 监控remotehost上的SSH服务的简单例子象是这样：
define service{
	use		generic-service		; Inherit default values from a template
	host_name		remotehost
	service_description	SSH
	check_command	check_ssh
	}
</pre>

这个服务对象定义将会在SSH服务10秒内不响应时给出一个警报。
下面对SSH做些高级监控。下面这个服务将检测SSH服务，如果5秒内不响应或是服务器版本串里没有能匹配字符串"OpenSSH_4.2"时产生一个警报。
<pre>
define service{
	use		generic-service		; Inherit default values from a template
	host_name		remotehost
	service_description	SSH Version Check 
	check_command	check_ssh!-t 5 -r "OpenSSH_4.2"
	}
</pre>

### 监控SMTP

用check_smtp插件来监控EMail服务。commands.cfg文件里包含有使用check_smtp插件的命令，象是这样：
<pre>
define command{
	command_name	check_smtp
	command_line	$USER1$/check_smtp -H $HOSTADDRESS$ $ARG1$
	}

// 监控remotehost上的SMTP服务的简单例子象是这样：
define service{
	use		generic-service		; Inherit default values from a template
	host_name		remotehost
	service_description	SMTP
	check_command	check_smtp
	}
</pre>

这个服务对象定义将会在SMTP服务器10秒内不响应时给出一个警报。
下面是更高级的监控。下面这个服务将检测SMTP服务，如果服务在5少内不响应或是返回串里没有包含字符串"mygreatmailserver.com"时将产生一个警报。
<pre>
define service{
	use		generic-service		; Inherit default values from a template
	host_name		remotehost
	service_description	SMTP Response Check 
	check_command	check_smtp!-t 5 -e "mygreatmailserver.com"
	}
</pre>

### 监控POP3

用check_pop插件来监控EMail的POP3服务。commands.cfg文件里包含有使用check_pop插件的命令，象是这样：
<pre>
define command{
	command_name	check_pop
	command_line	$USER1$/check_pop -H $HOSTADDRESS$ $ARG1$
	}

// 监控remotehost上的POP3服务的简单例子象是这样：
define service{
	use		generic-service		; Inherit default values from a template
	host_name		remotehost
	service_description	POP3
	check_command	check_pop
	}
</pre>

这个服务对象定义将会在POP3服务10秒内不响应时给出一个警报。
下面是更高级监控。下面这个服务将检测POP3服务，当服务在5秒内不响应或是返回串里没有包含字符串"mygreatmailserver.com"时将产生一个警报。
<pre>
define service{
	use		generic-service		; Inherit default values from a template
	host_name		remotehost
	service_description	POP3 Response Check 
	check_command	check_pop!-t 5 -e "mygreatmailserver.com"
	}
</pre>

### 监控IMAP

用check_imap插件可以监控EMail的IMAP4服务。commands.cfg文件里包含有使用check_imap插件的命令，象是这样：
<pre>
define command{
	command_name	check_imap
	command_line	$USER1$/check_imap -H $HOSTADDRESS$ $ARG1$
	}

// 监控remotehost上的IMAP服务的简单例子象是这样：
define service{
	use		generic-service		; Inherit default values from a template
	host_name		remotehost
	service_description	IMAP
	check_command	check_imap
	}
</pre>

这个服务对象定义将在IMAP服务10秒内不响应时给出一个警报。
下面是更高级监控。下面这个服务将检测IAMP4服务，当服务在5秒内不响应或是返回串里没有包含有"mygreatmailserver.com"时将产生一个警报。
<pre>
define service{
	use		generic-service		; Inherit default values from a template
	host_name		remotehost
	service_description	IMAP4 Response Check 
	check_command	check_imap!-t 5 -e "mygreatmailserver.com"
	}
</pre>

### 重启动Nagios

一旦在配置文件里加入了一个新的主机或是服务对象，要开始对其进行监控，必须要验证配置文件n并重启动Nagios。
如果验证过程里报出错误信息，要修正配置后再继续。要保证在验证过程里没有发现任何错误之后再启动或重启Nagios！
