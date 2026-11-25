# VIctory RTX development instructions

This is a vendor-neutral, offline Xbox Series compatibility-layer research project.
Work on this fork, not upstream repositories. Keep the last passing build available.
Do not add title executables, DLLs, shader dumps, game assets, keys, or user logs
containing proprietary data to the repository. Use generated fixtures for CI and
authorized local test material only. Do not change the offline network restrictions.

## Baseline and build

- Branch inherited from victory-rtx-dxr, commit 5c9a1c30.
- Last complete passing CI before the texture-span fix: 972e499, run 45528040501.
- Windows x64, pinned Rust toolchain, D3D12 shader model 6.6; target test machine
  Ryzen 7 5800X3D / RTX 4090. Support AMD and Intel through standard DXR.
- Windows build commands (PowerShell, from repository root):

      cargo test --locked --workspace
      cargo build --locked --profile diag -p victory --features diagnostics
      cargo build --locked --release -p victory

  For local configuration copy victory.example.toml to victory.toml and set paths
  outside the repository. Run cargo run -p victory -- config to check them.
  Game data is not required for compilation and unit tests.

## Priority backlog

1. Texture submission ordering: inspect host.rs prelude(), Deferred, Step,
   texture.rs upload(), and list.rs copy/resolve operations. Do not set global
   host-written ownership at command recording. Design submission-ordered
   ownership with per-list read/write intent and resource lifetime protection.
   Generate a small RGBA texture test with overlapping copies, reverse recording
   order, identical submission order, and reset/abandoned lists. Check state
   transitions, first upload, GPU writes, staging buffer reuse and late flights.
   Do not enable a new path until the pixel readback matches native D3D12.
2. Wave64: inspect emit/lower.rs, wave.rs, wave64_tests.rs, structure.rs, and
   emit/mod.rs. Preserve native wave64 where supported. Test exact cross-half
   prefix sums, ballots, reductions, EXEC masks and lane reads on Wave32.
   Test 64/128/512-thread groups, multiple logical waves, empty halves and
   divergent branches. Do not silently substitute approximate results for
   builder shaders. Do not introduce group barriers on paths that may diverge.
   Keep VICTORY_WAVE_WIDE off by default until hardware evidence is available.
3. Existing ray-tracing builder: examine docs/fork-ray-tracing.md; identify
   translation errors and validate fixes with synthetic shaders. Keep all-or-none
   builder gating. No claim of working lighting until images demonstrate it.
4. Native DXR: dxr.rs currently queries capability and synthetic BLAS/TLAS sizes,
   not actual builds. Implement independent synthetic BLAS/TLAS construction
   with GPU lifetime/barrier tests, then inline RayQuery against CPU reference.
   Investigate actual geometry/instance inputs before attempting a game backend.
   Preserve compute fallback; no NVIDIA-only API required.
5. Performance: measure submission preparation, readback/relay waits, GPU time,
   faults, memory residency and cache behavior. Optimize only with measurements.
   Do not disable correctness checks or skip required work to report FPS.

## Verification and reporting

Use small commits with regression tests and run Windows CI. The fork workflow
publishes executables only after the workspace suite succeeds. A passing CI
build is not a hardware or gameplay pass. Report actual failures, skips,
device removal, shader rejections, and remaining gaps. Avoid claiming 30/60 FPS
or working ray-tracing without measurements. Keep experimental flags opt-in.
Do not merge to main or submit upstream PRs without explicit owner direction.
