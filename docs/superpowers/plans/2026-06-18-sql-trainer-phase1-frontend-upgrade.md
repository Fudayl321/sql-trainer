# SQL Trainer Phase 1 Frontend Upgrade Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Upgrade the SQL Trainer PWA with a gamified dark visual design, mobile improvements, and a JS animation layer — all within the existing single-file architecture.

**Architecture:** All CSS changes go inside the single `<style>` block in `index.html`. All JS changes go inside the single `<script>` block. A new UI/UX Designer agent is added to the developer squad. Service worker cache bumps to `sql-trainer-v11` after implementation is complete.

**Tech Stack:** Vanilla HTML/CSS/JS · AlaSQL 4.2.3 (CDN) · localStorage · GitHub Pages PWA

**Spec:** `docs/superpowers/specs/2026-06-18-sql-trainer-phase1-frontend-upgrade-design.md`

---

## File Map

| File | Change |
|------|--------|
| `index.html` | All CSS + JS changes (single file) |
| `sw.js` | Bump cache version to `sql-trainer-v11` |
| `C:\Project\.claude\agents\ui-ux-designer.md` | Create project-level agent |
| `C:\Users\User\.claude\agents\ui-ux-designer.md` | Create global agent |

---

## Task 1: Create UI/UX Designer Agent

**Files:**
- Create: `C:\Project\.claude\agents\ui-ux-designer.md`
- Create: `C:\Users\User\.claude\agents\ui-ux-designer.md`

- [ ] **Step 1: Create project-level agent file**

Write to `C:\Project\.claude\agents\ui-ux-designer.md`:

```markdown
---
name: ui-ux-designer
description: UI/UX Designer agent. Use after Project Manager and before Coder for any frontend feature. Produces component specs with exact CSS values, animation timings, markup structure, and accessibility requirements. Writes no implementation code — output is consumed by the Coder.
tools: Read, Grep, Glob, Write
---

# UI/UX Designer

You are the UI/UX Designer in the Developer Squad. You receive requirements from the Project Manager and produce a detailed component spec the Coder implements directly.

## Your Output

For each UI change, specify:
- **CSS values** — exact variable names, px values, timing functions, opacity levels
- **Markup structure** — HTML element types, class names, id attributes, nesting
- **Animation timings** — duration in ms, easing function, what triggers it, what it does
- **Accessibility** — ARIA labels for dynamic content, `prefers-reduced-motion` guards, minimum contrast ratios (4.5:1 for body text, 3:1 for large text)
- **States** — default, hover, focus, active, disabled, error, success

## Rules

- Follow the existing CSS variable system in `:root`. Never introduce new hardcoded color values.
- All animations MUST include a `prefers-reduced-motion` guard.
- Touch targets must be ≥ 44px height on mobile.
- Font sizes in inputs/textareas must be ≥ 16px to prevent iOS auto-zoom.
- Write a spec document — do NOT write HTML/CSS/JS code files.

## Squad Position

PM → **UI/UX Designer** → Coder → DevOps → Tester → Reviewer → Consultant
```

- [ ] **Step 2: Create global agent file**

Copy identical content to `C:\Users\User\.claude\agents\ui-ux-designer.md`.

- [ ] **Step 3: Commit**

```bash
git add .claude/agents/ui-ux-designer.md
git commit -m "feat: add UI/UX Designer agent to developer squad"
```

---

## Task 2: Add Glow Tokens + Level Color Variables to `:root`

**Files:**
- Modify: `index.html:11-25` (`:root` block)

The `:root` block currently ends at line 25 with `--code-text: #7ee787;`. Add new variables after `--code-text`.

- [ ] **Step 1: Add variables to :root**

Replace the closing of `:root` (line 25 area):

Find this exact text:
```css
      --code-text: #7ee787;
    }
```

Replace with:
```css
      --code-text: #7ee787;

      /* Glow tokens */
      --glow-cyan:   0 0 12px rgba(0,212,255,0.6),  0 0 24px rgba(0,212,255,0.2);
      --glow-purple: 0 0 12px rgba(124,58,237,0.6),  0 0 24px rgba(124,58,237,0.2);
      --glow-green:  0 0 12px rgba(34,197,94,0.5),   0 0 24px rgba(34,197,94,0.15);
      --glow-red:    0 0 12px rgba(239,68,68,0.5);
      --glow-amber:  0 0 10px rgba(245,158,11,0.5);

      /* Level accent colors */
      --level-1: #00d4ff;
      --level-2: #7c3aed;
      --level-3: #22c55e;
      --level-4: #f59e0b;
      --level-5: #ef4444;
    }
```

