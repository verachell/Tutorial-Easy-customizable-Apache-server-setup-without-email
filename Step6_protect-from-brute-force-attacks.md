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

## Setting up SSH authentication keys for login
For security reasons, you don't want to continue authenticating via password every time you log in, whether as root or as non-privileged user. It leaves you too open to brute-force attacks. So we will set up SSH keys.

The setting up of authentication keys needs to be done on your local desktop or laptop machine, not on the remote server. The instructions I'm giving here are for a local machine that runs Linux or a Linux-like environment. If you have a Windows machine, you can get equivalent instructions for generating SSH keys at https://phoenixnap.com/kb/generate-ssh-key-windows-10. For Mac, instructions for generating the SSH keys can be found at https://www.macworld.com/article/671164/how-to-generate-ssh-keys.html

For a Linux local machine, you would do:

`ssh-keygen -N "securepassphrase" -t ed25519`

where "securepassphrase" is replaced with whatever passphrase you want. Keep it simple and easy to type, it doesn't have to be as strong as a password - the key will be doing the authenticating, the passphrase just gives an extra layer of protection. You don't have to include a passphrase at all if you'd rather not.

When asked for what file to save to, **don't accept the defaults** unless you're sure you have no other keys already in existence on your local machine. If you have a Github account, you probably already have your GitHub keys on your local machine. So **to avoid accidentally overwriting any existing keys**, when it asks you for the filename, **type in a filename that's meaningful to YOU** in your .ssh directory, e.g. `~/.ssh/myserver_ed25519`

If you're not already logged into the remote server as the non-privileged user, open a new terminal and log in as that user. Go to your home directory in the remote server (you should already be there) and make a new directory .ssh then go into that directory and make a file authorized_keys as follows:

```
mkdir .ssh
cd .ssh
nano authorized_keys
```
Then copy and paste only the public key from your local machine (its file extension ends with `.pub`) into the remote server's authorized_keys file that you just made.

Change permissions of the authorized_keys file on the remote server so that no-one besides you can read them:

`chmod 0600 authorized_keys`

Then, without closing any existing terminals, open yet another terminal on your local machine and connect to the server via ssh as follows. This method specifies exactly which key file you're using, which is important. If you don't specify the key file (and if you have more than 1 key file on your local machine) you might wind up with an authentication error and a refusal to connect. This is because your local machine might default to using the wrong key when there is a choice of multiple. We'll also have to specify the new port every time too, otherwise it will default to port 22. Here is how to specify everything on your local machine:

`ssh -i .ssh/myserver_ed25519 -p 65000 newuser@your.server.ip.address`

Obviously, you'll need to replace almost everything here with your values - the key file, your port number, your non-privileged user name, and your server IP address.

Now, if everything has been done correctly, it will ask for a passphrase (not a password!) and this is where you type in the one from when you generated the keys.

At this point it should let you log in. If it doesn't, go back through these steps. Make sure you're able to log in as the non-privileged user through your SSH keys before moving on to the next step.

### Disable password logins for SSH access and only allow authentication keys, disable root login

Only do this step after you're able to log in through your SSH keys as your non-privileged user. If you complete this step before being able to connect via authentication keys, you will likely be locked out of your server via ssh, although you can always get back in via Webmin.

Assuming you can SSH into your server as non-privileged user with authentication keys, your next step is to disable password SSH access.

Edit the ssh config file:

`sudo nano /etc/ssh/sshd_config`

1. uncomment the PasswordAuthentication line and set it to `no`

2. Look for the following and change

`PermitRootLogin no `

3. Make sure the following line is there - you may need to add it yourself

`AllowUsers newuser`

Be sure to type in your new username correctly or you could be locked out!

Then save and exit

then restart ssh

`sudo systemctl restart ssh`

and now log in on a new terminal as new user with SSH keys as before. This should work. Try a new connection and try to log in as root, it should be denied. 
