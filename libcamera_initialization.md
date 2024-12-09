# Setting Up Raspberry Pi Camera Module on Ubuntu 24.04 with RPi 5

This guide covers the installation and setup process for using a Raspberry Pi Camera Module with Ubuntu 24.04 on a Raspberry Pi 5.

## Prerequisites
- Raspberry Pi 5
- Raspberry Pi Camera Module (compatible with RPi 5)
- Ubuntu 24.04 installed
- Active internet connection
- User with sudo privileges

## Hardware Setup
1. Ensure the Raspberry Pi is powered off
2. Connect the camera module to the camera port using the ribbon cable
   - Ensure the cable is properly oriented (blue side facing away from the Ethernet port)
   - Make sure the cable is fully inserted and the connectors are properly secured

## Software Installation Steps

### 1. Install Required System Packages
```bash
sudo apt update
sudo apt install -y libcamera0.2 libcamera-dev python3-libcamera libcamera-tools
```

### 2. Setting Up Conda Environment
```bash
# Download and install Miniforge
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-aarch64.sh
chmod +x Miniforge3-Linux-aarch64.sh
./Miniforge3-Linux-aarch64.sh

# Initialize conda (if you said "no" during installation)
~/miniforge3/bin/conda init
source ~/.bashrc

# Create conda environment
conda create -n camera_env python=3.12
conda activate camera_env
```

### 3. Install Camera Dependencies
```bash
# Install system dependencies
sudo apt install -y cmake meson ninja-build libdrm-dev python3-dev

# Clone and build kmsxx
git clone https://github.com/tomba/kmsxx.git
cd kmsxx
meson setup build
cd build
ninja
sudo ninja install
sudo ldconfig
```

### 4. Set Up User Permissions
```bash
# Add user to required groups
sudo usermod -a -G video,input,render $USER

# Create udev rules for camera access
sudo bash -c 'cat > /etc/udev/rules.d/99-camera.rules << EOF
SUBSYSTEM=="video4linux", GROUP="video", MODE="0666"
EOF'

# Reload udev rules
sudo udevadm control --reload-rules
sudo udevadm trigger
```

## Testing the Camera

### Basic Tests
1. Test image capture:
```bash
libcamera-still -o test.jpg
```

2. Test live preview:
```bash
libcamera-hello --qt-preview
```

## Common Issues and Solutions

### 1. Camera Not Detected
**Symptoms:**
- "Failed to detect camera" error
- Empty device list when running `v4l2-ctl --list-devices`

**Solutions:**
- Check physical connection of ribbon cable
- Verify cable orientation
- Try reseating the cable
- Restart the Raspberry Pi

### 2. Permission Issues
**Symptoms:**
- "Permission denied" when accessing camera
- Can't open camera device

**Solutions:**
- Ensure user is in video group: `groups $USER`
- Log out and log back in after group changes
- Check udev rules are properly set
- Verify device permissions: `ls -l /dev/video*`

### 3. Preview Window Issues
**Symptoms:**
- "Preview window unavailable" error
- No display output

**Solutions:**
- Ensure X11 is installed and running
- Try different preview options:
  ```bash
  libcamera-hello --qt-preview
  # or
  libcamera-hello --display=0
  ```
- Install additional display dependencies:
  ```bash
  sudo apt install -y gstreamer1.0-x
  ```

### 4. Module Import Errors
**Symptoms:**
- "No module named 'libcamera'" error
- "No module named 'pykms'" error

**Solutions:**
- Verify conda environment is activated
- Reinstall libraries:
  ```bash
  pip uninstall picamera2
  pip install picamera2
  ```
- Check symbolic links:
  ```bash
  SITE_PACKAGES=$(python -c "import site; print(site.getsitepackages()[0])")
  sudo ln -sf /usr/lib/python3/dist-packages/libcamera* $SITE_PACKAGES/
  ```

## Additional Notes
- The camera preview may not work over SSH without X11 forwarding
- Some camera features might require additional configuration depending on your specific camera model
- System reboot might be required after initial setup
- Keep track of error messages as they provide valuable debugging information

