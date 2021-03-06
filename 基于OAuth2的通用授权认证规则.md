---
title: 基于OAuth2的通用授权认证规则
date: 2016-12-26 13:55:57
tags: [OAuth2, API认证]
---

##  普遍适用的接口授权及认证规则

### 简介

   
接口授权和认证有许多处理方案, 其中最简单的一种使用appkey和appsecret的方式进行验证

另外还有一种是通过对接口中的参数排序并按照一定加密算法求得哈希值, 作为授权验证的令牌

本文介绍基于OAuth2的通用授权认证规则

简言之, 客户端首先通过一定方式获取访问令牌, 然后在每次调用接口时, 携带该令牌, 服务端验证该令牌

### 为什么要使用

* 部分接口目前没有使用接口认证授权, 接口一旦泄露, 将存在很大安全隐患
* 不同项目的接口认证机制不统一, 加重了开发人员的工作负担
* 一旦接口泄露, 服务器亦不能及时对相关接口进行封锁

### 流程

* 在**用户中心**获取访问令牌, 访问令牌会同步到公用Redis缓存中, 以便其他项目访问
* 客户端调用相关查询接口
* 服务器通过查询Redis中的令牌, 验证该令牌是否合法, 如果不合法, 提醒客户端重新登录
* 令牌验证通过, 服务器执行后续操作, 返回结果

### 特点

* 访问令牌存储于公用Redis缓存中, 所有客户端共用一套授权认证机制
* 用户在修改/重置密码后, 令牌自动失效, 所有客户端会强制下线, 保证安全性
* 与**用户中心**深度结合, 统一注册和登录, 统一授权认证
* Redis可以使用集群进行扩展, 加速授权和访问
* 用户信息也将缓存在Redis中, 方便各个项目统一, 快速读取用户信息
* 基于OAuth2实现, 服务端可以方便的控制接口的访问权限, 例如一旦client_id和client_secret泄露, 可立即封杀