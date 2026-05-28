# Little Pony（小马出行）项目说明

## 项目概述
Little Pony（小马出行）是一个 H5 页面载体的小工具，面向15-60岁有出行计划的人群。

## 技术架构
- 单页 H5 应用，所有代码在 `index.html` 中
- 基于 screen 栈管理实现页面切换（`switchScreen()` / `goBack()`）
- 本地预览：`python3 -m http.server 8080`（从 `little-pony/` 目录启动）

## 核心页面
- **home** - 首页：统一输入框（自然语言）、定位栏、推荐卡片（含内嵌标签：天气/购票）
- **chat** - 对话页：上下文感知的灵活问答式出行规划
- **result** - 结果页：推荐方案列表
- **detail** - 详情页：Tab 切换每日行程，每景点内嵌标签，底部操作栏
- **food** - 美食页
- **photo** - 旅拍页
- **clothes** - 穿衣建议页
- **my** - 我的页面

## 已知技术问题
- `.screen` 元素设置了 `will-change: transform`，会创建新的包含块，导致子元素 `position: fixed` 失效
- **解决方案**：需要 fixed 定位的元素（如 `.detail-bottom-bar`）必须放在 `.screen` 容器**外部**

## 聊天页布局
- `#screen-chat` 使用 flex column 布局（覆盖 `.screen` 的 `overflow-y:auto` 和 `padding-bottom:90px`）
- 布局顺序：status-bar → plan-header → chat-container(flex:1) → chat-quick-replies → chat-input-bar
- `chat-container` 用 `flex:1; min-height:0` 占满剩余空间，内部 `.chat-messages` 为滚动区域
- `chat-input-bar` 用 `flex-shrink:0` 固定在底部，不再用 `position:absolute`，避免遮挡消息
- `chat-quick-replies` 用 `flex-shrink:0; max-height:120px; overflow-y:auto` 防止无限撑高

## 首页布局
- **Header**：Logo + 定位栏（📍当前城市，默认"杭州"，点击提示开发中）+ 通知铃铛
- **统一输入框**（`.search-box`）：合并原搜索框和目的地输入框，支持自然语言输入
  - placeholder："告诉我你想去哪、什么时候出发..."
  - 输入后按回车或点击 → 按钮调用 `handleHomeInput()`
  - `handleHomeInput()` → `parseTravelIntent()` 解析意图 → 设 `window._smartChatIntent` → `switchScreen('chat')`
- **快捷服务**：6项（买机票/订酒店/买门票/特色饮食/旅拍道具/穿衣建议），已移除"避坑指南"和"人流热力"
- **推荐卡片**：仅保留天气标签（`.card-tag.weather`），已移除人流和避坑标签（仅首页移除，详情页保留）

## 详情页设计规范
- 每日行程通过 Tab 切换（`switchDay()`），不纵向堆叠
- 天气/穿衣/人流/避坑/费用等信息**内嵌在每个景点条目中**，不单独分区展示
- 每日顶部显示天气穿衣栏（`day-weather-bar`），底部显示当日花费（`day-cost-bar`）
- 每个付费景点有 🎫 购票标签（`buy-tag`），点击调用 `buySingleTicket()`
- 底部操作栏含：分享、一键订票（已移除收藏按钮）
- 相邻景点之间有轻量导航条（`transit`），提供步行/打车/地铁公交选项，点击调用 `openNav()`

## 购票系统（已实现）
- **单票购买**：每个付费景点有 `buy-tag` 按钮，点击弹出购票弹窗（`single-ticket-modal`）
  - 支持调整购买数量（`adjustQty()`），自动计算小计
  - 确认购买调用 `confirmSingleBuy()`
- **一键订票**：底部"一键订票"按钮点击弹出批量购票弹窗（`batch-ticket-modal`）
  - `allPaidItems` 数组管理6项收费项目（湖滨酒店¥480、楼外楼¥180、灵隐寺¥75、河坊街¥60、西溪湿地¥180、高铁¥263）
  - 默认全选，支持反勾选（`toggleBatchItem()`），实时动态计算合计（`updateBatchTotal()`）
  - 确认购买调用 `confirmBatchBuy()`
- **弹窗通用**：`closeTicketModal()` / `closeModal()` 关闭弹窗
- **购票标签样式**：`.buy-tag` 渐变红色背景，白色文字，带阴影和按下缩放效果

