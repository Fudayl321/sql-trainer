# Login & Auth Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the username-only modal with a real login/register system backed by Supabase Auth (username + password), with logout buttons in the header and Progress tab.

**Architecture:** Supabase Auth handles all auth (`signUp`, `signInWithPassword`, `signOut`). Username is mapped to `username@sql-trainer.app` as the email internally — the user only ever sees/types their username. `sql_trainer_progress` switches from `user_id TEXT` (username string) to `user_id UUID` (Supabase `auth.uid()`). Session is persisted automatically by the Supabase client in localStorage. On page load, `getSession()` determines whether to show the login screen or load the app.

**Tech Stack:** Vanilla JS · Supabase JS SDK v2 (already in CDN) · Single-file PWA (`index.html`)

**Spec:** `docs/superpowers/specs/2026-06-19-login-auth-design.md`

---

## Task 1: Supabase Setup (manual — run in Supabase dashboard)

**Files:** None (Supabase dashboard changes)

- [ ] **Step 1: Recreate the progress table**

Go to `https://supabase.com/dashboard/project/fzpohuurshpakcnshafpt/sql` and run:

```sql
DROP TABLE IF EXISTS sql_trainer_progress;

CREATE TABLE sql_trainer_progress (
  user_id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  progress_json TEXT NOT NULL,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE sql_trainer_progress ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can read/write own row" ON sql_trainer_progress
  FOR ALL
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);
```

- [ ] **Step 2: Disable email confirmation**

Go to Supabase Dashboard → Authentication → Settings → Email → toggle **"Confirm email"** OFF. This is required because we use fake `@sql-trainer.app` emails — users cannot confirm via inbox.

- [ ] **Step 3: Verify**

In the SQL Editor, run:
```sql
SELECT column_name, data_type FROM information_schema.columns WHERE table_name = 'sql_trainer_progress';
```
Expected: `user_id` shows `uuid`, `progress_json` shows `text`, `updated_at` shows `timestamp with time zone`.

---

## Task 2: Replace Username Modal HTML with Auth Modal

**Files:**
- Modify: `sql-trainer/index.html` (lines 897–905 — username modal HTML)

- [ ] **Step 1: Replace the username modal div**

Find:
```html
<!-- Username modal — shown on first visit if no USER_ID_KEY in localStorage -->
<div id="username-modal" style="display:none">
  <div class="username-box">
    <h2>Welcome to SQL Trainer ⚡</h2>
    <p>Choose a username to save your progress across devices. Pick something unique.</p>
    <input id="username-input" type="text" placeholder="e.g. dello123" maxlength="30" autocomplete="off" />
    <button id="username-submit" disabled>Start Training</button>
  </div>
</div>
```

Replace with:
```html
<!-- Auth modal — shown when no Supabase session exists -->
<div id="auth-modal" style="display:none">
  <div class="auth-box">
    <h2>⚡ SQL Trainer</h2>
    <div class="auth-tabs">
      <button class="auth-tab-btn active" id="tab-register-btn">Register</button>
      <button class="auth-tab-btn" id="tab-login-btn">Login</button>
    </div>
    <div id="auth-register-form">
      <input id="reg-username" type="text" placeholder="Username (min 3 chars)" maxlength="30" autocomplete="off" />
      <input id="reg-password" type="password" placeholder="Password (min 6 chars)" maxlength="100" autocomplete="new-password" />
      <input id="reg-confirm" type="password" placeholder="Confirm password" maxlength="100" autocomplete="new-password" />
      <button id="reg-submit" disabled>Create Account</button>
      <div class="auth-error" id="reg-error"></div>
    </div>
    <div id="auth-login-form" style="display:none">
      <input id="login-username" type="text" placeholder="Username" maxlength="30" autocomplete="off" />
      <input id="login-password" type="password" placeholder="Password" maxlength="100" autocomplete="current-password" />
      <button id="login-submit" disabled>Login</button>
      <div class="auth-error" id="login-error"></div>
    </div>
  </div>
</div>
```

- [ ] **Step 2: Add logout button to header**