- [ ] **Step 2: Verify in browser**

Open `index.html` in browser. Open DevTools Console and run:
```js
getComputedStyle(document.documentElement).getPropertyValue('--glow-cyan')
```
Expected: a non-empty string with box-shadow values.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add glow tokens and level color CSS variables"
```

---

## Task 3: Card Depth + Typography Upgrades

**Files:**
- Modify: `index.html` — `.card` rule (line 127), heading sizes, nav label letter-spacing, code block accent

- [ ] **Step 1: Upgrade .card with gradient top-highlight**

Find:
```css
    .card {
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: 12px;
      padding: 16px;
      margin-bottom: 12px;
    }
```

Replace with:
```css
    .card {
      background: linear-gradient(180deg, rgba(255,255,255,0.03) 0%, transparent 60%), var(--surface);
      border: 1px solid var(--border);
      border-radius: 12px;
      padding: 16px;
      margin-bottom: 12px;
    }
```

- [ ] **Step 2: Add textarea focus glow**

Find:
```css
    textarea:focus { border-color: var(--accent); }
```

Replace with:
```css
    textarea:focus { border-color: var(--accent); box-shadow: var(--glow-cyan); }
```

- [ ] **Step 3: iOS textarea zoom fix — set font-size to 16px**

Find:
```css
      font-size: 13px;
```
(inside the `textarea` rule at line ~240)

Replace with:
```css
      font-size: 16px;
```

- [ ] **Step 4: Add nav label letter-spacing**

Find:
```css
    .nav-btn {
      flex: 1;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      gap: 3px;
      padding: 10px 4px;
      background: none;
      border: none;
      color: var(--text-dim);
      font-size: 11px;
      cursor: pointer;
      min-height: 56px;
      transition: color 0.15s;
    }
```

Replace with:
```css
    .nav-btn {
      flex: 1;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      gap: 3px;
      padding: 10px 4px;
      background: none;
      border: none;
      color: var(--text-dim);
      font-size: 11px;
      letter-spacing: 0.03em;
      cursor: pointer;
      min-height: 56px;
      transition: color 0.15s, box-shadow 0.25s, transform 0.25s;
    }
```

- [ ] **Step 5: Upgrade nav active state with glow + scale**

Find:
```css
    .nav-btn.active { color: var(--accent); }
```

Replace with:
```css
    .nav-btn.active {
      color: var(--accent);
      text-shadow: 0 0 8px rgba(0,212,255,0.6);
      transform: scale(1.08);
    }
```

- [ ] **Step 6: Safe area insets for bottom nav**

Find:
```css
    #bottom-nav {
      position: fixed;
      bottom: 0;
      left: 50%;
      transform: translateX(-50%);
      width: 100%;
      max-width: 640px;
      background: var(--surface);
      border-top: 1px solid var(--border);
      display: flex;
      z-index: 100;
    }
```

Replace with:
```css
    #bottom-nav {
      position: fixed;
      bottom: 0;
      left: 50%;
      transform: translateX(-50%);
      width: 100%;
      max-width: 640px;
      background: var(--surface);
      border-top: 1px solid var(--border);
      display: flex;
      z-index: 100;
      padding-bottom: env(safe-area-inset-bottom, 0px);
    }
```

- [ ] **Step 7: Verify in browser**

Open `index.html`. Check:
- Cards have a subtle lighter top edge (barely visible in dark mode — compare with before)
- Focused textarea shows a cyan glow ring
- Active nav tab icon appears slightly larger and has a glow text effect

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "feat: card depth, textarea glow, nav active upgrade, safe area insets, iOS zoom fix"
```

---

## Task 4: Level Badge + Streak Flame CSS + Dynamic Classes

**Files:**
- Modify: `index.html` — `.badge` CSS (line 70), `updateHeader()` function (line 4425)

The header already has `<span class="badge level" id="hdr-level">` and `<span class="badge streak" id="hdr-streak">`. We add level-specific CSS classes and update `updateHeader()` to apply them.

- [ ] **Step 1: Add level badge CSS after existing .badge rule**

Find:
```css
    .badge {
      background: var(--surface2);
      border: 1px solid var(--border);
      border-radius: 20px;
      padding: 4px 10px;
      font-size: 13px;
      font-weight: 600;
      color: var(--text-dim);
    }
```

