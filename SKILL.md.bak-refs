---
name: harmonyos-app-development
description: 鸿蒙单框架（HarmonyOS NEXT）APP开发全流程——DevEco Studio环境搭建、ArkTS严格模式编码规范、签名证书管理、网络请求优化（rcp）、真机调试与部署。适用Hermes Agent移动客户端鸿蒙版本。
version: 2.8.0
tags: [harmonyos, arktS, mobile, hap, hvigor]
related_skills: [task-delivery-workflow, android-app-development, hermes-mobile-app]
---

# HarmonyOS NEXT APP 开发全流程

鸿蒙单框架（HarmonyOS NEXT，无AOSP兼容层）APP开发，从环境搭建、ArkTS编码到签名部署的完整流水线。

### 📚 相关支持文档
- [鸿蒙 NEXT 状态管理与下拉刷新架构最佳实践](references/state_management_and_refresh_patterns.md) — 针对多页面状态联动同步、AppStorage 响应式数据流、下拉刷新组件防漏及本地 preferences 离线缓存保底的最佳实践沉淀。
- [鸿蒙 APP 代码审查与功能测试方法论](references/code_review_methodology.md) — 系统性审查已有鸿蒙APP代码的流程：数据流追踪 → 响应式链路验证 → 自动刷新生命周期审计 → 预测/积分结算逻辑验证 → 结构化测试报告生成。
- [WebSocket 断线重连与流式消息优化模式](references/websocket_reconnect_patterns.md) — 小Q APP聊天实战沉淀：断线分阶段消息策略（温和占位→超时兜底）、sync_pending精确标签替换（禁用pop）、定时器全生命周期接管。

---

## ⚠️ 首要原则：先跑通Hello World，再加业务逻辑

**铁律**：不要一上来就写业务代码。先验证工具链：

```
① 创建最小Empty Ability项目 → ② 配置签名 → ③ 编译推手机 → ④ 确认Hello World显示
→ ⑤ 再添加ChatPage/HermesApi等业务代码
```

用户原话纠正过：「现在是在写Hello Word程序吗，还是在把安卓的小Q APP移植到鸿蒙单框架？」。

---

## 环境差异：Android vs HarmonyOS NEXT

| 维度 | Android | HarmonyOS NEXT |
|------|---------|----------------|
| IDE | Android Studio (Win/Mac/Linux) | **DevEco Studio (仅Win/Mac)** |
| SDK管理 | sdkmanager CLI | DevEco Studio GUI内安装（SDK已捆绑） |
| 构建工具 | Gradle | **hvigor**（Hvigor Wrapper: `hvigorw`） |
| 包格式 | APK | **HAP**（HarmonyOS Ability Package） |
| 签名 | .keystore / .jks | **.p12 + .cer + .p7b**（三方分离） |
| 调试 | adb | **hdc**（HarmonyOS Device Connector） |
| 编程语言 | Kotlin/Java + XML/Compose | **ArkTS**（TypeScript超集，严格模式） |
| UI框架 | Jetpack Compose / XML | **ArkUI**（声明式，类似Compose） |
| 真机连接 | USB/WiFi | USB/WiFi，开发者模式开启方式相同 |
| 项目结构 | Gradle多模块 | **hvigor + oh-package**多模块 |
| 包管理 | Gradle依赖 | **ohpm**（OpenHarmony包管理器） |

**核心差异**：DevEco Studio只有Windows/Mac版。代码编辑可以从WSL写（文件存在Windows分区），编译部署通过 **PowerShell委托** 从WSL调用Windows工具链（详见下方「没有deveco CLI」章节）。完整链路已验证通过：`WSL patch → powershell hvigorw → hdc install`。

---

## 🚫 没有"deveco" CLI

⚠️ 常有安卓开发者问"能不能在终端跑 `deveco build` 或 `deveco run`"——**不存在这样的命令**。DevEco Studio是IntelliJ系IDE，没有像VS Code `code` 或 Android `adb` 那样的统一CLI。具体工具有各的：

| 你想要的功能 | 实际命令 | 位置 |
|------------|---------|------|
| 编译项目 | `hvigorw assembleDebug` / `hvigorw assembleRelease` | `tools/hvigor/bin/hvigorw(.bat)` |
| 连接设备/部署 | `hdc install` / `hdc list targets` | `sdk/.../toolchains/hdc.exe` |
| 包管理 | `ohpm install` | `tools/ohpm/bin/ohpm(.bat)` |
| 开IDE | `devecostudio64.exe <project-path>` | `bin/devecostudio64.exe` |
| 在现有IDE窗口里打开文件 | ❌ 不支持 | IDE没暴露此接口 |

**调这些工具必须走 PowerShell 委托**（鸿蒙工具链都是Windows PE格式，WSL不能直接exec）。已验证通过的完整链路（2026-06-15）：详见下方「编译→部署→验证全链路」章节。常用命令速查见 `references/hvigor-cli-commands.md`。

### WSL → PowerShell 调用模板（编译 → 部署 → 验证全链路）

**关键环境变量**（缺一不可，必须在**同一个 PowerShell 会话**中设置）：
#### 版本自动自增（从此忘记手工改号）

每次编译前自动检测源码变更（`.ets`、`app.json5`、`resources/`），有变更则自增 patch 版本：

```bash
# 项目根目录下
python3 auto_bump_version.py
# 正常跑，检测到源码变更才自增

python3 auto_bump_version.py --dry-run  # 预览
python3 auto_bump_version.py --force     # 强制自增（即使无变更）
```

脚本做什么：
- `versionCode` +10
- `versionName` patch +1（1.4.0 → 1.4.1）
- 同步更新 `MainPage.ets` 的 `APP_VERSION` 常量
- 无变更不触发，避免提交空版本号

已预装至项目根（`auto_bump_version.py`），技能模板 `templates/auto_bump_version.py`。

| 变量 | 值 | 用途 |
|------|-----|------|
| `NODE_HOME` | `D:\\Program Files\\Huawei\\DevEco Studio\\tools\\node` | hvigorw需要node，不设报`NODE_HOME is not set` |
| `DEVECO_SDK_HOME` | `D:\\Program Files\\Huawei\\DevEco Studio\\sdk` | 指向sdk根目录（含`default/`子目录） |
| `JAVA_HOME` | `D:\\Program Files\\Huawei\\DevEco Studio\\jbr` | 内嵌JBR JRE，`PackageHap`步骤需要 |
| `PATH` | `%NODE_HOME%;%JAVA_HOME%\\bin;...` | 让node和java命令都在PATH中 |

**容易踩的坑**：
- `DEVECO_SDK_HOME` 指向 `sdk/` 即包含 `default/openharmony/` 的目录，不是 `sdk/default/openharmony/`
- `JAVA_HOME` 要用 DevEco Studio 自带的 `jbr/`，不是系统安装的 JDK。**不设这变量，`PackageHap` 签名步骤会报 `spawn java ENOENT`**
- 环境变量必须在 **同一个 PowerShell 会话中设置**，不能跨 `&`
- 不要用 `cmd.exe` 委托——PowerShell 才能正确处理带空格的 Windows 长路径
- `cmd.exe /c "pushd D:\ && ..."` 是必要的——WSL的CWD是UNC路径，cmd.exe 不支持
- ⚠️ **PowerShell 执行策略**：从 WSL 调用 `.ps1` 脚本时，PowerShell 默认禁止执行未签名脚本。用 `-ExecutionPolicy Bypass` 绕过：`powershell.exe -ExecutionPolicy Bypass -File script.ps1`
- ⚠️ **PowerShell 内联 -Command 复杂脚本的陷阱**：用 `powershell.exe -Command '...'` 传复杂命令时，正则表达式和字符串转义极易出错（PowerShell 和 bash 的引号/转义规则互相干扰）。**优先写 .ps1 文件然后用 `-File` 执行**，而不是内联长命令。

**hvigorw 正确任务名**：
- `assembleHap` — 编译签名单模块HAP（单模块debug场景最简命令，22秒完成，已验证）
- `assembleApp` — 编译 + 打包 + 签名整个应用（多模块、release发布时用）
- `tasks` — 列出可用任务（调试用）
- ❌ `assembleDebug`/`assembleRelease` 不是有效任务（HarmonyOS hvigor 与 Android Gradle 不同）
- ❌ `--mode=debug` 不是有效参数（报 `Unknown mode 'debug'`）。用 `-p buildMode=debug`

**一键编译部署脚本**：
推荐用 `build-deploy.ps1`（见 `references/build-deploy-pattern.md`）替代手写多行 PowerShell 命令。WSL 中一行搞定：
```bash
/mnt/c/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -ExecutionPolicy Bypass -File "D:\05_HarmonyNext\XiaoQ\build-deploy.ps1"
```
该脚本自动：同步源码 → 编译 HAP → 部署到模拟器 → 验证版本号，全部在 1 次工具调用内完成。

**hvigorw 正确任务名**：
- `assembleHap` — 编译签名单模块HAP（单模块debug场景最简命令，22秒完成，已验证）
- `assembleApp` — 编译 + 打包 + 签名整个应用（多模块、release发布时用）
- `tasks` — 列出可用任务（调试用）
- ❌ `assembleDebug`/`assembleRelease` 不是有效任务（HarmonyOS hvigor 与 Android Gradle 不同）
- ❌ `--mode=debug` 不是有效参数（报 `Unknown mode 'debug'`）。用 `-p buildMode=debug`

**hdc（设备连接 + 部署 + 启动 + 验证）**：
```powershell
# 编译（一行，从WSL调用）
powershell.exe -Command '$env:DEVECO_SDK_HOME="D:\Program Files\Huawei\DevEco Studio\sdk"; $env:JAVA_HOME="D:\Program Files\Huawei\DevEco Studio\jbr"; $env:PATH="$env:JAVA_HOME\bin;$env:PATH"; Set-Location D:\05_HarmonyNext\YourProject; & "D:\Program Files\Huawei\DevEco Studio\tools\node\node.exe" "D:\Program Files\Huawei\DevEco Studio\tools\hvigor\bin\hvigorw.js" assembleApp -p product=default -p buildMode=debug'

# 检查设备
powershell.exe -Command '& "D:\Program Files\Huawei\DevEco Studio\sdk\default\openharmony\toolchains\hdc.exe" list targets'

# 安装HAP
powershell.exe -Command '& "D:\Program Files\Huawei\DevEco Studio\sdk\default\openharmony\toolchains\hdc.exe" install -r "D:\05_HarmonyNext\YourProject\entry\build\default\outputs\default\entry-default-signed.hap"'

# 启动APP
powershell.exe -Command '& "D:\Program Files\Huawei\DevEco Studio\sdk\default\openharmony\toolchains\hdc.exe" shell "aa start -a EntryAbility -b com.your.bundle"'

# 验证APP进程
powershell.exe -Command '& "D:\Program Files\Huawei\DevEco Studio\sdk\default\openharmony\toolchains\hdc.exe" shell "bm dump -n com.your.bundle"'
powershell.exe -Command '& "D:\Program Files\Huawei\DevEco Studio\sdk\default\openharmony\toolchains\hdc.exe" shell "ps -ef | grep -i yourbundle"'
```

### 可复用一键脚本

| 脚本 | 位置 | 用途 | 从WSL调用 |
|------|------|------|-----------|
| `build_app.bat` | `templates/build_app.bat` | 编译 + 签名（参数化） | `cmd.exe /c "pushd D:\ && D:\path\to\build_app.bat D:\05_HarmonyNext\YourProject debug"` |
| `deploy_app.bat` | `templates/deploy_app.bat` | 部署 HAP 到设备 | `cmd.exe /c "pushd D:\ && D:\path\to\deploy_app.bat D:\05_HarmonyNext\YourProject"` |

两个模板都支持参数化项目路径，复制到项目目录后即可复用。详情见 `references/hvigor-cli-commands.md`。

## 工具链搭建

### 安装DevEco Studio + SDK

1. 从华为开发者官网下载DevEco Studio（Windows版）
2. 安装后首次启动自动弹出SDK配置向导
3. 勾选：HarmonyOS SDK + Node.js + ohpm
4. DevEco Studio 6.1.1及以上版本**SDK已内置**，无需单独下载
5. SDK路径默认：`C:\Users\<用户名>\DevEcoStudio\sdk` 或安装目录下

### 配置hdc（设备连接器）

hdc在DevEco Studio SDK安装目录下：
```
D:\Program Files\Huawei\DevEco Studio\sdk\default\openharmony\toolchains\hdc.exe
```

推荐加到Windows环境变量PATH中。验证：`hdc version`

### 配置hvigorw（CLI构建包装器）

hvigorw.bat在DevEco Studio安装目录下：
```
D:\Program Files\Huawei\DevEco Studio\tools\hvigor\bin\hvigorw.bat
```

**CLI编译前必须先把它复制到项目根目录**——手动创建的项目不自带hvigorw包装器：

```batch
copy "D:\Program Files\Huawei\DevEco Studio\tools\hvigor\bin\hvigorw.bat" D:\YourProject\hvigorw.bat
```

hvigorw.bat内容与项目无关，复制一次后每个项目都可用。也可以用全局版本直接调用：
```batch
"D:\Program Files\Huawei\DevEco Studio\tools\hvigor\bin\hvigorw.bat" --mode module -p module=entry@default -p product=default -p buildMode=debug assembleHap
```

同时确保 `NODE_HOME` 环境变量指向 DevEco Studio 自带 Node.js：
```
D:\Program Files\Huawei\DevEco Studio\tools\node\node.exe  (v18.20.1)
```
hvigorw 需要 node，不设会报 `NODE_HOME is not set and no 'node' command found in PATH`。

### 连接真机

1. 手机上：设置→关于手机→连续点击版本号7次（开开发者模式）
2. 设置→系统和更新→开发人员选项→USB调试→开启
3. USB线连接电脑，手机上点"允许"
4. 验证：`hdc list targets`（显示设备序列号即成功）

### 模拟器调试（推荐优先使用）

DevEco Studio内置华为模拟器管理器，支持以下设备类型（需通过IDE下载镜像）：
- **Pura 90** — 标准直板手机
- **Mate X7** — 折叠屏手机
- **MatePad Pro 13** — 平板
- **MateBook Pro** — 笔记本形态

启动模拟器后 `hdc list targets` 会显示设备序列号，与真机调试完全一致。

**注意**：模拟器镜像需在DevEco Studio的Device Manager中下载，首次下载约2-4GB。**鸿蒙NEXT没有x86模拟器（类似Android的AVD）**，只有基于ARM指令翻译的仿真器。

---

## 签名与证书管理

鸿蒙NEXT签名体系比Android复杂，需要4个文件：

### 签名文件清单

| 文件 | 用途 | 获取方式 |
|------|------|---------|
| `.p12` 密钥库 | 包含私钥，谁有这个谁就能代表你签名 | DevEco Studio: Build → Generate Key and CSR |
| `.csr` 证书请求 | 用于向华为申请正式证书 | 生成.p12时同步生成 |
| `.cer` 数字证书 | 华为认证开发者身份 | AppGallery Connect ← 上传.csr后下载 |
| `.p7b` Profile文件 | 绑定包名+证书+设备 | AppGallery Connect ← 选择应用+证书+设备后下载 |

### 生成密钥对

DevEco Studio菜单：`Build → Generate Key and CSR`

关键参数：
- **Key Store File**: 保存.p12的位置（如 `D:\harmony\project.p12`）
- **Password**: 记住，后续配置签名要用
- **Key Alias**: 如 `xiaoq`
- **Validity**: `10000`（天）
- **名/姓/组织单位/组织**: 填开发者信息
- **国家/地区**: `CN`

### 申请证书

