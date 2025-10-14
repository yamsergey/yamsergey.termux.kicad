# Building KiCad for Termux

This document describes how to build KiCad for Termux on Android devices.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Building Locally on Termux](#building-locally-on-termux)
- [Building with GitHub Actions](#building-with-github-actions)
- [Creating a Termux Package](#creating-a-termux-package)
- [Installation](#installation)
- [Troubleshooting](#troubleshooting)

## Prerequisites

### Hardware Requirements

- Android device with ARM or ARM64 architecture
- At least 4GB of RAM (8GB recommended)
- At least 10GB of free storage space

### Software Requirements

- Termux app installed from F-Droid or GitHub releases (NOT Google Play Store version)
- Terminal access to your Android device

## Building Locally on Termux

### Step 1: Install Termux

Download and install Termux from one of these sources:
- [F-Droid](https://f-droid.org/packages/com.termux/)
- [GitHub Releases](https://github.com/termux/termux-app/releases)

**Important:** Do NOT use the Google Play Store version as it is outdated and unsupported.

### Step 2: Update Termux

```bash
pkg update && pkg upgrade
```

### Step 3: Install Git

```bash
pkg install git
```

### Step 4: Clone the Repository

```bash
git clone https://github.com/yourusername/yamsergey.termux.kicad.git
cd yamsergey.termux.kicad
```

### Step 5: Run the Build Script

The build script will automatically install all dependencies and build KiCad:

```bash
./build-termux.sh
```

For a clean build (removes previous build directory):

```bash
./build-termux.sh --clean
```

To build and install in one step:

```bash
./build-termux.sh --clean --install
```

### Build Time

Building KiCad on Termux can take a significant amount of time depending on your device:
- High-end devices (8+ cores): 1-3 hours
- Mid-range devices (4-8 cores): 3-6 hours
- Low-end devices (2-4 cores): 6+ hours

**Tip:** Keep your device plugged in and prevent it from sleeping during the build process.

## Building with GitHub Actions

The repository includes a GitHub Actions workflow that automatically builds KiCad for Termux on both ARM and ARM64 architectures.

### Automatic Builds

Builds are triggered on:
- Push to the `master` branch
- Pull requests to the `master` branch
- Manual workflow dispatch

### Downloading Build Artifacts

1. Go to the [Actions tab](../../actions) in the GitHub repository
2. Click on the latest successful workflow run
3. Download the artifacts:
   - `kicad-termux-aarch64.deb` for ARM64 devices
   - `kicad-termux-arm.deb` for ARM devices

### Releases

When code is pushed to the `master` branch, a new release is automatically created with the built packages.

## Creating a Termux Package

After building KiCad locally, you can create a `.deb` package for easy installation:

```bash
./build-termux-package.sh
```

This will create a file named `kicad_<version>_<architecture>.deb` in the current directory.

## Installation

### Installing from a .deb Package

```bash
# Install the package
apt install ./kicad_*.deb

# Or use dpkg directly
dpkg -i kicad_*.deb
```

### Installing from Source Build

If you built KiCad without creating a package, you can install it directly:

```bash
cd build-termux
ninja install
```

### Running KiCad

After installation, you can run KiCad from the command line:

```bash
kicad
```

Or run individual tools:

```bash
eeschema      # Schematic editor
pcbnew        # PCB editor
gerbview      # Gerber viewer
```

## Configuration

### Display Server

KiCad requires a display server to run. In Termux, you need to set up one of these:

#### Option 1: VNC Server (Recommended)

```bash
# Install VNC server
pkg install tigervnc

# Start VNC server
vncserver -localhost

# In a new session, set DISPLAY
export DISPLAY=:1

# Run KiCad
kicad
```

Then connect to the VNC server using a VNC client app on your Android device.

#### Option 2: X11 with Termux:X11

```bash
# Install Termux:X11 app from GitHub
# Then in Termux:
pkg install x11-repo
pkg install termux-x11-nightly

# Start X11 server (in Termux:X11 app)
# Then in Termux terminal:
export DISPLAY=:0
kicad
```

### Environment Variables

You may need to set these environment variables:

```bash
export KICAD_CONFIG_HOME="$HOME/.config/kicad"
export KICAD_DATA_PATH="$PREFIX/share/kicad"
```

Add them to your `~/.bashrc` or `~/.zshrc` for persistence.

## Troubleshooting

### Build Failures

#### Out of Memory

If the build fails with memory-related errors:

1. Close other apps to free up RAM
2. Reduce parallel jobs: Edit `build-termux.sh` and change `ninja -j$(nproc)` to `ninja -j2`
3. Enable swap space:

```bash
# Create swap file (requires root)
dd if=/dev/zero of=/sdcard/swapfile bs=1M count=2048
chmod 600 /sdcard/swapfile
mkswap /sdcard/swapfile
swapon /sdcard/swapfile
```

#### Missing Dependencies

If dependencies are missing:

```bash
pkg update
pkg upgrade
```

Then try the build again.

### Runtime Issues

#### Cannot Open Display

Make sure you have a display server running (VNC or X11) and the `DISPLAY` environment variable is set correctly.

#### Segmentation Fault

This may be caused by:
- Incompatible libraries: Try rebuilding with `--clean`
- Out of memory: Close other apps
- Corrupted installation: Reinstall the package

#### Library Not Found

If you get errors about missing libraries:

```bash
# Check library dependencies
ldd $PREFIX/bin/kicad

# Install missing packages
pkg search <library-name>
pkg install <package-name>
```

## Known Limitations

1. **Performance**: KiCad may run slower on Termux compared to native Linux systems due to Android's overhead
2. **3D Viewer**: May have limited functionality depending on device GPU capabilities
3. **Python Scripting**: wxPython support is disabled by default due to compatibility issues
4. **Touch Input**: Touch gestures may not work as expected; using a USB mouse is recommended

## Contributing

If you encounter issues or have improvements for the Termux build:

1. Check existing issues on GitHub
2. Create a new issue with:
   - Device model and Android version
   - Termux version
   - Build log or error messages
   - Steps to reproduce

## Additional Resources

- [KiCad Official Documentation](https://docs.kicad.org/)
- [Termux Wiki](https://wiki.termux.com/)
- [KiCad Developer Documentation](https://dev-docs.kicad.org/)
- [KiCad Forums](https://forum.kicad.info/)

## License

KiCad is released under the GNU General Public License v3.0. See the [LICENSE](LICENSE) file for details.