Find:
```html
<div id="header">
  <span class="app-name">⚡ SQL Trainer</span>
  <div class="badges">
    <span class="badge level" id="hdr-level">Lvl 1</span>
    <span class="badge streak" id="hdr-streak">🔥 0</span>
    <span id="sync-pill"></span>
  </div>
</div>
```

Replace with:
```html
<div id="header">
  <span class="app-name">⚡ SQL Trainer</span>
  <div class="badges">
    <span class="badge level" id="hdr-level">Lvl 1</span>
    <span class="badge streak" id="hdr-streak">🔥 0</span>
    <span id="sync-pill"></span>
    <button id="logout-header-btn" aria-label="Logout">⏻</button>
  </div>
</div>
```

- [ ] **Step 3: Replace Change button with Logout in Progress tab**

Find:
```html
    <div class="sync-info">
      <span>Syncing as: <strong id="sync-username">—</strong></span>
      <button id="change-username-btn">Change</button>
    </div>
```

Replace with:
```html
    <div class="sync-info">
      <span>Syncing as: <strong id="sync-username">—</strong></span>
      <button id="logout-progress-btn">Logout</button>
    </div>
```

- [ ] **Step 4: Commit**

```bash
git add sql-trainer/index.html
git commit -m "feat: auth modal HTML, logout buttons in header and progress tab"
```

---

## Task 3: Replace Username Modal CSS with Auth Modal CSS

**Files:**
- Modify: `sql-trainer/index.html` (lines 788–889 — username modal and sync-info CSS)

- [ ] **Step 1: Replace username modal CSS**

Find:
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
```

Replace with:
```css
    /* ── Auth modal ─────────────────────────────────────────── */
    #auth-modal {
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,0.85);
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 1000;
    }
    .auth-box {
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: 16px;
      padding: 32px 24px;
      max-width: 340px;
      width: 90%;
      text-align: center;
    }
    .auth-box h2 {
      font-size: 22px;
      font-weight: 800;
      color: var(--accent);
      margin-bottom: 18px;
    }
    .auth-tabs {
      display: flex;
      gap: 8px;
      margin-bottom: 20px;
    }
    .auth-tab-btn {
      flex: 1;
      background: var(--surface2);
      border: 1px solid var(--border);
      border-radius: 8px;
      color: var(--text-dim);
      font-size: 14px;
      font-weight: 600;
      padding: 8px;
      cursor: pointer;
      transition: all 0.15s;
    }
    .auth-tab-btn.active {
      background: var(--accent);
      color: #000;
      border-color: var(--accent);
    }
    .auth-box input {
      width: 100%;
      background: var(--code-bg);
      border: 1px solid var(--border);
      border-radius: 8px;
      color: var(--text);
      font-size: 16px;
      padding: 12px;
      margin-bottom: 10px;
      outline: none;
      box-sizing: border-box;
    }
    .auth-box input:focus { border-color: var(--accent); box-shadow: var(--glow-cyan); }
    #reg-submit, #login-submit {
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
      margin-top: 4px;
    }
    #reg-submit:disabled, #login-submit:disabled { opacity: 0.35; cursor: not-allowed; }
    .auth-error {
      color: var(--error);
      font-size: 13px;
      margin-top: 10px;
      min-height: 18px;
    }
```

- [ ] **Step 2: Replace #change-username-btn CSS with logout button styles**

Find:
```css
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

Replace with:
```css
    .sync-info strong { color: var(--text); }
    #logout-progress-btn {
      background: none;
      border: 1px solid var(--border);
      color: var(--text-dim);
      border-radius: 6px;
      padding: 4px 10px;
      font-size: 12px;
      cursor: pointer;
    }
    #logout-progress-btn:hover { color: var(--error); border-color: var(--error); }
    #logout-header-btn {
      background: none;
      border: none;
      color: var(--text-dim);
      font-size: 16px;
      cursor: pointer;
      padding: 2px 6px;
      line-height: 1;
      transition: color 0.15s;
    }
    #logout-header-btn:hover { color: var(--error); }
```

- [ ] **Step 3: Verify in browser**

