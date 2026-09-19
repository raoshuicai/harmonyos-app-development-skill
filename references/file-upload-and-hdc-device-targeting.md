# 文件上传坑 + hdc 多设备抓日志

2026-09-19 小Q APP「上传图片一直失败」排查沉淀。两个独立的坑，都会浪费时间。

---

## 坑一：`request.uploadFile` 只吃应用沙箱的文件

### 症状

选了相册里的图，APP 提示「上传失败」，但服务端日志里**一条 `POST /api/upload` 都没有**。

### 真机日志（唯一能定位的地方）

```
MediaPicker: picked image uri=file://media/Photo/13058/IMG_1789787436_22980/screenshot_xxx.jpg
MediaPicker: uploading screenshot_xxx.jpg to http://192.168.3.87:8866/api/upload
MediaPicker: upload threw: The parameters check fails
             Parameter verification failed, user file can only for request.agent.
```

注意是 `upload threw` —— 抛异常，不是 HTTP 返回码不对。请求**根本没发出去**，
在 `request.uploadFile` 的参数校验阶段就被拒了。

### 根因

`request.uploadFile` 跑在系统的下载/上传服务进程里，**只接受应用自己拥有的文件**
（沙箱内路径）。相册选择器给的 `file://media/Photo/...`、文档选择器给的
document 域 URI 都归系统图库/文件管理，不是应用的 —— 直接被参数校验挡掉。

报错原文里那句 `user file can only for request.agent` 说的就是这个：
"user file"（应用自己的文件）只能给 request.agent 用。

### 修法（APP 端，服务端无解）

图还在手机里，服务端做不了任何事，**必须在客户端先落沙箱再传**：

```ts
import { fileIo as fs } from '@kit.CoreFileKit';

/** 把选择器给的 URI 拷进应用沙箱，返回沙箱 URI（request.uploadFile 只认这个） */
private copyToSandbox(ctx: common.Context, item: PickedItem): string {
  const dst: string = ctx.cacheDir + '/' + item.name;
  const src = fs.openSync(item.uri, fs.OpenMode.READ_ONLY);
  try {
    fs.copyFileSync(src.fd, dst);
  } finally {
    fs.closeSync(src);
  }
  return 'file://' + dst;
}
```

`upload()` 里把 `uri: item.uri` 换成 `uri: this.copyToSandbox(ctx, item)`，
传完之后 `fs.unlink(dst)` 清掉缓存副本（别用 filesDir，也别不清，会越攒越多）。

其余逻辑**不要动** —— 系统上传能力自带进度与重试，比手拼 multipart 稳。

### 排查时走过的弯路

一开始怀疑服务端 → 网络 → 公网入口，全都排除了：

- 服务端日志：今天唯一的 `/api/upload` 记录是本地测试，手机的一个都没到
- `cloudflared config.yml`：`xiaoq.xiao-q.com` 通配到 `localhost:8866`，路径没限制
- 手动 POST `https://xiaoq.xiao-q.com/api/upload` → HTTP 200，公网这条路是好的

**结论：服务端 + 网络全干净时，去看客户端日志，别继续在服务端绕。**

---

## 坑二：hdc 多设备下抓错设备的日志

### 症状

`hdc hilog` 抓了半天，一条 APP 日志都没有，让人以为「APP 不打日志」。

### 真因

`hdc list targets` 同时列出两台：

```
127.0.0.1:5555       TCP   Connected    ← 模拟器（param model = "emulator"）
5JV0225B15002640     USB   Connected    ← 真机（用户实际测试的是这台）
```

不带 `-t` 时 hdc 不会自动选，`shell` 类命令直接报
`[Fail]ExecuteCommand need connect-key? please confirm a device by help info`。
而我当时**误以为 127.0.0.1:5555 就是手机**，抓的是模拟器的日志 —— 白费一轮。

### 正确姿势

```bash
HDC="/mnt/d/Program Files/Huawei/DevEco Studio/sdk/default/openharmony/toolchains/hdc.exe"

# 先看清有哪几台、各是什么
"$HDC" list targets -v

# 认设备：模拟器会回 "emulator"，真机回机型号
"$HDC" -t 127.0.0.1:5555     shell param get const.product.model
"$HDC" -t 5JV0225B15002640   shell param get const.product.model

# 装了什么版本
"$HDC" -t 5JV0225B15002640 shell "bm dump -n <bundleName> | grep -E 'versionName|versionCode'"

# 抓日志 —— 必须带 -t
"$HDC" -t 5JV0225B15002640 hilog -x        # dump 现有缓冲区，抓完即退
"$HDC" -t 5JV0225B15002640 hilog           # 持续跟随，抓实时事件
```

从 WSL **可以直接调 hdc.exe**（不必走 `cmd.exe /c`；只有 .bat 才必须走）。
路径带空格，整体加引号即可。

### 缓冲区会滚

`hilog -x` 拿的是环形缓冲区，撑不了多久。用户 16:08 的失败，到 17:38 就已经
被冲掉了。**要抓复现，就让用户当场再操作一次，同时挂着实时 `hilog`**，
不要指望事后去捞。

### 日志里怎么认 APP

带 bundleName 的那一列，例如 `A00001/xiaoq.debug.profile/MediaPicker:`。
APP 自己写的是 `hilog.error(0x0001, TAG, ...)`，domain 0x0001 → `A00001`。
按 TAG 或消息文本 grep 比按 domain 稳。

---

## 顺手记一笔：客户端自己拼上传后的 URL 是有风险的

APP 侧 `MediaPicker.upload()` 最后 `return '/api/upload/' + item.name` ——
因为系统 `uploadFile` **不返回响应体**，客户端拿不到服务端 JSON 里的 url，
只能靠「服务端按原名存储」来反推。

服务端 `_safe_upload_name()` 会把非 `[A-Za-z0-9._-]` 的字符换成下划线，
所以**只要文件名里出现空格、中文、`%` 之类，两边就会对不上** ——
表现是「传成功了但引用不到图」。

排查上传问题时如果遇到「传上去 200 但模型说读不到」，先比这个名字。
