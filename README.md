# InitialSetupUbuntu20.04

Introduction
When you first create a new Ubuntu 20.04 server, you should perform some important configuration steps as part of the initial setup. These steps will increase the security and usability of your server, and will give you a solid foundation for subsequent actions.

Step 1 — Logging in as root
To log into your server, you will need to know your server’s public IP address. You will also need the password or — if you installed an SSH key for authentication — the private key for the root user’s account. If you have not already logged into your server, you may want to follow our guide on how to Connect to Droplets with SSH, which covers this process in detail.

If you are not already connected to your server, log in now as the root user using the following command (substitute the highlighted portion of the command with your server’s public IP address):

ssh root@your_server_ip

Accept the warning about host authenticity if it appears. If you are using password authentication, provide your root password to log in. If you are using an SSH key that is passphrase protected, you may be prompted to enter the passphrase the first time you use the key each session. If this is your first time logging into the server with a password, you may also be prompted to change the root password.

Step 2 — Creating a New User
Once you are logged in as root, you’ll be able to add the new user account. In the future, we’ll log in with this new account instead of root.

This example creates a new user called sammy, but you should replace that with a username that you like:

adduser sammy
You will be asked a few questions, starting with the account password.

Enter a strong password and, optionally, fill in any of the additional information if you would like. This is not required and you can just hit ENTER in any field you wish to skip.

Step 3 — Granting Administrative Privileges
Now we have a new user account with regular account privileges. However, we may sometimes need to do administrative tasks.

To avoid having to log out of our normal user and log back in as the root account, we can set up what is known as superuser or root privileges for our normal account. This will allow our normal user to run commands with administrative privileges by putting the word sudo before the command.

To add these privileges to our new user, we need to add the user to the sudo group. By default, on Ubuntu 20.04, users who are members of the sudo group are allowed to use the sudo command.

As root, run this command to add your new user to the sudo group (substitute the highlighted username with your new user):

usermod -aG sudo sammy
Now, when logged in as your regular user, you can type sudo before commands to run them with superuser privileges.

Step 4 — Setting Up a Basic Firewall
Ubuntu 20.04 servers can use the UFW firewall to make sure only connections to certain services are allowed. We can set up a basic firewall using this application.

Note: If your servers are running on DigitalOcean, you can optionally use DigitalOcean Cloud Firewalls instead of the UFW firewall. We recommend using only one firewall at a time to avoid conflicting rules that may be difficult to debug.

Applications can register their profiles with UFW upon installation. These profiles allow UFW to manage these applications by name. OpenSSH, the service allowing us to connect to our server now, has a profile registered with UFW.

You can see this by typing:

ufw app list
Output
Available applications:
  OpenSSH
We need to make sure that the firewall allows SSH connections so that we can log back in next time. We can allow these connections by typing:

ufw allow OpenSSH
Afterwards, we can enable the firewall by typing:

ufw enable
Type y and press ENTER to proceed. You can see that SSH connections are still allowed by typing:

ufw status
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
As the firewall is currently blocking all connections except for SSH, if you install and configure additional services, you will need to adjust the firewall settings to allow traffic in. You can learn some common UFW operations in our UFW Essentials guide.

Step 5 — Enabling External Access for Your Regular User
Now that we have a regular user for daily use, we need to make sure we can SSH into the account directly.

Note: Until verifying that you can log in and use sudo with your new user, we recommend staying logged in as root. This way, if you have problems, you can troubleshoot and make any necessary changes as root. If you are using a DigitalOcean Droplet and experience problems with your root SSH connection, you can regain access to Droplets using the Recovery Console.

The process for configuring SSH access for your new user depends on whether your server’s root account uses a password or SSH keys for authentication.

If the root Account Uses Password Authentication
If you logged in to your root account using a password, then password authentication is enabled for SSH. You can SSH to your new user account by opening up a new terminal session and using SSH with your new username:

ssh sammy@your_server_ip
After entering your regular user’s password, you will be logged in. Remember, if you need to run a command with administrative privileges, type sudo before it like this:

sudo command_to_run
You will be prompted for your regular user password when using sudo for the first time each session (and periodically afterwards).

To enhance your server’s security, we strongly recommend setting up SSH keys instead of using password authentication. Follow our guide on setting up SSH keys on Ubuntu 20.04 to learn how to configure key-based authentication.

If the root Account Uses SSH Key Authentication
If you logged in to your root account using SSH keys, then password authentication is disabled for SSH. You will need to add a copy of your local public key to the new user’s ~/.ssh/authorized_keys file to log in successfully.

Since your public key is already in the root account’s ~/.ssh/authorized_keys file on the server, we can copy that file and directory structure to our new user account in our existing session.

The simplest way to copy the files with the correct ownership and permissions is with the rsync command. This will copy the root user’s .ssh directory, preserve the permissions, and modify the file owners, all in a single command. Make sure to change the highlighted portions of the command below to match your regular user’s name:

Note: The rsync command treats sources and destinations that end with a trailing slash differently than those without a trailing slash. When using rsync below, be sure that the source directory (~/.ssh) does not include a trailing slash (check to make sure you are not using ~/.ssh/).

If you accidentally add a trailing slash to the command, rsync will copy the contents of the root account’s ~/.ssh directory to the sudo user’s home directory instead of copying the entire ~/.ssh directory structure. The files will be in the wrong location and SSH will not be able to find and use them.

rsync --archive --chown=sammy:sammy ~/.ssh /home/sammy
Now, open up a new terminal session on your local machine, and use SSH with your new username:

ssh sammy@your_server_ip
You should be logged in to the new user account without using a password. Remember, if you need to run a command with administrative privileges, type sudo before it like this:

sudo command_to_run
You will be prompted for your regular user password when using sudo for the first time each session (and periodically afterwards).
