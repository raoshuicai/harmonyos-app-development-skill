# request.uploadFile 只吃应用沙箱文件（相册/文档 URI 会被参数校验拒掉）

## 症状

APP 里选照片上传，界面提示"上传失败"，但**服务端一条请求日志都没有**
（uvicorn access log 里既没有 POST /api/upload，也没有任何 4xx/5xx）。

真机 hilog 里能看到真正的原因：

```
MediaPicker: picked image uri=file://media/Photo/13058/IMG_1789787436_22980/screenshot_xxx.jpg
MediaPicker: uploading screenshot_xxx.jpg to http://192.168.3.87:8866/api/upload
MediaPicker: upload threw: The parameters check fails
             Parameter verification failed, user file can only for request.agent.
```

关键点：报错发生在 `await request.uploadFile(ctx, cfg)` **这一行**，
请求根本没离开手机。所以排查时「服务端收不到」不等于「网络不通」。

## 根因

`@ohos.request` 的 `uploadFile` 只接受**应用自己沙箱里的文件**。
相册（`file://media/Photo/...`）和文档选择器给出的 URI 属于系统媒体库/文档域，
不是应用自己的文件，于是在 UploadConfig 参数校验阶段就被拒，
报错文案就是 `user file can only for request.agent`。

注意 `PhotoViewPicker` / `DocumentViewPicker` 选出来的 URI 能直接用 `fs` 读
（应用拿到了临时读权限），但**不能**直接交给 `request.uploadFile`——
上传任务跑在独立的系统服务进程里，不继承应用的临时授权。

## 正确做法：先落沙箱，再上传

```ts
import { fileIo as fs } from '@kit.CoreFileKit';

/** 把选择器给的 URI 拷进应用沙箱，返回 request.uploadFile 能吃的 file:// URI */
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

然后 `files[0].uri` 用 `copyToSandbox()` 的返回值，其余 UploadConfig 不变
（`url` / `header` / `method: 'POST'` / `files` / `data` 都照旧，`data` 是必填）。

上传完成后删掉沙箱副本，别让 cacheDir 长胖：

```ts
try { fs.unlinkSync(ctx.cacheDir + '/' + item.name); } catch (e) { /* 清理失败无所谓 */ }
```

放 `cacheDir` 而不是 `filesDir`：系统会在存储紧张时自动回收 cacheDir，
而且这份副本本来就是一次性的。

## 排查这类问题的顺序（省时间）

1. 先看服务端 access log 有没有这条请求 —— **没有**就说明还卡在客户端，
   别在服务端和网络里绕。
2. 确认手机走的是哪条路：APP 的 `ApiConfig.detect()` 按 WiFi SSID 切
   内网 `http://192.168.3.87:8866` 或公网 `https://xiaoq.xiao-q.com`。
   hilog 里的 `uploading ... to <url>` 直接告诉你。
3. 抓真机 hilog 定位到具体抛错行。

## 抓真机 hilog 的坑

- `hdc list targets -v` 看清设备：`127.0.0.1:5555` 是**模拟器**，
  真机是 USB 那条（形如 `5JV0225B15002640`）。抓错设备会得到"一条日志都没有"
  的假象。
- 必须带 `-t <设备>`，否则静默失败或报
  `[Fail]ExecuteCommand need connect-key`。
- `hilog -x` 只dump一次且缓冲区滚得快；要复现就先用
  `timeout 420 hdc -t <设备> hilog > /tmp/phone_cap.txt 2>&1` 挂后台，
  再让用户操作一次。
- APP 的 `hilog.error(0x0001, TAG, ...)` 在 hilog 里长这样，
  直接 grep 业务 TAG 最快：
  `E A00001/xiaoq.debug.profile/MediaPicker: ...`
