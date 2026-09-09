# Technical Skills 2 — Operating Environments (Linux, macOS, Windows)

This covers the "Operating Environments" line on my resume. The theory of operating systems is in **OS_Interview_Prep.md**; this file is about the three systems as *environments I actually work in*, which is what the questions will be about.

---

# PART A — Linux

## A1. What is Linux, and what is the difference between Linux and a distribution?
**Linux** is strictly the **kernel** — the part that manages processes, memory, devices, and the file system. A **distribution** is the kernel plus everything that makes it usable: GNU userland tools, a package manager, an init system, a shell, and usually a desktop environment. Ubuntu, Debian, Fedora, Arch, and Kali are all distributions of the same kernel. That is why the correct pedantic name is GNU/Linux.

**Common distributions and their focus**: Ubuntu/Debian (general purpose, `apt`, `.deb`), Fedora/RHEL/CentOS (enterprise, `dnf`/`yum`, `.rpm`), Arch (rolling release, `pacman`, minimal by default), Kali (penetration testing, ships security tooling — the environment I run NetSpecter on).

## A2. Why do developers use Linux?
Free and open source, so you can read and change any part of it. Extremely stable for long-running servers. A powerful and composable command line. Native support for almost every development toolchain and server software. Excellent package management. Lightweight — it runs well on modest hardware and in containers. It is what production servers actually run, so developing on it removes an entire class of "works on my machine" problems. And it gives you real control: you can inspect and change process priorities, hardware interfaces, and system services directly — which is exactly what my ModeOS project does.

## A3. The Linux directory hierarchy
| Path | Contains |
|---|---|
| `/` | Root of everything; there are no drive letters |
| `/bin`, `/usr/bin` | Essential and general user command binaries |
| `/sbin`, `/usr/sbin` | System administration binaries |
| `/etc` | System-wide configuration files (text, editable) |
| `/home/<user>` | User home directories |
| `/var` | Variable data — logs (`/var/log`), spools, caches |
| `/tmp` | Temporary files, usually cleared on reboot |
| `/dev` | Device files — everything is a file, including disks and terminals |
| `/proc` | Virtual filesystem exposing kernel and per-process state |
| `/sys` | Virtual filesystem exposing devices and drivers — ModeOS writes to `/sys/class/backlight/*` to change screen brightness |
| `/opt` | Optional third-party software |
| `/lib`, `/usr/lib` | Shared libraries |
| `/mnt`, `/media` | Mount points for other filesystems |
| `/boot` | Kernel and bootloader files |

## A4. Everything is a file
A core Unix design principle: regular files, directories, devices, sockets, and pipes are all accessed through the same `open`/`read`/`write`/`close` interface. `/dev/null` discards anything written to it; `/dev/urandom` produces random bytes; `/dev/sda` is a whole disk. This uniformity is why shell pipelines compose so well.

## A5. Permissions
`-rwxr-xr--` reads as: file type (`-` regular, `d` directory, `l` symlink), then three triplets — owner, group, others — each being read/write/execute.
- Numeric: read 4, write 2, execute 1. So `chmod 755` = owner rwx (7), group r-x (5), others r-x (5). `chmod 644` is the normal permission for a data file.
- On a **directory**, execute means "may enter/traverse", read means "may list contents", write means "may create or delete entries".
- `chown user:group file` changes ownership. `umask` sets the default permissions for new files.
- **setuid** (`chmod u+s`) makes a program run as its owner rather than the caller — this is how `passwd` can edit `/etc/shadow`, and it is a classic privilege-escalation target in security work.
- **Root / superuser** has uid 0 and bypasses permission checks. `sudo` runs a single command as another user, governed by `/etc/sudoers`, and logs it. NetSpecter requires root because opening a raw socket is a privileged operation.

## A6. Package management
| Distribution family | Tool | Install |
|---|---|---|
| Debian/Ubuntu | `apt`, `dpkg` | `sudo apt install nginx` |
| RHEL/Fedora | `dnf`, `rpm` | `sudo dnf install nginx` |
| Arch | `pacman` | `sudo pacman -S nginx` |
| macOS | Homebrew | `brew install nginx` |
| Language-level | `pip`, `npm`, `cargo` | `pip install scapy` |
A package manager resolves dependencies, verifies signatures, and tracks what is installed so it can be cleanly removed — which is why you should not install software by copying binaries around.

## A7. Processes and services
- `ps aux`, `top`, `htop` to see what is running; `pgrep`/`pkill` to find and signal by name.
- **systemd** is the init system on most modern distributions — PID 1, which starts and supervises services. `systemctl start|stop|status|enable|disable <service>`, and `journalctl -u <service>` for its logs.
- Signals: `kill -15` (SIGTERM, polite, catchable) then `kill -9` (SIGKILL, uncatchable). My ModeOS process manager implements exactly that escalation with a 2-second grace period, and protects a whitelist of system daemons so it can never terminate a shell, a terminal multiplexer, an audio server, or the compositor.
- `nice`/`renice` change scheduling priority from −20 (highest) to +19 (lowest); lowering below 0 needs root or `CAP_SYS_NICE`. ModeOS uses this to boost the app you are focusing on and de-prioritise background ones — without root, which is a deliberate design choice.

