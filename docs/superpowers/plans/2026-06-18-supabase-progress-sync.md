# Supabase Progress Sync Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Auto-sync SQL Trainer progress to Supabase so it survives cache clears and restores automatically on any device using a user-chosen username.

**Architecture:** Supabase JS v2 (CDN) is added to `index.html`. `loadProgress()` becomes async with a Supabase fallback when localStorage is empty. `saveProgress()` fires a background upsert after every local save. A one-time username prompt blocks app init on first visit. All changes stay inside the single `index.html` file.

**Tech Stack:** Vanilla JS · Supabase JS v2 (CDN) · localStorage · GitHub Pages

**Spec:** `docs/superpowers/specs/2026-06-18-supabase-progress-sync-design.md`

---

## Prerequisites (do before Task 1)

Two values are needed before touching any code:

**1. Supabase anon key**
- Go to `https://supabase.com/dashboard/project/vxbylefdrmcdbngvpjnt/settings/api`
- Copy the `anon` `public` key (starts with `eyJ...`)
- You'll paste it into Task 2

**2. Supabase CDN URL + SRI hash**
- Visit `https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2` in a browser — note the exact resolved version (e.g. `2.49.1`)
- Go to `https://www.srihash.org/`, enter the full URL: `https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2.X.X/dist/umd/supabase.min.js`
- Copy the `integrity` hash (sha384-...)
- You'll use both in Task 2

---

## File Map

| File | Change |
|------|--------|
| `index.html` | Add Supabase SDK `<script>` tag; add client constants, username modal HTML+CSS, sync pill HTML, updated `loadProgress`/`saveProgress`/reset/`initApp`, sync section in Progress tab |
| `sw.js` | Bump cache to `sql-trainer-v12`, add Supabase CDN URL to ASSETS |

---

## Task 1: Create Supabase Table

**Files:** None — run SQL in Supabase dashboard

- [ ] **Step 1: Open Supabase SQL editor**

Go to `https://supabase.com/dashboard/project/vxbylefdrmcdbngvpjnt/sql/new`

- [ ] **Step 2: Run table creation SQL**

Paste and run:

```sql
CREATE TABLE IF NOT EXISTS sql_trainer_progress (
  user_id       text PRIMARY KEY,
  progress_json text NOT NULL,
  updated_at    timestamptz DEFAULT now()
);

ALTER TABLE sql_trainer_progress ENABLE ROW LEVEL SECURITY;

CREATE POLICY "allow_all" ON sql_trainer_progress
  FOR ALL USING (true) WITH CHECK (true);
```

- [ ] **Step 3: Verify**

In the Supabase Table Editor, confirm `sql_trainer_progress` appears with columns `user_id`, `progress_json`, `updated_at`.

---

## Task 2: Add Supabase SDK + Client Constants to index.html

**Files:**
- Modify: `C:\Project\sql-trainer\index.html` — `<head>` block and top of `<script>` block

- [ ] **Step 1: Add Supabase CDN script tag to `<head>`**

Find:
```html
</head>
```

Replace with (substitute real version and hash from Prerequisites):
```html
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2.X.X/dist/umd/supabase.min.js"
        integrity="sha384-REPLACE_WITH_REAL_HASH"
        crossorigin="anonymous"></script>
</head>
```

- [ ] **Step 2: Add client constants at the top of the `<script>` block**

Find:
```js
const STORAGE_KEY = 'sql_trainer_v2';
```

Replace with:
```js
const STORAGE_KEY = 'sql_trainer_v2';
const USER_ID_KEY = 'sql_trainer_user_id';
const SUPABASE_URL = 'https://vxbylefdrmcdbngvpjnt.supabase.co';
const SUPABASE_ANON_KEY = 'REPLACE_WITH_REAL_ANON_KEY';
const _sb = window.supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
```

- [ ] **Step 3: Verify in browser console**

