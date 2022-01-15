This repository is developed in Fish-IoT project

https://www.tequ.fi/en/project-bank/fish-iot/ 

---

# tequ-jetson-basler
Install and configure Basler pylon components for Computer Vision to NVIDIA Jetson Xavier NX / Nano board.

## Install Basler Pylon Package for ARM64

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






Sources:

https://www.baslerweb.com/fp-1636374969/media/downloads/software/pylon_software/INSTALL~5.txt
