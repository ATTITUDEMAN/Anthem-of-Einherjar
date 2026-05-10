# Codex Work Log

## 记录规范
- 本文件用于记录 Codex 在本项目中的所有修改与操作。
- 从本次开始，后续每次代码修改都追加详细日志，至少包含：时间、目标、改动文件、核心改动点、验证结果、遗留事项。

## 时间
- 2026-05-10

## 阶段一：基于 Rules.md 完成首版前后端

### 目标
- 按 `Rules.md` 实现 Vue 前端 + Node.js WebSocket 后端。
- 使用组件化方案，不仅修改 `App.vue`。
- 提供现代化 UI 与基础动画。

### 主要操作与改动
1. 读取并解析需求
- 读取 `Rules.md`、`src/App.vue`、`src/components/HelloWorld.vue`、`ylsg-server.js`。
- 处理 `Rules.md` 编码问题，确认游戏规则、大厅流程、对战流程、后端通信要求。

2. 前端组件化重构
- 改造入口与主结构：`src/App.vue`。
- 新增组合式通信层：`src/composables/useGameSocket.js`。
- 新增组件：
  - `src/components/LobbyView.vue`
  - `src/components/GameView.vue`
  - `src/components/CardTile.vue`
  - `src/components/AdminPanel.vue`
  - `src/components/ToastOverlay.vue`
- 更新样式：`src/style.css`（现代化视觉与动画）。

3. 后端首版
- 完成 `ylsg-server.js` WebSocket 房间逻辑：创建、加入、开始、排序、同步状态、提示广播。
- 端口：`11451`。
- 在文件头注释中说明运行依赖与启动方式。

4. 依赖与脚本
- 更新 `package.json`：新增 `ws` 依赖和 `server` 启动脚本。

### 构建与验证
- 执行 `npm install`：成功。
- 执行 `npm run build`：初次因 `package.json` BOM 报错，修复编码后成功。
- 启动检查 `ylsg-server.js`：成功输出监听信息。

---

## 阶段二：按你的新要求重构为独立页面 + 后台认证 + 持久化

### 目标
- 对战页和后台页独立 URL。
- 简化 UI 配色（降低渐变视觉疲劳），保持现代感。
- 后台增加密码认证、可查看进行中游戏。
- 卡牌改为后台配置并持久化到 JSON；支持增删改查与查看详情。
- 未配置卡牌时前端不显示卡牌。
- 修复复制房间链接。
- 对战布局参考 Kards：左侧手牌拖到 6 个虚线槽位并适配尺寸。

### 主要操作与改动
1. 页面结构拆分
- 新增页面：
  - `src/pages/LobbyPage.vue`
  - `src/pages/RoomPage.vue`
  - `src/pages/AdminPage.vue`
- `src/App.vue` 按 `pathname` 分发页面：
  - `/` -> 大厅
  - `/room/:code` -> 对战
  - `/admin` -> 后台

2. 前端通信层重写
- 新增 `src/composables/useGameClient.js`：统一管理 WebSocket 与后台 HTTP API。
- WebSocket 地址：`ws://localhost:11451/ws`。
- HTTP 地址：`http://localhost:11451`。

3. 后端重写为 HTTP + WS 一体服务
- `ylsg-server.js` 改为 `http.createServer + WebSocketServer(noServer)`。
- HTTP 接口：
  - `POST /api/admin/login`（后台密码登录）
  - `GET /api/admin/cards`
  - `POST /api/admin/cards`
  - `PUT /api/admin/cards/:id`
  - `DELETE /api/admin/cards/:id`
  - `GET /api/admin/games`
- 认证方式：`Bearer token`。
- 后台密码：环境变量 `YLSG_ADMIN_PASSWORD`，默认 `admin123`。

4. 卡牌持久化
- 新增持久化文件：`data/cards-db.json`。
- `img-url.md` 逻辑废弃，房间卡牌来源改为后台持久化数据。
- 当卡牌为空时，对战页显示“后台未添加卡牌，前端不显示卡牌”。

5. 对战布局与交互
- `RoomPage.vue` 实现：
  - 左侧手牌区
  - 中部 6 个虚线槽位
  - 拖拽放置与回收
  - 提交排序
  - 对手卡背显示
  - 复制房间链接按钮

6. UI 简化
- 重写 `src/style.css`：
  - 浅色简洁配色
  - 降低渐变与花哨效果
  - 保持现代化间距、圆角、边框、状态反馈

7. 依赖调整
- 一度引入 `vue-router` 方案，因当前构建链解析异常回退为无依赖路由分发（按 URL path 渲染页面）。
- 最终 `package.json` 保留：`vue`、`ws`。

