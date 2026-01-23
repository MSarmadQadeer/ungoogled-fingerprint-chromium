# Profile Grid Fingerprint Spoofing Features

This fork of ungoogled-chromium adds comprehensive browser fingerprint spoofing capabilities for the Profile Grid anti-detect browser.

## Feature Summary

| Feature | Command-Line Switch | Status |
|---------|-------------------|--------|
| Canvas Noise | `--fingerprinting-canvas-image-data-noise` | ✅ From ungoogled |
| ClientRects Noise | `--fingerprinting-client-rects-noise` | ✅ From ungoogled |
| MeasureText Noise | `--fingerprinting-canvas-measuretext-noise` | ✅ From ungoogled |
| Hardware Concurrency | `--fingerprint-hardware-concurrency=N` | ✅ New |
| Device Memory | `--fingerprint-device-memory=N` | ✅ New |
| Screen Resolution | `--fingerprint-screen-width/height=N` | ✅ New |
| Device Pixel Ratio | `--fingerprint-device-pixel-ratio=N` | ✅ New |
| WebGL Vendor | `--fingerprint-webgl-vendor=STRING` | ✅ New |
| WebGL Renderer | `--fingerprint-webgl-renderer=STRING` | ✅ New |
| User Agent | `--fingerprint-user-agent=STRING` | ✅ New |
| Platform | `--fingerprint-platform=STRING` | ✅ New |
| Audio Noise | `--fingerprinting-audio-noise` | ✅ New |
| Audio Seed | `--fingerprint-audio-seed=N` | ✅ New |
| Battery Level | `--fingerprint-battery-level=N` | ✅ New |
| Battery Charging | `--fingerprint-battery-charging=BOOL` | ✅ New |
| Disable Battery API | `--disable-battery-api` | ✅ New |
| Network Type | `--fingerprint-network-type=TYPE` | ✅ New |
| Network Effective Type | `--fingerprint-network-effective-type=TYPE` | ✅ New |
| Disable Network Info | `--disable-network-info-api` | ✅ New |
| Media Devices | `--fingerprint-media-devices=CONFIG` | ✅ New |
| Speech Voices | `--fingerprint-speech-voices=PLATFORM` | ✅ New |
| Disable Speech | `--disable-speech-synthesis` | ✅ New |
| Disable Gamepad | `--disable-gamepad-api` | ✅ New |
| Disable Bluetooth | `--disable-bluetooth-api` | ✅ New |
| Disable USB | `--disable-usb-api` | ✅ New |

## Patch Files

All fingerprint patches are in `patches/extra/fingerprint/`:

| Patch | Description |
|-------|-------------|
| `add-fingerprint-switches.patch` | Adds all new command-line switch definitions |
| `fingerprint-battery-api-spoofing.patch` | Battery API spoofing with configurable values |
| `fingerprint-media-devices-spoofing.patch` | Media devices enumeration spoofing |
| `fingerprint-speech-synthesis-spoofing.patch` | Speech synthesis voices spoofing |
| `fingerprint-network-info-spoofing.patch` | Network Information API spoofing |
| `fingerprint-webgl-spoofing.patch` | WebGL vendor/renderer spoofing |
| `fingerprint-audio-noise.patch` | Audio context fingerprint noise |
| `fingerprint-block-apis.patch` | Block Gamepad, Bluetooth, USB, Sensor APIs |
| `fingerprint-hardware-spoofing.patch` | Hardware/Navigator property spoofing |

## Usage Examples

### Desktop Profile (Windows)
```bash
chrome.exe \
  --fingerprinting-client-rects-noise \
  --fingerprinting-canvas-image-data-noise \
  --fingerprinting-audio-noise \
  --fingerprint-hardware-concurrency=8 \
  --fingerprint-device-memory=8 \
  --fingerprint-webgl-vendor="Intel Inc." \
  --fingerprint-webgl-renderer="Intel(R) UHD Graphics 620" \
  --fingerprint-screen-width=1920 \
  --fingerprint-screen-height=1080 \
  --fingerprint-platform="Win32" \
  --fingerprint-network-effective-type=4g \
  --fingerprint-speech-voices=windows \
  --disable-battery-api \
  --disable-gamepad-api \
  --disable-bluetooth-api \
  --user-data-dir=/path/to/profile
```

