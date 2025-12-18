# 数据库设计文档（Database Design）

本数据库设计用于支持广告管理系统，包括管理员、广告业主、内容网站三类用户的核心业务流程：广告上传、审核、投放、统计、站点接入广告等功能。

---

# 1. 系统概述

本系统的数据库用于管理：

- 用户与角色（管理员 / 广告业主 / 内容站点负责人）
- 广告分类
- 广告基本信息与审核流程
- 广告预算与投放状态
- 广告展示与点击统计
- 内容网站信息与验证流程
- 广告事件级统计与推荐相关行为数据

---

# 2. 数据库 ER 图

（在此处插入 er_diagram.png）

---

# 3. 广告表结构设计

## 3.1 users（用户基础表）

用于统一管理登录账号。

| 字段名       | 类型         | 描述                                 |
| ------------ | ------------ | ------------------------------------ |
| userId       | BIGINT       | 主键                                 |
| username     | VARCHAR(256) | 用户名                               |
| userPassword | VARCHAR(512) | 密码 Hash (例如 Bcrypt) 禁止明文     |
| userRole     | VARCHAR(50)  | 用户角色：admin/advertiser/publisher |
| email        | VARCHAR(100) | 邮箱                                 |
| phone        | VARCHAR(50)  | 电话                                 |
| createTime   | DATETIME     | 创建时间                             |

```mysql
create table if not exists users
(
    userId           bigint auto_increment comment 'id' primary key,
    username  varchar(256)                           not null comment '用户名',
    userPassword varchar(512)                           not null comment '密码 Hash',
    userRole     varchar(50) default 'advertiser'       not null comment '用户角色：admin/advertiser/publisher',
    email        varchar(100)                         null comment '邮箱',
    phone        varchar(50)                          null comment '电话',
    createTime   datetime     default CURRENT_TIMESTAMP not null comment '创建时间',
    UNIQUE KEY uk_username (username)
) comment '用户' collate = utf8mb4_unicode_ci;
```

---

## 3.2 advertisers（广告业主信息表）

广告业主的信息。

| 字段名       | 类型         | 描述        |
| ------------ | ------------ | ----------- |
| advertiserId | BIGINT       | 广告业主 ID |
| companyName  | VARCHAR(256) | 公司名称    |

```mysql
create table if not exists advertisers
(
    advertiserId bigint                               not null comment '广告业主 ID' primary key,
    companyName  varchar(256)                         not null comment '公司名称',
    FOREIGN KEY (advertiserId) REFERENCES users(userId)
) comment '广告业主信息表' collate = utf8mb4_unicode_ci;

```

---

## 3.3 advertiser_payments（广告业主付款信息）

用于存储广告主的付款方式。

| 字段名       | 类型         | 描述        |
| ------------ | ------------ | ----------- |
| paymentId    | BIGINT       | 支付信息 ID |
| advertiserId | BIGINT       | 广告业主 ID |
| cardNumber   | VARCHAR(200) | 银行卡号    |
| bankName     | VARCHAR(100) | 银行名称    |

```mysql
create table if not exists advertiser_payments
(
    paymentId    bigint                               not null comment '支付信息 ID' primary key,
    advertiserId bigint                               not null comment '广告业主 ID',
    cardNumber   varchar(200)                         not null comment '银行卡号',
    bankName     varchar(100)                         null comment '银行名称',
    FOREIGN KEY (advertiserId) REFERENCES users(userId)
) comment '广告业主付款信息' collate = utf8mb4_unicode_ci;

```

---

## 3.4 publishers（网站站长信息表）

用于管理对接广告的内容网站站长相关数据。

| 字段名            | 类型         | 描述                               |
| ----------------- | ------------ | ---------------------------------- |
| websiteId         | BIGINT       | 网站 ID                            |
| publisherId       | BIGINT       | 网站站长 ID                        |
| websiteName       | VARCHAR(100) | 网站名称                           |
| domain            | VARCHAR(200) | 域名                               |
| verificationToken | VARCHAR(200) | 验证代码                           |
| isVerified        | TINYINT      | 是否已通过验证（0-未通过；1-通过） |
| createTime        | DATETIME     | 创建时间                           |
| verifyTime        | DATETIME     | 验证时间                           |

