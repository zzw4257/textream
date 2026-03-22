# Migration Note: Stable Default Experience

本说明描述旧用户升级到这一轮默认体验后的兼容行为。

## 迁移原则

- 已保存的用户设置优先，不强行覆盖。
- 只有“此前从未保存过”的字段，才会采用新的默认值。
- 旧 preset 和旧协议字段继续兼容，不做破坏性迁移。

## 设置兼容行为

### TrackingGuard

- 如果用户之前已经保存过 `matchWindowSize`、`advanceThreshold`、`offScriptFreezeDelay`，升级后继续沿用原值。
- 如果这些 key 之前不存在，升级后会采用新的保守默认值：
  - `matchWindowSize = 6`
  - `advanceThreshold = 3.4`
  - `offScriptFreezeDelay = 0.9`
- `legacyTrackingFallbackEnabled` 仍保留且默认开启，但在 `strictTrackingEnabled = true` 时不会主导默认体验，只作为止损回退开关存在。

### Persistent HUD

- 如果用户以前已经手动配置过 `persistentHUDEnabled` 或 `hudModules`，升级后保持不变。
- 如果用户从未配置过 HUD，本次升级会启用最小 HUD：
  - `persistentHUDEnabled = true`
  - `hudModules = [trackingState, expectedWord]`
- 这意味着旧用户在“无显式 HUD 偏好”的情况下，升级后会看到一个更克制、但默认可见的 HUD。

### Attached Overlay

- `attachedFallbackBehavior` 默认收敛为 `screenCorner`。
- `attachedHideWhenWindowUnavailable` 默认是 `false`，因此 attached 失败时优先回退到屏幕角落，而不是直接消失。
- 如果用户已经保存过目标窗口 ID / 标签，升级后仍会尝试继续绑定；若目标窗口当前不可用，行为改为显式 fallback。

## Preset 兼容行为

- 推荐展示的 built-in presets 现在只保留：
  - `Interview`
  - `Live Stream`
  - `Presentation`
- `Dual Display` 和 `Sidecar iPad` 没有被删除：
  - 旧的 `activeLayoutPreset` 值仍可正常解析
  - 旧自定义 preset 中引用这些模式也仍能使用
  - 只是它们不再作为默认推荐模板展示
- 如果旧用户当前仍处于 `Dual Display` 或 `Sidecar iPad`，不会被自动改写；Settings 会提示这属于保留兼容 preset，建议需要长期保留时另存为 custom preset。

## 远端兼容行为

- Browser / Director 的新增字段仍然是向后兼容扩展。
- 旧客户端继续读取已有字段：
  - `highlightedCharCount`
  - `words`
  - `audioLevels`
  - `lastSpokenText`
- 新客户端可额外读取：
  - `trackingState`
  - `confidenceLevel`
  - `expectedWord`
  - `nextCue`
  - `manualAsideActive`

## 对升级用户的实际影响

最容易感知到的变化只有四点：

1. Word Tracking 会比以前更保守，更倾向于先停住再确认。
2. attached 丢锚时更常见的是“退回屏幕角落”，而不是“看起来像消失了”。
3. 如果你之前没有显式配置 HUD，升级后会看到默认开启的最小 HUD。
4. Settings 中推荐 preset 变少，但旧 preset 不会丢。
