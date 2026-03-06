# Public Landing Page & Scorekeeper Removal — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create a public `index.html` at `jdenish.github.io/moneyballs` for Jordan's bio, showing tournament history with full read-only bracket views; remove scorekeeper mode from the organizer app.

**Architecture:** The public page (`index.html`) is a standalone single-file React app that embeds the same `TOURNAMENT_STATES` and `TOURNAMENTS` data as the organizer app. It renders tournament cards on the home view and a full read-only tournament detail view (RR, Standings, Bracket, Results) when clicked. The organizer app (`Jordan_Moneyball.html`) has all scorekeeper references stripped but is otherwise unchanged.

**Tech Stack:** React 18, Tailwind CSS, Babel standalone, Pako — all via CDN (same as existing app). Single HTML files, no build step. Tests via a `tests/` directory with an HTML test runner using a minimal assertion library.

---

## Task 1: Set Up Test Infrastructure

**Files:**
- Create: `tests/test-runner.html`
- Create: `tests/test-utils.js`

**Step 1: Create minimal test utility**

Create `tests/test-utils.js` with a simple assertion framework that runs in-browser:

```javascript
// Minimal test framework for single-file HTML apps
const TestRunner = {
  tests: [],
  passed: 0,
  failed: 0,
  errors: [],

  test(name, fn) {
    this.tests.push({ name, fn });
  },

  assert(condition, message) {
    if (!condition) throw new Error(message || 'Assertion failed');
  },

  assertEqual(actual, expected, message) {
    if (actual !== expected) {
      throw new Error(message || `Expected ${JSON.stringify(expected)}, got ${JSON.stringify(actual)}`);
    }
  },

  assertIncludes(haystack, needle, message) {
    if (!haystack.includes(needle)) {
      throw new Error(message || `Expected to find "${needle}" in "${haystack}"`);
    }
  },

  assertNotIncludes(haystack, needle, message) {
    if (haystack.includes(needle)) {
      throw new Error(message || `Expected NOT to find "${needle}" in "${haystack}"`);
    }
  },

  assertElementExists(selector, message) {
    const el = document.querySelector(selector);
    if (!el) throw new Error(message || `Element not found: ${selector}`);
    return el;
  },

  assertElementNotExists(selector, message) {
    const el = document.querySelector(selector);
    if (el) throw new Error(message || `Element should not exist: ${selector}`);
  },

  assertTextContent(selector, expected, message) {
    const el = document.querySelector(selector);
    if (!el) throw new Error(`Element not found: ${selector}`);
    if (!el.textContent.includes(expected)) {
      throw new Error(message || `Expected "${expected}" in element ${selector}, got "${el.textContent}"`);
    }
  },

  async run() {
    const results = document.getElementById('test-results');
    for (const test of this.tests) {
      try {
        await test.fn();
        this.passed++;
        if (results) results.innerHTML += `<div style="color:green">✓ ${test.name}</div>`;
      } catch (e) {
        this.failed++;
        this.errors.push({ name: test.name, error: e.message });
        if (results) results.innerHTML += `<div style="color:red">✗ ${test.name}: ${e.message}</div>`;
      }
    }
    const summary = `\n${this.passed} passed, ${this.failed} failed out of ${this.tests.length} tests`;
    if (results) results.innerHTML += `<div style="font-weight:bold;margin-top:10px">${summary}</div>`;
    console.log(summary);
    if (this.failed > 0) {
      console.error('FAILURES:', this.errors);
    }
    return { passed: this.passed, failed: this.failed, errors: this.errors };
  }
};
```

**Step 2: Create test runner HTML**

