# Project 4 — ModeOS

**Repository:** https://github.com/praneeth3696/modeos
**Stack:** Python 3.8+, psutil, PyYAML, Docker (Ubuntu 24.04 test sandbox)
**Nature:** A terminal-based Linux system utility — an adaptive OS mode manager

This is the project that demonstrates operating-systems knowledge in practice, and it is the best OOP example I have. If they ask about OS concepts or design patterns, bring this up.

---

## 1. The one-minute pitch

"ModeOS is a command-line utility that reconfigures your whole Linux environment from a single named profile. You define a mode in YAML — say `deep_work` — with a target brightness, volume, night-light state, which applications to keep, which to close, and which to boost or de-prioritise. Running `modeos mode deep_work` applies all of it at once, records exactly what the system looked like beforehand, and `modeos revert` puts it back precisely. Every piece of hardware control goes through a pluggable backend interface, so it works across PipeWire, PulseAudio and ALSA for audio; sysfs, brightnessctl and xrandr for display; and GNOME, KDE, gammastep and redshift for night light — with a mock backend so it can be developed and tested on macOS or in a container. It runs entirely in user space with no root."

## 2. The problem it solves

Switching between what you are doing is not one action, it is many: dim the screen, mute or lower the volume, turn on the blue-light filter, close Discord and Steam, give your editor more CPU priority, de-prioritise the browser. Doing that by hand every time is friction, so people do not do it — they just leave twenty things running and get distracted.

"Do Not Disturb" only silences notifications. What was missing was something that treats a *mode* as a first-class object: a named, declarative description of a whole system state that can be applied and, crucially, **undone exactly**.

The undo is the actual hard part, and it is what most such scripts get wrong. A tool that changes your system and cannot restore it is worse than no tool.

## 3. Architecture

```
modeos/
  cli.py         argparse command surface: list, mode, revert, reset, current, doctor, scan, validate
  core.py        orchestration: apply_mode, revert_system, reset_system
  models.py      ModeConfig dataclass with a validating from_dict factory
  config.py      XDG paths, mock-mode detection, mode discovery
  backends/
    base.py      abstract base classes: AudioBackend, DisplayBackend, NightLightBackend
    audio.py     WirePlumber (wpctl), PulseAudio (pactl), ALSA (amixer), Mock
    display.py   sysfs /sys/class/backlight/*, brightnessctl, xrandr, Mock
    nightlight.py GNOME gsettings, KDE D-Bus (qdbus), gammastep/wlsunset, redshift, Mock
  process.py     process discovery, matching, protection, termination, nice values
  state.py       capture, save, load, restore the pre-mode system state
  scanner.py     index installed applications (system, user, Flatpak, Snap)
  doctor.py      diagnostic report of which backends are available on this machine
  logger.py      structured logging
modes/           YAML profiles: deep_work, gaming, presentation, coding, focus, music, battery_saver, minimal, custom
Dockerfile, docker-compose.yml   Ubuntu 24.04 sandbox for testing from any host
```

## 4. The pluggable backend design — my best OOP answer

`backends/base.py` declares abstract base classes using Python's `abc` module:

```python
class AudioBackend(ABC):
    @property
    @abstractmethod
    def name(self) -> str: ...
    @abstractmethod
    def is_available(self) -> bool: ...
    @abstractmethod
    def get_volume(self) -> Optional[int]: ...
    @abstractmethod
    def set_volume(self, target_percent: int, dry_run: bool = False) -> bool: ...
```

Each concrete backend wraps one real system tool. At startup the application asks each candidate `is_available()` — which checks whether the required binary or interface actually exists on this machine — and selects the first that works, falling back to a mock if none do.

**Why this matters, and what to say:** the Linux desktop is genuinely fragmented. Audio might be PipeWire with WirePlumber, or PulseAudio, or bare ALSA. Display brightness might be the kernel sysfs interface, or `brightnessctl`, or `xrandr` on an external monitor. Night light is `gsettings` on GNOME, D-Bus on KDE, `gammastep` on Wayland, and `redshift` on X11. Writing `if gnome: ... elif kde: ...` through the codebase would be unmaintainable and untestable.

The interface makes it:
- **Abstraction** — `core.py` calls `set_volume(30)` and never learns which tool ran.
- **Polymorphism** — one call site, many implementations, resolved at runtime.
- **The Strategy pattern** — interchangeable algorithms behind a common interface, chosen at runtime.
- **Dependency inversion** — the high-level orchestration depends on the abstraction, not on `wpctl`.
- **Testability** — the mock backend means the entire application logic can be exercised on macOS, in CI, or in a container with no audio hardware at all. That is not a side benefit; it is the reason I can develop this on a Mac.

