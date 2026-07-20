# Codex Skill / Plugin 使用说明

这份笔记记录我当前常用的 Codex 扩展能力。先区分几个概念：

- `Skill`：一组本地说明文档和可选脚本，用来让 Codex 在特定任务里按固定流程工作。例如前端设计评审、图片生成、反过度工程化 review。
- `Plugin`：更完整的扩展包，可以包含 skills、hooks、MCP/app 能力、命令和 marketplace 信息。例如 `ponytail`。
- `Hook`：插件里的自动触发脚本，常见于会话启动、用户输入、文件变更后自动注入规则或做检查。
- `AGENTS.md`：项目或全局规则。适合放长期偏好和项目约束，不适合塞太长的执行流程。

推荐原则：

```text
长期偏好写 AGENTS.md
专项流程做成 skill
需要命令、hook、跨平台分发时再用 plugin
```

## 当前分类总览

| 类别 | 名称 | 主要用途 | 默认使用方式 |
| --- | --- | --- | --- |
| 反过度工程化插件 | `ponytail` | 控制 Codex 少写无必要代码、少加抽象、少引入依赖 | 默认关闭，需要时 `@ponytail lite` 或 `@ponytail-review` |
| 前端设计质量 skill | `$impeccable` | UI 评审、打磨、响应式、可访问性、文案、设计系统 | `$impeccable critique/audit/polish ...` |
| 前端动效专项 skills | `emil-design-eng`、`apple-design`、`improve-animations`、`animation-vocabulary` | 动效设计、手势物理、全项目动画审计和术语反查 | 在需求中明确点名对应 skill |
| 网页视觉参考图 skill | `imagegen-frontend-web` | 生成网站、Landing Page、产品页 section 参考图 | 先生成图，再交给 Codex 实现 |
| 移动端视觉参考图 skill | `imagegen-frontend-mobile` | 生成 App 多屏、移动端流程和手机框参考图 | 先探索移动端视觉方向 |
| 品牌视觉 skill | `brandkit` | 生成品牌板、Logo 方向、色板、字体和品牌应用 | 用于产品早期定视觉气质 |

## 插件：Ponytail

### 它是什么

Ponytail 是一个给 Coding Agent 用的“lazy senior developer mode”插件。它的目标不是让代码变花哨，而是强迫 Codex 在写代码前先判断：

1. 这个功能真的需要存在吗？
2. 标准库能不能直接解决？
3. 浏览器、数据库、系统平台有没有原生能力？
4. 已安装依赖能不能解决？
5. 能不能一行解决？
6. 最后才写最小可用实现。

它适合用来防止：

- 为一个简单需求创建太多文件。
- 为一个值创建配置系统。
- 为一个实现创建 interface / factory。
- 能用浏览器原生能力，却引入组件库或新依赖。
- 为“以后可能需要”提前搭一堆结构。

它不应该删除这些东西：

- 信任边界处的输入校验。
- 防止数据丢失的错误处理。
- 安全和权限检查。
- 可访问性基础。
- 用户明确要求保留的学习解释、文档和架构边界。

### 当前安装状态

本机已经安装：

```text
ponytail@ponytail
version: 4.7.0
path: ~/.codex/plugins/cache/ponytail/ponytail/4.7.0
```

当前默认模式已经设置为关闭：

```json
{
  "defaultMode": "off"
}
```

配置文件位置：

```text
~/.config/ponytail/config.json
```

这样新会话不会自动进入 Ponytail，避免影响当前 Agent 学习项目。

### 在 Codex 中如何使用

临时开启轻量模式：

```text
@ponytail lite
```

关闭：

```text
@ponytail off
```

也可以单独发送：

```text
stop ponytail
normal mode
```

强度：

| 模式 | 命令 | 适合场景 |
| --- | --- | --- |
| `off` | `@ponytail off` | 默认状态，不影响学习和结构化开发 |
| `lite` | `@ponytail lite` | 做正常开发，但提醒有没有更简单方案 |
| `full` | `@ponytail full` | 强制按 YAGNI / stdlib / native 优先 |
| `ultra` | `@ponytail ultra` | 极端删减模式，不建议在学习项目默认使用 |

