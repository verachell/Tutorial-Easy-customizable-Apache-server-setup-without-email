## 8. Connecting your first domain and setting up SSL
We will do a quick and easy connect of domain to server via the A record at the registrar. You won’t need to set up nameservers for this

To use the A record, your domain must not be using any nameservers besides the registrar’s defaults. So set your domain’s nameservers to registrar’s defaults (otherwise changes to A record will not work).

Then still in your registrar account, go to the DNS settings for that domain. Change the existing A record so that hostname is either blank or has only the @ symbol. Then in address, enter your server’s ip address. If there is an additional A record, edit that (if no additional one, create a new one). This time with hostname as www and in address put same server IP address again. Delete any additional A records. And delete any AAA records.
