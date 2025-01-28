### How to make vagrant image rhel9 using vmware fusion

Step 1. Download RHEL 9 iso
- https://developers.redhat.com/products/rhel/download#publicandprivatecloudreadyrhelimages
- Select "Red Hat Enterprise Linux 9.5 for ARM 64"
    - Below link required sign up to obtain the iso file
        - https://developers.redhat.com/content-gateway/file/rhel/Red_Hat_Enterprise_Linux_9.5/rhel-9.5-aarch64-boot.iso


Step 2. Install VMware fusion and create a vm using the RHEL 9 iso
- User account 

Username:
vagrant
password:
vagrant

- Root account:

Username:
root
password:
root

root/root


Step 3. Install vmware tools and depenedencies.
- [How to install VMware tools on RHEL](https://access.redhat.com/solutions/1447193)

https://knowledge.broadcom.com/external/article/315363/how-to-install-vmware-tools.html

- Click "Virtual Machine" => Find the "Install Vmtools"
Select:

For Fusion: Virtual Machine > Install VMware Tools.
For Workstation: VM > Install VMware Tools.
For Player: Player > Manage > Install VMware Tools.

- If the option is grey out for "Install VMtools", use this 

```bash
- source from https://community.broadcom.com/vmware-cloud-foundation/communities/community-home/digestviewer/viewthread?MessageKey=8b728044-2cf4-42b7-a03c-6789b599814b&CommunityKey=0c3a2021-5113-4ad1-af9e-018f5da40bc0#bm8b728044-2cf4-42b7-a03c-6789b599814b


You should still be able to open the iso images and run vmware tools that way...

To do so here are the steps fully written out.

First copy the iso out of the VMware Fusion application bundle.

- Start Finder, go to Applications

- Right-click on VMware Fusion -> select "Show Package Contents"

- Navigate to Content -> Library -> isoimages

- copy "windows.iso" to your documents folders (do not move, but copy!)

Now attach the iso to your virtual CD drive

- menu -> Virtual Machine -> CD/DVD -> Choose Disc of Disc Image...

- navigate to the documents folder and select "windows.iso"

- menu -> Virtual Machine -> CD/DVD -> Connect CD/DVD

The installer should now be available in your guest OS under drive
```

Step 4. Setup vagrant user with an insecure ssh keypair, so Vagrant is able to ssh into our VM.

- Warning: do not use this key to any production server! the key has been uploaded to Github.

```bash
mkdir .ssh
chmod 0700 .ssh
cd .ssh/
wget https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub -O authorized_keys
chmod 0600 authorized_keys
```


Lastly, we have to allow users in the wheel group to run commands with sudo without the need to enter the password. Type sudo visudo and change the following line

```
%wheel ALL=(ALL) ALL
to
%wheel ALL=(ALL) NOPASSWD: ALL
```

The VM is now ready to be exported to a vagrant box. First shutdown the VM and then on your own computer (i.e. not in the VM) run the following commands
```
vagrant package --base <vm_name_in_virtualbox>
vagrant box add --name rhel7 package.box
```

That’s all it takes! Now the new RedHat machine can be utilized with Vagrant. You can try it out by running

```
vagrant init rhel7
vagrant up
```