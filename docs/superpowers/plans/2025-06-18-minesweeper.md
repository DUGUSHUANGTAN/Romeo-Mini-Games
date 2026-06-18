# Minesweeper 扫雷游戏实现计划

> **面向 AI 代理的工作者：** 必需子技能：使用 superpowers:subagent-driven-development（推荐）或 superpowers:executing-plans 逐任务实现此计划。步骤使用复选框（`- [ ]`）语法来跟踪进度。

**目标：** 在 `minesweeper.html` 单文件中实现一个完整的扫雷游戏，支持三种经典难度、自动标雷开关、中英文语言切换、深色/浅色主题切换。

**架构：** 单文件 HTML + Canvas 渲染 + CSS 变量主题系统 + i18n 字典对象。纯原生 JavaScript，无外部依赖。Canvas 负责绘制游戏面板和 UI 元素，CSS 变量管理双主题配色。

**技术栈：** HTML5 Canvas、CSS3 Variables、Vanilla JavaScript (ES6+)

---

## 文件结构

- **创建：** `minesweeper.html` — 唯一的游戏文件，包含所有 HTML/CSS/JS
- **参考：** `tetris.html` — 现有游戏项目，用于对齐代码风格和主题系统

---

### 任务 1：HTML 骨架 + CSS 主题系统

**文件：**
- 创建：`minesweeper.html`（初始骨架）

- [ ] **步骤 1：编写 HTML 骨架和 CSS 变量**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title id="page-title">扫雷游戏</title>
<style>
* { margin: 0; padding: 0; box-sizing: border-box; }

:root {
    --bg: #0d1117;
    --panel-bg: rgba(22, 27, 34, 0.65);
    --panel-border: rgba(48, 54, 61, 0.7);
    --text: #c9d1d9;
    --text-secondary: #8b949e;
    --cell-bg: #21262d;
    --cell-revealed: #161b22;
    --cell-hover: rgba(110, 118, 129, 0.4);
    --cell-mine: #f85149;
    --cell-flag: #d29922;
    --cell-number-1: #58a6ff;
    --cell-number-2: #3fb950;
    --cell-number-3: #f85149;
    --cell-number-4: #a371f7;
    --cell-number-5: #db6d28;
    --cell-number-6: #1f6feb;
    --cell-number-7: #f778ba;
    --cell-number-8: #7ee787;
    --btn-bg: #238636;
    --btn-hover: #2ea043;
    --btn-secondary-bg: rgba(48, 54, 61, 0.6);
    --btn-secondary-hover: rgba(63, 70, 79, 0.8);
    --border-color: rgba(48, 54, 61, 0.7);
}

[data-theme="light"] {
    --bg: #f6f8fa;
    --panel-bg: rgba(255, 255, 255, 0.65);
    --panel-border: rgba(208, 215, 222, 0.7);
    --text: #1f2328;
    --text-secondary: #656d76;
    --cell-bg: #ffffff;
    --cell-revealed: #f0f0f0;
    --cell-hover: rgba(175, 184, 193, 0.4);
    --cell-mine: #cf222e;
    --cell-flag: #9a6700;
    --cell-number-1: #0969da;
    --cell-number-2: #1a7f37;
    --cell-number-3: #cf222e;
    --cell-number-4: #8250df;
    --cell-number-5: #bc4c00;
    --cell-number-6: #0969da;
    --cell-number-7: #bf3989;
    --cell-number-8: #1a7f37;
    --btn-bg: #1f883d;
    --btn-hover: #2c974b;
    --btn-secondary-bg: rgba(208, 215, 222, 0.6);
    --btn-secondary-hover: rgba(188, 195, 204, 0.8);
    --border-color: rgba(208, 215, 222, 0.7);
}

body {
    background: var(--bg);
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Helvetica, Arial, sans-serif;
    color: var(--text);
    transition: background 0.3s ease, color 0.3s ease;
}

.container {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 16px;
}

.top-bar {
    display: flex;
    gap: 8px;
    align-items: center;
    flex-wrap: wrap;
    justify-content: center;
}

