
# 广告管理系统 API 文档

## 0. 通用说明

### 0.1 统一响应结构

所有接口（除个别健康检查外）统一返回 JSON，结构如下：

| 字段名 | 类型   | 是否必须 | 说明                             |
| ------ | ------ | -------- | -------------------------------- |
| code   | number | 必须     | 响应码，0 表示成功，其他表示失败 |
| data   | object | 非必须   | 具体业务数据，结构因接口而不同   |
| msg    | string | 非必须   | 提示信息（成功或错误原因）       |

示例：

```json
{
  "code": 0,
  "data": { },
  "msg": "success"
}
```

### 0.2 鉴权与身份

- 登录成功后，后端签发 **JWT Token**。
- 前端在后续请求中通过 HTTP Header 传递：

| Header 名称   | 示例                        |
| ------------- | --------------------------- |
| Authorization | `Bearer eyJhbGciOiJIUzI...` |

角色说明（对应 `users.userRole`）：

- `admin`：管理员
- `advertiser`：广告业主
- `publisher`：网站站长

---

## 1. 基础服务与认证 (Auth & Common)

### 1.1 用户注册

#### 1.1.1 基本信息

- **请求路径**：`/auth/register`
- **请求方式**：`POST`
- **接口描述**：用户注册为广告业主或网站站长，写入 `users` 表（必要时再由后端插入 `advertisers` / `publishers` 关联数据）。

#### 1.1.2 请求参数

- **参数格式**：`application/json`

| 参数名       | 类型   | 是否必需 | 说明                                         |
| ------------ | ------ | -------- | -------------------------------------------- |
| username     | string | 必须     | 登录用户名，唯一                             |
| userPassword | string | 必须     | 登录密码明文，后端保存为 `userPassword` Hash |
| role         | string | 必须     | 用户角色：`advertiser` 或 `publisher`        |
| email        | string | 非必须   | 邮箱，对应 `users.email`                     |
| phone        | string | 非必须   | 电话，对应 `users.phone`                     |

请求示例：

```json
{
  "username": "alice",
  "userPassword": "123456",
  "role": "advertiser",
  "email": "alice@example.com",
  "phone": "13800000000"
}
```

#### 1.1.3 响应数据

- **参数格式**：`application/json`

`data` 字段结构：userId

响应示例：

```json
{
    "code": 0,
    "data": 1999839285981343745,
    "message": "ok"
}
```

---

### 1.2 用户登录

#### 1.2.1 基本信息

- **请求路径**：`/auth/login`
- **请求方式**：`POST`
- **接口描述**：用户登录，验证 `users` 表用户名和密码，返回 Token 和用户基本信息。

#### 1.2.2 请求参数

- **参数格式**：`application/json`

| 参数名       | 类型   | 是否必需 | 说明   |
| ------------ | ------ | -------- | ------ |
| username     | string | 必须     | 用户名 |
| userPassword | string | 必须     | 密码   |

请求示例：

```json
{
    "username": "qiao2",
    "userPassword": "Password123!"
}
```

#### 1.2.3 响应数据

`data` 字段结构：

| 参数名   | 类型   | 是否必需 | 说明                                         |
| -------- | ------ | -------- | -------------------------------------------- |
| username | string | 必须     | 用户名                                       |
| userRole | string | 必须     | 角色：`admin` / `advertiser` / `publisher`   |
| token    | string | 必须     | JWT Token，前端存储后放入 `Authorization` 头 |

响应示例：

```json
{
    "code": 0,
    "data": {
        "username": "qiao2",
        "userRole": "advertiser",
        "token": "eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyUm9sZSI6ImFkdmVydGlzZXIiLCJ1c2VySWQiOjE5OTk4MzkyODU5ODEzNDM3NDUsInVzZXJuYW1lIjoicWlhbzIiLCJleHAiOjE3NjU3MjA0NjV9.zJphnqlNDhUouBC6zBojLPRMH1IC3CXhwLxP-YIQvjU"
    },
    "message": "ok"
}
```

---

### 1.3 退出登录

该接口后端不需要实现

#### 1.3.1 基本信息

- **请求路径**：`/auth/logout`
- **请求方式**：`POST`
- **接口描述**：服务端清理当前 Token（如放入黑名单），前端删除本地 Token。

#### 1.3.2 请求参数

- **请求参数**：无（仅需在 Header 中带上 Token）

#### 1.3.3 响应数据

- `data` 为空或 `null`。

```json
{
  "code": 1,
  "msg": "logout success",
  "data": null
}
```

---

### 1.4 获取当前登录用户信息

#### 1.4.1 基本信息

- **请求路径**：`/users/me`
- **请求方式**：`GET`
- **接口描述**：根据 Token 获取当前登录用户的基础信息（来自 `users`）。

#### 1.4.2 请求参数

- 无（从 Token 中解析）

#### 1.4.3 响应数据

`data` 字段结构：

| 参数名     | 类型   | 是否必需 | 说明                                |
| ---------- | ------ | -------- | ----------------------------------- |
| username   | string | 必须     | 用户名                              |
| userRole   | string | 必须     | 角色                                |

```json
{
    "code": 0,
    "data": {
        "username": "qiao2",
        "userRole": "advertiser",
    },
    "message": "ok"
}
```



---

### 1.5 修改当前用户信息（邮箱、电话）(待定)

#### 1.5.1 基本信息

- **请求路径**：`/users/me`
- **请求方式**：`PUT`
- **接口描述**：修改当前登录用户的 `email`、`phone` 字段。

#### 1.5.2 请求参数

- **参数格式**：`application/json`

| 参数名 | 类型   | 是否必需 | 说明   |
| ------ | ------ | -------- | ------ |
| email  | string | 非必须   | 新邮箱 |
| phone  | string | 非必须   | 新电话 |

请求示例：

```json
{
  "email": "newalice@example.com",
  "phone": "13900000000"
}
```

#### 1.5.3 响应数据

- `data`：更新后的用户简要信息（结构同 `/users/me`）。

---

### 1.6 通用文件上传

#### 1.6.1 基本信息

- **请求路径**：`file/uploads`
- **请求方式**：`POST`
- **接口描述**：上传图片或视频，保存到服务器本地目录，返回可访问的 `mediaUrl`，供 `advertisements.mediaUrl` 使用。

#### 1.6.2 请求参数

