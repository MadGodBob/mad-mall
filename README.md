# 商城系统学习方案

## 目录

- [1. 项目定位](#1-项目定位)
- [2. 技术栈与职责划分](#2-技术栈与职责划分)
  - [2.1 Spring Boot](#21-spring-boot)
  - [2.2 MySQL](#22-mysql)
  - [2.3 Redis](#23-redis)
  - [2.4 RabbitMQ](#24-rabbitmq)
  - [2.5 Search](#25-search)
- [3. 各个部件原理](#3-各个部件原理)
  - [3.1 Spring Boot 原理](#31-spring-boot-原理)
  - [3.2 Redis 原理](#32-redis-原理)
  - [3.3 RabbitMQ 原理](#33-rabbitmq-原理)
  - [3.4 Search 原理](#34-search-原理)
- [4. 可以增加的模块以供学习](#4-可以增加的模块以供学习)
  - [4.1 第一阶段必须有的核心模块](#41-第一阶段必须有的核心模块)
  - [4.2 第二阶段建议增加的进阶模块](#42-第二阶段建议增加的进阶模块)
  - [4.3 第三阶段可扩展的高级模块](#43-第三阶段可扩展的高级模块)
- [5. 数据库表的搭建（精简版）](#5-数据库表的搭建精简版)
  - [5.1 精简原则](#51-精简原则)
  - [5.2 第一阶段只保留的核心表](#52-第一阶段只保留的核心表)
  - [5.3 延后再加的表](#53-延后再加的表)
  - [5.4 搜索索引字段建议](#54-搜索索引字段建议)
  - [5.5 建表 SQL 示例](#55-建表-sql-示例)
- [6. 模块间调用关系](#6-模块间调用关系)
- [7. 学习流程](#7-学习流程)
  - [7.1 第一阶段：Spring Boot 基础搭建](#71-第一阶段spring-boot-基础搭建)
  - [7.2 第二阶段：数据库设计与业务分层](#72-第二阶段数据库设计与业务分层)
  - [7.3 第三阶段：接入 Redis](#73-第三阶段接入-redis)
  - [7.4 第四阶段：接入 RabbitMQ](#74-第四阶段接入-rabbitmq)
  - [7.5 第五阶段：接入 Search](#75-第五阶段接入-search)
  - [7.6 第六阶段：补订单与库存闭环](#76-第六阶段补订单与库存闭环)
  - [7.7 第七阶段：扩展秒杀与高并发能力](#77-第七阶段扩展秒杀与高并发能力)
- [8. 搭建过程](#8-搭建过程)
  - [8.1 第一步：创建项目](#81-第一步创建项目)
  - [8.2 第二步：连接 MySQL](#82-第二步连接-mysql)
  - [8.3 第三步：接 Redis](#83-第三步接-redis)
  - [8.4 第四步：接 RabbitMQ](#84-第四步接-rabbitmq)
  - [8.5 第五步：接 Search](#85-第五步接-search)
  - [8.6 第六步：打通订单链路](#86-第六步打通订单链路)
  - [8.7 第七步：补项目亮点](#87-第七步补项目亮点)
- [9. 推荐开发顺序](#9-推荐开发顺序)
- [10. 简历中可以强调的亮点](#10-简历中可以强调的亮点)
- [11. 最后建议](#11-最后建议)

## 1. 项目定位

本项目建议做成一个适合学习和面试展示的“小型商城后端系统”，核心目标不是一次性做成完整商业产品，而是通过一个业务主线把 `Spring Boot`、`Redis`、`RabbitMQ`、`Search` 串起来。

建议项目名称：

- `mall-learning`
- `smart-mall`
- `springboot-mall`

建议先做后端接口和管理后台能力，前端可以后补，或者只用 Swagger / Apifox 进行接口联调。

---

## 2. 技术栈与职责划分

### 2.1 Spring Boot

Spring Boot 是整个项目的基础框架，负责：

- Web 接口开发
- 业务分层组织
- 数据访问整合
- 中间件集成
- 配置管理
- 异常处理
- 参数校验
- 日志输出

在这个项目里，Spring Boot 相当于“总调度中心”，把商品、订单、用户、缓存、消息队列、搜索这些模块组织起来。

### 2.2 MySQL

MySQL 负责存储核心业务数据，适合保存：

- 用户
- 商品
- 分类
- 库存
- 订单
- 订单项
- 购物车
- 优惠券
- 操作日志

原则是：

- 强一致的数据优先放数据库
- 需要事务的数据优先放数据库
- 需要长期保存的数据优先放数据库

### 2.3 Redis

Redis 是高性能内存数据库，适合做缓存、计数、分布式控制。

在商城中常见用途：

- 商品详情缓存
- 分类列表缓存
- 热门商品排行
- 登录 token 存储
- 短信验证码
- 购物车临时存储
- 秒杀库存预扣减
- 用户限流
- 防重复提交
- 分布式锁

### 2.4 RabbitMQ

RabbitMQ 是消息队列，用来做异步解耦和削峰填谷。

在商城中常见用途：

- 下单后异步扣减库存
- 下单后异步发送通知
- 支付成功后异步更新订单状态
- 商品变更后异步同步搜索索引
- 超时未支付订单自动关闭

### 2.5 Search

这里的 Search 建议使用 `Elasticsearch` 或 `OpenSearch`。

它主要负责复杂搜索能力，例如：

- 商品关键词检索
- 高亮显示
- 按分类筛选
- 按价格区间筛选
- 按销量、价格、时间排序
- 搜索联想
- 热门搜索统计

数据库能查数据，但不适合做复杂全文搜索。搜索引擎的意义是让“查得快、查得准、查得灵活”。

---

## 3. 各个部件原理

## 3.1 Spring Boot 原理

### 3.1.1 自动配置

Spring Boot 最大的优势之一是自动配置。

原理可以简单理解为：

- 你引入一个 starter
- Spring Boot 检测类路径中是否存在相关依赖
- 如果存在，就自动帮你创建默认 Bean
- 你可以继续覆盖默认配置

例如：

- 引入 `spring-boot-starter-web`，自动配置 MVC
- 引入 `spring-boot-starter-data-redis`，自动配置 RedisTemplate / 连接工厂
- 引入 `spring-boot-starter-amqp`，自动配置 RabbitMQ 相关组件

### 3.1.2 IOC 与 AOP

Spring 的核心思想是 IOC 和 AOP。

IOC：

- 对象不由你自己 new
- 由 Spring 容器统一管理
- Bean 之间通过依赖注入关联

AOP：

- 把通用逻辑从业务逻辑里抽出来
- 比如日志、事务、权限校验、接口耗时统计

商城里最常见的 AOP 场景：

- 接口日志
- 幂等控制
- 统一异常处理
- 事务控制

### 3.1.3 分层结构

推荐按经典分层写：

- `controller`：接收请求，参数校验，返回结果
- `service`：处理核心业务逻辑
- `mapper` / `repository`：访问数据库
- `model` / `entity`：数据对象
- `dto` / `vo`：参数对象和返回对象

这样做的目的是让代码更容易扩展，避免把所有逻辑都塞进一个类。

---

## 3.2 Redis 原理

### 3.2.1 Redis 为什么快

Redis 快的主要原因：

- 基于内存
- 单线程执行命令，避免线程切换和锁竞争
- IO 多路复用
- 数据结构设计高效

注意：

- Redis 不是“只能单线程”，而是核心命令执行模型以单线程为主
- 新版本在网络处理等方面做了优化，但面试和学习阶段可以先按“内存 + 单线程命令模型 + IO 多路复用”理解

### 3.2.2 Redis 常见数据结构

商城项目重点掌握这些：

- `String`：缓存商品详情、token、验证码
- `Hash`：存储用户购物车
- `List`：较少用，可做简单消息队列
- `Set`：去重，如已领取优惠券用户集合
- `ZSet`：热门商品排行、热搜排行
- `Bitmap`：签到统计
- `Geo`：附近门店

### 3.2.3 缓存原理

缓存的基本思想：

- 先查 Redis
- Redis 没有，再查 MySQL
- 查到后回写 Redis

这是典型的 `Cache Aside Pattern`。

商城中适合缓存的数据：

- 商品详情
- 分类树
- 品牌列表
- 首页推荐

### 3.2.4 缓存问题

#### 1. 缓存穿透

查询一个根本不存在的数据：

- Redis 没有
- MySQL 也没有
- 每次请求都打到数据库

解决：

- 缓存空值
- 布隆过滤器

#### 2. 缓存击穿

一个热点 key 失效瞬间，大量请求同时打到数据库。

解决：

- 互斥锁
- 热点数据不过期
- 逻辑过期

#### 3. 缓存雪崩

大量 key 同时过期，数据库瞬间被打爆。

解决：

- 过期时间加随机值
- 多级缓存
- 服务降级和限流

### 3.2.5 分布式锁原理

在分布式环境下，多台服务同时操作同一份资源时，需要锁来控制并发。

Redis 分布式锁常见做法：

- `SET lockKey value NX EX 10`

意思是：

- key 不存在才设置成功
- 并且给锁设置过期时间

释放锁时要注意：

- 只能释放自己的锁
- 通常需要用 Lua 脚本保证“判断锁值 + 删除锁”是原子操作

### 3.2.6 秒杀库存原理

高并发秒杀时，直接查数据库扣库存很容易把数据库压垮。

常见思路：

1. 库存预热到 Redis
2. 请求先在 Redis 中原子扣减
3. 扣减成功后发送 MQ 消息
4. 消费者再落库生成订单
5. 失败则回滚或补偿

这里 Redis 的价值主要是抗并发和快速失败。

---

## 3.3 RabbitMQ 原理

### 3.3.1 核心概念

RabbitMQ 中最重要的概念有：

- `Producer`：生产者，发送消息
- `Consumer`：消费者，接收消息
- `Queue`：队列，存放消息
- `Exchange`：交换机，负责路由消息
- `RoutingKey`：路由规则
- `Binding`：交换机与队列的绑定关系

### 3.3.2 消息流转过程

基本流程：

1. 生产者发送消息到 Exchange
2. Exchange 根据路由规则把消息投递到 Queue
3. 消费者监听 Queue
4. 消费成功后 ACK

### 3.3.3 常见交换机类型

- `Direct`：精准匹配路由键
- `Topic`：模糊匹配路由键
- `Fanout`：广播到所有绑定队列
- `Headers`：按消息头匹配，业务中相对少见

商城项目常见用法：

- 订单创建消息：`Direct`
- 商品更新同步：`Topic`
- 广播通知：`Fanout`

### 3.3.4 为什么要用 MQ

#### 1. 异步

用户下单时，如果同步做所有事情：

- 扣库存
- 写订单
- 发短信
- 记日志
- 更新搜索索引

接口就会很慢。

用了 MQ 后，可以先完成核心流程，再异步处理非核心任务。

#### 2. 解耦

订单系统不需要直接依赖搜索系统、通知系统、积分系统。

它只需要发消息，其他系统自己消费。

#### 3. 削峰

秒杀或者大促时，瞬时请求很多。MQ 可以作为缓冲区，把瞬时压力摊平。

### 3.3.5 消息可靠性

学习时要重点理解这几个问题：

- 生产者消息如何不丢
- 消费者如何避免重复消费
- 消息消费失败如何重试
- 数据库和 MQ 如何保证最终一致性

常见方案：

- 生产者确认机制
- 持久化队列和消息
- 手动 ACK
- 死信队列
- 幂等消费
- 本地消息表 / 事务消息 / 补偿机制

### 3.3.6 延迟队列在商城中的用途

典型场景：订单超时未支付自动取消。

流程：

1. 创建订单时发送一个延迟消息
2. 到期后消费者检查订单状态
3. 如果未支付，则关闭订单并恢复库存

如果 RabbitMQ 版本和插件不方便，也可以用：

- TTL + 死信队列

---

## 3.4 Search 原理

### 3.4.1 倒排索引

搜索引擎的核心是倒排索引。

数据库通常是：

- 一条记录对应一行
- 想找“包含某关键词的内容”效率不高

倒排索引会把“词”和“文档”建立映射：

- 词 -> 出现在哪些文档中

这样搜索关键词时，不是全表扫描，而是快速定位相关文档。

### 3.4.2 分词

中文搜索必须考虑分词。

例如商品名：

- `华为手机Mate60`

搜索时可能会按：

- 华为
- 手机
- Mate60

进行切分。

所以你需要理解：

- 索引时分词
- 搜索时分词
- 两边分词器不一致会影响结果

### 3.4.3 搜索与数据库的分工

数据库负责：

- 准确存储
- 事务
- 精确查询

搜索引擎负责：

- 全文检索
- 排序
- 聚合
- 高亮
- 联想提示

一般模式是：

- MySQL 保存原始数据
- Search 保存检索副本

### 3.4.4 商品搜索在商城中的设计

搜索文档可以包含：

- 商品 ID
- 商品名称
- 副标题
- 分类名称
- 品牌名称
- 标签
- 销量
- 价格
- 上架状态
- 创建时间

常见能力：

- 按关键词搜索
- 按品牌筛选
- 按分类筛选
- 按价格区间筛选
- 排序
- 高亮展示

### 3.4.5 搜索数据同步原理

商品数据一般先更新 MySQL，再同步到搜索引擎。

同步方式有两种：

#### 1. 同步调用

商品保存后立即更新索引。

优点：

- 简单直接

缺点：

- 接口更慢
- 耦合高

#### 2. MQ 异步同步

商品保存后发消息，由消费者更新索引。

优点：

- 解耦
- 性能更好

缺点：

- 会有短暂延迟
- 需要处理一致性

商城项目建议用 MQ 异步同步，这样能把四个技术点串起来。

---

## 4. 可以增加的模块以供学习

下面按“推荐优先级”划分模块。

## 4.1 第一阶段必须有的核心模块

### 4.1.1 用户模块

功能建议：

- 用户注册
- 用户登录
- 用户信息查询
- 收货地址管理

学习点：

- 参数校验
- 密码加密
- token 设计
- Redis 存 token

### 4.1.2 商品模块

功能建议：

- 商品 SPU 管理
- 商品 SKU 管理
- 商品图片
- 商品详情
- 商品上下架

学习点：

- 表设计
- 一对多关系
- 分页查询
- 数据缓存

### 4.1.3 分类模块

功能建议：

- 一级分类
- 二级分类
- 分类树查询

学习点：

- 树形结构设计
- 分类缓存

### 4.1.4 购物车模块

功能建议：

- 加入购物车
- 修改数量
- 勾选商品
- 删除商品
- 查看购物车

学习点：

- Redis Hash
- 登录态处理
- 价格快照

### 4.1.5 订单模块

功能建议：

- 提交订单
- 订单列表
- 订单详情
- 取消订单
- 支付状态更新

学习点：

- 事务
- 状态流转
- 防重复下单
- 幂等

---

## 4.2 第二阶段建议增加的进阶模块

### 4.2.1 商品搜索模块

功能建议：

- 关键词搜索
- 条件筛选
- 排序
- 高亮

学习点：

- 搜索索引设计
- DTO 设计
- 索引同步

### 4.2.2 秒杀模块

功能建议：

- 秒杀活动
- 秒杀商品
- 秒杀库存控制
- 秒杀下单

学习点：

- Redis Lua
- 限流
- MQ 削峰
- 高并发防超卖

### 4.2.3 优惠券模块

功能建议：

- 优惠券发放
- 优惠券领取
- 下单使用优惠券

学习点：

- 规则设计
- 用户领取限制
- 状态控制

### 4.2.4 库存模块

功能建议：

- 普通库存
- 锁定库存
- 扣减库存
- 回滚库存

学习点：

- 预扣减
- 库存回补
- 一致性处理

### 4.2.5 支付模块

功能建议：

- 模拟支付
- 支付回调
- 支付成功更新订单

学习点：

- 幂等回调
- 签名校验的理解
- 延迟关单

---

## 4.3 第三阶段可扩展的高级模块

### 4.3.1 评价模块

- 商品评价
- 追评
- 图片评价
- 评分统计

### 4.3.2 会员积分模块

- 下单赠送积分
- 积分抵扣
- 积分流水

### 4.3.3 热门榜单模块

- 商品热度排行
- 搜索热词排行
- 分类热度排行

学习点：

- Redis ZSet
- 定时任务

### 4.3.4 消息通知模块

- 下单通知
- 发货通知
- 优惠券过期提醒

学习点：

- RabbitMQ 异步通知
- 重试机制

### 4.3.5 后台管理模块

- 商品管理
- 订单管理
- 用户管理
- 活动管理
- 数据看板

### 4.3.6 风控与限流模块

- 接口限流
- 恶意刷单识别
- 登录失败限制

学习点：

- Redis 计数
- 滑动窗口限流
- 黑名单机制

---

## 5. 数据库表的搭建（精简版）

如果目标是“先把商城主流程跑通”，不建议一开始就把表铺太满。学习版项目先保留最小闭环即可：用户登录、商品浏览、下单、查订单。

## 5.1 精简原则

第一阶段只保留满足主流程的表，原则如下：

- 能合并的先合并，先不强行区分 SPU / SKU
- 能放快照的先放快照，例如收货地址直接落到订单表
- 能不落库的先不落库，例如购物车可以先放 Redis
- 能放到第二阶段的模块全部后置，例如优惠券、秒杀、品牌、图片库

建议第一阶段只保留 5 张核心表：

- `user`
- `category`
- `product`
- `orders`
- `order_item`

## 5.2 第一阶段只保留的核心表

### 5.2.1 `user`

用途：保存用户基础信息。

关键字段建议：

- `id`
- `username`
- `password`
- `nickname`
- `phone`
- `status`
- `create_time`
- `update_time`

### 5.2.2 `category`

用途：商品分类。

关键字段建议：

- `id`
- `parent_id`
- `name`
- `sort`
- `status`
- `create_time`
- `update_time`

说明：

- 第一阶段只做两级分类就够了
- `level` 字段都可以先省掉，靠 `parent_id` 判断层级

### 5.2.3 `product`

用途：商品表，合并原来的 `product_spu` 和 `product_sku`。

关键字段建议：

- `id`
- `product_name`
- `category_id`
- `sale_price`
- `origin_price`
- `stock`
- `main_image`
- `product_status`
- `detail`
- `create_time`
- `update_time`

说明：

- 如果当前只是做单规格商品，没必要先拆 SPU / SKU
- 一张 `product` 表已经足够支撑商品列表、商品详情、库存扣减
- 以后如果想练多规格，再把它拆成 `product_spu` 和 `product_sku`

### 5.2.4 `orders`

用途：订单主表。

关键字段建议：

- `id`
- `order_no`
- `user_id`
- `total_amount`
- `pay_amount`
- `order_status`
- `receiver_name`
- `receiver_phone`
- `receiver_address`
- `remark`
- `create_time`
- `update_time`

建议状态：

- `0` 待支付
- `1` 已支付
- `2` 已取消
- `3` 已完成

说明：

- 第一阶段先不拆支付状态、发货状态
- 一个 `order_status` 就能支撑学习版流程

### 5.2.5 `order_item`

用途：订单商品明细。

关键字段建议：

- `id`
- `order_id`
- `order_no`
- `product_id`
- `product_name`
- `product_image`
- `price`
- `quantity`
- `total_amount`
- `create_time`

说明：

- 这里保存商品快照，避免商品后续改名影响历史订单

## 5.3 延后再加的表

以下表建议全部放到第二阶段以后，再按需求逐步补：

- `user_address`：如果要做“地址簿”再加；第一阶段可直接在下单时传地址
- `brand`：如果没有品牌筛选需求，可以先删
- `product_image`：先保留一张主图字段 `main_image` 即可
- `cart_item`：如果想练 Redis，购物车先不落 MySQL
- `coupon`、`user_coupon`：属于营销能力，不影响主流程
- `stock_log`：属于库存审计，后期再补
- `seckill_activity`、`seckill_product`：属于高并发扩展能力，放项目亮点阶段

这样删减后，数据库从“十几张表起步”降到“5 张核心表起步”，对学习和面试展示都更友好。

## 5.4 搜索索引字段建议

如果使用 Elasticsearch / OpenSearch，商品索引文档建议包含：

- `id`
- `productName`
- `categoryId`
- `categoryName`
- `price`
- `stock`
- `mainImage`
- `createTime`

其中：

- `productName` 适合全文检索
- `categoryId` 适合过滤
- `price` 适合排序

---

## 5.5 建表 SQL 示例

下面给出一套更适合学习版的最小 SQL 示例，足够用于第一阶段开发。

```sql
CREATE TABLE user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(64) NOT NULL UNIQUE,
    password VARCHAR(128) NOT NULL,
    nickname VARCHAR(64) DEFAULT NULL,
    phone VARCHAR(20) DEFAULT NULL,
    email VARCHAR(128) DEFAULT NULL,
    status TINYINT NOT NULL DEFAULT 1,
    create_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    update_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE category (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    parent_id BIGINT NOT NULL DEFAULT 0,
    name VARCHAR(64) NOT NULL,
    sort INT NOT NULL DEFAULT 0,
    status TINYINT NOT NULL DEFAULT 1,
    create_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    update_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE product (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    product_name VARCHAR(128) NOT NULL,
    category_id BIGINT NOT NULL,
    sale_price DECIMAL(10, 2) NOT NULL,
    origin_price DECIMAL(10, 2) DEFAULT NULL,
    stock INT NOT NULL DEFAULT 0,
    main_image VARCHAR(255) DEFAULT NULL,
    product_status TINYINT NOT NULL DEFAULT 1,
    detail TEXT DEFAULT NULL,
    create_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    update_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE orders (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    order_no VARCHAR(64) NOT NULL UNIQUE,
    user_id BIGINT NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    pay_amount DECIMAL(10, 2) NOT NULL,
    order_status TINYINT NOT NULL DEFAULT 0,
    receiver_name VARCHAR(64) DEFAULT NULL,
    receiver_phone VARCHAR(20) DEFAULT NULL,
    receiver_address VARCHAR(255) DEFAULT NULL,
    remark VARCHAR(255) DEFAULT NULL,
    create_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    update_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE order_item (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    order_id BIGINT NOT NULL,
    order_no VARCHAR(64) NOT NULL,
    product_id BIGINT NOT NULL,
    product_name VARCHAR(128) NOT NULL,
    product_image VARCHAR(255) DEFAULT NULL,
    price DECIMAL(10, 2) NOT NULL,
    quantity INT NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    create_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### 5.5.1 建表建议

- 所有表尽量统一主键风格
- 所有表统一 `create_time`、`update_time`
- 状态字段尽量使用枚举值约定
- 高频查询字段加索引，如 `user_id`、`category_id`、`product_id`、`order_no`

---

## 6. 模块间调用关系

建议先理解下面这条主线：

### 6.1 商品发布主线

1. 后台新增商品到 MySQL
2. 商品上架后发送 MQ 消息
3. 搜索模块消费消息并建立索引
4. 前台用户可搜索商品
5. 商品详情页优先查询 Redis，没有则查 MySQL 并回填缓存

### 6.2 下单主线

1. 用户登录
2. 加入购物车
3. 提交订单
4. 校验库存和价格
5. 创建订单
6. 发送延迟消息，超时未支付自动关闭
7. 支付成功后更新订单状态
8. 异步发送通知和更新库存流水

### 6.3 秒杀主线

1. 秒杀商品库存预热到 Redis
2. 用户请求进入秒杀接口
3. Redis Lua 原子判断库存和用户资格
4. 成功后写入抢购标记
5. 发送下单消息到 RabbitMQ
6. 消费者异步创建订单
7. 失败则记录原因并返回

---

## 7. 学习流程

学习这个项目，最容易失败的方式是“一上来就想把全部功能做完”。更合理的方式是按阶段推进，每一阶段只引入一个关键技术点。

## 7.1 第一阶段：Spring Boot 基础搭建

目标：

- 跑起一个能正常开发的 Spring Boot 项目
- 完成基础 CRUD

建议学习内容：

- Spring Boot 项目结构
- 配置文件 `application.yml`
- 统一返回对象
- 全局异常处理
- 参数校验
- MyBatis / MyBatis-Plus 或 JPA
- Lombok
- Swagger / knife4j

建议完成的接口：

- 用户注册登录
- 分类新增查询
- 商品新增查询

输出结果：

- 后端项目可以启动
- 能连上 MySQL
- 能完成基本增删改查

## 7.2 第二阶段：数据库设计与业务分层

目标：

- 把核心表先建起来
- 把代码分层结构稳定下来

建议完成：

- `user`
- `category`
- `product`
- `orders`
- `order_item`

同时完成：

- DTO / VO 设计
- service 层封装
- 分页查询
- 状态字段统一管理

## 7.3 第三阶段：接入 Redis

目标：

- 学会在商城业务中使用缓存

建议先做这几个点：

1. 用户登录 token 存 Redis
2. 商品详情缓存
3. 分类缓存
4. 热门商品排行

重点学习：

- RedisTemplate 基本使用
- key 设计规范
- 过期时间设计
- 缓存一致性
- 缓存击穿、穿透、雪崩

输出结果：

- 你能解释“为什么这里要用 Redis”
- 你能实现“查缓存 -> 查数据库 -> 回写缓存”

## 7.4 第四阶段：接入 RabbitMQ

目标：

- 学会在业务中做异步解耦

建议先做两个场景：

1. 商品上架后异步同步搜索索引
2. 订单创建后发送延迟关单消息

重点学习：

- Exchange / Queue / Binding
- 生产者发送消息
- 消费者监听消息
- ACK 机制
- 死信队列
- 重复消费幂等处理

输出结果：

- 你能完整讲清楚一条消息是怎么流转的
- 你能说明为什么同步调用改成异步调用

## 7.5 第五阶段：接入 Search

目标：

- 实现真实商品搜索

建议完成：

- 商品索引结构设计
- 商品上架时建立索引
- 搜索接口
- 条件筛选
- 排序
- 高亮

重点学习：

- 倒排索引
- mapping 设计
- keyword 和 text 区别
- 索引同步策略

输出结果：

- 你能搜索商品
- 你能解释数据库搜索和搜索引擎搜索的区别

## 7.6 第六阶段：补订单与库存闭环

目标：

- 让商城主流程基本可用

建议完成：

- 购物车
- 提交订单
- 扣库存
- 取消订单
- 模拟支付

重点学习：

- 事务
- 幂等
- 防止重复下单
- 数据一致性

## 7.7 第七阶段：扩展秒杀与高并发能力

目标：

- 做出项目亮点

建议完成：

- 秒杀活动
- Redis 预扣库存
- Lua 脚本原子操作
- RabbitMQ 削峰
- 限流
- 防超卖

这一阶段是简历中的高价值部分，因为能体现你不仅会 CRUD，还理解中间件的实际用途。

---

## 8. 搭建过程

下面给出一个适合学习的搭建顺序。

## 8.1 第一步：创建项目

建议依赖：

- Spring Web
- Lombok
- Validation
- MySQL Driver
- MyBatis / MyBatis-Plus
- Spring Data Redis
- Spring AMQP

如果要接搜索，再增加：

- Elasticsearch Client 或 OpenSearch Java Client

### 8.1.1 项目结构建议

```text
com.example.mall
├── controller
├── service
├── service.impl
├── mapper
├── entity
├── dto
├── vo
├── config
├── mq
├── search
├── common
└── exception
```

## 8.2 第二步：连接 MySQL

完成内容：

- 配置数据源
- 建表
- 配置 MyBatis
- 先跑通用户、分类、商品接口

建议先不引入复杂业务，只保证：

- 项目能启动
- 数据能增删改查

## 8.3 第三步：接 Redis

完成内容：

- 配置 Redis 连接
- 封装缓存工具类
- 实现商品缓存
- 实现 token 登录态

建议同时定义 key 规范，例如：

- `mall:user:token:{token}`
- `mall:product:detail:{productId}`
- `mall:category:tree`
- `mall:hot:product`

## 8.4 第四步：接 RabbitMQ

完成内容：

- 定义交换机、队列、路由键
- 商品上架发消息
- 消费者同步索引
- 创建订单时发送延迟消息

建议先做最简单的消息链路，再逐步补充：

- 确认机制
- 重试
- 死信队列

## 8.5 第五步：接 Search

完成内容：

- 建立商品索引
- 编写搜索接口
- 支持关键词和过滤查询

建议搜索接口参数：

- `keyword`
- `categoryId`
- `minPrice`
- `maxPrice`
- `sort`
- `pageNum`
- `pageSize`

## 8.6 第六步：打通订单链路

完成内容：

- 购物车结算
- 创建订单
- 扣减库存
- 模拟支付
- 延迟关闭订单

这里开始要重点关注：

- 事务边界
- 幂等
- 回滚和补偿

## 8.7 第七步：补项目亮点

建议按优先级补：

1. 秒杀
2. 热门搜索榜
3. 优惠券
4. 操作日志
5. 限流
6. 数据监控

---

## 9. 推荐开发顺序

如果你想最快做出一个“能讲、能演示、能写简历”的版本，推荐顺序如下：

1. Spring Boot + MySQL 跑通用户、商品、分类 CRUD
2. 做商品详情和分类缓存
3. 做用户登录与 Redis token
4. 做商品搜索
5. 做商品上架后 MQ 异步同步索引
6. 做购物车和订单
7. 做延迟关单
8. 做秒杀和限流

---

## 10. 简历中可以强调的亮点

项目完成后，你可以重点讲这些：

- 基于 Spring Boot 搭建商城系统，完成商品、分类、购物车、订单等核心模块
- 使用 Redis 实现商品缓存、登录态存储、热榜统计和秒杀库存预扣减
- 使用 RabbitMQ 实现商品索引异步同步、订单超时关闭、异步通知等功能
- 基于 Elasticsearch / OpenSearch 实现商品全文搜索、筛选、排序和高亮
- 通过缓存空值、互斥锁、随机过期时间解决缓存穿透、击穿和雪崩问题
- 通过 Redis Lua、MQ 削峰和幂等控制提升秒杀场景下的稳定性

---

## 11. 最后建议

这个项目不建议一开始就追求“功能很多”，而是要先保证每个技术点都真正落在业务里。

建议你按这个原则推进：

- 先实现业务主线
- 再引入中间件优化
- 最后补高并发亮点

如果只从学习价值看，最值得优先做好的四条主线是：

1. 商品 CRUD
2. 商品缓存
3. 商品搜索
4. 订单 + MQ 异步处理

把这四条做好，你的项目就已经具备很好的学习和面试价值了。
