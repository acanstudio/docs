Nagios配置入门
===============

配置文件
--------

| 文件名或目录名           | 用途                                                                       |
| -------------------------|:--------------------------------------------------------------------------:|
| cgi.cfg                  | 控制CGI访问的配置文件
| nagios.cfg               | Nagios 主配置文件
| resource.cfg             | 变量定义文件，又称为资源文件，在些文件中定义变量，以便由其他配置文件引用，如$USER1$
| objects                  | objects 是一个目录，在此目录下有很多配置文件模板，用于定义Nagios 对象
| objects/commands.cfg     | 命令定义配置文件，其中定义的命令可以被其他配置文件引用
| objects/contacts.cfg     | 定义联系人和联系人组的配置文件
| objects/localhost.cfg    | 定义监控本地主机的配置文件
| objects/printer.cfg      | 定义监控打印机的一个配置文件模板，默认没有启用此文件
| objects/switch.cfg       | 定义监控路由器的一个配置文件模板，默认没有启用此文件
| objects/templates.cfg    | 定义主机和服务的一个模板配置文件，可以在其他配置文件中引用
| objects/timeperiods.cfg  | 定义Nagios 监控时间段的配置文件
| objects/windows.cfg      | 监控Windows 主机的一个配置文件模板，默认没有启用此文件

在nagios的配置过程中涉及到的几个定义有：主机、主机组，服务、服务组，联系人、联系人组，监控时间，监控命令等，从这些定义可以看出，nagios各个配置文件之间是互为关联，彼此引用的。
成功配置出一台nagios监控系统，必须要弄清楚每个配置文件之间依赖与被依赖的关系，最重要的有四点：

    * 定义监控哪些主机、主机组、服务和服务组；
    * 定义这个监控要用什么命令实现；
    * 定义监控的时间段；
    * 定义主机或服务出现问题时要通知的联系人和联系人组。

为了能更清楚的说明问题，同时也为了维护方便，建议将nagios各个定义对象创建独立的配置文件：

    * 创建hosts.cfg文件来定义主机和主机组
    * 创建services.cfg文件来定义服务
    * 用默认的contacts.cfg文件来定义联系人和联系人组
    * 用默认的commands.cfg文件来定义命令
    * 用默认的timeperiods.cfg来定义监控时间段
    * 用默认的templates.cfg文件作为资源引用文件

timeperiods.cfg文件
---------------------

<pre>
define contact{
        name                            generic-contact    ; 联系人名称
        service_notification_period     24x7               ; 当服务出现异常时，发送通知的时间段，这个时间段"24x7"在timeperiods.cfg文件中定义
        host_notification_period        24x7               ; 当主机出现异常时，发送通知的时间段，这个时间段"24x7"在timeperiods.cfg文件中定义
        service_notification_options    w,u,c,r            ; 这个定义的是“通知可以被发出的情况”。w即warn，表示警告状态，u即unknown，表示不明状态;
                                                           ; c即criticle，表示紧急状态，r即recover，表示恢复状态;
                                                           ; 也就是在服务出现警告状态、未知状态、紧急状态和重新恢复状态时都发送通知给使用者。
        host_notification_options       d,u,r                   ; 定义主机在什么状态下需要发送通知给使用者，d即down，表示宕机状态;
                                                                ; u即unreachable，表示不可到达状态，r即recovery，表示重新恢复状态。
        service_notification_commands   notify-service-by-email ; 服务故障时，发送通知的方式，可以是邮件和短信，这里发送的方式是邮件;
                                                                ; 其中“notify-service-by-email”在commands.cfg文件中定义。
        host_notification_commands      notify-host-by-email    ; 主机故障时，发送通知的方式，可以是邮件和短信，这里发送的方式是邮件;
                                                                ; 其中“notify-host-by-email”在commands.cfg文件中定义。 
        register                        0                    ; DONT REGISTER THIS DEFINITION - ITS NOT A REAL CONTACT, JUST A TEMPLATE!
        }