Replace with:
```css
    .badge {
      background: var(--surface2);
      border: 1px solid var(--border);
      border-radius: 20px;
      padding: 4px 10px;
      font-size: 13px;
      font-weight: 600;
      color: var(--text-dim);
      transition: box-shadow 0.3s;
    }
    .badge.lvl-1 { color: var(--level-1); border-color: rgba(0,212,255,0.4);   background: rgba(0,212,255,0.08);   box-shadow: 0 0 8px rgba(0,212,255,0.25); }
    .badge.lvl-2 { color: var(--level-2); border-color: rgba(124,58,237,0.4);  background: rgba(124,58,237,0.08);  box-shadow: 0 0 8px rgba(124,58,237,0.25); }
    .badge.lvl-3 { color: var(--level-3); border-color: rgba(34,197,94,0.4);   background: rgba(34,197,94,0.08);   box-shadow: 0 0 8px rgba(34,197,94,0.25); }
    .badge.lvl-4 { color: var(--level-4); border-color: rgba(245,158,11,0.4);  background: rgba(245,158,11,0.08);  box-shadow: 0 0 8px rgba(245,158,11,0.25); }
    .badge.lvl-5 { color: var(--level-5); border-color: rgba(239,68,68,0.4);   background: rgba(239,68,68,0.08);   box-shadow: 0 0 8px rgba(239,68,68,0.25); }
    .streak-glow { box-shadow: var(--glow-amber) !important; color: var(--warning) !important; border-color: rgba(245,158,11,0.5) !important; }
```

- [ ] **Step 2: Update updateHeader() to apply level class and streak glow**

Find:
```js
function updateHeader() {
  document.getElementById('hdr-level').textContent = `Lvl ${progress.level} · ${CURRICULUM[progress.level].name}`;
  // Show 0 if the streak is stale (missed at least one day and today not yet completed)
  const today = getTodayString();
  const yesterday = (function(){
    const d = new Date(); d.setDate(d.getDate()-1);
    return `${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,'0')}-${String(d.getDate()).padStart(2,'0')}`;
  })();
  const streakAlive = progress.lastStreakDate === today || progress.lastStreakDate === yesterday;
  const displayStreak = streakAlive ? progress.streak : 0;
  document.getElementById('hdr-streak').textContent = `🔥 ${displayStreak}`;
}
```

Replace with:
```js
function updateHeader() {
  const lvlEl = document.getElementById('hdr-level');
  lvlEl.textContent = `Lvl ${progress.level} · ${CURRICULUM[progress.level].name}`;
  lvlEl.className = `badge level lvl-${progress.level}`;

  const today = getTodayString();
  const yesterday = (function(){
    const d = new Date(); d.setDate(d.getDate()-1);
    return `${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,'0')}-${String(d.getDate()).padStart(2,'0')}`;
  })();
  const streakAlive = progress.lastStreakDate === today || progress.lastStreakDate === yesterday;
  const displayStreak = streakAlive ? progress.streak : 0;
  const streakEl = document.getElementById('hdr-streak');
  streakEl.textContent = `🔥 ${displayStreak}`;
  if (displayStreak >= 3) streakEl.classList.add('streak-glow');
  else streakEl.classList.remove('streak-glow');
}
```

- [ ] **Step 3: Verify in browser**

Open `index.html`. Check:
- Header level badge shows cyan glow (Level 1 = cyan)
- Header badge reads "Lvl 1 · Foundations" with colored text
- At streak ≥ 3, streak badge glows amber

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: level badge colors and streak flame glow in header"
```

---

## Task 5: Segmented XP Bar in Progress Tab

**Files:**
- Modify: `index.html` — Progress tab HTML (line 618), `.progress-bar-wrap/.progress-bar-fill` CSS (lines 367-379), `renderProgress()` function (lines 4368-4398)

Replace the single plain bar with a segmented bar (one segment per concept in the current level).

- [ ] **Step 1: Replace progress bar HTML in Progress tab**

Find:
```html
      <div class="stat-row"><span>Level progress</span><span class="stat-val" id="prog-pct">0%</span></div>
      <div class="progress-bar-wrap"><div class="progress-bar-fill" id="prog-bar" style="width:0%"></div></div>
```

Replace with:
```html
      <div class="stat-row"><span>Level progress</span><span class="stat-val" id="prog-pct">0%</span></div>
      <div class="xp-bar-wrap" id="xp-bar-wrap"></div>
```

- [ ] **Step 2: Add XP bar CSS (replace old progress-bar CSS)**

Find:
```css
    .progress-bar-wrap {
      background: var(--surface2);
      border-radius: 8px;
      height: 10px;
      margin: 8px 0 14px;
      overflow: hidden;
    }
    .progress-bar-fill {
      height: 100%;
      background: linear-gradient(90deg, var(--accent2), var(--accent));
      border-radius: 8px;
      transition: width 0.4s;
    }
