# Master Blueprint: NitroXR Runtime Engine

This document defines the architecture for the NitroXR Runtime, a cloud-native XR engine designed to separate game logic (The Brain) from visual assets (The Body).

## 1. System Architecture: The "Thin-Client" Model

The NitroXR Engine is not a traditional game engine; it is a **Spatial Orchestrator**. It allows developers to write high-level logic that controls a high-fidelity environment streamed from the cloud.

### A. The Brain (The Client Runtime)
The Brain is a lightweight JavaScript library that runs on the user's device.
- **Scene Graph**: A virtual representation of the world.
- **The Heartbeat**: A deterministic 60Hz loop that ensures input $\to$ physics $\to$ render happens without jitter.
- **Asset Resolver**: A system that converts IDs (e.g., `nitro_concrete`) into actual 3D meshes and materials.

### B. The Body (The Cloud Resource Registry)
The Body is a distributed set of services that provide the sensory experience.
- **Resource Registry**: A database of PBR materials and GLB models.
- **Asset Streaming**: A system that delivers only the assets currently required by the Brain.
- **Global State**: Persistence for leaderboards, ghost paths, and UGC.

---

## 2. Technical Stack Specification

### Frontend (The Client)
- **Rendering Engine**: Three.js (as the primary WebGL/WebGPU abstraction).
- **XR Interface**: WebXR Device API (to support Oculus, Vive, Index, etc.).
- **State Management**: Event-driven, singleton-based architecture.
- **Logic Language**: JavaScript/TypeScript.

### Backend (The Cloud)
- **API Layer**: Node.js / Fastify (High-throughput, low-latency).
- **Database**: 
  - **MongoDB/PostgreSQL**: For user data and UGC layouts.
  - **Redis**: For real-time leaderboard caching.
- **Storage**: AWS S3 or similar for hosting `.glb` and texture files.

---

## 3. Core Component Interaction Map

`Input (XR Controller)` $\to$ `NitroXR.onUpdate()` $\to$ `User Logic` $\to$ `NitroXR.Scene.update()` $\to$ `Renderer` $\to$ `GPU`.

**Cloud Interaction Flow:**
`Request Asset ID` $\to$ `NitroXR Cloud Registry` $\to$ `S3 Stream` $\to$ `GPU Texture/Mesh`.

---

## 4. Performance & Motion-Sickness Guardrails

To ensure a professional XR experience, the engine must enforce these "Golden Rules":

1. **The 16.6ms Rule**: The total time from input to render must never exceed 16.6ms (60fps). Any logic exceeding this must be offloaded to a Web Worker.
2. **Zero-Clipping Physics**: Collision must be absolute. No "soft" boundaries that allow the player to slide through walls.
3. **Async everything**: No synchronous network calls. All `NitroXR.Cloud` calls must use `async/await` to avoid blocking the render thread.
4. **Asset Throttling**: Load assets in priority order (Floor $\to$ Walls $\to$ Hazards $\to$ Decor).

---

## 5. Roadmap to "Gold Master" (The Laps)

The engine will be built in three phases:
1. **The Skeleton (Laps 1-4)**: Scene Graph $\to$ Asset Resolver $\to$ Physics $\to$ XR Input.
2. **The Nervous System (Laps 5-7)**: Cloud API $\to$ Ghosting $\to$ UGC Persistence.
3. **The Skin (Laps 8-10)**: Culling/Optimization $\to$ DevTools $\to$ SDK Release.
