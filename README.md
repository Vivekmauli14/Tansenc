[![License](https://img.shields.io/badge/License-GPLv3-blue.svg "License")](https://www.gnu.org/licenses/gpl-3.0 "License")
[![Driver](https://img.shields.io/badge/Driver-passing-green.svg "Driver")](https://github.com/hkx3upper "Driver")
[![Test](https://img.shields.io/badge/Test-passing-green.svg "Test")](https://github.com/hkx3upper "Test")
[![PR](https://img.shields.io/badge/PR-welcome-blue.svg "PR")](https://github.com/hkx3upper/FOKS-TROT/pulls "PR")
[![Issue](https://img.shields.io/badge/Issue-welcome-blue.svg "Issue")](https://github.com/hkx3upper/FOKS-TROT/issues "Issue")
<a href="https://github.com/hkx3upper/FOKS-TROT"><img align="right" width="231" height="117" src="https://user-images.githubusercontent.com/41336794/172169698-65afd346-38c4-4fd6-861b-20940a6ee493.jpg" alt="FOKS-TROT"></a></br>
# FOKS-TROT
## Double buffer transparent encryption and decryption driver based on Minifilter framework
## It's a minifilter used for transparent-encrypting. <a href="https://github.com/hkx3upper/FOKS-TROT/wiki/Foxtrot">English</a>

## Foreword:
**Foxtrot** is an experimental project, and the author's understanding of the file system is inevitably biased, so it may be misleading. Readers are advised to study dialectically and follow the relevant open source agreements. And the versions before May 12 of this project have been used as graduation projects. If there are any similarities, they are purely similar `((/- -)/）`
**After five months of maintenance, **Foxtrot** has ushered in the first stable version 1.0.0.2265. The project can now run stably on various versions of Windows 10 x64 (it should be possible). It is recommended that you clone it again, but this version does not support files that have been encrypted by the driver before (see Article 11). **
`P.S. The project has been packaged into an exe installation package, so you don't need to compile the project manually`
## Description:
**Foxtrot** is a double-buffered transparent encryption and decryption filter driver using the minifilter framework. It automatically encrypts files when the process has a tendency to write files with specific file extensions (such as txt, docx). When the authorized process wants to read the ciphertext file, it will automatically decrypt it. The unauthorized process will not decrypt it, display the ciphertext, and will not allow the ciphertext to be modified.
The desktop can send privileged encryption and privileged decryption commands to encrypt or decrypt files individually; or configure process permissions, confidential folders, and file types to be encrypted.
```
1. This project uses double buffering, and the authorized process and the unauthorized process use plaintext buffering and ciphertext buffering respectively;
2. Use StreamContext to store the file information when the driver is running, and use a 4KB file identifier at the end of the file to store the information required for decryption;
3. Use AES-128 ECB mode, and expand the file size to 16 bytes when SetInfo->EOF and WriteCachedIo are used within 16 bytes.
If it is larger than 16 bytes, use the ciphertext stealing method to avoid the problem that the plaintext must be aligned in blocks;
4. Write and Read use SwapBuffers for transparent encryption and decryption;
5. Privileged encryption and privileged decryption use the reentry method to enable the driver to encrypt and decrypt files;
6. Do processing during FileRenameInformationEx and FileRenameInformation renaming operations,
which can automatically encrypt and decrypt docx, doc, pptx, ppt, xlsx, xls and other files that are read and written using the tmp file renaming method;
7. Register process-related callbacks and use linked lists to uniformly manage authorized and unauthorized processes;
Register process and thread object callbacks to protect process EPROCESS and ETHREAD objects; perform integrity check on the code segments of authorized processes.
8. Set up a confidential folder, and the file will be transparently encrypted only when it is in this folder.
And you can configure the confidential folder and the file extensions to be controlled from the desktop PocUser @wangzhankun
9. PostOperation uniformly uses the function FltDoCompletionProcessingWhenSafed (except PostRead),
Use Dpc+WorkItem callback (encapsulated as PocDoCompletionProcessingWhenSafe) during InstanceSetup,
Avoid blue screens such as IRQL_NOT_LESS_OR_EQUAL at DISPATCH_LEVEL;
10. Use a separate thread during PostClose, wait for all authorized processes operating the file to end,
Then re-enter encryption or write to the file identifier tail, solving the deadlock problem of docx files;
11. Change ULONG to LONGLONG, in principle, it can support files larger than 4GB (currently privileged encryption and privileged decryption do not support files larger than 4GB,
And in the case of limited memory, privileged decryption may fail due to lack of non-paged memory, I don’t want to write it, here you can put it in the loop to read and write large files)
12. The user interface is written in WPF, which can configure the authorization process, the file types to be controlled, the confidential folder, and the privileged encryption and decryption files;
Use InstallShield to package the installation package;
13. Increase process permissions: backup privileged processes, such as VMTools and explorer.exe, can copy the complete ciphertext file from the virtual machine, or
copy the ciphertext file from the confidential folder to the outside, and paste the file to the confidential folder (unencrypted files are automatically encrypted, and encrypted files are automatically decrypted once after repeated encryption)
14. After Write encryption, ObDereferenceObject a previously created FileObject, trigger Close to create a thread to prepare for writing to the end or re-entering encryption;
15. Allow the driver to encrypt or decrypt files outside the confidential folder with privileges; clear the plaintext buffer when the driver is uninstalled to prevent plaintext leakage;
16. When WRITE_THROUGH, Ntfs does not update Fcb->FileSize before PagingIo, it uses TopLevelIrpContext + 184 to truncate
The data of PagingIo updates the FileSize when CachedIo returns, so SC->WriteThroughFileSize is used here to temporarily save the FileSize.
```
## Build & Installation:
1. It is recommended to run in Windows 10 x64, NTFS environment
```
Tested system and software versions:
Windows 10 x64 1809 (17763.2928) LTSC Enterprise Edition [WPS 11.1.0.11365]
Windows 10 x64 1903 (18362.30) Education Edition [Microsoft Office Professional Plus 2021 x64]
[WPS 11.1.0.11744] [360 Security Guard 15.0.0.1061]
Windows 10 x64 1909 (18363.592) Education Edition [WPS 11.1.0.11744]
Windows Server 2019 DataCenter 1809 (17763.379)
```
2. Open the test mode of the system, run cmd as an administrator, enter `bcdedit /set testsigning on` and restart the computer
3. Output the driver log (optional)
```
Find the registry key: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Debug Print Filter
If there is no Debug Print Filter, create a new one. Create a new dword value "default" under this key, which is 0xF in hexadecimal, and then restart the computer
Run DebugView as an administrator, set `Capture->Capture Kernel` to display the driver log
```
4. Download the installation package (optional)

[![Download](https://img.shields.io/badge/Download-5.09MB-green.svg "Download")](https://github.com/hkx3upper/FOKS-TROT/releases/ "Download")
`If you download the installation package, you don't need to compile the project manually, you can jump directly to step 10`
5. Install and import the CNG library
```
https://www.microsoft.com/en-us/download/details.aspx?id=30688
You need to download the Cryptographic Provider Development Kit on the Microsoft official website,
Project->Properties of the VC++ directory include directory, library directory set the corresponding location
Linker General->Additional Library Directory C:\Windows Kits\10\Cryptographic Provider Development Kit\Lib\x64
Input->Additional Dependencies ksecdd.lib
```
6. Set the target file extension, confidential folder, and authorized process and backup process in `Config.c`
7. Use Visual Studio 2019 to compile Debug x64 Poc driver, UserDll (optional) and UserPanel (optional)
8. Add privileged encryption and privileged decryption functions to the right-click menu (optional)
```
This function can directly select a file with the right mouse button, and then click privileged encryption or privileged decryption, without the need for cmd command line
Create a new registry key: HKEY_CLASSES_ROOT\*\shell\Encrypt, change the "default" data of this key to privileged encryption
Create a new registry key: HKEY_CLASSES_ROOT\*\shell\Encrypt\command,
Change the data of the "default" value of this key to "path\PocUserPanel.exe" 8 "%1"

Create a new registry key: HKEY_CLASSES_ROOT\*\shell\Decrypt, change the "default" data of this key to privileged decryption
Create a new registry key: HKEY_CLASSES_ROOT\*\shell\Decrypt\command,
Change the data of the "default" value of this key to "path\PocUserPanel.exe" 4 "%1"
```
9. If you use a loader such as OsrLoader to load the driver, please select Minifilter for Type, or you can use cmd to load it, as follows
```
:: Load
rundll32.exe setupapi.dll,InstallHinfSection DefaultInstall 132 path\Poc.inf
net start Poc
pause
:: Uninstall
## Credits & References：
hkx3upper:(<a href="https://github.com/hkx3upper">@hkx3upper</a>)
wangzhankun:(<a href="https://github.com/wangzhankun">@wangzhankun</a>)
MaterialDesignInXAML:(<a href="https://github.com/MaterialDesignInXAML">@MaterialDesignInXAML</a>)
CSharpDesignPro:(<a href="https://github.com/CSharpDesignPro/WPF---MVVM-Based-Modern-Dashboard">@CSharpDesignPro</a>)
Windows-driver-samples\filesys\fastfat:(<a href="https://github.com/microsoft/Windows-driver-samples/tree/main/filesys">Fastfat</a>)
Windows-driver-samples\filesys\minifilter:(<a href="https://github.com/microsoft/Windows-driver-samples/tree/main/filesys">Minifilter</a>)
Windows-Research-Kernel:(<a href="https://github.com/HighSchoolSoftwareClub/Windows-Research-Kernel-WRK-">WRK</a>)
WinNT4:(<a href="https://github.com/ZoloZiak/WinNT4">WinNT4</a>)
Design and implementation of file encryption and decryption system based on Minifilter microframework He Ming 2014.06
Research and implementation of transparent encryption system based on double buffer filter driver Liu Han 2010.04
Research and implementation of bidirectional transparent file encryption system based on IBE and FUSE He Xiang 2017.04
Windows NT File System Internals - A Developer's Guide
Windows kernel security and driver development Tan Wen
Windows kernel scenario analysis Mao Decao
Windows kernel principle and implementation Pan Aimin
## License:
**Foxtrot**, and all its submodules and repos, unless a license is otherwise specified, are licensed under **GPLv3** LICENSE.
Dependencies are licensed by their own.
