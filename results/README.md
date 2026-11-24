# Results

Raw output of the measurement scripts (JSON) and a generated summary. Every number in [the quality report](../docs/quality-report.md) comes from a file in this directory; `npm run results:tables` rebuilds the tables. Files are produced by `npm run measure` and `npm run experiments`; see the README for the commands. Seller: The Diamond Empire, <https://thedempire.shop/>.

| File | Produced by |
|---|---|
| `test-run.json` | `npm run test:record` |
| `e1-isolation-fuzz.json` | experiment E1 (`npm run experiments`) |
| `e2-webhook-replay.json` | experiment E2 |
| `e3-ratelimit.json` | experiment E3 |
| `run-metadata.json` | the experiment runner |
| `licenses.json` | `npm run licenses` |
| `dependency-audit.json` | `npm run measure` |
| `repo-scan.json` | `npm run scan:repo` (through `npm run measure`) |
| `contrast.json` | `npm run contrast` |
| `build-smoke.json` | `npm run smoke:build` |
| `measurement-run.json` | `npm run measure` |
| `summary.md` | `npm run results:tables` |
