---
layout: post
title: TryHackMe - Crack The Hash Walkthrough
tags: [Password Cracking, MDXfind, Crack The Hash]
---

Since I've been working on the MDXfind bible the last couple of days, I'm putting all that good work to use and writing up the solutions to the "Crack The Hash" room on tryhackme.com. 

# Level 1

The aim of the game here is to demonstrate how useful running MDXFind is for identifying hashtypes and running mixed hash types at the same time. 

**Task 1 Solution:**

```
cat mixedhashes.list | ./mdxfind.osx -h 'ALL' -h '!salt,!user,!mdx5' rockyou.txt | ./mdsplit.osx mixedhashes.list 
```

```
Done - 4 threads caught
14,344,391 lines processed in 69125 seconds
207.51 lines per second
69124.61 seconds hashing, 6,968,842,180 total hash calculations
0.10M hashes per second (approx)
1 total files
1 MD5x01 hashes found
1 SHA1x01 hashes found
1 SHA256x01 hashes found
3 Total hashes found
3 result lines processed, 3 types found
MD5x01 SHA1x01 SHA256x01 
mixedhashes.list had 3 hits

Total 3 hashes found
```

So with a single run of MDXfind, we were able to crack three of the hashes in one go. Not the fastest run time in the world, but we let MDXfind do the hard work of figuring out the hashes, which are as follows.
 
``
ls -lah mixedhashes.*
``

___
```
-rw-r--r--  1 user  user    38B 29 Sep 14:57 mixedhashes.MD5x01
-rw-r--r--  1 user  user    53B 29 Sep 14:57 mixedhashes.SHA1x01
-rw-r--r--  1 user  user    73B 29 Sep 14:57 mixedhashes.SHA256x01
-rw-r--r--  1 user  user    35B  1 Oct 17:09 mixedhashes.list
```
```
cat mixedhashes.MD5x01 mixedhashes.SHA1x01 mixedhashes.SHA256x01
```

```
48bb6e862e54f2a795ffc4e541caed4d:easy
cbfdac6008f9cab4083784cbd1874f76618d2a97:password123
1c8bfe8f801d79745c4631d09fff36c82aa37fc4cce4fc946683d7b336b63032:letmein
```
---
You'll notice that we still have two hashes left to crack. 

```
$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom
279412f945939ba78ce0758d3fd83daa
```
Lets deal with the hash starting ``$2``. 
One of the quickest way to identify a hash, is by its length and composition. As a starter for 10, lets identify how many characters it is. 

```
echo '$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom' | wc -c
```

Which returns ``61``. The next thing for us to do is find any example hashes that are that length, and prefixed with ``$2``. 


The [Hashcat Example Hashes Page](https://hashcat.net/wiki/doku.php?id=example_hashes) has plenty of references to known hash structures and types, and a quick find in the page for ``$2`` gives us the following. 


| Hash-Mode|Hash-Name|Example|
| -------- | -------- | -------- |
| 3200|bcrypt $2*$, Blowfish (Unix)|$2a$05$LhayLxezLhK1LhWvKxCyLOj0j1u.Kj0jZ0pEmm134uzrQlFvQJLF6 |


Judging by the length and construction it looks like a suitable fit candidate, so we will instruct MDXfind to use the bcrypt algo only. This can be achieved by place ``^`` and``$`` around the algo. 

```
cat mixedhashes.list | ./mdxfind.osx -h '^bcrypt$' -h rockyou.txt | ./mdsplit.osx mixedhashes.list 
```

![tooooolong]({{ site.url }}/assets/img/Oneeternitylater.jpg)



