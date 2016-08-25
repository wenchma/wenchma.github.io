---
layout: post
title: "Asciinema: Record and share your terminal session"
categories: tech
tags: asciinema
date: 2016-08-25 17:03:30
---

# What's Asciinema ?

Asciinema is a cool command-line based terminal session recorder, easy to share with javascript player.

# Installation

Tested with Ubuntu:

```bash
sudo apt-add-repository ppa:zanchey/asciinema

sudo apt-get update

sudo apt-get install asciinema
```

More detailed [installation][1]

# Usage

To start recording run the following command:

```bash
asciinema rec
```

This spawns a new shell instance and records all terminal output. When you're ready to finish simply exit the shell either by typing exit or hitting Ctrl-D.

```bash
$ asciinema rec
~ Asciicast recording started.
~ Hit Ctrl-D or type "exit" to finish.
mars@OSEE:~$ sl
mars@OSEE:~$ exit
~ Asciicast recording finished.
~ Press <Enter> to upload, <Ctrl-C> to cancel.

https://asciinema.org/a/5aayecan8c2bfe4gwroquqraj
```

# Sharing

You can get the share link for a specific asciicast by clicking on "Share" link on asciicast page.

<script type="text/javascript" src="https://asciinema.org/a/5aayecan8c2bfe4gwroquqraj.js" id="asciicast-5aayecan8c2bfe4gwroquqraj" async data-autoplay="true" data-size="small"></script>


# Reference

* [Asciinema Home][2]
* [Installation][1]
* [How it works][3]
* [Asciinema Usage][4]

[1]: https://asciinema.org/docs/installation
[2]: https://asciinema.org/docs
[3]: https://asciinema.org/docs/how-it-works
[4]: https://asciinema.org/docs/usage
