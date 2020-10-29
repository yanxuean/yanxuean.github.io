---
layout: post
title: template page
categories: [script]
description: use expect
keywords: expect
---

## 脚本
```
expect -c "spawn /usr/bin/scp -q $2 user@$1:$2
		set timeout 60
		expect {
		    "*password:*" { send myPassword\n; interact }
		    eof { exit }
		    timeout { puts \n--TIMEOUT!--\n;exit}
		}
		exit
           “
```

## 下面是试过的（回车符要是\n ）
```
#!/usr/bin/expect -f

set timeout 60
spawn scp -P 22 "cloud@x.x.x.x:/home/cloud/go/src/k8s.io/kubernetes/_output/bin/kube-apiserver" "/home/cloud/jenkins/"
expect {
    yes/no { send yes\r ; exp_continue }
    "cloud@*'s password:" {
                      #puts "send password for pid=$spawn_id";
                      send "cloud\n";
                      puts "send over";
           }

    timeout         { puts "connect is timeout" }
    "100%"          { puts "over!"}
}

#sleep 60
#wait $spawn_id
#send "exit\n"
expect eof
```

## 又一个
```
#!/usr/bin/expect

set hostName [lindex $argv 0]
set filePath [lindex $argv 1]

spawn /usr/bin/scp -q "$filePath" "user@$hostName:$filePath"
set timeout 60
expect {
 -re ".*password:.*" { send "myPassword\n"; interact }
  eof { exit }
  timeout { puts "\n--TIMEOUT!--\n";exit}
}
exit

```


