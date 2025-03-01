## 9. Setting up your second or subsequent domains and adding SSL
For each additional domain, you will want to set up a new document root just for that domain, and you will want SSL certificates for those domains too.

### Setting up a new document root for a new domain
Navigate (however you want to do it - Webmin or terminal) to the non-privileged user home directory, then into public_html. Then make a new directory and call it whatever your next domain is minus the extension (e.g. for `anotherdomain.com` just call the directory `anotherdomain`). Ensure it has o+x permissions if it does not already (it should already have that). Then go inside that new directory and create a new file called `index.html` and put some content in it that is different from your previous one(s) e.g. `<h1>Another site</h1>`

## Adding the new site onto Apache
First, you need to point your new domain to your server IP address. Log into your domain name registrar and go to the settings for your new domain. Point it to yor server exactly the same way you did for your first domain (e.g. by changing the A record as described previously)

Then wait a little while (~20 mins) for DNS changes to propagate. Then in Webmin as root user, go to Webmin menu -> Servers -> Apache Webserver. Then click on Create Virtual Host. 

- In the field "Handle connections to address", select Specific Address and put your server's IP address
- In Document Root, select the new directory you just set up
- in Port, put 80
- in Server Name, put your new domain e.g `anotherdomain.com`

Leave Add virtual sever to file in New file under virtual servers directory. No need to change anything else, then click on Create Now

Then go to that virtual host again, click on the newly created one, then click on Networking and Addresses, then under alternate server names put the www form of your domain e.g. `www.anotherdomain.com`

Then go to virtual host again, click on the newly created one, click on Edit Directives, and make sure it looks like this, of course replacing newuser and anotherdomain with your values: 
```
DocumentRoot "/home/newuser/public_html/anotherdomain"
<Directory "/home/newuser/public_html/anotherdomain">
	Options +FollowSymLinks
	Require all granted
	AllowOverride All
</Directory>

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined
```
or whatever you want for the logs if you want error and access logs to be unique for different domains. 

Click save

Then in terminal windo restart apache:

`sudo systemctl restart apache2`

then navigate to your new domain (http only) and it should come up with your new html that you putin the new document root.

## Adding SSL to your second or subsequent domain
in your terminal window, do
`sudo certbot --apache`

and when prompted which domains, select **just the new domain**

Then restart apache 

`sudo systemctl restart apache2`

As for your first domain, you may optionally wish to consider whether to expand out your SSL certificate to include the www version of your domain. If so, simply follow the instructions in that subsection for your new domain.

_Tip: If you accidentally issued a cert or need to delete your virtual server, remake the server and therefore issue a newcert, you can delete any of your ssl certs using `sudo certbot delete` and follow the prompts to specify for which domains to delete the cert_

Congratulations! You have now set up Apache to serve sites from multiple domains using different web roots. What you do next with the server is up to you.
