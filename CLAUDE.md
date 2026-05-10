## Environment
- AEDT 2022 R1 + PyAEDT - use project .venv, install both `pyaedt==0.18.1` and `pythonnet`, otherwise PyAEDT falls back to broken gRPC startup on 2022.1.
- Project venv path - use `C:/F/CODEFILE/HFSS_agent/.venv/Scripts/python.exe` for PyAEDT, pytest, and automation runs in this repo.

## Commands and Workflow
- AEDT CLI `-ng` - must be combined with `-batchsolve`, `-batchsave`, or `-batchextract`; `ansysedt.exe -ng` alone is invalid.
- HFSS automation workflow - validate changes in order: Desktop smoke test -> model build test -> solve/export test -> optimization loop.
- HFSS automation tests - use pytest-driven smoke/build/solve tests and verify exported CSVs plus JSON metrics before trusting a script change.

## Gotchas
- HFSS automation smoke test - verify Desktop starts in COM/PythonNET mode before building geometry; gRPC mode on AEDT 2022.1 can fail looking for `PyDesktopPlugin.dll`.
- PyAEDT desktop cleanup - `Desktop.release_desktop(close_projects=True, close_on_exit=True)` uses `close_on_exit`, not `close_desktop`.
- HFSS temp projects - avoid reusing a fixed `.aedt` filename across failed runs, because leftover `.aedt.lock` files will block reopening the project.
- HFSS 2022.1 solve failures - inspect `*.aedtresults/<Design>.results/*.profile` and `nominalmesh.g3derr` first; they exposed the real root causes here.
- Microstrip patch in HFSS 2022.1 - prefer `create_open_region()` over a hand-built airbox when meshing or radiation issues appear.
- Wave port on this patch model - an internal microstrip wave port failed validation; a lumped port between feed and ground solved reliably for this automation path.
- Inset-feed tuning - S11 improved only after parameterizing inset depth (`Fi`); coarse tuning of `Lp/Wf` alone was not enough.

## Repository Hygiene
- Repository root hygiene - keep the root limited to project folders plus `.gitignore` and `CLAUDE.md`; do not leave HFSS result files loose in the root.
- HFSS result placement - keep `.aedt`, `.aedtresults`, CSV exports, JSON summaries, and automation scripts together inside the simulation project folder (for example `test0510/`).
