## Environment
- AEDT 2022 R1 + PyAEDT - use project .venv, install both `pyaedt==0.18.1` and `pythonnet`, otherwise PyAEDT falls back to broken gRPC startup on 2022.1.
- Project venv path - use `C:/F/CODEFILE/HFSS_agent/.venv/Scripts/python.exe` for PyAEDT, pytest, and automation runs in this repo.

## Collaboration
- Default autonomous decision-making - when multiple reasonable implementation approaches exist, choose the most practical option yourself and continue without asking the user to pick.
- Default execution style - once implementation direction is approved, continue through remaining tasks without per-task check-ins unless blocked.
- Escalation threshold - only ask the user to choose when the decision is high-risk, irreversible, affects external/shared systems, or is a true business/product tradeoff.
- Pause threshold - only stop for destructive actions, external/shared-state actions, or genuine ambiguity/blockers.

## Commands and Workflow
- AEDT CLI `-ng` - must be combined with `-batchsolve`, `-batchsave`, or `-batchextract`; `ansysedt.exe -ng` alone is invalid.
- HFSS automation workflow - validate changes in order: Desktop smoke test -> model build test -> solve/export test -> optimization loop.
- HFSS automation tests - use pytest-driven smoke/build/solve tests and verify exported CSVs plus JSON metrics before trusting a script change.
- HFSS script verification - verify both `pytest ...` and a full `python test1/run_microstrip_experiments.py` run; unit tests alone won’t catch AEDT post-processing issues.

## Gotchas
- HFSS automation smoke test - verify Desktop starts in COM/PythonNET mode before building geometry; gRPC mode on AEDT 2022.1 can fail looking for `PyDesktopPlugin.dll`.
- PyAEDT desktop cleanup - `Desktop.release_desktop(close_projects=True, close_on_exit=True)` uses `close_on_exit`, not `close_desktop`.
- HFSS temp projects - avoid reusing a fixed `.aedt` filename across failed runs, because leftover `.aedt.lock` files will block reopening the project.
- HFSS 2022.1 solve failures - inspect `*.aedtresults/<Design>.results/*.profile` and `nominalmesh.g3derr` first; they exposed the real root causes here.
- Microstrip patch in HFSS 2022.1 - prefer `create_open_region()` over a hand-built airbox when meshing or radiation issues appear.
- Wave port on this patch model - an internal microstrip wave port failed validation; a lumped port between feed and ground solved reliably for this automation path.
- HFSS lumped ports - explicit `integration_line=[[x,0,trace_top_z],[x,0,0]]` on port sheets was more reliable here than reference-based lumped-port setup.
- HFSS post expressions - working modal expressions here were `S(P2,P1)` / `S(P2:1,P1:1)` and `Z(P1:1,P1:1)` against the solved `<setup_name> : LastAdaptive`.
- HFSS result extraction - use `<setup_name> : LastAdaptive` for `post.get_solution_data`; `hfss.nominal_adaptive` can point to the wrong setup in multi-setup designs.
- HFSS phase extraction - `SolutionData.data_phase()` may return radians; convert to degrees when `abs(value) <= 2π`.
- HFSS design insertion - on AEDT 2022.1 / PyAEDT 0.18.1, `insert_design()` may return `None`; keep using the existing HFSS object after the call.
- HFSS repeated variant solves - reuse one project with per-variant designs and clear model geometry before rebuilding to avoid stale object/setup collisions.
- HFSS tuning preference - when the goal is one final antenna design, keep one design and tune by updating HFSS variables in place instead of rebuilding geometry or creating many variant designs.
- HFSS setup naming - `create_setup(name="Setup1")` may auto-rename to `Setup1_2`, `Setup1_2_3`, etc.; use `setup.name` after creation rather than assuming the requested name.
- Inset-feed tuning - S11 improved only after parameterizing inset depth (`Fi`); coarse tuning of `Lp/Wf` alone was not enough.

## Repository Hygiene
- Repository root hygiene - keep the root limited to project folders plus `.gitignore` and `CLAUDE.md`; do not leave HFSS result files loose in the root.
- HFSS result placement - keep `.aedt`, `.aedtresults`, CSV exports, JSON summaries, and automation scripts together inside the simulation project folder (for example `test0510/`).
- HFSS debug hygiene - introspection/diagnostic `.aedt` projects can accumulate quickly; keep only final deliverables in the main simulation folder after debugging.
