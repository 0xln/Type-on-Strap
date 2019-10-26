---
layout: post
title: TryHackMe - Crack The Hash Walkthrough
tags: [Password Cracking, MDXfind, Crack The Hash]
---

Since I've been working on the MDXfind bible the last couple of weeks, I'm putting all that good work to use and writing up the solutions to the "Crack The Hash" room on tryhackme.com. 

# Level 1

The aim of the game here is to demonstrate how useful running MDXFind is for identifying unknown hashtypes and cracking list of mixed password hashes. First, lets create a file containing all the Level 1 hashes. 

**Solution:**

```
cat mixedhashes.list | ./mdxfind.osx -h 'ALL' -h '!salt,!user,!mdx5' rockyou.txt | ./mdsplit.osx mixedhashes.list 
```

```
Done - 4 threads caught
14,344,391 lines processed in 11807 seconds
1214.91 lines per second
11807.49 seconds hashing, 6,968,842,180 total hash calculations
0.59M hashes per second (approx)
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

Which returns ``61``. The next thing for us to do is find any example hashes that are roughly that length, and prefixed with ``$2``. 

The [Hashcat Example Hashes Page](https://hashcat.net/wiki/doku.php?id=example_hashes) has plenty of references to known hash structures and types, and a quick find in the page for ``$2`` gives us the following. 


| Hash-Mode|Hash-Name|Example|
| -------- | -------- | -------- |
| 3200|bcrypt $2*$, Blowfish (Unix)|$2a$05$LhayLxezLhK1LhWvKxCyLOj0j1u.Kj0jZ0pEmm134uzrQlFvQJLF6 |


Judging by the length and construction it looks like a suitable candidate, so we will instruct MDXfind to use the bcrypt algo only. This can be achieved by placing ``^`` and``$`` around the algo. 

```
cat mixedhashes.list | ./mdxfind.osx -h '^bcrypt$' -h rockyou.txt | ./mdsplit.osx mixedhashes.list 
```

![tooooolong](https://0xln.pw/assets/img/Oneeternitylater.jpg)

```
Done - 4 threads caught
14,344,391 lines processed in 124187 seconds
115.51 lines per second
124186.42 seconds hashing, 173,927 total hash calculations
0.00M hashes per second (approx)
1 total files
1 BCRYPTx01 hashes found
1 Total hashes found
1 result lines processed, 1 type found
BCRYPT 
mixedhashes.list had 1 hits
```
```
cat mixedhashes.BCRYPT 
$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom:bleh
```
That leaves one hash left to solve...

```
279412f945939ba78ce0758d3fd83daa
```

---

Since we known we have to use rockyou to crack the hashes, and we have already run all known MDXfind algorithms already, lets try this again, this time adding some rules. This is enabled by using the ``-r`` flag. 

```
cat mixedhashes.list | ./mdxfind.osx -h 'ALL' -h '!user,!salt,!mdx5' -r best64.rule rockyou.txt | ./mdsplit.osx mixedhashes.list
```

And after a short while, we've got a hit. 

```
Working on rockyou.txt, w=124, line 190945, Found=1
Working on rockyou.txt, w=124, line 190945, Found=1
Working on rockyou.txt, w=124, line 190945, Found=1
Working on rockyou.txt, w=124, line 190945, Found=1
```

I really don't feel like waiting for the other 14153446 lines (not including rules) for this job to complete, so the easiest way to see the output that has been fed into mdsplit, is to kill the MDXfind process. 

```
ps -ax | grep mdxfind
```
```
31163 ttys003  4753:28.31 ./mdxfind.osx -h ALL -h !user !salt !mdx5 -r.....
```
So lets kill the MDXfind process so ``mdsplit`` can do its thing.

```
sudo kill -9 31163
```
As the process terminates, all found hashes are dumped into mdsplit to be processed. 

```
1 result lines processed, 1 type found
MD4x01 
```
```
cat mixedhashes.MD4x01 
279412f945939ba78ce0758d3fd83daa:Eternity22
```

And bingo, the final hash is cracked and Level 1 is complete.
