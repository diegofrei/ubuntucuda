# Ubuntu CUDA & docker

Utility script to get CUDA Toolkit on ubuntu

## References
https://developer.nvidia.com/cuda-10.2-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1804&target_type=runfilelocal
https://github.com/NVIDIA/nvidia-docker


## Installation

### Dependencies

```
sudo apt-get install -y build-essential curl wget
```

### Docker

Install docker
```
sudo apt install -y docker.io
```

Add user to docker group
```
theUser=$USER
sudo groupadd docker
sudo usermod -aG docker $theUser
newgrp docker
```

Test docker
```
docker run hello-world
```

### docker-compose

Install docker
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### CUDA Toolkit

```
wget http://developer.download.nvidia.com/compute/cuda/10.2/Prod/local_installers/cuda_10.2.89_440.33.01_linux.run
sudo sh cuda_10.2.89_440.33.01_linux.run
```

CUDA installer issues:

```
nvidia-installer log file '/var/log/nvidia-installer.log'
creation time: Wed Jul  8 18:41:56 2020
installer version: 440.33.01

PATH: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin

nvidia-installer command line:
    ./nvidia-installer
    --ui=none
    --no-questions
    --accept-license
    --disable-nouveau
    --no-cc-version-check
    --install-libglvnd

Using built-in stream user interface
-> Detected 8 CPUs online; setting concurrency level to 8.
-> Installing NVIDIA driver version 440.33.01.
-> Running distribution scripts
   executing: '/usr/lib/nvidia/pre-install'...
-> done.
-> The distribution-provided pre-install script failed!  Are you sure you want to continue? (Answer: Continue installation)
ERROR: The Nouveau kernel driver is currently in use by your system.  This driver is incompatible with the NVIDIA driver, and must be disabled before proceeding.  Please consult the NVIDIA driver README and your Linux distribution's documentation for details on how to correctly disable the Nouveau kernel driver.
-> For some distributions, Nouveau can be disabled by adding a file in the modprobe configuration directory.  Would you like nvidia-installer to attempt to create this modprobe file for you? (Answer: Yes)
-> One or more modprobe configuration files to disable Nouveau have been written.  For some distributions, this may be sufficient to disable Nouveau; other distributions may require modification of the initial ramdisk.  Please reboot your system and attempt NVIDIA driver installation again.  Note if you later wish to reenable Nouveau, you will need to delete these files: /etc/modprobe.d/nvidia-installer-disable-nouveau.conf
ERROR: Installation has failed.  Please see the file '/var/log/nvidia-installer.log' for details.  You may find suggestions on fixing installation problems in the README available on the Linux driver download page at www.nvidia.com.
``` 

After reboot install looks fine

```
===========
= Summary =
===========

Driver:   Installed
Toolkit:  Installed in /usr/local/cuda-10.2/
Samples:  Installed in /home/user/, but missing recommended libraries

Please make sure that
 -   PATH includes /usr/local/cuda-10.2/bin
 -   LD_LIBRARY_PATH includes /usr/local/cuda-10.2/lib64, or, add /usr/local/cuda-10.2/lib64 to /etc/ld.so.conf and run ldconfig as root

To uninstall the CUDA Toolkit, run cuda-uninstaller in /usr/local/cuda-10.2/bin
To uninstall the NVIDIA Driver, run nvidia-uninstall

Please see CUDA_Installation_Guide_Linux.pdf in /usr/local/cuda-10.2/doc/pdf for detailed information on setting up CUDA.
Logfile is /var/log/cuda-installer.log
```

Added env vars to .profile

```
echo "PATH=\$PATH:/usr/local/cuda-10.2/bin" >> ~/.profile
echo "LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:/usr/local/cuda-10.2/lib64" >> ~/.profile
```
### NVIDIA docker support

Add the package repositories
```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```

Install nvidia container toolkit
```
sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker
```

Test GPU docker container
```
docker run --gpus all nvidia/cuda:10.2-cudnn7-devel nvidia-smi

Wed Jul  8 17:15:52 2020
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.33.01    Driver Version: 440.33.01    CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 960     Off  | 00000000:02:00.0  On |                  N/A |
| 30%   37C    P8    10W / 130W |    180MiB /  1993MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+
```

### NVIDIA docker runtime

```
curl -s -L https://nvidia.github.io/nvidia-container-runtime/gpgkey | sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-container-runtime/$distribution/nvidia-container-runtime.list | sudo tee /etc/apt/sources.list.d/nvidia-container-runtime.list
sudo apt-get update
sudo apt-get install nvidia-container-runtime
```
==> BROKEN
```
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 nvidia-container-runtime : Depends: nvidia-container-toolkit (>= 1.2.0) but 1.1.2-1 is to be installed
E: Unable to correct problems, you have held broken packages.
```