Open `index.html` in browser. In DevTools Console:
```js
_sb.from('sql_trainer_progress').select('*').limit(1).then(r => console.log(r))
```
Expected: `{ data: [], error: null }` (empty table, no error).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add Supabase SDK and client constants"
```

---

## Task 3: Username Prompt Modal — HTML + CSS

**Files:**
- Modify: `C:\Project\sql-trainer\index.html` — add modal HTML after `<body>`, add CSS before `</style>`

- [ ] **Step 1: Add modal HTML immediately after `<body>` tag**

Find:
```html
<body>

<div id="header">
```

Replace with:
```html
<body>

<!-- Username modal — shown on first visit if no USER_ID_KEY in localStorage -->
<div id="username-modal" style="display:none">
  <div class="username-box">
    <h2>Welcome to SQL Trainer ⚡</h2>
    <p>Choose a username to save your progress across devices. Pick something unique.</p>
    <input id="username-input" type="text" placeholder="e.g. dello123" maxlength="30" autocomplete="off" />
    <button id="username-submit" disabled>Start Training</button>
  </div>
</div>

<div id="header">
```

- [ ] **Step 2: Add modal CSS before `</style>`**

Find `</style>` and insert immediately before it:

```css
    /* ── Username modal ───────────────────────────────────── */
    #username-modal {
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,0.85);
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 1000;
    }
    .username-box {
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: 16px;
      padding: 32px 24px;
      max-width: 340px;
      width: 90%;
      text-align: center;
    }
    .username-box h2 {
      font-size: 20px;
      font-weight: 800;
      color: var(--accent);
      margin-bottom: 10px;
    }
    .username-box p {
      font-size: 14px;
      color: var(--text-dim);
      margin-bottom: 18px;
      line-height: 1.5;
    }
    #username-input {
      width: 100%;
      background: var(--code-bg);
      border: 1px solid var(--border);
      border-radius: 8px;
      color: var(--text);
      font-size: 16px;
      padding: 12px;
      margin-bottom: 14px;
      outline: none;
      text-align: center;
      box-sizing: border-box;
    }
    #username-input:focus { border-color: var(--accent); box-shadow: var(--glow-cyan); }
    #username-submit {
      width: 100%;
      background: var(--accent);
      color: #000;
      border: none;
      border-radius: 8px;
      padding: 13px;
      font-size: 15px;
      font-weight: 700;
      cursor: pointer;
      transition: opacity 0.2s;
    }
    #username-submit:disabled { opacity: 0.35; cursor: not-allowed; }

    /* ── Sync pill ────────────────────────────────────────── */
    #sync-pill {
      font-size: 11px;
      font-weight: 600;
      border-radius: 20px;
      padding: 2px 8px;
      margin-left: 4px;
      display: none;
    }
    #sync-pill.saving {
      display: inline;
      color: var(--text-dim);
      background: var(--surface2);
      border: 1px solid var(--border);
    }
    #sync-pill.error {
      display: inline;
      color: var(--warning);
      background: rgba(245,158,11,0.1);
      border: 1px solid rgba(245,158,11,0.4);
    }

    /* ── Sync info in Progress tab ────────────────────────── */
    .sync-info {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 12px 0 4px;
      border-top: 1px solid var(--border);
      margin-top: 8px;
      font-size: 13px;
      color: var(--text-dim);
    }
    .sync-info strong { color: var(--text); }
    #change-username-btn {
      background: none;
      border: 1px solid var(--border);
      color: var(--text-dim);
      border-radius: 6px;
      padding: 4px 10px;
      font-size: 12px;
      cursor: pointer;
    }
```

- [ ] **Step 3: Add sync pill to header**

Find:
```html
    <span class="badge streak" id="hdr-streak">🔥 0</span>
  </div>
</div>
```

Replace with:
```html
    <span class="badge streak" id="hdr-streak">🔥 0</span>
    <span id="sync-pill"></span>
  </div>
</div>
```

- [ ] **Step 4: Verify modal CSS renders**

Open `index.html`. In DevTools Console:
```js
document.getElementById('username-modal').style.display = 'flex';
```
Expected: a centered dark overlay with "Welcome to SQL Trainer ⚡" heading, input, and disabled button appears.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: username modal and sync pill HTML + CSS"
```

