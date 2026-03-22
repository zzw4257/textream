# Textream 产品化回归清单

## 1. 可执行 macOS 回归矩阵

### 1.1 回归前准备

- 构建基线：
  `xcodebuild -project Textream/Textream.xcodeproj -scheme Textream -configuration Debug -sdk macosx CODE_SIGNING_ALLOWED=NO CODE_SIGNING_REQUIRED=NO build`
- 测试基线：
  `xcodebuild -project Textream/Textream.xcodeproj -scheme Textream -configuration Debug -sdk macosx CODE_SIGNING_ALLOWED=NO CODE_SIGNING_REQUIRED=NO test`
- 定向 smoke tests：
  `xcodebuild -project Textream/Textream.xcodeproj -scheme Textream -configuration Debug -sdk macosx CODE_SIGNING_ALLOWED=NO CODE_SIGNING_REQUIRED=NO -only-testing:TextreamTests/TrackingGuardTests -only-testing:TextreamTests/WindowAnchorServiceTests -only-testing:TextreamTests/RemoteStateCompatibilityTests -only-testing:TextreamIntegrationTests/RemoteStateCompatibilityIntegrationTests test`
- 打开 `Settings -> QA & Debug`
- 如需看 overlay 内部状态：
  打开 `Show Debug Overlay`
- 如需记录操作轨迹：
  打开 `Tracking Logs` 与 `Anchor Logs`
- Attached 模式建议准备：
  1. 至少一个主流多窗口 App，例如 Finder / Safari / Chrome
  2. 一台外接显示器或 Sidecar
  3. 一段包含正常朗读、插话、暂停词 `[pause]` 的脚本

### 1.2 观察信号

- Tracking 观察点：
  `TRACK <state> | expected <word> | conf <level> | freeze <reason>`
- Anchor 观察点：
  `ANCHOR <AX|Quartz|Fallback> | AX on/off | <window> | <frame> | <message>`
- QA 面板观察点：
  `Live Tracking`、`Live Anchor`、`Recent QA Logs`

### 1.3 场景矩阵

