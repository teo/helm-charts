# Games-on-Whales Security Improvements

This document outlines the security improvements made to the Games-on-Whales Helm chart based on research into the project's actual requirements.

## What is Games-on-Whales?

Games-on-Whales is a streaming solution that runs applications/games in Docker containers and streams them via Sunshine/Moonlight. It consists of multiple containers:
- **sunshine**: Streaming server (main container)
- **xorg**: X11 server for graphics
- **pulseaudio**: Audio server
- **retroarch/steam/firefox**: Application containers

## Security Issues Identified

### Previous Configuration (Insecure)
- ✗ All containers running with `privileged: true`
- ✗ Using `hostNetwork: true` (major security risk)
- ✗ Mounting entire directories: `/dev/input`, `/run/udev`, `/var/log`
- ✗ Broad device access patterns
- ✗ No capability restrictions

### Current Configuration (Secured)
- ✅ Containers run as non-root user (UID 1000) with `runAsNonRoot: true`
- ✅ Specific capability grants only where needed (`SYS_ADMIN` for uinput/DRI access)
- ✅ All other capabilities dropped (`drop: [ALL]`)
- ✅ No host network - using emptyDir volumes for inter-container communication
- ✅ Minimal host path mounts: only `/dev/uinput` and `/dev/dri`
- ✅ Removed unnecessary mounts: `/run/udev`, `/var/log`, host `/tmp/.X11-unix`

## Security Improvements Made

### 1. Removed Privileged Access
**Before**: `privileged: true` on all containers
**After**: Specific security contexts with minimal capabilities

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  runAsGroup: 1000
  capabilities:
    add:
      - SYS_ADMIN  # Only where needed for device access
    drop:
      - ALL
```

### 2. Eliminated Host Network
**Before**: `hostNetwork: true`
**After**: Containers communicate via emptyDir volumes and Unix sockets

This prevents containers from accessing the host's network stack.

### 3. Minimized Host Path Mounts
**Before**: 
- `/dev/input` (entire directory)
- `/run/udev` (not needed)
- `/var/log` (unnecessary host access)
- `/tmp/.X11-unix` (host path)

**After**:
- `/dev/uinput` (specific device only)
- `/dev/dri` (GPU devices only)
- X11 communication via emptyDir volumes

### 4. Capability-Based Device Access
Instead of full privileged access, containers now use:
- `SYS_ADMIN` capability for uinput device creation and DRI access
- All other capabilities dropped
- Specific device mounts instead of broad directory access

### 5. Init Container Security
The init container (mkhomeretrodirs) runs as root only for the chown operation:
```yaml
securityContext:
  runAsUser: 0  # Need root to chown, but only in init container
  capabilities:
    add:
      - CHOWN
    drop:
      - ALL
```

## What Still Works

These changes maintain full functionality:
- **Input devices**: uinput device for controller/keyboard/mouse simulation
- **Graphics**: DRI device access for GPU acceleration
- **Audio**: PulseAudio communication via Unix sockets
- **X11**: Inter-container X11 communication via shared volumes
- **Applications**: Steam, RetroArch, Firefox all function normally

## Impact Assessment

### Security Benefits
- **Reduced attack surface**: No privileged containers
- **Network isolation**: No host network access
- **File system protection**: Minimal host path access
- **Capability isolation**: Principle of least privilege applied

### Compatibility
- ✅ Maintains full Games-on-Whales functionality
- ✅ Compatible with existing configurations
- ✅ No user-visible changes in behavior
- ✅ Performance unchanged

## Deployment Notes

1. The chart still requires specific kernel support for uinput devices
2. GPU access (DRI devices) is maintained for hardware acceleration
3. Audio streaming continues to work via PulseAudio sockets
4. Controller input is preserved through uinput device access

These improvements significantly enhance the security posture while maintaining full functionality of the Games-on-Whales streaming solution.