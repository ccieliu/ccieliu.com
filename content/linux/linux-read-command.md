---
title: "Linux Read Command"
date: 2020-09-15T01:03:21+08:00
draft: true
summary: Linux shell 用 "read" 输入 读取 标准输出 或 文件。
categories:
  - "Linux"
---

### read 输入
用read 输入 读取 标准输出 或 文件。

---

#### -p 选项
弹出提示符。其他的参数会一次赋值。例如下：

```
[root@host1-docker tmpdir]#read -p "Please input some things: " a b c
Please input some things: aaa bbb ccc
[root@host1-docker tmpdir]#echo $aaa

[root@host1-docker tmpdir]#echo $a
aaa
[root@host1-docker tmpdir]#echo $b
bbb
[root@host1-docker tmpdir]#echo $c
ccc
```
#### -s 选项  用于密码
输入无回显，通常用于用户输入密码。
但是-s可能会导致下一个read在同行显示。
workaround:
单独echo一次即可，例如
```
read -s -p "Please input your name: " myname ; echo
```

#### 匹配默认值，互交界面
```
#!/bin/bash -
read -p "Please input your name: " myname
read -p "Please input your email: " myemail

while true
do
	read -p "Display all information:[Y]/n" input
	case $input in
		[yY] | "")
			echo "Your name is $myname"
			echo "Your email address is $myemail"
			exit 0
		;;

		[nN])
			echo "Okay. Goodbye~"
			exit 0
		;;

		*)
			continue
		;;
	esac
done
```


### read 用于 文件读取 输入
在while和until中使用read来读取文件。
举例：

测试text文件如下：
```
[root@host1-docker tmpdir]#more demo.txt 
line1a line1b line1c
line2a line2b line2c
line3a line3b line3c
line4a line4b line4c
line5a line5b line5c
```

测试在循环中read文件：
```
[root@host1-docker tmpdir]#more readFile.sh 
#!/bin/bash -
[ $# -ne 1 ] && exit 1

for line in $(cat $1)
do
	echo ${line}
done 
```

这里linux会使用IFS变量座位默认的分隔符, 并不是我们期待的根据每行读取 并 赋值。
这里和read aaa bbb ccc ddd的效果是一样的。在`-p 选项` 第一部分中演示过。
所以这是说得通的行为。
IFS的默认值是 <space><tab><newline>  所以空格优先于tab和\n

```
[root@host1-docker tmpdir]#set | grep IFS
IFS=$' \t\n'
```
```
[root@host1-docker tmpdir]#./readFile.sh  demo.txt
line1a
line1b
line1c
line2a
line2b
line2c
line3a
line3b
...
```

#### 安全的备份变量 并 处理 按行读取的问题
```
[root@host1-docker tmpdir]#more readFile.sh 
#!/bin/bash -
IFS_Bak=$IFS
IFS=$'\n'   << 这里重新定义了IFS的意义。只使用 新行 作为分隔，因此每次循环均是一整行。这里注意 $ 和 单引号.
[ $# -ne 1 ] && exit 1

for line in $(cat $1)
do
	echo ${line}
done 

IFS=$IFS_Bak
```
最终效果：
```
[root@host1-docker tmpdir]#./readFile.sh demo.txt 
line1a line1b line1c
line2a line2b line2c
line3a line3b line3c
line4a line4b line4c
line5a line5b line5c
```
