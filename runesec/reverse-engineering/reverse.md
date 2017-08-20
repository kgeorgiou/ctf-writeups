**Name**: Reverse  
**Points**: 80  
**Instructions**: Code runs forward, but is judged in [reverse](https://challenges.runesec.com/static/a94af1d438cc6bef9a2f4e17410b14d0/reverse), which runs at challenges.runesec.com:52965.  
**Hint**: Hansel and Gretel waited for the moon to rise and then followed the pebbles back home.  

## Exploration
// TODO

## Exploitation
```
$ echo -e '\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00' | nc challenges.runesec.com 52965
Enter the password: Well done!
flag{40aacaab5ac5f735e51dec0c0194f7b5}
```