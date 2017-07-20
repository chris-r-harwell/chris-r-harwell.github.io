---
layout: post
title: Viewing markdown in the terminal with Pandoc and lynx.
tags: markdown md terminal Pandoc lynx
---


All of a sudden I am encountering markdown more than I did before.  Sure it has gained in popularity, but the courses I am taking are providing class schedules, some lecture notes and assignments in markdown as well.  Jekyll uses markdown for blogs.  Many open source projects use a README.md for the first page.  Anyway sometimes I just want to view this from the terminal with opening a gui web browser.  OK, yes I know this puts me in a small minority, but whatever. Use [Pandoc](https://pandoc.org/) and [pipe](https://en.wikipedia.org/wiki/Peter_H._Salus) the output to [lynx](http://lynx.browser.org/). Here it is:


```
pandoc READme.md | lynx -stdin
```

If you should get a file not found for Pandoc or lynx you can typically install them with your distribution's package manager something like:
```
sudo dnf install -y pandoc lynx
```

P.S, I was asked recently in an interview what the pipe does in Unix. Wow had I been waiting on that question for years.  I happen to have met Peter Salus at a Usenix conference in Boston and got his book on the history of Unix. Yep, he invented/suggested pipes where you can chain together commands by hooking up the output from one command to the input of another. Nice.