define host{
        name                            generic-host    ; 主机名称，这里的主机名，并不是直接对应到真正机器的主机名;
                                                        ; 乃是对应到在主机配置文件里所设定的主机名。
        notifications_enabled           1               ; Host notifications are enabled
        event_handler_enabled           1               ; Host event handler is enabled
        flap_detection_enabled          1               ; Flap detection is enabled
        failure_prediction_enabled      1               ; Failure prediction is enabled
        process_perf_data               1               ; 其值可以为0或1，其作用为是否启用Nagios的数据输出功能;
                                                        ; 如果将此项赋值为1，那么Nagios就会将收集的数据写入某个文件中，以备提取。
        retain_status_information       1               ; Retain status information across program restarts
        retain_nonstatus_information    1               ; Retain non-status information across program restarts
        notification_period             24x7            ; 指定“发送通知”的时间段，也就是可以在什么时候发送通知给使用者。
        register                        0               ; DONT REGISTER THIS DEFINITION - ITS NOT A REAL HOST, JUST A TEMPLATE!
        }
define service{
        name                            generic-service         ; 定义一个服务名称
        active_checks_enabled           1                       ; Active service checks are enabled
        passive_checks_enabled          1                       ; Passive service checks are enabled/accepted
        parallelize_check               1                       ; Active service checks should be parallelized;
                                                                ; (disabling this can lead to major performance problems)
        obsess_over_service             1                       ; We should obsess over this service (if necessary)
        check_freshness                 0                       ; Default is to NOT check service 'freshness'
        notifications_enabled           1                       ; Service notifications are enabled
        event_handler_enabled           1                       ; Service event handler is enabled
        flap_detection_enabled          1                       ; Flap detection is enabled
        failure_prediction_enabled      1                       ; Failure prediction is enabled
        process_perf_data               1                       ; Process performance data
        retain_status_information       1                       ; Retain status information across program restarts
        retain_nonstatus_information    1                       ; Retain non-status information across program restarts
        is_volatile                     0                       ; The service is not volatile
        check_period                    24x7             ; 这里的check_period告诉nagios检查服务的时间段。
        max_check_attempts              3                ; nagios对服务的最大检查次数。
        normal_check_interval           5                ; 此选项是用来设置服务检查时间间隔，也就是说，nagios这一次检查和下一次检查之间所隔的时间;
                                                         ; 这里是5分钟。
        retry_check_interval            2                ; 重试检查时间间隔，单位是分钟。
        contact_groups                  ts           ; 指定联系人组
        notification_options            w,u,c,r          ; 这个定义的是“通知可以被发出的情况”。w即warn，表示警告状态;
                                                         ; u即unknown，表示不明状态;
                                                         ; c即criticle，表示紧急状态，r即recover，表示恢复状态;
                                                         ; 也就是在服务出现警告状态、未知状态、紧急状态和重新恢复后都发送通知给使用者。
        notification_interval           10               ; Re-notify about service problems every hour
        notification_period             24x7             ; 指定“发送通知”的时间段，也就是可以在什么时候发送通知给使用者。
        register                        0                ; DONT REGISTER THIS DEFINITION - ITS NOT A REAL SERVICE, JUST A TEMPLATE!
        }
define service{
        name                            local-service           ; The name of this service template
        use                             generic-service         ; Inherit default values from the generic-service definition
        max_check_attempts              4             ; Re-check the service up to 4 times in order to determine its final (hard) state
        normal_check_interval           5             ; Check the service every 5 minutes under normal conditions
        retry_check_interval            1             ; Re-check the service every minute until a hard state can be determined
        register                        0             ; DONT REGISTER THIS DEFINITION - ITS NOT A REAL SERVICE, JUST A TEMPLATE!
        }
</pre>

resource.cfg文件
-----------------

resource.cfg是nagios的变量定义文件，文件内容只有一行：

<pre>
$USER1$=/usr/local/nagios/libexec
</pre>

其中，变量$USER1$指定了安装nagios插件的路径，如果把插件安装在了其它路径，只需在这里进行修改即可。需要注意的是，变量必须先定义，然后才能在其它配置文件中进行引用。

commands.cfg文件
-----------------

此文件默认是存在的，无需修改即可使用，当然如果有新的命令需要加入时，在此文件进行添加即可。

<pre>
#notify-host-by-email命令的定义 
define command{
        command_name    notify-host-by-email             #命令名称，即定义了一个主机异常时发送邮件的命令。
        command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\nHost: $HOSTNAME$\nState: $HOSTSTATE$\nAddress: $HOSTADDRESS$\nInfo: $HOSTOUTPUT$\n\nDate/Time: $LONGDATETIME$\n" | /bin/mail -s "** $NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$ **" $CONTACTEMAIL$                                     #命令具体的执行方式。
        }
