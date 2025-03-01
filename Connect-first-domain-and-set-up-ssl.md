## 8. Connecting your first domain and setting up SSL
We will do a quick and easy connect of domain to server via the A record at the domain name registrar. You won’t need to set up nameservers for this. 

### Change the A record of the domain at the registrar
To use the A record, your domain must not be using any nameservers besides the registrar’s defaults. So at your domain name registrar, set your domain’s nameservers to registrar’s defaults (otherwise changes to A record will not work).

Then still in your registrar account, go to the DNS settings for that domain. Change the existing A record so that hostname is either blank or has only the @ symbol. Then in address, enter your server’s ip address. If there is an additional A record, edit that (if no additional one, create a new one). This time with hostname as www and in address put same server IP address again. Delete any additional A records. And delete any AAA records.

You will need to wait around 20 mins (sometimes as long as 2 days but usually just 20mins) to let changes propagate.

### Configure Apache to use your domain
Now you’ve changed it at your registrar, but you still need to let your Apache server know what domain to expect. To do that, log into webmin as root, then click on Webmin menu -> Servers -> Apache webserver. Click on Virtual Server, and in the virtual server section,  on address section type your server IP address. On the same page, under Server Name, instead of default, type your domain name e.g. example.com. Then click save.

If it complains that something is not valid, wait 5 to 10 mins and make each change separately, then save. It should be after DNS changes have propagated. Also, be certain that your firewall has port 53 open, which we did on one of the first steps but double check it with `sudo ufw status verbose` (your DNS records use port 53).

Then, in the virtual server again, click on on Networking and addresses, then under alternate server names put the www form of your domain e.g. `www.example.com`

You also want your internal server name to match your main domain name - this doesn't affect too much but if you don't do this step, it may cause issues when you later try to connect to Webmin under SSL.

So, in the Webmin menu, go to Networking -> Network configuration -> Hostname and DNS client, then leave everything the same but change hostname from whatever your web host gave you (it will be something meaningless like vps_central_34453), to your main domain name e.g. `example.com`

Then restart apache by going in Webmin menu -> Tools ➜ Terminal and then typing
`systemctl restart apache2`

Now to check everything works type your domain name into the browser (with http not https since you do not have SSL yet). Do it with the regular and the www version.  You should see the same apache default page you saw earlier when you navigated to the IP address. If you do not see it, you may need to wait a bit, because it can take some time for DNS changes to propagate. In reality I have never had it take more than 20 mins. If you still do not see it after 20 mins, 1. use a DNS record checker to see if the A record has been changed. 2. double check the spellings of your domain in your Apache settings in Webmin. 3. Restart apache. 4. Ensure you are connecting by http, not https- you may need to temporarily change the settings on your browser to have it not force http  to https.

### Setting up SSL for your first domain
If you have one or more domains, you really should be using SSL. Fortunately, it's free and easy to set up using Certbot, and best of all they auto-renew. 

In your ssh terminal window, install snap:

`sudo apt install snapd`

Then we will install certbot. The full certbot documentation is at https://certbot.eff.org/instructions but at the time of writing, you need to do these steps:

Install certbot:
```
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

Now generate and auto-install your certificates for Apache like this:

`sudo certbot --apache`

It asks you interactively what domains you want to activate for. You'll want to accept.

Done, and done. Restart Apache:

`sudo systemctl restart apache2`

Your domain should be available in https now - check this on a browser e.g. `https://example.com`

Your certificates are in `/etc/letsencrypt/live`

If you ever need to check the status of your certifcates, do 
`sudo certbot certificates`

You will see on Webmin on the Apache server virtual host list that certbot automatically created a new virtual host at port 443 for you for each domain you got certificates for. That's normal and is what you want.

### (optional): Expanding out the SSL certificate to include the wwww version of the site
This subsection is optional depending what you plan to do with your server later. The certbot certificate you installed does not protect the www version of the site e.g. `www.example.com` but you can get around that later by redirecting wwww to non-wwww in your .htaccess file of your website. This is outside the scope of this tutorial but that information is easy to find online and involves making a .htaccess file inside your document root and adding a few lines of code to that.

However, it is safer to add SSL to the www version of the site. If you wish to do that, you do not need to request a new certificate, you can just expand the existing one as follows. Run the command first with the `--dryrun` option, it will inform you if there are any issues without actually making any changes:

`sudo certbot certonly --dry-run --expand -d example.com,www.example.com`

The format of listing the domains is to list the already covered domains first, followed by the ones to expand to. No spaces between the domain names, just commas.

NOTE: It will give you the option of authenticating with 1. Apache Web server plugin, 2. Local HTTP server, 3. webroot path. Select option 1, Apache.

Assuming no errors were reported, you can repeat the command removing the `--dry-run` option. Then restart apache 

`sudo systemctl restart apache2`

Now open a browser and check that the www version of your domains are available in https, e.g. `https://www.example.com`

### (optional): Ensure Webmin uses your SSL certificate
For your first or main domain, you will likely want to allow Webmin to use your newly issued SSL certificate instead of the self-signed one it came with. This way you won't get a browser warning.

**Warning:** it can be all to easy to accidentally do this part wrong, and if so, it can become next to impossible to log into Webmin on a regular browser (speaking from experience). If that becomes the case, you can get back to your original settings from the terminal window ssh connection and edit the `/etc/webmin/miniserv.conf` file to its original settings and try again.

So before starting, download a copy locally of the original Webmin configuration file that is at `/etc/webmin/miniserv.conf`. At the very least, do `sudo cat /etc/webmin/miniserv.conf` to print it on your terminal. 

In Webmin, go to the Webmin menu -> Webmin configuration -> SSL encryption, and go to SSL settings tab. BEFORE DOING ANYTHING, make a note of current settings on this tab, or just take a screenshot. Now, set the following options in that Webmin window:
```
	Enable SSL?: Yes
	Private key file: /etc/letsencrypt/live/example.com/privkey.pem
	Certificate file: Separate file: /etc/letsencrypt/live/example.com/cert.pem
	Additional certificate files (for chained certificates): /etc/letsencrypt/live/example.com/chain.pem
```
Note that the additional cert field offers a file picker but that doesn't seem to work properly (at least on my version of Webmin), so just type in the filename manually in there.

Log out of Webmin. In your terminal connection, restart Webmin:
`sudo systemctl restart webmin.service`

You should now be able to log into Webmin securely at your primary domain name e.g. `https://example.com:10000`

