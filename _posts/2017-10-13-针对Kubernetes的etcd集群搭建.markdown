---
layout: post
title: 针对Kubernetes的etcd集群搭建-标题title
date: 2017-10-13 12:53:20 +0800
description: 针对于Kubernetes的etcd集群搭建方法-description
img: i-rest.jpg # Add image post (optional)
tags: [etcd, Kubernetes]
---
## 针对kubernetes的etcd集群搭建
## 环境
- CentOS Linux release 7.3.1611 (Core) 三台 
- kernel 3.10.0-514.16.1.el7.x86_64 
- etcd 3.1.3 （yum install）
## 介绍
etcd组件作为一个高可用、强一致性的服务发现存储仓库，渐渐为开发人员所关注。在云计算时代，如何让服务快速透明地接入到计算集群中，如何让共享配置信息快速被集群中的所有机器发现，更为重要的是，如何构建这样一套高可用、安全、易于部署以及响应快速的服务集群，已经成为了迫切需要解决的问题。etcd为解决这类问题带来了福音。   
  **A highly-available key value store for shared configuration**  
  etcd在kubernetes集群中的作用  
  Kubernetes中的大部分概念如Node、Pod、Replication Controller、Service等都可以看做一种“资源对象”，几乎所有的资源对象都可以在etcd库中持久化存储。    
  Kubernetes的高度自动化功能也是通过跟踪对比etcd库里保存的“资源期望状态”与当前环境的“实际资源状态”的差异来实现的。   

## configure etcd config file
 **如果配置TLS加密的话，所有的集群节点用于通信的IP地址都要写在证书的配置文件里** 
```bash
[root@node1 k8sCA]# cat /etc/etcd/etcd.conf | grep -v "^#"
#[member]
ETCD_NAME=etcd1.xuhui.com
#最好和证书中的DNS一致
ETCD_DATA_DIR="/var/lib/etcd/"
ETCD_LISTEN_PEER_URLS="https://192.168.100.125:2380"
#本地绑定的IP地址，用于和集群中的其他member同步
ETCD_LISTEN_CLIENT_URLS="https://0.0.0.0:2379"
#提供服务的地址
#[cluster]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.100.125:2380"
#与集群中其他成员通信的地址
ETCD_INITIAL_CLUSTER="etcd1.xuhui.com=https://192.168.100.125:2380,etcd2.xuhui.com=https://192.168.100.126:2380,etcd3.xuhui.com=https://192.168.100.127:2380"
#所有集群中member的地址都要写在这里
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.100.125:2379"
#[security]
ETCD_CERT_FILE="/etc/kubernetes/ssl/etcd.crt"
ETCD_KEY_FILE="/etc/kubernetes/ssl/etcd.key"
ETCD_TRUSTED_CA_FILE="/etc/kubernetes/ssl/ca.crt"
ETCD_PEER_CERT_FILE="/etc/kubernetes/ssl/etcd.crt"
#通过证书配置文件生成的证书
ETCD_PEER_KEY_FILE="/etc/kubernetes/ssl/etcd.key"
ETCD_PEER_TRUSTED_CA_FILE="/etc/kubernetes/ssl/ca.crt"
```
 剩下的节点以此类推 
 **需修改name 和 IP地址** 

## 生成TLS证书（v1.6的话证书请查看k8s-v1.6-高可用集群搭建笔记中的安全设置一篇）
```bash
openssl genrss -out etcd.key 2048
#生成私钥
openssl req -new -key etcd.key -subj "/CN=etcd" -config openssl.cnf -out etcd.csr
#生成证书请求
openssl x509 -req -in etcd.csr -CA ca.crt -CAkey ca.key -CAcreateserial -days 5000 -extensions v3_req -extfile openssl.cnf -out etcd.crt
#生成证书
#证书配置文件
[root@snmp k8sCA]# cat openssl.cnf 
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
DNS.5 = rocket
DNS.6 = etcd1.xuhui.com
DNS.7 = etcd2.xuhui.com
DNS.8 = etcd3.xuhui.com
IP.1 = 10.254.0.1
IP.2 = 192.168.61.137
IP.3 = 192.168.100.137
#etcd集群节点地址
IP.6 = 192.168.100.125
IP.7 = 192.168.100.126
IP.8 = 192.168.100.127
#IP.1对应DNS.1 .8 对应 .8
```
## 调整Kubernetes apiserver 选项
```bash
[root@snmp k8sCA]# cat /etc/kubernetes/apiserver | grep -i etcd
# Comma separated list of nodes in the etcd cluster
KUBE_ETCD_SERVERS="--etcd-servers=https://192.168.100.125:2379,https://192.168.100.126:2379,https://192.168.100.127:2379"
#加入下边三行
 --etcd-cafile=/etc/kubernetes/ssl/ca.pem --etcd-certfile=/etc/etcd/ssl/etcd.pem --etcd-keyfile=/etc/etcd/ssl/etcd-key.pem "
```
