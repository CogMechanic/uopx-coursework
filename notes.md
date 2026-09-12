
# Lab Notes: Windows & Linux Fundamentals

Notes from foundational OS navigation labs — first entries in my cybersecurity coursework log. Organized by skill area rather than as a step-by-step walkthrough, since the goal here is documenting what I can actually do, not transcribing lab instructions.

---

## Windows Environment

### Navigation & the shell

Windows exposes two main ways to launch and manage applications day to day: the **Start menu / search bar** for GUI-driven access, and **PowerShell** for command-line task automation and configuration management. Knowing both matters — GUI tools are faster for one-off tasks, but PowerShell is what scales: scriptable, repeatable, and the standard for any real Windows administration or security work (event log queries, remote management, automation).

### System information via PowerShell

```powershell
Get-WmiObject -Class win32_computersystem
```

`Get-WmiObject` queries **WMI (Windows Management Instrumentation)** — a built-in Windows subsystem that exposes hardware, OS, and configuration data through a queryable interface. The `win32_computersystem` class specifically returns baseline machine info (manufacturer, model, domain membership, total RAM). This is worth remembering beyond "a command that prints info" — WMI is a core mechanism security tools (and attackers, via "living-off-the-land" techniques) use to enumerate a Windows host, since it's a legitimate, always-present interface rather than something that has to be installed.

**Note:** `Get-WmiObject` is actually deprecated in newer PowerShell versions in favor of `Get-CimInstance`, which is functionally similar but uses a more modern underlying protocol (WSMan instead of DCOM). Worth knowing both exist.

### Launching GUI apps from the command line

```powershell
explorer
```

Confirms PowerShell isn't limited to text output — it can launch full GUI applications too, which matters for scripting workflows that need to open something visual as part of an automated task.

### File Explorer basics

Standard Windows file manager — navigable either through the left sidebar (Quick Access, This PC, drive letters) or directly via path in the navigation bar (e.g. `C:\Users\Public`). Every file/folder has a unique absolute path, same underlying concept as a Linux filesystem path, just different separator conventions (`\` vs `/`) and drive-letter-based roots instead of a single `/` root.

**Practiced:** creating a new text file directly from File Explorer's right-click context menu (`New -> Text Document`), confirming basic read/write file operations in the Windows GUI.

### Server Manager

Windows Server's central admin console — `Manage` handles installing new roles/features (e.g., adding a web server role, DNS, Active Directory Domain Services), while `Tools` gives access to configuring already-installed services. Useful mental model: **Manage = add capability, Tools = configure what's already there.**

---

## Linux Environment (Kali)

### Navigation & the shell

Kali's taskbar layout mirrors the Windows one conceptually — an applications menu (organized by category, e.g. "Information Gathering," "Password Attacks") plus pinnable favorites. Kali also supports **workspaces** — multiple virtual desktops, useful for keeping a terminal, browser, and tool output visually separated without window-juggling on a single screen. This is genuinely useful in real pentest/security work, where you're often running several tools simultaneously.

### Core filesystem navigation commands

```bash
ls              # list contents of current directory
cd <dir>        # change into a directory
cd ..           # move up one directory level
```

Terminal Emulator opens by default at the filesystem root (`/`) — worth contrasting with Windows, which has no single root; every drive is its own top-level letter. Linux unifies everything (including other drives, network shares, USB devices) under one tree starting at `/`.

### Privilege escalation for admin actions

```bash
sudo mkdir test
```

`mkdir` creates a directory; `sudo` ("substitute user, do") temporarily elevates a command to run with another user's privileges — almost always root, by default. Directories like `/etc` require elevated privileges to modify, which is why `sudo` is necessary here but not for navigating into the directory to look around.

**This is worth understanding conceptually, not just as a command to memorize:** `sudo` is the core mechanism behind the principle of least privilege on Linux — regular commands run as a normal, limited user, and only specific actions get temporarily escalated, rather than working from a root shell by default (which is exactly the kind of habit that turns a small mistake into a catastrophic one).

```bash
sudo touch zybooks.txt
```

`touch` creates an empty file (or updates a file's modified timestamp if it already exists) — one of the most-used basic Linux utilities, frequently combined with redirection (`>`) or piped output in real scripting.

### `/etc` — why it matters beyond "a folder you cd into"

`/etc` holds system-wide configuration files for almost everything installed on a Linux box — network configs, service configs, user account data (`/etc/passwd`), cron jobs (`/etc/crontab`), and more. Recognizing this directory and what typically lives there is foundational for basically all later Linux security work, from hardening to enumeration during a pentest.

---

## Why these fundamentals matter going forward

Everything here is deliberately basic — that's the point of a first lab. But PowerShell/WMI on the Windows side and `sudo`/`/etc` navigation on the Linux side are exactly the primitives that show up constantly in later, more advanced security work: Windows enumeration during an assessment often starts with WMI queries; Linux privilege escalation checks almost always begin with understanding what `sudo` rights an account has and what lives under `/etc`. Treating these as the building blocks they are, rather than disposable "intro lab" commands, is the mindset I'm carrying into the rest of this coursework.