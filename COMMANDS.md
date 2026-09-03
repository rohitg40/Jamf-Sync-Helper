# 🛠️ Jamf Sync Helper — Button → Command Reference

This document provides a reference for the commands executed by each **Jamf Sync Helper** button.

### 🔑 Execution Types

* **`root`** 🔐 — Runs with administrator privileges using the password provided during **Unlock**.
* **`user`** 👤 — Runs as the currently logged-in user.
* **`guard`** 🛡️ — Every `jamf` command is checked against the available Jamf binary paths before execution.

### 📍 Jamf Binary

The application checks both Jamf binary locations:

```text
/usr/local/bin/jamf
/usr/local/jamf/bin/jamf
```

The binary is resolved automatically, so a missing `/usr/local/bin/jamf` symlink alone will not cause the command to fail.

> 💡 Full binary paths are used because `sudo` can reset the `PATH` environment variable.

---

## 🎛️ App Controls

These controls are part of the application and are **not troubleshooting commands**.

| Control                    | Command / Action                                             |
| -------------------------- | ------------------------------------------------------------ |
| 🔓 **Unlock**              | `sudo -k -S -p '' -v` — validates the administrator password |
| 🔒 **Lock / Auto-lock**    | Clears the password from application memory                  |
| ⏹️ **Stop**                | Sends `SIGTERM` to the running command                       |
| 📋 **Copy Output / Clear** | Clipboard / UI operation only                                |

---

## 🚀 Hero Button

| Button                     | Runs As | Command                                                                                                                                                                                |
| -------------------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 🚀 **Jamf Sync — Fix All** | `root`  | `$JAMF checkJSSConnection -retry 3 && { /usr/bin/killall jamf 2>/dev/null; sleep 2; $JAMF manage && $JAMF policy && $JAMF recon && echo '=== Jamf Sync completed successfully ==='; }` |

---

## 🔄 Sync & Inventory

| Button                              | Runs As | Command                             |
| ----------------------------------- | ------- | ----------------------------------- |
| 🔄 **Check In & Run Policies**      | `root`  | `$JAMF policy`                      |
| 📊 **Update Inventory (Recon)**     | `root`  | `$JAMF recon`                       |
| 🌐 **Check Jamf Server Connection** | `root`  | `$JAMF checkJSSConnection -retry 3` |

---

## 📱 Enrollment & Push

| Button                            | Runs As | Command                                                                                                                                                                                                                                                                                   |
| --------------------------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 📱 **MDM Enrollment Status**      | `user`  | `/usr/bin/profiles status -type enrollment`                                                                                                                                                                                                                                               |
| 🔍 **Check ADE (DEP) Assignment** | `root`  | `/usr/bin/profiles show -type enrollment`                                                                                                                                                                                                                                                 |
| 🔄 **Renew MDM Profile** ⚠️       | `user`  | `/usr/bin/profiles renew -type enrollment`                                                                                                                                                                                                                                                |
| 📋 **List Installed Profiles**    | `root`  | `/usr/bin/profiles list`                                                                                                                                                                                                                                                                  |
| 🍎 **Check Apple Push (APNs)**    | `user`  | `/usr/bin/nc -z -w 5 1-courier.push.apple.com 5223 && echo 'APNs reachable on port 5223 ✓' \|\| { /usr/bin/nc -z -w 5 1-courier.push.apple.com 443 && echo 'APNs reachable on port 443 (fallback) ✓' \|\| echo 'APNs NOT reachable — push/MDM commands may be blocked by the network'; }` |
| 🔄 **Restart Apple Push Daemon**  | `root`  | `/usr/bin/killall apsd 2>/dev/null; echo 'Push daemon restarted (relaunches automatically).'`                                                                                                                                                                                             |

---

## 🛠️ Fix Issues

| Button                                | Runs As | Command                                                                                                                                                                                                                           |
| ------------------------------------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 🔄 **Restart Jamf Binary**            | `root`  | `/usr/bin/killall jamf 2>/dev/null; sleep 3; $JAMF policy`                                                                                                                                                                        |
| 🔧 **Repair Management Framework** ⚠️ | `root`  | `$JAMF manage`                                                                                                                                                                                                                    |
| 🧹 **Flush Jamf Caches**              | `root`  | `$JAMF flushCaches`                                                                                                                                                                                                               |
| 🔐 **Trust Jamf Server Certificate**  | `root`  | `$JAMF trustJSS`                                                                                                                                                                                                                  |
| 🧹 **Clear Self Service Cache** ⚠️    | `user`  | `/usr/bin/killall 'Self Service' 2>/dev/null; sleep 1; /bin/rm -rf "$HOME/Library/Caches/com.jamfsoftware.selfservice.mac" 2>/dev/null; /usr/bin/open -a 'Self Service' && echo 'Self Service cache cleared and app relaunched.'` |
| 🔄 **Restart Self Service**           | `user`  | `/usr/bin/killall 'Self Service' 2>/dev/null; sleep 1; /usr/bin/open -a 'Self Service' && echo 'Self Service relaunched.'`                                                                                                        |
| ▶️ **Run Custom Policy Trigger**      | `root`  | `$JAMF policy -event <trigger>` — `<trigger>` must match `[A-Za-z0-9._-]+`                                                                                                                                                        |

