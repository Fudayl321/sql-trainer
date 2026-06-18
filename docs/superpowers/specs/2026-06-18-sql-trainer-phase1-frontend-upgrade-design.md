# SQL Trainer — Phase 1 Frontend Upgrade Design

**Date:** 2026-06-18  
**Project:** SQL Trainer PWA  
**Phase:** 1 of 2 — Visual Polish + Mobile Experience  
**Approach:** Option B — CSS overhaul + JS animation layer (single-file architecture preserved)

---

## Scope

Phase 1 upgrades the visual design and mobile experience of the existing SQL Trainer PWA (`index.html`). Game logic, question content, and tab structure are untouched. Phase 2 (UX flow + new features) is a separate spec.

**In scope:**
1. Visual design system — glow tokens, XP bar, level badges, streak flame, card depth, typography
2. Mobile experience — touch targets, iOS textarea zoom fix, PWA install banner, offline indicator, safe area insets, auto-scroll
3. JS animation layer — correct/wrong answer feedback, session complete overlay, level-up modal, streak milestone toast
4. UI/UX Designer agent — new squad member in `.claude/agents/`

**Out of scope (Phase 2):**
- Progress tab charts/graphs
- Achievement badge collection screen
- UX flow improvements (hint system, session navigation)
- Question content changes

---

## 1. Visual Design System

### CSS Variable Upgrades

Add to existing `:root` block — no existing variables removed:

```css
--glow-cyan: 0 0 12px rgba(0, 212, 255, 0.6), 0 0 24px rgba(0, 212, 255, 0.2);
--glow-purple: 0 0 12px rgba(124, 58, 237, 0.6), 0 0 24px rgba(124, 58, 237, 0.2);
--glow-green: 0 0 12px rgba(34, 197, 94, 0.5), 0 0 24px rgba(34, 197, 94, 0.15);
--glow-red: 0 0 12px rgba(239, 68, 68, 0.5);
--glow-amber: 0 0 10px rgba(245, 158, 11, 0.5);

/* Level-specific accent colors */
--level-1: #00d4ff;   /* cyan  — Foundations */
--level-2: #7c3aed;   /* purple — Aggregation */
--level-3: #22c55e;   /* green  — Joins & Subqueries */
--level-4: #f59e0b;   /* amber  — CTEs & Windows */
--level-5: #ef4444;   /* red    — DML & Data Changes */
```

### Glow Application

| Element | Glow token | Trigger |
|---|---|---|
| Focused textarea | `--glow-cyan` | `:focus` |
| Correct answer card | `--glow-green` | `.correct` state class |
| Wrong answer card | `--glow-red` | `.wrong` state class |
| Active nav button | `--glow-cyan` | `.active` class |
| Streak badge (≥3 days) | `--glow-amber` | JS adds `.streak-glow` |

### XP Bar

Replaces the plain level progress bar in the Progress tab. Implementation:
- Segmented bar: one segment per concept in current level (11 segments for Level 1, etc.)
- Filled segments = completed concepts, unfilled = remaining
- CSS `@keyframes xp-fill` plays on page load: segments fill left-to-right over 800ms
- Color matches `--level-N` variable for current level

### Level Badges

Pill badge in Today tab header and Progress tab showing `Lv N · Level Name`. Styled with:
- Background: 15% opacity of `--level-N` color
- Border: 1px solid `--level-N` at 40% opacity
- Box-shadow: `--glow-N` at reduced intensity (30%)

### Streak Flame

Streak counter in Progress tab:
- Prefix flame emoji `🔥` on streaks ≥ 1
- `.streak-glow` class adds `--glow-amber` at milestones: 3, 7, 14 days
- Milestone thresholds checked in existing `renderProgress()` — add class conditionally

### Card Depth

All `.card` elements get:
```css
background: linear-gradient(180deg, rgba(255,255,255,0.03) 0%, transparent 60%), var(--surface);
```
Adds a subtle top-edge highlight — no color change, just perceived lift.

### Typography

- `h2`, `h3` headings: `font-size` bumped one step (e.g., 1rem → 1.1rem)
- Nav tab labels: `letter-spacing: 0.03em`
- Code blocks: `border-left: 2px solid rgba(0, 212, 255, 0.4)` accent

---

## 2. Mobile Experience

### Touch Targets

All interactive elements audited to meet 44px minimum tap height:
- `.nav-btn` — raise to `min-height: 52px`
- `.accordion-toggle` — raise to `min-height: 44px`
- `.btn` (submit, hint) — raise to `min-height: 44px`

### iOS Textarea Zoom Fix

```css
textarea, input[type="text"] {
  font-size: 16px; /* prevents iOS auto-zoom on focus */
}
```

### Bottom Nav Active State

