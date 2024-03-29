This repository is developed in Fish-IoT project

https://www.tequ.fi/en/project-bank/fish-iot/ 

---

# tequ-jetson-basler

This repository is updated to use official Basler Gstreamer plug-in to fetch data from Basler cameras. Follow track 1 to install and use official plugin and track 2 to install unofficial plugin. Unofficial plug-in guide is no longer updated.

Tested platforms:
- NVIDIA Jetson AGX Orin Developer Kit, Jetpack 5.0.2
- NVIDIA Jetson Nano, Jetpack 4.6.1
- Neousys NRU-120S (Jetson AGX Xavier, Jetpack 4.6.1)

Basler software & hardware used:
- Basler daA3840-45uc USB3 camera
- Pylon 7.2.1.25747

Notes:
- Installing official Basler plug-in to Jetson Nano requires extra steps which takes ~1-2 hours extra time
- Unofficial track is only tested with Jetson Nano with Jetpack 4.6.1 

## 1. Install official Basler plugin

### 1.1 Install Basler Pylon Package for ARM64 

```
cd $home
```

```
mkdir basler
```

```
cd basler
```

```
wget https://tequ-files.s3.eu.cloud-object-storage.appdomain.cloud/pylon_7.2.1.25747_aarch64_debs.tar.gz
```

```
wget https://tequ-files.s3.eu.cloud-object-storage.appdomain.cloud/pylon-supplementary-package-for-mpeg-4_1.0.2.120-deb0_arm64.deb
```

```
tar -xvf pylon_7.2.1.25747_aarch64_debs.tar.gz
```

```
sudo apt-get install ./pylon_*.deb ./codemeter*.deb ./pylon-supplementary*.deb
```

### 1.2 Test your setup

- Find and Open Pylon Viewer from applications

Or use terminal:
 
```
/opt/pylon/bin/pylonviewer
```

- Connect your camera cables and power it up

- Search your camera from devices list

- Check that you can see video

- Configure cameras and their settings for your needs 
 - Pylon IP configurator
 - Configure Static IP addresses for PoE ports and cameras (Neousys NRU-120S)

- Use Basler Bandwidth manager to optimize your GigE & USB cameras transmission settings

- Save settings to file for later use with Gstreamer => Select Tools -> Save Features


### 1.3 Install Basler Gstreamer plugin

Instructions source: https://github.com/basler/gst-plugin-pylon

Steps for Jetson AGX Orin (Jetpack 5):
```
sudo apt remove meson ninja-build
```

```
sudo apt install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
```

```
sudo -H python3 -m pip install meson ninja --upgrade
```

```
sudo apt install  cmake
```

Steps for (Jetpack 4.6.1), this updates Python and Cmake and it takes a long time:
```
sudo apt remove meson ninja-build
sudo apt install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
sudo apt install python3.8-dev
sudo rm /usr/bin/python3
sudo ln -s /usr/bin/python3.8 /usr/bin/python3
cd $home
wget https://tequ-files.s3.eu.cloud-object-storage.appdomain.cloud/cmake-3.20.0.tar.gz
tar -zvxf cmake-3.20.0.tar.gz
cd cmake-3.20.0
sudo apt-get install libssl-dev 
./bootstrap
make -j8
sudo apt-get install checkinstall
sudo checkinstall --pkgname=cmake --pkgversion="3.20-custom" --default
hash -r
sudo -H pip3 install scikit-build
sudo -H pip3 install meson
sudo -H pip3 install setuptools
sudo -H pip3 install ninja
```

Common steps for both Jetpacks continue from here 

You can also start from here if you are building again:

```
cd $home
```

```
git clone https://github.com/basler/gst-plugin-pylon
```

```
cd gst-plugin-pylon
```

```
export PYLON_ROOT=/opt/pylon
```

```
meson builddir --prefix /usr/
```

```
ninja -C builddir
```

```
sudo ninja -C builddir install
```

```
gst-inspect-1.0 pylonsrc
```

### 1.4 example pipelines

```
gst-launch-1.0 pylonsrc ! queue ! nvvidconv ! xvimagesink
```

```
gst-launch-1.0 pylonsrc ! queue ! bayer2rgb ! queue ! nvvidconv ! xvimagesink
```

```
gst-launch-1.0 pylonsrc pfs-location=40257292.pfs device-serial-number="40257292"  ! queue ! bayer2rgb ! queue ! nvvidconv ! xvimagesink
```

```
gst-launch-1.0 pylonsrc pfs-location=40257292.pfs device-serial-number="40257292"  ! queue ! bayer2rgb ! queue ! nvvidconv ! xvimagesink
```

```
gst-launch-1.0 pylonsrc pfs-location=40257292.pfs device-serial-number="40257292"  ! queue ! nvvidconv ! nvjpegenc ! queue ! tcpclientsink port=55555
```

```
gst-launch-1.0 -v pylonsrc pfs-location=40257292.pfs device-serial-number="40257292"   ! nvvidconv ! tee name=t t.! queue ! nvjpegenc ! queue ! tcpclientsink port=50001 t. ! queue ! omxh264enc ! queue ! rtspclientsink location=rtsp://localhost:8554/40122260
```

```
gst-launch-1.0 -v pylonsrc pfs-location=40257292.pfs device-serial-number="40257292"  ! nvvidconv ! tee name=t t.! queue ! nvjpegenc ! queue ! tcpclientsink port=50002 t. ! queue ! omxh264enc ! queue ! rtspclientsink location=rtsp://localhost:8554/23751808
```

```
gst-launch-1.0 pylonsrc pfs-location=40257292.pfs device-serial-number="40257292" ! queue ! bayer2rgb ! queue ! nvvidconv ! queue ! nvjpegenc ! tcpclientsink port=55555
```