---

## Task 4: Username Modal JS Logic

**Files:**
- Modify: `C:\Project\sql-trainer\index.html` — add JS functions near `// ─── INIT` section

- [ ] **Step 1: Add username modal JS before `// ─── INIT` comment**

Find:
```js
// ─── INIT ────────────────────────────────────────────────────────────────────

renderToday();
```

Replace with:
```js
// ─── USERNAME MODAL ──────────────────────────────────────────────────────────

function showUsernameModal() {
  const modal = document.getElementById('username-modal');
  modal.style.display = 'flex';
  const input = document.getElementById('username-input');
  const btn = document.getElementById('username-submit');
  input.addEventListener('input', () => {
    btn.disabled = input.value.trim().length < 3;
  });
  btn.addEventListener('click', () => {
    const val = input.value.trim();
    if (val.length < 3) return;
    localStorage.setItem(USER_ID_KEY, val);
    modal.style.display = 'none';
    initApp();
  });
  input.addEventListener('keydown', e => {
    if (e.key === 'Enter' && !btn.disabled) btn.click();
  });
  setTimeout(() => input.focus(), 50);
}

function showSyncPill(state) {
  const pill = document.getElementById('sync-pill');
  if (!pill) return;
  pill.className = state;
  pill.textContent = state === 'saving' ? '↑ Saving' : '⚠ Offline';
}

function hideSyncPill() {
  const pill = document.getElementById('sync-pill');
  if (!pill) return;
  setTimeout(() => { pill.className = ''; pill.textContent = ''; }, 1500);
}

// ─── INIT ────────────────────────────────────────────────────────────────────

renderToday();
```

- [ ] **Step 2: Verify in browser console**

Open `index.html`. Run:
```js
showUsernameModal();
```
Expected: modal appears. Type 2 chars — button still disabled. Type 3+ chars — button enabled. Press Enter or click → modal hides, `localStorage.getItem('sql_trainer_user_id')` returns the typed value.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: username modal JS logic and sync pill helpers"
```

---

## Task 5: Async loadProgress() with Supabase Fallback

**Files:**
- Modify: `C:\Project\sql-trainer\index.html` — `loadProgress()` at line ~2869, `let progress` at line ~2881

- [ ] **Step 1: Replace loadProgress() with async version**

Find:
```js
function loadProgress() {
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (raw) return Object.assign({}, DEFAULT_PROGRESS, JSON.parse(raw));
  } catch(e) {}
  return Object.assign({}, DEFAULT_PROGRESS);
}
```

Replace with:
```js
async function loadProgress() {
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (raw) {
      const p = Object.assign({}, DEFAULT_PROGRESS, JSON.parse(raw));
      if (p.completedSessions > 0 || p.level > 1 || p.completedQuestions.length > 0) {
        return p;
      }
    }
  } catch(e) {}

  // Fallback: fetch from Supabase when localStorage has no real progress
  const userId = localStorage.getItem(USER_ID_KEY);
  if (userId) {
    try {
      const { data } = await _sb
        .from('sql_trainer_progress')
        .select('progress_json')
        .eq('user_id', userId)
        .single();
      if (data?.progress_json) {
        const p = Object.assign({}, DEFAULT_PROGRESS, JSON.parse(data.progress_json));
        localStorage.setItem(STORAGE_KEY, data.progress_json);
        return p;
      }
    } catch(e) {}
  }

  return Object.assign({}, DEFAULT_PROGRESS);
}
```

- [ ] **Step 2: Change `let progress` declaration to use DEFAULT_PROGRESS directly**

Find:
```js
let progress = loadProgress();
```

Replace with:
```js
let progress = Object.assign({}, DEFAULT_PROGRESS); // set properly by initApp()
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: async loadProgress with Supabase fallback"
```

---

## Task 6: Wrap App Init in async initApp()

**Files:**
- Modify: `C:\Project\sql-trainer\index.html` — `// ─── INIT` section

- [ ] **Step 1: Replace the bare `renderToday()` call with `initApp()`**

