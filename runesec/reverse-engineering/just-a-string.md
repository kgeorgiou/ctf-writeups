**Name**: Just a String  
**Points**: 50  
**Instructions**: CWE-798 - Let's see what you've got: [Just a String](https://challenges.runesec.com/static/5e320dd7210845ba914ea64d6e1358bd/just_a_string)  
**Hint**: You don't need a hint for a string!  

## Exploration
- [CWE-798](https://cwe.mitre.org/data/definitions/798.html) 

## Exploitation
We open the file with our text editor of preference. The flag is sitting there. What we're looking for is easy to spot:  
`Usage: login <Your_Username> <Your_Password> flag{2887ab1203b48f 2434b3a0654b34bb0f}    Good Job`

We can also look for the hardcoded flag in the file with `strings`:  
```
$ strings just_a_string
...
flag{2887ab1203b48f
2434b3a0654b34bb0f}
...
```