### 构建与验证
- `npm install`：成功。
- `npm run build`：成功。
- `node -e "import('./ylsg-server.js')..."`：成功输出服务监听地址。

---

## 当前关键文件清单
- 后端：`ylsg-server.js`
- 持久化：`data/cards-db.json`
- 前端入口：`src/main.js`、`src/App.vue`
- 页面：
  - `src/pages/LobbyPage.vue`
  - `src/pages/RoomPage.vue`
  - `src/pages/AdminPage.vue`
- 组件：
  - `src/components/CardTile.vue`
  - `src/components/ToastOverlay.vue`
- 通信层：`src/composables/useGameClient.js`
- 样式：`src/style.css`
- 依赖脚本：`package.json`

## 运行方式
1. `npm run server`
2. `npm run dev`

## 后续承诺
- 后续每次修改我都会在本文件追加详细变更记录，不再只做口头说明。

## 追加记录（2026-05-10）：修复 cards-db.json BOM 导致的服务崩溃

### 问题现象
- 执行 `node ylsg-server.js` 后，访问读取卡牌数据时抛错：
- `SyntaxError: Unexpected token '﻿', "﻿{ ... is not valid JSON`。

### 根因
- `data/cards-db.json` 被写入了 UTF-8 BOM 头，`JSON.parse` 直接解析时失败。

### 本次处理
1. 修复现有数据文件编码
- 将 `data/cards-db.json` 重写为 UTF-8 无 BOM。

2. 增加后端容错
- 修改 `ylsg-server.js` 的 `readDb()`：
- 读取后先执行 `replace(/^\uFEFF/, '')` 去除 BOM，再做 `JSON.parse`。

### 修改文件
- `data/cards-db.json`
- `ylsg-server.js`

### 结果
- 后续即使 JSON 文件再次出现 BOM，后端也能正常解析，不会再因为该问题崩溃。

## 追加记录（2026-05-10）：后台权限、交互反馈、大厅房间列表、昵称与节奏配置升级

### 目标
- 修复后台“添加卡牌未授权”问题。
- 提供按钮动效和失败提示框。
- 后台改为左侧导航大栏，支持多页面信息管理。
- 后端操作统一持久化 JSON。
- 加入者可设置昵称。
- 大厅显示等待中的缺人房间并可点击进入。
- 增加“对局节奏”后台可配置页面。

### 关键改动
1. 前端鉴权与错误提示修复
- 文件：`src/composables/useGameClient.js`
- `api()` 统一解析错误，401 时清除本地旧 token 并提示重新登录。
- 增加全局 `notice`（success/error）用于上方提示框。
- 增加昵称状态：`nickname`，并在创建/加入房间时上送。

2. 大厅增强
- 文件：`src/pages/LobbyPage.vue`
- 新增昵称输入。
- 新增等待房间列表（`/api/lobby/rooms`）与一键进入。
- 新增操作反馈提示（创建成功、输入错误等）。

3. 对战页增强
- 文件：`src/pages/RoomPage.vue`
- 增加操作提示：复制链接成功/失败、提交排序成功。
- 保留左侧手牌 + 6 槽位拖拽布局。

4. 后台重构为左侧导航信息架构
- 文件：`src/pages/AdminPage.vue`
- 左侧模块：`所有卡牌`、`添加卡牌`、`对局信息`、`对局节奏`、`修改密码`。
- 登录后加载卡牌、对局、节奏配置。
- 支持卡牌查看/编辑/删除、节奏保存、密码修改。

5. 后端接口与持久化升级
- 文件：`ylsg-server.js`
- 新增 JSON 持久化文件：`data/system-db.json`。
- 新增接口：
  - `GET /api/lobby/rooms`（大厅待加入房间）
  - `GET /api/admin/settings`
  - `PUT /api/admin/settings`
  - `PUT /api/admin/password`
- 后台登录密码改为读取/写入 `system-db.json`（不再只靠内存或环境变量）。
- 房间状态增加 `hostNickname/guestNickname`，对局列表可展示昵称。
- 开局排序阶段时长改为读取后台配置 `settings.sortingSeconds`。

6. 样式与交互反馈
- 文件：`src/style.css`
- 增加按钮点击缩放动效和 hover 阴影反馈。
- 增加顶部 `notice` 成功/失败样式。
- 增加后台左侧导航和活动态样式。

### 新增/修改文件
- 修改：`src/composables/useGameClient.js`
- 修改：`src/pages/LobbyPage.vue`
- 修改：`src/pages/RoomPage.vue`
- 修改：`src/pages/AdminPage.vue`
- 修改：`ylsg-server.js`
- 修改：`src/style.css`
- 新增：`data/system-db.json`

