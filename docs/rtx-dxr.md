# Native DXR experiment

This branch retains the existing DXIL renderer. It uses the standard D3D12
DXR API and does not require NVAPI, CUDA or AMD-specific extensions.

At device creation, the driver-reported raytracing tier is logged.
With `VICTORY_DXR_PROBE=1`, a DXR 1.1-capable device is queried for native
BLAS and TLAS prebuild sizes for synthetic geometry. No GPU commands are
submitted by this probe. This is a capability/prebuild test, not a
working acceleration structure or a game ray-tracing implementation.

PowerShell: `$env:VICTORY_DXR_PROBE="1"`. Look for the host DXR capability
and native DXR synthetic prebuild size probe messages.

Next gates:
- Build a synthetic triangle BLAS and instance TLAS on a real device.
- Compare inline RayQuery hit/miss, distance and barycentrics against a CPU reference.
- Identify actual title geometry and instance inputs and verify their lifetime,
  resource transitions and updates.
- Reconstruct prebuilt static structures or recover equivalent geometry,
  and validate shader hit IDs and material semantics before replacing traversal.

Until those gates pass, do not enable native DXR for the title. Keep the
existing BVH builder safety rules unchanged. Windows CI checks CPU-side
descriptor and capability logic; it does not prove hardware rendering.
