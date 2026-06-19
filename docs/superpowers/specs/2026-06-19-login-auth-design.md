# SQL Trainer — Login & Auth Design Spec

**Date:** 2026-06-19  
**Feature:** User login, registration, and logout via Supabase Auth  
**File scope:** `sql-trainer/index.html` (single-file PWA)

---

## Goal

Replace the current username-only modal with a real login/register system backed by Supabase Auth. Users create accounts with a username + password. Progress is tied to their Supabase Auth UUID, not a username string.

---

## Auth Flow

On every page load, call `_sb.auth.getSession()`:
- **Session exists** → load the app normally (skip login screen)
- **No session** → show the full-screen login/register overlay

### Login Screen

Full-screen overlay replacing the current `#username-modal`. Same dark theme, same visual style. Contains two tabs:

**Register tab (default):**
- Username field (min 3 chars)
- Password field (min 6 chars)
- Confirm Password field (must match password)
- "Create Account" button — disabled until all fields valid
- On submit: `_sb.auth.signUp({ email: '${username}@sql-trainer.app', password: '...' }, { data: { username } })`
- Username stored in `user.user_metadata.username` for display
- On success: session auto-persists via Supabase, app loads

**Login tab:**
- Username field
- Password field
- "Login" button — disabled until both fields filled
- On submit: `_sb.auth.signInWithPassword({ email: '${username}@sql-trainer.app', password: '...' })`
- On success: session auto-persists, app loads

**Error handling:**
- Username already taken → show "Username already exists. Try logging in."
- Wrong password → show "Incorrect username or password."
- Network error → show "Could not connect. Check your internet."
- All errors shown inline below the submit button (no alerts)

**Session management:** Supabase client auto-stores the JWT in localStorage and handles token refresh. No manual session code needed.

---

## Progress Data

### Table schema change

The `sql_trainer_progress` table changes from `user_id TEXT` (username string) to `user_id UUID` (Supabase Auth UID).

**Migration:** Drop and recreate the table (no data migration — project is early stage).

New schema:
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

### Code changes

`_syncToSupabase(p)`:
- Remove `localStorage.getItem(USER_ID_KEY)`
- Use `(await _sb.auth.getSession()).data.session?.user.id` as `userId`
- If no session, return early (no sync)

`loadProgress()`:
- Remove `localStorage.getItem(USER_ID_KEY)` fallback
- Use `session.user.id` to query Supabase

`USER_ID_KEY` constant and all `localStorage` usages of it: **removed entirely**.

### Username display

`user.user_metadata.username` (set during `signUp`) is used wherever the username is shown:
- Progress tab: "Syncing as: **dello**"
- Retrieved via `session.user.user_metadata.username`

---

## Logout

Two entry points, both call the same logout action:

**Logout action:**
1. `await _sb.auth.signOut()`
2. `localStorage.removeItem(STORAGE_KEY)` (clear local progress)
3. `location.reload()` (returns to login screen)

**Header logout button:**
- Position: inside `.badges` row, after the streak badge
- Appearance: `⏻` icon, ghost style (`color: var(--text-dim)`), red on hover (`color: var(--error)`)
- No label — icon only, with `aria-label="Logout"`

**Progress tab logout button:**
- Replaces the current "Change" button in the "Syncing as" row
- Label: "Logout"
- Same style as existing "Change" button

The "Change username" feature is removed — users switch accounts by logging out and logging in.

---

## Removed

- `USER_ID_KEY = 'sql_trainer_user_id'` constant
- `#username-modal` HTML, CSS, and `showUsernameModal()` JS
- `change-username-btn` and its event listener
- `localStorage.setItem/getItem(USER_ID_KEY, ...)` calls
- Public-access RLS policy on `sql_trainer_progress`

---

## Supabase Setup (run in SQL Editor before deploying)

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

Also in Supabase Dashboard → Authentication → Settings:
- Disable "Confirm email" (users register without email verification since we use fake emails)

---

## Files Changed

| File | Change |
|------|--------|
| `index.html` | Replace username modal with login/register overlay; update `_syncToSupabase`, `loadProgress`, `initApp`; add logout buttons; remove `USER_ID_KEY` |
| `sw.js` | Bump cache version to `sql-trainer-v13` |
