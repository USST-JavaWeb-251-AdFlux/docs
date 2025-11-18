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

## 3.1 users（用户基础表）

用于统一管理登录账号。

| 字段名     | 类型                                        | 描述     |
| ---------- | ------------------------------------------- | -------- |
| user_id    | INT PK AUTO_INCREMENT                       | 用户 ID  |
| username   | VARCHAR(50) UNIQUE                          | 登录名   |
| password   | VARCHAR(255)                                | 密码     |
| user_type  | ENUM('superadmin','advertiser','publisher') | 用户类型 |
| created_at | DATETIME                                    | 创建时间 |

---

## 3.2 advertisers（广告业主信息表，可选）

扩展广告业主的专属信息。

| 字段名        | 类型                        | 描述         |
| ------------- | --------------------------- | ------------ |
| advertiser_id | INT PK, FK → users.user_id | 广告业主 ID |
| company_name  | VARCHAR(100)                | 公司名称     |
| email         | VARCHAR(100)                | 邮箱         |
| phone         | VARCHAR(50)                 | 电话         |

---

## 3.3 advertiser_payment_info（广告业主付款信息）

用于存储广告主的付款方式（可扩展）。

| 字段名        | 类型                                | 描述        |
| ------------- | ----------------------------------- | ----------- |
| payment_id    | INT PK AUTO_INCREMENT               | 支付信息 ID |
| advertiser_id | INT FK → advertisers.advertiser_id | 广告业主 ID |
| card_number   | VARCHAR(200)                        | 银行卡号    |
| bank_name     | VARCHAR(100)                        | 银行名称    |

---

## 3.4 publishers（网站站长信息表）

用于管理对接广告的内容网站站长相关数据。

| 字段名             | 类型                    | 描述           |
| ------------------ | ----------------------- | -------------- |
| website_id         | INT PK AUTO_INCREMENT   | 网站 ID        |
| publisher_id       | INT FK → users.user_id | 网站站长 ID    |
| website_name       | VARCHAR(100)            | 网站名称       |
| website_url        | VARCHAR(200) UNIQUE     | 网站地址       |
| verification_token | VARCHAR(200)            | 验证代码       |
| is_verified        | BOOLEAN                 | 是否已通过验证 |
| verified_at        | DATETIME                | 验证时间       |

---

## 3.5 categories（广告分类表）

用于管理员可添加的单一分类标签。

| 字段名        | 类型                  | 描述        |
| ------------- | --------------------- | ----------- |
| category_id   | INT PK AUTO_INCREMENT | 广告类别 ID |
| category_name | VARCHAR(100)          | 类别名称    |
| created_at    | DATETIME              | 创建时间    |

---

## 3.6 ads（广告信息表）

存储广告的核心数据。

| 字段名           | 类型                                  | 描述                         |
| ---------------- | ------------------------------------- | ---------------------------- |
| ad_id            | INT PK AUTO_INCREMENT                 | 广告 ID                      |
| advertiser_id    | INT FK → advertisers.advertiser_id   | 广告业主 ID                  |
| ad_type          | ENUM('image','video')                 | 广告类型                     |
| media_url        | VARCHAR(255)                          | 素材路径                     |
| title            | VARCHAR(200)                          | 广告标题                     |
| landing_page_url | VARCHAR(255)                          | 点击跳转地址                 |
| category_id      | INT FK → categories.category_id      | 广告类别 ID                  |
| weekly_budget    | DECIMAL(10,2)                         | 周预算                       |
| ad_format        | VARCHAR(50)                           | 广告尺寸或格式（如 300x250） |
| audit_status     | ENUM('pending','approved','rejected') | 审核状态                     |
| is_active        | BOOLEAN                               | 是否启用投放                 |
| created_at       | DATETIME                              | 创建时间                     |

---

## 3.7 content_visit_stats（访问内容行为统计）

记录匿名用户每一次在内容网站上的访问行为，用于推荐广告。

| 字段名       | 类型                              | 描述         |
| ------------ | --------------------------------- | ------------ |
| visit_id     | INT PK AUTO_INCREMENT             | 主键         |
| publisher_id | INT FK → publishers.publisher_id | 网站站长 ID  |
| category_id  | INT FK → categories.category_id  | 广告类别 ID  |
| anon_user_id | CHAR(64)                          | 匿名用户标识 |
| duration     | INT                               | 停留时长     |
| timestamp    | DATETIME                          | 访问时间     |

---

## 3.8 user_category_stats（匿名用户类别统计表）

记录匿名用户在内容网站上的访问行为，用于推荐广告。

| 字段名       | 类型                                 | 描述             |
| ------------ | ------------------------------------ | ---------------- |
| anon_user_id | CHAR(64) PK                          | 匿名用户标识     |
| category_id  | INT PK FK → categories.category_id | 广告类别 ID      |
| visit_count  | INT                                  | 该类别访问次数   |
| dwell_time   | INT                                  | 该类别总停留时长 |
| last_visit   | DATETIME                             | 最近一次访问时间 |

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
