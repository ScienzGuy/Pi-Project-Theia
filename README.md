# Theia: High-Performance Pi Cluster for BOINC
## Optimized Distributed Computing on Raspberry Pi 5

### Overview
Theia is a dedicated computational node engineered to maximize the throughput of the **Berkeley Open Infrastructure for Network Computing (BOINC)**. By eschewing virtualization and containerization layers, Theia operates on a "bare-metal" configuration, ensuring that every available CPU cycle of the Broadcom BCM2712 is dedicated to scientific research.

The project serves as a blueprint for high-efficiency, small-form-factor volunteer computing, contributing to global projects like Rosetta@home, World Community Grid, and Einstein@Home.

> **Note on Hardware Environment:** Theia resides within the **Jupiter** hardware platform—an extensible, PoE-powered cooling and housing solution. While Theia is functionally independent, the Jupiter environment allows for future horizontal scaling, enabling the addition of more nodes to increase the total credit-generating capacity of the cluster.

### Technical Specifications
Theia is tuned for a 100% duty cycle, requiring a balance between clock speed stability and thermal integrity.

* **Compute:** Raspberry Pi 5 (8GB LPDDR4X-4267 SDRAM).
* **Processor:** Broadcom BCM2712 (2.4GHz quad-core 64-bit Arm Cortex-A76).
* **Operating System:** Raspberry Pi OS Lite (64-bit) / Debian Bookworm.
* **Architecture:** aarch64 native binaries for maximum instruction set efficiency.
* **Thermal Management:** Integrated active cooling with a custom PWM fan curve to maintain a ceiling of 80°C (176°F) under sustained full load.



### Engineering Challenges and Solutions
Developing Theia involved overcoming several hurdles inherent in high-load ARM computing:

**1. The Docker Departure (Bare Metal vs. Containers)**
Early iterations utilized Docker for ease of deployment, but benchmarks revealed a non-negligible overhead in context switching and network abstraction. Moving to a bare-metal installation reduced the memory footprint by ~150MB and eliminated the storage I/O latency that occasionally caused BOINC task "stuttering" on the SD card.

**2. Dependency Management**
Running on bare metal required manual handling of `libstdc++6` and `zlib1g:arm64` dependencies. This ensures that scientific binaries compiled for generic aarch64 run natively without the need for compatibility layers.

**3. Power and Thermal Stability**
Using Power-over-Ethernet (PoE+) provides a stable 25W ceiling. The challenge was preventing voltage drops during peak workloads on all four cores. We addressed this by optimizing the `config.txt` to prevent aggressive frequency scaling, ensuring a consistent 2400MHz clock.

### Deployment and Configuration
Theia is deployed as a headless node. The following steps outline the optimized configuration for the BOINC client:

**1. Base Installation**
Update the host OS and install the core client and text-based interface:
$sudo apt update && sudo apt upgrade -y$ sudo apt install boinc-client boinctui -y

**2. Remote Management Setup**
To allow for centralized monitoring from a management workstation, configure the RPC access:

# Add the management IP to remote_hosts
$ echo "192.168.1.XX" | sudo tee /var/lib/boinc-client/remote_hosts.cfg

# Set a strong GUI RPC password
$echo "YOUR_PASSWORD" | sudo tee /var/lib/boinc-client/gui_rpc_auth.cfg$ sudo systemctl restart boinc-client

**3. Performance Tuning (global_prefs_override.xml)**
We explicitly configure the client to utilize all cores:
* Max CPU usage: 100%
* Max multi-core usage: 100%
* Disk Limit: 20GB (optimized for high-I/O projects)

### Future Scaling
The design of Theia is intentionally modular. Within the **Jupiter** platform, the next phases include:
* **Horizontal Expansion:** Integrating identical Theia nodes (Theia-02, Theia-03) to create a high-density BOINC farm.
* **NVMe Integration:** Moving the BOINC /var/lib/boinc-client directory to NVMe storage via the Pi 5’s PCIe lane to further reduce I/O wait times and extend hardware longevity over traditional SD cards.
