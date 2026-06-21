# 🌐 Cloudflare Tunnel on Windows — Complete Setup Guide

> Expose your `localhost` to the internet on **Windows** — no port forwarding, no firewall rules, no static IP.

---

## 📖 Table of Contents

- [Prerequisites](#prerequisites)
- [Install cloudflared on Windows](#install-cloudflared-on-windows)
  - [Option A — Download EXE (Easiest)](#option-a--download-exe-easiest)
  - [Option B — Winget (Windows Package Manager)](#option-b--winget-windows-package-manager)
  - [Option C — Scoop](#option-c--scoop)
- [Quick Start — Random URL (No Account Needed)](#quick-start--random-url-no-account-needed)
- [Setup with Custom Domain](#setup-with-custom-domain)
  - [Step 1 — Add Domain to Cloudflare](#step-1--add-domain-to-cloudflare)
  - [Step 2 — Login with cloudflared](#step-2--login-with-cloudflared)
  - [Step 3 — Create the Tunnel](#step-3--create-the-tunnel)
  - [Step 4 — Create Config File](#step-4--create-config-file)
  - [Step 5 — Route DNS](#step-5--route-dns)
  - [Step 6 — Run the Tunnel](#step-6--run-the-tunnel)
- [Run as Windows Service (Auto-start on Boot)](#run-as-windows-service-auto-start-on-boot)
- [Multiple Local Services on One Tunnel](#multiple-local-services-on-one-tunnel)
- [Useful Commands](#useful-commands)
- [Troubleshooting on Windows](#troubleshooting-on-windows)
- [Security Tips](#security-tips)

---

## Prerequisites

| Requirement | Details |
|---|---|
| Windows Version | Windows 10 / 11 (64-bit) |
| Terminal | PowerShell (Admin) or Command Prompt |
| Cloudflare Account | Free — only needed for custom domain |
| Domain Name | Only for custom domain setup |

> ✅ **No account needed** for a quick random public URL!

---

## Install cloudflared on Windows

### Option A — Download EXE (Easiest)

1. Go to the [latest releases page](https://github.com/cloudflare/cloudflared/releases/latest)
2. Download **`cloudflared-windows-amd64.exe`**
3. Rename it to **`cloudflared.exe`**
4. Move it to `C:\Windows\System32\` so it's available everywhere

Verify in PowerShell:

```powershell
cloudflared --version
```

Expected output:
```
cloudflared version 2024.x.x
```

---

### Option B — Winget (Windows Package Manager)

Open **PowerShell** or **CMD** and run:

```powershell
winget install --id Cloudflare.cloudflared
```

Then restart your terminal and verify:

```powershell
cloudflared --version
```

---

### Option C — Scoop

```powershell
scoop install cloudflared
```

---

## Quick Start — Random URL (No Account Needed)

The fastest way to share your localhost — great for demos and testing.

**Step 1 — Start your local app** (e.g., on port 3000):

```powershell
npm run dev
# or
python -m http.server 3000
# or whatever starts your app
```

**Step 2 — Open a new PowerShell window and run:**

```powershell
cloudflared tunnel --url http://localhost:3000
```

You'll see output like this:

```
+-----------------------------------------------------------------------+
| Your quick Tunnel has been created! Visit it at:                      |
| https://orange-sunset-bears-developer.trycloudflare.com               |
+-----------------------------------------------------------------------+
```

✅ Done! Share that URL with anyone — it works instantly from anywhere.

> ⚠️ The random URL **changes every time** you restart cloudflared.  
> Use a custom domain below for a permanent URL.

---

## Setup with Custom Domain

Get a permanent URL like `https://app.yourdomain.com`.

### Step 1 — Add Domain to Cloudflare

1. Log in to [dash.cloudflare.com](https://dash.cloudflare.com)
2. Click **"Add a Site"** → enter your domain → choose **Free plan**
3. Update your domain registrar's **nameservers** to those Cloudflare provides
4. Wait for activation (a few minutes to a few hours)

> Skip this step if your domain is already on Cloudflare.

---

### Step 2 — Login with cloudflared

Open **PowerShell as Administrator** and run:

```powershell
cloudflared tunnel login
```

- A browser window opens at Cloudflare
- Select your domain and click **Authorize**
- A certificate is saved to:

```
C:\Users\YOUR_USERNAME\.cloudflared\cert.pem
```

---

### Step 3 — Create the Tunnel

```powershell
cloudflared tunnel create my-app-tunnel
```

Output:

```
Tunnel credentials written to C:\Users\YOUR_USERNAME\.cloudflared\xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.json
Created tunnel my-app-tunnel with id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

📌 Copy and save that **tunnel ID** — you'll need it next.

---

### Step 4 — Create Config File

Cloudflare looks for the config at:

```
C:\Users\YOUR_USERNAME\.cloudflared\config.yml
```

**Create it using Notepad:**

```powershell
notepad $env:USERPROFILE\.cloudflared\config.yml
```

Paste this content (replace with your values):

```yaml
tunnel: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
credentials-file: C:\Users\YOUR_USERNAME\.cloudflared\xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.json

ingress:
  - hostname: app.yourdomain.com
    service: http://localhost:3000
  - service: http_status:404
```

Save and close Notepad.

**Verify your config is valid:**

```powershell
cloudflared tunnel ingress validate
```

---

### Step 5 — Route DNS

This automatically adds a CNAME record in your Cloudflare DNS:

```powershell
cloudflared tunnel route dns my-app-tunnel app.yourdomain.com
```

For multiple subdomains, repeat the command:

```powershell
cloudflared tunnel route dns my-app-tunnel api.yourdomain.com
cloudflared tunnel route dns my-app-tunnel admin.yourdomain.com
```

---

### Step 6 — Run the Tunnel

```powershell
cloudflared tunnel run my-app-tunnel
```

Your app is now live at `https://app.yourdomain.com` 🎉

---

## Run as Windows Service (Auto-start on Boot)

Don't want to keep a terminal open? Install cloudflared as a **Windows Service** so it starts automatically every time Windows boots.

**Open PowerShell as Administrator**, then:

```powershell
# Install the service
cloudflared service install

# Start it immediately
Start-Service cloudflared

# Check the status
Get-Service cloudflared
```

**Manage via Services GUI:**

1. Press `Win + R` → type `services.msc` → hit Enter
2. Find **"Cloudflared"** in the list
3. Right-click → **Start / Stop / Restart** as needed

**View logs (Event Viewer):**

1. Press `Win + R` → type `eventvwr` → hit Enter
2. Go to **Windows Logs** → **Application**
3. Filter by Source: `cloudflared`

**Or view logs via PowerShell:**

```powershell
Get-EventLog -LogName Application -Source cloudflared -Newest 50
```

**Uninstall the service:**

```powershell
cloudflared service uninstall
```

---

## Multiple Local Services on One Tunnel

Expose multiple apps on different subdomains through a single tunnel.

Edit your `config.yml`:

```yaml
tunnel: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
credentials-file: C:\Users\YOUR_USERNAME\.cloudflared\xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.json

ingress:
  # React / Next.js frontend
  - hostname: app.yourdomain.com
    service: http://localhost:3000

  # Node.js / FastAPI backend
  - hostname: api.yourdomain.com
    service: http://localhost:8000

  # pgAdmin / DB admin panel
  - hostname: db.yourdomain.com
    service: http://localhost:5050

  # Vite dev server
  - hostname: dev.yourdomain.com
    service: http://localhost:5173

  # Required catch-all (always last)
  - service: http_status:404
```

Then add DNS routes for each:

```powershell
cloudflared tunnel route dns my-app-tunnel app.yourdomain.com
cloudflared tunnel route dns my-app-tunnel api.yourdomain.com
cloudflared tunnel route dns my-app-tunnel db.yourdomain.com
cloudflared tunnel route dns my-app-tunnel dev.yourdomain.com
```

Run the tunnel:

```powershell
cloudflared tunnel run my-app-tunnel
```

All subdomains are now live! 🚀

---

## Useful Commands

```powershell
# Check version
cloudflared --version

# List all your tunnels
cloudflared tunnel list

# Get info about a specific tunnel
cloudflared tunnel info my-app-tunnel

# Validate your config.yml
cloudflared tunnel ingress validate

# Run tunnel manually
cloudflared tunnel run my-app-tunnel

# Delete a tunnel
cloudflared tunnel delete my-app-tunnel

# Install as Windows service
cloudflared service install

# Uninstall Windows service
cloudflared service uninstall

# Start/stop service via PowerShell
Start-Service cloudflared
Stop-Service cloudflared
Restart-Service cloudflared

# Quick random URL (no account)
cloudflared tunnel --url http://localhost:3000
```

---

## Troubleshooting on Windows

### ❌ `'cloudflared' is not recognized as an internal or external command`

The EXE is not in your PATH. Fix it:

```powershell
# Check if it's in System32
Test-Path "C:\Windows\System32\cloudflared.exe"

# Or add its folder to PATH permanently
[System.Environment]::SetEnvironmentVariable(
  "PATH",
  $env:PATH + ";C:\path\to\cloudflared-folder",
  "Machine"
)
```

Then **restart PowerShell**.

---

### ❌ `Access is denied` when installing service

You must run PowerShell **as Administrator**:

1. Press `Win + X`
2. Click **"Windows PowerShell (Admin)"** or **"Terminal (Admin)"**

---

### ❌ `failed to sufficiently increase receive buffer size`

This is a warning, not an error — your tunnel will still work. To fix it:

```powershell
# Run as Admin
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters" `
  -Name "DefaultReceiveWindow" -Value 16777216 -Type DWord
```

---

### ❌ Tunnel connects but website doesn't load

Make sure your local server is actually running:

```powershell
# Test if your app responds locally
Invoke-WebRequest -Uri http://localhost:3000
# or
curl http://localhost:3000
```

Also check Windows Firewall isn't blocking the local port:

```powershell
# Allow port 3000 inbound (if needed)
New-NetFirewallRule -DisplayName "Allow Port 3000" -Direction Inbound `
  -Protocol TCP -LocalPort 3000 -Action Allow
```

---

### ❌ `ERR_TOO_MANY_REDIRECTS` in browser

In Cloudflare Dashboard:
- Go to your domain → **SSL/TLS** → set mode to **"Full"** (not Flexible, not Full Strict)

---

### ❌ Config file not found

Ensure the file is at exactly this path (use PowerShell to check):

```powershell
Test-Path "$env:USERPROFILE\.cloudflared\config.yml"

# View contents
Get-Content "$env:USERPROFILE\.cloudflared\config.yml"
```

---

### ❌ WebSocket not working

Add `originRequest` settings to your ingress in `config.yml`:

```yaml
ingress:
  - hostname: app.yourdomain.com
    service: http://localhost:3000
    originRequest:
      connectTimeout: 30s
      tcpKeepAlive: 30s
      keepAliveConnections: 10
  - service: http_status:404
```

---

## Security Tips

| Tip | How |
|---|---|
| 🔐 Add login protection | Use **Cloudflare Access** (free for 50 users) at [one.dash.cloudflare.com](https://one.dash.cloudflare.com) |
| 🛡️ Keep credentials safe | Never share `.json` files from `C:\Users\YOU\.cloudflared\` |
| 🚫 Don't expose databases | Only tunnel your app's HTTP port, not DB ports |
| 🔄 Rotate secrets | `cloudflared tunnel rotate-secret my-app-tunnel` |
| 📋 Use WAF | Enable Cloudflare's Web Application Firewall in the dashboard |

---

## 📁 Key File Locations on Windows

| File | Location |
|---|---|
| cloudflared binary | `C:\Windows\System32\cloudflared.exe` |
| Auth certificate | `C:\Users\YOU\.cloudflared\cert.pem` |
| Tunnel credentials | `C:\Users\YOU\.cloudflared\<tunnel-id>.json` |
| Config file | `C:\Users\YOU\.cloudflared\config.yml` |

---

## Resources

- 📚 [Cloudflare Tunnel Docs](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
- 📦 [cloudflared Windows Releases](https://github.com/cloudflare/cloudflared/releases/latest)
- 🔐 [Cloudflare Access (Zero Trust)](https://developers.cloudflare.com/cloudflare-one/policies/access/)
- 🌐 [Cloudflare Dashboard](https://dash.cloudflare.com)

---

> **Made for Windows developers who just want their localhost online — fast.** 🪟⚡