## 景点间导航（已实现）
- 相邻景点之间嵌入轻量导航条（`.transit`），视觉弱化，不抢景点信息的主视觉
- 每段导航提供 🚶步行、🚗打车、🚌地铁/公交 三个按钮（步行不现实时显示"—"）
- 每个按钮显示图标+方式名+预估时间，紧凑排列，用竖线分隔
- 点击按钮调用 `openNav(mode, from, to)`，传入出行方式和起终点，后续可对接高德/百度地图 SDK
- CSS：`.transit-mode-btn` 无背景透明按钮，hover 浅灰，按下缩放；`.transit-sep` 竖线分隔符
- 导航条在 `.timeline` 容器内，左侧有 `::before` 小圆点与时间线竖线对齐

## 首页卡片（已修复）
- 5 张卡片：杭州/北京(→default)/成都/大理/青岛
- 每张卡片结构：`.card` > `.card-img`(背景图+badge) + `.card-body`(标题+描述+meta+tags)
- **重要**：修改卡片 onclick 时，不能丢失 `.card-img` div，否则图片区域缺失导致排版错乱

## 底部导航
- 三个入口：首页、+（新建对话/规划）、我的
- 已移除"发现"和"收藏"Tab
- "+"按钮进入聊天页时走原固定流程（`startChat()`），首页输入框进入走灵活流程（`startSmartChat()`）

## 智能对话系统（已实现）

### 双入口
1. **首页输入框** → `handleHomeInput()` → `parseTravelIntent()` → `startSmartChat(intent)` — 灵活问答
2. **"+"按钮** → `switchScreen('chat')` → `startChat()` — 固定10步流程（保留作为备用）

### 自然语言解析 `parseTravelIntent(text)`
纯前端关键词+模糊匹配，从用户输入中提取：
- **目的地**（5层提取）：
  1. 匹配 cityDB 的城市名 + 景点特征关键词
  2. 国际国家/地区名直接匹配（约120个国际城市/国家名，优先于模糊匹配）
  3. 模糊匹配（"海边"→三亚/青岛/厦门、"极光"→冰岛、"樱花"→日本、"金字塔"→埃及 等，约43组）
  4. 模式提取：从"去XX玩/旅游/度假/度蜜月"、"XX有什么好玩的"、"XX怎么样"、"XX度假"等句式中提取任意地名
  5. 兜底匹配：`[想去飞到] + 2-6字` 模式
  - **否定目的地检测**：识别"不想去XX了""别去XX了""不去XX了"等否定表达，将被否定的目的地从结果中排除（`negatedDest` 字段记录）
  - **否定后正向重扫描**：否定后从文本剩余部分重新提取新目的地（如"不想去法国了，想去土耳其"→土耳其），扫描时排除已否定的文本片段
  - **地区别名映射**：子地区名自动映射到父目的地（南法/普罗旺斯/尼斯→法国、托斯卡纳/米兰→意大利、关东/关西→日本等约40组）
  - 提取后自动去除首尾冗余（前缀"去/到/飞/想"，尾部数字+单位"3天"等）
  - 过滤无意义词（"哪里""什么""去看极光"等）
- **人数**：正则+语义匹配"3人"、"情侣"、"亲子"、"带爸妈"、"和闺蜜"等
- **预算**：正则匹配"3千"、"2万"、"预算5000" + 语义推断（"穷游"→1500、"不差钱"→8000）
- **天数**：正则匹配"3天"、"5日" + 语义推断（"周末"→2天、"小长假"→3天、"长假"→7天、"深度"→5天）
- **偏好**：关键词匹配美食/文化/自然/休闲/拍照（大幅扩展同义词，如"烧烤""火锅"→美食、"潜水""冲浪"→自然）
- **话题**（`topics[]`）：识别用户在问什么（weather/food/hotel/transport/cost/safety/features/itinerary）

