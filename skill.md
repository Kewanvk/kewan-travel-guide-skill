---
description: "生成一个完整的旅行攻略网站：从目的地研究到网站部署"
---

# Travel Guide Generator

根据用户提供的旅行目的地和基本信息，生成一个完整的、可部署的旅行攻略静态网站。

## 输入

用户需要提供：
- 目的地城市
- 出行日期和天数
- 出行人数和人群特点（如老人/小孩/饮食偏好）
- 已有的航班/住宿信息（截图或链接）
- 风格偏好（如温暖/极简/活泼）
- 部署地址（域名+路径）

## 工序

### 第一步：信息结构设计

确定网站要包含的板块，典型结构：
1. 目的地介绍（历史文化 + 现代旅游卖点 + 概览地图）
2. 航班信息
3. 住宿列表（已确认/待确认）
4. 景点推荐（10个）
5. 餐厅推荐（10家）
6. 特色体验/活动
7. 实用信息（天气/交通/注意事项）

### 第二步：数据研究

**住宿**：
- 用户提供 Airbnb 截图 → 提取房源名称/价格/日期/人数
- Google Maps `search_places` + `place_details` → 评分、评价数、坐标、照片
- 写入结构化数据文件

**景点**：
- 用户提供的参考资料（小红书截图、攻略帖等）提取高频推荐
- Google Maps 交叉验证：评分 + 评价数 + 距离
- 按"博主推荐频次 × Google 评分"排序取 Top 10

**餐厅**：
- 同上逻辑，但改为按具体餐厅推荐（不是按菜品）
- 每家标注：Google 评分/评价数/推荐菜/人均价格（当地货币+人民币）/饮食注意
- 价格换算：查当前汇率，取整到个位

**活动/体验**：
- Google Maps 搜索 "[activity] class [city]" + minRating 4.5
- 按类别取评分最高的推荐店

### 第三步：图片采集

优先级排序：
1. **Google Maps Place Photos**（最准确）：`place_details` maxPhotos 拿 URL → curl 下载 → Read 工具目视验证
2. **Pexels**（补位）：WebFetch 搜索页 → 提取直链 → 下载
3. **用户自有图片**：截图/相册

**关键规则**：
- 每张图必须 Read 验证内容与标签匹配，不能盲信文件名
- 所有图片本地化存储，不依赖外部 CDN（保证国内可访问）
- 住宿图 4 张（首图选最有辨识度的视角），景点/餐厅各 1 张

### 第四步：地图方案

**住宿内嵌地图**：
- 用外部 JS 文件（不用 React client component，避免 hydration 问题）
- IP 检测决定地图提供商：国内 → 高德，海外 → Google Maps
- ipapi.co 超时 fallback 先试高德（不通大概率在国内）
- 高德需要 Web端 JS API key + securityJsCode
- Google Maps 需要 JS Maps key + `v=weekly` + callback 模式

**概览地图**：
- 用 SVG 手绘风格，全中文标注
- 标注住宿点（编号+名称+日期）、关键地标、地理要素（山/河/古城）
- 矢量渲染，不依赖外部图片

### 第五步：技术实现

```
框架：Next.js (App Router, static export)
样式：Tailwind CSS v4 (@import "tailwindcss" + @theme inline)
配置：basePath 对应部署子路径, trailingSlash: true
构建：npm run build → out/ 目录
部署：rsync -avz --delete out/ server:/path/
```

**已知坑**：
- Turbopack + 中文路径会崩，用 webpack 或直接 build
- 地图代码不能用 inline script（hydration 会销毁），必须 external JS + defer
- Google Maps Marker 用 `callback` 模式加载，不用 `onload`

### 第六步：迭代验证

- 每次改动后 build + deploy + 刷新验证
- 图片逐张目视检查
- 国内/海外双端验证地图加载
- 价格/日期/地址交叉核对

## 输出

一个完整的静态网站，包含：
- 响应式布局（移动端优先）
- 所有图片本地化
- 地图自动适配国内/海外
- 全中文内容
- 一键部署脚本
