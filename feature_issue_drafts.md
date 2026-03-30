# Feature Issue Drafts

Use `.github/ISSUE_TEMPLATE/feature_request.md` for all of these when creating issues manually in GitHub.

## Issue 1

**Title:** `Add derived efficiency metrics to emissions tracking outputs`

```md
**Is your feature request related to a problem? Please describe.**
CodeCarbon reports energy use and emissions, but many ML practitioners also need normalized efficiency metrics to compare runs fairly across datasets and model sizes. Today users have to compute these manually outside the library, which makes experiment comparison inconsistent and error-prone.

**Describe the solution you'd like**
Add optional derived efficiency metrics to the tracker and output methods, such as:
- emissions per sample
- energy per sample
- emissions per epoch
- emissions per prediction
- emissions per token for LLM inference/training
- emissions per training step

This could be implemented by allowing users to pass workload counters during or after a run, then including the derived metrics in CSV/API/output-method payloads.

A possible API shape could be:
- tracker metadata fields set at initialization
- tracker methods to update counters during execution
- derived fields included in emitted records only when counters are available

**Describe alternatives you've considered**
Users can already compute these metrics themselves from `emissions.csv` and external metadata, but that leads to duplicated logic and inconsistent definitions across teams.
Another option would be to provide only examples in the documentation, but that would not standardize outputs across integrations.

**Additional context**
Relevant code paths:
- `codecarbon/emissions_tracker.py`
- `codecarbon/output_methods/emissions_data.py`
- `codecarbon/output.py`

This would be especially useful for ML training, inference benchmarking, and LLM workloads where per-token or per-sample reporting is often more actionable than total run emissions.
```

## Issue 2

**Title:** `Add first-class examples for Hugging Face and PyTorch training/inference workflows`

```md
**Is your feature request related to a problem? Please describe.**
CodeCarbon is widely relevant to ML workflows, but the repo currently lacks enough polished, modern examples for common training and inference patterns used by ML engineers and data scientists. This creates friction for adoption and makes it harder to understand best practices for real-world usage.

**Describe the solution you'd like**
Add a set of example scripts and docs for common ML workflows, such as:
- Hugging Face Trainer fine-tuning
- plain PyTorch training loop
- evaluation/inference batch jobs
- notebook-style usage
- optional distributed training example with per-process tracking notes

The examples should show:
- tracker setup
- best placement of `start()` / `stop()`
- experiment metadata
- output handling
- caveats for multi-process or GPU-heavy workloads

**Describe alternatives you've considered**
Users can adapt the existing examples manually, but examples focused on generic usage do not answer the practical questions that come up in modern ML pipelines.
Another option would be to document this only in the website, but runnable examples in the repository are easier to validate and maintain.

**Additional context**
Relevant repo areas:
- `examples/`
- `codecarbon/emissions_tracker.py`
- documentation under `docs/getting-started/`

This would be a good contributor-friendly feature because it improves adoption without requiring changes to the core estimation logic.
```

## Issue 3

**Title:** `Improve fallback CPU power estimation using calibration datasets`

```md
**Is your feature request related to a problem? Please describe.**
When direct hardware power readings are unavailable, CodeCarbon falls back to estimation logic. Those fallback paths are essential for broad usability, but they can be less accurate for modern CPUs and heterogeneous workloads. There is an opportunity to improve estimation quality using the calibration assets already present in the repository.

**Describe the solution you'd like**
Improve CPU fallback estimation by using calibration datasets and benchmark traces already available in the repo to refine the current power model.

Possible scope:
- review current fallback assumptions
- compare estimated values against calibration traces
- fit a better mapping from utilization/workload profile to estimated CPU power
- validate behavior across several CPU families
- document limitations and expected accuracy

Ideally this would improve estimation quality without breaking the current public API.

**Describe alternatives you've considered**
The current fallback logic is better than having no estimate, but it can remain coarse for unsupported or partially supported hardware.
Another alternative is to add more static hardware tables only, but that does not address dynamic workload-dependent estimation quality.

**Additional context**
Relevant repo areas:
- `codecarbon/core/cpu.py`
- `codecarbon/core/resource_tracker.py`
- `codecarbon/data/hardware/cpu_load_profiling/`
- `codecarbon/data/hardware/cpu_dataset_builder/`

This feature is a good fit for contributors with data science, benchmarking, and modeling experience.
```

