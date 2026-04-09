# 🤖 Self-Hosted N8N Automation Server on macOS via VirtualBox

> Run a fully local N8N workflow automation server on Mac using VirtualBox, Ubuntu 24, Docker, and EasyPanel — no cloud required.

---

## 📋 Overview

This project documents how to set up a **self-hosted N8N instance** running inside a Linux virtual machine on macOS, accessible directly from Mac browser. It covers the full stack from VM creation to browser access, including networking configuration and common troubleshooting steps.

---

## 🛠️ Stack

| Tool | Role |
|------|------|
| **VirtualBox** | Hypervisor — runs a Linux VM on macOS |
| **Ubuntu 24** | Guest OS inside the VM |
| **Docker** | Container runtime installed inside Ubuntu |
| **EasyPanel** | Web-based server dashboard to deploy services |
| **N8N** | Open-source workflow automation (self-hosted) |
| **Traefik** | Reverse proxy bundled with EasyPanel |

---

## 🏗️ Architecture

```
macOS Chrome → :8080 → VirtualBox NAT → :80 (Ubuntu VM) → Traefik → N8N :5678
```

---

## 🚀 Setup Guide

### 1. Install VirtualBox
Download and install [VirtualBox](https://www.virtualbox.org/) on your Mac.

### 2. Create an Ubuntu 24 VM
- Download the [Ubuntu 24 ISO](https://ubuntu.com/download/desktop)
- In VirtualBox: New VM → select the ISO → allocate at least **4GB RAM** and **25GB disk**
- Complete the Ubuntu installation inside the VM

Note: I have followed this tutorial to successfully install Ubuntu in my VM: https://www.youtube.com/watch?v=dKJ3Wee8w9w


### 3. Install Docker and EasyPanel inside Ubuntu

For this part I have used the official EasyPanel guide: https://easypanel.io/docs

Open a terminal inside the VM and run:

curl -sSL https://get.docker.com | sh

I had to install curl first, then I was able to run this

Then I was able to continue

However, in the guide the command to copy-paste was given in different lines so bash interpreted easypanel/easypanel as a script to run locally rather than a Docker image name:

docker run --rm -it \
  -v /etc/easypanel:/etc/easypanel \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  easypanel/easypanel setup

I then asked support to Claude.ai and created the same command in just one line:
  
sudo docker run --rm -it -v /etc/easypanel:/etc/easypanel -v /var/run/docker.sock:/var/run/docker.sock:ro easypanel/easypanel setup

In case this method doesn't work, AI has also given me the following commands:

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
```

Access EasyPanel inside the VM at: `http://localhost:3000`

### 5. Deploy N8N via EasyPanel
- In EasyPanel, go to **Services → Create Service → Template**
- Search for **N8N** and deploy it

### 6. Configure N8N Domain
- Go to your N8N service → **Domains → Create Domain**
- Set **Host** to `n8n.localhost`
- Disable **HTTPS**
- Set destination port to **5678**
- Save and redeploy

### 7. Disable Secure Cookie
- Go to N8N service → **Environment**
- Add: `N8N_SECURE_COOKIE` = `false`
- Redeploy the service

### 8. Configure VirtualBox Port Forwarding
Shut down the VM, then go to:
**Settings → Network → Adapter 1 → Advanced → Port Forwarding**

Add this rule:

| Name | Protocol | Host IP | Host Port | Guest Port |
|------|----------|---------|-----------|------------|
| traefik | TCP | 127.0.0.1 | 8080 | 80 |

Start the VM again.

### 9. Update macOS Hosts File
On your Mac terminal:
```bash
sudo nano /etc/hosts
```
Add at the bottom:
```
127.0.0.1   n8n.localhost
```
Save with `Ctrl+O` then `Ctrl+X`.

### 10. Access N8N from macOS
Open Chrome on your Mac and go to:
```
http://n8n.localhost:8080
```

---

## 🔧 Troubleshooting

**Port 80 not working on Mac**
Use Host Port `8080` instead — port 80 requires root on macOS.

**N8N secure cookie error**
Set `N8N_SECURE_COOKIE=false` in the N8N environment variables in EasyPanel.

**Docker network errors after restart**
```bash
sudo docker network prune -f
```
Then redeploy services from EasyPanel.

**N8N container not showing in `docker ps`**
Redeploy from EasyPanel → N8N service → Deploy button.

**Default EasyPanel domain not loading**
The `easypanel.host` domain requires a public server. Use `n8n.localhost` for local access instead.

---

## 📁 Key Commands Reference

```bash
# Check running containers
sudo docker ps

# View N8N logs
sudo docker logs $(sudo docker ps -q --filter "name=n8n") --tail 50

# Restart all containers
sudo docker restart $(sudo docker ps -q)

# Clean up broken networks
sudo docker network prune -f

# Test connectivity from Mac
curl -H "Host: n8n.localhost" http://127.0.0.1:8080
```

---

## 📚 Resources

- [N8N Documentation](https://docs.n8n.io)
- [EasyPanel Documentation](https://easypanel.io/docs)
- [VirtualBox Manual](https://www.virtualbox.org/manual/)
- [Docker Documentation](https://docs.docker.com)
- IA: Claude.ai
- N8N tips and info from Raiola Networks course (in Spanish) (https://cursos.raiola.link/)
---

## 👤 Author

**Dario Gambino**  
Cloud Engineering & DevOps enthusiast | AWS Cloud Support Specialist in progress | AZ-900 certified  
[LinkedIn](https://www.linkedin.com/in/dario-gambino/) • [GitHub](https://github.com/d-gam/portfolio)
