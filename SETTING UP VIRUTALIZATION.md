*Note that this documenation is for setting up virtualization on Ubuntu 20.04 focal*
# VT-x, VT-d and Sr-IOV
- Hardware virutalization (VT-x) makes the virtual machine run faster
- Virtualization Technology for Directed I/O (VT-d) lets us use Input-Output Memory Management Unit (IOMMU) which is useful for configuring and using PCI stuff
- Single Root I/O Virtualization (SR-IOV) like VT-d, makes working with PCI easier
## Enabling
- First, make sure that hardware virualization is supported for your CPU
```bash
$ lscpu | grep  'Virtualization\|Architecture'
Architecture:                    x86_64
Virtualization:                  VT-x
```
- This shows that Virtualization is enabled, and the architecture of the cpu is 64 bit. The cpu isn't required to be 64 bit, but it is significantly better than a 32 bit processor
- **Enable VT-x**
	- Note that VT-x is enabled by default a lot of the time. 
	- Check if it's enabled with: ```$ sudo kvm-ok```
	- If it returns *KVM acceleration can be used* then you're good to go, otherwise it needs to be enabled in BIOS
- **Enable VT-d and SR-IOV**
	- You can verify that VT-d is enabled by checking dmesg, which is kind of like log files for the linux kernel. 
	- run ```dmesg | grep -i 'iommu'```
	- if you see any hits, then VT-d  and SR-IOV are enabled, if not then it probably needs turning on in BIOS like VT-x
	- SR-IOV is almost never enabled by defualt, so to enable it do the following:
		- Note this can be checked by grepping dmesg for iommu again, as this should enable it
```bash
# Use your favorite text editor to edit /etc/default/grub (with sudo)
# Change the following option to read like so:
GRUB_CMD_LINE="intel_iommu=on"

# Next, run these commands
$ sudo update-grub
$ sudo update-initramfs -u
$ sudo reboot

# Once you're relogged in, verify iommu is enabled like before
$ dmesg | grep -i iommu
# This should show some messages, one of which says 'Intel-IOMMU: enabled'
```

# PCI stuff
Now, we need to enable virtualization for our network interface. First, we must find the maximum amount of VF's allowed by the hardware.
```bash
# First, need to get the PCI ID of the network card
$ lshw -class network -businfo
# Note this command may need sudo

# Now, take the PCI ID (number with colons right next to the @) and do the following
$ sudo lspci -vvv -s <PCI ID>

# This will spit out a lot of info, look for the line that looks like this:
Initial VFs: 8, Total VFs: 8, Number of VFs: 0, Function Dependency Link: 00
# The 'Total VF's is how many VF's this card allows

# Now, enable the virutalization of the interface like so:
$ echo <number of VFs> > /sys/class/net/<device>/device/sriov_numvfs
# Note that the path can be different for each system. 'net' could be something like 'network' or something unrelated like 'ininiband' 
```
Unfortuantely, this is a change that must be made every time the system is booted. So save the echo command to  */etc/rc.local* and make it executable.
```bash
# Also worth noting that you should add a shebang at the top of the file
# ex. #!/usr/bin/bash
$ chmod +x /etc/rc.local
```

Next, we need to make sure that the PCI devices are grouped into separate IOMMU groups. These devices are what the last step created, and if they're grouped together that's very bad news.

Run this script to verify they're all in different groups. There will likely be more ouput than you expect/need so I've added a grep to make it cleaner and easier to read.
```bash
shopt -s nullglob
for g in /sys/kernel/iommu_groups/*; do
        echo "IOMMU Group ${g##*/}:"
        for d in $g/devices/*; do
                echo -e "\t$(lspci -nns ${d##*/})"
        done;
done;
```
Run with sudo and greps to make checking easier
```bash
$ sudo ./pci_checker.sh  | grep -A1 -B1 -i ether
```
You're looking for output like this, where you'll notice there's no more than one to a group
```bash
IOMMU Group 160:
-e      18:14.3 Ethernet controller [0200]: Intel Corporation X550 Virtual Function [8086:1565]
IOMMU Group 161:
-e      18:14.5 Ethernet controller [0200]: Intel Corporation X550 Virtual Function [8086:1565]
IOMMU Group 162:
-e      18:14.7 Ethernet controller [0200]: Intel Corporation X550 Virtual Function [8086:1565]
IOMMU Group 163:
-e      18:15.1 Ethernet controller [0200]: Intel Corporation X550 Virtual Function [8086:1565]
IOMMU Group 164:
-e      18:15.3 Ethernet controller [0200]: Intel Corporation X550 Virtual Function [8086:1565]
IOMMU Group 165:
-e      18:15.5 Ethernet controller [0200]: Intel Corporation X550 Virtual Function [8086:1565]
```
# Install packages
## libvirt
Formal documentantion for installing Libvirt available @ *https://ubuntu.com/server/docs/virtualization-libvirt*
- Install libvirt daemon 
```bash
$ sudo apt update
$ sudo apt install qemu-kvm libvirt-daemon-system
```
- Add management user to libvirt group
```bash
$ sudo adduser $USER libvirt
```
- Relogin to make these changes effective
## virt-manager
- Note this needs to be installed on a machine with a GUI, does not need to be machine that will be doing virtualization
- This guide will assume that you are using a separate linux host to run virt-manager. A VM works for this if you don't have a physical host, so long as it has a GUI.

