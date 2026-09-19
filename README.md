# HarmonyOS NEXT APP Development Skill

> 鸿蒙单框架（HarmonyOS NEXT，无AOSP兼容层）APP开发全流程实战指南

基于真实项目（WorldCup2026 世界杯APP）从零到上架的全过程踩坑沉淀，覆盖**环境搭建、ArkTS严格模式编码、签名证书管理、网络请求、性能优化、上架发布**全链路。

## 内容结构

```
harmonyos-app-development-skill/
├── SKILL.md                              ← 主文档：完整开发指南
├── references/                           ← 专题参考文档（17篇）
│   ├── harmonyos-arkts-strict-mode-violations.md  ← ArkTS严格模式错误全集
│   ├── arkts-object-literal-workarounds.md        ← 对象字面量/类型转换
│   ├── http-datatype-object-type-conversion-trap.md  ← API JSON类型陷阱
│   ├── instant-load-async-refresh-pattern.md      ← 秒加载+异步刷新
│   ├── auto-refresh-with-request-guard.md         ← setInterval防堆积
│   ├── local-data-cache-pattern.md                ← 本地缓存架构
│   ├── arkui-refresh-component-pattern.md         ← 下拉刷新
│   ├── worldcup2026-api-patterns.md               ← 体育API对接模式
│   ├── worldcup2026-project-patterns.md           ← 多页面APP架构模式
│   ├── appgallery-publishing-checklist.md         ← 上架发布核对清单
│   ├── appgallery-publishing-guide.md             ← 上架全流程
│   ├── harmonyos-signing-checklist.md             ← 签名证书清单
│   ├── hvigor-cli-commands.md                     ← 命令行编译
│   ├── harmonyos-rcp-api-usage.md                 ← rcp网络库用法
│   ├── wifi-file-open-debug-chain.md              ← WiFi调试
│   ├── preferences-helper.md                      ← 持久化方案
│   └── xiaoq-api-file-endpoints.md                ← 文件浏览API
└── templates/                            ← 可复用模板
    └── harmonyos-next-file-browser/       ← 文件浏览器模板
        ├── FilesPage.ets
        ├── FileViewPage.ets
        └── FileApi.ets
```

## 核心亮点

### ⚡ 实战验证，不是理论

所有内容来自 **WorldCup2026 APP** 真实开发过程——5个模块（赛程、积分榜、射手榜、晋级图、进球预测），72场小组赛硬编码，API实时比分合并，Preferences缓存，排行榜系统，完整上架流程。

### 覆盖的坑点（已修复，附根因）

| 问题 | 根因 | 章节 |
|------|------|------|
| 2948个编译错误 | 大文件patch截断→大括号不平衡 | §11b |
| 1853个编译错误 | read_file行号前缀写入文件 | §11b |
| 预测按钮没反应 | Dialog嵌在Column内→改Stack根节点 | §9b |
| 赛程时间全错 | API local_date时区不一致 | §18 |
| 比分全串了 | hardcoded ID顺序≠API ID顺序 | §17 |
| 比分显示0:0 | HttpDataType.OBJECT自动转类型 | §12 |
| 加载慢3-4秒 | 拆成同步骨架+异步刷新 | §23 |
| API请求堆积 | 加isRefreshing锁+3秒间隔 | §24 |

### 技术栈

- **HarmonyOS NEXT**（单框架，无AOSP）
- **ArkTS** + **ArkUI**（声明式UI框架）
- **DevEco Studio 6.1.1**
- **hvigor** 构建系统
- **rcp** / **http** 网络请求
- **Preferences** 数据持久化
- **hdc** 设备连接与调试
- **AppGallery Connect** 上架分发

## 快速开始

```bash
# 克隆本仓库
git clone https://github.com/raoshuicai/harmonyos-app-development-skill.git

# 打开 SKILL.md 开始阅读
# 或导入到 Hermes Agent 作为技能使用
```

## 前提条件

- Windows 11（DevEco Studio仅支持Windows/Mac）
- DevEco Studio 6.1.1+
- 华为开发者账号（已实名认证）
- 一台HarmonyOS NEXT真机或模拟器

## 适用人群

- 正在从 Android 迁移到 HarmonyOS NEXT 的开发者
- 鸿蒙 APP 新手，想避开常见陷阱
- Hermes Agent 用户，想开发鸿蒙客户端

## 协议

MIT License
