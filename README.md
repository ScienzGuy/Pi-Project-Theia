# Theia: High-Performance Bare-Metal BOINC
## Distributed Scientific Computing on Overclocked Raspberry Pi 5

### Part of the [Project Jupiter] Ecosystem

## 🌌 Overview
Theia is a dedicated computational workhorse engineered for the **Berkeley Open Infrastructure for Network Computing (BOINC)**. Designed to contribute maximum FLOPs to global scientific research, this node represents an optimization-first approach to ARM-based distributed computing. 

By stripping away virtualization layers in favor of a **Bare Metal** architecture, Theia ensures that every clock cycle of the Cortex-A76 is dedicated to scientific throughput rather than container overhead.

Theia resides in the **Project Jupiter** 4-bay tower, functioning as the cluster's primary "Contribution Layer."

> **Note on Extensibility:** While Theia currently operates as a standalone high-performance node, the Jupiter hardware environment is designed for horizontal scaling. The rack is ready to accept additional "Theia-class" nodes (Theia-02, Theia-03) as research demands increase.

## 🛠️ Hardware Specification (Node: Theia)
The infrastructure is tuned for 100% CPU duty cycles with aggressive thermal management.

| Component | Specification |
| :--- | :--- |
| **Compute Node** | Raspberry Pi 5 (8GB RAM) |
| **Cooling** | Official Pi 5 Active Cooler (Custom PWM Fan Curves) |
| **Power** | Waveshare PoE HAT (G) via Netgear PoE+ Managed Switch |
| **Network** | Dedicated Physical Port; Static Internal IP Assignment |
| **Storage** | SanDisk 128GB Max Endurance MicroSD (High-Cycle SLC) |
| **Chassis** | UCTronics 4-Bay Tower (Integrated into Jupiter Rack) |
| **Environment** | Dedicated Basement Rack (Sub-20°C Ambient Floor) |

## 🏗️ The "Bare Metal" Engineering Pivot
Theia’s architecture was redefined to solve specific performance bottlenecks discovered during early cluster testing.

### 1. The Container Exit
To maximize computational efficiency, **BOINC was moved to a Bare Metal installation.** This eliminated the ~15% performance penalty observed during high-I/O scientific tasks (like Rosetta@home) when running inside Docker. Operating on the host OS allows for direct memory addressing and superior process scheduling.

### 2. Dependency Hardening
Running on bare metal required a manual audit of the `aarch64` library stack. By managing `libstdc++6` and `zlib1g` directly on the host, Theia supports a wider array of project binaries without the "DLL Hell" often encountered in mismatched container environments.

### 3. Thermal Integrity
Unlike general-purpose nodes, Theia runs at 100% load indefinitely. We utilize the Pi 5's dedicated fan header and a host-level PWM driver to maintain a strict thermal ceiling, ensuring the SoC never enters a "soft-throttle" state.

## ⚡ Performance Tuning & Thermal Engineering
Theia is pushed to the hardware's limit to maximize "Credit-Per-Day" (CPD) generation.

* **Aggressive Overclock:** Tuned to **2.6 GHz** with manual voltage offsets. This provides the necessary headroom to handle complex astrophysical simulations without stalling.
* **Instruction Set Optimization:** Native `aarch64` binaries are prioritized to utilize the full capabilities of the ARMv8.2-A architecture.
* **Thermal Ceiling:** The system is configured to maintain a target temperature of **< 75°C (167°F)** even during summer ambient spikes, protected by the basement's natural heat-sink properties.

## 📈 Monitoring (SITREP)
Real-time telemetry is critical for a 100% duty cycle node.

* **SoC Thermals:** Continuous monitoring of the delta between the basement floor and the 2.6GHz peak load.
* **SITREP Function:** A custom `.bashrc` function provides an instant hardware health snapshot upon SSH login, displaying CPU frequency, core temp, and BOINC task progress.
* **Remote RPC:** Secured GUI RPC access allows for centralized monitoring via a management workstation without requiring local GUI overhead on the node.

## 🛡️ Security & Hardening
* **Granular Firewall:** UFW rules restrict incoming RPC traffic to specific management IPs.
* **Stateless Operation:** Scientific data is processed in a dedicated `/var/lib/boinc-client` partition to protect system-level stability.

---

Maintained by ScienzGuy | 2026