```

Replace with:
```css
    .xp-bar-wrap {
      display: flex;
      gap: 3px;
      margin: 8px 0 14px;
      height: 10px;
    }
    .xp-segment {
      flex: 1;
      border-radius: 3px;
      background: var(--surface2);
      transition: background 0.3s, box-shadow 0.3s;
    }
    .xp-segment.filled {
      animation: xp-fill 0.5s ease forwards;
    }
    .xp-segment.lvl-1.filled { background: var(--level-1); box-shadow: 0 0 6px rgba(0,212,255,0.5); }
    .xp-segment.lvl-2.filled { background: var(--level-2); box-shadow: 0 0 6px rgba(124,58,237,0.5); }
    .xp-segment.lvl-3.filled { background: var(--level-3); box-shadow: 0 0 6px rgba(34,197,94,0.5); }
    .xp-segment.lvl-4.filled { background: var(--level-4); box-shadow: 0 0 6px rgba(245,158,11,0.5); }
    .xp-segment.lvl-5.filled { background: var(--level-5); box-shadow: 0 0 6px rgba(239,68,68,0.5); }
```

- [ ] **Step 3: Update renderProgress() to render segmented XP bar**

Find:
```js
  document.getElementById('prog-level').textContent = `${level} — ${lvlName}`;
  document.getElementById('prog-streak').textContent = progress.streak;
  document.getElementById('prog-sessions').textContent = progress.completedSessions;
  document.getElementById('prog-pct').textContent = `${pct}%`;
  document.getElementById('prog-bar').style.width = `${pct}%`;
```

Replace with:
```js
  document.getElementById('prog-level').textContent = `${level} — ${lvlName}`;
  document.getElementById('prog-streak').textContent = progress.streak;
  document.getElementById('prog-sessions').textContent = progress.completedSessions;
  document.getElementById('prog-pct').textContent = `${pct}%`;

  // Segmented XP bar — one segment per concept in current level
  const xpWrap = document.getElementById('xp-bar-wrap');
  if (xpWrap) {
    xpWrap.innerHTML = concepts.map((c, i) => {
      const done = QUESTIONS.filter(q => q.concept === c).some(q => progress.completedQuestions.includes(q.id));
      return `<div class="xp-segment lvl-${level}${done ? ' filled' : ''}"></div>`;
    }).join('');
  }
```

- [ ] **Step 4: Verify in browser**

Open `index.html` → Progress tab. Check:
- A row of small pill-shaped segments appears where the progress bar was
- Completed concepts show a colored glowing segment; uncompleted show dark gray
- Percentage text still shows correctly

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: segmented XP bar in Progress tab with level-color glow"
```

---

## Task 6: PWA Install Banner

**Files:**
- Modify: `index.html` — CSS (add banner styles), JS (add install banner logic near bottom of script)

- [ ] **Step 1: Add install banner CSS**

Find the end of the `</style>` tag. Insert before it:

```css
    /* PWA install banner */
    #pwa-banner {
      background: var(--surface2);
      border-bottom: 1px solid var(--border);
      padding: 10px 16px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 10px;
      font-size: 13px;
      color: var(--text);
      position: relative;
      z-index: 99;
    }
    #pwa-banner button {
      background: var(--accent);
      color: #000;
      border: none;
      border-radius: 6px;
      padding: 6px 14px;
      font-size: 12px;
      font-weight: 700;
      cursor: pointer;
      white-space: nowrap;
    }
    #pwa-banner .pwa-dismiss {
      background: none;
      color: var(--text-dim);
      border: none;
      font-size: 18px;
      cursor: pointer;
      padding: 0 4px;
      line-height: 1;
    }
```

- [ ] **Step 2: Add offline pill CSS**

Append to the same location (just after the PWA banner CSS, before `</style>`):

```css
    /* Offline indicator */
    #offline-pill {
      background: rgba(245,158,11,0.15);
      border: 1px solid rgba(245,158,11,0.4);
      color: var(--warning);
      border-radius: 20px;
      padding: 2px 10px;
      font-size: 11px;
      font-weight: 600;
      margin-left: 8px;
    }
```

- [ ] **Step 3: Add PWA install + offline JS logic**

Find the very end of the `<script>` block — look for the final `})();` or `init()` call. Append the following BEFORE the closing `</script>` tag:

