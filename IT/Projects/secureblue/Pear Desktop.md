# Installing Pear Desktop Flatpak on SecureBlue/Wayland
> Note: I'm referring to the YouTube Music package as "Pear Desktop" due to the recent name change. The releases are still called YouTube Music for some reason.

## Finding the Correct Package
I couldn't find an official Pear Desktop package on Flathub — only the [YTMDesktop](https://flathub.org/en/apps/app.ytmdesktop.ytmdesktop) package. *This package might be easier to install but has fewer features.* Luckily, the [Pear Devs](https://github.com/pear-devs) provide an official [YouTube-Music-3.11.0-x86_64.flatpak](https://github.com/pear-devs/pear-desktop/releases/download/v3.11.0/YouTube-Music-3.11.0-x86_64.flatpak) version on their GitHub.

## Installing the Package
1. Make sure you have added the official Flathub remote. Pear Desktop refuses to install on the flathub-verified remote.
2. Install the Flatpak with:  
   ```bash
   flatpak install /path/to/YouTube-Music-3.11.0-x86_64.flatpak
   ```
This should prompt you to install all dependencies from the Flathub remote.
3. Configure the Flatpak:

As you may have noticed, the app refuses to launch on Wayland — instead, it tries to launch with X11 and fails. We can fix this by running it with the following parameters:

```bash
flatpak run \
  --socket=wayland \
  --env=ELECTRON_ENABLE_WAYLAND=1 \
  --env=ELECTRON_OZONE_PLATFORM_HINT=wayland \
  com.github.th_ch.youtube_music \
  --enable-features=UseOzonePlatform,WaylandWindowDecorations \
  --ozone-platform=wayland
```

If this works, we can make the changes permanent by adding the environment variables and modifying the launch flags in the application shortcut.

### Adding Environment Variables Permanently

```bash
flatpak override --user \
  --env=ELECTRON_ENABLE_WAYLAND=1 \
  --env=ELECTRON_OZONE_PLATFORM_HINT=wayland \
  com.github.th_ch.youtube_music
```

### Editing Launch Flags

The `.desktop` file should be located at:
`/var/lib/flatpak/exports/share/applications/com.github.th_ch.youtube_music.desktop`

Make a backup of this file, then change the Exec line to:
```text
Exec=flatpak run com.github.th_ch.youtube_music --enable-features=UseOzonePlatform,WaylandWindowDecorations --ozone-platform=wayland
```
