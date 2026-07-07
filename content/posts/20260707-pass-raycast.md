---
title: "My Password Workflow with Raycast and Pass"
date: 2026-07-08T00:22:12+02:00
draft: false

tags: []
author: "Fabricio Jiménez Morales"

description: "I don't like browser password managers. I use pass and Raycast, and it's smooth."
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

My current workstation runs macOS, and I SSH into my Linux homelab. I use [Raycast](https://www.raycast.com/) as an application launcher and a general productivity tool. Since I believe this hybrid setup (macOS + Linux) is fairly common, maybe what follows will be of use to you.

I'm not a big fan of browser password managers, so I use `pass` instead. So far, so good. But there are two technical details that make my life easier: syncing passwords among hosts and having a keybind for quick access to them when I'm not in the terminal.

## Pass basics

`pass` is the standard UNIX password manager. It's simple: [it does one thing and does it well](https://en.wikipedia.org/wiki/Unix_philosophy). Passwords are stored in GPG-encrypted files, organized in a directory hierarchy (by default, under `~/.password-store`).

We need a GPG key to use this tool, so make sure to have the `gnupg` package installed:

```bash
gpg --full-generate-key
gpg --list-secret-keys --keyid-format LONG
pass init <gpg-key-id>
```

We can interact with the password store easily:

```bash
pass insert mail/gmail # Use '/' just like in a regular UNIX directory
pass insert chat/aol # :-)
pass edit chat/aol # Edit an entry: we can add further lines with extra info
pass # See the entries in the password store
pass show mail/gmail # Show a password
```

You can check out [this Dreams of Code video](https://www.youtube.com/watch?v=FhwsfH2TpFA) on `pass`.

We also set up the `~/.gnupg/gpg-agent.conf` file:

```bash
pinentry-program /usr/bin/pinentry-curses
default-cache-ttl 3600      # 1 hour
max-cache-ttl 28800         # 8 hours
```

The optional last two lines mean that the passphrase will be cached for an hour after each use and for a total of eight hours.

## Integration with Git

Something I like about `pass` is its integration with Git. The password store can be a Git repository and pushed to a (private!) remote repo, e.g. on GitHub. If you're familiar with Git basics, this will all make sense:

```bash
pass git init
pass git remote add origin git@github.com:$GH_USER/$PASSWORD_STORE_REPO.git
pass git push --set-upstream origin master
```

Even if we use a private repo and the passwords themselves are GPG-encrypted, the filenames and directory structure are not. That is, if someone managed to read the repository, they could see that an entry called `mail/gmail.gpg` exists, but not its contents. So, there's a risk of metadata leakage there.

The `pass` commands for adding and editing entries listed above integrate with Git and commit the changes. I then push those changes manually.

## Transferring the GPG Key to the Mac

Imagine that the passwords above were created on a Linux host, and now we want to fetch the store onto our Mac. We'll need to `git pull`, but before being able to read (decrypt) the passwords, we need to copy the GPG key used to encrypt the store in the first place to the Mac—let's do that first.(It's difficult to manually type an em-dash these days.)

Find the identity of the key, `<gpg-key-id>`, the same one we used above:

```bash
gpg --list-secret-keys --keyid-format LONG
```

Export the private key and the ownertrust database:

```bash
gpg --armor --export-secret-keys <gpg-key-id> > pass-private-key.asc
gpg --armor --export-ownertrust > ownertrust.txt
```

Then copy the files to the Mac.

On the Mac side, make sure to install the necessary packages:

```bash
brew install gnupg pass pinentry-mac
```

Then import the key and ownertrust:

```bash
gpg --import pass-private-key.asc
gpg --import-ownertrust ownertrust.txt
```

Finally, we set up the `~/.gnupg/gpg-agent.conf` file on the Mac:

```bash
pinentry-program /opt/homebrew/bin/pinentry-mac
default-cache-ttl 3600
max-cache-ttl 28800
```

Then restart the agent:

```bash
gpgconf --kill gpg-agent
gpgconf --launch gpg-agent
```

## Password Store on the Mac

Clone the password store repo:

```bash
git clone git@github.com:$GH_USER/$PASSWORD_STORE_REPO.git ~/.password-store
```

Check that we're able to see the store and decrypt an entry:

```bash
pass ls
pass show mail/gmail
```

If everything works fine, we can now remove the temporary exported key and ownertrust files:

```bash
rm pass-private-key.asc ownertrust.txt
```

## Quick and dirty hack for propagating new passwords

As I said above, passwords added or edited with `pass` integrate with Git, but we still need to push the changes after modifying the store. I added this snippet to my `.zshrc` so that every time I log in, it updates the local password store:

```zsh
if command -v pass >/dev/null 2>&1 && [ -d "$HOME/.password-store/.git" ]; then
    pass git pull --ff-only --quiet
fi
```

This is certainly not the most elegant way of doing it, and it adds a tiny but noticeable amount of time to the shell startup. Perhaps it'd be a good idea to define a user timer via a systemd service. But I'll leave that for another opportunity.

## The Raycast Setup

This is straightforward. Install [the `pass` extension](https://www.raycast.com/capipo/pass) from the Raycast extension store. In the extension settings, set the Homebrew path to: `/opt/homebrew/bin`. I also added a keybinding for quick searching/pasting the passwords. And that's it: I keep now my passwords in more than one place, with a smooth experience when I need to input my passwords in a browser, without using a browser password manager.
