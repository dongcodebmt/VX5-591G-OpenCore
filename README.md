# Acer Aspire VX15 Hackintosh

![Specs](/image.png)

__My Specs__

| Specs | Details |
|------------|-------------------------------|
| Model | Acer Aspire VX15 (VX5-591G) |
| OS | macOS Monterey && Windows 11  |
| CPU | Intel(R) Core(TM) i5 7300HQ |
| RAM | 16 GB DDR4 2400MHz |
| iGPU | Intel HD Graphics 630 |
| dGPU | NVIDIA GeForce GTX 1050M |
| Touchpad | ELAN 0501 |
| Wireless | Intel(R) Wireless-AC 7265 |
| Audio | ALC255 |

__Tested and working__

- [x] Intel HD Graphics 630
- [x] USB
- [x] Webcam
- [x] LAN
- [x] Screen brightness
- [x] Battery status
- [x] Sleep/Wake
- [x] TouchPad with gestures
- [x] WiFi
- [x] Bluetooth
- [x] HDMI + HDMI Audio
- [x] Airdrop + Handoff
- [x] Audio + Headphone + Internal Mic
- [x] Keyboard with backlight (Some function keys not work)
- [x] iServices

__Not working__

- [ ] NVIDIA GeForce GTX 1050M
- [ ] SD Card reader
- [ ] DRM
- [ ] and a few other minor bugs

## Guide

__BIOS Settings (Version 1.08)__

- Set Supervisor Password
- Disable Secure Boot

__OpenCore config__

- Follow these instructions to configure your OpenCore: https://dortania.github.io/OpenCore-Install-Guide/  
- ACPI Hotpatch: https://github.com/daliansky/OC-little  
- Audio layout ID 29

__Install MacOS__

- Create bootable USB: https://dortania.github.io/OpenCore-Install-Guide/installer-guide/  
- If you have problems installing Catalina or later. Install Mojave then you can update to newer version.

__USB Mapping__

- USB Mapping: https://github.com/corpnewt/USBMap

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

__UEFI Secure Boot__

