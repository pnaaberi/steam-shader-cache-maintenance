# Steam Shader Cache Maintenance

User-level automation for reclaiming disposable Steam shader-cache space on a
Steam Deck OLED with 1 TB internal storage.

## What it does

Steam stores compiled graphics shaders to improve game startup and reduce
shader-compilation stutter. The cache can grow over time, so this setup purges
its contents automatically every 30 days.

I saved approximately **13 GB** when the cache was first purged.

Steam rebuilds deleted shaders automatically when games need them again. A game
may start more slowly or stutter briefly while shaders are rebuilt.

The cleanup does not remove installed games, save files, settings, Proton
compatibility data, or the Steam library.

## Installation

The installed setup uses a user-level systemd timer:

```bash
chmod 700 ~/.local/bin/purge-steam-shadercache
systemctl --user daemon-reload
systemctl --user enable --now purge-steam-shadercache.timer
```

## Verify

```bash
systemctl --user is-enabled purge-steam-shadercache.timer
systemctl --user is-active purge-steam-shadercache.timer
systemctl --user list-timers purge-steam-shadercache.timer
systemctl --user start purge-steam-shadercache.service
systemctl --user show purge-steam-shadercache.service \
  -p Result -p ExecMainStatus
```

Expected service output:

```text
Result=success
ExecMainStatus=0
```

## Installed files

- Script: `~/.local/bin/purge-steam-shadercache`
- Service: `~/.config/systemd/user/purge-steam-shadercache.service`
- Timer: `~/.config/systemd/user/purge-steam-shadercache.timer`
- Cache target: `~/.local/share/Steam/steamapps/shadercache`

The service runs with systemd hardening enabled: no privilege escalation,
private temporary storage, a read-only system view, and write access only to
the shader-cache directory. It also uses a private default file-creation mask.

## Safety boundary

Only the immediate contents of the shader-cache directory may be removed. The
script uses an absolute path and refuses to operate on an unexpected target.

Agents must never delete the Steam library, game directories, save files, or
Proton compatibility data.

## Documentation

See [steam-shader-cache-maintenance.md](steam-shader-cache-maintenance.md) for
the full installation procedure, agent instructions, hardware and software
profile, operational commands, tests, and limitations.

## Verified environment

- Device: Steam Deck OLED (`Galileo`)
- SteamOS: 3.8.16
- Kernel: 6.16.12-valve24.5-1-neptune
- CPU: AMD Custom APU 0932, 8 logical CPUs
- GPU: AMD Custom GPU 0405, RADV VANGOGH
- RAM: 14 GiB
- Storage: 1 TB Kingston NVMe drive
