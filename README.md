# Acer Aspire VX15 Hackintosh

__Why is there this repo?__

Some other models of VX15 don't work with patches from master branch. This repo will fix those problems  
Changes compared to master branch:  
- Trackpad: SSDT-TPAD.aml => SSDT-XOSI.aml 
- USB Ports: USBMap.kext => USBInjectAll.kext 
- Brightness adjustment: SSDT-PNLF.aml  

## Guide

__BIOS Settings (Version 1.08)__

- Set Supervisor Password
- Disable Password on Boot
- Disable Secure Boot
- Set touchpad: Advance

__OpenCore config__

- Follow these instructions to configure your OpenCore: https://dortania.github.io/OpenCore-Install-Guide/  
- ACPI Hotpatch: https://github.com/daliansky/OC-little  
- Audio layout ID 29

__USB Mapping__

- USB Mapping: https://github.com/corpnewt/USBMap

__Fix HDMI audio__

- Patch framebuffer: https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md
- Add kext: `FakePCIID.kext` (kext at this repo)  

__Fix iServices__

- Fix iServices: https://dortania.github.io/OpenCore-Post-Install/universal/iservices.html

__Fix brightness key__

Use patch `SSDT-BKEY.aml`  
And at `Root > ACPI > Patch`:  
- XQ11 Rename

|Key|Type|Value|
|---|---|---|
|Comment|String|Change _Q11 to XQ11 (Brightness Down)|
|Count|Number|0|
|Enabled|Boolean|True|
|Find|Data|5F513131|
|Limit|Number|0|
|Replace|Data|58513131|

- XQ12 Rename

|Key|Type|Value|
|---|---|---|
|Comment|String|Change _Q12 to XQ12 (Brightness Up)|
|Count|Number|0|
|Enabled|Boolean|True|
|Find|Data|5F513132|
|Limit|Number|0|
|Replace|Data|58513132|

__Fix Dual Boot__

- Fix timezone: https://www.tonymacx86.com/threads/fix-incorrect-time-in-windows-osx-dual-boot.133719/  

## Credits

Thanks to Acidanthera, RehabMan, dortania, alexandred, daliansky, OpenIntelWireless

*__Enjoy__*
