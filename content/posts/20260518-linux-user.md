---
title: "My First and Best Linux Lesson"
date: 2026-05-18T12:09:25+02:00
draft: false

tags: ["linux", "sysadmin", "container"]
author: "Fabricio Jiménez Morales"

description: "Creating a new user in Linux by hand can teach you more than anything else if you're a beginner"
showToc: true
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

About a half a life ago, I had my first Linux lesson, and the best so far. I was a freshman at uni and started learning Linux at [Laboratorio Docente de Computación](https://www.ldc.usb.ve). After a few weeks there, the first important homework was a seemingly simple one: create a new user (your user) in a Linux machine using the terminal.

But there's a catch (see what I'm doing here?): **no** `adduser` or `useradd` scripts allowed. I could bring my notes but not my laptop.

At the time, our reference was [The UNIX System Administration Handbook, 3rd Edition](https://www.oreilly.com/library/view/unix-system-administration/9780137002740/), which already a few years old at the time. The answer was actually there in Chapter 6, but deciphering all that took about a day per page. People at the lab were half friendly, half observant on how you took on the challenge, but without a doubt this homework let me figure out so many things, to name a few:
- Basic commands: bash, filesystem navigation, file inspection and manipulation.
- VIm basics: opening, navigating, editing, saving a file.
- UNIX users basics: permissions, root vs. non-root users, groups, the `/etc/passwd` and `/etc/shadow` files and the `passwd` command.

When you felt ready, you would come to a staff member of the lab and they'd give you a root shell on the machine and supervise your steps. The idea is that you knew what you were doing before running anything you do, from running a command, to the meaning of an information typed in a file. In hindsight, the amount of information contained in all steps was pretty massive; most of it was foundational Linux knowledge and some of it ended up in what would become funny anecdotes after some time; for example, I remember pondering for a couple of days what would be the right way of filling the GECOS field (mostly historical and virtually irrelevant), or which would be the best shell interpreter among sh, ksh, csh or tcsh (me distinctly skipping bash, which was the standard).

Creating a user could have been an unremarkable interaction with `useradd` but it was instead an enriching experience. Everyone I knew at the lab felt this power of having touched the inside of Linux after creating a user.

To do justice to my past self, here's how to create a new user in a Docker container:

Spin up a `busybox` container:
```bash
docker run -it --name c-user busybox sh
```

Inside the container, add entries for user (and group), create home directory, change ownership and permissions, modify password and check:
```bash
# Instead of the two commands below I used vim at the time
echo 'foo:x:88:88:foo:/home/foo:/bin/sh' >> /etc/passwd
echo 'bar:x:88:' >> /etc/group

mkdir -p /home/foo
chown foo:bar /home/foo
chmod 700 /home/foo

passwd foo # Enter password twice

su - foo
whoami # foo
```

If we detach from the container (`CTRL+P` then `CTRL+Q`) we can run from outside:
```bash
docker exec -u foo c-user sh -c 'echo Hello from $(whoami)' # foo
```

---

Of course, managing users across multiple hosts and services is no longer done this way. Even back then, the lab relied on centralized authentication services (i.e. LDAP) and local users were mostly a pedagogical exercise.

Today, identity and access management are handled through much more sophisticated systems, often integrated with cloud platforms and centralized authorization policies. But the underlying ideas are still there: users, groups, ownership, permissions, and the question of who is allowed to do what on a system. Because, in the end, almost everything either is or runs in Linux.

The bottom line is: if I were to teach Linux to someone, I would start by telling them to figure out how to create a new user. It is a great first example of learning beyond the abstraction layer. And even if it sounds like bait, surprisingly many engineers do not know how to do it.
