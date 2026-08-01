# Steam Shader Cache Maintenance

Small, user-level Linux automation for reclaiming Steam shader-cache space.

## Verified hardware profile

This installation was tested on a **Steam Deck OLED with 1 TB internal NVMe
storage**.

- Device platform: Steam Deck OLED (`Galileo`)
- CPU: AMD Custom APU 0932, 8 logical CPUs
- GPU: AMD Custom GPU 0405, RADV VANGOGH
- RAM: 14 GiB
- Internal storage: 1 TB Kingston NVMe drive
- SteamOS: 3.8.16
- Kernel: 6.16.12-valve24.5-1-neptune

The timer and purge service passed functional testing on this hardware.

## Overview

The installed timer purges the contents of Steam's shader cache every 30 days.
It runs as the `deck` user and needs no root access.

Purging is safe for installed games and saves, but it is not a permanent space
optimization: Steam recreates shader data as games run. The first launch after
a purge may take longer and may stutter while shaders are compiled again.

## Requirements

- Linux with a systemd user manager
- Steam installed at `/home/deck/.local/share/Steam`
- `find` and `rm`

The script resolves the target under the installing user's home directory and
refuses to operate if the target path is not the expected directory.

## Files

- Script: `~/.local/bin/purge-steam-shadercache`
- Service: `~/.config/systemd/user/purge-steam-shadercache.service`
- Timer: `~/.config/systemd/user/purge-steam-shadercache.timer`
- Target: `~/.local/share/Steam/steamapps/shadercache`

Only immediate children of the shader-cache directory are removed. Installed
games, saves, and the directory itself are left in place.

The service is sandboxed with no privilege escalation, a private temporary
directory, a read-only system view, and write access only to the shader-cache
directory.

## Installation

The current installation is already active. To install the same files on a
matching machine, copy the script and the two systemd units to the paths above,
make the script executable, then run:

```bash
chmod 700 ~/.local/bin/purge-steam-shadercache
systemctl --user daemon-reload
systemctl --user enable --now purge-steam-shadercache.timer
```

The timer uses `Persistent=true`, so a missed run is handled when the user
systemd manager becomes available again.

## Agent installation instructions

An agent installing this on a matching machine should:

1. Confirm systemd and the Steam shader-cache path:

   ```bash
   command -v systemctl
   systemctl --user is-system-running
   test -d ~/.local/share/Steam/steamapps/shadercache
   ```

2. Install the script at `~/.local/bin/purge-steam-shadercache` and set its
   permissions:

   ```bash
   chmod 700 ~/.local/bin/purge-steam-shadercache
   ```

3. Install the service and timer at:

   ```text
   ~/.config/systemd/user/purge-steam-shadercache.service
   ~/.config/systemd/user/purge-steam-shadercache.timer
   ```

4. Reload and enable the timer:

   ```bash
   systemctl --user daemon-reload
   systemctl --user enable --now purge-steam-shadercache.timer
   ```

5. Verify the installation:

   ```bash
   systemctl --user is-enabled purge-steam-shadercache.timer
   systemctl --user is-active purge-steam-shadercache.timer
   systemctl --user list-timers purge-steam-shadercache.timer
   ```

6. Run a functional test and confirm `Result=success` and
   `ExecMainStatus=0`:

   ```bash
   systemctl --user start purge-steam-shadercache.service
   systemctl --user show purge-steam-shadercache.service \
     -p Result -p ExecMainStatus
   ```

An agent must never delete the Steam library, game directories, save files, or
compatibility data. Only the contents of
`~/.local/share/Steam/steamapps/shadercache` may be purged.

## Check the timer

```bash
systemctl --user list-timers purge-steam-shadercache.timer
systemctl --user status purge-steam-shadercache.timer
```

View the last service run:

```bash
systemctl --user status purge-steam-shadercache.service
journalctl --user -u purge-steam-shadercache.service
```

## Run it manually

```bash
systemctl --user start purge-steam-shadercache.service
```

## Disable or re-enable it

```bash
systemctl --user disable --now purge-steam-shadercache.timer
systemctl --user enable --now purge-steam-shadercache.timer
```

## Verification

Run these checks after installation or changes:

```bash
bash -n ~/.local/bin/purge-steam-shadercache
systemd-analyze --user verify ~/.config/systemd/user/purge-steam-shadercache.service
systemd-analyze --user verify ~/.config/systemd/user/purge-steam-shadercache.timer
systemctl --user is-enabled purge-steam-shadercache.timer
systemctl --user is-active purge-steam-shadercache.timer
```

The manual service command is the functional test. It should exit with status
0 and leave the shader-cache directory present.

## Limitations

- The path is specific to this machine and user.
- The timer does not track whether Steam is running. Run it manually if a game
  is actively compiling shaders.
- Shader caches may grow again between scheduled purges.

## License

This project is available under the [MIT License](LICENSE).
