# Acer Aspire VX15 Hackintosh

_I no longer use hackintosh so this repo will not be maintained!_    
_Just update OpenCore and kexts and you'll be fine 😉_

![Specs](/image.png)

__My Specs__

| Specs | Details |
|------------|-------------------------------|
| Model | Acer Aspire VX15 (VX5-591G) |
| Operating System | macOS Monterey && Windows 11  |
| Processor | Intel Core i5 7300HQ |
| Memory | Micron & G.Skill 2x8GB DDR4 2400MHz |
| M.2 Hard Drive | Kingston A1000 240GB |
| 2.5" Hard Drive | TOSHIBA MQ01ABD100 1TB |
| Integrated Graphics | Intel HD Graphics 630 |
| Dedicated Graphics | NVIDIA GeForce GTX 1050M |
| Sound Card | Realtek ALC255 |
| Wireless Card | Intel Wireless-AC 7265 |
| Trackpad | ELAN 0501 |

__Tested and working__

- [x] Intel HD Graphics 630
- [x] USB 3.0 + USB 2.0 + USB Type C
- [x] Webcam
- [x] LAN + Wifi + Bluetooth
- [x] M.2 and 2.5" SATA hard drive
- [x] Screen brightness
- [x] Battery status
- [x] Sleep/Wake
- [x] TouchPad with gestures
- [x] HDMI + HDMI Audio
- [x] Airdrop + Handoff (with broadcom card)
- [x] Audio (Speaker + Internal Mic + Headphone + External Mic)
- [x] Keyboard with backlight (Some keys not work)
- [x] iServices
- [x] FileVault

__Not working__

- [ ] NVIDIA GeForce GTX 1050M
- [ ] SD Card reader
- [ ] DRM
- [ ] and a few other minor bugs

## Guide

__BIOS Settings (Version 1.08)__

- Set Supervisor Password
- Disable Secure Boot (Can be enabled if OpenCore signed)

__OpenCore config__

- Details and configuration of OpenCore:    
https://dortania.github.io/OpenCore-Install-Guide/  
https://github.com/acidanthera/OpenCorePkg  
- ACPI Hotpatch: https://github.com/daliansky/OC-little  
- USB Mapping: https://github.com/corpnewt/USBMap

__Install MacOS__

- Create bootable USB: https://dortania.github.io/OpenCore-Install-Guide/installer-guide/  
- If you have problems installing Catalina or later. Install Mojave then you can update to newer version.

__Fix Audio__

- Layout ID 29/31/90
- Install Combo Jack (with LayoutID 99) to select input like Windows and fix audio problems. 
- Combo Jack: https://github.com/lvs1974/ComboJack  

__Fix iServices__

- Fix iServices: https://dortania.github.io/OpenCore-Post-Install/universal/iservices.html

__Fix Dual Boot__

- Fix timezone: https://www.tonymacx86.com/threads/fix-incorrect-time-in-windows-osx-dual-boot.133719/  

__UEFI Secure Boot__

I will not give detailed instructions here. See details at [here](https://github.com/dortania/OpenCore-Post-Install/blob/secureboot/universal/security/uefisecureboot.md).  
You will need linux. However for simplicity I will use [CloudShell](https://console.cloud.google.com/getting-started?cloudshell=true).  
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