```js
// ─── PWA INSTALL BANNER ──────────────────────────────────────────────────────

let _installPrompt = null;

window.addEventListener('beforeinstallprompt', e => {
  e.preventDefault();
  _installPrompt = e;
  if (!localStorage.getItem('pwa-dismissed')) showInstallBanner();
});

function showInstallBanner() {
  if (document.getElementById('pwa-banner')) return;
  const banner = document.createElement('div');
  banner.id = 'pwa-banner';
  banner.innerHTML = `
    <span>📱 Install SQL Trainer as an app</span>
    <div style="display:flex;gap:6px;align-items:center">
      <button onclick="triggerInstall()">Install</button>
      <button class="pwa-dismiss" onclick="dismissInstallBanner()" aria-label="Dismiss">×</button>
    </div>`;
  const main = document.getElementById('main');
  main.insertBefore(banner, main.firstChild);
}

function triggerInstall() {
  if (!_installPrompt) return;
  _installPrompt.prompt();
  _installPrompt.userChoice.then(() => { _installPrompt = null; });
  dismissInstallBanner();
}

function dismissInstallBanner() {
  localStorage.setItem('pwa-dismissed', '1');
  const b = document.getElementById('pwa-banner');
  if (b) b.remove();
}

// ─── OFFLINE INDICATOR ───────────────────────────────────────────────────────

function updateOnlineStatus() {
  const existing = document.getElementById('offline-pill');
  if (!navigator.onLine) {
    if (!existing) {
      const pill = document.createElement('span');
      pill.id = 'offline-pill';
      pill.textContent = 'Offline';
      document.querySelector('#header .badges').appendChild(pill);
    }
  } else {
    if (existing) existing.remove();
  }
}

window.addEventListener('online',  updateOnlineStatus);
window.addEventListener('offline', updateOnlineStatus);
updateOnlineStatus();
```

- [ ] **Step 4: Verify in browser**

Open `index.html`. Check:
- No install banner appears (won't fire outside PWA context / if already dismissed)
- Offline pill: in DevTools → Network tab, set throttle to "Offline", then run `window.dispatchEvent(new Event('offline'))` in Console. A small amber "Offline" pill should appear in the header. Run `window.dispatchEvent(new Event('online'))` — pill disappears.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: PWA install banner and offline indicator"
```

---

## Task 7: Auto-Scroll After Answer Submit

**Files:**
- Modify: `index.html` — `submitAnswer()` function (line ~3236)

After a correct answer, the page should scroll the feedback block into view on small screens.

- [ ] **Step 1: Add scroll call after renderToday() in the success path**

Find:
```js
    saveProgress(progress);
    setTimeout(() => renderToday(), 400);
  } else {
```

Replace with:
```js
    saveProgress(progress);
    setTimeout(() => {
      renderToday();
      document.getElementById(`fb-${qKey}`)?.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
    }, 400);
  } else {
```

Also add scroll for the fail path. Find:
```js
    saveProgress(progress);
  }
}
```
(the final two lines of submitAnswer)

Replace with:
```js
    saveProgress(progress);
    document.getElementById(`fb-${qKey}`)?.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
  }
}
```

- [ ] **Step 2: Verify in browser**

Open `index.html` on a small window (< 400px wide). Submit a correct or wrong answer — the feedback text should scroll into view automatically.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: auto-scroll feedback into view after answer submit"
```

---

## Task 8: Add CSS @keyframes

**Files:**
- Modify: `index.html` — add `@keyframes` rules inside `<style>` block

- [ ] **Step 1: Add keyframes before `</style>`**

Find the closing `</style>` tag. Insert immediately before it:

```css
    /* ── Animation keyframes ─────────────────────────────── */
    @keyframes correct-pulse {
      0%   { box-shadow: var(--glow-green); }
      100% { box-shadow: none; }
    }
    @keyframes shake {
      0%, 100% { transform: translateX(0); }
      20%       { transform: translateX(-6px); }
      60%       { transform: translateX(6px); }
    }
    @keyframes xp-fill {
      from { opacity: 0; transform: scaleX(0.5); }
      to   { opacity: 1; transform: scaleX(1); }
    }
    @keyframes slide-in-right {
      from { transform: translateX(110%); opacity: 0; }
      to   { transform: translateX(0);    opacity: 1; }
    }
    @keyframes fade-in-up {
      from { opacity: 0; transform: translateY(12px); }
      to   { opacity: 1; transform: translateY(0); }
    }
    @keyframes overlay-flash {
      0%   { opacity: 0; }
      15%  { opacity: 1; }
      80%  { opacity: 1; }
      100% { opacity: 0; }
    }

    /* Session-complete overlay */
    #session-overlay {
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,0.75);
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 500;
      animation: overlay-flash 1.2s ease forwards;
      pointer-events: none;
    }
    #session-overlay .overlay-content {
      text-align: center;
      animation: fade-in-up 0.4s ease;
    }
    #session-overlay .overlay-title {
      font-size: 28px;
      font-weight: 800;
      color: var(--success);
      text-shadow: var(--glow-green);
      margin-bottom: 6px;
    }
    #session-overlay .overlay-sub {
      font-size: 15px;
      color: var(--text-dim);
    }

    /* Level-up modal */
    #levelup-modal {
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,0.8);
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 600;
    }
    #levelup-modal .modal-box {
      background: var(--surface);
      border-radius: 16px;
      padding: 32px 28px;
      text-align: center;
      max-width: 320px;
      width: 90%;
      animation: fade-in-up 0.35s ease;
    }
    #levelup-modal .modal-title {
      font-size: 32px;
      font-weight: 900;
      margin-bottom: 8px;
    }
    #levelup-modal .modal-level {
      font-size: 18px;
      font-weight: 700;
      margin-bottom: 16px;
    }
    #levelup-modal .modal-close {
      background: var(--accent);
      color: #000;
      border: none;
      border-radius: 8px;
      padding: 10px 24px;
      font-size: 15px;
      font-weight: 700;
      cursor: pointer;
    }

    /* Streak toast */
    #streak-toast {
      position: fixed;
      top: 72px;
      right: 12px;
      background: var(--surface);
      border: 1px solid rgba(245,158,11,0.5);
      border-radius: 10px;
      padding: 10px 16px;
      font-size: 14px;
      font-weight: 700;
      color: var(--warning);
      box-shadow: var(--glow-amber);
      z-index: 550;
      animation: slide-in-right 0.35s ease;
    }
```