Open `index.html` in browser. Open DevTools Console and run:
```js
document.getElementById('auth-modal').style.display = 'flex';
```
Expected: the auth modal appears with Register/Login tabs, input fields, and a "Create Account" button.

- [ ] **Step 4: Commit**

```bash
git add sql-trainer/index.html
git commit -m "feat: auth modal CSS and logout button styles"
```

---

## Task 4: Remove USER_ID_KEY and Update _syncToSupabase + loadProgress

**Files:**
- Modify: `sql-trainer/index.html` (lines 2975–3045 — state constants and sync functions)

- [ ] **Step 1: Remove USER_ID_KEY constant**

Find:
```js
const STORAGE_KEY = 'sql_trainer_v2';
const USER_ID_KEY = 'sql_trainer_user_id';
```

Replace with:
```js
const STORAGE_KEY = 'sql_trainer_v2';
```

- [ ] **Step 2: Update loadProgress() to use auth session**

Find:
```js
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
```

Replace with:
```js
  // Fallback: fetch from Supabase when localStorage has no real progress
  const { data: { session } } = await _sb.auth.getSession();
  if (session) {
    try {
      const { data } = await _sb
        .from('sql_trainer_progress')
        .select('progress_json')
        .eq('user_id', session.user.id)
        .single();
      if (data?.progress_json) {
        const p = Object.assign({}, DEFAULT_PROGRESS, JSON.parse(data.progress_json));
        localStorage.setItem(STORAGE_KEY, data.progress_json);
        return p;
      }
    } catch(e) {}
  }
```

- [ ] **Step 3: Update _syncToSupabase() to use auth session**

Find:
```js
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
    console.error('[SQL Trainer] Supabase sync error:', e);
    showSyncPill('error');
    setTimeout(() => hideSyncPill(), 4000);
  }
}
```

Replace with:
```js
async function _syncToSupabase(p) {
  const { data: { session } } = await _sb.auth.getSession();
  if (!session) return;
  showSyncPill('saving');
  try {
    const { error } = await _sb
      .from('sql_trainer_progress')
      .upsert({ user_id: session.user.id, progress_json: JSON.stringify(p), updated_at: new Date().toISOString() });
    if (error) throw error;
    hideSyncPill();
  } catch(e) {
    console.error('[SQL Trainer] Supabase sync error:', e);
    showSyncPill('error');
    setTimeout(() => hideSyncPill(), 4000);
  }
}
```

- [ ] **Step 4: Update reset-btn listener to use auth session instead of USER_ID_KEY**

Find:
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

Replace with:
```js
document.getElementById('reset-btn').addEventListener('click', async () => {
  if (window.confirm('Reset all progress? This cannot be undone.')) {
    localStorage.removeItem(STORAGE_KEY);
    progress = Object.assign({}, DEFAULT_PROGRESS);
    const { data: { session } } = await _sb.auth.getSession();
    if (session) _sb.from('sql_trainer_progress').delete().eq('user_id', session.user.id);
    renderToday();
    renderProgress();
    updateHeader();
    switchTab('today');
  }
});
```

- [ ] **Step 5: Commit**

```bash
git add sql-trainer/index.html
git commit -m "feat: remove USER_ID_KEY, update sync and loadProgress to use auth session"
```

---

## Task 5: Replace showUsernameModal() with Auth Modal JS + Update initApp()

**Files:**
- Modify: `sql-trainer/index.html` (lines 4910–4958 — username modal JS and initApp)

- [ ] **Step 1: Replace showUsernameModal() and initApp() with auth modal logic**

Find:
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
```

Replace with:
```js
// ─── AUTH MODAL ──────────────────────────────────────────────────────────────

