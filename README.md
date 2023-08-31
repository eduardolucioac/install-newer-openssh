# OpenSSH Service - Install a newer version (Debian)

![OpenSSH Service](./images/openssh-service.png)

## Introduction

Using a OpenSSH service newer version (9.4 at the time of this guide) is easy and crucial for security.

We will install a OpenSSH service newer version while keeping and disabling the one installed by the package manager (Debian official repositories).

Tested on Debian GNU/Linux 11 (bullseye).

**NOTE:** For easier understanding, we will refer to the version installed by the package manager (Debian official repositories) as the OpenSSH service repo version.

**IMPORTANT:** Installing a OpenSSH service newer version alongside the OpenSSH service repo version is recommended because components of the OpenSSH service repo version are used by other components and packages, and its absence can be a source of problems.

**IMPORTANT:** My life, my work and my passion is free software. Corrections, tweaks and improvements are very welcome (**pull requests** 😉)! Please consider giving us a ⭐, fork, support this project or even visit our professional profile (see [About](#about)). **Thanks!** 🤗

## Install the required packages

Install the necessary packages to build and run the OpenSSH service newer version...

```
apt -y install autoconf
apt -y install build-essential
apt -y install cmake
apt -y install libssh-dev
apt -y install libtool
apt -y install netcat
apt -y install wget
```

## Download and build the OpenSSH service newer version

Commands to download and build...

```
wget https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-9.4p1.tar.gz
OPENSSH_VER="9.4p1"
tar -xvf openssh-${OPENSSH_VER}.tar.gz
cd openssh-${OPENSSH_VER}
./configure --prefix=/opt/openssh-${OPENSSH_VER}
make
make install
cd ..
rm -rf openssh-${OPENSSH_VER}.tar.gz
```

**TIP:** OpenSSH service newer versions can be found at https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/ .

**IMPORTANT:** You should use a *portable* version.

Create a symbolic link (*symbolic link*) so that we can update the OpenSSH service more easily in the future, reusing some configurations...

```
ln -s /opt/openssh-${OPENSSH_VER} /opt/openssh-latest
```

## Change the OpenSSH service repo version configuration

Change the OpenSSH service repo version port in `/etc/ssh/sshd_config` configuration file by changing the "#Port 22" parameter to "Port 2222", for example.

**NOTE:** The configuration file for the OpenSSH service newer version will be in the path `/opt/openssh-latest/etc/sshd_config`.

## Create the default environment file for the OpenSSH service newer version

Commands to create the default environment file and a folder for it...

```
mkdir -p "/opt/openssh-latest/default"
read -r -d '' FILE_CONTENT << 'HEREDOC'
BEGIN
# Default settings for OpenSSH Server.

# Options to pass to sshd.
SSHD_OPTS=

END
HEREDOC
echo -n "${FILE_CONTENT:6:-3}" > '/opt/openssh-latest/default/ssh'
```

## Create service configuration files (systemd)

Commands to create the "ssh-latest.service" file...

```
read -r -d '' FILE_CONTENT << 'HEREDOC'
BEGIN
[Unit]
Description=OpenBSD Secure Shell server
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target auditd.service
ConditionPathExists=!/opt/openssh-latest/etc/sshd_not_to_be_r

[Service]
EnvironmentFile=-/opt/openssh-latest/default/ssh
ExecStartPre=/opt/openssh-latest/sbin/sshd -t
ExecStart=/opt/openssh-latest/sbin/sshd -D $SSHD_OPTS
ExecReload=/opt/openssh-latest/sbin/sshd -t
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartPreventExitStatus=255
Type=exec
RuntimeDirectory=sshd-latest
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
Alias=sshd-latest.service

END
HEREDOC
echo -n "${FILE_CONTENT:6:-3}" > "/usr/lib/systemd/system/ssh-latest.service"
```

Commands to create the "ssh-latest@.service" file...

```
read -r -d '' FILE_CONTENT << 'HEREDOC'
BEGIN
[Unit]
Description=OpenBSD Secure Shell server per-connection daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=auditd.service

[Service]
EnvironmentFile=-/opt/openssh-latest/default/ssh
ExecStart=/opt/openssh-latest/sbin/sshd -i $SSHD_OPTS
StandardInput=socket
RuntimeDirectory=sshd-latest
RuntimeDirectoryMode=0755

END
HEREDOC
echo -n "${FILE_CONTENT:6:-3}" > "/usr/lib/systemd/system/ssh-latest@.service"
```

Commands to create the "ssh-latest.socket" file...

```
read -r -d '' FILE_CONTENT << 'HEREDOC'
BEGIN
[Unit]
Description=OpenBSD Secure Shell server socket
Before=ssh-latest.service
Conflicts=ssh-latest.service
ConditionPathExists=!/opt/openssh-latest/etc/sshd_not_to_be_r

[Socket]
ListenStream=22
Accept=yes

[Install]
WantedBy=sockets.target

END
HEREDOC
echo -n "${FILE_CONTENT:6:-3}" > "/usr/lib/systemd/system/ssh-latest.socket"
```

Commands to create the "rescue-ssh-latest.target" file...

```
read -r -d '' FILE_CONTENT << 'HEREDOC'
BEGIN
[Unit]
Description=Rescue with network and ssh
Documentation=man:systemd.special(7)
Requires=network-online.target ssh-latest.service
After=network-online.target ssh-latest.service
AllowIsolate=yes

END
HEREDOC
echo -n "${FILE_CONTENT:6:-3}" > "/usr/lib/systemd/system/rescue-ssh-latest.target"
```

## Service configurations (systemd)

Disable the OpenSSH service repo version...

```
systemctl disable ssh.service
systemctl disable ssh.socket
```

Enable the OpenSSH service newer version...

```
systemctl enable ssh-latest.service
systemctl enable ssh-latest.socket
```

Reboot your system...

```
reboot
```

## Confirm the version of the OpenSSH service newer version

Confirm the version via the binary...

```
/opt/openssh-latest/bin/ssh -V
```

Confirm the version via the service...

```
echo | nc 127.0.0.1 22
```

[Ref(s).: https://gist.github.com/jtmoon79/745e6df63dd14b9f2d17a662179e953a ]

```
  .~.  Have fun! =D
  /V\  
 // \\ Tux
/(   )\
 ^`~'^ 
```

# About

okd_bare_metal 🄯 BSD-3-Clause  
Eduardo Lúcio Amorim Costa  
Brazil-DF  
https://www.linkedin.com/in/eduardo-software-livre/

![Brazil](./images/brazil.png)

# Donations

I'm just a regular everyday normal guy with bills and family.

This is an open-source project and will continue to be so forever.

Please consider to deposit a donation through PayPal...

[![Donation Account](./images/paypal.png)](https://www.paypal.com/donate/?hosted_button_id=TANFQFHXMZDZE)

**Support free software and my work!** ❤️👨‍👩‍👧🐧