1. 登录 [AppGallery Connect](https://developer.huawei.com/consumer/cn/)
2. **需要已实名认证的华为开发者账号，审核1-3天**
3. 我的项目 → 新建项目 → 添加应用（填应用名+包名）
4. 左侧菜单 → "HarmonyOS API" → "证书" → 新增证书
5. 类型选"调试证书"，上传.csr文件 → 下载.cer
6. "设备管理" → 注册设备（填设备名+UDID）

**获取UDID**：`hdc shell bm get --udid` 或手机拨号盘 `*#*#1357946#*#*`

### 申请Profile

1. "HarmonyOS API" → "Profile" → 新增Profile
2. 类型：调试Profile
3. 选择：应用 + 证书（.cer） + 设备
4. 下载.p7b文件

**发布证书**（上架用）：流程一样，选"发布证书"+ "发布Profile"。每个账号1个正式+2个调试证书。

### 发布上架 AppGallery 全流程

debug签名只能直装开发设备。**release签名的包不能通过hdc直装**（报 9568322），必须通过应用市场分发。

| 步骤 | 操作 | 说明 |
|------|------|------|
| ① 申请发布证书 | AGC → HarmonyOS API → 证书 → 新增 → 类型选「发布证书」 | 每账号限1个发布证书 |
| ② 创建发布Profile | AGC → HarmonyOS API → Profile → 新增 → 类型选「发布Profile」→ 选应用+发布证书 | **不需要绑定设备**（调试Profile才需要） |
| ③ 配置签名 | DevEco Studio → File → Project Structure → Signing Configs → 新增`release`方案 | p12同一个，cer选release的，profile选release的 |
| ④ Build APP(s) | Build → Build Hap(s)/APP(s) → Build APP(s) | 输出`.app`文件（不是`.hap`），路径在`entry/build/outputs/default/` |
| ⑤ 上传AGC | AGC → 我的项目 → 应用 → 版本管理 → 新建版本 → 上传`.app` |  |
| ⑥ 填写信息 | 应用名、分类、简介、截图(≥4张)、隐私政策URL、测试账号（如有登录功能） |  |
| ⑦ 提交审核 | 提交后1-3个工作日 | 审核通过自动上架 |

**⚠️ Build Mode下拉框找不到时**：直接用菜单 `Build → Build Hap(s)/APP(s) → Build APP(s)` 即可，签名方案由 `build-profile.json5` 中 `products[].signingConfig` 决定。不需要手动切换 Build Mode。

**⚠️ 命令行构建（hvigorw.bat）**：项目可能没有 `hvigorw.bat`（手动创建的项目不自带，只有DevEco Studio向导创建的才有）。没有时**不要试图用命令行构建**，直接在DevEco Studio IDE里用菜单构建。

### build-profile.json5 schema差异（根级 vs 模块级）

**根级** `build-profile.json5` 使用 `app` + `modules` 结构：

```json5
{
  "app": {
    "signingConfigs": [...],
    "products": [...],
    "buildModeSet": [
      { "name": "debug" },
      { "name": "release" }
    ]
  },
  "modules": [
    { "name": "entry", "srcPath": "./entry", "targets": [ ... ] }
  ]
}
```

**模块级** `entry/build-profile.json5` 的 schema **完全不同**，顶层只能用以下字段：
`apiType`, `targets`, `showInServiceCenter`, `buildOption`, `buildOptionSet`, `buildModeBinder`, `entryModules`

不能用 `module` 做包装！也不能在 targets[0] 中用 `applyToProducts`（targets[0] 只允许：`name`, `config`, `source`, `resource`, `runtimeOS`, `output`）。

正确写法：
```json5
{
  "apiType": "stageMode",
  "targets": [
    { "name": "default" }
  ]
}
```

没这个文件会报 `Can not find build config file build-profile.json5 at 'entry'`。

根目录 `build-profile.json5`（核心文件）：

```json5
{
  "app": {
    "signingConfigs": [
      {
        "name": "debug",
        "type": "HarmonyOS",
        "material": {
          "storeFile": "D:\\path\\to\\project.p12",
          "storePassword": "",     // DevEco Studio会自动加密填入
          "keyAlias": "xiaoq",
          "keyPassword": "",       // DevEco Studio会自动加密填入
          "certpath": "D:\\path\\to\\debug.cer",
          "profile": "D:\\path\\to\\debug.p7b",
          "signAlg": "SHA256withECDSA"
        }
      }
    ],
    "products": [
      {
        "name": "default",
        "signingConfig": "debug",
        "compatibleSdkVersion": "6.1.1(24)",  // SDK版本(API版本)
        "targetSdkVersion": "6.1.1(24)",
        "runtimeOS": "HarmonyOS"
      }
    ],
    "buildModeSet": [
      { "name": "debug" },
      { "name": "release" }
    ]
  },
  "modules": [
    {
      "name": "entry",
      "srcPath": "./entry",
      "targets": [
        { "name": "default", "applyToProducts": ["default"] }
      ]
    }
  ]
}
```

**模块级build-profile.json5**（`entry/build-profile.json5`）：

```json5
{
  "apiType": "stageMode",
  "targets": [
    { "name": "default" }
  ]
}
```

**密码配置**：DevEco Studio → File → Project Structure → Signing Configs，选中签名方案后填写Store Password和Key Password。IDE会自动加密写入。

### ⚠️ 不能覆盖 build-profile.json5（密码丢失）

DevEco Studio 填写的密码会被加密为 32+ 位长字符串写入 `build-profile.json5`。**如果用 write_file/sed/echo 等工具覆盖此文件，加密密码会被清空**，导致 `00303116: length of storePassword or keyPassword field is less than 32` 报错。

**铁律**：
1. 用户说"密码已填好"后，**绝对不再动 build-profile.json5**
2. 如需修改签名配置的其他字段（如 profile 路径），只能让用户在 IDE 中手动改
3. 空密码字段 `""` 不是"没填"，而是"被覆盖了"

### ⚠️ bundleName / Profile 包名匹配

`app.json5` 中的 `bundleName` **必须与 `.p7b` Profile 文件在 AGC 上绑定的包名完全一致**。

常见陷阱：
- Profile 文件名 ≠ 绑定的包名。例如 `WorldCup.debug.profileDebug.p7b` 绑定的包名可能是 `WorldCup.xiaoq.profile`，不是文件名推导出来的
- 新建项目时，如果使用新的 Profile，`app.json5` 的 `bundleName` 必须改成 AGC 上注册的那个包名
- 报错 `00303074: bundleName does not match` 就是不匹配

**诊断步骤**：
1. 去 AGC → 我的项目 → 应用 → 查看"包名"字段
2. 确认 `app.json5` 的 `bundleName` 与之一致
3. 确认 `build-profile.json5` 的 `profile` 路径指向正确的 `.p7b` 文件

---

## ArkTS严格模式编码规范（关键踩坑集锦）

ArkTS严格模式（`arkts-no-untyped-obj-literals`、`arkts-limited-throw`等）比TypeScript限制多得多。以下是从实战中总结的规则：

### 1. 对象字面量必须显式声明类型

```typescript
// ❌ 不行 — 匿名对象字面量
const obj = { a: 1, b: 'hello' };
const messages = [{ role: 'user', content: 'hi' }];

// ✅ 正确 — 使用接口
export interface ChatMessage {
  role: string;
  content: string;
}
const obj: Record<string, Object> = { 'a': 1, 'b': 'hello' };
const messages: ChatMessage[] = [{ role: 'user', content: 'hi' }];
```

### 2. throw必须使用Error子类

```typescript
// ❌ 不行 — 任意类型
throw 'error';
throw new Error('msg');  // 也不行，Error也不行

// ✅ 正确 — 自定义Error子类
class HermesError extends Error {
  constructor(message: string) {
    super(message);
  }
}
throw new HermesError('msg');
```

### 3b. Promise.catch() 必须显式标注 Error 类型

**症状**：编译报 `10605008 ArkTS Compiler Error: Use explicit types instead of "any", "unknown" (arkts-no-any-unknown)`，指向 `.catch((_e) => { ... })`。

**根因**：ArkTS严格模式下，Promise `.catch()` 回调的异常参数默认类型为 `any` 或 `unknown`，需要显式标注为 `Error`。

```ets
// ❌ 编译错误 — catch参数隐含any
promise.then(...).catch((_e) => {
  // 处理错误
});

// ✅ 正确 — 显式标注Error类型
promise.then(...).catch((_e: Error) => {
  // 处理错误
});
```

**注意**：这个错误也适用于 `.then()` 回调的参数——如果ArkTS无法推断类型，需要显式标注。通常 `.then((result: Type) => ...)` 是安全的写法。

**排查方法**：搜索 `\.catch\(\(_e\)` 或 `\.catch\(\(error\)` 模式，给所有catch参数加上 `: Error` 类型标注。

```typescript
// ❌ 不行
catch (error) {
  throw error;
}

// ✅ 正确 — 先判断类型再抛
catch (error) {
  if (error instanceof HermesError) {
    throw error;
  }
  if (error instanceof Error) {
    throw new HermesError((error as Error).message);
  }
  throw new HermesError('未知错误');
}
```

### 4. 包装弃用API消除编译警告

ArkTS编译器会标记 `showToast`、`pushUrl`、`back`、`getParams` 等 API 为 deprecated（弃用），但当前 SDK 版本（API 24）仍完全可用。用私有方法包装即可将 warnings 集中到一处，不影响编译通过：

```typescript
// ❌ 每次都写 promptAction.showToast → 10个调用=10条警告
promptAction.showToast({ message: msg });

// ✅ 包装一次，调用时无警告
private showToast(message: string): void {
  promptAction.showToast({ message: message });
}
// 调用
this.showToast('操作成功');
```

同样适用于 `router.pushUrl` 和 `router.back`。ArkTS 编译器不会对 deprecation warnings 报错，但包装后代码更干净。

### 9. @State string 变更后需要重新赋值触发 re-render\n\nArkTS 的 `@State` 通过引用是否变化来判断是否需要重绘。字符串拼接 `+=` 不改变引用，UI 不会更新。\n\n```typescript\n// ❌ UI 不更新\nthis.streamContent += token;  // streamContent 的引用没变\n\n// ✅ 强制触发 re-render\nthis.streamContent += token;\nthis.streamContent = this.streamContent;  // 重新赋值触发 @State setter\n```\n\n这个模式在 WebSocket 流式回复中尤其重要——每个 token 到达时都需要强制触发 UI 刷新才能显示打字机效果。\n\n### 10. 回调函数类型：undefined vs null 与可选参数

**ArkTS严格模式下，可选参数 `?:` 的类型是 `T | undefined`，而显式声明 `| null` 的类型是 `T | null`。两者不能互相赋值！**

```typescript
// ❌ 编译报错：Type '(() => void) | undefined' is not assignable to type '(() => void) | null'
private onConnect: (() => void) | null = null;
constructor() {
  this.setup(onConnect => { this.onConnect = onConnect; });  // 参数类型是 (() => void) | undefined
}

// ✅ 正确——全程用 undefined
private onConnect: (() => void) | undefined = undefined;
public setCallbacks(onConnect?: () => void): void {  // ?: = undefined
  this.onConnect = onConnect;  // 兼容
}
```

**规则**：类字段的初始化值和所有 `null` 检查，必须与参数的可选性一致。参数用 `?:`（undefined），字段就用 `| undefined = undefined`；参数用 `| null`，字段就用 `| null = null`。

### 4b. Wrap 组件在 API 24 不可用

`Wrap` 容器在 API 24 的 SDK 中不是有效组件。编译报 `Cannot find name 'Wrap'`。

```ets
// ❌ API 24 不可用
Wrap() {
  ForEach(items, item => { Text(item).padding(8).backgroundColor('#F0F0FF').borderRadius(10) })
}

// ✅ 改用 Flex + FlexWrap.Wrap
Flex({ direction: FlexDirection.Row, wrap: FlexWrap.Wrap }) {
  ForEach(items, (item: string) => {
    Text(item)
      .fontSize(11).fontColor('#5B5BD6')
      .padding({ left: 8, right: 8, top: 3, bottom: 3 })
      .backgroundColor('#F0F0FF').borderRadius(10)
      .margin({ right: 4, bottom: 4 })
  })
}
.width('100%')
```

### 5. 包装弃用API消除编译警告

ArkTS编译器会标记 `showToast`、`pushUrl`、`back`、`getParams` 等 API 为 deprecated（弃用），但当前 SDK 版本（API 24）仍完全可用。用私有方法包装即可将 warnings 集中到一处：

```typescript
// ❌ 每次都写 promptAction.showToast → 10个调用=10条警告
promptAction.showToast({ message: msg });

// ✅ 包装一次，调用时无警告
private showToast(message: string): void {
  promptAction.showToast({ message: message });
}
// 调用
this.showToast('操作成功');
```

同样适用于 `router.pushUrl` 和 `router.back`。ArkTS 编译器不会对 deprecation warnings 报错（不影响编译通过），但包装后代码更干净、调用更简洁。

```typescript
@Builder
MessageBubble(message: ChatMessage) { ... }

// ❌ 不行 — 被包装成对象字面量
this.MessageBubble({ message: msg })

// ✅ 正确 — 直接传参
this.MessageBubble(msg)
```

### 5. 对齐类型要匹配容器

```typescript
// Column的alignItems → HorizontalAlign（水平方向对齐）
Column() { ... }
.alignItems(HorizontalAlign.Start)  // 左对齐
.alignItems(HorizontalAlign.Center)

// Row的alignItems → VerticalAlign（垂直方向对齐）
Row() { ... }
.alignItems(VerticalAlign.Top)
.alignItems(VerticalAlign.Center)
```

### 6. HttpRequest没有 setRequestHeader()

HarmonyOS `@kit.NetworkKit.http` 的 `HttpRequest` **没有** `setRequestHeader()` 方法。Headers在 `request()` 的选项参数中设置。

### 7. 静态方法中不能用 this

```typescript
class WorldCupApi {
  static async fetchMatches(): Promise<MatchInfo[]> {
    if (teamsMap.size === 0) {
      await this.fetchTeams();  // ❌ arkts-no-standalone-this
      await WorldCupApi.fetchTeams();  // ✅ 用类名
    }
  }
}
```

静态方法里的 `this` 在ArkTS严格模式下不允许。用 `ClassName.method()` 替代。

### 8. 不支持索引访问类型（Indexed Access Types）

```typescript
// ❌ arkts-no-aliases-by-index
stage: g.type as MatchInfo['stage'];

// ✅ 直接赋值，用 as Type 断言
stage: g.type;
```

`Type['field']` 这种索引访问语法在ArkTS中不支持。直接赋值或用具体类型。

### 9. Image组件空URL会崩溃

```typescript
// ❌ 空URL导致APP闪退
Image(flagUrl).width(24).height(16)

// ✅ 先判断URL是否有效
@Builder
TeamFlag(flagUrl: string) {
  if (flagUrl.length > 0) {
    Image(flagUrl).width(24).height(16).objectFit(ImageFit.Contain)
  } else {
    Text('⚽').fontSize(16)
  }
}
```

**铁律**：渲染远程图片前必须检查URL非空。用 `@Builder` 封装带fallback的图片组件。

### 10. 变量必须有显式类型声明

```typescript
// ❌ 不行
let x = 10;
const data = { name: 'test' };

// ✅ 正确
let x: number = 10;
const data: Record<string, string> = { 'name': 'test' };
```

### 11. API数据中的Unicode智能引号

外部API返回的数据可能包含Unicode智能引号（`'` U+2019、`"` U+201C/201D），而非标准ASCII引号。直接显示会导致乱码。

```typescript
// API返回: {"J. Quiñones 9'","R. Jiménez 67'"}
// 其中 ' 是 U+2019（RIGHT SINGLE QUOTATION MARK）

// ❌ 只替换ASCII引号
cleaned = scorersStr.replace(/'/g, '"');  // 不生效

// ✅ 替换所有Unicode智能引号
cleaned = scorersStr.replace(/[\u2018\u2019\u201C\u201D']/g, '"');
```

**最佳实践**：对外部API数据做清洗时，将所有非ASCII字符替换为ASCII等价物后再解析。进球者名字等展示数据也建议清理后使用纯ASCII。

### 11d. interface不能定义在函数/方法体内

**症状**：编译报 `Unexpected token` 或 `Declaration or statement expected`，指向函数内的 `interface` 声明。

```ets
// ❌ 错误——interface在函数体内
static extractTopScorers(matches: MatchInfo[]): TopScorer[] {
  interface ScorerStat { name: string; goals: number; }  // 编译报错
  const map: Record<string, ScorerStat> = {};
}

// ✅ 正确——移到模块级
interface ScorerStat { name: string; goals: number; }
export class WorldCupApi {
  static extractTopScorers(matches: MatchInfo[]): TopScorer[] {
    const map: Record<string, ScorerStat> = {};
  }
}
```

**规则**：所有 `interface` 和 `type` 声明必须在模块顶层（文件级），不能在 `class` 内部、`function` 内部、`if/else` 块内部。

### 11c. 联合类型不能赋string变量（status字段踩坑）

**症状**：`Type 'string' is not assignable to type '"scheduled" | "live" | "finished"'`

**根因**：ArkTS严格模式下，联合类型字段（如 `status: "scheduled" | "live" | "finished"`）不能接受 `string` 类型的变量赋值，即使运行时值完全匹配。

```ets
// ❌ string变量不能赋给联合类型
let status: string = 'finished';
m.status = status;  // 编译错误！

// ❌ 从缓存读出的string也不能直接赋
const cachedStatus: string = cache['status'] || 'scheduled';
m.status = cachedStatus;  // 编译错误！

// ✅ 用if/else逐值赋字面量
if (cachedStatus === 'finished') {
  m.status = 'finished';
} else if (cachedStatus === 'live') {
  m.status = 'live';
} else {
  m.status = 'scheduled';
}

// ✅ 或直接在判断处赋值，不经过中间变量
if (isFinished) {
  m.status = 'finished';
  m.timeElapsed = 'FT';
} else if (elapsed !== 'notstarted') {
  m.status = 'live';
  m.timeElapsed = elapsed;
} else {
  m.status = 'scheduled';
  m.timeElapsed = 'notstarted';
}
```

**规则**：联合类型字段的赋值必须直接使用字面量（`'finished'`、`'live'`、`'scheduled'`），不能通过string变量中转。从缓存/API读出的值需要通过 `if/else` 或 `switch` 分支逐值匹配。

### 11b. 大文件编辑后必须验证大括号平衡（2948错误连锁陷阱）

**症状**：编译报 2948+ 个 Error，报错信息含 `Unexpected token` 或 `RollupError`。这不是代码逻辑问题，而是某个 .ets 文件**语法不完整**（大括号不平衡），导致整个编译管线崩溃。

**根因**：用 `write_file` 或 `patch` 工具多次编辑 .ets 文件时，如果某次操作截断了文件（写入不完整、替换范围计算错误），会导致大括号 `{` 和 `}` 数量不匹配。一个文件的语法错误会连锁导致所有引用它的文件报错。

**铁律——每次编辑 .ets 文件后立即验证**：
```bash
python3 -c "
with open('path/to/file.ets') as f:
    c = f.read()
opens = c.count('{')
closes = c.count('}')
diff = opens - closes
if diff != 0:
    print(f'*** IMBALANCED: {{ {opens} }} {closes} diff={diff}')
else:
    print(f'OK: {{ {opens} }} {closes}')
"
```

**批量检查所有 .ets 文件**：
```bash
python3 -c "
import glob
for f in sorted(glob.glob('entry/src/main/ets/**/*.ets', recursive=True)):
    with open(f) as fh:
        content = fh.read()
    d = content.count('{') - content.count('}')
    name = f.split('ets/')[-1]
    if d != 0:
        print(f'*** {name}: diff={d}')
"
```

**批量检查行号前缀污染**（2000+ Error时优先执行）：
```bash
for f in entry/src/main/ets/**/*.ets; do
    first=$(head -1 "$f")
    if echo "$first" | grep -qP '^\d+\|'; then
        echo "CORRUPTED: $f"
    fi
done
```
**常见根因：`read_file` 行号前缀污染（1853错误连锁陷阱）**

当用 `execute_code` 批量修改 .ets 文件时，如果脚本中调用了 `read_file()` 读取文件内容，返回值包含行号前缀（`1|import...`、`2|import...`）。如果将这些内容直接用 `write_file()` 写回文件，行号前缀会成为文件内容的一部分，导致 ArkTS 编译器完全无法解析文件。

**症状**：编译报 1800+ 个 Error，第一个错误是 `@State decorator can only be used with 'struct'`（因为编译器看到的是 `8|  @State` 而不是 `@State`）。

**诊断**：
```bash
head -1 path/to/file.ets
# 如果输出是 "1|import { router }..." 而不是 "import { router }..."，就是被污染了
```

**修复**：strip行号前缀：
```bash
python3 -c "
import re
with open('path/to/file.ets') as f:
    lines = f.readlines()
cleaned = []
for line in lines:
    m = re.match(r'^\d+\|', line)
    if m:
        cleaned.append(line[m.end():])
    else:
        cleaned.append(line)
with open('path/to/file.ets', 'w') as f:
    f.writelines(cleaned)
"
```

**预防**：在 `execute_code` 中使用 `read_file` 后写回文件时，**必须先 strip `N|` 前缀**。或者直接用 `terminal` + `sed`/`python3` 操作文件，不用 `read_file`/`write_file` 组合。

**修复方法**：
1. 如果 diff > 0（缺少 `}`），在文件末尾补上缺失的闭合括号
2. 如果 diff < 0（多余 `}`），找到多余的闭合括号删除
3. 如果文件明显被截断（最后一行不是 `}`），从 git 恢复：`git checkout HEAD -- path/to/file.ets`，然后重新应用修改
4. 如果多个文件同时报错，先检查所有文件是否有行号前缀污染
5. **不要在不平衡状态下尝试编译**——会报 2948 个错误，浪费时间

**预防措施**：
- 每次 `write_file` 后立即验证大括号平衡
- 每次 `patch` 后立即验证
- 如果一个文件需要多次大修改，考虑一次性 `write_file` 整个文件，而非多次 `patch`
- 修改超过 200 行的 .ets 文件时，优先用 `write_file` 整体重写而非增量 `patch`
- **增量patch截断风险**：多次 `patch` 替换同一文件时，如果某次替换的 `old_string` 匹配到错误位置（如匹配到了注释中的同名代码），会导致文件结构被破坏。对于复杂重构，直接 `git checkout HEAD -- file.ets` 恢复干净版本，再一次性 `write_file` 重写，比反复 `patch` 安全得多
- **find-and-replace 碰撞风险**：用 `str.replace(old, new)` 修改多个类似条目时，如果多个条目的 `old` 值相同（如 `kickoff:'06/19 03:00'`），`replace()` 用 `count=1` 只会替换第一个匹配，可能导致错误的条目被改。**应该按唯一ID逐行编辑**，而不是按值全局替换

### 12. http.HttpDataType.OBJECT 的类型自动转换陷阱

使用 `expectDataType: http.HttpDataType.OBJECT` 时，HarmonyOS HTTP客户端会自动将JSON值转为原生类型：

```ets
// API返回JSON: {"home_score": "2", "finished": "TRUE", "time_elapsed": "finished"}
// 经过 HttpDataType.OBJECT 解析后：
//   g['home_score'] → 运行时是 number 2（不是 string "2"）
//   g['finished']   → 运行时是 boolean true（不是 string "TRUE"）
//   g['time_elapsed'] → 运行时是 string "finished"（不变）

// ❌ 字符串比较对boolean值永远失败
const finished: string = g['finished'] || '';  // finished = "true" (boolean→string via assignment)
if (finished === 'TRUE') { ... }  // 永远false！boolean true 不等于 string "TRUE"

// ❌ parseInt对已转为number的值行为不可预测
m.homeScore = parseInt(g['home_score']) || 0;  // parseInt(2) 可能出问题

// ✅ 正确——用 '' + value 强制转字符串，再检查所有可能的值
const finStr: string = '' + g['finished'];    // boolean true → "true", string "TRUE" → "TRUE"
const isFinished: boolean = (finStr === 'true' || finStr === 'TRUE');

const hsStr: string = '' + g['home_score'];   // number 2 → "2", string "2" → "2"
m.homeScore = parseInt(hsStr) || 0;

const elapsed: string = '' + g['time_elapsed']; // 保证是字符串
```

**规则**：从 `http.HttpDataType.OBJECT` 返回的数据，**永远用 `'' + value` 转字符串后再处理**。不要信任 `as string` / `as number` / `as boolean` 断言——ArkTS运行时的类型断言不改变实际值的类型。

**boolean值的双重检查**：API返回 `"TRUE"` 可能被转为 boolean `true`，`'' + true` 得到 `"true"`（小写）。所以检查finished状态时必须同时匹配：
```ets
const finStr: string = '' + g['finished'];
const isFinished: boolean = (finStr === 'true' || finStr === 'TRUE');
```

#### ⚠️ 进阶致命暗坑：OBJECT 自动解析可能失效（网络传输与网关压缩影响）

* **症状**：即使在 `request` 中配置了 `expectDataType: http.HttpDataType.OBJECT`，网络接口由于使用了 Gzip 压缩、Chunked 编码或 `Content-Type` 格式不够标准，**运行时 `resp.result` 依然会被当做 `string` 返回**！
* **毁灭后果**：在 ArkTS 中，直接用 `as Record<string, string>` 强转 `resp.result` 在编译期不报错。但由于在运行期它实际上依然是个 `string`，此时调用 `Object.keys(remoteMap)` 拿到的不是 JSON 键，而是**整个 JSON 字符串的字符索引**（即 `["0", "1", "2", ...]`）！接着遍历它进行覆写，会导致本地核心字典（如中英文映射字典 `SCORER_CN`）瞬间被一堆乱码索引毁灭性破坏。
* **终极防御方案（typeof 双重验证判定）**：
```ets
if (resp.responseCode === 200) {
  let remoteMap: Record<string, string> = {} as Record<string, string>;
  
  // 🛡️ 强制进行运行期类型检测，对未自动转换成功的 string 进行手动解析，实现 100% 鲁棒性
  if (typeof resp.result === 'string') {
    try {
      remoteMap = JSON.parse(resp.result as string) as Record<string, string>;
    } catch (e) {
      console.error('Failed to parse remote scorer names string');
    }
  } else {
    remoteMap = resp.result as Record<string, string>;
  }

  const remoteKeys: string[] = Object.keys(remoteMap);
  if (remoteKeys.length > 0) {
    // 安全地合并与缓存本地数据...
  }
}
```

### 13. API不可靠时的Fallback策略

当外部API可能不可用（网络问题、SSL、域名封锁等），有三种策略：

**方案A：API优先 + 纯fallback（简单场景）**
```ets
static async fetchData(): Promise<Data[]> {
  const data = await tryFetchJson('/api/data');
  if (data !== null) return parseApiData(data);
  return FALLBACK_DATA;  // 完整的fallback数据
}
```

**方案B：hardcoded骨架 + API动态数据merge-by-ID（推荐，见§18详述）**
```ets
// 硬编码提供准确的静态数据（时间、名称等）
// API提供实时的动态数据（比分、状态等）
// 按共享ID合并两者
static async fetchMatches(): Promise<MatchInfo[]> {
  const result = buildFallbackMatches();  // 骨架（时间准确）
  const apiData = await tryFetchJson('/get/games');
  if (apiData !== null) {
    // 建立ID索引，将API的score/status合并到骨架
    mergeApiScores(result, apiData);
  }
  return result;  // API挂了也能显示（0:0 scheduled）
}
```

**方案C：直接硬编码（最简单，数据量小且几乎不变时用）**
```ets
static async fetchMatches(): Promise<MatchInfo[]> {
  return ALL_MATCHES;  // 启动即有数据，零网络请求
}
```

**选择指南**：
| 场景 | 推荐方案 |
|------|---------|
| API数据完全准确，只是可能不可用 | A |
| API静态数据不准（如时间），动态数据准（如比分） | **B** |
| 数据量小、几乎不变、API不可靠 | C |

**何时用方案B**：API的某些字段（如时间）不可靠，但其他字段（如实时比分）是准确的。用hardcoded保证不可靠字段的准确性，用API保证实时字段的新鲜度。前提：两套数据有共享的ID可以匹配。

### 13. bindContentCover 不能用 `||` 表达式作为绑定值（弹窗不显示的根因）

**ArkTS 的 `bindContentCover(isShow, builder)` 不追踪 `||` 表达式内的 `@State` 变化。** 当用 `this.showA || this.showB` 作为 isShow 参数时，即使 `showA` 或 `showB` 变化，`bindContentCover` 也不会重新计算绑定值，弹窗永远不会显示。

```typescript
// ❌ 弹窗不显示——ArkTS 不追踪 || 表达式
@State showSettings: boolean = false;
@State showApproveDialog: boolean = false;
.bindContentCover(this.showSettings || this.showApproveDialog, this.DialogLayer())

// ✅ 用单一 @State 变量控制
@State showDialog: boolean = false;
.bindContentCover(this.showDialog, this.DialogLayer())

// 打开设置
this.showSettings = true;
this.showDialog = true;

// 打开授权卡片（同时关设置）
this.showSettings = false;
this.showDialog = true;

// 关闭
this.showDialog = false;

@Builder
DialogLayer() {
  if (this.showSettings) { this.SettingsPanel(); }
  else { this.ApproveDialog(); }
}

```

**铁律**：`bindContentCover` 的 `isShow` 参数必须是一个单独的 `@State boolean` 变量。任何形式的计算表达式（`||`、`&&`、`?:`）都会导致弹窗不可见。如需区分多个弹窗内容，用第二个 `@State` 变量做条件判断。详见 `references/approval-card-pattern.md`。

### 14. `for...of` / `for...in` / `.forEach()` 全部禁用

**这是最容易导致"编译通过但运行时页面空白/APP闪退"的坑。** ArkTS严格模式下，所有非索引循环都不支持。写了一大堆代码全用 `for...of`，编译无错，运行时页面永远是空的、APP闪退。

**快速诊断**：如果多个页面同时显示空白（加载完成但无数据），且APP没有崩溃，99%是某个数据加载函数里用了 `for...of`/`Map.forEach`，导致函数静默抛异常，`@State` 数组永远为空。搜索 `for\s*\(const` 和 `.forEach(` 全部替换即可。

```ets
// ❌ 错误（编译可能过，运行时崩溃/静默失败/页面空白）
for (const item of this.items) { ... }
for (const key in obj) { ... }
this.items.forEach((item: Item) => { ... });
someMap.forEach((value: Object, key: string) => { ... });

// ✅ 正确——唯一安全的循环方式
for (let i = 0; i < this.items.length; i++) {
  const item: Item = this.items[i];
  // ...
}
```

**排查方法**：全项目搜索 `for\s*\(const` 和 `.forEach(`，全部替换为 `for (let i = 0; ...)` 索引循环。

### 14. `get` 属性访问器在含循环的场景下不可靠

`get` 属性访问器 + `for...of` 组合经常导致页面空白或闪退。即使把 `for...of` 改成索引循环，getter在某些复杂场景下仍有问题。**建议全部改用普通方法**。

```ets
// ❌ 不稳定——get + 内部循环
get filteredData(): Item[] {
  const result: Item[] = [];
  for (const item of this.items) { ... }  // for...of是雷区
  return result;
}
// build() 中引用：ForEach(this.filteredData, ...)

// ✅ 稳定——普通方法 + 索引循环
getFilteredData(): Item[] {
  const result: Item[] = [];
  for (let i = 0; i < this.items.length; i++) { ... }
  return result;
}
// build() 中引用：ForEach(this.getFilteredData(), ...)
```

**规则**：数据过滤/排序逻辑统一用 `getXxx()` 方法，不用 `get xxx()` 访问器。

### 15. `Map` 类型在运行时不可靠

`Map.forEach`、`Map.get` 等方法在ArkTS运行时可能静默失败（编译通过，运行时不执行或崩溃）。

```ets
// ❌ 不可靠
const cache: Map<string, Object> = new Map();
cache.set('key', data);
cache.forEach((v, k) => { ... });  // 可能静默失败

// ✅ 用数组模拟 Map
interface CacheEntry { id: string; data: Object; }
const cache: CacheEntry[] = [];

function findById(id: string): Object | null {
  for (let i = 0; i < cache.length; i++) {
    if (cache[i].id === id) return cache[i].data;
  }
  return null;
}
```

### 17. 不同ID体系的数据不能合并（含ID顺序不一致陷阱）

**陷阱2：ID范围相同、含义相同、但顺序不同（更隐蔽！）**

WorldCup2026踩坑：API和hardcoded都用ID 1-72，都代表72场小组赛。但：
- 我的hardcoded按组排：AABBCCDD... (id1=A1, id2=A2, id3=B1, id4=B2...)
- API按开赛时间排：AABDCDCE... (id1=A1, id2=A2, id3=B1, id4=D1...)

结果：id=5 在hardcoded里是巴西vs摩洛哥，在API里是海地vs苏格兰——比分全串了！

**诊断方法**：打印API前20条数据的id+队伍名，跟hardcoded逐条对比：
```bash
curl -s 'https://api.example.com/games' | python3 -c "
import json,sys
data=json.load(sys.stdin)
for g in data['games'][:20]:
    print(f\"id={g['id']} {g['home']} vs {g['away']}\")
"
```

**铁律**：
1. hardcode ID时，**必须从API实际返回数据中提取ID**，不能自己推导
2. 合并后用已知比分验证：找一场用户确认过的比赛，检查合并后的score是否正确
3. 如果API和hardcoded的ID顺序不一致，必须重写hardcoded的ID使其与API完全对齐

```ets
// ❌ 危险——自己推导ID（假设按组排序）
const SCHEDULE = [
  {id:'1', home:'Mexico', away:'South Africa'},      // A组第1场
  {id:'2', home:'South Korea', away:'Czech Republic'}, // A组第2场
  {id:'3', home:'Canada', away:'Bosnia'},              // B组第1场
  {id:'4', home:'Qatar', away:'Switzerland'},          // B组第2场  ← 实际API id=4是USA vs Paraguay!
  // ...
];

// ✅ 正确——从API输出中提取ID，逐条复制
// 先 curl API，看清楚每条的id是什么，再写入hardcoded
const SCHEDULE = [
  {id:'1', home:'Mexico', away:'South Africa'},        // API确认id=1
  {id:'2', home:'South Korea', away:'Czech Republic'}, // API确认id=2
  {id:'3', home:'Canada', away:'Bosnia'},              // API确认id=3
  {id:'4', home:'United States', away:'Paraguay'},     // API确认id=4（不是Qatar!）
  // ...
];
```

### 17. MainPage内联内容 vs router.pushUrl

**陷阱**：MainPage用 `router.pushUrl` 跳转子页面时，MainPage本身没有任何内容显示——用户打开APP只看到空的标题栏和TabBar。

```ets
// ❌ MainPage用router.pushUrl——主页本身是空的
onClick(() => {
  this.currentIndex = index;
  router.pushUrl({ url: item.page });  // 跳走了，主页留空
})

// ✅ MainPage内联所有Tab内容——用Stack布局保证底部导航栏始终可见
build() {
  Stack({ alignContent: Alignment.Bottom }) {
    Column() {
      // 顶部标题
      Row() { ... }.height(44)
      // 内容区——根据currentIndex切换
      if (this.currentIndex === 0) { this.HomeContent() }
      else if (this.currentIndex === 1) { this.StandingsContent() }
      // ...
    }
    .width('100%').height('100%')
    .padding({ bottom: 56 })  // 给底部导航留空间

    // 底部导航——固定在Stack底部
    Row() {
      this.TabItem('📅', '首页', 0)
      this.TabItem('🏆', '积分', 1)
      // ...
    }.width('100%').height(56).backgroundColor('#0F3460')
  }
  .width('100%').height('100%')
}
```

**关键**：
- 用 `Stack({ alignContent: Alignment.Bottom })` 而非 `Column`，保证TabBar始终浮在底部
- 内容区加 `padding({ bottom: 56 })` 避让TabBar高度
- 子页面内容用 `@Builder` 方法封装（`HomeContent()`、`StandingsContent()`等），不用独立的 `@Entry @Component struct`
- 如果子页面也有 `@Entry` 装饰器，嵌入TabContent时生命周期会冲突——去掉子页面的 `@Entry`

### 18. 外部API时间不可靠时的混合数据策略（重要）

**核心教训**：外部体育/赛事API的`local_date`时间字段往往不可靠（时区不一致、偏移量因比赛地点而异）。**不要用固定偏移做时区转换**。

### local_date 时区转换教训（2026-06-16 验证）

**核心问题**：API `local_date` 是**场馆当地时间**，不同场馆时区不同，单一固定偏移量（如 +15h）对所有比赛不准确。

2026年世界杯场馆时区及对应的北京时间偏移：

| 场馆地区 | 当地时间(6月) | 北京偏移 |
|---------|-------------|---------|
| 美国东部(亚特兰大/迈阿密/波士顿/纽约/费城/多伦多) | UTC-4 (EDT) | **+12h** |
| 美国中部(达拉斯/休斯顿/堪萨斯城) | UTC-5 (CDT) | **+13h** |
| 美国山地(丹佛/盐湖城) | UTC-6 (MDT) | **+14h** |
| 墨西哥城 | UTC-6 (CST, 无夏令时) | **+14h** |
| 美国太平洋(洛杉矶/旧金山/西雅图/温哥华) | UTC-7 (PDT) | **+15h** |

**新规则**：不用API时间转换，在SCHEDULE中**硬编码正确北京时间**。详见 `references/worldcup2026-api-patterns.md` 的 `local_date Timezone Trap` 章节。

### WorldCup2026踩坑实录（代码修改时验证）：
- 用 `+12h`（美国东部）转北京时间比用 `+15h`（美国太平洋）准确
- 但央视CCTV5节目表显示这场比赛是北京时间 `06/14 03:00`（差3小时）
- 原因：API的local_date时区因场馆地点不同而不同（墨西哥城UTC-6、美国东部UTC-4、美国中部UTC-5、美国太平洋UTC-7），固定偏移不可能对所有比赛都正确
- **修复方案**：在SCHEDULE硬编码正确北京时间（API local_date + 场馆对应时区偏移），不再动态转换。详见 `references/worldcup2026-api-patterns.md`

**正确方案：hardcoded骨架 + API动态数据按ID合并（注意：北京时间需硬编码，不能用API time + 固定偏移）**

```ets
// 1. 硬编码权威静态数据（时间来自官方赛程图，100%准确）
const SCHEDULE: MatchRaw[] = [
  {id:'1', home:'Mexico', away:'South Africa', group:'A', kickoff:'06/12 03:00'},
  {id:'2', home:'South Korea', away:'Czech Republic', group:'A', kickoff:'06/12 10:00'},
  // ... 72场全部硬编码
];

// 2. 从hardcoded构建骨架
function buildFallbackMatches(): MatchInfo[] {
  // 遍历SCHEDULE，构建MatchInfo[]，status全部为'scheduled'，score全部为0
}

// 3. fetchMatches() 始终用hardcoded骨架，API只补动态数据
static async fetchMatches(): Promise<MatchInfo[]> {
  const result: MatchInfo[] = buildFallbackMatches();  // 时间永远准确

  const data = await tryFetchJson('/get/games');
  if (data !== null) {
    // 建立API索引 id -> game（用Record而非Map，ArkTS运行时Map不可靠）
    const apiMap: Record<string, Record<string, string>> = {};
    for (let i = 0; i < games.length; i++) {
      const g: Record<string, string> = games[i] as Record<string, string>;
      apiMap[g['id']] = g;
    }
    // 按ID合并：骨架提供时间，API提供比分/状态/射手
    for (let i = 0; i < result.length; i++) {
      const m: MatchInfo = result[i];
      const g: Record<string, string> | undefined = apiMap[m.id];
      if (g !== undefined) {
        m.homeScore = parseInt(g['home_score']) || 0;
        m.awayScore = parseInt(g['away_score']) || 0;
        m.homeScorers = parseScorers(g['home_scorers'] || '');
        // ... 状态、射手等动态字段
      }
    }
  }
  return result;  // API挂了也能显示赛程（0:0 scheduled）
}
```

**关键前提**：hardcoded的ID必须与API的ID一致。WorldCup2026中API返回`id: "1"~"72"`，hardcoded也用`"1"~"72"`，所以按ID匹配成功。

**何时用此模式**：
- API的静态数据（时间、名称）不准，但动态数据（比分、状态）是实时的
- 需要离线graceful fallback
- 数据量不大（<100条），可以全部hardcoded

**不要用此模式**：
- API数据完全准确 → 直接用API
- 数据量大（>500条）或频繁变化 → 不适合hardcoded

### 19. API数据两步查找（Data Joining）

当一个API端点只返回ID、不返回名称时，需要先查另一个端点获取映射表：

```ets
// /get/groups 只返回 team_id，不返回队名
// 需要先加载 /get/teams 获取 id→name 映射

let teamsCache: Object[] = [];

async function ensureTeamsLoaded(): Promise<void> {
  if (teamsCache.length > 0) return;
  const data: Object | null = await tryFetchJson('/get/teams');
  if (data !== null) {
    const obj: Record<string, Object> = data as Record<string, Object>;
    teamsCache = obj['teams'] as Object[];
  }
}

function findTeamNameById(teamId: string): string {
  for (let i = 0; i < teamsCache.length; i++) {
    const t: Record<string, string> = teamsCache[i] as Record<string, string>;
    if (t['id'] === teamId) return getCnName(t['name_en'] || '');
  }
  return '队伍' + teamId;  // 找不到时显示ID
}

// 在fetchStandings中：
static async fetchStandings(): Promise<TeamStanding[]> {
  await ensureTeamsLoaded();  // 先加载映射表
  const data = await tryFetchJson('/get/groups');
  // ... 解析groups，用 findTeamNameById(teamId) 查名称
}
```

**通用模式**：`缓存变量 + ensureXxxLoaded() + findById()` 三件套。用数组而非Map（ArkTS运行时Map不可靠）。

### 20. 日期格式化getMonth()必须+1

`Date.getMonth()` 返回0-11（0=一月），格式化时必须+1：

```ets
// ❌ 6月显示成"05"
const month: number = now.getMonth();

// ✅ 正确
const month: number = now.getMonth() + 1;
```

### 20b. bindContentCover 不能传 || 表达式（弹窗不显示的根源）

**症状**：`bindContentCover` 绑定后，当某个 `@State` 变量变 `true` 时，弹窗不显示。

**错误写法**：
```typescript
// ❌ `||` 表达式不会被 @State 变化追踪
.bindContentCover(this.showSettings || this.showApproveDialog, this.DialogLayer())
```
即使 `showApproveDialog` 变 `true`，`false || true` 也不会被 ArkTS 重新求值——弹窗不出现。

**正确写法：用单一 @State + 模式字符串**：
```typescript
@State showDialog: boolean = false;
@State dialogMode: string = '';  // 'settings' | 'approve' | ''

// 打开设置
this.showDialog = true;
this.dialogMode = 'settings';

// 打开授权卡片
this.showDialog = true;
this.dialogMode = 'approve';

// 绑定单一 @State
.bindContentCover(this.showDialog, this.DialogLayer())

// Builder 中根据模式切换
@Builder DialogLayer() {
  if (this.dialogMode === 'settings') { this.SettingsPanel(); }
  else if (this.dialogMode === 'approve') { this.ApproveDialog(); }
}
```

**规则**：`bindContentCover` 的绑定变量必须是**单一 `@State` 布尔值**，不能是表达式。多个弹窗/面板用 `dialogMode` 字符串区分。

### 21. 日期匹配必须同时检查status + "明天没比赛"降级

用 `indexOf` 匹配日期字符串时，必须同时过滤比赛状态，否则已结束的比赛也会被返回：

```ets
// ❌ 可能返回已结束的比赛
if (match.kickoff.indexOf(tomorrowStr) >= 0) { ... }

// ✅ 只返回未开始的比赛
if (match.kickoff.indexOf(tomorrowStr) >= 0 && match.status === 'scheduled') { ... }
```

**"明天没有比赛"场景**：只查明天会导致页面空白。用 `getNextAvailableMatches()` 降级模式：
1. 先查明天的比赛（过滤 status === 'scheduled'）
2. 没有→排除今天的比赛→取最近一天的所有比赛

```ets
getNextAvailableMatches(): MatchInfo[] {
  // 1. 先找明天
  const tomorrowDate: Date = new Date();
  tomorrowDate.setDate(tomorrowDate.getDate() + 1);
  const tomorrowStr: string = this.formatDate(tomorrowDate);
  // filter by tomorrowStr + status === 'scheduled'
  if (result.length > 0) return result;
  // 2. 没有→找最近的未来比赛日（排除今天）
  // filter status === 'scheduled' && kickoff.indexOf(today) < 0
  // 取第一场日期，返回同日所有比赛
}
```

**教训**：赛事类APP必须考虑"某天没有比赛"的边界场景，用降级逻辑保证用户总能看到可预测的比赛。

### 25. 本地数据缓存：preferences缓存API结果（减少API调用）

**核心思路**：已获取的API数据持久化到本地preferences，下次打开APP直接读缓存，只对live/未缓存的比赛调API。

```
首次打开:  loadPersistedCache() → getMatches() → refreshScores() → startAutoRefresh()
第二次打开: loadPersistedCache() → getMatches()（缓存比分秒显示） → refreshScores()（只刷live）
```

**两层缓存**：
| 层级 | 存储 | 速度 | 生命周期 |
|------|------|------|---------|
| 内存 | `static scoreCache` | 0ms | 当前会话 |
| 持久化 | `preferences` | ~50ms | 跨会话 |

**关键**：`refreshScores()`必须传`context`参数才能持久化；`getMatches()`是同步的，读内存缓存。

详见 `references/local-data-cache-pattern.md`。

### 24. 自动刷新：setInterval + isRefreshing锁（防请求堆积）

**核心教训**：`setInterval` 每500ms调一次异步API，如果API响应要3-5秒，就会堆积6-10个并发请求，把API压垮，反而更慢。

**错误模式**：
```ets
// ❌ 每500ms无脑发请求，API被并发请求压垮
setInterval(() => {
  WorldCupApi.refreshScores(this.matches);  // 不await，每次都是新请求
}, 500);
// 结果：6个请求排队等API → 20-30秒才全部返回
```

**正确模式**：isRefreshing锁 + 合理间隔 + then/catch释放锁
```ets
private refreshCount: number = 0;
private refreshTimerId: number = -1;
private isRefreshing: boolean = false;

private startAutoRefresh(): void {
  this.refreshCount = 0;
  this.refreshTimerId = setInterval(() => {
    this.refreshCount++;
    // 防止请求堆积：上一次还没回来就跳过
    if (!this.isRefreshing) {
      this.isRefreshing = true;
      WorldCupApi.refreshScores(this.matches).then(() => {
        this.isRefreshing = false;
      }).catch(() => {
        this.isRefreshing = false;  // 异常也要释放锁
      });
    }
    // 停止条件：超时 或 无live比赛
    if (this.refreshCount >= 7 || !this.hasLiveMatch()) {
      this.stopAutoRefresh();
    }
  }, 3000);  // 3秒间隔，不是500ms
}

private stopAutoRefresh(): void {
  if (this.refreshTimerId >= 0) {
    clearInterval(this.refreshTimerId);
    this.refreshTimerId = -1;
  }
}

private hasLiveMatch(): boolean {
  for (let i = 0; i < this.matches.length; i++) {
    if (this.matches[i].status === 'live') return true;
  }
  return false;
}

aboutToDisappear(): void {
  this.stopAutoRefresh();  // 页面销毁时清除定时器，防内存泄漏
}
```

**关键参数**：
- 间隔：3000ms（不是500ms——API响应本身就慢，500ms只会堆积请求）
- 最大次数：7次 × 3秒 ≈ 21秒
- 停止条件：`refreshCount >= 7` 或 `!hasLiveMatch()`
- 锁机制：`isRefreshing` 保证同时只有1个请求在飞
- `.then()/.catch()` 都要释放锁——异常时也要 `isRefreshing = false`
- `aboutToDisappear()` 必须清除定时器

**效果对比**：
| 方案 | 请求数 | 首次结果显示 | 总耗时 |
|------|--------|-------------|--------|
| 旧：500ms无锁 | 6-10并发 | 等所有排队完 | 20-30秒 |
| 新：3s+isRefreshing锁 | 1个在飞 | 首次返回即显示 | 2-3秒/次 |

**注意**：`setInterval` 回调不能直接用 `async/await`，用 `.then()/.catch()` 链式调用。

### 26. 下拉刷新：Refresh组件（数据更新型页面标配）

用户在积分榜、排行榜、列表页需要按需刷新数据时，用ArkTS的 `Refresh` 组件实现下拉刷新。

```typescript
// 状态声明
@State isRefreshing: boolean = false;

// 刷新回调
private async onRefresh(): Promise<void> {
  this.isRefreshing = true;
  try {
    this.standings = await WorldCupApi.fetchStandings();
  } catch (e) {
    // ignore
  }
  this.isRefreshing = false;  // 松手收回刷新动画
}

// build() 中用 Refresh 包裹内容区
Refresh({ refreshing: $$this.isRefreshing, offset: 12, friction: 100 }) {
  Column() {
    // 表头 + 数据列表
    // ...
  }
  .width('100%').layoutWeight(1)
}
.onRefreshing(() => {
  this.onRefresh();
})
.width('100%').layoutWeight(1)
```

**关键参数**：
- `refreshing: $$this.isRefreshing` — **双向绑定**（`$$` 是关键），框架自动控制下拉动画的起止
- `offset: 12` — 触发刷新的下拉距离（vp）
- `friction: 100` — 下拉摩擦系数（越大越难拉）
- `.onRefreshing()` — 用户松手后触发的回调，必须在里面执行数据刷新并最终设 `isRefreshing = false`

**与setInterval自动刷新的对比**：

| 方式 | 适用场景 | 触发方式 |
|------|---------|---------|
| setInterval自动刷新 | 实时比分、直播状态 | 后台定时，用户无感 |
| Refresh下拉刷新 | 积分榜、排行榜、列表数据 | 用户主动下拉 |

**组合使用**：实时比分页用setInterval，积分榜/射手榜用Refresh下拉刷新。同一个APP中不同页面可以混用两种策略。

### 26a. aboutToAppear 异步阻塞陷阱

**症状**：页面加载后白屏/loading转圈3-13秒才显示内容。
**根因**：`aboutToAppear` 声明为 `async`，`await` 串行阻塞UI渲染。
**修复**：改为 `void aboutToAppear()`，同步操作先执行，异步操作放 `.then()` 链。

**铁律**：
- `aboutToAppear()` 必须是 `void`，不是 `Promise<void>`
- 同步数据（骨架）和 `isLoading=false` 必须在第一个 `.then()` 之前
- 异步操作（API、Preferences）放 `.then()` 链中不阻塞UI
- `.catch()` 必须带显式类型 `(_e: Error) => {}`

详见 `references/auto-refresh-with-request-guard.md`。

### 23. 秒加载模式：同步骨架 + 异步刷新（首屏性能优化）

**核心教训**：`await fetchAPI()` / `await loadUserPrefs()` / `await calculateScores()` 会让页面白屏等3-13秒。拆成同步骨架+异步刷新，首屏瞬间显示。

```ets
// ❌ 慢——等所有异步操作完成才显示
async aboutToAppear(): Promise<void> {
  this.matches = await WorldCupApi.fetchMatches();  // 3-4秒白屏
  await UserStore.getCurrentUser(ctx);              // +1秒
  await this.loadUserData();                        // +1秒
  await this.autoCalculateScores();                 // +5-10秒 ← 总13秒！
  this.isLoading = false;
}

// ✅ 快——aboutToAppear不用async，骨架秒显示，后台异步分批刷新
aboutToAppear(): void {
  // 1. 本地数据优先（Preferences，~50ms，不阻塞UI）
  const ctx = getContext(this);
  UserStore.getCurrentUser(ctx).then(async (phone: string) => {
    if (phone.length > 0) {
      this.currentUser = phone;
      this.isRegistered = true;
      await this.loadUserData();  // 本地预测记录秒加载
    }
  }).catch((_e: Error) => { /* 用户未注册 */ });

  // 2. 同步骨架秒显示（0ms网络等待）
  this.allMatches = WorldCupApi.getMatches();
  this.isLoading = false;  // ← 页面立即显示

  // 3. 后台异步刷比分+自动计分（不阻塞UI）
  WorldCupApi.refreshScores(this.allMatches).then(async () => {
    if (this.isRegistered) {
      await this.autoCalculateScores();  // 后台慢慢算，用户无感
    }
  }).catch((_e: Error) => { /* API失败，缓存数据兜底 */ });
}
```

**关键原则**：
- `aboutToAppear(): void`（不要 `async aboutToAppear(): Promise<void>`）——去掉async，不让生命周期等异步操作
- 本地数据（Preferences）用 `.then()` 链异步加载，不阻塞UI
- API请求 + 重计算（计分/统计）放在 `.then()` 尾部，页面显示后才执行
- `.catch()` 必须显式标注 `: Error` 类型（ArkTS `arkts-no-any-unknown` 规则）

**API层拆分**：
```ets
export class WorldCupApi {
  // 同步方法——返回硬编码数据，0网络等待
  static getMatches(): MatchInfo[] {
    return buildFallbackMatches();  // 遍历SCHEDULE常量构建
  }

  // 异步方法——后台刷新动态数据（比分/状态）
  static async refreshScores(matches: MatchInfo[]): Promise<void> {
    const data = await tryFetchJson('/get/games');
    if (data === null) return;
    // 建立ID索引，将API的score/status合并到matches数组
    // matches是引用类型，修改后UI自动感知更新
  }
}
```

**关键点**：
- `getMatches()` 是同步的，不返回Promise，不能await
- `refreshScores()` 是异步的，**不加await**，让它在后台跑
- `matches` 数组是引用类型，`refreshScores` 修改其中元素的属性后，`@State` 自动触发UI刷新
- 所有页面（SchedulePage、MainPage、PredictionPage等）都要改，保持一致
- 多步异步初始化（读用户→读预测→刷API→算分）用 `.then()` 链串联，不要让 `aboutToAppear` 承担任何 `await`

**适用场景**：
- 首屏数据量大（>50条），API响应慢（>1秒）
- 数据分静态（时间、名称）和动态（比分、状态）两部分
- 静态数据可以hardcode，动态数据需要实时刷新
- 页面初始化有多个异步步骤（本地Preferences + API + 本地计算）

## 27. @State不检测数组元素属性修改（新！）

`@State` 只检测数组引用变化，不检测对象属性就地修改。`refreshScores()` 后必须 `this.matches = [...this.matches]` 触发UI刷新。详见 `references/worldcup2026-project-patterns.md`。

## 28. aboutToAppear同步骨架+异步后台（新！）

`aboutToAppear` 不要串行await阻塞UI。先设 `isLoading=false` 显示骨架，后台 `.then()` 异步刷新。注意 `.catch((_e: Error) => {})` 需显式类型标注。详见 `references/worldcup2026-project-patterns.md`。

## 29. 比赛时间查权威来源（新！）

多时区赛事用中文权威来源（worldcup2026cn.com）的北京时间硬编码，不要尝试从API local_date推算。详见 `references/worldcup2026-project-patterns.md`。

## 27. 排行榜/用户列表隐私设计

显示排行榜或用户列表时，只展示昵称，不展示手机号、邮箱等敏感信息。注册时昵称应设为必填项，避免自动生成的"球迷XXXX"暴露手机号尾号。

```ets
// ❌ 排行榜同时显示昵称和手机号
Text(user.nickname).fontSize(14)
Text(user.phone).fontSize(11).fontColor('#9E9E9E')  // 泄露隐私

// ✅ 只显示昵称
Text(user.nickname).fontSize(14)
```

### 16. ForEach需类型化

```typescript
// ✅ 正确
ForEach(this.messageList, (message: ChatMessage, index: number) => {
  ListItem() { ... }
}, (message: ChatMessage, index: number) => index.toString())
```

### 9. null ≠ undefined（最容易踩的坑）

**ArkTS严格模式下 `null` 和 `undefined` 是完全不同的类型，不能互换！**

```typescript
interface BracketMatch {
  winner: TeamInfo | null;      // 只能赋 null
  penaltyScore?: string;        // 类型是 string | undefined，不能赋 null
}

// ❌ 错误 — winner 类型是 TeamInfo | null，不能用 undefined
{ winner: undefined }   // → "Type 'undefined' is not comparable to type 'TeamInfo | null'"

// ❌ 错误 — penaltyScore 类型是 string | undefined，不能用 null
{ penaltyScore: null }  // → "Type 'null' is not comparable to type 'string | undefined'"

// ✅ 正确
{ winner: null }         // 匹配 TeamInfo | null
// penaltyScore 不写     // 默认 undefined，匹配 string | undefined
```

**遇到复杂对象字面量 `as Type` 转换失败时**，用工厂函数构建数据避免一行写完所有字段。

### 9b. 全屏弹窗必须用Stack做根节点（不是Column内嵌）

**核心教训**：弹窗（Dialog/Modal）不能嵌套在Column内部——会被Column的布局约束裁剪，导致弹窗不显示或无法交互。

```ets
// ❌ 错误——弹窗嵌在Column内部，被布局约束
build() {
  Column() {
    // ... 主内容 ...
    if (this.showDialog) {
      this.MyDialog()  // 被Column约束，不是全屏覆盖
    }
  }
  .width('100%').height('100%')
}

// ✅ 正确——Stack做根节点，弹窗作为独立层覆盖
build() {
  Stack() {
    Column() {
      // ... 主内容 ...
    }
    .width('100%').height('100%').backgroundColor('#1A1A2E')

    // 弹窗层——Stack的第二个子节点，自动覆盖在主内容之上
    if (this.showDialog) {
      this.MyDialog()
    }
  }
  .width('100%').height('100%')
}

@Builder
MyDialog() {
  Column() {
    // ⚠️ 内层Column必须显式设置宽高和背景色，否则弹窗不可见！
    Column() {
      Text('标题').fontSize(18)
      // ... 弹窗内容（白色卡片）...
    }
    .width('85%').backgroundColor('#FFFFFF').borderRadius(16).padding(24)
  }
  .width('100%').height('100%').backgroundColor('#66000000')  // 半透明遮罩
  .justifyContent(FlexAlign.Center)  // 居中，或 FlexAlign.End 底部弹出
  .onClick(() => { this.showDialog = false; })  // 点背景关闭
}
```

**规则**：只要页面有弹窗/浮层/Modal，`build()` 的根节点就必须是 `Stack`，弹窗和主内容是 Stack 的两个子节点。

### 9c. ❌ 不要用 bindContentCover 做授权/审批弹窗（踩坑实录）

ArkTS API 24 的 `bindContentCover(isShow, builder)` 在以下场景有严重表现问题：

| 问题 | 表现 | 原因 |
|------|------|------|
| 表达式不响应 | `bindContentCover(A\|\|B, ...)` 中 `B` 变 true 也不弹窗 | `\|\|` 是计算表达式，不是一个 `@State` 变量的引用。ArkTS 不追踪表达式变更 |
| 单 overlay 限制 | 只能挂一个 `bindContentCover` | 无法同时支持设置面板和授权面板 |
| 内容不可见 | 弹窗层背景透明，白色卡片看不见 | build() 中 `Column { 卡片 }` 外层 Column 没有显式 `.width('100%').height('100%').backgroundColor(...)` |

**替代方案：`Stack` + `if(showDialog)` 条件渲染**

```ets
// ❌ bindContentCover（不可靠）
build() {
  Column() { ... }
  .bindContentCover(this.showDialog, this.DialogLayer())
}

// ✅ Stack + 条件渲染（可靠）
build() {
  Stack() {
    Column() { /* 主内容 */ }
    if (this.showDialog && !this.showSettings) {
      this.ApproveOverlay()  // 覆盖层
    }
  }
}

@Builder
ApproveOverlay() {
  Column() {  // 遮罩层——宽高背景色必设！
    Column() { /* 白色卡片内容 */ }
    .width('100%').padding(24).backgroundColor('#FFFFFF')
    .borderRadius({ topLeft: 20, topRight: 20 })
  }
  .width('100%').height('100%').backgroundColor('#66000000')
  .justifyContent(FlexAlign.End)
}
```

### 10. @Entry build() 必须单根节点

```typescript
// ❌ 错误 — 两个并列根节点
build() {
  Column() { /* 主内容 */ }
  if (this.showDialog) { Column() { /* 弹窗 */ } }
}

// ✅ 正确 — Stack 包裹
build() {
  Stack() {
    Column() { /* 主内容 */ }
    if (this.showDialog) { Column() { /* 弹窗 */ }.width('100%').height('100%') }
  }
  .width('100%').height('100%')
}
```

### 11. getContext() 已弃用

```typescript
// ❌ 弃用
const ctx = getContext(this);

// ✅ 推荐
import { common } from '@kit.AbilityKit';
const ctx: common.UIAbilityContext = getContext(this) as common.UIAbilityContext;
// 或更规范
const ctx: common.UIAbilityContext = this.getUIContext().getHostContext() as common.UIAbilityContext;
```

---

## 网络请求：WebSocket 长连接

### 服务端（xiaoq-api / FastAPI + Uvicorn）

```python
from fastapi import WebSocket, WebSocketDisconnect
import asyncio

class ConnectionManager:
    def __init__(self):
        self.active_connections: dict[str, WebSocket] = {}
        self.lock = asyncio.Lock()

    async def connect(self, client_id: str, websocket: WebSocket):
        await websocket.accept()
        async with self.lock:
            self.active_connections[client_id] = websocket

    async def disconnect(self, client_id: str):
        async with self.lock:
            self.active_connections.pop(client_id, None)

    async def push(self, client_id: str, message: dict):
        async with self.lock:
            ws = self.active_connections.get(client_id)
        if ws:
            try:
                await ws.send_json(message)
            except Exception:
                await self.disconnect(client_id)

manager = ConnectionManager()

@app.websocket("/ws/push")
async def websocket_push(websocket: WebSocket):
    client_id = websocket.query_params.get("client_id", f"anon-{id(websocket)}")
    await manager.connect(client_id, websocket)
    try:
        await websocket.send_json({"type": "connected", "client_id": client_id})
        while True:
            data = await websocket.receive_json()
            if data.get("type") == "ping":
                await websocket.send_json({"type": "pong"})
            elif data.get("type") == "approve_response":
                print(f"[WS] 授权回复 from {client_id}: {data}")
    except WebSocketDisconnect:
        await manager.disconnect(client_id)
```

### 客户端（HarmonyOS NEXT ArkTS）

```typescript
import { webSocket } from '@kit.NetworkKit';

const WS_PING_INTERVAL: number = 15000;  // 15秒心跳（隧道不稳定环境需更短间隔检测断线）
const WS_RECONNECT_DELAY: number = 2000; // 断线重连延时2秒（原5秒，隧道下需更快重连）

export interface WsMessage {
  type: string;
  id?: string;
  title?: string;
  description?: string;
  details?: Object[];
  requester?: string;
  body?: string;
  approved?: boolean;
  timestamp?: number;
}

export class WebSocketClient {
  private ws: webSocket.WebSocket | null = null;
  private url: string = '';
  private clientId: string = '';
  private connected: boolean = false;
  private pingTimer: number | undefined = undefined;
  private reconnectTimer: number | undefined = undefined;
  private onMessage: ((msg: WsMessage) => void) | undefined = undefined;
  private onConnect: (() => void) | undefined = undefined;
  private onDisconnect: (() => void) | undefined = undefined;

  constructor(clientId: string) { this.clientId = clientId; }

  public setCallbacks(onMessage: (msg: WsMessage) => void,
                      onConnect?: () => void,
                      onDisconnect?: () => void): void {
    this.onMessage = onMessage;
    this.onConnect = onConnect;
    this.onDisconnect = onDisconnect;
  }

  public connect(url: string): void {
    this.url = url;
    this.doConnect();
  }

  private doConnect(): void {
    if (this.ws !== null) this.close();
    try {
      const fullUrl: string = this.url + '/ws/push?client_id=' + this.clientId;
      this.ws = webSocket.createWebSocket();

      this.ws.on('open', () => {
        this.connected = true;
        if (this.onConnect !== undefined) this.onConnect();
        this.startPing();
      });
      this.ws.on('message', (err: Error, data: string | ArrayBuffer) => {
        if (err !== null && err !== undefined) return;
        try {
          const msg: WsMessage = JSON.parse(data as string) as WsMessage;
          if (msg.type === 'pong') return;
          if (this.onMessage !== undefined) this.onMessage(msg);
        } catch (error) { console.error('WS parse error: ' + error); }
      });
      this.ws.on('close', () => {
        this.connected = false;
        this.stopPing();
        if (this.onDisconnect !== undefined) this.onDisconnect();
        this.scheduleReconnect();
      });
      this.ws.on('error', () => { this.connected = false; this.stopPing(); this.scheduleReconnect(); });
      this.ws.connect(fullUrl, (err: Error) => {
        if (err !== null && err !== undefined) this.scheduleReconnect();
      });
    } catch (error) { this.scheduleReconnect(); }
  }

  public send(msg: WsMessage): void {
    if (this.ws !== null && this.connected) {
      this.ws.send(JSON.stringify(msg));
    }
  }

  private startPing(): void {
    this.stopPing();
    this.pingTimer = setInterval(() => this.send({ type: 'ping' }), WS_PING_INTERVAL);
  }
  private stopPing(): void {
    if (this.pingTimer !== undefined) { clearInterval(this.pingTimer); this.pingTimer = undefined; }
  }
  private scheduleReconnect(): void {
    if (this.reconnectTimer !== undefined) return;
    this.reconnectTimer = setTimeout(() => {
      this.reconnectTimer = undefined;
      if (!this.connected) this.doConnect();
    }, WS_RECONNECT_DELAY);
  }

  public close(): void {
    this.stopPing();
    if (this.reconnectTimer !== undefined) { clearTimeout(this.reconnectTimer); this.reconnectTimer = undefined; }
    if (this.ws !== null) {
      this.ws.off('open'); this.ws.off('message'); this.ws.off('close'); this.ws.off('error');
      this.ws.close(); this.ws = null;
    }
    this.connected = false;
  }
}
```

### 集成到 ChatPage

```typescript
import { WebSocketClient, WsMessage } from '../utils/WebSocketClient.ets';

// 启动时建连
private setupWebSocket(baseUrl: string): void {
  const wsBase: string = baseUrl.startsWith('https') ? baseUrl.replace('https', 'wss') : baseUrl.replace('http', 'ws');
  this.wsClient.setCallbacks(
    (msg: WsMessage) => {
      if (msg.type === 'approve_request') this.handleApproveRequest(msg);
      else if (msg.type === 'notification') this.showToast(msg.body as string);
    },
    () => { console.info('WS connected'); },
    () => { console.info('WS disconnected'); }
  );
  this.wsClient.connect(wsBase);
}
```

## 华为 Push Kit 推送（系统级通知）

**仅限鸿蒙NEXT**。当 APP 被杀后，通过华为 Push Kit 发送系统级通知栏消息。个人开发者实名认证后有免费额度。

### 服务端（xiaoq-api）

| 端点 | 用途 |
|------|------|
| `POST /api/push/register` | APP注册push token |
| `_push_to_huawei()` | 调用华为Push Kit REST API |

配置在 `~/.hermes/.env`：

```
PUSH_APP_ID=xxx
PUSH_APP_SECRET=xxx
```

当消息到达 `POST /api/push` 时，自动同时触发三条路径：
1. SSE 广播（推送到实时网页）
2. WebSocket 广播（推送到连接中的APP）
3. Push Kit REST API（推送到系统通知栏）

### 客户端（HarmonyOS NEXT）

```typescript
// EntryAbility.ets
import { pushService } from '@kit.PushKit';

onCreate(): void {
  pushService.getToken().then((token: string) => {
    if (token !== '') this.registerPushToken(token);
  });
}

private async registerPushToken(token: string): Promise<void> {
  // POST /api/push/register 注册到服务器
}
```

详见 `references/push-kit-integration.md`。

### OAuth 凭证踩坑（2026-06-15 验证）

华为 Push Kit REST API 的 OAuth2.0 `client_credentials` 认证对凭证来源敏感：

| 来源 | client_id | client_secret | OAuth 结果 |
|------|-----------|--------------|-----------|
| 应用配置 → 通用凭证 | APP ID | Client Secret (64 hex) | ❌ error 1101 |
| API密钥 → Service Account | 118035617 | c17bc... (32 hex) | ❌ error 1101 |
| API密钥 → OAuth客户端 | OAuth Client ID | OAuth Client Secret | ❌ error 1101 |

**结论**：在没有找到正确的 App Secret 之前，Push Kit REST API 不可用。替代方案：WebSocket 长连接推送（APP前台时正常），或在 AGC 中寻找专门的 Push Kit Secret 字段。

## 网络请求：用 rcp 代替 http
  this.hermesApi.destroy();
  this.wsClient.close();
}

