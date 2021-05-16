---
layout: post
title: TryHackMe - Crack The Hash Walkthrough Level 2
tags: [Password Cracking, MDXFind, Crack The Hash, tryhackme]
---

This is a follow up to my original post for level 1, giving solutions to the "Crack The Hash" room on [tryhackme.com](https://tryhackme.com). 
This time, it's level 2.

# Level 2 Walkthrough

Again, we are going to demonstrate how useful running MDXFind is for identifying unknown hashtypes and cracking lists of mixed hashes. First, lets create a file containing all the level 2 hashes. 

---


### Tools

If you don't already have a copy, you can grab it from [Hashes.org](https://hashes.org/mdxfind.php). In this walkthrough I'm also using mdsplit to remove solved hashes from the original list as we crack them. I'm not going to directly cover it in this post, but more details on how to use mdsplit can be found in my [MDXFind bible](https://0xln.pw/MDXfindbible) post.



## Solution 1 & 2:

As all the solutions to the hashes can be found in the well known rockyou wordlist, lets run all the challenge hashes though MDXFind with that wordlist, and see what we get.  

#### Command

```
cat mixedhashes.list | ./mdxfind.osx -h 'ALL' -h '!salt,!user,!mdx5' rockyou.txt | ./mdsplit.osx mixedhashes.list 
```

Almost immediately, MDXfind has identified one of the hashes.


#### Results
```
Reading hash list from stdin...
Took 0.00 seconds to read hashes
Searching through 3 unique hashes from <STDIN>
Searching through 1 unique SHA512CRYPT hashes
```

Lets park that nugget of information right now, and continue running the command above. 

```
Done - 4 threads caught
14,344,391 lines processed in 10697 seconds
1340.97 lines per second
10696.98 seconds hashing, 6,968,842,180 total hash calculations
0.65M hashes per second (approx)
1 total files
1 SHA256x01 hashes found
1 NTLMx01 hashes found
1 MD4UTF16x01 hashes found
3 Total hashes found
3 result lines processed, 3 types found
MD4UTF16x01 NTLMx01 SHA256x01 
mixedhashes.list had 2 hits

Total 2 hashes found
```
So 2 hashes found, 2 files output and types found 🤔
 
**NB** 
-`MD4UTF16` and `NTLM` are effectively the same in MDXFind hence three types being found, but only two actual results. 

A quick `ls` and `cat` of the files MDXFind produced will make this a bit clearer.

___
``
ls -lah mixedhashes.*
``

```
-rw-r--r--  1 NOBODY  staff    46B 23 Apr 14:08 mixedhashes.NTLMx01
-rw-r--r--  1 Nobody  staff    71B 23 Apr 14:08 mixedhashes.SHA256x01
-rw-r--r--  1 Nobody  staff   147B 23 Apr 14:08 mixedhashes.list
```

```
cat mixedhashes.NTLMx01 mixedhashes.SHA256x01
```

```
1dfeca0c002ae40b8619ecf94819cc1b:n63umy8lkf4i
f09edcb1fcefc6dfb23dc3505a882655ff77375ed8aa2d1c13f640fccc2d0c85:paule
```

So that solves quesiton one and two. Two left to crack.

---
## Solution 3:

We already had a clue from MDXFind that one of the hashes was `SHA512CRYPT`. Because that algorithmn is salted, and we specified not to cracked salted hashes (`-h '!salt,!user,!mdx5'`), it identified the hash type, but didn't attempt to crack that algorithm. 

Since we know the algorithmn, we can specify it directly and run the original command again.
 
 
#### Command
```
cat mixedhashes.list | ./mdxfind.osx -h '^SHA512CRYPT$' -h rockyou.txt | ./mdsplit.osx mixedhashes.list 
```

```
0.67M hashes per second (approx)
1 total files
1 SHA512CRYPTx01 hashes found
1 Total hashes found
1 result lines processed, 1 type found
SHA512CRYPT 
mixedhashes.list had 1 hits
```

#### Results

```
cat mixedhashes.SHA512CRYPT 
```

```
$6$aReallyHardSalt$6WKUTqzq.UQQmrm0pT7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.:waka99
```
One to go, and this one is salted with `tryhackme`.

```
e5d8870e5bdd26602cab8dbe07a942c8669e56d6:trackhackme
```
---
## Solution 4:

To save us trying every salted algorithmn available in MDXFind, a quick tap of the hint button indicates that the hash is `HMAC-SHA1`.  

As `HMAC-SHA1` is a keyed hash (not salted as the challenge would lead us to believe), we will need to supply the key that the hash was 'salted' with. 

Two command switches we need to know when cracking salted hashes or keyed hashes are:
 
```
-s      File to read salts from
-u      File to read Userid/Usernames from
```

Since the hint has given us the exact algorithm to run, its just a case of specifying it with the `-h` switch, and adding in the `-u` switch pointing to the file that contains the key `tryhackme`. 

**Footnote:** Normally you would only need the `-s` switch for salted hashes, but since this is actually keyed, we need to use the Userid switch to supply the value it has been keyed with.

---

Lets write the key to a file, and then run MDXfind as before, and include a file containing our key.

``` 
echo "tryhackme" > key.txt
``` 

#### Command
```
cat mixedhashes.list | ./mdxfind.osx -h 'hmac-sha1' -u key.txt rockyou.txt | ./mdsplit.osx mixedhashes.list  
```
---
#### Results

```
1 userids read from key.txt
Working on hash types: HMAC-SHA1 
1 total userids in use
Reading hash list from stdin...
Took 0.00 seconds to read hashes
Searching through 1 unique hashes from <STDIN>
Maximum hash chain depth is 1
Minimum hash length is 40 characters
Using 4 cores
Working on rockyou.txt, w=0, line 14344391, Found=0

Done - 4 threads caught
14,344,391 lines processed in 20 seconds
717219.55 lines per second
20.07 seconds hashing, 14,344,391 total hash calculations
0.71M hashes per second (approx)
1 total files
1 HMAC-SHA1x01 hashes found
1 Total hashes found
1 result lines processed, 1 type found
HMAC-SHA1 
mixedhashes.list had 1 hits
```

```
cat mixedhashes.HMAC-SHA1
```

```
e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme:481616481616
```

And bingo, the final hash is cracked and Level 2 is complete.
