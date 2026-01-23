# Building Ungoogled-Fingerprint-Chromium

This guide covers building your custom Chromium fork with fingerprint spoofing patches for the Profile Grid anti-detect browser.

## Prerequisites

### System Requirements
- **Windows 10/11** (64-bit)
- **At least 16GB RAM** (32GB recommended)
- **100GB+ free disk space** (SSD strongly recommended)
- **Fast internet connection** (source is ~35GB)

### Required Software

1. **Visual Studio 2022** (Community edition is fine)
   - Install "Desktop development with C++" workload
   - Install "Windows 10 SDK" (10.0.22621.0 or later)
   - Install "C++ ATL for latest build tools"
   - Install "C++ MFC for latest build tools"

2. **Python 3.11+**
   ```powershell
   winget install Python.Python.3.11
   ```

3. **Git**
   ```powershell
   winget install Git.Git
   ```

4. **depot_tools**
   ```powershell
   cd C:\
   git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
   ```
   Add `C:\depot_tools` to your PATH (at the beginning!)

## Build Steps

### Step 1: Fetch Chromium Source

```powershell
mkdir C:\chromium
cd C:\chromium

# Configure git
git config --global core.autocrlf false
git config --global core.filemode false
git config --global branch.autosetuprebase always

# Fetch chromium (this takes 1-2 hours)
fetch chromium
```

### Step 2: Checkout Matching Chromium Version

Check the version in `chromium_version.txt` (currently 144.0.7559.96):

```powershell
cd C:\chromium\src
git checkout tags/144.0.7559.96

# Sync dependencies
gclient sync --with_branch_heads --with_tags -D
```

### Step 3: Apply Ungoogled-Fingerprint Patches

```powershell
# Clone this repo if you haven't
cd C:\
git clone https://github.com/MSarmadQadeer/ungoogled-fingerprint-chromium.git

# Run the patching script
cd C:\chromium\src
python C:\ungoogled-fingerprint-chromium\utils\patches.py apply C:\ungoogled-fingerprint-chromium\patches C:\chromium\src
```

### Step 4: Apply Domain Substitutions (Optional but Recommended)

```powershell
python C:\ungoogled-fingerprint-chromium\utils\domain_substitution.py apply \
  -r C:\ungoogled-fingerprint-chromium\domain_regex.list \
  -f C:\ungoogled-fingerprint-chromium\domain_substitution.list \
  C:\chromium\src
```

### Step 5: Prune Binaries (Optional)

```powershell
python C:\ungoogled-fingerprint-chromium\utils\prune_binaries.py \
  C:\chromium\src \
  C:\ungoogled-fingerprint-chromium\pruning.list
```

### Step 6: Configure Build

Create the build directory and configuration:

```powershell
cd C:\chromium\src
gn gen out\Release --args="
  is_official_build=true
  is_debug=false
  enable_nacl=false
  target_cpu=\"x64\"
  proprietary_codecs=true
  ffmpeg_branding=\"Chrome\"
  enable_widevine=false
  google_api_key=\"\"
  google_default_client_id=\"\"
  google_default_client_secret=\"\"
  chrome_pgo_phase=0
  is_cfi=false
  is_component_build=false
  blink_symbol_level=0
  symbol_level=0
  treat_warnings_as_errors=false
"
```

### Step 7: Build

```powershell
# Build Chrome (takes 4-8+ hours depending on hardware)
autoninja -C out\Release chrome

# Or build installer
autoninja -C out\Release mini_installer
```

### Step 8: Package the Build