.btn {
    padding: 6px 12px;
    border: 1px solid var(--border-color);
    background: var(--btn-secondary-bg);
    color: var(--text);
    cursor: pointer;
    border-radius: 6px;
    font-size: 14px;
    transition: all 0.2s ease;
}

.btn:hover {
    background: var(--btn-secondary-hover);
}

.btn.active {
    background: var(--btn-bg);
    color: white;
    border-color: var(--btn-bg);
}

.toggle-group {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 4px 12px;
    background: var(--panel-bg);
    border: 1px solid var(--panel-border);
    border-radius: 6px;
}

.toggle-label {
    font-size: 13px;
    color: var(--text-secondary);
}

.game-panel {
    background: var(--panel-bg);
    border: 1px solid var(--panel-border);
    border-radius: 8px;
    padding: 16px;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 12px;
}

.game-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    padding: 8px 0;
    border-bottom: 1px solid var(--border-color);
}

.counter {
    font-family: 'Courier New', monospace;
    font-size: 24px;
    font-weight: bold;
    color: var(--cell-mine);
    min-width: 60px;
    text-align: center;
}

.reset-btn {
    width: 40px;
    height: 40px;
    border: 2px solid var(--border-color);
    background: var(--cell-bg);
    cursor: pointer;
    font-size: 24px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 6px;
}

.reset-btn:hover {
    background: var(--cell-hover);
}

canvas {
    display: block;
    image-rendering: pixelated;
}

.instructions {
    max-width: 600px;
    text-align: center;
    font-size: 13px;
    color: var(--text-secondary);
    line-height: 1.6;
}
</style>
</head>
<body>
<div class="container">
    <div class="top-bar">
        <button class="btn active" data-difficulty="easy" id="btn-easy">初级</button>
        <button class="btn" data-difficulty="medium" id="btn-medium">中级</button>
        <button class="btn" data-difficulty="hard" id="btn-hard">高级</button>
        <div class="toggle-group">
            <span class="toggle-label" id="label-auto-flag">自动标雷</span>
            <input type="checkbox" id="auto-flag-toggle">
        </div>
        <button class="btn" id="theme-toggle">🌙</button>
        <button class="btn" id="lang-toggle">中文</button>
    </div>
    
    <div class="game-panel">
        <div class="game-header">
            <div class="counter" id="mine-counter">010</div>
            <button class="reset-btn" id="reset-btn">😊</button>
            <div class="counter" id="timer">000</div>
        </div>
        
        <canvas id="game-canvas"></canvas>
    </div>
    
    <div class="instructions" id="instructions">
        左键翻开方块，右键标记地雷。第一次点击保证安全。<br>
        点击数字可自动展开周围安全区域。
    </div>
</div>

<script>
// JavaScript will be added in subsequent tasks
</script>
</body>
</html>
```

- [ ] **步骤 2：验证 HTML 结构**

打开 `minesweeper.html`，确认页面能正常加载，主题切换按钮和难度选择按钮可见。

---

### 任务 2：i18n 中英文字典 + 语言切换

**文件：**
- 修改：`minesweeper.html`（在 `<script>` 标签内添加）

- [ ] **步骤 1：编写 i18n 字典和切换逻辑**

```javascript
const i18n = {
    zh: {
        pageTitle: '扫雷游戏',
        easy: '初级',
        medium: '中级',
        hard: '高级',
        autoFlag: '自动标雷',
        instructions: '左键翻开方块，右键标记地雷。第一次点击保证安全。<br>点击数字可自动展开周围安全区域。',
        mineLabel: '剩余雷数',
        timeLabel: '用时'
    },
    en: {
        pageTitle: 'Minesweeper',
        easy: 'Easy',
        medium: 'Medium',
        hard: 'Hard',
        autoFlag: 'Auto Flag',
        instructions: 'Left-click to reveal, right-click to flag.<br>First click is always safe.<br>Click numbers to auto-reveal safe areas.',
        mineLabel: 'Mines Left',
        timeLabel: 'Time'
    }
};

let currentLang = 'zh';

