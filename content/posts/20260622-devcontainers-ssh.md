---
title: "Dev Contaniners and SSH"
date: 2026-06-22T17:13:24+02:00
draft: false

tags: ["linux", "containers", "dev containers", "ssh", "chezmoi", "mise"]
author: "Fabricio Jiménez Morales"

description: "Development containers don't need private keys explicitly to communicate with GitHub, and I find that pretty neat."

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

Recently, I've learned about development in [dev containers](https://containers.dev/). This idea is great because it allows for siloed, reproducible, full-featured environments for development. Since DevOps is about enabling product work, dev containers are a great example of that.

This idea can actually go further: dev containers can also be used for management, admin, and DevOps tasks, like Kubernetes cluster management. In that case, tools and configuration files are as important as application code. One can have a dedicated management environment for a single cluster, reducing the risk of accidentally targeting the wrong cluster.

The setup I'm currently using is based on [Rio Kierkel’s approach](https://github.com/rio/dotfiles), combining dev containers (managed by [DevPod](https://devpod.sh/)) with [chezmoi](https://www.chezmoi.io/) for dotfile management and templating, and [mise](https://mise.jdx.dev/) for tool installation and environment configuration. This results in a powerful and portable environment. In a sense, the “product” being enabled here is DevOps itself.

One day I’ll write more about how this setup is made. (For now, you can already take a look at [my homelab](https://github.com/fabriciojm/homelab) and [my dotfiles](https://github.com/fabriciojm/dotfiles).) But today I wanted to just point out how to connect the host's SSH agent to a dev container.

I'll state the obvious: it is expected for the dev container to be able to communicate with remote repos. If the host machine is already configured with an SSH key for GitHub, and the key is loaded into an SSH agent (for example using `keychain`):
```bash
eval "$(keychain --eval --quiet ~/.ssh/<key-name>)"
```
**the dev container does not need its own private key**, as it's possible to forward the host's SSH agent socket into the container. That is done by adding to the dev container configuration file, i.e. the `devcontainer.json` file:
```json
"mounts": [
  "source=${localEnv:SSH_AUTH_SOCK},target=/ssh-agent,type=bind"
],
"remoteEnv": {
  "SSH_AUTH_SOCK": "/ssh-agent"
}
```
Here `localEnv` refers to the environment of the machine running the container. The env variable `SSH_AUTH_SOCK`, exported by `keychain` (pointing to the host's `ssh-agent`), exposes only the ssh socket but the private key itself stays on the host. This can, of course, be done for multiple dev containers simultaneously. Pretty neat, I think.
