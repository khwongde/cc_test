# PROJECT_CONTEXT.md

> 架构决策记录（ADR）与核心设计说明。
> 记录"为什么这么做"，而非"做了什么"（后者见 README.md）。

---

## 1. 架构概览

```
单文件 HTML（零依赖、零构建）
├── CSS Variables 主题层    ← 9 个设计 Token，统一管控颜色
├── BEM-lite 命名          ← 语义化 class，避免全局冲突
├── 事件驱动状态管理        ← 无框架，通过 DOM + 闭包变量管理状态
└── 渐进增强的交互           ← 核心功能无 JS 也能看 UI，JS 增强交互
```

**原则：每一行代码都有存在理由。无冗余抽象。**

---

## 2. 决策记录

### ADR-1: 单文件架构

**决策：** 整个应用打包为一个 `.html` 文件，CSS 和 JS 内联。

**理由：**
- 目标用户直接双击打开，不应要求安装 Node.js、npm、或启动本地服务器
- 避免 HTTP 请求开销，首屏即渲染
- 复制一个文件即可分享，降低分发摩擦
- 工具性质决定规模不大（~440 行），拆分为多个文件属于过度设计

**代价：** 超过 1000 行后将难以维护。届时应拆分为 `index.html` + `style.css` + `app.js`，并引入构建步骤。

---

### ADR-2: 零依赖（No Framework / No Library）

**决策：** 不使用任何第三方库或框架（React、Vue、jQuery、Lodash 等）。

**理由：**
- 交互路径简单：读取输入 → Math.random() → 更新 DOM → 写入历史数组
- 引入框架的 gzip 体积（React ~42KB）远超应用代码本身
- 框架版本升级 = 维护负担，原生 JS 向后兼容性极好
- 学习价值：让初学者能看懂每一行逻辑，不被框架黑盒遮挡

**代价：** 如果未来需求变为复杂状态同步（多组件共享实时状态）、路由、SSR，则需要引入框架。

---

### ADR-3: CSS Variables 主题系统

**决策：** 使用 9 个 CSS 自定义属性（`--bg`、`--card`、`--accent` 等）作为唯一颜色来源。

**理由：**
- 暗色主题一键切换：只需修改变量值，无需逐条覆写
- 无预处理器依赖（不需要 Sass/Less），浏览器原生支持
- 设计 Token 语义化 — `--accent` 比 `#6366f1` 更可读
- Tailwind 风格的 slate 色系（`#0f172a`、`#1e293b`、`#334155`），视觉专业

**代价：** IE11 不支持 CSS Variables。目标用户使用现代浏览器（Chrome/Firefox/Safari/Edge），此代价可接受。

---

### ADR-4: 状态管理策略

**决策：** 状态通过闭包变量（`let history = []`、`let lastResult = null`）+ DOM 作为单一真实来源。

**状态结构：**

```
history: number[]       ← 最近 12 条记录，头部插入，尾部淘汰
lastResult: number|null ← 当前显示值，用于复制功能
DOM:                    ← min/max 输入通过 .value 直读，无独立 state
```

**理由：**
- 对于此规模的应用，引入 Redux/Zustand/Signal 是过度抽象
- 状态消费者只有 4 个 DOM 节点（结果显示区、复制按钮、历史列表、空状态提示），直接命令式更新成本最低
- `lastResult` 独立于 `history`：清空历史不影响复制功能

**代价：** 如果交互路径增长到 10+ 条，命令式 DOM 更新将难以追踪。届时应引入数据绑定层。

---

### ADR-5: 历史记录上限 12 条

**决策：** 历史记录最多保留 12 条，超出后自动淘汰最早记录。

**理由：**
- 视觉上，12 个 pill 标签在一行中刚好不拥挤（flex-wrap 布局下约 2-3 行）
- 认知负荷：用户不会从 50 条历史中寻找目标，12 条足够回溯
- 内存占用可忽略，但 DOM 节点数受限，避免 layout thrashing

