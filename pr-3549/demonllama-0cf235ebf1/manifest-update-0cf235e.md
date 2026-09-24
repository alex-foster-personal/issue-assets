# PR #3549 / issue #3528 - demon-llama evidence update, head SHA 0cf235ebf1

- Repo: alex-foster-personal/music-dj-tools
- Branch: af--idd-9d749cbc-issue-3528-resolve-github-issue-3528-https-github-c
- Previous evidence head: 431c8ac8b127318f16f1b9ff072bcc745b8b0c3e (screenshots + first
  21/0/0 run, posted earlier)
- New head (this update): 0cf235ebf1dd621eac0b3b43712e9e1b49d4b539
- Why the head moved: origin/main advanced 20 commits past this branch's prior merge
  base, so `just pre-push` required a real rebase. Merging origin/main produced a real
  conflict in apps/webui/frontend/src/lib/rb/performance-hotkeys.ts between this PR's
  shared performance-shortcut-routing.ts module and an independent, older inline
  isNativeInteractiveTarget-based listener that had also landed on main; resolved by
  keeping this branch's routing-module implementation (the reviewed superset that issue
  #3528 requires). Two small follow-up fixes after the merge: (1) a source-literal test
  from the P1 fix (feedback-pin-reveal.test.mjs) re-pointed at FeedbackPinLayer.svelte,
  where an unrelated already-merged pins-lane refactor (FB-16) had moved the code it
  checked; (2) a `frontend.unknown_casts` quality-gate regression from this PR's own P2
  fix, resolved by switching to this repo's existing single-cast window-twin pattern.
  Neither follow-up changes the shortcut-routing behavior itself.
- Host: demon-llama (alex@, Tailscale), same host as the original evidence, never silver
- Checkout: `~/code/music-dj-tools-wt-pr3549`, a fresh `git worktree` at the head SHA
  above (detached HEAD), clean working tree at capture time (`git status --short` empty)
- Captured: 2026-09-24T07:19Z (UTC, `date -u` on demon-llama)

## Re-verification result

```
cd ~/code/music-dj-tools-wt-pr3549/apps/webui/frontend
npx playwright test --config tests/e2e/playwright.comment-hotkey-gate.config.ts tests/e2e/comment-hotkey-browser.spec.ts --reporter=list
```

**21 passed, 0 failed, 0 skipped (1.9m)** - same committed acceptance suite, same real
backend/frontend/fixture setup as the original evidence, at the new head SHA. Includes
the exact issue #3528 acceptance-bullet test again:
"hovering a library row (no text focus) then pressing M opens the pin composer"
(comment-hotkey-browser.spec.ts:184).

Full raw output in `acceptance-3528-rerun-0cf235e.txt`, uploaded alongside this file.

The original screenshots (before-hover, after-M-armed, composer-open, negative-control
text-input) are unchanged and still accurate: this merge and its two follow-up fixes
never touched the M/hover/composer visual flow, only a Tab-key guard and two unrelated
test/quality-gate literals. They remain at their original commit-pinned URLs under
`pr-3549/demonllama-431c8ac8b1/`.

## Manifest

| file | sha256 |
|---|---|
| acceptance-3528-rerun-0cf235e.txt | 3661ca20ded78e16a6076d231f9750a9ab5c0e2389c23d74ddff91ff4b5f3f60 |
