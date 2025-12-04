# Package Manager in Linux Runbook

## Snap Debugging

### Check Package Info
```bash
snap find <pkg-name>
snap info <pkg-name>
snap list | grep <pkg-name>
```
### Update / Refresh
```bash
sudo snap refresh <pkg-name>
snap refresh --time
snap refresh --unhold <pkg-name>
```

### Check Logs
```bash
journalctl -u snapd
```

## Flatpak Debugging
### Inspect Installed Apps
```bash
flatpak list
flatpak info <app-id>
```

### Search and Install
```bash
flatpak search <pkg>
flatpak install <repo> <pkg>
```

### Debug App Launch
```bash
flatpak run -v <app-id>
```

### Logs
```bash
journalctl -xe | grep flatpak
```