-------------------------------------------------------------------------------------------------

## 2. Install gst-plugins-vision (unofficial plugin)

Install and configure Basler pylon components for Computer Vision to NVIDIA Jetson board. 

Setup:
- NVIDIA Jetson Nano
- Jetpack 4.6.1
- Basler daA3840-45uc USB3 camera
- Pylon 6.2.0.21487


### 2.1 Install Basler Pylon Package for ARM64 

```
cd $home
```

```
wget https://tequ-files.s3.eu.cloud-object-storage.appdomain.cloud/pylon_6.2.0.21487_aarch64_setup.tar.gz
```

```
mkdir ./pylon_setup
```

```
tar -C ./pylon_setup -xzf ./pylon_*_setup.tar.gz
```

```
cd pylon_setup
```

```
sudo mkdir /opt/pylon
```

```
sudo tar -C /opt/pylon -xzf ./pylon_*.tar.gz
```

```
sudo chmod 755 /opt/pylon
```

```
sudo /opt/pylon/share/pylon/setup-usb.sh
```

Answer yes to both question.

### 2.2 Test your setup

- Open Pylonviewer on Ubuntu desktop from /opt/pylon/bin/pylonviewer

```
/opt/pylon/bin/pylonviewer
```

- Connect your camera cables and power it up

- Search your camera from devices list

- Check that you can see video

- Configure cameras and their settings for your needs 
 - Pylon IP configurator
 - Configure Static IP addresses for PoE ports and cameras (Neousys NRU-120S)

- Use Basler Bandwidth manager to optimize your GigE & USB cameras transmission settings

- Save settings to file => Select Tools -> Save Features

### 2.3 Install Pypylon for Python programming

```
cd $home
```

```
git clone https://github.com/basler/pypylon
```

```
cd pypylon
```

```
sudo apt-get install swig
```

```
pip3 install .
```

### 2.4 Test that Pypylon library is working

```
cd /samples
```

```
python3 save_image.py
```

5 images should appear to folder. Running this example might take a while if your camera is configured to grab large images.


### 2.5 Install unofficial Gstreamer plugin for Basler cameras
                                               
```
cd $home
```

```
apt-get install git cmake libgstreamer-plugins-base1.0-dev liborc-0.4-dev
```

```
git clone https://github.com/Lapland-UAS-Tequ/gst-plugins-vision
```

```
cd gst-plugins-vision
```

```
sudo nano cmake/modules/FindPylon.cmake
```

Replace "/opt/pylon5" => "/opt/pylon"

```
mkdir build
```

```
cd build
```

```
cmake -DCMAKE_INSTALL_PREFIX=/usr/lib/aarch64-linux-gnu ..
```

```
make
```

```
sudo make install
```

```
sudo nano /etc/ld.so.conf.d/pylon.conf
```

Add following path to the file:

```
/opt/pylon/lib
```

```
sudo ldconfig
```


### 2.6 Example Gstreamer pipelines, depending on camera settings

```
gst-launch-1.0 pylonsrc ! queue ! nvvidconv ! xvimagesink
```

```
gst-launch-1.0 pylonsrc ! queue ! bayer2rgb ! queue ! nvvidconv ! xvimagesink
```

```
gst-launch-1.0 pylonsrc config-file=config.pfs ! queue ! bayer2rgb ! queue ! nvvidconv ! xvimagesink
```

```
gst-launch-1.0 pylonsrc config-file=config.pfs ! queue ! nvvidconv ! nvjpegenc ! queue ! tcpclientsink port=55555
```

```
gst-launch-1.0 -v pylonsrc camera=0 config-file=/home/tequ/40122260.pfs  ! nvvidconv ! tee name=t t.! queue ! nvjpegenc ! queue ! tcpclientsink port=50001 t. ! queue ! omxh264enc ! queue ! rtspclientsink location=rtsp://localhost:8554/40122260
```

```
gst-launch-1.0 -v pylonsrc camera=1 config-file=/home/tequ/23751808.pfs  ! nvvidconv ! tee name=t t.! queue ! nvjpegenc ! queue ! tcpclientsink port=50002 t. ! queue ! omxh264enc ! queue ! rtspclientsink location=rtsp://localhost:8554/23751808
```

More Gstreamer examples:

https://github.com/Lapland-UAS-Tequ/tequ-basler-gstreamer

## Install rtsp-simple-server for RTSP streaming

```
cd $home
```

```
mkdir rtsp-simple-server
```

```
cd rtsp-simple-server
```

```
wget https://github.com/aler9/rtsp-simple-server/releases/download/v0.17.13/rtsp-simple-server_v0.17.13_linux_arm64v8.tar.gz
```

```
tar -xzf rtsp-simple-server_v0.17.13_linux_arm64v8.tar.gz
```

```
sudo apt install gstreamer1.0-rtsp
```

```
./rtsp-simple-server
```

# Next steps:

Use Basler camera & Node-RED & Triton Inference Server to create computer vision system.

https://github.com/Lapland-UAS-Tequ/tequ-jetson-basler-nodered

# Notes:

If you see "dconf permission errors" on GStreamer pipelines, try 

```sudo chmod -R a+rwx /home/nvidia/.cache```

Where nvidia is your user name

# Sources:

https://github.com/basler/gst-plugin-pylon

https://www.baslerweb.com/fp-1666012566/media/downloads/software/pylon_software/INSTALL~8.txt

https://www.baslerweb.com/fp-1636374969/media/downloads/software/pylon_software/INSTALL~5.txt

https://github.com/joshdoe/gst-plugins-vision

https://gist.github.com/bmegli/4049b7394f9cfa016c24ed67e5041930
