# OpenCore EFI for ASRock X570 Creator

This repository provides the basic contents for an EFI folder to successfully boot an ASRock X570 Creator motherboard,
using a Ryzen 9 CPU such as a 3950X. The contents work with either Mojave or Catalina. The intended SMBIOS is iMacPro1,1
although provisions are available for running MacPro7,1 which will be described below.

This repository is only designed for OpenCore bootloader. OpenCore (OC) is best updated via Pavo's [OCBuilder](https://github.com/Pavo-IM/ocbuilder/releases) app. Accordingly, once you have a working EFI boot folder based on this repository, you can update various components of it as you see fit based on OCBuilder. But do be careful not to over write files or folders unique to this build. If updated, please study the Docs to see if the structure of the config.plist file needs to be changed (this respository will attempt to be current with the most stable release). Keep in mind that OpenCore is evolving, and consequently, new versions can substantially effect the overall structure and functioning of the presently used config.plist file.

This repository will also attempt to keep up-to-date the basics of this EFI folder unique to this build (in particular, the ACPI and Kexts folders). This respository assumes you are fine tuning an established build. If you are looking for details regarding how to setup OpenCore, how to create a bootable installation, how to trouble shoot errors, how to optimize your system and other related matters, please see the AMD-OSX/AMD-Vanilla [repository](https://github.com/AMD-OSX/AMD_Vanilla/tree/master) and especially, the AMD section in the very detailed [OpenCore-Guide](https://khronokernel.github.io/Opencore-Vanilla-Desktop-Guide/).

The EFI folder in this repository should be placed on the EFI partition of your boot drive (see Usage and Structure information below).

While the external menu by NDK is noted at the bottom of this page, this repository will not use it nor provide sample files. It is entirely optional and not necessary for OC functionality. Please refer to the Discussion section E below if you are interested in this external menu system.

OpenCore version numbers are not incremented for each minor adjustment, but incremented once stable. These small changes within a version can have marked structural changes and yet not be fully documented. Accordingly, it is best to use final release versions. Due to the sometimes daily changes, this repository will only upload changes if the commit seems stable and then note the date of compilation along with the version number. The present EFI folder is: 

***v056 - 3/3/2020***



## A. Contents

### 1. ACPI

The first few files, those without a "-X570- prefix, are for setting up AGPM injector, and EC with other power adjustments. The x-AmdTable are fixes found by CaseySJ to ASRock SSDT mistakes. Meanwhile, the -NVMe- and -AQC107 files adjust the device names and correct internal drives appearing as external icons. The SSDTs -BXBR and -GP13 rename the USB devices. 

The GPU SSDT files, such as SSDT-X570-RX580-slot-1.aml, primarily provide correct re-naming of the devices (although much is provided by WEG) and nice displays of the drivers within SystemInformaion/PCI on the Mac. Two of these do adjust the functioning of the PowerTables (Vega 56 and 64). These SSDT GPU files do not inject AGPM; that is provided through kext files described in the next section.

The other two NVMe files listed as GPP0-ANS3 and GPP2-ANS3 are described below in section A5.

Finally, the SSDT-X570-TB3-basic.aml file injects the correct XHC5 setting for USB3 functionality and renames the TB nodes. While TB3 is working, it is still incomplete: the TB device must be connected before boot and there is no hot-plug capability. Check the discusson sites listed below for current updates. Hopefully, the only update required to make TB3 fully functional will be a more complete SSDT-TB file replacing the one presently being used.


### 2. Kexts

The contents of the Kexts folder can be broken down into various groups. 

First are the AGPMInjector kexts, which were made using Pavo's [AGPMInjector](https://github.com/Pavo-IM/AGPMInjector/releases) app. A few variations were created by selecting different, commonly used GPUs, while keeping the SMBIOS set at iMacPro1,1. These different versions allow flexible selection by the user. The AGPMInjector kext should be paired with a similarly named SSDT-GPU file within the ACPI folder. That is, you use one SSDT-GPU file for your selected GPU and one AGPMInjector kext specific for that same GPU. These should be entered and enabled within the ACPI and Kernel sections of the config.plist file. Example (shown below): SSDT-X570-RX580-slot-1.aml and AGPMInjector-iMacPro1,1-RX580.kext both enabled as a pair in the config.plist file.

SSDT-RX580:
![Test Image 1](Images/SSDT-RX580.jpg)

AGPMInjector:
![Test Image 2](Images/AGPMInj-RX580.jpg)


Other groupings within the Kexts folder include the BT/Wifi kexts: AirportBrcmFixup, BrcmBluetoothInjector, BrcmFirmwareData, BrcmPatchRAM3, and BT4LEContinuityFixup. If you've swapped out the stock Intel BT module for a Mac-compatible version, as described here [Swapping BT Module](https://forum.amd-osx.com/viewtopic.php?p=53060#p53060), you'll want all of these enabled within the config.plist file. On the other hand, if you've added a PCIe BT/WiFi card such as the Fenvi FV-T919 (which has a Broadcom 94360CD), then most of these kext files are optional. A few other files will vary depending on whether you're using a swapped BT (SBT) or PCIe BT (PCIeBT). Those changes will be described below.

Yet another grouping are the essential kexts: AppleALC, AppleMCEReporterDisabler, Lilu, SmallTreeIntel82576_mod, VirtualSMC and WhateverGreen (WEG). MacProMemoryNotificationDisabler is required for SMBIOS MacPro7,1. Within the config.plist file, in the Kernel section, Lilu must be first in order, followed by VirtualSMC. Similarly, WEG should be present before other graphics related kext files.

[NVMeFix](https://github.com/acidanthera/NVMeFix), [ThunderboltReset](https://github.com/osy86/ThunderboltReset) and [SMCAMDProcessor](https://github.com/trulyspinach/SMCAMDProcessor) are useful for adjusting the functioning of NVMe drives, setting up power for TB3, and providing CPU temperature and frequency information. ACPIDebug (and the companion SSDT-RMDT.aml file) will only be used to debug and trouble shoot TB3 SSDTs at a future date. Both of these files should presently be disabled and may be deleted at your choice.

The above kext files may be updated independent of this repository using Hackintool, Kext Updater or OCBuilder. However, the final kext group described in the next paragraph are unique to this build and should not normally need updating, especially by a third party source. Nor, should other USBPort kext files be used in conjunction with them.

This final group consists of two USBPort injector kext files specific for this motherboard: USBPorts-X570-ASRock-CR and USBPorts-X570-ASRock-CR-PCIe_BT. The former is for SBT builds; the latter, for PCIeBT builds. The repository default within the config.plist file is for PCIeBT, not SBT, builds. 

Use one of these two USBPort injector kext files in parallel one of two ACPI files: SSDT-X570-BXBR_BYUP_BYD8_XHC2-XHC and SSDT-X570-BXBR_BYUP_BYD8_XHC2-XHC-PCIe_BT. Again, the former is for SBT builds and by default is disabled; the latter is for PCIeBT builds and is by default enabled in the config.plist file. This pairing is re-explained below in section A3.

Together, these SSDT and kext files properly inject the USB ports, while in the case of PCIeBT version, disabling the internal Intel BT device (removing it's USB power supply) to as not to interfere with the BT add-on card, ideally located at slot-5. (See the included Images folder for JPGs of the main mobo layout and the rear panel USB/Internal USB layout.)


### 3. BT Settings

To clarify the above description, there are 2 sets of ACPI and kext files that you need to use. You need to enable one, not both, within OpenCore. The two sets can be described as follows:

SET 1. SBT - Internal swapped BT, enable following (but disable those in SET 2):

    A. SSDT-X570-BXBR_BYUP_BYD8_XHC2-XHC.aml

    B. USBPorts-X570-ASRock-CR.kext

SET 2. PCIeBT - PCIe BT module, enable following (but disable those in SET 1):

    A. SSDT-X570-BXBR_BYUP_BYD8_XHC2-XHC-PCIe_BT.aml

    B. USBPorts-X570-ASRock-CR-PCIe_BT.kext
    
The images below show the 2 sections, the ACPI and the Kernel sections, in the config.plist file, where these files are to be enabled or disabled.

ACPI section:
![Test Image 3](Images/ACPI-X570X-USB-BT.jpg)

Kernel section:
![Test Image 4](Images/USBPorts-X570.jpg)


### 4. Drivers

Only a few drivers are required with OpenCore: ApfsDriverLoader and FwRuntimeServices. Even HSSPlus is optional, but useful. AudioDxe, a new addition for OpenCore, is only needed if BootChime or some of the other newly introduced audio features are desired. The OC/Resources/Audio folder with its included WAV files are required for audio. The boot chime is the file OCEFIAudio_VoiceOver_Boot.wav. There are many other WAV files in the Audio folder when OC is freshly compiled; in face, over 90MB worth. Since this size can be too large for some EFI partitions, it was elected to remove all but the most rudimentary audio files from this folder for this repository. (See the Docs/Configuration.pdf for details on how to set up the audio features.) If you wish to have more WAV files, then compile OC on your own with OCBuilder and add them.

VirtualSMC.efi is now part of OpenCore. This file, along with various settings in the config.plist file, are required if you choose to use FileVault. This repository does not use FileVault and so those settings along with any associated files will be neither discussed nor included. If you wish to use FileVault, then read the documentation and adjust the config.plist as needed.


### 5. Problems with TB enabling and the M2_2 site (an X570 problem) - disapparing drives

If the PCIe slot 6 is populated with an NVMe SSD in a PCIe adapter and if TB is enabled, the M2_2 drive will disappear from BIOS, which means the M2_2 drive is not available for booting. If the PCIe6 slot is empty and TB is enabled, the M2_2 SSD will be present in BIOS, and thus bootable. When the M2_2 slot has disappeared from BIOS, the M2_2 drive will nevertheless appear in the Finder and be available for reading and writing. (See the included image of the motherboard: the M2_1 slot is closest to the CPU, while the M2_2 slot is farthest from the CPU, under the fan/shroud heatsink.)

The problem is not just an ASRock issue, but can be found on MSI and ASUS forums, where people complain of 'disappearing' M2_2 drives. The problem lies with the X570 chip and how the PCIe and M2_2 slots are lane-shared. Meanwhile, the PCIe1 slot is direct from the cpu and only has issues if another GPU is placed in PCIe4 (PCIe1 goes from x16 to x8 when PCIe4 contains a GPU).

To summarize:

+ PCIe6 + TB enabled  --->  no M2_2 in BIOS
- PCIe6 + TB enabled  --->  M2_2 will appear in BIOS
+ PCIe6 + TB disabled --->  M2_2 will appear in BIOS
- PCIe6 + TB disabled --->  M2_2 will appear in BIOS

One other problem appears when TB is enabled. When TB is enabled, the M2_1 slot device is assigned to PCI0/GGP2. However, if TB is disabled, teh M2_1 slot is assigned to PCI0/GPP0. This is reflected in the two SSDT aml files known as SSDT-X570-NVMe-GPP0-ANS3-noTB.aml and SSDT-X570-NVMe-GPP2-ANS3+TB.aml. Both can actually be left enabled within OpenCore and either one will activate based on whether you have TB enabled or disabled.


### 6. BIOS ROM

Working within a PC environment means using BIOS and the manufacturer's typcial boot methods, which includes their logo. If we'd like a more Mac-like tone, what about changing the usual manufacturer's boot logo to one that is more Mac-like? This can be done through a modified BIOS. The BIOS included in this repository (X570CTR2-10-mod.rom.zip) is the latest v2.10 BIOS but contains a modified boot logo as shown below. 

![Test Image 5](Images/AppleLogo_small.jpg)

Of course, to use you need to follow the X570 Creator manual in how to flash a BIOS to the X570 Creator motherboard. If you stored any settings for v2.10 on a drive, you can re-load these settings once you've flashed this BIOS (a settings file is also included in this repository). If you didn't save your settings externally, you'll have to re-enter all of your settings again: so prepare things ahead of time to make flashing easier. (When you flash a different version of a BIOS, settings cannot be re-loaded from another version; however, when staying within a version, you can re-load settings.)


### 7. BIOS Settings

If you load the included file, Auto+TB-CSM.bin, from within BIOS v2.10 (see motherboard manual on how to do this), you will have the basic optimal settings for this motherboard with TB enabled. The "Auto" portion in the file name refers to the fact that XMP has not been set, but left on Auto. Do note that this v2.10 settings file has manually reduced fan speeds; please adjust as necessary. Also, please confirm your boot order is correct under the BIOS Boot menu item before re-saving the settings.

Also, on the Advanced\AMD PBS page, in addition to enabling TB, the PCIe lanes were set to Gen3. Reportedly, the Gen3 setting is better for maxmizing performance from currently available GPUs. Experiment with the Gen3 vs Auto setting and see what works best for your build. Except for these discussed items, all other BIOS settings are stock.

|                    |              |
| ------------------ | ------------ |
| TB                 |  Enabled     |
| Security Level     |  No Security |
| CSM                |  Disabled    |
| Above 4G decoding  |  Enabled     |



### 8. SMBIOS - How to Easily Update in OC

SMBIOS data can be generated using an old copy of Clover (but do NOT use Clover to edit the config.plist files for OpenCore), or using the recommended [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS).

If you already have SNs and UUIDs values in an existing OpenCore config file, then cloning that SMBIOS data is easy. OC allows you to simpliy copy and paste sections, such as the PlatformInfo section, between config files.

The images below show the steps. When editing the config.plist file, the recommended editors are PlistEdit Pro, Xcode or [ProperTree](https://github.com/corpnewt/ProperTree).

- Backup the config files before starting.
- Open both files you're to copy between.
- Highlight and copy the old SMBIOS section (PlatformInfo) that has your working SNs, etc.
- Go to the new config file that has a SMBIOS with no SNs and highlight its PlatformInfo section.
- Do a paste, which leaves you with a file like the one shown in the image below.
- Delete the PlatformInfo section marked "PlatformInfo" (highlight and press the delete key). 
- Highlight and click into the remaining section marked "PlatformInfo 2", editing out the space and 2 (" 2").
- Then save the file.

![Test Image 6](Images/OC_copy.jpg)

![Test Image 7](Images/OC_paste.jpg)



## B. Usage
 
- To build OpenCore using Pavo's OCBuilder, it is recommended to use the Release version with or without kexts update.
- Move the repository included folders for ACPI, Kexts and the plist files into EFI/OC folder created by OCBuilder.
- Verify that the proper driver efi files are in place, based on what is indicated within the config.plist file.
- NOTE: the config.plist file does not contain SNs or UUIDs but place-holders that say "FILL-IN". You must supply these 
        values on your own (see section A8 above for details).
- Again, editing of config.plist files should only be done with PlistEdit Pro, Xcode or [ProperTree](https://github.com/corpnewt/ProperTree).
- There is a file named config-Only-For-Storage.plist. This file stores data that can be copy and pasted to the main
        config.plist file. For example, inside is an entry "PlatformInfo-MacPro7,1". With both files open, you can high- 
        light and copy this section from the storage file to your config.plist file, pasting immediately below your current
        PlatformInfo section. You can then remove the original PlatformInfo, replacing it with PlatformInfo-MacPro7,1. Then
        rename PlatformInfo-MacPro7,1 as PlatformInfo. Next, provide new SNs and UUID values for this section. (Alternately,
        you can enter SNs and UUIDs into the storage portion and keep sets of SN-entered PlatformInfo sections ready for
        either iMacPro1,1 or MacPro7,1, switching as needed with little effort. (See A8 above for more details.) 
        Other items stored in this file are DevicePropertiesfor swapped or PCIe BT modules. These are all present for convenience; 
        they are not required. 
- Remember, the EFI folder, containing the Boot and OC folders, goes onto the EFI partition of your boot drive. 
        Don't make the rookie mistake of placing the Boot and OC folders directly onto the EFI partition: this won't boot.
- Finally, the EFI folder should have a structure as shown below (current version as posted at the beginning of this README file).

## C. EFI Folder Structure

       EFI____Boot___Bootx64.efi
        |
        |_____OC_____ACPI
               |       |____ *.aml files
               |_____config.plist
               |_____config-Only-For-Storage.plist
               |_____Docs
               |       |_____AcipSamples
               |       |          |_______various SSDT files
               |       |_____Changelog.md, Configuration.pdf, Differences.pdf, Sample.plist, SampleFull.plist
               |_____Drivers
               |       |______ApfsDriverLoader.efi, AudioDxe.efi, FwRuntimeServices.efi, HFSPlus.efi
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
               |_____Utilities
                       |_____BootInstall
                       |_____CreateVault
                       |_____LogoutHook
                      
                          
## D. Config.plist Structure (under construction)

       config.plist
            |
            |_____ACPI
            |       |____Add: *.aml files (discussed above in section A1)
            |       |____Block, Patch, Quirks: all inactive at this time
            |_____Booter
            |       |____MmioWhitelist: not active
            |       |____Quirks: YES: AvoidRuntimeDefrag, EnableSafeModeSlide, EnableWriteUnprotector, ProvideCustomSlide, SetupVirtualMap
            |                     NO: all others
            |_____DeviceProperties
            |       |____Add: I211 Controller, BT Controller (slot 5), Realtek ALC1220 Audio Controller
            |       |____Block: inactive at this time
            |_____Kernel
            |       |____Add: *.kext files (discussed above in section A2)
            |       |____Block, Emulate: inactive at this time
            |       |____Patch: all of the Al Grey secret sauce for AMD CPU
            |       |____Quirks: YES: DisableIoMapper, DummyPowerManagement (esp for AMD build), ExternalDiskIcons, PanicNoKextDump, PowerTimeoutKernelPanic
            |                     NO: rest
            |_____Misc
            |       |____BlessOverride: inactive
            |       |____Boot: HibernateMode AUTO, HideAuxiliary YES, HideSelf YES, PickerAttributes 4 (red), PickerAudioAssist NO, PickerMode Builtin, PollAppleHotKeys NO, ShowPicker YES, TakeoffDelay 100, Timeout 10
            |       |____Debug: DisableWatchDog YES, DisplayDelay 0, DisplayLevel 64, Target 65 (last 2 allow for min text warnings)
            |       |____Entries: inactive (req. running Debug version to identify drive addresses)
            |       |____Security: AllowNvramReset YES, AllowSetDefault YES, AuthRestart NO, ExposeSensitiveData 14, HaltLevel 2147483648, ScanPolicy 2820355, Vault Optional
            |       |____Tools: YES for UEFI Shell
            |_____NVRAM
            |       |____Add
            |       |      |____4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14
            |       |      |                                      |____DefaultBackgroundColor 00000000 (black), UIScale 01
            |       |      |____7C436110-AB2A-4BBB-A880-FE41995C9F82
            |       |                                             |____boot-args: keepsyms=1 debug=0x100 shikigva=80 (for sidecar/Catalina)
            |       |                                             |____csr-active-config: E7030000 
            |       |                                             |____nvda_drv: 31 
            |       |                                             |____prev-lang:kbd: 656E5F55 533A30 
            |       |____Block: inactive at this time
            |       |____LegacyEnable: NO
            |       |____LegacyOverwrite: NO
            |       |____LegacySchema: (see Docs)
            |       |____WriteFlash: YES
            |_____PlatformInfo (pending)
            |_____UEFI
                    |____Audio: YES
                    |____ConnectDrivers: YES
                    |____Drivers: HFSPlus, ApfsDriverLoader, FwRuntimeServices, AudioDxe
                    |____Input: KeyForgetThreshold 5, KeyMergeThreshold 2, KeySupport YES, KeySupportMode Auto, KeySwap NO, PointerSupport NO, PointerSupportMode (blank), TimerResolution 50000
                    |____Output: ProvideConsoleGop YES, ConsoleMode and Resolution left blank; rest NO
                    |____Protocols: all NO
                    |____Quirks: ExitBootServicesDelay 0, RequestBootVarFallback YES, RequestBootVarRouting YES; rest NO
            

## E. Discussion

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
- [CorpNewt](https://github.com/corpnewt) for many things such as GenSMBIOS and ProperTree editor
- [trulyspinach](https://github.com/trulyspinach/SMCAMDProcessor) for CPU Temp/Freq monitoring
- [vit9696](https://github.com/vit9696) for OpenCore and many of the kexts we use