- **参数格式**：`multipart/form-data`

| 参数名 | 类型 | 是否必需 | 说明                       |
| ------ | ---- | -------- | -------------------------- |
| file   | file | 必须     | 上传文件（图片 / 视频）    |

请求示例（表单）：

- `file`：选择本地文件 `banner.png`

#### 1.6.3 响应数据

响应示例：

```json
{
    "code": 0,
    "data": "/api/file/2025/12/13/0bcec2c5-42bd-4bf6-ac30-43cea429898a.jpg",
    "message": "ok"
}
```

访问格式：http://localhost:8820/api/ad-resource/2025/12/14/ad75beae-e61a-418b-9b97-8a3d7c170ab4.jpg

---

## 2. 广告业主模块 (Advertiser)

### 2.1 获取业主公司信息

#### 2.1.1 基本信息

- **请求路径**：`/advertisers/profile`
- **请求方式**：`GET`
- **接口描述**：广告主查看自己的公司信息（来自 `advertisers` + `users`）。

#### 2.1.2 请求参数

- 无（根据当前登录用户的 `userId` 查 `advertisers.advertiserId`）。

#### 2.1.3 响应数据

`data` 字段结构：

| 参数名       | 类型   | 是否必需 | 说明                                  |
| ------------ | ------ | -------- | ------------------------------------- |
| advertiserId | number | 必须     | 广告业主 ID，对应 `advertiserId`     |
| userId       | number | 必须     | 对应 `users.userId`                   |
| companyName  | string | 必须     | 公司名称                              |
| email        | string | 非必须   | 邮箱（来自 `users.email`）            |
| phone        | string | 非必须   | 电话（来自 `users.phone`）            |

响应：

```json
{
  "code": 0,
  "data": {
    "advertiserId": 1999839285981343745,
    "userId": 1999839285981343745,
    "companyName": "上海理工大学",
    "email": "test@usst.edu.cn",
    "phone": "13800138000"
  },
  "message": "ok"
}
```



---

### 2.2 新增业主公司信息

#### 2.2.1 基本信息

- **请求路径**：`/advertisers/company-name`
- **请求方式**：`PUT`
- **接口描述**：广告主添加公司名称。

#### 2.2.2 请求参数

- **参数格式**：`application/json`

| 参数名      | 类型   | 是否必需 | 说明     |
| ----------- | ------ | -------- | -------- |
| companyName | string | 必须     | 公司名称 |

```json
{
  "companyName": "上海理工大学"
}
```

响应：

```json
{
  "code": 0,
  "data": true,
  "message": "ok"
}
```

---

### 2.3 获取支付方式列表

#### 2.3.1 基本信息

- **请求路径**：`/advertisers/payment-methods`
- **请求方式**：`GET`
- **接口描述**：根据当前广告主 ID 查询其所有支付方式，对应 `advertiser_payments`。

#### 2.3.2 请求参数

- 无。

#### 2.3.3 响应数据

`data` 为数组，每项结构：

| 参数名     | 类型   | 是否必需 | 说明                          |
| ---------- | ------ | -------- | ----------------------------- |
| paymentId  | number | 必须     | 支付信息 ID                   |
| cardNumber | string | 必须     | 银行卡号（可做脱敏处理返回）  |
| bankName   | string | 非必须   | 银行名称                      |

```json
{
  "code": 0,
  "data": [
    {
      "paymentId": 2000090668605206530,
      "cardNumber": "1234********3456",
      "bankName": "中国银行"
    }
  ],
  "message": "ok"
}
```



---

### 2.4 新增支付方式

#### 2.4.1 基本信息

- **请求路径**：`/advertisers/payment-methods`
- **请求方式**：`POST`
- **接口描述**：为当前广告业主新增支付方式，写入 `advertiser_payments`。

#### 2.4.2 请求参数

- **参数格式**：`application/json`

| 参数名     | 类型   | 是否必需 | 说明     |
| ---------- | ------ | -------- | -------- |
| cardNumber | string | 必须     | 银行卡号 |
| bankName   | string | 非必须   | 银行名称 |

```json
{
  "cardNumber": "1234567890123456",
  "bankName": "中国银行"
}
```

响应：

```json
{
  "code": 0,
  "data": {
    "paymentId": 2000090668605206530,
    "cardNumber": "1234********3456",
    "bankName": "中国银行"
  },
  "message": "ok"
}
```



---

### 2.5 删除支付方式（待定）

#### 2.5.1 基本信息

- **请求路径**：`/advertisers/payment-methods/{paymentMethodId}`
- **请求方式**：`DELETE`
- **接口描述**：删除指定支付方式记录。

#### 2.5.2 请求参数

- **路径参数**

| 参数名         | 类型   | 是否必需 | 说明        |
| -------------- | ------ | -------- | ----------- |
| paymentMethodId| number | 必须     | 支付信息 ID |

---

### 2.6 获取广告主广告列表

#### 2.6.1 基本信息

- **请求路径**：`/advertisers/ads`
- **请求方式**：`GET`
- **接口描述**：获取当前广告主名下的广告列表，可按审核状态筛选，数据来自 `advertisements`。

#### 2.6.2 请求参数

- **Query 参数**

| 参数名       | 类型   | 是否必需 | 说明                                         |
| ------------ | ------ | -------- | -------------------------------------------- |
| reviewStatus | number | 非必须   | 审核状态：0-待审核；1-通过；2-拒绝          |
| isActive     | number | 非必须   | 是否启用：0-否；1-是                         |
| page         | number | 非必须   | 页码，从 1 开始，默认 1                     |
| pageSize     | number | 非必须   | 每页数量，默认 10                           |

#### 2.6.3 响应数据

`data` 结构（分页）：

| 字段名   | 类型     | 说明       |
| -------- | -------- | ---------- |
| total    | number   | 总记录数   |
| page     | number   | 当前页码   |
| pageSize | number   | 每页数量   |
| records  | object[] | 广告列表   |

`records` 每项对应一条 `advertisements`：

| 字段名       | 类型   | 说明                                      |
| ------------ | ------ | ----------------------------------------- |
| adId         | number | 广告 ID                                   |
| title        | string | 广告标题                                  |
| adType       | number | 广告类型（0-image; 1-video）              |
| mediaUrl     | string | 素材路径                                  |
| landingPage  | string | 点击跳转地址                              |
| categoryId   | number | 广告类别 ID                               |
| adLayout     | string | 广告版式                                  |
| weeklyBudget | number | 周预算                                    |
| reviewStatus | number | 审核状态（0-待审核;1-通过;2-拒绝）        |
| isActive     | number | 是否启用投放（0-否；1-是）                |
| createTime   | string | 创建时间                                  |
| editTime     | string | 编辑时间                                  |

