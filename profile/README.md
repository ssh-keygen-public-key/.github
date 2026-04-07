
If you just rented your first VPS and the setup wizard is asking for something called a "public key," relax. You're not alone. This confuses pretty much everyone the first time around.

Here's what's actually happening: instead of logging into your server with a plain password (which bots can brute-force in minutes), SSH key authentication works like a physical padlock. You keep the private key on your laptop — that's the actual key. The public key goes on the server — that's the lock. Only your private key can open that lock. No password ever travels across the internet.

This guide walks you through how to ssh generate public key on Linux, macOS, and Windows, explains what to do with it, and covers how to apply it to a real VPS setup. We'll also look at DMIT, a VPS provider that actually defaults to SSH key authentication — no passwords even offered at first login.

---

## What Is an SSH Key Pair, Really?

Think of it as two mathematically linked files:

- **Private key** (`id_ed25519` or `id_rsa`) — stays on your computer, never shared
- **Public key** (`id_ed25519.pub` or `id_rsa.pub`) — placed on the server, safe to share

When you try to connect, the server encrypts a challenge message using your public key. Only your private key can decrypt it. If it matches, you're in — no password transmitted, no brute-force possible.

Two common algorithms you'll encounter:

- **Ed25519** — modern, faster, shorter keys, recommended for Linux/macOS instances
- **RSA (4096-bit)** — older but very widely supported, works everywhere including Windows-based servers

---

## Step 1: Generate Your SSH Key Pair

### On Linux or macOS

Open a terminal. Run this one command:

bash
ssh-keygen -t ed25519 -C "your@email.com"


You'll see:


Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/you/.ssh/id_ed25519):


Press Enter to accept the default path. Then:


Enter passphrase (empty for no passphrase):


Adding a passphrase is optional but strongly recommended — it encrypts your private key locally. If someone steals your laptop, they still can't use the key without the passphrase.

Your keys are now at:
- `~/.ssh/id_ed25519` (private — keep this safe)
- `~/.ssh/id_ed25519.pub` (public — this goes to the server)

To view your public key, run:

bash
cat ~/.ssh/id_ed25519.pub


It'll look something like this:


ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... your@email.com


That long string starting with `ssh-ed25519` is your public key. Copy all of it.

### On Windows (OpenSSH — Windows 10/11)

Windows 10 version 1809 and later include OpenSSH built-in. Open **PowerShell** or **Command Prompt** and run:

powershell
ssh-keygen -t ed25519 -C "your@email.com"


