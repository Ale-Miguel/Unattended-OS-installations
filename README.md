# Unattended-OS-installations

## Create custom iso for unattended Windows install

### Requirements
**It is recommended to do this process inside a VM**
* Install [Windows ADK](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install#other-adk-downloads)
* Have an official Windows ISO of your choice
* All the commands must be run as Administrator
* "[Official guide](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/update-windows-settings-and-scripts-create-your-own-answer-file-sxs?view=windows-11)"
### Steps
1. Create an *autounattend.xml* file with the help of [this](https://schneegans.de/windows/unattend-generator/) online tool for Windows 10/11 Iso's. 
2. Copy all the files inside the ISO image to a folder. This can be done in 2 ways:
	* Use 7z to extract all the files
	* Mount the ISO image and copy/paste its contents
3. Locate the install.wim file at *\sources\install.wim* inside the folder where all the files of the ISO image were copied.
4. Get the SourceIndex of the desired Windows version 
	```cmd
    dism /Get-WimInfo /WimFile:C:\Users\User\Desktop\Windows10_22H2-16-10-2023\sources\<install.wim or install.esd>
	``` 
	(modify the previous path if the image contains an *install.wim* instead of *install.esd*)
	Example:
	```cmd
	C:\>dism /Get-WimInfo /WimFile:C:\Users\User\Desktop\Windows10_22H2-16-10-2023\sources\install.esd

    Deployment Image Servicing and Management tool
    Version: 10.0.25398.1
    
    Details for image : C:\Users\User\Desktop\Windows10_22H2-16-10-2023\sources\install.esd
    
    Index : 1
    Name : Windows 10 Home
    Description : Windows 10 Home
    Size : 15,139,674,619 bytes
    
    Index : 2
    Name : Windows 10 Home N
    Description : Windows 10 Home N
    Size : 14,368,233,902 bytes
    
    Index : 3
    Name : Windows 10 Home Single Language
    Description : Windows 10 Home Single Language
    Size : 15,142,346,396 bytes
    
    Index : 4
    Name : Windows 10 Education
    Description : Windows 10 Education
    Size : 15,475,605,732 bytes
    
    Index : 5
    Name : Windows 10 Education N
    Description : Windows 10 Education N
    Size : 14,684,197,328 bytes
    
    Index : 6
    Name : Windows 10 Pro
    Description : Windows 10 Pro
    Size : 15,472,691,215 bytes
	```
5. Locate the install.wim file at *\sources\install.wim* inside the folder where all the files of the ISO image were copied. 
	* In case that the file is not found and instead there is an install.esd, run the following commands in the Command Prompt with Admin privileges: 
	```cmd
	dism /export-image /SourceImageFile:C:\Users\User\Desktop\Windows10_22H2-16-10-2023\sources\install.esd /SourceIndex:6 /DestinationImageFile:C:\Users\User\Desktop\install.wim /Compress:max /CheckIntegrity
	``` 
	(Change the SourceIndex value to the index of the desired Windows version)

6. **Maybe these steps are optional:** Open the Windows System Image Manager (WISM) and inside it do the following:
	1. Open the autounattend.xml file (File>Open Answer File)
	2. Open the install.wim file (File>Select Windows Image)
	3. The WISM will modify the autoattend.xml to prepare it for the selected Windows Image
	4. Check if there is no errors by going to Tools>Validate Answer File
	5. Save the changes to the autounattend.xml file
7. Copy/move the autounattend.xml file to the root of the folder that contains the ISO files (it is important that the file must be named `autounattend.xml`).
8. Open the *Deployment and Imaging Tools Environment* (DITE) with Admin privileges.
9. Create the ISO in the console of the DITE with the command:
	```cmd
    oscdimg.exe -m -o -u2 -udfver102 -bootdata:2#p0,e,b<Full path to folder with the ISO files>\boot\etfsboot.com#pEF,e,b<Full path to folder with the ISO files>\efi\microsoft\boot\efisys.bin <Full path to folder with the ISO files> <Path to the destination of the new ISO image>
	```
	* Note that the paths does not contain spaces between the comma-separated values. For Example:
	```cmd
	oscdimg.exe -m -o -u2 -udfver102 -bootdata:2#p0,e,bC:\Users\User\Desktop\Windows10_22H2-16-10-2023\boot\etfsboot.com#pEF,e,bC:\Users\User\Desktop\Windows10_22H2-16-10-2023\efi\microsoft\boot\efisys.bin C:\Users\User\Desktop\Windows10_22H2-16-10-2023 C:\Users\User\Desktop\Unattended-Windows10_22H2.iso
	```