### 城市数据库 `cityDB` 与未知城市支持 `getOrInitCity(city)`
20个预设城市，每个包含：`{ type, features[], food[], weather, img, dailyCost }`
- 预设7城（有完整 resultData/detailData）：杭州、北京、成都、大理、厦门、西安、青岛
- 动态生成13城：三亚、丽江、重庆、南京、长沙、苏州、桂林、拉萨、昆明、哈尔滨、新疆、西藏、云南
- **未知城市自动生成**：`getOrInitCity(city)` 对不在 cityDB 中的地名，基于关键词推断生成城市档案：
  - 15个国内地区规则：西域(伊犁/阿勒泰/喀什等)、蜀地山水(九寨/峨眉/稻城等)、湘楚(张家界/凤凰等)、彩云秘境(香格里拉/腾冲等)、徽派(黄山/宏村等)、丝路走廊(敦煌/张掖等)、海滨(威海/烟台等)、水乡古镇(乌镇/周庄等)、草原(呼伦贝尔/鄂尔多斯等)、东北(长白山/延吉等)、山水秘境(阳朔/黄果树等)、晋商(平遥/五台等)、赣鄱(庐山/景德镇等)、高原明珠(青海湖/茶卡等)
  - 23个国际地区规则：欧陆风情(法国/意大利/英国等)、东南亚风情(泰国/越南/柬埔寨等)、和风秘境(日本)、韩流之旅(韩国)、美利坚探索(美国)、南半球探险(澳洲/新西兰)、中东奢华(迪拜/阿联酋)、南亚秘境(印度/尼泊尔)、非洲探奇(非洲/埃及/摩洛哥/肯尼亚/坦桑尼亚等)、北欧极光(冰岛/挪威/芬兰)、海岛天堂(马尔代夫/斐济)、花园城市(新加坡)、横跨欧亚(土耳其)、拉美风情(南美/巴西/阿根廷/秘鲁)、俄式壮美(俄罗斯)、枫叶之国(加拿大)、中东风情(中东/沙特/约旦/以色列等)、丝路古国(中亚/乌兹别克斯坦/哈萨克斯坦等)、东欧秘境(东欧/波兰/捷克/匈牙利等)、北美之旅(北美)、极地探险(南极)
  - 国际国家/地区名直接匹配（约130个），优先于 fuzzyMap 体验关键词匹配
  - fuzzyMap 国际体验关键词约49组（含safari→肯尼亚、动物大迁徙→肯尼亚、狮子→肯尼亚等）
  - 推断字段：type(类型)、features(景点)、food(美食)、weather(天气)、dailyCost(日均花费)
  - 结果缓存到 cityDB，后续调用直接返回
  - 所有下游函数（buildCityIntro、buildFeaturesReply、buildCostReply、generateDynamicResultData、generateDynamicDetailData 等）均使用 `getOrInitCity(city)` 替代 `cityDB[city]`，确保未知城市也能正常出方案

### 灵活问答流程（`startSmartChat` → `handleSmartFreeText`）
- 每轮自由文本输入经 `parseTravelIntent()` 提取旅行参数+话题，直接基于解析结果决定回复
- **基于优先级的回复逻辑**（不用 switch/case 意图分类）：
  1. 用户明确要出方案 → `fillDefaults()` + `generateFromSmartChat()`
  2. 用户要换目的地（有新目的地）→ 更新 chatState + 城市介绍
  3. 用户否定当前目的地 + 主题偏好 → 清除目的地 + 基于主题推荐（如"丝绸之路"→推荐土耳其/埃及/摩洛哥）
  4. 用户有话题（topics 不为空）→ 走对应回复构建器（天气/美食/住宿/交通/费用/安全/景点/行程）
  5. 解析到新旅行信息 → 确认记录 + 引导
  6. 社交意图 → 友好回复
  7. 智能兜底 `buildSmartReply()` → 分析推荐/比较/纠结/随意等意图
- **不追问缺失字段**：用户主动对话时，只回复+显示操作按钮，不再自动追问
- **`fillDefaults()`**：出方案时自动填充默认值（2人/3天/休闲/按城市日均×天数）
- **上下文感知**：回复内容基于 chatState 中已有的目的地、人数、预算、偏好等上下文
- 方案生成后输入栏不隐藏，用户可继续对话调整方案

