# Helium on AerynOS

> **Testing.** User-space Helium AppImage setup for AerynOS with GNOME/Wayland, Widevine DRM and automatic updates.

Nothing is installed into `/usr` or `/opt`.

## Layout

```text
~/.local/opt/helium/helium.AppImage
~/.local/bin/helium
~/.local/share/applications/helium.desktop
~/.local/share/icons/hicolor/256x256/apps/helium.png

~/.local/share/helium-widevine/
~/.config/net.imput.helium/WidevineCdm/latest-component-updated-widevine-cdm

~/.local/bin/update-helium
~/.local/bin/update-helium-widevine

~/.config/systemd/user/helium-update.service
~/.config/systemd/user/helium-update.timer
~/.config/systemd/user/helium-widevine-update.service
~/.config/systemd/user/helium-widevine-update.timer
```

## Requirements

Required commands:

```bash
for cmd in ar bsdtar curl grep sed; do
    command -v "$cmd" || echo "MISSING: $cmd"
done
```

On the tested AerynOS install, `ar` comes from `binutils`. Install missing packages with `moss`.

Example:

```bash
sudo moss install curl binutils
```

## Install the AppImage

```bash
mkdir -p ~/.local/opt/helium ~/.local/bin
chmod +x ~/.local/opt/helium/helium.AppImage
ln -sf ~/.local/opt/helium/helium.AppImage ~/.local/bin/helium
```

Make sure `~/.local/bin` is in `PATH`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Verify:

```bash
command -v helium
helium --version
```

## GNOME integration

Extract the AppImage once:

```bash
cd /tmp
rm -rf squashfs-root
helium --appimage-extract >/dev/null
```

Install the icon:

```bash
mkdir -p ~/.local/share/icons/hicolor/256x256/apps
cp /tmp/squashfs-root/usr/share/icons/hicolor/256x256/apps/helium.png \
   ~/.local/share/icons/hicolor/256x256/apps/helium.png
```

Install the desktop file:

```bash
mkdir -p ~/.local/share/applications
sed "s|@HOME@|$HOME|g" desktop/helium.desktop \
  > ~/.local/share/applications/helium.desktop

desktop-file-validate ~/.local/share/applications/helium.desktop
update-desktop-database ~/.local/share/applications
rm -rf /tmp/squashfs-root
```

## Widevine

Helium does not ship Widevine.

`scripts/update-helium-widevine` downloads the current official Google Chrome stable `.deb`, extracts only `WidevineCdm`, installs it into the user profile, writes Helium's Widevine hint file and deletes the temporary Chrome package/extraction on exit.

Install:

```bash
install -Dm755 scripts/update-helium-widevine \
  ~/.local/bin/update-helium-widevine
```

Run:

```bash
update-helium-widevine
```

Installed layout:

```text
~/.local/share/helium-widevine/
├── LICENSE
├── manifest.json
└── _platform_specific/linux_x64/libwidevinecdm.so
```

Hint:

```text
~/.config/net.imput.helium/WidevineCdm/latest-component-updated-widevine-cdm
```

Example:

```json
{"Path":"/home/USER/.local/share/helium-widevine"}
```

No launch flags are required.

## Helium auto-update

Install:

```bash
install -Dm755 scripts/update-helium ~/.local/bin/update-helium
```

The updater:

1. resolves the latest `imputnet/helium-linux` GitHub release;
2. compares it to the installed AppImage version;
3. downloads nothing if current;
4. downloads the new x86_64 AppImage if required;
5. verifies the downloaded AppImage's reported version;
6. replaces `~/.local/opt/helium/helium.AppImage`.

The filename stays constant, so the symlink and desktop entry do not change after updates.

## systemd user timers

Helium checks daily.

Widevine checks weekly because checking it requires downloading/extracting current Chrome stable.

Install:

```bash
mkdir -p ~/.config/systemd/user
cp systemd-user/* ~/.config/systemd/user/

systemctl --user daemon-reload
systemctl --user enable --now helium-update.timer
systemctl --user enable --now helium-widevine-update.timer
```

Verify:

```bash
systemctl --user list-timers --all | grep -E 'helium|widevine'
```

## Verify Widevine

```bash
cat ~/.config/net.imput.helium/WidevineCdm/latest-component-updated-widevine-cdm

grep '"version"' ~/.local/share/helium-widevine/manifest.json

ls -lh ~/.local/share/helium-widevine/_platform_specific/linux_x64/libwidevinecdm.so
```

Fully restart Helium after installing/updating Widevine.

## Wayland / GPU

Current Helium runs natively on Wayland on the tested GNOME setup.

Verify:

```bash
ps -eo pid,cmd | grep '[h]elium.*--type=gpu-process'
```

Expected to include:

```text
--ozone-platform=wayland
```

Check `chrome://gpu`.

Tested NVIDIA 610 / GTX 1650 result:

```text
Canvas: Hardware accelerated
Compositing: Hardware accelerated
Rasterization: Hardware accelerated
OpenGL: Enabled
Video Decode: Hardware accelerated
WebGL: Hardware accelerated
WebGPU: Hardware accelerated
```

Vulkan is not forced. ANGLE/OpenGL is left at the working default.

Hardware video encode may remain disabled on NVIDIA Linux. Hardware video decode still works.

## Browser settings

Recommended:

```text
Hardware acceleration: ON
Memory Saver: Balanced
Preload pages: OFF
Background apps after closing Helium: OFF
```

## Logs

Helium:

```bash
journalctl --user -u helium-update.service -n 30 --no-pager
```

Widevine:

```bash
journalctl --user -u helium-widevine-update.service -n 30 --no-pager
```

## Status

**Beta.**

Tested with:

```text
AerynOS 2026.08
GNOME 50 / Wayland
Helium 0.16.2.1
Chromium 152.0.7977.64
NVIDIA 610.57.04
Widevine 4.10.3050.0
```

Those versions are examples from the tested system, not pinned requirements.

## References

- https://github.com/imputnet/helium-linux
- https://github.com/imputnet/helium
- https://aerynos.com/tooling/moss/