我的默认建议：

```text
日常默认 off
觉得 Codex 写复杂了再用 lite
只做复杂度审查时用 review
不要默认 ultra
```

### Ponytail 附带的 skills

| Skill | 用法 | 作用 |
| --- | --- | --- |
| `ponytail` | `@ponytail lite/full/ultra/off` | 切换最小实现模式 |
| `ponytail-review` | `@ponytail-review` | 只 review 当前 diff 里哪些复杂度能删 |
| `ponytail-audit` | `@ponytail-audit` | 扫整个仓库，找过度工程化点 |
| `ponytail-debt` | `@ponytail-debt` | 收集代码里 `ponytail:` 注释形成技术债清单 |
| `ponytail-gain` | `@ponytail-gain` | 查看 Ponytail benchmark 里的节省效果 |
| `ponytail-help` | `@ponytail-help` | 查看 Ponytail 命令速查 |

### 在当前 Agent 学习项目里的使用边界

当前项目是 Agent 应用开发学习项目，所以 Ponytail 不能覆盖这些规则：

- 写代码前需要先确认。
- 中等或复杂任务要说明计划、文件和实现思路。
- Agent 概念要适当解释，不追求极短回答。
- 前端和后端仍要保持模块职责清楚。
- Controller / Service / LLMService / ToolService 等分层不能为了“少文件”硬合并。
- 阶段性功能仍要记录 `docs/work-log.md` / `docs/learning-log.md`。

适合使用 Ponytail 的情况：

```text
@ponytail-review
看看我这次 diff 有没有过度抽象、没必要的依赖、能用原生能力替代的地方。
```

不适合这样用：

```text
@ponytail ultra
帮我实现完整 Agent tool calling 流程。
```

因为学习项目里有些结构、解释和边界本身就是学习目标。

## Skill：Impeccable

### 它是什么

Impeccable 是一个前端设计辅助 skill，用来让 Codex 更有章法地做界面评审、打磨、响应式检查、动效、文案、设计系统整理等工作。

它不是 UI 组件库，也不是替代业务开发的框架，主要适合处理前端界面质量问题。

### 在 Codex 中如何使用

在 Codex 里优先使用 `$impeccable`：

```text
$impeccable <命令> <目标范围>
```

示例：

```text
$impeccable critique 当前首页
$impeccable audit src/pages/dashboard
$impeccable polish 设置页
$impeccable harden 登录表单
```

`<目标范围>` 可以是页面名、组件名、功能描述，也可以是具体文件路径。

### 中途引入项目的推荐流程

如果项目已经开发了一部分，不要一上来让它整体重设计。推荐这样做：

1. 初始化项目设计上下文：

```text
$impeccable init
```

它会扫描现有项目，并生成或补全 `PRODUCT.md`。如果项目已有代码，建议同时生成 `DESIGN.md`，让后续任务能遵循当前项目已有的颜色、字体、组件和布局习惯。

2. 先只做评审，不改代码：

```text
$impeccable critique 当前主要页面，先只评审不要改代码
```

3. 再选择一个小范围打磨：

```text
$impeccable polish 设置页
```

这样能避免一次性把现有项目风格改乱。

### 常用命令

| 命令 | 作用 | 示例 |
| --- | --- | --- |
| `init` | 初始化项目设计上下文 | `$impeccable init` |
| `document` | 从现有代码整理视觉设计规范 | `$impeccable document` |
| `critique` | 做 UX / 视觉设计评审 | `$impeccable critique 首页` |
| `audit` | 查可访问性、响应式、性能、反模式 | `$impeccable audit src/pages/dashboard` |
| `polish` | 上线前视觉细节打磨 | `$impeccable polish 设置页` |
| `harden` | 补错误态、加载态、边界数据 | `$impeccable harden 登录表单` |
| `layout` | 专门修布局、间距、密度、对齐 | `$impeccable layout 用户列表页` |
| `typeset` | 专门修字体和文字层级 | `$impeccable typeset 文章详情页` |
| `colorize` | 改善颜色策略 | `$impeccable colorize 数据看板` |
| `adapt` | 做移动端、平板、窄屏适配 | `$impeccable adapt 用户列表页 mobile` |
| `clarify` | 改善 UX 文案 | `$impeccable clarify 支付失败弹窗` |
| `distill` | 简化复杂界面，收敛重点 | `$impeccable distill 首页` |
| `shape` | 先规划页面结构和设计方向 | `$impeccable shape 会员订阅页` |
| `craft` | 从规划到实现完整页面或功能 | `$impeccable craft 新用户 onboarding 流程` |

