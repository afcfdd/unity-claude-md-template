# AGENTS.md

## Purpose

This file defines how AI coding agents should work in this Unity XR project.

The project starts with Meta Quest development, but the architecture must keep future PCVR / VIVE Pro Eye support realistic. Agents must avoid short-term Quest-only shortcuts that make OpenXR, XR Interaction Toolkit, PCVR, VIVE, or body-tracking support difficult later.

---

## Project Goal

Build a VR / MR application in Unity with the following priorities:

1. Make development fast on Meta Quest first.
2. Keep core gameplay and interaction logic portable across OpenXR devices.
3. Support future PCVR / VIVE Pro Eye work with minimal rewrites.
4. Use agentic development tools to speed up iteration, debugging, and validation.
5. Keep platform-specific SDK usage isolated behind clear adapter layers.

---

## Primary Target Platforms

### Primary development target

- Meta Quest standalone
- Android build target
- OpenXR backend
- Meta XR SDK for Quest-specific features

### Future target

- PCVR
- VIVE Pro Eye
- Windows build target
- OpenXR backend
- VIVE OpenXR Plugin and VIVE-specific SDKs only inside platform-specific adapters

---

## Required / Expected Tools

### Unity and XR

Use the following as the default stack:

- Unity 6.x or newer when possible
- Universal Render Pipeline unless there is a clear reason not to use it
- XR Plug-in Management
- Unity OpenXR Plugin
- XR Interaction Toolkit
- Unity Input System
- Meta XR SDK
- Meta XR Core SDK
- Meta XR Simulator
- Meta Quest Developer Hub
- Android Debug Bridge
- Perfetto when performance tracing is needed

### Quest-specific development tools

Use these tools for Quest-specific development and validation:

- Meta Horizon Link / Quest Link
- Meta XR Simulator
- Meta XR Unity MCP Extension
- Meta Movement SDK when body tracking, face tracking, or eye tracking is required
- Meta Quest Developer Hub
- Immersive Debugger
- `meta-quest/agentic-tools`
- `hzdb` when useful for device logs, screenshots, documentation lookup, performance analysis, or Quest debugging

### Agentic development tools

Agents may use:

- Codex CLI / Codex IDE extension
- Unity MCP
- Meta XR Unity MCP Extension
- Other MCP-compatible AI clients if available

The preferred agentic development setup is:

```text
Local Unity Editor
+ local Codex or another MCP-compatible coding agent
+ Unity MCP
+ Meta XR Unity MCP Extension
+ optional meta-quest/agentic-tools / hzdb
```

Do not assume cloud-only agents can directly control the local Unity Editor. Unity MCP requires access to the local Unity Editor and its MCP relay.

---

## Core Architecture Rules

### Use OpenXR as the runtime foundation

OpenXR is the cross-device XR runtime foundation. Do not write gameplay systems that depend directly on Oculus, Meta, VIVE, or SteamVR APIs unless the code is isolated behind a platform adapter.

### Use XR Interaction Toolkit for portable interaction

Use XR Interaction Toolkit as the default layer for common XR interactions:

- XR Origin
- controller input
- ray interaction
- direct interaction
- object grabbing
- teleportation
- locomotion
- UI interaction
- common interactables

Agents should prefer XR Interaction Toolkit for cross-platform interaction code.

### Use Meta XR SDK only for Quest-specific functionality

Meta XR SDK is allowed and expected for Quest-specific functionality, including:

- Quest project setup
- Quest performance settings
- Meta XR Simulator integration
- passthrough
- MRUK / scene understanding
- Meta-specific hand tracking features
- Meta Movement SDK
- Meta platform features
- Quest-specific interaction samples or prefabs

However, Meta SDK usage must not leak into core gameplay logic unless unavoidable.

### Do not confuse XR Interaction Toolkit and Meta XR Interaction SDK

XR Interaction Toolkit and Meta XR Interaction SDK can both provide grabbing, pointing, teleportation, UI interaction, and hand/controller interaction.

Default rule:

```text
Use XR Interaction Toolkit for portable baseline interactions.
Use Meta XR Interaction SDK only when Quest-specific functionality or faster Quest prototyping clearly justifies it.
```

If Meta XR Interaction SDK is used, isolate it so that future PCVR / VIVE support can replace or bypass it.

---

## Platform Separation Rules

Platform-specific code must be separated by provider interfaces or adapter components.

Recommended pattern:

```text
Assets/
  Scripts/
    Core/
      Gameplay/
      Interaction/
      Body/
      Interfaces/
    Platforms/
      Quest/
      Vive/
      PCVR/
    Debug/
```

Examples:

```csharp
public interface IBodyTrackingProvider
{
    bool IsAvailable { get; }
    BodyPoseData CurrentPose { get; }
}
```

Then implement platform-specific versions:

```text
QuestBodyTrackingProvider
ViveBodyTrackingProvider
SimulatedBodyTrackingProvider
```

Core gameplay should depend on `IBodyTrackingProvider`, not directly on Meta Movement SDK, VIVE APIs, OVR classes, or SteamVR classes.

---

## Body Tracking Rules

Body tracking must be treated as a platform-dependent input provider.