The built browser will be in `C:\chromium\src\out\Release\`.

Key files:
- `chrome.exe` - Main browser executable
- `chrome.dll` - Core browser library
- `*.pak` files - Resources

## Fingerprint Command-Line Switches

After building, your browser supports these switches:

### Core Fingerprinting (from Bromite/ungoogled)
- `--fingerprinting-client-rects-noise` - ClientRects noise
- `--fingerprinting-canvas-measuretext-noise` - Canvas measureText noise
- `--fingerprinting-canvas-image-data-noise` - Canvas image data noise

### Profile Grid Additions

#### Hardware Spoofing
- `--fingerprint-hardware-concurrency=8` - CPU cores
- `--fingerprint-device-memory=8` - RAM in GB
- `--fingerprint-screen-width=1920` - Screen width
- `--fingerprint-screen-height=1080` - Screen height
- `--fingerprint-device-pixel-ratio=1.0` - Pixel ratio

#### WebGL Spoofing
- `--fingerprint-webgl-vendor="Intel Inc."` - GPU vendor
- `--fingerprint-webgl-renderer="Intel(R) UHD Graphics 620"` - GPU renderer

#### Audio Fingerprinting
- `--fingerprinting-audio-noise` - Enable audio noise
- `--fingerprint-audio-seed=12345` - Deterministic seed

#### User Agent / Platform
- `--fingerprint-user-agent="Mozilla/5.0..."` - Custom UA
- `--fingerprint-platform="Win32"` - Platform string

#### Battery API
- `--fingerprint-battery-level=0.85` - Battery level (0-1)
- `--fingerprint-battery-charging=true` - Charging status
- `--disable-battery-api` - Disable completely

#### Network Information
- `--fingerprint-network-type=wifi` - Connection type
- `--fingerprint-network-effective-type=4g` - Effective type
- `--disable-network-info-api` - Disable completely

#### Media Devices
- `--fingerprint-media-devices=audio:2,video:1,audiooutput:1` - Device counts

#### Speech Synthesis
- `--fingerprint-speech-voices=windows` - Platform voices (windows/macos/linux/empty)
- `--disable-speech-synthesis` - Disable completely

#### API Blocking
- `--disable-gamepad-api` - Block Gamepad API
- `--disable-bluetooth-api` - Block Web Bluetooth
- `--disable-usb-api` - Block WebUSB

## Example Usage

```powershell
chrome.exe ^
  --fingerprinting-client-rects-noise ^
  --fingerprinting-canvas-measuretext-noise ^
  --fingerprinting-canvas-image-data-noise ^
  --fingerprinting-audio-noise ^
  --fingerprint-hardware-concurrency=8 ^
  --fingerprint-device-memory=8 ^
  --fingerprint-webgl-vendor="Intel Inc." ^
  --fingerprint-webgl-renderer="Intel(R) UHD Graphics 620" ^
  --fingerprint-screen-width=1920 ^
  --fingerprint-screen-height=1080 ^
  --fingerprint-network-effective-type=4g ^
  --disable-battery-api ^
  --disable-gamepad-api ^
  --disable-bluetooth-api ^
  --user-data-dir=C:\Profiles\Profile1
```

## Updating to New Chromium Version

1. Check the latest ungoogled-chromium release
2. Update `chromium_version.txt` with the new version
3. Fetch and checkout the new Chromium version
4. Fix any patch conflicts
5. Rebuild

## Troubleshooting

### "Patch failed to apply"
- Patches may need updating for new Chromium versions
- Check the patch file and manually resolve conflicts

### "Build errors"
- Ensure all Visual Studio components are installed
- Check you're using the correct Windows SDK version
- Try `gclient sync` again

### "Out of memory during build"
- Close other applications
- Add more RAM or use a smaller number of parallel jobs:
  ```
  autoninja -C out\Release -j 4 chrome
  ```

## Integration with Profile Grid

After building, copy the browser to Profile Grid's Chromium installation path:

```
%USERPROFILE%\AppData\Local\ProfileGrid\chromium\
```

The Profile Grid launcher will automatically detect and use your custom build.

## Contributing

1. Fork this repository
2. Make your changes to patches
3. Test the build
4. Submit a pull request

## License

BSD-3-Clause (same as Chromium)