### 意图→回复映射（上下文感知回复构建器）
- `buildWeatherReply(text)` — 天气+穿搭建议，区分热/雨/冷/舒适场景
- `buildFoodReply(text)` — 美食推荐，列出必吃榜单+觅食地+省钱tips
- `buildHotelReply(text)` — 住宿建议，含推荐区域+价格区间+人群适配
- `buildTransportReply(text)` — 交通建议，区分高铁/飞机/市内三种场景
- `buildCostReply(text)` — 费用明细，分项列出住宿/餐饮/门票/交通+预算对比
- `buildSafetyReply(text)` — 安全提示，基于城市类型（高原/海边/雨季）给出针对性建议
- `buildFeaturesReply(text)` — 景点推荐，分经典必去+小众推荐+偏好路线
- `buildItineraryReply(text)` — 行程建议，按天列出上午/下午/晚上安排
- `buildCityIntro(city)` — 城市简介（类型+天气+景点+美食+日均花费）
- `buildContextSummary()` — 当前计划摘要
- `extractCityFromText(text)` — 从自由文本中提取城市名
- `mergeIntentToState(intent)` — 将新解析的信息合并到chatState（允许更新目的地，其余字段不覆盖）

### 快捷操作
- `showFollowUpActions()` — 根据当前上下文动态显示快捷按钮（有目的地时：出方案/有什么好玩/美食推荐/天气穿搭/费用明细/住宿推荐/换个目的地；无目的地时：推荐目的地；未知城市也显示全部按钮）
- `handleFollowUpAction(value)` — 处理快捷操作点击，触发对应的回复构建器（`__change_dest` 使用对话式引导，不再调用 askNextQuestion）
- `mergeIntentToState(intent)` — 合并新信息到 chatState，支持 `negatedDest` 清除当前目的地
- `generateFromSmartChat(isRegenerate)` — 生成结果摘要并跳转结果页，方案生成后继续显示后续操作

### 关键函数
- `addBotBubble(text)` — 添加机器人消息气泡（支持换行）
- `addUserBubble(text)` — 添加用户消息气泡
- `askNextQuestion(missing)` — 仅在缺少目的地时追问，其余字段不追问
- `handleSmartQuickReply(value, text, field)` — 处理快捷回复点击，不触发追问
- `handleSmartMultiDone(field)` — 处理多选完成，不触发追问
- `handleSmartFreeText(text)` — 核心对话引擎：解析意图→基于优先级回复→显示操作按钮
- `buildSmartReply(text, intent)` — 智能兜底：分析推荐/比较/纠结/随意等意图
- `parseTravelIntent(text)` — 增强版解析器：城市+模糊匹配、语义推断预算/天数/人数、话题识别
- `fillDefaults()` — 出方案前自动填充默认值

### `switchScreen()` 聊天启动逻辑
- 优先检查 `window._smartChatIntent`（首页输入框进入），有则启动 `startSmartChat`
- 无 intent 且 `currentChatStep < 0` 时启动 `startChat`（"+"按钮首次进入）
- **注意**：`_smartChatIntent` 不受 `currentChatStep` 限制，确保从首页输入框进入时始终能启动对话

## 聊天问答流程（原固定流程，"+"按钮入口）
- Step 0: 是否有明确目的地
- Step 1: 目的地选择（有目的地时）
- Step 2: 是否已有攻略
- Step 3: 出行时间是否确定（无目的地时）
- Step 4: 时间/预算详情（无目的地时）
- Step 5: 出行偏好（多选）
- Step 6: 出行人数（已移除年龄段问题）
- Step 7: 预算
- Step 8: 特殊需求（多选）
- Step 9: 自动生成建议（`autoHandler: true`，消息显示后自动执行）
- **重要**：Step 9 无快捷回复，handler 通过 `autoHandler` 标记在 `advanceChat` 中自动调用，否则会卡住无下文

## 动态方案/详情生成（已实现）
- **`generateDynamicResultData(city)`** — 为 cityDB 中非预设城市动态生成 resultData
  - 根据天数、偏好、预算生成 2-3 个方案（经典/深度/性价比）
  - 方案名和标签根据偏好调整（如美食→"风味之旅"，自然→"风光之旅"）
- **`generateDynamicDetailData(city, planIndex)`** — 为非预设城市动态生成 detailData
  - 按天拆分景点（`splitFeatures()`），穿插交通导航和餐饮
  - 生成人流/提示标签，计算每日费用和总费用
  - 结构与预设城市 detailData 完全一致
- **`renderDetail()` 降级逻辑**：先查 `detailData[city]`，无则调 `generateDynamicDetailData()`，最后 fallback `detailData['default']`
- **`generateAndShowResults()` 降级逻辑**：先查 `resultData[dest]`，无则调 `generateDynamicResultData()`，最后 fallback `resultData['default']`

