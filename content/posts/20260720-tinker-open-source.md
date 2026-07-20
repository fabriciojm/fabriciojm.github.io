---
title: "Tinker then Contribute to Open Source"
date: 2026-07-20T14:55:54+02:00
draft: false

tags: ["containerlab", "netwokring", "GitOps", "DevOps", "Open Source"]
author: "Fabricio Jiménez Morales"

description: "Open source contributions are small and incremental, and they can come from anyone. Here's one example."
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


Open source contributions need not be great achievements. Actually, most of them are not, and that's the way it's supposed to be. In this paradigm, projects get better simply by being public and having people with different levels of expertise and involvement contribute to them.

I recently contributed to a project that has allowed me to learn a good amount about networking.

My contribution was eight lines of code.

I admit that even if [networking](https://fabriciojm.github.io/tags/networking/) is something I already have some experience with and have been expanding my skills in recently, I don't claim to be well versed in that specific project. Also, coming from a science background, I am fluent in Python and C++, and can understand, but not write much, Go (yet 😉). But I think that's exactly the whole point of an open source contribution: we don't need to be experts or hardcore programmers in the language a project is written in if we want to contribute. What we do need is the willingness to contribute, a problem to solve (or a feature to add), and the systems thinking needed to integrate it without breaking anything else (your PR won't be merged otherwise anyway). This is even more true with the help of AI coding agents.

I was recently tinkering with [containerlab](https://containerlab.dev/). It provides a CLI for orchestrating container-based networking labs. Through it, users can create different lab topologies by starting and [wiring containers together]({{< relref "posts/20260623-container-networking-by-hand" >}}), while also managing the network lifecycle. In practice, it provides a single Infrastructure as Code interface for managing labs with arbitrarily meshed network topologies.

Just like with plain containers, the learning possibilities grow a lot because you can explore, make and break things, and demo a full network running NOSs. But it goes beyond that: since network topologies (routers, switches, links, services) are specified in code, everything fits within the GitOps paradigm.

Networks are inherently visual: we draw the nodes, connections, and labels in order to understand them. Naturally, containerlab has a way to produce diagrams from topology specification files. A simple way to do that is via the CLI: `clab graph -t <topology-filename> --dot`, which generates a DOT file that's then rendered into a PNG by Graphviz.

And that's exactly where I found room for improvement: that graph generation was broken, and it was pretty easy to diagnose. The name of the generated top-level graph was taken from the topology filename by removing only the `.yaml` extension, meaning that the name still ended in `.clab`. DOT graph identifiers, however, cannot contain characters like `.` or `-` unless they are quoted.

The fix itself was pretty simple. In just a few lines of code, I quoted the top-level graph name. And just like that, I sent [my PR](https://github.com/srl-labs/containerlab/pull/3202), and it got merged into a project I really like.

One feature of containerlab that the developers have invested a lot of effort into recently is [its GUI](https://containerlab.dev/manual/gui/), especially the VS Code extension. One of the main developers has [a video presenting it](https://www.youtube.com/watch?v=NIw1PbfCyQ4&t=41s). I admit I haven't played around with it much because I try to stay inside my terminal as much as I can, using Vim. Still, it seems like a really nice interface that lowers the barrier to getting started with containerlab: with just a few clicks, one can install the extension, install containerlab itself, and deploy network topologies without needing to remember the CLI commands. If you're interested, I'd suggest taking a look.

Regarding my experiments with containerlab, I'll write a dedicated post about them in the future.
