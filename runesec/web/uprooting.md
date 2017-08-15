**Name**: Uprooting  
**Points**: 600  
**Instructions**: There is a website running at http://challenges.runesec.com:53945. Misconfigurations sometimes kill the cats  
**Hint**: Some things are just relative...  

## Exploration
- From the name of this challenge and the given hint, we can guess that the website provided is vulnerable to [path traversal](https://www.owasp.org/index.php/Path_Traversal) attacks.  
- We are given a login interface (Username & Password.) We attempt to login, and as expected, we can't go through, but we get the following message from `/login.php`:  
> Cannot access php config file  

- Based on the instructions for this challenge, and the message we get when we attempt to login, we should start by looking for a PHP configuration file.

## Exploitation
The default configuration file for PHP is named `php.ini`. We can try finding it using relative path traversals.  
- 1st attempt: `/php.ini`  
- 2nd attempt: `/../php.ini`  

*(Maybe we should URL encode `../` to `%2e%2e%2f`)*  

- 3rd attempt: `/%2e%2e%2fphp.ini`  

*(Bingo!)*  

The file has the following content:  
`database_file = "users.db";`  

No we can try find `users.db`.  

`/%2e%2e%2fusers.db` works and we get the following:  
`SQLite format 3@ -� ��@atableusersusersCREATE TABLE users (id integer, flag text) ��R	�)0x666c61677b31303465636536666138373164336365663963303865363939623133656338397d`  

We grab that hex string at the end and convert it to UTF8 to get the flag.

## Notes
We were lucky `php.ini` was only 1 level up from the directory `index.html` resides; the relative path could've been more complicated.  

The directory structure on the server was - more or less - the following:   
```
.  
├── webroot  
│   ├── index.html <- *we start from here*  
│   └── login.php  
├── php.ini        <- *we can get here with `../`*  
└── users.db
```
