title: "expect 脚本编程"
id: 132
date: 2012-05-16 14:36:30
tags: 
- Expect
- Linux
- Shell
- Automation
categories: 
- Engineering
- Productivity
---

在多个机器上批量执行任务脚本的时候，经常需要输ssh密码，但是出于安全性（明文密码命令容易被history出来）和交互性命令（ssh等命令需要强制用户输入而不能通过参数输入）的限制，需要找一些新的办法。拍脑袋想到的有2个。

1、将自己的ssh 公钥上传到这些机器上（但第一次上传的时候，依然需要输密码，又绕回去了）

2、使用ssh -F configfile （看了下man ssh 和 man ssh_config，感觉太难配）

网上搜索了一阵，发现 expect 脚本编程。大概来说，基本上登录交互的东西都可以搞定。
> expect - programmed dialogue with interactive programs, Version 5 （一个可以编程的交互对话程序）
下面上代码，完成登录，执行cmd command，返回结果，退出的功能。

1 #!/usr/bin/expect -f
2
3 # -- set all file list --
4 #set db_box myssh-target
5 set db_box [lrange $argv 0 0]
6 set pw mypassword
7
8 spawn ssh $db_box
9 set timeout -1
10
11 expect {
12         "Are you sure you want to continue connecting (yes/no)? " {
13                 send "yes\r";
14                 exp_continue
15         }
16         "hector@$db_box's password: " {
17                 send "$pw\r"
18                 set timeout -1
19                 expect "*bash-3.2$ " {
20                         send "grep topic= `yinst set | grep coke_queue_daemon.daemon_dir | cut -f2 -d ':'`/subscription/ -r | cut -f2 -d ':' | cut -f2 -d '='\r"
21                         send "exit\r"
22                 }
23         }
24 }
25 interact

expect的精髓就在于这个词本身，你expect什么输出，对应什么输入，都可以控制。具体情况man写得很详细，但一般工作照着上面这个改就可以了。<!--more-->

Note：

[lrange $argv l r] 取命令行参数的第l个到第r个（从0开始）。

spawn ssh $db_box  通过spawn 执行ssh命令（ssh 命令其实是可以后接cmd命令的，但是直接将20行的cmd命令接到ssh后面会因为引号无法解析，所以采取了一种walk around 的方法在下面发送命令）

timeout使用-1比较合适，表示一直等待响应。

这样就可以一次性在多个机器上执行操作了。
