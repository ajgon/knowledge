# Create custom Vagrant Box

## Create VirtualBox VM

* Download desired ISO image (I'll use debian as an example here)
* create new virtual machine, using VirtualBox GUI
    * set desired disk and memory size
    * set *PAE/NX* on
    * disable audio
    * disable USB
* Boot and install the ISO image
    * **root password**: vagrant
    * **full name for the new user**: Vagrant
    * **username for your account**: vagrant
    * **password for the new user**: vagrant
    * software to install:
        * SSH server
        * standard system utilities
* log in, and install *guest additions*:
    * apt-get install linux-headers-amd64 build-essential dkms
    * **Devices -> Insert Guest Additions CD Image...**
    * mount /dev/cdrom /media/cdrom
    * sh /media/cdrom/VBoxLinuxAdditions.run
* for easier information transfer between the guest and the host (namely: cut and paste + command history), ssh to the guest:
    * with VirtualBox GUI, add a port forwarding rule: host port 2222 to guest port 22
    * *ssh* to the guest with port 2222 and username `vagrant` (root is not authorized)
    * `su`
* copy [Vagrant public key](https://raw.githubusercontent.com/hashicorp/vagrant/master/keys/vagrant.pub) into `/home/vagrant/.ssh/authorized_keys`
* `chmod 0700 /home/vagrant/.ssh`
* `chmod 0600 /home/vagrant/.ssh/authorized_keys`
* `apt-get install sudo`
* add to *sudoers* file: `vagrant ALL=(ALL) NOPASSWD: ALL`
* add `UseDNS no` to `/etc/ssh/sshd_config`, and uncomment the line starging with `AuthorizedKeysFile`. Restart sshd: `systemctl restart sshd`
* cleanup
    * `rm -rf /usr/share/doc`
    * `find /var/cache -type f -exec rm -rf {} \;`
    * `dd if=/dev/zero of=/EMPTY bs=1M`
    * `rm -rf /EMPTY`
* power off the VM

## Create Vagrant Box packaging

* `vagrant package --base "My VirtualBox Name"`
* `vagrant box add mypackage package.box`
* `vagrant init mypackage`
* `vagrant up`
* `vagrant ssh`
