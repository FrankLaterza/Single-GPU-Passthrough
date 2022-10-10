
# Single GPU Passthrough
Welcome to my guide on setting up a virtual machine with GPU passthrough with only a single graphics card. This guide was made along side this youtube tutorial [here](https://youtu.be/UOk5Mzu53lI). If you get stuck anywhere along the way please check out this guide [here](https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/1)-Preparations) for more information. What are you waiting for? Switch to linux!!
### My specs 
amd cpu <br />
nvidia graphics card

## Preparations

### Bois Settings 
virtualization is enabled <br />
iommu is enable <br />
secure boot is disabled before install <br /> 

your os needs to be installed in UEFI mode run the command below to check
```
[ -d /sys/firmware/efi ] && echo "Installed in UEFI mode" || echo "Installed in Legacy mode"
```

## Edit Boot Options
Normaly you would change the grub for boot options, but popos doesn't use grub. so you need to change the boot optoins with kernalstub

### change boot options

#### For Amd:
```
sudo kernelstub -a "amd_iommu=on iommu=pt video=efifb:off"
```

#### For Intel
```
sudo kernelstub -a "intel_iommu=on iommu=pt video=efifb:off"
```
update with
```
sudo bootctl update
```
You can check if this was done correctly by checking the file below
```
sudo cat /etc/kernelstub/configuration
```
You should see the new boot options that we placed in there.

Make sure to reboot and check that your system booted with the new options with the command below.
```
sudo cat /proc/cmdline
```

## check that iommu is valid

To see if iommu is working run below.
```
sudo dmesg | grep IOMMU
```
Now run the command below to check what pci slot your grapics card is in.
```
lspci
```

Now you want to check your iommu grouping. Run the script in this repository label check_iommu_group.sh to see the grouping. You want to make sure that it matches with your pci slots. Make sure this executable as well. 

MAKE SURE TO CREATE THE SCRIPT!!! run with
```
sudo ./check_iommu_group.sh
```

# Set Up Libvirt

This command will install virtual machine manger and libvert things
```
sudo apt install qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils virt-manager ovmf
```

Once installed we need to edit the libvirt conf file
```
sudo nano /etc/libvirt/libvirtd.conf
```

Next find and uncomment the following lines 

```
unix_sock_group = "libvirt"
unix_sock_rw_perms = "0770"
```

Make sure this is at the end of the file (helps with logs)
```
log_filters="1:qemu"
log_outputs="1:file:/var/log/libvirt/libvirtd.log"
```
### Give Permissions
run the following to give your user the proper premissions

```
sudo usermod -a -G libvirt $(whoami)
sudo systemctl start libvirtd
sudo systemctl enable libvirtd
```
Check with 
```
sudo groups $(whoami)
```
should show that you are in the group


### Change The Qemu Conf

run the following below to edit the qemu file
```
sudo nano /etc/libvirt/qemu.conf
```
You want to change the following lines. Replace "your username" with your username
```
#user = "root" to user = "your username"
#group = "root" to group = "your username"
```
Now restart libvert
```
sudo systemctl restart libvirtd
```
Next we need to add the grouping
```
sudo usermod -a -G kvm,libvirt $(whoami)
sudo groups $(whoami)
```
Enabling network
```
sudo virsh net-autostart default
sudo virsh net-start default
```
# Set Up Virtual Machine

Download virt io drivers
https://docs.fedoraproject.org/en-US/quick-docs/creating-windows-virtual-machines-using-virtio-drivers/index.html

Download windows iso 
https://www.microsoft.com/en-us/software-download/windows10ISO

Now setup windows and add the vfio drivers. If you need help with this part check out the video listed at the top for a step by step guide.

# Libvirt Hooks

Look at these for a guide
https://github.com/joeknock90/Single-GPU-Passthrough

check the log files 
```
sudo cat /var/log/libvirt/qemu/win10.log 
```

make sure to set up an ssh and vnc 

launch and test!
good luck