## 5. Safety engineering — the part I am most proud of

A tool that terminates processes and writes to hardware interfaces has to be conservative, and every one of these was a deliberate decision.

**1. Two-stage graceful termination.** `SIGTERM` first, then wait a **2-second grace period**, and only escalate to `SIGKILL` for processes that did not exit. SIGTERM is catchable, so an editor gets the chance to flush unsaved buffers; SIGKILL cannot be caught, so it would destroy that work. This is the standard Unix shutdown contract, and implementing it correctly — rather than reaching straight for `kill -9` — is the difference between a tool people trust and one they uninstall after it eats their work.

**2. A system daemon protection whitelist.** `is_system_protected()` refuses to touch shells, terminal multiplexers (`tmux`, `screen`), audio servers, the display compositor, IDEs, and system daemons. Without this, `kill_all_except_allow: true` would terminate the very terminal running ModeOS, or the compositor, and take the desktop session down. It also only ever considers **user-owned processes**, filtered by uid.

**3. A genuinely guaranteed dry run.** `--dry-run` performs zero destructive actions: no processes signalled, no nice values changed, no hardware writes. The flag is threaded all the way down into every backend and every process operation rather than being checked once at the top, because a partial dry run is worse than none — it makes you trust a preview that is not accurate.

**4. Exact state restoration.** Before applying a mode, `state.py` captures the current brightness, volume, night-light state, the **original nice value of every process it is about to change**, and which applications it terminated, and writes it as JSON to `~/.local/state/modeos`. `modeos revert` reads that file and restores each value individually. It stores the *original* nice value per PID, not a global default, so reverting is genuinely a restore rather than a reset. (`modeos reset` is the separate, blunter operation that returns everything to defaults.)

**5. Input validation at the boundary.** `ModeConfig.from_dict` clamps brightness and volume to 0–100 and CPU limit to 1–100, coerces types defensively, and rejects a non-mapping configuration with a clear error naming the mode. A typo in a YAML file cannot become a nonsensical hardware write.

**6. Rootless by design.** Everything runs in user space. Relative CPU prioritisation is achieved without `sudo` — which is possible because **raising** a nice value (lowering priority) needs no privilege; only lowering it below zero does. So de-prioritising a distraction always works, and boosting is attempted and reported honestly when permission is denied, with a message explaining that negative nice values require root or `CAP_SYS_NICE`. Not requiring root for a tool that kills processes is a security decision: a bug in it cannot damage the system.

**7. XDG compliance.** Configuration in `~/.config/modeos`, state in `~/.local/state/modeos`, cache in `~/.cache/modeos` — following the specification rather than scattering dotfiles, and keeping regenerable data (the app index) separate from data that must survive (the saved state).

## 6. Operating-systems concepts demonstrated

This is the direct bridge to the OS interview questions.

- **Process discovery** — `psutil.process_iter(['pid', 'name', 'cmdline', 'uids', 'nice'])` reads from `/proc` on Linux. Requesting only the fields needed avoids a syscall per attribute per process.
- **Signals** — SIGTERM (15) is a request that can be caught, blocked, or ignored; SIGKILL (9) is delivered by the kernel and cannot be intercepted. The escalation between them is the whole point.
- **Scheduling priority** — the `nice` value ranges from −20 (highest priority) to +19 (lowest). On Linux this feeds the Completely Fair Scheduler's weighting, so it is proportional rather than absolute — a niced-down process still runs, it just gets a smaller share when there is contention. Only root or `CAP_SYS_NICE` may go below zero.
- **Race conditions in process handling** — a PID can disappear between listing it and acting on it, so every operation catches `NoSuchProcess` and `AccessDenied`. PID reuse means a stale PID in the saved state might now be a different process, which is why restoration verifies before acting.
- **sysfs** — `/sys/class/backlight/*/brightness` is the kernel exposing a device control as a file. Writing an integer to it changes the hardware. This is "everything is a file" made literal.
- **The desktop stack** — X11 versus Wayland, and why the night-light mechanism differs; ALSA as the kernel driver layer with PulseAudio and PipeWire as user-space sound servers above it.
- **Containers** — the Docker sandbox runs the Linux code paths on any host by sharing the host kernel through namespaces and cgroups.

