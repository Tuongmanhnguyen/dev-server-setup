# 🛠️ Azure Ubuntu Development Machine Setup Guide

This document details the tools, runtimes, and dependencies installed on this Ubuntu VM, designed for a full-stack development environment utilizing Python, Go, Java, and a Vue/Nuxt frontend.

---

## 1. Core Utilities & Version Control

| Tool | Purpose | Installation Method |
| :--- | :--- | :--- |
| **Git** | Distributed Version Control | `apt install git` |
| **curl/wget** | Downloading files | `apt install curl wget` |

### Installation Commands:
```bash
sudo apt update && sudo apt install -y curl wget git build-essential ca-certificates software-properties-common

# Add Docker GPG key and repository
sudo apt-get update
sudo apt-get install ca-certificates curl

sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Post-install: Add user to docker group (requires logout/login)
sudo usermod -aG docker $USER

# Install k3s as a single-node cluster
curl -sfL https://get.k3s.io | sh -
# Verification: sudo kubectl get nodes

# Python (includes pip and virtual environment tools)
sudo apt install -y python3-pip python3-venv

# Go
sudo apt install -y golang

# Java 17 (OpenJDK)
sudo apt install -y openjdk-17-jdk openjdk-17-jre

# PostgreSQL Client
sudo apt install -y postgresql-client

# Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Load NVM (run this after installation/login)
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# Install and set LTS Node.js
nvm install --lts
nvm alias default lts/*
