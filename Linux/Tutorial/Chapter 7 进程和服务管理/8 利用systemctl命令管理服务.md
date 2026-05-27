# 利用 systemctl 命令管理服务

在 Linux 系统中提供了很多服务。这些服务依照其功能可以分为系统服务与网络服务。我们需要管理的主要是各种网络服务，如提供远程登录的 sshd 服务、提供网站浏览功能的 httpd 服务等。

在 CentOS 中对服务的管理主要通过 systemd 中的 systemctl 工具来对服务进行管理。systemctl 是 systemd 提供的一个重要管理工具，主要负责控制 systemd 系统和服务管理器，它集 service 和 chkconfig 等众多命令的功能于一体，功能非常强大。

## 管理服务运行状态

利用 systemctl 命令管理服务运行状态的语法格式如下：

```shell
systemctl start|stop|status|restart|reload  服务名
```

`start|stop|status|restart|reload`表示可以对服务执行的管理动作，它们之间是`或`的关系，每次只能选择执行其中一种动作。start 表示启动服务，stop 表示停止服务，status 表示查看服务运行状态，restart 表示重启服务，reload 表示重新加载服务。

systemd 将系统中的每个服务都看作一个服务单元（Service unit），在服务的名称后面加上`.service`作为后缀。在利用 systemctl 命令对服务进行管理时，服务名称后面加不加`.service`后缀均可。例如，执行`systemctl status sshd.service`或`systemctl status sshd`命令都可以查看 sshd 服务的运行状态。

```sh
[root@spider ~]# systemctl status sshd.service
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
     Active: active (running) since Tue 2025-04-15 08:18:13 CST; 14min ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 794 (sshd)
      Tasks: 1 (limit: 10882)
     Memory: 6.1M
        CPU: 64ms
     CGroup: /system.slice/sshd.service
             └─794 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Apr 15 08:18:13 localhost.localdomain systemd[1]: Starting OpenSSH server daemon...
Apr 15 08:18:13 localhost.localdomain sshd[794]: Server listening on 0.0.0.0 port 22.
```

下面介绍在执行`systemctl status`命令时所显示的服务状态信息。

在所显示的信息中，第二行的`Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)`，表示服务已经被加载，该服务的 Service unit 配置文件为`/usr/lib/systemd/system/sshd.service`，`enabled`表示该服务已经被设为开机自动启动，` vendor preset: enabled`表示该服务在系统中默认被预设为开机自动启动。

第三行的`Active`部分显示`active（running）`，表示服务正处于运行状态。

第六行的`Main PID: 1061 (sshd)`表示该服务运行之后所产生的主进程 PID 为 1061，进程名为 sshd。

最后会显示若干行在日志文件中所记录的与该服务有关的一些信息。

下面在系统中安装 vsftpd 服务（如果已安装，那么此处可忽略），然后查看 vsftpd 服务的运行状态，在`Active`部分显示`inactive (dead)`，表示服务已经安装，但是没有运行。再继续查看 httpd 服务的运行状态，由于我们并没有安装 httpd 服务，因此系统提示没有发现该服务。

```sh
[root@localhost ~]# systemctl status vsftpd
○ vsftpd.service - Vsftpd ftp daemon
     Loaded: loaded (/usr/lib/systemd/system/vsftpd.service; disabled; vendor preset: disabled)
     Active: inactive (dead)
[root@localhost ~]# systemctl status httpd
Unit httpd.service could not be found.
```

下面执行`systemctl start vsftpd`命令启动 vsftpd 服务，并再次查看服务状态，发现已经处于运行状态。

```sh
[root@localhost ~]# systemctl start vsftpd
[root@localhost ~]# systemctl status vsftpd
● vsftpd.service - Vsftpd ftp daemon
     Loaded: loaded (/usr/lib/systemd/system/vsftpd.service; disabled; vendor preset: disabled)
     Active: active (running) since Mon 2022-03-28 13:19:36 CST; 6s ago ] # running
......
```

在之后对服务的配置管理中，如果修改了某项服务的设置，那么必须将服务重启，才可使得修改后的设置生效。例如，重启 vsftpd 服务，可以执行`systemctl restart vsftpd`命令。

```sh
[root@localhost ~]# systemctl restart vsftpd
```