- [ ] **Step 2: Verify in browser**

Open `index.html`. Run in DevTools Console:
```js
const d = document.createElement('div');
d.style.cssText = 'position:fixed;inset:0;background:red;z-index:999;animation:overlay-flash 1.2s forwards';
document.body.appendChild(d);
setTimeout(() => d.remove(), 1300);
```
Expected: a red flash overlay appears and fades out.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add CSS keyframes and overlay/modal/toast styles"
```

---

## Task 9: animate() Dispatcher Function

**Files:**
- Modify: `index.html` — add `animate()` function near the bottom of `<script>` (before the PWA banner code from Task 6)

- [ ] **Step 1: Add animate() function**

Find the `// ─── PWA INSTALL BANNER` comment added in Task 6. Insert the following BEFORE that comment:

```js
// ─── ANIMATION LAYER ─────────────────────────────────────────────────────────

const _shownToasts = new Set(JSON.parse(localStorage.getItem('shown-toasts') || '[]'));

function animate(type, arg) {
  if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return;

  if (type === 'correct-pulse' && arg) {
    arg.style.animation = 'none';
    arg.offsetHeight; // reflow
    arg.style.animation = 'correct-pulse 0.6s ease forwards';
    setTimeout(() => { if (arg) arg.style.animation = ''; }, 650);
  }

  if (type === 'shake' && arg) {
    arg.style.animation = 'none';
    arg.offsetHeight;
    arg.style.animation = 'shake 0.3s ease';
    setTimeout(() => { if (arg) arg.style.animation = ''; }, 350);
  }

  if (type === 'session-complete') {
    const overlay = document.createElement('div');
    overlay.id = 'session-overlay';
    overlay.innerHTML = `<div class="overlay-content">
      <div class="overlay-title">Session Complete!</div>
      <div class="overlay-sub">Concept complete · Keep the streak going 🔥</div>
    </div>`;
    document.body.appendChild(overlay);
    setTimeout(() => overlay.remove(), 1250);
  }

  if (type === 'level-up' && arg) {
    const newLevel = arg;
    const levelNames = { 1:'Foundations', 2:'Aggregation', 3:'Joins & Subqueries', 4:'CTEs & Windows', 5:'DML & Data Changes' };
    const colors = { 1:'#00d4ff', 2:'#7c3aed', 3:'#22c55e', 4:'#f59e0b', 5:'#ef4444' };
    const modal = document.createElement('div');
    modal.id = 'levelup-modal';
    modal.innerHTML = `<div class="modal-box" style="border: 2px solid ${colors[newLevel]}; box-shadow: 0 0 24px ${colors[newLevel]}44;">
      <div class="modal-title" style="color:${colors[newLevel]}">Level Up! 🎉</div>
      <div class="modal-level">Level ${newLevel} · ${levelNames[newLevel] || ''}</div>
      <button class="modal-close" onclick="document.getElementById('levelup-modal').remove()">Let's Go!</button>
    </div>`;
    document.body.appendChild(modal);
  }

  if (type === 'streak-toast' && arg) {
    const streak = arg;
    const milestones = [3, 7, 14];
    const key = `streak-${streak}`;
    if (!milestones.includes(streak) || _shownToasts.has(key)) return;
    _shownToasts.add(key);
    localStorage.setItem('shown-toasts', JSON.stringify([..._shownToasts]));
    const toast = document.createElement('div');
    toast.id = 'streak-toast';
    toast.textContent = `🔥 ${streak}-day streak!`;
    document.body.appendChild(toast);
    setTimeout(() => toast.remove(), 3000);
  }
}
```