Find:
```js
// ─── INIT ────────────────────────────────────────────────────────────────────

renderToday();
```

Replace with:
```js
// ─── INIT ────────────────────────────────────────────────────────────────────

async function initApp() {
  const userId = localStorage.getItem(USER_ID_KEY);
  if (!userId) {
    showUsernameModal();
    return; // modal submit calls initApp() again after saving username
  }
  progress = await loadProgress();
  updateHeader();
  renderProgress();
  renderToday();
}

initApp();
```

- [ ] **Step 2: Verify full init flow in browser**

Open `index.html` with no `sql_trainer_user_id` in localStorage (open DevTools → Application → Local Storage → delete the key, then reload). Expected: username modal appears.

Enter a username (e.g. `testuser`), click Start Training. Expected: modal closes, app renders normally with Today tab.

Reload page. Expected: no modal, app loads directly.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: async initApp() wraps app startup with username check"
```

---

## Task 7: Background Supabase Upsert in saveProgress()

**Files:**
- Modify: `C:\Project\sql-trainer\index.html` — `saveProgress()` at line ~2877

- [ ] **Step 1: Replace saveProgress() with background-sync version**

Find:
```js
function saveProgress(p) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(p));
}
```

Replace with:
```js
function saveProgress(p) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(p));
  _syncToSupabase(p);
}

async function _syncToSupabase(p) {
  const userId = localStorage.getItem(USER_ID_KEY);
  if (!userId) return;
  showSyncPill('saving');
  try {
    const { error } = await _sb
      .from('sql_trainer_progress')
      .upsert({ user_id: userId, progress_json: JSON.stringify(p), updated_at: new Date().toISOString() });
    if (error) throw error;
    hideSyncPill();
  } catch(e) {
    showSyncPill('error');
    setTimeout(() => hideSyncPill(), 4000);
  }
}
```

- [ ] **Step 2: Verify sync in browser**

Open `index.html`. Submit a correct answer. Watch the header — a dim "↑ Saving" pill should briefly appear and disappear. Then in Supabase Table Editor, check `sql_trainer_progress` — a row with your username should appear.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: background Supabase upsert in saveProgress"
```

---

## Task 8: Delete Supabase Row on Reset

**Files:**
- Modify: `C:\Project\sql-trainer\index.html` — reset button listener at line ~4682

- [ ] **Step 1: Update reset listener to also delete Supabase row**

Find:
```js
document.getElementById('reset-btn').addEventListener('click', () => {
  if (window.confirm('Reset all progress? This cannot be undone.')) {
    localStorage.removeItem(STORAGE_KEY);
    progress = Object.assign({}, DEFAULT_PROGRESS);
    renderToday();
    renderProgress();
    updateHeader();
    switchTab('today');
  }
});
```

Replace with:
```js
document.getElementById('reset-btn').addEventListener('click', () => {
  if (window.confirm('Reset all progress? This cannot be undone.')) {
    localStorage.removeItem(STORAGE_KEY);
    progress = Object.assign({}, DEFAULT_PROGRESS);
    const userId = localStorage.getItem(USER_ID_KEY);
    if (userId) _sb.from('sql_trainer_progress').delete().eq('user_id', userId);
    renderToday();
    renderProgress();
    updateHeader();
    switchTab('today');
  }
});
```

- [ ] **Step 2: Verify in browser**

Open `index.html`. Click "Reset all progress" and confirm. Then check Supabase Table Editor — the row for your username should be gone.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: delete Supabase row on progress reset"
```

---

## Task 9: Sync Info Section in Progress Tab

**Files:**
- Modify: `C:\Project\sql-trainer\index.html` — Progress tab HTML (~line 626), `renderProgress()` (~line 4577)

- [ ] **Step 1: Add sync info HTML to Progress tab**

Find:
```html
    <button class="reset-btn" id="reset-btn">Reset all progress</button>
  </div>
```

Replace with:
```html
    <button class="reset-btn" id="reset-btn">Reset all progress</button>
    <div class="sync-info">
      <span>Syncing as: <strong id="sync-username">—</strong></span>
      <button id="change-username-btn">Change</button>
    </div>
  </div>