---

## 🌐 Network & System

| Button                  | Runs As | Command                                                                                                                                                                                                                                                                      |
| ----------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 🧹 **Flush DNS Cache**  | `root`  | `/usr/bin/dscacheutil -flushcache; /usr/bin/killall -HUP mDNSResponder 2>/dev/null; echo 'DNS cache flushed.'`                                                                                                                                                               |
| 🔄 **Renew DHCP Lease** | `root`  | `IF=$(/sbin/route -n get default 2>/dev/null \| /usr/bin/awk '/interface:/{print $2}'); IF=${IF:-en0}; /usr/sbin/ipconfig set "$IF" DHCP && { sleep 2; echo "DHCP lease renewed on $IF — new IP: $(/usr/sbin/ipconfig getifaddr "$IF" 2>/dev/null \|\| echo none)"; }`       |
| 📶 **Restart Wi-Fi** ⚠️ | `root`  | `WIFI=$(/usr/sbin/networksetup -listallhardwareports \| /usr/bin/awk '/Wi-Fi/{getline; print $2}'); WIFI=${WIFI:-en0}; /usr/sbin/networksetup -setairportpower "$WIFI" off; sleep 3; /usr/sbin/networksetup -setairportpower "$WIFI" on && echo "Wi-Fi restarted on $WIFI."` |
| 🕐 **Fix System Clock** | `root`  | `/usr/bin/sntp -sS time.apple.com && echo 'System clock synchronized with time.apple.com.'`                                                                                                                                                                                  |

---

## 🔍 Diagnostics

| Button                                  | Runs As | Command                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| --------------------------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 📋 **Full Diagnostics Report**          | `root`  | `echo '────── Jamf binary ──────'; $JAMF version 2>&1; echo; echo '────── MDM enrollment ──────'; /usr/bin/profiles status -type enrollment 2>&1; echo; echo '────── Server connection ──────'; $JAMF checkJSSConnection -retry 2 2>&1; echo; echo '────── Jamf services ──────'; /bin/launchctl list \| /usr/bin/grep -i jamf \|\| echo 'No Jamf services loaded!'; echo; echo '────── Last 40 log lines ──────'; /usr/bin/tail -n 40 /var/log/jamf.log 2>&1`                                                                 |
| 🔢 **Jamf Binary Version**              | `user`  | `$JAMF version`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ⚙️ **Jamf Services Status**             | `root`  | `/bin/launchctl list \| /usr/bin/grep -i jamf \|\| echo 'No Jamf services loaded!'`                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| 📜 **View Jamf Log**                    | `root`  | `/usr/bin/tail -n 100 /var/log/jamf.log`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| 🔄 **Check for macOS Updates**          | `root`  | `/usr/sbin/softwareupdate -l 2>&1`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ⏱️ **Uptime / Restart Check**           | `user`  | `/usr/bin/uptime; BOOT=$(/usr/sbin/sysctl -n kern.boottime \| /usr/bin/awk -F'sec = ' '{print $2}' \| /usr/bin/awk -F',' '{print $1}'); NOW=$(/bin/date +%s); DAYS=$(( (NOW - BOOT) / 86400 )); echo "Last restart: $DAYS day(s) ago."; if [ "$DAYS" -ge 14 ]; then echo '⚠ Please restart your Mac soon — restarts fix many issues.'; else echo 'Restart cadence looks fine.'; fi`                                                                                                                                            |
| 🔐 **Kerberos / SSO Status**            | `user`  | `echo '── Platform SSO ──'; /usr/bin/app-sso platform -s 2>/dev/null \|\| echo 'Platform SSO not configured on this Mac.'; echo; echo '── Kerberos tickets ──'; /usr/bin/klist 2>/dev/null \|\| echo 'No Kerberos tickets found.'`                                                                                                                                                                                                                                                                                             |
| 🔑 **Jamf Connect Password Sync Check** | `user`  | `if [ -d '/Applications/Jamf Connect.app' ] \|\| /usr/bin/defaults read com.jamf.connect.state >/dev/null 2>&1; then echo 'Jamf Connect state:'; /usr/bin/defaults read com.jamf.connect.state 2>/dev/null \| /usr/bin/grep -iE 'PasswordCurrent\|NetworkAuth\|UserPass\|LastSignIn' \|\| echo '(no state details available)'; echo; echo 'If passwords are out of sync: click the Jamf Connect menu bar icon and sign in — it will offer to sync your password.'; else echo 'Jamf Connect is not installed on this Mac.'; fi` |
| 💾 **Disk Space Check**                 | `user`  | `echo '── Disk space ──'; /bin/df -h / \| /usr/bin/awk 'NR==1 \|\| /\/$/'; echo; echo '── Largest folders in your home (top 10) ──'; /usr/bin/du -xhd 1 "$HOME" 2>/dev/null \| /usr/bin/sort -hr \| /usr/bin/head -11`                                                                                                                                                                                                                                                                                                         |

---

### ⚠️ Confirmation Required

Actions marked with **⚠️** display a confirmation dialog in the application before running.

These actions may temporarily interrupt the user's connection, MDM state, or application session.