- [ ] **Step 2: Verify animate() exists in browser**

Open `index.html`. In DevTools Console, run:
```js
animate('session-complete');
```
Expected: a dark overlay with "Session Complete!" text flashes and disappears after ~1.2s.

```js
animate('level-up', 2);
```
Expected: a purple-bordered modal with "Level Up! 🎉" appears. Clicking "Let's Go!" dismisses it.

```js
animate('streak-toast', 7);
```
Expected: an amber toast "🔥 7-day streak!" slides in from the right, disappears after 3s. (Note: once shown it won't show again for 7 until localStorage is cleared.)

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: animate() dispatcher with session-complete, level-up, streak-toast, correct/wrong animations"
```

---

## Task 10: Wire Animations into submitAnswer()

**Files:**
- Modify: `index.html` — `submitAnswer()` function (lines ~3217-3257)

The question card element ID follows the pattern `q-card-${qKey}` — but looking at the render code, cards are rendered as `<div class="q-card ...">` without an explicit id. We'll target the feedback element's parent card.

Actually, looking at renderToday() line 3148: `html += \`<div class="q-card${isLocked?' locked':''}">\`` — no id on the card. We need to add one so animate() can find it. We'll add `id="qcard-${k}"` in renderToday().

- [ ] **Step 1: Add id to q-card in renderToday()**

Find:
```js
    html += `<div class="q-card${isLocked?' locked':''}">
```

Replace with:
```js
    html += `<div class="q-card${isLocked?' locked':''}" id="qcard-${k}">
```

- [ ] **Step 2: Wire correct animation in submitAnswer() success path**

Find:
```js
    fb.className = 'feedback pass show';
    fb.innerHTML = `Correct! Your query returned ${ev.result.length} row(s).`;
    rp.className = 'result-preview show';
    rp.innerHTML = buildResultTable(ev.result);
```

Replace with:
```js
    fb.className = 'feedback pass show';
    fb.innerHTML = `Correct! Your query returned ${ev.result.length} row(s).`;
    rp.className = 'result-preview show';
    rp.innerHTML = buildResultTable(ev.result);
    animate('correct-pulse', document.getElementById(`qcard-${qKey}`));
```

- [ ] **Step 3: Wire shake animation in submitAnswer() fail path**

Find:
```js
    fb.className = 'feedback fail show';
    let msg = ev.hint || 'Not quite right.';
```

Replace with:
```js
    fb.className = 'feedback fail show';
    animate('shake', document.getElementById(`qcard-${qKey}`));
    let msg = ev.hint || 'Not quite right.';
```

- [ ] **Step 4: Verify in browser**

Open `index.html` → Today tab. Submit a correct answer — the card should briefly pulse green. Submit a wrong answer — the card should shake horizontally.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: wire correct-pulse and shake animations into submitAnswer"
```

---

## Task 11: Session-Complete and Level-Up in completeSession()

**Files:**
- Modify: `index.html` — `completeSession()` function (lines 2772-2823)

- [ ] **Step 1: Track previous level before incrementing**

Find:
```js
  // check level up: concepts.length * 2 sessions = one full pass through all concepts with both question types
  const currentLevel = progress.level;
  if (currentLevel < 5) {
    const concepts = CURRICULUM[currentLevel].concepts;
    const sessionsNeeded = concepts.length;
    if (progress.completedSessions >= sessionsNeeded) {
      const sessionsForLevel = progress.completedSessions % sessionsNeeded === 0 &&
        progress.completedSessions > 0 &&
        CURRICULUM[currentLevel].concepts.every(c =>
          QUESTIONS.some(q => q.concept===c && q.level===currentLevel && progress.completedQuestions.includes(q.id))
        );
      if (sessionsForLevel) {
        progress.level = Math.min(5, currentLevel + 1);
      }
    }
  }

  saveProgress(progress);
}
```

Replace with:
```js
  // check level up
  const currentLevel = progress.level;
  let didLevelUp = false;
  if (currentLevel < 5) {
    const concepts = CURRICULUM[currentLevel].concepts;
    const sessionsNeeded = concepts.length;
    if (progress.completedSessions >= sessionsNeeded) {
      const sessionsForLevel = progress.completedSessions % sessionsNeeded === 0 &&
        progress.completedSessions > 0 &&
        CURRICULUM[currentLevel].concepts.every(c =>
          QUESTIONS.some(q => q.concept===c && q.level===currentLevel && progress.completedQuestions.includes(q.id))
        );
      if (sessionsForLevel) {
        progress.level = Math.min(5, currentLevel + 1);
        didLevelUp = true;
      }
    }
  }

  saveProgress(progress);

  // Fire animations after save
  setTimeout(() => {
    animate('session-complete');
    if (didLevelUp) setTimeout(() => animate('level-up', progress.level), 1300);
  }, 50);
}
```

- [ ] **Step 2: Verify in browser**

Open `index.html`. In DevTools Console, simulate a session completion:
```js
completeSession();
```
Expected: "Session Complete!" overlay flashes for ~1.2s. No level-up modal (since level won't increment without full concept coverage).

To test level-up: temporarily run `progress.level = 1; progress.completedSessions = 10; saveProgress(progress);` then call `completeSession()` with conditions met — level-up modal should appear after the overlay fades.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: session-complete overlay and level-up modal in completeSession"
```