```json
{
  "code": 0,
  "data": {
    "records": [
      {
        "adId": 2000105075087269890,
        "title": "双11大促",
        "adType": 0,
        "mediaUrl": "https://cdn.example.com/uploads/banner-1.png",
        "landingPage": "https://shop.example.com/activity/1111",
        "categoryId": 1,
        "adLayout": "banner",
        "weeklyBudget": 5000,
        "reviewStatus": 0,
        "isActive": 0,
        "createTime": "2025-12-14T07:26:25.000+00:00",
        "editTime": "2025-12-14T13:32:37.000+00:00"
      },
      {
        "adId": 2000570446818955265,
        "title": "2024新款智能手机发布",
        "adType": 0,
        "mediaUrl": "https://cdn.example.com/tech/phone-new.jpg",
        "landingPage": "https://tech.example.com/products/phone15",
        "categoryId": 2,
        "adLayout": "banner",
        "weeklyBudget": 12000,
        "reviewStatus": 0,
        "isActive": 0,
        "createTime": "2025-12-15T14:15:38.000+00:00",
        "editTime": "2025-12-15T14:15:38.000+00:00"
      },
      {
        "adId": 2000570660875259906,
        "title": "低息消费贷，极速审批",
        "adType": 1,
        "mediaUrl": "https://cdn.example.com/finance/loan-video.mp4",
        "landingPage": "https://bank.example.com/loan/apply",
        "categoryId": 3,
        "adLayout": "sidebar",
        "weeklyBudget": 4500,
        "reviewStatus": 0,
        "isActive": 0,
        "createTime": "2025-12-15T14:16:29.000+00:00",
        "editTime": "2025-12-15T14:16:29.000+00:00"
      },
      {
        "adId": 2000570701702615041,
        "title": "Java后端架构师特训营",
        "adType": 0,
        "mediaUrl": "https://cdn.example.com/edu/java-course.png",
        "landingPage": "https://edu.example.com/course/java-arch",
        "categoryId": 4,
        "adLayout": "card",
        "weeklyBudget": 2000,
        "reviewStatus": 0,
        "isActive": 0,
        "createTime": "2025-12-15T14:16:39.000+00:00",
        "editTime": "2025-12-15T14:16:39.000+00:00"
      },
      {
        "adId": 2000570734757924865,
        "title": "进口美妆年终清仓",
        "adType": 0,
        "mediaUrl": "https://cdn.example.com/shop/cosmetics.jpg",
        "landingPage": "https://shop.example.com/category/beauty",
        "categoryId": 5,
        "adLayout": "card",
        "weeklyBudget": 8500,
        "reviewStatus": 0,
        "isActive": 0,
        "createTime": "2025-12-15T14:16:47.000+00:00",
        "editTime": "2025-12-15T14:16:47.000+00:00"
      },
      {
        "adId": 2000570778131222530,
        "title": "日本京都樱花季5日游",
        "adType": 1,
        "mediaUrl": "https://cdn.example.com/travel/kyoto-sakura.mp4",
        "landingPage": "https://travel.example.com/tours/jp-kyoto",
        "categoryId": 6,
        "adLayout": "banner",
        "weeklyBudget": 6000,
        "reviewStatus": 0,
        "isActive": 0,
        "createTime": "2025-12-15T14:16:57.000+00:00",
        "editTime": "2025-12-15T14:16:57.000+00:00"
      }
    ],
    "total": 6,
    "size": 10,
    "current": 1,
    "pages": 1
  },
  "message": "ok"
}
```



---

### 2.7 创建新广告

#### 2.7.1 基本信息

- **请求路径**：`/advertisers/ads`
- **请求方式**：`POST`
- **接口描述**：广告主创建广告记录，初始 `reviewStatus=0`（待审核），`isActive=0`。

#### 2.7.2 请求参数

- **参数格式**：`application/json`

| 参数名       | 类型   | 是否必需 | 说明                                        |
| ------------ | ------ | -------- | ------------------------------------------- |
| title        | string | 必须     | 广告标题                                    |
| adType       | number | 必须     | 广告类型：0-image；1-video                  |
| mediaUrl     | string | 必须     | 媒体 URL（通常来自 `/uploads` 返回）        |
| landingPage  | string | 非必须   | 点击跳转地址                                |
| categoryId   | number | 必须     | 广告类别 ID，对应 `ad_categories`          |
| adLayout     | string | 必须     | 广告版式：`banner` / `sidebar` / `card` 等 |
| weeklyBudget | number | 必须     | 周预算，映射到 `DECIMAL(10,2)`             |

> `advertiserId` 从当前登录用户解析，不在请求体中传递。

请求示例：

```json
{
  "title": "双11大促",
  "adType": 0,
  "mediaUrl": "https://cdn.example.com/uploads/banner-1.png",
  "landingPage": "https://shop.example.com/activity/1111",
  "categoryId": 1,
  "adLayout": "banner",
  "weeklyBudget": 5000.00
}
```

响应：

```json
{
  "code": 0,
  "data": {
    "adId": 2000105075087269890,
    "title": "双11大促",
    "adType": 0,
    "mediaUrl": "https://cdn.example.com/uploads/banner-1.png",
    "landingPage": "https://shop.example.com/activity/1111",
    "categoryId": 1,
    "adLayout": "banner",
    "weeklyBudget": 5000,
    "reviewStatus": 0,
    "isActive": 0,
    "createTime": "2025-12-14T07:26:24.720+00:00",
    "editTime": "2025-12-14T07:26:24.720+00:00"
  },
  "message": "ok"
}
```

#### 2.7.3 响应数据

`data` 为新建广告的完整信息（同上 `records` 结构），其中：

- `reviewStatus` 固定返回 `0`
- `isActive` 固定返回 `0`

---

### 2.8 获取单个广告详情

#### 2.8.1 基本信息

- **请求路径**：`/advertisers/ads/{adId}`
- **请求方式**：`GET`
- **接口描述**：获取当前广告主某一条广告的完整详情。

