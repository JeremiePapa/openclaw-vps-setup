# 🦞 OpenClaw Setup on VPS (with existing n8n + Traefik)

This guide shows how to install and run OpenClaw on a VPS that already has:

* Docker + Traefik
* n8n running
* Root SSH access

---

## 📌 Overview

We will:

1. Install Node.js (v22+)
2. Install OpenClaw via npm
3. Initialize OpenClaw (onboarding)
4. Start the Gateway
5. Access the Web UI via SSH tunnel
6. Fix system limits (important for stability)

---

## ⚙️ Prerequisites

* Ubuntu/Debian VPS
* Root access
* n8n already running (optional but assumed)
* Ports 80/443 already handled by Traefik

---

## 1️⃣ Install Node.js (v22+)

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs
```

Verify:

```bash
node -v
npm -v
```

---

## 2️⃣ Install Build Dependencies

Required for native modules (e.g. `sharp`):

```bash
apt update
apt install -y build-essential python3 make g++ cmake libvips-dev
```

---

## 3️⃣ Install OpenClaw

```bash
npm install -g openclaw
```

Verify:

```bash
openclaw --help
```

---

## 4️⃣ Fix File Descriptor Limit (IMPORTANT)

OpenClaw uses sockets → prevents crashes.

Edit limits:

```bash
nano /etc/security/limits.conf
```

Add at the bottom:

```bash
root soft nofile 65535
root hard nofile 65535
* soft nofile 65535
* hard nofile 65535
```

---

Update systemd limits:

```bash
nano /etc/systemd/system.conf
```

Add:

```bash
DefaultLimitNOFILE=65535
```

```bash
nano /etc/systemd/user.conf
```

Add:

```bash
DefaultLimitNOFILE=65535
```

Apply:

```bash
systemctl daemon-reexec
reboot
```

---

## 5️⃣ Initialize OpenClaw

Run:

```bash
openclaw onboard --mode local
```

### During setup:

* **Security** → Yes
* **Setup Mode** → QuickStart
* **Config Handling** → Reset
* **Reset Scope** → Config only

---

### Model Provider

* Select: **OpenRouter**
* Enter your API key

---

### Model

```bash
openrouter/anthropic/claude-3-haiku
```

---

### Other Options

* Channels → Skip for now
* Web Search → Skip for now
* Skills → Yes
* Dependencies → Skip for now

---

### API Keys

* Google Places → No
* Notion → No
* Whisper → No
* ElevenLabs → No

---

### Hooks

* Skip for now

---

### Gateway Service

* Restart

---

### UI Launch

* Open the Web UI

---

## 6️⃣ Start OpenClaw Gateway

```bash
openclaw gateway
```

Verify:

```bash
ss -tulnp | grep 18789
```

Expected:

```bash
127.0.0.1:18789 LISTEN
```

---

## 7️⃣ Access OpenClaw UI (SSH Tunnel)

⚠️ Run from your LOCAL machine (not VPS):

```bash
ssh -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

Open browser:

```bash
http://localhost:18789/#token=YOUR_TOKEN
```

---

## ✅ Final Result

You now have:

* OpenClaw Gateway running
* OpenClaw Web UI accessible
* OpenRouter connected
* Clean minimal setup
* Ready for automation (n8n integration)

---

## 🧠 Recommended Architecture

```
OpenClaw (AI)
     ↓
n8n (workflow engine)
     ↓
Playwright (execution)
```

---

## 🚀 Next Steps

* Connect OpenClaw → n8n via webhook
* Build automation workflows
* Add Playwright scripts
* Optionally expose via Traefik

---

## ⚠️ Notes

* OpenClaw binds to `127.0.0.1` (secure by default)
* Use SSH tunnel or reverse proxy (Traefik)
* Avoid installing unnecessary skills early

---

## 🧩 Troubleshooting

### UI not loading

```bash
ss -tulnp | grep 18789
```

### Command not found

```bash
npm list -g --depth=0
```

### Too many open files

```bash
ulimit -n
```

---

## 🎯 Done

You now have a working AI gateway ready to integrate with automation tools like n8n.