### 验证
- `npm run build`：通过。
- `node ylsg-server.js` 启动检查：通过。

### 说明
- “未授权”现象已通过前端 401 处理与重新登录提示修复。
- 由于服务重启会清空内存 token，需重新登录后台，这是预期行为。

## 追加记录（2026-05-10）：对战入口身份弹层、前线/备战位修正、手牌堆叠、卡牌新字段

### 本次目标
- 修复 URL 直进对战未提示输入ID/昵称。
- 战场槽位改为“前线5 + 备战1（后位）”。
- 手牌改为堆叠露出约三分之一，战场占满浏览器窗口，其他UI悬浮覆盖。
- 后台卡牌新增：简介、血量、攻击，并持久化。

### 主要修改
1. 对战页身份确认
- 文件：`src/pages/RoomPage.vue`
- 新增进入遮罩层 `identity-mask`。
- 必填 `用户ID` 和 `昵称` 后才会调用 `attachRoom(code)` 加入房间。

2. 战场布局调整
- 文件：`src/pages/RoomPage.vue`
- 槽位改为：
  - `frontline-row`：5个前线槽位（索引0~4）
  - `reserve-row`：1个备战槽位（索引5）
- 手牌区保留拖拽到槽位与回收逻辑。

3. 全屏战场 + 手牌堆叠
- 文件：`src/style.css`
- 新增 `battle-page/full-board/battle-overlay` 全屏结构。
- 手牌区改 `stacked-cards` 绝对定位堆叠，每张偏移固定距离，露出上方部分。
- 其他元素（标题/提示/计时器）作为悬浮层叠加。

4. 卡牌字段扩展
- 前端后台录入：`src/pages/AdminPage.vue`
  - 新增输入：`desc`（简介）、`hp`（血量）、`atk`（攻击）
  - 列表显示攻血摘要。
- 卡牌展示：`src/components/CardTile.vue`
  - 元信息显示 `atk/hp`。
- 后端持久化：`ylsg-server.js`
  - `POST/PUT /api/admin/cards` 写入新字段。
  - `buildCards()` 同步返回 `desc/hp/atk`。

### 修改文件
- `src/pages/RoomPage.vue`
- `src/components/CardTile.vue`
- `src/pages/AdminPage.vue`
- `src/style.css`
- `ylsg-server.js`

### 验证
- `npm run build`：通过。
- `node ylsg-server.js` 启动检查：通过。

## 追加记录（2026-05-10）：对战对称布局、强同步、投降/强制结束、断联重连、用户注册登录

### 修复与新增目标
- 修复敌我布局不同步与不对称。
- 统一双方卡牌尺寸，敌方也按前线/备战布局显示。
- 手牌堆叠显示优化（露出约1/2，最后一张完整显示）。
- 手牌 hover 小幅上浮动画。
- 修复房间号与阶段重叠。
- 后端强同步双方对局展示数据。
- 新增投降结束对局。
- 后台新增强制结束对局。
- 新增断联180秒重连机制，超时自动结束。
- 新增主页重连提示。
- 新增注册/登录机制，用户数据持久化 JSON，会话令牌写入 Cookie/本地。
- 显示对手昵称。

### 后端改动（ylsg-server.js）
1. 用户系统
- 新增 `data/users-db.json` 持久化用户明文信息。
- 新增接口：
  - `POST /api/auth/register`
  - `POST /api/auth/login`
  - `GET /api/auth/me`
- 会话令牌使用 `sessionToken`，前端通过 `X-Session-Token` + Cookie 传递。

2. 重连机制
- 新增 `GET /api/lobby/reconnect`，返回当前用户可重连对局。
- WS 断联检测：玩家断联后进入 180s 宽限期。
- 若超时未重连：自动结束对局（`disconnect_timeout`）。

3. 对局强同步
- 玩家布局从原 `order` 升级为结构化 `layout`：
  - `frontline[5]`
  - `reserve`
  - `hand[]`
- 新增 WS 事件：`update_layout`（拖拽实时同步）
- `submit_order` 改为提交整套 `layout`。

4. 对局结束控制
- 新增 WS 事件：`surrender`（玩家投降）。
- 新增后台接口：`PUT /api/admin/games/:code/end`（强制结束对局）。

5. 卡牌字段扩展（持久化）
- `desc`、`hp`、`atk` 在 `POST/PUT /api/admin/cards` 中持久化并同步到对局卡牌。

### 前端改动
1. 通信层（src/composables/useGameClient.js）
- 加入会话恢复、注册/登录 API 封装。
- 支持 `currentUser`、`sessionToken`、重连查询。
- WS 消息携带 `sessionToken`。
- 新增 `updateLayout`、`surrender`。

