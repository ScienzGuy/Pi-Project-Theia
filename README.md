# Project Bramble: "Big Science" Nomad Cluster

An orchestrated Raspberry Pi 5 cluster dedicated to distributed computing and scientific research via BOINC, managed by HashiCorp Nomad.

## 🏗️ Hardware Architecture

The cluster, nicknamed **"Bramble,"** is designed for high compute density and long-term stability.

* **Nodes:** 2x Raspberry Pi 5 (8GB RAM)
    * **Ganymede:** Nomad Server/Client (Master) - `192.168.86.126`
    * **Callisto:** Nomad Client (Worker) - `192.168.86.127`
* **Power & Cooling:** * UCTronics PoE HATs with PWM-controlled active cooling.
    * Managed via Netgear Managed Switch with 63W Total PoE Budget.

---

## 🛠 Hardware Configuration: The 2026 "Wide & Cool" Baseline

As of January 2026, the cluster has been migrated from an unstable 2.6GHz overclock to a stabilized, underclocked profile to ensure long-term reliability.

| Setting | Value | Impact |
| :--- | :--- | :--- |
| **CPU Clock Speed** | **2.2GHz** (2200 MHz) | Prevents thermal throttling and kernel panics. |
| **Voltage Offset** | **-50mV** (Undervolt) | Lowers power draw and heat while maintaining stability. |
| **Target Temp** | **~70-73°C** | Optimal thermal signature under 100% load. |

---

## 🚀 Software Stack

* **OS:** Raspberry Pi OS (64-bit) / Ubuntu 25.04
* **Orchestration:** HashiCorp Nomad 1.7+
* **Containerization:** Docker
* **Workload:** BOINC (Asteroids@home)

### Self-Healing "Janitor" Logic
To combat stale lockfiles during power cycles, the Nomad job includes a pre-start task:

    task "janitor" {
      driver = "raw_exec"
      lifecycle {
        hook    = "pre_start"
        sidecar = false
      }
      config {
        command = "/usr/bin/sudo"
        args    = ["/usr/bin/rm", "-f", "/opt/boinc_data/lockfile"]
      }
    }

---

## 🔧 Core Configuration

### Nomad Driver Setup (`nomad.hcl`)
Configured to allow privileged Docker containers and host volume mounting:

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

---

## 🛠️ Troubleshooting & Lessons Learned

1.  **Authorization Failure (-155):**
    * **Cause:** RPC password desync between host and container.
    * **Fix:** Use `remote_hosts.cfg` to trust `127.0.0.1` and bypass password requirements for local `boinccmd` calls.

2.  **Port Conflict (Error 500):**
    * **Cause:** Native `boinc-client.service` running on the host OS grabbing Port 31416.
    * **Fix:** Disable native service: `sudo systemctl disable boinc-client`.

3.  **Architecture Mismatch:**
    * **Fix:** Ensure use of `boinc/client:arm64v8` tags for Pi hardware.

4.   **Thermal Management:**
    * **Observation:** Callisto (top node) consistently runs ~2-5°C hotter than Ganymede due to thermal stacking.
    * **Fix:** Implemented aggressive PWM fan curves starting at 50°C.

---

## 📊 Management
The cluster is managed remotely via the BOINC Manager (Advanced View) or `boinctui`.

* **Port:** 31416
* **Remote Access:** Enabled via `--allow_remote_gui_rpc` in the job environment.
