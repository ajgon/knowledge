# How to create a VirtualBox machine with encrypted storage with Vagrant

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

PASSWORD_PATH = ".password"
PASSWORD_ID_PATH = ".password_id"

# Make sure to have installed vagrant-triggers plugin
# > vagrant plugin install vagrant-triggers

# After the first `vagrant up` stop the VM and execute the following steps
# Take the identifier of the storage you want to encrypt
# > HDD_UUID=`VBoxManage showvminfo <VM_NAME> | grep 'SATA.*UUID' | sed 's/^.*UUID: \(.*\))/\1/'`
# Store your usernname (whitespaces are not allowed) in a variable
# > USERNAME="<YOUR_USER_NAME_WITHOUT_WHITESPACES>"
# Encrypt the storage, enter the password when asked
# > VBoxManage encryptmedium $HDD_UUID --newpassword - --newpasswordid $USERNAME --cipher "AES-XTS256-PLAIN64"
# Store the username in a file named .password_id
# > echo $USERNAME > .password_id
# Now, the next time you start the VM you'll be asked for the same password

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/vivid64"
  config.vm.box_check_update = false
  config.vm.hostname = "secure"

  config.trigger.before :up do
    if File.exists?(PASSWORD_ID_PATH)
      password_id = File.read(PASSWORD_ID_PATH).strip
      print "The VM is encrypted, please enter the password\n#{password_id}: "
      password = STDIN.noecho(&:gets).strip
      File.write(PASSWORD_PATH, password)
      puts ""
    end
  end

  config.trigger.after :up do
    File.delete(PASSWORD_PATH) if File.exists?(PASSWORD_PATH)
  end

  config.trigger.after :destroy do
    File.delete(PASSWORD_ID_PATH) if File.exists?(PASSWORD_ID_PATH)
  end

  config.vm.provider :virtualbox do |vb|
    vb.name = "secure"
    vb.gui = false

    if File.exists?(PASSWORD_ID_PATH)
      password_id = File.read(PASSWORD_ID_PATH).strip
      vb.customize "post-boot", [
        "controlvm", :id, "addencpassword", password_id, PASSWORD_PATH, "--removeonsuspend", "yes"
      ]
    end
  end
end
```
