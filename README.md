# SoundIt – User Manual

## Introduction

SoundIt is an adaptive audio system for Unity that allows you to shape how audio behaves in 3D space using **Meshes, Paths, and Points**.

Instead of a traditional point-based AudioSource, SoundIt lets you define *areas and shapes* that influence how sound is perceived in the scene.

---

# Getting Started

## 1. Setup

1. Add the component:
``SoundIt_AdaptativeAudioSource``


2. Ensure your GameObject has an:
- ``AudioSource`` (required)

3. Assign:
- ``Audio Clip``
- Optional Target (listener override)

---

## 2. Basic Configuration

### Audio Source Settings

- **Audio Source**
- The Unity AudioSource used for playback

- **Target**
- Overrides listener position (optional)
- If empty, uses AudioListener in scene

---

# Emitter Types

SoundIt supports 3 emission modes:

## 1. Mesh Audio Emitter

Uses a mesh surface to calculate audio behavior.

### Use cases:
- Large surfaces
- Buildings
- Terrain-based audio zones

### Setup:
- Assign a `MeshFilter`
- Ensure mesh is not static if dynamic updates are needed

---

## 2. Path Audio Emitter

Defines a custom shape using points.

### Features:
- Freehand drawing in SceneView
- Closed or open shapes
- Real-time editing

### Settings:

- **Close Path**
- Connects last point to first

- **Follow Inside**
- Audio follows listener when inside shape

- **Draw Threshold**
- Minimum distance between points

- **Debug Mesh**
- Shows filled preview of the path

---

## 3. Point Audio Emitter

Simple point-based system.

### Features:
- Multiple points
- Lightweight behavior
- Good for sparse audio placement

---

# Draw Mode (Path & Points)

## Activation

Click: ``Enter Draw Mode``


---

## Controls

| Input | Action |
|------|--------|
| Left Click | Add / Move point |
| Right Click | Remove point |
| Ctrl + Z | Undo |
| Ctrl + Y | Redo |
| ESC | Exit draw mode |

---

## Behavior

- Points snap based on distance threshold
- Dragging moves existing points
- New points are created if none selected

---

# Preview System

## What is Preview Mode?

Preview Mode lets you:
- Simulate audio behavior in Edit Mode
- Use SceneView camera as listener
- Hear changes without entering Play Mode

---

## How to use

1. Click: ``Start Preview``
2. SceneView camera becomes listener reference
3. Audio plays in editor

---

## Stop Preview

Click: ``Stop Preview``


Everything returns to original state.

---

# Camera Behavior (Preview)

When Preview Mode is active:

- Game Audio follows SceneView camera
- Original camera position is saved
- On exit, camera is restored automatically

---

# Visual Debug

## Path Debug Mesh

When enabled:
- Shows filled polygon preview
- Helps visualize audio area coverage

## Gizmos

SoundIt displays:

- Min Distance radius (inner range)
- Max Distance radius (outer range)
- Axis-based visualization:
    - Red → X axis
    - Green → Y axis
    - Blue → Z axis

---

# Audio Behavior

Sound is evaluated based on:

- Listener position
- Shape geometry
- Distance thresholds
- Inside/outside detection (Path mode)

---

# Performance Tips

- Disable debug mesh in production builds
- Avoid overly dense path points
- Use Mesh emitter for large static zones
- Use Point emitter for lightweight setups

---

# Common Issues

## No sound in Editor Preview
- Ensure Preview Mode is active
- Check AudioSource is assigned
- Ensure SceneView is open

---

## Gizmos not visible
- Select the GameObject in hierarchy
- Enable Gizmos in SceneView

---

## Camera not following
- Ensure only one Camera.main exists
- Restart preview mode

---

# Recommended Workflow

1. Start with **Point emitter**
2. Move to **Path emitter** for custom zones
3. Use **Mesh emitter** for environment shapes
4. Use Preview Mode constantly to iterate quickly

---

# Summary

SoundIt allows you to:
- Design spatial audio visually
- Edit shapes directly in SceneView
- Preview sound without entering Play Mode
- Control audio with geometry instead of points

> Think of SoundIt as "level design for sound"

# 📡 Event API — SoundIt Adaptive Audio Source

## Overview

`SoundIt_AdaptativeAudioSource` exposes a structured event system that allows developers to react to listener proximity in a consistent and performant way.

The system supports both:

