## 6. Protect your server from brute force attacks

This will be done using the command line via an SSH connection, not Webmin. 

If you still have your connection open to your server on the terminal, go to that. If not, re-connect using `ssh root@your.server.ip.address` and enter the root password when prompted.

### Change root password
Change the root password from the original one that was assigned to you. This part is optional, but recommended if your VPS server specified the original password for you.

`passwd`

and follow the prompts.

### Change the SSH port away from the default of 22
This tactic will stop a lot of brute force attacks

`nano /etc/ssh/sshd_config`

in the file, uncomment the port line and change the port from 22 to some other port not in use. So, avoid ports 53, 80, 443, 10000 and 22. For example, change to port 65000

in the same file, add a line saying

`Protocol 2`

then after saving the file, open that port

`ufw allow 65000`

if on Clouding.io, in your dashboard add that port into your firewall.

Then you'll need to restart ssh:

`systemctl restart ssh`

Now, **keeping your existing terminal open**, go to a new terminal and open a fresh connection to the server under your non-privileged user, specifying the new port number like this:

`ssh -p 65000 newuser@your.server.ip.address`

Check that this works and allows you to log in as the new user. 

Assuming that this works after entering the new user's password when prompted, you can now move on to logging in via authentication keys.