## 7. A mode file

```yaml
description: Deep focus session for coding
brightness: 80              # target display brightness percentage
volume: 10                  # target audio volume percentage
night_light: true           # blue light filter on
cpu_limit: 100

kill_all_except_allow: true # strict focus: close everything except the allow list
allow_apps:
  - code
  - terminal
  - nvim

boost_apps:
  code: -10                 # negative nice = higher priority (needs privilege)
reduce_apps:
  slack: 10                 # positive nice = lower priority (always works)
```

Shipped modes (11): `deep_work`, `coding`, `focus`, `gaming`, `presentation`, `music`, `reading`, `sleep`, `battery_saver`, `minimal`, `custom`. `modeos scan` indexes installed applications across system, user, Flatpak, and Snap directories so profiles can name apps by friendly name (`code`, `discord`, `steam`) rather than full executable paths — and `modeos validate <name>` checks a profile's syntax and schema before you apply it.

## 8. Commands

| Command | What it does |
|---|---|
| `modeos list` | List discovered modes and their rules |
| `modeos mode <name>` | Apply a mode |
| `modeos mode <name> --dry-run` | Preview with guaranteed zero side effects |
| `modeos revert` | Restore the exact pre-mode state |
| `modeos reset` | Return hardware and priorities to defaults |
| `modeos current` | Show the active mode and session telemetry |
| `modeos doctor` | Report which backends are available on this machine |
| `modeos scan` | Index installed applications |
| `modeos validate <name>` | Check a mode file's syntax and schema |

## 9. Questions they will ask, with answers

**"Why not just write a shell script?"**
A shell script can set the brightness. What it cannot easily do is the part that actually matters: capture and restore the exact prior state per process, abstract over four different night-light mechanisms so it works on both GNOME and KDE, guarantee a dry run all the way down, protect system daemons from a kill-everything rule, and be unit-testable. The moment you need state, abstraction, and safety guarantees, you need structure — and that structure is the project.

**"How do you avoid killing something important?"**
Three layers. First, only user-owned processes are ever considered. Second, an explicit whitelist protects shells, terminal multiplexers, audio servers, the compositor, IDEs, and system daemons. Third, termination is always SIGTERM with a grace period before SIGKILL, so anything that can save state gets the chance. And `--dry-run` lets you see exactly what would happen first.

**"What happens if the machine loses power mid-mode?"**
The state file is written to disk before any changes are applied, so on the next boot `modeos revert` still has the record and can restore the hardware settings. Process priorities and terminated applications do not survive a reboot anyway, so that part is moot. What is *not* handled is a crash partway through applying, which could leave the state file describing a system that was only partially changed — reverting would still be safe because it restores per-item, but a transactional apply with a journal would be the proper fix.

**"How do you test a tool that changes hardware?"**
That is exactly what the mock backends are for. `--mock` or `MODEOS_MOCK=1` substitutes an implementation of every backend interface that records what it was asked to do without touching anything, so the whole orchestration, state capture, and restore logic can be exercised in CI or on macOS. On top of that there is a Docker Ubuntu 24.04 sandbox (`docker compose run modeos-test`) for exercising the Linux code paths, and a unittest suite covering backends, process management, state reversion, and YAML schema validation. `modeos doctor` reports what is actually available on a real machine.

**"What is the hardest bug you hit?"**
Priority restoration. Storing "what I set it to" is useless — you have to store the *original* nice value per PID, before changing it, and restore each one individually. And then a PID can be reused by a completely different process before you revert, so restoring blindly would set an unrelated process's priority. So restoration verifies the process still exists and skips it if its current nice value is already the target. It made me appreciate that undo is a genuinely harder problem than do.

**"What is next for it?"**
The README says it, and I would say it honestly: a persistent background daemon so modes can react to events — time of day, which application has focus, whether you are on battery — rather than only being applied manually. And broader hardware backend support beyond the current audio and display tooling. The backend interface already exists, so adding a new one is implementing four methods and registering it, which is the payoff of having designed it that way.

**"How does this relate to what an OS actually does?"**
It is a small, user-space version of the resource management an OS does in the kernel: it inspects the process table, changes scheduling priority, sends signals to terminate processes, and writes to device interfaces. The difference is that the kernel does it by policy for the whole system and this does it by user intent for one session. Building it is what made process states, signals, nice values, and `/proc` concrete for me rather than exam material.