// 网络切换时重连
private async saveSettings(): Promise<void> {
  // ... 保存逻辑
  this.wsClient.close();
  this.setupWebSocket(detectedUrl);
}
```

**要点**：
- 30秒ping保活，5秒后断线重连
- WiFi用 `ws://`，5G用 `wss://`（通过URL替换自动推导）
- `aboutToDisappear` 必须清理WebSocket，防内存泄漏
- 网络切换（WiFi↔5G）时关闭旧连接建新连接

### ⚠️ WS 坑：on('error') 也必须调 scheduleReconnect()

**症状**：WebSocket 断线后不重连，`on('close')` 和 `on('error')` 都触发了，但只有 `on('close')` 调了 `scheduleReconnect()`。如果只收到 error 事件（隧道超时、连接被拒等场景），不会安排重连，WS 永远断着。

```typescript
// ❌ 错误——error 后静默终止
this.ws.on('error', () => {
  this.connected = false;
  this.stopPing();
  // 忘了调 scheduleReconnect()！
});

// ✅ 正确——error 后也安排重连
this.ws.on('error', () => {
  this.connected = false;
  this.stopPing();
  this.scheduleReconnect();  // ← 必须加
});
```

**调试**：journalctl 看 xiaoq-api 日志，如果 `connection closed` 后没有新的 `accepted`，说明重连没触发。

