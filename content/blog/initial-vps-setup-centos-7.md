---
title: "Initial VPS setup with Linux CentOS 7"
date: 2023-03-10T22:38:11Z
draft: false
---
# Initial VPS setup with Linux CentOS 7
Virtual Private Server (VPS) have become increasingly popular over the years due to their flexibility and affordability. They allow you to have a dedicated server environment without the need for expensive hardware.

For a while now, I've been wanting to migrate my web apps from personal projects to a Virtual Private Server (VPS). This desire stems not only from the challenge it presents, but also because it would give me complete control over software versions, updates, tools, and more.

Furthermore, moving to a VPS would allow me to achieve a [personal goal for the year](2023-year-goals.md): setting up a complete CI/CD pipeline.

In this blog post, I''l will walk you through the steps I took for the initial setup of my VPS with Linux CentOS 7.

## Step 1: Logging in and Updating Packages

Once you have your VPS credentials, the first thing you will need to do is log in to your server using SSH. You will need to have an SSH client installed on your local machine. On a Mac or Linux computer, you can use the Terminal application. On Windows, you can use a program like PuTTY.

Once you are logged in, the first thing you should do is update your system packages. To do this, run the following command:

{{< highlight bash >}}
sudo yum update
{{< /highlight >}}

This command will update all of the packages on your system to their latest versions.

## Step 2: Creating a New User

By default, the root user is the only user on the system. It is not recommended to use the root user for regular use because it has full access to the system. Therefore, it is best to create a new user and use that user for day-to-day operations.

To create a new user, run the following command:

{{< highlight bash >}}
sudo adduser username
{{< /highlight >}}

Replace "username" with the name you want to give your new user. You will then be prompted to create a password and enter some basic information about the user.

## Step 3: Granting Sudo Access

By default, the new user does not have sudo (superuser) access. Sudo access is required to perform administrative tasks on the system. To grant sudo access, run the following command:

{{< highlight bash >}}
sudo usermod -aG wheel username
{{< /highlight >}}

This command adds the new user to the "wheel" group, which has sudo access.

## Step 4: Adding Public Key Authentication

Public key authentication is a more secure way to log in to your VPS compared to using passwords. It involves generating a public and private key pair, where the private key is kept on your local machine and the public key is added to your VPS.

To generate a key pair, run the following command on your local machine:

{{< highlight bash >}}
ssh-keygen
{{< /highlight >}}

You will be prompted to enter a filename for the key pair and a passphrase to protect the private key. Once you have generated the key pair, you need to add the public key to your VPS.

Log in to your VPS with your non-root user and run the following command:

{{< highlight bash >}}
mkdir ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
{{< /highlight >}}

This creates a new directory for your ssh keys, sets the permissions, and opens the authorized_keys file in the nano text editor.

Copy the public key from your local machine (it should be in the file you specified when generating the key pair, with a .pub extension) and paste it into the authorized_keys file on your VPS. Save and exit the file.

Finally, set the permissions on the authorized_keys file by running the following command:

{{< highlight bash >}}
chmod 600 ~/.ssh/authorized_keys
{{< /highlight >}}

From now on, you will be able to log in to your VPS with public key authentication instead of a password.

## Step 5: Disabling Root Login

To improve the security of your VPS, it is recommended to disable direct root login. To do this, edit the sshd configuration file by running the following command:

{{< highlight bash >}}
sudo nano /etc/ssh/sshd_config
{{< /highlight >}}

Find the line that says "PermitRootLogin yes" and change it to "PermitRootLogin no". Save and exit the file.

Restart the sshd service by running the following command:

{{< highlight bash >}}
sudo systemctl restart sshd
{{< /highlight >}}

From now on, you will need to log in with your non-root user and then use sudo to perform administrative tasks.

## Step 6: Configuring Firewall with iptables

Finally, it's important to configure the firewall on your VPS to protect it from unauthorized access. CentOS 7 comes with a default firewall called firewalld, but we'll be using the older and more established iptables instead. Before we install and configure iptables, we need to disable firewalld:

{{< highlight bash >}}
sudo systemctl stop firewalld
sudo systemctl mask firewalld
{{< /highlight >}}

This stops the firewalld service and masks it, which prevents it from starting automatically on boot.

Next, we can install and configure iptables:

{{< highlight bash >}}
sudo yum install iptables-services
sudo systemctl enable iptables
sudo systemctl start iptables
{{< /highlight >}}

This installs the iptables services package, enables the iptables service to start on boot, and starts the service.

Now we can configure the iptables rules. For example, to allow incoming SSH traffic, we need to add a rule to allow traffic on port 22:

{{< highlight bash >}}
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
{{< /highlight >}}

This adds a rule to the INPUT chain to accept TCP traffic on port 22 (SSH).

To allow incoming HTTP traffic, we can add a rule to allow traffic on port 80:

{{< highlight bash >}}
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
{{< /highlight >}}

To allow incoming HTTPS traffic, we can add a rule to allow traffic on port 443:

{{< highlight bash >}}
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
{{< /highlight >}}

You can add additional rules as needed, depending on the services you'll be running on your VPS.

To make sure these rules are saved and applied after a reboot, we need to save them to the iptables configuration file:

{{< highlight bash >}}
sudo service iptables save
{{< /highlight >}}

This saves the current iptables rules to the configuration file.

## Step 7: Installing fail2ban

Another important security measure you can take to protect your VPS is to install and configure fail2ban, which is a tool that helps prevent brute-force attacks by monitoring log files and banning IP addresses that show suspicious behavior.

To install fail2ban, run the following command:

{{< highlight bash >}}
sudo yum install epel-release fail2ban
{{< /highlight >}}

Once fail2ban is installed, you need to create a copy of the default configuration file:

{{< highlight bash >}}
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
{{< /highlight >}}

This creates a local copy of the configuration file that you can customize without affecting the default settings.

Next, open the jail.local file in your preferred text editor:

{{< highlight bash >}}
sudo nano /etc/fail2ban/jail.local
{{< /highlight >}}

This file contains all the default settings for fail2ban, as well as any custom settings you add.

You can customize fail2ban by modifying the various settings in the file. For example, you can adjust the ban time and findtime settings to control how long IPs are banned and how frequently the logs are checked for suspicious activity.

Once you've made any necessary changes to the configuration file, save and close it.

Finally, start the fail2ban service and enable it to start on boot:

{{< highlight bash >}}
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
{{< /highlight >}}

fail2ban is now installed and configured on your VPS to help prevent brute-force attacks and enhance your overall security.

That's it! Your VPS is now configured with a secure user account, public key authentication, sudo access, disabled root login, and a firewall using iptables.

By following these steps, you will have a solid foundation for your VPS and be ready to start deploying your applications.
