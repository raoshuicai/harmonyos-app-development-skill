# KanKan Hybrid APP 开发踩坑全记录

> 2026-07-19 · 基于 KanKan V0.1.8 开发实战总结
> Hybrid 架构：ArkWeb(WebView) + JSBridge + 原生播放器

---

## 一、ArkTS 编译错误

### 1.1 `onAreaChange` 的 width 类型

**错误**：`Type 'Length' is not assignable to type 'number'.`

**根因**：`onAreaChange` 回调的 `newVal.width` 类型为 `Length`（可含单位如 `100%`），不能直接赋值给 `number`。

**修复**：
```typescript
this.trackWidth = newVal.width as number;
```

### 1.2 Preferences 没有 `getAllKeys`

**错误**：`Property 'getAllKeys' does not exist on type 'Preferences'.`

**根因**：API 24 的 `@ohos.data.preferences` 不提供 `getAllKeys()`。

**修复**：自行维护一个 URL 键列表。
```typescript
private async addUrlKey(url: string): Promise<void> {
  const p = await this.getPrefs();
  const raw = await p.get('KanKan.url_keys', '[]') as string;
  let keys: string[] = raw ? JSON.parse(raw) as string[] : [];
  if (keys.indexOf(url) < 0) {
    keys.push(url);
    await p.put('KanKan.url_keys', JSON.stringify(keys));
    await p.flush();
  }
}
```

### 1.3 `PlaybackInfo` 类型无法直接访问属性

**错误**：`Property 'currentTime' does not exist on type 'PlaybackInfo'`

**修复**：JSON 序列化桥接。
```typescript
.onUpdate((event) => {
  const info = JSON.parse(JSON.stringify(event));
  if (info) this.currentTime = info['currentTime'] || info['time'] || 0;
})
```

### 1.4 `VideoController.play()` 不存在

**错误**：`Property 'play' does not exist on type 'VideoController'.`

**修复**：API 24 中用 `start()` 替代。
```typescript
this.videoController.start();  // 不是 play()
```

### 1.5 `Video.speed()` 修饰符不存在

**错误**：`Property 'speed' does not exist on type 'VideoAttribute'.`

**修复**：通过构造函数参数传入。
```typescript
Video({ src: url, controller: ctrl, currentProgressRate: 1.5 })
```

### 1.6 `onStateChange` 不存在

**修复**：用 `onStart` / `onPause` 替代。
```typescript
Video({...})
  .onStart(() => { this.isPlaying = true; })
  .onPause(() => { this.isPlaying = false; })
```

---

## 二、ArkUI 布局与交互

### 2.1 Stack.onClick 吞掉子组件点击

**现象**：点击进度条/按钮无响应。

**修复**：
1. 去掉全屏遮罩 `Rect().width('100%').height('100%')`
2. 用 `layoutWeight(1)` 弹性撑开替代 `height('100%')`
3. 子组件直接设 onClick

### 2.2 `requestFullscreen()` 隐藏自定义覆盖层

**修复**：不要调用 `requestFullscreen()`，让 Video 自然填满 Stack + `AUTO_ROTATION`。

### 2.3 沉浸式布局白条

**修复**：EntryAbility 中设置系统栏颜色。
```typescript
win.setWindowLayoutFullScreen(true);
win.setSystemBarProperties({
  statusBarColor: '#0f0f0f',
  navigationBarColor: '#0f0f0f',
});
```

H5 侧 CSS 加 `--safe-top: 48px` 给顶部内容留间距。

### 2.4 HitTestMode.None 使按钮不可点

`HitTestMode.None` = 组件自身也不可点击。不需要显式设置。

---

## 三、ArkWeb / JSBridge

### 3.1 `alert()` / `confirm()` 被静默拦截

**修复**：用自定义 Toast + 确认弹窗替代。

### 3.2 JSBridge methodList 必须完整

新增方法时必须同步更新 `javaScriptProxy` 的 `methodList`。

---

## 四、数据持久化

### 4.1 Preferences 异步方法在非 async 生命周期

`aboutToDisappear()` 默认为 `void`。需要 await 则声明为 `async aboutToDisappear(): Promise<void>`。

---

## 五、编译部署

### 5.1 版本号升级清缓存

```powershell
Remove-Item ".hvigor","entry\build" -Recurse -Force
```

### 5.2 WSL 调 PowerShell

```bash
/mnt/c/Windows/System32/WindowsPowerShell/v1.0/powershell.exe \
  -ExecutionPolicy Bypass -File build-deploy.ps1
```

---

## 六、网络与 API

### 6.1 AList API 方法差异

Admin 设置 API 用 **GET** 而非 POST：
```bash
curl "http://host:5244/api/admin/setting/list?group=5"
```

### 6.2 AList 离线下载模块可能未编译

直接通过 Aria2 JSON-RPC 提交磁力：
```javascript
await aria2RPC('aria2.addUri', ['token:secret', ['magnet:?xt=...']]);
await aria2RPC('aria2.tellStatus', ['token:secret', 'gid']);
```

### 6.3 HTTP POST body

`@ohos.net.http` 的 `extraData` 用于 POST body，必须是字符串。

---

## 七、CMS 采集站

### 7.1 CORS 跨域

H5 fetch 请求 CMS API 会被 CORS 拦截，全部走 ArkTS 原生 http 代理。

### 7.2 源失效策略

1. 设置页可配 CMS 地址（Preferences 持久化）
2. APP 启动时从 Preferences 加载
3. 提供「从网络更新」从 GitHub 拉取
4. 搜索时记录健康状态（连续失败 3 次标记 💀）

---

## 八、其他

### 8.1 AVPlayer 状态机
```
create → setUrl → [initialized] → setSurface → prepare → [prepared] → play
```

### 8.2 javaScriptProxy controller 必须复用
```typescript
private controller = new webview.WebviewController();
Web({..., controller: this.controller})
  .javaScriptProxy({..., controller: this.controller})
```