2. 大厅页（src/pages/LobbyPage.vue）
- 新增注册/登录切换表单。
- 登录后才能创建房间。
- 新增“可重连对局”提示区块，支持一键重连。

3. 对战页（src/pages/RoomPage.vue）
- 敌我布局改为轴对称：
  - 敌方前线5 + 敌方备战1
  - 我方前线5 + 我方备战1
- 敌方卡背尺寸与我方一致。
- 手牌区堆叠展示，最后一张完整显示。
- 手牌 hover 上浮动效。
- 头部信息拆分，避免房间号与阶段重叠。
- 新增投降按钮。
- 新增显示对手昵称。
- 拖拽后实时调用 `update_layout` 强制同步。

4. 后台页（src/pages/AdminPage.vue）
- 对局信息页新增“强制结束”按钮，调用 `PUT /api/admin/games/:code/end`。
- 卡牌表单/编辑增加 `简介/血量/攻击` 字段。

5. 组件与样式
- `src/components/CardTile.vue`：信息区改为显示攻血。
- `src/style.css`：全屏战场、对称行布局、hover 动画、重叠修复、按钮与提示样式更新。

### 数据文件
- 新增：`data/users-db.json`（首次运行自动创建）
- 继续使用：`data/cards-db.json`、`data/system-db.json`

### 验证
- `npm run build`：通过。
- `node ylsg-server.js` 启动：通过。

## 追加记录（2026-05-10）：修复“已登录仍提示先登录”与“无法修改个人信息”

### 问题1：登录后创建房间进入对局仍提示先登录
- 根因：房间页挂载时仅在 `currentUser` 已就绪时才加入房间，未先执行会话恢复；页面刷新/跳转瞬间会出现空态。
- 处理：
  1. `RoomPage.vue` 在 `onMounted` 中先 `await restoreSession()`，成功后再 `attachRoom(code)`。
  2. `useGameClient.js` 初始化时直接尝试从 `localStorage.ylsg_user` 恢复 `currentUser`，减少闪断。

### 问题2：注册登录后无法修改个人信息
- 处理：新增完整的用户资料更新链路。

#### 后端新增接口
- `PUT /api/auth/profile`
- 支持：
  1. 修改昵称 `nickname`
  2. 修改密码（需提供 `currentPassword` + `newPassword`）
- 持久化：写入 `data/users-db.json`

#### 前端能力新增
- `useGameClient.js` 新增 `updateProfile(payload)`。
- `LobbyPage.vue` 新增“个人信息”区域：
  1. 修改昵称
  2. 修改密码
  3. 提交后顶部提示成功/失败

### 编码修复
- 同步重写 `RoomPage.vue`，修复上一版个别中文乱码文案。

### 修改文件
- `src/composables/useGameClient.js`
- `src/pages/RoomPage.vue`
- `src/pages/LobbyPage.vue`
- `ylsg-server.js`

### 验证
- `npm run build`：通过。
- `node ylsg-server.js` 二次启动检测时提示 `EADDRINUSE:11451`（端口已被现有服务占用，属于预期环境状态，不是代码错误）。

## 追加记录（2026-05-10）：战场轴对称修正与尺寸压缩

### 本次问题修复
1. 敌我布局轴对称修正
- 我方保持：前线5在上、后线1在下。
- 敌方改为：后线1在上、前线5在下（对称）。

2. 卡牌尺寸与对齐
- 前后线卡槽统一尺寸：同宽同高。
- 后线槽位宽度改为与前线一致，不再偏大。
- 后线自动居中，对齐前线中位。

3. 前线紧凑排布
- 前线5个卡槽缩小并减小间距，避免超出屏幕。
- 移动端同步缩小。

4. 放牌阶段结束隐藏手牌区
- 新增 `showHand` 逻辑：仅 `sorting` 阶段显示左侧手牌区。
- 排序结束后自动隐藏手牌区，战场扩展为全宽对局视图。

5. 文案乱码修复
- 重新整理 `RoomPage.vue` 文案，修复乱码文本。

### 修改文件
- `src/pages/RoomPage.vue`
- `src/style.css`

### 验证
- `npm run build`：通过。

## 追加记录（2026-05-10）：排序按钮移除、卡背展示优化、结束返厅、观战限制

### 本次需求处理
1. 移除“提交排序”按钮
- 对局排序改为拖拽即实时同步（`update_layout`），不再需要手动提交按钮。
- 房间页已删除该按钮与对应调用。

