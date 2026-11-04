# CarLife 互联投屏原型 · vivo × 红旗 HS5 23款

高保真**交互仿真原型**：左侧为 vivo 手机上的 CarLife App（投屏源），右侧为红旗 HS5 2023 车机的 12.3 英寸双联屏 + HUD 抬头显示（投屏目标），双屏状态**实时联动**，可交互驾驶。

> ⚠️ 说明：本原型用于演示与评审，**不接入百度 CarLife 私有协议**。真正投屏到实车需要百度官方 CarLife SDK / 厂商白名单，或协议逆向（存在合规风险）。若要落地真机，见文末「后续路径」。

> 📌 **架构定位**：手机端是**数据源 / 控制器**（CarLife App 提供导航路线、媒体、车况信号，并显示"正在投射"状态）；车机端**原生渲染**仪表盘与 HUD，由 CarLife 数据通道驱动，**不是手机屏幕镜像**。两端共享同一份车况/导航状态，因此显示一致。

## 已实现功能

| 模块 | 内容 |
|------|------|
| 数字仪表盘 | 数字车速、转速表（SVG 指针 + 刻度）、油量弧、续航里程、挡位、总里程、水温 |
| HUD 抬头显示 | 风挡投影风格：车速 + 转向箭头 + 道路名 + 距转向距离 |
| 导航投射 | 中央屏 SVG 地图、车标沿路线移动、转向提示横幅（左/右/直行/掉头/到达） |
| 音乐 | 播放/暂停、进度条、曲目切换（多首列表） |
| 电话 | 模拟来电 → 接听 / 挂断、通话计时 |
| 语音助手 | 唤醒态动画 + 指令示例 |
| 双屏联动 | 手机端操作实时镜像到车机端，反之亦然 |
| 驾驶仿真 | 油门/滑行/刹车/城市/郊区/高速档位联动转速与油耗 |

## 目录结构

```
carlife-hongqi-hs5/
├── index.html            # 页面结构（手机 + 车机双屏 + 控制面板）· 部署版
├── styles.css            # 红旗红 + 科技蓝主题、双联屏与 HUD 样式
├── app.js                # 仿真核心：动力学 + 双屏渲染 + 交互
├── carlife-hs5-app.html  # ★ 单文件 APP：CSS/JS 全内联，传手机即用（推荐）
├── manifest.webmanifest  # PWA 安装清单
├── sw.js                 # Service Worker，离线缓存
├── icon-192.png          # 应用图标 192
├── icon-512.png          # 应用图标 512
├── icon-maskable-512.png # 自适应图标（PWA 掩码）
└── README.md
```

## 如何运行

方式一（推荐，避免字体跨域）：用本地服务器打开

```bash
cd carlife-hongqi-hs5
python3 -m http.server 8123
# 浏览器访问 http://localhost:8123
```

方式二：直接双击 `index.html` 用浏览器打开即可（纯前端，无构建步骤）。

## 打包成 APP（手机上试用）

> 沙箱无 Android 编译环境（整套 Android SDK 上 GB），无法在此出真机 APK。
> 改用「单文件 HTML + PWA」方案，在 vivo 上即可当 APP 安装、离线运行。

### A. 单文件 APP（最快：传手机即用）
`carlife-hs5-app.html` 已把 CSS/JS 全部内联、**零外部依赖**，可直接用于手机：

1. 把 `carlife-hs5-app.html` 传到 vivo（微信文件传输/USB/云盘均可）。
2. 用 **Chrome** 打开该文件 → 右上角 `⋮` → **「添加到主屏幕」**。
3. 桌面即生成「HS5 投屏」图标，点开就是全屏 APP，**不需要网络、不需要服务器**。
   - 注意：用系统「文件管理」直接打开可能被浏览器限制；若打不开，先把文件发到微信/QQ 里再点开，或用 Chrome 下载到本地后打开。

### B. PWA 安装版（像原生 APP：可安装 + 离线）
把整个文件夹部署到任意 **https** 地址（GitHub Pages / Vercel / 自有服务器），手机 Chrome 访问后：
1. 地址栏出现「安装」图标，或 `⋮` → **「安装应用」**。
2. 安装后完全离线可用（`sw.js` 已缓存全部资源），图标与名称来自 `manifest.webmanifest`。

```bash
# 本地起服务预览 PWA（安装需 https 或 localhost）
cd carlife-hongqi-hs5 && python3 -m http.server 8123
```

### C. 真机 Android APK（如需）
若一定要 `.apk`，可在你本机用 Capacitor 把本目录打包：

```bash
npm i -g @capacitor/cli && npm init -y
npm i @capacitor/core @capacitor/android
npx cap init HS5App com.example.hs5app --web-dir .
npx cap add android && npx cap sync && npx cap open android   # 用 Android Studio 出包
```
（需本机具备 Android SDK / Android Studio；此沙箱环境无法完成该步骤。）

## 交互说明

- **底部控制面板 · 驾驶模拟**：刹车 / 滑行 / 40 / 80 / 120 设定目标车速；可暂停/继续仿真、重置。
- **底部控制面板 · 互联功能**：播放暂停音乐、模拟来电、语音助手、重算导航。
- **手机端 / 车机端按钮**：导航、音乐、电话、语音、主页，点击即联动双屏高亮。
- 车速会自动趋近目标并联动转速/挡位；油耗随里程下降；导航里程递减并触发下一个转向动作。

## 技术要点

- 纯前端（HTML + CSS + 原生 JS），`requestAnimationFrame` 仿真循环，`dt` 限幅防跳变。
- 仪表指针/弧线用 SVG `stroke-dasharray` + `transform: rotate` 实现。
- 地图车标用 `path.getPointAtLength()` 沿路线平滑移动。
- 电话/语音卡片仅在状态变化时重建 DOM，避免每帧 `innerHTML` 抖动。

## 后续路径（接真实车机）

1. **官方通道**：向百度 Apollo/CarLife 开放平台申请车机端 SDK，vivo 侧按 CarLife 手机端规范实现投屏源，走厂商白名单联调。
2. **协议层**：实现 CarLife 的 USB/Wi-Fi 发现、认证、音视频与控件通道（HU 与 Phone 双向指令），替换本原型的 `state` 数据源为真实协议事件。
3. **HUD 真渲染**：将 HUD 叠加层改为车机 OEM 提供的 HUD 合成接口或 AR-HUD 投影参数。
4. **合规**：涉及车辆控制/ADAS 信息投射需符合 GB/T 与车企信息安全要求。

> 本目录代码仅用于原型演示与交互评审，不含任何私有协议实现。
