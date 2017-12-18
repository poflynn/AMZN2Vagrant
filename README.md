# How to make a  VirtualBox-based Vagrant Amazon 2 Linux Base Box
Please excuse the sloppyish formatting. I did this quickly from memory. Feel free to submit improvements..

* Download .VDI file from [here](https://cdn.amazonlinux.com/os-images/2017.12.0.20171212.2/virtualbox/)

* Build virtual CD that contains cloud-init data using the files in this directory
###### Linux:
```bash
genisoimage -output seed.iso -volid cidata -joliet -rock user-data meta-data
```
###### Mac:
```bash
brew install cdrtools
mkisofs -output seed.iso -volid cidata -joliet -rock user-data meta-data
```

* Create a new VM in VirtualBox, call it "AMZN"
* Add the seed.iso file generated above to the virtual CD drive
* Add the .VDI file downloaded above as the hard drive (i.e. do not create a new one)
* Start the Amazon Linux 2 virtual machine

### Installing VirtualBox Guest Additions

Remove the cloud-init ISO image from VirtualBox settings

Insert the CD image of VirtualBox Guest Additions from the menu

Log in to the virtual machine as root/vagrant

From the VirtualBox menu select Devices->Insert Guest Additions CD Image
```bash
mount -r -t iso 9660 /dev/cdrom/media
cd /media
./VBoxLinuxAdditions.run
systemctl enable vboxadd.service
```
Ignore tainted kernel message

#### Post processing
```bash
#Delete Bash command history
export HISTSIZE = 0
#Delete cache of yum
yum clean all
rm -rf /var/cache/yum
#Optimize the area of ​​the virtual hard disk
dd if=/dev/zero of =/ZERO bs=1M
rm -f /ZERO
shutdown -h now
```

#### Create Base Box from virtual machine
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


