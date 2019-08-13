# Building an Ubuntu 18.04 VM

This is a very inaccurate guide to building an Ubuntu VM that you can use until we have a kickstart file to do it for you.

## Build a golden ubuntu 18.04 image for reuse

* TBD

### Now that you have a VM built....

You need to shut it down so we can take a copy of this manual installation disk image so we can use it again later.  In `virt-manager` or via the command line, shutdown the VM.

You can now copy the disk image away, i.e. `cp --sparse=always u1804.raw u1804-gold.raw`


## Using your disk image

Take a copy, i.e.  `cp --sparse=always u1804-gold.raw u1804-$(date +%Y%m%d).raw`
Now in `virt-manager`:
* create a new virtual machine
** select "Import existing disk image", click Forward
** navigate to where u1804-$(date +%Y%m%d).raw is, select the image, click Choose Volume
** Enter Ubuntu 18.04 as the operating system type, click Forward
** Change the memory size to 14888 and the CPUs to 4, click Forward
** Change the name of the VM to `u1804-$(date +%Y%m%d)`, click Finish

Once the VM boots, in another terminal window do the following:
* [laptop]$ HOSTNAME=u1804-$(date +%Y%m%d)
* [laptop]$ IPADDR=$(sudo virsh domifaddr $HOSTNAME | tail -n +3 | awk '{print $4}' | cut -d'/' -f1 | while read ipa ; do ping -c1 $ipa >& /dev/null; [[ $? -eq 0 ]] && echo "$ipa" && break; done)
* [laptop]$ ssh-copy-id $IPADDR  # This assumes the username on your local laptop is what you used in the VM
* [laptop]$ ssh $IPADDR
* [vm]$ sudo hostnamectl set-hostname hostname     # where hostname is `u1804-$(date +%Y%m%d)`
* [vm]$ sudo sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
* [vm]$ sudo service ssh restart
* [vm]$ sudo passwd root
* [vm]$ sudo apt update && sudo apt upgrade -y
* [vm]$ exit
* [laptop]$ ssh-copy-id root@$IPADDR

And now you're ready to run the start-devstact.yml play
