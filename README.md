# 🔐 PassWall2 on OpenWrt 24.x — Installation Guide

> **Clean, error-free method — fully up to date**  
> 📺 [Watch the full video tutorial on YouTube](https://www.youtube.com/watch?v=uxQW2Dr0qJg)

---

## ⚠️ Before You Start — Important Check

Make sure the official OpenWrt site loads on your internet connection:

🔗 [https://openwrt.org](https://openwrt.org)

> If this site is **blocked**, packages won't download and installation will fail.  
> You'll need an unrestricted internet connection behind your router before proceeding.

---

## 📋 Prerequisites

- Router running **OpenWrt 24.x.x** (latest: 24.10.x)
- SSH access to your router
- A Windows computer on the same network
- `.ipk` files downloaded from the PassWall2 releases page

> ⚠️ Version 24 uses the **`opkg`** package manager and **`.ipk`** format.  
> For OpenWrt 25.x (which uses `apk` and `.apk`), see [this guide](https://www.youtube.com/watch?v=mtQ-Vtan1yk).

---

## 🚀 Installation Steps

### Step 1 — Find Your Router IP

Search for `cmd` in Windows, open **Command Prompt as Administrator**, then run:

```bash
ipconfig
```

> Under **Ethernet adapter Ethernet**, find the **Default Gateway** — that's your router's IP. Copy it and keep it handy.

---

### Step 2 — (Optional but Recommended) Change Router IP

If your router's default IP is `192.168.1.1` and conflicts with your modem:

1. Open the router IP in your browser and log into LuCI
2. Go to **Network** → **Interfaces**
3. Click **Edit** next to **LAN**
4. Change the third octet of the **IPv4 address** (e.g. `192.168.10.1`)
5. Click **Save & Apply** — wait 80–90 seconds
6. Run `ipconfig` again to get the new IP

---

### Step 3 — Connect via SSH

```bash
ssh root@192.168.x.x
```

> If this is your first time connecting, type `yes` and press Enter.  
> When typing your password, no characters will appear — this is normal.

---

### Step 4 — Check Your CPU Architecture

```bash
opkg print-architecture
```

> On the **third line** of the output, look for the value next to **arch** — that's your CPU architecture code.  
> Copy it exactly — you'll need it in the next step.

---

### Step 5 — Download PassWall2 Files

Go to the official releases page:

🔗 **[github.com/Openwrt-Passwall/openwrt-passwall2/releases](https://github.com/Openwrt-Passwall/openwrt-passwall2/releases)**

**You need two types of files:**

**1. The main PassWall2 file** (same for all routers):
- Find the file with a **`.ipk`** extension
- This file is not architecture-specific

**2. The dependency package** for your specific CPU architecture:
- Press **Ctrl+F** and search for the architecture code from Step 4
- From the results, download the file with the **`.ipk`** extension (not `.apk` — that's for v25)
- **Extract** the downloaded zip file

> 💡 Always download the **Latest** release — PassWall2 is updated frequently.

**Create a folder on your Desktop** (e.g. `passwall`) and place all `.ipk` files inside it.

---

### Step 6 — Enable Repositories & Install Dependencies

SSH into your router and run these commands **in order**:

```bash
sed -i 's/# src/src/g' /etc/opkg/distfeeds.conf
```

```bash
opkg update
```

```bash
opkg remove dnsmasq
```

```bash
opkg install dnsmasq-full
```

```bash
opkg update
```

```bash
opkg install ip-full ipset iptables iptables-nft iptables-mod-tproxy kmod-ipt-tproxy kmod-nft-tproxy kmod-nft-socket kmod-nft-nat coreutils coreutils-base64 coreutils-nohup curl unzip lua liblua5.1.5 libuci-lua
```

> ⏳ Wait for all packages to finish downloading and installing.  
> If you get errors, the OpenWrt site is likely blocked on your connection (see the check at the top).

---

### Step 7 — Transfer `.ipk` Files to the Router

In Windows, click on the address bar of your `passwall` folder, type `powershell`, and press Enter — this opens PowerShell directly in that folder path.

Then run (replace the IP with your router's IP):

```bash
scp -O *.ipk root@192.168.x.x:/tmp/
```

> Enter your router password when prompted. Files will transfer one by one.

---

### Step 8 — Install PassWall2 on the Router

SSH back into the router:

```bash
ssh root@192.168.x.x
```

```bash
cd /tmp
```

```bash
opkg install *.ipk
```

> ⏳ Wait for installation to complete. Some dependencies may auto-download during this step — that's normal. You may see around 34 packages installed even if you only transferred 17 files.

---

### Step 9 — Restart Services & Reboot

```bash
opkg update
```

```bash
/etc/init.d/rpcd restart
```

```bash
/etc/init.d/uhttpd restart
```

```bash
/etc/init.d/passwall2 restart
```

```bash
reboot
```

> ⏳ Wait for the router to fully reboot and come back online.

---

### Step 10 — Verify Successful Installation

After the reboot, open your router IP in the browser and log into LuCI.

✅ If installation was successful, a new **Services** tab appears with **PassWall2** underneath it.

---

## 🧭 Quick Reference

| # | Action | Command |
|---|--------|---------|
| 1 | Find router IP | `ipconfig` |
| 2 | Connect via SSH | `ssh root@192.168.x.x` |
| 3 | Check CPU arch | `opkg print-architecture` |
| 4 | Enable repos | `sed -i 's/# src/src/g' /etc/opkg/distfeeds.conf` |
| 5 | Update packages | `opkg update` |
| 6 | Remove default dnsmasq | `opkg remove dnsmasq` |
| 7 | Install full dnsmasq | `opkg install dnsmasq-full` |
| 8 | Install dependencies | `opkg install ip-full ipset iptables ...` |
| 9 | Transfer files | `scp -O *.ipk root@192.168.x.x:/tmp/` |
| 10 | Install PassWall2 | `cd /tmp` → `opkg install *.ipk` |
| 11 | Restart services | `rpcd` → `uhttpd` → `passwall2 restart` |
| 12 | Reboot | `reboot` |

---

## 🔄 Updating PassWall2 Later

No commands needed! From LuCI:

**Services** → **PassWall2** → **App Update**

Click **Check Update** — install if a new version is available.

---

## ⚠️ Troubleshooting

- **`opkg update` fails** — Check if `openwrt.org` opens in your browser. If not, your internet is filtered.
- **dnsmasq conflict** — Always run `opkg remove dnsmasq` before `opkg install dnsmasq-full`.
- **`scp` not working** — Make sure PowerShell was opened inside the `passwall` folder, not from another path.
- **PassWall2 missing from LuCI** — Re-run `rpcd restart` and `uhttpd restart`, then hard-refresh your browser (Ctrl+Shift+R).
- **Wrong `.ipk` file** — Re-check your architecture with `opkg print-architecture` and search for that exact code on the releases page.

---

## 🔗 Resources

- 📦 [PassWall2 Releases](https://github.com/Openwrt-Passwall/openwrt-passwall2/releases)
- 📺 [Video Tutorial (v24)](https://www.youtube.com/watch?v=KOcdWkXvJ4I)
- 📺 [Video Tutorial (v25)](https://www.youtube.com/watch?v=mtQ-Vtan1yk)
- 🌐 [OpenWrt Official Site](https://openwrt.org)

---

> 💡 *If this guide helped you, consider giving the repo a ⭐ and sharing the video!*
