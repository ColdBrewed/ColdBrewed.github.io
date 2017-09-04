
# Kioptrix 3 

I'm back with my third walkthrough in the Kioptrix series. 
![](goat.gif)

- [Kioptrix 1 Walkthrough]()
- [Kioptrix 2 Walkthrough]()

## Setup
As usual, lets start by identifying our attacker and target IP addresses.
![](/images/kioptrix3/K3-1.png)

![](/images/kioptrix3/K3-2.png)

This vm has some special DNS setup instructions. DNS is the service that converts the web page name like google.com into the IP address. Once you know the IP address of your target VM, in this case 192.168.65.134, you must echo it into the /etc/hosts file with the page name "kioptrix3.com"

In your atacker VM run:
>echo "192.168.65.134 kioptrix3.com" >> /etc/hosts
## Recon
Let's take a closer look:

>nmap -sV 192.168.65.134
![](/images/kioptrix3/K3-3.png)


When we visit kioptrix3.com in our browser we find a blog.
![](/images/kioptrix3/K3-4.png)

When testing against a web app, it's a good idea to manually crawl the site and inspect each page. We start by reading the blog page and gain some helpful pieces of information. Our attention is directed to the /gallery page, where the company's new webapp is hosted. We also see that a new 13 year old wizard admin has been hired and his name is "loneferret"
![](/images/kioptrix3/K3-5.png)

We also inspect the login page and notice that is it powered by LotusCMS

![](/images/kioptrix3/K3-6.png)

Let's take a deeper look at the gallery app:
![](/images/kioptrix3/K3-7.png)


On the top of the page we can see the application's name: Gallarific. If you are dumb and unobservant like me, you will probably waste time scouring the page source for the webapp name.

Let's do a little research to see if any of these apps (Gallarific or LotusCMS) have public vulnerabilities. You can always do a Google search or check on exploit-db.com but I prefer to start with a quick commandline tool called searchsploit that runs an offline version of exploit-db, that way we wont have to do that annoying CAPCHA everytime

### Method 1: Gallarific Exploit

![](/images/kioptrix3/K3-8.png)
There are a couple of public exploits for the Gallarific gallery app. The one that looks most interesting.
![](/images/kioptrix3/K3-9.png)

For SQL injection on easy mode, let's use a tool called SQLmap to exploit this vulnerability.

The most simple SQLmap scan just identifies that the provided link is injectable and enumerates some information about the back end database, web server and OS.

Once an exploitable injection point is found, I like to take a look at the tables, to target which will be most useful.

>sqlmap -u http://kioptrix3.com/gallery/gallery.php?id=null --tables

We can see that the gallery database has a dev_accounts table.
![](/images/kioptrix3/K3-10.png)

Let's run the command again, but with the **--dump** flag and identify the gallery database and dev_accounts table. When asked if we'd like to run a dictionary attack against the discovered hashes, we choose yes:

![](/images/kioptrix3/K3-11.png)

We discover two users with weak passwords. One really common vulnerability is password reuse. Do you think that the 13 year old wizard dev, loneferret, reused his password on another part of the system? Maybe the ssh service?

![](/images/kioptrix3/K3-12.png)

Bingo! We have a low priv shell under the loneferret user account. If we look in loneferret's home folder, we find an interesting file called CompanyPolicy.README
![](/images/kioptrix3/K3-13.png)

I also ran `sudo -l` to see if loneferret could run any other privileged commands.
![](/images/kioptrix3/K3-14.png)

I ran into a small probelm here when trying to run the `sudo ht` command. "Error opening ternminal: xterm-256color

I found the solution [here](https://stackoverflow.com/questions/6804208/nano-error-error-opening-terminal-xterm-256color) on stackoverflow. 
`export TERM=xterm`

Now the ht editor opens
![](/images/kioptrix3/K3-15.png)
I chose to edit the /etc/sudoers file to give loneferret root privileges.
Then it's just a matter of spawning a root shell.
![](/images/kioptrix3/K3-16.png)

That's one way to get root. How about that other service? LotusCMS.



### Method 2: LotusCMS Exploit

![](/images/kioptrix3/K3-17.png)
LotusCMS has a remote command execution exploit in the metasploit framework. If this is your first time using metasploit, [start here](https://docs.kali.org/general-use/starting-metasploit-framework-in-kali).


We open the metasploit framework by running `service postgresql start` to start the backend database. And then run `mfconsole`

![](/images/kioptrix3/K3-18.png)

When we search for lotusCMS one exploit surfaces. There are only a few compatible payloads, all php based, you can choose between a regular command shell or a meterpreter shell. On linux hosts I generally prefer the command shell, but it's pretty trivial to switch between the two.

![](/images/kioptrix3/K3-19.png)

I haven't found a good way to do privesc on this approach, without doing some workarounds to replicate the previous loneferret privesc method. The usual suspects (common kernel exploits, metasploit post modules) are not landing. Let me know if you have some ideas. 


