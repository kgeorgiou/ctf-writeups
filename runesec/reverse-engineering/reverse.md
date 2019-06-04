**Name**: Reverse  
**Points**: 80  
**Instructions**: Code runs forward, but is judged in [reverse](https://challenges.runesec.com/static/a94af1d438cc6bef9a2f4e17410b14d0/reverse), which runs at challenges.runesec.com:52965.  
**Hint**: Hansel and Gretel waited for the moon to rise and then followed the pebbles back home.  

## Exploration
- We open the file with radare2. We notice a string that says "Well done!".
```
...
      0x0804872d  85c0        test eax, eax
┌─< 0x0804872f  7566        jne 0x8048797      ;[6]
│   0x08048731  83ec0c      sub esp, 0xc
│   0x08048734  6875880408  push str.Well_done_ ; str.Well_done_ ; "Well done!" @ 0x8048875
│   0x08048739  e8e2fcffff  call sym.imp.puts  ;[7]
│   0x0804873e  83c410      add esp, 0x10
│   0x08048741  83ec08      sub esp, 8
│   0x08048744  6880880408  push 0x8048880
│   0x08048749  6883880408  push str.flag.txt ; str.flag.txt ; "flag.txt" @ 0x8048883
...
```

- But that string is only reachable id `test eax, eax` evaluates to `true`. Otherwise thw program's flow jumps over that string, and the flag.

## Exploitation
`test eax, eax` will evaluate to true if `eax` is 0. Actual zero, not the ASCII character 0 which is 0x30 in hex.
```
$ echo -e '\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00' | nc challenges.runesec.com 52965
Enter the password: Well done!
flag{40aacaab5ac5f735e51dec0c0194f7b5}
```