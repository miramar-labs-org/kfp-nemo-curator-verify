# run-006 — Commentary

Narrative observations from each monitoring tick.

---

### 21:53 PDT

Run-006 is the first run using the pre-built `kfp-base-cpu:latest` image instead of inline pip installs. The preflight_check pod is currently pulling the image from GHCR — this is a one-time pull per node, so subsequent runs will be faster. The pull is succeeding (no ErrImagePull; the pipeline-runner SA was patched with a long-lived PAT-based pull secret). Once the image lands on the node and the container starts, preflight_check should complete in seconds since all deps are pre-baked.

### 21:58 PDT

Still pulling `kfp-base-cpu:latest` after 5 minutes — this is expected for a first-time node pull of a large image (python:3.11-slim + nemo-curator + presidio + spaCy with en_core_web_sm baked in). No errors, just downloading. Once the image lands, it will be cached on the node permanently and all subsequent runs (including every stage that uses the CPU image) will start in seconds rather than downloading again.

### 22:01 PDT

The `kfp-base-cpu:latest` image is 93% downloaded (3.22 of 3.47 GB). The image is large because it bakes in nemo-curator, presidio, spaCy with the en_core_web_sm model, and several other packages. This is a one-time cost — once it lands, the node cache means all future runs (CPU stages: preflight, extract, pii-redaction, report) start in seconds. Expected completion in under a minute.

### 22:06 PDT

The kfp-base-cpu image is now cached and paying off — preflight_check and extract_text both completed in under a minute with no pip install overhead. extract_text expanded 5 input files into 85 documents (good text extraction + splitting). Now quality_filter is pulling `kfp-base-gpu:latest` for the first time — the PyTorch NGC base is large and multi-layer, so this pull will take several minutes. Once cached, both GPU stages (quality_filter and deduplication) will start instantly on all future runs.

### 22:15 PDT

Four of six stages have SUCCEEDED. quality_filter accepted 82/85 docs (3.5% rejection — very clean corpus, as expected from run-005). deduplication found zero exact or fuzzy duplicates (also consistent with run-005). The GPU stages ran fast because kfp-base-gpu was already cached from the build workflow earlier today.

However, pii_redaction has revealed a gap in the kfp-base-cpu image: `en_core_web_lg` (400 MB) is not pre-baked, only `en_core_web_sm`. The presidio-analyzer NLP engine is requesting the large model, forcing a GitHub download at runtime. This needs to be fixed in `kfp-images/cpu/Dockerfile` before the next build. The download will complete (GitHub releases are reliable), so this run will still finish — just slower on pii_redaction than it should be.

### 22:19 PDT — PASS

Run-006 completed successfully — all 6 stages FINISHED. Results match run-005 exactly (82 docs, 55,245 words), confirming the pre-built base image switch didn't change any pipeline logic. The big new number this run: 615 PII instances detected and redacted across 82 docs (~7.5 per doc), which is the first run where we can actually see this metric (pii_redaction now logs to MLflow via the CPU base image with boto3 + mlflow pre-installed).

The main actionable finding: `en_core_web_lg` must be added to `kfp-images/cpu/Dockerfile`. Without it, every pii_redaction stage incurs a ~400 MB GitHub download before it can run. Once fixed and the image rebuilt, pii_redaction will start instantly like the other CPU stages. Everything else about the base image approach is working correctly.