## 网络请求：用 rcp 代替 http

**核心建议**：使用 `@kit.RemoteCommunicationKit.rcp` 而非 `@kit.NetworkKit.http`。

### rcp vs http 对比

| 特性 | `@kit.NetworkKit.http` | `@kit.RemoteCommunicationKit.rcp` |
|------|----------------------|-----------------------------------|
| 连接复用 | ❌ 每次请求新建连接 | ✅ Session自动管理连接池 |
| TLS握手 | 每次请求都握手 | Session内复用，仅首次握手 |
| 延迟优化 | 7-8s（每次握手+请求） | 3-4s（复用连接+快速请求） |
| 代码量 | 多（手动管理HttpRequest） | 少（Session自动管理） |
| 推荐度 | 基本可用 | **官方推荐，持续迭代** |

### rcp用法

```typescript
import { rcp } from '@kit.RemoteCommunicationKit';

// 1. 创建Session（一次，复用所有请求）
const sessionConfig: rcp.SessionConfiguration = {
  baseAddress: 'https://xiaoq.xiao-q.com',    // 基址
  timeout: 120000                               // ⚠️ 总超时(ms), 必设! AI模型可能10-30秒才响应
};
const session: rcp.Session = rcp.createSession(sessionConfig);

// 2. 发起请求
const headers: Record<string, string> = {
  'Content-Type': 'application/json',
  'Authorization': 'Bearer ' + apiKey
};
const body: Record<string, Object> = {
  'model': 'current',
  'messages': messageList
};
const request: rcp.Request = new rcp.Request('/v1/chat/completions', 'POST', headers, body);
const response: rcp.Response = await session.fetch(request);
const result: Record<string, Object> = await response.toJSON() as Record<string, Object>;
```

