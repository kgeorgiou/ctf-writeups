**Name**: EveCodesBetter  
**Points**: 400  
**Instructions**: There is a website running at http://challenges.runesec.com:49670. Try to see if you can become an administrator.  
**Hint**: Everyone knows that mode is bad because we can see the penguin!  

### Exploration
- We're given a login screen (Username & Password)

- Inspecting the page's source we see this comment:
> \<!-- Guys use dev:dev_testing until we go live --\>  

- We are in. Now the source contains the following comment:
> \<!-- DEBUG - REMOVE THIS BEFORE WE GO LIVE!
Cookie Value:
Encrypted Cookie Value: 5e72ac7a72f2100c2a89b80cd2e171706215010861f697b2b5ab2692d5b75925d664080769d6e8f02c5b5713a26b70bf5258ace96b0c51d322effbefea935f9ef718ac79f1281b4d77deff88cbac078847523918f1647186d71f4ef3476ae487ecb7c92019cfe7ec7447505374b81eb15e72ac7a72f2100c53d565f81ee58dce7848fdea6f83922a461fad5d1b842ac2
Decrypted Cookie Value: username=dev;first_name=developer;last_name=developer;is\__admin=\__False\__;status=0;_environment=\__True\__;is_the_username_enabled=yes;loggedin=1;
--\>  

- We notice 5e72ac7a72f2100c is repeated twice in the string. 

- Looking up "cryptography we can see the penguin" we quickly find out about ECB encryption - and about its weakness. 

- ECB operates - usually - on blocks of either 8 or 16 bytes. The ciphertext we have at hand seems to be using blocks of 8 bytes.

- ECB's weakness explains our 1st observation. 5e72ac7a72f2100c in the cipher text maps to "username" in plaintext. Because it's ECB, if we split the plain text in blocks of 8 bytes, any blocks that have the same value in the plaintext, they will have the same value in the ciphertext as well. That's the case with 5e72ac7a72f2100c -> username.

### Exploitation 
Let's break both the ciphertext and tha plain text in blocks of 8 bytes:
```
1.  5e72ac7a72f2100c -> username
2.  2a89b80cd2e17170 -> =dev;fir
3.  6215010861f697b2 -> st_name=
4.  b5ab2692d5b75925 -> develope
5.  d664080769d6e8f0 -> r;last_n
6.  2c5b5713a26b70bf -> ame=deve
7.  5258ace96b0c51d3 -> loper;is
8.  22effbefea935f9e -> __admin=
9.  f718ac79f1281b4d -> __False_
10. 77deff88cbac0788 -> _;status
11. 47523918f1647186 -> =0;_envi
12. d71f4ef3476ae487 -> ronment=
13. ecb7c92019cfe7ec -> __True__
14. 7447505374b81eb1 -> ;is_the_
15. 5e72ac7a72f2100c -> username
16. 53d565f81ee58dce -> _enabled
17. 7848fdea6f83922a -> =yes;log
18. 461fad5d1b842ac2 -> gedin=1;
```
Our eyes are, of course, on `is__admin=__False__`, we must find a way to make it positive, a.k.a `__True__`.  

We have a puzzle at hand. We need to replace `__False__` with `__True__`.  

The 13th block is our *deus ex machina*.  

We can replace the the 9th block with the 13th to get a key-value pair of: `is__admin=__True___;` - we need to get rid of that extra underscore.
We can do that by replacing block 10 with block 14. Now we have the following key-value pair `is__admin=__True__;` followed by `is_the_=0`, which is essentially junk, but still a valid key-value pair.

*Since ECB works on blocks of 8 bytes, each block can be moved around without breaking the validity of the ciphertext.*

```
1.  5e72ac7a72f2100c -> username
2.  2a89b80cd2e17170 -> =dev;fir
3.  6215010861f697b2 -> st_name=
4.  b5ab2692d5b75925 -> develope
5.  d664080769d6e8f0 -> r;last_n
6.  2c5b5713a26b70bf -> ame=deve
7.  5258ace96b0c51d3 -> loper;is
8.  22effbefea935f9e -> __admin=
13. ecb7c92019cfe7ec -> __True__ <--
14. 7447505374b81eb1 -> ;is_the_ <--
11. 47523918f1647186 -> =0;_envi
12. d71f4ef3476ae487 -> ronment=
13. ecb7c92019cfe7ec -> __True__
14. 7447505374b81eb1 -> ;is_the_
15. 5e72ac7a72f2100c -> username
16. 53d565f81ee58dce -> _enabled
17. 7848fdea6f83922a -> =yes;log
18. 461fad5d1b842ac2 -> gedin=1;
```

We concatenate the above blocks to a single string which can use to replace our current cookie in our browser.  
*In Chrome: Dev Tools > Application > Cookies > Double click on the value of `session` to edit*  

We refresh the page for the request to be sent with the cookie we crafted, and *voilà*, we capture the flag.

### Notes
The way the key-value string was written, shows the author of the challenge aimed for a specific alignment of the blocks so they can be re-arranged (by padding with underscores, using 3 different formats that resemble booleans.)  
There are plenty ways to re-arrange the above blocks while maintaining `is__admin=__True__;`.   

Since we can't be certain how the server parses the cookie, a lot of time was spent trying to address the following questions:  
- Do all the keys need to be present?
- Does the order matter?
- Are boolean formats `no`/`yes`, `0`/`1` and `__False__`/`__True__` interchangeable?   

But in the end it seems that the judge system was comparing against an exact string sequence rather a key-value map. 
