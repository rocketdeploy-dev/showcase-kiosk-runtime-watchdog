## Reference architecture notice

> This repository is a **reference architecture showcase** extracted from a real, production kiosk system.  
> It focuses on execution control, runtime supervision, and failure recovery — not on business logic
> or a reusable application component.
>
> The broader system context and outcomes are described in the related case study:  
> 👉 https://rocketdeploy.dev/en/case-studies/kiosk-runtime-watchdog

---

# Overview

This repository presents a **minimal runtime watchdog layer** used to supervise
a web-based kiosk application operating on **public, unattended machines**
(e.g. parcel lockers or self-service terminals).

The codebase consists of a **single, self-contained HTML file** that:

- renders a neutral startup and fallback screen,
- embeds the target kiosk application in an iframe,
- supervises application readiness and liveness via `postMessage`,
- recovers automatically when the embedded application becomes unresponsive.

The purpose of this layer is not to run the application —  
but to **protect the kiosk runtime from it**.

---

# System context

The kiosk system is intentionally split into **two physically separate execution environments**:

- a **physical kiosk device**, running a locked-down operating system and a browser in kiosk mode,
- a **local server unit**, hosting the main kiosk web application.

This repository covers the **device-level execution layer**.

The runtime watchdog:
- runs directly on the physical kiosk device,
- is loaded locally by the kiosk browser,
- remains operational regardless of the state of the application runtime.

The main kiosk SPA:
- runs on a **separate machine**,
- is deployed as a long-running web service,
- is accessible only within the machine’s internal LAN.

This separation ensures that application-level failures
do not compromise the kiosk device itself.

---

# Start here (reading map)

1. **`index.html`**
   - Single-file HTML, CSS, and JavaScript.
   - No build step, no external dependencies.
2. **Startup flow**
   - Fallback UI is shown immediately.
   - The kiosk application is loaded into an iframe.
3. **Readiness handshake**
   - The embedded application must emit `APP_READY`.
4. **Heartbeat supervision**
   - Periodic `APP_HEARTBEAT` messages are required to keep the application visible.
5. **Failure recovery**
   - Missing heartbeats or timeouts trigger a controlled fallback and reload.

---

# Architecture at a glance

```
+----------------------------------------------------+
| Physical Kiosk Device                              |
|                                                    |
|  Browser (kiosk mode)                              |
|                                                    |
|  +----------------------------------------------+  |
|  | Runtime Watchdog (this repository)           |  |
|  |                                              |  |
|  |  - startup / fallback UI                     |  |
|  |  - iframe lifecycle control                  |  |
|  |  - heartbeat & timeout supervision           |  |
|  |                                              |  |
|  |      +------------------------------+        |  |
|  |      | Remote Kiosk Web Application |        |  |
|  |      | (SPA on local server, LAN)   |        |  |
|  |      +------------------------------+        |  |
|  +----------------------------------------------+  |
+----------------------------------------------------+
```

Visibility of the embedded application is **earned**, not assumed.

---

# Execution model

## Initialization

1. The watchdog renders a neutral startup screen immediately.
2. The target kiosk application is loaded into an iframe.
3. Until an `APP_READY` message is received, the iframe remains hidden.
4. If initialization exceeds a defined timeout, the iframe is reloaded.

---

## Normal operation

- The embedded application emits periodic `APP_HEARTBEAT` messages.
- Each heartbeat refreshes a timestamp maintained by the watchdog.
- As long as heartbeats arrive on time, the application remains visible.

---

## Failure and recovery

If heartbeats stop or timeouts are exceeded:

- the iframe is hidden,
- a fallback or maintenance message is displayed,
- the embedded application is reloaded after a delay.

Recovery is:
- automatic,
- deterministic,
- independent of user interaction.

---

# Design principles

- **No trust in the embedded application**  
  The iframe is treated as an untrusted execution surface.
- **Deterministic recovery**  
  Every failure path converges back to a known state.
- **Minimal surface area**  
  Single file, no frameworks, no runtime dependencies.
- **Single responsibility**  
  This layer supervises execution; it does not implement application logic.
- **Static deployment**  
  Can be served by a trivial local HTTP server or embedded runtime.

---

# Why this layer exists

In public kiosk environments:

- browser processes may freeze,
- remote applications may fail independently,
- network conditions may fluctuate,
- manual restarts are slow or impossible.

This watchdog provides:

- a **last line of defense** for the kiosk device,
- isolation between device runtime and application logic,
- predictable behavior under partial system failure.

It is intentionally small — and intentionally reliable.

---

# What this is *not*

This repository does **not** provide:

- a reusable kiosk framework,
- a UI toolkit,
- routing or state management,
- business logic of any kind.

It is a **runtime guard**, not an application.

---

# How to evaluate this codebase

- Start with the `postMessage` handling logic.
- Review heartbeat timing and timeout thresholds.
- Inspect how iframe visibility is controlled.
- Verify that every failure case leads to a reset and reload.
- Note how little state is retained between cycles.

The value of this code lies in **what it prevents**, not in what it does.

---

# Relationship to the kiosk application

This runtime watchdog complements the main kiosk SPA:

- the **kiosk application** handles user interaction and business flow,
- the **watchdog layer** handles execution supervision and recovery.

Together, they form a system capable of operating continuously
in a public environment without human supervision.

---

# Closing note

This showcase highlights a small but critical architectural layer
that is often overlooked in kiosk and embedded web systems.

If you are building systems that must:
- run unattended,
- recover automatically,
- and remain predictable under failure,

this pattern is worth considering.

👉 https://rocketdeploy.dev/en/contact
