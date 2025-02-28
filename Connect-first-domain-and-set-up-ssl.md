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

Then restart apache by going in Webmin menu -> Tools ➜ terminal and then typing
`systemctl restart apache2`

Now to check everything works type your domain name into the browser (with http not https since you do not have SSL yet). Do it with the regular and the www version.  You should see the same apache default page you saw earlier when you navigated to the IP address. If you do not see it, you may need to wait a bit, because it can take some time for DNS changes to propagate. In reality I have never had it take more than 20 mins. If you still do not see it after 20 mins, 1. use a DNS record checker to see if the A record has been changed. 2. double check the spellings of your domain in your Apache settings in Webmin. 3. Restart apache. 4. Ensure you are connecting by http, not https- you may need to tempiorarily change the settings on your browser to have it not force http  to https.

### Setting up SSL for your first domain
If you have one or more domains, you really should be using SSL. Fortunately, it's free and easy to set up using Certbot. 

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

