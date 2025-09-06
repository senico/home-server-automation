# Home Server Automation
Project goal is to automate the management and configuration of the home server:

    Simplified, IaC-based installation of Home Assistant OS, Ubuntu, or CentOS.
    Configuration of file servers: SMB, FTP.
    Automation of management tasks.

## Hardware:
Any Generic x86-64 UEFI PC
(Optional) 1 or 2 USB disks. Zigbee or Wifi USB adapter.
My setup: Running on Intel NUC with 8GB of memory, 120GB SSD + 500GB HDD, USB wireless adapter, USB disks.

## Host Installation:
Install the latest LTS version of Ubuntu on the home server.
Create a user (e.g., senico) and set a password.
Configure a static IP on the host (e.g., 192.168.1.11).
## Host configuration
To enable Ansible configuration, copy the public key from your laptop using:
(If not previously generated) ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
ssh-copy-id senico@192.168.1.11
(Optional) Edit sudoers to allow passwordless sudo commands:
senico ALL=(ALL) NOPASSWD: ALL

## Ansible project, on your laptop
(Optional) 0. In a new folder, create a virtual environment and install Ansible and required collections:
mkdir homeserver

create python virtual environment:
python3 -m venv venv
or attach to the existing one:
source venv/bin/activate

install ansible:
pip install ansible

1. clone the git repository:
git clone https://github.com/senico/home-server-automation.git
2. Edit the group_vars/all.yml file
set your timezone and network settings
3. edit host_vars/ files
Select the latest LTS cloud version and update image_url.
4. Use "blkid" to find the disk UUID and update disk_identifier.
Check "lsusb" to identify the USB wireless dongle and update usb_dongle_vendor and usb_dongle_product.

### Configure localhost
ansible-playbook playbooks/localhost.yml

### Install KVM:
ansible-playbook -i inventory.ini playbooks/kvm.yml

### Ubuntu installation:
there is ubuntu vm installation.
ansible-playbook -i inventory.ini playbooks/vm.yml -e "vm=ubuntu"

### Centos installation:
there is centos vm installation.
ansible-playbook -i inventory.ini playbooks/vm.yml -e "vm=centos"

### HAOS installation:
the HAOS image does not include Cloud-Init by default, but you can manually install it later on the VM 
ansible-playbook -i inventory.ini playbooks/vm.yml -e "vm=haos"

### Any OS installation
To add a new OS, copy a template file from the host_vars directory (e.g., myos).
Add an entry to the inventory.ini:
docker ansible_host=192.168.1.15 ansible_user=senico ansible_hostname=ubuntu
then run the playbook:
ansible-playbook -i inventory.ini playbooks/vm.yml -e "vm=docker"

### Make consistent system settings for all systems
this playbook will
-- set the hostname
-- configure network and DNS
-- set the timezone
-- enable timesync
-- configure NTP
ansible-playbook -i inventory.ini playbooks/system.yml

#### Install SMB and FTP server
set SMB_PASSWORD as env variable
ansible-playbook -i inventory.ini playbooks/share.yml

## Troubleshooting
VM will take IP address from the DHCP server.

If you want to troubleshoot, run VM from the CLI on the host:
virt-install --name ubuntu --description "T-SHOOT" --os-variant=generic --ram=2048 --vcpus=2 --disk ubuntu_image.qcow2,bus=scsi,size=32 --controller type=scsi,model=virtio-scsi --import --graphics none --boot uefi

check parameters:
virsh dumpxml ...

check VM status:
virsh list --all

attach to the VM:
virsh console ...

remove the VM:
virsh destroy ... && virsh undefine ... --nvram