2. 卡背只显示图片
- 卡背图仅展示图片，不再显示“卡背”文字。
- 卡背 URL 改为：`https://img.xscnet.cn//i/2026/05/10/6a007027baddc.jpg`

3. 己方卡牌显示信息 + 简介区域
- 己方卡牌保留名称与攻血。
- 在卡牌下部新增简介区域（`desc`），用于显示卡片介绍。

4. 投降后双方返回大厅
- 房间页监听 `room.phase === 'ended'`，触发提示后 1.5s 自动跳回大厅。

5. 对局结束后双方返回大厅
- 与投降共用结束逻辑：任何结束状态（包括后台强制结束、断联超时）都会自动返厅。

6. 观战者限制 + 右上返回大厅
- 对局开始后新进入者为 `viewer` 角色，仅观战。
- 观战者无法操作任何拖拽/投降/开始等按钮。
- 观战者右上角显示“返回大厅”按钮。

### 修改文件
- `src/pages/RoomPage.vue`
- `src/components/CardTile.vue`
- `src/style.css`

### 验证
- `npm run build`：通过。

## 追加记录（2026-05-10）：卡背留白与计时器实时刷新修复

### 修复1：卡背底部留白
- 原因：隐藏卡背场景仍按普通卡片比例渲染，容器高度变化时图片未强制铺满。
- 处理：新增 `.slot-card.hidden img` 规则，设置 `height:100%` + `object-fit:cover` + `aspect-ratio:auto`。
- 结果：卡背图片填满整个卡槽，不再出现底部留白。

### 修复2：计时器不实时更新
- 原因：`computed` 内部直接使用 `Date.now()`，没有响应式依赖，UI不会每秒重算。
- 处理：在 `RoomPage.vue` 增加 `nowTick` 响应式时间戳，并用 `setInterval` 每秒刷新；组件卸载时清理定时器。
- 结果：倒计时实现实时跳秒更新。

### 修改文件
- `src/pages/RoomPage.vue`
- `src/style.css`

### 验证
- `npm run build`：通过。

## 追加记录（2026-05-10）：卡背铺满与实时计时器修复

### 修复项
1. 卡背底部留白
- `CardTile.vue` 隐藏态仅渲染图片。
- `style.css` 中为 `.slot-card.hidden img` 增加铺满规则（`height:100%`, `aspect-ratio:auto`, `object-fit:cover`）。
- 结果：卡背图完整填满卡槽，无底部空白。

2. 计时器不实时更新
- `RoomPage.vue` 增加 `nowTick` 响应式时间戳和 `setInterval` 每秒更新。
- 计时器改为基于 `sortingEndsAt - nowTick` 计算。
- 在 `onUnmounted` 清理定时器，避免泄漏。
- 结果：倒计时每秒跳动更新。

### 修改文件
- `src/pages/RoomPage.vue`
- `src/style.css`
- `src/components/CardTile.vue`

### 验证
- `npm run build`：通过。

## 追加记录（2026-05-10）：全站背景替换与开局后隐藏复制链接

### 需求变更
1. 全站背景替换为可无缝拼接图：
- `https://img.xscnet.cn//i/2026/05/10/6a007c7a5ed58.jpg`
- 图片尺寸 `1244x944`。

2. 对局开局后右上角“复制链接”去掉。

### 实现
- `src/style.css`
  - `body` 背景改为指定 URL。
  - 设置 `background-repeat: repeat` 与 `background-size: 1244px 944px`，保证平铺。
  - 移除 `battle-page` 额外渐变背景，让全站统一使用该背景图。

- `src/pages/RoomPage.vue`
  - 复制链接按钮改为仅 `room.phase === 'lobby'` 时显示。
  - 开局后（进入 sorting/playing）自动不显示。

### 修改文件
- `src/style.css`
- `src/pages/RoomPage.vue`

### 验证
- `npm run build`：通过。

## 追加记录（2026-05-10）：排序结束自动补位规则

### 需求
- 放牌计时结束后，若手牌区仍有卡牌，自动随机放入己方空槽位。

### 实现
- 在后端 `ylsg-server.js` 新增：
  - `shuffleInPlace(arr)`：原地随机打乱。
  - `autoPlaceRemainingCards(layout)`：
    1. 收集空槽（前线5 + 后线1）。
    2. 将手牌与空槽分别随机打乱。
    3. 按最小数量逐一放入。
    4. 未放下的手牌保留在 `layout.hand`。
- 在排序阶段倒计时结束回调中，对每位玩家执行自动补位，再切换到 `playing` 并广播同步。

### 修改文件
- `ylsg-server.js`

### 验证
- `npm run build`：通过。
- 服务启动检测：通过。
