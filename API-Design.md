# 广告系统 API 接口设计文档 (API Design)

## 一、 接口设计基础规范 (Base Info)

* **基础路径 (Base URL)**：`http://{server_ip}:8080/api/v1`
* **通信协议**：HTTP/1.1
* **数据格式**：JSON
* **字符编码**：UTF-8
* **跨域支持 (CORS)**：公共投放接口必须允许所有域名访问 (`Access-Control-Allow-Origin: *`)。

### 统一响应结构 (Response Wrapper)

所有接口（无论成功失败）均返回以下结构：

```json
{
  "code": 200,
  "message": "success",
  "data": { }
}
```

---

## 二、 公共投放 API (Public Ad Serving APIs)

**使用者**：新闻网、购物网、视频网的前端 JS SDK。
**鉴权方式**：无需登录，通过 `placement_id`识别来源。

### 1. 获取广告 (Fetch Ad)

核心接口。根据广告位配置和用户画像，返回一个最佳匹配的广告。

* **URL**: GET /ad/serve
* **描述**: 获取广告素材。
* **请求参数 (Query Params)**:

| **参数名** | **类型** | **必填** | **示例** | **说明**                                      |
| ---------------- | -------------- | -------------- | -------------- | --------------------------------------------------- |
| placement_id     | Long           | 是             | 1001           | 广告位ID（后端据此确定广告版式，如 Banner/Sidebar） |
| cookie_id        | String         | 否             | usr_123        | 用户的唯一标识（若前端 Cookie 为空则不传）          |
| context_tag      | String         | 否             | digital        | 当前页面内容的标签（用于更新画像和上下文匹配）      |
| mime_type        | String         | 否             | video/mp4      | 期望的媒体类型（仅视频插播时需要传）                |

* **响应示例 (Success Response)**:

```
{
  "code": 200,
  "message": "success",
  "data": {
    "ad_id": 505,
    "type": "image",
    "layout": "banner",
    "src": "http://img.ad-server.com/uploads/iphone15_banner.jpg",
    "link": "http://apple.com/buy",
    "cookie_id": "usr_999_new", 
    "tracking_url": "http://api.ad-server.com/api/v1/ad/track"
  }
}
```

> **注意**：如果请求中没传 **cookie_id**，后端会在响应中的 **cookie_id** **字段返回一个新生成的 ID，前端 SDK 必须将其写入浏览器 Cookie。**

### 2. 行为上报 (Tracking)

**用于统计展示量（CPM）和点击量（CPC）。**

* **URL**: **POST /ad/track**
* **Content-Type**: **application/json**
* **请求体示例**:

**code**JSON

```
{
  "ad_id": 505,
  "placement_id": 1001,
  "cookie_id": "usr_999_new",
  "event_type": "impression", 
  "timestamp": 1718888888
}
```

> **event_type** **枚举值：**impression **(展示),** **click** **(点击).**

---

## 三、 广告业主后台 API (Advertiser APIs)

  **使用者**：广告主管理面板。
鉴权方式**：Header 需携带 Token。**

### 1. 上传素材

* **URL**: **POST /upload/media**
* **Content-Type**: **multipart/form-data**
* **参数**: **file** **(文件流)**
* **响应**:

**code**JSON

```
{
  "code": 200,
  "message": "success",
  "data": {
    "url": "http://server/uploads/ads/video_01.mp4",
    "mime_type": "video/mp4"
  }
}
```

### 2. 创建广告

* **URL**: **POST /advertiser/ads**
* **请求体示例**:

**code**JSON

```
{
  "title": "夏季运动鞋大促",
  "category_id": 102,
  "media_url": "http://server/uploads/ads/shoe_sidebar.jpg",
  "ad_type": 0,
  "layout": "sidebar",
  "landing_page": "http://nike.com/promo",
  "weekly_budget": 5000.00
}
```

> **ad_type**: 0 (图片), 1 (视频). **layout**: banner / sidebar / card.

### 3. 获取广告列表

* **URL**: **GET /advertiser/ads**
* **参数**: **status** **(可选，如** **pending**, **active**)
* **响应**:

**code**JSON

```
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "ad_id": 1,
      "title": "测试广告",
      "status": "active"
    }
  ]
}
```

---

## 四、 站长后台 API (Publisher APIs)

 **使用者**：内容网站负责人。

### 1. 提交网站认证

* **URL**: **POST /publisher/site/verify**
* **请求体**:

**code**JSON

```
{
  "domain": "http://www.news.com",
  "name": "每日新闻网"
}
```

* **后端逻辑**: 访问 **http://www.news.com/token.txt** **验证所有权。**

### 2. 创建广告位 (Create Placement)

**站长定义自己的网站哪里有空位，以及空位的形状。**

* **URL**: **POST /publisher/placements**
* **请求体示例**:

**code**JSON

```
{
  "website_id": 12,
  "name": "文章详情页右侧栏",
  "layout": "sidebar"
}
```

* **响应**:

**code**JSON

```
{
  "code": 200,
  "message": "success",
  "data": {
    "placement_id": 2002
  }
}
```

### 3. 获取我的广告位

* **URL**: **GET /publisher/placements**
* **响应**:

**code**JSON

```
{
  "code": 200,
  "message": "success",
  "data": [
    { "placement_id": 2002, "name": "右侧栏", "layout": "sidebar" }
  ]
}
```

---

## 五、 管理员 API (Admin APIs)

### 1. 审核广告

* **URL**: **POST /admin/audit**
* **请求体**:

**code**JSON

```
{
  "ad_id": 505,
  "pass": true,
  "reason": "合规"
}
```

### 2. 添加分类标签

* **URL**: **POST /admin/categories**
* **请求体**:

**code**JSON

```
{ 
  "name": "数码电子" 
}
```

---

## 六、 视频网站特殊对接说明

**针对** **视频分享网站 (Server 4)** **的流媒体插播功能，对接逻辑如下：**

* **触发时机**：
  前端 JS 监听 **video** **标签的** **timeupdate** **事件，当播放时间 > 阈值（如10秒）时触发。**
* **API 调用参数**：

  * **mime_type**: 必须传 **video/mp4**，告诉后端我只要视频。
* **placement_id**: 必须是一个在后台配置为 **layout=card** **(或者专用的 video layout) 的广告位 ID。**
* **前端代码逻辑示例**:

**code**JavaScript

```
// 伪代码
fetch('http://server1/api/v1/ad/serve?placement_id=3001&mime_type=video/mp4')
  .then(res => res.json())
  .then(response => {
      if(response.code === 200) {
          const adUrl = response.data.src;
          // 逻辑：暂停主视频，播放 adUrl，播放完恢复
          playAdOverlay(adUrl);
      }
  });
```
