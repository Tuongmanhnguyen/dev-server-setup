Here is the consolidated, master documentation for your Azure Development Infrastructure. It integrates the specifications for both VMs, the detailed "Bot-Proofing" implementation, and the standard development environment setup.

You can commit this directly to your GitHub repo (`dev-server-setup`) as `INFRASTRUCTURE.md`.

---

# Azure Development Infrastructure & Governance Bypass Documentation

**Repository:** [https://github.com/Tuongmanhnguyen/dev-server-setup](https://github.com/Tuongmanhnguyen/dev-server-setup)
**Resource Group:** `tn1ndevrg`
**Location:** Southeast Asia
**Network:** Shared VNet/Subnet

---

## 1\. Virtual Machine Specifications

### **Primary Node: `dev-standard` (The "Bot-Proof" Machine)**

_Current active development machine configured for persistence._

| Category           | Specification               | Notes                                          |
| :----------------- | :-------------------------- | :--------------------------------------------- |
| **Machine Size**   | **Standard_D4as_v5**        | 4 vCPUs, 16 GB RAM (AMD EPYC)                  |
| **OS**             | **Ubuntu Server 22.04 LTS** | Selected for native Azure Hibernation support. |
| **Hibernation**    | **Enabled**                 | Critical for bypassing governance shutdowns.   |
| **Storage (OS)**   | 128 GB Premium SSD          |                                                |
| **Storage (Swap)** | 24 GB                       | `/hibfile.sys` (Reserved for memory state).    |
| **SSH Port**       | **2277**                    | Custom port (requires `-p 2277`).              |
| **Private IP**     | _[Internal VNet IP]_        |                                                |
| **Public Access**  | Dynamic IP                  | **Connection:** `ssh tn1n@<PUBLIC-IP> -p 2277` |

### **Secondary Node: `devUbuntu` (Legacy/Backup)**

_Previous high-spec machine (currently vulnerable to shutdown)._

| Category         | Specification                  | Notes                                         |
| :--------------- | :----------------------------- | :-------------------------------------------- |
| **Machine Size** | **Standard_D8s_v5**            | 8 vCPUs, 32 GB RAM                            |
| **OS**           | **Ubuntu Server 24.04 LTS**    | Newer OS, but hibernation support is complex. |
| **Hibernation**  | Disabled/Unsupported           | State is lost upon deallocation.              |
| **SSH Port**     | **2277**                       | Custom port.                                  |
| **Connection**   | `ssh tn1n@<PUBLIC-IP> -p 2277` |                                               |

---

## 2\. The "Bot-Proof" Strategy (Governance Evasion)

**Constraint:** The subscription enforces a "Governance Bot" (`MCAPSGov`) that force-deallocates VMs roughly between 12:00 AM - 01:00 AM.
**Solution:** Automated Pre-emptive Hibernation ("Duck & Cover").

### The Mechanism

1.  **Safety-Hibernate (23:00):** We hibernate the VM manually via automation before the bot arrives. This saves RAM/Processes to disk.
2.  **The Pass (00:00 - 01:00):** The bot scans the subscription, sees the VM is already "Stopped," and takes no action.
3.  **Morning-WakeUp (02:00):** We wake the VM automatically. RAM restores, and we patch the networking.

### Implementation Steps (Log)

#### **Phase 1: OS Configuration (Inside `dev-standard`)**

These steps were performed to allow the Kernel to suspend to disk.

```bash
# 1. Installed Microsoft Hibernation Tool
curl -sSL https://packages.microsoft.com/keys/microsoft.asc | sudo tee /etc/apt/trusted.gpg.d/microsoft.asc
sudo apt-add-repository https://packages.microsoft.com/ubuntu/22.04/prod
sudo apt-get update && sudo apt-get install hibernation-setup-tool -y

# 2. Configured Swap & GRUB
sudo hibernation-setup-tool
# (Tool automatically created /hibfile.sys and updated /etc/default/grub.d/99-hibernate-settings.cfg)

# 3. Verified
# Rebooted and ran `uptime` after a test hibernate to confirm state persistence.
```

#### **Phase 2: Azure Automation**

An Azure Automation Account (Managed Identity enabled) was created with `Virtual Machine Contributor` rights on `dev-standard`.

**Runbook 1: `Safety-Hibernate`**

- **Schedule:** Daily @ 23:00 (11:00 PM) Indochina Time.
- **Action:** `Stop-AzVM -Hibernate -Force`.

**Runbook 2: `Morning-WakeUp`**

- **Schedule:** Daily @ 02:00 (02:00 AM) Indochina Time.
- **Action:** `Start-AzVM`.
- **Fix:** Runs `sudo systemctl restart k3s` via RunCommand to repair Container Networking Interfaces (CNI) after the time-jump.

---

## 3\. Development Environment Setup

**Source of Truth:** [GitHub Repo: dev-server-setup](https://github.com/Tuongmanhnguyen/dev-server-setup)

The following tools are provisioned on the `dev-standard` machine. Refer to the repository for the Ansible/Shell scripts used to configure them.

### **Core Tools**

- **Docker & Docker Compose:** Container runtime.
- **K3s (Lightweight Kubernetes):** configured for single-node cluster testing.
- **Git:** Version control.
- **Python:** `pip`, `venv`, and `virtualenv` installed.
- **Java:** OpenJDK 17 (LTS).
- **Go:** Latest stable release.
- **Database Clients:** `postgresql-client`.

### **Workflow Best Practices**

#### **1. Dealing with Hibernation**

Since the network disconnects at 23:00:

- **VS Code Remote SSH:** Requires a window reload in the morning (`Ctrl+Shift+P` \> `Reload Window`).
- **Terminals:** Use `tmux` if running long jobs outside of VS Code to prevent broken pipes.

#### **2. K3s Status**

If Kubernetes pods cannot communicate after waking up:

```bash
# The automation script does this, but if it fails:
sudo systemctl restart k3s
```

---

## 4\. Automation Scripts Reference (PowerShell)

For disaster recovery, here are the scripts deployed in the Automation Account.

**`Safety-Hibernate`**

```powershell
Param([string]$RG="tn1ndevrg", [string]$VM="dev-standard")
$Connect = Connect-AzAccount -Identity
Stop-AzVM -ResourceGroupName $RG -Name $VM -Hibernate -Force -ErrorAction Stop
```

**`Morning-WakeUp`**

```powershell
Param([string]$RG="tn1ndevrg", [string]$VM="dev-standard")
$Connect = Connect-AzAccount -Identity
Start-AzVM -ResourceGroupName $RG -Name $VM -ErrorAction Stop
# Stabilization wait
Start-Sleep -Seconds 30
# Network Fix
Invoke-AzVMRunCommand -ResourceGroupName $RG -Name $VM -CommandId 'RunShellScript' -Script 'sudo systemctl restart k3s'
```