### 管理命令

查看或启用 hook：

```text
$impeccable hooks status
$impeccable hooks on
$impeccable hooks off
```

Hook 会在 UI 文件编辑后自动运行设计检测。Codex 里 hook 是项目级的，需要项目里有 `.codex/hooks.json`，并在 Codex 中打开 `/hooks` 批准。

中途接入项目时，建议先手动使用 `critique` / `audit` / `polish`，确认合适后再开 hook。

固定常用命令：

```text
$impeccable pin audit
```

之后可以用：

```text
$audit
```

不常用时可以不管。

## Skills：Emil Kowalski Design Engineering

### 它们是什么

这组 skills 来自 `emilkowalski/skills`，主要负责前端动效、交互手感和动画质量判断。

当前保留：

```text
emil-design-eng
apple-design
improve-animations
animation-vocabulary
```

`review-animations` 已卸载。它只负责严格审查动画 diff，和现有的 Impeccable 评审流程重合较多，当前没有保留必要。

### 与 Impeccable 的分工

```text
Impeccable：负责页面整体设计质量、布局、颜色、字体、响应式、状态和可访问性
Emil skills：负责动画为什么存在、怎么运动、手势是否跟手、缓动和时长是否合理
```

普通 UI 任务优先使用 Impeccable。只有任务明显涉及动画、拖拽、弹簧、手势或全项目动效治理时，再点名使用这组 skills。

### `emil-design-eng`

这是这组里的核心综合 skill。它负责判断动画是否应该存在，并指导缓动、时长、弹簧、组件反馈、CSS transform、clip-path、拖拽手势、性能和 reduced motion。

适合：

- 弹窗、Popover、Tooltip、Toast 等组件动效。
- 按钮按压反馈和微交互。
- 页面切换、列表进入和状态过渡。
- 调整动画的时长、缓动和 transform origin。
- 已经能运行，但整体手感不够自然的交互。

Demo：

```text
使用 emil-design-eng 优化当前商品筛选弹层的交互动效。
要求：
- 先判断哪些状态真的需要动画
- 弹层从触发按钮对应的方向出现
- 高频操作不要拖慢
- 支持 prefers-reduced-motion
- 不要改变业务逻辑和页面结构
```

### `apple-design`

这个 skill 把 Apple 的流体交互原则转换成 Web 实现方式，重点不是把页面做得“像苹果”，而是让手势和动画更符合物理直觉。

适合：

- Drawer、Sheet、Carousel 等可拖拽组件。
- Swipe to dismiss。
- 拖拽释放后的速度继承和动量。
- 可被中途打断、反向操作的动画。
- Rubber-banding、边界阻尼和回弹。
- 需要 iOS 式直接操控感的 Web 交互。

Demo：

```text
使用 apple-design 优化移动端筛选 Sheet。
要求：
- 拖动过程与手指 1:1 跟随
- 释放时根据速度判断关闭还是回到原位
- 越过边界时使用渐进阻力，不要硬停止
- 动画进行中允许再次抓取和反向拖动
```

### `animation-vocabulary`

这是一个动画术语反查表。它不负责设计或实现动画，只负责把模糊描述转换成准确术语，方便继续给 Codex、设计师或动画工具下指令。

适合：

- 不知道某个动画效果叫什么。
- 想把视觉感觉整理成准确需求。
- 区分容易混淆的动画概念。

Demo：

```text
使用 animation-vocabulary 告诉我：
“列表里的卡片不是一起出现，而是一个接一个出现”叫什么？
```

预期会得到类似：

```text
Stagger：多个项目之间带少量延迟，依次进入。
```

### `improve-animations`