## 真实数据（各城市详情页）
所有景点、价格、时间、交通均为真实参考数据：
- **杭州** (¥680-¥1,400)：3个方案（西湖慢生活3天¥1,238 / 文艺深度3天¥1,400 / 特种兵2天¥680）
- **北京** (¥2,800-¥3,800)：2个方案（皇城文化5天¥3,800 / 美食打卡4天¥2,800）
- **成都** (¥1,800-¥2,600)：2个方案（熊猫美食4天¥2,600 / 慢生活3天¥1,800）
- **大理** (¥2,500-¥3,200)：2个方案（风花雪月5天¥3,200 / 环海骑行4天¥2,500）
- **厦门** (¥1,600-¥2,800)：2个方案（海岛文艺4天¥2,800 / 美食探店3天¥1,600）
- **西安** (¥1,500-¥2,800)：2个方案（千年古都4天¥2,800 / 美食考古3天¥1,500）
- **青岛** (¥2,000-¥2,200)：2个方案（海鲜啤酒3天¥2,200 / 海岸线徒步3天¥2,000）
- 美食页：楼外楼⭐4.3人均¥150、知味观⭐4.6人均¥60、新白鹿⭐4.4人均¥80、奎元馆⭐4.5人均¥45

## 动态详情页（已实现）
- **数据结构**：`detailData` 对象，键名与 `resultData` 一致（`'杭州'`/`'成都'`/`'大理'`/`'厦门'`/`'西安'`/`'青岛'`/`'default'`），每个城市含 plans 数组，与 resultData 的 plans 索引对应
- 每个 plan 结构：`{ title, subtitle, heroImg, totalCost, paidItems[], shareBody, days[] }`
- 每个 day 结构：`{ label, weather, cost, items[] }`
- items 中景点：`{ time, activity, cost, note, tags[] }`；导航：`{ transit: [{ mode, from, to, time }] }`
- tag type 映射：`buy` → `buy-tag`（含 name/ticketType/price），`crowd-green/yellow/red`/`avoid`/`tip` → 对应 CSS class
- **渲染函数**：
  - `openDetail(city, planIndex)` — 入口函数，调用 renderDetail() + switchScreen('detail')
  - `renderDetail(city, planIndex)` — 更新 hero 图/标题、替换 .detail-content innerHTML、更新 allPaidItems、底栏价格、分享弹窗
  - `buildDetailHTML(plan)` — 生成日标签+日面板 HTML（动态天数，ID 为 `day-0`/`day-1`/...）
  - `buildTimelineItemHTML(item)` — 生成单个景点条目
  - `buildTransitHTML(modes)` — 生成导航条（图标映射：步行🚶/打车🚗/公交🚌/地铁🚇/骑行🚴）
  - `updateShareModal(plan)` — 更新 #share-body 和 .sp-sub
- **全局状态**：`currentDetail = { city, planIndex }`
- **allPaidItems**：改为 `let allPaidItems = []`，由 renderDetail() 动态赋值
- **详情页 HTML**：骨架结构，`.detail-content` 为空容器，由 renderDetail() 动态填充
- **底栏价格**：`.dbb-total` 由 renderDetail() 动态更新
- **首页卡片 onclick**：`openDetail('杭州',0)` / `openDetail('default',0)` / `openDetail('成都',0)` / `openDetail('大理',0)` / `openDetail('青岛',0)`
- **结果页卡片 onclick**：`openDetail('${dest}',${i})`，dest 和 i 来自 generateAndShowResults()
- **各城市方案数量**：杭州3个、北京2个、成都2个、大理2个、厦门2个、西安2个、青岛2个、default 1个
- **北京卡片**：首页有卡片，onclick 为 `openDetail('北京',0)`，已添加北京 resultData 和 detailData

## 详情页排版优化（已实现）
- **视觉层次**：时间 13px/700 > 景点名 15px/600 > 备注 12px/1.5行高 > 标签 11px，字体由大到小递进
- **价格更显眼**：`.item-cost` 14px/700，去掉 float:right，改为 flex 行内布局
- **购票按钮移至价格旁**：buy-tag 从 `.spot-tags` 标签组中分离，放到 `.activity` 行内紧挨 `.item-cost`
  - `.activity` 改为 flex 布局，景点名包裹在 `<span class="activity-name">`（flex:1）
  - 布局顺序：景点名 → 价格 → 购票按钮
  - `.spot-tags` 中只保留 crowd/avoid/tip 类标签
