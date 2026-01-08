# Quick FTP Server for Penetration Testing Labs (pyftpdlib)

A minimal FTP file transfer setup for penetration testing labs.  
Tested with **Kali Linux** in local labs and **TryHackMe VPN** environments.

---

## 🎯 Use case

- Upload and download files **from targets to Kali**
- Works for:
  - Local labs (virt-manager / NAT)
  - TryHackMe / HTB targets via VPN (`tun0`)
- No system-wide Python installs
- No PEP 668 issues

---

## 🧠 Design choices

- **pyftpdlib** is used only when FTP upload capability is required
- Installed in a **Python virtual environment**
- Started via a **shell alias (`ftpstart`)**
- Server listens on `0.0.0.0` (all interfaces)
- Alias prints **both LAN and VPN IPs** for clarity

---

## 📦 One-time setup (install once)

### 1. Create Python virtual environment
```bash
python3 -m venv ~/venvs/ftp
source ~/venvs/ftp/bin/activate
pip install pyftpdlib
deactivate
````

This is done **once**.
You do **not** reinstall this for every lab.

---

### 2. Add alias to `~/.zshrc`

```bash
nano ~/.zshrc
```

Add the following block:

```bash
# ===============================
# Quick FTP Server (pyftpdlib)
# ===============================
ftpstart() {
  local PORT="${1:-2121}"
  local USER="${2:-test}"
  local PASS="${3:-Test123!}"
  local VENV="$HOME/venvs/ftp"

  if [[ ! -d "$VENV" ]]; then
    echo "[!] Python venv not found: $VENV"
    return 1
  fi

  source "$VENV/bin/activate" || return 1

  local LANIP TUNIP
  LANIP="$(hostname -I | awk '{print $1}')"
  TUNIP="$(ip -4 -o addr show dev tun0 2>/dev/null | awk '{print $4}' | cut -d/ -f1)"

  echo "[+] FTP server starting"
  echo "    Directory : $(pwd)"
  [[ -n "$LANIP" ]] && echo "    LAN       : ftp://$USER:$PASS@$LANIP:$PORT/"
  [[ -n "$TUNIP" ]] && echo "    VPN       : ftp://$USER:$PASS@$TUNIP:$PORT/"
  echo "    Stop with : CTRL+C"
  echo

  python -m pyftpdlib -p "$PORT" -u "$USER" -P "$PASS" -w
}

```

Reload your shell:

```bash
source ~/.zshrc
```

---

## 🚀 Usage

### Start FTP server (default)

```bash
ftpstart
```

Default values:

* Port: `2121`
* User: `test`
* Password: `Test123!`
* Directory: `~/loot`

---

### Custom parameters

```bash
ftpstart 2121 mau S3cr3t! ~/loot
```

---

## 🔌 Which IP should be used?

### Local Windows VM (virt-manager / NAT)

Use the **LAN IP**:

```
192.168.122.x
```

### TryHackMe / HTB target

Use the **VPN IP (tun0)**:

```
192.168.x.x   or   10.x.x.x
```

The server listens on all interfaces (`0.0.0.0`).
**The client must connect to an IP it can reach.**

---

## ⚠️ FTP passive mode note

If login works but `ls`, `get`, or `put` hangs:

* This is usually caused by **FTP passive mode + NAT/VPN**

Fix:

```bash
python -m pyftpdlib -p 2121 -u test -P Test123! -w --passive-ports 30000-30010
```

(Only required if you encounter this issue.)

---

## 🧠 Penetration testing mindset

* Use FTP only when **upload capability** is required
* Avoid shared folders and GUI copy/paste
* Demonstrate understanding of:

  * Interfaces vs routing
  * VPN reachability
  * Reliable file transfer methods

---

## ✅ Summary

* Install once (venv)
* Start instantly (`ftpstart`)
* Works for local labs and VPN-based targets
* Clean, repeatable, and lab-safe
