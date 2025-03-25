# Tutorial-Easy-customizable-Apache-server-setup-without-email
Tutorial of how to do a setup of a Ubuntu Linux VPS server with apache with or without domains, but without email. Tutorial offers customizable options depending on your use case. For beginners 

This tutorial has customizable options depending on what level of security you want on your server. You therefore have choices about which sections you wish to implement. This will all be made clear during the tutorial. 

This tutorial was done on Linux VPS servers using Ubuntu 22.04 and 24.04 with all the options. It was also done on Debian 12 without domain names.

**NOTE:** if you want a fully featured web server with email, I already have a separate tutorial on that at [https://github.com/verachell/tutorial-setup-linux-VPS-server-with-apache](https://github.com/verachell/tutorial-setup-linux-VPS-server-with-apache), but please be aware it takes a LOT more time to do (because of nameservers and email). I wouldn't recommend running your own email server as it is a lot of hassle if things go wrong. Also, be aware that not all hosting providers will allow you to open the ports necessary for email anyway. It's much easier instead to outsource email separately by using a third party email provider and pointing your domain's mail records to it. Alternatively, you can always direct users to send mail to an existing email account you already own.

## What and who is this tutorial designed for?
This beginner tutorial is suitable for use with a Linux VPS server without setting up email. There are 2 main use cases this is designed for:

**1. If you just want to do a quick tryout for a few hours super-cheap,** without actually requiring a domain and without having to spend ongoing amounts, I recommend an hourly computing provider such as DigitalOcean (US) or Clouding.io (EU). A cheap 1G RAM server with 10G disk space is all you need. Then follow the instructions in this tutorial. Afterwards, destroy (not suspend or stop, you need to actually destroy) your server to avoid being charged any further.

or

**2. If you want to set up a longer-term Linux VPS server** e.g. to create ongoing website(s) with zero, one or more domains, go for a cheap linux VPS provider. There are several providers which offer 1G RAM at or under $5 per month, which likely will work out cheaper long term than DigitalOcean - although not necessarily cheaper than Clouding.io. 

## Step 0: Getting started
**For the most part, everything in this tutorial is the same for both use cases.** Whenever there are differences between 1. the quick tryout at an hourly computing provider and 2. the traditional long term VPS server, **I will indicate what to do in which situation.**

In reality, your use case may fall somewhere in a continuum between these 2 stereotypical cases - for example, you might want to do a quick tryout with a domain at an hourly computing provider. Or run a long-term website on a traditional VPS host - but without a domain. Both types of hosts can handle any of the use cases. I'm just going for both ends of the continuum of what most people might typically want to do, instead of every possibility in between. **The tutorial is written in a way that you will easily be able to know what to do for your use case if it falls somewhere in between.**

Your server will need to have 1G RAM and 10G disk space minimum, unless your requirements are higher. Just go with the cheapest set of options in your hosting provider which still give you those minimums.

## Step 1: Setting up and logging in
**Goal:** log into your server

When you first set up at your provider, **pick the option to set up your authentication via a root password** as opposed to SSH keys. We will add SSH keys later. **It is assumed you know the root password and the IP address.**

_Tips: For DigitalOcean and Clouding.io, when you select your options, either provider will let you specify your own root password. Once you create your server, it will tell you the IP address. For a traditional VPS server, the signup process will differ between providers but all of them will either allow you to specify your root password, or if not, they will provide you with your initial root password. They will also tell you the IP address._

Assuming you have your root password and your server's IP address, open a terminal on your computer and type

`ssh root@your.server.ip.address`

it will ask you about fingerprint, say yes, then it will prompt for the root password. Once logged in, you are ready to proceed with setting up your server.

_Tip: for DigitalOcean, while you could open a terminal and do ssh as above, you have the option to not bother with that. Instead in your droplets list in dashboard, go in the right hand column of that droplet, then select access console, then "Launch droplet console". This automatically logs you in to a terminal window._

## Step 2: Open the ports you need
**Goal:** open ports you'll later need and optionally activate the firewall

**NOTE:** Unlike traditional long-term VPS servers, DigitalOcean and Clouding.io each have firewall-related things to be aware of. In DigitalOcean, the activation of your firewall will likely interfere with the console window that you launch from within the DigitalOcean dashboard. So on DigitalOcean, if you choose to implement the firewall you may like to avoid console usage and simply ssh in via a terminal window as you would for other servers. 
Clouding.io takes a different approach - they have a pre-set firewall in your dashboard which allows you to specify ports that you want open in those settings. Their basic default firewall has reasonable settings; you'll likely only need to add port 10000 to it for now (unless you need to open other ports later) and skip the commands below.

Now that you're logged in, you need to ensure the ports you need will be open. Type the following series of commands into your server to open these ports. 

**If you are just doing a quick tryout that you will delete in a short timeframe, you can skip this step.** In either case, if on Clouding.io, you will still need to add port 10000 to their default firewall on your dashboard, see note above.

_Tip: if your server does not recognize the ufw command, install it with `apt install ufw`_
```
ufw allow 10000
ufw allow 22
ufw allow 80
ufw allow 443
ufw allow 53/tcp
ufw allow 53/udp
ufw enable
```

## Step 3: Install Webmin
**Goal:** install the Webmin control panel and log into your server on it

The Webmin control panel, which is written entirely in Perl, provides convenient GUI access to managing your VPS server. It allows you to proceed a lot faster than if you were doing everything via the command line.

Still in the server command line, install Webmin - it's very easy and only has a few lines to enter. At the time of writing, these commands were as shown below, but you may prefer to use the info on the official Webmin documentation at https://webmin.com/download/ since the info could change over time.
```
curl -o webmin-setup-repos.sh https://raw.githubusercontent.com/webmin/webmin/master/webmin-setup-repos.sh
sh webmin-setup-repos.sh
apt-get install webmin --install-recommends
```
If your server does not recognize the `curl` command, install it first with `apt install curl`

### Log into your server on Webmin
Next open a browser window and go to `http://your.server.ip.address:10000`

(make sure to use http not https)

You should arrive at the Webmin login page for your server. The browser will give a warning that there is no ssl, but accept the risk and continue. Webmin uses a self-signed certificate to begin with, so information is at least encrypted and not in clear text. 

_Tip: Ensure your browser settings do not force https, and ensure you begin entering the address with http:// (not https)_

_Tip: If you plan to use a domain, we will add SSL to it and have Webmin use that certificate so you won't be bothered by that pesky warning. But if you don't use a domain, that's ok, you'll just encounter the warning._

At the login screen, log in with username root and the root password you had at the start.

**Optional but recommended:** At this point, you will likely have a notification in Webmin that you have some (or a lot) of packages to update. This could take 5 mins or so. Even if you plan on deleting the instance within 24h, you probably want to still update, since you don't want for some stuff to be out of date - even if not for security reasons, at least for compatibility reasons.

If you chose to update, you might then get a notice in Webmin that you should reboot server. If so, for simplicity, go ahead and do it from inside Webmin (there will be a reboot now button - don't close the window, just hit button and wait - typically takes about 1 - 2 mins). NOT from inside your terminal window, although you could do it that way, but just keep it simple and stick inside Webmin.

Assuming you're done with the update (or you didn't do it) move along to next step.

## Step 4: Install Apache
**Goal:** install the Apache web server on your VPS using Webmin and see the default placeholder page

In the Webmin menu on the left, go to Unused Modules, click on Apache webserver, and hit install - you'll need to approve the installation of several packages, that's fine. Hit Install Now. Then on Webmin menu click on Refresh Modules.

To check the install was successful, open a new browser window and navigate to the ip address of your server via http (not https). You should see the default apache page. This is good. 

## Step 5: Add a non-privileged user
**Goal:** For security reasons, add a non-privileged user. You will also add this user to Webmin

You shouldn't be doing everything as root - in fact, you should do as little as possible as root. It takes next to no time to add a safe user. In the server terminal window, do this, replacing newuser with your desired username - ideally something hard to guess:

`adduser newuser `

Follow prompts to add user password. Then, add user to sudo:

`usermod -aG sudo newuser`

again, replacing newuser with the username you made.

_Tip: Now that you have webmin, you don't have to use ssh on a terminal window to use the command line on your server. If you prefer, you can use the terminal built into Webmin. You can access it on the Webmin menu, under Tools -> Terminal._

### Add the non-privileged user to Webmin
Go to the Webmin menu, click on Webmin -> Webmin Users (do NOT go to System -> Users)

On the Webmin Users window, click on the tab "Create a new safe user"

For username, type in the name of your existing non-privileged user e.g. newuser (don't worry, this will not overwrite any of your user's existing files)

Password: choose "set to" and type in a password for your user to use with Webmin. If this is for a long-term website, this password should be different to the user's regular password, because it can otherwise get confusing when you change passwords. For a quick tryout it's OK to keep both passwords the same.

Then scroll down a bit to "Available Webmin Modules" and pick what your user should have access to. I recommend clicking Select All toget everything. At the very least, you'll definitely want File Manager, Terminal, upload and download. But I recommend everything. If you change your mind later you can always edit this later on.

Now click create.

### Log into Webmin as the new user
Then log out of Webmin and check you can log in again to Webmin `http://your.server.ip.address:10000` but as your new user with the new Webmin password.

_Tip: Remember, your new user only uses the webmin password on the webmin login screen. On any terminal window (even if doing a sudo command inside the built-in Webmin terminal window) use the original password for the new user._

## Step 6 (optional): protect your server from brute force attacks
You can omit this step if you're doing a quick tryout of less than 24 hours and not using any sensitive information.

If you're setting up a long term website, you definitely need to do this.

See separate sub-page on this repository at https://github.com/verachell/Tutorial-Easy-customizable-Apache-server-setup-without-email/blob/main/Protect-from-brute-force-attacks.md and come back here after.

## Step 7: set up the web root and display a static html test file

**Goal:** serve a static html page as a placeholder for your first site

We will set up the web root for your sites. We will have different folders for different sites. Here we will start with your first site, and later you will add others.

As the non-privileged user, go in your home directory e.g. via Webmin menu -> Tools -> Terminal (if you are not sure of your home directory, type cd in the terminal with no arguments and it will put you in your home dir). Then make a directory public_html

`mkdir public_html`

go into that directory

`cd public_html`

Now we will make the document root for your first site - this will be a directory. You need to pick a name for it. If you will not be using a domain name, call it anything you want e.g. firstsite. If you will be using a domain name (even though your domain is not yet connected) then call it your domain name minus the extension. e.g. for `example.com` you will use `example` as the directory name.

Make that directory now, inside of public_html, e.g.

`mkdir example`

Then in Webmin (non-priv user), to to Webmin menu -> Tools -> File manager. Navigate to your document root (i.e. `example` or whatever you called it)

Once inside that directory, create a new file called `index.html`. Put in some placeholder content e.g. `<h1>First site</h1>`

Now we need to connect this location to Apache.

Log out of Webmin as the non-priv user and log into it as root. Go to the Webmin menu -> Servers -> Apache

Now we will tell it where the document root is.

Go to Existing Virtual Hosts tab, then click on Virtual Server. That is the server for your first site. On that page, there is a field called Document Root. it says `/var/www/html` but you want to change that. So click to specify another directory and select your `/home/newuser/public_html/example` directory (replace newuser and example with your values).

Then click save. 

Then, going again to Virtual Server, click on the box saying "Edit directives"

Under DocumentRoot, you need to add the Directory section, unless those lines are already here. Replace newuser and example with your values.

```
DocumentRoot "/home/newuser/public_html/example"
<Directory "/home/newuser/public_html/example">
	Options +FollowSymLinks
	Require all granted
	AllowOverride All
</Directory>
```

Then in a terminal window, as the non-privileged user, type `cd` to get to your home directory, then `cd ..` then `ls -l`. If the perms of your user directory do not end with `r-x`, add execute permissions (Apache will need a full path of perms to be able to see your website files)

`chmod o+rx newuser`

Then restart apache:

`sudo systemctl restart apache2`

Now go to the browser and navigate to `http://your.server.ip.address` You should now see the result of the placeholder html you typed in earlier. If you do not, check that there are the same read and execute perms of o+rx on public_html and on the example directory: Apache needs a full path of permissions. Be sure to restart apache 2 as above (there is no harm in re-doing it). If problems still persist, firstly, go through the steps in this section and check you have done all the steps, and double check spelling - a misspelled directory name or a missed step in the Apache settings mentioned above can make all the difference and this has been the source of almost all my problems. It really should "just work".  If the problem persists, check Apache errors in the terminal window by doing `sudo cat /var/log/apache2/error.log | tail `. If that does not help, raise an issue on this repo stating what version of Ubuntu or Debian (there is no wrong answer - this helps me if I am trying to replicate the problem).

If you are not planning on using a domain, then you are done with the tutorial! You have set up a basic Apache server and served a static html file. What you choose to do next depends on your use case.

Otherwise, move on to find out how to connect one or more domains to Apache. You can serve up however many different sites according to the number of domains you connect.

## Step 8 (optional): connect your first domain name and set up SSL
Rest assured you do NOT need to connect a domain name. If you're doing a quick tryout, you can stop here. If you're unsure, you can skip this for now and come back to it at any time later. Here is how to know what to do:

You will want to connect domain name(s) if:

- you plan on serving more than one website on the server (covered here). Or
- you want to use an SSL certificate on your site from a trusted certificate authority (also covered). Or
- you want something more memorable than typing the IP address every time you want to view your site

See separate sub-page on this repository at https://github.com/verachell/Tutorial-Easy-customizable-Apache-server-setup-without-email/blob/main/Connect-first-domain-and-set-up-ssl.md and come back here after.

## Step 9 (optional): connect your second or subsequent domain and set up SSL
You may wish to set up more than one site. In that case, you will want to serve your other site at a different domain name, still at the same server. Apache is designed to be easily able to do that.

See the separate sub-page on this repository at https://github.com/verachell/Tutorial-Easy-customizable-Apache-server-setup-without-email/blob/main/connect-second-or-subsequent-domain-and-add-ssl.md 