```

- [ ] **Step 2: Populate sync-username and wire Change button in renderProgress()**

Find the end of `renderProgress()`:
```js
  document.getElementById('concept-list').innerHTML = html;

  // Streak milestone toast
  const today = getTodayString();
  const streakAlive = progress.lastStreakDate === today;
  if (streakAlive) animate('streak-toast', progress.streak);
}
```

Replace with:
```js
  document.getElementById('concept-list').innerHTML = html;

  // Streak milestone toast
  const today = getTodayString();
  const streakAlive = progress.lastStreakDate === today;
  if (streakAlive) animate('streak-toast', progress.streak);

  // Sync info
  const syncUsernameEl = document.getElementById('sync-username');
  if (syncUsernameEl) syncUsernameEl.textContent = localStorage.getItem(USER_ID_KEY) || '—';
  const changeBtn = document.getElementById('change-username-btn');
  if (changeBtn && !changeBtn._wired) {
    changeBtn._wired = true;
    changeBtn.addEventListener('click', () => {
      if (window.confirm('Change username? Your current progress will remain in Supabase under the old username.')) {
        localStorage.removeItem(USER_ID_KEY);
        location.reload();
      }
    });
  }
}
```

- [ ] **Step 3: Verify in browser**

Open `index.html` → Progress tab. Should show "Syncing as: **yourusername**" at the bottom. Click "Change" → confirm dialog → page reloads → username modal appears.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: sync info section in Progress tab with Change username"
```

---

## Task 10: Bump SW Cache to v12 + Add Supabase CDN to ASSETS

**Files:**
- Modify: `C:\Project\sql-trainer\sw.js`

- [ ] **Step 1: Bump cache version and add Supabase CDN to ASSETS**

Find:
```js
const CACHE = 'sql-trainer-v11';
const ASSETS = [
  './index.html',
  'https://cdn.jsdelivr.net/npm/alasql@4.2.3/dist/alasql.min.js'
];
```

Replace with (use the exact pinned version URL from Prerequisites):
```js
const CACHE = 'sql-trainer-v12';
const ASSETS = [
  './index.html',
  'https://cdn.jsdelivr.net/npm/alasql@4.2.3/dist/alasql.min.js',
  'https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2.X.X/dist/umd/supabase.min.js'
];
```

- [ ] **Step 2: Final smoke test**

Open `index.html`. Open DevTools → Application → Clear site data (check all boxes including localStorage). Reload.

Expected sequence:
1. Username modal appears
2. Enter username → app loads fresh (default progress)
3. Submit one correct answer → "↑ Saving" pill flashes in header
4. Open Supabase Table Editor → row appears for your username
5. Clear site data again (localStorage only — uncheck "Cache storage")
6. Reload → username modal appears with same username pre-filled? No — modal appears again since USER_ID_KEY is gone. Enter same username → app loads → progress restores from Supabase automatically

- [ ] **Step 3: Commit and push**

```bash
git add index.html sw.js
git commit -m "feat: bump SW cache to v12, add Supabase CDN to SW assets"
git push origin main
```

---

## Self-Review

**Spec coverage:**
- [x] Supabase table — Task 1
- [x] SDK + anon key — Task 2
- [x] Username prompt modal HTML/CSS — Task 3
- [x] Username modal JS (show, submit, Enter key) — Task 4
- [x] `loadProgress()` async with Supabase fallback — Task 5
- [x] `let progress` init fix — Task 5
- [x] `initApp()` async wrapper — Task 6
- [x] `saveProgress()` background upsert — Task 7
- [x] Sync pill (saving/error states) — Tasks 3 + 4
- [x] Reset deletes Supabase row — Task 8
- [x] Progress tab sync info + Change button — Task 9
- [x] SW cache bump + Supabase CDN in ASSETS — Task 10
- [x] Offline behavior (silent fail in `_syncToSupabase`) — Task 7
- [x] `prefers-reduced-motion` — N/A (no animations added)