### Quest

For Quest, use Meta Movement SDK when body tracking is required. The Quest body tracking path may rely on Meta-specific APIs and runtime features, but this code must remain inside Quest-specific providers.

### VIVE / PCVR

For VIVE Pro Eye or future tracker-based full-body tracking, use VIVE / OpenXR / PCVR-specific providers. Do not assume Quest Movement SDK and VIVE tracking provide the same data, confidence values, skeleton model, or calibration flow.

### Required abstraction

Do not write gameplay logic like this:

```csharp
// Avoid in core gameplay
var body = FindObjectOfType<OVRBody>();
```

Prefer:

```csharp
// Preferred in core gameplay
IBodyTrackingProvider bodyProvider;
var pose = bodyProvider.CurrentPose;
```

### Simulation fallback

Always keep a simulated or fallback provider for editor testing.

Examples:

- `SimulatedBodyTrackingProvider`
- keyboard-driven pose provider
- recorded pose playback
- simple default standing pose

This allows agents and developers to test game logic without always wearing the headset.

---

## Debugging and Preview Workflow

Use three separate debug loops. Do not treat Quest Link as final device validation.

### 1. Fast no-headset loop

Use this loop for the majority of iteration:

```text
Codex / agent
→ edit scripts or scene
→ Unity MCP checks Unity state and Console
→ Unity Play Mode
→ Meta XR Simulator when XR simulation is needed
→ fix errors
```

Use this for:

- C# compilation errors
- gameplay logic
- UI logic
- scene object placement
- simple interaction testing
- non-sensor-dependent behavior
- agent-driven checks

Agents should prefer this loop before requesting headset testing.

### 2. Headset preview loop with Quest Link

Quest Link / Meta Horizon Link is allowed and recommended for fast headset preview.

Use Link when checking:

- actual headset feel
- scale
- comfort
- motion sickness risk
- controller behavior
- hand tracking feel
- body tracking preview
- real head movement
- interaction reach and ergonomics

Quest Link is especially useful when Unity Play Mode needs real headset input without building and installing an APK every time.

For body tracking or runtime features over Link, ensure the relevant developer runtime features are enabled in the Meta Horizon Link desktop app.

### 3. Real Quest APK loop

Use standalone APK deployment for real validation.

Use this for:

- final Quest behavior
- Android permissions
- performance
- thermal behavior
- frame rate
- memory usage
- input latency
- device-only bugs
- release-readiness
- store-readiness
- Meta platform integration
- anything involving device-specific runtime behavior

Use:

- Meta Quest Developer Hub
- ADB logs
- Immersive Debugger
- screenshots and recordings
- Perfetto traces when needed
- `hzdb` and `meta-quest/agentic-tools` when useful

### Debugging rule

Quest Link is not obsolete. It is part of the workflow. However:

```text
Quest Link is a preview tool, not final validation.
```

A feature is not complete until it has been tested in the appropriate standalone APK or target runtime environment.

---

## Agentic Development Workflow

### Codex + Unity MCP

Agents may use Codex with Unity MCP to:

- inspect Unity Console errors
- edit C# scripts
- create or modify GameObjects
- inspect project files
- run or guide Play Mode tests
- fix compiler errors
- update scenes and prefabs when supported
- validate package setup where possible

When using Unity MCP, agents should:

1. Check Console errors before making broad changes.
2. Prefer small, reversible edits.
3. Avoid large scene or prefab rewrites unless requested.
4. Keep generated scripts organized under the expected folders.
5. Report any manual Unity Editor steps that cannot be automated.
6. Avoid changing platform settings without explaining why.

### Meta XR Unity MCP Extension

Use Meta XR Unity MCP Extension for Quest-specific Unity operations such as:

- Meta XR setup checks
- Quest build setting assistance
- Meta XR component setup
- Quest-oriented scene creation
- grabbable / interactable setup when appropriate
- XR Simulator related workflows
- Meta-specific validation

Do not let Meta XR MCP operations accidentally make the entire project Quest-only.

### meta-quest/agentic-tools and hzdb

Use these tools for Quest-focused agentic development tasks such as:

- device inspection
- app management
- log retrieval
- screenshots
- recordings
- performance traces
- documentation retrieval
- VRC / Quest validation assistance
- XR Simulator setup help
- debugging Quest-specific issues

These tools complement Unity MCP. They do not replace XR Interaction Toolkit, Meta XR SDK, or Unity project architecture.

---

## Build Targets

### Quest standalone

Default Quest build assumptions:

```text
Platform: Android
Architecture: ARM64
XR backend: OpenXR
Graphics: Vulkan unless project requirements say otherwise
SDK: Meta XR SDK
Interaction baseline: XR Interaction Toolkit
```

### PCVR / VIVE Pro Eye

Future PCVR build assumptions:

```text
Platform: Windows
XR backend: OpenXR
VIVE support: VIVE OpenXR Plugin and VIVE SDKs where necessary
Interaction baseline: XR Interaction Toolkit
Device-specific code: isolated in VIVE / PCVR adapters
```

Do not add direct SteamVR or VIVE dependencies to core gameplay unless specifically requested and isolated.

