
## 基础服务与认证 (Auth & Common)

POST /auth/register # 用户注册（广告主/站长）
POST /auth/login # 用户登录，获取 Token
POST /auth/logout # 退出登录
GET /users/me # 获取当前登录用户的详细信息
PUT /users/me # 修改个人信息（邮箱、电话）（待定）
POST /uploads # 通用文件上传（图片/视频），返回 URL

---

## 广告业主模块 (Advertiser)

### 资料与支付

GET /advertisers/profile # 获取业主公司信息
PUT /advertisers/profile # 修改业主公司名称信息（待定）
GET /advertisers/payment-methods # 获取已绑定的支付方式列表
POST /advertisers/payment-methods # 添加新的银行卡/支付方式
DELETE /advertisers/payment-methods/{paymentMethodId} # 删除某个支付方式（待定）

### 广告管理

GET /advertisers/ads # 获取我的广告列表（支持按状态筛选）
POST /advertisers/ads # 创建新广告（初始状态：待审核）
GET /advertisers/ads/{adId} # 获取单个广告详情
PUT /advertisers/ads/{adId} # 修改广告内容（修改后可能需重审）
DELETE /advertisers/ads/{adId} # 删除广告（逻辑删除）
PUT /advertisers/ads/{adId}/status # 切换广告投放状态（开启/暂停）

### 数据统计

GET /advertisers/statistics/summary # 获取账户级广告数据概览（总花费、总展示）
GET /advertisers/ads/{adId}/statistics # 获取指定广告的详细报表

---

## 网站站长模块 (Publisher)

### 网站接入

GET /publishers/sites # 获取我的网站列表
POST /publishers/sites # 提交新网站信息
GET /publishers/sites/{siteId}/verification-token # 获取验证所需的 Token 字符串
POST /publishers/sites/{siteId}/verification # 触发系统去验证网站所有权

### 广告位管理

GET /publishers/ad-slots # 获取我的广告位列表
POST /publishers/ad-slots # 创建广告位（指定版式 Layout）
GET /publishers/ad-slots/{adSlotId} # 获取广告位详情
PUT /publishers/ad-slots/{adSlotId} # 修改广告位名称
GET /publishers/ad-slots/{adSlotId}/integration-code # 获取该广告位的集成 SDK 代码

### 收益统计

GET /publishers/statistics # 获取流量与预估收益统计（待定）

---

## 管理员模块 (Admin)

### 广告审核

GET /admin/ads?status=pending # 获取所有待审核的广告
PUT /admin/ads/{adId}/review # 提交审核结果（通过/拒绝）

### 分类与配置

GET /admin/categories # 获取所有广告分类列表
POST /admin/categories # 添加广告分类标签（如：数码、美妆）

### 用户管理

GET /admin/users # 获取系统用户列表
GET /admin/users?role=admin # 获取系统内所有管理员列表（用于查看谁拥有管理权限）（待定）
POST /admin/users # 创建一个新的管理员账号（直接设置用户名、密码、角色为admin）
DELETE /admin/users/{userId} # 删除指定管理员账号（或移除其管理员权限）（待定）

---

## 公共广告引擎 (Public Ad Engine)

### 核心投放

GET /public/ads/serve # [核心] 请求广告（根据 PlacementId + TrackId + Context）
POST /public/ads/impressions # [核心] 上报广告展示数据
POST /public/ads/clicks # [核心] 上报广告点击数据

### 用户画像积累

POST /public/user-behaviors # 上报内容页访问数据（记录 trackId 在哪个分类停留了多久）

---

## 内容网站

### 新闻网站内容接口 (News Site API)

GET /news/categories # 获取新闻分类列表
GET /news/articles # 获取新闻列表（支持按分类筛选）
GET /news/articles/{articleId} # 获取单篇新闻详情

### 购物网站内容接口 (Shop Site API)

GET /shop/categories # 获取商品分类列表
GET /shop/products # 获取商品列表（支持按分类筛选）
GET /shop/products/{productId} # 获取商品详情

### 视频网站内容接口 (Video Site API)

GET /videos/categories # 获取视频分类列表
GET /videos # 获取视频列表
GET /videos/{videoId} # 获取视频详情与播放地址