这个 skill 用于扫描整个代码库里的动画和动效实现，找出高优先级问题，并生成可以交给其他 Agent 执行的独立计划。它默认先审计和写计划，不直接修改业务源码。

常用方式：

| 用法 | 作用 |
| --- | --- |
| `improve-animations` | 扫描全部交互 UI，输出完整问题表 |
| `improve-animations quick` | 只检查高频组件和高优先级问题 |
| `improve-animations deep` | 深入检查整个项目，包括营销页面和低优先级细节 |
| `improve-animations performance` | 只检查动画性能 |
| `improve-animations accessibility` | 只检查 reduced motion 等可访问性问题 |
| `improve-animations plan <描述>` | 跳过全量审计，直接为指定改进生成计划 |
| `improve-animations reconcile` | 检查已有动画计划是否完成或已经过期 |

Demo：

```text
使用 improve-animations quick 检查当前项目。
先只输出高优先级动画问题和证据，不修改源码。
```

当前边界：

- 可以使用审计、专项检查、生成计划和 reconcile。
- 暂时不要使用 `improve-animations execute <plan>`。
- `execute` 模式会依赖已经卸载的 `review-animations` 做专项复审；以后确实需要自动执行和复审时再重新安装。

## 图片生成类 Skills

下面这几个来自 `taste-skill`，和 Impeccable 的代码优化能力有重叠的已删除，只保留图片生成类。

核心用途：

```text
先生成高质量视觉参考图
再把参考图交给 Codex 实现
最后用 Impeccable 做落地打磨
```

已安装：

```text
imagegen-frontend-web
imagegen-frontend-mobile
brandkit
```

注意：这些 skill 主要生成图片，不直接写代码。适合在正式改 UI 前先探索视觉方向。

### `imagegen-frontend-web`

生成网站、Landing Page、产品页、营销页的视觉参考图。

适合：

- 官网首页
- Landing Page
- SaaS 产品介绍页
- 活动页
- 作品集网站
- 多 section 的网页视觉方向

特点：它倾向于每个 section 单独生成一张横向参考图，而不是把整页压成一张长图。

Demo：

```text
使用 imagegen-frontend-web，为一个 AI 知识库产品生成 6 个网站 section 的视觉参考图。
要求：
- 面向团队协作文档场景
- 风格克制、高级，不要紫蓝渐变和普通 SaaS 卡片堆叠
- 每个 section 单独一张横向图
- 包含 hero、痛点、核心功能、协作场景、定价引导、CTA
```

```text
使用 imagegen-frontend-web，为当前项目首页生成新的首屏视觉方向。
要求：
- 只生成 hero section 参考图
- 保留现有品牌色的大方向
- 提升层级、字体表现和空间感
- 不要直接生成代码
```

### `imagegen-frontend-mobile`

生成移动端 App 页面、流程和多屏视觉参考图。

适合：

- App 首页
- 登录/注册流程
- Onboarding
- 设置页
- 个人中心
- 移动端 dashboard
- 多屏产品流程

特点：会更关注手机屏幕里的可读性、组件密度、导航结构和多屏一致性。

Demo：

```text
使用 imagegen-frontend-mobile，为一个习惯打卡 App 生成 4 张移动端页面参考图。
页面包括：
- 首页今日任务
- 习惯详情
- 数据统计
- 空状态
要求：
- iOS 风格但不要太像系统模板
- 文本清晰可读
- 色彩温和但不要奶油风
- 每张图展示在手机框内
```

```text
使用 imagegen-frontend-mobile，为当前项目的移动端设置页生成 3 个视觉方案。
要求：
- 保持功能结构不变
- 优化信息分组、间距和触控目标
- 不要写代码，只输出参考图方向
```

### `brandkit`

生成品牌视觉方向、Logo 方向、色板、字体、品牌应用场景的参考图。

适合：

- 新产品品牌探索
- Logo 方向探索
- 品牌色和字体方向
- 品牌板 / moodboard
- 官网改版前的视觉世界观探索

特点：更像品牌设计板，不是具体页面 UI。适合先定“这个产品应该长什么气质”。

Demo：