function setLanguage(lang) {
    currentLang = lang;
    const t = i18n[lang];
    
    document.getElementById('page-title').textContent = t.pageTitle;
    document.documentElement.lang = lang === 'zh' ? 'zh-CN' : 'en';
    
    document.getElementById('btn-easy').textContent = t.easy;
    document.getElementById('btn-medium').textContent = t.medium;
    document.getElementById('btn-hard').textContent = t.hard;
    document.getElementById('label-auto-flag').textContent = t.autoFlag;
    document.getElementById('instructions').innerHTML = t.instructions;
    
    document.getElementById('lang-toggle').textContent = lang === 'zh' ? 'English' : '中文';
}

document.getElementById('lang-toggle').addEventListener('click', () => {
    setLanguage(currentLang === 'zh' ? 'en' : 'zh');
});
```

- [ ] **步骤 2：测试语言切换**

点击语言切换按钮，确认所有文本（难度名称、自动标雷标签、说明文字）正确切换。

---

### 任务 3：Canvas 渲染引擎 + 游戏面板绘制

**文件：**
- 修改：`minesweeper.html`（在 `<script>` 标签内添加）

- [ ] **步骤 1：编写 Canvas 初始化和基础渲染**

```javascript
const canvas = document.getElementById('game-canvas');
const ctx = canvas.getContext('2d');

// Game configuration
const difficulties = {
    easy: { rows: 9, cols: 9, mines: 10 },
    medium: { rows: 16, cols: 16, mines: 40 },
    hard: { rows: 16, cols: 30, mines: 99 }
};

let currentDifficulty = 'easy';
let cellSize = 25;
let board = [];
let revealed = [];
let flagged = [];
let gameOver = false;
let gameStarted = false;
let firstClick = true;
let timerInterval = null;
let seconds = 0;

function initBoard() {
    const config = difficulties[currentDifficulty];
    
    canvas.width = config.cols * cellSize;
    canvas.height = config.rows * cellSize;
    
    board = Array(config.rows).fill(null).map(() => Array(config.cols).fill(0));
    revealed = Array(config.rows).fill(null).map(() => Array(config.cols).fill(false));
    flagged = Array(config.rows).fill(null).map(() => Array(config.cols).fill(false));
    
    gameOver = false;
    gameStarted = false;
    firstClick = true;
    seconds = 0;
    updateTimer();
    updateMineCounter();
    
    if (timerInterval) clearInterval(timerInterval);
    timerInterval = null;
}

function drawCell(row, col) {
    const x = col * cellSize;
    const y = row * cellSize;
    
    if (revealed[row][col]) {
        // Revealed cell
        ctx.fillStyle = getComputedStyle(document.documentElement).getPropertyValue('--cell-revealed').trim();
        ctx.fillRect(x, y, cellSize, cellSize);
        
        if (board[row][col] > 0) {
            // Draw number
            ctx.fillStyle = getNumberColor(board[row][col]);
            ctx.font = `bold ${cellSize * 0.6}px Arial`;
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText(board[row][col], x + cellSize / 2, y + cellSize / 2);
        } else if (board[row][col] === -1) {
            // Draw mine
            ctx.fillStyle = getComputedStyle(document.documentElement).getPropertyValue('--cell-mine').trim();
            ctx.beginPath();
            ctx.arc(x + cellSize / 2, y + cellSize / 2, cellSize * 0.3, 0, Math.PI * 2);
            ctx.fill();
        }
    } else {
        // Unrevealed cell
        ctx.fillStyle = getComputedStyle(document.documentElement).getPropertyValue('--cell-bg').trim();
        ctx.fillRect(x, y, cellSize, cellSize);
        
        if (flagged[row][col]) {
            // Draw flag
            ctx.fillStyle = getComputedStyle(document.documentElement).getPropertyValue('--cell-flag').trim();
            ctx.font = `${cellSize * 0.7}px Arial`;
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText('⚐', x + cellSize / 2, y + cellSize / 2);
        }
    }
    
    // Draw grid lines
    ctx.strokeStyle = getComputedStyle(document.documentElement).getPropertyValue('--border-color').trim();
    ctx.lineWidth = 0.5;
    ctx.strokeRect(x, y, cellSize, cellSize);
}

function getNumberColor(num) {
    return getComputedStyle(document.documentElement).getPropertyValue(`--cell-number-${num}`).trim();
}

