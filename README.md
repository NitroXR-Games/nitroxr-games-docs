# NitroXR Games Cloud: Architectural Specification

This repository serves as the central technical reference for the NitroXR Games ecosystem. It defines the relationship between the Client-side logic (the "Brain") and the NitroXR Cloud infrastructure (the "Body").

## 1. The Cloud-Native Architecture
NitroXR is a **Cloud-Native XR Engine**. Unlike traditional game engines that bundle assets into a local binary, NitroXR separates the **Execution Logic** from the **Visual Assets**.

### The Brain (Client-side)
- **Role**: Orchestrates state, handles input, and manages game rules.
- **Mechanism**: Lightweight JavaScript/TypeScript running in the NitroXR Runtime.
- **Responsibility**: 
  - Frame-by-frame updates via `NitroXR.onUpdate`.
  - Entity management via the `NitroXR.Scene` graph.
  - Interface with the Cloud via the `NitroXR.Cloud` API.

### The Body (Cloud-side)
- **Role**: Provides the high-fidelity visual and spatial environment.
- **Mechanism**: A distributed Resource Registry of 3D assets.
- **Responsibility**:
  - **Asset Hosting**: Hosting PBR materials, 3D models, and textures.
  - **Dynamic Loading**: Streaming assets to the client based on the entity's `material` or `model` IDs.
  - **State Persistence**: Storing Global Leaderboards, Ghost paths, and User-Generated Content (UGC) layouts.

---

## 2. Core Technical Patterns

### A. Resource Pointing (The ID System)
To keep the client lean, we use **Material/Model IDs**.
- **Pattern**: `entity.update({ material: 'nitro_concrete_wall' })`
- **Process**: The client sends the ID $\to$ NitroXR Cloud resolves the ID $\to$ The high-res texture is streamed to the GPU.

### B. The Deterministic Loop
To prevent VR motion sickness, the engine follows a strict sequence:
`Input Polling` $\to$ `Physics/Collision` $\to$ `State Update` $\to$ `Render`.

### C. Asynchronous Data Layer (`NitroXR.Cloud`)
All network operations are asynchronous to prevent "frame drops."
- **Leaderboards**: `NitroXR.Cloud.submit({ gameId, userId, value })`
- **Ghosting**: `NitroXR.Cloud.getGhost(userId)` returns a compressed array of `{x, z, t}` coordinates.
- **UGC**: `NitroXR.Cloud.submit({ gameId: 'editor', layoutId, data })` saves spatial configurations.

---

## 3. Implementation Guidelines for Developers

### Creating a New Game
1. **Define the Vibe**: Use a Mission Script to set the frequency (Stealth vs. Market-Capture).
2. **Build the Brain**: Implement the logic using the `NitroXR.Scene` API.
3. **Reference the Body**: Use official NitroXR material IDs for visuals.
4. **Enable the Social Layer**: Integrate `NitroXR.Cloud` for competitive features.

### Testing without XR Hardware
Use a **Mock Runtime** (e.g., `mock-nitroxr.js`) to simulate the `Scene` and `Cloud` objects in a standard browser environment. This allows for rapid logic iteration before deploying to a headset.