| 场景 | 覆盖模式 | 操作步骤 | 预期行为 | 调试信号 |
|---|---|---|---|---|
| 多显示器主副屏切换 | pinned / floating / attached / fullscreen | 在主屏启动，再把主操作切到副屏；切换固定显示器设置 | overlay 不丢失、不闪烁；fullscreen 重新落到目标屏 | QA 面板 `Live Anchor` 或屏幕选择项同步 |
| 外接显示器拔插 | attached / fullscreen / external | attached 绑在外接屏窗口后拔掉显示器；fullscreen 正在外接屏时拔掉 | attached 立即走 fallback；fullscreen 正常结束或回主屏，不遗留不可见 panel | Anchor 来源切为 `Fallback`，日志记录显示器切换后的 fallback |
| 全屏 App | attached | 选中一个全屏 App 窗口，再进入/退出全屏 | 拿不到稳定窗口矩形时不抖动，直接 fallback 或隐藏 | Anchor 来源在 `AX/Quartz/Fallback` 间可解释，不能无声失联 |
| Stage Manager | attached | 在 Stage Manager 中切换前后台舞台、收起/展开侧边窗口组 | 目标窗口不可见时进入 fallback；重新出现后恢复附着 | Anchor 日志出现 `window unavailable -> fallback -> AX/Quartz relock` |
| Mission Control / Spaces 切换 | attached / pinned / floating | 快速切换 Space、Mission Control 中拖动窗口到另一 Space | overlay 不能残留在错误 Space 上乱闪；attached 无目标时 fallback | Anchor 日志和 `Live Anchor` 一致 |
| Accessibility 未授权 | attached | 关闭授权后启动 attached | 允许进入 attached，但默认走 Quartz / fallback，并给出解释 | `AX Trusted = No`，来源优先 `Quartz` 或 `Fallback` |
| Accessibility 授权中 / 授权后返回 | attached | attached 启动后打开系统设置授权，再回到 App | 无需重启 overlay；后续几何解析自动切换到 AX | Anchor 来源从 `Quartz` 升级为 `AX` |
| Accessibility 撤销授权 | attached | attached 运行中撤销权限 | 不崩溃；自动退回 Quartz / fallback | Anchor 来源从 `AX` 降到 `Quartz` 或 `Fallback` |
| AX 成功 / AX 失败走 Quartz | attached | 多窗口 App 中选择目标窗口，重复移动/缩放 | 能看出当前来源；AX 失败时不会误贴到其它窗口 | QA 面板明确显示 `AX` 或 `Quartz` |
| 目标窗口最小化 / 隐藏 / 关闭 / 切后台 | attached | 最小化、隐藏、关闭目标窗口；把 App 切后台再切回 | overlay 根据设置隐藏或 fallback；目标回来后自动恢复附着 | Anchor 日志出现 unavailable / fallback / relock |
| attached 下移动 / 缩放 / 跨屏拖拽 | attached | 拖动目标窗口、改变尺寸、跨屏移动，尤其让窗口贴住上下左右边缘 | overlay 持续跟随，不掉队、不明显滞后；上下边缘也按目标窗口贴紧；不应出屏 | Anchor frame 连续更新，来源在 `AX / Quartz / Fallback` 间可解释 |
| pinned / floating / attached / fullscreen 来回切换 | 全部 | 运行中不断切模式 | 无残留 panel、无 hotkey 泄漏、无错误 anchor 状态 | QA 面板 `Live Anchor` 在非 attached 时回到 `Inactive` |
| Tracking 正常朗读 | word tracking | 正常逐词朗读脚本 | state 保持 `Tracking`，expectedWord 持续前移 | `freeze None`，detail 为 advance 信息 |
| Tracking 插话 / 旁白 | word tracking | 朗读中说一段脱稿旁白 2 秒 | state 进入 `Uncertain/Lost`，高亮冻结，不明显误推进 | `freeze Off-script Audio / Low Match Score` |
| Hold to Ignore / Aside | word tracking | 按住 `Fn`、双击 `Option`，再恢复 | 100ms 内冻结；释放或再次双击后 1 秒内重锁 | `freeze Manual Aside / Recovery Pending` |
| `[pause]` / 跳词 / 漏词 / 重复词 | word tracking | 使用包含注释词和口语化变化的脚本 | 不因单个弱命中而乱跑；允许重新锁定 | QA 面板 detail 体现 low score / insufficient words |
| `[wave]` / bracket cue 自动跳过 | word tracking | 在脚本中插入 `[wave]`、`[smile]` 等括号 cue，只朗读正文 | cue 保留显示样式，但 tracking 不等待它被念出；done 不被括号 cue 卡住 | `Expected` 直接跳到下一个正文词，末尾时 `Expected = -` |
| HUD 默认关闭 / 模块全关 | pinned / floating / attached / fullscreen | 关闭 Persistent HUD，或清空 HUD modules 后打开 overlay | overlay 顶部不应留下空白占位；预览和实际表现一致 | 关闭 HUD 后 `PersistentHUDStripView` 不渲染，QA overlay 仍可独立开启 |
| Teleprompter 模式专属设置 | fullscreen / attached / floating | 分别切到 fullscreen、attached、floating 检查 Teleprompter 页 | 只展示当前模式相关控件；fullscreen 不出现 pointer follow；attached 不混入跟鼠标走入口 | 设置页内容和模式一致，底部不会因为无关 section 造成误解 |
| Browser 旧客户端兼容 | browser remote | 用旧字段子集消费当前状态帧 | 旧客户端忽略新增字段，不断协议 | 回归测试 `RemoteStateCompatibilityTests` 通过 |
| Director 旧客户端兼容 | director | 用旧字段子集消费当前状态帧 | 旧客户端忽略新增字段，不断协议 | 回归测试 `RemoteStateCompatibilityTests` 通过 |

## 2. 开发态调试面板与日志开关

### 2.1 入口

- `Settings -> QA & Debug`

### 2.2 开关说明

- `Show Debug Overlay`
  直接在 teleprompter overlay 内展示 Tracking 与 Anchor 的可视化标签
- `Tracking Logs`
  将 `TrackingGuard` 的状态机变化、freeze 原因、恢复阶段写入 QA 日志流
- `Anchor Logs`
  将 `WindowAnchorService` 的 `AX / Quartz / fallback` 决策变化写入 QA 日志流

### 2.3 UI 读法

- `Live Tracking`
  用来确认当前 `state / expected word / confidence / freeze reason / detail`
- `Live Anchor`
  用来确认当前是否走 `AX / Quartz / fallback`，以及是否具备 Accessibility 授权
- `Recent QA Logs`
  用来保留跨场景切换时的决策轨迹，方便复盘“为什么没跟上 / 为什么 freeze / 为什么 fallback”

