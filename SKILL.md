---
name: Code Soothsayer
description: AI-powered code failure prediction system that analyzes codebases to identify potential bugs, runtime failures, and vulnerabilities before they manifest in production
version: 2.4.1
author: Kilo Engineering Team
tags:
  - prediction
  - failures
  - code
  - bugs
  - static-analysis
  - ml
dependencies:
  - python>=3.9
  - tree-sitter>=0.20.0
  - torch>=2.0.0
  - transformers>=4.30.0
  - astroid>=2.15.0
  - bandit>=1.7.0
  - semgrep>=1.50.0
  - pre-commit>=3.0.0
required_services: []
conflicts:
  - old-soothsayer
  - legacy-analyzer
---

# Purpose

Code Soothsayer predicts code failures before they manifest by combining static analysis, machine learning models trained on millions of bug-fix commits, and runtime pattern recognition. It identifies subtle issues that traditional linters miss: race conditions, resource leaks, API misuse, security vulnerabilities, and architectural flaws.

**Real use cases:**
- Prevent production outages by catching race conditions in async Go code (`soothsayer analyze --lang=go ./services/payment --pattern=race`)
- Identify memory leaks in C++ before they consume all RAM (`soothsayer predict --lang=cpp --memory-profile=high ./src/core`)
- Discover SQL injection vulnerabilities in legacy Python 2 code (`soothsayer check --lang=py --version=2.7 ./legacy/app.py`)
- Predict which dependency upgrades will break your build (`soothsayer simulate --upgrade=true ./package.json`)
- Find unreachable code paths in complex React components (`soothsayer analyze --framework=react --dead-code ./components`)

---

# Scope

## Commands

### `analyze`
Deep static analysis with pattern matching and ML inference.

**Flags:**
- `--lang=<language>` (required): `python`, `javascript`, `go`, `rust`, `java`, `cpp`, `ruby`, `swift`, `kotlin`, `typescript`
- `--pattern=<type>`: `race`, `memory`, `null`, `sql`, `xss`, `dead-code`, `all` (default)
- `--output=<format>`: `json`, `sarif`, `terminal`, `html` (default: `terminal`)
- `--severity-threshold=<int>`: 1-5 (default: 3)
- `--include-paths=<glob>`: e.g., `src/**,services/**`
- `--exclude-paths=<glob>`: e.g., `node_modules/**,test/**`
- `--context-lines=<int>`: Lines of context around issue (default: 3)
- `--confidence-threshold=<float>`: 0.0-1.0 (default: 0.7)
- `--no-cache`: Skip cached results
- `--fix-suggestions`: Include auto-fix patches (experimental)

**Example:**
```bash
soothsayer analyze --lang=python --pattern=memory --output=sarif --severity-threshold=4 ./app/ > report.sarif
```

### `predict`
Predict future failures based on code evolution patterns and developer behavior.

**Flags:**
- `--horizon=<time>`: `1w`, `1m`, `3m`, `6m` (default: `3m`)
- `--risk-threshold=<float>`: 0.0-1.0 (default: 0.5)
- `--hotspot-analysis`: Focus on high-change files
- `--developer-risks`: Account for developer experience levels (requires `--git-repo`)
- `--git-repo=<path>`: Path to git repo (default: current)
- `--since=<commit>`: Analyze commits since (default: last 100)
- `--output=<format>`: `json`, `table`, `graph` (default: `table`)

**Example:**
```bash
soothsayer predict --horizon=1m --hotspot-analysis --git-repo=./myproject --output=graph > risk-graph.html
```

### `check`
Quick validation of specific files or directories.

**Flags:**
- `--strict`: Fail on any warning
- `--fail-under=<float>`: Minimum score (0-100) to pass (default: 80)
- `--ci`: CI/CD optimized output (machine parseable)
- `--baseline=<file>`: Compare against baseline report
- `--diff`: Only check changed files (requires git)
- `--format=<format>`: `compact`, `detailed`, `summary` (default: `detailed`)

**Example:**
```bash
soothsayer check --strict --ci --diff ./src/ | tee soothsayer.log
exit ${PIPESTATUS[0]}
```

### `simulate`
Simulate the impact of changes (dependency upgrades, refactors, config changes).

**Flags:**
- `--upgrade=<spec>`: e.g., `django:4.2`, `'requests:*'`
- `--refactor=<pattern>`: `extract-method`, `rename-variable`, `split-class`
- `--config-change=<file>`: Path to modified config
- `--dry-run`: Show predictions without applying changes
- `--confidence`: Show confidence intervals
- `--compare-to=<commit>`: Compare predictions before/after

