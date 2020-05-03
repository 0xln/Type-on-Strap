---
layout: post
title: TryHackMe - Crack The Hash Walkthrough Level 2
tags: [Password Cracking, MDXfind, Crack The Hash]
---

This is a follow up to my original post for level one, giving solutions to the "Crack The Hash" room on tryhackme.com. This time, it's Level 2.

# Level 2

Again, we are going to demonstrate how useful running MDXFind is for identifying unknown hashtypes and cracking list of mixed password hashes. First, lets create a file containing all the Level 2 hashes.

**Solution 1 & 2:**

```
cat mixedhashes.list | ./mdxfind.osx -h 'ALL' -h '!salt,!user,!mdx5' rockyou.txt | ./mdsplit.osx mixedhashes.list
```

Before we even got off the mark, MDXfind has identified one of the hashes

```
Reading hash list from stdin...
Took 0.00 seconds to read hashes
Searching through 3 unique hashes from <STDIN>
Searching through 1 unique SHA512CRYPT hashes
```
Lets park that nugget of information right now, and continue running the above.


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

So 2 hashes found, and three hits.

**NB** - `MD4UTF16` and `NTLM` are effectively the same in MDXfind.


``
ls -lah mixedhashes.*
``

___
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
**Solution 3 & 4:**

We already had a clue from MDXfind that one of the hashes was `SHA512CRYPT`. Because the algorithmn is salted, and we specified not to cracked salted hashes (`-h '!salt,!user,!mdx5'`), it didn't attempt it.

Since we know the algorithmn, we can specify it directly and run it again.

```
cat mixedhashes.list | ./mdxfind.osx -h '^SHA512CRYPT$' -h rockyou.txt |S ./mdsplit.osx mixedhashes.list
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
```
cat mixedhashes.SHA512CRYPT
$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.:waka99
```
One to go, and this one is salted with `tryhackme`.

```
e5d8870e5bdd26602cab8dbe07a942c8669e56d6
```

Salted 40 character hash? Hummm. SHA-1 is pretty common for this length, so lets try SHA-1 with a salt file.
We can run salted hashes using the `-s` switch, and removing `-h !salt` from the algorithm switch. Lets go again.  

---

```
cat mixedhashes.list | ./mdxfind.osx -h '^sha1' -s salts.txt rockyou.txt | ./mdsplit.osx mixedhashes.list
```

Our hunch was spot on.


```
1 result lines processed, 1 type found
MD4x01
```
```
cat mixedhashes.MD4x01
279412f945939ba78ce0758d3fd83daa:Eternity22
```

And bingo, the final hash is cracked and Level 2 is complete.
