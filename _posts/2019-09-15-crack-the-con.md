---
layout: post
title: Crack The Con 2019 Writeup
tags: [Password Cracking, Cynosure Prime, Crack The Con, Hashcat, NSEC3]
---


I spent some time milling around the CrackMeIfYouCan stand at Defcon last year, but never got round to asking “Hey, how do I get involved”. 

Fast forward to this year and a number of the Hashes.org community talked about putting a team together for Crack The Con. I thought, "Yeah that would be a laugh", so I jumped on in. 

This is what I learnt... 

## InTheZone 

As most of the team were off to a head start on the hashes when the competition started, I decided to work on the InTheZone challenge to see if we could get some early points. All there was to begin with, was the following in a text file: 


```
dig +dnssec starthere.crackthecon.0x23.pw TXT
```

Hmmmm. So naturally, I assumed it was some sort of DNS related hash. I quickly checked the Hashcat examples, and mode 8300 was listed for cracking NSEC3 hashes (NSEC stands for ‘next secure record’). So I set about researching. 

NSEC3, is effectively a chain of DNS records linking one to the other, in a zone secured with DNSSEC. 

As some of the examples online detailed zone walking, it looked like I’d need to use this technique, to collect the hashes that were linked from one to the other. 

I started using a couple of different tools but hit some dependency snags along the way. I finally settled on an Nmap script, that did the trick:

```
sudo nmap  -sU -p 53 ns.crackthecon.0x23.pw 
--script=dns-nsec3-enum --script-args dns-nsec3-enum.domains=starthere.crackthecon.0x23.pw 
-oA hashv2   -vvvv
```

Out came a bunch of hashes: 

```
00gbb3vv0ofs2p513u104iisss0nsec3
00nr4pu24aq9ab5t9u934ik72h0isfun
```
#####Formating And Cracking The Hashes

Checking back on the Hashcat example, I realised I was missing a big chunk of information. The salt, the iterations and the domain, in the following format: 

```
Hash:.domain:SALT:Iterations 
```
The next question was, where do i find those 🧐? 
After some more Googling, it turned out the NSEC3PARAM record had all the goodies I needed:

```
crackthecon.0x23.pw IN NSEC3PARAM    
hash algorithm:SHA-1 (1)
flags:None (0)
iterations:19
salt:(128 bits)    
4754464F214E6F7468696E6748657265
```

Some command line Kung Fu later to format the hashes:

```
awk {print $0”:.crackthecon.0x23.pw:4754464F214E6F7468696E6748657265:19”} hashes-extracted.txt 
```

#####  Example of Correctly Formatted Hashes

```
00gbb3vv0ofs2p513u104iisss0nsec3:.crackthecon.0x23.pw:4754464F214E6F7468696E6748657265:19
00hfaricfrm4mlcc1mnss6i4jtgnsec3:.crackthecon.0x23.pw:4754464F214E6F7468696E6748657265:19
00nr4pu24aq9ab5t9u934ik72h0isfun:.crackthecon.0x23.pw:4754464F214E6F7468696E6748657265:19
00oco3n80stmell97ncc8u3av80nsec3:.crackthecon.0x23.pw:4754464F214E6F7468696E6748657265:19
00qti8rpelus78s8cssr2ufn9jgnsec3:.crackthecon.0x23.pw:4754464F214E6F7468696E6748657265:19
00rpjhtgen69qdq9hjojsbfnjngnsec3:.crackthecon.0x23.pw:4754464F214E6F7468696E6748657265:19
00s41hb79d4ik4roa0pa7vm3ld0nsec3:.crackthecon.0x23.pw:4754464F214E6F7468696E6748657265:19
00t0ridpt1emn55kom8ev4qsm8gnsec3:.crackthecon.0x23.pw:4754464F214E6F7468696E6748657265:19
```

Once I finally got the hashes formatted correctly, it was a matter of running them against a subdomain wordlist. I decided to use the SecLists GitHub repo to start with. 

[SecLists top 5000 subdomains](https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/subdomains-top1million-5000.txt)

##### Example Command 

```
hashcat -m 8300 -a 0 nsec3hashes.txt subdomains-top1million-5000.txt 
```
Once I got a few matches back, it was just a case of checking the results, analysing the patterns, rinse and repeat. 🛀 


## 100% salt-free 110% hassle

After reading [@Evil_Mog](https://twitter.com/Evil_Mog)’s blog post about the creation of the [generated2.rule](https://github.com/evilmog/evilmog/wiki/Hashcat-Raking---generated2.rule) I thought we’ve only got a few hours left of the competition, why not see if I could find some other patterns since my cards were idling. 

Our found percentage was lower than we anticipated for the 100 % salt-free 110% hassle list (raw-sha512), so it they seemed like the perfect candidate to try it on. 

As the hashing algorithm is relatively fast, we are able to run multiple passes of random rules, to effectively rake through combinations till we find a pattern hit. Once we got a hit, we added that rule to a rule file, to run against other algorithms present in the competition and the hashes themselves. 

An example command might look something like this: 

```
hashcat -m 1700 -a 0 hashes.txt founds.txt -g 100000 --debug-mode=1 --debug-file=matched.rule --loopback
```

During raking, I noticed that experimenting  with the minimum and maximum generated rule function flags, was producing more results:

```
--generate-rules-func-min=NUM 
--generate-rules-func-max=NUM
```

Cranking the minimum rule function to four, resulted in more hits. Some of the rules found can be seen below: 

```
T5 *A7
y2 s}S o4Y
TB O82
T4 'B *1A
O42 *92 O59
i8t s3| T4
K k OA4
T6 s&N o8e
$B s|\’  ^<
```

##### Example Output from Matched Rules


```
$ cat word.txt
testwordrules

$ cat rules/custom.rule
T5 *A7
y2 s}S o4Y
TB O82
T4 'B *1A
O42 *92 O59
i8t s3| T4
K k OA4
T6 s&N o8e
$B s|' ^<

$ ./hashcat64 word.txt --stdout -r rules/custom.rule
testwOrlrudes
teteYtwordrules
testwordlEs
tlstWordrue
teetrdrulss
testWordtrules
etstwordrulse
testwoRdeules
<testwordrulesB
```
 
*(Output courtesy of the [hashes.org](https://discord.gg/pbbyBbd) discord)*

As the minimum number of functions was increased to four, the likelihood of hitting a pattern was increased per run. As you can see above, this didn’t necessarily mean that four rules needed to be applied to each candidate. Maybe only one or two of the rules would result in successful cracks, but the percentage chance of a crack per rule, increased.

Using this method, and stacking the matched.rule, we managed to grind additional hashes and patterns we were unlikely to ever find through traditional rules alone. Here are some example founds:

##### Example Founds 
```
woaitaitongtong062
Rhtottotybt juy`v
2111201111cmpunk
AngelHeartrt201
17.05.25.2003.s
```

## Wrap Up
Big thank you to the Cynosure Prime team for putting together the competition and for all their hard work running it. Check out their blog post if you want to find out more: [Crackthecon year One](https://blog.cynosureprime.com/2019/05/crackthecon-year-1.html)

Shout to all the hashes.org team members for their killer work during the competition.