### Mac Profile
```bash
chrome \
  --fingerprinting-client-rects-noise \
  --fingerprinting-canvas-image-data-noise \
  --fingerprint-hardware-concurrency=8 \
  --fingerprint-device-memory=8 \
  --fingerprint-webgl-vendor="Apple Inc." \
  --fingerprint-webgl-renderer="Apple M1" \
  --fingerprint-screen-width=2560 \
  --fingerprint-screen-height=1440 \
  --fingerprint-device-pixel-ratio=2.0 \
  --fingerprint-platform="MacIntel" \
  --fingerprint-speech-voices=macos \
  --disable-battery-api \
  --user-data-dir=/path/to/profile
```

## Common WebGL Configurations

### Intel (Windows)
```
--fingerprint-webgl-vendor="Intel Inc."
--fingerprint-webgl-renderer="Intel(R) UHD Graphics 620"
```

### Intel (Windows - newer)
```
--fingerprint-webgl-vendor="Intel Inc."
--fingerprint-webgl-renderer="Intel(R) Iris(R) Xe Graphics"
```

### NVIDIA (Windows)
```
--fingerprint-webgl-vendor="NVIDIA Corporation"
--fingerprint-webgl-renderer="NVIDIA GeForce GTX 1060"
```

### AMD (Windows)
```
--fingerprint-webgl-vendor="ATI Technologies Inc."
--fingerprint-webgl-renderer="AMD Radeon RX 580"
```

### Apple (Mac - Intel)
```
--fingerprint-webgl-vendor="Intel Inc."
--fingerprint-webgl-renderer="Intel Iris Pro OpenGL Engine"
```

### Apple (Mac - M1/M2)
```
--fingerprint-webgl-vendor="Apple Inc."
--fingerprint-webgl-renderer="Apple M1"
```

## Common Screen Resolutions

| Resolution | Width | Height | Pixel Ratio | Use Case |
|------------|-------|--------|-------------|----------|
| 1080p | 1920 | 1080 | 1.0 | Standard desktop |
| 1440p | 2560 | 1440 | 1.0 | High-end desktop |
| 4K | 3840 | 2160 | 1.5 | 4K monitor |
| MacBook Pro 13" | 2560 | 1600 | 2.0 | Retina display |
| MacBook Pro 16" | 3456 | 2234 | 2.0 | Retina display |
| iMac 24" | 4480 | 2520 | 2.0 | Retina 4.5K |

## Valid Device Memory Values

The `deviceMemory` API only returns specific values:
- 0.25, 0.5, 1, 2, 4, 8 GB

Use one of these values with `--fingerprint-device-memory`.

## Network Types

### --fingerprint-network-type
- `wifi` (most common)
- `ethernet`
- `cellular`
- `bluetooth`
- `none`

### --fingerprint-network-effective-type
- `4g` (fast connection, most common)
- `3g` (moderate)
- `2g` (slow)
- `slow-2g` (very slow)

## Speech Voice Platforms

### --fingerprint-speech-voices options:
- `windows` - Windows default voices (David, Zira, Mark)
- `macos` - macOS voices (Samantha, Alex, Victoria)
- `linux` - Linux eSpeak voices
- `empty` - No voices (Firefox-like)

## Integration Notes

These patches are designed to work with Profile Grid's launcher, which:

1. Generates consistent fingerprints per profile
2. Passes fingerprint values via command-line switches
3. Can combine native switches with CDP injection for maximum coverage

## Comparison with Competitors

| Feature | Profile Grid + This Fork | GoLogin | MultiLogin |
|---------|-------------------------|---------|------------|
| Engine-level spoofing | ✅ Yes | ✅ Yes | ✅ Yes |
| Open source | ✅ Yes | ❌ No | ❌ No |
| Self-hosted | ✅ Yes | ❌ No | ❌ No |
| Customizable | ✅ Full control | ❌ Limited | ❌ Limited |
| Battery API | ✅ Yes | ✅ Yes | ✅ Yes |
| Media Devices | ✅ Yes | ✅ Yes | ✅ Yes |
| Speech Synthesis | ✅ Yes | ✅ Yes | ✅ Yes |
| Network Info | ✅ Yes | ✅ Yes | ✅ Yes |
| Gamepad Block | ✅ Yes | ✅ Yes | ✅ Yes |
| Bluetooth Block | ✅ Yes | ✅ Yes | ✅ Yes |

## Contributing

See [BUILD.md](BUILD.md) for build instructions.

1. Fork this repository
2. Create your feature branch
3. Add/modify patches in `patches/extra/fingerprint/`
4. Update `patches/series` if adding new patches
5. Test the build
6. Submit a pull request
