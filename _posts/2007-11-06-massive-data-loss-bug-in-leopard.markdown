---
layout: post
title: Massive Data Loss Bug in Leopard
tags:
- leopard
- mac
- os-x
linkblog: http://tomkarpik.com/articles/massive-data-loss-bug-in-leopard/
---

I've been reading about this in a few places now.

> Leopard's Finder has a glaring bug in its directory-moving code, leading to horrendous data loss if a
> destination volume disappears while a move operation is in action. I first came across it when Samba
> crashed while I was moving a directory from my desktop over to a Samba mount on my FreeBSD server.

This is why you should __NEVER__ just move data from one drive to another - copy it first, only then delete
the original.