## A8. The XDG Base Directory Specification
Where a well-behaved Linux application stores files:
- `$XDG_CONFIG_HOME` (default `~/.config/<app>`) — user configuration.
- `$XDG_STATE_HOME` (default `~/.local/state/<app>`) — state that should persist between runs but is not configuration.
- `$XDG_CACHE_HOME` (default `~/.cache/<app>`) — regenerable cached data.
- `$XDG_DATA_HOME` (default `~/.local/share/<app>`) — user data.
ModeOS follows this precisely: modes in `~/.config/modeos/modes`, the saved pre-mode state in `~/.local/state/modeos`, and the scanned application index in `~/.cache/modeos`. It is worth mentioning because it shows you know what "integrating properly with the OS" means rather than dumping dotfiles in the home directory.

## A9. Linux desktop stack (relevant to ModeOS)
- **Display server**: X11 (the older protocol, `xrandr` controls displays) versus **Wayland** (the modern replacement, more secure because clients cannot read each other's input or screen). This split is why ModeOS needs different night-light backends: `gammastep`/`wlsunset` on Wayland, `redshift` on X11.
- **Desktop environment**: GNOME (configured through `gsettings`) and KDE Plasma (configured through D-Bus/`qdbus`) — again two separate backends in ModeOS.
- **Audio stack**: ALSA is the kernel-level driver layer (`amixer`); PulseAudio was the long-standing user-space sound server (`pactl`); **PipeWire** with WirePlumber is the modern replacement (`wpctl`). ModeOS tries `wpctl`, falls back to `pactl`, then `amixer`, then a mock — a concrete example of graceful degradation and the Strategy pattern.

## A10. Linux questions that come up
- **Hard link vs soft (symbolic) link?** A hard link is a second directory entry pointing at the same inode — the file survives until the last link is removed, and it cannot cross filesystems or link to directories. A symlink is a small file containing a path — it can cross filesystems and point at directories, but it breaks if the target moves.
- **What is the inode?** The metadata structure for a file: type, permissions, owner, size, timestamps, link count, and block pointers. It does **not** contain the name — names live in directories.
- **What happens at boot?** BIOS/UEFI → POST → bootloader (GRUB) → kernel + initramfs → systemd (PID 1) → services and login. See OS_Interview_Prep.md section 38.
- **How do you find what is eating disk space?** `df -h` for filesystem usage, then `du -sh * | sort -rh | head` to walk down into the offending directory.
- **How do you check what is listening on a port?** `ss -tulpn | grep :8080` or `lsof -i :8080`.
- **What is a shell environment variable?** A key-value pair in the process environment, inherited by children. `export PATH=$PATH:/new/dir`, `env` to list, `.bashrc`/`.zshrc` to persist.
- **How do you run something after logout?** `nohup cmd &`, `tmux`/`screen`, or a systemd service.

---

# PART B — macOS

## B1. What is macOS technically?
A Unix-certified operating system built on **Darwin**, whose kernel is **XNU** — a hybrid combining the Mach microkernel with a BSD layer and I/O Kit drivers. Because the userland is BSD-derived, most Unix commands work, but they are the **BSD versions**, not the GNU ones, which is the single most practical difference for a developer.

## B2. macOS vs Linux — the differences that actually bite
| | Linux | macOS |
|---|---|---|
| Kernel | Monolithic (Linux) | Hybrid (XNU: Mach + BSD) |
| Userland tools | GNU coreutils | BSD coreutils |
| `sed -i` | `sed -i 's/a/b/' f` | `sed -i '' 's/a/b/' f` (requires a backup suffix argument) |
| Package manager | apt/dnf/pacman | Homebrew (third party) |
| Default shell | bash or zsh | zsh (since Catalina) |
| Init system | systemd | launchd (`launchctl`) |
| Filesystem | ext4, btrfs, xfs | APFS (case-insensitive by default — a real source of bugs when a repo has `File.js` and `file.js`) |
| Config location | `/etc`, `~/.config` | `~/Library/Preferences` (plist files), `defaults` command |
| Package format | .deb / .rpm | .app bundles, .dmg, .pkg |
| Backlight/audio control | sysfs, ALSA/PipeWire | CoreAudio, IOKit — no sysfs |

## B3. Security features on macOS
- **SIP (System Integrity Protection)** — even root cannot modify protected system directories.
- **Gatekeeper** — only allows applications signed by an identified developer or notarised by Apple to run by default.
- **TCC (Transparency, Consent, Control)** — prompts for access to the camera, microphone, disk, and screen recording. This is why packet capture and screen tools need explicit permission grants.
- **Keychain** — the system credential store.
- **FileVault** — full-disk encryption.

## B4. Why this matters for my work
I develop on macOS and deploy to Linux, which is exactly the environment mismatch problem that containers exist to solve. ModeOS is a Linux tool, so it ships **mock backends** (`--mock` / `MODEOS_MOCK=1`) and a **Docker Ubuntu 24.04 sandbox** so I can develop and test the Linux behaviour from macOS without a Linux machine. NetSpecter's v2 branch is explicitly cross-platform — libpcap on macOS (`/dev/bpf*`), libpcap on Linux, Npcap on Windows — because the capture driver differs even though Scapy's API does not.

---

# PART C — Windows

## C1. Architecture in one paragraph
Windows uses a **hybrid kernel** (NT kernel). Applications run in user mode and call the Win32 API, which goes through `ntdll.dll` into kernel mode. It uses the **NTFS** file system (journaling, ACL-based permissions, alternate data streams), the **Registry** as a central hierarchical configuration database instead of scattered text files, and **drive letters** (C:, D:) rather than a single root tree.

## C2. Windows vs Linux
| | Windows | Linux |
|---|---|---|
| Source | Proprietary | Open source |
| Filesystem root | Per-volume drive letters, `\` separator | Single tree from `/`, `/` separator |
| Config | Registry | Text files in `/etc` and `~/.config` |
| Permissions | ACLs, users and groups | rwx triplets plus ACLs |
| Shell | PowerShell, cmd | bash, zsh |
| Package manager | winget, Chocolatey, MSI | apt, dnf, pacman |
| Services | Windows Services (`services.msc`) | systemd units |
| Case sensitivity | Case-insensitive paths | Case-sensitive |
| Line endings | CRLF | LF |
| Executables | .exe, .dll | ELF binaries, .so |

## C3. Things worth being able to name
- **PowerShell** is object-oriented rather than text-oriented: `Get-Process | Where-Object CPU -gt 100` passes .NET objects down the pipeline, not lines of text. That is the fundamental difference from bash.
- **WSL (Windows Subsystem for Linux)** — WSL2 runs a real Linux kernel in a lightweight VM with tight Windows integration, which is how most Windows developers get a Linux toolchain now.
- **Task Manager / `tasklist` / `taskkill`** are the process tools.
- **CRLF vs LF** is a real problem in cross-platform repositories; Git's `core.autocrlf` setting and a `.gitattributes` file are the fix. This is also why my ClassRoom Code grader deliberately normalises CRLF versus LF before comparing student output — a student on Windows should not fail a test case over line endings.

---

# PART D — Cross-Platform Questions You Should Expect

**1. Which OS do you prefer and why?**
I use macOS as my daily driver because it gives a Unix shell with reliable hardware and good battery life, and Linux for anything that touches the system directly — NetSpecter needs raw sockets and libpcap, and ModeOS controls Linux-specific subsystems like sysfs backlight, PipeWire, and GNOME's gsettings. I keep a Kali VM for security work. The important part is that I write code assuming it may run somewhere else: ModeOS abstracts every hardware interaction behind a backend interface with a mock implementation, so the same code runs on GNOME, KDE, Wayland, X11, and in a container.

**2. Why do containers matter for OS differences?**
A container packages the application with its dependencies and userland, and shares the host kernel through namespaces and cgroups. It removes the "my machine has a different library version" problem without the weight of a full VM. My ModeOS repository has a `Dockerfile` and `docker-compose.yml` precisely so the Linux behaviour can be exercised from any host.

**3. What is the difference between a VM and a container?**
A VM virtualises hardware and runs a complete guest OS with its own kernel — strong isolation, gigabytes in size, seconds to boot. A container virtualises the OS: it shares the host kernel and isolates only the process view using namespaces (PID, mount, network, user, IPC, UTS) and limits resources with cgroups — megabytes in size, milliseconds to start, but weaker isolation and it must match the host kernel's family.

**4. How would you make a script work on both Linux and macOS?**
Avoid GNU-only flags, or detect the platform (`uname -s`) and branch. Use `#!/usr/bin/env bash` rather than a hard-coded path. Prefer POSIX-compatible constructs. Use portable tools (`python3`) for anything nontrivial. Or, most robustly, run it in a container. NetSpecter handles this in code: it checks `os.name != "posix"` and refuses on unsupported platforms with a clear message rather than failing obscurely.

**5. What is a file system journal?**
A log of pending metadata changes written before the changes themselves, so an interrupted write can be replayed or rolled back on the next mount instead of leaving the filesystem inconsistent. ext4, NTFS, and APFS all journal. It is the same write-ahead-logging idea databases use for durability.
