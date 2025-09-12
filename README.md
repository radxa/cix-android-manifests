## Build environment setup

Recommend build host is Ubuntu 22.04 64bit, for other hosts, refer official Android documents [Establishing a Build Environment](https://source.android.com/setup/build/initializing).


```shell
$ mkdir -p ~/bin
$ wget 'https://storage.googleapis.com/git-repo-downloads/repo' -P ~/bin
$ chmod +x ~/bin/repo
```

Android's source code primarily consists of Java, C++, and XML files. To compile the source code, you'll need to install OpenJDK 8, GNU C and C++ compilers, XML parsing libraries, ImageMagick, and several other related packages.


```shell
$ sudo apt-get -y update
$ sudo apt-get -y install lsb-release autoconf autopoint bc \
    bison build-essential cpio curl device-tree-compiler \
    dosfstools doxygen fdisk flex gdisk gettext-base git \
    libncurses5 libssl-dev libtinfo5 m4 mtools pkg-config \
    python2 python3 python3-distutils python3-pyelftools \
    rsync snapd unzip uuid-dev wget scons perl libwayland-dev \
    wayland-protocols indent libtool dwarves libarchive-tools \
    xorriso jigdo-file python3-pip vim sudo parted cmake \
    golang libffi-dev u-boot-tools img2simg libxcb-randr0 \
    libxcb-randr0-dev libxcb-present-dev libxau-dev \
    python3-mako libglib2.0-dev-bin binfmt-support qemu \
    qemu-user-static debootstrap multistrap debian-archive-keyring \
    ser2net git-lfs zstd debhelper jq pigz zip
$ wget https://bootstrap.pypa.io/get-pip.py
$ python3 get-pip.py
$ pip uninstall pyOpenSSL
$ pip install --upgrade --force-reinstall 'requests==2.31.0' 'urllib3==1.26.0' \
    'meson==1.3.0' 'ply==3.11' 'cryptography==41.0.7' 'docutils==0.18.1' \
    openpyxl nexus3-cli launchpadlib pyOpenSSL -i https://pypi.tuna.tsinghua.edu.cn/simple
            
```

## Download source code

```shell
$ mkdir cix-android14
$ cd cix-android14
```
Then run:

```shell
$ ~/bin/repo init -u https://github.com/radxa/cix-android-manifests.git -b cix_radxa_o6_rc2 -m cix_radxa_o6_release.xml
$ repo sync -j$(nproc)
$ repo forall -c 'git lfs pull'
```
It might take quite a bit of time to fetch the entire AOSP source code!

In China:
Download Repo
```shell
$ curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo
$ export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'
```

AOSP download can be changed to Tsinghua University source.
```bash
$ cd .repo/manifests
$ git diff cix_radxa_o6_release.xml
diff --git a/cix_radxa_o6_release.xml b/cix_radxa_o6_release.xml
index 76eb64b..890d610 100644
--- a/cix_radxa_o6_release.xml
+++ b/cix_radxa_o6_release.xml
@@ -10,14 +10,14 @@
   <remote name="cix_open" fetch="https://gitlab.com/cix-android14" rreview=""/>
   <remote name="linux_repo" fetch="https://gitlab.com/cix-android14" rreview=""/>
   <remote name="cix_android" fetch="https://gitlab.com/cix-android14" review=""/>
-  <include name="aosp-default.xml" />
-  <!-- <include name="aosp-radxa-default.xml" />
+  <!-- <include name="aosp-default.xml" /> -->
+  <include name="aosp-radxa-default.xml" />
   <remove-project name="device/google/felix"/>
   <remove-project name="platform/tools/tradefederations/contrib"/>
   <remove-project name="platform/tools/tradefederations/prebuilts"/>
   <project path="device/google/felix" name="device/google/felix" remote="cix_android" groups="device,felix" revision="cix-radxa-o6" upstream="cix-radxa-o6"/>
   <project path="tools/tradefederation/contrib" name="platform/tools/tradefederations/contrib" remote="cix_android" groups="pdk,tradefed" revision="cix-radxa-o6" upstream="cix-radxa-o6"/>
-  <project path="tools/tradefederation/prebuilts" name="platform/tools/tradefederations/prebuilts" remote="cix_android" groups="pdk,tradefed" revision="cix-radxa-o6" upstream="cix-radxa-o6" clone-depth="1" /> -->
+  <project path="tools/tradefederation/prebuilts" name="platform/tools/tradefederations/prebuilts" remote="cix_android" groups="pdk,tradefed" revision="cix-radxa-o6" upstream="cix-radxa-o6" clone-depth="1" />
 
   <project path="bootable/recovery" name="platform/bootable/recovery" remote="cix_android" groups="cix" revision="cix-radxa-o6" upstream="cix-radxa-o6" base="5880e9e924fbae9e1b55ff8747537ebb64323d87" />
   <project path="build-scripts" name="cix_build_scripts" remote="linux_repo" groups="cix" revision="cix-radxa-o6" upstream="cix-radxa-o6" base="">
```
Then run:
```shell
$ ~/bin/repo init -u https://github.com/radxa/cix-android-manifests.git -b cix_radxa_o6_rc2 -m cix_radxa_o6_release.xml
$ repo sync -j$(nproc)
$ repo forall -c 'git lfs pull'
```

## Build image

```shell
$ source build/envsetup.sh
$ lunch sky1_orion_o6-ap2a-userdebug
$ ./build-android.sh
```
It takes a long time, take a break and wait...
The generated image is in out/target/product/sky1_orion_o6/images.
```bash
images/
├── android_flush_images.bat
├── android_flush_images.sh
├── boot.img
├── cix_flash_all.bin                   
├── cix_flash_all_rsa_pr.bin
├── cix_flash_ota.bin
├── cix_flash_ota_rsa_pr.bin
├── dtbo-sky1.img
├── init_boot.img
├── LinuxLoader.efi.cap
├── partition-table-default-4096.img
├── partition-table-default.img
├── super.img
├── vbmeta-sky1.img
└── vendor_boot.img
```

## Burn image
Ubuntu or Windows requires the installation of Android SDK Platform-Tools. Platform-Tools includes ADB and Fastboot.

```bash
Download:
https://developer.android.google.cn/tools/releases/platform-tools
```

#### Need to install the BIOS first, you can refer to the [radxa document](https://docs.radxa.com/en/orion/o6/bios/install-bios).need to use the compiled "cix_flash_all.bin"

### Fastboot flash bootloader

Use a jumper cap to short the FAST BOOT HDR jumper pin, press the RESET BTN button to enter fastboot mode.

Enter the directory where cix_flash_all_rsa_pr.bin is located and run:
```shell
$ fastboot flash bootloader cix_flash_all_rsa_pr.bin
```

### Fastboot flash Android image
Enter the images directory and execute the script android_flush_image.sh

```shell
./android_flush_images.sh
```




