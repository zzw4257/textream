# Textream Stable Defaults

本文件定义这一轮“稳定默认体验”的推荐默认值。目标只有一个：

- 宁可停，不乱走
- attached 失败时优先回退到屏幕角落
- HUD 默认只给出最关键状态
- built-in presets 只保留最少集合

## TrackingGuard

| 参数 | 推荐默认值 | 说明 |
| --- | --- | --- |
| `strictTrackingEnabled` | `true` | 默认启用 Guarded Word Tracking。 |
| `legacyTrackingFallbackEnabled` | `true` | 保留旧逻辑回退开关，但仅在 `strictTrackingEnabled = false` 时生效。 |
| `manualAsideHotkey` | `optionDoubleTap` | 双击 `Option` 切换 `Aside`。 |
| `temporaryIgnoreHotkey` | `fnHold` | 按住 `Fn` 临时冻结 tracking。 |
| `matchWindowSize` | `6` | 收窄候选窗口，减少脱稿时误推进。 |
| `advanceThreshold` | `3.4` | 提高推进门槛，优先保证“不乱走”。 |
| `offScriptFreezeDelay` | `0.9` | 在两帧未命中且持续约 0.9 秒后进入 `lost`。 |

## Attached Overlay

| 参数 | 推荐默认值 | 说明 |
| --- | --- | --- |
| `attachedAnchorCorner` | `topRight` | 默认贴在目标窗口右上角。 |
| `attachedMarginX` | `16` | 水平外边距。 |
| `attachedMarginY` | `14` | 垂直外边距。 |
| `attachedFallbackBehavior` | `screenCorner` | 默认失败后回退到屏幕角落，不直接消失。 |
| `attachedHideWhenWindowUnavailable` | `false` | 默认保持可见 fallback，便于用户理解当前状态。 |
| `attachedTargetWindowID` | `0` | 首次启动不预选目标窗口。 |
| `attachedTargetWindowLabel` | `""` | 首次启动无窗口标签。 |
| `hasSeenAttachedOnboarding` | `false` | 首次进入 attached 时展示权限引导。 |

## Layout Presets

推荐展示的 built-in presets 只保留三套：

| Preset | Overlay | HUD | 用途 |
| --- | --- | --- | --- |
| `Interview` | `floating` | `trackingState` | 小窗、克制、适合访谈或会议角落。 |
| `Live Stream` | `attached` | `trackingState + expectedWord` | 贴窗使用，优先兼顾状态与当前词。 |
| `Presentation` | `pinned` | `trackingState + expectedWord` | 大字、高可读性，适合演讲。 |

兼容性保留：

- `Dual Display`
- `Sidecar iPad`

它们仍然可以被旧配置识别，但不再作为推荐默认模板展示。

## Persistent HUD

| 参数 | 推荐默认值 | 说明 |
| --- | --- | --- |
| `activeLayoutPreset` | `custom` | 基线启动体验不强行套某个 preset。 |
| `persistentHUDEnabled` | `true` | 默认开启最小 HUD，帮助解释 tracking / fallback 状态。 |
| `hudModules` | `[trackingState, expectedWord]` | 只显示“当前状态 + 当前预计词”，避免首开过载。 |

## 默认体验摘要

- 首次启动使用基线 `custom` 布局，而不是自动套某个强场景 preset。
- HUD 默认开启，但只显示最关键的两项。
- attached 一旦失去稳定锚点，默认回退到屏幕角落。
- strict tracking 默认更保守，更快冻结、更晚推进。