## 3. 发现的问题清单与修复

### 问题 1：Attached fallback 绑定了启动时的旧屏幕

- 复现步骤：
  1. 在外接显示器上的目标窗口启用 attached
  2. 最小化目标窗口或直接拔掉外接显示器
- 预期行为：
  overlay 立即退回当前可见屏幕的角落，不留在已经失效的屏幕坐标系里
- 修复前实际行为：
  fallback 使用的是 attached 启动时捕获的 `fallbackScreen`，屏幕拓扑变化后可能退回到陈旧坐标，出现“看似消失”或退回到错误显示器的风险
- 修复内容：
  fallback 屏幕改为按当前 panel 所在屏幕动态解析；找不到时回退到主屏
- 当前结论：
  代码路径已收敛，仍需实机覆盖“外接屏拔插 + Space 变化”联动场景

### 问题 2：AX 多窗口匹配只按标题，容易附着到错误窗口

- 复现步骤：
  1. 打开同一 App 的多个窗口，标题相同或标题为空
  2. 选择其中一个窗口进入 attached
  3. 移动或缩放目标窗口
- 预期行为：
  AX 命中应优先选择与 Quartz bounds 最接近的那个窗口
- 修复前实际行为：
  AX 路径在标题相同或空标题时会命中第一个窗口，导致 overlay 跟错 sibling window
- 修复内容：
  AX 匹配改为“标题优先 + 与 Quartz bounds 的几何距离评分”，不再只靠标题第一个命中
- 当前结论：
  常见多窗口场景的风险已经明显下降，仍建议用 Finder / Safari / Chrome 做实机复查

### 问题 3：Tracking freeze 缺少可视化原因，QA 很难判断是误判还是有意冻结

- 复现步骤：
  1. 朗读过程中插话 2 秒
  2. 或者按住 `Fn` 触发 hold-to-ignore
- 预期行为：
  QA 能直接看到当前 state、expectedWord、confidence 和 freeze reason
- 修复前实际行为：
  overlay 只有笼统状态文案，无法区分 `Manual Aside / Off-script / Recovery Pending / Low Match Score`
- 修复内容：
  `TrackingGuard` 新增 `decisionReason + debugSummary`；QA 面板和 overlay 调试条会直接展示 freeze reason
- 当前结论：
  现在可以在不连 Xcode 的情况下判断“它为什么没动”

### 问题 4：Remote / Director 协议扩展需要显式兼容验证

- 复现步骤：
  1. 用只识别旧字段的客户端消费当前 `BrowserState / DirectorState`
  2. 发送带新增字段的状态帧
- 预期行为：
  旧客户端忽略新增字段，不中断连接
- 修复前实际行为：
  设计上是向后兼容，但缺少自动化验证，发布前心智模型不够稳
- 修复内容：
  新增 `RemoteStateCompatibilityTests`，用旧字段子集解码当前状态帧，显式验证 JSON 扩展兼容
- 当前结论：
  协议层兼容性现在有自动化回归兜底

### 问题 5：Bracket cue 如 `[wave]` 会阻塞 strict tracking 与 done

- 复现步骤：
  1. 脚本写成 `hello [wave] there`
  2. 用户只朗读 `hello there`，不念 `[wave]`
  3. 或者脚本以 `hello [wave]` 结尾
- 预期行为：
  方括号 cue 只保留视觉提示，不应成为必须命中的 tracking token；末尾 cue 也不应阻塞 done
- 修复前实际行为：
  `[wave]` 的字母内容被归一化后参与匹配，导致 expectedWord 卡在 cue 上，末尾 cue 甚至会让页面无法完成
- 修复内容：
  bracket cue 统一标记为 `styled annotation + auto skip`；`TrackingGuard` 的 token 列表、expectedWord、nextCue、done 尾部推进全部只面向参与 tracking 的正文词
- 当前结论：
  `TrackingGuardTests` 已覆盖 `[wave]` 自动跳过和尾部 done，两条路径均通过

### 问题 6：Persistent HUD 在无内容时仍然占位，造成 teleprompter 顶部空白

- 复现步骤：
  1. 关闭 `Persistent HUD`
  2. 或保留 HUD 开关但去掉所有模块
  3. 打开 preview、notch、floating 或 external teleprompter
- 预期行为：
  HUD 没有内容时不渲染，也不应该给顶部预留空白条
- 修复前实际行为：
  预览和实际 overlay 都会保留一段无内容的垂直间距，表现为“顶部多了一截空白”