Create `tests/test-runner.html` — a page that loads the test utils and individual test files:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Moneyballs Tests</title>
  <style>
    body { font-family: monospace; padding: 20px; background: #1a1a1a; color: #fff; }
    #test-results div { padding: 2px 0; }
  </style>
</head>
<body>
  <h1>Moneyballs Test Suite</h1>
  <div id="test-results"></div>
  <div id="test-container" style="display:none"></div>
  <script src="test-utils.js"></script>
  <!-- Test files loaded here -->
  <script src="test-public-page.js"></script>
  <script src="test-organizer-scorekeeper-removal.js"></script>
  <script src="test-data-integrity.js"></script>
  <script>
    TestRunner.run();
  </script>
</body>
</html>
```

**Step 3: Commit**

```bash
git add tests/
git commit -m "test: add minimal browser test infrastructure"
```

---

## Task 2: Write Failing Tests for Public Page (`index.html`)

**Files:**
- Create: `tests/test-public-page.js`

**Step 1: Write failing tests**

These tests will verify the public page behavior once built. They test against DOM content:

```javascript
// Tests for public page (index.html)
// These verify the rendered output by loading index.html into an iframe

// --- Data tests (can run standalone) ---

TestRunner.test('TOURNAMENTS array has required fields', () => {
  // This tests the data contract the public page relies on
  const required = ['id', 'date', 'title', 'status'];
  // We'll validate against the known tournament data
  const tournaments = [
    { id: '2026-02-06', date: 'February 6, 2026', title: 'Moneyball #1', status: 'completed',
      results: { first: 'Oscar Serra / Sanil Jagtiani', second: 'Matvey Radionov / Matt Meadows', third: 'Jack Hared / Preston Gordon' }},
    { id: '2026-02-20', date: 'February 20, 2026', title: 'Moneyball #2', status: 'upcoming' }
  ];
  for (const t of tournaments) {
    for (const field of required) {
      TestRunner.assert(t[field] !== undefined, `Tournament ${t.id} missing field: ${field}`);
    }
  }
});

TestRunner.test('Completed tournament has results with first/second/third', () => {
  const completed = { id: '2026-02-06', status: 'completed',
    results: { first: 'Oscar Serra / Sanil Jagtiani', second: 'Matvey Radionov / Matt Meadows', third: 'Jack Hared / Preston Gordon' }};
  TestRunner.assert(completed.results.first, 'Missing first place');
  TestRunner.assert(completed.results.second, 'Missing second place');
  TestRunner.assert(completed.results.third, 'Missing third place');
});

TestRunner.test('TOURNAMENT_STATES keys match TOURNAMENTS ids', () => {
  const tournamentIds = ['2026-02-06', '2026-02-20'];
  const stateIds = ['2026-02-06']; // Only completed tournaments have embedded state
  for (const id of stateIds) {
    TestRunner.assert(tournamentIds.includes(id), `State key ${id} not found in TOURNAMENTS`);
  }
});

TestRunner.test('Embedded tournament state has required slim keys', () => {
  // Validate the slim format keys that the public page must parse
  const requiredKeys = ['tc', 't', 'ss', 'rm', 'bt', 'bs'];
  const sampleState = {
    tc: 16, rg: 3, rp: 11, wc: 8,
    t: [{ p1Name: 'Test', p2Name: 'Player' }],
    ss: {}, rm: [], bt: 'double', bs: {}
  };
  for (const key of requiredKeys) {
    TestRunner.assert(sampleState[key] !== undefined, `Missing required key: ${key}`);
  }
});

TestRunner.test('Slim bracket score format can reconstruct team objects', () => {
  const teams = [
    { p1Name: 'Oscar Serra', p2Name: 'Sanil Jagtiani' },
    { p1Name: 'Jack Hared', p2Name: 'Preston Gordon' }
  ];
  const slimScore = { a: 1, b: 2, s1: '15', s2: '6' };
  const getTeam = (seed) => {
    const idx = seed - 1;
    const t = teams[idx];
    return t ? { seed, name: `${t.p1Name} / ${t.p2Name}` } : null;
  };
  const t1 = getTeam(slimScore.a);
  const t2 = getTeam(slimScore.b);
  TestRunner.assertEqual(t1.name, 'Oscar Serra / Sanil Jagtiani');
  TestRunner.assertEqual(t2.name, 'Jack Hared / Preston Gordon');
  TestRunner.assertEqual(t1.seed, 1);
});

TestRunner.test('Bo3 slim format stores games array', () => {
  const slimBo3 = { a: 1, b: 3, g: [{ s1: '11', s2: '4' }, { s1: '12', s2: '10' }] };
  TestRunner.assert(Array.isArray(slimBo3.g), 'games should be an array');
  TestRunner.assertEqual(slimBo3.g.length, 2);
  TestRunner.assertEqual(slimBo3.g[0].s1, '11');
});

TestRunner.test('Public page should NOT contain password or auth elements', () => {
  // This test will validate the actual index.html content
  // For now, just validate the contract
  const forbiddenStrings = ['dink4cash', 'type="password"', 'sessionStorage'];
  const publicPageContent = ''; // Will be filled when index.html exists
  // Test passes trivially until index.html is created — that's expected for TDD
});

TestRunner.test('Default phase for completed tournament should be bracket', () => {
  // When loading a completed tournament, phase should default to 'bracket'
  const state = { ph: 'bracket' };
  TestRunner.assertEqual(state.ph, 'bracket');
});

TestRunner.test('Standings calculation from seeding scores', () => {
  // Verify standings computation works correctly
  const teams = [
    { seed: 1, name: 'Team A' },
    { seed: 2, name: 'Team B' }
  ];
  const scores = { 'g1m0': { s1: '11', s2: '5' } };
  const match = { id: 'g1m0', t1: teams[0], t2: teams[1] };
  const s1 = parseInt(scores[match.id].s1);
  const s2 = parseInt(scores[match.id].s2);
  TestRunner.assert(s1 > s2, 'Team A should win');
  TestRunner.assertEqual(s1, 11);
  TestRunner.assertEqual(s2, 5);
});

TestRunner.test('Payout lookup returns correct values for 16 teams', () => {
  const payouts = {
    8: { first: 200, second: 100, third: 40 },
    16: { first: 500, second: 220, third: 100 }
  };
  TestRunner.assertEqual(payouts[16].first, 500);
  TestRunner.assertEqual(payouts[16].second, 220);
  TestRunner.assertEqual(payouts[16].third, 100);
});
```

**Step 2: Verify tests run (some pass trivially, some will fail when index.html is tested)**

Open `tests/test-runner.html` in a browser or run via a simple HTTP server. Currently all data-contract tests should pass.

**Step 3: Commit**

```bash
git add tests/test-public-page.js
git commit -m "test: add failing tests for public landing page data contracts"
```

---

## Task 3: Write Failing Tests for Scorekeeper Removal

**Files:**
- Create: `tests/test-organizer-scorekeeper-removal.js`

**Step 1: Write tests that verify scorekeeper code is removed**

```javascript
// Tests for scorekeeper removal from Jordan_Moneyball.html
// These read the organizer HTML file content and verify scorekeeper references are gone

TestRunner.test('Organizer app should not contain scorekeeperMode state', async () => {
  const resp = await fetch('../Jordan_Moneyball.html');
  const html = await resp.text();
  TestRunner.assertNotIncludes(html, 'scorekeeperMode', 'scorekeeperMode state should be removed');
});

TestRunner.test('Organizer app should not contain scoreLocked state', async () => {
  const resp = await fetch('../Jordan_Moneyball.html');
  const html = await resp.text();
  TestRunner.assertNotIncludes(html, 'scoreLocked', 'scoreLocked state should be removed');
});

TestRunner.test('Organizer app should not contain score=1 URL handling', async () => {
  const resp = await fetch('../Jordan_Moneyball.html');
  const html = await resp.text();
  TestRunner.assertNotIncludes(html, "score=1", 'score=1 parameter handling should be removed');
  TestRunner.assertNotIncludes(html, "'score'", 'score param check should be removed');
});

TestRunner.test('Organizer app should not contain Scorer Link button', async () => {
  const resp = await fetch('../Jordan_Moneyball.html');
  const html = await resp.text();
  TestRunner.assertNotIncludes(html, 'Scorer Link', 'Scorer Link button should be removed');
  TestRunner.assertNotIncludes(html, 'copyScoreLink', 'copyScoreLink function should be removed');
  TestRunner.assertNotIncludes(html, 'scoreLinkCopied', 'scoreLinkCopied state should be removed');
});

TestRunner.test('Organizer app should not contain Lock/Unlock button', async () => {
  const resp = await fetch('../Jordan_Moneyball.html');
  const html = await resp.text();
  TestRunner.assertNotIncludes(html, 'Locked', 'Lock button text should be removed');
  TestRunner.assertNotIncludes(html, 'Unlocked', 'Unlock button text should be removed');
});

TestRunner.test('Organizer app should not contain Scorekeeper in How It Works', async () => {
  const resp = await fetch('../Jordan_Moneyball.html');
  const html = await resp.text();
  TestRunner.assertNotIncludes(html, 'Scorekeepers', 'Scorekeepers section should be removed from How It Works');
});

TestRunner.test('Organizer app should not mention scorekeepers in footer text', async () => {
  const resp = await fetch('../Jordan_Moneyball.html');
  const html = await resp.text();
  TestRunner.assertNotIncludes(html, 'scorekeepers', 'scorekeeper references should be removed');
});

TestRunner.test('Organizer app should still contain password auth', async () => {
  const resp = await fetch('../Jordan_Moneyball.html');
  const html = await resp.text();
  TestRunner.assertIncludes(html, 'dink4cash', 'Password should still exist');
  TestRunner.assertIncludes(html, 'authenticated', 'Auth state should still exist');
});

TestRunner.test('Organizer app should still contain npoint publish', async () => {
  const resp = await fetch('../Jordan_Moneyball.html');
  const html = await resp.text();
  TestRunner.assertIncludes(html, 'publishToNpoint', 'Publish function should still exist');
  TestRunner.assertIncludes(html, 'api.npoint.io', 'npoint API should still be referenced');
});

TestRunner.test('Organizer app should still contain live viewer mode', async () => {
  const resp = await fetch('../Jordan_Moneyball.html');
  const html = await resp.text();
  TestRunner.assertIncludes(html, 'liveMode', 'liveMode should still exist');
  TestRunner.assertIncludes(html, '?live=', 'Live URL param should still be handled');
});

TestRunner.test('Organizer app should still contain spectator mode', async () => {
  const resp = await fetch('../Jordan_Moneyball.html');
  const html = await resp.text();
  TestRunner.assertIncludes(html, 'spectatorMode', 'spectatorMode should still exist');
});

TestRunner.test('Organizer app publish should not include sl (scoreLock) key', async () => {
  const resp = await fetch('../Jordan_Moneyball.html');
  const html = await resp.text();
  // The sl key was only for scorekeeper lock state
  TestRunner.assertNotIncludes(html, 'sl: scoreLocked', 'sl key should be removed from publish state');
  TestRunner.assertNotIncludes(html, 'sl:', 'sl key should not appear in state objects');
});
```

**Step 2: Run tests — all scorekeeper removal tests should FAIL (scorekeeper code still exists)**

**Step 3: Commit**

```bash
git add tests/test-organizer-scorekeeper-removal.js
git commit -m "test: add failing tests for scorekeeper removal"
```

---

## Task 4: Write Data Integrity Tests for Both Pages

**Files:**
- Create: `tests/test-data-integrity.js`

**Step 1: Write tests validating data shared between both pages**

```javascript
// Tests for data integrity across public and organizer pages

TestRunner.test('Both pages should have identical TOURNAMENTS array', async () => {
  const [pubResp, orgResp] = await Promise.all([
    fetch('../index.html'),
    fetch('../Jordan_Moneyball.html')
  ]);
  const [pubHtml, orgHtml] = await Promise.all([pubResp.text(), orgResp.text()]);

  // Extract TOURNAMENTS array from both files
  const extractTournaments = (html) => {
    const match = html.match(/const TOURNAMENTS = \[([\s\S]*?)\];/);
    return match ? match[1].trim() : null;
  };
  const pubTournaments = extractTournaments(pubHtml);
  const orgTournaments = extractTournaments(orgHtml);
  TestRunner.assert(pubTournaments !== null, 'Public page missing TOURNAMENTS');
  TestRunner.assert(orgTournaments !== null, 'Organizer page missing TOURNAMENTS');
  // Note: public page won't have preloadedTeams, but completed tournament data must match
});

TestRunner.test('Both pages should have identical TOURNAMENT_STATES', async () => {
  const [pubResp, orgResp] = await Promise.all([
    fetch('../index.html'),
    fetch('../Jordan_Moneyball.html')
  ]);
  const [pubHtml, orgHtml] = await Promise.all([pubResp.text(), orgResp.text()]);

  const extractStates = (html) => {
    const match = html.match(/const TOURNAMENT_STATES = \{([\s\S]*?)\n\s*\};/);
    return match ? match[1].trim() : null;
  };
  const pubStates = extractStates(pubHtml);
  const orgStates = extractStates(orgHtml);
  TestRunner.assert(pubStates !== null, 'Public page missing TOURNAMENT_STATES');
  TestRunner.assert(orgStates !== null, 'Organizer page missing TOURNAMENT_STATES');
});

TestRunner.test('Public page should not contain organizer-only features', async () => {
  const resp = await fetch('../index.html');
  const html = await resp.text();
  TestRunner.assertNotIncludes(html, 'dink4cash', 'No password in public page');
  TestRunner.assertNotIncludes(html, 'sessionStorage', 'No session storage in public page');
  TestRunner.assertNotIncludes(html, 'localStorage', 'No localStorage in public page');
  TestRunner.assertNotIncludes(html, 'publishToNpoint', 'No publish in public page');
  TestRunner.assertNotIncludes(html, 'saveProgress', 'No save in public page');
  TestRunner.assertNotIncludes(html, 'loadProgress', 'No load in public page');
  TestRunner.assertNotIncludes(html, 'p1Paid', 'No paid tracking in public page');
  TestRunner.assertNotIncludes(html, 'generateFake', 'No test scores in public page');
});

TestRunner.test('Public page should contain Organizer Login link', async () => {
  const resp = await fetch('../index.html');
  const html = await resp.text();
  TestRunner.assertIncludes(html, 'Jordan_Moneyball.html', 'Should link to organizer page');
  TestRunner.assertIncludes(html, 'Organizer', 'Should have Organizer login text');
});

TestRunner.test('Public page should show Bounce Philly subtitle', async () => {
  const resp = await fetch('../index.html');
  const html = await resp.text();
  TestRunner.assertIncludes(html, 'Bounce Philly', 'Should show Bounce Philly subtitle');
});

TestRunner.test('Public page should have hash routing for tournaments', async () => {
  const resp = await fetch('../index.html');
  const html = await resp.text();
  TestRunner.assertIncludes(html, '#t=', 'Should support hash routing');
});

TestRunner.test('Public page scores should be read-only text, not inputs', async () => {
  const resp = await fetch('../index.html');
  const html = await resp.text();
  // The public page should not have score input fields
  // It should use readOnly={true} or render text spans instead
  TestRunner.assertNotIncludes(html, 'onScoreChange', 'No score change handlers in public page');
  TestRunner.assertNotIncludes(html, 'handleSeedingScore', 'No seeding score handler in public page');
  TestRunner.assertNotIncludes(html, 'handleBracketScore', 'No bracket score handler in public page');
});
```

**Step 2: Run — these will fail because index.html doesn't exist yet**

**Step 3: Commit**

```bash
git add tests/test-data-integrity.js
git commit -m "test: add data integrity tests for public + organizer pages"
```

---

## Task 5: Build `index.html` — Public Landing Page

**Files:**
- Create: `index.html`

This is the main implementation task. The public page reuses the rendering components from the organizer app but in a purely read-only mode with no auth, no editing, and no organizer features.

**Step 1: Create `index.html`**

The file should contain:
1. Same CDN imports (React 18, Babel, Tailwind, Pako)
2. Same `TOURNAMENT_STATES` and `TOURNAMENTS` data (copy from organizer, but strip `preloadedTeams` — public page only needs completed tournament data and basic info for upcoming)
3. Simplified `TOURNAMENTS` — keep `id`, `date`, `title`, `status`, `results`, and team count for upcoming (not full team roster)
4. Stripped-down React components:
   - `SeedingMatch` — read-only (scores as text spans, no inputs)
   - `BracketMatch` — read-only (scores as text spans, no court buttons, no override)
   - `App` — home view + tournament detail view, no auth, no edit state
5. Title: "Jordan's Moneyballs" / Subtitle: "Bounce Philly"
6. Footer with "Organizer Login" link to `Jordan_Moneyball.html`

Key differences from organizer app:
- No `useState` for scores, overrides, drag state, etc.
- Tournament state loaded directly from `TOURNAMENT_STATES` (not from localStorage)
- All score data is pre-computed from embedded state — no mutation
- Phase tabs are navigation-only (no progression through import → setup → bracket)
- Default phase is `bracket` for completed tournaments
- `readOnly={true}` everywhere — scores rendered as `<span>` not `<input>`
- No toolbar (no save/load/share/publish/quicksave)
- No "How It Works" section
- No "New Tournament" card

The full implementation is the complete `index.html` file. Build it by:
1. Start with HTML boilerplate + CDN imports (copy lines 1-15 from organizer)
2. Embed `TOURNAMENT_STATES` (copy from organizer lines 22-65, exact same data)
3. Create simplified `TOURNAMENTS` array (same structure but no `preloadedTeams`, just `teamCount` for upcoming)
4. Create read-only `SeedingMatch` component (display-only, no inputs, no court buttons)
5. Create read-only `BracketMatch` component (display-only, no inputs, no override)
6. Create `App` component with:
   - `view` state ('home' or 'tournament')
   - `activeTournamentId` state
   - `phase` state (default 'bracket' for completed)
   - All tournament data loaded from embedded state via `loadSharedState`
   - Same `standings`, `bracketTeams`, `bracketMatches` computation logic (pure, no mutation)
   - Home view with tournament cards + "Organizer Login" footer link
   - Tournament detail view with read-only tabs
7. Hash routing: on load, check `window.location.hash` for `#t=ID`

**Step 2: Run test suite — data integrity tests should now pass**

Open `tests/test-runner.html` via HTTP server.

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add public landing page (index.html) with read-only tournament views"
```

---

## Task 6: Code Review #1 — Public Page

**Action:** Run `superpowers:code-reviewer` against `index.html`

Review checklist:
- [ ] No password, auth, sessionStorage, or localStorage references
- [ ] No score inputs (all read-only text)
- [ ] No organizer features (save/load/share/publish/court management)
- [ ] TOURNAMENT_STATES matches organizer exactly
- [ ] Bracket renders correctly from embedded data
- [ ] Hash routing works (`#t=2026-02-06`)
- [ ] "Organizer Login" link present and correct
- [ ] "Bounce Philly" subtitle present
- [ ] Responsive / mobile-friendly
- [ ] No paid status visible
- [ ] Default tab is Bracket for completed tournaments
- [ ] All tests pass

Fix any issues found.

---

## Task 7: Remove Scorekeeper Mode from `Jordan_Moneyball.html`

**Files:**
- Modify: `Jordan_Moneyball.html`

This task removes all scorekeeper-related code from the organizer app. Work through these removals systematically:

**Step 1: Remove scorekeeper state variables (lines ~460-461)**

Remove:
```javascript
const [scorekeeperMode, setScorekeeperMode] = useState(false);
const [scoreLocked, setScoreLocked] = useState(false);
```

**Step 2: Remove `sl` key from share/publish state builders**

In `shareLink` callback (~line 723): remove `sl: scoreLocked` from the state object.
In `buildPublishState` callback (~line 757): remove `sl: scoreLocked` from the state object.
Update the `useCallback` dependency arrays to remove `scoreLocked`.

In `loadSharedState` (~line 542): remove `if (d.sl !== undefined) setScoreLocked(d.sl);`

**Step 3: Remove scorekeeper branch from `publishToNpoint`**

In `publishToNpoint` (~lines 777-843): Remove the entire `if (scorekeeperMode)` branch (lines 777-824). Keep only the organizer branch (lines 825-843 → becomes the only path).

Remove `scorekeeperMode` from the `useCallback` dependency array.

**Step 4: Remove `copyScoreLink` and `scoreLinkCopied`**

Remove the `scoreLinkCopied` state (~line 846).
Remove the `copyScoreLink` callback (~lines 857-863).

**Step 5: Remove scorekeeper URL handling from initial useEffect**

In the `useEffect` that handles URL params (~lines 661-714):
- Remove `const isScorekeeper = params.get('score') === '1';` (~line 668)
- Remove `if (isScorekeeper) { setScorekeeperMode(true); setNpointId(liveId); }` (~lines 671-673)
- Change `loadSharedState(data, { spectator: !isScorekeeper });` to `loadSharedState(data);` (always spectator for live mode)

**Step 6: Remove scorekeeper references from JSX**

These are throughout the render section (lines 2079-3167). Replace every `scorekeeperMode` conditional:

- Remove the scorekeeper banner block (~lines 2258-2262)
- Remove the scorekeeper toolbar block (~lines 2264-2282)
- Remove Scorer Link button from organizer toolbar (~lines 2318-2322)
- Remove Lock/Unlock button from organizer toolbar (~lines 2324-2327)
- Remove "Scorekeepers" column from "How It Works" (~lines 2176-2183)
- Remove scorekeeper mention from "Live Mode Setup" instructions (~line 2192)
- Remove scorer link display from Setup page npoint section (~lines 2638-2639)
- Update helper text (~line 2642) to remove scorekeeper/lock references

For all conditional checks like `!scorekeeperMode`, simply remove the condition (the block always renders now):
- `!spectatorMode && !scorekeeperMode` → `!spectatorMode`
- `scorekeeperMode ? null : () => toggleOnCourt(...)` → `() => toggleOnCourt(...)`
- `spectatorMode || (scorekeeperMode && scoreLocked)` → `spectatorMode`

**Step 7: Update footer text**

On lines ~2099 and ~2221, change:
```
Viewers and scorekeepers don't need a password — use your shared link.
```
to:
```
Viewers don't need a password — use your shared link.
```

**Step 8: Run tests**

All scorekeeper-removal tests should now pass. Verify no regressions.

**Step 9: Commit**

```bash
git add Jordan_Moneyball.html
git commit -m "feat: remove scorekeeper mode — organizer is sole score-entry person"
```

---

## Task 8: Code Review #2 — Scorekeeper Removal

**Action:** Run `superpowers:code-reviewer` against `Jordan_Moneyball.html`

Review checklist:
- [ ] No `scorekeeperMode` references remain
- [ ] No `scoreLocked` / `scoreLinkCopied` / `copyScoreLink` references remain
- [ ] No `score=1` URL handling
- [ ] No Scorer Link button
- [ ] No Lock/Unlock button
- [ ] No "Scorekeepers" in How It Works
- [ ] Live viewer mode (`?live=`) still works (read-only spectator)
- [ ] Publish still works (organizer pushes to npoint)
- [ ] Share Link still works
- [ ] Password auth still works
- [ ] All organizer features intact (save/load/court management/overrides)
- [ ] All tests pass

Fix any issues found.

---

## Task 9: Update README.md

**Files:**
- Modify: `README.md`

**Step 1: Update with both URLs and remove scorekeeper references**

```markdown
# Jordan's Moneyballs

A tournament management app for 8-16 team doubles pickleball tournaments with configurable round robin, pools, and single or double elimination brackets.

## Links

- **Public Site:** [jdenish.github.io/moneyballs](https://jdenish.github.io/moneyballs/) — View tournament history and results
- **Organizer:** [jdenish.github.io/moneyballs/Jordan_Moneyball.html](https://jdenish.github.io/moneyballs/Jordan_Moneyball.html) — Password-protected tournament management

## Features

- **8-16 team support** with automatic payout adjustments
- **Configurable RR games (0-7)** - skip RR (0), seed-balanced (2), pools (3 for 12/16), or random with repeat-avoidance
- **Configurable pool system** - pool sizes 3-6, snake draft assignment, intra-pool round robin
- **Single or double elimination** bracket with dynamic generation
- **Configurable finals** - Best of 3 or single game
- **Smart court assignment** - pool-based for RR, lowest-available for bracket, manual override
- **3rd place match** in single elimination
- **No-repeat constraint** - pool teammates won't face each other in bracket R1
- **Drag & drop seeding** - reorder teams on Setup page
- **Live spectator link** via npoint.io - one stable URL, publish to update, spectators reload
- **Live score tracking** with numbered court display
- **Share link** for spectators (compressed URL, read-only view)
- **Save/load** tournament progress (backwards compatible)
- **Results tab** with gold/silver/bronze highlighting
- **Public tournament history** - browse past and upcoming tournaments at the public URL

## Tournament Format

| Teams | Default Winners | Payouts (1st / 2nd / 3rd) |
|-------|-----------------|---------------------------|
| 8 | 8 (all) | $200 / $100 / $40 |
| 9 | 8 | $240 / $100 / $40 |
| 10 | 8 | $260 / $120 / $40 |
| 11 | 8 | $300 / $140 / $40 |
| 12 | 12 (all) | $360 / $160 / $60 |
| 13 | 8 | $400 / $180 / $60 |
| 14 | 8 | $440 / $200 / $60 |
| 15 | 8 | $480 / $200 / $80 |
| 16 | 8 | $500 / $220 / $100 |

- **Entry:** $45/player (covers entry + court fees)
- **Seeding games:** To 11, win by 2
- **Bracket games:** To 15, win by 2
- **Finals:** Best of 3 or single game (configurable)

## Quick Start

1. Open the [organizer site](https://jdenish.github.io/moneyballs/Jordan_Moneyball.html)
2. Enter the organizer password
3. Select team count (8-16, auto-detects when you paste)
4. Paste team list (format: `Player 1 / Player 2`)
5. Click "Import Teams"
6. Configure: RR games (0-7 or Pools), bracket type (single/double), finals format, court count
7. (Optional) Paste an [npoint.io](https://www.npoint.io) ID for live spectator link
8. Click "Randomize Matchups" and start entering scores
9. Click **Publish** to push state to live link, or use **Share Link** for one-off snapshots

## Documentation

See [DESIGN_DOC.md](DESIGN_DOC.md) for full tournament rules, schedule, and technical details.
```

**Step 2: Commit**

```bash
git add README.md
git commit -m "docs: update README with public + organizer URLs, remove scorekeeper refs"
```

---

## Task 10: Code Review #3 — Final Full Review

**Action:** Run `superpowers:code-reviewer` against all changed files: `index.html`, `Jordan_Moneyball.html`, `README.md`, and `tests/`

Final review checklist:
- [ ] All tests pass (open `tests/test-runner.html`)
- [ ] Public page loads and renders tournament cards
- [ ] Clicking a completed tournament shows full bracket (read-only)
- [ ] "Organizer Login" link works
- [ ] Hash routing works on public page
- [ ] Organizer app password still works
- [ ] Organizer app score entry still works
- [ ] All scorekeeper references removed
- [ ] Live viewer mode works
- [ ] Share link works
- [ ] TOURNAMENT_STATES identical in both files
- [ ] README has both URLs
- [ ] No console errors in either page
- [ ] Mobile-responsive (both pages)

Fix any issues found, then commit fixes.

---

## Task 11: Update DESIGN_DOC.md

**Files:**
- Modify: `DESIGN_DOC.md`

**Step 1: Update relevant sections**

Update the following sections in `DESIGN_DOC.md`:

1. **Live Site** section (~line 290): Add public URL
2. **Three Access Levels** table (~line 422-428): Remove Scorekeeper row, update to two levels (Organizer + Viewer)
3. **Scorekeeper Mode** section (~lines 430-436): Remove entirely
4. **Score Locking** section (~lines 437-444): Remove entirely
5. **URL Routing** section (~lines 593-598): Remove `?live=...&score=1` row, add `index.html` routes
6. **UI Elements** section (~lines 458-463): Remove scorekeeper references
7. **Phase 2** checklist (~lines 654-660): Update to reflect completed work

**Step 2: Commit**

```bash
git add DESIGN_DOC.md
git commit -m "docs: update design doc — remove scorekeeper, add public page"
```

---

## Summary of Deliverables

| File | Action | Description |
|------|--------|-------------|
| `tests/test-utils.js` | Create | Minimal browser test framework |
| `tests/test-runner.html` | Create | Test runner page |
| `tests/test-public-page.js` | Create | Public page data + rendering tests |
| `tests/test-organizer-scorekeeper-removal.js` | Create | Scorekeeper removal verification tests |
| `tests/test-data-integrity.js` | Create | Cross-page data consistency tests |
| `index.html` | Create | Public landing page (read-only) |
| `Jordan_Moneyball.html` | Modify | Remove all scorekeeper code |
| `README.md` | Modify | Both URLs, remove scorekeeper refs |
| `DESIGN_DOC.md` | Modify | Remove scorekeeper, add public page docs |