```mysql
create table if not exists publishers
(
    websiteId         bigint                               not null comment '网站 ID' primary key,
    publisherId       bigint                               not null comment '网站站长 ID',
    websiteName       varchar(100)                         not null comment '网站名称',
    domain            varchar(200)                         not null comment '网站地址',
    verificationToken varchar(200)                         null comment '验证代码',
    isVerified        tinyint       default 0              not null comment '是否已通过验证',
    createTime        datetime      default CURRENT_TIMESTAMP not null comment '创建时间',
    verifyTime        datetime      default NULL comment '验证时间',
    FOREIGN KEY (publisherId) REFERENCES users(userId)
) comment '网站站长信息表' collate = utf8mb4_unicode_ci;

```

---

## 3.5 ad_categories（广告分类表）

用于管理员可添加的单一分类标签。

| 字段名       | 类型         | 描述        |
| ------------ | ------------ | ----------- |
| categoryId   | BIGINT       | 广告类别 ID |
| categoryName | VARCHAR(100) | 类别名称    |
| createTime   | DATETIME     | 创建时间    |

```mysql
create table if not exists ad_categories
(
    categoryId   bigint                not null comment '广告类别 ID' primary key,
    categoryName varchar(100)                          not null comment '类别名称',
    createTime   datetime      default CURRENT_TIMESTAMP not null comment '创建时间'
) comment '广告分类表' collate = utf8mb4_unicode_ci;
```

---

## 3.6 advertisements（广告信息表）

存储广告的核心数据。

| 字段名       | 类型          | 描述                                      |
| ------------ | ------------- | ----------------------------------------- |
| adId         | BIGINT        | 广告 ID                                   |
| advertiserId | BIGINT        | 广告业主 ID                               |
| adType       | INT           | 广告类型（0-image; 1-video）              |
| mediaUrl     | VARCHAR(255)  | 素材路径                                  |
| title        | VARCHAR(200)  | 广告标题                                  |
| landingPage  | VARCHAR(255)  | 点击跳转地址                              |
| categoryId   | BIGINT        | 广告类别 ID                               |
| adLayout     | VARCHAR(20)   | 广告版式（0-banner； 1-sidebar； 2-card） |
| weeklyBudget | DECIMAL(10,2) | 周预算                                    |
| reviewStatus | INT           | 审核状态（0-待审核; 1-通过; 2-拒绝）      |
| isActive     | TINYINT       | 是否启用投放（0-否；1-是）                |
| createTime   | DATETIME      | 创建时间                                  |
| editTime     | DATETIME      | 编辑时间                                  |

```mysql
create table if not exists advertisements
(
    adId         bigint                                not null comment '广告 ID' primary key,
    advertiserId bigint                                not null comment '广告业主 ID',
    adType       int                                   not null comment '广告类型（0-image; 1-video）',
    mediaUrl     varchar(255)                          not null comment '素材路径',
    title        varchar(200)                          not null comment '广告标题',
    landingPage  varchar(255)                          null comment '点击跳转地址',
    categoryId   bigint                                not null comment '广告类别 ID',
    adLayout     varchar(20)                           not null comment '广告版式（0-banner； 1-sidebar； 2-card）',
    weeklyBudget decimal(10,2)                         not null comment '周预算',
    reviewStatus int           default 0               not null comment '审核状态（0-待审核; 1-通过; 2-拒绝）',
    isActive     tinyint       default 0               not null comment '是否启用投放（0-否；1-是）',
    createTime   datetime      default CURRENT_TIMESTAMP not null comment '创建时间',
    editTime     datetime      default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP not null comment '编辑时间',
    FOREIGN KEY (advertiserId) REFERENCES users(userId),
    FOREIGN KEY (categoryId) REFERENCES ad_categories(categoryId)
) comment '广告信息表' collate = utf8mb4_unicode_ci;

```

---

## 3.7 content_visit_stats（访问内容行为统计）

记录匿名用户每一次在内容网站上的访问行为，用于推荐广告。

| 字段名     | 类型     | 描述         |
| ---------- | :------- | ------------ |
| visitId    | BIGINT   | 主键         |
| websiteId  | BIGINT   | 网站 ID      |
| categoryId | BIGINT   | 广告类别 ID  |
| trackId    | CHAR(36) | 匿名用户标识 |
| duration   | INT      | 停留时长     |
| timestamp  | DATETIME | 访问时间     |

