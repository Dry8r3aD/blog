---
layout: post
title: "nmap으로 리눅스 네트워크 트러블슈팅하기"
description: "Nmap is a complex tool. Check out this short list of Nmap commands that will carry you through basic network troubleshooting."
date: 2020-02-12T21:57:08+09:00
author: Gunyoung Yoon
categories: [ "Tech" ]
tags: [ "Learning", "Linux", "Network" ]
draft: true
---

> 이 글은 [Enable Sysadmin](https://www.redhat.com/sysadmin/)에 기고된 [Shashank Nandishwar Hegde](https://www.redhat.com/sysadmin/users/shegde)의 <[Basic network troubleshooting in Linux with nmap](https://www.redhat.com/sysadmin/nmap-troubleshooting?fbclid=IwAR3u08g6tcb6e6-drJgdTnb50el0joxkmSsGLPKcycwhEzqkh7pPszjSs6c)>을 번역한 글입니다.

> This article is a translated version of the <[Basic network troubleshooting in Linux with nmap](https://www.redhat.com/sysadmin/nmap-troubleshooting?fbclid=IwAR3u08g6tcb6e6-drJgdTnb50el0joxkmSsGLPKcycwhEzqkh7pPszjSs6c)>, which was posted on the [Enable Sysadmin](https://www.redhat.com/sysadmin/), by [Shashank Nandishwar Hegde](https://www.redhat.com/sysadmin/users/shegde).


--------------------------------------------------


Troubleshooting a network is fun, challenging, and interesting, especially for sysadmins. Linux comes with various tools under its belt to help you along the way, including Nmap, ping, traceroute, netstat, ns lookup, and dig. For the purpose of this article, I will stick to Nmap and will explain the steps for how to troubleshoot a network.
시스템이나 네트워크, 혹은 서버 관리자들에게 네트워크 트러블 슈팅은 재미있고, 도전적이고, 또 흥미로운 것일 것입니다. 리눅스 운영체제에는 기본적으로 Nmap, ping, traceroute, netstat, ns lookup, dig 등의 다양한 도구들이 시스템 관리자가 트러블슈팅을 할 때 사용 할 수 있게끔 탑재되어있습니다. 이 글에서는 Nmap 이라는 도구를 사용해서 네트워크의 문제를 진단하고 해결하는, 트러블슈팅 방법을 소개할 것입니다.

Nmap is a useful tool for troubleshooting networks by detecting open ports, the OS version, routes between servers, and much more. Read the official documentation yourself to get a more in-depth look at this tool. Now, let’s dive into using some of the Nmap commands. First, check the version of Nmap (I’m using Red Hat Enterprise Linux 7.5), like this:
Nmap은 열린 포트나 운영체제 정보, 그리고 서버간의 라우팅 정보 등을 알아낼 수 있어,  네트워크 트러블슈팅을 할 때 유용한 도구입니다. 이 도구에 대해서 더 많은 정보를 얻기 위해서는 [공식 문서](https://official-doc-nmap)를 참고해주세요. 이제 Nmap을 사용하는 방법에 대해서 알아보겠습니다. 첫째로, Nmap의 버전정보를 확인해 주세요. (원글 작성자는 Red Had Enterprise, Linux 7.5 버전을 사용하였지만 저는 Ubuntu 18.04, Linux Kernel 4.5를 기준으로 작성하였습니다. 확인하는 명령어는 아래와 같습니다.


If Nmap is not installed, type the following on an RHEL-based system:
```
sudo yum install -y nmap
```

Now, run Nmap on our gateway IP:
```
sudo nmap -sn <Your-IP>
```
The results look like this:

[ Related Story: How to validate your security measures ]

Determine this host's OS with the -O switch:

$ sudo nmap -O <Your-IP>
The results look like this:

[ You might also like: Six practical use cases for Nmap ]

Then, run the following to check the common 2000 ports, which handle the common TCP and UDP services. Here, -Pn is used to skip the ping scan after assuming that the host is up:

$ sudo nmap -sS -sU -PN <Your-IP>
The results look like this:

Note: The -Pn combo is also useful for checking if the host firewall is blocking ICMP requests or not.

Also, as an extension to the above command, if you need to scan all ports instead of only the 2000 ports, you can use the following to scan ports from 1-66535:

$ sudo nmap -sS -sU -PN -p 1-65535 <Your-IP>
The results look like this:

Image


You can also scan only for TCP ports (default 1000) by using the following:

$ sudo nmap -sT <Your-IP>
The results look like this:


Now, after all of these checks, you can also perform the "all" aggressive scans with the -A option, which tells Nmap to perform OS and version checking using -T4 as a timing template that tells Nmap how fast to perform this scan (see the Nmap man page for more information on timing templates):

$ sudo nmap -A -T4 <Your-IP>
The results look like this, and are shown here in two parts:


There you go. These are the most common and useful Nmap commands. Together, they provide sufficient network, OS, and open port information, which is helpful in troubleshooting. Feel free to comment with your preferred Nmap commands as well.