### 注意事项

- **Session必须复用**：每次请求都创建新Session会导致第16次请求后失败（hvigor限制）。应当作为类成员保存，重复使用
- **baseAddress**使用后，request URL用相对路径
- **updateConfig时需要重建Session**：先`session.close()`再`rcp.createSession(newConfig)`
- **⚠️ 必须设置 timeout**：rcp Session 默认无超时或超时极短。AI模型推理可能10-30秒才响应，不设超时会报 `timeout was reached`。特别在5G公网路径（Cloudflare Tunnel → 反向代理 → Hermes Gateway → 模型）下，每跳都增加延迟，更容易超时。设 `timeout: 120000`（2分钟）保险。

**⚠️ timeout 兼容性**：如果编译报 `Type '{ baseAddress: string; timeout: number; }' is not assignable to type 'SessionConfiguration'`，说明当前 SDK 版本的 `SessionConfiguration` 接口不支持 `timeout` 属性（API 24 某些构建中可能缺失）。移除 `timeout` 字段即可，用 `rce.fetch` 的第二参数 `timeout` 替代。
- module.json5中需添加`ohos.permission.INTERNET`权限

### 简单GET请求（http，适合外部API数据拉取）

对于只需要GET外部API（如体育数据、天气、公开接口）的场景，用 `@kit.NetworkKit.http` 更简单：

```typescript
import { http } from '@kit.NetworkKit';

async function fetchJson(url: string): Promise<Object> {
  const req = http.createHttp();
  try {
    const resp = await req.request(url, {
      method: http.RequestMethod.GET,
      connectTimeout: 10000,
      readTimeout: 15000,
      expectDataType: http.HttpDataType.OBJECT  // 自动解析JSON
    });
    return resp.result as Object;
  } finally {
    req.destroy();  // ⚠️ 必须销毁，否则连接泄漏
  }
}

// 使用
const data = await fetchJson('https://worldcup26.ir/get/teams') as Record<string, Object>;
const teams = data['teams'] as ApiTeam[];
```

**何时用http vs rcp**：
- 简单GET拉取公开数据 → http（代码少，无状态）
- 需要认证、连接复用、频繁请求 → rcp（Session管理连接池）

---

## 项目结构

### 最小可行结构

```
XiaoQ/
├── AppScope/
│   ├── app.json5                      ← 包名、版本、图标
│   └── resources/base/
│       ├── element/string.json         ← app_name
│       └── media/app_icon.png          ← 应用图标
├── entry/
│   ├── build-profile.json5             ← 模块级构建配置（apiType + targets）
│   ├── src/main/
│   │   ├── ets/
│   │   │   ├── entryability/EntryAbility.ets  ← APP入口
│   │   │   ├── pages/                        ← 页面组件
│   │   │   └── utils/                         ← 工具类
│   │   ├── module.json5                ← 模块配置 + 权限声明
│   │   └── resources/base/
│   │       ├── element/                ← 字符串、颜色
│   │       ├── media/                   ← 图标
│   │       └── profile/main_pages.json  ← 页面路由
│   ├── hvigorfile.ts
│   └── oh-package.json5
├── build-profile.json5                 ← 根级：签名 + SDK版本 + 模块列表
├── hvigor/hvigor-config.json5
├── hvigorfile.ts
└── oh-package.json5
```

### 关键配置文件详解

**AppScope/app.json5** —— 应用级配置：
```json5
{
  "app": {
    "bundleName": "xiaoq.debug.profile",      // 包名（需与Profile匹配）
    "vendor": "xiaoq",
    "versionCode": 1000000,                    // 每次版本递增
    "versionName": "1.0.0",
    "icon": "$media:app_icon",
    "label": "$string:app_name"
  }
}
```

**entry/src/main/module.json5** —— 模块配置（⚠️ 必填字段已标注）：
```json5
{
  "module": {
    "name": "entry",
    "type": "entry",
    "mainElement": "EntryAbility",
    "deliveryWithInstall": true,           // ⚠️ 必填！缺少报 00303038
    "deviceTypes": ["phone"],
    "pages": "$profile:main_pages",
    "requestPermissions": [
      { "name": "ohos.permission.INTERNET" }
    ],
    "abilities": [
      {
        "name": "EntryAbility",
        "srcEntry": "./ets/entryability/EntryAbility.ets",
        "icon": "$media:app_icon",
        "label": "$string:EntryAbility_label",
        "startWindowIcon": "$media:app_icon",  // ⚠️ 必填！缺少报 00303038
        "startWindowBackground": "$color:start_window_background",
        "exported": true,
        "skills": [
          { "entities": ["entity.system.home"], "actions": ["action.system.home"] }
        ]
      }
    ]
  }
}
```

**entry/src/main/resources/base/profile/main_pages.json** —— 页面路由：
```json5
{
  "src": ["pages/ChatPage"]
}
```

---

## EntryAbility模板（APP入口）

```typescript
import { UIAbility } from '@kit.AbilityKit';
import { window } from '@kit.ArkUI';

export default class EntryAbility extends UIAbility {
  onWindowStageCreate(windowStage: window.WindowStage): void {
    windowStage.loadContent('pages/ChatPage', (err) => {
      if (err.code) {
        console.error('load page failed');
      }
    });
  }
}
```

---

## AppGallery 上架发布

### API Level 限制

AGC 自检系统对手机应用有 API Level 上限限制。截至 2026-06 实测：
- **手机应用**: compatibleSdkVersion API Level ≤ 22（对应 `6.0.2(22)`）
- **PC/2in1应用**: API Level ≤ 21

超过限制会报：`上架自检启动失败，当前仅支持API Level≤22的手机`

**修复**：`build-profile.json5` 中 `compatibleSdkVersion` 降级到 `6.0.2(22)`，`targetSdkVersion` 保持高版本不变。

### SDK 版本映射