**代价：** 用户可能期望无限历史。可通过 localStorage 持久化并按需分页展示。

---

### ADR-6: 复制按钮的"条件可见"策略

**决策：** 复制按钮默认 `opacity: 0; pointer-events: none`，生成数字后才通过 `.active` class 显示。

**理由：**
- 未生成时无内容可复制，显示按钮是无效 UI
- 使用 `opacity` + `pointer-events` 而非 `display: none` 避免 layout shift
- 按钮始终占位（`position: absolute`），不会在出现时把结果文字挤开

---

### ADR-7: 键盘快捷键设计

**决策：** `Enter` 和 `Space` 均可触发生成。输入框内仅 `Enter` 触发。

**理由：**
- `Space` 是全局最易触达的键，符合"快速连续生成"的使用场景
- 输入框中 `Space` 不应拦截（用户可能在输入数字时无意触发），仅 `Enter` 在输入框中触发
- `preventDefault()` 防止 Space 导致页面滚动

---

### ADR-8: 边界保护

**决策：** 对非法输入静默纠正而非报错弹窗。

**场景与处理：**

| 场景 | 处理 |
|------|------|
| 输入为空 / NaN | min 回退为 0，max 回退为 100 |
| min > max | 自动交换两值，并写回输入框 |
| 复制失败（非 HTTPS / 旧浏览器） | 显示"失败"1.5 秒后恢复，不中断操作 |
| 点击历史记录 | 仅回显到结果区，不触发新增历史 |

**理由：** 弹窗打断用户工作流。随机数生成器是"轻松工具"，容错设计应让用户无感恢复。

---

## 3. CSS 设计系统

### 间距体系

- 组件内间距：8px 基准递增（6, 8, 10, 12, 14, 16, 20, 24, 28, 32, 36, 48）
- 设计意图：避免 `rem` 依赖根字体大小，px 直接对应视觉稿

### 圆角层级

- 小元素（按钮、输入框、标签）：10–14px（现代圆角风格）
- 容器卡片：20px（与内容区分）

### 动效原则

- 交互动效 ≤ 200ms（hover 0.15s、active 0.2s），无感知延迟
- 结果弹跳动画 150ms（`scale(1.15) → scale(1)`），快速反馈
- 历史项淡入 300ms（`fadeIn`），比按钮动效慢一倍以形成节奏层次
- 无 `ease-in-out` 滥用：弹跳用默认 ease，交互用隐式 ease（transition 默认值）

### 字体策略

- 系统字体栈：`-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif`
- 无外部字体加载，零 FOUT（Flash of Unstyled Text）
- macOS 渲染 San Francisco，Windows 渲染 Segoe UI，Android 渲染 Roboto

---

## 4. JavaScript 数据流

```
用户输入(min, max) ──→ parseInt() 清洗
                          │
                    NaN / 越界检查
                          │
                    Math.random() 生成
                          │
                    ┌─────┴─────┐
                    ▼           ▼
              DOM 更新      历史数组
           (resultDisplay   (history.unshift
            + 动画)          + renderHistory)
                    │
                    ▼
              lastResult 赋值
              （供复制按钮使用）
```

**关键：** 生成 → 显示 → 入历史的顺序不可变。先生成再入历史保证"当前结果"总在历史第一位。

---

## 5. 迁移路径

当前架构的预设演进方向：

| 当前 | 触发条件 | 迁移目标 |
|------|----------|----------|
| 单文件 HTML | 超过 1000 行 | `index.html` + `style.css` + `app.js` |
| CSS Variables | 需亮色模式 | 拆分 `[data-theme="light"]` 变量块 |
| 内存历史 | 需跨会话持久化 | `localStorage` + 历史分页 |
| 原生 DOM 操作 | 组件 > 5 个 | Alpine.js 或 Petite-Vue（保持零构建） |
| 静态托管 | 需后端逻辑 | Express/Python HTTP server |

**底线：** 任何迁移不改变"双击即可使用"的核心体验。