Active tab:
```css
.nav-btn.active {
  border-bottom: 2px solid var(--accent);
  box-shadow: var(--glow-cyan);
  transform: scale(1.08);
  transition: transform 0.25s ease, box-shadow 0.25s ease;
}
```

### PWA Install Banner

- Listens for `beforeinstallprompt` event, stores it
- On first visit (no `pwa-dismissed` key in localStorage): renders slim banner at top of `<main>`
- Banner: "Install SQL Trainer as an app" + Install button + × dismiss button
- Install tap: calls `prompt()` on stored event, sets `pwa-dismissed = true`
- Dismiss tap: sets `pwa-dismissed = true`, removes banner
- Does not appear if already installed or dismissed

### Offline Indicator

- `window.addEventListener('online/offline')` listeners
- Offline: injects `<div id="offline-pill">Offline</div>` into header, styled amber pill
- Online: removes pill
- No modal, no alert — silent status only

### Auto-Scroll After Answer

After `submitAnswer()` resolves, call:
```js
document.querySelector('.feedback-block')?.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
```

### Safe Area Insets

```css
.bottom-nav {
  padding-bottom: calc(8px + env(safe-area-inset-bottom));
}
```

---

## 3. JS Animation Layer

Single `animate(type, targetEl)` dispatcher added to JS section. All animations guarded:

```js
if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return;
```

### Animations

| Trigger | Type | Duration | Behavior |
|---|---|---|---|
| Correct answer | `correct-pulse` | 600ms | Green glow pulse on answer card, checkmark flash on submit btn |
| Wrong answer | `shake` | 300ms | Horizontal shake on card, red glow fade-in |
| Session complete (Q5 pass) | `session-complete` | 800ms | Fullscreen overlay: level badge + "Session Complete · Concept complete!", auto-dismisses |
| Level up | `level-up` | — | Modal panel: new level name + glow border + "Level Up!". User taps to dismiss |
| Streak milestone (3/7/14) | `streak-toast` | 3000ms | Toast slides from top-right: "🔥 N-day streak!", auto-dismisses |

### CSS Keyframes Added

```css
@keyframes correct-pulse {
  0%   { box-shadow: var(--glow-green); }
  100% { box-shadow: none; }
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25%       { transform: translateX(-6px); }
  75%       { transform: translateX(6px); }
}

@keyframes xp-fill {
  from { width: 0; }
  to   { width: var(--fill-pct); }
}

@keyframes slide-in-right {
  from { transform: translateX(110%); opacity: 0; }
  to   { transform: translateX(0);    opacity: 1; }
}
```

### Integration Points

- `submitAnswer()` — calls `animate('correct-pulse' | 'shake', cardEl)` after evaluation
- `completeSession()` — calls `animate('session-complete')` before gate prompt
- `loadProgress()` / level increment detection — calls `animate('level-up')`
- `renderProgress()` — calls `animate('streak-toast')` when milestone threshold crossed for first time (guarded by `shownToasts` Set in localStorage)

---

## 4. UI/UX Designer Agent

**File locations:**
- Project: `C:\Project\.claude\agents\ui-ux-designer.md`
- Global: `C:\Users\User\.claude\agents\ui-ux-designer.md`

**Squad position:** PM → **UI/UX Designer** → Coder → DevOps → Tester → Reviewer → Consultant

**Responsibilities:**
- Receives PM requirements, produces a component spec with exact CSS values, animation timings, and markup structure
- Defines accessibility requirements: contrast ratios, ARIA labels for dynamic elements
- Reviews Coder output for visual consistency (before Tester)
- Produces specs only — writes no implementation code

**Agent tools:** Read, Grep, Glob, Write (spec files only)

---

## Implementation Notes

- Single-file architecture (`index.html`) preserved — no new files except the agent definition
- All CSS changes go into the existing `<style>` block
- All JS changes go into the existing `<script>` block
- SW cache bumped to `sql-trainer-v11` after implementation
- No external libraries added

---

## Success Criteria

- [ ] Glow effects visible on focused textarea, correct answers, active nav tab
- [ ] XP bar renders in Progress tab with fill animation
- [ ] Level badge shows in Today header
- [ ] Streak flame appears with glow at milestone days
- [ ] All touch targets ≥ 44px on mobile
- [ ] iOS textarea no longer auto-zooms
- [ ] PWA install banner appears on fresh visit (if installable)
- [ ] Offline pill appears/disappears correctly
- [ ] Correct answer triggers glow pulse animation
- [ ] Wrong answer triggers shake animation
- [ ] Session complete overlay shows after Q5 pass
- [ ] Level-up modal shows on level increment
- [ ] Streak toast shows at 3/7/14 day milestones
- [ ] All animations skipped when `prefers-reduced-motion` is set
- [ ] UI/UX Designer agent file created and functional
