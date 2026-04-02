---
layout: post
ytembedid: WmVy7jQseBs
artist: Aphex Twin
song: Cirklon3 (Kolkhoznaya Mix)
title: Installing Kali NetHunter (Svirusx Kernel) on Samsung Galaxy S20 FE 5G (Snapdragon) running Lineage OS 23.2 (Android 16)
tags: ['infosec','kali','android','computers']
thumbnail: 5.png
date: 2026-04-02 14:59:00 +0100
---
NetHunter (the custom kernel version) supports a myriad of devices, old and new, and I decided to run a little project where I'd set up my own NetHunter capable phone.

I decided to go with the Samsung S20 FE 5G, as it is the only one with support for very recent versions of Android, courtesy of the team at [LineageOS](https://lineageos.org).

If you work only from the available front page downloads, you'll see that NetHunter is only available, however, if your S20 FE 5G is running the stock firmware -  OneUI 5.0, i.e., Android 13. Not optimal.

![](/img/posts/2026-04-02-nethuntersvirusxS20FE5G/nethunter_builds.png)

Anything we can do to change this? Surprisingly, yes! Following a paper trail of documentation, you can see that there's a custom kernel specifically for the Samsung Galaxy S20 FE 5G running LineageOS 23.

![](/img/posts/2026-04-02-nethuntersvirusxS20FE5G/nethunter_kernels.png)

Offensive Security has a number of scripts available to use, one of them being used for building [NetHunter Installer Images]([https://gitlab.com/kalilinux/nethunter](https://gitlab.com/kalilinux/nethunter/build-scripts/kali-nethunter-installer)). I used that to create an image specifically to install NetHunter 2026.1 on the S20 FE 5G using this custom kernel, but when it came to installing it through Magisk, I ran into problems.

![](/img/posts/2026-04-02-nethuntersvirusxS20FE5G/nethunter_failed_install.png)

It was giving me a cryptic error referencing an inability to copy system modules to the `/system` partition of the phone, something you'd normally be able to do with root access.

I quickly opened an [issue](https://gitlab.com/kalilinux/nethunter/build-scripts/kali-nethunter-kernels/-/work_items/295) on their GitLab project to track this problem but didn't think much of it.

![](/img/posts/2026-04-02-nethuntersvirusxS20FE5G/gitlab_issue_1.png)

Incredibly a few hours later, the original author of the kernel, [**Svirusx**](https://github.com/Svirusx) (KUDOS!) responded with some insight.

![](/img/posts/2026-04-02-nethuntersvirusxS20FE5G/gitlab_issue_2.png)

This response gave me an idea for a strategy which, surprisingly, worked for me in installing the kernel. So here's a guide on "How to Install NetHunter on the Samsung S20 FE 5G for LineageOS 23.2":

## 1. Installing LineageOS 23.2 on the Phone
This is the simple part. I'll assume you have your phone still using the stock firmware. Good! It's a pre-requirement for a LineageOS installation.

Make sure that the phone's OEM is unlocked and that you've unlocked the bootloader. You can do this by setting the OEM unlock setting to "On" on the Developer Settings, then turning off the phone and turning it on in "Download mode", which is done by holding the `Volume Up+Volume Down` keys and plugging the USB cable while doing it. It should show a screen similar to this:

![](/img/posts/2026-04-02-nethuntersvirusxS20FE5G/samsung_download_mode.png)

Once you see this screen, let go of both keys. To unlock the bootloader, hold the `Volume Up` key for a few seconds. Reboot the phone and consult the Developer Options to make sure the OEM unlock setting is now greyed out. You're ready to go.

### 1.1. Flashing the Verified Boot Metadata Partition
First thing we need is to do is flash the VBMeta (Verified Boot Metadata) partition, which is the central data structure used in a process called [Android Verified Boot](https://android.googlesource.com/platform/external/avb/+/master/README.md#what-is-it). Download `vbmeta.img` from [here](https://download.lineageos.org/devices/r8q/builds). To flash these partitions we're talking about, you'll need a proper program to flash partitions on a Samsung phone.

A good program back in the day was Odin, but that's closed source and platform dependent (Windows). A much better alternative nowadays is [Heimdall](https://glassechidna.com.au/heimdall/) which is cross-platform and cross-architecture.

Put your phone back in "Download mode". Press the `Volume Up` key once to start a flashing procedure. Open your terminal and proceed to where you've downloaded `vbmeta.img` to. Once you're there, the command to flash it is:

```bash
$ heimdall flash --VBMETA vbmeta.img
```

Let the partition flash, then check to see if the phone reboots by itself. Once it does, it'll enter recovery mode and tell you to factory reset the phone. Proceed.

### 1.2. Flashing the Recovery Partition
Next, we'll flash the recovery partition with LineageOS' own custom recovery. Once you're finished with the factory reset, let the phone finish setting up, then go back to "Download mode" again.

Download `recovery.img` from [here](https://download.lineageos.org/devices/r8q/builds). With the flashing procedure started, run the following command:

```bash
$ heimdall flash --RECOVERY recovery.img --no-reboot
```

This time, we do not want the phone to reboot automatically once it is finished flashing. Follow the procedure on the terminal to check that it finishes flashing completely.

Once that happens, hold the `Volume Down` and `Side` keys for 7 seconds to reboot the phone. The moment the screen turns off and the reboot procedure initiates, hold the `Volume Up` and `Side` keys to make the phone boot in Recovery mode.

If successful, your phone will boot up into this screen:

![](/img/posts/2026-04-02-nethuntersvirusxS20FE5G/lineage_recovery.png)

### 1.3. Patching LineageOS 23.2 on top of the Stock ROM
Now that both VBMeta and Recovery partitions are flashed and persistent, it's time to install LineageOS on the phone - specifically, to patch LineageOS on top of the stock ROM.

First, use the `Volume` Keys to scroll down to `Factory reset` and select `Format data/Factory Reset` to remove the data and the encryption underneath.

Once that is completed, download the most recent build of LineageOS (`lineage-23.2-YYYYMMDD-nightly-r8q-signed.zip`) from [here](https://download.lineageos.org/devices/r8q/builds). To patch this in, select `Apply update`, then `ADB sideload`. On the terminal, go to where you downloaded the LineageOS build to and run the following command:

```bash
$ adb -d sideload lineage-23.2-YYYYMMDD-nightly-r8q-signed.zip
```

Let the .zip build install itself. It's completely normal if it stalls at 47%, then suddenly finishes the installation.

#### 1.3.1. (Optional) Installing Google Apps
If you feel like it will be absolutely necessary to install Google Apps on the phone, you'll need to take care of this now before booting into LineageOS. If not, skip this step.

You'll want to download the "MintTheGapps" add-on for LineageOS 23.0 (16.0.0), for ARM64 and repeat the same steps as patching LineageOS, but for this addon. `Apply Update` -> `ADB Sideload` and run the following command:

```bash
$ adb -d sideload MindTheGapps-16.0.0-arm64-YYYYMMDD_xxxxxx.zip
```

You'll most likely get a failed signature validation error. Allow the installation anyway.

---
Once you're done, select "Reboot system now" and proceed with the device configuration! Your phone is now using LineageOS 23!

## 2. Rooting the device using Magisk
Now we have to root the phone itself. **Rooting** is essentially just a way escalating privileges and persisting that elevated access on an Android device. It's called rooting because, like with Linux, the user with highest privileges in the device is called the `root` user. However, the access to this user is normally blocked by carriers and hardware manufacturers in order to segregate privileges and allow for uninstallable software to be present in the device (so called "bloatware"). We know better than that.

### 2.1. Patching Magisk on top of LineageOS
To do this, we'll simply patch the latest release of **Magisk** on top of LineageOS. Magisk is a userspace systemless rooting tool for Android versions 6.0 or higher. Normally, it has to be patched into the phone's firmware (`init_boot.img` or `boot.img`), but since LineageOS is a more open OS, we can just patch it from the Recovery mode.

First, download the most recent version of Magisk from [here](https://github.com/topjohnwu/Magisk/releases). You'll download a release with the following naming scheme: `Magisk-vXX.Y.apk`.

Before flashing it, we'll need to change this into a format that the custom recovery will accept, a .zip archive. You'd think this process would be hard, but APKs are just ZIP files actually. Sorry to shatter your illusions of the world.

```bash
$ mv Magisk-vXX.Y.apk Magisk-vXX.Y.zip
```

Once you've renamed your file, it's just a matter of rebooting back into recovery mode. With LineageOS, this process is much simpler too. In the `Settings` app, go to `System` > `Buttons` > `Power Menu` and turn on the `Advanced Reboot` option. Once this is done, you'll notice that, when the phone is unlocked and you attempt to restart the phone, you'll be prompted with the mode you want to restart the phone into. Select "Recovery".

In the Recovery menu, follow the steps to patch. `Apply Update` -> `ADB Sideload` and run the following command:

```bash
$ adb -d sideload Magisk-vXX.Y.zip
```

Once more, you'll get a failed signature validation error. Allow the installation anyway, let it install, then reboot the phone.

Once you reboot the phone, open the Magisk app and finish the setup. The phone will reboot once more. You can check the Magisk app and confirm the device is rooted.

## 3. Installing NetHunter on top of the Rooted LineageOS install
Now's the interesting part. Installing NetHunter on top of your rooted installation of LineageOS. Make sure you have a lot of internal storage capacity on your computer. You're gonna need it.

### 3.1. Building the Installation Module
First we'll need to build the module to install NetHunter through Magisk. Clone the building script:

```bash
$ git clone https://gitlab.com/kalilinux/nethunter/build-scripts/kali-nethunter-installer.git
```

Then head into the directory and initialize the kernels repo with the `bootstrap.sh` script.

```shell
$ ./bootstrap.sh
[?] Would you like to grab the full history of kernels? (y/N): N
[?] Would you like to use SSH authentication (faster, but requires a GitLab account with SSH keys)? (y/N): N
[i] Running command: git clone --depth 1 https://gitlab.com/kalilinux/nethunter/build-scripts/kali-nethunter-kernels.git kernels
Cloning into 'kernels'...
```

>[!danger] OI!
>This is the most important fucking step. Do not lose yourself.

Navigate into the S20 FE 5G's custom kernel directory.

```shell
$ cd kernels/sixteen/r8q-los
```

After that, delete all the contents inside the modules directory. We can restore them later to introduce on the system once NetHunter is done installing.

```shell
$ rm modules/*
```

After that, go back to the `kali-nethunter-installer` base directory. You'll first need to setup the build script's dependencies.

```shell
$ python3 -m venv .env
$ source .env/bin/activate
(.env) $ python3 -m pip install requests pyyaml
```

Finally, all we have to do is build the image itself. The command to build a 2026.1 release of NetHunter with this kernel is the following:

```bash
(.env) $ python3 build.py --kernel r8q-los -16 --rootfs full --release 2026.1
```

### 3.2. Installing NetHunter through Magisk
Now that the installation module is built, all we have to do is install it through Magisk as a module. Go to the Settings app, turn on `Developer Options` and turn on `USB Debugging`. Plug your phone into your PC, then, on the directory where your NetHunter ZIP module is, run the following command:

```shell
$ adb push nethunter-XXXYYYZZZ.zip /sdcard/Download
```

Wait for the push to finish and that's likely one of the last times you'll need to plug the phone into your PC. Now all you have to do is open Magisk, click on `Modules` > `Install from local storage` and then select the ZIP file you have in your Downloads folder (how'd it get in there?)

Wait for the installation to finish and boom! NetHunter is installed!

#### 3.2.1. Installing the missing modules
Ok, maybe not completely. If those modules for CARsenal are really itching your stitching, you can install them manually now.

A good utility for this is [MiXplorer](https://xdaforums.com/t/app-2-3-mixplorer-v6-x-released-fully-featured-file-manager.1523691/#post-23109280), which is straight up just a file browser that can request Root access to browse system files. Download the most recent APK release for `arm64`, then plug your phone into the PC for USB debugging. To install the APK, run the following command:

```shell
$ adb -d install MiXplorer_vX.YY.Z_Byymmddnn-arm64.apk
```

Once it's installed, go to the kernels directory of your `kali-nethunter-installer` again and restore all the files you deleted.

```shell
$ cd kali-nethunter-installer/kernels/sixteen/r8q-los/modules
$ git restore *
```

Once it's restored, you can just push all the kernel modules over to your phone on the Downloads folder:

```shell
$ adb push *.ko /sdcard/Download
```

Now, just use MiXplorer to move these .ko files into the `/system/lib/modules` directory. Once they're moved, just reboot the phone and you're done!

## Credits
Thanks to the LineageOS team for the great work they do, and to the Offensive Security team, especially the volunteer developers doing a lot of the development work on Kali Linux and NetHunter - way too specifically, thanks to [Volk3n](https://gitlab.com/V0lk3n) for pointing me in the right direction and to [Tomasz Pograniczny](https://github.com/Svirusx) (aka Svirusx) for the support on getting the kernel to work.

Happy hunting. :)