| HarmonyOS | API Level | compatibleSdkVersion |
|-----------|-----------|---------------------|
| 5.0.0 | 12 | `5.0.0(12)` |
| 5.0.1 | 13 | `5.0.1(13)` |
| 5.0.2 | 14 | `5.0.2(14)` |
| 6.0.0 | 20 | `6.0.0(20)` |
| **6.0.2** | **22** | **`6.0.2(22)`** ← AGC当前上限 |
| 6.1.0 | 23 | `6.1.0(23)` |
| 6.1.1 | 24 | `6.1.1(24)` ← DevEco Studio默认 |

### Release 签名直装失败（9568322）

release 签名的 `.app` 包**不能通过 hdc 直装到手机**，报错 `signature verification failed due to not trusted app source`。release 包只能通过应用市场分发。

**开发调试一直用 debug 签名**，release 签名只在上架时使用。

### Build APP(s) vs Build HAP(s)

- **Build HAP(s)**: 生成 `.hap`（单模块），debug/直装用
- **Build APP(s)**: 生成 `.app`（应用包），上架 AGC 用

DevEco Studio 的 Build Mode 下拉框在某些配置下不可见。可以通过 **Build → Build Hap(s)/APP(s) → Build APP(s)** 菜单直接操作。

### 上架流程概要

1. AGC 申请发布证书 + 发布 Profile（不需要绑定设备）
2. DevEco Studio 配置 release 签名（p12 + release.cer + release.p7b）
3. 降低 compatibleSdkVersion 到 ≤22
4. Build → Build APP(s) 生成 .app
5. AGC 上传 .app → 填应用信息（截图、简介、隐私政策）→ 提交审核
6. 审核 1-3 个工作日

---

## 真机调试

### DevEco Studio操作

1. 打开项目：`File → Open` → 选择项目根目录
2. 等自动同步完成（右下角进度条）
3. 配置签名密码：`File → Project Structure → Signing Configs` → 选签名方案 → 填密码 → OK
4. 顶部工具栏：模块选`entry`、设备选手机 → 点绿色 **Run ▶️**

**注意**：第一次运行后如果改了bundleName，DevEco Studio缓存里可能还是旧包名。此时：
- 手机上的APP是正确安装的（从桌面图标打开即可）
- 点`Build → Clean Project`清缓存，或重启DevEco Studio
- 也可直接手动点桌面图标打开

### 常见编译/运行报错

| 报错 | 原因 | 修复 |
|------|------|------|
| `Can not find build config file build-profile.json5 at 'entry'` | 缺少模块级build-profile.json5 | 创建`entry/build-profile.json5`，内容`{"apiType":"stageMode","targets":[{"name":"default"}]}` |
| `platform version and API version do not match` | `compatibleSdkVersion`版本号错误 | 检查SDK版本（DevEco Studio About），格式为`"X.X.X(API)"`如`"6.1.1(24)"` |
| `bundleName does not match signing config` | Profile绑定的包名与项目不一致 | 统一为AppGallery Connect注册的包名 |
| `Resource Pack Error: Invalid qualifier key 'media'` | `media`目录位置错误 | 必须放在`resources/base/media/`下，不能直接`resources/media/` |
| `Invalid JSON file: mediaQuery.json` | 多余的或格式错误的资源文件 | 不需要mediaQuery直接删除 |
| ArkTS strict mode errors | 违反严格模式规则 | 见上方"ArkTS严格模式编码规范" |
| `failed to start ability` (编译成功但启动报错) | DevEco Studio缓存了旧包名 | 直接点手机桌面图标打开APP即可 |
| `hvigorw: command not found` (命令行使用时) | 项目缺少hvigorw/hvigorw.bat包装器 | **原因**：DevEco Studio"新建项目"向导自动生成这两个文件，但手动创建的项目不会自带。**修复**：创建`hvigorw`（Linux）和/或`hvigorw.bat`（Windows）脚本。参考`harmonyos-app-testing`技能的`templates/hvigorw/`模板，或在DevEco Studio通过File→New→Project重新生成 |
| `modelVersion in hvigor-config.json5 is X.X.X, and the modelVersion in oh-package.json5 is X.X.X` | 版本号不一致 | `hvigor/hvigor-config.json5`中的`modelVersion`必须和根目录`oh-package.json5`中的`modelVersion`保持一致。**两者必同步**，DevEco Studio创建项目时自动保持一致，手动复制项目时容易忘记更新 |
| `Invalid value of 'DEVECO_SDK_HOME' in the system environment path` | SDK路径格式或层级不对 | `DEVECO_SDK_HOME`应指向`sdk/`目录（包含`default/`子目录），不是`sdk/default/`。验证：`ls $DEVECO_SDK_HOME/default/openharmony/toolchains/hdc.exe` 应能找到文件 |
| `SDK component missing` | SDK结构异常或资源文件缺失 | 检查SDK目录完整性：需有`ets/`、`js/`、`native/`、`toolchains/`等子目录。如果是从低版本升级的SDK，建议重新下载 |
| `NODE_HOME is not set and no 'node' command found in PATH` | hvigorw需要node但不环境变量中 | 加`NODE_HOME=D:\Program Files\Huawei\DevEco Studio\tools\node`并加到PATH前缀 |
| `Task 'runTest' was not found` | `runTest`任务在某些hvigor版本不可用 | 先用`assembleHap`编译测试hap，再手动安装执行。或确保命令包含完整参数：`-p module=entry@default -p product=default -p buildMode=debug` |
| `Not an internal or external command: restool` (WSL运行时报错) | SDK工具链是Windows原生exe | **编译必须在Windows上进行**。WSL可以运行`hdc`检测设备、执行流水线脚本做覆盖扫描，但不能编译。鸿蒙工具链（restool.exe、ark_disasm.exe等）都是Windows PE格式 |
| `must have required property 'deliveryWithInstall'` | module.json5缺少必填字段 | 在 `module` 层级添加 `"deliveryWithInstall": true` |
| `must have required property 'startWindowIcon'` | abilities[0]缺少必填字段 | 在 ability 对象中添加 `"startWindowIcon": "$media:app_icon"` |
| `bundleName does not match signing config` (00303074) | app.json5的bundleName与Profile绑定的包名不一致 | 去AGC查Profile绑定的包名，修改app.json5的bundleName |
| `storePassword length less than 32` (00303116) | build-profile.json5的密码被覆盖为空 | 在IDE中重新填写签名密码，**之后不要再用工具覆盖此文件** |
| `install sign info inconsistent` (9568332) | 签名证书换了（如debug→release），设备上已装的app签名不一致 | **必须先卸载旧app**：`hdc shell bm uninstall -n <bundleName>`，再Run。鸿蒙不允许签名不一致覆盖安装 |
| `signature verification failed` (9568322) | 试图通过hdc直装release签名的包 | release包**不能hdc直装**，只能通过AppGallery分发。开发调试必须用debug签名 |
| API Level≤22 (AGC自检) | compatibleSdkVersion设为API 24，AGC只接受≤22 | 降到`6.0.2(22)`或更低，并在DevEco Studio中安装对应SDK |
| `Cannot find name 'Wrap'` | API 24 不含 Wrap 组件 | 用 `Flex({ direction: FlexDirection.Row, wrap: FlexWrap.Wrap })` 替代 |
| `extensionAbilities[0].type must be one of allowed values` | PushServiceAbility 不在当前 SDK 的 extensionAbilities 类型枚举中 | 不要在 module.json5 中声明，改在 EntryAbility 中通过 Push Kit SDK 的 `pushService.getToken()` 运行时初始化 |
| `signature verification failed due to not trusted app source` (9568322) | 试图用hdc直装release签名的包 | **release包不能直装**，只能通过AppGallery分发。开发调试请用debug签名，上架时才用release签名构建 |
| `failed to install bundle` (various) | 先卸旧版再装新版仍失败 | 确认`app.json5`的`bundleName`与Profile绑定的包名完全一致。确认p12密码在IDE中已正确填写（`build-profile.json5`中`storePassword`/`keyPassword`字段长度≥32） |
| `Type 'undefined' is not comparable to type 'T \null'` | 可空类型用错null/undefined | `T \null` → 赋 `null`；`T \undefined` → 不赋值或赋 `undefined`；不能混用 |
| `Conversion of type ... may be a mistake` | 对象字面量 as Type 转换失败 | 用工厂函数构建数据，或逐字段赋值避免一行写完 |
| `Using "this" inside stand-alone functions` | 静态方法中用了this | 改为 `ClassName.method()` |
| `Indexed access types are not supported` | 用了 `Type['field']` 语法 | 直接赋值，不使用索引访问 |
| `Object literal must correspond to some explicitly declared class or interface` | 对象字面量没有显式类型 | 加 `as InterfaceName` 或声明变量类型 |
| APP闪退：Image空URL | `Image('')` 空字符串导致崩溃 | 渲染前检查 `if (url.length > 0)` |
| APP闪退：Image空URL | `Image('')` 空字符串导致崩溃 | 渲染前检查 `if (url.length > 0)` |
| `Property 'scrollable' does not exist on type 'RowAttribute'` | API 24的Row不支持`.scrollable()` | 用 `Scroll { Row() { ... } }` 包裹替代单行上的 `.scrollable()` |
| `Invalid resource directory name 'rawfile'` | `rawfile`不能放在`resources/base/rawfile/` | 平移到 `resources/rawfile/`（与 `base/` 同级） |
| `Invalid value of 'DEVECO_SDK_HOME'` | 环境变量指向的路径不存在或层级不对 | 指向 `sdk/` 根目录（含 `default/openharmony/`），不是 `sdk/default/` |
| `spawn java ENOENT` during PackageHap | 缺少Java运行环境 | 设置 `JAVA_HOME=D:\\Program Files\\Huawei\\DevEco Studio\\jbr` 并加到PATH |
| `Cannot find name 'Wrap'` | API 24 不含 Wrap 组件 | 用 `Flex({ direction: FlexDirection.Row, wrap: FlexWrap.Wrap })` 替代 |
| `extensionAbilities[].type: 'pushService' schema fail` | PushServiceAbility 不在当前 SDK extensionAbilities 类型枚举中 | 不要在 module.json5 中声明，改在 EntryAbility 中通过 Push Kit SDK 的 `pushService.getToken()` 运行时初始化 |
| 射手榜/文字乱码 | API返回Unicode智能引号 | 替换 `\\u2018\\u2019\\u201C\\u201D` 为ASCII引号 |

### 项目结构补充：hvigorw包装器

项目的**命令行入口**是 `hvigorw`（Linux/Mac）和 `hvigorw.bat`（Windows），它们是Hvigor构建工具的Wrapper，负责自动下载正确版本的hvigor引擎并转发命令。

**手动创建的项目（非DevEco Studio向导）缺少这两个文件**，命令行执行`./hvigorw clean assembleHap`会报`command not found`。

解决方案：
1. 从DevEco Studio新建一个空白项目，把这两个文件复制过来
2. 或手动创建（内容见`harmonyos-app-testing`技能的前置条件章节）

---

## 文件浏览功能（xiaoq-api集成）

小Q APP 通过 xiaoq-api 后端服务（端口 8866）提供 Obsidian Vault 文件浏览能力。之前使用端口 8089，但因 Windows 端口排除范围导致局域网不可达，已于 v1.3.0 迁移至 8866（详见下方「⚠️ Windows 端口排除陷阱」）。

### 后端API端点

| 端点 | 用途 | 返回 |
|------|------|------|
| `GET /api/files?path=` | 列出目录 | `{path, items: [{type, name, path}]}` |
| `GET /api/files/read?path=xxx.md` | 读取文件 | `{content: "...", path: "..."}` |
| `GET /api/files/search?q=关键词` | 搜索 .md 文件 | `{results: [...]}` |

### 网络架构

- `xiaoq.xiao-q.com`（Cloudflare Tunnel → `localhost:8866`）→ xiaoq-api
- xiaoq-api 同时代理 `/v1/*` 路径到 Hermes Gateway（`localhost:8642`）
- **所以聊天和文件浏览共用同一个域名**，API Key 也共用

### 关键实现模式

**跨页面共享数据**：使用 `AppStorage` 在 ChatPage、FilesPage、FileViewPage 之间共享 API Key：

```typescript
// 存（ChatPage保存时）
AppStorage.set<string>('apiKey', this.apiKey);

// 取（FilesPage/FileViewPage启动时）
const savedKey: string = AppStorage.get<string>('apiKey') as string;
if (savedKey !== undefined) {
  this.fileApi.updateConfig(PUBLIC_API, savedKey);
}
```

**跨页面导航**：使用 `router.pushUrl()` 和 `router.back()`：

```typescript
// 从ChatPage跳转到FilesPage
import { router } from '@kit.ArkUI';
router.pushUrl({ url: 'pages/FilesPage' });

// 从FileViewPage返回
router.back();
```

**页面路由注册**：所有页面必须在 `entry/src/main/resources/base/profile/main_pages.json` 中注册：

```json5
{
  "src": ["pages/ChatPage", "pages/FilesPage", "pages/FileViewPage"]
}
```

**文件浏览页面结构**：面包屑导航 + 文件列表（目录/文件图标区分）+ 点击目录进入 / 点击文件阅读。

**文件阅读页面**：纯文本显示，支持 .md / .json / .txt 等文本格式，滚动阅读。

### 无网络时注意事项

文件浏览通过 xiaoq-api（端口 8866）提供。与聊天 API（端口 8642）不同，注意区分两个端口。详情见下方「WiFi自动切换」章节中关于「所有页面需一致检测SSID」的陷阱，以及「⚠️ Windows 端口排除陷阱」中关于端口选择的经验。

详见 templates/harmonyos-next-file-browser/ 下的 FilesPage.ets、FileViewPage.ets 和 FileApi.ets 模板源码，以及 references/xiaoq-api-file-endpoints.md 参考文档。

## 局域网直连优化（WiFi自动切换）

当手机和Hermes运行在同一个局域网时，直接走内网IP可以大幅降低延迟（从7-8秒降到<10ms）。原理是通过检测当前连接的WiFi SSID，自动切换API地址。

### WiFi SSID检测

```typescript
import { wifiManager } from '@kit.ConnectivityKit';

async function detectApiUrl(): Promise<string> {
  try {
    const linkedInfo: wifiManager.WifiLinkedInfo = await wifiManager.getLinkedInfo();
    const currentSsid: string = linkedInfo.ssid;
    if (currentSsid === 'HUAWEI-404') {
      return 'http://192.168.3.87:8642';  // 局域网直连
    }
    return 'https://xiaoq.xiao-q.com';    // 公网
  } catch (error) {
    return 'https://xiaoq.xiao-q.com';    // 无法获取WiFi信息时走公网
  }
}
```

**注意**：需要在 `module.json5` 中添加 `ohos.permission.GET_WIFI_INFO` 权限。

### FilesPage WiFi自动切换

文件浏览页也需要同样的WiFi检测，但注意 `detectApiUrl` 必须是 `async` 方法（返回 `Promise<string>`），因为 `wifiManager.getLinkedInfo()` 是异步的。不存在同步版本 `getLinkedInfoSync()`。

```typescript
private async detectFilesApiUrl(): Promise<string> {
  try {
    const linkedInfo: wifiManager.WifiLinkedInfo = await wifiManager.getLinkedInfo();
    if (linkedInfo.ssid === HOME_WIFI_SSID) {
      return LOCAL_FILE_API;      // 如 http://192.168.3.87:8866（xiaoq-api端口）
    }
  } catch (error) { /* 走公网 */ }
  return PUBLIC_FILE_API;         // 如 https://xiaoq.xiao-q.com
}
```

然后在 `aboutToAppear` 中通过 `.then()` 链式调用（不能直接 await——`aboutToAppear` 不强制 async）：

### ⚠️ 常见陷阱：所有发起网络请求的页面必须一致检测SSID

**症状**：文件列表能加载，但点开文件就失败。聊天能发消息，但文件阅读页一直转圈。

**根因**：`FilesPage.ets` 有 WiFi 检测，但 `FileViewPage.ets` 没有——后者硬编码了公网地址。在家 WiFi 下，公网 tunnel 因 NAT hairpin（内网设备访问自己公网IP，大部分家用路由器不支持）而不可达，导致文件打开失败。

**修复**：每个发起网络请求的页面都必须独立调用 `detectApiUrl()`。用同一个常量文件和同一套检测函数：

```typescript
// 每个页面都要写！不要只在一个页面里检测然后传给另一个页面
private async detectApiUrl(): Promise<string> {
  try {
    const linkedInfo = await wifiManager.getLinkedInfo();
    if (linkedInfo.ssid === HOME_WIFI_SSID) return LOCAL_API;
  } catch (error) { /* 走公网 */ }
  return PUBLIC_API;
}
```

在 `aboutToAppear` 中使用 `.then()` 链式调用：

```typescript
aboutToAppear(): void {
  this.detectApiUrl().then((apiUrl: string) => {
    this.fileApi.updateConfig(apiUrl, this.apiKey);
    this.loadContent();
  });
}
```

**检查清单**：项目中所有包含网络请求的页面（ChatPage、FilesPage、FileViewPage 等）都要有 WiFi 检测。

**检查清单（扩展版）**——每次改局域网配置后执行：
- [ ] 所有页面（ChatPage/FilesPage/FileViewPage）的 LOCAL_API 端口号**一致**
- [ ] LOCAL_API 端口号与 `server.py` 或 `xiaoq-api.service` 的 `PORT` 环境变量一致
- [ ] Windows `netsh interface portproxy` 有此端口的转发规则
- [ ] Windows 防火墙有入站规则放行此端口
- [ ] 端口不在 `netsh interface ipv4 show excludedportrange` 的保留范围内
- [ ] 手机连上WiFi后，浏览器访问 `http://192.168.3.87:<PORT>/health` 能返回 JSON

### 全自动流程图

```
APP启动 → 检测WiFi → 是 HUAWEI-404？ → 是 → 局域网直连 (http://192.168.3.87:8642)
                    ↓ 否
                  走公网 (https://xiaoq.xiao-q.com)
```

每次发消息前都重新检测一次，防止用户从家里走到外面时WiFi切换。

### 5G公网路径优化：Cloudflare Tunnel路径路由

**痛点**：5G下聊天请求走 `xiaoq.xiao-q.com/v1/chat/completions`，原配置 tunnel → xiaoq-api(8866) → 反向代理 → Hermes Gateway(8642)，多了一层代理转发，每轮对话多2-3秒。

**优化**：利用 Cloudflare Tunnel 的 path-based routing，让 `/v1/*` 请求直达 Hermes Gateway，其他路径（`/api/files/*`、`/health`等）继续走 xiaoq-api。

```yaml
# ~/.cloudflared/config.yml
  - hostname: xiaoq.xiao-q.com
    path: /v1/*                          # ← 聊天API直通Hermes
    service: http://localhost:8642        # ← Hermes Gateway
    originRequest:
      noTLSVerify: true
  - hostname: xiaoq.xiao-q.com
    service: http://localhost:8866        # ← xiaoq-api (文件API等)
```

