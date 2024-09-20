---
title: User-SSH-Keys
description: 
published: true
date: 2024-09-20T21:35:05.216Z
tags: 
editor: markdown
dateCreated: 2024-09-20T08:10:27.394Z
---

### SSH Configuration & Setup for Beginners

In this tutorial, you'll learn how to set up and configure SSH (Secure Shell) on your Raspberry Pi or any Linux-based system. This guide is meant for absolute beginners, so we'll explain every step in detail.

### 1. **Install `sudo` if not present**

Some Linux distributions don’t have `sudo` pre-installed. Sudo allows you to run commands as a superuser (administrator). To install it, use the following command:

```bash
apt-get install sudo -y
```

Once installed, you'll want to add your user to the sudo group to allow them to execute commands with elevated privileges:

```bash
sudo adduser admin
```

After creating the user, add them to the `sudo` group:

```bash
sudo usermod -aG sudo admin
```

To verify the user is part of the `sudo` group:

```bash
groups admin
```

This will show the groups the user belongs to. You should see `sudo` in the list.

To make sure the system is properly configured, edit the sudoers file:

```bash
sudo visudo
```

If the `%sudo` group entry is missing, add the following line to ensure users in the sudo group have full privileges:

```bash
%sudo   ALL=(ALL:ALL) ALL
```

If you don't want to be prompted for a password every time you run a command with `sudo`, you can configure it to not ask for a password:

```bash
admin ALL=(ALL) NOPASSWD:ALL
```

Now your user `admin` can run `sudo` commands without entering a password each time.


![screenshot_2024-09-20_095917.png](/defaults/ssh-key-user/screenshot_2024-09-20_095917.png)

---

### 2. **Generate an SSH Key**

SSH keys provide a more secure way to log in to an SSH session compared to using passwords. To generate an SSH key, use the following command:

Execute this Command on your Client. Use a Linux Distro with Windows-WSL ( tutorial soon )


```bash
ssh-keygen -t rsa -b 4096 -C "admin"
```

- `-t rsa`: Specifies the type of key, in this case, RSA (Rivest-Shamir-Adleman).
- `-b 4096`: Generates a 4096-bit key, which is more secure.
- `-C "admin"`: Adds a comment to help identify the key.

You can also use **PuTTYgen** if you're using Windows. You can download it from [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) and follow a guide to generate an SSH key using PuTTY (link to tutorial coming soon).

![screenshot_2024-09-20_094437.png](/defaults/ssh-key-user/screenshot_2024-09-20_094437.png)

---

### 3. **Configure SSH Key Authentication**

To allow SSH key-based login for your `admin` user:

1. Switch to the `admin` user:

   ```bash
   su admin
   ```

2. Create the `.ssh` directory and set the correct permissions:

   ```bash
   mkdir -p ~/.ssh
   chmod 700 ~/.ssh
   ```



3. Add your SSH public key to the `authorized_keys` file:



   ```bash
   nano ./.ssh/authorized_keys

   ```
	Dein SSH Key einfügen


4. Set the appropriate permissions:

   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

5. Exit the admin user:

   ```bash
   exit
   ```

Now, SSH key authentication is set up for the `admin` user.

---

### 4. **Editing the SSH Configuration**

You need to adjust the SSH configuration to improve security and allow key-based authentication. Open the SSH configuration file for editing:

```bash
sudo nano /etc/ssh/sshd_config
```

Make the following changes:

1. Disable root login for security:

   ```bash
   PermitRootLogin no
   ```

2. Enable public key authentication:

   ```bash
	 # uncomment
   # PubkeyAuthentication yes
   PubkeyAuthentication yes
   ```

3. Keep password authentication enabled for testing, but **only for initial setup**. Disable it later for better security:

   ```bash
   PasswordAuthentication yes
   ```

   > **Important**: Once SSH key authentication works, set this to `no` to prevent password-based logins.

4. Ensure the following lines are correctly set to enable X11 forwarding and PAM authentication:

   ```bash
   UsePAM yes
   X11Forwarding yes
   ```

5. Save the file (`Ctrl + O`), and exit the editor (`Ctrl + X`).

---

### 5. **Restart SSH Service**

After editing the configuration, restart the SSH service to apply the changes:

```bash
sudo service ssh restart
```

To check if the service is running properly, use:

```bash
sudo service ssh status
```

You should see that SSH is active and running.

---

### Explanation for Absolute Beginners

1. **What is SSH?**
   SSH (Secure Shell) is a protocol that allows you to connect to another computer securely over the internet or local network. It encrypts the communication between you and the server, making it safe from eavesdroppers.

2. **What are SSH Keys?**
   SSH keys are a pair of cryptographic keys (a private key and a public key) that provide a more secure way to log in to a server compared to passwords. The public key is placed on the server, and only the person with the corresponding private key can log in.

3. **Why Disable Root Login?**
   The root user is the most powerful user in a Linux system. Allowing remote root login can be a security risk because attackers target it. By disabling it, you make it harder for unauthorized users to gain control of your system.

4. **What is PAM?**
   PAM (Pluggable Authentication Modules) is a system that handles authentication tasks in Linux. It allows different authentication methods, including password and key-based logins.

---

### Final Thoughts

After completing these steps, your SSH setup will be more secure, and you will be able to log in using your SSH key without needing a password. Remember to disable password authentication after confirming that your SSH key works.

If you need further explanations or additional steps, feel free to ask!

