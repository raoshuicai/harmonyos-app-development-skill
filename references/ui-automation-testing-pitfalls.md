# uitest 自动化验证：让坐标和日志别骗你

用 `hdc shell uitest` 驱动 APP 做端到端验证时，**最容易烧时间的不是命令不会用，而是拿到的证据是假的**。下面每条都实际踩过。

## 1. 坐标会随布局变 —— 每次操作前重新 dump

**同一个像素位置，在不同状态下可能是完全不同的控件。**

真实案例：设置面板打开时，聊天输入框在 `y=849`；面板关闭后，同一个"位置"变成了聊天区域，而真正的输入框在 `y=2538`。用面板打开时抓的坐标去点，文字会输进一个不存在的框，或者点到别处。

**规则**：
- 每次 UI 状态变化（弹层开/关、页面跳转、列表滚动）后**重新 dump 拿坐标**，不要复用上一轮的值
- 按钮数量变化也会整体挪位（如顶部从 3 个图标按钮加到 4 个，后面所有按钮的 x 都会平移）

拿坐标的写法（顺便算出中心点）：

```python
import json, re
d = json.load(open(path, encoding='utf-8'))
out = []
def w(n):
    a = n.get('attributes', {})
    if a.get('type') == 'Button' and a.get('bounds'):
        out.append(a['bounds'])
    for c in n.get('children', []) or []:
        w(c)
w(d)
for b in out:
    m = re.findall(r'\d+', b)
    if len(m) == 4:
        print(b, '中心=(%d,%d)' % ((int(m[0])+int(m[2]))//2, (int(m[1])+int(m[3]))//2))
```

## 2. `TextInput` 的占位符在 `hint`，不在 `text`

用 `text` 字段判断"输入框在不在"会**误判为空**：

```python
if a.get('type') == 'TextInput':
    print('hint=', a.get('hint', ''), '| text=', a.get('text', ''))
    # 占位符 -> hint；用户输入的内容 -> text
```

密码框的 `text` 是全 `*`（脱敏），**可以用星号数量判断长度**，这也是不读明文就能校验凭据的方法。

## 3. 操作后立刻 dump 可能拿到变化前的状态

弹层关闭、列表刷新这类动作有渲染延迟。刚点完就 dump，经常看到「还在」的旧状态，误判成操作失败。

**做法**：操作后 `sleep 3~5` 再 dump；对"应该消失"的元素**再确认一次**（隔几秒二次 dump），避免把时序问题误诊为功能 bug。

## 4. 分步做，每步 dump 确认 —— 不要连发多个操作

一次连做「关面板 → 点输入框 → 输入 → 点发送」时，中间任何一步落空都会让后续全部作用在错误的控件上，而**最终输出看起来只是"没生效"，完全没有线索**。

**规则**：状态改变类操作一步一确认。输入文字后先 dump 确认文字进去了（`text` 字段有内容），再点发送。

## 5. `inputText` 之前必须 click 聚焦

不聚焦时 `uitest uiInput inputText` 是静默失败的（没有报错，文字就是没进去）。

## 5b. `Toggle`（开关）要点轨道区，点正中心无效

`uitest uiInput click` 点在 Toggle 的**几何中心**时**不触发 `onChange`** —— 中心落在滑块（thumb）上。
要点**轨道左侧**（对未选中）或**右侧**（对已选中）：

```python
# bounds = "[1104,603][1258,694]"
x1, y1, x2, y2 = 1104, 603, 1258, 694
track_x = x1 + 12          # 靠左 12px 处，避开滑块
cy = (y1 + y2) // 2
# hdc shell "uitest uiInput click {track_x} {cy}"
```

**判断点击是否真的命中**：别只看 UI（Toggle 会因数据刷新而回弹，看起来"没变"），
去查**服务端有没有收到请求**（`journalctl ... | grep -c toggle`）—— 这才是命中的硬证据。

**延伸**：任何"点了但状态没变"的情况，优先确认**请求是否发出**，再怀疑业务逻辑。
UI 状态可能因为重新拉取而回弹，掩盖掉一次成功的写入。

## 6. 底部按钮：软键盘会挡住

