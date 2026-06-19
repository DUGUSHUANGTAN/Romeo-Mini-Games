# 游戏项目 — 开发经验文档

> 最后更新：2026-06-19  
> 项目路径：`/Users/romeoke/Documents/Game 1/`

---

## 一、项目概览

### 文件清单

| 文件 | 行数 | 游戏类型 |
|------|------|----------|
| `tetris.html` (899) | 俄罗斯方块（网格消除） |
| `gomoku.html` (827) | 五子棋（AI 评分算法） |
| `minesweeper.html` (454) | 扫雷（flood fill 展开） |
| `tanks.html` (1088) | 坦克大战（BFS 寻路 + 炮弹预判） |

### 架构共性

所有游戏均为 **单文件 HTML**，零外部依赖：
- Canvas 2D 渲染
- CSS 变量实现深色/浅色双主题
- `data-lang-key` 属性驱动的中英文 i18n
- `localStorage` 持久化用户偏好（主题 + 语言）

---

## 二、i18n 国际化模式

### 标准实现结构

```javascript
// 字典对象，key 与 HTML 中 data-lang-key 一一对应
const LANG = {
    en: { title: 'Tetris', gameOver: 'Game Over', ... },
    zh: { title: '俄罗斯方块', gameOver: '游戏结束', ... }
};

function applyLang(lang) {
    currentLang = lang;
    const t = LANG[lang];
    document.querySelectorAll('[data-lang-key]').forEach(el => {
        el.textContent = t[el.dataset.langKey] || '';
    });
}
```

### ⚠️ 已知 Bug：footer 元素在 `</script>` 之后

**现象：** 刷新页面后，`applyLang()` 执行时 footer div 尚未被浏览器解析，导致英文模式下水印仍显示中文。

**根因：** footer `<div data-lang-key="footer">` 写在 `</script>` 标签之后，而 `applyLang()` 在脚本中同步调用，此时 DOM 树还未构建到该元素。

**修复方案（已应用）：**
```javascript
// 用 setTimeout 延迟到当前任务完成后执行
setTimeout(() => {
    const footerEl = document.querySelector('[data-lang-key="footer"]');
    if (footerEl && t.footer) {
        footerEl.textContent = t.footer;
    }
}, 0);
```

**替代方案（未采用）：** 将 footer div 移到 `<script>` 之前，或监听 `DOMContentLoaded`。

### 语言切换流程

1. 用户点击语言按钮 → `toggleLang()` → 切换 `currentLang`
2. 写入 `localStorage.setItem('game-lang', newLang)`
3. 调用 `applyLang(newLang)` 更新所有 `data-lang-key` 元素
4. 页面刷新时从 localStorage 读取并恢复

### 各游戏 i18n 差异

| 游戏 | 字典变量名 | 特殊点 |
|------|-----------|--------|
| tetris.html | `LANG` | footer 有 setTimeout 延迟修复 |
| gomoku.html | `LANG` | 标准实现，无 bug |
| minesweeper.html | `i18n` | 变量名不同但逻辑一致 |
| tanks.html | `LANG` | 标准实现，无 bug |

---

## 三、主题系统（CSS Variables）

### 核心模式

```css
:root {
    --bg: #0d1117; --text: #c9d1d9; /* 深色主题默认值 */
}
[data-theme="light"] {
    --bg: #f6f8fa; --text: #1f2328; /* 浅色主题覆盖 */
}
```

### 关键设计决策

- **CSS 变量命名：** 语义化（`--cell-number-1` ~ `--cell-number-8`），而非颜色值
- **backdrop-filter：** 所有面板使用 `backdrop-filter: blur(24px) saturate(170%)` 实现毛玻璃效果
- **主题切换按钮位置：** 固定在右上角（`.top-bar`）
- **localStorage key：** `'game-theme'`，默认 dark

### ⚠️ 已知问题：CSS 变量重复定义

**现象：** `tetris.html` 中 `--cell-number-*` 等变量在 `:root` 和 `[data-theme="light"]` 中有冗余定义。

