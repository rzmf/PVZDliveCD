# PVZD Live CD

## Purpose
This is a secure boot environment for X11-applications deployed via docker.
The CentOS 7 Live CD comes with a GNOME desktop an is booting automatically 
into a predefined docker image. Docker data and the container user's home 
are stored on a device with a writeable filesystem. The device is tagged 
with a filename in the root directory for auto-discovery at boot time.

## Concept

- When Gnome is starting it executes a script 'startall_user.sh'. This in turn executes 'predocker.sh'.
  This script identifies the writeable filesystem to be used for storing docker data (images, container etc.).
  The respective filesystem is marked by a file with the name 'UseMe4DockerData' in the root directory.
- If the UseMe4DockerData filesystem is found, the docker daemon is reconfigured to use this filesystem
- Then the script startall_user.sh is executed to start the docker container


## Install build environment

    sudo yum install livecd-tools
    cd <path to contain project>
    git clone identinetics/PVZDliveCD
    cd PVZDliveCD
    PROJ_HOME=$PWD
    echo $PWD > CLCDDIRvar
    mkdir livecache

## Configure & Build

- Set the DOCKER_IMAGE to be executed in install/scripts/startapp.sh
- Build the ISO image

    sudo livecd-creator -d -v  -c sig-core-livemedia/centos-7-live-gnome-docker.cfg --cache=$PROJ_HOME/livecache/ --nocleanup

- Copy the ISO image to your boot device
The resulting file is in the project root (livecd-centos-7-live-gnome-docker-*.iso). Copy it to USB drive (2GB ore more)

    dd if=livecd-centos-7-live-gnome-docker-<timestamp>.iso of=/dev/<usb-drive>

## Usage (generic)



- You require 2 media:
    1. the boot medium with the LiveCD (should be read-only, such as CD-ROM), and
    2. a writeable medium, large enough to contain a docker image and docker work files. Start with at least 8GB for a GUI.
- Format the writeable medium with a standard linux filesystem, such as ext4 and mark it with a a file named 'UseMe4DockerData'. E.g.:

    # check dmesg for the actual device of your USB flash drive
    mkfs.vfat /dev/sdb1
    mount /dev/dsb1  /mnt
    touch /mnt/UseMe4DockerData

- Insert both media into the PC
- Boot from the boot-medium (you might have to modify the boot sequence in the BIOS)
- Wait for the system to come up

## Security Considerations
- The execution environment (i.e. the hardware to boot the system) must be trusted.
- Booting the system on a virtual machine on a machine used for other purposes is
  weakening the security properties. However, using a virtual environment 
  is OK to enable installation on a dedicated machine which cannot boot the livecd 
  otherwise.
- Docker content security (to be implemented): The docker image must be signed with 
  a key generated by the trusted Docker notary.
- Docker container security: containers reside on unprotected writeable media. 
  To prevent tampering, the container is discarded after each usage and a new
  one is on rebooting the system.