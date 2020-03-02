---
layout: post
title: Hack The Box - OpenAdmin
---
My write-up / walkthrough for OpenAdmin from Hack The Box.

![_config.yml]({{ site.baseurl }}/images/HTB/boxes/OpenAdmin/OpenAdmin.png)

# Walkthrough
Hey guys, today *OpenAdmin* retired and here’s my write-up about it. It’s an easy rated Linux box and its ip is *10.10.10.171*, I added it to */etc/hosts* as *openadmin.htb*. Let’s jump right in!

## Recon
First thing I always do is firing up my nmap and start to gather some info about the open ports and their services.

![_config.yml]({{ site.baseurl }}/images/HTB/boxes/OpenAdmin/nmap.png)

We got *ssh* on port 22 and *http* on port 80.

***
## Web Enumeration

When I visitd *openadmin.htb*, It showed me *Apache2 Ubuntu Default page*
![_config.yml]({{ site.baseurl }}/images/HTB/boxes/OpenAdmin/apache2_ubuntu.png)

So I started to enumerate the domain by firing up my *dirbuster*.
![_config.yml]({{ site.baseurl }}/images/HTB/boxes/OpenAdmin/dirbuster.png)

During enumerating the directories, I found an intersting directory -> */ona*, So I fired up my firfox and visited it and it showed me that:
![_config.yml]({{ site.baseurl }}/images/HTB/boxes/OpenAdmin/dirbuster.png)

The first thing I intersted about was the phrase *You are NOT on the latest release version
Your version    = v18.1.1*, So I Searhced about *OpenNetAdmin v18.1.1 exploitation* because it was outdated, maybe it had a vulnerability in the previous version and could I exploit it.

And Yeah! I found an exploitation on [Exploit-db](https://www.exploit-db.com/exploits/47691) and it was a *RCE* and the security researcher that found that vulnerability wrote a bash script to exploit it.
![_config.yml]({{ site.baseurl }}/images/HTB/boxes/OpenAdmin/ona_bash.png)

I read that bash script and understanded how to use it, So I fired up my terminal and run:
*bash ona_rce.sh openadmin.htb/ona/*

I got a shell and first thing I did was the command *ls* to list all the directories:
![_config.yml]({{ site.baseurl }}/images/HTB/boxes/OpenAdmin/ona_rce.png)

After a while of listing recursively all the directories I found the database settings at */opt/ona/www
/local/config* directory and the file was *database_settings.inc.php*, So I Started to read the file and Voila! the datbase password was in it:
![_config.yml]({{ site.baseurl }}/images/HTB/boxes/OpenAdmin/db_password.png)

I got the database password, But I didn't know the usernames were logged in the system, So the first thing I did was to identify the users logged in with the command *w*, after that I found a two users were logged in and an some intersting parts we will come to it later.
![_config.yml]({{ site.baseurl }}/images/HTB/boxes/OpenAdmin/logged_users.png)

I remembered that I got a *ssh* port open during *reconn*, So I tried to login with *jimmy* (one of the users I found) with db_password via the *ssh* services and Voila! I logged in to the server. 

I was thinking for 10 minutes what should I do in the next step, So I remembered the the intersting part that i told you before, If you see the image above again, You will see that jimmy was doing *cURL* in some dirs in *127.0.0.1*, So I tried to visit the web server direcotry and see what was there..
![_config.yml]({{ site.baseurl }}/images/HTB/boxes/OpenAdmin/web_server.png)

I found an intersting directory called internal, So I listed it and found 3 *PHP* files, The first file I read was main.php, because *jimmy* was *cURLing* that file.

It showed:
![_config.yml]({{ site.baseurl }}/images/HTB/boxes/OpenAdmin/main_php.png)

Hmmmm, that's very intersting, I saw again what jimmy was doing and it showed that he was *cURLing* a specific port, when I *cURL* that server with that domain, BOOM It showed me a *RSA* private key.

![_config.yml]({{ site.baseurl }}/images/HTB/boxes/OpenAdmin/rsa_pk.png)