<details>
<summary>完整 tunnel config</summary>

```yaml
tunnel: d48bbd9d-8f68-4949-8f5e-b31294a134dc
credentials-file: /home/xiaoq/.cloudflared/d48bbd9d-8f68-4949-8f5e-b31294a134dc.json

ingress:
  - hostname: hermes.xiao-q.com
    service: http://localhost:8787
    originRequest:
      noTLSVerify: true
  - hostname: xiaoq.xiao-q.com
    path: /v1/*
    service: http://localhost:8642
    originRequest:
      noTLSVerify: true
  - hostname: xiaoq.xiao-q.com
    service: http://localhost:8866
  - hostname: price.xiao-q.com
    service: http://localhost:8088
  - service: http_status:404
```
</details>

**效果对比**（APP端不用改代码，URL不变）：

```
优化前: 手机 → tunnel → xiaoq-api(8866) → proxy → Hermes(8642)   +1跳 ~2-3s
优化后: 手机 → tunnel → Hermes(8642)                             直达
```

**验证**：`/v1/models` 返回401（Hermes Gateway需要auth）= ✅；`/health` 返回200（xiaoq-api）= ✅

**注意**：path matching 按顺序生效。先匹配 `/v1/*`，其余走第二条。重启 cloudflared 后生效。DNS 记录无需额外配置。

### 网络模式显示

在设置面板显示当前网络模式（🏠 本地网络 / 🌐 公网），方便用户确认走的是哪条路径。

## Windows端口转发（WSL2局域网访问）

### 问题背景

Hermes运行在WSL2中，WSL2使用NAT网络（172.27.x.x），与Windows主机的物理网络（192.168.x.x）不在同一子网。Windows不会自动将外部LAN请求转发到WSL端口。

### 解决方案：netsh portproxy

```powershell
# 添加端口转发规则
netsh interface portproxy add v4tov4 listenaddress=192.168.3.87 listenport=8642 connectaddress=127.0.0.1 connectport=8642

# 查看已有规则
netsh interface portproxy show all

# 删除规则
netsh interface portproxy delete v4tov4 listenaddress=192.168.3.87 listenport=8642
```

**原理**：手机（192.168.3.220）→ 请求 `192.168.3.87:8642` → Windows转发到 `localhost:8642` → WSL2自动接收（WSL2特殊处理localhost流量）

**需要转发的端口**：
- `8642` — Hermes Gateway（核心API，供APP调用）
- `8787` — WebUI（可选，调试用）

**Windows重启后规则会保留**（netsh规则是持久的），但如果WSL IP变了（罕见情况），需要重新配置。

**APP端配置**：局域网URL用 `http://<Windows主机IP>:8642`，不要用 `8787`（WebUI不代理API调用）。

### ⚠️ Windows 端口排除陷阱（portproxy 无声失效）

**症状**：`netsh interface portproxy add` 执行成功（无报错），`show all` 也能看到规则，但 `curl http://192.168.3.87:<PORT>/health` 超时。同时其他端口的 portproxy 正常工作。

**根因**：Windows Hyper-V / WSL2 安装后会在内核级别保留一段 TCP 端口范围供系统内部使用。端口在此范围内时，portproxy 规则虽然会出现在列表中，但 Windows 不会真正转发流量。

```powershell
# 查看被保留的端口范围
netsh interface ipv4 show excludedportrange protocol=tcp
```

输出示例：
```
开始端口    结束端口
----------    --------
      5357        5357
      8055        8154    ← 8089 在此范围内，portproxy 无声失效！
      8155        8254
      ...
```

**诊断步骤**（当 portproxy 不生效时）：

| 步骤 | 命令 | 预期 |
|------|------|------|
| ① 本地测试 | `curl --noproxy '*' http://127.0.0.1:<PORT>/health` | 200 ✅（服务本地可用） |
| ② 测试 WSL 内部 IP | `curl --noproxy '*' http://172.27.x.x:<PORT>/health` | 200 ✅（WSL内部可达） |
| ③ 测试 Windows IP | `curl --noproxy '*' http://192.168.3.87:<PORT>/health` | ❌ timeout（portproxy 不工作） |
| ④ 检查 portproxy | `netsh interface portproxy show v4tov4` | 规则存在但无效 |
| ⑤ 查排除范围 | `netsh interface ipv4 show excludedportrange protocol=tcp` | 端口在保留范围内！ |

**修复**：
1. 选一个**不在任何保留范围内**的端口（如 8866、8642 等已验证可用的端口）
2. 更新服务配置 + portproxy + 防火墙规则
3. 更新 APP 代码中的端口号

**已验证可用端口**：8642（Hermes Gateway）、8787（WebUI）、8866（xiaoq-api，v1.3.0 迁移后）
**已验证不可用端口**：8089（在 8055-8154 排除范围内）

注意：端口保留范围在每次 WSL 重启后可能变化。选端口时避开所有当前保留范围，并预留弹性。

## 版本号管理

### 方案：常量 + 同步app.json5 + 全局显示

在主页面（MainPage）定义版本常量，**不要放在功能页面**（如ChatPage、PredictionPage），否则切换页面后看不到：

```typescript
@Component
struct MainPage {
  private readonly APP_VERSION: string = '1.2.0';
  // ...
}
```

**显示位置**（选至少一个全局可见的位置）：
- 主页面标题栏右侧：`Text('v' + this.APP_VERSION).fontSize(11).fontColor('#555555')`
- 主页面底部：放在TabBar上方或设置页底部
- 欢迎消息（仅Chat类APP）

**不要**把版本号放在只有注册用户才能看到的子页面里——未注册用户完全看不到。

### 版本号递增规则

| 字段 | 位置 | 递增规则 |
|------|------|---------|
| `versionCode` | `AppScope/app.json5` | 整型，每次发版+10（如 1000100→1000110→1000120） |
| `versionName` | `AppScope/app.json5` | 语义版本（如 "1.0.0"→"1.1.0"→"1.2.0"） |
| `APP_VERSION` | MainPage.ets | 必须与 versionName 一致 |

### 后续升级三步走

1. 改 `AppScope/app.json5` 的 `versionCode`（+10）和 `versionName`
2. 改 `MainPage.ets` 的 `APP_VERSION` 常量（与 versionName 一致）
3. 重新编译推送

### 版本号更新清单（每次发版必须全做）

版本号藏在**两个地方**，漏改任何一个都会出问题：

| # | 文件 | 改什么 | 漏改后果 |
|---|------|--------|---------|
| 1 | `AppScope/app.json5` | `versionCode`(+10)、`versionName`(如`1.3.0`) | 新版无法覆盖安装旧版（versionCode不递增） |
| 2 | `MainPage.ets` | `APP_VERSION`常量（必须与versionName一致） | 用户在APP内看到的还是旧版本号 |

**搜索确认没有遗漏**：`grep -rn "v1\.\|1\.2\.0\|version" entry/src/ --include="*.ets"` 确保没有其他地方硬编码了旧版本号。

### 💡 已自动化：bug修复版本自动自增

`auto_bump_version.py` 已预装项目根目录，每次编译前自动跑：

```bash
# 检测到源码变更 → 自动 patch +1
python3 auto_bump_version.py && [编译命令]

# 无变更 → 静默跳过，不改版本号
```

逻辑：
* 检测 `entry/src/main/ets/`、`AppScope/app.json5`、`entry/src/main/resources/` 的 git 变更
* 有变更：`versionCode += 10`，`versionName` patch +1，同步改 `MainPage.ets`
* 无变更：跳过，不提交空版本号

**以后你说"修bug"，我改完代码编译时版本号自动就变了，你不用再管。**

### ⚠️ 常见坑

- **版本号不刷新**：改了 app.json5 但没改 MainPage 的常量，用户看到的还是旧版本号
- **版本号放在子页面**：用户在赛程页看不到版本号，只有进预测页才看到——应该放在全局位置
- **versionCode不递增**：每次发版必须递增 versionCode，否则手机上旧版本不会被覆盖安装
- **环境变量写长值用 Python 不用 echo**：`echo 'KEY=very_long_value...' >> .env` 可能截断。用 `python3 -c "open('file','a').write('KEY=value\\n')"` 或 `execute_code` 写。事后用 `cut -d= -f2 | wc -c` 验证长度
- **AGC 证书文件提交到 Git**：`agconnect-services.json` 和 `agc-apiclient-*.json` 含 API 密钥。如果不小心 git add 了，用 `git rm --cached` 移除再追加到 `.gitignore`
- **换签名证书后安装失败（error 9568332）**：设备上已有debug签名的app，换release签名后鸿蒙不允许覆盖安装。**必须先卸载**：`hdc shell bm uninstall -n <bundleName>`，再Run。注意：卸载会清空所有Preferences数据（用户账号、缓存等）
- **卸载重装后版本号消失**：卸载会清空Preferences，用户状态回退到"未注册"。如果版本号只在"已注册"分支显示，未注册用户看不到。**规则：版本号必须在所有UI分支（注册/未注册/错误/加载中）都显示**，放在条件分支之外或每个分支内都放一份
- **release签名不能直装（error 9568322）**：release签名的包通过hdc直装会报`signature verification failed due to not trusted app source`。release包**只能通过AppGallery分发**，不能hdc直装。开发调试永远用debug签名
- **compatibleSdkVersion API Level限制**：AGC上架自检要求`compatibleSdkVersion`的API Level ≤22。设成`6.1.1(24)`会被拒绝（"当前仅支持API Level≤22的手机"）。需要降级到`6.0.2(22)`并安装对应SDK版本

---

## AppGallery Connect 上架发布流程

### 前置条件

- 华为开发者账号（已实名认证，审核1-3天）
- 发布证书（`.cer`）和发布Profile（`.p7b`）已在AGC申请
- `compatibleSdkVersion` ≤ API 22（AGC自检限制）

### Step 1：申请发布证书和Profile

在 AppGallery Connect 操作（与调试证书流程类似，但类型选「发布」）：

1. AGC → HarmonyOS API → 证书 → 新增证书 → 类型选**发布证书** → 上传 `.csr` → 下载 `.cer`
2. AGC → HarmonyOS API → Profile → 新增Profile → 类型选**发布Profile** → 选择应用+发布证书（**不需要绑定设备**）→ 下载 `.p7b`

每个账号限：1个发布证书 + 2个调试证书。

### Step 2：配置 release 签名

在 `build-profile.json5` 中新增 `release` 签名配置（不要覆盖 debug 的）：

```json5
"signingConfigs": [
  {
    "name": "debug",
    "material": { /* debug签名 */ }
  },
  {
    "name": "release",
    "type": "HarmonyOS",
    "material": {
      "storeFile": "D:/path/to/xiaoq.p12",
      "storePassword": "",  // IDE自动加密填入
      "keyAlias": "xiaoq",
      "keyPassword": "",    // IDE自动加密填入
      "signAlg": "SHA256withECDSA",
      "profile": "D:/path/to/release.p7b",
      "certpath": "D:/path/to/xiaoq_release.cer"
    }
  }
]
```

⚠️ **不要用 write_file/patch 覆盖 build-profile.json5**（会丢失加密密码）。让用户在 DevEco Studio GUI 中操作：File → Project Structure → Signing Configs。

### Step 3：降级 compatibleSdkVersion

AGC 上架自检要求 API Level ≤ 22：

```
compatibleSdkVersion: "6.0.2(22)"  ← 必须≤22
targetSdkVersion: "6.1.1(24)"      ← 保持不变
```

**HarmonyOS SDK 版本映射**（截至2026年6月）：

| HarmonyOS 版本 | API Level | compatibleSdkVersion |
|---------------|-----------|---------------------|
| 5.0.0 | 12 | `"5.0.0(12)"` |
| 5.0.1 | 13 | `"5.0.1(13)"` |
| 5.0.2 | 14 | `"5.0.2(14)"` |
| 5.0.3 | 15 | `"5.0.3(15)"` |
| 5.0.4 | 16 | `"5.0.4(16)"` |
| 5.1.0 | 18 | `"5.1.0(18)"` |
| 6.0.0 | 20 | `"6.0.0(20)"` |
| **6.0.2** | **22** | **`"6.0.2(22)"`** ← AGC上限 |
| 6.1.0 | 23 | `"6.1.0(23)"` |
| 6.1.1 | 24 | `"6.1.1(24)"` ← 当前开发 |

如果只装了 API 24 SDK，需要在 DevEco Studio 中额外安装 6.0.2 SDK：File → Settings → SDK → 勾选 `HarmonyOS 6.0.2 (API 22)`。

### Step 4：Build APP（不是HAP）

- **Build → Build Hap(s)/APP(s) → Build APP(s)**
- 输出 `.app` 文件（上传到AGC用的格式，不是 `.hap`）
- 注意：DevEco Studio 顶部工具栏的「Build Mode」下拉框可能不显示，用菜单直接构建即可

### Step 5：上传到 AGC

1. AGC → 我的项目 → 应用 → 版本管理 → 新建版本
2. 上传 `.app` 文件
3. 填写应用信息（名称、分类、简介、截图、隐私政策URL等）
4. 提交审核（1-3个工作日）

### Step 6：上架自检常见报错

| 报错 | 原因 | 修复 |
|------|------|------|
| `API Level≤22` | compatibleSdkVersion设太高 | 降到 `6.0.2(22)` 或更低 |
| `signature verification failed` (9568322) | 试图hdc直装release包 | release包只能通过AGC分发 |
| `install sign info inconsistent` (9568332) | 签名不一致 | 先卸载旧包：`hdc shell bm uninstall -n <bundleName>` |

### privacy policy 快速方案

AGC要求提供隐私政策URL。最简方案：
- 用 GitHub Pages 托管一个简单的隐私政策页面
- 或在应用描述中附上隐私政策文本（部分分类允许）

## 设置持久化（AppStorage / PersistentStorage）

### 基础模式：AppStorage get/set

```typescript
// 存
AppStorage.set<string>('apiKey', this.apiKey);

// 取
const savedKey: string = AppStorage.get<string>('apiKey') as string;
if (savedKey !== undefined) {
  this.apiKey = savedKey;
}
```

适合保存：API密钥、用户偏好。不适合保存：大数据量、敏感信息。

### 通用方案：AppStorage + @StorageLink（运行时跨页面共享）

运行时跨页面同步用 `@StorageLink`：

```typescript
@Component
struct ChatPage {
  @StorageLink('apiKey') apiKey: string = '';   // 替代 @State
  
  private saveSettings(): void {
    this.apiKey = this.tempApiKey;   // ← 自动同步到所有页面
  }
}

@Component  
struct FilesPage {
  @StorageLink('apiKey') apiKey: string = '';   // 自动读到 ChatPage 存的值
  // 无需在 aboutToAppear 中 AppStorage.get
}
```

**坑**：不要手动 `AppStorage.get` + `AppStorage.set`。直接 `@StorageLink` 双向绑定，一改全改。

### 持久化方案：Preferences API（推荐，比 PersistentStorage 可靠）

`PersistentStorage.persistProp` 有时序问题——如果组件初始化先于持久化恢复完成，保存的值会丢失。**实测 PersistentStorage 在下一次 APP 启动后可能读不到上次保存的值**。

改用 `@kit.ArkData` 的 Preferences API 手动读写，数据写入磁盘文件，重启后在 `aboutToAppear` 中读取：

```typescript
// utils/PreferencesHelper.ets
import { preferences } from '@kit.ArkData';
import { common } from '@kit.AbilityKit';

const STORE_NAME: string = 'xiaoq_preferences';
const KEY_API_KEY: string = 'api_key';

let preferencesInstance: preferences.Preferences | null = null;

async function getPreferences(context: common.Context): Promise<preferences.Preferences> {
  if (preferencesInstance !== null) {
    return preferencesInstance;
  }
  preferencesInstance = await preferences.getPreferences(context, STORE_NAME);
  return preferencesInstance;
}

export async function saveApiKey(context: common.Context, apiKey: string): Promise<void> {
  const pref = await getPreferences(context);
  await pref.put(KEY_API_KEY, apiKey);
  await pref.flush();
}

export async function loadApiKey(context: common.Context): Promise<string> {
  const pref = await getPreferences(context);
  const valueObj: preferences.ValueType = await pref.get(KEY_API_KEY, '');
  const value: string = valueObj as string;
  return value;
}
```

**注意**：`pref.get()` 返回 `ValueType`（string|number|boolean），需要显式 cast 为 `string` 才能通过 ArkTS 严格模式。

**使用方式**（ChatPage.ets）：

```typescript
import { common } from '@kit.AbilityKit';
import { saveApiKey, loadApiKey } from '../utils/PreferencesHelper.ets';

// 启动时加载
aboutToAppear(): void {
  const ctx: common.Context = this.getUIContext().getHostContext() as common.Context;
  loadApiKey(ctx).then((savedKey: string) => {
    if (savedKey !== '') {
      this.apiKey = savedKey;
    }
  });
}

// 保存时写入
private async saveSettings(): Promise<void> {
  this.apiKey = this.tempApiKey;
  const ctx: common.Context = this.getUIContext().getHostContext() as common.Context;
  await saveApiKey(ctx, this.apiKey);
  // ...
}
```

**与 @StorageLink 配合**：`@StorageLink` 负责运行时跨页面同步，Preferences 负责持久化到磁盘。两者互补：Preferences 在 `aboutToAppear` 加载 → 赋值给 `@StorageLink apiKey` → 所有页面自动更新。

## AppGallery上架发布流程

### 前置条件

- 华为开发者账号（已实名认证，审核1-3天）
- 发布证书（`.cer`）+ 发布Profile（`.p7b`）— 在AGC上申请，类型选「发布」
- 发布Profile**不需要绑定设备**（调试Profile才需要）

### HarmonyOS SDK版本与API Level对应关系

| HarmonyOS版本 | API Level | compatibleSdkVersion |
|---------------|-----------|---------------------|
| 5.0.0 | 12 | `"5.0.0(12)"` |
| 5.0.1 | 13 | `"5.0.1(13)"` |
| 5.0.2 | 14 | `"5.0.2(14)"` |
| 6.0.0 | 20 | `"6.0.0(20)"` |
| 6.0.2 | 22 | `"6.0.2(22)"` |
| 6.1.0 | 23 | `"6.1.0(23)"` |
| **6.1.1** | **24** | **`"6.1.1(24)"`** |

⚠️ **AGC自检限制**：提交上架时AGC会校验 `compatibleSdkVersion`，目前手机应用要求 **API Level ≤ 22**。如果设成 `6.1.1(24)` 会报错：`上架自检启动失败，当前仅支持API Level≤22的手机`。需要在DevEco Studio安装对应低版本SDK，然后将 `compatibleSdkVersion` 改为 `"6.0.2(22)"` 或更低。`targetSdkVersion` 可以保持高版本不变。