```mysql
create table if not exists content_visit_stats
(
    visitId     bigint                                not null comment '主键' primary key,
    websiteId   bigint                                not null comment '网站 ID',
    categoryId  bigint                                not null comment '广告类别 ID',
    trackId     char(36)                              not null comment '匿名用户标识',
    duration    int                                   not null comment '停留时长',
    timestamp   datetime      default CURRENT_TIMESTAMP not null comment '访问时间',
    FOREIGN KEY (websiteId) REFERENCES publishers(websiteId),
    FOREIGN KEY (categoryId) REFERENCES ad_categories(categoryId)
) comment '访问内容行为统计' collate = utf8mb4_unicode_ci;

```

---

## 3.8 ad_displays（广告展示表）

记录每一次向匿名用户展示广告和相关统计信息，用于统计和推荐广告。

| 字段名      | 类型     | 描述         |
| ----------- | -------- | ------------ |
| displayId   | BIGINT   | 主键         |
| trackId     | CHAR(36) | 匿名用户标识 |
| adId        | BIGINT   | 广告 ID      |
| websiteId   | BIGINT   | 网站 ID      |
| duration    | INT      | 停留时长     |
| clicked     | BOOL     | 是否点击     |
| displayTime | DATETIME | 展示时间     |
| clickTime   | DATETIME | 点击时间     |

```mysql
create table if not exists ad_displays
(
    displayId   bigint                                not null comment '主键' primary key,
    trackId     char(36)                              not null comment '匿名用户标识',
    adId        bigint                                not null comment '广告 ID',
    websiteId   bigint                                not null comment '网站 ID',
    duration    int           default 0               not null comment '停留时长',
    clicked     tinyint       default 0               not null comment '是否点击',
    displayTime datetime      default CURRENT_TIMESTAMP not null comment '展示时间',
    clickTime   datetime      default NULL comment '点击时间',
    FOREIGN KEY (adId) REFERENCES advertisements(adId)
) comment '广告展示表' collate = utf8mb4_unicode_ci;

```

---

## 3.9 ad_placements(广告位表)

| 字段名        | 类型         | 描述       |
| ------------- | ------------ | ---------- |
| placementId   | BIGINT       | 广告位 ID  |
| websiteId     | BIGINT       | 网站 ID    |
| placementName | VARCHAR(100) | 广告位名称 |
| adLayout      | VARCHAR(20)  | 广告版式   |
| createTime    | DATETIME     | 创建时间   |

```mysql
create table if not exists ad_placements
(
    placementId   bigint                               not null comment '广告位 ID' primary key,
    websiteId     bigint                               not null comment '网站 ID，关联 publishers.websiteId',
    placementName varchar(100)                         not null comment '广告位名称',
    adLayout      varchar(20)                          not null comment '广告版式',
    createTime    datetime     default CURRENT_TIMESTAMP not null comment '创建时间',
    foreign key (websiteId) references publishers(websiteId)
) comment '广告位表' collate = utf8mb4_unicode_ci;

```
---

# 4. 新闻表结构设计

## 4.1 news_categories（新闻分类表）

| 字段名       | 类型         | 描述        |
| ------------ | ------------ | ----------- |
| categoryId   | BIGINT       | 新闻类别 ID |
| categoryName | VARCHAR(100) | 类别名称    |
| createTime   | DATETIME     | 创建时间    |
```mysql
create table if not exists news_categories
(
    categoryId   bigint                              not null comment '新闻类别 ID' primary key,
    categoryName varchar(100)                        not null comment '类别名称，例如：科技、体育',
    createTime   datetime default CURRENT_TIMESTAMP  not null comment '创建时间'
) comment '新闻分类表' collate = utf8mb4_unicode_ci;

```
---

## 4.2 news（新闻内容表）

| 字段名     | 类型         | 描述        |
| ---------- | ------------ | ----------- |
| newsId     | BIGINT       | 新闻 ID     |
| title      | VARCHAR(255) | 新闻标题    |
| content    | text         | 新闻正文    |
| categoryId | BIGINT       | 新闻类别 ID |
| createTime | DATETIME     | 创建时间    |

