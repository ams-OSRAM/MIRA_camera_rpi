# Mira Image Sensors: Software
### [main page](/README.md)

There's a few options out there to get a Mira camera board working with your Raspberry Pi.


## Option 1: Pre-built image with Mira drivers

Get a pre-built image from the [ams website](https://ams-osram.com/products/boards-kits-accessories/kits/ams-mira-evm-sn-raspberry-evaluation-kit
)

This is pre-built image is bullseye OS (not for pi5) and works with all Mira image sensors. (220,050,016)
Simply download and flash to SD with Raspberry Pi Imager.

## Option 2: Customize your own image.
For Mira220, I've upstreamed a driver to the raspberr pi kernel github repo.
Libcamera changes reside in a fork
Other Mira sensor drivers are not yet public, but available on request.

* [Linux kernel with mira220 driver](https://github.com/raspberrypi/linux/tree/rpi-6.12.y) 
* [Libcamera with mira220 support](https://github.com/ams-OSRAM/libcamera)

### Step 1: Prepare an SD card

Start from a fresh [Raspberry Pi 64bit image]([https://www.raspberrypi.com/software/operating-systems/#raspberry-pi-os-64-bit](https://www.raspberrypi.com/software/operating-systems/)).
Or, use Raspberry Pi imager to prepare an SD card for you. [recommended] [rpi imager](https://www.raspberrypi.com/software/)

OS customization is possible during flashing (users, wifi, ssh access ..)

Insert the SD card and boot the system.

### Step 2: Compile your kernel
According to [this](https://www.raspberrypi.com/documentation/computers/linux_kernel.html#download-kernel-source
) guide. 

Use this branch: 

```
git clone --depth=1 --branch rpi-6.12.y https://github.com/raspberrypi/linux
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

### Step 3: install libcamera

Libcamera is the ISP framework, and cooperates with the driver.
Follow [this](https://www.raspberrypi.com/documentation/computers/camera_software.html#building-libcamera) guide.

```
sudo apt remove --purge rpicam-apps
sudo apt install -y libcamera-dev libepoxy-dev libjpeg-dev libtiff5-dev libpng-dev
sudo apt install libavcodec-dev libavdevice-dev libavformat-dev libswresample-dev

sudo apt install -y libboost-dev
sudo apt install -y libgnutls28-dev openssl libtiff5-dev pybind11-dev
sudo apt install -y qtbase5-dev libqt5core5a libqt5gui5 libqt5widgets5
sudo apt install -y meson cmake
sudo apt install -y python3-yaml python3-ply
sudo apt install -y libglib2.0-dev libgstreamer-plugins-base1.0-dev

```

```
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
```
```
git clone https://github.com/raspberrypi/rpicam-apps.git
```


Navigate into the root directory of the repository:
```
cd rpicam-apps
```
For desktop-based operating systems like Raspberry Pi OS, configure the rpicam-apps build with the following meson command:
```
meson setup build -Denable_libav=enabled -Denable_drm=enabled -Denable_egl=enabled -Denable_qt=enabled -Denable_opencv=disabled -Denable_tflite=disabled -Denable_hailo=disabled
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