#notify-service-by-email命令的定义 
define command{
        command_name    notify-service-by-email          #命令名称，即定义了一个服务异常时发送邮件的命令
        command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\n\nService: $SERVICEDESC$\nHost: $HOSTALIAS$\nAddress: $HOSTADDRESS$\nState: $SERVICESTATE$\n\nDate/Time: $LONGDATETIME$\n\nAdditional Info:\n\n$SERVICEOUTPUT$\n" | /bin/mail -s "** $NOTIFICATIONTYPE$ Service Alert: $HOSTALIAS$/$SERVICEDESC$ is $SERVICESTATE$ **" $CONTACTEMAIL$
        }
define command{
        command_name    check_local_disk
        command_line    $USER1$/check_disk -w $ARG1$ -c $ARG2$ -p $ARG3$            #$ARG1$是指在调用这个命令的时候，命令后面的第一个参数。
        }
define command{
        command_name    check_local_load
        command_line    $USER1$/check_load -w $ARG1$ -c $ARG2$
        }
define command{
        command_name    check_local_procs
        command_line    $USER1$/check_procs -w $ARG1$ -c $ARG2$ -s $ARG3$
        }
define command{
        command_name    check_local_users
        command_line    $USER1$/check_users -w $ARG1$ -c $ARG2$
        }
</pre>

hosts.cfg文件
--------------

此文件默认不存在，需要手动创建，hosts.cfg主要用来指定被监控的主机地址以及相关属性信息，根据实验目标配置如下：

<pre>
define host{   
        use                     linux-server          #引用主机linux-server的属性信息，linux-server主机在templates.cfg文件中进行了定义。
        host_name               Nagios-Linux          #主机名
        alias                   Nagios-Linux          #主机别名
        address                 192.168.1.111         #被监控的主机地址，这个地址可以是ip，也可以是域名。
        }   
#定义一个主机组   
define hostgroup{      
        hostgroup_name          bsmart-servers        #主机组名称，可以随意指定。
        alias                   bsmart servers        #主机组别名
        members                 Nagios-Linux          #主机组成员，其中“Nagios-Linux”就是上面定义的主机。     
      }
</pre>

注意：在/usr/local/nagios/etc/objects 下默认有localhost.cfg和windows.cfg 这两个配置文件，localhost.cfg 文件是定义监控主机本身的，windows.cfg文件是定义windows 主机的，其中包括了对host 和相关services 的定义。所以在本次实验中，将直接在localhost.cfg 中定义监控主机（Nagios-Server），在windows.cfg中定义windows 主机（Nagios-Windows）。根据自己的需要修改其中的相关配置，详细如下：

<pre>
#localhost.cfg
define host{
        use                     linux-server            ; Name of host template to use
                                                        ; This host definition will inherit all variables that are defined
                                                        ; in (or inherited by) the linux-server host template definition.
        host_name               Nagios-Server
        alias                   Nagios-Server
        address                 127.0.0.1
        }
define hostgroup{
        hostgroup_name  linux-servers ; The name of the hostgroup
        alias           Linux Servers ; Long name of the group
        members         Nagios-Server ; Comma separated list of hosts that belong to this group
        }
define service{
        use                             local-service         ; Name of service template to use
        host_name                       Nagios-Server
        service_description             PING
        check_command                   check_ping!100.0,20%!500.0,60%
        }
define service{
        use                             local-service         ; Name of service template to use
        host_name                       Nagios-Server
        service_description             Root Partition
        check_command                   check_local_disk!20%!10%!/
        }
define service{
        use                             local-service         ; Name of service template to use
        host_name                       Nagios-Server
        service_description             Current Users
        check_command                   check_local_users!20!50
        }
define service{
        use                             local-service         ; Name of service template to use
        host_name                       Nagios-Server
        service_description             Total Processes
        check_command                   check_local_procs!250!400!RSZDT
        }
define service{
        use                             local-service         ; Name of service template to use
        host_name                       Nagios-Server
        service_description             Current Load
        check_command                   check_local_load!5.0,4.0,3.0!10.0,6.0,4.0
        }
