OpenStack Installation Using Packstack on UCS Server

This guide outlines the steps to deploy OpenStack using Packstack on a UCS server or equivalent hardware.

**1. Prepare the Hardware**
	1.	Acquire Hardware
Ensure UCS-C or equivalent servers are equipped with adequate storage drives and meet OpenStack’s minimum hardware requirements.
	2.	Add Virtual Media for CentOS ISO
	•	Mount the CentOS ISO image as virtual media on the server.
	3.	Create Virtual Drives
	•	Configure the RAID levels and create virtual drives according to your storage needs.
	4.	Load CentOS ISO and Reboot
	•	Map the ISO to the server’s virtual CD/DVD drive.
	•	Reboot the server to boot from the ISO.

**2. Install and Configure CentOS**
	1.	Install CentOS
	•	Follow the CentOS installation wizard.
	•	Use a minimal or standard installation as per your requirement.
	•	Configure the partition scheme, network settings, and root password.
	2.	Initial Network Setup
	•	After installation, log in and configure the network.

**3. Pre-Configuration for All Servers**

3.1 Set Hostnames
	•	(Optional, depends on requirements) Set a unique hostname for each server.

hostnamectl set-hostname <hostname>
hostnamectl status

3.2 Assign IP with VLAN Tagging
	•	Configure network interfaces with tagged VLAN settings:

cd /etc/sysconfig/network-scripts
nano ifcfg-enp5s0  # Example interface file

Example Configuration (ifcfg-enp5s0.2140):
TYPE=Ethernet
BOOTPROTO=none
DEFROUTE=yes
IPADDR=10.5.131.128
PREFIX=24
GATEWAY=10.5.131.1
VLAN=yes
DNS1=10.0.1.1
NAME=enp5s0.2140
DEVICE=enp5s0.2140
ONBOOT=yes

3.3 Restart Networking

systemctl restart network
	•	If the above fails, stop and disable NetworkManager:

systemctl stop NetworkManager
systemctl disable NetworkManager
systemctl restart network

3.4 Test Network Connectivity
	•	Test connectivity to external hosts (e.g., Google DNS):

ping 8.8.8.8
ping google.com
ssh <remote-server>

**4. Install OpenStack Packstack**

4.1 Configure Repositories
	•	Enable the desired OpenStack release:

yum install centos-release-openstack-mitaka -y  # Mitaka release
yum install centos-release-openstack-newton -y  # Newton release

	•	Update the system and install Packstack:

yum update -y
yum install openstack-packstack -y

4.2 Configure Host Entries
	•	Add host entries for all servers in /etc/hosts:

nano /etc/hosts
Example:
10.5.131.107 dc13-tb2-ctrl
10.5.131.108 dc13-tb2-comp08

4.3 Check DNS Resolution
	•	Verify /etc/resolv.conf has proper DNS servers:

cat /etc/resolv.conf
4.4 Generate and Use Answer File
	•	Generate a default answer file:

packstack --gen-answer-file=<answer-file-name>

	•	Modify the answer file as required.
	•	Run the installation using the answer file:

packstack --answer-file=<answer-file-name>


**5. Post-Installation Steps**
	1.	Access OpenStack Dashboard
	•	Open the OpenStack dashboard in a browser:
http://<controller-ip>/dashboard/
	2.	Verify OpenStack Services
	•	Ensure all services are running:
systemctl status openstack-*
	3.	Check Network Bridges
	•	Verify if OVS bridges exist:
ovs-vsctl show
	•	If not present, create them:
ovs-vsctl add-br br-ex
ovs-vsctl add-port br-ex enp6s0
	4.	Upload an Image to Glance
	•	Upload a sample image to Glance:
glance image-create --name "CentOS-7.0-cloud" \
  --disk-format qcow2 --container-format bare \
  --file <image-file-path>

	5.	Create Network and Subnets
	•	Example commands:
neutron net-create vlan-2144 --provider:network_type vlan \
  --provider:segmentation_id 2144 --provider:physical_network physnet1
neutron subnet-create vlan-2144 10.5.135.0/24





![image](https://github.com/user-attachments/assets/75642f45-97fd-4618-a6d0-2f9ef56abaea)
