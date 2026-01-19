# Install Android SDK on Linux (Manual Setup)

A lightweight, well-documented guide and script to install the Android SDK on Linux (Ubuntu/Debian) without the overhead of Android Studio. Ideal for CI/CD pipelines, Flutter, React Native, or minimal development environments.

## Table of Contents
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Quick Start (Automated)](#quick-start-automated-installation)
- [Detailed Manual Installation](#detailed-manual-installation)
- [Verification](#verification)
- [Emulator Setup (AVD)](#emulator-setup-avd)
- [Managing the SDK](#managing-the-sdk)
- [Troubleshooting](#troubleshooting)
- [License](#license)

## Features

- **Automated Script**: Quick setup with `install.sh`.
- **Manual Guide**: Detailed step-by-step instructions.
- **Troubleshooting**: Solutions for common Java and permission issues.
- **Emulator Support**: Guide to creating and running AVDs.

## Prerequisites

- Linux (Ubuntu/Debian recommended).
- `curl`, `unzip`, and `sudo` privileges.

## Quick Start (Automated Installation)

Clone this repository and run the installation script:

```bash
chmod +x install.sh
./install.sh
```

The script will:
1. Install OpenJDK 17 (if missing).
2. Download Android Command Line Tools.
3. Install Platform-tools, Build-tools, and a standard Android Platform (API level 34).
4. Configure environment variables in your `.bashrc` or `.zshrc`.

After running the script, remember to reload your profile:
```bash
source ~/.bashrc  # or ~/.zshrc
```

## Detailed Manual Installation

Follow these steps if you want to understand the underlying process or customize your setup.

### 1. Install Java Development Kit (JDK)
The Android SDK tools are written in Java and require a JDK to run. We recommend OpenJDK 17.

```bash
# Update package list and install OpenJDK 17
sudo apt update
sudo apt install openjdk-17-jdk
```

**Verify**: Run `java -version` to ensure it is correctly installed.

### 2. Prepare the Workspace
Android expects a specific directory structure for its tools to work correctly without errors like "could not determine SDK root".

```bash
# Create the base directory for your SDK
mkdir -p ~/android-sdk/cmdline-tools
```

### 3. Download and Configure Command Line Tools
The "Command Line Tools" are the core binaries (`sdkmanager`, `avdmanager`, etc.) used to manage the rest of the SDK.

1.  Download the Linux zip from [Android Studio Downloads](https://developer.android.com/studio#command-line-tools-only).
2.  Unzip it into the folder we created:

```bash
# Unzip the downloaded tools
unzip commandlinetools-linux-*.zip -d ~/android-sdk/cmdline-tools

# CRITICAL: The sdkmanager expects to be inside a folder named 'latest'
# to correctly identify the SDK root.
mv ~/android-sdk/cmdline-tools/cmdline-tools ~/android-sdk/cmdline-tools/latest
```

### 4. Set Up Environment Variables
To use tools like `adb` or `sdkmanager` from any terminal window, you need to tell your shell where they are located.

Add these lines to your `~/.bashrc` (or `~/.zshrc`):

```bash
# Define the root of your Android SDK
export ANDROID_HOME=$HOME/android-sdk

# Add the various tool directories to your PATH
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin  # For sdkmanager, avdmanager
export PATH=$PATH:$ANDROID_HOME/platform-tools             # For adb, fastboot
export PATH=$PATH:$ANDROID_HOME/emulator                   # For running emulators
```

**Apply changes**: `source ~/.bashrc`

### 5. Install SDK Components
Now that the management tools are set up, use `sdkmanager` to download the actual Android platform files and build tools.

```bash
# Install platform tools, a specific API level, and build tools
sdkmanager "platform-tools" "platforms;android-34" "build-tools;34.0.0"
```

*Note: You will be prompted to accept licenses. Type `y` and press Enter.*

### 6. Accept All Licenses (Optional)
If you are setting this up for CI/CD or just want to get it over with:
```bash
yes | sdkmanager --licenses
```

## Verification

After installation, verify that the tools are accessible and the SDK is correctly recognized.

1.  **Check ADB**:
    ```bash
    adb version
    ```
    *Should output: Android Debug Bridge version 1.0.xx*

2.  **Check SDK Manager**:
    ```bash
    sdkmanager --list_installed
    ```
    *Should list `platform-tools`, `platforms;android-34`, etc.*

## Emulator Setup (AVD)

To run your apps on a virtual device, you need to download a system image and create an Android Virtual Device (AVD).

### 1. Download a System Image
System images are the OS versions the emulator runs. For most modern Linux systems, `x86_64` is the fastest.

```bash
# Download the system image for Android 34
sdkmanager "system-images;android-34;default;x86_64"
```

### 2. Create the Virtual Device
Use `avdmanager` to create a device named "testDevice".

```bash
# Create the AVD
avdmanager create avd -n testDevice -k "system-images;android-34;default;x86_64"
```
*Note: If asked about a custom hardware profile, you can usually press Enter to use the default.*

### 3. Launch the Emulator
You can now list your devices and start the emulator.

```bash
# List available AVDs
emulator -list-avds

# Start the emulator
emulator @testDevice
```

## Managing the SDK

### Updating Components
To update all installed components to their latest versions:
```bash
sdkmanager --update
```

### Installing New Versions
If you need a different API level (e.g., API 35):
```bash
sdkmanager "platforms;android-35" "build-tools;35.0.0"
```

### Listing Available Packages
To see everything you can install:
```bash
sdkmanager --list
```

## Troubleshooting

### `NoClassDefFoundError`
If you see `java.lang.NoClassDefFoundError: javax/xml/bind/annotation/XmlSchema`, it usually means you are using an incompatible Java version or a deprecated sdkmanager. Ensure you are using the manual installation method described here.

### Deprecated Java Version
If your `java -version` and `javac -version` mismatch or point to an old version:
```bash
sudo update-alternatives --config java
sudo update-alternatives --config javac
```

### Emulator Issues
If the emulator fails with "missing hardware acceleration" on a Virtual Machine:
```bash
emulator @yourDevice --no-accel
```

## License

This project is licensed under the MIT License.
