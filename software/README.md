# Mira Image Sensors: Software Guide

### [← Back to Main Page](/README.md)

---

## Overview

Mira image sensors feature a standard **MIPI CSI-2 camera interface**, ensuring broad compatibility with various platforms:

- **Single Board Computers**: Raspberry Pi, NXP i.MX
- **Microprocessors**: Qualcomm, STM32
- **Microcontrollers**: ESP32-P4

This guide focuses on Raspberry Pi setup, though the V4L2 Linux driver is compatible with most Linux devices.

---

## Requirements

To operate a Mira camera board with your Raspberry Pi, you'll need:

1. **Mira V4L2 Driver** - Kernel module for hardware communication
2. **Libcamera Framework** - ISP (Image Signal Processor) framework with Mira support

---

## Setup Instructions

### Step 1: Prepare Your SD Card

1. Download the latest [Raspberry Pi OS 64-bit](https://www.raspberrypi.com/software/operating-systems/)
2. Use [Raspberry Pi Imager](https://www.raspberrypi.com/software/) to flash your SD card (**recommended**)
   - Configure OS settings during flashing:
     - User credentials
     - WiFi network
     - SSH access
     - USB gadget mode (if needed)
3. Insert the SD card and boot your Raspberry Pi

---

### Step 2: Install the Kernel Module and Device Tree

Choose your sensor model:

- **Mira220**: [Linux Driver & Installation Guide](https://github.com/ams-OSRAM/mira220_v4l2_driver)
- **Mira050**: [Linux Driver & Installation Guide](https://github.com/ams-OSRAM/mira050_v4l2_driver)
- **Mira016**: [Linux Driver & Installation Guide](https://github.com/ams-OSRAM/mira016_v4l2_driver)


---

### Step 3: Build Libcamera

Libcamera is the ISP framework that works alongside the V4L2 driver to process camera data.

#### Install Libcam Dependencies

```bash
# Remove existing camera packages
sudo apt remove --purge rpicam-apps libcamera-apps libcamera-dev

# Install build dependencies
sudo apt install -y libboost-dev libgnutls28-dev openssl libtiff5-dev pybind11-dev
sudo apt install -y qtbase5-dev libqt5core5a libqt5gui5 libqt5widgets5
sudo apt install -y meson cmake python3-yaml python3-ply
sudo apt install -y libglib2.0-dev libgstreamer-plugins-base1.0-dev
```

#### Clone and Build Libcamera

```bash
# Clone the Mira-enabled libcamera fork
cd ~
git clone https://github.com/ams-OSRAM/libcamera.git
cd libcamera

# Configure the build
meson setup build --buildtype=release \
  -Dpipelines=rpi/vc4,rpi/pisp \
  -Dipas=rpi/vc4,rpi/pisp \
  -Dv4l2=enabled \
  -Dgstreamer=enabled \
  -Dtest=false \
  -Dlc-compliance=disabled \
  -Dcam=disabled \
  -Dqcam=disabled \
  -Ddocumentation=disabled \
  -Dpycamera=enabled

# Compile and install
ninja -C build
sudo ninja -C build install
```

---


#### Test Your Setup

```bash
rpicam-hello
```

If successful, you should see a live camera preview on your display.

---

## Optional: Python Bindings (not needed)

To use your Mira camera with Python, install **picamera2** in a virtual environment.

### Setup Virtual Environment

```bash
cd ~
python -m venv .venv --system-site-packages
source .venv/bin/activate
```

### Test Python Integration

```python
from picamera2 import Picamera2

picam2 = Picamera2()
picam2.start_preview()
picam2.start()
```

---

## Troubleshooting

### Camera Not Detected

- Verify the ribbon cable is properly seated in both the camera and Raspberry Pi connectors
- Check that the camera interface is enabled: `sudo raspi-config` → **Interface Options** → **Camera**
- Confirm the driver loaded correctly: `dmesg | grep mira`

### Build Errors

- Ensure all dependencies are installed
- Try updating your system first: `sudo apt update && sudo apt upgrade`
- Check for adequate disk space: `df -h`

### Python Import Errors

- Verify the Python version in the symlink matches your system Python
- Ensure the virtual environment has `--system-site-packages` enabled
- Check libcamera installation: `ldconfig -p | grep libcamera`

---

## Additional Resources

- [Raspberry Pi Camera Documentation](https://www.raspberrypi.com/documentation/accessories/camera.html)
- [Libcamera Documentation](https://libcamera.org/docs.html)
- [V4L2 Documentation](https://www.kernel.org/doc/html/latest/userspace-api/media/v4l/v4l2.html)

---

## Support

For issues specific to Mira sensors, please visit:
- [ams-OSRAM GitHub Organization](https://github.com/ams-OSRAM)
- Check the respective driver repository for your sensor model

---

*Last updated: March 2026*
