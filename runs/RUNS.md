# Run History — kfp-nemo-curator-verify

| Run | Date | Result | Files In | Docs Out | Words | PII Found | Notes |
|---|---|---|---|---|---|---|---|
| run-001 | 2026-06-23 | FAIL | 5 | — | — | — | PermissionError on PVC subdirs (chmod fix needed) |
| run-002 | 2026-06-23 | FAIL | 5 | — | — | — | cuDF/RAPIDS not loading (missing cudf-cu12 explicit install) |
| run-003 | 2026-06-23 | FAIL | 5 | 0 | — | — | 100% quality rejection (symbol ratio was char-level, not word-level) |
| run-004 | 2026-06-23 | FAIL | 5 | 82 | 55,245 | — | ModuleNotFoundError: boto3 in curator_report (mlflow.log_dict removed) |
| run-005 | 2026-06-23 | PASS | 5 | 82 | 55,245 | — | First full success — baseline run (inline pip installs) |
| run-006 | 2026-06-23 | PASS | 5 | 82 | 55,245 | 615 | First run on pre-built kfp-base-cpu + kfp-base-gpu images; en_core_web_lg missing from CPU image (400MB download at runtime) |
