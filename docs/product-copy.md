# Product Copy

本文件收敛这一轮默认体验中对用户可见的统一文案。原则：

- 同一状态只保留一种主说法
- HUD、status line、引导弹窗尽量使用同一套词
- 优先解释“为什么现在没动 / 为什么退回角落”

## Tracking States

| 状态 | UI Label | Status Line | 使用场景 |
| --- | --- | --- | --- |
| `tracking` | `Tracking` | `Tracking: {word}` 或 `Tracking your script` | 正常跟随脚本推进。 |
| `uncertain` | `Checking Script` | `Heard you. Checking the script before moving.` | 识别到了语音，但还不够确定，不推进。 |
| `aside` | `Aside` | `Aside active. Tracking is paused while you hold.` / `Aside mode is on. Tracking is paused.` | 手动旁白或临时忽略。 |
| `lost` | `Off Script` | `Off script. Waiting to lock back on.` | 连续未命中脚本窗口，已冻结等待重锁。 |

## Attached Short Copy

这些短文案用于 HUD / `statusLine` / attached 诊断区。

| 场景 | 短文案 |
| --- | --- |
| 未选目标窗口 | `No target window selected; using screen corner` |
| 未授予 Accessibility | `Accessibility off; using screen corner` |
| AX 失败但 Quartz 可兜底 | `Using visible window bounds (AX fallback)` |
| 已授权但目标窗口不可读 | `Can't read selected window; using screen corner` |
| 目标窗口丢失 | `Target window lost; back to screen corner` |
| 用户主动选择隐藏 fallback | `Target window unavailable; overlay hidden` |

## Attached Long Copy

这些长文案用于 onboarding、诊断区说明和 QA 日志。

| 场景 | 长文案 |
| --- | --- |
| 未选目标窗口 | `Choose a target window to attach to. Until then, Textream will stay in the screen corner.` |
| 未授予 Accessibility | `Attached Overlay needs Accessibility access before it can follow another app window. Textream will stay in the screen corner until access is granted.` |
| AX 正常附着 | `Attached using Accessibility geometry for {target}.` |
| Quartz 兜底 | `Accessibility geometry is unavailable for {target}. Textream is following the visible window bounds instead.` |
| 已授权但窗口不可读 | `Accessibility is enabled, but macOS is not exposing usable geometry for {target}. Textream is staying in the screen corner.` |
| 目标窗口丢失 | `The selected window is no longer available. Textream moved back to the screen corner.` |
| fallback 被隐藏 | `The selected window is unavailable, so the overlay was hidden by the attached fallback setting.` |

## Onboarding Copy

| 位置 | 文案 |
| --- | --- |
| 标题 | `Allow Accessibility for Attached Overlay` |
| 说明 | `Attached Overlay needs Accessibility access before it can follow another app window. Until access is granted, Textream will stay in the screen corner.` |
| 主按钮 | `Open System Settings` |
| 次按钮 | `Continue with Fallback` |

## Launch Guide Copy

| 位置 | 文案 |
| --- | --- |
| 标题（未进入 attached） | `Accessibility unlocks Attached Overlay` |
| 标题（当前就是 attached） | `Attached Overlay is using screen-corner fallback` |
| 说明（未进入 attached） | `If you want Textream to follow another app window, grant Accessibility before you use Attached Overlay. Until then, Textream will fall back to the screen corner instead of silently failing.` |
| 说明（当前就是 attached） | `Accessibility is still off, so Textream cannot lock onto other app windows yet. Attached Overlay will stay in the screen corner until you allow access.` |
| 次按钮 | `Review Attached Setup` |
| 关闭按钮 | `Later` |

## 用词约束

- 统一使用 `screen corner`，不要混用 `screen-corner fallback`。
- 统一使用 `Off Script`，不要再对外显示 `Lost`。
- 统一使用 `Checking Script`，不要再对外显示只有内部感的 `Uncertain`。
- `fallback` 这个词可以保留在 QA / 诊断语境里，但面向普通用户时优先用“using screen corner”或“using visible window bounds”。