Same prompts as above. Your keys end up at `C:\Users\YourName\.ssh\`. To view the public key:

powershell
type C:\Users\YourName\.ssh\id_ed25519.pub


### On Windows (PuTTYgen)

If you use PuTTY for SSH connections:

1. Download and install PuTTY (includes PuTTYgen)
2. Open **PuTTYgen**
3. Select **EdDSA** as key type at the bottom
4. Click **Generate** and move your mouse around the blank area to generate randomness
5. Once done, copy the text from the "Public key for pasting into OpenSSH authorized_keys file" box
6. Click **Save private key** — store this `.ppk` file somewhere safe
7. Also click **Conversions → Export OpenSSH key** to get the key in standard format for other tools

---

## Step 2: Put the Public Key on Your Server

Once you have your public key, it needs to land in the right place on the server: the `~/.ssh/authorized_keys` file of the user you're connecting as.

### Method 1: ssh-copy-id (easiest, Linux/macOS)

If your server still allows password login temporarily:

bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub username@your-server-ip


Enter your password once, and it handles everything automatically.

### Method 2: Manual Paste

Log into the server via password first, then:

bash
mkdir -p ~/.ssh && chmod 700 ~/.ssh
echo "paste-your-public-key-here" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys


**Permissions matter a lot here.** SSH will silently refuse to use the key if the permissions on `~/.ssh/` or `authorized_keys` are too open.

### Method 3: VPS Control Panel

Most VPS providers let you paste your public key during signup or in the dashboard. DMIT, for instance, integrates SSH key upload directly into the instance creation flow — paste your public key there, and it gets injected automatically when the server first boots.

---

## Step 3: Connect Using Your Key

Once the public key is on the server:

bash
ssh -i ~/.ssh/id_ed25519 username@your-server-ip


If you used the default filename (`id_ed25519`), SSH often finds it automatically:

bash
ssh username@your-server-ip


### Setting Up an SSH Config File (Saves Time)

Create or edit `~/.ssh/config`:


Host myserver
    HostName 203.0.113.100
    User root
    IdentityFile ~/.ssh/id_ed25519


Now you can connect with just:

bash
ssh myserver


---

## Step 4: Lock It Down — Disable Password Authentication

After confirming your key works, disable password login entirely. On the server:

bash
sudo nano /etc/ssh/sshd_config


Find and set:


PasswordAuthentication no
PermitRootLogin no


Reload SSH:

bash
sudo systemctl reload sshd


From this point on, only someone with your private key can connect. Brute-force attacks become pointless.

---

## Why DMIT VPS Makes SSH Keys Even More Relevant

Here's an interesting detail: DMIT doesn't give you a root password on first login. They default to SSH key authentication from day one. This is actually the right call — it forces a secure setup from the start instead of leaving it as an optional step most people skip.

👉 [View DMIT's latest VPS plans and pricing](https://www.dmit.io/aff.php?aff=13832)

DMIT has been running premium VPS hosting since 2018, operating data centers in Los Angeles, Hong Kong, and Tokyo. What sets them apart isn't just the hardware (AMD EPYC processors, enterprise SSD storage). It's the network routing — they specialize in CN2 GIA and CMIN2 connections, which are optimized pathways to mainland China that stay fast even during evening peak hours when regular international routes crawl.

For anyone setting up a server that needs to communicate with Asia-Pacific users, getting your SSH key setup right from day one — the way DMIT encourages — is both a security and a workflow win.

---

## DMIT VPS Plans — Full Comparison Table

All plans include KVM virtualization, AMD EPYC processors, enterprise SSD, 1 IPv4 + 1 IPv6/64, free IP replacement every 15 days, and a 3-day money-back guarantee.

### Los Angeles Eyeball Series (CMIN2 Optimized)

*Promo code: `LAX-EB-LAUNCH-NON-MONTHLY-RECURRING-20OFF` — 20% recurring discount on quarterly/annual billing*

| Plan | CPU | RAM | Storage | Bandwidth | Monthly Traffic | Price | Get It |
|------|-----|-----|---------|-----------|-----------------|-------|--------|
| LAX.EB.TINY | 1 Core | 2 GB | 20 GB SSD | 2 Gbps | 1.2 TB | ~$6.90/mo |  [Order Now](https://www.dmit.io/aff.php?aff=13832&pid=167) |
| LAX.EB.POCKET | 1 Core | 2 GB | 40 GB SSD | 4 Gbps | 2 TB | ~$12.90/mo |  [Order Now](https://www.dmit.io/aff.php?aff=13832&pid=168) |
| LAX.EB.STARTER | 2 Cores | 2 GB | 40 GB SSD | 4 Gbps | 2.4 TB | ~$16.90/mo |  [Order Now](https://www.dmit.io/aff.php?aff=13832&pid=169) |
| LAX.EB.MEDIUM | 2 Cores | 4 GB | 80 GB SSD | 8 Gbps | 4.5 TB | ~$29.90/mo |  [Order Now](https://www.dmit.io/aff.php?aff=13832&pid=170) |

### Los Angeles Premium Series (CN2 GIA — Top-Tier China Routing)

| Plan | CPU | RAM | Storage | Bandwidth | Monthly Traffic | Price | Get It |
|------|-----|-----|---------|-----------|-----------------|-------|--------|
| LAX.Pro.TINY | 1 Core | 2 GB | 20 GB SSD | 1 Gbps | 1 TB | $88.88/yr |  [Order Now](https://www.dmit.io/aff.php?aff=13832&pid=183) |
| LAX.Pro.POCKET | 2 Cores | 2 GB | 40 GB SSD | 4 Gbps | 1.5 TB | $159.98/yr |  [Order Now](https://www.dmit.io/aff.php?aff=13832&pid=184) |
| LAX.Pro.STARTER | 2 Cores | 2 GB | 80 GB SSD | 10 Gbps | 3 TB | $322.99/yr |  [Order Now](https://www.dmit.io/aff.php?aff=13832&pid=185) |

### Hong Kong Premium Series (CN2 GIA)

| Plan | CPU | RAM | Storage | Bandwidth | Monthly Traffic | Price | Get It |
|------|-----|-----|---------|-----------|-----------------|-------|--------|
| HKG.Pro.STARTER | 1 Core | 2 GB | 40 GB SSD | 300 Mbps | 500 GB | $298/yr |  [Order Now](https://www.dmit.io/aff.php?aff=13832&pid=135) |
| HKG.Pro.MEDIUM | 2 Cores | 4 GB | 80 GB SSD | 500 Mbps | 1 TB | ~$498/yr |  [Order Now](https://www.dmit.io/aff.php?aff=13832&pid=136) |

### Hong Kong Eyeball Series (CMI Routes)

| Plan | CPU | RAM | Storage | Bandwidth | Monthly Traffic | Price | Get It |
|------|-----|-----|---------|-----------|-----------------|-------|--------|
| HKG.EB.TINY | 1 Core | 1 GB | 20 GB SSD | 1 Gbps | 1 TB | $25.90/mo |  [Order Now](https://www.dmit.io/aff.php?aff=13832&pid=145) |
| HKG.EB.STARTER | 1 Core | 2 GB | 40 GB SSD | 2 Gbps | 2 TB | $55.90/mo |  [Order Now](https://www.dmit.io/aff.php?aff=13832&pid=146) |

### Hong Kong Tier 1 (Budget International Routing)

*Promo code: `HKG-T1-ANNUALLY-45OFF-RECUR` — 45% lifetime off on annual plans + upgraded specs*

| Plan | CPU | RAM | Storage | Bandwidth | Monthly Traffic | Price | Get It |
|------|-----|-----|---------|-----------|-----------------|-------|--------|
| HKG.T1.WEE | 1 Core | 0.5 GB | 10 GB SSD | 10 Gbps | 800 GB | $36.90/yr |  [Order Now](https://www.dmit.io/aff.php?aff=13832&pid=154) |
| HKG.T1.TINY | 1 Core | 1 GB | 20 GB SSD | 10 Gbps | 1 TB | $73.80/yr |  [Order Now](https://www.dmit.io/aff.php?aff=13832&pid=155) |

### Tokyo Premium Series (CN2 GIA)

| Plan | CPU | RAM | Storage | Bandwidth | Monthly Traffic | Price | Get It |
|------|-----|-----|---------|-----------|-----------------|-------|--------|
| TYO.Pro.TINY | 1 Core | 1 GB | 20 GB SSD | 1 Gbps | 500 GB | $262.80/yr |  [Order Now](https://www.dmit.io/aff.php?aff=13832&pid=195) |
| TYO.Pro.STARTER | 1 Core | 2 GB | 40 GB SSD | 1 Gbps | 1 TB | $478.80/yr |  [Order Now](https://www.dmit.io/aff.php?aff=13832&pid=196) |

### Tokyo Lite Series (CMI Direct)

| Plan | CPU | RAM | Storage | Bandwidth | Monthly Traffic | Price | Get It |
|------|-----|-----|---------|-----------|-----------------|-------|--------|
| TYO.Lite.STARTER | 1 Core | 2 GB | 40 GB SSD | 1 Gbps | 1 TB | $6.90/mo (annual) |  [Order Now](https://www.dmit.io/aff.php?aff=13832&pid=200) |

### Tokyo Tier 1 Series

*Promo code: `2025-TYO-T1-HI-GSL-NON-MONTHLY-30OFF` — 30% lifetime off quarterly/annual billing*

| Plan | CPU | RAM | Storage | Bandwidth | Monthly Traffic | Price | Get It |
|------|-----|-----|---------|-----------|-----------------|-------|--------|
| TYO.T1.STARTER | 1 Core | 1 GB | 20 GB SSD | Standard routing | 1 TB | Check site |  [Order Now](https://www.dmit.io/aff.php?aff=13832&pid=205) |

---

## Common Beginner Mistakes with SSH Keys

**Mistake 1: Sharing the private key.** The file that doesn't have `.pub` in the name never leaves your computer. Ever. If someone asks for your private key, something has gone wrong.

**Mistake 2: Wrong file permissions.** If `~/.ssh` is world-readable (`chmod 777`), SSH will refuse the key. The correct settings are `700` for the directory and `600` for the key files.

**Mistake 3: Generating a new key for every server.** This is actually fine for security (one key = one device), but many beginners think they need a different key for each server. You can use the same public key on as many servers as you want — just paste it into each server's `authorized_keys`.

**Mistake 4: Never setting a passphrase.** If your laptop gets stolen and there's no passphrase on the private key, whoever has your laptop has full access to every server with that public key installed. A passphrase adds local encryption to the private key file.

**Mistake 5: Not disabling password auth after setting up keys.** The whole point is to eliminate passwords from the equation. If you set up keys but leave password authentication enabled, bots can still attempt brute-force attacks.

---

## Quick Reference: Active DMIT Promo Codes

| Code | Discount | Applies To |
|------|----------|-----------|
| `LAX-EB-LAUNCH-NON-MONTHLY-RECURRING-20OFF` | 20% recurring | LA Eyeball (quarterly+) |
| `HKG-T1-ANNUALLY-45OFF-RECUR` | 45% recurring + spec upgrades | HK Tier 1 (annual) |
| `2025-TYO-T1-HI-GSL-NON-MONTHLY-30OFF` | 30% lifetime | Tokyo Tier 1 (quarterly+) |
| `2025-TYO-T1-HI-GSL-MONTHLY-10OFF` | 10% | Tokyo Tier 1 (monthly) |
| `SJC-Unmetered-Annually-30OFF` | 30% | San Jose unmetered (annual) |
| `7L8O3PQTHNXCFS2TXPLP` | 5% | Select plans (non-monthly) |

---

## Wrapping Up

Generating an SSH public key takes about 30 seconds once you know the command. The bigger picture is what it unlocks: a genuinely secure login method that eliminates the entire category of brute-force password attacks.

The flow is simple: run `ssh-keygen`, copy your `.pub` file to the server's `authorized_keys`, verify it works, then turn off password authentication. That's it.

DMIT builds this expectation in from the start — their VPS setup requires SSH key authentication by default, which is the kind of sensible baseline that saves new server owners from themselves. If you're spinning up a VPS for the first time and want hardware that takes security as seriously as your fresh SSH setup does, their LA Eyeball series is a solid starting point at reasonable pricing.

👉 [Check out DMIT's current VPS plans and promotions](https://www.dmit.io/aff.php?aff=13832)
