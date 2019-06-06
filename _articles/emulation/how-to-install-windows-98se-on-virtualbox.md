# How to install Windows 98SE on VirtualBox

> Original article: <https://forums.virtualbox.org/viewtopic.php?f=28&t=59559>

## Prerequisities

* VirtualBox 5.2.8 (or later) plus extension pack
* [Small Patches Archive](http://www.mediafire.com/file/658ce4lfrzj6xxa/Win98se_Small_Patches.7z)
* [Medium Patches Archive](http://www.mediafire.com/file/4gjc2l0gc3s179z/Win98se_Medium_Patches.7z)
* [Auto-Patcher December 2007 Full](http://www.mdgx.com/spx/AP2007.EXE)
* [Auto-Patcher December 2008 Upgrade](http://www.mdgx.com/spx/AP2008UP.EXE)
* [Open Office 3.2.1](http://archive.apache.org/dist/incubator/ooo/stable/3.2.1/OOo_3.2.1_Win_x86_install_en-US.exe)

## Create "Utitlities" ISO

* Extract small patches, medium patches, auto-patchers and open office to one directory
* Keep all extracted files in root of the directory (to make automatic scripts work out of the box)
    ```console
    $ ls ./util-dir
    7z920.msi                           NDP1.1sp1-KB971108-X86.exe          Run2E.reg                           jre-6u22-windows-i586.exe
    AP2007.EXE                          OOo_3.2.1_Win_x86_install_en-US.exe Run2F.bat                           klmcodec345.exe
    AP2008UP.EXE                        PlainOldFavorites_1.3.1.xpi         Run2F.reg                           klmcp.ini
    Chair_Agenda.pdf                    Rain20_CPU_HLT_Driver               SumatraPDF-2.2.1-install.exe        msxml.msi
    Dictionaries.7z                     Realtek_Win95_AC97_Audio_Driver     Universal_VESA_VBE_Video_Driver     qtalt.ini
    Firefox Setup 10.0.12esr.exe        Run1D.bat                           VMware_Player                       quicktimealt156.exe
    Intel_Gigabit_Ethernet_Driver.exe   Run1E.bat                           Virtual_PC_2007                     sdd-win-7.0.0.340-beta.exe
    KernelEx-4.5.2.exe                  Run1F.bat                           dotnetfx.exe                        vcredist_x86.exe
    MenuReorg.bat                       Run2D.bat                           flashplayer10_3r183_90_winax.exe
    NDP1.1sp1-KB2833941-X86.exe         Run2D.reg                           gdiplus.dll
    NDP1.1sp1-KB867460-X86.exe          Run2E.bat                           install_flash_player_32bit.exe
    ```
* Create an ISO file from this directory
    * On MAC: `hdiutil makehybrid -o win98utils.iso ./util-dir -iso -joliet`

## Virtual Machine settings

Create a new virtual machine named "Win 98se" with (preferred but optional) 512MB of RAM and an 8GB or larger hard drive (fixed size drives are usually faster) then adjust the VM settings to match as closely as possible the list given below. The changes from default settings are in bold, if an item is mentioned then its matching box is ticked.

* General
    * Basic
        * Name: **Win 98se**
        * Type: Microsoft Windows
        * Version: Windows 98
    * Advanced
        * Snapshot Folder: <Default Setting>
        * Shared Clipboard: Disabled
        * Drag'n'Drop: Disabled
    * Encryption
        * Not Enabled
* System
    * Motherboard
        * Base Memory: **512MB**
        * Boot Order: **Floppy, Optical, Hard Disk**
        * Chipset: PIIX3
        * Pointing Device: **USB Tablet**
        * Extended Features: **Enable I/O APIC** (only)
    * Processor
        * Processor(s): 1 CPU
        * Execution Cap: 100%
        * Extended Features: **Enable PAE/NX**
    * Acceleration
        * Paravirtualization Interface: Default
        * Hardware Virtualization: Enable VT-x/AMD-V & Enable Nested Paging
* Display
    * Screen
        * Video Memory: **64MB**
        * Monitor Count: 1
        * Scale Factor: 100%
        * Acceleration: **Enable 3D Acceleration & Enable 2D Video Acceleration**
    * Remote Display
        * Not Enabled
    * Video Capture
        * Not Enabled
* Storage
    * Controller: Floppy
        * Type: I82078 - Use Host I/O Cache
        * Floppy Drive: Floppy Device 0
    * Controller: IDE
        * Type: PIIX4 - Use Host I/O Cache
    * **Win 98se.vdi - 8 + GB Fixed**
        * Hard Disk: IDE Primary Master
    * **Win 98SE Install CD**
        * **Optical Drive: IDE Secondary Master**
    * **Win 98SE Utilities CD**
        * **Optical Drive: IDE Secondary Slave**
* Audio
    * Enable Audio
        * Host Audio Drive: Windows DirectSound or Linux/OSX equivalent
        * Audio Controller: SoundBlaster 16
        * Extended Features: Enable Audio Output
* Network
    * Adapter 1
        * Enable Network Adapter
        * Attached to: **Bridged Adapter**
        * Name: <system specific>
        * Adapter Type: **PCne-PCI II (Am79C970A)**
        * Promiscuous Mode: Deny
        * MAC Address: <randomly generated>
        * Cable Connected
    * Adapters 2-4
        * Not Enabled
* Ports
    * Serial Ports
        * Ports 1-4
            * Not Enabled
    * USB
        * Enable USB Controller
            * USB 2.0 (EHCI) Controller
* Shared Folders
    * Machine Folders
* User Interface

## Required command

After creating your virtual machine and applying all the settings but before starting it up exit VirtualBox and run the following terminal command in the same user account, it does not require admin privileges.

* Windows (replace "Win 98se" with the name of your virtual machine if you changed it)
    ```console
    "%ProgramFiles%\Oracle\VirtualBox\VBoxManage.exe" setextradata "Win 98se" VBoxInternal/USB/HidMouse/0/Config/CoordShift 0
    ```
* Linux & OSX (replace "Win 98se" with the name of your virtual machine if you changed it)
    ```console
    VBoxManage setextradata "Win 98se" VBoxInternal/USB/HidMouse/0/Config/CoordShift 0
    ```

You are now ready to start the installation

## Installation sequence

As Windows 98se can be more than a bit unstable you may wish to save your image at various points, just chose to "restart later" and shutdown instead of restarting then take a snapshot of the virtual hard drive before proceeding. I've given the time taken for some processes on my system (with the AntiVirus turned off) if you want to take a break while they run.

* Start Windows 98 Setup and configure the hard drive (enable large disk support)
* Start computer with CD-ROM support then enter the following commands
    * `D:`
    * `cd Win98`
    * `format C: /V:Win98se`
    * `setup /p j`
* Custom - add ALL Windows components except additional Accessibility, Multilanguage Support and Web TV for Windows
* **Setup will hang on the first reboot after entering the license code, just force a restart**
* _Setup might BSoD on the second reboot, press any key to continue_
* When Windows asks to install new drivers from Windows CD-ROM, make sure the search directory contains proper drive letter (for example it may show `E:\WIN98` while your system CD-ROM is in `D:\WIN98`)
* Switch to Windows Logon (Network Neighbourhood >> Properties >> Configuration) - _save point 1_
* Set Virtual Memory to 1GB (My Computer >> Properties >> Performance)
* **Skip the following steps if you install different language version than English:**
    * Auto-Patcher December 2007 Full (install only)
    * Auto-Patcher December 2008 Upgrade (install and run)
        * Enter the following key sequence - `M N U 5 1 3 7 B P S I I` - 35 minutes
* _You might have to do the odd restart but if the VM continually aborts then it's time to refer to the troubleshooting section_
* sdd-win-7.0.0.340-beta.exe (SciTech Display Doctor)
    * Install and reboot - _save point 2_
    * [Register](https://scitechdd.wordpress.com/) and reboot
        * Name: `CSCKnight`
        * Serial: `0B5E-12B4-A8A4-0B`
    * Choose Monitor Model - (Standard monitor types) - Super VGA 1600x1200
    * Click Display Driver then change Active Driver to SciTech Nucleus Driver and reboot
    * Change Display Properties - Settings to 256 Colors - 1024 by 768 pixels and reboot
    * **Windows will hang on startup, force a restart in normal mode**
    * _If you get a one dimensional screen resolution then force an ACPI shutdown_
    * Change Display Settings to True Color (32 bit) and either 1152x864 or 1280x1024 pixels
    * Run the Optimize Wizard - Keep Current Mode - Continue Without Centring
    * Run MSconfig and disable "Check for SDD updates" on the Startup tab and reboot - _save point 3_
* Network Setup Wizard (Accessories >> System Tools, cable must be connected, keep files x3)
* Internet Connection Wizard (Accessories >> Communications, connect via LAN)
* Lock down Internet zones (Internet & Restricted to High, others to Medium)
* KernelEx-4.5.2.exe (compatibility layer to run XP programs) - _save point 4_

* Run1X.bat (& Run2X.bat) where X is your CD drive with utilities - installs the following, reboots on completion - 10 minutes
    * uninstall MSXML 4.0 SP2
    * copy GDIplus.dll (5.1.3102.5581) to C:\Windows
    * flashplayer10\_3r183\_90\_winax.exe (flash player for I.E.)
    * msxml.msi (MSXML 4.0 SP3 - extracted from msxml4-KB2758694-enu.exe)
    * vcredist\_x86.exe (Visual C++ 2005 SP1 KB2538242)
    * dotnetfx.exe (.NET Framework 1.1 RtM)
    * NDP1.1sp1-KB867460-X86.exe (Service Pack 1)
    * NDP1.1sp1-KB971108-X86.exe (hotfix)
    * NDP1.1sp1-KB2833941-X86.exe (hotfix)
    * reboot and autostart Run2X.bat
    * extract OOo\_3.2.1\_Win\_x86\_install\_en-**.exe (Open Office)
    * 7z920.msi (7-Zip file archiver)
    * Firefox Setup 10.0.12esr.exe (KernelEx - XP SP2)
    * extract Dictionaries.7z for Firefox
    * install\_flash\_player\_32bit.exe (flash player for Firefox)
    * jre-6u22-windows-i586.exe (Java Runtime Environment)
    * install openofficeorg32.msi
    * SumatraPDF-2.2.1-install.exe
    * klmcodec345.exe (K-Lite Mega Codec)
    * quicktimealt156.exe (QuickTime Alternative)
    * Set Firefox as the default web browser
    * MenuReorg.bat (tidy quick launch, desktop & start menu)

* Windows Explorer settings (details view, don't hide file extensions)
* Open PlainOldFavorites\_1.3.1.xpi with Windows Explorer and link it to Firefox
* Restart Firefox three times to enable Plain Old Favourites
* DeFrag & ScanDisk

## File Associations

To register the file associations:

* 7-Zip - Open 7-Zip and it's under Tools >> Options >> System (clear the email options on the 7-Zip tab)
* K-Lite Mega Codec - Open Media Player Classic and it's under View >> Options >> Player >> Formats
* SumatraPDF - Open a .pdf file in Windows Explorer and link it to C:\SumatraPDF\SumatraPDF.exe
* Open Office - The easiest reset method is to uninstall Open Office, reboot and then reinstall Open Office with the command
    ```console
    C:\Windows\System\msiexec.exe /I C:\OpenOffice3\openofficeorg32.msi REGISTER_ALL_MSO_TYPES=1 /Qb

    ```
    or for individual file types you can open the file in Windows Explorer (shift-right-click if already linked) and link it to `C:\Program Files\OpenOffice.org 3\Program\[word=swriter|excel=scalc|powerpoint=simpress]` and then edit the file type (Tools >> Folder Options) adding `-o "%1"` to the end of the open command - Open Office file association commands (these do not work on network shares, local files only)
    ```console
    "C:\Program Files\OpenOffice.org 3\program\swriter.exe" -o "%1"
    "C:\Program Files\OpenOffice.org 3\program\scalc.exe" -o "%1"
    "C:\Program Files\OpenOffice.org 3\program\simpress.exe" -o "%1"
    "C:\Program Files\OpenOffice.org 3\program\sbase.exe" -o "%1"
    "C:\Program Files\OpenOffice.org 3\program\smath.exe" -o "%1"
    "C:\Program Files\OpenOffice.org 3\program\sdraw.exe" -o "%1"
    ```

## Troubleshooting

* Security software has been known to interfere with VirtualBox (for instance it is impossible to install Windows 2000 in VirtualBox if Avast Free anti-virus is installed on the host) so first try disabling such programs and if that doesn't help you might even have to uninstall them

* Sometimes problems with installing Win98se in VirtualBox 5.x.x are due to [incompatibilites with newer CPUs](http://forums.virtualbox.org/viewtopic.php?f=2&t=84858#p403359), so if you still have issues try mimicking a 2012 vintage processor by executing this command (with the VM shut down):
    * Windows (replace "Win 98se" with the name of your virtual machine if you changed it)
        ```console
        "%ProgramFiles%\Oracle\VirtualBox\VBoxManage.exe" modifyvm "Win 98se" --cpu-profile "Intel Core i5-3570"
        ```

    * Linux & OSX (replace "Win 98se" with the name of your virtual machine if you changed it)
        ```console
        VBoxManage modifyvm "Win 98se" --cpu-profile "Intel Core i5-3570"
        ```
    If the Core i5 doesn't work here is a [list of supported CPUs](http://www.virtualbox.org/svn/vbox/trunk/src/VBox/VMM/VMMR3/cpus/) you can try instead, you can remove the modification by running the command again with the arguement `--cpu-profile ""`

* Finally, you could try installing Win 98se using VirtualBox 4.3.40 plus extension pack and then upgrading to the 5.x.x series

## Usability tips

* Some games can have trouble with the mouse, if so try turning mouse integration off
* Some games require that the install CD be in the drive, if so connect to a physical (or advanced virtual - Windows, Linux) drive and enable passthrough
* You don't have the ability to switch to a true full screen mode so you will have to adjust the host's resolution to close to that of the guest
* The supported modes are 640x480 800x600 1024x768 1152x864 1280x1024 1600x1200
* Try to match the vertical resolution as closely as possible while maintaining your monitor's aspect ratio, full screen mode will save you about 120 pixels

## Useful websites

* [KernelEx wiki (application compatibility database)](http://kernelex.sourceforge.net/wiki/Main_Page)
* [Auto-Patcher MSFN topic](http://www.msfn.org/board/topic/80800-auto-patcher-for-windows-98se-english/)
* [Main install guide used](https://forums.virtualbox.org/viewtopic.php?f=28&t=9918)

## Other driver options AKA wet bear

The `setup /p j` command is used to force ACPI support (item 1) on installation, the other method commonly used is to install [Rain 2.0](http://www.benchtest.com/rain.html), that should not be necessary in this case but if you are getting 100% CPU load from the VM even when it is sitting idle you could try it

SciTech Display Doctor was released for free (item 2) when the company went out of business, but if you don't feel comfortable using it or want to try something different you can install the Bear [Universal VESA/VBE Video Display Driver](http://bearwindows.zcm.com.au/vbe9x.htm) using the one in the VBE9X\UNI folder [(install tips)](https://forums.virtualbox.org/viewtopic.php?f=28&t=9918#p163805)

You can use the [Intel PRO/1000 MT Desktop (82540EM)](https://downloadcenter.intel.com/Detail_Desc.aspx?agr=Y&DwnldID=4235) network adapter as drivers are available, however it won't increase your network speed as VirtualBox will push data through the virtual interface at the highest speed your physical network will support, also don't change to it until after you've run the Network Setup Wizard

To use AC97 audio download the [Realtek AC97 Windows 95 driver](http://www.realtek.com.tw/downloads/downloadsView.aspx?Langid=1&PNid=14&PFid=23&Level=4&Conn=3&DownTypeID=3&GetDown=false) (not the 98se one) then extract it and use the device manager to install it

## Other virtualization options

There are two other free virtualization options available, however VirtualBox with SciTech has better graphics performance for gaming.

In both cases install the VM drivers after running the Auto-Patcher (at the same point you'd install SciTech on VirtualBox).

If you are using Microsoft Windows 7 (32 or 64 bit) or earlier you can use MS Virtual PC 2007 which has full VM driver support for Windows 98se.

If you also want access to Windows XP mode (only available in Windows 7 Professional & Ultimate) you will have to use [some tricks](http://nookkin.com/articles/how-tos-and-guides/run-virtual-pc-2007-and-windows-virtual-pc-on-the-same-machine.ndoc) to get both working.

MS Virtual PC 2007 emulates a 1995 vintage 4MB 2D S3 Trio32/64 PCI (732/764) graphics adapter.

Install MS Virtual PC 2007 in the following order:
* [Virtual PC 2007 SP1](http://www.microsoft.com/en-us/download/details.aspx?id=24439)
* [Hotfix rollup](https://web.archive.org/web/20151018045532/https://support.microsoft.com/en-us/kb/958162)
* [Security update](http://www.microsoft.com/en-us/download/details.aspx?id=25161) (also in the small patches archive and torent)

If you are using Microsoft Windows 7 (64 bit only) or later or a current 64 bit version of Linux you can use [VMware Player 12](https://my.vmware.com/en/web/vmware/free#desktop_end_user_computing/vmware_workstation_player/12_0|PLAYER-1210|product_downloads).

VMware Player uses a custom SVGA implementation but it lacks acceleration, also the VM driver installation is buggy.

Install VMware drivers in the following order
    * Install VMware Tools (Player >> Manage), the mouse driver will fail
    * Manually install the PS/2 mouse driver via the device manager, inf file at "C:\Program Files\VMware\VMware Tools\Drivers\mouse"
    * Install the [audio driver](https://mega.nz/#!5IUFjbqI!DdbgBohypIYunbji3lS00y21DVWYRVcoQP15TNBxyUA) (also in the small patches archive and torent)
