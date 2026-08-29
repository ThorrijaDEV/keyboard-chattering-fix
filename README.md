# Keyboard Chattering Fix — Linux

A simple, effective solution for keyboard switch bounce (chattering) on Linux using a systemd service and a bash filter script. Stops duplicate keypresses caused by worn mechanical switches.

## What is it?

A lightweight fix for "chattering" — when a mechanical keyboard switch registers multiple presses for a single physical keystroke due to worn contacts. This runs as a systemd service that intercepts input events from your keyboard device and filters out rapid duplicate events within a configurable time window.

## What does it do?

- **Filters key chatter**: Drops key press/release events that occur within a debounce window (default 25ms)
- **Systemd service**: Runs automatically at boot, restarts on failure
- **Device-specific**: Targets only your keyboard device (by stable `/dev/input/by-id/` path)
- **Preserves key repeats**: Holding a key (auto-repeat) works normally — only rapid bounce is filtered
- **Configurable**: Adjust debounce time via systemd drop-in or environment variable
- **Zero dependencies**: Pure bash + standard Linux tools (evtest, uinput)
- **Logs to journal**: `journalctl -u chattering-fix` for debugging

## How it works

```
Physical Keyboard → /dev/input/eventX → chattering_fix.sh (filter) → uinput → Virtual Keyboard → X11/Wayland
```

The script reads raw `input_event` structs from the kernel, remembers the last timestamp per key+state, and drops events that arrive too quickly. The filtered stream is written to a uinput virtual device that the desktop environment uses instead of the raw device.

## Installation

### Quick install (Arch/Omarchy)

```bash
git clone https://github.com/ThorrijaDEV/keyboard-chattering-fix
cd keyboard-chattering-fix
sudo make install
```

This installs:
- `chattering_fix.sh` → `/usr/local/bin/`
- `chattering-fix.service` → `/etc/systemd/system/`
- Enables and starts the service

### Manual install

1. Find your keyboard device path:
   ```bash
   ls /dev/input/by-id/*-event-kbd
   # Example: /dev/input/by-id/usb-Keychron_Keychron_K2-event-kbd
   ```

2. Edit `chattering-fix.service` and set `DEVICE_PATH` to your keyboard

3. Install files:
   ```bash
   sudo cp chattering_fix.sh /usr/local/bin/
   sudo cp chattering-fix.service /etc/systemd/system/
   sudo systemctl daemon-reload
   sudo systemctl enable --now chattering-fix
   ```

## Configuration

Create a drop-in for custom debounce time:

```bash
sudo systemctl edit chattering-fix
```

Add:
```ini
[Service]
Environment=DEBOUNCE_MS=30
```

Or edit the service file directly and change `Environment=DEBOUNCE_MS=25`.

## Requirements

- Linux with systemd
- `uinput` kernel module (`modprobe uinput` or ensure `uinput` in `/etc/modules-load.d/`)
- User in `input` group (for reading `/dev/input/event*`)
- `evtest` (for debugging: `sudo evtest /dev/input/by-id/...`)

## Verification

```bash
# Check service status
systemctl status chattering-fix

# View logs
journalctl -u chattering-fix -f

# Test: type rapidly and verify no duplicate characters
```

## Status

**Complete / Stable**

Simple, focused solution used daily on mechanical keyboards with worn switches (Cherry MX, Gateron, Kailh). No known issues.

## Repository Note

This repository contains documentation only. The implementation is two files: a bash filter script and a systemd service unit.

## License

MIT