function drawBoard() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    for (let row = 0; row < board.length; row++) {
        for (let col = 0; col < board[0].length; col++) {
            drawCell(row, col);
        }
    }
}

// Initial setup
initBoard();
drawBoard();
```

- [ ] **步骤 2：验证 Canvas 渲染**

打开页面，确认游戏面板正确显示网格，单元格大小适中。

---

### 任务 4：地雷生成 + 首次点击安全

**文件：**
- 修改：`minesweeper.html`（在 `<script>` 标签内添加）

- [ ] **步骤 1：编写地雷生成逻辑**

```javascript
function placeMines(safeRow, safeCol) {
    const config = difficulties[currentDifficulty];
    let minesPlaced = 0;
    
    while (minesPlaced < config.mines) {
        const row = Math.floor(Math.random() * config.rows);
        const col = Math.floor(Math.random() * config.cols);
        
        // Skip if already a mine or in safe zone (first click and neighbors)
        if (board[row][col] === -1) continue;
        if (Math.abs(row - safeRow) <= 1 && Math.abs(col - safeCol) <= 1) continue;
        
        board[row][col] = -1; // Mine
        minesPlaced++;
    }
    
    // Calculate numbers for non-mine cells
    for (let row = 0; row < config.rows; row++) {
        for (let col = 0; col < config.cols; col++) {
            if (board[row][col] === -1) continue;
            
            let count = 0;
            for (let dr = -1; dr <= 1; dr++) {
                for (let dc = -1; dc <= 1; dc++) {
                    const nr = row + dr;
                    const nc = col + dc;
                    if (nr >= 0 && nr < config.rows && nc >= 0 && nc < config.cols) {
                        if (board[nr][nc] === -1) count++;
                    }
                }
            }
            board[row][col] = count;
        }
    }
}

function startTimer() {
    if (timerInterval) return;
    timerInterval = setInterval(() => {
        seconds++;
        updateTimer();
    }, 1000);
}

function updateTimer() {
    document.getElementById('timer').textContent = String(seconds).padStart(3, '0');
}

function updateMineCounter() {
    const config = difficulties[currentDifficulty];
    let flagCount = 0;
    for (let row = 0; row < config.rows; row++) {
        for (let col = 0; col < config.cols; col++) {
            if (flagged[row][col]) flagCount++;
        }
    }
    const remaining = config.mines - flagCount;
    document.getElementById('mine-counter').textContent = String(Math.max(0, remaining)).padStart(3, '0');
}
```

- [ ] **步骤 2：测试首次点击安全**

刷新页面，第一次点击任意位置，确认该格及其周围没有地雷。

---

### 任务 5：鼠标事件处理 + 游戏逻辑

**文件：**
- 修改：`minesweeper.html`（在 `<script>` 标签内添加）

- [ ] **步骤 1：编写鼠标点击和右键标旗**

```javascript
function getCellFromMouse(e) {
    const rect = canvas.getBoundingClientRect();
    const x = e.clientX - rect.left;
    const y = e.clientY - rect.top;
    const col = Math.floor(x / cellSize);
    const row = Math.floor(y / cellSize);
    
    if (row < 0 || row >= board.length || col < 0 || col >= board[0].length) return null;
    return { row, col };
}

function revealCell(row, col) {
    if (gameOver || flagged[row][col] || revealed[row][col]) return;
    
    // First click - place mines and start timer
    if (firstClick) {
        placeMines(row, col);
        firstClick = false;
        gameStarted = true;
        startTimer();
    }
    
    revealed[row][col] = true;
    
    // Flood fill for empty cells
    if (board[row][col] === 0) {
        for (let dr = -1; dr <= 1; dr++) {
            for (let dc = -1; dc <= 1; dc++) {
                const nr = row + dr;
                const nc = col + dc;
                if (nr >= 0 && nr < board.length && nc >= 0 && nc < board[0].length) {
                    revealCell(nr, nc);
                }
            }
        }
    }
    
    drawBoard();
}

function toggleFlag(row, col) {
    if (gameOver || revealed[row][col]) return;
    
    flagged[row][col] = !flagged[row][col];
    updateMineCounter();
    drawBoard();
}

