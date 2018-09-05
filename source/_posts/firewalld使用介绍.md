---
title: firewalld使用介绍
author: Evobot
categories: Centos7
tags:
  - Linux
  - Centos
abbrlink: 79e258ff
date: 2018-05-10 22:10:06
image:
---

# iptables规则备份与恢复

- 命令`service iptables save`会将我们shell中配置的iptables规则保存到**/etc/sysconfig/iptables**配置文件中；
- 如果想将规则保存到指定的文件中，可以使用命令`iptables-save > /path/to/file`；
- 从备份文件恢复规则则使用命令`iptables-restore < /path/to/file`即可；
- 如果想要开机启动自动加载iptables规则，建议规则还是保存在iptables的配置文件中。

# firewalld操作配置

## 启用firewalld

- 由于之前使用的iptables服务，所以需要重新关闭iptables服务并打开firewalld服务：

  ```bash
  [root@www ~]# systemctl disable iptables
  [root@www ~]# systemctl is-enabled iptables.service 
  disabled
  [root@www ~]# systemctl stop iptables
  [root@www ~]# systemctl enable firewalld.service 
  Created symlink from /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service to /usr/lib/systemd/system/firewalld.service.
  Created symlink from /etc/systemd/system/basic.target.wants/firewalld.service to /usr/lib/systemd/system/firewalld.service.
  [root@www ~]# systemctl start firewalld.service 
  ```

- 启动后执行`iptables -nvL`可以看到系统多了非常多规则，这些规则就是firewalld自带的规则。

## firewalld的zone介绍

- firewalld默认有9个zone，一个zone相当于一个规则集，默认的zone为**public**；

- 查看所有的zone使用命令`firewall-cmd --get-zones`：

  ```bash
  [root@www ~]# firewall-cmd --get-zones
  work drop internal external trusted home dmz public block
  ```

- 查看默认的zone使用命令`firewall-cmd --get-default-zone`：

  ```bash
  [root@www ~]# firewall-cmd --get-default-zone 
  public
  ```

- firewalld的9个zone之间的区别见下表：

<style>
table th {
    text-align: center;
}
table th:first-of-type {
    width: 120px;
}
</style>

|     zone     | 作用                                       |
| :----------: | --------------------------------------- |
|   drop(丢弃)   | 任何接收的网络数据包都被丢弃，没有任何回复。仅能有发送出去的网络连接。      |
|  block(限制)   | 任何接收的网络连接都被IPv4的**icmp-host-prohibited**信息和IPv6的**icmp6-adm-prohibited**信息所拒绝。 |
|  public(公共)  | 在公共区域内使用，不能相信网络内其他计算机不会对你的计算机造成危害，只能接收经过选取的连接，即部分数据包放行。 |
| external(外部) | 特别是为路由器启用了伪装功能的外部网，不能信任来自网络其他计算机不会对我方计算机造成危害，只接收经过选择的连接。 |
|  dmz(非军事区)   | 用于非军事区内的电脑，次区域内可公开访问，可以有限的进入我们的内部网络，仅接收经选择的连接。 |
|   work(工作)   | 用于工作区，即可以基本相信网络内的其他电脑不会危害我的电脑。仅接收经过选择的连接。 |
|   home(家庭)   | 用于家庭网络，基本信任网络内其他计算机不会危害我的计算机，仅接收经过选择的连接。 |
| internal(内部) | 用于内部网络，基本上信任网络内其他计算机不会威胁我的计算机。仅接收经过选择的连接。 |
| trusted(信任)  | 可接受所以的网络连接。                              |

## zone的操作

- 命令`firewall-cmd --set-default-zone=work`可以更改默认的zone:

  ```bash
  [root@www ~]# firewall-cmd --set-default-zone=work
  success
  [root@www ~]# firewall-cmd --get-default-zone 
  work
  ```

- firewalld可以针对网卡操作，命令`firewall-cmd --get-zone-of-interface=[网卡名]`可以查看指定网卡的默认zone：

  ```bash
  [root@www ~]# firewall-cmd --get-zone-of-interface=enp0s3
  work
  [root@www ~]# firewall-cmd --get-zone-of-interface=enp0s8
  no zone
  [root@www ~]# firewall-cmd --get-zone-of-interface=lo
  no zone
  ```

