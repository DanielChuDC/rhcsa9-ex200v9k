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

- You can generate a key by `ssh-keygen -f vagrantKey`


- Warning: do not use this key to any production server! the key has been uploaded to Github. Source: https://developer.hashicorp.com/vagrant/docs/providers/vmware/boxes


```bash


mkdir .ssh
chmod 0700 .ssh
cd .ssh/
wget https://raw.githubusercontent.com/hashicorp/vagrant/refs/heads/main/keys/vagrant.pub -O authorized_keys


chmod 0600 authorized_keys
```


Lastly, we have to allow users in the wheel group to run commands with sudo without the need to enter the password. Type sudo visudo and change the following line

```
%wheel ALL=(ALL) ALL
to
%wheel ALL=(ALL) NOPASSWD: ALL
```

Unregister the rhel account and remove log history

```
sudo subscription-manager register --username <redhat_account_username> --password <redhat_account_password> --auto-attach


Unregister the system from the Red Hat subscription:
sudo subscription-manager unregister

Remove subscription manager configurations and history:
sudo subscription-manager clean

(Optional) Remove the log files related to subscription manager:
sudo rm -rf /var/log/rhsm/*

```


# Clear bash History for All Users: For each user, clear the bash history:

```bash
for user in $(awk -F':' '{ print $1}' /etc/passwd); do
    home_dir=$(getent passwd $user | cut -d: -f6)
    if [ -d "$home_dir" ]; then
        sudo rm -f "$home_dir/.bash_history"
        sudo touch "$home_dir/.bash_history"
    fi
done
history -c
history -w

```
Clear System-Wide Logs: Use these commands to clear system logs:

```bash
sudo rm -rf /var/log/*
sudo journalctl --rotate
sudo journalctl --vacuum-time=1s
```

Clear bash History of the Current Session: For the active session:

```
history -c
history -w
```

Remove Users' Command History from /root and /home
```
sudo find /root /home -name ".bash_history" -type f -exec rm -f {} \;


```


---


The VM is now ready to be exported to a vagrant box. First shutdown the VM and then on your own computer (i.e. not in the VM) run the following commands

- Export it into OVF

```bash
cd '/Applications/VMware Fusion.app/Contents/Library'
cd 'VMware OVF Tool'


# example
#  ./ovftool --acceptAllEulas --noSSLVerify <source_path_to_vm>.vmx <destination_path>

./ovftool --acceptAllEulas --noSSLVerify  "$HOME/Virtual Machines.localized/rhel9-arm.vmwarevm/rhel9-arm.vmx" ~/Downloads/rhel9-arm-1.ovf


```


- Use vagrant 

Create a Vagrant Box from OVF
the --base option in vagrant package is typically for VirtualBox

For VMware, already exported the VM as rhel9-arm.ovf (along with the VMDK and MF files), run the command below to pack


Command:

```bash
vagrant package --base ./rhel9-arm.ovf --output rhel9-arm.box --provider vmware_desktop


cd  "$HOME/Virtual Machines.localized/rhel9-arm.vmwarevm/rhel9-arm.vmx"

cp -r * /vmwarevmx

tee metadata.json <<EOF
{
  "provider": "vmware_desktop"
}
EOF


cd /

# remove the lock file
rm *.lck 

# include the contents of the ovf/ directory in the .tar.gz file but exclude the directory name itself
tar cvzf rhel9-arm.vmware.box -C vmwarevmx .
# tar cvzf rhel9-arm.vmware.box -C ovf .  # A "vmx" file was not found in the box specified. A "vmx" file is required to clone, boot, and manage VMware machines. 

md5 rhel9-arm.vmware.box
# MD5 (rhel9-arm.vmware.box) = 3ed2a6b4dd37b48c580eec87469cdd09

# metadata.json file used with the vagrant box add
tee metadata.json << EOF
{
  "name": "dc/rhel9-arm",
  "description": "rhel 9 arm version.",
  "versions": [
    {
      "version": "0.1.0",
      "providers": [
        {
          "name": "vmware_desktop",
          "url": "file:///$HOME/Documents/GitHub/rhcsa9-ex200v9k/lab_setup/rhcsa9env-master/rhel9-arm.vmware.box",
          "checksum_type": "md5",
          "checksum": "3ed2a6b4dd37b48c580eec87469cdd09"
        }
      ]
    }
  ]
}
EOF

# Add the Vagrant Box
# Once the box is packaged, you need to add it to Vagrant using:

vagrant box add metadata.json --provider vmware_desktop --force

vagrant init dc/rhel9-arm

VAGRANT_VAGRANTFILE=Vagrantfile.vmware-arm vagrant up 


# --- To use local box 

Vagrant.configure("2") do |config|
  config.vm.box = "dc/rhel9-arm"
end


```


```
vagrant package --base <vm_name_in_virtualbox>
vagrant box add --name rhel7 package.box
```

That’s all it takes! Now the new RedHat machine can be utilized with Vagrant. You can try it out by running

```
vagrant init rhel7
vagrant up
```




### Reference:

https://stackoverflow.com/questions/38056018/how-do-i-set-the-version-of-vagrant-box-created-with-vmware-fusion-using-a-meta
