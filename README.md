# NEXA

**跨平台代理客户端**(Windows / macOS,机场面板配套),Go + Wails v2,同一代码库按平台编译。
UI 为定稿的「极光夜」玻璃拟态设计:深蓝底 + 蓝/紫/青极光渐变,内置日/夜双主题与中/英界面语言(均持久化)。

![icon](icon.png)

## 界面预览

| 首页(夜间) | 首页(日间) |
| --- | --- |
| ![home-dark](images/home-dark.png) | ![home-light](images/home-light.png) |

| 节点 | 策略组 |
| --- | --- |
| ![nodes-dark](images/nodes-dark.png) | ![groups-dark](images/groups-dark.png) |

## 下载(Windows / macOS 双版本)

| 平台 | 当前版号 | 下载 | 安装方式 |
| --- | --- | --- | --- |
| **Windows** 10(1803+)/ 11 | 2.x | 到 [Releases 页](https://github.com/kungec/NEXA/releases) 选 **v2.x** 版本,下载 `NEXA-*-installer.exe` | 双击安装(内置 WebView2 检测) |
| **macOS** 12+(Apple Silicon) | 1.x | 到 [Releases 页](https://github.com/kungec/NEXA/releases) 选 **mac-v1.x** 版本,下载 `NEXA-darwin-*.zip` | 解压后将 NEXA.app 拖入「应用程序」 |

> **双版号线说明**:Windows 与 macOS 版号独立演进(Windows 2.x / macOS 1.x),更新清单分离(`latest.json` / `latest-mac.json`),互不干扰、不会撞库。应用内「检查更新」会自动拉取对应平台的最新版并完成更新。
>
> mac 安装包未做公证,首次打开如被 Gatekeeper 拦截:右键 NEXA.app → 打开,或到 系统设置 → 隐私与安全性 → 仍要打开。

## 构建与运行

```bash
wails build                  # Windows 产物: build/bin/NEXA.exe
wails build -platform darwin/arm64   # macOS 产物: build/bin/NEXA.app
build/darwin/package.sh      # macOS:构建 + zip(加 -dmg 出 DMG,-universal 出通用二进制)
wails build -debug           # 带 devtools 的调试构建
wails dev                    # 开发热重载
go test ./...                # 单元/集成测试(parser/app/qrscan)
```

环境要求:Go 1.25+、Node 24+;Windows 需 WebView2 Runtime,macOS 需 Xcode Command Line Tools(cgo 托盘/截图)。国内网络需 `go env -w GOPROXY=https://goproxy.cn,direct`;frontend/.npmrc 已指向 npmmirror。

**发版**:

```bash
# Windows:沿用既有 NSIS 管线(publish/ 目录),清单 latest.json
# macOS:构建 + 签名 + Release + 推送 latest-mac.json 一条龙
./publish/publish-mac.sh 1.0.1        # 版号需与 app_darwin.go 的 appVersion 一致
```

## 系统要求与兼容性

| 系统 | 支持情况 |
| --- | --- |
| Windows 11 | ✅ 完整支持(开发与测试基准) |
| Windows 10(1803+) | ✅ 完整支持;高 DPI 按显示器缩放自适应 |
| Windows 8.1 / 7 | ❌ 不支持(启动时弹窗提示);WebView2 与 Go 工具链均已终止对旧系统的支持 |
| macOS 12+ | ✅ 支持(arm64 / amd64;开发测试基准 macOS 27 arm64);窗口钮为系统红绿灯,托盘为菜单栏图标 |
| macOS 11- | ❌ 未测试(WKWebView 底线 11,不建议) |

说明:TUN 模式在 Windows 需管理员运行;在 macOS 连接时经系统弹窗授权一次、内核单独以 root 运行(断开静默,无需重复输密码)。内核(sing-box)按平台自动下载 windows-amd64 / darwin-arm64 / darwin-amd64;实时速率与策略组走本机回环 + 随机密钥鉴权,与其他代理工具端口冲突时自动顺延。

## 各平台差异说明

### Windows

- 系统代理:WinINET 接管;开机自启:HKCU Run 键
- TUN:以管理员身份运行 NEXA 后开启
- 关闭窗口默认驻留右下角托盘(设置可关)

### macOS

- **权限**:「屏幕识别二维码」需 系统设置 → 隐私与安全性 → 屏幕录制 授权;「相机扫码」需 相机 授权;TUN 连接时弹窗输入管理员密码(仅一次/每次连接)。
- **开机自启**:LaunchAgent(`~/Library/LaunchAgents/com.nexa.NEXA.plist`),登录后配合「静默启动」驻留菜单栏。
- **系统代理**:networksetup 对全部网络服务设置 Web/Secure 代理,绕过域名与 Windows 版一致(含 QQ/微信本地回环桥)。
- **关闭语义**:点红色关闭钮 = 驻留菜单栏(设置「关闭时最小化到菜单栏」);退出走菜单栏图标菜单、顶部菜单栏 退出 或 ⌘Q。
- **顶部菜单栏**:中文应用菜单(关于/退出/窗口/帮助),跟随界面语言。

### 共同的自动更新

应用内「设置 → 检查更新」自动完成下载、校验(ed25519 签名 + SHA256)与换装重启。清单按平台分离:Windows `latest.json` / macOS `latest-mac.json`。

## 功能总览

| 模块 | 说明 |
| --- | --- |
| 系统托盘 | 图标双态(连接=彩色 / 未连接=灰色):Windows 为右下角托盘,macOS 为菜单栏图标;菜单:打开/连接断开/开机自启/规则模式/全局模式/退出;关闭窗口可驻留(设置可关) |
| 导入中心 | 分享链接(vless/vmess/trojan/ss/hysteria2)解析回填、粘贴导入、保存前 TCP 测试连接、**屏幕识别二维码**(多屏+多码,优先代理链接)、**相机扫码**(getUserMedia + jsQR) |
| 订阅 | Clash YAML / V2Ray base64 自动识别,流量与到期显示(subscription-userinfo),**二维码弹窗**(手机扫码用)、**编辑(改名/换链)**、两步确认删除 |
| 定时任务 | 订阅按周期自动更新(1-24h 可调)+ 更新后自动测速 |
| 连接 | sing-box 内核管理、系统代理接管/还原(Windows: WinINET;macOS: networksetup)、混合端口、允许 LAN、TUN 模式(Windows 需管理员运行;macOS 连接时弹窗授权内核) |
| 节点/策略组 | 地区识别 + SVG 国旗、收藏、搜索/筛选/排序;Clash API 实时切换 selector,真实 HTTP 延迟测速(并发 6,未连接时 TCP 握手兜底) |
| 实时状态 | 顶栏与首页实时上下行速率(内核 /traffic WebSocket)+ 速率曲线 |
| 内核管理 | **一键自动下载**(GitHub 多镜像回退 + SHA256 校验 + 进度事件,按平台取 windows-amd64.zip / darwin-*.tar.gz),版本识别,运行中换内核自动重启 |
| 诊断 | 网络/系统代理/TUN/DNS/DoH 泄漏/内核,逐项实时上报 |
| 自动更新 | 应用内检查更新:ed25519 签名校验 + SHA256 完整性 + 自动换装重启(清单按平台分离) |
| 其他 | 日志实时滚动/分类过滤/导出、中英 i18n(菜单栏同步)、静默启动、开机自启(Windows: HKCU Run;macOS: LaunchAgent) |

## 目录结构

```
main.go                  Wails 入口(main_windows/main_darwin 承载平台窗口选项与前置)
app.go                   App 主体:前端绑定方法 + 事件 + 托盘/调度器
app_windows/app_darwin   平台实现:自启、dpi、诊断文案、更新换装
app_menu.go              应用菜单(macOS 顶部菜单栏,中文,跟随界面语言)
internal/model/          数据模型
internal/parser/         分享链接与订阅解析(Clash YAML / base64,导入期字段强校验)
internal/config/         JSON 配置存储(%AppData% / ~/Library/Application Support,原子写)
internal/core/           sing-box 进程管理、配置生成、Clash API、内核下载(含 darwin TUN 提权)
internal/sysproxy/       系统代理(sysproxy_windows: WinINET / sysproxy_darwin: networksetup)
internal/qrscan/         屏幕截图 + gozxing 多策略二维码解码(darwin 含录屏权限预检)
internal/tray/           系统托盘(手写实现:win32 Shell_NotifyIcon / macOS NSStatusItem,双态 PNG)
internal/admin/          管理员/root 权限检测(TUN)
frontend/src/main.js     i18n + 状态渲染 + 绑定 + 相机扫码 + 实时曲线(平台词条自适应)
frontend/src/style.css   「极光夜」设计系统(含 body.mac 红绿灯适配)
build/darwin/            Info.plist、打包脚本 package.sh
publish/                 发布管线(Windows: NSIS/latest.json;macOS: publish-mac.sh/latest-mac.json)
tools/icopack            PNG→ICO 打包工具(托盘图标再生成用)
tools/qrprobe,coreprobe  扫码/内核管线诊断探针
tools/traymacprobe       darwin 托盘独立探针
tools/signlatest         更新清单 ed25519 签名工具
tools/e2e/               UIA 端到端脚本(注意 .ps1 必须带 UTF-8 BOM)
```

## 使用

1. 首次运行:设置 → 内核 → 「下载内核」(自动下载对应平台的 sing-box,已内置多镜像回退);或手动放数据目录 `core/bin/` 下
2. 导入中心:添加订阅 / 粘贴分享链接 / 屏幕识别二维码
3. 首页点电源连接;关闭窗口默认驻留托盘/菜单栏,托盘菜单可快速连接/断开/退出
4. TUN 模式:Windows 以管理员身份运行 NEXA 后再开启;macOS 直接开启,连接时在弹窗输入管理员密码

## 里程碑(未完成)

- 相机扫码首次使用需允许摄像头权限;macOS 相机扫码依赖 WKWebView 的 getUserMedia 实现
- 诊断页与部分运行态文案的 i18n key 化(当前后端下发中文);规则集 .srs 下载增加 SHA256 锚定;订阅地址 SSRF 面收敛
- 多窗口(独立区域选择扫码)、深链协议回调(nexa://)