#### 2.8.2 请求参数

- **路径参数**

| 参数名 | 类型   | 是否必需 | 说明   |
| ------ | ------ | -------- | ------ |
| adId   | number | 必须     | 广告 ID |

#### 2.8.3 响应数据

- `data` 为一条广告对象，字段同 2.6 中 `records`。

```json
{
  "code": 0,
  "data": {
    "adId": 2000105075087269890,
    "title": "双11大促",
    "adType": 0,
    "mediaUrl": "https://cdn.example.com/uploads/banner-1.png",
    "landingPage": "https://shop.example.com/activity/1111",
    "categoryId": 1,
    "adLayout": "banner",
    "weeklyBudget": 5000,
    "reviewStatus": 0,
    "isActive": 0,
    "createTime": "2025-12-14T07:26:25.000+00:00",
    "editTime": "2025-12-14T07:26:25.000+00:00"
  },
  "message": "ok"
}
```



---

### 2.9 修改广告内容

#### 2.9.1 基本信息

- **请求路径**：`/advertisers/ads/{adId}`
- **请求方式**：`PUT`
- **接口描述**：修改广告内容；通常需要将 `reviewStatus` 重新置为 `0`（待审核），具体逻辑由后端控制。

#### 2.9.2 请求参数

- **参数格式**：`application/json`
- 任意字段可选，至少一个：`title, adType, mediaUrl, landingPage, categoryId, adLayout, weeklyBudget`。

```json
{
  "title": "双12大促",
  "adType": 0,
  "mediaUrl": "https://cdn.example.com/uploads/banner-1.png",
  "landingPage": "https://shop.example.com/activity/1111",
  "categoryId": 1,
  "adLayout": "banner",
  "weeklyBudget": 5000.00
}
```



#### 2.9.3 响应数据

- `data`：更新后的广告详情。

```json
{
  "code": 0,
  "data": {
    "adId": 2000105075087269890,
    "title": "双12大促",
    "adType": 0,
    "mediaUrl": "https://cdn.example.com/uploads/banner-1.png",
    "landingPage": "https://shop.example.com/activity/1111",
    "categoryId": 1,
    "adLayout": "banner",
    "weeklyBudget": 5000,
    "reviewStatus": 0,
    "isActive": 0,
    "createTime": "2025-12-14T07:26:25.000+00:00",
    "editTime": "2025-12-15T14:43:21.628+00:00"
  },
  "message": "ok"
}
```



---

### 2.10 删除广告（逻辑删除）

#### 2.10.1 基本信息

- **请求路径**：`/advertisers/ads/{adId}`
- **请求方式**：`DELETE`
- **接口描述**：逻辑删除广告，可实现为“将 `isActive` 置为 0，并禁止后续投放”，不物理删除记录。

#### 2.10.2 请求参数

- **路径参数**

| 参数名 | 类型   | 是否必需 | 说明   |
| ------ | ------ | -------- | ------ |
| adId   | number | 必须     | 广告 ID |

#### 2.10.3 响应数据

- `data` 为空或返回删除结果标记。

---

### 2.11 切换广告投放状态（开启 / 暂停）

#### 2.11.1 基本信息

- **请求路径**：`/advertisers/ads/{adId}/status`
- **请求方式**：`PUT`
- **接口描述**：修改广告的 `isActive` 字段，前提是 `reviewStatus=1`（已通过）。

#### 2.11.2 请求参数

- **参数格式**：`application/json`

| 参数名  | 类型   | 是否必需 | 说明             |
| ------- | ------ | -------- | ---------------- |
| isActive| number | 必须     | 0-暂停；1-开启   |

```json
{
  "isActive": 1
}
```



#### 2.11.3 响应数据

- `data`：更新后的广告详情。

```json
{
  "code": 0,
  "data": {
    "adId": 2000105075087269890,
    "title": "双12大促",
    "adType": 0,
    "mediaUrl": "https://cdn.example.com/uploads/banner-1.png",
    "landingPage": "https://shop.example.com/activity/1111",
    "categoryId": 1,
    "adLayout": "banner",
    "weeklyBudget": 5000,
    "reviewStatus": 1,
    "isActive": 1,
    "createTime": "2025-12-14T07:26:25.000+00:00",
    "editTime": "2025-12-15T14:49:08.144+00:00"
  },
  "message": "ok"
}
```



---

### 2.12 广告主数据概览（未完成，考虑收益计算方案）

#### 2.12.1 基本信息

- **请求路径**：`/advertisers/statistics/summary`
- **请求方式**：`GET`
- **接口描述**：基于 `ad_displays` 等表，返回当前广告主账户总览（总展示数、总点击数等）。

#### 2.12.2 请求参数

- **Query 参数（可选）**

| 参数名    | 类型   | 说明                    |
| --------- | ------ | ----------------------- |
| startDate | string | 开始日期 `yyyy-MM-dd`  |
| endDate   | string | 结束日期 `yyyy-MM-dd`  |

#### 2.12.3 响应数据

`data` 字段结构示例：

```json
{
  "code": 0,
  "data": {
    "totalImpressions": 0,
    "totalClicks": 0,
    "ctr": 0,
    "totalSpend": 0
  },
  "message": "ok"
}
```

---

### 2.13 单条广告统计（未完成，设计收益方案）

#### 2.13.1 基本信息

- **请求路径**：`/advertisers/ads/{adId}/statistics`
- **请求方式**：`GET`
- **接口描述**：获取指定广告的详细统计数据（可按天汇总）。

#### 2.13.2 请求参数

- **路径参数**

| 参数名 | 类型   | 是否必需 | 说明   |
| ------ | ------ | -------- | ------ |
| adId   | number | 必须     | 广告 ID |

- **Query 参数（可选）**

| 参数名    | 类型   | 说明                    |
| --------- | ------ | ----------------------- |
| startDate | string | 开始日期 `yyyy-MM-dd`  |
| endDate   | string | 结束日期 `yyyy-MM-dd`  |

#### 2.13.3 响应数据

```json
{
  "adId": 1,
  "totalImpressions": 1000,
  "totalClicks": 123,
  "ctr": 0.123,
  "daily": [
    { "date": "2025-01-01", "impressions": 100, "clicks": 10 },
    { "date": "2025-01-02", "impressions": 120, "clicks": 15 }
  ]
}
```

---