**Example:**
```bash
soothsayer simulate --upgrade='django:4.2,react:18.0' --dry-run --confidence ./ > upgrade-predictions.txt
```

### `learn`
Retrain or fine-tune models with your own codebase data.

**Flags:**
- `--dataset=<path>`: Path to labeled dataset (CSV/JSON)
- `--model-type=<type>`: `bug-classifier`, `race-detector`, `memory-analyzer` (default: `bug-classifier`)
- `--epochs=<int>`: Training epochs (default: 10)
- `--validate-split=<float>`: Validation split ratio (default: 0.2)
- `--export=<path>`: Export trained model
- `--import-model=<path>`: Import existing model for fine-tuning
- `--eval-only`: Only evaluate, don't train

**Example:**
```bash
soothsayer learn --dataset=./historical-bugs.csv --model-type=bug-classifier --epochs=20 --export=./custom-model.pt
```

### `report`
Generate comprehensive reports from previous analyses.

**Flags:**
- `--input=<file>`: Input JSON/SARIF report (default: `soothsayer-report.json`)
- `--format=<format>`: `pdf`, `html`, `markdown`, `json` (default: `html`)
- `--template=<name>`: `executive`, `developer`, `audit` (default: `developer`)
- `--since=<date>`: Filter by date (ISO 8601)
- `--group-by=<field>`: `file`, `severity`, `pattern`, `developer` (default: `file`)
- `--include-fixes`: Include suggested fixes in report

**Example:**
```bash
soothsayer report --input=last-scan.json --format=pdf --template=executive --since=2024-01-01 > q1-report.pdf
```

### `validate-model`
Verify model accuracy against known test cases.

**Flags:**
- `--test-suite=<path>`: Path to test suite directory
- `--benchmark`: Run benchmark suite (takes ~30min)
- `--precision-threshold=<float>`: Minimum precision (default: 0.85)
- `--recall-threshold=<float>`: Minimum recall (default: 0.75)
- `--output-dir=<path>`: Where to save results (default: `./validation-results/`)

**Example:**
```bash
soothsayer validate-model --test-suite=./tests/validation/ --benchmark --precision-threshold=0.90
```

---

# Work Process

## Standard Workflow

1. **Initialization**
   ```bash
   soothsayer init --project-type=webapp --langs=python,js
   ```
   Creates `.soothsayer.yml` config file and caches language models.

2. **Baseline Analysis**
   ```bash
   soothsayer analyze --lang=python --output=json > baseline.json
   ```
   Establishes initial failure prediction baseline.

3. **CI/CD Integration**
   ```bash
   soothsayer check --strict --ci --diff
   ```
   Runs on every PR/commit, fails if new high-severity predictions detected.

4. **Periodic Prediction**
   ```bash
   soothsayer predict --horizon=1m --hotspot-analysis > risk-forecast.json
   ```
   Scheduled weekly to identify emerging risk areas.

5. **Pre-Upgrade Simulation**
   ```bash
   soothsayer simulate --upgrade='package.json' --dry-run > upgrade-risks.json
   ```
   Before any dependency upgrades.

## Advanced Workflow: Multi-Stage Gate

```bash
# Stage 1: Fast check on changed files
soothsayer check --diff --format=compact > quick-check.txt

# Stage 2: Deep analysis on hotspots (only if Stage 1 finds issues)
if grep -q "SEVERITY.*[45]" quick-check.txt; then
  soothsayer analyze --lang=python --severity-threshold=3 ./src/ > deep-analysis.json
fi

# Stage 3: Predict future failures in modified modules
soothsayer predict --hotspot-analysis --git-repo=. --since=HEAD~5 > future-risks.json

# Stage 4: Block merge if risk score > threshold
RISK_SCORE=$(jq '.risk_score' future-risks.json)
if (( $(echo "$RISK_SCORE > 0.7" | bc -l) )); then
  echo "Merge blocked: predicted failure risk too high (score: $RISK_SCORE)"
  exit 1
fi
```

---

# Golden Rules

1. **Never ignore high-confidence predictions (≥0.9) with severity 4-5** - these correlate 94% with production incidents based on our internal data.

2. **Always use `--diff` in CI** - analyzing the entire codebase on every commit is wasteful and produces noise.

3. **Set `--confidence-threshold` appropriately for your codebase maturity:**
   - New projects: 0.5 (more permissive)
   - Mature projects: 0.8 (strict)
   - Critical systems: 0.95 (very strict)

4. **Review and retrain models quarterly** - code patterns evolve, models degrade without fresh data.

5. **Never disable caching in production** - full analysis without cache is 10-50x slower.

