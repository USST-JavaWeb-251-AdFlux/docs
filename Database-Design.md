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

# 3. 表结构设计

## 3.1 user（用户基础表）

用于统一管理登录账号。

| 字段名       | 类型         | 描述                                  |
| ------------ | ------------ | ------------------------------------- |
| userId       | BIGINT       | 主键                                  |
| userAccount  | VARCHAR(256) | 账号                                  |
| userPassword | VARCHAR(512) | 密码                                  |
| userRole     | VARCHAR(50)  | 用户角色：admin/advertiser/publicsher |
| createTime   | DATETIME     | 创建时间                              |

```mysql
create table if not exists user
(
    userId           bigint auto_increment comment 'id' primary key,
    userAccount  varchar(256)                           not null comment '账号',
    userPassword varchar(512)                           not null comment '密码',
    userRole     varchar(50) default 'advertiser'       not null comment '用户角色：admin/advertiser/publicsher',
    createTime   datetime     default CURRENT_TIMESTAMP not null comment '创建时间',
    UNIQUE KEY uk_userAccount (userAccount)
) comment '用户' collate = utf8mb4_unicode_ci;
```

---

## 3.2 advertisers_info（广告业主信息表）

广告业主的信息。

| 字段名        | 类型                        | 描述         |
| ------------- | --------------------------- | ------------ |
| advertiserId | BIGINT | 广告业主 ID |
| companyName | VARCHAR(256)             | 公司名称     |
| email         | VARCHAR(100)                | 邮箱         |
| phone         | VARCHAR(50)                 | 电话         |

```mysql
create table if not exists advertisers_info
(
    advertiserId bigint                               not null comment '广告业主 ID' primary key,
    companyName  varchar(256)                         not null comment '公司名称',
    email        varchar(100)                         null comment '邮箱',
    phone        varchar(50)                          null comment '电话'
) comment '广告业主信息表' collate = utf8mb4_unicode_ci;

```

---

## 3.3 advertiser_payment_info（广告业主付款信息）

用于存储广告主的付款方式。

| 字段名        | 类型                                | 描述        |
| ------------- | ----------------------------------- | ----------- |
| paymentid   | BIGINT       | 支付信息 ID |
| advertiserId | BIGINT | 广告业主 ID |
| cardNumber  | VARCHAR(200)                        | 银行卡号    |
| bankName    | VARCHAR(100)                        | 银行名称    |

```mysql
create table if not exists advertiser_payment_info
(
    paymentId    bigint                               not null comment '支付信息 ID' primary key,
    advertiserId bigint                               not null comment '广告业主 ID',
    cardNumber   varchar(200)                         not null comment '银行卡号',
    bankName     varchar(100)                         null comment '银行名称'
) comment '广告业主付款信息' collate = utf8mb4_unicode_ci;

```

---

## 3.4 publishers（网站站长信息表）

用于管理对接广告的内容网站站长相关数据。

| 字段名             | 类型                    | 描述           |
| ------------------ | ----------------------- | -------------- |
| websiteId        | BIGINT | 网站 ID        |
| publisherId      | BIGINT | 网站站长 ID    |
| websiteName      | VARCHAR(100)            | 网站名称       |
| domain | VARCHAR(200)     | 域名   |
| verificationToken | VARCHAR(200)            | 验证代码       |
| isVerified       | TYNYINT        | 是否已通过验证（0-未通过；1-通过） |
| createTime | DATETIME                | 创建时间     |
| verifyTime | DATETIME | 验证时间 |

```mysql
create table if not exists publishers
(
    websiteId         bigint                               not null comment '网站 ID' primary key,
    publisherId       bigint                               not null comment '网站站长 ID',
    websiteName       varchar(100)                         not null comment '网站名称',
    domain              varchar(200)                         not null comment '网站地址',
    verificationToken varchar(200)                         null comment '验证代码',
    isVerified        tinyint       default 0              not null comment '是否已通过验证',
    createTime        datetime      default CURRENT_TIMESTAMP not null comment '创建时间',
    verifyTime        datetime      default CURRENT_TIMESTAMP not null comment '验证时间'
) comment '网站站长信息表' collate = utf8mb4_unicode_ci;

```

---

## 3.5 ad_kinds（广告分类表）

用于管理员可添加的单一分类标签。

| 字段名       | 类型         | 描述        |
| ------------ | ------------ | ----------- |
| categoryId   | BIGINT       | 广告类别 ID |
| categoryName | VARCHAR(100) | 类别名称    |
| createTime   | DATETIME     | 创建时间    |

```mysql
create table if not exists ad_kinds
(
    categoryId   bigint                not null comment '广告类别 ID' primary key,
    categoryName varchar(100)                          not null comment '类别名称',
    createTime   datetime      default CURRENT_TIMESTAMP not null comment '创建时间'
) comment '广告分类表' collate = utf8mb4_unicode_ci;
```



---

## 3.6 advertisement（广告信息表）

存储广告的核心数据。

