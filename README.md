# 🎲 随机数字生成器

一个简洁、交互流畅的随机数字生成器，单文件，浏览器即开即用。

## 快速开始

打开 `random-number-generator.html` 即可使用：

```bash
open random-number-generator.html
```

或者用任意浏览器打开该文件。

## 功能

- **自定义范围** — 自由设置最小值和最大值（默认 1–100）
- **一键生成** — 点击按钮或按 `Enter` / `Space` 快速生成
- **生成动画** — 结果出现时带缩放弹跳效果
- **一键复制** — 生成后右上角显示复制按钮，自动使用 Clipboard API
- **历史记录** — 保留最近 12 条记录，点击可回溯
- **自动纠错** — 最小值 > 最大值时自动互换
- **输入保护** — 处理空值、NaN 等异常输入
- **键盘友好** — 输入框中 `Enter` 也可触发；非输入框中 `Space` / `Enter` 均可生成

## 技术栈

| 层级 | 技术 |
|------|------|
| 结构 | HTML5 |
| 样式 | 纯 CSS（CSS Variables 主题系统） |
| 逻辑 | 原生 JavaScript（ES6+，零依赖） |
| 部署 | 单文件，无构建步骤，可直接部署到任意静态托管服务 |

## 文件结构

```
cc_test/
├── .gitignore
├── random-number-generator.html   ← 主程序（单文件应用）
├── README.md                      ← 项目说明（本文件）
└── PROJECT_CONTEXT.md             ← 架构决策记录
```

## 许可证

MIT