function showAuthModal() {
  const modal = document.getElementById('auth-modal');
  modal.style.display = 'flex';

  // Tab switching
  document.getElementById('tab-register-btn').addEventListener('click', () => {
    document.getElementById('tab-register-btn').classList.add('active');
    document.getElementById('tab-login-btn').classList.remove('active');
    document.getElementById('auth-register-form').style.display = '';
    document.getElementById('auth-login-form').style.display = 'none';
    document.getElementById('reg-error').textContent = '';
    setTimeout(() => document.getElementById('reg-username').focus(), 50);
  });
  document.getElementById('tab-login-btn').addEventListener('click', () => {
    document.getElementById('tab-login-btn').classList.add('active');
    document.getElementById('tab-register-btn').classList.remove('active');
    document.getElementById('auth-login-form').style.display = '';
    document.getElementById('auth-register-form').style.display = 'none';
    document.getElementById('login-error').textContent = '';
    setTimeout(() => document.getElementById('login-username').focus(), 50);
  });

  // Register form validation
  const regUsername = document.getElementById('reg-username');
  const regPassword = document.getElementById('reg-password');
  const regConfirm  = document.getElementById('reg-confirm');
  const regSubmit   = document.getElementById('reg-submit');
  const regError    = document.getElementById('reg-error');
  function validateReg() {
    regSubmit.disabled = !(regUsername.value.trim().length >= 3 &&
      regPassword.value.length >= 6 &&
      regConfirm.value === regPassword.value);
  }
  regUsername.addEventListener('input', validateReg);
  regPassword.addEventListener('input', validateReg);
  regConfirm.addEventListener('input', validateReg);
  regSubmit.addEventListener('click', async () => {
    regError.textContent = '';
    regSubmit.disabled = true;
    regSubmit.textContent = 'Creating…';
    const username = regUsername.value.trim();
    const password = regPassword.value;
    const { error } = await _sb.auth.signUp(
      { email: `${username}@sql-trainer.app`, password },
      { data: { username } }
    );
    if (error) {
      regError.textContent = error.message.includes('already registered')
        ? 'Username already exists. Try logging in.'
        : 'Could not create account. Check your connection.';
      regSubmit.disabled = false;
      regSubmit.textContent = 'Create Account';
    } else {
      modal.style.display = 'none';
      initApp();
    }
  });
  regUsername.addEventListener('keydown', e => { if (e.key === 'Enter') regPassword.focus(); });
  regPassword.addEventListener('keydown', e => { if (e.key === 'Enter') regConfirm.focus(); });
  regConfirm.addEventListener('keydown',  e => { if (e.key === 'Enter' && !regSubmit.disabled) regSubmit.click(); });

  // Login form validation
  const loginUsername = document.getElementById('login-username');
  const loginPassword = document.getElementById('login-password');
  const loginSubmit   = document.getElementById('login-submit');
  const loginError    = document.getElementById('login-error');
  function validateLogin() {
    loginSubmit.disabled = !(loginUsername.value.trim().length >= 3 && loginPassword.value.length >= 1);
  }
  loginUsername.addEventListener('input', validateLogin);
  loginPassword.addEventListener('input', validateLogin);
  loginSubmit.addEventListener('click', async () => {
    loginError.textContent = '';
    loginSubmit.disabled = true;
    loginSubmit.textContent = 'Logging in…';
    const username = loginUsername.value.trim();
    const password = loginPassword.value;
    const { error } = await _sb.auth.signInWithPassword(
      { email: `${username}@sql-trainer.app`, password }
    );
    if (error) {
      loginError.textContent = 'Incorrect username or password.';
      loginSubmit.disabled = false;
      loginSubmit.textContent = 'Login';
    } else {
      modal.style.display = 'none';
      initApp();
    }
  });
  loginUsername.addEventListener('keydown', e => { if (e.key === 'Enter') loginPassword.focus(); });
  loginPassword.addEventListener('keydown', e => { if (e.key === 'Enter' && !loginSubmit.disabled) loginSubmit.click(); });

  setTimeout(() => document.getElementById('reg-username').focus(), 50);
}

async function logout() {
  await _sb.auth.signOut();
  localStorage.removeItem(STORAGE_KEY);
  location.reload();
}
```

- [ ] **Step 2: Update initApp() to use getSession()**

Find:
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
```

Replace with:
```js
// ─── INIT ────────────────────────────────────────────────────────────────────

async function initApp() {
  const { data: { session } } = await _sb.auth.getSession();
  if (!session) {
    showAuthModal();
    return;
  }
  progress = await loadProgress();
  updateHeader();
  renderProgress();
  renderToday();
}
```

