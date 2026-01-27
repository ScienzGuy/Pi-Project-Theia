# Theia: Distributed Scientific Computing Pi Cluster
## High-Efficiency BOINC Implementation on Underclocked Pi 5

### 🌌 Overview
Theia is a dedicated two-node sub-cluster designed for high-throughput, 24/7 volunteer computing via the **Berkeley Open Infrastructure for Network Computing (BOINC)**. The cluster is comprised of two Raspberry Pi 5 nodes—Ganymede and Callisto—operating in a unified bare-metal environment.

By moving away from containerization, Theia maximizes computational efficiency and eliminates the security abstractions that previously hindered specialized scientific workloads. The cluster is engineered for "Thermal Sustainability," prioritizing long-term hardware health and consistent credit generation over raw burst speed.

### 🛠️ Hardware Specification (Nodes: Ganymede & Callisto)
Theia is tuned for 100% duty-cycle stability with a focus on strict thermal ceilings.

| Component | Specification |
| :--- | :--- |
| **Compute Nodes** | 2x Raspberry Pi 5 (8GB RAM) |
| **Cooling** | 2x Official Pi 5 Active Coolers (Aggressive Fan Curves) |
| **Power** | 2x Waveshare PoE HAT (G) via PoE+ Managed Switch |
| **Network** | Static Internal IP Assignment; Dedicated Physical Ports |
| **Storage** | 2x SanDisk 128GB Max Endurance MicroSD (High-Cycle SLC) |
| **Environment** | Dedicated Basement Rack (Sub-20°C Ambient Floor) |

### 🏗️ The "Bare Metal" Engineering Pivot
Originally tested in a containerized environment, Theia underwent a total architectural reset to address performance and security bottlenecks.

**1. The Docker Departure**
Docker was eliminated to remove the networking jitter and filesystem abstraction layers that caused latency in high-I/O scientific tasks. Running BOINC on bare metal allows the Linux kernel to manage the Pi 5's 8GB LPDDR4X SDRAM directly, ensuring maximum efficiency for memory-intensive projects.

**2. Security Hardening**
By removing the Docker daemon, we eliminated a significant attack surface. We transitioned to a "Zero Trust" host-level security model, using granular `ufw` rules and restricted RPC access to ensure Ganymede and Callisto remain isolated from the broader network while still communicating with the management console.

### ⚡ Thermal Engineering & Efficiency Tuning
Unlike standard builds, Theia is intentionally "Down-Tuned" to achieve a perfect balance of performance and longevity.

* **Underclocking for Stability:** Both nodes are pinned to **2.3 GHz**. This slight reduction from the stock 2.4 GHz allows for a significant reduction in core voltage.
* **Undervolting:** By applying a manual voltage offset, we have reduced the power draw and heat signature of the BCM2712 silicon.
* **Thermal Ceiling:** The combination of underclocking and aggressive cooling maintains a rock-solid **55°C (131°F)** while running at 100% load 24/7. This prevents any possibility of thermal throttling and extends the lifespan of the PoE hardware.

### 📈 Monitoring (SITREP)
Theia utilizes a custom telemetry stack for real-time health tracking:

* **SITREP Function:** A custom `.bashrc` function providing an instant hardware snapshot (Temp, Clock, Load, and BOINC status) upon SSH login.
* **Headless Management:** Nodes are managed via `boinctui` and `boinc_cmd`, keeping the OS footprint as lean as possible.
* **Telemetry:** Continuous monitoring of the delta between the 55°C core temp and the basement's sub-20°C ambient floor.

---

Maintained by Scienz_Guy | 2026
