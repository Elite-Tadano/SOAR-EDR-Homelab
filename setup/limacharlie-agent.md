# LimaCharlie Agent Setup Guide

## 🛡️ Overview
This guide explains how to install and configure the LimaCharlie EDR agent on a Windows 10 machine (VMware) for endpoint detection, logging, and alerting.

---

## ✅ Prerequisites

- A **LimaCharlie account** with an active deployment.
- **Windows 10 VM** (via VMware or other hypervisor).
- Administrative access to the Windows system.
- Internet access for the agent to communicate with LimaCharlie backend.

---

## 🔧 Step-by-Step Installation

### 1. **Log in to LimaCharlie**
- Go to: [https://console.limacharlie.io](https://console.limacharlie.io)
- Create a new **Sensor Deployment Key** if not already available.
- Note your **Deployment Key** and **Tenant ID**.

---

### 2. **Download the Agent Installer**
- Navigate to **Deployments → Windows**
- Select the appropriate installer (`.exe` or `.msi`) for Windows.
- Copy the installer to your Windows 10 VM.

---

### 3. **Install the Agent**
Open **Command Prompt as Administrator** and run:
```powershell
LimaCharlieInstaller.exe /install /key=YOUR_DEPLOYMENT_KEY /tenant=YOUR_TENANT_ID
```
or `.msi`
``` powershell
msiexec /i LimaCharlieSensor.msi KEY=YOUR_DEPLOYMENT_KEY TENANT_ID=YOUR_TENANT_ID /qn
```
✅ If successful, the agent will start as a service and connect to your LimaCharlie dashboard.

---

## 📊 Verify the Agent

- Go to the Sensors section in your LimaCharlie console.
- Your Windows 10 machine should appear as Online.
- You can now run commands, push detection rules, or monitor telemetry.

---

## Testing the Agent
Run simple tests to confirm functionality:
```powershell
whoami
ipconfig
powershell -enc <encoded-reverse-shell>
```
Check if the commands and behaviors are logged or detected in LimaCharlie.

---

## 📚 References
1. https://docs.limacharlie.io/
2. https://docs.limacharlie.io/docs/deploying-sensors

