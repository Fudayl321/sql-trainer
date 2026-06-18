# SQL Trainer — Supabase Progress Sync Design

**Date:** 2026-06-18  
**Feature:** Auto-sync progress to Supabase so it survives cache clears and restores automatically on any device  
**Approach:** Option A — Supabase sync with user-chosen username, no login required

---

## Scope

Add Supabase auto-sync to `index.html`. Progress is saved locally to localStorage (unchanged) and also upserted to Supabase in the background on every save. On app load with empty localStorage, progress is fetched from Supabase automatically.

**In scope:**
- Supabase table creation (`sql_trainer_progress`)
- Username prompt modal (one-time, first visit)
- `saveProgress()` — add async Supabase upsert
- `loadProgress()` — add Supabase fetch fallback when localStorage is empty
- Reset flow — delete Supabase row on reset
- Sync status pill in header
- "Logged in as" section in Progress tab

**Out of scope:**
- Authentication / passwords
- Multi-device real-time sync (not needed for single-user)
- Progress history / versioning

---

## Supabase Setup

**Project:** `https://vxbylefdrmcdbngvpjnt.supabase.co` (existing, shared with portfolio tracker)

**New table:** `sql_trainer_progress`

```sql
CREATE TABLE sql_trainer_progress (
  user_id      text PRIMARY KEY,
  progress_json text NOT NULL,
  updated_at   timestamptz DEFAULT now()
);

-- RLS: enable row-level security
ALTER TABLE sql_trainer_progress ENABLE ROW LEVEL SECURITY;

-- Allow anyone to read/write their own row (anon key is safe)
CREATE POLICY "allow_all" ON sql_trainer_progress
  FOR ALL USING (true) WITH CHECK (true);
```

**SDK:** Supabase JS v2 via CDN — add to `<head>` before existing scripts. Must include Subresource Integrity (SRI) hash to guard against CDN compromise. Pin to an exact version:

```html
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2.x.x/dist/umd/supabase.min.js"
        integrity="sha384-<hash>"
        crossorigin="anonymous"></script>
```

To get the correct hash for the pinned version during implementation:
1. Go to `https://www.srihash.org/`
2. Enter the exact CDN URL with pinned version
3. Copy the `integrity` value

Alternatively, use the jsdelivr combined API: `https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2` and check the exact resolved version, then generate the hash.

**Client init** (in `<script>` block, near top):
```js
const SUPABASE_URL = 'https://vxbylefdrmcdbngvpjnt.supabase.co';
const SUPABASE_ANON_KEY = '<anon-key>';
const supabase = window.supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
const USER_ID_KEY = 'sql_trainer_user_id';
```

---

## User ID Flow

1. App loads → check `localStorage.getItem(USER_ID_KEY)`
2. If found → proceed to normal app init
3. If not found → show username prompt modal, block app init until submitted
4. Username stored as `localStorage.setItem(USER_ID_KEY, username)` — never asked again
5. "Change username" in Progress tab clears `USER_ID_KEY` and calls `location.reload()`

---

## Sync Logic

### loadProgress() — updated

```
1. Read from localStorage (as today)
2. If result has real progress (completedSessions > 0 OR level > 1 OR completedQuestions.length > 0):
     → return it (fast path, no Supabase call)
3. Else:
     → fetch from Supabase WHERE user_id = USER_ID
     → if row found: parse progress_json, write to localStorage, return it
     → if no row / error: return DEFAULT_PROGRESS
```

Note: `loadProgress()` becomes async. Call site at startup (`let progress = loadProgress()`) becomes `let progress = await loadProgress()` inside an async init function.

### saveProgress(p) — updated

```
1. localStorage.setItem(STORAGE_KEY, JSON.stringify(p))  ← synchronous, unchanged
2. syncToSupabase(p)  ← fire-and-forget async, not awaited
```

```js
async function syncToSupabase(p) {
  const userId = localStorage.getItem(USER_ID_KEY);
  if (!userId) return;
  showSyncIndicator('saving');
  const { error } = await supabase
    .from('sql_trainer_progress')
    .upsert({ user_id: userId, progress_json: JSON.stringify(p), updated_at: new Date().toISOString() });
  if (error) showSyncIndicator('error');
  else hideSyncIndicator();
}
```

### Reset flow — updated

After `localStorage.removeItem(STORAGE_KEY)`:
```js
const userId = localStorage.getItem(USER_ID_KEY);
if (userId) {
  supabase.from('sql_trainer_progress').delete().eq('user_id', userId);
}
```

---

## UI Components

### Username Prompt Modal

Shown when `USER_ID_KEY` is absent from localStorage. Blocks app render.

```html
<div id="username-modal">
  <div class="username-box">
    <h2>Welcome to SQL Trainer</h2>
    <p>Choose a username to save your progress across devices. Pick something unique.</p>
    <input id="username-input" type="text" placeholder="e.g. dello123" maxlength="30" />
    <button id="username-submit" disabled>Start Training</button>
  </div>
</div>
```

- Submit button enabled only when input length ≥ 3
- On submit: trim input, save to `USER_ID_KEY`, hide modal, call `initApp()`
- No cancel/skip

### Sync Indicator

Small pill appended to `#header .badges`:

```html
<span id="sync-pill" style="display:none"></span>
```

States:
- Hidden (default)
- Saving: `↑ Saving` — dim text, shown while upsert in flight
- Error: reuse existing `#offline-pill` style (amber) — shown if upsert fails

Auto-hides 1.5s after successful save.

### Progress Tab — Sync Section

Appended inside the Progress tab card, below the reset button:

```html
<div class="sync-info">
  <span>Syncing as: <strong id="sync-username"></strong></span>
  <button id="change-username-btn">Change</button>
</div>
```

`renderProgress()` sets `#sync-username` text from `localStorage.getItem(USER_ID_KEY)`.

---

## Error Handling

| Scenario | Behaviour |
|---|---|
| Supabase upsert fails (offline) | Silent — localStorage already saved. Amber sync pill shown briefly. |
| Supabase fetch fails on load | Fall back to DEFAULT_PROGRESS — user starts fresh but data is still in Supabase for next load attempt |
| Username already taken | Not enforced — last write wins. Users should pick a unique username. |
| Username cleared / changed | `location.reload()` triggers fresh prompt; old Supabase row remains (orphaned but harmless) |

---

## Success Criteria

- [ ] Username modal appears on first visit, never again after
- [ ] Progress survives hard refresh (localStorage)
- [ ] Progress survives full cache clear + localStorage wipe — restores from Supabase on next load
- [ ] Sync pill appears briefly after each answer submit, then disappears
- [ ] Amber sync pill appears when offline
- [ ] Progress tab shows "Syncing as: dello" with Change button
- [ ] Reset clears both localStorage and Supabase row
- [ ] App works fully offline (localStorage only, Supabase calls fail silently)

---

## Files Changed

| File | Change |
|---|---|
| `index.html` | Add Supabase SDK script tag, client init, username modal HTML/CSS, sync pill HTML, updated loadProgress/saveProgress/reset logic, sync section in Progress tab |
| `sw.js` | Bump cache to `sql-trainer-v12`, add Supabase CDN URL to ASSETS array |

---

## Supabase Credentials Needed

Before implementation, the developer needs the **anon public key** from the Supabase project dashboard:  
Dashboard → Project Settings → API → `anon` `public` key