| 字段名           | 类型                                  | 描述                         |
| ---------------- | ------------------------------------- | ---------------------------- |
| adId       | BITINT           | 广告 ID                      |
| advertiserId   | BIGINT | 广告业主 ID                  |
| adType         | INT              | 广告类型（0-image; 1-video） |
| mediaUrl    | VARCHAR(255)                          | 素材路径                     |
| title            | VARCHAR(200)                          | 广告标题                     |
| skipUrl | VARCHAR(255)                          | 点击跳转地址                 |
| categoryId     | BIGINT | 广告类别 ID                  |
| weeklyBudget | DECIMAL(10,2)                         | 周预算                       |
| adFormat     | VARCHAR(50)                           | 广告尺寸或格式（如 300x250） |
| reviewStatus | INT | 审核状态（0-待审核; 1-通过; 2-拒绝） |
| isActive     | TYNYINT                        | 是否启用投放（0-是；1-否）   |
| createTime   | DATETIME                              | 创建时间                     |
| editTime | DATETIME | 编辑时间 |

```mysql
create table if not exists advertisement
(
    adId         bigint                                not null comment '广告 ID' primary key,
    advertiserId bigint                                not null comment '广告业主 ID',
    adType       int                                   not null comment '广告类型（0-image; 1-video）',
    mediaUrl     varchar(255)                          not null comment '素材路径',
    title        varchar(200)                          not null comment '广告标题',
    skipUrl      varchar(255)                          null comment '点击跳转地址',
    categoryId   bigint                                not null comment '广告类别 ID',
    weeklyBudget decimal(10,2)                         not null comment '周预算',
    adFormat     varchar(50)                           null comment '广告尺寸或格式（如 300x250）',
    reviewStatus int           default 0               not null comment '审核状态（0-待审核; 1-通过; 2-拒绝）',
    isActive     tinyint       default 0               not null comment '是否启用投放（0-是；1-否）',
    createTime   datetime      default CURRENT_TIMESTAMP not null comment '创建时间',
    editTime     datetime      default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP not null comment '编辑时间'
) comment '广告信息表' collate = utf8mb4_unicode_ci;

```



---

## 3.7 content_visit_stats（访问内容行为统计）

记录匿名用户每一次在内容网站上的访问行为，用于推荐广告。

| 字段名       | 类型                              | 描述         |
| ------------ | :-------------------------------- | ------------ |
| visitId  | BIGINT       | 主键         |
| publisherid | BIGINT | 网站站长 ID  |
| categoryId | BIGINT | 广告类别 ID  |
| userKey | CHAR(64)                          | 匿名用户标识 |
| duration     | INT                               | 停留时长     |
| timestamp    | DATETIME                          | 访问时间     |

```mysql
create table if not exists content_visit_stats
(
    visitId     bigint                                not null comment '主键' primary key,
    publisherId bigint                                not null comment '网站站长 ID',
    categoryId  bigint                                not null comment '广告类别 ID',
    userKey     char(64)                              not null comment '匿名用户标识',
    duration    int                                   not null comment '停留时长',
    timestamp   datetime      default CURRENT_TIMESTAMP not null comment '访问时间'
) comment '访问内容行为统计' collate = utf8mb4_unicode_ci;

```

---

## 3.8 user_category_stats（匿名用户类别统计表）

记录匿名用户在内容网站上的访问行为，用于推荐广告。

| 字段名      | 类型     | 描述             |
| ----------- | -------- | ---------------- |
| userKey     | CHAR(64) | 匿名用户标识     |
| category_id | BIGINT   | 广告类别 ID      |
| visitCount  | INT      | 该类别访问次数   |
| dwellTime   | INT      | 该类别总停留时长 |
| lastVisit   | DATETIME | 最近一次访问时间 |

```mysql
create table if not exists user_category_stats
(
    userKey     char(64)                              not null comment '匿名用户标识' primary key,
    categoryId  bigint                                not null comment '广告类别 ID',
    visitCount  int           default 0               not null comment '该类别访问次数',
    dwellTime   int           default 0               not null comment '该类别总停留时长',
    lastVisit   datetime      default CURRENT_TIMESTAMP not null comment '最近一次访问时间'
) comment '匿名用户类别统计表' collate = utf8mb4_unicode_ci;

```

---

# 4. 字段设计与业务规则说明

1. 用户采用统一 user 表管理，不同用户类型通过子表扩展。
2. 广告默认创建状态为：
   - audit_status = pending
   - is_active = false
3. 广告业主要上传：广告类型、素材文件、标题、跳转 URL、所属分类。
4. 广告预算 weekly_budget 必须为正数。
5. 广告审核只能由管理员执行。
6. 内容网站的验证通过 verification_token 实现，站长需在网站根目录放置指定文件。
7. 广告展示与点击统计通过ad_stats 维护。
8. 推荐算法依赖 content_visit_stats 和 user_category_stats。

---

# 5. 后续扩展方向

- 绑定多种付款方式
- 视频广告托管服务
- 更复杂的广告推荐算法
- 按小时统计广告数据
- 站点黑名单和广告过滤规则

---

# 6. 结语

本数据库设计覆盖当前全部功能需求，并预留扩展空间，可支持未来的统计分析、广告推荐、复杂投放逻辑等业务。
