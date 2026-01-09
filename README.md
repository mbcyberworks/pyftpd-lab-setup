# Quick FTP Server for Penetration Testing Labs (pyftpdlib)

A minimal FTP file transfer setup for penetration testing labs.  
Tested with **Kali Linux** in local labs and **TryHackMe / HTB VPN** environments.

This setup is designed for **reliable file transfer** when upload capability is required during labs or exam-style scenarios.

---

## 🎯 Use case

- Upload files from targets to Kali when upload capability is required
- Works for:
  - Local labs (virt-manager / NAT)
  - TryHackMe / HTB targets via VPN (`tun0`)
- No system-wide Python installs
- No PEP 668 issues
- No reliance on VM shared folders

---

## 🧠 Design choices

- `pyftpdlib` is used **only** when FTP upload capability is required
- Installed inside a dedicated Python virtual environment
- Started via a shell alias (`ftpstart`)
- Server listens on `0.0.0.0` (all interfaces)
- Alias prints both **LAN** and **VPN** IPs for clarity
- Authenticated access only (no anonymous login)

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
  echo

  if [[ -n "$LANIP" ]]; then
    echo "    Windows ftp.exe (LAN):"
    echo "      ftp"
    echo "      open $LANIP $PORT"
    echo "      user: $USER"
    echo "      pass: $PASS"
    echo "      binary"
  fi

  if [[ -n "$TUNIP" ]]; then
    echo
    echo "    Windows ftp.exe (VPN):"
    echo "      ftp"
    echo "      open $TUNIP $PORT"
    echo "      user: $USER"
    echo "      pass: $PASS"
    echo "      binary"
  fi

  echo
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

### Defaults

* Port: `2121`
* User: `test`
* Password: `Test123!`
* Directory: current working directory (`pwd`)

---

### Custom parameters (port / user / password)

Change directory **before** starting the server:

```bash
cd ~/loot
ftpstart 2121 test S3cr3t!
```

Format:

```text
ftpstart <PORT> <USER> <PASS>
```

---

## 🔌 Which IP should be used?

### Local Windows VM (virt-manager / NAT)

Use the **LAN IP**:

```text
192.168.122.x
```

### TryHackMe / HTB target

Use the **VPN IP** (`tun0`):

```text
10.x.x.x
192.168.x.x
```

The server listens on **all interfaces** (`0.0.0.0`).
The client must connect to an IP it can actually reach.

---

## 🔗 FTP URL notes

The script prints URLs such as:

```text
ftp://test:Test123!@192.168.134.168:2121/
```

These URLs are intended for:

* GUI clients (FileZilla, WinSCP)
* Tools like `curl` or `wget`
* Notes and reporting during labs

⚠️ **Important:**
Windows built-in `ftp.exe` does **not** support FTP URL syntax.

---

## 🪟 Windows client notes (`ftp.exe`)

Use the step-by-step commands printed by the script.

Example:

```text
ftp
open <KALI_IP> 2121
user: test
pass: Test123!
binary
```

Always use `binary` when transferring `.exe` or `.dll` files.
Failure to do so may corrupt binaries during transfer.

---

## ⚠️ FTP passive mode note

If login works but `ls`, `get`, or `put` hangs:

* This is usually caused by **FTP passive mode + NAT/VPN**

Temporary fix:

```bash
python -m pyftpdlib -p 2121 -u test -P Test123! -w --passive-ports 30000-30010
```

Only required if you encounter this issue.

---

## 🧠 Penetration testing mindset

* Use FTP **only** when upload capability is required
* Avoid shared folders and GUI copy/paste shortcuts
* Demonstrates understanding of:

  * Interfaces vs routing
  * VPN reachability
  * Reliable file transfer methods
  * Tool limitations (URL vs client syntax)

---

## ✅ Summary

* Install once (virtual environment)
* Start instantly (`ftpstart`)
* Works for local labs and VPN-based targets
* Clean, repeatable, and lab-safe
* Designed for PNPT-style thinking

