# Game Portal — Design Spec

**Date:** 2026-06-19  
**Status:** Approved by user  

---

## Summary

Create a single `index.html` entry page that serves as the portal to four existing browser games (Tetris, Gomoku, Tank Battle, Minesweeper). The page uses GitHub's design system for visual consistency — dark/light themes, EN/CN language toggle, glassmorphism cards. Clicking a game card navigates directly to the corresponding `.html` file.

---

## Design Decisions

| Decision | Choice |
|----------|--------|
| Logo | User-provided image → `assets/logo.png` |
| Entry mode | Navigate to existing `.html` files (no iframe) |
| Tech stack | Pure HTML/CSS/JS, zero dependencies |
| Visual style | GitHub design system (`#0d1117` dark / `#f6f8fa` light) |
| Game cards | Emoji icon + game name only |
| Layout | Centered logo → 2×2 card grid → footer watermark |

---

## File Structure

```
Game 1/
├── index.html              ← NEW: portal entry page
├── assets/                 ← NEW directory
│   └── logo.png            ← user's logo image (to be placed)
├── tetris.html             ← existing
├── gomoku.html             ← existing
├── tanks.html              ← existing
└── minesweeper.html        ← existing
```

---

## Page Layout

### Top Bar (fixed, right side)
- Language toggle button: `EN` / `中文` — switches all UI text between English and Chinese
- Theme toggle button: `🌙` / `☀️` — switches dark/light theme
- Both buttons use the same glassmorphism style as existing games

### Center Area
1. **Logo section** (top-center): `assets/logo.png` image, sized appropriately (~200px max width)
2. **Tagline**: "Romeo's Game Collection" / "Romeo 的游戏合集" — subtle subtitle below logo
3. **Game card grid** (2×2 layout):
   - Each card: emoji icon + game name + brief description
   - Cards are clickable links to the corresponding `.html` file
   - Hover effect: slight lift + border highlight
   - Card content is i18n-aware

### Footer (fixed, bottom-left)
- Watermark text: `© 2026 Romeo Games` — subtle, low opacity

---

## Game Cards Content

| Emoji | EN Name | CN Name | EN Description | CN Description | Target File |
|-------|---------|---------|----------------|----------------|-------------|
| 🧱 | Tetris | 俄罗斯方块 | Classic block puzzle | 经典方块消除 | `tetris.html` |
| ⚫ | Gomoku | 五子棋 | Five-in-a-row strategy | 传统策略对弈 | `gomoku.html` |
| 🔫 | Tank Battle | 坦克大战 | Classic arcade shooter | 怀旧射击游戏 | `tanks.html` |
| 💣 | Minesweeper | 扫雷 | Logic and deduction | 经典逻辑推理 | `minesweeper.html` |

---

## Technical Details

### CSS Architecture
- Use CSS custom properties (`--bg`, `--text`, etc.) matching existing games' design tokens
- Dark theme is default (matches GitHub dark)
- Light theme via `[data-theme="light"]` attribute on `<html>`
- Glassmorphism: `backdrop-filter: blur(24px)` + semi-transparent backgrounds
- Responsive: cards stack to single column on narrow screens (< 600px)

### JavaScript Features
1. **Theme toggle**: reads/writes `localStorage.getItem('theme')`, sets `data-theme` attribute
2. **Language toggle**: reads/writes `localStorage.getItem('lang')`, swaps all text content via a translation dictionary object
3. No frameworks, no build step — vanilla JS only

### Translation Dictionary (complete)
```js
const i18n = {
  en: {
    title: "Romeo's Game Collection",
    tagline: "Classic games, rebuilt with modern web tech.",
    games: {
      tetris: { name: "Tetris", desc: "Classic block puzzle" },
      gomoku: { name: "Gomoku", desc: "Five-in-a-row strategy" },
      tanks: { name: "Tank Battle", desc: "Classic arcade shooter" },
      minesweeper: { name: "Minesweeper", desc: "Logic and deduction" }
    },
    watermark: "© 2026 Romeo Games"
  },
  zh: {
    title: "Romeo 的游戏合集",
    tagline: "经典游戏，用现代 Web 技术重新构建。",
    games: {
      tetris: { name: "俄罗斯方块", desc: "经典方块消除" },
      gomoku: { name: "五子棋", desc: "传统策略对弈" },
      tanks: { name: "坦克大战", desc: "怀旧射击游戏" },
      minesweeper: { name: "扫雷", desc: "经典逻辑推理" }
    },
    watermark: "© 2026 Romeo Games"
  }
};
```

### Accessibility
- All game cards use `<a>` tags (semantic, keyboard navigable)
- `aria-label` on toggle buttons
- Sufficient color contrast in both themes

---

## Assumptions & Defaults

1. **Logo file**: User will place their logo image at `assets/logo.png`. If the filename differs, it can be adjusted easily.
2. **Existing games**: No changes needed to existing `.html` files — they already have their own language/theme toggles and footer watermarks. The portal page is a separate entry point.
3. **No routing**: Simple direct links (`href="tetris.html"`), no SPA routing needed.
4. **localStorage keys**: `theme` for dark/light preference, `lang` for en/zh preference.

---

## Testing Checklist

- [ ] Page loads correctly in Chrome/Firefox/Safari
- [ ] Dark theme renders correctly with proper contrast
- [ ] Light theme renders correctly with proper contrast  
- [ ] Language toggle switches all text between EN/CN
- [ ] Theme toggle persists across page reloads (localStorage)
- [ ] All 4 game cards navigate to correct `.html` files
- [ ] Logo image displays correctly at `assets/logo.png`
- [ ] Responsive layout works on mobile widths (< 600px)
- [ ] No console errors or warnings

---

## Out of Scope (for v1)

- Animated transitions between theme/language switches
- Game preview thumbnails/screenshots on cards
- User accounts, scores, leaderboards
- Mobile app wrapper
