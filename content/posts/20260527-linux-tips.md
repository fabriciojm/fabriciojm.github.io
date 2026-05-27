---
title: "Two Linux Security Tips"
date: 2026-05-27T16:23:11+02:00
draft: false

tags: ["linux", "sysadmin", "container", "security"]
author: "Fabricio Jiménez Morales"

description: "Two small habits to improve Linux security"
showToc: false
TocOpen: false

ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true

cover:
  image: ""
  alt: ""
  caption: ""
  relative: false
  hidden: true
---


Security is one (if not _the_) main priority of many Linux administrators, and now more than ever this is the case in the context of the large number of systems we typically manage today: containerized systems running in clusters, cloud microservices, ephemeral environments, and so on.

We talk a lot about the essentials: good password, key and secrets management, firewall policies, encryption, and similar topics; which is obviously critical. But I also believe that threats can be mitigated through small habits, in the commands we do daily.

Here are two small practices I’ve adopted over the years that I think are especially relevant for Linux users and, of course, for anyone working in cloud, infrastructure, or DevOps.

# Use absolute paths for password-related commands

The first practice is invoking sensitive commands through their absolute paths. Think about `/bin/su` instead of `su` and `/usr/bin/ssh` instead of `ssh`, instead of relying blindly on `$PATH`. The reason is simple: command lookup itself can become part of the attack surface.

If an attacker manages to compromise a user account (but not root), they may be able to place a fake `ssh`, `scp`, or similar executable earlier in the victim’s `$PATH`, or alias those commands in shell configuration files. A convincing fake password prompt can be enough to capture credentials. Using absolute paths bypasses that entire class of tricks.

This is, of course, not something most individual Linux users should worry too much about. But for people who regularly administer remote systems, clusters, or production infrastructure, I think it is a reasonable habit to build.

I even have [shortcuts in my `.inputrc`](https://github.com/fabriciojm/dotfiles/blob/5aaa6f8b3991dc506c9013e1b00a33b7fbfe80ad/inputrc) for the commands I use most often, so the absolute paths are inserted automatically.

# `sudo` > `sudo -i` > `sudo su`

Some people escalate privileges with:

```bash
sudo su
```

but in most cases this is unnecessary, because Linux already provides `sudo` with  a more direct way to obtain a root login shell, if needed:

```bash
sudo -i
```

Depending on the use case, `sudo -s` or simply `sudo <command>` may be more appropriate. (Just ironymaxxing here, no absolute paths in blog code blocks 😉)

Using `sudo su` adds an extra layer: `sudo` launches `su`, which then launches a shell. In practice, there is usually little benefit over `sudo -i`. Beyond that, auditability and accountability are also part of the discussion. One of the strengths of `sudo` is that privileged actions are associated with a specific sudo-capable user.

However, once administrators routinely jump into long-lived root shells, attribution becomes weaker. Inside the shell, commands are simply being executed as `root`, and distinguishing who did what afterwards becomes harder without [additional auditing tooling](https://man7.org/linux/man-pages/man8/auditctl.8.html).

We can see it in a little demo here. Start a Debian container:

```bash
docker run -it debian:stable bash
```

Install `sudo` and [create two users](https://fabriciojm.github.io/posts/20260518-linux-user/), Alice and Bob:

```bash
# Install sudo and syslog
apt update && apt install -y sudo rsyslog
# Add users, assign their passwords (same as user names),
# give them sudo privileges
useradd -m alice
useradd -m bob
echo 'alice:alice' | chpasswd
echo 'bob:bob' | chpasswd
usermod -aG sudo alice
usermod -aG sudo bob
# Turn on logging daemon
rsyslogd -n -iNONE &
```

Now if Alice logs in (`docker exec -it --user alice <container-name> bash`) and performs an action with sudo:

```bash
sudo touch /root/alice-file
```

The logs will show that Alice executed exactly that command; in `/var/sys/auth.log`:

```
alice : ... USER=root ; COMMAND=/usr/bin/touch /root/alice-file
```

But if Bob logs in and, on the other hand, does:

```bash
sudo su - # password
touch /root/privileged-user-file{1..3}
rm /root/privileged-user-file2
echo "foobar" > /root/privileged-user-file3
```

the logs mainly show that Bob opened a root shell.
```
bob : ... USER=root ; COMMAND=/usr/bin/su -
```
The individual commands executed afterwards are no longer clearly attributed through `sudo` itself.

To be rigorous, the same limitation also applies to `sudo -i`: once you are inside a root shell, attribution naturally becomes weaker. So the rule of thumb is: use sudo as much as possible and, if unavoidable get a root shell via `sudo -i`.

---

Perhaps the broader lesson here for cloud and DevOps practitioners is that security is not only about “big” security tooling. Infrastructure work is full of little repeated actions: logging into servers, escalating privileges, copying files, running automation, and jumping between environments.