I will not give detailed instructions here. See details at [here](https://github.com/dortania/OpenCore-Post-Install/blob/secureboot/universal/security/uefisecureboot.md).  
You will need linux. However for simplicity and speed I will use [CloudShell](https://console.cloud.google.com/getting-started?cloudshell=true).  
```console
user@cloudshell:~$ sudo apt-get install -y gnu-efi help2man sbsigntool uuid-runtime libfile-slurp-unicode-perl
user@cloudshell:~$ mkdir sign && cd sign
user@cloudshell:~$ git clone https://github.com/vathpela/efitools.git && cd efitools && make
user@cloudshell:~$ cp cert-to-efi-sig-list sign-efi-sig-list KeyTool.efi .. && cd ..
user@cloudshell:~$ openssl req -new -x509 -newkey rsa:4096 -sha256 -days 365 -subj "/C=VN/O=DongDev/CN=Platform Key" -keyout PK.key -out PK.pem
Generating a RSA private key
........................................................................................++++
....................................................................................................................................................................++++
writing new private key to 'PK.key'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
user@cloudshell:~$ openssl req -new -x509 -newkey rsa:4096 -sha256 -days 365 -subj "/C=VN/O=DongDev/CN=Key Exchange Key" -keyout KEK.key -out KEK.pem
Generating a RSA private key
....................................................................................................................................................................................................................................................................................................................................................................++++
...........................................................................................................................................................................................................++++
writing new private key to 'KEK.key'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
user@cloudshell:~$ openssl req -new -x509 -newkey rsa:4096 -sha256 -days 365 -subj "/C=VN/O=DongDev/CN=Image Signing Key" -keyout ISK.key -out ISK.pem
Generating a RSA private key
........................................................++++
...........++++
writing new private key to 'ISK.key'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
user@cloudshell:~$ ./cert-to-efi-sig-list -g "$(uuidgen)" PK.pem PK.esl
user@cloudshell:~$ ./cert-to-efi-sig-list -g "$(uuidgen)" KEK.pem KEK.esl
user@cloudshell:~$ ./cert-to-efi-sig-list -g "$(uuidgen)" ISK.pem ISK.esl
user@cloudshell:~$ wget https://www.dropbox.com/s/un9q4ryu9il0ynd/MicWinProPCA2011_2011-10-19.crt?dl=1 -O MicWinProPCA2011_2011-10-19.crt
user@cloudshell:~$ wget https://www.dropbox.com/s/qwbi3gk7h9qc716/MicCorUEFCA2011_2011-06-27.crt?dl=1 -O MicCorUEFCA2011_2011-06-27.crt
user@cloudshell:~$ openssl x509 -in MicWinProPCA2011_2011-10-19.crt -inform DER -out MsWin.pem -outform PEM
user@cloudshell:~$ openssl x509 -in MicCorUEFCA2011_2011-06-27.crt -inform DER -out UEFI.pem -outform PEM
user@cloudshell:~$ ./cert-to-efi-sig-list -g "$(uuidgen)" MsWin.pem MsWin.esl
user@cloudshell:~$ ./cert-to-efi-sig-list -g "$(uuidgen)" UEFI.pem UEFI.esl
user@cloudshell:~$ cat ISK.esl MsWin.esl UEFI.esl > db.esl
user@cloudshell:~$ ./sign-efi-sig-list -k PK.key -c PK.pem PK PK.esl PK.auth
Timestamp is 2021-7-8 13:02:52
Authentication Payload size 1445
Enter PEM pass phrase:
Signature of size 2058
Signature at: 40
user@cloudshell:~$ ./sign-efi-sig-list -k PK.key -c PK.pem KEK KEK.esl KEK.auth
Timestamp is 2021-7-8 13:03:30
Authentication Payload size 1455
Enter PEM pass phrase:
Signature of size 2058
Signature at: 40
user@cloudshell:~$ ./sign-efi-sig-list -k KEK.key -c KEK.pem db db.esl db.auth
Timestamp is 2021-7-8 13:03:59
Authentication Payload size 4598
Enter PEM pass phrase:
Signature of size 2070
Signature at: 40
user@cloudshell:~$ mkdir OC
user@cloudshell:~$ git clone https://github.com/dongcodebmt/VX5-591G-OpenCore.git && cd VX5-591G-OpenCore
user@cloudshell:~$ cp Drivers/* OpenCore.efi ../OC && cd ..
user@cloudshell:~$ sbsign --key ISK.key --cert ISK.pem HfsPlus.efi
warning: data remaining[33792 vs 33796]: gaps between PE/COFF sections?
warning: data remaining[33792 vs 33800]: gaps between PE/COFF sections?
Enter PEM pass phrase:
Signing Unsigned original image
user@cloudshell:~$ sbsign --key ISK.key --cert ISK.pem OpenCanopy.efi
warning: gap in section table:
    .data   : 0x00015000 - 0x00016000,
    .reloc  : 0x00017000 - 0x00018000,
gaps in the section table may result in different checksums
warning: data remaining[94208 vs 98304]: gaps between PE/COFF sections?
Enter PEM pass phrase:
Signing Unsigned original image
user@cloudshell:~$ sbsign --key ISK.key --cert ISK.pem OpenCore.efi
warning: gap in section table:
    .data   : 0x0006b000 - 0x0007b000,
    .reloc  : 0x00082000 - 0x00084000,
gaps in the section table may result in different checksums
warning: data remaining[512000 vs 540672]: gaps between PE/COFF sections?
Enter PEM pass phrase:
Signing Unsigned original image
user@cloudshell:~$ sbsign --key ISK.key --cert ISK.pem OpenRuntime.efi
warning: data remaining[20480 vs 20484]: gaps between PE/COFF sections?
warning: data remaining[20480 vs 20488]: gaps between PE/COFF sections?
Enter PEM pass phrase:
Signing Unsigned original image
user@cloudshell:~$ rm -rf *.efi && cd .. && mv OC .. && cd .. &&  zip -r OC.zip OC
user@cloudshell:~$ cloudshell download OC.zip
```

You now have the necessary files. Let's continue [here](https://github.com/dortania/OpenCore-Post-Install/blob/secureboot/universal/security/uefisecureboot.md#insyde-h2o).   
*__Note:__*
- Keep `ISK.key` and `ISK.pem` so that each OpenCore upgrade you will only need to sign the necessary files and replace them in the `EFI/OC` directory. You won't need to repeat the key generation and key adding steps.   
- To booting `KeyTool.efi`. Let's rename `KeyTool.efi` to `BOOTx64.efi` and put it in `USB/EFI/BOOT`    

## Credits

Thanks to Acidanthera, RehabMan, dortania, alexandred, daliansky, OpenIntelWireless

*__Enjoy__*
