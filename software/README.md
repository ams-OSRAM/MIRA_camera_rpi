# Mira Image Sensors: Software
### [main page](/README.md)

There's a few options out there to get a Mira camera board working with your Raspberry Pi.


For Mira220, I've upstreamed a driver to the raspberry pi kernel github repo.
For Mira016 you need to use the ams repository.
Libcamera changes reside in a fork

* [Linux kernel with mira220 driver](https://github.com/raspberrypi/linux/tree/rpi-6.12.y) 
* [Libcamera with mira220 support](https://github.com/ams-OSRAM/libcamera)
* [Linux kernel with mira016 driver](https://github.com/ams-OSRAM/linux_mira_ams)

### Step 1: Prepare an SD card

Start from a fresh [Raspberry Pi 64bit image]([https://www.raspberrypi.com/software/operating-systems/#raspberry-pi-os-64-bit](https://www.raspberrypi.com/software/operating-systems/)).
Or, use Raspberry Pi imager to prepare an SD card for you. [recommended] [rpi imager](https://www.raspberrypi.com/software/)

OS customization is possible during flashing (users, wifi, ssh access ..)

Insert the SD card and boot the system.

### Step 2: Compile your kernel (no longer needed for mira220 when using the latest image from raspberry pi)
According to [this](https://www.raspberrypi.com/documentation/computers/linux_kernel.html#download-kernel-source
) guide. 

Use this branch: 

```
git clone --depth=1 --branch rpi-6.12.y https://github.com/ams-OSRAM/linux_mira_ams
```

The below guide explains how to [build the image on the pi itself](https://www.raspberrypi.com/documentation/computers/linux_kernel.html#natively-build-a-kernel), but you can also [cross-compile](https://www.raspberrypi.com/documentation/computers/linux_kernel.html#cross-compile-the-kernel) on another system (advanced users)

Prepare the build dependencies:

```
sudo apt install bc bison flex libssl-dev make
```

cd into the folder you just cloned and prepare the build configuration:

```
cd linux
KERNEL=kernel8
make bcm2711_defconfig
```
```
make -j6 Image.gz modules dtbs
```

```
sudo make -j6 modules_install
```

```
sudo cp /boot/firmware/$KERNEL.img /boot/firmware/$KERNEL-backup.img
sudo cp arch/arm64/boot/Image.gz /boot/firmware/$KERNEL.img
sudo cp arch/arm64/boot/dts/broadcom/*.dtb /boot/firmware/
sudo cp arch/arm64/boot/dts/overlays/*.dtb* /boot/firmware/overlays/
sudo cp arch/arm64/boot/dts/overlays/README /boot/firmware/overlays/
```

Then, in the file `/boot/firmware/config.txt` - add this line at the bottom:
```
kernel=kernel8.img
```
```
sudo reboot
```

### Step 3: install libcamera (still required)

Libcamera is the ISP framework, and cooperates with the driver.
Follow [this](https://www.raspberrypi.com/documentation/computers/camera_software.html#building-libcamera) guide.

```
sudo apt remove --purge rpicam-apps
sudo apt remove libcamera-apps
sudo apt remove libcamera-dev

sudo apt install -y libboost-dev
sudo apt install -y libgnutls28-dev openssl libtiff5-dev pybind11-dev
sudo apt install -y qtbase5-dev libqt5core5a libqt5gui5 libqt5widgets5
sudo apt install -y meson cmake
sudo apt install -y python3-yaml python3-ply
sudo apt install -y libglib2.0-dev libgstreamer-plugins-base1.0-dev

```

```
cd ~
git clone https://github.com/ams-OSRAM/libcamera.git
```

```
cd libcamera
```

```
meson setup build --buildtype=release -Dpipelines=rpi/vc4,rpi/pisp -Dipas=rpi/vc4,rpi/pisp -Dv4l2=enabled -Dgstreamer=enabled -Dtest=false -Dlc-compliance=disabled -Dcam=disabled -Dqcam=disabled -Ddocumentation=disabled -Dpycamera=enabled
```
```
ninja -C build
```
```
sudo ninja -C build install
```

### Step 4: rpicam-apps

These are some little helper applications to show a video stream.
```
sudo apt install -y cmake libboost-program-options-dev libdrm-dev libexif-dev
sudo apt install -y meson ninja-build
sudo apt install -y libpng-dev libepoxy-dev
```
```
cd ~
git clone https://github.com/raspberrypi/rpicam-apps.git
```


Navigate into the root directory of the repository:
```
cd rpicam-apps
```
For desktop-based operating systems like Raspberry Pi OS, configure the rpicam-apps build with the following meson command:
```
meson setup build -Denable_libav=disabled -Denable_drm=enabled -Denable_egl=enabled -Denable_qt=enabled -Denable_opencv=disabled -Denable_tflite=disabled -Denable_hailo=disabled
```

```
meson compile -C build
```

```
sudo meson install -C build
sudo ldconfig
```

Try it out!

```
rpicam-hello
```

### optional: Python bindings
To install python bindings, you will need picamera2.

create a virtual env:
```
cd ~
python -m venv .venv --system-site-packages
source .venv/bin/activate
git clone https://github.com/raspberrypi/picamera2
pip install ./picamera2
```

after this, you might need to link the libcamera.so files to within a python path.
make sure to match python version with the version on your system. In this case its 3.13.

```
sudo ln -sf /usr/local/lib/aarch64-linux-gnu/python3.13/site-packages/libcamera	 /usr/local/lib/python3.13/dist-packages/libcamera
```