6. **Always pair `predict` with `--hotspot-analysis`** - predicting on low-activity files produces false positives.

7. **Treat `soothsayer check` exit codes as canonical:**
   - 0: No issues found
   - 1: Issues found but within threshold
   - 2: Threshold exceeded (fail)
   - 3: Configuration error
   - 4: Analysis failed (crash)

8. **Never commit auto-generated fixes without manual review** - the fix suggestions are right 78% of the time for simple issues, but only 42% for complex ones.

9. **Always run `validate-model` after custom training** - don't deploy models that don't meet precision/recall thresholds.

10. **Configure `--exclude-paths` for generated code and vendored dependencies** - analyzing 3rd party code wastes time and creates false positives.

---

# Examples

## Example 1: Detecting Race Condition in Go Service

**User command:**
```bash
soothsayer analyze --lang=go --pattern=race --confidence-threshold=0.8 --output=json ./services/order/
```

**Output (formatted):**
```json
{
  "scan_id": "scan_20240308_001",
  "timestamp": "2024-03-08T14:23:45Z",
  "predictions": [
    {
      "file": "services/order/processor.go",
      "line": 247,
      "column": 12,
      "pattern": "race",
      "severity": 5,
      "confidence": 0.92,
      "title": "Potential race condition in order status update",
      "description": "Unsynchronized access to order.status between goroutines. The order object is shared across multiple workers without mutex protection.",
      "code_snippet": "func (w *OrderWorker) process(o *Order) {\n    if o.Status == PENDING {  // RACE HERE\n      o.Status = PROCESSING\n    }\n  }",
      "fix_suggestion": "Add sync.Mutex to Order struct and lock around status checks:\n\nvar mu sync.Mutex\nmu.Lock()\nif o.Status == PENDING {\n  o.Status = PROCESSING\n}\nmu.Unlock()",
      "cwe_id": "CWE-362",
      "references": [
        "https://go.dev/doc/race",
        "https://cwe.mitre.org/data/definitions/362.html"
      ]
    }
  ],
  "statistics": {
    "total_files": 42,
    "files_analyzed": 41,
    "predictions_count": 1,
    "analysis_time_seconds": 12.3,
    "cache_hit_rate": 0.73
  }
}
```

**User action:** Developer reviews the prediction, confirms the race condition, adds mutex protection, and re-runs analysis to verify fix.

---

## Example 2: Predicting Failure After Dependency Upgrade

**User command:**
```bash
soothsayer simulate --upgrade='django:4.2,psycopg2:3.1' --dry-run --confidence --output=table ./backend/
```

**Output (formatted):**
```
Simulation Results (Dry Run)
Generated: 2024-03-08 14:30:15 UTC
Project: ./backend/
Upgrades: django:4.2, psycopg2:3.1

┌──────────────────────────────────────┬─────────┬─────────────┬─────────────────────────────────────────────┐
│ Issue                                │ Severity│ Confidence │ Impact                                      │
├──────────────────────────────────────┼─────────┼─────────────┼─────────────────────────────────────────────┤
│ Django 4.2 drops support for         │   4     │    0.87     │ 3 modules will fail on import after upgrade │
│   django.utils.six                   │         │             │                                             │
│ Psycopg2 3.0+ requires PostgreSQL    │   3     │    0.91     │ Connection failures if PG < 12               │
│   12+                                 │         │             │                                             │
│ Django 4.2 changes default           │   3     │    0.78     │ All 'url' template tags must be updated     │
│   'url' tag to 'uri'                 │         │             │                                             │
└──────────────────────────────────────┴─────────┴─────────────┴─────────────────────────────────────────────┘

Summary:
- 3 issues detected (1 critical, 2 medium)
- Estimated upgrade effort: 8-12 hours
- Risk score: 0.76 (HIGH)
Recommendation: Address issues before upgrading. See full details in django-upgrade-plan.md (generated with --report flag).
```

**User action:** Developer spends a day updating code for Django 4.2 compatibility, then re-runs simulation to verify all issues resolved before actual upgrade.

---

## Example 3: CI/CD Block on High-Risk Change

**User command (in CI pipeline):**
```bash
soothsayer check --strict --ci --diff --output=compact
```

**Output when failure detected:**
```
::error::Code Soothsayer detected critical issues
Scan ID: scan_ci_83274
Files changed: 3
Prediction: HIGH_RISK

ISSUE 1: services/auth/middleware.py:89:5
  Pattern: sql
  Severity: 5
  Confidence: 0.94
  Title: Potential SQL injection in user lookup
  Fix: Use parameterized queries

ISSUE 2: tests/test_api.py:234:12
  Pattern: dead-code
  Severity: 3
  Confidence: 0.81
  Title: Test function never called (dead code)

Overall Risk Score: 0.83 (threshold: 0.5)
```

