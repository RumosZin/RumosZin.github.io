---
title: \[Tips] 로컬 컴퓨터에서 원격 CentOS 접속하기
author: 
date: 2024-03-11 13:30:00 +0900
categories: [Tips, Server]
tags: [Tips]
---

## **로컬에서 .ssh/config 파일 생성**

### **.ssh 경로 확인**

```shell
# .ssh 파일 확인
$ ls -al
$ cd .ssh
$ vi config
```

### **config 파일 작성**

```shell
# dev_server
host dev_server
	user rumos
	hostname 192.168.@@.@@
```

### **.ssh 폴더 권한 설정**

루트 경로에서 `ls -al` 명령어를 이용해 `.ssh` 폴더의 권한을 확인하고, `read`, `execute` 권한을 추가한다.
```shell
# before
$ ls -al
total N
...
drwx------   4 {root_name}  staff    128  3 11 10:58 .ssh
...

# read, execute
$ chmod +rx .ssh
$ ls -al
total N
...
rwxr-xr-x   4 {root_name}  staff    128  3 11 10:58 .ssh
...
```

## **원격 CentOS 접속**

config에 `user`를 `rumos`, `hostname`을 `192.168.@@.@@`로 설정했기 때문에, `ssh rumos@192.168.@@.@@` 명령어와 같다.
```shell
$ ssh dev_server
rumos@192.168.@@.@@'s password: 
Last login: Mon Mar 11 11:04:26 2024 from gimyunjcbookpro
[rumos@RUMOS-CentOS ~]$ 
```