- 修复内容：
  preview、notch、floating、external 四条渲染路径都改为 `items.isEmpty` 时不插入 HUD strip；QA debug overlay 也改成单独按开关显示
- 当前结论：
  代码路径已统一，仍建议手工确认“HUD 关掉时预览与实窗是否完全一致”

### 问题 7：Teleprompter 设置页混入跨模式控件，Fullscreen/Attached 易误解

- 复现步骤：
  1. 切到 `Fullscreen`
  2. 打开 `Settings -> Teleprompter`
  3. 观察是否仍出现 `Pointer Follow` 等只属于 floating 的控件
- 预期行为：
  Teleprompter 页只展示当前模式相关的控制项；切到别的模式不应继续暴露无关入口
- 修复前实际行为：
  `Pointer Follow` 被做成全局快捷区，容易造成“Fullscreen 也该跟鼠标走”的错误心智；内容变多时也更容易让设置页下半部分显得拥挤
- 修复内容：
  `Pointer Follow` 重新收口到 floating 专属 section；attached 只保留窗口绑定、角落、margin 和 attached size；fullscreen 保持显示器与退出提示
- 当前结论：
  设置页信息架构已经更贴近模式心智，需继续做一次小屏幕窗口高度的人工检查

### 问题 8：Attached 在贴边或跨屏时可能按“整块屏幕角”选屏，导致出屏或上下不够紧

- 复现步骤：
  1. 把目标窗口拖到显示器边缘，尤其是跨屏分界处
  2. 使用 `Top Left / Top Right / Bottom Left / Bottom Right` 四个角反复切换
  3. 观察 attached overlay 的落点
- 预期行为：
  选屏与 clamp 应该围绕“目标窗口的内部角落”判断，而不是一旦触到屏幕边界就按整块屏幕算
- 修复前实际行为：
  锚点直接取窗口边界角点时，跨屏或贴边场景可能把 overlay 算到错误屏幕上，出现出屏或上下边缘不够贴紧
- 修复内容：
  选屏锚点改成“窗口内侧探针点”，并保留 visible frame clamp，降低跨屏边界的误选概率
- 当前结论：
  `WindowAnchorServiceTests` 已覆盖 top/bottom clamp；但真实多显示器 + Stage Manager 联动仍需人工复核

## 4. 回归结论与剩余风险

### 4.1 当前结论

- P0/P1/P2 主线已经具备产品化收尾所需的回归支撑：
  - 有可执行回归矩阵
  - 有 Settings 内 QA 面板
  - 有 overlay 内调试标签
  - 有 Tracking / Anchor 两条日志流
  - 有 Remote / Director 兼容性自动化验证
- 当前最适合进入“实机手工回归 + 问题单闭环”的阶段，而不是继续扩功能
- 已完成的代码级验证：
  - 无签名 Debug 构建通过
  - `TrackingGuardTests` 通过
  - `WindowAnchorServiceTests` 通过
  - `RemoteStateCompatibilityTests` 与 `RemoteStateCompatibilityIntegrationTests` 通过
- 自动化证据：
  - `TrackingGuardTests` xcresult:
    `/tmp/textream-productize-tests-2/Logs/Test/Test-Textream-2026.03.21_13-05-03-+0800.xcresult`
  - `WindowAnchorService / Remote compatibility` xcresult:
    `/tmp/textream-qa-evidence/Logs/Test/Test-Textream-2026.03.21_13-12-02-+0800.xcresult`

### 4.2 剩余风险

- `全屏 App / Stage Manager / Mission Control / Spaces` 仍然强依赖系统窗口可见性语义，无法仅靠单元测试完全覆盖
- `外接显示器拔插` 已修正 fallback 路径，但仍需真实硬件做拔插节奏与多 Space 联动测试
- `AX -> Quartz -> fallback` 的切换现在可观察、可解释，但个别第三方 App 仍可能暴露出不稳定的窗口元数据
- 设置窗口与 preview panel 的相对位置仍需要一轮人工视觉确认，尤其是小屏幕或较低窗口高度时，确认不会让用户误以为“底部被覆盖”

### 4.3 建议退出标准

- 上表所有场景至少完成一轮手工回归
- `Recent QA Logs` 中没有出现无法解释的 source 抖动或 state 抖动
- Attached 相关问题单只保留“平台限制类”风险，不再保留“可修但未修”的工程问题