**CI system output:**
```
##[error]Process completed with exit code 2.
```

**User action:** Developer fixes SQL injection vulnerability, removes dead code, pushes new commit, CI passes.

---

## Example 4: Long-Term Risk Prediction

**User command:**
```bash
soothsayer predict --horizon=6m --hotspot-analysis --developer-risks --output=graph ./src/
```

**Output (graph saved to `risk-forecast.html` with interactive D3.js visualization):**

Interactive HTML with:
- Heat map of files by predicted failure probability (red = high risk)
- Timeline showing risk trend over 6 months
- Developer impact overlay (files touched by junior engineers flagged)
- Click-to-drill-down to specific prediction details

**Text summary also printed:**
```
Risk Forecast Summary (6 month horizon)
Project: ./src/
Hotspots identified: 7 files

Top 3 High-Risk Files:
1. src/utils/cache.py (risk: 0.87) - Memory leak pattern, complex caching logic
2. src/api/auth.py (risk: 0.82) - Authentication bypass potential, touched recently by 3 devs
3. src/workers/order.py (risk: 0.79) - Race conditions in async worker

Recommendation: Prioritize refactoring cache.py before Q3 release.
```

**User action:** Tech lead schedules sprint to refactor high-risk files, assigns senior engineer to cache.py due to complexity.

---

# Rollback Commands

## Rollback Analysis State

**Clear analysis cache:**
```bash
soothsayer cache --clear --all
```

**Restore from backup (if `.soothsayer-backup/` exists):**
```bash
soothsayer restore --backup-dir=.soothsayer-backup --date=2024-03-07
```

## Rollback Model Updates

**Revert to previous model version:**
```bash
soothsayer model --version=2.3.0 --set-default
```

**Disable custom model and use defaults:**
```bash
soothsayer model --reset-to-default
```

## Undo Auto-Fixes

**Revert applied fixes (git-based):**
```bash
git revert --no-commit HEAD~N  # N = number of fix commits
```

**List applied fixes:**
```bash
soothsayer history --applied-fixes
```
Output:
```
Applied Fixes:
2024-03-07 10:23: fix-race-condition-abc123/src/worker.py
2024-03-06 14:45: fix-sql-injection-def456/auth/middleware.py
```

**Selective revert of specific fix:**
```bash
soothsayer revert --fix-id=fix-race-condition-abc123
```

## Revert Prediction Thresholds

**Restore previous thresholds from config backup:**
```bash
cp .soothsayer.backup.yml .soothsayer.yml
soothsayer config --reload
```

## Full System Rollback

**If soothsayer causes issues in CI:**
```bash
# Disable in pre-commit temporarily
soothsayer disable --global --pr-mode

# Or remove from pre-commit config entirely
sed -i '/soothsayer/d' .pre-commit-config.yaml

# Restore previous working configuration
git checkout HEAD~1 -- .soothsayer.yml
```

---

# Verification Steps

## After Installation
```bash
soothsayer --version  # Should print 2.4.1
soothsayer validate-model --test-suite=./tests/smoke/  # Should pass
soothsayer analyze --lang=python ./tests/sample/  # Should produce predictions
```

## After Configuration Change
```bash
soothsayer config --validate  # Verify YAML syntax
soothsayer analyze --lang=python ./tests/sample/ --no-cache  # Ensure cache not used
```

## After Model Training
```bash
soothsayer validate-model --test-suite=./tests/validation/ --precision-threshold=0.85 --recall-threshold=0.75
# Exit code 0 = model acceptable, non-zero = reject
```

## CI/CD Integration Test
```bash
# Simulate CI environment
soothsayer check --ci --diff --format=compact > ci-output.txt
if [ $? -eq 2 ]; then
  echo "CI would fail - issues detected"
  cat ci-output.txt
  exit 1
fi
```

---

# Troubleshooting

## Issue: Analysis extremely slow (5+ minutes per file)
**Cause:** No cache configured or cache directory full.
**Fix:**
```bash
# Check cache settings
soothsayer config --get cache_dir

# Clear and rebuild cache
soothsayer cache --clear
soothsayer analyze --no-cache=false ./  # Re-populate cache

# Increase cache TTL in .soothsayer.yml:
# cache_ttl_hours: 168  # (default 24)
```

## Issue: "Model not found" errors
**Cause:** Models not downloaded or corrupted.
**Fix:**
```bash
# Re-download all models (5-10GB)
soothsayer models --download --force

# Verify installation
soothsayer models --list
```

