# RHCSA 8 Automated Practice Deployment
_Powered by Ansible and Vagrant_ 

## Installation options below:
## macOS
_Gatekeeper will block virtualbox from installing. All you have to do is go into Security & Privacy of System Preferences and click Allow under the General tab and rerun installation._
##### Install all at once with the command below:
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" && xcode-select --install &&brew install ansible ; brew install python ; brew install --cask vagrant ; brew install --cask VirtualBox ; brew install --cask virtualbox-extension-pack ; vagrant plugin install vagrant-guest_ansible
```

##### Alternatively, you can install everything individually below.
- [Install the Latest Version of Vagrant](https://www.vagrantup.com/downloads.html) - (`brew install --cask vagrant`)
    - Vagrant Plugin - `vagrant plugin install vagrant-guest_ansible`
- [Install the Latest Version of Virtualbox](https://www.virtualbox.org/wiki/Downloads) (`brew install --cask VirtualBox`)
    - Virtual Box Extension Pack (`brew install --cask virtualbox-extension-pack`)

##### Once the above software is installed. Do the following if you're running the environment on Mac:
1. Create a separate `~/bin` directory and `cd` to it.  (The directory doesn't have to be ~/bin, it can be anything you want.)
2. Change to the `rhcsa8env` directory that is now in your `~/bin` directory.
3. Run `vagrant up` to deploy the environment (If the environment has a designated repo VM it will take the longest to deploy the first time only, this is because the repo system has all the packages available to the base release but will be quicker on subsequent deployments.)



##### Also, install the Virtualbox extension pack below
- [Install the Virtual Box Extension Pack](https://www.virtualbox.org/wiki/Downloads)



##### Once the above software is installed. Do the following if you're running the environment on Linux:
1. Create a separate `~/bin` directory and `cd` to it.  (The directory doesn't have to be ~/bin, it can be anything you want.)

2. Change to the `rhcsa8env` directory that is now in your `~/bin` directory.
3. Run `vagrant up` to deploy the environment (If the environment has a designated repo VM it will take the longest to deploy the first time only, this is because the repo system has all the packages available to the base release but will be quicker on subsequent deployments.)

**Also, don't be spooked by any scary red font during the setup process. There are known issues that won't have a negative affect on the environment.**

_Now the deployment should be up and running!_



## Notable commands to control the environment:
- `ansible-playbook playbooks/reset.yml` - Used for resetting Servers 1 and 2 after attempting the practice exam in the Red Hat Certs Slack workspace practice exam channel. 
- `vagrant up` - Boots and provisions the environment
- `vagrant destroy -f` - Shuts down and destroys the environment
- `vagrant halt` - Only shuts down the environment VMs (can be booted up with `vagrant up`)
- `vagrant suspend` - Puts the VMs in a suspended state
- `vagrant resume` - Takes VMs out of a suspended state

## Other Useful Information:
You can also use the VirtualBox console to interact with the VMs or through a terminal. If you need to reset the root password, you would need to use the console. I'm constantly making upgrades to the environments, so every once and awhile run `git pull` in the repo directory to pull down changes. If you're using Windows, it's recommended to use Github Desktop so you can easily pull changes that are made to the environment. The first time you run the vagrant up command, it will download the OS images for later use. In other words, it will take longest the first time around but will be faster when it is deployed again. You can run `vagrant destroy -f` to destroy your environment at anytime. **This will erase everything**. This environment is meant to be reuseable, If you run the `vagrant up` command after destroying the environment, the OS image will already be downloaded and environment will deploy faster. Deployment should take around 15 minutes depending on your computer. You shouldn't need to access the IPA server during your practice exams. Everything should be provided that you would normally need during an actual exam. Hope this helps in your studies!

## Included systems:
- repo.eight.example.com
- server1.eight.example.com
- server2.eight.example.com

## System Details:
> server1
- 192.168.55.150
- Gateway - 192.168.55.1
- DNS - 8.8.8.8
> server2
- 192.168.55.151
- Gateway - 192.168.55.1
- DNS - 8.8.8.8

There is a Repo/AppStream available to use from `http://repo.eight.example.com/BaseOS` and `http://repo.eight.example.com/AppStream`

## Accessing the systems
Remember to add the IP addresses to your local host file if you want to connect to the guest systems with the hostname.
Username - vagrant
Password - vagrant
- For root - use `sudo` or `sudo su`
Access example - `ssh vagrant@192.168.55.150` or `vagrant ssh system`

## Help
If you're having problems with the environment, please submit an issue by going to the `ISSUES` tab at the top. If you have more questions, looking for practice exams to use against this environment, or just looking for a fantastic Red Hat community to join to get your questions answered, check out the Red Hat Certs Slack Workspace. You can find the invite link at the top of this page next to the description.

## Known Issues:

Running the 'vagrant up' environment build will fail If HyperV is installed on the Windows VirtualBox host.
Error is usually "VT-x is not available. (VERR_VMX_NO_VMX)" or similar, when the script attempts to boot the first VM.

Resolution seems to be either remove HyperV, or preventing its hypervisor from starting with the command:
bcdedit /set hypervisorlaunchtype off, followed by a reboot.
