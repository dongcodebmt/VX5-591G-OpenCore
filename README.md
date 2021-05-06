# Acer Aspire VX15 Hackintosh

__Why is there this repo?__

Some other models of VX15 don't work with patches from master branch. This repo will fix those problems  
Changes compared to master branch:  
- Trackpad: SSDT-TPAD.aml => SSDT-XOSI.aml 
- USB Ports: USBMap.kext => USBInjectAll.kext 
- Brightness adjustment: SSDT-PNLF.aml => SSDT-PNLF.aml 
- config.plist: Add patch XOSI 

`Root > ACPI > Patch`: 
|Key|Type|Value|
|---|---|---|
|Comment|String|Change _OSI to XOSI|
|Count|Number|0|
|Enabled|Boolean|True|
|Find|Data|5F4F5349|
|Limit|Number|0|
|Replace|Data|584F5349|

Watch video to create your config: https://youtu.be/bIsj1rCC0vk