## Issue: High false positive rate
**Cause:** Confidence threshold too low for your codebase.
**Fix:**
```bash
# Increase confidence threshold
soothsayer analyze --confidence-threshold=0.85 ./src/

# Persist in config
soothsayer config --set confidence_threshold=0.85

# Report false positives to improve models
soothsayer feedback --false-positive --file=utils.py --line=123 --pattern=race
```

## Issue: Out of memory during analysis
**Cause:** Large files or insufficient RAM for ML models.
**Fix:**
```bash
# Use batch mode (process files sequentially)
soothsayer analyze --batch-size=10 ./src/

# Exclude large generated files
# Add to .soothsayer.yml:
# exclude_paths:
#   - "**/migrations/*.py"
#   - "**/*.pb.go"

# Or use smaller model
soothsayer analyze --model=lightweight ./src/
```

## Issue: CI/CD jobs timing out
**Cause:** Full analysis on every commit.
**Fix:**
```bash
# Always use --diff in CI
soothsayer check --ci --diff --format=compact

# Cache across CI runs (GitLab example):
# before_script:
#   - soothsayer cache --restore --remote=ci-cache
# after_script:
#   - soothsayer cache --save --remote=ci-cache
```

## Issue: Pre-commit hooks failing with "permission denied"
**Cause:** soothsayer not in PATH or incorrect shebang.
**Fix:**
```bash
# Ensure soothsayer in PATH for hook environment
which soothsayer  # Should return path

# Reinstall pre-commit hooks
pre-commit uninstall-all
pre-commit install

# Or use absolute path in .pre-commit-config.yaml:
# - repo: local
#   hooks:
#     - id: soothsayer
#       entry: /usr/local/bin/soothsayer
```

## Issue: Predictions don't match actual runtime failures
**Cause:** Models may be outdated or not trained on your domain.
**Fix:**
```bash
# Collect your own failure data (last 6 months of incident tickets)
soothsayer collect-incidents --since=2024-01-01 --output=incidents.json

# Retrain with domain-specific data
soothsayer learn --dataset=incidents.json --model-type=bug-classifier --epochs=30

# A/B test: compare old vs new predictions
soothsayer predict --model=default ./src/ > old-predictions.json
soothsayer predict --model=custom ./src/ > new-predictions.json
soothsayer compare old-predictions.json new-predictions.json
```

## Issue: SARIF output not showing in GitHub Code Scanning
**Cause:** SARIF file format incorrect or too large.
**Fix:**
```bash
# Generate proper SARIF
soothsayer analyze --output=sarif --max-sarif-size=100MB ./src/ | gzip -c > results.sarif.gz

# Upload using GitHub CLI
gh code-scanning upload --sarif results.sarif.gz

# Or ensure proper format
soothsayer analyze --output=sarif --sarif-version=2.1.0 ./src/
```

---

# Environment Variables

- `SOOTHSAYER_HOME`: Override default config directory (`~/.soothsayer/`)
- `SOOTHSAYER_CACHE_DIR`: Override cache directory
- `SOOTHSAYER_MODEL_DIR`: Override model storage location
- `SOOTHSAYER_CONFIG`: Path to custom config file (overrides `.soothsayer.yml`)
- `SOOTHSAYER_LOG_LEVEL`: `DEBUG`, `INFO`, `WARNING`, `ERROR` (default: `INFO`)
- `SOOTHSAYER_DISABLE_ANALYTICS`: Set to `true` to disable usage reporting
- `SOOTHSAYER_MAX_WORKERS`: Number of parallel analysis workers (default: number of CPU cores)
- `SOOTHSAYER_MEMORY_LIMIT_MB`: Maximum memory usage before throttling (default: 4096)

Example:
```bash
export SOOTHSAYER_CACHE_DIR=/mnt/ssd/soothsayer-cache
export SOOTHSAYER_MAX_WORKERS=8
soothsayer analyze --lang=python ./large-project/
```

---

# Exit Codes

- `0`: Success (no issues or issues within threshold)
- `1`: Issues found but within acceptable threshold (non-fatal)
- `2`: Issues exceed severity/risk threshold (fail for CI)
- `3`: Configuration error (invalid YAML, missing required flags)
- `4`: Analysis failed (crash, out of memory, model load error)
- `5`: Invalid arguments or unknown command
- `6`: Dependency missing (run `soothsayer doctor` to diagnose)

---

# Support

- Report bugs: `soothsayer report-bug`
- View documentation: `soothsayer docs --topic=troubleshooting`
- Check system health: `soothsayer doctor`
- Get version info: `soothsayer version --full`

```