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

MDXfind's strength is it's ability to run wordlists against lists of mixed hashes or lists of unknown hashes. Got a bunch of 32hex hashes but no idea what they are? Let Mdfind do the hard work for you. When running mixed lists or known algo's, it's always worth running it with mdsplit to save the hassle of passing it out later

```
cat mixedhashes.list | ./mdxfind.static -h 'ALL' -h '!salt,!user,!md5x' wordlist.txt | ./mdsplit.static mixedhashes.list 
```
This command runs a mixed file of hashes through all known algorithms in MDXfind apart from those requiring salts, usernames and skipping iterations like `MD5(MD5(plain))`. 

### Running MDXFind From a File 

Alternatively, if you want to run it from a file instead of STDIN.

```
./mdxfind.static -h ALL -h '!salt,!user,!md5x' -f mixedhashes.list wordlist.txt >results.file
```
This means that if you get bored wait for results, working from a shared network drive or want to quickly kill the process, you've still got all those delicious founds.


### Running salted hashes with MDXfind 

In the cases where you've got salted hashes where the salt is not part of the hashes (for example Vbulletin). You will need to give MDXfind a salt file. This is provided using the  `-s` switch command. A quick oneliner to seperate the salts from the hashes. 

```
cat vbulletin-hashes.txt | cut -f 2 -d ":" | sort -u > salts.txt
```

```
 cat vbulletin-hashes.txt | ./mdxfind -m 2611 -s salts.txt wordlist.txt | ./mdsplit.static vbulletin-hashes.txt
```


### Generating Example Hashes

MDXfind has the option to generate examples of what a specific hash should look like and its plaintext pair. This can be helpful when trying to work out if a hash has been truncated, or if you are identifying which variation of a hash you might have found. 

```
echo -n 'Password' | mdxfind -h 'MYSQL' -h '!salt,!user' -z -f /dev/null -i 5 stdin 2>&1
```

```
MYSQL3x01 2f18d4d923ccae07:Password
MYSQL3x02 6c570ef02890ea40:Password
MYSQL3x03 6aa0bd3f0e7cfb70:Password
MYSQL3x04 2fe7115b6128d301:Password
MYSQL3x05 577828bc229cf0ea:Password
```


#### MDXfind to Hashes.org Format

Now that you've found a bunch of hashes using the tips and instructions above, you might want to upload them to Hashes.org so you can jump up the leader board 😉. The following reference table can be used to help you figure out the algo you need to provide, against the ones described in MDXfind. As a

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
|MD5-1xSHA1MD5x01|MD5(SHA1MD5(PLAIN))|
|MD5SQL5|MD5MYSQL5TOT|
|MD5SHA1PASSMD5PASSSHA1PASS|MD5(SHA1(PLAIN)MD5(PLAIN)SHA1(PLAIN))|
|MD5RAW|MD5RAW(MD5(PLAIN))|
|MD5PASSSHA1MD5|MD5(PLAINSHA1MD5(PLAIN))

### Where To Download MDXfind

[Download MDXfind](https://hashes.org/mdxfind.php) 
