---
title: OpenStack 从入门到继续
date: 2018-10-11 09:12:20
tags: 
- openstack 
- cloud
categories: tech
---
近期给自己挖的技术坑有点多，我忍不住再挖一个，最近又了解到了云服务的相关技术（算是技术储备吧）

## What is OpenStack?
[OpenStack](https://www.openstack.org)是被广泛使用的云操作系统，它管啥，如图：
![](https://www.openstack.org/software/images/map/openstack-map-v20180601.svg)

<!-- more -->

## Components

所有组件详见：
[网站列出的所有组件](https://www.openstack.org/software/project-navigator/openstack-components#operations-services)

找几个介绍多的，简单说一下

### [HORIZON](https://www.openstack.org/software/releases/rocky/components/horizon)
前端Web管理界面，包括Dashboard控件展示，便于用户对相关服务进行操作

### [NOVA](https://www.openstack.org/software/releases/rocky/components/nova)
计算服务,应该是管理计算资源的组件，提供相应的管理服务

### [NEUTRON](https://www.openstack.org/software/releases/rocky/components/neutron) 
提供云计算的网络虚拟化技术，为OpenStack其他服务提供网络连接服务，实现SDN

### [SWIFT](https://www.openstack.org/software/releases/rocky/components/swift)
Swift是高可用、分布式、最终一致性的对象存储，具体文件对象

### [CINDER](https://www.openstack.org/software/releases/rocky/components/cinder)
块存储服务，更像是硬盘，数据卷

### [KEYSTONE](https://www.openstack.org/software/releases/rocky/components/keystone)
认证服务，支持LDAP Oauth OpenID等

### [GLANCE](https://www.openstack.org/software/releases/rocky/components/glance)
镜像服务，不通的操作系统镜像

### [HEAT](https://www.openstack.org/software/releases/rocky/components/heat)
编排架构资源

不通场景中需要不通的服务来构建，可以参考[sample configurations](https://www.openstack.org/software/sample-configs)

## What is the relationship between Docker and OpenStack
![](/images/post/20181011/docker openstack.png)
- Docker 主要针对 Paas 平台，是以应用为中心。
- OpenStack 主要针对 Iaas 平台，以资源为中心，可以为上层的 PaaS 平台提供存储、网络、计算等资源。

这个话题先开个头，后面有机会再继续深入吧。
