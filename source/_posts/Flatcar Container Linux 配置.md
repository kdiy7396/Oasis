---
title: Flatcar Container Linux
date: 2026-02-23 16:00:00
tags:
  - 技术向
---
# Getting Started
- 以paopaodns镜像为例，esxi平台
## 前提
1. 下载butane win版👉:https://github.com/coreos/butane/releases/
2. 下载最新版flatcar👉：[[[https://stable.release.flatcar-linux.net/amd64-usr/current/flatcar_production_vmware_ova.ova]]
3. 参考flatcar.butane.yaml修改paopaodns.yaml
``` Yaml
variant: flatcar

version: 1.1.0

kernel_arguments:

  should_not_exist:

    - flatcar.autologin

passwd:

  users:

    - name: root

      password_hash: $6$SALT$io0TPmhM8ythCm7Idt0AfYvTuFCLyA1CMVmeT3EUqarf2NQcTuLKEgP9.4Q8fgClzP7OCnyOY1wo1xDw0jtyH1

      ssh_authorized_keys:

        - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAtWYS4SAiOLhsrgrMk8W4nquBqALsfhjNi1r2Odmpfe kidy789@linux.com

storage:

  files:

    - path: /etc/hostname

      contents:

        inline: paopaodns-vm

    - path: /etc/systemd/network/00-ens192.network

      mode: 0644

      contents:

        inline: |

          [Match]

          Name=ens192

          [Network]

          DHCP=no

          Address=192.168.21.20/24

          Gateway=192.168.21.1

          DNS=223.5.5.5

          DNS=1.1.1.1

    - path: /etc/iptables.rules

      mode: 0644

      contents:

        inline: |

          *filter

          :INPUT ACCEPT [0:0]

          -A INPUT -p udp --dport 53 -j ACCEPT

          -A INPUT -p tcp --dport 53 -j ACCEPT

          COMMIT

systemd:

  units:

    - name: systemd-resolved.service

      enabled: false

      mask: true

    - name: docker.service

      enabled: true

    - name: iptables-restore.service

      enabled: true

      contents: |

        [Unit]

        Description=Restore iptables rules

        Before=network-pre.target

        [Service]

        Type=oneshot

        ExecStart=/sbin/iptables-restore /etc/iptables.rules

        RemainAfterExit=yes

        [Install]

        WantedBy=multi-user.target

    - name: paopaodns.service

      enabled: true

      contents: |

        [Unit]

        Description=PaoPaoDNS Service

        After=docker.service network-online.target

        Requires=docker.service network-online.target

        [Service]

        TimeoutStartSec=0

        ExecStartPre=-/usr/bin/docker rm --force paopaodns

        ExecStart=/usr/bin/docker run --name paopaodns --net host --restart always -v /data:/data -e CNAUTO=yes -e IPV6=yes -e TZ=Asia/Shanghai -e UPDATE=daily -e DNS_SERVERNAME=PaoPaoDNS -e CUSTOM_FORWARD=119.29.29.29,1.1.1.1:53 public.ecr.aws/sliamb/paopaodns

        ExecStop=/usr/bin/docker stop paopaodns

        Restart=always

        RestartSec=5s

        [Install]

        WantedBy=multi-user.target
```
4. 已安装wsl 2 for ubuntu 22.04(linux系统需要下载对应的butane版本)
5. 创建ed25519密钥对，并将公钥*.pub内容粘贴至passwd·users·ssh_authorized_keys·- ssh-ed25519后面
6. 已安装MobaXterm_Personal（当然也可以是其他ssh工具）
# Installing Flatcar
## **win端**
1. 将paopaodns.yaml生成json
   ```bash
   .\butane-x86_64-pc-windows-gnu.exe -p paopaodns.yaml
   ```
2. 新建json文件保存生成的代码
3. WSL 2生成base64码
  ```bash
  base64 -w 0 paopaodns.json
  ```
## **esxi端**
4. 用flatcar_production_vmware_ova.ova创建虚拟机
   - 输入主机名称：其他配置·options·Hostname
   - 输入base64码：其他配置·options·Ignition/coreos-cloudinit data
   - 输入数据编码格式（base64）：其他配置·options·Ignition/coreos-cloudinit data encoding

## **MobaXterm端**
5. 创建SSH节点
   - 输入节点ip:Basic SSH setting·Remote host
   - 输入登入账号root：Basic SSH setting·Specify username
   - 加载前述生成的私钥：Basic SSH setting·Advanced SSH setting·Use private key
5. 访问节点

Flatcar Container Linux 在运行docker方面还是不错的，但是由于是提前配置好了，缺点也非常明显，修改成本高，如果不需要调试的docker项目还是可以使用的。