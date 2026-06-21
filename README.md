# 🌐 Cloudflare Tunnel — Expose Localhost to the World (Free & Secure)

> Access your `localhost` from anywhere in the world using **Cloudflare Tunnel** — no port forwarding, no firewall changes, no static IP required.

---

## 📖 Table of Contents

- [What is Cloudflare Tunnel?](#what-is-cloudflare-tunnel)
- [How It Works](#how-it-works)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
  - [Linux / macOS](#linux--macos)
  - [Windows](#windows)
- [Quick Start — Random URL (No Account Needed)](#quick-start--random-url-no-account-needed)
- [Setup with Custom Domain (Cloudflare Account Required)](#setup-with-custom-domain-cloudflare-account-required)
  - [Step 1 — Add Domain to Cloudflare](#step-1--add-domain-to-cloudflare)
  - [Step 2 — Login with cloudflared](#step-2--login-with-cloudflared)
  - [Step 3 — Create the Tunnel](#step-3--create-the-tunnel)
  - [Step 4 — Configure the Tunnel](#step-4--configure-the-tunnel)
  - [Step 5 — Route DNS to the Tunnel](#step-5--route-dns-to-the-tunnel)
  - [Step 6 — Run the Tunnel](#step-6--run-the-tunnel)
- [Run Tunnel as a Background Service](#run-tunnel-as-a-background-service)
- [Multiple Services on One Tunnel](#multiple-services-on-one-tunnel)
- [Useful Commands](#useful-commands)
- [Troubleshooting](#troubleshooting)
- [Security Tips](#security-tips)
- [How It Compares](#how-it-compares)
- [Resources](#resources)

---

## What is Cloudflare Tunnel?

**Cloudflare Tunnel** (formerly *Argo Tunnel*) is a free service by Cloudflare that creates a secure, outbound-only connection from your machine to Cloudflare's global edge network.

Instead of:
- Opening ports on your router 🔓
- Setting up a VPN 🔐
- Getting a static IP 💸

You simply run one command and your `localhost:3000` (or any port) becomes accessible at a public URL like:

```
https://my-app.yourdomain.com
https://random-name.trycloudflare.com   ← no account needed!
```

---

## How It Works

```
Your Machine                  Cloudflare Edge              Internet
┌─────────────────┐          ┌──────────────────┐         ┌────────────┐
│  localhost:3000 │◄────────►│  cloudflared     │◄───────►│   Anyone   │
│                 │  tunnel  │  (your daemon)   │  HTTPS  │  anywhere  │
└─────────────────┘          └──────────────────┘         └────────────┘
```

- `cloudflared` runs locally and establishes an **outbound** connection to Cloudflare
- All traffic is **encrypted end-to-end**
- No inbound ports need to be opened on your machine or router
- Cloudflare handles SSL/TLS automatically

---

## Prerequisites

| Requirement | Details |
|---|---|
| OS | Linux, macOS, or Windows |
| Cloudflare Account | Free — only for custom domain setup |
| Domain Name | Required only for custom domain setup |
| Domain added to Cloudflare | Nameservers must point to Cloudflare |

> **No account needed** if you just want a quick random URL!

---

## Installation

### Linux / macOS

**Option A — Using the install script (recommended)**

```bash
# Linux (amd64)
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 \
  -o /usr/local/bin/cloudflared
chmod +x /usr/local/bin/cloudflared
cloudflared --version
```

```bash
# macOS (using Homebrew)
brew install cloudflared
cloudflared --version
```

**Option B — Using package manager (Debian / Ubuntu)**

```bash
# Add Cloudflare's repo
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | \
  sudo tee /usr/share/keyrings/cloudflare-main.gpg > /dev/null

echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] \
  https://pkg.cloudflare.com/cloudflared focal main' | \
  sudo tee /etc/apt/sources.list.d/cloudflared.list

sudo apt-get update && sudo apt-get install cloudflared
```

### Windows

1. Download the latest `.exe` from the [releases page](https://github.com/cloudflare/cloudflared/releases/latest)
2. Rename it to `cloudflared.exe` and place it somewhere in your `PATH` (e.g., `C:\Windows\System32`)
3. Open **PowerShell** or **CMD** and verify:

```powershell
cloudflared --version
```

---

## Quick Start — Random URL (No Account Needed)

This is the **fastest way** to expose localhost — perfect for demos, testing, and sharing with teammates.

```bash
# Start your local server first (example: Node.js on port 3000)
npm run dev   # or whatever starts your app

# Then in a new terminal:
cloudflared tunnel --url http://localhost:3000
```

You'll see output like:

```
2024-01-15T10:23:45Z INF +--------------------------------------------------------------------------------------------+
2024-01-15T10:23:45Z INF |  Your quick Tunnel has been created! Visit it at (it may take some time to be reachable):  |
2024-01-15T10:23:45Z INF |  https://orange-sunset-bears-developer.trycloudflare.com                                    |
2024-01-15T10:23:45Z INF +--------------------------------------------------------------------------------------------+
```

✅ That's it! Share that URL with anyone — it's live immediately.

> **Note:** The random URL changes every time you restart `cloudflared`. Use a custom domain for a permanent URL.

---

## Setup with Custom Domain (Cloudflare Account Required)

This gives you a **permanent, branded URL** like `https://api.yourdomain.com`.

### Step 1 — Add Domain to Cloudflare

1. Log in to [dash.cloudflare.com](https://dash.cloudflare.com)
2. Click **"Add a Site"** and enter your domain
3. Select the **Free plan**
4. Update your domain's **nameservers** at your registrar to the ones Cloudflare provides
5. Wait for DNS propagation (usually a few minutes to a few hours)

> If you already use Cloudflare for your domain, skip this step.

---

### Step 2 — Login with cloudflared

```bash
cloudflared tunnel login
```

This will:
- Open a browser window at Cloudflare
- Ask you to authorize `cloudflared` for your account
- Download a certificate to `~/.cloudflared/cert.pem`

---

### Step 3 — Create the Tunnel

```bash
cloudflared tunnel create my-app-tunnel
```

Replace `my-app-tunnel` with any name you like.

You'll see output like:

```
Tunnel credentials written to /home/user/.cloudflared/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.json
Created tunnel my-app-tunnel with id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

📌 **Save the tunnel ID** — you'll need it in the next step.

---

### Step 4 — Configure the Tunnel

Create the configuration file at `~/.cloudflared/config.yml`:

```bash
nano ~/.cloudflared/config.yml
```

Paste the following (replace values accordingly):

```yaml
tunnel: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx   # Your tunnel ID from Step 3
credentials-file: /home/YOUR_USERNAME/.cloudflared/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.json

ingress:
  - hostname: app.yourdomain.com      # The public URL you want
    service: http://localhost:3000    # Your local server
  - service: http_status:404          # Catch-all (required)
```

**Example for multiple services:**

```yaml
tunnel: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
credentials-file: /home/user/.cloudflared/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.json

ingress:
  - hostname: app.yourdomain.com
    service: http://localhost:3000    # Frontend

  - hostname: api.yourdomain.com
    service: http://localhost:8000    # Backend API

  - hostname: pgadmin.yourdomain.com
    service: http://localhost:5050    # Database Admin

  - service: http_status:404
```

---

### Step 5 — Route DNS to the Tunnel

```bash
cloudflared tunnel route dns my-app-tunnel app.yourdomain.com
```

This automatically creates a **CNAME DNS record** in your Cloudflare dashboard pointing `app.yourdomain.com` → your tunnel.

> Repeat this for each hostname you added in `config.yml`:
> ```bash
> cloudflared tunnel route dns my-app-tunnel api.yourdomain.com
> cloudflared tunnel route dns my-app-tunnel pgadmin.yourdomain.com
> ```

---

### Step 6 — Run the Tunnel

```bash
cloudflared tunnel run my-app-tunnel
```

Or using the config file explicitly:

```bash
cloudflared tunnel --config ~/.cloudflared/config.yml run
```

Your app is now live at `https://app.yourdomain.com` 🎉

---

## Run Tunnel as a Background Service

Don't want to keep a terminal open? Install it as a **system service** that starts automatically on boot.

### Linux (systemd)

```bash
# Install as a service
sudo cloudflared --config /home/YOUR_USER/.cloudflared/config.yml service install

# Enable and start
sudo systemctl enable cloudflared
sudo systemctl start cloudflared

# Check status
sudo systemctl status cloudflared

# View logs
sudo journalctl -u cloudflared -f
```

### macOS (launchd)

```bash
sudo cloudflared service install
sudo launchctl start com.cloudflare.cloudflared
```

### Windows (Service)

```powershell
# Run as Administrator
cloudflared service install
```

Then go to **Services** in Windows and start `Cloudflare Tunnel`.

---

## Multiple Services on One Tunnel

You can expose multiple local ports through **one tunnel** using the `ingress` rules in `config.yml`:

```yaml
tunnel: your-tunnel-id
credentials-file: /home/user/.cloudflared/your-tunnel-id.json

ingress:
  # React frontend
  - hostname: app.yourdomain.com
    service: http://localhost:3000

  # FastAPI / Express backend
  - hostname: api.yourdomain.com
    service: http://localhost:8000

  # WebSocket server
  - hostname: ws.yourdomain.com
    service: http://localhost:4000

  # SSH access (bonus!)
  - hostname: ssh.yourdomain.com
    service: ssh://localhost:22

  # Required catch-all
  - service: http_status:404
```

> **SSH via Cloudflare Tunnel** works too! Access your machine securely without exposing port 22 to the internet.

---

## Useful Commands

```bash
# List all your tunnels
cloudflared tunnel list

# Check tunnel status
cloudflared tunnel info my-app-tunnel

# Delete a tunnel
cloudflared tunnel delete my-app-tunnel

# Clean up DNS routes before deleting
cloudflared tunnel route dns --overwrite-dns my-app-tunnel app.yourdomain.com

# Test your config file for errors
cloudflared tunnel ingress validate

# Run tunnel in background (quick)
cloudflared tunnel run my-app-tunnel &

# View cloudflared version
cloudflared --version
```

---

## Troubleshooting

### ❌ `failed to authenticate` or `cert.pem not found`

```bash
# Re-run login
cloudflared tunnel login
```

---

### ❌ `ERR_NAME_NOT_RESOLVED` or DNS not working

- Check that the CNAME record was created in your Cloudflare DNS dashboard
- Wait a few minutes for propagation
- Verify with: `dig app.yourdomain.com` or `nslookup app.yourdomain.com`

---

### ❌ `connection refused` — tunnel connects but app doesn't load

Make sure your local server is actually running:

```bash
curl http://localhost:3000
```

Also check that the port in `config.yml` matches your app's port.

---

### ❌ WebSocket connections not working

Add this to your `config.yml` ingress rule:

```yaml
ingress:
  - hostname: app.yourdomain.com
    service: http://localhost:3000
    originRequest:
      noTLSVerify: true
      connectTimeout: 30s
      tcpKeepAlive: 30s
      keepAliveConnections: 10
```

---

### ❌ Tunnel starts but shows `ERR_TOO_MANY_REDIRECTS`

In your Cloudflare dashboard:
- Go to **SSL/TLS** → set to **"Full"** (not Full Strict, not Flexible)

---

## Security Tips

| Tip | Why |
|---|---|
| 🔒 Enable **Cloudflare Access** | Add authentication (Google, GitHub login) in front of your tunnel |
| 🚫 Don't expose sensitive services publicly | Use tunnels for dev/staging, not production databases |
| 🔑 Protect your `.json` credentials file | It grants full control of your tunnel |
| 🛡️ Enable **WAF rules** in Cloudflare dashboard | Block malicious traffic before it reaches your app |
| 📋 Rotate tunnel credentials periodically | `cloudflared tunnel rotate-secret my-app-tunnel` |

### Adding Cloudflare Access (Optional but Recommended)

Protect your tunnel with a login page — free for up to 50 users:

1. Go to [one.dash.cloudflare.com](https://one.dash.cloudflare.com)
2. Navigate to **Access** → **Applications**
3. Click **"Add an Application"** → **"Self-hosted"**
4. Enter your hostname (e.g., `app.yourdomain.com`)
5. Set up an **Identity Provider** (Google, GitHub, Email OTP, etc.)

Now only authorized users can reach your tunnel! 🔐

---

## How It Compares

| Feature | Cloudflare Tunnel | ngrok (free) | localtunnel |
|---|---|---|---|
| **Price** | Free | Free (limited) | Free |
| **Custom Domain** | ✅ Yes | ❌ Paid only | ✅ Yes |
| **Random URL** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Persistent URL** | ✅ Yes (custom domain) | ❌ Changes on restart | ❌ Changes |
| **Auth / Access Control** | ✅ Built-in | ❌ Paid | ❌ No |
| **Multiple ports** | ✅ One tunnel | ❌ One per session (free) | ✅ Yes |
| **Speed** | 🚀 Cloudflare CDN | ⚡ Good | 🐢 Variable |
| **Run as service** | ✅ Yes | ✅ Paid | ❌ No |
| **Self-host option** | ❌ | ❌ | ✅ |

> **Winner for most use cases: Cloudflare Tunnel** — especially if you own a domain.

---

## Resources

- 📚 [Official Cloudflare Tunnel Docs](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
- 🐙 [cloudflared GitHub Repository](https://github.com/cloudflare/cloudflared)
- 📦 [cloudflared Releases](https://github.com/cloudflare/cloudflared/releases/latest)
- 🔐 [Cloudflare Access — Zero Trust](https://developers.cloudflare.com/cloudflare-one/policies/access/)
- 🌐 [Cloudflare Dashboard](https://dash.cloudflare.com)
- 📖 [Cloudflare Zero Trust Dashboard](https://one.dash.cloudflare.com)

---

## 📝 License

This guide is open-source and free to use. Feel free to share, fork, or contribute!

---

> **Made with ❤️ for developers who hate port-forwarding.**  
> *Star ⭐ this repo if it helped you!*