## 3. 网站站长模块 (Publisher)

### 3.1 获取我的网站列表

#### 3.1.1 基本信息

- **请求路径**：`/publishers/sites`
- **请求方式**：`GET`
- **接口描述**：站长查看自己名下的网站列表，对应 `publishers` 表。

#### 3.1.2 请求参数

- 无（从 Token 中获取 `publisherId`）。

#### 3.1.3 响应数据

`data` 为数组，每项字段：

| 字段名            | 类型   | 说明                                    |
| ----------------- | ------ | --------------------------------------- |
| websiteId         | number | 网站 ID                                 |
| websiteName       | string | 网站名称                                |
| domain            | string | 网站地址 / 域名                         |
| isVerified        | number | 是否已通过验证（0-未通过；1-通过）      |
| createTime        | string | 创建时间                                |
| verifyTime        | string | 验证时间，未验证则为 `null`             |
| verificationToken | string | 验证代码（是否返回由后端安全策略决定）  |

```json
{
  "code": 0,
  "data": [
    {
      "websiteId": 2000592343371620354,
      "websiteName": "测试",
      "domain": "ahaha.com",
      "isVerified": 0,
      "createTime": "2025-12-15T15:42:39.000+00:00",
      "verifyTime": null,
      "verificationToken": "verify-0f6e1cd5fea588cda8bd782ef7b65099"
    }
  ],
  "message": "ok"
}
```



---

### 3.2 提交新网站信息

#### 3.2.1 基本信息

- **请求路径**：`/publishers/sites`
- **请求方式**：`POST`
- **接口描述**：添加新网站，创建 `publishers` 记录，生成 `verificationToken` 用于站长验证。

#### 3.2.2 请求参数

- **参数格式**：`application/json`

| 字段名      | 类型   | 是否必需 | 说明     |
| ----------- | ------ | -------- | -------- |
| websiteName | string | 必须     | 网站名称 |
| domain      | string | 必须     | 网站域名 |

```json
{
  "domain": "ahaha.com",
  "websiteName": "测试"
}
```



#### 3.2.3 响应数据

`data` 字段：

| 字段名            | 类型   | 说明         |
| ----------------- | ------ | ------------ |
| websiteId         | number | 新增网站 ID  |
| websiteName       | string | 网站名称     |
| domain            | string | 网站域名     |
| verificationToken | string | 验证代码     |
| isVerified        | number | 初始为 0     |

```json
{
  "code": 0,
  "data": {
    "websiteId": 2000592343371620354,
    "websiteName": "测试",
    "domain": "ahaha.com",
    "isVerified": 0,
    "createTime": "2025-12-15T15:42:38.529+00:00",
    "verifyTime": null,
    "verificationToken": "verify-0f6e1cd5fea588cda8bd782ef7b65099"
  },
  "message": "ok"
}
```



---

### 3.3 获取网站验证 Token

#### 3.3.1 基本信息

- **请求路径**：`/publishers/sites/{siteId}/verification-token`
- **请求方式**：`GET`
- **接口描述**：站长查询指定网站的验证 Token，用于前端展示“把这段代码放到你的网站上”。

#### 3.3.2 请求参数

- **路径参数**

| 字段名 | 类型   | 是否必需 | 说明                  |
| ------ | ------ | -------- | --------------------- |
| siteId | number | 必须     | 网站 ID（`websiteId`） |

#### 3.3.3 响应数据

`data` 字段：

| 字段名            | 类型   | 说明     |
| ----------------- | ------ | -------- |
| websiteId         | number | 网站 ID  |
| verificationToken | string | 验证代码 |

```json
{
  "code": 0,
  "data": "verify-0f6e1cd5fea588cda8bd782ef7b65099",
  "message": "ok"
}
```



---

### 3.4 触发网站所有权验证

#### 3.4.1 基本信息

- **请求路径**：`/publishers/sites/{siteId}/verification`
- **请求方式**：`POST`
- **接口描述**：后端根据 `domain` 和 `verificationToken` 去访问网站，检测是否正确放置验证代码，成功则更新 `isVerified` 和 `verifyTime`。

#### 3.4.2 请求参数

- **路径参数**

| 字段名 | 类型   | 是否必需 | 说明     |
| ------ | ------ | -------- | -------- |
| siteId | number | 必须     | 网站 ID  |

- **请求体**：无。

#### 3.4.3 响应数据

`data` 字段：

| 字段名     | 类型   | 说明                 |
| ---------- | ------ | -------------------- |
| websiteId  | number | 网站 ID              |
| isVerified | number | 验证结果：0 或 1     |
| verifyTime | string | 验证时间（成功时返回） |

---

### 3.5 获取广告位列表

#### 3.5.1 基本信息

- **请求路径**：`/publishers/ad-slots`
- **请求方式**：`GET`
- **接口描述**：获取当前站长名下所有网站的广告位列表，对应 `ad_placements` 表。

#### 3.5.2 请求参数

- **Query 参数（可选）**

| 字段名    | 类型   | 说明            |
| --------- | ------ | --------------- |
| websiteId | number | 按网站筛选广告位 |

#### 3.5.3 响应数据

`data` 为数组，每项字段：

| 字段名        | 类型   | 说明                          |
| ------------- | ------ | ----------------------------- |
| placementId   | number | 广告位 ID                     |
| websiteId     | number | 网站 ID                       |
| placementName | string | 广告位名称                    |
| adLayout      | string | 广告版式（banner/sidebar 等） |
| createTime    | string | 创建时间                      |

---

### 3.6 创建广告位

#### 3.6.1 基本信息

- **请求路径**：`/publishers/ad-slots`
- **请求方式**：`POST`
- **接口描述**：在指定网站下创建一个广告位，绑定 `websiteId` 与版式 `adLayout`，写入 `ad_placements`。

#### 3.6.2 请求参数

- **参数格式**：`application/json`

| 字段名        | 类型   | 是否必需 | 说明                                  |
| ------------- | ------ | -------- | ------------------------------------- |
| websiteId     | number | 必须     | 网站 ID，必须是当前站长名下的网站     |
| placementName | string | 必须     | 广告位名称，如“首页顶部横幅”         |
| adLayout      | string | 必须     | 广告版式：`banner` / `sidebar` / ... |

#### 3.6.3 响应数据

- 新建广告位的全部字段（同 3.5 中列表）。

---