**修复建议：** 只保留必要的覆盖，减少 CSS 体积。

---

## 四、Canvas 渲染模式

### 四种游戏的渲染策略对比

| 游戏 | 渲染方式 | 帧率控制 |
|------|---------|---------|
| tetris.html | `requestAnimationFrame` + 时间戳节流（16ms） | 固定 ~60fps，但逻辑更新受下落速度控制 |
| gomoku.html | 事件驱动（落子后重绘） | 无循环，按需渲染 |
| minesweeper.html | 事件驱动（点击/右键后重绘） | 无循环，按需渲染 |
| tanks.html | `requestAnimationFrame` 持续循环 | 固定 ~60fps |

### Tetris 渲染细节

```javascript
// 游戏循环：时间戳节流
function gameLoop(ts) {
    if (ts - lastFrame > 16) { lastFrame = ts; draw(); }
    animationId = requestAnimationFrame(gameLoop);
}
```

- **Ghost piece：** 半透明预览落点位置（`ctx.globalAlpha`）
- **粒子系统：** 消行时生成爆炸粒子效果，独立于主渲染循环
- **音效：** Web Audio API `OscillatorNode` + `GainNode`，复用同一个 AudioContext

### Tanks 渲染细节

```javascript
// 坦克绘制：坐标系变换实现旋转
ctx.save();
ctx.translate(this.cx, this.cy);
ctx.rotate(this.dir);
// ... 绘制坦克主体和炮管
ctx.restore();
```

- **炮弹反弹：** 射线与线段求交，反射向量计算
- **碰撞检测：** BFS 网格寻路 + 矩形-线段相交测试

---

## 五、AI 算法经验

### Gomoku（五子棋）AI — 评分启发式

**核心思路：** 对每个空位评估进攻分和防守分，取最大值。

```javascript
function evaluateAll() {
    for each empty cell:
        attackScore = evaluatePoint(x, y, AI_COLOR)   // AI 在此落子的价值
        defenseScore = evaluatePoint(x, y, PLAYER_COLOR) // 玩家在此落子的威胁
        score = max(attackScore, defenseScore)
}
```

**评分模式：**
- 连五：100000（必胜）
- 活四：10000（必杀）
- 冲四 + 活三：5000（高威胁）
- 活三：1000
- 活二：100

### Tanks AI — BFS 寻路 + 炮弹预判

**移动策略：**
1. BFS 网格寻路到目标位置
2. 检测 incoming bullet，计算闪避方向
3. 态势评估（敌人在瞄准/有炮弹飞来/太近）决定进攻或撤退

**炮弹预判：**
```javascript
// 沿子弹当前方向模拟飞行，最多 600px 或 5 次反弹
for (let step = 0; step < 120; step++) {
    simX += simVx * dt; simY += simVy * dt;
    // 检测与墙壁的交点 → 反射向量
    // 检测模拟点是否离 AI 很近 → 闪避
}
```

---

## 六、扫雷算法经验

### Flood Fill 展开（零数字格自动展开）

```javascript
function revealCell(row, col) {
    if (flagged[row][col] || revealed[row][col]) return false;
    revealed[row][col] = true;
    
    if (board[row][col] === 0) {
        // 零数字格：递归展开相邻格子
        for each neighbor:
            revealCell(nr, nc);
    }
}
```

### Chord（双击数字自动展开）

```javascript
function chordReveal(row, col) {
    const neighbors = getNeighbors(row, col);
    const flaggedCount = neighbors.filter(n => flagged[n]).length;
    
    if (flaggedCount === board[row][col]) {
        // 标雷数等于数字 → 自动展开未标记的邻居
        for each unrevealed neighbor:
            revealCell(nr, nc);
    }
}
```

### 首击安全保证

- 第一次点击后再生成地雷，确保首击位置及其周围无雷
- 如果首击恰好落在预生成的雷上，交换该格与一个非关键格

---

## 七、localStorage 偏好存储

