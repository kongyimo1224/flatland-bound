# Flatland Bound

A networked two-player co-op puzzle platformer that I built on my own in September–October 2026 for the CSD3116 distributed game lab at SIT–DigiPen.

**Stack:** C11 · OpenGL 3.3 · GLFW · Cyclone DDS (with DDS Security) · Linux / WSL

> The source code is course work, so that repository stays private under DigiPen's academic-integrity rules. This page is the project write-up. I'm happy to walk through the code in an interview.

## The idea

Two players look at the same 3D voxel world from perpendicular sides, so each one sees a different flat level. Neither can finish a puzzle alone. Either player can turn the world to any angle, and the physics keeps running during the turn.

## Problems, what I changed, what happened

### 1. Input felt laggy over the network
- **Problem:** with a server-authoritative loop, every key press waited for a round trip. Under emulated ping, input-to-screen latency was 135–318 ms.
- **Change:** client-side prediction with server reconciliation, over a sequence-locked input stream sent every tick.
- **Result:** input-to-screen latency stays at one frame (17 ms) at any emulated ping. With 10% input loss, only 4 of 928 snapshots needed a correction.

### 2. The server burned CPU while idle
- **Problem:** the first server busy-waited on DDS reads and used 199% CPU while doing nothing.
- **Change:** an event-driven loop on DDS waitsets, with lock-free single-producer single-consumer queues (C11 atomics) between threads.
- **Result:** idle CPU dropped to 1.7%.

### 3. Both players must start a world turn together
- **Change:** NTP-style clock sync between the clients and the server.
- **Result:** turn starts line up within 2–5 ms at 100 ms emulated latency.

### 4. Testing networked code by hand does not scale
- **Change:** an automated test harness drives both clients with scripted inputs under emulated latency and packet loss, then measures latency and corrections. A level generator and a two-player solver run on the real physics, and the solver's routes are replayed as input tapes to record the demo footage automatically.

## Rendering and physics

- Any-angle projected collision, and deterministic physics that keeps running mid-turn
- Renderer with shadows, HDR bloom and ACES tone mapping

## Contact

Yimo Kong · kongyimoo@gmail.com · [LinkedIn](https://www.linkedin.com/in/yimokong1224/) · [Portfolio](https://kongyimo1224.github.io/)