## Issue 4

**Title:** `Report uncertainty bounds for emissions estimates`

```md
**Is your feature request related to a problem? Please describe.**
CodeCarbon currently reports a single emissions estimate, but the true estimate quality depends on hardware support, region inference quality, cloud metadata availability, and fallback paths. Presenting only a point estimate can make the output look more precise than it really is.

**Describe the solution you'd like**
Add uncertainty-aware reporting for emissions estimates.

Possible output:
- confidence or quality level
- uncertainty range or interval
- flags describing which parts of the estimate were direct measurements vs fallbacks
- optional summary of main uncertainty sources

This could initially be heuristic-based, for example:
- lower uncertainty for direct hardware measurements
- higher uncertainty for CPU/GPU fallback paths
- higher uncertainty when location or carbon intensity must be inferred approximately

**Describe alternatives you've considered**
The current approach is simple and easy to consume, but it does not communicate estimate reliability.
Another alternative would be to improve methodology documentation only, but surfacing uncertainty directly in outputs would be more actionable for users.

**Additional context**
Relevant repo areas:
- `codecarbon/core/emissions.py`
- `codecarbon/core/measure.py`
- `codecarbon/core/resource_tracker.py`
- `docs/introduction/methodology.md`

This would be especially valuable for research users and sustainability reporting where transparency about measurement quality matters.
```

## Issue 5

**Title:** `Improve cloud and region inference for modern ML infrastructure`

```md
**Is your feature request related to a problem? Please describe.**
Many ML workloads run on managed cloud environments, and accurate emissions estimation depends heavily on region and infrastructure detection. Current cloud inference can be improved for modern ML deployment patterns such as managed notebooks, training jobs, and hybrid cloud usage.

**Describe the solution you'd like**
Improve cloud and region inference so CodeCarbon can more accurately detect or represent:
- managed ML environments
- multi-region or region-ambiguous workloads
- cloud provider metadata edge cases
- spot/preemptible instances
- hybrid or partially inferred environments

Potential improvements could include:
- broader metadata detection coverage
- clearer fallback logic
- surfacing inference quality in logs/output
- more test coverage around cloud environment detection

**Describe alternatives you've considered**
Users can manually provide region and cloud-related configuration, but automatic detection is more convenient and less error-prone.
Another option is to focus only on documentation, but better detection logic would improve the default experience.

**Additional context**
Relevant repo areas:
- `codecarbon/core/cloud.py`
- `codecarbon/core/electricitymaps_api.py`
- `tests/test_cloud.py`

This would be valuable for teams running training and inference on AWS, Azure, GCP, or managed ML platforms.
```

## Issue 6

**Title:** `Add distributed training emissions aggregation support`

```md
**Is your feature request related to a problem? Please describe.**
Modern ML training often runs across multiple processes, GPUs, or nodes. In those settings, it is difficult to obtain a single coherent emissions report for the whole job. Users currently have to aggregate emissions manually or rely on process-local measurements.

**Describe the solution you'd like**
Add first-class support for aggregating emissions across distributed training workers.

Possible capabilities:
- identify a logical run shared across workers
- aggregate worker-local measurements into one final run summary
- support common distributed setups such as multi-GPU and multi-node training
- document recommended patterns for launcher-based workflows

This could be implemented in stages, starting with explicit aggregation hooks and a documented reference example.

**Describe alternatives you've considered**
Manual aggregation outside the library is possible, but it is repetitive and easy to get wrong.
Another alternative is to provide examples only, but built-in support would make distributed tracking much more reliable.

**Additional context**
Relevant repo areas:
- `codecarbon/emissions_tracker.py`
- `codecarbon/core/api_client.py`
- output and API-related code paths

This would be a high-impact feature for large-scale ML users.
```

## Issue 7

**Title:** `Add MLflow output integration for tracked emissions`

