# SQL Trainer — Project Status

**Last updated:** 2026-06-17  
**Live URL:** https://fudayl321.github.io/sql-trainer/  
**Repo:** https://github.com/Fudayl321/sql-trainer  
**Branch:** main  
**Service worker cache:** `sql-trainer-v8`

---

## What's Built

Single-file PWA (`index.html` + `sw.js` + `manifest.json`). No build step.  
Stack: Vanilla HTML/CSS/JS · AlaSQL v4.2.3 (CDN) · localStorage · GitHub Pages.

### Features
- **5-level SQL curriculum** — Foundations → Aggregation → Joins & Subqueries → CTEs & Windows → DML & Data Changes
- **Daily sessions** — 5 questions per session (Q1 warmup → Q2 consolidate → Q3 stretch → Q4 review from another concept → Q5 challenge/bonus)
- **Day Gate** — after completing a session, user is asked "Continue to next topic?". If yes: 2 gate questions from next concept (max 3 attempts each). Pass both → next session unlocks immediately. Fail → come back tomorrow.
- **Wiki** — accordion-style reference cards for all SQL syntax
- **Practice mode** — free practice across all 5 levels
- **Progress tab** — sessions done, streak, level progress bar, concept checklist
- **Schema tab** — sample database tables
- **Streak tracking** — daily streak with stale-streak display fix
- **DML questions** — INSERT/UPDATE/DELETE evaluated via verify SELECT pattern

---

## Curriculum

```js
CURRICULUM = {
  1: { name: 'Foundations',        concepts: ['select_basics','where_filter','where_operators','order_by','distinct','limit','between','like','is_null','aliases','wildcards_adv'] },
  2: { name: 'Aggregation',        concepts: ['aggregate_functions','group_by','having','case_when','union','string_functions','date_functions'] },
  3: { name: 'Joins & Subqueries', concepts: ['inner_join','left_join','subquery','in_exists','self_join','coalesce','right_join','cross_join','any_all'] },
  4: { name: 'CTEs & Windows',     concepts: ['cte_basics','row_number','rank_dense_rank','running_total','lag_lead','multiple_cte','partition_agg','nullif_fn'] },
  5: { name: 'DML & Data Changes', concepts: ['insert_rows','update_rows','delete_rows'] }
}
```

---

## Question Pool Status

| Type | Count | Coverage |
|------|-------|----------|
| warmup | 38 | All 38 concepts ✓ |
| consolidate | 38 | All 38 concepts ✓ |
| stretch | 35 | 3 concepts missing |
| challenge | 19 | **19 concepts missing** ← pending work |

**Concepts missing `challenge` questions** (Q5 currently falls back to a cross-concept bonus question):  
Run `validateQuestionPool()` in browser DevTools console to get the exact list.

---

## Pending Work

### 1. Add missing challenge questions (19 concepts) — HIGH PRIORITY
The Consultant identified this as the root cause of the recurring "only 4 questions" bug. Q5 currently shows a cross-concept fallback, but each concept should have its own native challenge question.

To find the exact list:
1. Open https://fudayl321.github.io/sql-trainer/ in browser
2. Open DevTools → Console
3. Run:
```js
const types = ['warmup','consolidate','stretch','challenge'];
Object.entries(CURRICULUM).forEach(([lvl, {concepts}]) => {
  concepts.forEach(c => {
    types.forEach(t => {
      if (!QUESTIONS.some(q => q.concept === c && q.type === t)) {
        console.warn(`Missing: level ${lvl} / ${c} / ${t}`);
      }
    });
  });
});
```

### 2. Progress not updating visually (phone) — INVESTIGATE
Fix was applied (`renderProgress()` called inside `renderToday()`). User reported it still not updating on phone. Likely service worker cache — user needs to clear browser cache on phone. Verify once challenge questions are added.

### 3. Real PNG icon for PWA manifest — LOW PRIORITY
`manifest.json` currently uses a `data:` URI SVG icon. Replace with a real PNG for better PWA install experience.

---

## Recent Commits

| Hash | Description |
|------|-------------|
| `53ed4bc` | fix: always show 5 questions — fallback to cross-concept question when type missing |
| `9bd5a60` | fix: treat null-id question slots as complete so 4-question sessions can finish |
| `fd80173` | feat: redesign day gate — show continue prompt after session complete |
| `06a7be8` | feat: day gate + fix progress update after session complete |
| `fdd5895` | fix: XSS, remove dead unlock feature, Q4 level scoping, streak display, Level 5 practice, pickQ null |
| `abc5602` | feat: full SQL trainer with wiki, 5-level curriculum, DML questions |

---

## Key Code Locations (index.html)

| Section | Line |
|---------|------|
| TABLES (sample data) | ~1 |
| QUESTIONS array | ~200 |
| CURRICULUM / CONCEPT_LABELS | ~1800 |
| DEFAULT_PROGRESS / loadProgress | ~1848 |
| generateSession() | ~1910 |
| getOrCreateSession() | ~1963 |
| completeSession() | ~1976 |
| Day Gate: initGate / submitGateAnswer / renderGate | ~2029 |
| evaluate() / compareResults() | ~2179 |
| submitAnswer() | ~2374 |
| renderToday() | ~2283 |
| renderProgress() | ~3545 |
| updateHeader() | ~3602 |
| switchTab() | ~3617 |

---

## How to Resume

1. Open `C:\Project\sql-trainer\index.html` in your editor
2. The immediate next task is adding ~19 missing `challenge` questions — one per concept that's missing one
3. Each challenge question follows this shape:
```js
{
  id: 'q_<concept_short>_ch1',
  level: <1-5>,
  concept: '<concept_key>',
  type: 'challenge',
  prompt: '...',
  expected: [...],  // array of result row objects
  solution: 'SELECT ...',
  hint: '...'
}
```
4. After adding, bump sw.js cache to `sql-trainer-v9` and push
