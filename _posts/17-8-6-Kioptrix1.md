## Kioptrix 1 Walkthrough

Noisebridge has a weekly infosec lab night on Thursdays at 6:30pm. Our first lab went over Kioptrix 1. Below is a walkthrough to guide you from setup to a root shell. 

#### Setting up the VM: 

First, if you don't have it already you'll need virtualization software. There are two main free options for hobbyists, VirtualBox and VMware. Use whichever one you're more comfortable with, but in terms of Kioptrix specifically VMware is simpler to set up. So once you have some virtualization software installed you need to set up some guest VMs. Download the latest version of [kali linux](https://www.offensive-security.com/kali-linux-vmware-virtualbox-image-download/) (they offer images specifically for VirtualBox and VMware too), and be sure the check the hash of your download against the one published by Offensive Security. Next you're going to go to Vulnhub.com and download some vulnerable machines to practice hacking against. Today we're working on [Kiotrix #1](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/), not to be confused with Kioptrix 1.1. 

When adding your VMs to your virtualization software you must choose a networking mode. There are a couple different ones, depending on your needs. Most vulnerable VM sites will tell you to segregate a vulnerable VM on your network. Host only mode is a good option for this. It creates a mini network between your host os and the guests running on it. However, the guests will not have access to the internet. NAT is another option that creates a network on your host machine for your guests, but it does allow access to the internet. Finally bridged mode, uses the host machine's IP information to connect to the network but is not ideal when you are hosting both your target and attacker. For today's exercise I've chosen NAT network for both my attacker and my host, since one of the methods we'll try requires the target to have internet access. 

##### Troubleshooting

The first step in this lab is to fire up both your attacker VM (kali) and your target VM (kioptrix). Virtualbox users, if you get a kernel panic error when firing up Kioptrix, go to your VM settings under Storage make sure that your storage type is IDE and not SATA. Also, some virtualbox users had trouble getting their Kioptrix VM to get an IP address from DHCP. Go back to your VM settings and under Network>Advanced change your adapter type from Intel PRO/1000 to PCnet PCI II. These are the two most common errors encountered by people in the lab.

#### Let's Start Hacking!

##### Step 1: Enumeration

First you need to identify both your IP address and the IP of the target machine. Start by running an **ifconfig** and noting down your IP address. 

Next, discover the IP address of your target machine. I have both my attacker and target set to NAT mode I started by looking up the assigned MAC address in the VMWare settings and comparing it to the results of my nmap Ping scan.
Let's use nmap to do a ping sweep on our local network to identify the target IP.
**nmap -sP 192.168.189.0/24**
[](/images/kioptrix1/kioptrix1-1.png)

In my case, the target system is at 192.168.189.128. To check this you can enter the ip address into the browser of your attacking bo and you should see: 
[](/images/kioptrix1/kioptrix1-2.png)


Let's start with a syn scan to see the open ports. This is a less intrusive scan that doesn't run as high of a risk of knocking down fragile services or closing ports.
**nmap -sS 192.168.189.128**
[](/images/kioptrix1/kioptrix1-3.png]




