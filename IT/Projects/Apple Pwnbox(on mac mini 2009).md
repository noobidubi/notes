In this project I wanted to turn my apple mac mini into a pwnbox server using NoVNC. I also wanted to mimic the looks of the pwnbox using their mate config. Luckily Parrot provides a pwnbox image that already has the config, they also provide a Debian conversion script that contains the pwnbox config as well.
# Installing Parrot on mac
- ❌ **Failed Attempts:**
	- **using the official Parrot OS amd64 image:**
	 the ISO just failed to boot because of some secure boot issue that I couldn't fix due to the lack of BIOS settings
	- **following _[mattgadient's guide](https://mattgadient.com/linux-dvd-images-and-how-to-for-32-bit-efi-macs-late-2006-models/)_:** 
	 modding the parrot OS ISO failed because the guide is for older mac's(2006 models) and there's actually no need to use 32Bit EFI boot because the mac mini 2009(*on latest drivers*) already supports 64Bit EFI
after all of this trouble using the parrot OS image I just decided to use Debian and the Parrot Conversion Script

# Installing Debian on mac
Installing Debian was a Lot simpler than installing Parrot it was basically just using Belena etcher to create a bootable USB stick(using the Debian amd64 netinst image) plugging that into the mac mini and then booting from it

## Driver Issues
the Debian netinstaller images don't provide the `linux-firmware-nonfree` packages which are required to use the mac's onboard wifi chip in specific it was missing the b43 kernel module for that chip luckily these are already in the debian repository so all I had to do was
```bash
sudo apt update
sudo apt install firmware-linux-nonfree firmware-b43-installer
# dont install the b43leagacy drivers as they conflict with the main b43 drivers
# if they are installed just run
# sudo apt purge firmware-b43leagacy-installer
```

### Blacklisted Driver
Theres a chance that even if the b43 drivers are installed they still dont load into the kernel. This might be a conflict so we should first try manually loading it into the kernel and checking if that works
```bash
sudo modprobe -r b43 # unloads the kernel module
sudo modprobe b43 # try's to load the kernel module(if this fails it might be due to failed installation or conflicts)
lsmod | grep b43 # if its loaded but still not working it's most likley a conflict with other wifi drivers
```

If the kernel module works when manually loading it in but doesn't work after a reboot it might be caused by it either not being(*or commented out*) in the /etc/modules file or it being blacklisted in some other file. we can check this by looking at the file

```bash
cat /etc/modules | grep b43
echo 'b43' | sudo tee -a /etc/modules # adds the b43 kernel module to /etc/hosts file. Make sure to use -a to not overwrite the file
```

If this doesn't work it might be blacklisted

```bash
grep -r 'blacklist b43' /etc/modprobe.d/ # If you see it being blacklisted just remove the line or comment it out
```

## Using the [Parrot Conversion script](https://gitlab.com/parrotsec/project/debian-conversion-script)
just read the documentation for more info
```bash
git clone https://gitlab.com/parrotsec/project/debian-conversion-script.git
cd debian-conversion-script
sudo chmod +x ./install.sh
sudo ./install.sh
```
