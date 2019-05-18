# Server Set Up

### Steps

#### Add User; Disable Password Auth; Set up SSH

https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04

1. Get droplet setup on Digital Ocean. Use SSH key found in my directory `~/.ssh/id_rsa.pub`. Or create a new one with `ssh-keygen` See also https://help.github.com/en/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent.
2. Sign into the new server with the SSH info provided in the email from DO `root@<serverIP>`.
3. Create user on the new server. `adduser <username>`
4. See that the account was created with `tail /etc/passwd` or `id <username>`
5. Add user to sudo group `usermod -aG sudo <username>`
6. Verify with `groups <username>` or `id <username>`
7. Use a second terminal window to verify that I can login and use the sudo command with the new user before I shutdown the original terminal or I will have locked myself out.
8. Become new user with `su - <username>` check with `whoami`
9. Make a new directory that only the new user can access that will store the ssh key. Run

```
mkdir ~/.ssh
chmid 700 ~/.ssh
nano ~/.ssh/authorized_keys
```

9. Copy the ssh key into this file and save. `pbcopy < ~/.ssh/id_rsa.pub` To copy to clipboard.
10. After saving the file change permissions `chmod 600 ~/.ssh/authorized_keys`
11. Go back to root user with `exit`
12. Now logout as root and log back into the server as the newly created user. Should work without a password if the ssh key setup work right.
13. Now lockdown the root ssh access. `sudo nano|vim /etc/ssh/sshd_config`
14. Find the line with `PermitRootLogin yes` change to `no`
15. Turn off `PasswordAuthentication` from yes to `no`
16. Then add on a new line `AllowUsers <username>`
17. Save the file and then restart the service `sudo systemctl restart ssh` or `sudo systemctl reload sshd`
18. Run an update `sudo apt update && sudo apt dist-upgrade`
19. Reboot the server `sudo reboot` - will take a little time.

#### Firewall

While working on the server before final production I want to lock down the server so that only I can access the server.

**With Digital Ocean**

Starting in the D.O account page on the left.

1. Select Networking -> firewal -> create firewall
2. Change the Inbound Rules, SSH to All TCP and find my IP address from google search.
3. Add All UDP with ip as well.
4. Put it in incorrectly first to make sure firstwall is working.
5. Fill out the apply to droplet and click the create firewall button.

**important command**

`sudo netstat -tulpn` check network status for TCP(t), UDP(u), listening ports(l), processing(p), numerical(n)

**With ufc**

1. `sudo ufw allow OpenSSH`, `sudo ufw allow http`, `sudo ufw allo https`
2. Then run `sudo ufw enable`
3. Check `sudo ufw status`

#### Intalling Packages

**Git**

`sudo apt install git`

**Node**

Visit the nodesource github page for the commands.

https://github.com/nodesource/distributions

**Mongdb**

https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-ubuntu-18-04

https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-mongodb-on-ubuntu-16-04#part-two-securing-mongodb

1. Set up admin mongo db user
2. Create db specific user that will have read and write access to the database.

How to move a local db to the server:
https://www.digitalocean.com/community/tutorials/how-to-back-up-restore-and-migrate-a-mongodb-database-on-ubuntu-14-04

After create a backup file with mongodump, use the below command to copy[(scp)](https://linuxize.com/post/how-to-use-scp-command-to-securely-transfer-files/) the file (recursively -r) to the destination.

`scp -r <from path> <to Path>`

**Redis**

`sudo apt install redis-server`

https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-18-04

**Clone Project**

Create a folder for `apps` and clone the new project there.

**Create .env file**

Add the needed variable to the environment.

**pm2**

To run the app as a process.

#### Configure SSL and Nginx with Let's Encrypt.

Run `sudo apt install bc`

https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04

https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-18-04

https://www.digitalocean.com/community/tutorials/nginx-essentials-installation-and-configuration-troubleshooting

**nginx**

https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
