# PR #3549 / issue #3528 - demon-llama browser evidence manifest

- Repo: alex-foster-personal/music-dj-tools
- Branch: af--idd-9d749cbc-issue-3528-resolve-github-issue-3528-https-github-c
- Full head SHA: 431c8ac8b127318f16f1b9ff072bcc745b8b0c3e
- Host: demon-llama.local (alex@, macOS 26.5.1 / Darwin 25.5.0, arm64) - satisfies
  issue #3528's "Air or demon-llama, never silver" requirement
- Checkout: `~/code/music-dj-tools-wt-pr3549`, a `git worktree` at the exact head SHA
  above (detached HEAD), clean working tree at capture time
- Backend: real FastAPI dev server, `127.0.0.1:8696` (playwright.comment-hotkey-gate.config.ts)
- Frontend: real SvelteKit dev server, `127.0.0.1:5321`, proxying `/api` to the backend
  above
- Fixture: real generated-audio + real-ingest throwaway library
  (`support/deckload_fixture.py`, 2 tracks), not mocked data
- Capture tool: Playwright 1.61.1, Chromium (headless), viewport 1280x720 (Playwright's
  `devices['Desktop Chrome']` default; not resized for this capture)
- Captured: 2026-09-24, ~06:39-07:41 UTC (`date -u` on demon-llama)

## Acceptance-suite result (the actual evidence; screenshots are supplementary)

`npx playwright test --config tests/e2e/playwright.comment-hotkey-gate.config.ts --reporter=list`
run against the committed `comment-hotkey-browser.spec.ts` at the head SHA above:
**21 passed, 0 failed, 0 skipped** (2.1m). Full raw output and exit status in
`acceptance-3528-result.txt` alongside this file.

## Screenshots

Captured with a separate, non-committed ad hoc Playwright script
(`pr3549-evidence-capture.spec.ts`, run then deleted - never committed to the repo)
driving the SAME real backend/frontend/fixture the committed suite above boots, to
show visually what that suite's own assertions already verify programmatically.

| file | sha256 | what it shows |
|---|---|---|
| a-before-hover-m.png | 5d0b0e987a7e8622512ea4da8d3d99661759b30a4b3b22aecd3a497ab79b694f | real library row under real mouse hover, no text-entry focus (`document.activeElement` is not an `INPUT`) |
| b-after-m-placement-armed.png | 6bfd07f8b2dae12409fbd75bd7f3431be874cad013ffe02b1149f0917f9862b9 | after pressing `m`: `feedbackState.placementArmed === true`, `.fb-place-overlay` is visible per Playwright's own visibility check - the overlay is deliberately transparent (`background: transparent; cursor: crosshair`) so this frame looks identical to (a); the real state change is the DOM/store assertion, not anything visible here |
| c-composer-bubble-open.png | 2601d671fbd6aad249084427d0e26d63c5ce8a0be2adcf3369d19bed0372323e | after clicking the armed overlay: the actual visible pin composer (`FeedbackPinDraftBubble`, "What is wrong / right here?" textarea, Save pin / Cancel) is open - this is the frame a human recognizes as "the composer opened" |
| d-negative-control-text-input.png | a961bf9dce035e96c44170e99e9e03fa83894005d703295563cee7791e02c4cc | negative control: a real focused `<input>` receives the literal character `m` (`value === 'm'`) instead of arming placement (`placementArmed` stays `false`) |

## Demonstrated actions (per screenshot, in order)

1. Navigate to `/performance`, wait for the real feedback-availability probe to
   resolve `ok`.
2. Hover the first real library row with the mouse (`locator.hover()`); confirm via
   `el.matches(':hover')` and confirm `document.activeElement` is not a text-entry
   element.
3. Press `m` on the keyboard; confirm `feedbackState.placementArmed === true` and
   `.fb-place-overlay` is visible.
4. Click the armed overlay; confirm `.fb-bubble-text` (the real composer) is visible.
5. Separately: focus a real `<input>`, press `m`; confirm the input's value becomes
   `m` and `placementArmed` stays `false`.

## Limitations

- The fixture library is a small (2-track) generated/ingested library, not
  demon-llama's own personal library - sufficient for the real
  `/api/v1/preflight` `library-attached` check and the real hotkey/backend wiring
  this acceptance bullet is about, matching the same real-backend rationale the
  committed suite's own header comment documents (`comment-hotkey-browser.spec.ts`
  top-of-file docstring, r3918992947).
- The screenshots' capture script was ad hoc and intentionally not committed (it
  exists only to produce visual frames of state the committed suite's own
  assertions already prove); the committed suite's 21/0/0 pass result is the
  authoritative acceptance evidence, not the screenshots.
