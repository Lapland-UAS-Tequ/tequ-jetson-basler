This repository is developed in Fish-IoT project

https://www.tequ.fi/en/project-bank/fish-iot/ 

---

# tequ-jetson-basler
Install and configure Basler pylon components for Computer Vision to NVIDIA Jetson Xavier NX / Nano board.

# Install Basler Pylon Package for ARM64

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
cd /pylon_setup
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

## Test your setup

- Open Pylonviewer on Ubuntu desktop from /opt/pylon/bin/pylonviewer

- Connect your camera cables and power it up

- Search your camera from devices list

- Check that you can see video

- Configure camera for your needs 

- Save settings to file => Select Tools -> Save Features

# Install Pypylon for Python programming

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

## Test that Pypylon library is working

```
cd /samples
```

```
python3 save_image.py
```

5 images should appear to folder. Running this example might take a while if your camera is configured to grab large images.


# Install Gstreamer plugin for Basler cameras

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
sudo nano /cmake/modules/FindPylon.cmake
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
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/pylon/lib
```

```
gst-launch-1.0 pylonsrc ! queue ! bayer2rgb ! queue ! videoconvert ! autovideosink
```

More Gstremer examples:

https://github.com/Lapland-UAS-Tequ/tequ-basler-gstreamer


Sources:

https://www.baslerweb.com/fp-1636374969/media/downloads/software/pylon_software/INSTALL~5.txt

https://github.com/joshdoe/gst-plugins-vision

