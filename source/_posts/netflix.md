---
title: 使用 CloudFlare Warp 解锁坡县、港区的 Netflix
date: 2021-11-19 15:34:58
tags: [Netflix, Python, 脚本工具]
---
# 前言
## 什么是 Netflix 锁区
Netflix 为了版权方的利益，对视频内容进行了锁区，绝大部分的 IP 地址都被 Netflix 加入了黑名单。因此，一台 VPS 的 IP 是否能够解锁 Netflix 也成了一个重要指标。Netflix 的中文字幕仅在大多数华人居住地提供（港澳台新加坡马来西亚），但这些地方解锁本地流媒体的 VPS 普遍较贵。

## 什么是 CloudFlare Warp
Warp 是 CloudFlare 提供的一项**免费** VPN 服务，它提供的 Linux 客户端可以过 WireGuard 连接。

## 为什么需要反复刷新获取新的 Warp IP
Warp 每隔一段时间会切换 IP，并且 Warp 每次连接获取到的 IP 不一定是本地的IP，并且很大一部分的 IP 被玩坏了（被 Netflix 拉黑）
# 实现
## 思路
自动检测当前的 IP 是否解锁 Netflix，若不解锁，重新请求一个新的 IP。
## 操作
首先要有一台亚太地区的 VPS，能够获取到 Warp 提供的香港（新加坡）的 IP，这里以一台 1 刀的阿里云香港轻量为例。  
[下载](https://github.com/sjlleo/netflix-verify) 检测 Netflix 的工具，放（链接）到 `/root/nf`
使用 [Warp 一键脚本](https://p3terx.com/archives/cloudflare-warp-configuration-script.html)，确保正确开启 Warp
创建 `/root/nf.py`，填入下面的脚本，这里蹩脚的脚本只是抛砖引玉，您可以自行修改  
### 脚本附注
这里以解锁港区为例，如果是新加坡，将脚本里的 HK 都改成 SG。
脚本创建一个 `/root/nf.lock` 为判断脚本是否仍在运行，要是中途结束运行了，自己手动删一下。
````Python
import subprocess
import os
import sys

is_lockexist = os.path.exists("/root/nf.lock")
if is_lockexist == False:
    subprocess.getoutput("/usr/bin/touch /root/nf.lock")
    while True:
        ip_str = subprocess.getoutput("curl -4 ip.p3terx.com")
        print(ip_str)
        if ip_str.find("HK") != -1:
            while True:
                nf_str = subprocess.getoutput('/root/nf')
                print(nf_str)
                if nf_str.find("(HK)") == -1:
                    #nf_str = subprocess.getoutput('/root/nf')
                    print("failure")
                    subprocess.run(['/bin/systemctl','restart','wg-quick@wgcf'])
                    break
                else:
                    print("OK")
                    subprocess.getoutput("/bin/rm /root/nf.lock")
                    sys.exit()
                    break
        else:
            subprocess.run(['/bin/systemctl','restart','wg-quick@wgcf'])
else:
    print("nf is already running")
````
使用 `crontab -e`，创建规则  
````
*/10 * * * * /usr/bin/python3 /root/nf.py
````
至此，您的脚本应该能够顺利运行了。