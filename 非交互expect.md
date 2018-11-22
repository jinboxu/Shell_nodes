## 非交互expect

**参考文档:   http://www.xuetimes.com/archives/781**

expect命令简单说就是通过其内置的各种命令实现在交互式软件中自动交互的工具。

 

send：用于向进程发送字符串  (将参数作为交互端的输入)

expect：从进程接收字符串      (得到交互的的输出)

spawn：启动新的进程 

interact：允许用户交互 (停在当前终端)



```
#!/usr/bin/expect
##v1.0 by jinbo

set ip [lindex $argv 0]
set user root
set password root
set timeout 5

spawn ssh $user@$ip

expect {
        "Are you sure you want to continue connecting (yes/no)?" { send "yes\r";exp_continue }
        "password" { send "$password\r"}

}

expect "$*"
send "pwd\r"
send_user "over"
interact
```



```
#!/usr/bin/expect
set ip [lindex $argv 0]
set user root
set password root
set timeout 5

spawn ssh $user@$ip
expect {
	"yes/no" { send "yes\r"; exp_continue }
	"password:" { send "$password\r" };
}
#interact

expect "#"
#send "useradd yangyang\r"
send "pwd\r"
send "exit\r"
expect eof
```



#### 关于expect eof

