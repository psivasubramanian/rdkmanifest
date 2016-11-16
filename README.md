# rdkmanifest
This repository contains a local manifest that can be used to build RDK mediaclient image for hikey based on OE  buildsystem.

# 1) Setup OE system

$ mkdir ~/bin

$ PATH=~/bin:$PATH

$ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo

$ chmod a+x ~/bin/repo

Run repo init to bring down the latest version of Repo with all its most recent bug fixes. You must specify a URL for the manifest, which specifies where the various repositories included in the Android source will be placed within your working directory. To check out the current branch, specify it with -b:

$ repo init -u https://github.com/96boards/oe-rpb-manifest.git -b krogoth

When prompted, configure Repo with your real name and email address.

A successful initialization will end with a message stating that Repo is initialized in your working directory. Your client directory should now contain a .repo directory where files such as the manifest will be kept.

# 2) Add RDK overlay

$ cd .repo

$ git clone https://github.com/psivasubramanian/rdkmanifest.git local_manifests

$  cd ..

# 3) Sync
To pull down the metadata sources to your working directory from the repositories as specified in the default manifest, run

$ repo sync

When downloading from behind a proxy (which is common in some corporate environments), it might be necessary to explicitly specify the proxy that is then used by repo:

$ export HTTP_PROXY=http://<proxy_user_id>:<proxy_password>@<proxy_server>:<proxy_port>

$ export HTTPS_PROXY=http://<proxy_user_id>:<proxy_password>@<proxy_server>:<proxy_port>

More rarely, Linux clients experience connectivity issues, getting stuck in the middle of downloads (typically during "Receiving objects"). It has been reported that tweaking the settings of the TCP/IP stack and using non-parallel commands can improve the situation. You need root access to modify the TCP setting:

$ sudo sysctl -w net.ipv4.tcp_window_scaling=0

$ repo sync -j1

# 4) Setup Environment

MACHINE values can be:
hikey-32

DISTRO values can be:
rdk

$ source setup-environment

# 5) Configure BBLAYERS and local.conf

$ bitbake-layers add-layer ../layers/meta-rdk

$ bitbake-layers add-layer ../layers/meta-metrological/

Append the following to conf/local.conf

RDK_GIT="lhg-review.linaro.org:29418/lhg"

require ${OEROOT}/layers/meta-rdk/conf/distro/include/rdkv.inc

BBMASK = "sysint-broadband tdk-b recipes-ccsp closedcaption meta-rdk/recipes-core/systemd meta-rdk/recipes-devtools/e2fsprogs meta-rdk/recipes-graphics/directfb"

DISTRO_FEATURES_remove = " bluetooth wifi"

DISTRO_FEATURES_append = " directfb westeros"

EXTRA_OEMAKE += "-e MAKEFLAGS="

# 6) Build

$ bitbake rdk-generic-mediaclient-rpb-image

This will build the 32bit rootfs image with RDK Mediaclient, Westeros and WPE components i.e rdk-generic-mediaclient-rpb-image-hikey-32.ext4.gz. Need to copy 64bit kernel modules(/boot and /lib/modules) into this rootfs before flashing the image to hikey board as below

$ gunzip rdk-generic-mediaclient-rpb-image-hikey-32.ext4.gz

$ sudo mount -o loop,sync,rw rdk-generic-mediaclient-rpb-image-hikey-32.ext4 rootfs-32 (mount the 32bit image into a directory)

$ sudo rsync -avz rootfs-64/boot rootfs-32/ (rsync boot files from 64bit image)

$ sudo rsync -avz rootfs-64/lib/modules rootfs-32/lib/ (rsync kernel modules from 64bit image)

$ sudo umount rootfs-32 (unmount the rootfs)

$ ext2simg rdk-generic-mediaclient-rpb-image-hikey-32.ext4 rdk-generic-mediaclient-rpb-image-hikey-32.img

# 7) Flashing

Follow the link https://github.com/96boards/documentation/wiki/HiKeyUEFI#flash-binaries-to-emmc- for standard flashing procedure. To flash the above build rootfs, use the below command

$ sudo fastboot flash system rdk-generic-mediaclient-rpb-image.img


# 8) Launching Westeros compositor

$ export XDG_RUNTIME_DIR=/run/user/

$ LD_PRELOAD=/usr/lib/libwesteros_gl.so.0 westeros –renderer /usr/lib/libwesteros_render_gl.so.0 --enableCursor --display westeros-1-0 &

This will launch the Westeros compositor as you can see blank screen with mouse cursor on the display. The option --enableCursor is to enable the mouse pointer while --display displayName is used to set the display name.

# 9) Running westeros test application

$ westeros_test --display=westeros-1-0

This will run the westeros_test application which displays the continously rotating color triangle on the screen.

# 10) Running WPELauncher with westeros backend

$ export WAYLAND_DISPLAY=westeros-1-0

$ export WPE_BACKEND=westeros

$ ./WPELauncher <URL>

This will launch the webpage of the provided URL using WPELauncher. It will launch youtube if no URL is provided.

# Updating the sandbox

If you need to bring changes from upstream then use following commands

$ repo sync

Rebase your local committed changes

$ repo rebase

# Maintainer

Sivasubramanian Patchaiperumal sivasubramanian.patchaiperumal@linaro.org
