---
layout: page
title: "MDXfind Bible" 
subtitle: "(Work In Progress)"   
feature-img: "assets/img/pexels/computer2.webp" 
permalink: /MDXfindbible               # Set a permalink your your page
hide: False                          # Prevent the page title to appear in the navbar
icon: "fa-book"                    # Will Display only the fontawesome icon (here: fa-search) and not the title
tags: [hashes.org, mdxfind,]
---



## Intro (**WIP**)

This is a work in progress page of tips and tricks related to MDXfind. Mostly collated from using MDXfind and uploading hashes, as well as hassling hops and s3in!c for how to use it....

### Basic Usage Using STDIN

MDXfind's strength is it's ability to run wordlists against lists of mixed hashes or list of unknown hashes. Looks like an MD5, but are you sure that 32hex definately is? Let Mdfind do the hard work for you. When running mixed lists or known algo's, it's always worth running it with mdsplit.

```
cat mixedhashes.list | ./mdxfind -h 'ALL' -h '!salt,!user,!md5x' wordlist.txt | ./mdsplit-32 mixedhashes.list 
```

### Running MDXFind From a File 

Alternatively, if you want to run it from a file instead of STDIN

```
mdxfind -h ALL -h '!salt,!user,!md5x' -f mixedhashes.list wordlist.txt >results.file
```

### Generating Candidates (CHECKDIS)

Somtimes you might get a partial match, or you want to check the plaintext value against it's hashed counter part. 

```
echo -n '0xln.pw' | mdxfind -h ALL -h '!salt,!user' -z -f /dev/null -i 5 stdin 2>&1
```

```
```



#### MDXfind to Hashes.org Format

Now that you've found a bunch of hashes using the tips and instructions above, you might want to upload them to Hashes.org so you can jump up the leader board 😉. The following reference table can be used to help figure out the algo needed to upload to Hashes.org.

|MDXfind Algo|Hashes.org Format|
|------------|---------------|
|MD4x01      | MD4           |
|MD5MD5PASSSHA1x01|MD5(MD5(PLAIN)PLAINSHA1(PLAIN))|
|MD4UTF16MD5x01| NTLMMD5|
|HAV128_4MD5x01| HAVAL128-4MD5|
|HAVA128_4x01|HAVAL128-4|
|MD2MD5x01|MD2MD5|
|MD4MD5x01|MD4MD5|
|MD4SQL3x01|MD4MYSQL3|  
|MD5-1xSHA1MD5pSHA1px01|MD5(SHA1MD5(PLAIN)SHA1(PLAIN))|
|MD5-1xSHA1MD5x01|MD5(SHA1MD5(PLAIN))

### Where To Find

[Download MDXfind](https://hashes.org/mdxfind.php) 