define service{
        use                             local-service         ; Name of service template to use
        host_name                       Nagios-Server
        service_description             Swap Usage
        check_command                   check_local_swap!20!10
        }
define service{
        use                             local-service         ; Name of service template to use
        host_name                       Nagios-Server
        service_description             SSH
        check_command                   check_ssh
        notifications_enabled           0
        }
define service{
        use                             local-service         ; Name of service template to use
        host_name                       Nagios-Server
        service_description             HTTP
        check_command                   check_http
        notifications_enabled           0
        }
</pre>        

services.cfg文件
----------------

此文件默认也不存在，需要手动创建，services.cfg文件主要用于定义监控的服务和主机资源，例如监控http服务、ftp服务、主机磁盘空间、主机系统负载等等。Nagios-Server和Nagios-Windows相关服务已在相应的配置文件中定义，所以这里只需要定义Nagios-Linux相关服务即可，这里只定义一个检测是否存活的服务来验证配置文件的正确性，其他服务的定义将在后面讲到。

<pre>
define service{  
        use                     local-service          #引用local-service服务的属性值，local-service在templates.cfg文件中进行了定义。
        host_name               Nagios-Linux           #指定要监控哪个主机上的服务，“Nagios-Server”在hosts.cfg文件中进行了定义。
        service_description     check-host-alive       #对监控服务内容的描述，以供维护人员参考。
        check_command           check-host-alive       #指定检查的命令。
        }  
</pre>

contacts.cfg文件
----------------

contacts.cfg是一个定义联系人和联系人组的配置文件，当监控的主机或者服务出现故障，nagios会通过指定的通知方式（邮件或者短信）将信息发给这里指定的联系人或者使用者。

<pre>
define contact{
        contact_name                    David             #联系人的名称,这个地方不要有空格
        use                             generic-contact   #引用generic-contact的属性信息，其中“generic-contact”在templates.cfg文件中进行定义
        alias                           Nagios Admin
        email                           david.tang@bsmart.cn
        }
define contactgroup{
        contactgroup_name       ts                              #联系人组的名称,同样不能空格
        alias                   Technical Support               #联系人组描述
        members                 David                           #联系人组成员，其中“david”就是上面定义的联系人，如果有多个联系人则以逗号相隔
        }
</pre>

timeperiods.cfg文件
-------------------

此文件只要用于定义监控的时间段，下面是一个配置好的实例：

<pre>
#下面是定义一个名为24x7的时间段，即监控所有时间段  
define timeperiod{  
        timeperiod_name 24x7       #时间段的名称,这个地方不要有空格
        alias           24 Hours A Day, 7 Days A Week  
        sunday          00:00-24:00  
        monday          00:00-24:00  
        tuesday         00:00-24:00  
        wednesday       00:00-24:00  
        thursday        00:00-24:00  
        friday          00:00-24:00  
        saturday        00:00-24:00  
        }  
#下面是定义一个名为workhours的时间段，即工作时间段。  
define timeperiod{  
        timeperiod_name workhours   
        alias           Normal Work Hours  
        monday          09:00-17:00  
        tuesday         09:00-17:00  
        wednesday       09:00-17:00  
        thursday        09:00-17:00  
        friday          09:00-17:00  
        }  
</pre>

cgi.cfg文件
------------

此文件用来控制相关cgi脚本，如果想在nagios的web监控界面执行cgi脚本，例如重启nagios进程、关闭nagios通知、停止nagios主机检测等，这时就需要配置cgi.cfg文件了。
由于nagios的web监控界面验证用户为david，所以只需在cgi.cfg文件中添加此用户的执行权限就可以了，需要修改的配置信息如下：

<pre>
default_user_name=david
authorized_for_system_information=nagiosadmin,david  
authorized_for_configuration_information=nagiosadmin,david  
authorized_for_system_commands=david
authorized_for_all_services=nagiosadmin,david  
authorized_for_all_hosts=nagiosadmin,david
authorized_for_all_service_commands=nagiosadmin,david  
authorized_for_all_host_commands=nagiosadmin,david 
</pre>

nagios.cfg文件
---------------

nagios.cfg默认的路径为/usr/local/nagios/etc/nagios.cfg，是nagios的核心配置文件，所有的对象配置文件都必须在这个文件中进行定义才能发挥其作用，这里只需将对象配置文件在Nagios.cfg文件中进行引用即可。

