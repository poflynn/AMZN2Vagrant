# How to make a  VirtualBox-based Vagrant Amazon 2 Linux Base Box

*Improvements or suggestions towards this document are welcome.*

* Download latest `.vdi` file. You should be able to find the most recent `.vdi`
file at https://cdn.amazonlinux.com/os-images/latest/virtualbox/ Alternatively,
here is a direct link to the most current version at the time of writing this (January 2019) https://cdn.amazonlinux.com/os-images/2.0.20181114/virtualbox/amzn2-virtualbox-2.0.20181114-x86_64.xfs.gpt.vdi

* Build virtual CD that contains cloud-init data (Will generate a file `seed.iso`. Ensure that this is ran from the same
directly which contains the files `user-data` and `meta-data` from this repository.
###### Linux:
```bash
genisoimage -output seed.iso -volid cidata -joliet -rock user-data meta-data
```
###### Mac:
```bash
brew install cdrtools
mkisofs -output seed.iso -volid cidata -joliet -rock user-data meta-data
```

### Creating VM in VirtualBox

* Create new VM with the following criteria
  - name: AMZN
  - type: linux
  - version: Other Linux 64bit
* Select the `.vdi` image downloaded earlier as the Hard Disk (option might state
something like "Use an existing virtual hard disk file")
* Add the `seed.iso` file generated above to the virtual CD drive
* Start the Amazon Linux 2 virtual machine

### Installing VirtualBox Guest Additions

* Remove the cloud-init ISO (`seed.iso`)image from VirtualBox settings
* Log in to the virtual machine (this can be done in the virtual box terminal window)
  - user: root
  - password: vagrant
* From the VirtualBox menu select Devices->Insert Guest Additions CD Image
* Inside the terminal, we must now mount the inserted CD (I manually typed these commands into
the VirtualBox terminal, suggestions of a better approach are welcome)
```bash
sudo yum -y update

# next line suggested at https://stackoverflow.com/a/37706087/1241791
# to solve shared folder issue later on
sudo yum -y install kernel-headers kernel-devel

#mount the inserted guest additions CD
mount -r -t iso9660 /dev/cdrom /media

cd /media
./VBoxLinuxAdditions.run
systemctl enable vboxadd.service
```
Ignore tainted kernel message

#### Post processing
```bash
#Delete Bash command history
export HISTSIZE=0

#Delete cache of yum
yum clean all
rm -rf /var/cache/yum

#Optimize the area of the virtual hard disk
dd if=/dev/zero of=/ZERO bs=1M
rm -f /ZERO
shutdown -h now
```

#### Create Base Box from virtual machine

These commands to be executed on the host machine
```bash
vagrant init
vagrant package --base AMZN --output amazonlinux2.box
```
The output amazonlinux2.box is the file of Base Box

#### Notes:
The ssh key in the user-data file is the usual Vagrant insecure keypair from [here](https://github.com/hashicorp/vagrant/tree/master/keys)

#### References:
http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/amazon-linux-2-virtual-machine.html

https://qiita.com/aibax/items/7fd9a874cb7e88f95488

https://superuser.com/questions/1048091/can-i-install-ec2-amazon-linux-os-locally-on-virtual-machine


#### Contributors

[Paul O'Flynn](https://github.com/poflynn) Original Author

[Sam Anthony](https://github.com/expertcoder)