- **标签组优化**：gap 5px、margin-top 8px，标签字号 11px、padding 3px 8px，更易读
- **`buildTimelineItemHTML()` 改动**：先分离 buy 和其他 tags，buy 放 activity 行内，其他 tags 单独渲染

## 目的地图片系统（已实现）

### 图片来源
- 所有目的地配图使用 Unsplash 免费图片
- URL 格式：`https://images.unsplash.com/photo-{LONG_ID}?w=400&h=200&fit=crop`
- **必须使用长格式 ID**（如 `photo-1526481280693-3bfa7568e0f3`），短格式 ID（如 `photo-XuMFb5DjVZU`）返回 HTTP 404
- **禁止使用** `plus.unsplash.com/premium_photo-` 格式（付费图片，不可用），只能用 `images.unsplash.com/photo-` 格式

### cityDB 20城图片映射
| 城市 | Unsplash 长格式 ID | 描述 |
|---|---|---|
| 杭州 | `photo-1526481280693-3bfa7568e0f3` | 西湖 |
| 北京 | `photo-1509023464722-18d996393ca8` | 故宫 |
| 成都 | `photo-1588252910189-9c9f5535646b` | 成都 |
| 大理 | `photo-1725378812977-cc475abfa5a2` | 洱海 |
| 厦门 | `photo-1721794525689-d2bd76190f1e` | 厦门海滨 |
| 西安 | `photo-1710141925256-652f9ace7782` | 西安古城 |
| 青岛 | `photo-1627868153411-624a8dce0a12` | 青岛海岸 |
| 三亚 | `photo-1710428447167-ef713a6f42b9` | 三亚热带海滩 |
| 丽江 | `photo-1677922069750-944be2b9ad20` | 丽江古镇 |
| 重庆 | `photo-1620321281938-2522ba59a1df` | 重庆夜景 |
| 南京 | `photo-1589259411045-1841cec4f7a6` | 南京 |
| 长沙 | `photo-1583143574609-b2e508237650` | 长沙夜景 |
| 苏州 | `photo-1745114392692-ede344e12e34` | 苏州园林 |
| 桂林 | `photo-1773318901379-aac92fdf5611` | 桂林山水 |
| 拉萨 | `photo-1703842079863-20413f288d03` | 布达拉宫 |
| 昆明 | `photo-1724130344214-d66eab70b1d2` | 昆明 |
| 哈尔滨 | `photo-1716308352490-271359910f14` | 冰雪大世界 |
| 新疆 | `photo-1757069540493-8f74088b3269` | 天山 |
| 西藏 | `photo-1703842079863-20413f288d03` | 西藏（同拉萨） |
| 云南 | `photo-1776294984950-3f812aa0f797` | 云南梯田 |

### getOrInitCity 国际规则图片映射
| 规则关键词 | Unsplash 长格式 ID | 描述 |
|---|---|---|
| 欧洲/法国 | `photo-1679231926688-ef9cdab5ed2f` | 巴黎铁塔 |
| 东南亚 | `photo-1563492065599-3520f775eeed` | 东南亚佛塔 |
| 日本 | `photo-1490806843957-31f4c9a91c65` | 富士山 |
| 韩国 | `photo-1666670750287-176e3218ce12` | 景福宫 |
| 美国/北美 | `photo-1588384153148-ebd739ac430c` | 自由女神 |
| 澳洲 | `photo-1624138784614-87fd1b6528f8` | 悉尼歌剧院 |
| 迪拜 | `photo-1544092683-c0c9ebb368e5` | 迪拜塔 |
| 印度 | `photo-1564507592333-c60657eea523` | 印度 |
| 非洲 | `photo-1575550959106-5a7defe28b56` | 非洲safari |
| 冰岛/南极 | `photo-1531366936337-7c912a4589a7` | 极光 |
| 马尔代夫 | `photo-1573843981267-be1999ff37cd` | 马尔代夫水屋 |
| 新加坡 | `photo-1552790920-0ba96dc99d01` | 鱼尾狮 |
| 土耳其 | `photo-1604156789095-3348604c0f43` | 热气球 |
| 巴西/拉美 | `photo-1518639192441-8fce0a366e2e` | 基督像 |
| 俄罗斯 | `photo-1662679790388-e7b60d008ea0` | 圣瓦西里教堂 |
| 中东 | `photo-1549396535-c11d5c55b9df` | 中东清真寺 |
| 中亚 | `photo-1744177332411-9a57cd922af7` | 中亚 |
| 东欧 | `photo-1518471152222-d42e38ce6873` | 布达佩斯 |
| 加拿大 | `photo-1587381419916-78fc843a36f8` | 落基山脉 |

