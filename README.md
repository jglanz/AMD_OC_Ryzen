# OpenCore EFI for ASRock X570 Creator

This repository provides the basic contents for an EFI folder to successfully boot an ASRock X570 Creator motherboard,
using a Ryzen 9 CPU such as a 3950X. The contents work with either Mojave or Catalina. The intended SMBIOS is iMacPro1,1
although provisions are available for running MacPro7,1 which will be described below.

This repository is only designed for OpenCore bootloader. OpenCore (OC) is best updated via Pavo's OCBuilder app
(https://github.com/Pavo-IM/ocbuilder/releases). Accordingly, once you have a working EFI boot folder based on this repository, you can update various components of it as you see fit based on OCBuilder. But do be careful not to over write files or folders unique to this build. If updated, please study the Docs to see if the structure of the config.plist file needs to be changed (this respository will attempt to be current with the most stable release). Keep in mind that OpenCore is evolving, and consequently, new versions can substantially effect the overall structure and functioning of the presently used config.plist file.

This repository will attempt to keep an up-to-date basics (ACPI and Kexts) of an EFI folder that will work on an established computer. However, for details regarding how to setup OpenCore, how to create a bootable installation, how to trouble shoot errors, how to optimize your system and other related matters, please see the AMD-OSX/AMD-Vanilla repository at https://github.com/AMD-OSX/AMD_Vanilla/tree/master and in particular, https://khronokernel.github.io/Opencore-Vanilla-Desktop-Guide/ for loads of helpful details.

The EFI folder in this repository should be placed on the EFI partition of your boot drive (see Usage and Structure information below).

While the external menu by NDK is noted at the bottom of this page, this repository will not use it nor provide sample files. It is entirely optional and not necessary for OC functionality. Please refer to the Discussion section D below if you are interested in this external menu system.

Since the same version number can exist over a brief period of time until the final released version of that same number, it is best to reference a date with respect to the version being discussed. The present EFI folder is: 

***v056 - 3/1/2020***



### A) Contents

#### 1 ACPI

The first few files, those without a "-X570- prefix, are for setting up AGPM injector, and EC with other power adjustments. The x-AmdTable are fixes found by CaseySJ to ASRock SSDT mistakes. Meanwhile, the -NVMe- and -AQC107 files adjust the device names and correct internal drives appearing as external icons. THe -BXBR and -GP13 rename the USB devices. The GPU SSDT primarily provide correct renaming and nice displays of the drivers within SystemInformaion/PCI on the Mac, but a few adjust the functioning of the PowerTables (Vega 56 and 64).

The other two NVMe files listed as GPP0-ANS3 and GPP2-ANS3 are described below in section A5.

The SSDT-X570-TB3-basic.aml file injects the correct XHC5 setting for USB3 functionality and renames the TB nodes. While TB3 is working, it is still incomplete: the TB deice must be connected before boot and there is no hot-plug capability. Check the discusson sites listed below for current updates. Hopefully, the only update required to make TB3 fully functional will be an more complete SSDT-TB file than the one presently being used.


#### 2 Kexts

The contents of the Kexts folder can be broken down into various groups. First are the AGPMInjector kexts (created by
Pavo's AGPMInjector app (https://github.com/Pavo-IM/AGPMInjector/releases). Here is provided several variations to save 
time based on commonly used GPUs. These should be paired with similarly named files within the ACPI folder and each should
be enabled within the config.plist file as required. Example: selected AGPMInjector-iMacPro1,1-RX580.kext and SSDT-X570-RX580-slot-1.aml as pairs, ensuring both are entered in the proper sections of the config.plist file and that both are enabled.

Other groupings within the Kexts folder include the BT/Wifi kexts: AirportBrcmFixup, BrcmBluetoothInjector, BrcmFirmwareData, BrcmPatchRAM3, and BT4LEContinuityFixup. If you've swapped out the stock Intel BT module for a Mac-compatible version (as described here https://forum.amd-osx.com/viewtopic.php?p=53060#p53060), you'll want all of these enabled within the config.plist file. If you've added a PCIe BT/WiFi card such as the Fenvi FV-T919 (which has a Broadcom 94360CD), then most of these are optional. Other files will vary whether you're using a swapped BT (SBT) or PCIe BT (PCIeBT). These changes will be described below.

The other grouping are the essential kexts: AppleALC, AppleMCEReporterDisabler, Lilu, SmallTreeIntel82576_mod, VirtualSMC and WhateverGreen (WEG). MacProMemoryNotificationDisabler is required for SMBIOS MacPro7,1. Within the config.plist file, in the Kernel section, Lilu must be first in order, followed by VirtualSMC. Similarly, WEG should be present before other graphics related kext files.

NVMeFix (https://github.com/acidanthera/NVMeFix), ThunderboltReset (https://github.com/osy86/ThunderboltReset) and SMCAMDProcessor (https://github.com/trulyspinach/SMCAMDProcessor) are useful for adjusting the functioning of NVMe drives, setting up power for TB3, and providing CPU temperature and frequency information. ACPIDebug (and the companion SSDT-RMDT.aml file) will only be used to debug and trouble shoot TB3 SSDTs at a future date. Both of these files should presently be disabled and may be deleted at your choice.

The above kext files may be updated independent of this repository using Hackintool, Kext Updater or OCBuilder. However, the kext files described in the next paragraph are unique to this build and should not normally need updating, especially by a third party source. Nor, should other USBPort kext files be used in conjunction with them.

The two other kext files included are USBPort injectors: USBPorts-X570-ASRock-CR and USBPorts-X570-ASRock-CR-PCIe_BT. These files inject the proper USB ports. (These file are not to be updated by outside apps.) The former is for SBT and the latter for PCIeBT. The default within the config.plist file is for PCIeBT, not SBT. In parallel to these are the two ACPI aml files SSDT-X570-BXBR_BYUP_BYD8_XHC2-XHC and SSDT-X570-BXBR_BYUP_BYD8_XHC2-XHC-PCIe_BT. Again, the latter is the default and enabled (and the former is disbled) in the config.plist file. Together, the PCIeBt disable the internal Intel BT device (removing it's USB power supply) to as not to interfere with the PCIe BT add-on card, which should be at slot-5.

The Images folder contains a JPG of the main mobo layout including the slot descriptions. The other image shows the USB layout of the X570 Creator motherboard, the slots referenced above.


#### 3 BT Settings

To clarify the above description, the 2 sets hinted at above (you need to enable one, not both, within OpenCore):

SET 1. SBT - Internal swapped BT, enable following (but disable those in SET 2):
A. SSDT-X570-BXBR_BYUP_BYD8_XHC2-XHC.aml
B. USBPorts-X570-ASRock-CR.kext

SET 2. PCIeBT - PCIe BT module, enable following (but disable those in SET 1):
A. SSDT-X570-BXBR_BYUP_BYD8_XHC2-XHC-PCIe_BT.aml
B. USBPorts-X570-ASRock-CR-PCIe_BT.kext


#### 4 Drivers

Only a few drivers are needed with OC: ApfsDriverLoader, HSSPlus and FwRuntimeServices are required. AudioDxe is only needed if BootChime or other newly introduced audio features in OC are desired. The associated OC/Resources/Audio folder with the included WAV files are required for audio. The boot chime is the file OCEFIAudio_VoiceOver_Boot.wav. There many other WAV files in the Audio folder when you compile it, totally over 90MB. This can be too large for some EFI partitions, so all but the bare essentials are included in this repository. (See the Docs/Configuration.pdf for details on how to set up the audio features.)

VirtualSMC.efi is now part of OC. It, along with other settings in the config.plist file are required if you choose to use FileVault. This repository does not use FileVault and so those settings and other efi files are not included.


#### 5 Problems with TB enabling and the M2_2 site (an X570 problem) - disapparing drives

If the PCIe slot 6 is populated with an NVMe SSD in a PCIe adapter and if TB is enabled, the M2_2 drive will disappear from BIOS, which means the M2_2 drive is not available for booting. If the PCIe6 slot is empt and TB is still left enabled, the M2_2 SSD will be present in BIOS, and thus bootable. (See the included image of the motherboard: the M2_1 slot is closest to the CPU, while the M2_2 slot is farthest (and under the fan/shroud heatsink).

The problem is not just an ASRock issue, but can be found on MSI and ASUS forums, where people complain of 'disappearing' M2_2 drives. The problem lies with the X570 chip and how the PCIe and M2_2 slots are shared (lane sharing). While the M2_2 slot can disappear from BIOS, preventing booting from the drive in this slot, the drive will appear in the Finder and be available for reading and writing.

Meanwhile, the PCIe1 slot is direct from the cpu and only has issues if another GPU is placed in PCIe4 (PCIe1 goes from x16 to x8 when PCIe4 contains a GPU).

To summarize:

+ PCIe6 + TB enabled  --->  no M2_2 in BIOS
- PCIe6 + TB enabled  --->  M2_2 will appear in BIOS
+ PCIe6 + TB disabled --->  M2_2 will appear in BIOS
- PCIe6 + TB disabled --->  M2_2 will appear in BIOS

One other problem appears when TB is enabled. When TB is enabled, the M2_1 slot device is assigned to GGP2. However, if TB is disabled, teh M2_1 slot is assigned to GPP0. This is reflected in the two SSDT aml files known as SSDT-X570-NVMe-GPP0-ANS3-noTB.aml and SSDT-X570-NVMe-GPP2-ANS3+TB.aml. Both can actually be left enabled within OpenCore and either one will activate based on whether you have TB enabled or disabled.


#### 6 BIOS ROM

Working within a PC environment means using BIOS and the manufacturer's typcial boot methods as expected. However, if we'd like a more Mac-like flavor, how about changing the usual boot message from the manufacturer to a more Mac-like experience? This can be done through a modified BIOS. The downloadable BIOS (X570CTR2-10-mod.rom.zip) is the latest v2.10 but instead of the ASRock logo, it displays an Apple inspired logo. 

![Test Image 1](Images/AppleLogo_small.jpg)

Of course, you then need to follow the X570 Creator manual in flashing this BIOS to the mobo. If you stored any settings for v2.10 on a drive, you can re-load these settings once you've flashed this BIOS. If you didn't save your settings externally, you'll have to re-enter all of your settings again: so prepare things ahead of time to make flashing easier. (When you flash a newer version of BIOS, settings cannot be re-loaded, but going within a version, you can re-load settings.)


#### 7 BIOS Settings

If you load the included file, Auto+TB-CSM.bin, from within BIOS v2.10 (see motherboard manual on how to do this), you will have the stock settings. The Auto portion refers to the fact that XMP has not been set, but left at Auto. Do note that this v2.10 settings file has manually reduced fan speeds; please adjust as necessary.

Also, on the Advanced\AMD PBS page, in addition to enabling TB, the PCIe lanes were set to Gen3. Reportedly the Gen3 setting is better for maxmizing performance for currently available GPUs. Experiment with the Gen3 vs Auto setting and see what works best for your build. Except for these discussed items, all other BIOS settings were stock.

|                    |              |
| ------------------ | ------------ |
| TB                 |  Enabled     |
| Security Level     |  No Security |
| CSM                |  Disabled    |
| Above 4G decoding  |  Enabled     |



#### 8 SMBIOS - How to Easily Update in OC

Normally, typing in SMBIOS data can be a pain. But in OC, once you've entered it, it is very easy to copy and paste between config files.

The images below show the steps. When editing, use PlistEdit Pro, Xcode or ProperTree.

- Backup the config files in case you make a mistake.
- Open both files (and each file should not have the same name).
- Highlight and copy the old SMBIOS section (PlatformInfo) that has your working SNs, etc.
- Go to the new config file that has a SMBIOS with no SNs and highlight its PlatformInfo section.
- Do a paste, which leaves you with a file like the one shown in the other image below.
- Delete the PlatformInfo section marked "PlatformInfo" (highlight and press the delete key). 
- Highlight and click into the remaining section marked "PlatformInfo 2", editing out the space and 2 (" 2").
- Then save the file.

![Test Image 2](Images/OC_copy.jpg)

![Test Image 3](Images/OC_paste.jpg)



### B) Usage
 
- To build OpenCore using Pavo's OCBuilder. It is recommended to use the Release version.
- Move included folders for ACPI, Drivers, Kexts and the plist files into EFI/OC folder created by OCBuilder
- NOTE: the config.plist file does not contain SNs but place-holders that say "FILL-IN". You must supply these values on
        your own (here, a copy of Clover can be useful for deriving un-used SNs and UUIDs.
- Again, editing of config.plist files should only be done with PlistEdit Pro, Xcode or ProperTree (see Credits section for link).
- There is a file named config-Only-For-Storage.plist. This file stores data that can be copy and pasted to the main
        config.plist file. For example, inside is an entry "PlatformInfo-MacPro7,1". With both files open, you can high- 
        light and copy this section from the storage file to your config.plist file, pasting immediately below your current
        PlatformInfo section. You can then remove the original PlatformInfo, replacing it with PlatformInfo-MacPro7,1. Then
        rename PlatformInfo-MacPro7,1 as PlatformInfo. Next, provide new SNs and UUID values for this section. (Alternately,
        you can enter SNs and UUIDs into the storage portion and keep sets of SN-entered PlatformInfo sections ready for
        either iMacPro1,1 or MacPro7,1, switching as needed with little effort. (See A8 above for more details.) 
        Other items stored are DevicePropertiesfor swapped or PCIe BT modules. These are all present for convenience; 
        they are not required. 
- Remember, the EFI folder, containing the Boot and OC folders, goes onto the EFI partition of your boot drive. 
        Don't make the rookie mistake of placing the Boot and OC folders directly onto the EFI partition: this won't boot.
- Final EFI folder should have a structure as shown below (OC v056 as of 3/1/2020).

### C) EFI Folder

       EFI----Boot----Bootx64.efi
        |
        |_____OC_____ACPI
               |       |____various *.aml files
               |
               |_____config.plist
               |
               |_____config-Only-For-Storage.plist
               |
               |_____Docs
               |       |_____AcipSamples
               |       |          |_______various SSDT files
               |       |
               |       |_____Changelog.md, Configuration.pdf, Differences.pdf, Sample.plist, SampleFull.plist
               |
               |_____Drivers
               |       |______ApfsDriverLoader.efi, AudioDxe.efi, FwRuntimeServices.efi, HFSPlus.efi
               |
               |_____Kexts
               |       |______various *.kext files
               |
               |_____OpenCore.efi
               |
               |_____Resources
               |       |_____Audio
               |                |____ various WAV files
               |_____Tools
               |       |______BootKicker.efi, CleanNvram.efi, GopStop.efi, HdaCodecDump.efi, Shell.efi, VerifyMsE2.efi
               |
               |_____Utilities
                       |_____BootInstall
                       |
                       |_____CreateVault
                       |
                       |_____LogoutHook
                      
                          
    

### D) Discussion

- [X570 Creator](https://forum.amd-osx.com/viewtopic.php?f=35&t=9645) for AMD TB3 updates
- [OpenCore Discussion](https://www.insanelymac.com/forum/topic/338516-opencore-discussion/?page=1) for general OC issues
- [NDK Customized OC Menu](https://www.insanelymac.com/forum/topic/341402-customized-opencore-with-additional-features/) for NDK OC menu issues


### Credits

- [AlGrey](https://github.com/AlGreyy) for the idea and creation of the AMD patches
- [AMD OS X](https://github.com/AMD-OSX/AMD_Vanilla/tree/master) for AMD related information
- [CaseySJ](https://www.tonymacx86.com/threads/success-gigabyte-designare-z390-thunderbolt-3-i7-9700k-amd-rx-580.267551/) for tireless work and ingenuity in TB decoding
- [Download-Fritz](https://github.com/Download-Fritz) for OpenCore
- [Hackintool](https://www.insanelymac.com/forum/topic/335018-hackintool-v286/) for Hackintool utility
- [Kext Updater](https://bitbucket.org/profdrluigi/kextupdater/downloads/) for Kext Updater utility
- [khronokernel](https://khronokernel.github.io/Opencore-Vanilla-Desktop-Guide/) for a great OC guidebook
- [NDK OC Menu](https://github.com/n-d-k/NdkBootPicker) for NDKBootPicker Menu for OC
- [Pavo](https://github.com/Pavo-IM) for OCBuilder and AGPMInjector
- [CorpNewt](https://github.com/corpnewt/ProperTree) for many things including ProperTree editor
- [trulyspinach](https://github.com/trulyspinach/SMCAMDProcessor) for CPU Temp/Freq monitoring
- [vit9696](https://github.com/vit9696) for OpenCore and many of the kexts we use