```mysql
create table if not exists news
(
    newsId      bigint                                not null comment '新闻 ID' primary key,
    title       varchar(255)                          not null comment '新闻标题',
    content     text                                  not null comment '新闻正文',
    categoryId  bigint                                not null comment '新闻类别 ID，关联 news_categories.categoryId',
    coverUrl    varchar(500)      default null        null comment '封面图片地址',
    createTime  datetime          default CURRENT_TIMESTAMP not null comment '创建时间',
    FOREIGN KEY (categoryId) REFERENCES news_categories(categoryId)
) comment '新闻内容表' collate = utf8mb4_unicode_ci;

```
---

# 5. 购物表结构设计

## 5.1 product_categories（商品分类表）

| 字段名       | 类型         | 描述        |
| ------------ | ------------ | ----------- |
| categoryId   | BIGINT       | 商品类别 ID |
| categoryName | VARCHAR(100) | 类别名称    |
| createTime   | DATETIME     | 创建时间    |

```mysql
create table if not exists product_categories
(
    categoryId   bigint                              not null comment '商品类别 ID' primary key,
    categoryName varchar(100)                        not null comment '类别名称',
    createTime   datetime default CURRENT_TIMESTAMP  not null comment '创建时间'
) comment '商品分类表' collate = utf8mb4_unicode_ci;

```
---

## 5.2 products（商品内容表）

| 字段名      | 类型           | 描述        |
| ----------- | -------------- | ----------- |
| productId   | BIGINT         | 商品 ID    |
| productName | VARCHAR(255)   | 商品名称    |
| price       | decimal(10, 2) | 商品价格    |
| description | text           | 商品描述    |
| imageUrl    | varchar(255)   | 商品图片URL |
| categoryId  | BIGINT         | 商品类别 ID |
| createTime  | DATETIME       | 创建时间    |

```mysql
create table if not exists products
(
    productId    bigint                               not null comment '商品 ID' primary key,
    productName  varchar(255)                         not null comment '商品名称',
    price        decimal(10, 2)                       not null comment '商品价格',
    description  text                                 null comment '商品描述',
    imageUrl     varchar(255)     default null        null comment '商品图片 URL',
    categoryId   bigint                               not null comment '商品类别 ID，关联 product_categories.categoryId',
    createTime   datetime         default CURRENT_TIMESTAMP not null comment '创建时间',
    foreign key (categoryId) references product_categories(categoryId)
) comment '商品内容表' collate = utf8mb4_unicode_ci;

```
---

# 6. 视频表结构设计

## 6.1 video_categories（视频分类表）

| 字段名       | 类型         | 描述        |
| ------------ | ------------ | ----------- |
| categoryId   | BIGINT       | 视频类别 ID |
| categoryName | VARCHAR(100) | 类别名称    |
| createTime   | DATETIME     | 创建时间    |

```mysql
create table if not exists video_categories
(
    categoryId   bigint                              not null comment '视频类别 ID' primary key,
    categoryName varchar(100)                        not null comment '类别名称',
    createTime   datetime default CURRENT_TIMESTAMP  not null comment '创建时间'
) comment '视频分类表' collate = utf8mb4_unicode_ci;

```
---

## 6.2 videos（视频内容表）

| 字段名     | 类型         | 描述         |
| ---------- | ------------ | ------------ |
| videoId    | BIGINT       | 视频 ID      |
| video      | VARCHAR(255) | 视频标题     |
| videoUrl   | VARCHAR(255) | 视频文件地址 |
| coverUrl   | VARCHAR(255) | 封面图       |
| duration   | BIGINT       | 视频时长     |
| categoryId | BIGINT       | 视频类别 ID  |
| createTime | DATETIME     | 创建时间     |

```mysql
create table if not exists videos
(
    videoId     bigint                               not null comment '视频 ID' primary key,
    video       varchar(255)                         not null comment '视频标题',
    videoUrl    varchar(255)                         not null comment '视频文件地址',
    coverUrl    varchar(255)      default null       null comment '封面图',
    duration    bigint                               not null comment '视频时长',
    categoryId  bigint                               not null comment '视频类别 ID，关联 video_categories.categoryId',
    createTime  datetime          default CURRENT_TIMESTAMP not null comment '创建时间',
    foreign key (categoryId) references video_categories(categoryId)
) comment '视频内容表' collate = utf8mb4_unicode_ci;


```