- [ ] **Step 3: Commit**

```bash
git add sql-trainer/index.html
git commit -m "feat: auth modal JS with register/login/logout, update initApp to use getSession"
```

---

## Task 6: Update renderProgress() — Username Display and Logout Buttons

**Files:**
- Modify: `sql-trainer/index.html` (lines 4787–4799 — sync info block inside renderProgress)

- [ ] **Step 1: Replace sync-info update code in renderProgress()**

Find:
```js
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
```

Replace with:
```js
  // Sync info
  _sb.auth.getSession().then(({ data: { session } }) => {
    const syncUsernameEl = document.getElementById('sync-username');
    if (syncUsernameEl) syncUsernameEl.textContent = session?.user?.user_metadata?.username || '—';
  });
  const logoutProgressBtn = document.getElementById('logout-progress-btn');
  if (logoutProgressBtn && !logoutProgressBtn._wired) {
    logoutProgressBtn._wired = true;
    logoutProgressBtn.addEventListener('click', () => {
      if (window.confirm('Log out?')) logout();
    });
  }
  const logoutHeaderBtn = document.getElementById('logout-header-btn');
  if (logoutHeaderBtn && !logoutHeaderBtn._wired) {
    logoutHeaderBtn._wired = true;
    logoutHeaderBtn.addEventListener('click', () => {
      if (window.confirm('Log out?')) logout();
    });
  }
```

- [ ] **Step 2: Verify in browser**

Open `index.html` in browser. Open DevTools Console and run:
```js
document.getElementById('auth-modal').style.display = 'flex';
```
- Register tab should be visible with 3 inputs and "Create Account" button
- Click Login tab → switches to username + password inputs
- Click Register tab again → switches back

- [ ] **Step 3: Commit**

```bash
git add sql-trainer/index.html
git commit -m "feat: update renderProgress to show username from auth metadata and wire logout buttons"
```

---

## Task 7: End-to-End Browser Test

No code changes — verify the full flow works.

- [ ] **Step 1: Test registration**

1. Open `index.html` in browser
2. Auth modal should appear (no session)
3. Register tab is active — enter username `testuser1`, password `test123`, confirm `test123`
4. Click "Create Account"
5. Expected: modal closes, app loads normally, header shows "Lvl 1 · Foundations"

- [ ] **Step 2: Verify Supabase row created**

Go to `https://supabase.com/dashboard/project/fzpohuurshpakcnshafpt/editor` → `sql_trainer_progress` table.
After submitting any answer in the app, a row should appear with a UUID `user_id`.

- [ ] **Step 3: Test logout and login**

1. In the Progress tab, click "Logout" → confirm → page reloads to auth modal
2. Switch to Login tab → enter `testuser1` and `test123` → click "Login"
3. Expected: modal closes, app loads, "Syncing as: **testuser1**" shown in Progress tab

- [ ] **Step 4: Test header logout button**

1. While logged in, click the `⏻` button in the top-right header → confirm
2. Expected: returns to auth modal

- [ ] **Step 5: Test wrong password**

1. In Login tab, enter `testuser1` and `wrongpass` → click Login
2. Expected: "Incorrect username or password." shown in red below the button, button re-enables

- [ ] **Step 6: Test duplicate username**

1. In Register tab, try to register `testuser1` again with any password
2. Expected: "Username already exists. Try logging in." shown in red

---

## Task 8: Bump SW Cache + Push

**Files:**
- Modify: `sql-trainer/sw.js` (line 1)

- [ ] **Step 1: Bump cache version**

Find:
```js
const CACHE = 'sql-trainer-v12';
```

Replace with:
```js
const CACHE = 'sql-trainer-v13';
```

- [ ] **Step 2: Commit and push**

```bash
git add sql-trainer/index.html sql-trainer/sw.js
git commit -m "feat: bump SW cache to v13 for login auth feature"
git push origin main
```

- [ ] **Step 3: Verify live**

Open `https://fudayl321.github.io/sql-trainer/` in a private/incognito window. Auth modal should appear.
