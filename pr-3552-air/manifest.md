# PR #3552 / issue #3529 - Air evidence manifest

- Repository under test: `alex-foster-personal/music-dj-tools`
- PR: #3552
- Issue: #3529 "fix(mixer): deck level meters ignore the channel fader"
- Head SHA (full, exact): `8fac1ec9a081ea197ab46b301acea1c408fb1bbd`
- Host: `Alexs-MacBook-Air-3.local` (Alex's real Air, the literal machine issue #3529's
  acceptance bullet names - not a substitute like silver)
- Checkout: fresh `git worktree add` at `~/Music/music-dj-tools-wt-pr3552-evidence`,
  clean, advanced by `git fetch` + `git checkout <sha>` (never `git reset --hard`,
  which is banned repo-wide) each time the head moved
- Backend: standalone `just engine-serve` process against a private copy of a real
  ~10,003-track rekordbox library at `/tmp/pr3552-evidence-data/data`, on a
  legitimately claimed worktree port pair (`MUSIC_DJ_BACKEND_PORT=8710`,
  `MUSIC_DJ_FRONTEND_PORT=9430`) - never pointed at another agent's running engine
- Capture tool: Playwright (`chromium.launch()`), 1600x1000 viewport, script
  `capture-3552-air.mjs` (copied onto the Air so Node resolves `node_modules`
  relative to the running script, not `/tmp`)
- Flow: navigate to `/performance`, search-filter and load deck 1 with a real,
  `ls -la`-verified-present track ("Dr Packer Remix"), confirm load+play, then
  sample the deck 1 channel meter and master meter (`aria-valuenow` off the real
  `[role="meter"]` DOM nodes - no audio was heard, every number below is a DOM
  read of the value the production meter component actually computed) at 20
  samples over ~3s at each channel-fader position: full (1.0) -> half (0.5) ->
  zero (0.0) -> restored full (1.0), dragging the real `VFader` component with
  real mouse down/move/up events (not a synthetic value set)

## Deck 2 / isolation-control limitation

Deck 2 could not be reliably loaded on this run: every attempt hit
`cannot load: availability still checking (wait for disk probe)`, traced to a
pre-existing library-availability-probe TTL/caching interaction
(`FILE_EXISTS_TTL_S`, issue #1037 / PERF-RB-01) that is unrelated to this PR's
own changes, most visible right after copying a cold library onto a fresh engine
instance. Deck 2 loading was treated as best-effort and non-fatal since issue
#3529's literal acceptance text requires only "play A deck" (singular). This
means the master-meter reading below is **not a true isolation control** -
master necessarily tracks deck 1 exactly since deck 1 was the only playing
source, unlike the earlier silver-based capture where a second, independently
playing deck showed the master and an unrelated channel moving separately.

## Measured (deck 1 channel meter, dBFS, n=20 per position)

| Fader position | Mean | Median |
|---|---|---|
| Full (1.0) | -2.50 | -2.32 |
| Half (0.5) | -7.74 | -7.32 |
| Zero (0.0) | -48.66 (decaying to the floor within the sample window) | -60.00 (floor) |
| Restored full (1.0) | see `summary.json` | see `summary.json` |

- Full -> half delta: 5.24 dB (mean) / 5.00 dB (median) - within the acceptance
  criterion's "about 6 dB" (+/-1 dB window, at the edge on the median)
- Full -> zero delta (mean): 46.16 dB - the meter falls all the way to the
  documented -60 dBFS floor

## Master meter (same instants, dBFS, n=20 per position)

| Fader position | Mean | Median |
|---|---|---|
| Full (1.0) | -5.87 | -5.88 |
| Half (0.5) | -9.51 | -9.50 |
| Zero (0.0) | -49.93 | -60.00 (floor) |

(Moves with deck 1 as expected, since deck 1 was the only playing source - see
the isolation-control limitation above.)

## Artifacts

| File | SHA-256 |
|---|---|
| `01-fader-full.png` | `0ad450921e21e7dc349e5294f7834c919a6e7706c171ec7ce600085f385de649` |
| `02-fader-half.png` | `457493e0030f35dfff6a3f488b2f702eba97ae36b6194be5b87d7780de30a02a` |
| `03-fader-zero.png` | `b97c9b049f07d8961d871e0557b460eef321325e924f6b87a640e2e1e87fedab` |
| `04-fader-restored-full.png` | `de45b1fb07ed671cdb8378d0c42d3b9c8c26a903b73a6f02077183d758fb9828` |

Full sample series and event log: `state.json`. Aggregated stats: `summary.json`.
