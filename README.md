# Tutorial-Easy-customizable-Apache-server-setup-on-Ubuntu-Linux-VPS-without-email
Tutorial of how to do a setup of a Ubuntu Linux VPS server with apache with or without domains, but WITHOUT email. Tutorial offers customizable options depending on your use case. For beginners 

This tutorial has customizable options depending on what level of security you want on your server. You therefore have choices about which sections you wish to implement. This will all be made clear during the tutorial. 

This tutorial was done on Linux VPS servers using Ubuntu 22.04 and 24.04

## What and who is this tutorial designed for?
This beginner tutorial is suitable for use with a Linux VPS server without setting up email. There are 2 main use cases this is designed for:

**1. If you just want to do a quick tryout for a few hours super-cheap,** without actually requiring a domain and without having to spend ongoing amounts, I recommend an hourly computing provider such as DigitalOcean (US) or Clouding.io (EU) where you create a droplet. A cheap 1G RAM droplet with 10G disk space is all you need. Then follow the instructions in this tutorial. Afterwards, destroy (not suspend or stop, you need to actually destroy) your server to avoid being charged any further.

or

**2. If you want to set up a longer-term Linux VPS server** e.g. to create ongoing website(s) with zero, one or more domains, go for a cheap linux VPS provider. There are several providers which offer 1G RAM at or under $5 per month, which likely will work out cheaper long term than DigitalOcean - although not necessarily cheaper than Clouding.io. 

**NOTE:** if you want a fully featured web server with email, I already have a separate tutorial on that at [https://github.com/verachell/tutorial-setup-linux-VPS-server-with-apache](https://github.com/verachell/tutorial-setup-linux-VPS-server-with-apache), but please be aware it takes a LOT more time to do. I wouldn't recommend running your own email server as it is a lot of hassle if things go wrong. Also, be aware that not all hosting providers will allow you to open the ports necessary for email anyway. It's much easier instead to outsource email separately by using a third party email provider and pointing your domain's mail records to it. Alternatively, you can always direct users to send mail to an existing email account you already own.

## Step 0: Getting started
**For the most part, everything in this tutorial is the same for both use cases.** Whenever there are differences between 1. the quick tryout at an hourly computing provider and 2. the traditional long term VPS server, **I will indicate what to do in which situation. **

In reality, your use case may fall somewhere in a continuum between these 2 stereotypical cases - for example, you might want to do a quick tryout with a domain at an hourly computing provider. Or run a long-term website on a traditional VPS host - but without a domain. Both types of hosts can handle any of the use cases. I'm just going for both ends of the continuum of what most people might typically want to do, instead of every possibility in between. **The tutorial is written in a way that you will easily be able to know what to do for your use case if it falls somewhere in between.**

Your server will need to have 1G RAM and 10G disk space minimum, unless your requirements are higher. Just go with the cheapest set of options in your hosting provider which still give you those minimums.

## Step 1: Setting up and logging in
**Goal:** log into your server

When you first set up at your provider, **pick the option to set up your authentication via a root password** as opposed to SSH keys. We will add SSH keys later. **It is assumed you know the root password and the IP address.**

_Tips: For DigitalOcean and Clouding.io, when you select your options, either provider will let you specify your own root password. Once you create your server, it will tell you the IP address. For a traditional VPS server, the signup process will differ between providers but all of them will either allow you to specify your root password, or if not, they will provide you with your initial root password. They will also tell you the IP address._

Assuming you have your root password and your server's IP address, open a terminal on your computer and type

`ssh root@your.server.ip.address`

Note that these instructions may differ slightly if your local machine is Mac or Windows, so make sure you know how to do ssh from your local machine. For Mac, see https://osxdaily.com/2017/04/28/howto-ssh-client-mac/ and for Windows see https://learn.microsoft.com/en-us/windows/terminal/tutorials/ssh

it will ask you about fingerprint, say yest, then it will prompt for the root password. Once logged in, you are ready to proceed with setting up your server.

_Tip: for DigitalOcean, while you could open a terminal and do ssh as above, you have the option to not bother with that. Instead in your droplets list in dashboard, go in the right hand column of that droplet, then select access console, then "Launch droplet console". This automatically logs you in to a terminal window._

## Step 2: Open the ports you need
**Goal:** open ports you'll later need and optionally activate the firewall

Now that you're logged in, you need to ensure the ports you need will be open. Type the following series of commands into your server to open these ports. 

If you are just doing a quick tryout that you will delete in a short timeframe, you can skip this step. In either case, if on Clouding.io, you will still need to add port 10000 to their default firewall on your dashboard, see note below.

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
**NOTE:** Unlike traditional long-term VPS servers, DigitalOcean and Clouding.io each have things to be aware of. In DigitalOcean, the activation of your firewall will likely interfere with the console window that you launch from within the DigitalOcean dashboard. So on DigitalOcean, if you choose to implement the firewall you may like to avoid console usage and simply ssh in via a terminal window as you would for other servers. 
Clouding.io takes a different approach - they have a pre-set firewall in your dashboard which allows you to specify ports that you want open in those settings. Their basic default firewall has reasonable settings; you'll likely only need to add port 10000 to it for now (unless you need to open other ports later)

## Step 3: Install Webmin
**Goal:** install the Webmin control panel and log into your server on it

The Webmin control panel, which is written entirely in Perl, provides convenient GUI access to managing your VPS server. It allows you to proceed a lot faster than if you were doing everything via the command line.

Still in the server command line, install Webmin as described on the official Webmin documentation at https://webmin.com/download/ - it's very easy and only has a few lines to enter. At the time of writing, these commands were as shown below, but use the commands on the link mentioned above since the info could change.
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
