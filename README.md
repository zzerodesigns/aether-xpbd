# Aether-XPBD

A bare-metal, high-performance WebGPU physics engine and interactive volumetric simulation environment, self-contained in a single file with zero external dependencies.

[![WebGPU](https://img.shields.io/badge/WebGPU-Direct3D12%20%7C%20Metal%20%7C%20Vulkan-blue.svg)](#)
[![Dependencies](https://img.shields.io/badge/dependencies-0-success.svg)](#)
[![Version](https://img.shields.io/badge/version-1.4.5-indigo.svg)](#)
[![Architecture](https://img.shields.io/badge/architecture-single--file%20HTML-orange.svg)](#)

---

## Vision & Architecture

Modern web graphics stacks are frequently weighed down by megabytes of runtime wrappers, abstraction layers, and bundler configurations. **Aether-XPBD** was created with a contrarian engineering philosophy:

1. **Zero External Runtime Dependencies**: No Three.js, Babylon.js, gl-matrix, or external physics engines. All 3D projection math, quaternion calculus, raycasting, camera rigs, and UI styling are hand-crafted from bare primitives.
2. **Single-File Distribution**: The entire engine—simulation kernel, WGSL compute pipelines, instanced rendering shaders, and reactive telemetry UI—resides in a self-contained `index.html`.
3. **GPU-First XPBD Simulation**: Physics simulation is not bounded by single-threaded JavaScript execution. Extended Position-Based Dynamics (XPBD) calculations run natively on the GPU via parallel WGSL compute shaders with 10 substeps per frame.
4. **Compute-to-Vertex Stream**: Instance data never leaves GPU memory. The vertex and fragment pipelines sample directly from the ping-pong physics storage buffers, eliminating per-frame CPU-to-GPU memory transfer overhead.
5. **Zero-Allocation Main Loop**: All uniform structs, matrices, ray vectors, and camera parameters use pre-allocated static typed arrays. Per-frame garbage collection pauses are completely eliminated.

---

## Core Features

### 1. Extended Position-Based Dynamics (XPBD) Kernel
- **Multi-Stage Compute Dispatch**: Runs a continuous, substepped pipeline per frame across five specialized compute kernels:
  - `clear_grid`: GPU-side atomic clearing of spatial hash bins.
  - `integrate`: Symplectic Euler integration with gravity, velocity projection, and drag.
  - `update_grid`: Parallel binning of particles into a 3D uniform spatial hash table.
  - `solve_constraints`: Particle-to-particle penetration correction, restitution, dynamic friction, ragdoll angular impulse, and central obstacle signed distance field (SDF) collisions.
  - `finalize`: Velocity derivation ($v = \frac{x - x_{\text{prev}}}{\Delta t}$), angular velocity updates, ground friction torque, and substep-aware energy damping.
- **Adaptive Time-Stepping**: Time step $\Delta t$ dynamically adapts to real frame deltas to guarantee consistent simulation speeds across 60 Hz, 120 Hz, and 144 Hz displays without exploding solver energy.
- **Dynamic Spatial Hash Grid**: Uniform $O(N)$ broad-phase grid that accommodates up to 16,384 interacting particles simultaneously without GPU workgroup timeouts.

### 2. Hybrid Rendering Pipeline
- **Analytical Impostors**: True spheres and capsules rendered using exact analytical ray-surface intersections in the fragment shader with hardware depth-buffer writes (`frag_depth`), delivering perfect silhouettes with zero polygon faceting.
- **Platonic Solid Geometries**: Analytical mesh renderers for convex polyhedra:
  - Cube (Hexahedron)
  - Tetrahedron
  - Octahedron
  - Dodecahedron
  - Icosahedron
- **Dynamic Lighting**: Real-time Blinn-Phong specular shading, diffuse lighting from an orbiting emissive central collider, and depth-tested self-shadowing.

### 3. Tactile 3D Interaction System
- **Camera-Aligned Raycasting**: Cursor interactions project into 3D world space using an inverted view-projection matrix, anchoring force fields precisely at the visual focal plane regardless of camera orbit or pan.
- **Four Interaction Fields**:
  - **Spacetime Warp**: Repels particles away from the cursor focal point.
  - **Singularity (Attract)**: Draws particles toward the cursor to form high-density clusters.
  - **Vortex (Swirl)**: Applies tangential torque perpendicular to the camera line-of-sight for localized fluid whirlpools.
  - **Central Collider**: Direct physical manipulation of the central obstacle hull.
- **Zero-G Entropy Diffusion**: Tunable Brownian agitation that prevents static clumping in zero-gravity environments, keeping particles evenly dispersed throughout the bounding volume.

### 4. 6DOF Camera Rig
- **Arcball Orbit**: Smooth spherical camera rotation around an adjustable focal target (Right-Click + Drag).
- **Planar Panning**: Translates the focal center along the camera’s view plane (Middle-Click or Shift + Right-Click).
- **Precision Zoom**: Stepless focal distance adjustment via mouse wheel or dedicated UI slider.
- **Cinematic Orbit Drift**: Smooth automatic orbit with variable speed control that preserves the exact camera orientation when paused.

### 5. UI & Diagnostics
- **Glassmorphism Modular Cards**: Minimizable control sections for Simulation, Particle Settings, Physics & Interactions, and Viewport Atmosphere.
- **Telemetry Readouts**: Real-time smoothed FPS counter, GPU frame-budget load estimator, particle density selector ($1,024$ to $16,384$), and hardware tier indicator.
- **Dual Themes**: Hand-tuned Dark and Slate/Blue-Gray ("Dim" Light) themes with buttery smooth 500ms CSS transitions.

---

## Technical Specifications

| Parameter | Specification |
| :--- | :--- |
| **API** | WebGPU (WGSL) |
| **Backend Tiers** | Direct3D 12+, Metal, Vulkan |
| **Particle Capacity** | 1,024 – 16,384 dynamic instances ($2^n$ workgroup aligned) |
| **Compute Substeps** | 10 substeps per frame |
| **Workgroup Size** | 64 threads per workgroup (`@workgroup_size(64)`) |
| **Memory Architecture** | Ping-pong double-buffered storage buffers (`Particle` struct: 128 bytes, 16-byte aligned) |
| **Memory Allocation** | Zero allocations inside `requestAnimationFrame` |
| **Dependencies** | 0 external libraries, pure vanilla JavaScript + WGSL |

---

## Particle Data Layout

Each particle is represented in WGSL by a 128-byte aligned structure:

```wgsl
struct Particle {
    pos: vec4f,      // xyz: position, w: mass / padding
    vel: vec4f,      // xyz: linear velocity, w: padding
    prev_pos: vec4f, // xyz: position at t - dt
    ori: vec4f,      // xyzw: orientation quaternion
    ang_vel: vec4f,  // xyz: angular velocity vector
    prev_ori: vec4f, // xyzw: orientation quaternion at t - dt
    color: vec4f,    // rgba: instance base color
    aux: vec4f,      // auxiliary properties / future extensions
};
```

---

## Getting Started

Because Aether-XPBD is completely self-contained in `index.html`, running it requires only a local static server and a WebGPU-compatible browser (Chrome 113+, Edge 113+, Firefox Nightly, or Safari 18+).

### Quick Launch
```bash
# Clone the repository
git clone https://github.com/your-username/aether-xpbd.git
cd aether-xpbd

# Run with any static HTTP server (e.g. Python, Bun, or Node)
npx serve .
# or
python3 -m http.server 3000
# or
bun x serve .
```

Open your browser at `http://localhost:3000`.

---

## Controls

| Input | Action |
| :--- | :--- |
| **Left Click + Drag** | Trigger active interaction field (Warp / Attract / Vortex) |
| **Right Click + Drag** | Orbit camera around focal point |
| **Middle Click / Shift + Drag** | Pan camera focal point |
| **Mouse Wheel / Zoom Slider** | Zoom in / out |
| **Sidebar Menu Button** | Toggle engine control cards |

---

## Roadmap

- [ ] Multi-shape concurrent instancing (simulating mixed particle shapes in a single buffer).
- [ ] XPBD distance constraints for cloth, ropes, and articulated soft bodies.
- [ ] Spatial grid sorting using GPU Bitonic Radix Sort for higher particle counts ($65,536+$).
- [ ] Screen-space fluid rendering pass (SSFR) with depth smoothing and bilateral blur.

---

## License

MIT License. Free for personal, educational, and commercial use.
