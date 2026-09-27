# NitroXR Games Cloud: Architectural Specification

This repository serves as the central technical reference for the NitroXR Games ecosystem. It defines the relationship between the Client-side logic (the "Brain") and the NitroXR Cloud infrastructure (the "Body").

## 1. The Conceptual Model: The "Spatial Cloud OS"

To understand NitroXR, think of it as a **web browser for 3D worlds**. 

In a standard website, you don't save the actual images (JPG/PNG) inside your code; you provide a URL. The browser then fetches that image from a server and displays it. NitroXR works the same way but for 3D:

- **The "Pointers"**: In the code, references like `material: 'nitro_concrete_wall'` are unique IDs (pointers).
- **The "Library"**: NitroXR maintains a massive cloud library of high-quality 3D materials and models. The ID points to a specific, pre-made asset in their library.
- **The "Runtime"**: When the code is uploaded to the NitroXR platform, the engine sees these IDs, fetches the actual 3D assets from the cloud, and wraps them around the created geometry.

**The Result**: 
- **Your Code = The Brain** (Logic, Physics, Rules).
- **NitroXR Cloud = The Body** (The actual 3D models, Textures, Lighting).
- **The Integration**: When the Brain is plugged into the Body via the NitroXR Runtime, the game appears visually complete without the need for local asset files.

---

## 2. The Cloud-Native Architecture
NitroXR is a **Cloud-Native XR Engine**. It separates the **Execution Logic** from the **Visual Assets** to enable "instant-on" experiences.

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

## 3. Core Technical Patterns

### A. Resource Pointing (The ID System)
To keep the client lean and loading times near-zero, we use **Material/Model IDs**.
- **Pattern**: `entity.update({ material: 'nitro_concrete_wall' })`
- **Process**: The client sends the ID $\to$ NitroXR Cloud resolves the ID $\to$ The high-res texture is streamed to the GPU.

### B. The Game Loop (Deterministic XR)
Standard web apps use "Event Listeners," but high-performance XR requires a **Deterministic Game Loop** to prevent VR motion sickness.
- **The Sequence**: `Input Polling` $\to$ `Logic/Physics` $\to$ `State Update` $\to$ `Render`.
- **The Heartbeat**: `NitroXR.onUpdate` handles this high-frequency loop (60fps), ensuring that the visual update matches the user's physical movement perfectly.

### C. The Social Layer (Async Cloud)
In XR, any "hiccup" in the frame rate (a "lag spike") can be physically nauseating. All network operations are asynchronous.
- **Leaderboards**: `NitroXR.Cloud.submit({ gameId, userId, value })`
- **Ghosting**: `NitroXR.Cloud.getGhost(userId)` returns a compressed array of `{x, z, t}` coordinates for asynchronous racing.
- **UGC**: `NitroXR.Cloud.submit({ gameId: 'editor', layoutId, data })` saves spatial configurations.

---

## 4. Implementation Strategy

### Entity-Component-Lite System
We use a simplified Entity-Component pattern to manage the scene:
- **Entities**: Everything in the world (Wall, Player, Goal, Sentinel) is an entity.
- **Properties**: Instead of complex class hierarchies, we use property objects (position, model, material) that the NitroXR Scene Graph synchronizes in real-time.

### Spatial Logic & Precision
- **AABB Collision**: We use Axis-Aligned Bounding Boxes. In XR, precision is mandatory; if a player's "physical" presence clips through a wall, the immersion is broken.
- **Recursive Backtracking**: Used for maze generation to ensure "Perfect Mazes" (one single path between any two points), preventing impossible layouts.

---

## 5. Developer Guide: Testing without XR Hardware

To iterate quickly without a headset, developers should use a **Mock Runtime**.

### Using `mock-nitroxr.js`
1. Include the `mock-nitroxr.js` script in a standard HTML file.
2. This script simulates the `NitroXR.Scene` and `NitroXR.Cloud` objects.
3. Run the game in a browser and monitor the `console.log` for state changes and cloud submissions.

---

## 6. SDK Interface Reference

The SDK is designed as a **Global Singleton** provided by the runtime environment:

- **Scene Management**: `new NitroXR.Scene()` initializes the local spatial graph.
- **Entity System**: `scene.createEntity(id, props)` creates a 3D object linked to the Cloud Body.
- **Input Handling**: `NitroXR.onUpdate(callback)` provides a stream of XR controller/headset inputs.
- **Cloud Interface**: `NitroXR.Cloud` handles the persistence of high-scores and ghost paths.
