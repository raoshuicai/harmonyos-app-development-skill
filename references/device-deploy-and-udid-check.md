# 真机部署：UDID 不是序列号（签名白名单校验的是 UDID）

## 最容易误判的一点

`hdc list targets` 显示的是**设备序列号**，而 AGC 签名 profile（`.p7b`）白名单里校验的是 **UDID** —— 两者完全不同：

```
hdc list targets        ->  5JV0225B15002640        ← 序列号（16 位）
hdc -t <sn> shell "bm get --udid"
                        ->  FA86A714A936FA61A7072AA43E9ED89C606FF211DF3AC6C6C8C4F200F4ABF33C   ← 真正的 UDID（64 位）
```

拿序列号去 AGC 比对白名单会得出"设备没注册"的**错误结论**，然后白白去重办证书。

## 部署前先查白名单，别直接试

debug 类型的 `.p7b` 内含允许安装的设备列表。它是 PKCS#7 包装的 JSON，直接抓 JSON 段即可：

```python
import io, re
raw = io.open(r"D:\05_HarmonyNext\xiaoq-debug-profileDebug.p7b", 'rb').read()
txt = raw.decode('latin-1')
i = txt.find('{"version-name"')
depth = 0; j = i
while j < len(txt):
    if txt[j] == '{': depth += 1
    elif txt[j] == '}':
        depth -= 1
        if depth == 0: break
    j += 1
print(txt[i:j+1])
```

关键字段：

```json
{
  "type": "debug",
  "bundle-info": {"bundle-name": "xiaoq.debug.profile", ...},
  "debug-info": {"device-ids": ["FA86A714...F33C"], "device-id-type": "udid"},
  "validity": {"not-before": 1781353736, "not-after": 1812889736}
}
```

- `device-ids` 里**没有**目标 UDID → 必须去 AGC 加设备并重新生成 profile，试也白试
- `validity.not-after` 是**秒级时间戳**，换算出到期日；过期的 profile 装不上（现象是安装报签名校验失败）

**同一台机装过别的 APP ≠ 这个 bundleName 能装**：白名单是**按 profile** 绑定的，不同 bundleName 走各自的 p7b。

## 本项目的部署方式

```bash
# 必须显式指定设备，否则可能装到另一台
powershell -ExecutionPolicy Bypass -File build-deploy.ps1 -SkipSync -DeviceId 5JV0225B15002640
```

`build-deploy.ps1` 的 `-DeviceId` 默认是模拟器（`127.0.0.1:5555`）。**同时插着真机和模拟器时必须显式指定**，否则会装错设备。

装完用 `bm dump` 核对版本，别信脚本回显：

```bash
hdc -t <device> shell "bm dump -n <bundleName> | grep -E 'versionName|versionCode'"
```

## 真机 vs 模拟器的行为差异

| 项 | 模拟器 | 真机 |
|---|---|---|
| 软键盘 | 不弹（可点到底部按钮） | **弹出后会遮住底部按钮，`uitest uiInput click` 点不到** |
| WiFi SSID | `VirtWifi` 之类的虚拟值 | 真实 SSID（网络判定分支会走另一条路）|
| Push Kit | 不生效 | 真实系统通知（唯一能验证 Push Kit 的地方）|
| Preferences | 各自独立 | 各自独立 —— **旧版装过则保留**，全新安装需要重新配置凭据 |

**软键盘遮挡的处理**：不要用坐标点底部按钮，改用键盘事件走 `onSubmit`：

```bash
hdc -t <device> shell "uitest uiInput keyEvent 2054"   # Enter
```

## 凭据（API key 等）不要替用户输入

真机全新安装时 Preferences 为空，需要用户自己填。助手侧的职责是**验证凭据是否完好**，而不是输入：

- 用密码框的**星号个数**判断内容长度是否与预期一致（如 32 位 key → 32 个 `*`），**不要读取或回显明文**
- 一旦怀疑被误改，同样只看长度
