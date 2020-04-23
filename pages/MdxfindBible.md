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


## Intro

This is a work in progress page of tips and tricks that might be handy for those learning or using MDXfind. Most of the content was collated from using it, [Royce William's](https://www.techsolvency.com/pub/bin/mdxfind/) post on MDXfind and hassling hops and s3in!c on the hashes.org discord server for usage tips. Credit to waffle and hops for all the dev work.


### Basic Usage Using STDIN

MDXfind's strength is its ability to run wordlists against lists of mixed hashes / lists of unknown hashes. Got a bunch of 32hex hashes but no idea what they are? Let MDXfind do the hard work for you. When running mixed lists of unknown algo's, it's always worth running it with mdsplit to save the hassle of parsing it out later.

```
cat mixedhashes.list | ./mdxfind.static -h 'ALL' -h '!salt,!user,!md5x' wordlist.txt | ./mdsplit.static mixedhashes.list
```
This command runs a mixed file of hashes through all algorithms known to  MDXfind apart from those requiring salts, usernames and internally iterated hashes like `SHA1(MD5(MD5($pass))`.

### Running MDXFind From a File

Alternatively, if you want to run it from a file instead of STDIN.

```
./mdxfind.static -h ALL -h '!salt,!user,!md5x' -f mixedhashes.list wordlist.txt > results.file
```
This means that if you get bored waiting for results, are working from a shared network drive or want to quickly kill the process, you've still got all those delicious founds saved to a file.


### Running salted hashes with MDXfind

In the cases where you've got salted hashes where the salt is not part of the hashes (for example `MD5SALT`). You will need to give MDXfind a salt file. This is provided using the  `-s` switch command. A quick oneliner to separate the salts from the hashes.

```
cat vbulletin-hashes.txt | cut -f 2 -d ":" | sort -u > salts.txt
```

```
 cat vbulletin-hashes.txt | ./mdxfind -h 'MD5SALT' -s salts.txt wordlist.txt | ./mdsplit.static vbulletin-hashes.txt
```


### Generating Example Hashes

MDXfind has the option to generate hashed examples of a plaintext. This can be helpful when trying to work out if a hash has been truncated, or if you are identifying which specific variation of a hash you might be trying to crack.  

```
echo -n 'Password' | ./mdxfind.static -h 'MYSQL' -h '!salt,!user' -z -f /dev/null -i 5 stdin 2>&1
```

```
MYSQL3x01 2f18d4d923ccae07:Password
MYSQL3x02 6c570ef02890ea40:Password
MYSQL3x03 6aa0bd3f0e7cfb70:Password
MYSQL3x04 2fe7115b6128d301:Password
MYSQL3x05 577828bc229cf0ea:Password
```
It can help you hash a known plaintext, so you can compare and identify the algorithm.

#### Example 32hex and Plaintext Pair
`8be3c943b1609fffbfc51aad666d0a04:Password`

#### Generating Candidates And Grepping The Output
```
echo -n 'Password' | ./mdxfind.static -h 'ALL' -h '!salt,!user,!md5x' -z -f /dev/null -i 5 stdin 2>&1 | grep '8be3c943b1609fffbfc51aad666d0a04'
```
#### Example Output

SHA1x01  <span style="color:red">8be3c943b1609fffbfc51aad666d0a04</span>adf83c9d:Password

Truncated `SHA1` identified.

---

### Pausing MDXfind

MDXfind consuming all your cores ? Need to pause it but don't want to loose all your progress ? This command is for you.

`Ctrl-Z`

#### Resuming MDXfind

To bring that suspended MDXfind session back, you can use the `fg` command to continue it in the current shell window and watch those cracked hashes continue to roll in.


#### Killing MDXfind without losing STDOUT

Maybe you've got to shutdown, or maybe you've just decided you don't feel like waiting till 2am for this current run to finish. If you are outputting STDOUT straight into mdsplit, you can kill the MDXfind process rather than wait till it reaches its 500k buffer size. This still allows the founds to be parsed to mdsplit without losing them.

```
ps aux | grep mdxfind
sudo kill -9 PIDOFMDXFIND

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