* **Inspector-driven workflows** via `UnityEvent`
* **Code-driven workflows** via C# delegates

All events are evaluated **at runtime only** and are based on the distance between the listener and the dynamically evaluated emission point.

---

## Event Data

All events share a common payload:

```csharp
public struct SoundItEventData
{
    public SoundIt_AdaptativeAudioSource source;
    public Vector3 listenerPosition;
    public float distanceToShape;
    public float proximityNormalized;
}
```

### Fields

| Field                 | Description                                                             |
| --------------------- | ----------------------------------------------------------------------- |
| `source`              | The emitter that triggered the event                                    |
| `listenerPosition`    | World-space position of the listener                                    |
| `distanceToShape`     | Distance from listener to the evaluated emission point                  |
| `proximityNormalized` | Normalized proximity value (0 → maxDistance, 1 → minDistance or closer) |

---

## Event Types

The system defines two distance thresholds:

* **Outer Range** → `AudioSource.maxDistance`
* **Inner Range** → `AudioSource.minDistance`

---

### 1. Outer Range Events

#### `onListenerEnter` / `ListenerEntered`

Triggered **once** when the listener enters the audible range.

```csharp
dist <= maxDistance
```

---

#### `onListenerExit` / `ListenerExited`

Triggered **once** when the listener exits the audible range.

```csharp
dist > maxDistance
```

---

### 2. Inner Range Events

#### `onListenerEnterInner` / `ListenerEnteredInner`

Triggered **once** when the listener enters the inner (close proximity) range.

```csharp
dist <= minDistance
```

---

#### `onListenerExitInner` / `ListenerExitedInner`

Triggered **once** when the listener exits the inner range.

```csharp
dist > minDistance
```

---

### 3. Continuous Event

#### `onProximityChanged` / `ProximityChanged`

Triggered **every frame** while the listener is inside the outer range.

Provides continuous feedback via:

```csharp
data.proximityNormalized
```

---

## UnityEvent vs C# Events

### UnityEvents (Inspector)

```csharp
public SoundItEvent onListenerEnter;
```

* Serializable
* Editable in Inspector
* Suitable for designers
* Supports drag & drop bindings

---

### C# Delegates

```csharp
public event Action<SoundItEventData> ListenerEntered;
```

* Code-only
* Faster invocation
* Type-safe
* Recommended for gameplay logic

---

## Subscription (C#)

```csharp
void OnEnable()
{
    sound.ListenerEntered += HandleEnter;
    sound.ProximityChanged += HandleProximity;
}

void OnDisable()
{
    sound.ListenerEntered -= HandleEnter;
    sound.ProximityChanged -= HandleProximity;
}

void HandleEnter(SoundItEventData data)
{
    Debug.Log("Listener entered range");
}

void HandleProximity(SoundItEventData data)
{
    float t = data.proximityNormalized;
}
```

---

## Event Flow Logic

Each frame:

1. Distance is computed:

```csharp
dist = Vector3.Distance(listener, emissionPoint);
```

2. State flags are evaluated:

```csharp
isInsideOuter = dist <= maxDistance;
isInsideInner = dist <= minDistance;
```

3. Transitions are detected using previous state:

```csharp
_wasInsideOuter;
_wasInsideInner;
```

4. Events are fired only on **state changes** (except proximity).

---

## Proximity Normalization

Proximity is computed as:

```csharp
proximity = 1 - (dist - minDist) / (maxDist - minDist);
```

Clamped between `[0, 1]`.

### Meaning

| Value | Meaning                    |
| ----- | -------------------------- |
| `0`   | At or beyond `maxDistance` |
| `1`   | At or within `minDistance` |
| `0.5` | Midway between min and max |

---

## Best Practices

* Use **inner/outer events** for state transitions (enter/exit zones)
* Use **proximity** for continuous effects:

  * Reverb
  * Low-pass filtering
  * Volume modulation
* Prefer **C# events** for gameplay systems
* Prefer **UnityEvents** for quick prototyping or designer workflows

---

## Notes

* Events are only evaluated during **Play Mode**
* In Editor preview mode, audio updates but **events are not fired**
* The emission point is already resolved before event evaluation (`behavior.Execute`)

---

## Summary

The event API provides a robust abstraction over spatial distance, enabling:

* Clean separation between audio logic and gameplay logic
* Deterministic state transitions
* Continuous parameter control via normalized proximity

It is designed to scale from simple triggers to complex adaptive audio systems.