- 如果查看新网卡的zone时结果是`no zone`，那么需要将**/etc/sysconfig/network-scripts/**下的原网卡配置文件复制一份并改成新的网卡名，然后重启网络服务和firewalld服务，再查看时就能看到新的网卡的默认zone：

  ```bash
  [root@www network-scripts]# systemctl restart network.service
  [root@www network-scripts]# systemctl restart firewalld.service 
  [root@www network-scripts]# firewall-cmd --get-zone-of-interface=enp0s8
  work
  ```

- 给指定的网卡设置zone，命令为`firewall-cmd --zone=public --add-interface=enp0s8`:

  ```bash
  [root@www ~]# firewall-cmd --zone=public --add-interface=enp0s8
  The interface is under control of NetworkManager, setting zone to 'public'.
  success
  [root@www ~]# firewall-cmd --get-zone-of-interface=enp0s8
  public
  ```

- 更改指定网卡的zone，命令为`firewall-cmd --zone=dmz --change-interface=enp0s8`

  ```bash
  [root@www ~]# firewall-cmd --zone=dmz --change-interface=enp0s8
  The interface is under control of NetworkManager, setting zone to 'dmz'.
  success
  [root@www ~]# firewall-cmd --get-zone-of-interface=enp0s8
  dmz
  ```

- 删除指定网卡的zone，命令为`firewall-cmd --zone=dmz --remove-interface=enp0s8`:

  ```bash
  [root@www ~]# firewall-cmd --zone=dmz --remove-interface=enp0s8
  The interface is under control of NetworkManager, setting zone to default.
  success
  [root@www ~]# firewall-cmd --get-zone-of-interface=enp0s8
  work
  ```

  > 删除网卡的zone之后，网卡的zone会恢复到默认的zone设置。

- 查看系统所以网卡所在的zone，命令为`firewall-cmd --get-active-zones`：

  ```bash
  [root@www ~]# firewall-cmd --get-active-zones 
  work
    interfaces: enp0s3 enp0s8
  ```

## firewalld的service

- firewalld的service就是zone下面的子单元，可以理解为一个端口；

- 查看所有的service使用命令`firewall-cmd --get-services`：

  ```bash
  [root@www ~]# firewall-cmd --get-services 
  RH-Satellite-6 amanda-client amanda-k5-client bacula bacula-client ceph ceph-mon dhcp dhcpv6 dhcpv6-client dns docker-registry dropbox-lansync freeipa-ldap freeipa-ldaps freeipa-replication ftp high-availability http https imap imaps ipp ipp-client ipsec iscsi-target kadmin kerberos kpasswd ldap ldaps libvirt libvirt-tls mdns mosh mountd ms-wbt mysql nfs ntp openvpn pmcd pmproxy pmwebapi pmwebapis pop3 pop3s postgresql privoxy proxy-dhcp ptp pulseaudio puppetmaster radius rpc-bind rsyncd samba samba-client sane smtp smtps snmp snmptrap squid ssh synergy syslog syslog-tls telnet tftp tftp-client tinc tor-socks transmission-client vdsm vnc-server wbem-https xmpp-bosh xmpp-client xmpp-local xmpp-server
  ```

- 查看当前的zone的service使用命令`firewall-cmd --get-default-zone`：

  ```bash
  [root@www ~]# firewall-cmd --get-default-zone 
  work
  [root@www ~]# firewall-cmd --list-services 
  ssh dhcpv6-client
  ```

- 而查看指定的zone的service，则使用命令`firewall-cmd --zone=public --list-services`：

  ```bash
  [root@www ~]# firewall-cmd --zone=public --list-services 
  dhcpv6-client ssh
  [root@www ~]# firewall-cmd --zone=dmz --list-services 
  ssh
  ```

- 给指定的zone添加service，使用命令`firewall-cmd --zone=public --add-service=http`，这里添加的http实际上表示http服务对应的端口80：

  ```bash
  [root@www ~]# firewall-cmd --zone=public --add-service=http
  success
  [root@www ~]# firewall-cmd --zone=public --list-services 
  dhcpv6-client ssh http
  [root@www ~]# firewall-cmd --zone=public --add-service=ftp
  success
  [root@www ~]# firewall-cmd --zone=public --list-services 
  dhcpv6-client ssh http ftp
  ```

- 在shell中为zone添加了service之后，只是保存在内存中，想要将配置保存到配置文件中，则在添加service时使用下面的命令：

  ```bash
  [root@www ~]# firewall-cmd --zone=public --add-service=https --permanent
  success
  ```

- firewalld的配置文件在目录`/etc/firewalld/zones/`下：

  ```bash
  [root@www ~]# ls /etc/firewalld/zones/
  public.xml  public.xml.old
  [root@www ~]# cat /etc/firewalld/zones/public.xml
  <?xml version="1.0" encoding="utf-8"?>
  <zone>
    <short>Public</short>
    <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
    <service name="dhcpv6-client"/>
    <service name="ssh"/>
    <service name="https"/>
  </zone>
  ```

  可以看到https服务已经添加到public zone里了。

- firewalld的zone在保存新的配置时，会将旧的配置文件后缀名加上**.old**保存，同样的service配置文件则在`/etc/firewalld/services/`目录下；而zone和service的配置文件都有一个模板文件，位于`/usr/lib/firewalld/[zone|services]`目录下：

  ```bash
  [root@www ~]# ls /usr/lib/firewalld/zones/
  block.xml  dmz.xml  drop.xml  external.xml  home.xml  internal.xml  public.xml  trusted.xml  work.xml
  [root@www ~]# ls /usr/lib/firewalld/services/
  amanda-client.xml        https.xml         mountd.xml      puppetmaster.xml    telnet.xml
  amanda-k5-client.xml     http.xml          ms-wbt.xml      radius.xml          tftp-client.xml
  bacula-client.xml        imaps.xml         mysql.xml       RH-Satellite-6.xml  tftp.xml
  bacula.xml               imap.xml          nfs.xml         rpc-bind.xml        tinc.xml
  ceph-mon.xml             ipp-client.xml    ntp.xml         rsyncd.xml          tor-socks.xml
  ceph.xml                 ipp.xml           openvpn.xml     samba-client.xml    transmission-client.xml
  dhcpv6-client.xml        ipsec.xml         pmcd.xml        samba.xml           vdsm.xml
  dhcpv6.xml               iscsi-target.xml  pmproxy.xml     sane.xml            vnc-server.xml
  dhcp.xml                 kadmin.xml        pmwebapis.xml   smtps.xml           wbem-https.xml
  dns.xml                  kerberos.xml      pmwebapi.xml    smtp.xml            xmpp-bosh.xml
  docker-registry.xml      kpasswd.xml       pop3s.xml       snmptrap.xml        xmpp-client.xml
  dropbox-lansync.xml      ldaps.xml         pop3.xml        snmp.xml            xmpp-local.xml
  freeipa-ldaps.xml        ldap.xml          postgresql.xml  squid.xml           xmpp-server.xml
  freeipa-ldap.xml         libvirt-tls.xml   privoxy.xml     ssh.xml
  freeipa-replication.xml  libvirt.xml       proxy-dhcp.xml  synergy.xml
  ftp.xml                  mdns.xml          ptp.xml         syslog-tls.xml
  high-availability.xml    mosh.xml          pulseaudio.xml  syslog.xml
  ```

## firewalld配置案例

- 需求：将FTP服务端口改为1121，并且在work zone下放行ftp：

  - 拷贝ftp的配置文件到`/etc/firewalld/services/`目录下：

  ```bash
  [root@www ~]# cp /usr/lib/firewalld/services/ftp.xml /etc/firewalld/services/
  ```

  - 修改复制过来的FTP配置文件，将其中的端口配置改为1121：

  ![ftp-service](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/ftp_service.png)

  - 复制work zone的配置文件到`/etc/firewalld/zones/`目录下：

  ```bash
  [root@www ~]# cp /usr/lib/firewalld/zones/work.xml /etc/firewalld/zones/
  ```

  - 修改复制来的work zone配置文件，将ftp服务添加进去：

  ![work-zone](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/workzone.png)

  - 然后使用`firewall-cmd --reload`命令重新加载firewalld服务；并查看work zone的service列表ftp服务是否添加进去：

  ```bash
  [root@www ~]# firewall-cmd --reload 
  success
  [root@www ~]# firewall-cmd --zone=work --list-services 
  ssh dhcpv6-client ftp
  ```

---

