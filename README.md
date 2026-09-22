# FTP Web UI

A beautiful, futuristic FTP file manager built with Go (Gin) and a pure HTML/CSS/JS frontend.

## Features

- ⚡ Futuristic dark UI with animated grid background & glowing accents
- 📂 Browse, navigate, and sort files & directories
- ⬆️ Upload files - drag & drop or click to select
- ⬇️ Download files with one click
- 🗂️ Create new folders
- ✏️ Rename files and folders
- 🗑️ Delete files and folders (with confirmation)
- 🔍 Real-time file filter/search
- 🍞 Breadcrumb navigation
- 📱 Mobile responsive
- 🔔 Toast notifications & progress bar

---

## Deployment (from Windows to the VPS)

### Option A - Automated (PowerShell)

Open PowerShell, then:

```powershell
cd C:\ftpui
.\deploy.ps1
```

This will:
1. Copy all files to `/opt/ftpui` on the server via SCP
2. Install Go on the server if needed
3. Build the binary
4. Install and start a `systemd` service
5. Open port 8080 in the firewall

> **Note:** Windows must have OpenSSH installed (built in on Windows 10/11).
> Authenticate with the server's root credentials; they are not recorded here.

---

### Option B - Manual (step by step)

**1. Copy files to server:**
```powershell
scp -r C:\ftpui root@<server>:/opt/ftpui
```

**2. SSH into the server:**
```powershell
ssh root@<server>
```

**3. On the server - install Go (if not present):**
```bash
cd /tmp
wget https://go.dev/dl/go1.22.4.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.22.4.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> /etc/profile
source /etc/profile
go version   # should print: go version go1.22.4 linux/amd64
```

**4. Build & run:**
```bash
cd /opt/ftpui
go mod tidy
go build -o ftpui-server .

# Test it works:
FTP_HOST=localhost FTP_PORT=21 APP_PORT=8080 ./ftpui-server
```

**5. Install as a service (auto-start):**
```bash
cat > /etc/systemd/system/ftpui.service <<'EOF'
[Unit]
Description=FTP Web UI
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/opt/ftpui
Environment=FTP_HOST=localhost
Environment=FTP_PORT=21
Environment=APP_PORT=8080
ExecStart=/opt/ftpui/ftpui-server
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable ftpui
systemctl start ftpui
systemctl status ftpui
```

**6. Open the firewall port:**
```bash
# Ubuntu/Debian (ufw)
ufw allow 8080/tcp

# CentOS/RHEL (firewalld)
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload
```

**7. Open in browser:**
```
http://<server>:8080
```

---

## Configuration

| Environment Variable | Default     | Description              |
|----------------------|-------------|--------------------------|
| `FTP_HOST`           | `localhost` | FTP server hostname/IP   |
| `FTP_PORT`           | `21`        | FTP port                 |
| `APP_PORT`           | `8080`      | Web UI port              |

---

## Useful Commands

```bash
# View logs
journalctl -u ftpui -f

# Restart service
systemctl restart ftpui

# Stop service
systemctl stop ftpui

# Check status
systemctl status ftpui
```

---

## Project Structure

```
/opt/ftpui/
├── main.go          ← Go backend (Gin HTTP server + FTP proxy)
├── go.mod           ← Go module definition
├── go.sum           ← Dependency checksums
├── ftpui-server     ← Compiled binary (after build)
└── static/
    └── index.html   ← Full frontend (HTML + CSS + JS)
```