---

## Coding Rules

### Keep core gameplay device-independent

Core systems should not directly reference:

- `OVR*` classes
- Meta Movement SDK classes
- Meta MRUK classes
- VIVE-specific classes
- SteamVR classes
- Android-only APIs
- Windows-only APIs

If such references are necessary, place them in platform-specific folders and expose their behavior through interfaces.

### Use assembly definitions when useful

If platform code grows, add assembly definitions to keep dependencies clean.

Recommended split:

```text
Project.Core
Project.XR
Project.Platforms.Quest
Project.Platforms.Vive
Project.Debug
```

### Use compile guards carefully

Use platform compile guards only when necessary.

Examples:

```csharp
#if UNITY_ANDROID
// Quest / Android specific code
#endif

#if UNITY_STANDALONE_WIN
// PCVR / Windows specific code
#endif
```

Avoid scattering compile guards throughout gameplay scripts. Prefer adapter components.

---

## Scene and Prefab Rules

### XR Origin

Use one clear XR Origin setup. Do not create multiple competing rigs unless intentionally testing alternatives.

### Interaction setup

Default interaction stack:

- XR Origin
- XR Interaction Manager
- XR Ray Interactor
- XR Direct Interactor
- XR Grab Interactable
- XR UI Input Module
- locomotion providers as needed

### Quest-specific prefab use

Quest-specific prefabs are allowed when they save time, but they must not become hard dependencies for portable gameplay systems.

If a Meta prefab is used in a shared scene, document why it is needed and whether a portable replacement is required later.

---

## Validation Checklist

Before marking an XR feature complete, check the appropriate items.

### Required for most features

- Unity Console has no compile errors.
- Core logic works in Play Mode.
- Scene references are not broken.
- No unnecessary platform-specific API usage in core scripts.
- Feature works through XR Interaction Toolkit where possible.

### Required for Quest interaction features

- Works in Meta XR Simulator or Play Mode where applicable.
- Works through Quest Link if headset feel matters.
- Works on Quest standalone APK before final completion.

### Required for body tracking features

- Works with a simulated provider.
- Works with the Quest provider if targeting Quest.
- Does not directly couple gameplay to Meta Movement SDK.
- Calibration, missing tracking, and unavailable provider states are handled.
- Fallback behavior is defined.

### Required for performance-sensitive features

- Tested on standalone Quest APK.
- Frame timing checked.
- Logs checked.
- Perfetto or equivalent trace captured if the issue is non-trivial.
- Avoid assuming Link performance equals standalone Quest performance.

---

## Do Not Do

Agents must not:

- Convert the project into a Quest-only architecture unless explicitly requested.
- Put OVR / Meta / VIVE / SteamVR references into core gameplay scripts.
- Treat Quest Link as final validation.
- Assume Meta XR Simulator perfectly matches real Quest hardware.
- Assume Quest Movement SDK body tracking is equivalent to VIVE tracker-based full-body tracking.
- Add duplicate XR rigs without explaining why.
- Mix XR Interaction Toolkit and Meta XR Interaction SDK randomly in the same interaction path.
- Change Unity project settings broadly without documenting the reason.
- Add packages casually without checking whether existing packages already solve the problem.
- Ignore Unity Console errors.
- Delete scenes, prefabs, or project settings without explicit instruction.
- Make large architectural changes without preserving a clear migration path.

---

## Preferred Agent Behavior

When asked to implement or debug XR features, agents should:

1. Read this file first.
2. Identify whether the feature is core, Quest-specific, VIVE-specific, or debug-only.
3. Keep core gameplay portable.
4. Use interfaces for platform-specific inputs.
5. Prefer XR Interaction Toolkit for shared interactions.
6. Use Meta XR SDK for Quest-specific features only.
7. Test first without headset when possible.
8. Use Quest Link for fast headset preview when appropriate.
9. Use standalone APK deployment for final Quest validation.
10. Clearly report which validation steps were completed and which still require manual headset or device testing.

---

## Suggested Prompts for Agents

Use prompts like the following when working with Codex or another coding agent:

```text
This is a Unity XR project targeting Quest first and VIVE Pro Eye later.
Read AGENTS.md before making changes.
Use OpenXR and XR Interaction Toolkit for portable interaction logic.
Keep Meta XR SDK usage isolated to Quest-specific adapters.
Do not put OVR, Meta Movement SDK, VIVE, or SteamVR references in core gameplay scripts.
```

For body tracking work:

```text
Implement body tracking through an IBodyTrackingProvider abstraction.
Create a Quest provider for Meta Movement SDK only inside the Quest platform folder.
Create a simulated provider for editor testing.
Core gameplay must consume only the provider interface.
```

For debugging:

```text
Use Unity MCP to inspect Console errors and project state.
Use Meta XR Simulator for fast no-headset checks.
Use Quest Link only for headset preview.
Do not mark the feature complete until the required standalone Quest APK validation steps are listed.
```

---

## Final Rule

Favor a slightly slower architecture now if it prevents a major rewrite later.

The default decision should be:

```text
Portable core first.
Quest-specific power through adapters.
Real device validation before completion.
```
