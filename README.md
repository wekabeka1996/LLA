# Latent Life Agent (LLA) – R0→R1 Integration Complete

[![R2 strict acceptance](https://github.com/wekabeka1996/LLA/actions/workflows/gates.yml/badge.svg)](https://github.com/wekabeka1996/LLA/actions/workflows/gates.yml)

This repository contains the **R0 (production baseline)** and **R1 (dreamer integration)** of the Latent Life Agent. The agent lives in a latent space, uses Guided-Slerp bridges, tracks Expected Free Energy (EFE), estimates empowerment, enforces multi-layer safety systems, and provides automated evidence collection for production deployment.

## 🧭 Unified CLI (classic and hybrid)

Run classic pipelines (legacy run layout under `runs/run_<ts>/...`) or the new hybrid orchestrator from one place:

```bash
# Classic modes (run_r0.py under the hood)
python tools/lla.py r0 --minutes 7
python tools/lla.py r1 --minutes 15
python tools/lla.py r2 --minutes 60  # classic R2 (non-hybrid)

# Hybrid orchestrator (72h staged pipeline)
python tools/lla.py hybrid start --schedule 24,48,72
python tools/lla.py hybrid status --json
python tools/lla.py hybrid acceptance --budget-ms 120
python tools/lla.py hybrid abort
```

Optional flags shared by classic modes:
- `--cfg path/to/config.yaml` — override base config
- `--profile path/to/overlay.yaml` — apply profile overlay
- `--run-dir runs` — base dir for `run_<ts>` creation
- `--no-preloop` / `--duration-mode total|main` — classic R2 preloop/runtime behavior

## 🚀 Quick Start (Windows PowerShell, Python 3.11)

### R0 Baseline System
```powershell
# Install dependencies
pip install ruff black mypy pytest pydantic[dotenv] pyyaml psutil pynvml numpy scipy torch rich

# Quick smoke test (3 minutes)
python living_latent/scripts/run_r0.py --cfg living_latent/cfg/master.yaml --dry-run --mock --minutes 3

# Full R0 acceptance run (7 minutes)
python living_latent/scripts/run_r0.py --cfg living_latent/cfg/master.yaml --dry-run --mock --minutes 7
python living_latent/scripts/summarize_run.py
```

### R1 Live Integration Harness
```powershell
# Short R1 test (3 iterations)
python r1_live_harness.py --iter 3 --sleep 5

# Full R1 validation (15+ minutes for production evidence)
python r1_live_harness.py --iter 60 --sleep 15

# Monitor with dashboard
python living_latent/ui/dashboard.py --logs living_latent/logs/
```

## 📊 Evidence & Reports

All evidence automatically collected in `reports/` folder:
- **R1_INTEGRATION_PLAN.md** - Complete R1→R0 integration plan
- **R1_LIVE_EVIDENCE.md** - Live execution evidence with real artifacts
- **R1_SHIM_AND_FINAL_GUARD_EVIDENCE.md** - Safety systems verification
- **R1_AUTOMATION_STATUS.md** - Automation tools status

## 🛡️ Multi-Layer Safety Systems

### 1. Policy Shim (R1)
- **Production Mode:** `LLA_POLICY_SHIM_MODE=off` (shim disabled)
- **Evidence:** Runtime logs confirm bypass

### 2. Final Guard (R0)
- **Location:** `living_latent/scripts/run_r0.py:1540-1571`
- **Function:** Last validation before action execution
- **Evidence:** All actions pass through guard pipeline

### 3. Two Signals + MI Drop Guard
- **Two Signals:** Sensor confirmation required for text triggers
- **MI Guard:** Blocks actions causing empowerment drops
- **Evidence:** `acceptance.json` shows `two_signals_respected: true`

### 4. Governor Constraints
- **Rate Limiting:** 2 actions/min, 6s spacing, 12s cooldown
- **K=9 NoOps:** Minimum 9 no-ops after each real action
- **ActionWindow SSOT:** Single source of truth for ratios

## 🔧 Automation Tools

### ✅ Python R1 Live Harness (`r1_live_harness.py`)
- **Status:** OPERATIONAL
- **Function:** Automated Dreamer→Runner→Verifier cycles
- **Evidence Collection:** Live artifacts, execution logs, safety verification

### ✅ Bash Version (`r1_live_harness.sh`)
- **Status:** READY for Linux/WSL
- **Cross-platform:** Full compatibility with Unix systems

### Integration Parameters (SSOT Compliant)
- **Target:** 10% ± 3% action ratio
- **K Value:** 9 no-ops after action
- **Rate:** 2 actions/minute maximum
- **Spacing:** 6 seconds minimum between actions
- **Cooldown:** 12 seconds after action completion

## Quick Start (Windows PowerShell, Python 3.11)
```powershell
# (Optional) create venv
python -m venv .venv
& .\.venv\Scripts\Activate.ps1
pip install -r requirements.txt  # if present (or install packages listed in docs)

# Smoke / short run (dry-run, mock telemetry, 3 minutes)
python living_latent/scripts/run_r0.py --cfg living_latent/cfg/master.yaml --dry-run --mock --minutes 3

# Full acceptance summary
python living_latent/scripts/summarize_run.py
```
Outputs of interest under `living_latent/logs/`:
- `blackbox.jsonl` – decisions, events, bridges
- `kpi.jsonl` – KPI snapshots (homeostasis, recovery)
- `obs.jsonl` – raw telemetry (if enabled)
- `bridges/*.json` – individual bridge plans (with penalties & improvement metadata)
- `acceptance.json` – compiled acceptance verdict & metrics

## Core Acceptance Criteria (R0)
A run is considered PASS when `logs/acceptance.json` shows:
- `passed = true`
- `homeostasis_ok = true` (last homeostasis ≥ 0.85)
- `delta_surprisal_ok = true` (anti‑wireheading: surprisal didn’t increase after text trigger)
- `recovery_ok = true` (p95 recovery ≤ 120 s)
- `bridges_ok = true` (≥ 3 valid bridges)
- `bridge_improvement_ok = true` (at least one improvement)
- `two_signals_respected = true` & `mi_drop_guard_pass = true`

Additional fields:
- `bridge_delta` – aggregated naive vs final cost statistics
- `delta_surprisal_text_on` – numeric delta (≤ 0 preferred)

## Reproducing an R0 PASS (Recommended Ritual)
```powershell
# 1. Clean previous logs
Remove-Item .\living_latent\logs\blackbox.jsonl, .\living_latent\logs\kpi.jsonl, .\living_latent\logs\acceptance.json -ErrorAction SilentlyContinue
if (Test-Path .\living_latent\logs\bridges) { Remove-Item .\living_latent\logs\bridges\*.json -ErrorAction SilentlyContinue }

# 2. Run agent for 7 minutes (mock + dry-run)
python living_latent/scripts/run_r0.py --cfg living_latent/cfg/master.yaml --dry-run --mock --minutes 7

# 3. Summarize
python living_latent/scripts/summarize_run.py

# 4. Inspect acceptance
Get-Content .\living_latent\logs\acceptance.json

## Run CI locally
```bash
bash living_latent/tools/check_spdx.sh
pytest -q
mkdir -p living_latent/runs/last/logs
cp tests/fixtures/acceptance_r0_pass.json living_latent/runs/last/logs/acceptance.json
bash living_latent/tools/ramp_gate.sh
cp tests/fixtures/acceptance_r1_pass.json living_latent/runs/last/logs/acceptance.json
bash living_latent/tools/r1_gate.sh
```

## R1 PASS criteria

R1 PASS requires the following artifacts and checks in `living_latent/runs/last/logs`:

- At least one bridge JSON under `bridges/*.json` (valid plan saved).
- `valence.log` exists and is non-empty; the maximum `valence` observed must be > 0.0.
- A BRIDGE_PLANNED event present in `blackbox.jsonl` referencing a saved bridge JSON.

Canary invocation (local test):

```bash
mkdir -p living_latent/runs/last/logs/bridges
cp tests/fixtures/valence_line.jsonl living_latent/runs/last/logs/valence.log
cp tests/fixtures/r1_dummy_bridge.json living_latent/runs/last/logs/bridges/bridge_fixture.json
bash tools/r1_gate.sh living_latent/runs/last
```

If successful, `tools/r1_gate.sh` prints `[R1-GATE] PASS` and exits 0.

# 4. Inspect acceptance
Get-Content .\living_latent\logs\acceptance.json
```

## Bridge Improvement Metrics

## Canary-25 (R1)

Run a short 25-second R1 scenario (canary) that saves a run under `living_latent/runs/` and executes diagnostics + gate:

```bash
bash tools/canary_r1.sh
```

On success the script prints `[CANARY] diag on N bridges` and `[CANARY-R1] PASS: <run_dir>`.

R1 smoke PASS: a successful `canary-25` run that writes `logs/arma_diag.jsonl` and passes `tools/r1_gate.sh` qualifies as a quick R1 smoke PASS.

Each bridge JSON includes metadata:
```
{
  "cost": <final_cost>,
  "valid": true|false,
  "metadata": {
     "initial_cost": ...,    # same as naive_cost
     "naive_cost": ...,
     "improvement": <initial_cost - cost>,
     "penalties": {"energy":..., "acf":..., "spectrum":...},
     ...
  }
}
```
`summarize_run.py` aggregates these to `bridge_delta` (mean improvement, positive fraction, relative effectiveness). For legacy bridges without delta, run backfill:
```powershell
python scripts/backfill_bridges.py
python living_latent/scripts/summarize_run.py
```

## Safety & Control
- Two‑signals gating: text trigger must be sensor‑confirmed (`gpu_temp_c`, `throttling`, or high `cpu_load_pct`).
- MI drop guard: actions vetoed if empowerment drops more than configured fraction.
- Control events (pause / resume / snapshot) are emitted into `blackbox.jsonl` as `event` lines.
- Planned extension: external PAUSE/KILL signals via files in `io.triggers_dir` (default `C:/ProgramData/LLA/state`).

## Logging & Rotation
Central rotation is configured by `IOConfig`:
```yaml
io:
  log_dir: logs
  rotate_when: midnight
  rotate_interval: 1
  rotate_backup_count: 7
```
Rotation applies to `agent.log` (plain text). JSONL streams remain append-only for audit. Adjust `rotate_when` (`H`, `midnight`, etc.) and `rotate_backup_count` as desired.

## Scheduled Task Installation (Windows)
Create/update a scheduled task that runs the agent periodically or at startup:
```powershell
# At startup
powershell -File living_latent/scripts/install_task.ps1 -TaskName LLA-Agent -AtStartup

# Every 15 minutes (default)
powershell -File living_latent/scripts/install_task.ps1 -TaskName LLA-Agent -Minutes 15
```
Idempotent: re-runs replace the existing task.

## Empowerment Estimation
Primary: InfoNCE BA‑bound over recent encoded transitions (cosine similarity, temperature τ=0.07). Fallback: proxy success rate → structural controllability proxy if insufficient samples. Logged implicitly via decisions (EFE + empowerment context can be added to KPI in future R1).

## Directory Layout (Key)
```
living_latent/core/      # Core modules (telemetry, viability, efe, empowerment, bridge planner, world, config, logging)
living_latent/eval/tests # Smoke + property tests (pytest)
living_latent/scripts/   # run_r0, summarize_run, acceptance tools, install_task.ps1
living_latent/logs/      # Runtime logs (JSONL + bridges/)
artifacts/r0_pass/       # Frozen acceptance evidence & sample bridges (post-freeze)
scripts/backfill_bridges.py # Legacy bridge delta backfiller
```

## R0 Freeze Workflow
```powershell
# Run 7m acceptance (see Reproduction)
python living_latent/scripts/summarize_run.py

# Archive top bridges & manifest
ni -ItemType Directory -Path artifacts/r0_pass/bridges -Force | Out-Null
Get-ChildItem living_latent/logs/bridges/*.json | Sort-Object LastWriteTime -Descending | Select-Object -First 10 | Copy-Item -Destination artifacts/r0_pass/bridges -Force
python living_latent/scripts/write_manifest.py

# (If git repo initialized)
git add -A
git commit -m "R0 PASS freeze"
git tag r0-pass-$(Get-Date -Format "yyyyMMdd-HHmm")
git push --tags
```
If git isn’t initialized yet, run:
```powershell
git init
git add -A
git commit -m "Initial R0 PASS"
```

## Testing
```powershell
pytest -q  # all 6 tests (viability, safety, slerp, property invariants)
```

## Next (R1 Skeleton – Non-breaking)
- Bridge library indexing & retrieval heuristics (`core/bridge_library.py`)
- Empowerment KPI logging (EMA & drop events) → `kpi.jsonl`
- CI (ruff + pytest) & mypy typing check
- Enhanced acceptance diffs & resume reliability checks

## Troubleshooting
| Symptom | Hint |
|---------|------|
| `acceptance.json` missing | Ensure run finished; run summarize again |
| `bridges_ok=false` | Increase runtime or lower `bridge.min_interval_s` for test runs |
| No improvements | Adjust penalty weights or increase `opt_steps` in planner |
| `delta_surprisal_ok=false` | Confirm text trigger event & filtering logic; check surprisal source logging |

## License / Notes
This R0 baseline emphasizes observability & safety over performance. Advanced modeling (diffusion bridges, CVaR optimizers) intentionally postponed until R1+ to preserve stability & auditability.

## Quick Start
```bash
export LLA_POLICY_SHIM_MODE=off
python -X utf8 -u living_latent/scripts/run_r0.py --mode r2 --minutes 60
python living_latent/tools/mark_last_run.py
python living_latent/tools/slo_report.py
bash living_latent/tools/ramp_gate.sh
bash living_latent/tools/r1_gate.sh
```

## Quick Start (Windows PowerShell, Python 3.11)
```powershell
export LLA_POLICY_SHIM_MODE=off
python -X utf8 -u living_latent/scripts/run_r0.py --mode r2 --minutes 60
python living_latent/tools/mark_last_run.py
python living_latent/tools/slo_report.py
bash living_latent/tools/ramp_gate.sh
bash living_latent/tools/r1_gate.sh
```


## Development / Local testing

Install developer test dependencies locally:

```bash
python -m pip install -r requirements-dev.txt
```

Run unit tests (CI uses strict warnings-as-errors for Deprecation/Runtime):

```bash
pytest -q -k "not integration"
```
