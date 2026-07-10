---
title: "A Short Comment on Idempotence"
date: 2026-07-10T12:57:56+02:00
draft: false

tags: ["GitOps", "REST", "API"]
author: "Fabricio Jiménez Morales"

description: "I've found the same concept in the wild several times."
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

If you've read the tech news in the last few weeks, chances are you've encountered something along the lines of "HTTP adds the QUERY method". You can go take a look at why that's important, but I want to stop for a second on one of the properties of this method: all articles will say that QUERY is safe and _idempotent_. What does this fancy-sounding word mean?

In one sentence: idempotence is the property of an operation that has the same effect whether applied once or multiple times.

But I first heard about idempotent operators in algebra lectures. I was learning about vector spaces and inner products. At some point, the idea of a projection appears. One rough example of a projection is taking a photo: you take 3D space and project it onto paper or a screen, which are 2D. But what happens if you take a photo of the photo (with a perfect camera, of course)? Well, you get the same photo. If you take a photo of a photo of the photo? You get the point. Projections are idempotent.

This concept is used in computer science, and we see it down the line in methods found in web protocols, or even in GitOps. But what's the point of it all? Who cares if performing an action is idempotent or not? Well, imagine if that action means "send money". If the request fails halfway through and we try again, we probably don't want to send the money twice. In many cases, we *need* idempotent operations.

In the case of HTTP, the protocol needed (well, deserved) a standardized way of performing a complex queries with request content without using POST, a method that is not inherently idempotent. Now we have QUERY.

In GitOps, we find another example of an idempotent process: the reconciliation loop. The desired state of the system is stored (and versioned) in a Git repo, and the CD system continuously reconciles the actual state against it. We can also manually trigger reconciliation, whether changes have arrived or not. And, as you may have guessed, we want this process to be idempotent: once the desired state has been reached, reconciling again should leave the system in that same state.