### 3.7 获取广告位详情

#### 3.7.1 基本信息

- **请求路径**：`/publishers/ad-slots/{adSlotId}`
- **请求方式**：`GET`
- **接口描述**：获取单个广告位详情。

#### 3.7.2 请求参数

- **路径参数**

| 字段名   | 类型   | 是否必需 | 说明                     |
| -------- | ------ | -------- | ------------------------ |
| adSlotId | number | 必须     | 广告位 ID（`placementId`） |

#### 3.7.3 响应数据

- 字段同广告位列表单项。

---

### 3.8 修改广告位名称

#### 3.8.1 基本信息

- **请求路径**：`/publishers/ad-slots/{adSlotId}`
- **请求方式**：`PUT`
- **接口描述**：修改广告位的 `placementName`。

#### 3.8.2 请求参数

- **参数格式**：`application/json`

| 字段名        | 类型   | 是否必需 | 说明   |
| ------------- | ------ | -------- | ------ |
| placementName | string | 必须     | 新名称 |

---

### 3.9 获取广告位集成代码

#### 3.9.1 基本信息

- **请求路径**：`/publishers/ad-slots/{adSlotId}/integration-code`
- **请求方式**：`GET`
- **接口描述**：返回给站长的一段 JS 代码，前端直接复制到页面即可加载 `/public/ads/serve`。

#### 3.9.2 请求参数

- **路径参数**

| 字段名   | 类型   | 是否必需 | 说明     |
| -------- | ------ | -------- | -------- |
| adSlotId | number | 必须     | 广告位 ID |

#### 3.9.3 响应数据

`data` 字段：

| 字段名         | 类型   | 说明                      |
| -------------- | ------ | ------------------------- |
| placementId    | number | 广告位 ID                 |
| scriptTemplate | string | 嵌入页面的 JS 代码字符串  |

---

### 3.10 收益统计（待定）

#### 3.10.1 基本信息

- **请求路径**：`/publishers/statistics`
- **请求方式**：`GET`
- **接口描述**：基于 `ad_displays`、`content_visit_stats` 统计站长流量与预估收益。

#### 3.10.2 请求参数

- 可选：`startDate`, `endDate`（字符串，`yyyy-MM-dd`），统计时间区间。

#### 3.10.3 响应数据

```json
{
  "totalImpressions": 3000,
  "totalClicks": 200,
  "estimatedRevenue": 123.45
}
```

---

## 4. 管理员模块 (Admin)

### 4.1 获取待审核广告列表

#### 4.1.1 基本信息

- **请求路径**：`/admin/ads`
- **请求方式**：`GET`
- **接口描述**：管理员查看广告列表，可按 `status`（即 `reviewStatus`）筛选，默认查询待审核广告。

#### 4.1.2 请求参数

- **Query 参数**

| 字段名   | 类型   | 是否必需 | 说明                          |
| -------- | ------ | -------- | ----------------------------- |
| status   | number | 非必须   | 审核状态，默认为 0（待审核） |
| page     | number | 非必须   | 页码                          |
| pageSize | number | 非必须   | 每页数量                      |

#### 4.1.3 响应数据

- 分页结构同 2.6。

```json
{
  "code": 0,
  "data": {
    "records": [
      {
        "adId": 2000105075087269890,
        "title": "双12大促",
        "adType": 0,
        "mediaUrl": "https://cdn.example.com/uploads/banner-1.png",
        "landingPage": "https://shop.example.com/activity/1111",
        "categoryId": 1,
        "adLayout": "banner",
        "weeklyBudget": 5000,
        "reviewStatus": 1,
        "isActive": 1,
        "createTime": "2025-12-14T07:26:25.000+00:00",
        "editTime": "2025-12-15T14:49:08.000+00:00"
      }
    ],
    "total": 1,
    "size": 10,
    "current": 1,
    "pages": 1
  },
  "message": "ok"
}
```



---

### 4.2 提交广告审核结果

#### 4.2.1 基本信息

- **请求路径**：`/admin/ads/{adId}/review`
- **请求方式**：`PUT`
- **接口描述**：管理员对广告进行审核，通过 / 拒绝，修改 `reviewStatus` 字段。

#### 4.2.2 请求参数

- **路径参数**

| 字段名 | 类型   | 是否必需 | 说明   |
| ------ | ------ | -------- | ------ |
| adId   | number | 必须     | 广告 ID |

- **请求体**

| 字段名      | 类型   | 是否必需 | 说明                       |
| ----------- | ------ | -------- | -------------------------- |
| reviewStatus| number | 必须     | 审核结果：1-通过；2-拒绝   |
| reason      | string | 非必须   | 拒绝原因，审核通过时可为空 |

```json
{
  "reason": "",
  "reviewStatus": 1
}
```



#### 4.2.3 响应数据

- `data`：更新后的广告详情。

```json
{
  "code": 0,
  "data": {
    "adId": 2000105075087269890,
    "title": "双12大促",
    "adType": 0,
    "mediaUrl": "https://cdn.example.com/uploads/banner-1.png",
    "landingPage": "https://shop.example.com/activity/1111",
    "categoryId": 1,
    "adLayout": "banner",
    "weeklyBudget": 5000,
    "reviewStatus": 1,
    "isActive": 0,
    "createTime": "2025-12-14T07:26:25.000+00:00",
    "editTime": "2025-12-15T14:48:57.688+00:00"
  },
  "message": "ok"
}
```



---

### 4.3 获取广告分类列表

#### 4.3.1 基本信息

- **请求路径**：`/admin/categories`
- **请求方式**：`GET`
- **接口描述**：查询 `ad_categories` 表全部分类。

#### 4.3.2 请求参数

- 无。

#### 4.3.3 响应数据

`data` 为数组：

| 字段名      | 类型   | 说明        |
| ----------- | ------ | ----------- |
| categoryId  | number | 广告类别 ID |
| categoryName| string | 类别名称    |
| createTime  | string | 创建时间    |

