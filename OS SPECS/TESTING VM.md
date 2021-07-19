## Image
ubuntu-20.04.2.0-desktop-amd64.iso
## Virtual enviornment software
Virtual box
## Setting up in Virtualbox
* Click on the blue star near the top middle of the screen that says "New" underneath it

|Setting | Value|
--- |---
Ram | 8196
Hard disk | "Create virtual hard disk now"
Hard disk file type | VDI
Storage on physical hard disk | Dynamically allocated
File location | C:\Users\Oldwe\VirtualBox VMs\4_Documentation\4_Documentation.vdi
File size | 50 GB
## Settings -> Things to change before first boot
* System -> Motherboard
	* Boot order
		* Uncheck floppy
		* Check hard disk and optical
		* Boot order should be hard disk on top then optical right below it
* System -> Processor
	* Change number of processors to 4
* Display -> Screen
	* Change video memory to 128 MB
* Storage
	* Click on where it says "Empty" underneath where it says "Controller: IDE"
	* To the right of where you just clicked ther should be a section labeled Attributes
	* Click on the small blue disk in the attributes section
	* Select the "Choose a disk file" option
	* Browse to the .iso location in this case 
	```windows
	E:\Summer 2021 Research\ubuntu-20.04.2.0-desktop-amd64.iso
	```
	* Click open
	* Where it said "Empty" before it should now read the name of your .iso
* USB
	* Make sure the "Enable USB Controller" box is *Unchecked*
* Click "OK" in bottom right to close window

## First boot
* Select start-up disk
	* Click the drop down box where it says something like "Host Drive 'D:'"
	* Select the .iso file that you are installing
	* Click Start
* Install
	* Eventually a box will pop up letting you select a language on the right
		* The default should be English, but if it isn't scroll down the box until you see English and select it
	* Then click on the box that says "Install Ubuntu" on the right side of the screen
	* The next screen is about which keyboard you are using
		* Again, English(US) should be defaulted, but if it isn't scroll down in the box until you find it and select it
	* Click Continue in the bottomm right of the screen
	* The next screen is about what apps should be installed
		* Click on the circle next to "Normal installation"
		* Check the boxes that say "Download updates while installing Ubuntu" and "Install third-party..."
	* Click Continue in the bottom right of the screen
	* On the next screen click the circle next to "Erase disk and install Ubuntu"
	* Click "Install now" in the bottom right of the screen
	* When the window saying "Write changes to disks?" pops up, click Continue
	* On the next screen select the time zone region that you are in on the provided map
	* Click Continue in the bottom right of the screen
	* You need to enter some information on the next screen. For this install, the information I use is:
	
	Prompt| Value
	--- | ---
	Your name: | test
	Your computer's name | test-VirtualBox
	Pick a username: | test
	Choose a password : | test
	Confirm your password : | test
	 * *Note: you should **not** configure your system in this way on anything but testing*
	 * Make sure circle next to "Require password to login" is selected
	 * Eventually a box will pop up saying "Installation Complete at the top"
		 * Click the box in the bottom right that says "Restart Now"
	 * When the machine restarts there may be a popup saying something like "Remove installation media then click"
	 * I just exited the vm via the X button in top right then tried starting it up again and it worked fine