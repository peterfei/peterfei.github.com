title: Linux/UNIX的scp命令
date: 2015-07-15 16:45:50
tags:
- linux
- scp
categories: Linux
---
************************
*1、复制远程服务器的文件到本地：*
`scp -P888 root@120.18.50.33:/data/ha97.zip /home/
`
*2、复制远程服务器的目录到本地：*
`scp -vrp -P888 root@120.18.50.33:/data/ha97/ /home/
`
*3、复制本地的文件到远程服务器：*
`scp -P888 /home/ha97.zip root@120.18.50.33:/data/
`
*4、复制本地的目录到远程服务器：*
`scp -vrp -P888 /home/ root@120.18.50.33:/data/`