```text
使用 brandkit，为一个面向独立开发者的 AI 工具生成品牌方向板。
要求：
- 包含 logo 方向、主色/辅助色、字体方向、网页应用片段、社媒头像应用
- 气质：专业、安静、开发者友好
- 避免紫蓝 AI 渐变、机器人图标、过度科技感
- 输出一张完整品牌板参考图
```

```text
使用 brandkit，为当前项目做品牌视觉探索。
要求：
- 先不要改代码
- 生成 2 个不同品牌方向
- 每个方向包含色彩、字体、logo 氛围和网页首屏应用感
- 方向 A 偏专业工具
- 方向 B 偏年轻消费产品
```

## 常见组合流程

### 做一个新网页视觉方向

```text
1. 使用 brandkit 定品牌气质
2. 使用 imagegen-frontend-web 生成网页 section 参考图
3. 选定方向后让 Codex 用当前技术栈实现
4. 使用 $impeccable polish 做落地细节打磨
5. 如发现实现过度复杂，再用 @ponytail-review 审查能删什么
```

### 改一个已有页面

```text
1. $impeccable critique 页面，先只评审
2. $impeccable polish 页面，做小范围打磨
3. 页面包含复杂动效时，再用 emil-design-eng 校准手感
4. 涉及拖拽、Sheet、Swipe 时，再用 apple-design
5. 必要时 $impeccable harden 补状态和边界
6. 如 Codex 加了太多抽象，用 @ponytail-review 检查
```

### 当前 Agent 学习项目

```text
默认不用 Ponytail 接管
需要 UI 质量时用 Impeccable
需要动效和手势质量时用 Emil skills
需要视觉方向时用图片生成类 skill
需要防止过度工程化时用 @ponytail-review
```

原因：Agent 学习项目里，解释、模块分层、日志记录、代码前确认都有学习价值，不能只追求“最少代码”。

## 选择速查

| 需求 | 推荐能力 |
| --- | --- |
| 想知道页面哪里不好 | `$impeccable critique` |
| 查可访问性、响应式、性能、反模式 | `$impeccable audit` |
| 页面能用但不精致 | `$impeccable polish` |
| 补错误态、加载态、长文本、边界输入 | `$impeccable harden` |
| 布局间距不舒服 | `$impeccable layout` |
| 字体层级不舒服 | `$impeccable typeset` |
| 颜色太灰、太平、没有重点 | `$impeccable colorize` |
| 移动端适配不好 | `$impeccable adapt` |
| 文案不清楚 | `$impeccable clarify` |
| 界面太复杂 | `$impeccable distill` |
| 新功能先规划 | `$impeccable shape` |
| 新页面从 0 到 1 | `$impeccable craft` |
| 优化具体组件的动画手感 | `emil-design-eng` |
| 优化拖拽、Sheet、Swipe 和弹簧交互 | `apple-design` |
| 不知道某个动画效果叫什么 | `animation-vocabulary` |
| 扫描整个项目的动画问题 | `improve-animations quick/deep` |
| 网站视觉方向探索 | `imagegen-frontend-web` |
| 移动端视觉方向探索 | `imagegen-frontend-mobile` |
| 品牌视觉方向探索 | `brandkit` |
| 检查当前 diff 有没有过度工程化 | `@ponytail-review` |
| 扫整个仓库有哪些复杂度能删 | `@ponytail-audit` |
| 临时让 Codex 少写代码 | `@ponytail lite` |
| 关闭 Ponytail | `@ponytail off` |

## 使用边界

- 设计、UI、视觉质量：优先 Impeccable。
- 动画细节和组件手感：优先 `emil-design-eng`。
- 拖拽、动量和可打断手势：优先 `apple-design`。
- 全项目动画审计：优先 `improve-animations`，当前不使用其 `execute` 模式。
- 视觉方向探索：优先图片生成类 skills。
- 复杂度收敛：优先 Ponytail review/audit，不要直接默认 ultra。
- 业务代码、Agent 学习、后端分层：优先项目 `AGENTS.md` 和本地开发约束。
- 不要让任何 skill 覆盖明确的项目规则；skill 是辅助工具，不是最高优先级规则。