Formal documentation for installing virt-manager found @ *https://help.ubuntu.com/community/KVM/VirtManager*
```bash 
$ sudo apt-get install virt-manager
```
## qemu
This should have been installed as a dependency for libvirt

# Managing VMs
Now, make sure that the libvirt daemon is running on the machine you want to run the VMs on
```bash
# You will need to use your sudo password at least once for this
$ systemctl enable --now libvirtd.service
```
You ==NEED== to create an SSH key on the machine that you will be managing from and then use ```ssh-copy-id``` to transfer the private key to the remote machine. If you don't do this step you won't be able to manage or create VM's remotely.
```bash
# Create the public key/private key pair
$ ssh-keygen -t rsa
# You can use a passcode if you want to; it is not mandatory

# Now, copy the key from local to the remote machine
$ ssh-copy-id -i <path to private key> root@example.machine.com
# Enter the password for the user you're SSHing as (root in this example) and it should print a message asking you to try SSHing into the machine 
# Verify it worked via SSH
$ ssh -i id_rsa root@example.machine.com
```
Now, you can launch virt-manager and add a connection to the machine
- Check the box for connecting via SSH, enter the appropriate user (Root in this example) 
- Make sure the host name is correct: it should look like when you SSH

You should now see an entry under QEMU/KVM on the dashboard. Right click where it says QEMU/KVM and click on new.
- Install via 'Local install media (ISO image or CDROM)'
- Provide the path to the installation medium and select the operating system you are installing
	- Note the installation media should be on the host that will be running the VM not the one running virt-manager
- Configure memory and number of CPUs.
- For the disk, create a custom storage, and click Manage
	- Select a storage pool, or create a new one.
	- Select a name and capacity for the disk. qcow2 is the recommended format
	- Select the volume you just creted and click Choose Volume
- Click Forward, then name the new VM and make sure to select the checkbox to Customize configuration before install.
- Under Overview, change the Chipset to Q35 and the Firmware to an OVMF UEFI firmware and click Apply
- Select IDE Disk 1 and change the Disk type to VirtIO and click apply
- Select IDE CDROM 1 and change the Disk type to SATA and click Apply.
- Select the NIC and change the Adapter type to VirtIO and click Apply
- Select Video QXL and change the Model to Virtio
- Select Boot Options, check the CDROM's checkbox and move it to the top of the list
- Click Begin Installation at the top of the screen
- Install the operating system.
- Once the operating system is installed and you are prompted to remove the installation CD, click the green "i" button to go back into the VM configuration and remove the CDROM device. Do not delete the storage medium as this will remove the installation ISO from your system.
- Restart the VM and it should be ready for use.

