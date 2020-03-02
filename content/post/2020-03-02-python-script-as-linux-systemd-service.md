---
layout: post
title: "Python Slack Bot(슬랙봇)을 리눅스 서버에서 돌리기"
description: "파이썬으로 제작한 슬랙봇을 리눅스 서버의 systemd로 등록하는 방법"
date: 2020-03-02T22:07:08+09:00
image: "img/slack_logo.png"
author: Gunyoung Yoon
categories: [ "Tech" ]
tags: [ "Linux", "Slack", "Slack-Bot", "Python", "Systemd" ]
showtoc: false
draft: false
---

# 슬랙, 그리고 봇
지금 다니는 회사에서는 사내 메신저로 슬랙(Slack)이라는 서비스를 사용한다. 2016년 정도부터 현재까지 4년정도를 슬랙을 써와서 현재는 실시간으로 업무관련 커뮤니케이션 할 때는 슬랙 위주로 사용하게 하게되었다. 슬랙은 많은 장점들을 가지고 있지만, 그중 단연 최고는 바로 연동(Integration)이라고 생각한다. 많은 플랫폼과 제품들(심지어 경쟁사의 타 제품과도)간 연동을 지원하며, 자동화 봇(Bot)이나 연동을 통한 알림 등을 하기가 쉽게되어있다. 현재 회사의 슬랙 워크스페이스에도 많은 연동이 되어있고, 이번에 처음으로 봇을 작성하여 추가하게 되었다.

슬랙에 봇을 연동하기는 쉽고, 인터넷 여기저기에도 예제가 정말 많이 작성되어있다. 특히 파이썬 같은 경우는 슬랙에서 직접 [python-slackclient](https://github.com/slackapi/python-slackclient)라는 SDK까지 제공해주기 때문에 금방 작성 할 수 있다. 하지만 이 봇을 실제 서버에서 24시간 돌게끔 설정하는 방법은 많이 알려져있지 않은 것 같아서 이렇게 글을 작성해본다.

# 리눅스 서버에 슬랙봇 연동하기
우선 요구사항은 이렇다.
1. 구동 환경은 리눅스(Linux) 운영체제이고, 배포판은 Ubuntu
2. 파이썬으로 작성된 스크립트를 백그라운드에서 계속 실행 해 줄 것
3. 혹시 스크립트가 돌다가 중단되거나 하는 등의 불상사가 발생한 경우, 자동으로 재시작을 해줄 것

한마디로 요약하자만 "자동으로 파이썬 스크립트를 백그라운드로 띄워주고, 죽었을 때 다시 살려주면 된다."이다.

제일 처음 생각을 했던 방법은 Linux 계열의 OS에서 기본적으로 제공해주는 크론탭(crontab)을 이용하는 것이다. crontab과 쉘 스크립트(Shell Script)를 사용해서 각종 리눅스의 명령어들, 그리고 정규식을 섞어 파이썬 스크립트가 떠있는지 확인하고, 없다면 새로운 스크립트를 demonize( 쉘 명령어 끝에 &를 붙여서 할 수 있는 그것)해서 띄우는 방식이였다. 사실 예전에 이 방법을 [bikeseoulNotifier](https://github.com/Dry8r3aD/bikeseoulNotifier) 프로젝트에 사용했었는데, 어느순간  봇 프로세스가 두개가 떠버리는 문제가 있었다. 아마 타이밍 문제로 예상은 되었지만, 저 프로젝트를 더이상 사용하지 않아 추가적으로 디버깅을 하지는 않았었다.

그러다 이번에는 Linux 계열에서 많이 사용되는 systemd를 사용해보기로 했다. systemd service 형태로 작성을 한다면 굳이 봇이 살아있는지 내가 확인을 하지 않아도 되고, 또 죽었을 때도 내가 설정한 것에 따라서 systemd가 동적으로 다시 봇을 실행시키기 때문에 좋을 것이라고 생각했다. 무엇보다 Linux 운영체제에서 전반적으로 쓰이는 견고한 기능이였기때문에 활용하면 좋을 것 같았다.

# Linux systemd를 사용해서 스크립트를 service로 띄우기
systemd service를 만드는 방법은 간단하다. [규칙](https://www.freedesktop.org/software/systemd/man/systemd.service.html)에 맞는 service 파일을 작성하고, systemd에 등록만 해주면 된다.

```
$ sudo vi /lib/systemd/system/foobar.service
```
```
[Unit]
Description=Service's description

[Service]
ExecStart=/usr/bin/python3 SCRIPT.py
Restart=on-failure
StandardInput=tty-force
StandardOutput=syslog
StandardError=syslog

[Install]
WantedBy=multi-user.target
```
출처: [systemd.service Example 3](https://www.freedesktop.org/software/systemd/man/systemd.service.html)

## 설명
* Description: Service의 설명
* ExecStart: 실행할 명령어
* Restart: on-failure로 설정하는 경우, 정상종료를 제외한 모든 경우에 대해서 재시작을 보장
* StandardInput: 입력(STDIN)에 대한 file descriptor 설정
* StandardOutput: 출력(STDOUT)에 대한 file descriptor 설정
* StandardError: 에러출력(STDERR)에 대한 file descriptor 설정
더 자세한 정보는 [Systemd Execution environment configuration](https://www.freedesktop.org/software/systemd/man/systemd.exec.html)과 [Systemd Service unit configuration](https://www.freedesktop.org/software/systemd/man/systemd.service.html)에서 확인이 가능하다.

## 추가적인 설정
systmed는 워낙 견고하고 믿을 수 있는 기능 중 하나라, 굳이 필요는 없지만 만약 systemd의 재시작(restart)이 실패할 경우를 대비에서 crontab에 아래 내용을 추가하였다.

```
* * * * *   root    /bin/systemctl is-active foobar -q || /usr/sbin/service foobar start
```


# 결론
Systemd의 service로 스크립트를 실행하고, 위와같은 설정을 해주었을 때, 혹시라도 Slack bot이 죽었을 때 정상적으로 알아서 재실행되는 것을 확인 할 수 있었다.

```
service foobar status
```
위 명령어를 사용하면 systemd service의 상태를 볼 수 있으며, 현재 돌고있는 프로세스가 언제 실행되었는지 알 수 있다.

```
➜  ~ service foobar status
● foobar.service - Service foobar's description
   Loaded: loaded (/lib/systemd/system/foobar.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2020-03-03 23:12:29 KST; 29min ago
 Main PID: 111972 (python)
    Tasks: 1
   Memory: 23.8M
      CPU: 386ms
   CGroup: /system.slice/foobar.service
```
Active의 실행시간을 보았을 때, 내가 직접 봇을 설치하고 재시작한 시점 이후인 경우, sysemd(혹은 백업으로 설정한 cron)에 의해서 재시작되었다고 볼 수 있다.
저 foobar인 경우 수동으로 재시작을 어제 했고, 현재의 프로세스는 29분전에 실행 되었기에 자동으로 재시작 되었다고 볼 수 있다.
