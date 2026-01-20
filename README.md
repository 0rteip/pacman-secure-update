# pacman-systemd-inhibit

Inhibit system shutdown, reboot etc. when pacman is upgrading the system.

Accidental shutdown or reboot of the system when pacman is upgrading the system can lead to an unbootable/broken system or broken packages.

These hooks and script inhibit accidental system shutdown, reboot etc. when pacman is upgrading the system.

## Features

- **Shutdown/reboot inhibition**: Prevents accidental system shutdown, reboot, sleep, and lid switch actions during package transactions
- **Battery check**: Blocks updates if battery is below 20% (skipped on desktop/AC power)

## Installation

### Prerequisites

Create the symlink for the sleep process (required for both methods):

```bash
sudo ln -sf /usr/bin/sleep /usr/bin/pacman-systemd-inhibit-sleep
```

### Option 1: Local installation (recommended for manual setup)

Use this if you're installing manually or want to customize the hooks.

```bash
# Copy hooks to local hooks directory
sudo mkdir -p /etc/pacman.d/hooks
sudo cp hooks/*.hook /etc/pacman.d/hooks/

# Copy script
sudo cp scripts/systemd-inhibit-hook /usr/share/libalpm/scripts/
sudo chmod +x /usr/share/libalpm/scripts/systemd-inhibit-hook
```

**Why choose this?**

- Won't be overwritten by package updates
- Takes precedence over system hooks

### Option 2: System-wide installation

Use this if you're installing it for the whole system.

```bash
# Copy hooks to system hooks directory
sudo cp hooks/*.hook /usr/share/libalpm/hooks/

# Copy script
sudo cp scripts/systemd-inhibit-hook /usr/share/libalpm/scripts/
sudo chmod +x /usr/share/libalpm/scripts/systemd-inhibit-hook
```

**Why choose this?**

- Cleaner separation between package files and user configs

## How it works

1. When `pacman` begins a transaction, it calls the PreTransaction hook `00-50-systemd-inhibit.hook`:
   - Checks if battery is above 20% (if on battery power)
   - Puts an inhibition lock for shutdown, restart, sleep, etc. with a 15-minute timeout

2. When `pacman` finishes the transaction, it calls the PostTransaction hook `zz-50-systemd-inhibit.hook`:
   - Removes the inhibition lock

**Note:** If pacman takes more than 15 minutes (unlikely), the inhibition lock expires but pacman continues normally.