### 上架步骤

1. **申请发布证书**：AGC → HarmonyOS API → 证书 → 新增 → 类型选「发布」→ 上传.csr → 下载.cer
2. **创建发布Profile**：AGC → HarmonyOS API → Profile → 新增 → 类型选「发布」→ 绑定应用+证书（不需要设备）→ 下载.p7b
3. **配置release签名**：DevEco Studio → File → Project Structure → Signing Configs → 新增release方案 → 填入.p12 + .cer + .p7b + 密码
4. **Build APP(s)**：Build → Build Hap(s)/APP(s) → Build APP(s)（输出.app文件）
5. **上传到AGC**：版本管理 → 新建版本 → 上传.app文件
6. **填写应用信息**：名称、分类、简介、截图（至少4张）、隐私政策URL
7. **提交审核**：1-3个工作日

### ⚠️ Release签名不能直装

release签名的包（.app）**不能通过hdc直装到手机**，报错 `9568322: signature verification failed due to not trusted app source`。release包只能通过应用市场分发。开发调试始终用debug签名。

### ⚠️ 签名不一致覆盖安装失败（9568332）

从debug换release签名后，设备上已有的debug签名app会导致 `install sign info inconsistent`。**必须先卸载**：`hdc shell bm uninstall -n <bundleName>`，再安装。卸载会清空Preferences数据。

## 参考资料\n\n- `references/arkts-state-mutation-patterns.md` — @State数组对象属性修改后必须重新赋值数组引用\n- `references/api-fallback-pattern.md` — API离线兜底模式（硬编码48支队伍数据、已结束比赛比分、北美时区换算）
- `references/arkts-object-literal-workarounds.md` — 对象字面量类型转换失败的解决方案、null vs undefined 规则、非标准JSON解析
- `references/harmonyos-signing-checklist.md` — 签名证书申请的完整核对清单
- `references/appgallery-publishing-checklist.md` — AppGallery上架发布流程、API版本映射、AGC自检错误码、应用信息填写核对
- `references/appgallery-publishing-guide.md` — AppGallery发布上架全流程：release签名申请、Build APP(s)、上传AGC、审核、debug vs release对比
- `references/harmonyos-rcp-api-usage.md` — rcp网络请求API用法与踩坑记录
- `references/hvigor-cli-commands.md` — hvigor CLI编译命令、环境变量、失败清单（基于DevEco Studio 6.1.1实战验证）
- `references/worldcup2026-project-patterns.md` — 多页面APP架构、TabBar导航、弹窗模式、@Builder图片组件、Preferences存储、积分榜计算
- `references/non-blocking-about-to-appear.md` — 非阻塞aboutToAppear模式：多步异步（Preferences→API→重计算）不阻塞UI
- `references/worldcup2026-api-patterns.md` — 体育API端点、ID体系、local_date时区陷阱、进球者Unicode引号解析、hardcoded+API merge-by-ID模式
- `references/instant-load-async-refresh-pattern.md` — 秒加载模式：同步骨架+异步刷新，首屏性能优化（含缓存层）
- `references/auto-refresh-with-request-guard.md` — 自动刷新模式：setInterval + isRefreshing锁，防API请求堆积
- `references/arkui-refresh-component-pattern.md` — 下拉刷新模式：Refresh组件 + $$双向绑定，积分榜/排行榜按需刷新
- `references/http-datatype-object-type-conversion-trap.md` — http.HttpDataType.OBJECT自动类型转换陷阱（boolean/number/string混淆）
- `references/local-data-cache-pattern.md` — 本地数据缓存模式：preferences缓存API结果，两层缓存架构（内存+持久化），减少API调用
- `references/websocket-push-pattern.md` — WebSocket 长连接推送模式（心跳保活、断线重连、消息类型）
- `references/chat-message-persistence.md` — Chat消息持久化（Preferences + 防抖保存，恢复历史记录）
- `references/approval-card-pattern.md` — 交互式授权卡片（4选项：仅本次/会话/始终/拒绝 + Hermes API）
- `references/background-lifecycle-ws-reconnect.md` — 前台/后台生命周期 & WebSocket自动重连
- `references/wsl-powershell-cli-build.md` — WSL → PowerShell CLI 编译部署全流程
- `references/build-deploy-pattern.md` — 一键编译部署脚本 (build-deploy.ps1) 使用说明与踩坑
- `references/post-deploy-test-checklist.md` — 部署后自测清单（版本号、推送、授权卡等）

### 26. 从比赛数据实时提取射手榜（通用：从子记录聚合统计）

**场景**：射手榜不是独立API端点，需要从每场比赛的进球者字段中提取并统计。

**模式**：遍历finished比赛 → 提取进球者名字 → 按名字计数 → 排序 → 返回TopScorer[]

```ets
interface ScorerStat { name: string; goals: number; teamName: string; teamFlag: string; }
// ⚠️ interface必须在模块级，不能在函数体内（见§11d）

// 中英文名映射（硬编码已知射手）
private static readonly SCORER_CN: Record<string, string> = {
  'F. Balogun': '巴洛贡', 'V. Júnior': '维尼修斯', 'Breel Embolo': '恩博洛'
} as Record<string, string>;

static extractTopScorers(matches: MatchInfo[]): TopScorer[] {
  const scorerMap: Record<string, ScorerStat> = {} as Record<string, ScorerStat>;
  for (let i = 0; i < matches.length; i++) {
    const m: MatchInfo = matches[i];
    if (m.status !== 'finished') continue;
    // 处理主队+客队射手
    for (let j = 0; j < m.homeScorers.length; j++) {
      const name: string = m.homeScorers[j].replace(/\s*\d+.*/,'').replace(/\(.*\)/,'').trim();
      if (name.length === 0) continue;
      if (scorerMap[name] !== undefined) {
        scorerMap[name].goals++;
      } else {
        scorerMap[name] = { name: name, goals: 1, teamName: m.homeTeam.nameCn, teamFlag: m.homeTeam.flag } as ScorerStat;
      }
    }
    // 同理处理 awayScorers...
  }
  // 转数组 → 按进球数降序排 → 生成rank
  const arr: ScorerStat[] = [];
  const keys: string[] = Object.keys(scorerMap);
  for (let i = 0; i < keys.length; i++) { arr.push(scorerMap[keys[i]]); }
  arr.sort((a: ScorerStat, b: ScorerStat) => b.goals - a.goals);
  // 生成TopScorer[]，rank处理并列...
}
```

**进球者名字清洗**：API返回 `"F. Balogun 45'+5'"` 或 `"D. Bobadilla 7'(OG)"`，需去掉分钟数和标记：
```ets
const name: string = raw.replace(/\s*\d+.*/,'').replace(/\(.*\)/,'').trim();
```

**关键**：射手榜是**动态数据**，每次`refreshScores()`后需重新调用`extractTopScorers(matches)`更新。

### 27. execute_code中read_file/write_file的安全流程

**核心陷阱**：`execute_code`中的`read_file()`返回内容带行号前缀（`1|import...`），直接用`write_file()`写回会污染文件。

**安全流程**：
```python
from hermes_tools import read_file, write_file, terminal

# 读取
result = read_file("path/to/file.ets")
content = result['content']

# ⚠️ 必须strip行号前缀
import re
lines = content.split('\n')
cleaned = []
for line in lines:
    m = re.match(r'^\d+\|', line)
    if m:
        cleaned.append(line[m.end():])
    else:
        cleaned.append(line)
content = '\n'.join(cleaned)

# 修改
content = content.replace('old', 'new')

# 写回
write_file("path/to/file.ets", content)

# 验证大括号平衡
opens = content.count('{')
closes = content.count('}')
assert opens == closes, f"IMBALANCED: {{ {opens} }} {closes}"
```

**替代方案（更安全）**：不用`read_file`/`write_file`组合，直接用`terminal`+`python3`操作文件：
```python
terminal("python3 -c \"import re; ...\"")
```

**批量修改多个文件时**：逐个文件操作+验证，不要一次性处理所有文件。如果一个文件出错，不影响其他文件。

### 28. 不支持 `!`（非空断言）操作符

**ArkTS严格模式不支持 TypeScript 的 `!` 非空断言操作符**。所有可能为 `null` / `undefined` 的值必须通过 `if (x !== null)` / `if (x !== undefined)` 显式检查，不能使用 `x!`。

```ets
// ❌ 编译错误 — ArkTS不支持非空断言
if (this.isPredicted(m.id) && this.getPredForMatch(m.id) !== null) {
  Text(this.getPredForMatch(m.id)!.homePred + ' : ' + this.getPredForMatch(m.id)!.awayPred)
}

// ✅ 正确 — 用局部变量承接，显式null检查
const predRec: PredictionRecord | null = this.getPredForMatch(m.id);
if (this.isPredicted(m.id) && predRec !== null) {
  Text(predRec.homePred + ' : ' + predRec.awayPred)
}
```

**进阶**：局部变量声明不能在UI组件树内部（见§29），所以进一步抽成方法：

```ets
// ✅ 最终方案 — 抽成方法，从UI中调用
getPredictionText(matchId: string): string {
  const pred: PredictionRecord | null = this.getPredForMatch(matchId);
  if (pred !== null) {
    return pred.homePred + ' : ' + pred.awayPred;
  }
  return '0:0';
}

// UI中调用
Text(this.getPredictionText(m.id))
```

**排查方法**：全项目搜索 `.getXxx(m.id)!` 或 `something!` 模式，替换为方法调用 + 内部null检查。

### 29. UI组件树（build() / @Builder）内不能声明变量

**症状**：编译报 `10905209 ArkTS Compiler Error`，错误信息 `"Only UI component syntax can be written here"`。

**根因**：在 `build()` 或 `@Builder` 方法的UI组件树内（`Column{}` / `Row{}` / `Stack{}` / `if`条件渲染块内部等），不能使用 `const` / `let` 声明变量。ArkTS的UI DSL只允许组件调用和链式方法调用。

```ets
// ❌ 错误 — 变量声明在UI组件树内
build() {
  Column() {
    const pred: PredictionRecord | null = this.getPredForMatch(m.id);  // 报错！
    if (pred !== null) {
      Text(pred.homePred.toString())
    }
  }
}
```

**修复方案**：

**方案A（推荐）**：将逻辑抽成方法，在方法体内声明变量，UI中只调用方法：
```ets
getPredictionText(matchId: string): string {
  const pred = this.getPredForMatch(matchId);
  if (pred !== null) return pred.homePred + ':' + pred.awayPred;
  return '0:0';
}
// UI中只调用方法，不声明变量
build() { Column() { Text(this.getPredictionText(m.id)) } }
```

**方案B**：在条件中直接使用方法调用（适合简单场景）：
```ets
if (this.isPredicted(m.id)) {
  Text(this.getPredictionText(m.id))
}
```

**规则**：UI组件树中出现的所有 `const` / `let` 都应移到外部方法中。检查标准：`build()` 和 `@Builder` 方法内部的 `{}` 代码块中不应有变量声明语句。

### 开源发布

**ArkTS严格模式不支持 TypeScript 的 `!` 非空断言操作符**。所有可能为 `null` / `undefined` 的值必须通过 `if (x !== null)` / `if (x !== undefined)` 显式检查，不能使用 `x!`。

```ets
// ❌ 编译错误 — ArkTS不支持非空断言
if (this.isPredicted(m.id) && this.getPredForMatch(m.id) !== null) {
  Text(this.getPredForMatch(m.id)!.homePred + ' : ' + this.getPredForMatch(m.id)!.awayPred)
}

// ✅ 正确 — 用局部变量承接，显式null检查
const predRec: PredictionRecord | null = this.getPredForMatch(m.id);
if (this.isPredicted(m.id) && predRec !== null) {
  Text(predRec.homePred + ' : ' + predRec.awayPred)
}
```

**但是**：变量声明不能在UI组件树中（见§29）。所以进一步抽成方法：

```ets
// ✅ 最终方案 — 抽成方法，从UI中调用
getPredictionText(matchId: string): string {
  const pred: PredictionRecord | null = this.getPredForMatch(matchId);
  if (pred !== null) {
    return pred.homePred + ' : ' + pred.awayPred;
  }
  return '0:0';
}

// UI中调用
Text(this.getPredictionText(m.id))
```

**规则**：全项目搜索 `.getXxx(m.id)!` 或 `something!` 模式，全部替换为方法调用 + 内部null检查。

### 29. UI组件树（build/@Builder）内不能声明变量

**症状**：编译报 `10905209 ArkTS Compiler Error`，错误信息 `"Only UI component syntax can be written here"`。

**根因**：在 `build()` 或 `@Builder` 方法的UI组件树内（`Column{}`、`Row{}`、`Stack{}`、`if`条件渲染块内部等），不能使用 `const` / `let` 声明变量。ArkTS的UI DSL只允许组件调用和链式方法。

```ets
// ❌ 错误 — 变量声明在UI组件树内
build() {
  Column() {
    const pred: PredictionRecord | null = this.getPredForMatch(m.id);  // 编译报错！
    if (pred !== null) {
      Text(pred.homePred.toString())
    }
  }
}
```

**修复方案**：

**方案A（推荐）**：将逻辑抽成方法，在方法体内声明变量，UI中只调用方法返回结果：

```ets
// 方法体内可以声明变量
getPredictionText(matchId: string): string {
  const pred = this.getPredForMatch(matchId);
  if (pred !== null) return pred.homePred + ':' + pred.awayPred;
  return '0:0';
}

// UI中只调用方法
build() {
  Column() {
    Text(this.getPredictionText(m.id))
  }
}
```

**方案B**：在 `if` 条件中直接计算表达式（适合简单场景）：

```ets
// 可以这样 — 直接在条件中用方法
if (this.isPredicted(m.id)) {
  Text(this.getPredictionText(m.id))
}
```

**批量修改多个文件时**：逐个文件操作+验证，不要一次性处理所有文件。如果一个文件出错，不影响其他文件。

### 28. 不支持 `!`（非空断言）操作符

**ArkTS严格模式不支持 TypeScript 的 `!` 非空断言操作符**。所有可能为 `null` / `undefined` 的值必须通过 `if (x !== null)` / `if (x !== undefined)` 显式检查，不能使用 `x!`。

```ets
// ❌ 编译错误 — ArkTS不支持非空断言
if (this.isPredicted(m.id) && this.getPredForMatch(m.id) !== null) {
  Text(this.getPredForMatch(m.id)!.homePred + ' : ' + this.getPredForMatch(m.id)!.awayPred)
}

// ✅ 正确 — 用局部变量承接，显式null检查
const predRec: PredictionRecord | null = this.getPredForMatch(m.id);
if (this.isPredicted(m.id) && predRec !== null) {
  Text(predRec.homePred + ' : ' + predRec.awayPred)
}
```

**进阶**：局部变量声明不能在UI组件树内部（见§29），所以进一步抽成方法：

```ets
// ✅ 最终方案 — 抽成方法，从UI中调用
getPredictionText(matchId: string): string {
  const pred: PredictionRecord | null = this.getPredForMatch(matchId);
  if (pred !== null) {
    return pred.homePred + ' : ' + pred.awayPred;
  }
  return '0:0';
}

// UI中调用
Text(this.getPredictionText(m.id))
```

**排查方法**：全项目搜索 `.getXxx(m.id)!` 或 `something!` 模式，替换为方法调用 + 内部null检查。

### 29. UI组件树（build() / @Builder）内不能声明变量

**症状**：编译报 `10905209 ArkTS Compiler Error`，错误信息 `"Only UI component syntax can be written here"`。

**根因**：在 `build()` 或 `@Builder` 方法的UI组件树内（`Column{}` / `Row{}` / `Stack{}` / `if`条件渲染块内部等），不能使用 `const` / `let` 声明变量。ArkTS的UI DSL只允许组件调用和链式方法调用。

```ets
// ❌ 错误 — 变量声明在UI组件树内
build() {
  Column() {
    const pred: PredictionRecord | null = this.getPredForMatch(m.id);  // 报错！
    if (pred !== null) {
      Text(pred.homePred.toString())
    }
  }
}
```

**修复方案**：

**方案A（推荐）**：将逻辑抽成方法，在方法体内声明变量，UI中只调用方法：
```ets
getPredictionText(matchId: string): string {
  const pred = this.getPredForMatch(matchId);
  if (pred !== null) return pred.homePred + ':' + pred.awayPred;
  return '0:0';
}
// UI中只调用方法，不声明变量
build() { Column() { Text(this.getPredictionText(m.id)) } }
```

**方案B**：在条件中直接使用方法调用（适合简单场景）：
```ets
if (this.isPredicted(m.id)) {
  Text(this.getPredictionText(m.id))
}
```

**规则**：UI组件树中出现的所有 `const` / `let` 都应移到外部方法中。检查标准：`build()` 和 `@Builder` 方法内部的 `{}` 代码块中不应有变量声明语句。

### 开源发布

本技能已发布为独立开源项目：**[github.com/xiaoqagent/harmonyos-app-development-skill](https://github.com/xiaoqagent/harmonyos-app-development-skill)**

包含 SKILL.md + 17 篇 references + templates/ + README + MIT License。可自由 clone 使用。

## 关联技能

- `android-app-development` — Android版的小Q APP，架构设计可参考
- `task-delivery-workflow` — 包含"先验证工具链再写业务代码"的原则
- `xiaoq-build-deploy` — 一键编译部署脚本（编译→部署→验证，单次 PowerShell 调用完成）

## ⚠️ 常见 ArkTS 运行时陷阱（非编译错误）

### bindContentCover 不接受 `||` 表达式

**症状**：用 `bindContentCover` 控制弹窗显示，`@State` 变量变成 `true` 后弹窗不显示。

**根因**：`bindContentCover(this.showA || this.showB, builder)` 中的 `||` 表达式不会响应 `@State` 变更。虽然 `showA` 和 `showB` 各自是 `@State` 变量，但 `||` 组合后的表达式在 `bindContentCover` 中只被读取一次初始值，后续变化不触发 UI 更新。

```ets
// ❌ 错误 — || 表达式不会被追踪
@State showSettings: boolean = false;
@State showApprove: boolean = false;
Column() { ... }
.bindContentCover(this.showSettings || this.showApprove, this.DialogLayer())
// showApprove = true 后弹窗不显示！
```

**修复**：用单一 `@State` 布尔变量 + `@State` 字符串模式：

```ets
@State showDialog: boolean = false;
@State dialogMode: string = '';  // '' | 'settings' | 'approve'

Column() { ... }
.bindContentCover(this.showDialog, this.DialogLayer())

@Builder
DialogLayer() {
  if (this.dialogMode === 'settings') { this.SettingsPanel() }
  else if (this.dialogMode === 'approve') { this.ApproveDialog() }
}
```

**规则**：传递 `bindContentCover` 的第一个参数必须是直接的 `@State` 布尔变量，不能是表达式。
