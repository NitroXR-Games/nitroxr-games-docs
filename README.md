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

### B. The Game Loop (Deterministic XR)
To prevent VR motion sickness, the engine follows a strict, high-frequency sequence:
`Input Polling` $\to$ `Logic/Physics` $\to$ `State Update` $\to$ `Render`.
This ensures that the visual update matches the user's physical movement perfectly.

### C. The Social Layer (Async Cloud)
All network operations are asynchronous to prevent "frame drops."
- **Leaderboards**: `NitroXR.Cloud.submit({ gameId, userId, value })`
- **Ghosting**: `NitroXR.Cloud.getGhost(userId)` returns a compressed array of `{x, z, t}` coordinates for asynchronous racing.
- **UGC**: `NitroXR.Cloud.submit({ gameId: 'editor', layoutId, data })` saves spatial configurations.

---

## 3. Developer Guide: Testing without XR Hardware

To iterate quickly without a headset, developers should use a **Mock Runtime**.

### Using `mock-nitroxr.js`
1. Include the `mock-nitroxr.js` script in a standard HTML file.
2. This script simulates the `NitroXR.Scene` and `NitroXR.Cloud` objects.
3. Run the game in a browser and monitor the `console.log` for state changes and cloud submissions.

---

## 4. NitroXR SDK Implementation Strategy

The SDK is designed as a **Global Singleton** provided by the runtime environment.

- **Scene Management**: `new NitroXR.Scene()` initializes the local spatial graph.
- **Entity System**: `scene.createEntity(id, props)` creates a 3D object linked to the Cloud Body.
- **Input Handling**: `NitroXR.onUpdate(callback)` provides a stream of XR controller/headset inputs.
- **Cloud Interface**: `NitroXR.Cloud` handles the persistence of high-scores and ghost paths.