### 统一模式（所有游戏）

```javascript
// 保存
try { localStorage.setItem('game-lang', lang); } catch(e) {}
try { localStorage.setItem('game-theme', theme); } catch(e) {}

// 读取
const savedLang = localStorage.getItem('game-lang');
if (savedLang && (savedLang === 'en' || savedLang === 'zh')) {
    applyLang(savedLang);
} else {
    applyLang('en'); // 默认英文
}

// 清除（URL 参数 ?clear=true）
if (new URLSearchParams(window.location.search).has('clear')) {
    localStorage.removeItem('game-theme');
    localStorage.removeItem('game-lang');
}
```

### ⚠️ 注意事项

- `localStorage` 操作始终包裹在 try-catch 中（防止私有模式报错）
- 语言默认值统一为 `'en'`，除非用户手动切换过
- 清除功能通过 URL 参数触发，方便调试

---

## 八、已知 Bug 与修复记录

| # | 游戏 | 问题 | 状态 |
|---|------|------|------|
| 1 | tetris.html | footer 在 `</script>` 后导致 i18n 不生效 | ✅ 已修复（setTimeout） |
| 2 | tetris.html | CSS 变量重复定义 | ⚠️ 待优化 |
| 3 | minesweeper.html | 游戏结束后计时器未停止 | ✅ 已修复 |
| 4 | minesweeper.html | 默认语言为中文而非英文 | ✅ 已修复 |
| 5 | tetris.html | theme icon 逻辑错误（初始显示太阳） | ✅ 已修复 |

---

## 九、代码风格约定

### 命名规范

- **变量：** `camelCase`（`gameMode`, `currentLang`, `board`）
- **常量：** `UPPER_SNAKE_CASE`（`BLOCK_SIZE`, `GRID_WIDTH`）
- **函数：** `camelCase`，动词开头（`draw()`, `update()`, `revealCell()`）
- **CSS 变量：** `kebab-case`（`--cell-number-1`, `--panel-bg`）

### 文件结构顺序

```html
<!DOCTYPE html>
<html lang="en" data-theme="dark">
<head>
    <style> /* CSS Variables + 布局 */ </style>
</head>
<body>
    <!-- HTML UI -->
    <script>
        // 1. LANG / i18n 字典
        // 2. 游戏状态变量
        // 3. 初始化函数
        // 4. 渲染/更新函数
        // 5. 事件监听
        // 6. localStorage 恢复
    </script>
</body>
</html>
```

### Canvas 绘制约定

- 所有游戏使用 `ctx.save()` / `ctx.restore()` 保护状态
- 颜色通过 CSS 变量获取：`cs.getPropertyValue('--xxx').trim()`
- 文字居中：`textAlign = 'center'; textBaseline = 'middle'`

---

## 十、性能注意事项

### Tetris

- 粒子系统需要清理死亡粒子，避免内存泄漏
- `requestAnimationFrame` 节流到 ~60fps，但逻辑更新频率由下落速度决定

### Tanks

- BFS 寻路每帧执行，网格大小影响性能（当前 800x600 / cellSize = 20px → 40x30 网格）
- 炮弹预判模拟最多 120 步 × N 发子弹，高弹量时可能卡顿

### Minesweeper

- flood fill 递归深度最大为 `rows * cols`，大棋盘建议改用迭代实现
- chord 展开可能触发深层递归，注意栈溢出风险

---

## 十一、后续优化方向

1. **统一 i18n 基类：** 抽取公共的 `applyLang()` 到共享模块，消除 footer setTimeout hack
2. **CSS 变量精简：** 合并重复定义，提取公共样式到 `<link>` 或共享 CSS
3. **移动端适配：** 当前所有游戏为桌面端设计，需添加触摸控制和响应式布局
4. **音效系统统一：** 各游戏独立实现 Web Audio API，可抽取公共 `playSound()` 函数
5. **难度配置外部化：** 将地雷数、棋盘大小等硬编码参数提取到配置文件
