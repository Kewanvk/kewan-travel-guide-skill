# Travel Guide Generator — Claude Code Skill

A reusable methodology for building complete travel guide websites using Claude Code + Google Maps API.

## What it does

Takes a travel destination + basic trip info → outputs a fully deployable static website with:
- Destination intro with hand-drawn SVG map (Chinese-labeled)
- Flight info cards
- Accommodation listings with inline maps (auto-switches AMap/Google Maps by IP)
- Top 10 attractions (cross-referenced: blogger reviews × Google Maps ratings)
- Top 10 restaurants (with ratings, must-try dishes, price in local + CNY, dietary notes)
- Activity/experience recommendations (cooking class, Muay Thai, pottery, etc.)

## Key Techniques

### Research Pipeline
1. **User input**: screenshots from Airbnb/booking platforms, blogger recommendations (Xiaohongshu, etc.)
2. **Cross-validation**: Google Maps `search_places` + `place_details` for ratings, review counts, coordinates, photos
3. **Ranking**: `blogger_mention_frequency × google_rating` → Top N
4. **Image sourcing**: Google Maps Place Photos (most accurate) → Pexels (fallback) → always verify visually before deploying

### Map Embedding (China + Overseas)
```javascript
// IP detection → provider selection
fetch("https://ipapi.co/json/", { signal: AbortSignal.timeout(3000) })
  .then(r => r.json())
  .then(d => d.country_code === "CN" ? loadAMap() : loadGMap())
  .catch(() => loadAMap()); // ipapi blocked = likely China
```
- **AMap** for China (needs Web JS API key + securityJsCode)
- **Google Maps** for overseas (needs JS Maps key + `v=weekly` + `callback` pattern)
- External JS file, NOT inline script (React hydration destroys inline scripts)

### SVG Illustrated Map
Hand-drawn style overview map in pure SVG:
- All labels in Chinese
- Geographic features: mountains, rivers, old city walls
- Numbered accommodation markers with date labels
- Landmark indicators
- Compass + scale bar
- Vector rendering = crisp at any screen size, no external image dependency

### Tech Stack
- **Next.js** (App Router, `output: "export"` for static site)
- **Tailwind CSS v4** (`@import "tailwindcss"` + `@theme inline`)
- **Deploy**: `rsync -avz --delete out/ server:/path/`

## Known Gotchas

| Problem | Solution |
|---------|----------|
| Turbopack + CJK file paths | Use `next build` directly, avoid dev server |
| React hydration kills inline `<script>` | Use external `.js` file with `defer` |
| Google Maps `Marker` unreliable with `onload` | Use `callback` param + `v=weekly` |
| `ipapi.co` blocked in China → fallback loads Google (also blocked) | Fallback to AMap first (blocked = likely in China) |
| Airbnb pages return 403 | Use Google Maps reviews (guests often mention Airbnb location) |
| Stock photo mismatch | **Always** `Read` each downloaded image to visually verify before deploying |

## Usage

Install as a Claude Code skill:
```
~/.claude/commands/travel-guide.md
```

Then invoke with `/travel-guide` in Claude Code, providing your destination and trip details.

## License

MIT

---

# 旅行攻略生成器 — Claude Code Skill

用 Claude Code + Google Maps API 生成完整的旅行攻略网站的可复用方法论。

## 它做什么

输入旅行目的地 + 基本行程信息 → 输出一个可直接部署的静态攻略网站，包含：
- 目的地介绍 + 手绘风格 SVG 中文地图
- 航班信息卡片
- 住宿列表（内嵌地图，国内自动切高德/海外切 Google Maps）
- 10 个景点推荐（博主推荐 × Google Maps 评分交叉验证）
- 10 家餐厅推荐（评分/必点菜/当地货币+人民币价格/饮食注意）
- 特色体验活动推荐（泰餐课、泰拳、陶艺等）

## 核心工序

### 研究管线
1. **用户输入**：Airbnb/酒店截图、博主推荐（小红书等）
2. **交叉验证**：Google Maps `search_places` + `place_details` 拿评分、评价数、坐标、照片
3. **排序**：`博主提及频次 × Google 评分` → 取 Top N
4. **图片采集**：Google Maps Place Photos（最准确）→ Pexels（补位）→ 每张图下载后必须目视验证再上线

### 地图嵌入（国内 + 海外双端）
```javascript
// IP 检测 → 选择地图提供商
fetch("https://ipapi.co/json/", { signal: AbortSignal.timeout(3000) })
  .then(r => r.json())
  .then(d => d.country_code === "CN" ? loadAMap() : loadGMap())
  .catch(() => loadAMap()); // ipapi 不通 = 大概率在国内
```
- **高德地图**用于国内（需要 Web端 JS API key + securityJsCode）
- **Google Maps** 用于海外（需要 JS Maps key + `v=weekly` + `callback` 模式）
- 地图代码必须用外部 JS 文件，不能用 inline script（React hydration 会销毁）

### SVG 手绘地图
纯 SVG 绘制的概览地图：
- 全中文标注
- 地理要素：山脉、河流、古城轮廓
- 住宿点编号标记 + 日期标签
- 地标指示
- 指北针 + 比例尺
- 矢量渲染，任何屏幕清晰，不依赖外部图片

### 技术栈
- **Next.js**（App Router，`output: "export"` 静态导出）
- **Tailwind CSS v4**（`@import "tailwindcss"` + `@theme inline`）
- **部署**：`rsync -avz --delete out/ server:/path/`

## 已知的坑

| 问题 | 解法 |
|------|------|
| Turbopack + 中文路径崩溃 | 直接 `next build`，不走 dev server |
| React hydration 销毁 inline `<script>` | 用外部 `.js` 文件 + `defer` |
| Google Maps `Marker` 用 `onload` 不可靠 | 用 `callback` 参数 + `v=weekly` |
| `ipapi.co` 国内被墙 → fallback 加载 Google Maps（也被墙） | fallback 先试高德（不通大概率在国内） |
| Airbnb 页面返回 403 | 用 Google Maps 评价（住客经常提到 Airbnb 房源的实际位置） |
| 库存图片内容不匹配 | 每张图**必须** `Read` 目视验证再部署 |

## 使用方式

作为 Claude Code skill 安装：
```
~/.claude/commands/travel-guide.md
```

在 Claude Code 中输入 `/travel-guide`，提供目的地和行程信息即可。

## 许可证

MIT