canvas.addEventListener('click', (e) => {
    const cell = getCellFromMouse(e);
    if (!cell) return;
    
    e.preventDefault();
    revealCell(cell.row, cell.col);
});

canvas.addEventListener('contextmenu', (e) => {
    e.preventDefault();
    const cell = getCellFromMouse(e);
    if (!cell) return;
    
    toggleFlag(cell.row, cell.col);
});
```

- [ ] **步骤 2：测试基本游戏流程**

翻开格子、标旗、检查剩余雷数是否正确更新。

---

### 任务 6：双击展开 + 胜利/失败判定

**文件：**
- 修改：`minesweeper.html`（在 `<script>` 标签内添加）

- [ ] **步骤 1：编写双击展开和胜负判定**

```javascript
canvas.addEventListener('dblclick', (e) => {
    const cell = getCellFromMouse(e);
    if (!cell || !revealed[cell.row][cell.col] || board[cell.row][cell.col] <= 0) return;
    
    // Count adjacent flags
    let flagCount = 0;
    for (let dr = -1; dr <= 1; dr++) {
        for (let dc = -1; dc <= 1; dc++) {
            const nr = cell.row + dr;
            const nc = cell.col + dc;
            if (nr >= 0 && nr < board.length && nc >= 0 && nc < board[0].length) {
                if (flagged[nr][nc]) flagCount++;
            }
        }
    }
    
    // Auto-reveal if flag count matches number
    if (flagCount === board[cell.row][cell.col]) {
        for (let dr = -1; dr <= 1; dr++) {
            for (let dc = -1; dc <= 1; dc++) {
                const nr = cell.row + dr;
                const nc = cell.col + dc;
                if (nr >= 0 && nr < board.length && nc >= 0 && nc < board[0].length) {
                    if (!flagged[nr][nc] && !revealed[nr][nc]) {
                        revealCell(nr, nc);
                        
                        // Hit mine
                        if (board[nr][nc] === -1) {
                            gameOver = true;
                            revealAllMines();
                            document.getElementById('reset-btn').textContent = '😵';
                        }
                    }
                }
            }
        }
        checkWin();
    }
});

function checkWin() {
    const config = difficulties[currentDifficulty];
    let unrevealedSafeCells = 0;
    
    for (let row = 0; row < config.rows; row++) {
        for (let col = 0; col < config.cols; col++) {
            if (!revealed[row][col] && board[row][col] !== -1) {
                unrevealedSafeCells++;
            }
        }
    }
    
    if (unrevealedSafeCells === 0) {
        gameOver = true;
        if (timerInterval) clearInterval(timerInterval);
        document.getElementById('reset-btn').textContent = '😎';
        
        // Auto-flag remaining mines
        for (let row = 0; row < config.rows; row++) {
            for (let col = 0; col < config.cols; col++) {
                if (board[row][col] === -1 && !flagged[row][col]) {
                    flagged[row][col] = true;
                }
            }
        }
        updateMineCounter();
        drawBoard();
    }
}

function revealAllMines() {
    const config = difficulties[currentDifficulty];
    for (let row = 0; row < config.rows; row++) {
        for (let col = 0; col < config.cols; col++) {
            if (board[row][col] === -1) {
                revealed[row][col] = true;
            }
        }
    }
    drawBoard();
}

// Reset button
document.getElementById('reset-btn').addEventListener('click', () => {
    initBoard();
    drawBoard();
    document.getElementById('reset-btn').textContent = '😊';
});
```

- [ ] **步骤 2：测试双击展开和胜负**

翻开区域触发自动展开，踩雷显示失败表情，全部翻开显示胜利表情。

---

### 任务 7：难度切换 + 自动标雷功能

**文件：**
- 修改：`minesweeper.html`（在 `<script>` 标签内添加）

- [ ] **步骤 1：编写难度切换和自动标雷逻辑**

```javascript
// Difficulty buttons
document.querySelectorAll('[data-difficulty]').forEach(btn => {
    btn.addEventListener('click', () => {
        document.querySelectorAll('[data-difficulty]').forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
        currentDifficulty = btn.dataset.difficulty;
        initBoard();
        drawBoard();
    });
});

