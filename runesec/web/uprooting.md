*Name*: Uprooting  
*Instructions*: There is a website running at http://challenges.runesec.com:53945. Misconfigurations sometimes kill the cats  
*Hint*: Some things are just relative...  

### Exploration
From the name of this challenge and the given hint, we can guess that the website provided is vulnerable to [path traversal](https://www.owasp.org/index.php/Path_Traversal) attacks.  
Based on the instructions for this challenge, we should start by looking for some configuration file.

### Exploitation
The default configuration file for PHP is named `php.ini`. We can try finding it using relative path traversals.  
1st attempt: `/php.ini`, Nope  
2nd attempt: `/../php.ini`, Nope  
*(Maybe we should URL encode `../`)*  
3rd attempt: `/%2e%2e%2fphp.ini`, Success  
*(We were lucky this was only 1 level up from the directory index.html resides; the relative path could've been more complicated)* 

The file contains the following information: `database_file = "users.db";`  
No we can try find `users.db`.  
Trying `/%2e%2e%2fusers.db` works and we get the following:  
`SQLite format 3@ -� ��@atableusersusersCREATE TABLE users (id integer, flag text) ��R	�)0x666c61677b31303465636536666138373164336365663963303865363939623133656338397d`  

We grab that hex string at the end and convert it to UTF8 to get the flag.