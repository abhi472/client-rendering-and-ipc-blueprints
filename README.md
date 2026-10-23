Client Rendering & IPC Blueprints
A Living Architecture Lab for Frontend Systems Engineering
ðŸŽ¯ Purpose
Modern frontend development suffers from heavy framework churn. However, below the syntax layer, every UI toolkit solves the exact same physics problems: calculating state differences, calculating geometry, serializing data across thread boundaries, and pushing pixels to a display controller within 16.6ms.
This repository serves as a comparative benchmark suite and reference architecture. It deconstructs UI frameworks down to their C++, GPU, and memory roots. By isolating the exact mechanisms that push pixels and bridge native hardware, this lab proves that framework volatility is irrelevant when you understand the underlying rendering and IPC pipelines.
ðŸ”­ Architectural Overview
The benchmarks in this repository map how software architectures mutate based on hardware constraints. A flagship mobile device running a heavy Virtual DOM can absorb memory spikes that would instantly trigger an Out-Of-Memory (OOM) crash on a 1GB RAM Set-Top Box.
We classify all UI and Cross-Platform frameworks across four critical layers:
State Synchronization: How does the runtime know what changed? (VDOM Diffing vs. Positional Memoization vs. Structural Identity).
Layout Calculation: Does it use a bundled C++ engine, or delegate to the host OS?
Pixel Execution: Does it mount to native OEM components, or paint directly to a raw GPU canvas using a bundled 2D graphics engine?
Native IPC Bridging: How does the sandboxed UI thread escape to access raw hardware (Edge AI, BLE, Decoders) without dropping frames?
ðŸ—ºï¸ The Universal Rendering & IPC Matrix
graph TD
    subgraph L1 [1. State & Logic Execution]
        A[Compose / KMP <br> Kotlin Compiler & Slot Table]
        B[React Native <br> JS Engine & Virtual DOM]
        C[Flutter <br> Dart Isolate & Widget/Element Trees]
        D[Web-Native STB <br> Immediate WebGL]
    end

    subgraph L2 [2. Layout Calculation]
        A -->|Positional Memoization| A1[Android Measure/Layout]
        B -->|JSI Synchronous Call| B1[Fabric / Yoga C++ Engine]
        C -->|RenderObject Tree| C1[Flutter Custom Layout]
        D -->|Texture Eviction| D1[WPE WebKit]
    end

    subgraph L3 [3. Pixel Pushing / GPU]
        A1 -->|RenderNode| A2[Skia 2D Engine -> Vulkan/Metal]
        B1 -->|Mount to Native View| B2[CoreAnimation / Skia]
        C1 -->|Paint Commands| C2[Impeller 2D Engine -> Vulkan/Metal]
        D1 -->|EGL Buffers| D2[Westeros Compositor -> Wayland]
    end
    
    subgraph L4 [4. IPC & Hardware Bridging]
        B -.->|JSI C++ Pointers| Z[Native Hardware / Edge AI]
        C -.->|Method Channel Serialization| Z
        A -.->|JNI / KMP expect/actual| Z
        D2 -.->|holePunch API| Y[Hardware Video Decoder]
    end

    classDef mobile fill:#0b1e42,stroke:#3b71ca,stroke-width:2px,color:#fff;
    classDef edge fill:#331515,stroke:#dc4c64,stroke-width:2px,color:#fff;
    
    class A,A1,A2,B,B1,B2,C,C1,C2 mobile;
    class D,D1,D2,Y edge;


ðŸ”Œ Inter-Process Communication (IPC) Bridges
Frameworks are sandboxed. Escaping the UI layer to access native hardware exposes the system's true latency bottlenecks. This lab profiles:
Flutter Method Channels: Asynchronous binary serialization costs.
React Native JSI: Holding direct, synchronous C++ memory pointers via the JavaScript Interface.
KMP Expect/Actual: Compiling shared Kotlin directly to native binaries for zero-overhead API execution.
Headless POSIX Daemons: Bypassing OEM SDKs using C++ memory pipelines for deterministic execution on constrained hardware.
ðŸ“‚ Benchmark Modules
/android-compose - Slot Table vs. Legacy XML benchmarks.
/flutter-impeller - Element Tree retention and Method Channel latency tests.
/rn-jsi-fabric - JSI synchronous execution proofs.
/kmp-expect-actual - Compose Multiplatform state sync and native bridging tests.
/rdk-webgl-stb - Texture memory eviction simulators for 1GB RAM constraints.