---

## Task 12: Streak Milestone Toast in renderProgress()

**Files:**
- Modify: `index.html` — `renderProgress()` function (lines 4368-4398)

- [ ] **Step 1: Add milestone toast call to renderProgress()**

Find at the end of `renderProgress()`:
```js
  document.getElementById('concept-list').innerHTML = html;
}
```

Replace with:
```js
  document.getElementById('concept-list').innerHTML = html;

  // Streak milestone toast
  const today = getTodayString();
  const streakAlive = progress.lastStreakDate === today;
  if (streakAlive) animate('streak-toast', progress.streak);
}
```

- [ ] **Step 2: Verify in browser**

Open `index.html`. In Console, set a milestone streak and trigger renderProgress():
```js
progress.streak = 7;
progress.lastStreakDate = getTodayString();
localStorage.removeItem('shown-toasts'); // reset so toast fires again
renderProgress();
```
Expected: amber "🔥 7-day streak!" toast slides in from top-right, disappears after 3s.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: streak milestone toast in renderProgress"
```

---

## Task 13: Bump SW Cache + Final Push

**Files:**
- Modify: `sw.js` — bump cache version from `sql-trainer-v10` to `sql-trainer-v11`

- [ ] **Step 1: Bump cache version**

In `sw.js`, find:
```js
const CACHE = 'sql-trainer-v10';
```

Replace with:
```js
const CACHE = 'sql-trainer-v11';
```

- [ ] **Step 2: Final browser smoke test**

Open `index.html` and verify the following checklist:
- [ ] Header level badge shows correct color and name
- [ ] Streak badge glows amber if streak ≥ 3
- [ ] Progress tab shows segmented XP bar with colored glowing filled segments
- [ ] Cards have subtle top-edge highlight
- [ ] Focused textarea has cyan glow ring
- [ ] Active nav tab has glow text effect and slight scale
- [ ] Bottom nav has correct safe-area padding on iPhone (check in DevTools device emulation)
- [ ] Submitting correct answer triggers green glow pulse on card
- [ ] Submitting wrong answer triggers horizontal shake on card
- [ ] `animate('session-complete')` in console shows overlay flash
- [ ] `animate('level-up', 3)` in console shows green level-up modal
- [ ] Offline pill appears when network is offline

- [ ] **Step 3: Commit and push**

```bash
git add index.html sw.js
git commit -m "feat: bump SW cache to v11 for Phase 1 frontend upgrade"
git push origin main
```

---

## Self-Review Notes

**Spec coverage check:**
- [x] Glow tokens — Task 2
- [x] XP bar — Task 5
- [x] Level badges — Task 4
- [x] Streak flame — Task 4
- [x] Card depth — Task 3
- [x] Typography — Task 3
- [x] iOS textarea zoom fix — Task 3
- [x] Touch targets — already ≥ 44px per existing CSS (nav: 56px, run-btn: 48px, collapsible-btn: 48px) — no change needed
- [x] Nav active state — Task 3
- [x] PWA install banner — Task 6
- [x] Offline indicator — Task 6
- [x] Auto-scroll — Task 7
- [x] Safe area insets — Task 3
- [x] CSS keyframes — Task 8
- [x] animate() dispatcher — Task 9
- [x] correct-pulse + shake — Task 10
- [x] session-complete overlay — Task 11
- [x] level-up modal — Task 11
- [x] streak toast — Task 12
- [x] prefers-reduced-motion guard — Task 9 (animate() returns early)
- [x] UI/UX Designer agent — Task 1
- [x] SW cache bump — Task 13