Added gpgkey
```
curl -s -L https://nvidia.github.io/nvidia-container-runtime/gpgkey | sudo apt-key add -
sudo apt-get update
```
--> output
```
Hit:1 http://ch.archive.ubuntu.com/ubuntu bionic InRelease
Hit:2 http://ch.archive.ubuntu.com/ubuntu bionic-updates InRelease                                                                             
Hit:3 http://ch.archive.ubuntu.com/ubuntu bionic-backports InRelease                                                                           
Hit:4 http://security.ubuntu.com/ubuntu bionic-security InRelease                                                                              
Hit:5 http://packages.microsoft.com/repos/vscode stable InRelease                                                                          
Get:6 https://nvidia.github.io/libnvidia-container/stable/ubuntu18.04/amd64  InRelease [1'146 B]                     
Hit:7 https://nvidia.github.io/nvidia-container-runtime/ubuntu18.04/amd64  InRelease
Hit:8 https://nvidia.github.io/nvidia-docker/ubuntu18.04/amd64  InRelease      
Reading package lists... Done
E: Repository 'https://nvidia.github.io/libnvidia-container/stable/ubuntu18.04/amd64  InRelease' changed its 'Origin' value from 'https://nvidia.github.io/libnvidia-container' to 'https://nvidia.github.io/libnvidia-container/stable'
N: This must be accepted explicitly before updates for this repository can be applied. See apt-secure(8) manpage for details.
W: Target Packages (Packages) is configured multiple times in /etc/apt/sources.list.d/nvidia-container-runtime.list:3 and /etc/apt/sources.list.d/nvidia-docker.list:3
W: Target Translations (en_US) is configured multiple times in /etc/apt/sources.list.d/nvidia-container-runtime.list:3 and /etc/apt/sources.list.d/nvidia-docker.list:3
W: Target Translations (en) is configured multiple times in /etc/apt/sources.list.d/nvidia-container-runtime.list:3 and /etc/apt/sources.list.d/nvidia-docker.list:3
W: Target Packages (Packages) is configured multiple times in /etc/apt/sources.list.d/nvidia-container-runtime.list:1 and /etc/apt/sources.list.d/nvidia-docker.list:1
W: Target Translations (en_US) is configured multiple times in /etc/apt/sources.list.d/nvidia-container-runtime.list:1 and /etc/apt/sources.list.d/nvidia-docker.list:1
W: Target Translations (en) is configured multiple times in /etc/apt/sources.list.d/nvidia-container-runtime.list:1 and /etc/apt/sources.list.d/nvidia-docker.list:1
W: Target Packages (Packages) is configured multiple times in /etc/apt/sources.list.d/nvidia-container-runtime.list:3 and /etc/apt/sources.list.d/nvidia-docker.list:3
W: Target Translations (en_US) is configured multiple times in /etc/apt/sources.list.d/nvidia-container-runtime.list:3 and /etc/apt/sources.list.d/nvidia-docker.list:3
W: Target Translations (en) is configured multiple times in /etc/apt/sources.list.d/nvidia-container-runtime.list:3 and /etc/apt/sources.list.d/nvidia-docker.list:3
```

Retry
```
sudo apt-get --allow-releaseinfo-change update
sudo apt-get install nvidia-container-runtime
```
--> install ok


https://github.com/NVIDIA/nvidia-container-runtime#docker-engine-setup

```
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo tee /etc/systemd/system/docker.service.d/override.conf <<EOF
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --host=fd:// --add-runtime=nvidia=/usr/bin/nvidia-container-runtime
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

Daemon config file

```
sudo tee /etc/docker/daemon.json <<EOF
{
    "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
        }
    },
    "default-runtime": "nvidia"
}
EOF
sudo pkill -SIGHUP dockerd
```

==> BROKEN due to missing runtime!!

```
docker run --gpus all --runtime=nvidia nvidia/cuda:10.2-cudnn7-devel nvidia-smi
docker: Error response from daemon: OCI runtime create failed: unable to retrieve OCI runtime error (open /run/containerd/io.containerd.runtime.v1.linux/moby/317ca706a6352eb3e000feffc02183b9e3fea14c571b9976993f0c8b6cfce171/log.json: no such file or directory): fork/exec /usr/bin/nvidia-container-runtime: no such file or directory: unknown.
```

Final config files:

/etc/systemd/system/docker.service.d/override.conf

```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --host=fd:// -H tcp:0.0.0.0:2375 --add-runtime=nvidia=/usr/bin/nvidia-container-runtime
```

