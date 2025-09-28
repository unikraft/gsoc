# GSoC'25: Autokraft | Testing Framework for Unikraft Builds

<img src="https://unikraft.org/images/gsoc25.jpeg"/>

## Project Overview

This project delivers a robust, automated testing framework for Unikraft applications and libraries (aka "Autokraft testing framework").
It orchestrates end-to-end validation across multiple configuration architectures, VMMs/hypervisors, build systems, and boot protocols—by configuring, building, running, and functionally testing unikernel targets.

Starting from an experimental prototype embedded in the Unikraft `catalog` repository ([branch `razvand/generator/new-design`](https://github.com/unikraft-upb/catalog/tree/razvand/generator/new-design)), I extracted, modularized, and matured it into a standalone, CI-ready system with clear configuration, deterministic runtimes, structured logs/reports, and sessionized artifacts—ready for organization-wide use on pull requests.

## GSoC Contributor

Name: Shashank Srivastava

Email: shashank21005@gmail.com

Github profile: [shank250](https://github.com/shank250)

## Mentors

* [Răzvan Deaconescu](https://github.com/razvand)
* [Răzvan Vîrtan](https://github.com/razvanvirtan)

## Repository & Code

- Autokraft: Testing Framework: https://github.com/unikraft/autokraft

## Contributions

Highlights grouped by area:

1) Extraction, modularization, and quality
- Extracted in-development code from catalog into a standalone repo with clean structure (`src/`, `scripts/`, `utils/`, `docs/`).
- Enforced code quality with `pylint`, formatting with `black`, and import hygiene with `isort`.
- Comprehensive docstrings and clear TODO markers for future improvements.

2) Isolated test sandboxes and reliable cleanup
- Introduced a copy-to-`.app` workflow so each test runs in a pristine, isolated app directory.
- Added fixtures and scripts to clean `.tests` and ephemeral artifacts before/after runs, handling interruptions gracefully and avoiding stale state.

3) Configuration model and dynamic generation
- Mirrored `catalog` directory layout under `test-app-config/` to store per-app `BuildConfig.yaml` and `RunConfig.yaml`.
- Implemented a README parser to extract memory requirements, exposed ports, and validation commands; added optional LLM-backed synthesis of `RunConfig.yaml` when a project lacks one.
- Split prior monolithic `tester_config.yaml` into `config.yaml` and `variants.yaml` to better separate defaults from test matrix variants.

4) Execution model and test categories
- Refactored the runner into clear handlers for three test types:
  - curl: check a public HTTP endpoint for service availability.
  - list-of-commands: run arbitrary CLI commands and assert on stdout.
  - no-command: boot-and-verify minimal liveness.

5) Deterministic runtimes and examples support
- Prebuilt runtime kernels for examples with a deterministic naming scheme (arch/build/tool/VMM/etc.) stored under `runtime_kernels/`.
- For Kraft-based flows, integrated a local registry for `pkg push`/`pkg pull` while retaining on-disk kernel artifacts for VMM-based runs.
- Added CLI controls: `--test-dir`, `--app-dir-name`, and `-g/--generate-only` to prebuild or stage targets without executing tests.

6) Observability, sessions, and reports
- Implemented sessionized run directories under `.sessions/<scope>/<app>/<session>/` with per-run logs, configs, and artifacts.
- Automated CSV reporting: `build_report.csv` (per target build) and `run_report.csv` (per test execution), plus an aggregated CI summary (`.sessions/ci_report.json`) and detailed per-app reports for orchestration.

7) CI orchestration and automation
- Authored a GitHub Actions workflow to execute framework runs on PRs: clones the catalog, prepares environment, installs dependencies (including `maintainers-tool`), and collects/upload artifacts.
- Added `overall_test.py` to orchestrate full-library sweeps or targeted subsets for faster CI iteration.
- Integrated with a GitHub App to reflect PR status and improved artifact retention for debugging.

## Blog Posts

The whole project  work is documented in a 4‑part blog series capturing milestones and evolution:
- GSoC'25: Testing Framework for Unikraft Builds
    - [Blog Post I](https://github.com/unikraft/docs/pull/498)
    - [Blog Post II](https://github.com/unikraft/docs/pull/510)
    - [Blog Post III](https://github.com/unikraft/docs/pull/515)
    - [Blog Post IV](https://github.com/unikraft/docs/pull/524)

## Architecture & Workflow

High‑level flow for a testing:

1) Select app from `catalog/library` or `catalog/examples`
	- Check `BuildConfig.yaml` and `RunConfig.yaml` from `test-app-config/`.
	- If missing, parse README and optionally synthesize via LLM.

2) Prepare isolated workspace
	- Copy app to `.app/` and clean `.tests/` to ensure pristine runs.

3) Build → Run → Test
	- Build with configured toolchain/build system; prebuild runtime kernels when needed.
	- Launch with selected VMM/hypervisor; run category‑specific checks (curl / list‑of‑command / no‑command).

4) Record & persist
	- Emit structured logs; write `build_report.csv` and `run_report.csv`.
	- Store everything under `.sessions/.../<session>/` for CI artifacts and local debugging.

## Usage (concise)

The framework exposes a single CLI (`src/main.py`) and an orchestration helper (`overall_test.py`).

## Future Work

- Extend CI beyond building libraries to also run tests within the workflow.
- Add first‑class CI support for `catalog/examples` end‑to‑end (in addition to prebuilt runtimes).
- Continue to address and fix any issues that arise while the usage of the testing framework.

## Acknowledgements

My GSoC journey has been full of invaluable learnings and experiences.
From the very beginning, I learned how to read and navigate unfamiliar codebases, remain persistent when concepts were difficult to grasp, and celebrate the progress that followed.
Beyond technical growth, I also gained important skills in staying on track with timelines, clearly communicating ideas, and presenting code and work in a professional manner.

All of this was possible thanks to the Google Summer of Code program, the supportive Unikraft community, and especially my mentors [Razvan Deaconescu](https://github.com/razvand) and [Razvan Virtan](https://github.com/razvanvirtan).
I am deeply grateful for the opportunity to contribute to this project.
A special note of thanks goes to Razvan Deaconescu for his continuous guidance, both technical and non-technical, helping me adopt good practices and encouraging a long-term mindset toward building impactful open-source projects.
