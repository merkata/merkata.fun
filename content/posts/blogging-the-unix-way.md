---
title: 'Blogging the Unix Way'
date: '2026-11-13'
# weight: 1
# aliases: ["/first"]
tags: ["blogging"]
author: "Mariyan Dimitrov"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "KISS or: How I Learned to Stop Worrying and Love Blogging"
canonicalURL: "https://merkata.fun/to/blogging-the-unix-way"
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---
## TL;DR Simply writing is better than perfecting the frame

I started this blog with the idea to just write, however some things got in the
way early on. Being technical, I really wanted to have a nice setup. I have
personal and work related [Obsidian](https://obsidian.md/) vaults, so I went
[exploring](https://jasonheppler.org/2026/03/05/the-obsidian-hugo-workflow/)
[how](https://santacloud.dev/posts/creating-my-blog---a-developers-tale-of-over-engineering-using-obsidian-hugo-and-github-pages/#dead-eyed-tutorial)
to
[setup](https://dhewy.dev/posts/how-i-built-this-blog-using-hugo-obsidian-aws-and-claude/)
Obsidian with hugo in a git folder with automated commit and push. I was quickly
focused on having a perfect setup and got distracted from actually writing. I
felt overwhelmed and blogging was on hold for a while.

In the past few days I revisited the idea of blogging and approached it more
simplistically. I've always appreciated simplicity in software and in
engineering, always have been fascinated by the UNIX way of doing things. I
ended up with the minimal and most comfortable way (for me) to blog: git repo ->
vim -> commit and push -> post deployed on website.

Outside of Obsidian, I normally live in [(n)vim](https://neovim.io/). Whatever I
work on is in version control. It is clutter free and the flow reuses my muscle
memory.

Whenever I want to write something, I create a new post with `hugo new
content/post.md` and then `vim content/post.md` for as long as I like. When I
feel like there's nothing to add at the moment, I `git add content/post.md &&
git commit -m 'docs: New post'` and "git push". That flow is basically the same
as my usual work flow of `vim <file>;<tool>; git commands`, where file is some
"infra.tf" or "manifest.yaml" usually and tool is "terraform", "kubectl" or
something similar.

## Breakdown

Here is some more info on the whole setup.

### Content

I am using [Hugo](https://gohugo.io/) to generate the static content with
minimal customizations. I found a theme that I like and configured automatic
titles and publish dates when doing `hugo new content/blog-post.md --kind
post`. I don't think I need the customizations, but I created that post kind
when I was messing with that complicated Obsidian setup. Important thing is now
I have what I want and I learned a bit about customizing Hugo.

### Repo

Hugo configuration and content all live in one repo -
github/merkata/merkata.fun. I can commit, revert, go back and review what I was
experimenting with. Nothing fancy, plain ol' git. Two additional benefits are
that I can write from my personal and work laptops and hugo can make use of the
git metadata (which I use currently for setting the last updated date for a
post).

### Editor

I've been using vim for over two decades now. My current setup is nvim with
[lazyvim](https://www.lazyvim.org/) and I am a happy basic user, I don't have
to configure stuff most of the time and I am happy with the defaults.

### CI/CD

The CI/CD is a github workflow that will build the site and rsync the public
folder to my VPS. It relies on an ssh key and known hosts that are configured
as repository variables. The workflow is basic enough that it is not high
maintenance and does well what it is supposed to do - build and deploy the
site.

### Site

The site is hosted on a VPS at [Hetzner](https://www.hetzner.com/), it is
served via [caddy](https://caddyserver.com/) and the
[Caddyfile](https://caddyserver.com/docs/caddyfile) is also pretty basic (it is
just the site and some [Matrix](https://matrix.org/) related configs). There is
a user restricted to only rsyncing the public folder and nothing else. Caddy
and the Caddyfile syntax make serving the site pretty easy and it just works,
so I can focus on writing.
