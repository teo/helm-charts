# GPU Configuration for Games-on-Whales

## Overview

The Games-on-Whales Helm chart now supports different GPU configurations with appropriate security contexts and device mounting strategies.

## GPU Types

### NVIDIA GPUs (with Device Plugin + RuntimeClass)

When using NVIDIA GPUs with the NVIDIA Device Plugin and NVIDIA RuntimeClass:

```yaml
gpu:
  type: "nvidia"

# Example with GPU resource requests
graphic_resources:
  nvidia.com/gpu: 1
```

**What happens:**
- ✅ **No `/dev/dri` mount** - NVIDIA RuntimeClass handles device mounting
- ✅ **Reduced capabilities** for xorg/retroarch containers (no SYS_ADMIN for DRI)
- ✅ **Automatic NVIDIA device injection** by the runtime class
- ✅ **NVIDIA driver libraries** mounted automatically

### Mesa/AMD/Intel GPUs

When using AMD, Intel, or other GPUs that use Mesa drivers:

```yaml
gpu:
  type: "mesa"  # This is the default
```

**What happens:**
- ✅ **`/dev/dri` devices mounted** into containers that need GPU access
- ✅ **SYS_ADMIN capability** granted for DRI device access
- ✅ **Direct hardware access** via host path mounts

## Security Benefits

### NVIDIA Configuration (`gpu.type: "nvidia"`)
```yaml
# Containers that need GPU access (xorg, retroarch)
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  runAsGroup: 1000
  capabilities:
    # NO SYS_ADMIN needed for DRI - handled by runtime class
    drop:
      - ALL

# No host path mounts for GPU devices
volumeMounts:
  - name: audio-socket
    mountPath: /tmp/pulse
  - name: dev-uinput
    mountPath: /dev/uinput
  # NO /dev/dri mount
  - name: xorg
    mountPath: /tmp/.X11-unix
```

### Mesa Configuration (`gpu.type: "mesa"`)
```yaml
# Containers that need GPU access (xorg, retroarch)
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  runAsGroup: 1000
  capabilities:
    add:
      - SYS_ADMIN  # Required for DRI device access
    drop:
      - ALL

# Host path mount for DRI devices
volumeMounts:
  - name: audio-socket
    mountPath: /tmp/pulse
  - name: dev-uinput
    mountPath: /dev/uinput
  - name: dev-dri      # DRI devices mounted
    mountPath: /dev/dri
  - name: xorg
    mountPath: /tmp/.X11-unix
```

## Container-Specific Security

### Sunshine Container (Main)
- Always needs `SYS_ADMIN` for uinput device creation
- GPU type doesn't affect this container

### Steam Container
- Always needs `SYS_ADMIN` for bubblewrap sandboxing
- GPU access handled separately via volume mounts

### Xorg Container
- `SYS_ADMIN` only needed for Mesa/AMD/Intel (DRI access)
- NVIDIA setups don't need this capability

### RetroArch Container
- `SYS_ADMIN` only needed for Mesa/AMD/Intel (DRI access)
- NVIDIA setups don't need this capability

## Migration Guide

### From Previous Version (Always DRI Mount)
If you were using the previous version that always mounted `/dev/dri`:

**For NVIDIA users:**
```yaml
# Add this to your values.yaml
gpu:
  type: "nvidia"

# Ensure you have NVIDIA Device Plugin and RuntimeClass configured
```

**For AMD/Intel users:**
```yaml
# Add this to your values.yaml (or leave default)
gpu:
  type: "mesa"
```

### For New Installations

**NVIDIA Setup:**
1. Install NVIDIA Device Plugin
2. Configure NVIDIA RuntimeClass  
3. Set `gpu.type: "nvidia"` in values.yaml
4. Request GPU resources: `graphic_resources: {nvidia.com/gpu: 1}`

**AMD/Intel Setup:**
1. Set `gpu.type: "mesa"` in values.yaml (default)
2. Ensure `/dev/dri` exists on host nodes
3. Request appropriate resources if using device plugins

## Troubleshooting

### NVIDIA Issues
- **GPU not detected**: Check Device Plugin installation
- **Runtime errors**: Verify RuntimeClass configuration
- **Performance issues**: Ensure NVIDIA_VISIBLE_DEVICES is set correctly

### Mesa Issues  
- **DRI access denied**: Check if `/dev/dri` permissions allow group access
- **Missing devices**: Verify DRI devices exist on host nodes
- **Graphics acceleration disabled**: Check if containers have SYS_ADMIN capability

### General
- **Input not working**: Verify `/dev/uinput` exists and is accessible
- **Audio issues**: Check PulseAudio socket permissions
- **Container startup failures**: Review security context configurations