```json
{
  "code": 0,
  "data": [
    {
      "categoryId": 1,
      "categoryName": "商品",
      "createTime": "2025-12-14T07:25:55.000+00:00"
    },
    {
      "categoryId": 2,
      "categoryName": "科技",
      "createTime": "2025-12-15T13:51:23.000+00:00"
    },
    {
      "categoryId": 3,
      "categoryName": "金融",
      "createTime": "2025-12-15T13:51:23.000+00:00"
    },
    {
      "categoryId": 4,
      "categoryName": "教育",
      "createTime": "2025-12-15T13:51:23.000+00:00"
    },
    {
      "categoryId": 5,
      "categoryName": "电商",
      "createTime": "2025-12-15T13:51:23.000+00:00"
    },
    {
      "categoryId": 6,
      "categoryName": "旅游",
      "createTime": "2025-12-15T13:51:23.000+00:00"
    },
    {
      "categoryId": 2000104592897499138,
      "categoryName": "测试",
      "createTime": "2025-12-14T07:24:30.000+00:00"
    }
  ],
  "message": "ok"
}
```



---

### 4.4 新增广告分类

#### 4.4.1 基本信息

- **请求路径**：`/admin/categories`
- **请求方式**：`POST`
- **接口描述**：管理员新增广告分类，写入 `ad_categories`。

#### 4.4.2 请求参数

- **参数格式**：`application/json`

| 字段名      | 类型   | 是否必需 | 说明     |
| ----------- | ------ | -------- | -------- |
| categoryName| string | 必须     | 类别名称 |

```json
{
  "categoryName": "测试"
}
```

响应：

```json
{
  "code": 0,
  "data": {
    "categoryId": 2000104592897499138,
    "categoryName": "测试",
    "createTime": "2025-12-14T07:24:29.649+00:00"
  },
  "message": "ok"
}
```



#### 4.4.3 响应数据

- 新增分类对象，同 4.3 中字段。

---

### 4.5 获取用户列表

#### 4.5.1 基本信息

- **请求路径**：`/admin/users`
- **请求方式**：`GET`
- **接口描述**：管理员查看系统用户列表，可按角色过滤。

#### 4.5.2 请求参数

- **Query 参数（可选）**

| 字段名 | 类型   | 是否必需 | 说明                                   |
| ------ | ------ | -------- | -------------------------------------- |
| role   | string | 非必须   | 角色过滤：`admin/advertiser/publisher` |

#### 4.5.3 响应数据

`data` 为数组：

| 字段名    | 类型   | 说明     |
| --------- | ------ | -------- |
| userId    | number | 用户 ID  |
| username  | string | 用户名   |
| userRole  | string | 角色     |
| email     | string | 邮箱     |
| phone     | string | 电话     |
| createTime| string | 创建时间 |

```json
{
  "code": 0,
  "data": [
    {
      "userId": 1994253936127193090,
      "username": "qiao",
      "userPassword": "$2a$10$0qZJpQkPbexTSmul4Jwgh.aGqCbeKvnOO8/f7NjKWxW2ze8d2MSUq",
      "userRole": "admin",
      "email": "test@usst.edu.cn",
      "phone": "13800138000",
      "createTime": "2025-11-28T03:56:01.000+00:00"
    },
    {
      "userId": 1994653213232058370,
      "username": "qiao1",
      "userPassword": "$2a$10$0qZJpQkPbexTSmul4Jwgh.aGqCbeKvnOO8/f7NjKWxW2ze8d2MSUq",
      "userRole": "admin",
      "email": "test@usst.edu.cn",
      "phone": "13800138000",
      "createTime": "2025-11-29T06:22:35.000+00:00"
    },
    {
      "userId": 1994664809538813954,
      "username": "qiao3",
      "userPassword": "$2a$10$0qZJpQkPbexTSmul4Jwgh.aGqCbeKvnOO8/f7NjKWxW2ze8d2MSUq",
      "userRole": "admin",
      "email": "test@usst.edu.cn",
      "phone": "13800138000",
      "createTime": "2025-11-29T07:08:40.000+00:00"
    },
    {
      "userId": 1994681290718330881,
      "username": "BobLiu",
      "userPassword": "$2a$10$gGGN1ayxd85gsugMmShhIuC3.nbRAefCtdY2B.WEExwywpSs2W7FS",
      "userRole": "admin",
      "email": "bobliu@bobliu.tech",
      "phone": "15618381320",
      "createTime": "2025-11-29T08:14:13.000+00:00"
    },
    {
      "userId": 1994694844469104642,
      "username": "ad001",
      "userPassword": "$2a$10$rNjr.jrz6VjJZuzVW4u.U.YLZ86B34Onuv0QnJ5RWVjmLkXDYAYUe",
      "userRole": "admin",
      "email": "ad001@ad001.com",
      "phone": "18888888888",
      "createTime": "2025-11-29T09:08:05.000+00:00"
    },
    {
      "userId": 2000589616625229826,
      "username": "fafafa",
      "userPassword": "$2a$10$zET33j6URUJP1zcG/w6pG.5Phkk7r2fNIpGgk0XI8DlxSKQU13oRW",
      "userRole": "admin",
      "email": "fafafa@qq.com",
      "phone": "123456789",
      "createTime": "2025-12-15T15:31:44.000+00:00"
    },
    {
      "userId": 2000590269590282242,
      "username": "fafafas",
      "userPassword": "$2a$10$uExFFypsZrnjKwvqwGmPbuqOn/CG5gGev8rLNEDsBCc3PzT7hDvUi",
      "userRole": "admin",
      "email": "fafafa@qq.com",
      "phone": "123456789",
      "createTime": "2025-12-15T15:34:20.000+00:00"
    },
    {
      "userId": 2000590909271973889,
      "username": "fafafas0",
      "userPassword": "$2a$10$tuEvP08z/vcMpIDChAxMDumnuKJUGJ8jFlB9uAraVZcuGpoQtVcC6",
      "userRole": "admin",
      "email": "fafafa@qq.com",
      "phone": "123456789",
      "createTime": "2025-12-15T15:36:45.000+00:00"
    }
  ],
  "message": "ok"
}
```



---

### 4.6 创建管理员账号

#### 4.6.1 基本信息

- **请求路径**：`/admin/users`
- **请求方式**：`POST`
- **接口描述**：创建新管理员账号，对应往 `users` 表插入一条 `userRole='admin'` 记录。

#### 4.6.2 请求参数

- **参数格式**：`application/json`

| 字段名   | 类型   | 是否必需 | 说明     |
| -------- | ------ | -------- | -------- |
| username | string | 必须     | 用户名   |
| password | string | 必须     | 登录密码 |
| email    | string | 非必须   | 邮箱     |
| phone    | string | 非必须   | 电话     |