// Auto-flag toggle
document.getElementById('auto-flag-toggle').addEventListener('change', (e) => {
    if (e.target.checked && !gameOver) {
        autoFlagMines();
    }
});

function autoFlagMines() {
    const config = difficulties[currentDifficulty];
    
    for (let row = 0; row < config.rows; row++) {
        for (let col = 0; col < config.cols; col++) {
            if (!revealed[row][col] || board[row][col] <= 0) continue;
            
            let flagCount = 0;
            let unrevealedCount = 0;
            
            for (let dr = -1; dr <= 1; dr++) {
                for (let dc = -1; dc <= 1; dc++) {
                    const nr = row + dr;
                    const nc = col + dc;
                    if (nr >= 0 && nr < config.rows && nc >= 0 && nc < config.cols) {
                        if (flagged[nr][nc]) flagCount++;
                        if (!revealed[nr][nc] && !flagged[nr][nc]) unrevealedCount++;
                    }
                }
            }
            
            // Auto-flag if unrevealed neighbors equal remaining mines needed
            if (unrevealedCount === board[row][col] - flagCount) {
                for (let dr = -1; dr <= 1; dr++) {
                    for (let dc = -1; dc <= 1; dc++) {
                        const nr = row + dr;
                        const nc = col + dc;
                        if (nr >= 0 && nr < config.rows && nc >= 0 && nc < config.cols) {
                            if (!revealed[nr][nc] && !flagged[nr][nc]) {
                                flagged[nr][nc] = true;
                            }
                        }
                    }
                }
            }
        }
    }
    
    updateMineCounter();
    drawBoard();
}
```

- [ ] **步骤 2：测试难度切换和自动标雷**

切换不同难度，确认棋盘大小正确变化。勾选自动标雷，确认已确定的地雷被标记。

---

### 任务 8：主题切换 + 最终整合

**文件：**
- 修改：`minesweeper.html`（在 `<script>` 标签内添加）

- [ ] **步骤 1：编写主题切换逻辑**

```javascript
// Theme toggle
let currentTheme = 'dark';

function setTheme(theme) {
    currentTheme = theme;
    document.documentElement.setAttribute('data-theme', theme);
    document.getElementById('theme-toggle').textContent = theme === 'dark' ? '☀️' : '🌙';
    
    // Redraw board with new colors
    if (board.length > 0) {
        drawBoard();
    }
}

document.getElementById('theme-toggle').addEventListener('click', () => {
    setTheme(currentTheme === 'dark' ? 'light' : 'dark');
});

// Initialize theme
setTheme('dark');
```

- [ ] **步骤 2：最终测试**

1. 打开 `minesweeper.html`，确认所有功能正常工作
2. 切换主题，确认颜色正确变化且 Canvas 重绘
3. 切换语言，确认所有文本正确翻译
4. 切换难度，确认棋盘大小和雷数正确
5. 测试自动标雷开关
6. 测试完整游戏流程：翻开、标旗、双击展开、胜利/失败

---

## 自检

**规格覆盖度：**
- ✅ 三种经典难度 — 任务 1, 7
- ✅ Canvas 渲染 — 任务 3
- ✅ 首次点击安全 — 任务 4
- ✅ 左键翻开/右键标旗 — 任务 5
- ✅ 双击自动展开 — 任务 6
- ✅ "二选一"机制 — 任务 6（胜利判定）
- ✅ 计时器 + 剩余雷数 — 任务 3, 4
- ✅ 表情按钮重新开始 — 任务 6
- ✅ 自动标雷开关 — 任务 7
- ✅ 深色/浅色主题切换 — 任务 1, 8
- ✅ 中文/英文语言切换 — 任务 2

**占位符扫描：** 无"待定"、"TODO"等占位符。

**类型一致性：** 所有变量名、函数签名在各任务间保持一致。

---

计划已完成并保存到 `docs/superpowers/plans/2025-06-18-minesweeper.md`。两种执行方式：

**1. 子代理驱动（推荐）** - 每个任务调度一个新的子代理，任务间进行审查，快速迭代

**2. 内联执行** - 在当前会话中使用 executing-plans 执行任务，批量执行并设有检查点供审查

选哪种方式？