## PCI Stuff v2
This is where all the PCI configuration we did earlier will come in handy. 
- Launch virt-manager from your managment host
- Make sure the VM you are working on is turned off.
- Click on the blue circle with an 'i' in it at the top of the screen. This should show you the VM's hardware details.
- Click on 'Add Hardware' in the bottom left corner of the screen
- In the new screen that pops up click on 'PCI Host Device' on the left side of the screen
- You should now see a list of PCI devices. You need to select one of the virtual functions that you created in the PCI section earlier. If you don't know the names of these you can find them out by running ```
lshw -c network -businfo | grep 'Virtual Function' ``` on the host machine. For example:
```bash
honeynet@honeynet:~/raw_captures$ lshw -c network -businfo | grep 'Virtual Function'
WARNING: you should run this program as super-user.
pci@0000:18:10.1  enp24s0f1v0   network        X550 Virtual Function
pci@0000:18:10.3                network        X550 Virtual Function
pci@0000:18:10.5  enp24s0f1v2   network        X550 Virtual Function
pci@0000:18:10.7  enp24s0f1v3   network        X550 Virtual Function
```
- Like the warning says, if you don't see any output run it as super user
	- Also, if you are running multiple VM's it's best if you use one VF per VM.
- Once you've selected an appropriate VF, click on finish in the bottom right of the screen.
- Now, when you boot up the VM you should see an additional interface that was not there before which cooresponds to the Virtual Function

## Network Configuration
The next step is to give this VM the IP's that you are going to be monitoring so that traffic routed accordingly. 
- For this install, we have a /22 network that we are monitoring.
- You may need some assistance from your network admins to figure out your address range. In this installation our /22 is on the addresses of: 192.168.72.0/22
- So what you must do in this situation is create a *netplan* configuration file that has *every* address of this /22 in it.
- This configuration file will be located at /etc/netplan/*somthing*_config.yml
	- If this file/directory doesn't exist you will need to initialize it and also make sure your system has netplan. It should be default, though.
- So the netplan config will look something like this: 
```bash
network:
  version: 2
  renderer: networkd
  ethernets:
    enp24s0f1:
      addresses:
         -  128.105.72.10/22
         -  128.105.72.11/22
         -  128.105.72.12/22
         -  128.105.72.13/22
		 
		 # ... keep doing this for every addr in the /22
		 
		 -  128.105.75.254/22
      gateway4: 128.105.72.1
      nameservers:
        addresses: [128.104.254.254, 144.92.254.254]
    eno1:
      optional: true
    eno2:
      optional: true
    enp24s0f0:
      optional: true
```
- In this example, the first 10 addresses are intentionally left out. This is to give the subnet a gateway and also allocate some addresses for the VM's host machine.
	- Like I said, if you don't know your address range, consult your network admins or whoever is in charge of routing you the traffic you will be analyzing.
- Once you've made these changes, you need to apply them.
```bash
# Make sure your configuration is valid first
$ sudo netplan try
# This will attempt to apply changes and let you know if the config is not valid

# Once you know config is valid, apply changes
$ sudo netplan apply
```
- Now you can make sure everything is working by using tcpdump to see if packets addressed for your address range are hitting the VM.
## IPTables
One quick thing you need to do before running honeytrap is to configure your VM OS to drop incoming packets.
- Since you probably want to be able to manage the VM, you should only drop the packets received on your public interface, or the one you set up using virtual functions.
```bash
# Set iptables rule
$ sudo iptables -A INPUT -i <interface of VF> -j DROP
# This way you will still be able to ssh to the VM using the default shared subnet between host and VM
# Save the rules
$ sudo iptables-save > /etc/iptables/rules.v4
# Might need to initialize directory and run as super user
```

## Honeytrap
Now that all the VM and host configuration is taken care of, it's time to get honeytrap on the system.
```bash
# First, get the source code from github
$ cd <directory where you want the source>
$ git clone https://github.com/honeytrap/honeytrap

# Now to build the binary.
# These commands are taken from the Dockerfile in the git repo
$ CGO_ENABLED=1
$ GOOS=linux
$ go build -a -installsuffix cgo -tags="" -ldflags="$(go run scripts/gen-ldflags.go)" -o /usr/local/sbin/canary
# This will place the 'canary' binary in /usr/local/sbin
# It should be in the path by default. Test with
$ canary
# If you see 'canary: command not found' or something similar, add directory to the path
```
Now that the binary is built, you need a simple configuration file. Place the following in *config.toml*
```toml
[channel.console]  
type="console"  
  
[channel.filelog]  
type="file"  
filename="/data/honeytrap/honeytrap.log"  
  
[[filter]]  
channel=["console", "filelog"]
```
- Note that the directory /data/honeytrap may need to be initialized. Alternatively you can use a different location by specifying it with filename=


Now you can try to run the canary sensor!
```bash
$ sudo canary --config config.toml --data /data
# This should produce a good amount of output talking about honeytrap starting 
# must be ran as super user, unless your current user has some special permissions
# You should see output in the console whenever traffic hits honeytrap.
# Verify this by checking in the data directory you entered into the config file. /data/honeytrap/honeytrap.log in this example
```