真机上 `inputText` 会弹软键盘，遮住底部区域，`click` 点在键盘上。改用 `keyEvent 2054`（Enter）走 `onSubmit`。

## 7. ⚠️ 误读日志变量名 —— 最隐蔽的一类错误

**看到的日志字段名，未必是你以为的那个东西。**

真实案例：诊断"收不到广播"时看到日志

```
[WS-DIAG] recv from device-A: ... active_keys=['device-B']
```

据此断定"连接表里只有 B，A 没注册"，还基于这个结论改了连接管理代码。**后来发现 `active_keys` 打印的是 `active_streams`（活跃聊天流）而不是 `active_connections`（WS 连接表）** —— 两个变量名太像，而 `[]` 只说明"没有正在进行的流式聊天"，跟注册完全无关。

**教训**：
- 用日志字段下结论前，**去代码里 grep 一下这个字段到底是哪个变量打出来的**
- 优先找**语义明确的接口**取证，而不是从诊断日志里猜（本例中服务端有个接口能直接列出连接表，一条 curl 就出真相，比翻日志可靠得多）

## 7b. 执行环境：hdc 要在 Windows 侧调，大 dump 不要用 `cat` 取

两个都会让验证莫名卡住：

**① 别在 WSL 里调用 `hdc.exe`。** 在 WSL 的 bash 脚本里写 `HDC="D:/Program Files/.../hdc.exe"`
会报 `No such file or directory`（要么改成 `/mnt/d/...` 形式，要么**直接在 git-bash 侧跑脚本**）。
后者更省事：`bash /d/tmp_script.sh`，`hdc.exe` 和 Windows 的 `python` 都能直接用。
本会话前半段反复"脚本跑不通"，根因就是执行环境选错了。

**② `hdc shell cat <dump.json>` 对大文件会截断。** UI dump 到 100KB 量级时，
`cat` 回传的内容不完整 → JSON 解析报 `Expecting value: line 1 column 1`，
看起来像"dump 没生成"，其实是**传输被截断**。

三个可用的替代（按推荐顺序）：
- 直接在 Windows 侧跑脚本，`hdc shell cat` 小文件没问题，大文件也一样会截断 ——
  **只提取需要的字段**（见下）
- **只取需要的字段**：dump 是单行 JSON，在设备端就地 grep：
  `hdc shell "grep -o '\"text\":\"[^\"]*\"' /data/local/tmp/x.json"`
- `hdc file recv`（跨 WSL/Windows 时路径容易出错，不如前两个稳）

> 验证 UI 状态时**不需要整份 dump**。把"要确认什么"先想清楚（某个文本变没变、
> 某个 bounds 在哪），只提取那几项，又快又不会踩截断。

---

## 7c. 性能：`aboutToAppear` + `onPageShow` 会双重加载

最容易忽略、又最容易修的一类白干：

```ts
aboutToAppear(): void { this.load(); }   // ❌
onPageShow(): void    { this.load(); }   // ❌ 首次显示也会触发
```

`onPageShow` **在页面首次显示时同样触发**，所以进页面第一件事就是发两遍完全相同的请求。
页面越多这个浪费越分散，单看每个页面都不觉得有问题。

**约定：加载只挂 `onPageShow`**（首次与回到前台都覆盖），`aboutToAppear` 只留组件级初始化。

**验证方法**（不要靠"感觉变快了"）：改动前后各测一次服务端请求数。

```bash
# 进页面前记基线，进页面后再数一次，差值应 = 1
journalctl --user -u <svc> --since '12 seconds ago' --no-pager | grep -c 'GET /api/xxx'
```

**同类检查点**：`List` 里 `ForEach` 大数据集（>100 项）会全量渲染组件，
可先做客户端分页（一次渲染 40 项，触底追加），比直接上 `LazyForEach` 省事得多。

---

## 8. 造测试数据的两个原则

- **先备份后破坏**：验证"全部已读"这类会把数据改掉的功能时，先备份数据文件，测完恢复
- **先归零再递增**：验证角标/计数递增时，如果初始值已经是 `99+`，造一条新数据根本看不出变化。**先清到 0，再造数据**，才能看到 `1 → 2` 的清晰证据