### getOrInitCity 国内规则图片映射
| 规则关键词 | Unsplash 长格式 ID | 描述 |
|---|---|---|
| 西域(伊犁/新疆) | `photo-1757069540493-8f74088b3269` | 天山（同新疆） |
| 蜀地山水(九寨/峨眉) | `photo-1588252910189-9c9f5535646b` | 成都（同城市图） |
| 湘楚(张家界) | `photo-1583143574609-b2e508237650` | 长沙（同城市图） |
| 彩云秘境(香格里拉) | `photo-1776294984950-3f812aa0f797` | 云南（同城市图） |
| 徽派(黄山) | `photo-1568454158153-6bf6cfda9070` | 黄山 |
| 丝路走廊(敦煌) | `photo-1507169820737-80f8c50e81f7` | 敦煌 |
| 海滨度假 | `photo-1627868153411-624a8dce0a12` | 青岛（同城市图） |
| 水乡古镇 | `photo-1526481280693-3bfa7568e0f3` | 杭州（同城市图） |
| 草原天堂 | `photo-1755789450537-6e1838cb3558` | 草原蒙古包 |
| 东北风光 | `photo-1716308352490-271359910f14` | 哈尔滨（同城市图） |
| 山水秘境 | `photo-1773318901379-aac92fdf5611` | 桂林（同城市图） |
| 晋商文化 | `photo-1509023464722-18d996393ca8` | 北京（同城市图） |
| 赣鄱胜景 | `photo-1773318901379-aac92fdf5611` | 桂林（同城市图） |
| 高原明珠 | `photo-1703842079863-20413f288d03` | 拉萨（同城市图） |

### getOrInitCity 默认图片
- `img = 'photo-1476514525535-07fb3b4ae5f1'`（旅行胜地默认图，长格式，可正常加载）

### 图片涉及的位置
- **cityDB**（行 ~1276-1297）：20个城市的 `img` 字段
- **getOrInitCity 默认 img**（行 ~1303）
- **getOrInitCity 规则**（行 ~1308-1490）：15个国内 + 23个国际规则，每个有 `img = 'photo-xxx'`
- **首页卡片**（行 ~499-558）：5张卡片 `background-image:url(https://images.unsplash.com/photo-xxx?w=400&h=200&fit=crop)`
- **resultData**（行 ~2620-2655）：方案列表 `img` 字段
- **detailData**（行 ~2665-3410）：16个 `heroImg` 字段
- **generateDynamicResultData**（行 ~2494-2528）：使用 `img: info.img`（从 getOrInitCity 获取）

### 修改图片的注意事项
- **禁止使用短格式 ID**：从 Unsplash 网页搜索得到的短 ID（如 `XuMFb5DjVZU`）不能直接拼成 `images.unsplash.com/photo-XuMFb5DjVZU`，必须获取长格式 ID
- **获取长格式 ID 的方法**：在 Unsplash 搜索页面中，找到 `images.unsplash.com/photo-` 开头的 URL，其中的 `photo-XXXXXXXXXX-XXXXXXXXXX` 即为长格式 ID
- **验证方法**：`curl -sI "https://images.unsplash.com/photo-{ID}?w=400&h=200&fit=crop"` 检查是否返回 HTTP 200
- **含中文的文件编辑**：因 Edit 工具对中文支持不佳，推荐用 Python 脚本进行批量替换

## GitHub Pages 部署
- **仓库地址**：https://github.com/juzipi94-jpg/little-pony
- **在线访问**：https://juzipi94-jpg.github.io/little-pony/index.html
- **部署方式**：GitHub Pages，legacy 模式，从 main 分支根目录部署
- **推送方式**：通过 `gh` CLI 认证（OAuth token 带 repo scope），git remote URL 内嵌 token
- **注意**：OAuth App 无 workflow scope，无法推送 `.github/workflows/` 文件，因此使用 legacy 分支部署而非 Actions 部署

## 待实现功能
1. **我的门票**：在"我的"页面增加门票管理区域，支持查看已购门票、退票等操作
