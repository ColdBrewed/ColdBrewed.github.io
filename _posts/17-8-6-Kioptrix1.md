
Noisebridge has a weekly infosec lab night on Thursdays at 6:30pm. Our first lab went over Kioptrix 1. Below is a walkthrough to guide you from setup to a root shell. 
<img src="/images/Noisebridge-logo.png" width="100px">

### Setting up the VM: 

First, if you don't have it already you'll need virtualization software. There are two main free options for hobbyists, VirtualBox and VMware. Use whichever one you're more comfortable with, but in terms of Kioptrix specifically VMware is simpler to set up. So once you have some virtualization software installed you need to set up some guest VMs. Download the latest version of [kali linux](https://www.offensive-security.com/kali-linux-vmware-virtualbox-image-download/) (they offer images specifically for VirtualBox and VMware too), and be sure the check the hash of your download against the one published by Offensive Security. Next you're going to go to Vulnhub.com and download some vulnerable machines to practice hacking against. Today we're working on [Kioptrix #1](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/), not to be confused with Kioptrix 1.1. 

When adding your VMs to your virtualization software you must choose a networking mode. There are a couple different ones, depending on your needs. Most vulnerable VM sites will tell you to segregate a vulnerable VM on your network. Host only mode is a good option for this. It creates a mini network between your host os and the guests running on it. However, the guests will not have access to the internet. NAT is another option that creates a network on your host machine for your guests, but it does allow access to the internet. Finally bridged mode, uses the host machine's IP information to connect to the network but is not ideal when you are hosting both your target and attacker. For today's exercise I've chosen NAT network for both my attacker and my host, since one of the methods we'll try requires the target to have internet access. 

#### Troubleshooting

The first step in this lab is to fire up both your attacker VM (kali) and your target VM (kioptrix). Virtualbox users, if you get a kernel panic error when firing up Kioptrix, go to your VM settings under Storage make sure that your storage type is IDE and not SATA. Also, some virtualbox users had trouble getting their Kioptrix VM to get an IP address from DHCP. Go back to your VM settings and under Network>Advanced change your adapter type from Intel PRO/1000 to PCnet PCI II. These are the two most common errors encountered by people in the lab.

### Let's Start Hacking!

#### Step 1: Enumeration

First you need to identify both your IP address and the IP of the target machine. Start by running an **ifconfig** and noting down your IP address. 

Next, discover the IP address of your target machine. I have both my attacker and target set to NAT mode I started by looking up the assigned MAC address in the VMWare settings and comparing it to the results of my nmap Ping scan.
Let's use nmap to do a ping sweep on our local network to identify the target IP.

**nmap -sP 192.168.189.0/24**

![](/images/kioptrix1/kioptrix1-1.png)

In my case, the target system is at 192.168.189.128. To check this you can enter the ip address into the browser of your attacking box and you should see: 

![](/images/kioptrix1/kioptrix1-2.png)


Let's start with a syn scan to see the open ports. This is a less intrusive scan that doesn't run as high of a risk of knocking down fragile services or closing ports.

**nmap -sS 192.168.189.128**

![](/images/kioptrix1/kioptrix1-3.png)

So there are several running services including SSH, HTTP/S, SMB, and RPC. 

SMB, or Server Message Block, is a service that has had many many security problems over the years. When you encounter open SMB ports (139 or 443) it's a good idea to investigate further. Let's use a tool called enum4linux to find out more about the SMB service running.

**enum4linux 192.168.189.128**

This will spit out a TON of information but the most useful bit is under the Share Enumeration section.
![](/images/kioptrix1/kioptrix1-5.png)

Here we can see that the Samba version is 2.2.1a
Now it's time to do some research and see if this version of samba has any vulnerabilities and public exploits. One way is to visit (exploit-db.com)[https:/exploit-db.com/search], a great database of public exploits made by the folks at Offensive Security. I personally hate CAPTCHAs so I avoid the web client like the plague. By using the command line tool **searchsploit** we can quickly search an offline version of the exploit-db.

![](/images/kioptrix1/kioptrix1-6.png)

So I did a searchsploit search for Samba 2.2 and got some results! Let's try out the Remote Command Execution exploit for Samba 2.2.8

Start by locating the exploit in your Kali system (**locate /linux/remote/10.c**) and then copying that exploit into your filepath (**cp /usr/share/exploitdb/platforms/linux/remote/10.c kioptrix1.c**). By reading the exploit we can determine how to run and compile it. In this case compiling is easy (**gcc -o kioptrix1 kioptrix1.c**). Also we see the syntax for how to run the exploit (**./kioptrix1 -b 0 -v 192.168.189.128**). Fire away!

![](/images/kioptrix1/kioptrix1-7.png)

Congratulations! You just got a root shell.

That was pretty easy, huh? Can you find other methods to exploit this machine? We could also use metasploit to exploit Samba with the trans2open method.

### Metasploit Method

Fire up metasploit and do a quick search on trans2open (**search trans2open**), which was one of the methods that we saw listed when we did a searchsploit on Samba 2.2
![](/images/kioptrix/kioptrix1-8.png)

Since we're targeting a linux system lets use the following exploit.
**use exploit/linux/samba/trans2open**

If you're new to metasploit you can use commands like **show info** or **show options** to get more information about the module. We set the target system with **set RHOSTS {target IP}**. The standard payload for this module is a staged meterpreter, but it can be very unstable against kioptrix. I had the most success with setting a simple payload like **linux/x86/shell_reverse_tcp**. After you have set all the required settings show under show options, type **exploit** to run the module.

![](/images/kioptrix1/kioptrix1-9.png)

We just got another root shell!

Next week we'll be moving on to Kioptrix 2 (AKA Kioptrix 1.1).

Check back for more walkthroughs.