#### 4.6.3 响应数据

- 新建管理员的用户信息（同 4.5 单项）。

```json
{
  "code": 0,
  "data": {
    "userId": 2000590909271973889,
    "username": "fafafas0",
    "userPassword": "$2a$10$tuEvP08z/vcMpIDChAxMDumnuKJUGJ8jFlB9uAraVZcuGpoQtVcC6",
    "userRole": "admin",
    "email": "fafafa@qq.com",
    "phone": "123456789",
    "createTime": "2025-12-15T15:36:45.220+00:00"
  },
  "message": "ok"
}
```

---

### 4.7 删除管理员账号 / 移除权限（待定，没写）

#### 4.7.1 基本信息

- **请求路径**：`/admin/users/{userId}`
- **请求方式**：`DELETE`
- **接口描述**：删除指定管理员账号，或将其 `userRole` 改为其他角色。

#### 4.7.2 请求参数

- **路径参数**

| 字段名 | 类型   | 是否必需 | 说明   |
| ------ | ------ | -------- | ------ |
| userId | number | 必须     | 用户 ID |

---

## 5. 公共广告引擎 (Public Ad Engine)

> 这一部分主要给“内容网站前端 JS”调用，不需要登录，但要依赖 `placementId`、`trackId` 等参数，写数据到 `ad_displays` 和 `content_visit_stats`。

### 5.1 请求广告（核心投放）

#### 5.1.1 基本信息

- **请求路径**：`/public/ads/serve`
- **请求方式**：`GET`
- **接口描述**：根据 `placementId`、`trackId` 等信息，在已审核通过的广告中随机或简单选择一条返回。后台同时在 `ad_displays` 中创建一条展示记录，生成 `displayId`，用于后续上报展示时长和点击。

#### 5.1.2 请求参数

- **Query 参数**

| 字段名      | 类型   | 是否必需 | 说明                                           |
| ----------- | ------ | -------- | ---------------------------------------------- |
| placementId | number | 必须     | 广告位 ID，对应 `ad_placements.placementId`   |
| trackId     | string | 必须     | 匿名用户标识，通常为 UUID（`char(36)`）       |
| categoryId  | number | 非必须   | 页面内容所属分类 ID（可用于简单定向）         |
| url         | string | 非必须   | 当前页面 URL，便于日志记录                    |

#### 5.1.3 响应数据

`data` 字段结构：

| 字段名      | 类型   | 说明                                          |
| ----------- | ------ | --------------------------------------------- |
| displayId   | number | 展示记录 ID，对应 `ad_displays.displayId`    |
| adId        | number | 被投放的广告 ID                              |
| adType      | number | 广告类型（0-image;1-video）                  |
| mediaUrl    | string | 媒体 URL                                     |
| title       | string | 广告标题                                     |
| landingPage | string | 点击跳转地址                                 |
| categoryId  | number | 广告类别 ID                                  |
| adLayout    | string | 广告版式                                     |
| websiteId   | number | 广告位所属网站 ID（来自 `ad_placements`）    |
| placementId | number | 广告位 ID                                    |

响应示例：

```json
{
  "code": 1,
  "msg": "success",
  "data": {
    "displayId": 50001,
    "adId": 3,
    "adType": 0,
    "mediaUrl": "https://cdn.example.com/banner-3.png",
    "title": "春季上新",
    "landingPage": "https://shop.example.com/spring",
    "categoryId": 1,
    "adLayout": "banner",
    "websiteId": 10,
    "placementId": 1001
  }
}
```

---

### 5.2 上报广告展示数据

#### 5.2.1 基本信息

- **请求路径**：`/public/ads/impressions`
- **请求方式**：`POST`
- **接口描述**：内容页前端在广告展示结束或页面卸载时调用，用于更新 `ad_displays` 中的停留时长 `duration` 等信息。

#### 5.2.2 请求参数

- **参数格式**：`application/json`

| 字段名    | 类型   | 是否必需 | 说明                                           |
| --------- | ------ | -------- | ---------------------------------------------- |
| displayId | number | 必须     | 展示记录 ID，来自 `/public/ads/serve` 返回    |
| duration  | number | 必须     | 停留时长（秒），对应 `ad_displays.duration`   |

请求示例：

```json
{
  "displayId": 50001,
  "duration": 8
}
```

#### 5.2.3 响应数据

- 成功时 `data` 可为空。

---

### 5.3 上报广告点击数据

#### 5.3.1 基本信息

- **请求路径**：`/public/ads/clicks`
- **请求方式**：`POST`
- **接口描述**：用户点击广告时调用，更新 `ad_displays` 的 `clicked` 和 `clickTime` 字段。

#### 5.3.2 请求参数

- **参数格式**：`application/json`

| 字段名    | 类型   | 是否必需 | 说明                                      |
| --------- | ------ | -------- | ----------------------------------------- |
| displayId | number | 必须     | 展示记录 ID                               |
| trackId   | string | 非必须   | 匿名用户 ID，用于校验和统计（可选）      |

#### 5.3.3 响应数据

- 成功时 `data` 可为空。

---

### 5.4 上报内容访问行为（用户画像）

#### 5.4.1 基本信息

- **请求路径**：`/public/user-behaviors`
- **请求方式**：`POST`
- **接口描述**：前端在用户浏览内容页时上报其访问行为，用于在 `content_visit_stats` 中记录匿名用户在某类内容下的停留时长。

#### 5.4.2 请求参数

- **参数格式**：`application/json`

| 字段名    | 类型   | 是否必需 | 说明                                                   |
| --------- | ------ | -------- | ------------------------------------------------------ |
| websiteId | number | 必须     | 网站 ID，对应 `publishers.websiteId`                  |
| categoryId| number | 必须     | 内容所属分类 ID（可与广告分类共用表）                 |
| trackId   | string | 必须     | 匿名用户标识 `char(36)`                               |
| duration  | number | 必须     | 停留时长（秒），映射到 `content_visit_stats.duration` |
| timestamp | string | 非必须   | 访问时间，默认服务器当前时间                          |

请求示例：

```json
{
  "websiteId": 10,
  "categoryId": 1,
  "trackId": "550e8400-e29b-41d4-a716-446655440000",
  "duration": 30
}
```

#### 5.4.3 响应数据

- 成功时 `data` 可为空，或返回新建记录的 `visitId`。