<pre>
log_file=/usr/local/nagios/var/nagios.log                  # 定义nagios日志文件的路径
cfg_file=/usr/local/nagios/etc/objects/commands.cfg        # “cfg_file”变量用来引用对象配置文件，如果有更多的对象配置文件，在这里依次添加即可。
cfg_file=/usr/local/nagios/etc/objects/contacts.cfg
cfg_file=/usr/local/nagios/etc/objects/hosts.cfg
cfg_file=/usr/local/nagios/etc/objects/services.cfg
cfg_file=/usr/local/nagios/etc/objects/timeperiods.cfg
cfg_file=/usr/local/nagios/etc/objects/templates.cfg
cfg_file=/usr/local/nagios/etc/objects/localhost.cfg       # 本机配置文件
cfg_file=/usr/local/nagios/etc/objects/windows.cfg         # windows 主机配置文件
object_cache_file=/usr/local/nagios/var/objects.cache      # 该变量用于指定一个“所有对象配置文件”的副本文件，或者叫对象缓冲文件
precached_object_file=/usr/local/nagios/var/objects.precache
resource_file=/usr/local/nagios/etc/resource.cfg           # 该变量用于指定nagios资源文件的路径，可以在nagios.cfg中定义多个资源文件。
status_file=/usr/local/nagios/var/status.dat               # 该变量用于定义一个状态文件，此文件用于保存nagios的当前状态、注释和宕机信息等。
status_update_interval=10                                  # 该变量用于定义状态文件（即status.dat）的更新时间间隔，单位是秒，最小更新间隔是1秒。
nagios_user=nagios                                         # 该变量指定了Nagios进程使用哪个用户运行。
nagios_group=nagios                                        # 该变量用于指定Nagios使用哪个用户组运行。
check_external_commands=1                                  # 该变量用于设置是否允许nagios在web监控界面运行cgi命令;
                                                           # 也就是是否允许nagios在web界面下执行重启nagios、停止主机/服务检查等操作;
                                                           # “1”为运行，“0”为不允许。
command_check_interval=10s                                 # 该变量用于设置nagios对外部命令检测的时间间隔，如果指定了一个数字加一个"s"(如10s);
                                                           # 那么外部检测命令的间隔是这个数值以秒为单位的时间间隔;
                                                           # 如果没有用"s"，那么外部检测命令的间隔是以这个数值的“时间单位”的时间间隔。
interval_length=60                                         # 该变量指定了nagios的时间单位，默认值是60秒，也就是1分钟;
                                                           # 即在nagios配置中所有的时间单位都是分钟。

</pre>

验证Nagios配置文件的正确性
---------------------------

Nagios 在验证配置文件方面做的非常到位，只需通过一个命令即可完成：

<pre>
# /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
[root@cache-2 etc]# /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

Nagios Core 4.0.6
Copyright (c) 2009-present Nagios Core Development Team and Community Contributors
Copyright (c) 1999-2009 Ethan Galstad
Last Modified: 04-29-2014
License: GPL

Website: http://www.nagios.org
Reading configuration data...
   Read main config file okay...
   Read object config files okay...

Running pre-flight check on configuration data...

Checking objects...
	Checked 29 services.
	Checked 4 hosts.
	Checked 2 host groups.
	Checked 0 service groups.
	Checked 4 contacts.
	Checked 2 contact groups.
	Checked 26 commands.
	Checked 5 time periods.
	Checked 0 host escalations.
	Checked 0 service escalations.
Checking for circular paths...
	Checked 4 hosts
	Checked 0 service dependencies
	Checked 0 host dependencies
	Checked 5 timeperiods
Checking global event handlers...
Checking obsessive compulsive processor commands...
Checking misc settings...

Total Warnings: 0
Total Errors:   0

Things look okay - No serious problems were detected during the pre-flight check
[root@cache-2 etc]# 
</pre>

Nagios提供的这个验证功能非常有用，在错误信息中通常会打印出错误的配置文件以及文件中的哪一行，这使得nagios的配置变得非常容易，报警信息通常是可以忽略的，因为一般那些只是建议性的。 
看到上面这些信息就说明没问题了，然后启动Nagios 服务。 