```md
**Is your feature request related to a problem? Please describe.**
Many ML teams use MLflow to track experiments. Today, integrating CodeCarbon outputs with MLflow requires custom glue code, which slows adoption and creates inconsistent logging conventions.

**Describe the solution you'd like**
Add a built-in MLflow output method so emissions metrics and metadata can be logged directly to an active MLflow run.

Potential data to log:
- total emissions
- total energy consumed
- duration
- country/region if available
- hardware summary
- derived efficiency metrics when available

This could be implemented as a new output method consistent with the existing output method architecture.

**Describe alternatives you've considered**
Users can manually log CodeCarbon results to MLflow after a run, but that duplicates boilerplate and can lead to missing or inconsistent fields.
Another option would be to maintain examples only, but a built-in output method would be much easier to adopt.

**Additional context**
Relevant repo areas:
- `codecarbon/output_methods/`
- `codecarbon/output.py`

This is a practical integration feature that aligns well with common MLOps workflows.
```

## Issue 8

**Title:** `Add Weights & Biases output integration for emissions tracking`

```md
**Is your feature request related to a problem? Please describe.**
Weights & Biases is widely used in ML experimentation, but CodeCarbon does not currently provide a native output integration for it. Users have to manually bridge the two systems.

**Describe the solution you'd like**
Add a built-in Weights & Biases output method that logs emissions metrics and metadata to the active W&B run.

Potential scope:
- log core emissions metrics
- log hardware metadata
- log derived efficiency metrics when available
- make the integration optional and non-invasive when W&B is not installed

This should follow the same design pattern as the existing output methods.

**Describe alternatives you've considered**
Manual user-side integration is possible, but it creates duplicated code and fragmented conventions.
Another alternative would be to provide an example only, but native support would reduce friction significantly.

**Additional context**
Relevant repo areas:
- `codecarbon/output_methods/`
- `codecarbon/output.py`

This would be useful for ML practitioners who want sustainability metrics beside loss, accuracy, and throughput in a familiar dashboard.
```

## Issue 9

**Title:** `Expose measurement quality metadata in tracker outputs`

```md
**Is your feature request related to a problem? Please describe.**
Users often cannot tell whether a reported result came from direct hardware measurement, estimated fallback paths, or partially inferred metadata. This makes it hard to assess how trustworthy or comparable two runs are.

**Describe the solution you'd like**
Expose structured measurement-quality metadata in the outputs, for example:
- CPU measurement mode: direct vs fallback
- GPU measurement mode: direct vs fallback
- RAM measurement mode
- region/carbon-intensity resolution quality
- overall estimate quality classification

This metadata should be available in emitted records and user-facing outputs so downstream tools can reason about data quality.

**Describe alternatives you've considered**
This could be documented informally in logs, but structured metadata would be more useful for dashboards, experiments, and analysis pipelines.

**Additional context**
Relevant repo areas:
- `codecarbon/core/resource_tracker.py`
- `codecarbon/core/emissions.py`
- `codecarbon/output_methods/emissions_data.py`

This issue pairs naturally with uncertainty-aware reporting, but it can also stand alone as a useful first step.
```

## Issue 10

**Title:** `Add benchmark suite for validating emissions estimates on common ML workloads`

```md
**Is your feature request related to a problem? Please describe.**
It is difficult to quantify how well CodeCarbon estimates match real hardware behavior across representative ML workloads. Without a reusable benchmark suite, improvements to estimation logic are harder to validate and compare.

**Describe the solution you'd like**
Add a benchmark suite focused on common ML workloads and calibration scenarios.

Possible contents:
- standard CPU-bound and GPU-bound workloads
- training vs inference scenarios
- repeatable scripts for collecting benchmark traces
- comparison of estimated power/emissions against reference measurements when available
- reporting artifacts that help contributors evaluate model changes

This benchmark suite would help both maintainers and contributors validate improvements to estimation logic.

**Describe alternatives you've considered**
Ad hoc benchmarking through notebooks or one-off scripts is possible, but it makes long-term validation and collaboration much harder.
Another option is to add only documentation about benchmarking, but a maintained benchmark suite would be far more useful.

**Additional context**
Relevant repo areas:
- `examples/`
- `codecarbon/data/hardware/cpu_load_profiling/`
- `codecarbon/data/hardware/cpu_dataset_builder/`

This would be a strong foundation for future work on fallback model calibration and methodology validation.
```
