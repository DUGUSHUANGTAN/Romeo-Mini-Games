# 🎮 Romeo's Mini Games

4 款经典小游戏合集，纯 HTML 单文件实现 — 零依赖，双击即玩。

## 🚀 快速开始

双击 `index.html` 打开主界面，选择一个游戏即可开始。

## 🕹️ 游戏

### 🧱 Tetris — 俄罗斯方块
`games/tetris.html`

- **Ghost piece** — 半透明投影显示落点位置
- **Next preview** — 预览下一个方块
- **粒子消除效果** — 消除行时方块炸裂为彩色粒子，带延迟动画
- **8 种色彩** — 灵感来自 Apple 照片应用配色
- **音效反馈** — 通过 Web Audio API 生成，无外部音频文件
- **键盘/触屏** — 键盘方向键操作，移动端支持触摸按钮

### ⚫ Gomoku — 五子棋
`games/gomoku.html`

- **双模式** — Player vs AI / Player vs Player
- **AI 对手** — 基于位置评分的启发式 AI，同时评估进攻与防守
- **三个难度级别** — 通过 AI 搜索深度调节
- **Canvas 绘制** — 棋盘、棋子、胜负提示

### 💣 Minesweeper — 扫雷
`games/minesweeper.html`

- **三种难度** — 初级 (9×9/10雷)、中级 (16×16/40雷)、高级 (16×30/99雷)
- **自动标雷** — 可选开关，自动标记已确定的雷
- **数字快捷展开** — 点击已揭开的数字自动翻开周围安全区域
- **首次点击必安全** — 经典规则
- **Flood Fill 展开** — 揭开空白区域时递归展开

### 🎖️ Tank Battle — 坦克大战
`games/tanks.html`

- **双模式** — Player vs AI / Player vs Player（同屏双人）
- **AI 对手** — 基于向量计算的瞄准算法，计算弹道预判
- **5 张开放地图** — 每局随机选用
- **小地图** — 画面角落显示战场全览
- **HUD** — 分数、射击次数统计
- **爆炸动画** — 粒子爆炸特效

## ✨ 通用功能

- **深色/浅色主题** — 右上角一键切换，localStorage 持久化
- **中英文双语** — 内置 i18n 系统，所有 UI 文本随切换即时更新
- **双向同步** — 主界面与游戏内的主题/语言切换通过 postMessage 实时同步
- **零依赖** — 每个游戏都是单个 HTML 文件，原生 Canvas 2D 渲染

## 📁 项目结构

```
Game 1/
├── index.html    # 主界面入口
├── games/                     # 游戏文件
│   ├── tetris.html
│   ├── gomoku.html
│   ├── minesweeper.html
│   └── tanks.html
├── originals/                 # 原版备份
├── assets/                    # 图标资源 (png)
├── docs/                      # 文档
├── README.md
└── PROGRESS.md
```

## 🛠️ 技术栈

HTML5 Canvas · CSS Variables · Vanilla JS (ES6+) · Web Audio API · localStorage · postMessage

---

*Made with ❤️ by Romeo*
