w
j# Developing and testing on Windows

This is a sketch of what I do to test builds on Windows.

Like most Python developers, I do not usually use Windows.  In fact I use Mac
most of the time, and Linux sometimes via Docker.

This page covers the various ways I get a Windows shell or GUI, in decreasing
order of usefulness, at least to me.

These are:

1. Booting my (Mac) computer into Windows.
2. Using a Windows virtual machine with a fixed-size virtual hard drive.
3. Running on a cloud virtual machine via Google Cloud or Azure.

## Booting into Windows

This is the best option for a long Windows session.  With a decent machine,
this is noticeably more responsive than the other options.

There are, of course, many ways of booting into Windows, including having a
machine dedicated to running Windows, but I find you really need more than 8GB
of RAM to be comfortable running Windows 10.

I boot Windows using an Intel Mac Mini desktop that I've upgraded to 32GB of
RAM. The Intel part is important, because you [can't run Apple Bootcamp on the
newer M1 Macs](https://support.apple.com/en-gb/HT201468).

There are two barriers to doing this, both of which can be fairly easily
overcome:

* The default method of installing Bootcamp asks for at least 64GB of *your
  main drive*, and in practice, you'll need more than this to run Windows
  comfortably, after installing the various toolkits you need.  I fixed this by
  following a recipe (below) to install and then boot Windows from a spare
  external hard drive.
* Windows historically either required you pay for a license, or use a testing
  install that expires after a few months.  It's annoying to pay, and it's
  annoying to have to keep re-installing Windows.  Fortunately, at least for
  the image I downloaded from the Microsoft website, it is possible to install
  Windows without an activation code corresponding to a license, and the
  resulting image does not seem to expire, and differs from a full copy only
  in giving you a nag message to enter an activation code, and preventing you
  customizing the desktop and GUI.

## Using Bootcamp on an external drive

In summary the trick is: install Windows as a virtual machine, in VirtualBox,
but using a [real hard disk as the backing
store](https://www.virtualbox.org/manual/ch09.html#rawdisk), instead of a
virtual hard disk.  Then boot to the real hard disk and finish the Windows
installation, including the various Bootcamp drivers.

The instructions I was working from are:

* [Alessandro Morandi's version of Sven Kirsimäe's
  page](https://medium.com/@simbul/how-to-boot-windows-from-an-external-hdd-ssd-on-a-macbook-bdcf99a721c7)
  below.
* [Sven Kirsimäe's version of Tom Nelson's page
  (below)](https://medium.com/@svenkirsime/install-windows-on-the-external-ssd-hdd-for-your-mac-5d29eefe5d1)
* [Tom Nelson's guide on installing via
  Virtualbox](https://eshop.macsales.com/blog/40947-tech-tip-how-to-use-boot-camp-on-an-external-drive).
* [A nice modern
  walkthrough](https://teachernerd.com/2020/01/21/installing-windows-10/) with
  instructions for what to do if you can't then boot from the disk.

Please do read Alessando's and Tom's pages.  Here I'm summarizing the steps
from the third page by Tom Nelson, with modifications from the other pages,
particularly Alessandro's.

You will need:

* A spare external drive of at least 64GB, on which to install Windows.  I was
  using a standard 128GB USB memory stick for this walkthrough.
* A drive with about 2GB spare that can be mounted *from Windows*.  You will
  use this to store the Apple Bootcamp drivers, for when you first boot your
  Mac into Windows.  For example, you might have a spare USB memory stick
  formatted as ExFAT, that you can access from Mac or Windows.
* A freely-downloadable [copy of Windows
  10](https://www.microsoft.com/en-gb/software-download/windows10ISO). This will come as an ISO file.
* A wired mouse — because Windows won't be able to use your laptop trackpad or
  Bluetooth mouse, until you have installed the Bootcamp drivers (above).  You
  will also need a wired or built-in keyboard — something other than a
  wireless keyboard, and that will work before you have installed drivers.

### To prepare

* Download the Windows 10 ISO file from the link above.  I selected options
  "Windows 10 (multi-edition ISO)", "English" as the language, and "64-bit
  Download".  The resulting file was `Win10_21H2_English_x64.iso`.  You'll be be using VirtualBox to install from this ISO file.
* Make sure you have the wired mouse and maybe keyboard, as above.
* I would unplug all external storage, other than the hard drive you will
  install on, and the USB stick .  At various points I found the install
  failed when running with some extra hard drives attached.

### Configuration file for VirtualBox to use the disk you will install to

Your task here is to find the disk number of the disk you will install to, then
ask VirtualBox to make a so-called VMDK file that VirtualBox can later use to
give your virtual machine information about the disk.

First find the disk number of the disk *you will install Windows to*.  Call
this this the WIN_DISK.

The following commands are all to be run from the Terminal.

Review the output of:

```
diskutil list
```

to identify the disk.  In my case the relevant disk turned out to be
`/dev/disk5`.  This was my 128GB memory stick.

**Be very careful - if you get the wrong disk here, you will overwrite the disk
destructively and lose data**.

**Be very careful - the disk number - e.g. the 5 in `/dev/disk5` - can change,
when you plug and unplug the disk, and when you reboot.  Always check after you
have plugged in the disk**.

```
# Uncomment and set your disk number here
# WIN_DISK=/dev/disk5
```

```
# Format the disk as FAT, GUID partition table.
diskutil eraseDisk MS-DOS MYWIN GPT $WIN_DISK
# Unmount it.
diskutil unmountDisk $WIN_DISK
# Make a VMDK file that will point VirtualBox to the disk.
sudo VBoxManage internalcommands createrawvmdk -filename bootcamp.vmdk -rawdisk $WIN_DISK
```

You next *must* start VirtualBox with root permissions - in order to get adequate access to the disk.  Otherwise, you will get permission errors when trying to add the hard disk VMDK file below.

```
# Unmount the disk again, just in case.
diskutil unmountDisk $WIN_DISK
# Start VirtualBox as root.
sudo /Applications/VirtualBox.app/Contents/MacOS/VirtualBox
```

### Set up new virtual machine to use the disk

* Choose "New" from the "Machine" menu, enter name to taste, such as
  MYWINMACHINE, specify as Windows 10, 64-bit.
* Give the machine as much much memory as your system can afford.  I gave it
  8GB.
* Select "Use an existing virtual hard disk file".  Click browse button.  Click
  Add. Select `bootcamp.vmdk` above.
* Click "Create"
* Under "Settings" -> "System" check the “Enable EFI (special OSes only)”.
* Under "Settings" select the "Storage" tab.  You should have an "Empty" icon
  within the "Controller: SATA" tree to the left, with a disc icon.  Click on
  this, then click on the disk icon to the left of the Optical Drive dropdown
  list box.  Select "Choose disk file", then select the Windows ISO file.  It
  should appear in the tree window to the left.
* In "Setting" -> "Storage" choose the main root controller and check the “Use
  Host I/O Cache” as enabled.  This is marked as "Potentially optional" in
  [Sven's
  guide](https://medium.com/@svenkirsime/install-windows-on-the-external-ssd-hdd-for-your-mac-5d29eefe5d1),
  but I twice had the Windows VirtualBox Windows install fail in writing files,
  without this setting.
* Select OK to save the settings for the machine.

### Use the virtual machine to install Windows on the disk

Before you continue, you really want to make sure that the Windows install
process doesn't force a reboot mid-install, as this will mess up your
subsequent boot.   You could sit and wait and watch the install, and when it
prompts you to tell you it is rebooting, you could shut down the machine, but
that's boring.   So, following Alessandro's page above, I highly recommend you:

* Shut down VirtualBox
*   Run:

    ```
    # Set your machine name below.
    sudo VBoxManage setextradata MYWINMACHINE "VBoxInternal/PDM/HaltOnReset" 1
    ```

*   Then restart VirtualBox as root:

    ```
    sudo /Applications/VirtualBox.app/Contents/MacOS/VirtualBox
    ```

* Click "Start" for your new virtual machine, to begin the install. If you get
  "Failed to open session", check that your "MYWIN" disk above is not mounted.
  I found that Mac frequently remounts the disk without me asking it to.
* You should next see "Press any key to boot from CD/DVD".  If you get dropped
  to the EFI prompt at this point, follow the instructions in [Sven K's
  page](https://medium.com/@svenkirsime/install-windows-on-the-external-ssd-hdd-for-your-mac-5d29eefe5d1).
  Or, I found stopping the machine, and restarting got me to the "Press any key
  to boot" prompt, and then direct to the Windows installation menu.
* I left the defaults for location settings at English (United States)", and US
  keyboard layout.  I bet you can select what you need there. "Next".
* "Install Now".
* For "Activate Windows" choose, "I don't have a product key".
* At "Select the operating system you want to install", I chose "Windows 10
  Pro".
* Chose option "Custom: install Windows only".
* For "Where do you want to install Windows", you should see two partitions, of
  which one is a small ~200MB EFI "System" partition, and the other is
  a "Primary" partition covering the rest of the disk.  Chose the "Primary"
  partition.  Before you click "Next", click "Format" to format the partition
  as NTFS, as Windows requires.
* Wait!
* When the process is finished, "Power off the machine".
* Reboot and reset the PVRAM.  I'm not sure whether this is always needed, but
  I was getting frequent freezes when booting into Windows before I did this.
* Reboot again, and hold the Alt key.  Select the EFI drive.
* Continue your Windows install by 

### Using a Windows virtual machine

Here you just follow the usual steps to make a Windows virtual machine, along
with a virtual disk file.  Use the ISO download linked above.  Choose "I don't
have a product key" as above.

I found the VM to be annoyingly slower than the native boot to Windows, even on
a VM with lots of memory.

It did seem to make the [VM more
responsive](https://www.techrepublic.com/article/how-to-improve-virtualbox-guest-performance-in-five-steps/)
when I set it up with a large (100GB) *fixed-size* virtual disk, instead of
the default dynamically resized disk

### Setting up a cloud machine

You can get free credits on Azure, but these expire after 1 month: see
<https://azure.microsoft.com/services/virtual-machines/windows/>.  It's easy to
set up a virtual machine from there, and then connect with Remote Desktop
Protocol (RDP) to get a Windows GUI.  It's fairly cheap to spin up and run one
of these for a short time, but you'll be on pay-as-you-go after your 1 month
expiry or you use up your free credits.

Google Cloud also allows you to spin up Windows virtual machines.  It also has
free credits, that last for 90 days, but Windows VMs don't qualify for the free
credits.  See <https://cloud.google.com>.  See
<https://cloud.google.com/compute/docs/quickstart-windows> for instructions on
spinning up and using a Windows Server VM.
