# Development Log — Aether-XPBD

A chronological record of architectural milestones, solver refinements, and performance optimizations.

---

## [v1.4.5] — 2026-05-16
### Dynamic Grid Exit & Per-Frame GC Elimination

- **Zero-Allocation Simulation Loop**:
  - Replaced all per-frame array and object allocations in `requestAnimationFrame` with pre-allocated static typed arrays (`tData`, `sceneBuf`, `eyeData`, `mVP`).
  - Completely eliminated garbage collection latency spikes, preserving smooth 60+ FPS frame pacing during intense collisions.
- **Dynamic Grid Exit Optimization**:
  - Improved spatial hashing grid exit routines to handle boundary transitions cleanly, preventing ghost collisions or stale references when particles move across cell edges.
  - Replaced CPU-side `device.queue.clearBuffer` calls with a dedicated GPU `clear_grid` compute pipeline executed at the start of each substep.
- **Single-Pass Compute Dispatch**:
  - Consolidated all 10 simulation substeps into a continuous compute pass, eliminating encoder teardown overhead per substep.
  - Streamlined ping-pong buffer binding swapping directly within the active compute pass.
- **Rendering & State Polish**:
  - Cached the hardware depth texture view to eliminate redundant view allocations during render passes.
  - Smoothed clear color transitions between Dark and Dim Light themes without descriptor recreation.
  - Updated System Metadata branding, bumped version to v1.4.5, and stamped build compilation timestamp.

---

## [v1.4.0] — 2026-05-11
### Substepping Stability & Collision Precision

- **Adaptive Delta Time Integration**:
  - Fixed slow simulation playback on 60Hz displays by making `dt` adaptive to real frame delta times.
  - Re-anchored damping equations to be substep-aware, preventing unnatural kinetic energy decay across multi-pass integration.
- **Collision Precision & Hull Authority**:
  - Reordered collision solver so the central obstacle (Big Object) resolves after particle-particle constraints, giving the obstacle definitive collision authority.
  - Corrected type mapping misalignment in `solve_constraints` where the Big Object type was swapped (Sphere was being calculated as a Tetrahedron).
  - Removed artificial 5% radius shrink on the central collider, eliminating particle clipping into the hull.
  - Removed artificial 5% radius expansion on particle-particle collisions, fixing visible contact gaps.
  - Preserved 10 substeps for high-fidelity XPBD integration.
- **UI/UX Refactor & Standardization**:
  - Standardized engine versioning across all UI components to v1.4.0.
  - Reorganized branding card: vertically stacked logo, title block, and system metadata.
  - Added 'WebGPU Physics Engine' descriptor to main header.
  - Embedded developer guidelines within HTML source comments.

---

## [v1.3.5] — 2026-05-08
### Polyhedral Geometry Stabilization & Viewport Controls

- **Core Geometry Pipeline**:
  - Hard-coded Platonic solid vertex and normal definitions (Tetrahedron, Cube, Octahedron, Dodecahedron, and Icosahedron) alongside true sphere and capsule analytical raytraced impostors.
  - Resolved WebGPU column-major projection matrix depth clipping, guaranteeing exact depth mapping to the `[0, 1]` hardware depth range.
- **6DOF Camera & Raycast Precision**:
  - Implemented inverted view-projection raycasting, projecting cursor interaction tools (Warp, Attract, Vortex) directly onto the camera-facing focal plane.
  - Added full 6DOF arcball orbiting, planar panning, and smooth cinematic orbit drift.
- **Architecture Freeze**:
  - Enforced single-file, zero-external-dependency requirement for the entire engine codebase.
