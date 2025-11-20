# ☁️ Azure Ubuntu DevOps Development Server

## 🎯 Overview
This repository documents the setup and configuration of an **Azure Ubuntu Development Server** designed for full-stack, multi-language financial application development. The server is configured with containerization tools, multiple language runtimes, and a modern frontend stack to support DevOps and System Engineering workflows.

The entire setup process, including all commands, is detailed in the [`SETUP_GUIDE.md`](SETUP_GUIDE.md) file. A snapshot of the installed version numbers is available in [`dev_tools_manifest.txt`](dev_tools_manifest.txt).

---

## 🛠️ Technology Stack

This server is provisioned to support the following key technologies:

### 1. Backend & Data
* **Languages:** **Python 3**, **Go**, **Java 17 (OpenJDK)**
* **Database:** **PostgreSQL Client** (`psql`) installed, with the server typically run inside **Docker**.

### 2. DevOps & Infrastructure
* **Containerization:** **Docker Engine** and **Docker Compose** plugin.
* **Orchestration:** **k3s** (Lightweight Kubernetes distribution) for local cluster testing.
* **Version Control:** **Git**

### 3. Frontend
* **Runtime:** **Node.js** (LTS via **NVM**) and **NPM**.
* **Framework:** **Vue 3** powered by **Nuxt 3**.
* **Styling:** **Tailwind CSS**.
* **Language:** **TypeScript** for type-safe code.

---

## 🚀 Getting Started

### 1. Prerequisites
You need an SSH client and access credentials to connect to the Azure Ubuntu VM.

### 2. Connect to the Server
```bash
ssh your_user@your_azure_ip_address -p [port]
