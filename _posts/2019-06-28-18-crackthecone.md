---
layout: post
title: Crack The Con Writeup 2019
tags: [hashes.org, crackthecon, writeups,  hashcat, Password cracking]
color: green
---

# What i learned, competing in a password cracking contest



You can go fork and star hers too! 😉

## How does it work?

Basically you need to add just one thing, the color:

```yml
---
layout: post
title: Color Post
color: brown
---
```

It can either be a html color like `brown` (which look like red to me). Or with the rgb:

```yml
---
layout: post
title: Color Post
color: rgb(165,42,42)
---
```

## Using Nmap nsec3

``` bash
sudo nmap  -sU -p 53 ns.crackthecon.0x23.pw --script=dns-nsec3-enum --script-args dns-nsec3-enum.domains=starthere.crackthecon.0x23.pw dns-nsec3-enum.timelimit=270 -oA hashv2   -vvvv
```
> 
> 
> nsec3walker-20101223$ ./collect starthere.crackthecon.0x23.pw
> 
> servers = ["35.247.112.179"]
