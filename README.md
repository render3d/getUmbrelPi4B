# getUmbrelPi4B
Instructions for basic self hosting on a Pi 4, using Umbrel with a slim install of Ubuntu for SSH capability. Adapted from [this guide](https://thecryptogarage.com/umbrel-linux-ubuntu-node-build/) for compatibility on a Pi 4 Model B.

## Instructions

### 1. The Pi
1. Flash the [Ubuntu image](https://ubuntu.com/download/raspberry-pi/thank-you?version=20.04.3&architecture=server-arm64+raspi) to an SD card (8GB minimum) using the Raspberry Pi Imager. Do not flash with any custom settings (from Ctrl + Shift + X settings window).
2. Insert SD card into Pi 4. Connect the Pi to a router (ethernet), a screen, a USB keyboard, and a 1TB SSD.
3. Connect the Pi to a power source and allow it to power up. wait for SSH keys to install and login when prompted with "ubuntu" as username and password. Change password when prompted to do so by default. Make a note of the IP of the ubuntu node.

*At this time, the keyboard and screen may be disconnected as the node may now be accessed via SSH using "ssh ubuntu@<"ip-adress">" and the password set in the previous step. Equally, the process may be continued in the present state without disconnecting either, the author leaves the choice in doing so to the user's preference.*

4. Remove the daily update lock timer on the ubuntu node with the following commands:

        sudo systemctl disable apt-daily.service
        sudo systemctl disable apt-daily.timer

        sudo systemctl disable apt-daily-upgrade.timer
        sudo systemctl disable apt-daily-upgrade.service

5. Update the Ubuntu Sever install. This step may take some time.

        sudo apt update
        apt list --upgradable
        sudo apt upgrade

6. Reboot may be necessary.

        sudo reboot

### 2. Install docker engine

*This is the standard [Docker Engine installation sequence](https://docs.docker.com/engine/install/ubuntu/).*

1.  First, uninstall old versions

        sudo apt-get remove docker docker-engine docker.io containerd runc

2.  Check everything is up to date

        sudo apt-get update


3. Install packages to allow apt to use a repository over HTTPS:

        sudo apt-get install \
            ca-certificates \
            curl \
            gnupg \
            lsb-release

4. Next, add Docker’s official GPG key:

        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

5. Use the following command to set up the stable repository.

        echo \
        "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

6. Now install docker-engine

        sudo apt-get update
        sudo apt-get install docker-ce docker-ce-cli containerd.io

7. Verify that Docker Engine is installed correctly by running the hello-world image.

        docker --version
        sudo docker run hello-world

8. List containers

        docker info

9. Delete the container from the last step.

        sudo systemctl stop docker
        sudo rm -rf /var/lib/docker/containers/<container ID>
        sudo systemctl start docker

10. Check container has been removed from the list

        docker info

### Install docker-compose

*This installation sequence is suitable for Pi 4 model B arm64 architectures **specifically**. See [here](https://forum.armbian.com/topic/14763-%E2%80%98command-not-found%E2%80%99-when-i-try-to-run-docker-compose/), [here](https://github.com/docker/compose/issues/6268), and [here](https://github.com/docker/compose/issues/8445) for more information.*

1. Run

        sudo apt install docker-compose

2. Verify the installation with

        docker-compose --version

### Install Remaining Umbrel Requirements

1. To finish the installation requirements before we install Umbrel, these are the remaining items to install

        sudo apt-get install fswatch jq rsync curl

### Mounting the SSD

*Follow methods and advice given [here](https://help.ubuntu.com/community/InstallingANewHardDrive). Ensure the drive has been [cleansed](https://uk.crucial.com/support/articles-faq-ssd/reset-ssd-with-windows-diskpart) prior to commencing.*

1. List storage locations

        sudo fdisk -l

2. Initialise the drive.

        sudo fdisk /dev/sda

3. Submit as inputs the following command sequence:

        n       (add a new partition)
        p       (primary partition (1-4))
        1       (partition number)
        default (press <enter>)
        default (press <enter>)
        w       (write table to disk and exit)

4. Format the new partition as ext4 file system

        sudo mkfs -t ext4 /dev/sda1

5. Create a mounting point

        sudo mkdir /media/mynewdrive

6. Mount the drive

        sudo mount /dev/sda1 /media/mynewdrive

The external drive must be mounted manually (using the 6th command above) after each reboot or start up sequence.

### Installing Umbrel

1. Create a new directory for Umbrel on the external drive for the installation

        cd /media/mynewdrive
        mkdir umbrel
        cd umbrel

2. Initialise root user

        sudo su

3. Download Umbrel

        curl -L https://github.com/getumbrel/umbrel/archive/v0.4.11.tar.gz | tar -xz --strip-components=1

4. Become Bitcoin and start Umbrel

        sudo ./scripts/start

Umbrel should start and after a short while be accessible at http://<"ip address of the Pi">. Troubleshoot messages as they appear if launch sequence exits early. Restart launch as many times as needed.