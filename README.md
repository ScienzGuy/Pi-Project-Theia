# Project Bramble: "Big Science" Nomad Cluster

An overclocked, high-performance Raspberry Pi 5 cluster dedicated to distributed computing and scientific research via BOINC, orchestrated by HashiCorp Nomad.

## 🏗️ Hardware Architecture
The cluster, nicknamed **"Bramble,"** is designed for maximum compute density within a 63W PoE power budget.

* **Nodes:** 4x Raspberry Pi 5 (8GB RAM)
    * **Ganymede:** Nomad Server/Client (Master Node) - `192.168.86.126`
    * **Callisto:** Nomad Client (Worker Node) - `192.168.86.127`
    * **Io & Europa:** (Provisioning pending)
* **Performance Profile:**
    * **CPU:** Overclocked to **2.6GHz** (`arm_freq=2600`)
    * **Voltage:** `over_voltage_delta=50000`
* **Power & Cooling:**
    * **Infrastructure:** UCTronics PoE HATs with active cooling.
    * **Switch:** 63W Total PoE Budget.
* **Storage:**
    * **Persistence:** Host-volume mapping at `/opt/boinc_data`.



## 🚀 Software Stack
* **OS:** Raspberry Pi OS (64-bit)
* **Orchestration:** [HashiCorp Nomad](https://www.nomadproject.io/)
* **Containerization:** [Docker](https://www.docker.com/)
* **Workload:** [BOINC](https://boinc.berkeley.edu/)

## 🔧 Core Configuration

### Nomad Driver Setup (`nomad.hcl`)
To allow BOINC to access the CPU effectively and store data persistently, the Nomad agent must be configured to allow privileged Docker containers and volume mounting:

```hcl
client {
  enabled = true
  options = {
    "docker.privileged.enabled" = "true"
    "docker.volumes.enabled"    = "true"
  }
}

plugin "docker" {
  config {
    allow_privileged = true
    volumes {
      enabled = true
    }
  }
}
🛠️ Troubleshooting & Lessons Learned
1. exec format error (Architecture Mismatch)
Symptoms: Task restarts indefinitely; logs show exec format error. Cause: Attempting to run an x86_64 image on ARM64 hardware. Fix: Explicitly use the arm64v8 tag in the job file.

Bash

sudo docker pull boinc/client:arm64v8
2. connection refused (Agent Crash)
Symptoms: dial tcp 127.0.0.1:4646: connect: connection refused. Cause: Syntax error in nomad.hcl or driver initialization failure. Fix: Test config manually with sudo nomad agent -config=/etc/nomad.d/nomad.hcl.

3. DNS Lookup Timeouts
Symptoms: i/o timeout during image pulls. Cause: High CPU load interfering with DNS resolution during heavy 2.6GHz activity. Fix: Manually pull images on nodes before running jobs.

4. Thermal & Power Constraints
Symptoms: Clock speed throttles to 1.5GHz or nodes reboot. Cause: CPU > 80°C or PoE budget exceeded (>63W total). Fix: Monitor vitals with vcgencmd measure_temp and adjust resources in the Nomad job file if necessary.

📊 Management
The cluster is managed remotely via the BOINC Manager (Advanced View).

Port: 31416

RPC Password: secret_password_here

Remote Access: Enabled via --allow_remote_gui_rpc in the job environment.
