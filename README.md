# PP4

## Goal

In this exercise you will:

* Use SSH to connect to remote servers from WSL, macOS, or Linux shells, understanding the handshake and authentication process.
* Generate an Ed25519 SSH key pair and explain the concept of digital signatures.
* Configure your local SSH client via the `~/.ssh/config` file for streamlined access.
* Securely copy files between local and remote hosts using `scp`, including local-to-remote, remote-to-local, and remote-to-remote transfers.
* Automate startup tasks on the remote server by writing a shell script that runs at login and explaining the role of `~/.bashrc` vs. `~/.profile`.

**Important:** Start a stopwatch when you begin and work uninterruptedly for **90 minutes**. Once time is up, stop immediately and record exactly where you paused.

---

## Workflow

1. **Fork** this repository
2. **Modify & commit** your solution
3. **Submit your link for Review**

---

## Prerequisites

* Several starter repos are available here:
  [https://github.com/orgs/STEMgraph/repositories?q=SSH%3A](https://github.com/orgs/STEMgraph/repositories?q=SSH%3A)
* Consult the SSH and SCP man-pages for detailed options and explanations:

  * `man ssh`
  * `man scp`

---

## Tasks

### Task 1: SSH Login

**Objective:** Establish an SSH connection and observe each stage of the process.

1. From your local shell (WSL, macOS Terminal, or Linux), log into the `vorlesungsserver` (or any other remote machine of your choice, e.g. your own raspberry pi):

   ```bash
   ssh youruser@remotehost
   ```
2. Carefully observe and note each step:

   * **TCP connection** to port 22 on `remotehost`.
   * **SSH protocol handshake**: key exchange and algorithm negotiation.
   * **Authentication**: public-key or password exchange.
   * **Shell allocation**: your remote session starts.
3. After login, exit the session with `exit`.

**Provide:**

```bash
# 1) The exact ssh command you ran:
 ssh -v phil@192.168.178.28

# 2) A detailed, step-by-step explanation of what happened at each stage
At first, OpenSSH got opened on my Ubuntu spave, It showed the Version and Date of Instalation, then the program startet reading the config file from my .ssh ordner. It choose options for my *host(Which is the configuration that is used for connections to unknown hosts(hosts that are not implemented in my config). Than the program started connesting to my host over port 22. Connection worked.
After that, the program searched for an identity file (which is not configurated for host *(I think it did this, because I already have an .pubkey on that server but the private key is only  configuratet for my "Shortcut" that I normaly use). After the program was shure that there is no key it prompted me to login via password. After typing the password, the program opend a new Session on channel 0. After the connection was finished, it showed me Ubuntu Version and last login.
```



---

### Task 2: Ed25519 Key Pair

**Objective:** Create a secure key pair and explain how digital signatures verify identity.

1. Generate an Ed25519 SSH key pair:

   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

   * Accept the default file location (`~/.ssh/id_ed25519`). Or provide the `-f <filepath>` option additionally.
   * Enter a passphrase when prompted (optional).
2. Locate and inspect your `id_ed25519` (private key) and `id_ed25519.pub` (public key).
3. Install your key on the remote machine (e.g. `vorlesungsserver`.
4. Explain in writing:

   * How the **private key** is used to sign challenges.
     The private key uses an mathematical function to encrypt the Challenge and adds a Hash vallue.
  
     
   * How the **public key** on the server verifies signatures without revealing the private key.
     
     The public key works as an decrypter for the SSH verification. When you are starting an SSH Session, the private key uses an mathematical function to encrypt the Challenge and adds a Hash vallue. When the Challenge arrives the host, he uses his .pub key to generate a vallue called Hash (He generates it with an mathematical function of the Challenge). If the Hash is the same as the signed vallue to the Data, it get's decrypted.
   * Why Ed25519 is preferred (performance, security).
     
The Ed25519 key is shorter than for example a RSA key, this turns in to faster encryption and decryption, but the security level of the key is the same. (It uses a different function, that works more efficently)

**Provide:**

```bash
# 1) The ssh-keygen command you ran
 ssh-keygen -t ed25519 -f Aspire
# 2) The file paths of the generated keys
phil@phil-Aspire-E5-511:~/.ssh$ ls
authorized_keys
phil@phil-Aspire-E5-511:~/.ssh$ pwd
/home/phil/.ssh

# 3) Your written explanation (3–5 sentences) of the signature process

  The public key works as an decrypter for the SSH verification. When you are starting an SSH Session, the private key uses an mathematical function to encrypt the Challenge and adds a Hash vallue. When the Challenge arrives the host, he uses his .pub key to generate a vallue called Hash (He generates it with an mathematical function of the Challenge). If the Hash is the same as the signed vallue to the Data, it get's decrypted.

```

---

### Task 3: SSH Config File

**Objective:** Simplify SSH commands via `~/.ssh/config`.

1. Open (or create) `~/.ssh/config` in `vim`.
2. Add entries for your hosts, for example:

   ```text
   Host my-remote
       HostName remote.example.com
       User youruser
       IdentityFile ~/.ssh/id_ed25519

   Host backup-server
       HostName backup.example.com
       User backupuser
       Port 2222
       IdentityFile ~/.ssh/id_ed25519_backup
   ```
3. Save and close the file, then test:

   ```bash
   ssh my-remote
   ssh backup-server
   ```
4. Explain:

   * How SSH reads `~/.ssh/config` and matches hosts.
     SSH takes the Host alias and compares it with the Hosts at the ~/.ssh/config file. It starts at the top,
     and goes to the bottom till it found the matching "Partner".
     

   * The difference between `HostName` and `Host`.
     HostName is the Ip, Host is my chosen Alias for the machine.

   * How aliases prevent long commands.
     What is there to explain? The deposited Data behind my alias is the command, the alias could be an easy name like MyRemote, wich is way shoreter and easier than ssh phil@192.168.178.28 . I was always switching accidently the numbers.

**Provide:**

```text
# 1) The full contents of your ~/.ssh/config

Host github.com
    HostName github.com
    User git
    IdentityFile /home/philr/hierkönnteihrewerbungstehen

Host aspire
    HostName 192.168.178.28
    User phil
    IdentityFile ~/hierkönnteihrewerbungstehen

Host *
ServerAliveInterval 60
ServerAliveCountMax 3

# 2) A short explanation (3–4 sentences) of how the config simplifies connections
I personally love the config file, because it's way easier to type only my predifined "Host" Instad of typing the hole command +user+IP. Often typed the IP wrong, the config not only helps me with the shortcut, but automatically helps me to organize all my Server connections. It's easy to maintain and makes live very easy, before I used it, I always had to check the Ip on my Router.
```

---

### Task 4: SCP File Transfers

**Objective:** Practice copying files securely using `scp`.

1. **Local → Remote**:

   ```bash
   scp /path/to/localfile.txt youruser@remotehost:~/destination/
   ```
2. **Remote → Local**:

   ```bash
   scp youruser@remotehost:~/remotefile.log ./local_destination/
   ```
3. **Remote → Remote** (between two directories on the same remote host):

   ```bash
   scp -r youruser@remotehost:/path/dir1 youruser@remotehost:/path/dir2
   ```
4. For each command:

   * Verify file timestamps and sizes after transfer, using `ls -la`
     Thank you for saying that at the end after half of my output got deleted
   * Note any flags you used (e.g., `-r`, `-P` for port).
5. Explain:

   * How `scp` initiates an SSH session for each transfer.
   * The role of encryption in protecting data in transit.

**Provide:**

```bash
# 1) Each scp command you ran: Did some faults but not gonna do it a 5th time, my terminal limits the amount of lines that get stored, so when i used much space half of the output is away.

philr@3245-Laptop:~/Programieren$  scp ./Sens.txt aspire:~/Programmieren
Sens.txt                                                                              100%    0     0.0KB/s   00:00
philr@3245-Laptop:~/Programieren$ ssh aspire

Last login: Fri May  9 19:48:27 2025 from 192.168.178.30
phil@phil-Aspire-E5-511:~$ cd Programmieren
phil@phil-Aspire-E5-511:~/Programmieren$ ls -la
insgesamt 12
drwxrwxr-x  2 phil phil 4096 Mai  9 19:49 .
drwxr-x--x 31 phil phil 4096 Mai  9 19:41 ..
-rw-rw-r--  1 phil phil    0 Mai  9 19:41 Ehrenrunde.txt
-rw-r--r--  1 phil phil    0 Mai  9 19:48 Programieren
-rw-r--r--  1 phil phil    0 Mai  9 19:49 Sens.txt
-rw-rw-r--  1 phil phil   12 Mai  9 18:04 Supertest
phil@phil-Aspire-E5-511:~/Programmieren$
Connection to 192.168.178.28 closed.
philr@3245-Laptop:~/Programieren$  scp ./Programieren/Sens.txt aspire:~/Programmieren
scp: stat local "./Programieren/Sens.txt": Not a directory
philr@3245-Laptop:~/Programieren$  scp ./Sens.txt aspire:~/Programmieren
Sens.txt                                                                              100%    0     0.0KB/s   00:00
philr@3245-Laptop:~/Programieren$ ssh aspire

Last login: Fri May  9 19:48:27 2025 from 192.168.178.30
phil@phil-Aspire-E5-511:~$ cd Programmieren
phil@phil-Aspire-E5-511:~/Programmieren$ ls -la
insgesamt 12
drwxrwxr-x  2 phil phil 4096 Mai  9 19:49 .
drwxr-x--x 31 phil phil 4096 Mai  9 19:41 ..
-rw-rw-r--  1 phil phil    0 Mai  9 19:41 Ehrenrunde.txt
-rw-r--r--  1 phil phil    0 Mai  9 19:48 Programieren
-rw-r--r--  1 phil phil    0 Mai  9 19:49 Sens.txt
-rw-rw-r--  1 phil phil   12 Mai  9 18:04 Supertest
phil@phil-Aspire-E5-511:~/Programmieren$ exit
Abgemeldet
Connection to 192.168.178.28 closed.
philr@3245-Laptop:~/Programieren$ scp aspire:./Ehrenrunde.txt ~
scp: ./Ehrenrunde.txt: No such file or directory
philr@3245-Laptop:~/Programieren$ scp aspire: ./Ehrenrunde.txt ~
scp: download ./: not a regular file
philr@3245-Laptop:~/Programieren$ scp aspire:~/Programmieren/Ehrenrunde.txt ~
philr@3245-Laptop:~/Programieren$ cd ~
philr@3245-Laptop:~$ ls -la
total 156
drwxr-x--- 12 philr philr  4096 May  9 19:56 .
drwxr-xr-x  3 root  root   4096 Apr 13 10:15 ..
-rw-r--r--  1 philr philr 61440 Apr 11 21:10 .Tutorial.swp
-rw-------  1 philr philr 10479 May  9 17:58 .bash_history
-rw-r--r--  1 philr philr   220 Apr 11 15:01 .bash_logout
-rw-r--r--  1 philr philr  3940 May  4 10:43 .bashrc
drwx------  2 philr philr  4096 Apr 11 15:01 .cache
drwx------  3 philr philr  4096 Apr 11 15:16 .config
-rw-r--r--  1 philr philr    50 Apr 18 11:23 .gitconfig
drwxr-xr-x  2 philr philr  4096 Apr 11 15:01 .landscape
-rw-------  1 philr philr    20 May  7 10:36 .lesshst
drwxr-xr-x  3 philr philr  4096 Apr 12 11:40 .local
-rw-r--r--  1 philr philr     0 May  9 16:49 .motd_shown
-rw-r--r--  1 philr philr   807 Apr 11 15:01 .profile
drwx------  2 philr philr  4096 May  9 17:49 .ssh
-rw-r--r--  1 philr philr     0 Apr 11 15:39 .sudo_as_admin_successful
-rw-------  1 philr philr 16217 May  9 19:46 .viminfo
drwx------  2 philr philr  4096 Apr 19 13:10 .w3m
-rw-r--r--  1 philr philr     0 May  9 19:56 Ehrenrunde.txt
drwxr-xr-x  3 philr philr  4096 Apr 19 14:04 Praktikum
drwxr-xr-x  8 philr philr  4096 May  9 19:47 Programieren
drwxr-xr-x  4 philr philr  4096 Apr 19 12:21 Programmieren
drwxr-xr-x  2 philr philr  4096 Apr 19 14:11 html
philr@3245-Laptop:~$

phil@phil-Aspire-E5-511:~$ exit
Abgemeldet
Connection to 192.168.178.28 closed.
philr@3245-Laptop:~$ scp aspire:~/Programmieren/Ehrenrunde.txt aspire:~
philr@3245-Laptop:~$ ssh aspire

Last login: Fri May  9 19:59:36 2025 from 192.168.178.30
phil@phil-Aspire-E5-511:~$ ls -la
insgesamt 188
drwxr-x--x 31 phil phil 4096 Mai  9 20:00 .
drwxr-xr-x  3 root root 4096 Mär 12  2024 ..
-rw-------  1 phil phil 3038 Mai  9 20:00 .bash_history
-rw-r--r--  1 phil phil  220 Mär 12  2024 .bash_logout
-rw-r--r--  1 phil phil 3771 Mär 12  2024 .bashrc
drwxr-xr-x  2 phil phil 4096 Mär 12  2024 Bilder
drwxrwxr-x 12 phil phil 4096 Mär 14  2024 .cache
drwxr-xr-x 13 phil phil 4096 Mär 31 16:46 .config
-rw-r--r--  1 phil phil   23 Mär 12  2024 .dmrc
drwxr-xr-x  2 phil phil 4096 Mär 12  2024 Dokumente
drwxr-xr-x  2 phil phil 4096 Mär 12  2024 Downloads
-rw-rw-r--  1 phil phil    0 Mai  9 20:00 Ehrenrunde.txt
drwx------  3 phil phil 4096 Mär 12  2024 .gnupg
-rw-r--r--  1 phil phil   22 Mär 12  2024 .gtkrc-2.0
-rw-r--r--  1 phil phil  516 Mär 12  2024 .gtkrc-xfce
-rw-------  1 phil phil    0 Mär 12  2024 .ICEauthority
drwxrwxr-x  7 phil phil 4096 Mär 12  2024 kiauh
drwxrwxr-x  4 phil phil 4096 Mär 12  2024 kiauh-backups
-rw-rw-r--  1 phil phil  570 Mär 12  2024 .kiauh.ini
drwxrwxr-x 12 phil phil 4096 Apr  4 22:06 klipper
drwxrwxr-x  5 phil phil 4096 Mär 12  2024 klippy-env
drwxrwxr-x  4 phil phil 4096 Mär 23  2024 .linuxmint
drwxrwxr-x  3 phil phil 4096 Mär 12  2024 .local
drwxr-xr-x  6 phil phil 4096 Apr  4 22:04 mainsail
drwxrwxr-x  3 phil phil 4096 Apr  4 22:04 mainsail-config
drwxrwxr-x  8 phil phil 4096 Apr  4 22:05 moonraker
drwxrwxr-x  4 phil phil 4096 Mär 12  2024 moonraker-env
drwx------  4 phil phil 4096 Mär 12  2024 .mozilla
drwxr-xr-x  2 phil phil 4096 Mär 12  2024 Musik
drwxr-xr-x  2 phil phil 4096 Mär 12  2024 Öffentlich
drwxrwxr-x 11 phil phil 4096 Mär 12  2024 printer_1_data
drwxrwxr-x 11 phil phil 4096 Mär 12  2024 printer_2_data
drwxrwxr-x 11 phil phil 4096 Mär 12  2024 printer_3_data
drwxrwxr-x 11 phil phil 4096 Mär 12  2024 printer_4_data
-rw-r--r--  1 phil phil  807 Mär 12  2024 .profile
drwxrwxr-x  2 phil phil 4096 Mai  9 19:49 Programmieren
drwxrwxr-x  3 phil phil 4096 Mai  8 16:21 repos
drwxr-xr-x  2 phil phil 4096 Mär 12  2024 Schreibtisch
drwx------  2 phil phil 4096 Mai  7 16:47 .ssh
-rw-r--r--  1 phil phil    0 Mär 12  2024 .sudo_as_admin_successful
-rw-rw-r--  1 phil phil   12 Mai  9 19:39 Supertest
drwxr-xr-x  2 phil phil 4096 Mär 12  2024 Videos
-rw-------  1 phil phil 3682 Mai  9 19:41 .viminfo
drwxr-xr-x  2 phil phil 4096 Mär 12  2024 Vorlagen
-rw-rw-r--  1 phil phil  165 Mär 12  2024 .wget-hsts
-rw-------  1 phil phil   63 Mai  9 17:15 .Xauthority
-rw-------  1 phil phil 5459 Mai  9 17:15 .xsession-errors
-rw-------  1 phil phil 5459 Mai  7 13:27 .xsession-errors.old
phil@phil-Aspire-E5-511:~$

# 2) Any flags or options used -
# 3) A brief explanation (2–3 sentences) of scp’s mechanism
It's basically copying data save over ssh, it uses the same security methods as ssh. When used correct it's a fast was to copy data from another machine to your own machine or somewhere else.
```

---

### Task 5: Login Shell Script & Profile Explanation

**Objective:** Automate commands at login and understand shell initialization files.

1. On the **remote** server, create a script `~/login_tasks.sh` containing at least three commands you find useful (e.g., `echo "Welcome $(whoami)"`, `uptime`, `ls ~/projects`). You may either use `vim` or try the following to create a file from your commandline directely:

   ```bash
   cat << 'EOF' > ~/login_tasks.sh
   #!/usr/bin/env bash
   echo "Welcome $(whoami)! Today is $(date)."
   uptime
   ls ~/projects
   EOF
   chmod +x ~/login_tasks.sh
   ```

> The files content should be something akin to:
> ```bash
> #!/usr/bin/env bash
> echo "Welcome $(whoami)! Today is $(date)."
> uptime
> ls ~/projects
> ```

2. Append to your `~/.bashrc` (or `~/.profile` if using a login shell) a line to source this script on each new session:

   ```bash
   echo "source ~/login_tasks.sh" >> ~/.bashrc
   ```
3. Log out and log back in to trigger the script.
4. Explain:

   * The difference between `~/.bashrc` and `~/.profile` (interactive vs. login shells).

     .bashrc starts every time when a new shell is created (not loginshell) it's usefull for interactive Sessions, or scripts that often used, .profile only starts at the login for example SSH, it's usefull for nice Thinks that you want to see after your login for example : Hello therer, or Time and Date, ....

    * Why and when each file is read.

.bashrc starts every time when a new shell is created (not loginshell). .profile starts only at Login. (Loginshell)
     

   * How sourcing differs from executing.
  
  The main differents between sourcing and executing is the place where it's done. Sourcing means, that it's done in the actuall shell. Every change or modification that is done has an influence on the actual shell. Executing is the opposite of sorcing, something that gets executet is done in a new shell (subshell). Every modification or change has no influence on the first shell.
     

**Provide:**

```bash
# 1) The contents of login_tasks.sh
#!/bin/bash -ln

echo "Hello there"
uptime
ls ~/Programmieren

# 2) The lines you added to ~/.bashrc or ~/.profile
source ~/login_tasks.sh
# 3) Your explanation (3–5 sentences) of shell init files and sourcing vs. executing

 The main differents between sourcing and executing is the place where it's done. Sourcing means, that it's done in the actuall shell. Every change or modification that is done has an influence on the actual shell. Executing is the opposite of sorcing, something that gets executet is done in a new shell (subshell). Every modification or change has no influence on the first shell.
```

---

**Remember:** Stop working after **90 minutes** and record where you stopped.