重启服务实际上就是将服务先停止，然后再启动的过程。如果不需要再运行某个服务，那么也可以将服务直接停止。例如，停止 vsftpd 服务，可以执行`systemctl stop vsftpd`命令。

```sh
[root@localhost ~]# systemctl stop vsftpd
```

需要注意的是，利用 restart 方式重启服务，会使服务产生短暂的中断。如果需要在不中断服务的前提下使得修改后的服务设置生效，那么可以使用 reload 方式重新加载服务。例如，重新加载 vsftpd 服务，可以执行`systemctl reload vsftpd`命令。

```sh
[root@localhost ~]# systemctl reload vsftpd
```

使用 reload 方式重新加载服务，虽然可以避免服务中断，但是对某些服务配置所做的修改，必须要重启（restart）才能生效，比如修改了服务的端口号等。另外，reload 方式也不是适用于所有的服务，有些服务（如 network 服务）就无法通过 reload 方式重新加载，这一点在实际应用中需要注意。

最后介绍一下如何查看系统中所有正在运行的服务，可以执行命令`systemctl list-units --type service`。选项`list-units`表示列出所有的 unit，选项`--type service`则表示列出服务类的 unit。

```sh
[root@localhost ~]# systemctl list-units --type service
 UNIT                               LOAD   ACTIVE SUB     DESCRIPTION                                     >
  accounts-daemon.service            loaded active running Accounts Service
  alsa-state.service                 loaded active running Manage Sound Card State (restore and store)
  atd.service                        loaded active running Deferred execution scheduler
  auditd.service                     loaded active running Security Auditing Service
……
```

如果在该命令之后再加上`--all`选项，则表示列出系统中所有的服务，无论该服务是否正在运行。

```sh
[root@localhost ~]# systemctl list-units --type service --all
 UNIT                                   LOAD      ACTIVE   SUB     DESCRIPTION                            >
  accounts-daemon.service                loaded    active   running Accounts Service
  alsa-restore.service                   loaded    inactive dead    Save/Restore Sound Card State
  alsa-state.service                     loaded    active   running Manage Sound Card State (restore and st>
  atd.service                            loaded    active   running Deferred execution scheduler
  auditd.service                         loaded    active   running Security Auditing Service
● auto-cpufreq.service                   not-found inactive dead    auto-cpufreq.service
● autofs.service                         not-found inactive dead    autofs.service
……
```

## 管理服务启动状态

除系统服务之外，我们后来所安装运行的各种服务都是临时启动的，在系统关机或重启之后，这些服务不会自动运行。我们还可以通过 systemctl 命令来管理服务的启动状态。

利用 systemctl 命令管理服务启动状态的语法格式如下：

```sh
systemctl enable|disable|is-enabled  服务名
```

`enable|disable|is-enabled`表示可以执行的动作，同样每次只能选择执行其中的一种。enable 表示将服务设为开机自动启动，disable 表示禁止服务开机自动启动，is-enabled 表示查看服务的启动状态。

下面以 sshd 服务为例介绍对服务启动状态的设置方法。

例如，查看 sshd 服务是否为开机自动启动。命令执行后显示`enabled`表示服务是开机自动启动，显示`disabled`则表示服务不是开机自动启动。

```sh
[root@localhost ~]# systemctl is-enabled sshd
enabled
```

例如，禁止 sshd 服务开机自动启动。

```sh
[root@localhost ~]# systemctl disable sshd
Removed /etc/systemd/system/multi-user.target.wants/sshd.service.
[root@localhost ~]# systemctl is-enabled sshd
disabled
```

例如，将 sshd 服务重新设置为开机自动启动。

```sh
[root@localhost ~]# systemctl enable sshd
Created symlink /etc/systemd/system/multi-user.target.wants/sshd.service → /usr/lib/systemd/system/sshd.service.
[root@localhost ~]# systemctl is-enabled sshd
enabled
```

如果要查看系统中所有服务的开机启动状态，那么可以执行`systemctl list-unit-files --type service`命令。

```sh
[root@localhost ~]# systemctl list-unit-files --type service
UNIT FILE                                  STATE           VENDOR PRESET
accounts-daemon.service                    enabled         enabled
alsa-restore.service                       static          -
alsa-state.service                         static          -
arp-ethers.service                         disabled        disabled
atd.service                                enabled         enabled
auditd.service                             enabled         enabled
……
```
