**Name**: weaKEYness   
**Points**: 800  
**Instructions**: The enemy has employed their secure transmission device which sends a secret message to all friendlies nearby. Deep behind enemy lines, you need to intercept the secret message, decrypt it, send the plaintext back to the device and get the flag. Fortunately, the transmission device's [source code](https://challenges.runesec.com/static/35b38c123066f452b28339793f67559a/weaKEYness_disclosed.py) was leaked. Device running at challenges.runesec.com:19853.  
**Hint**: A new session key is generated every time the device transmits a message. You either need to be the Flash, or scripting is your friend  


## Exploration
- This is the source of the service that produces the secret message we're asked to decrypt:  

```
#!/usr/bin/python

from Crypto.PublicKey import RSA
from Crypto.Random import get_random_bytes
from Crypto.Cipher import AES, PKCS1_OAEP
import random
import string
import time
import signal
import sys

public = RSA.importKey(open("public",'r').read())
private = RSA.importKey(open('private','r').read())

session_key = get_random_bytes(11)
encrypted_session_key = public.encrypt(session_key,32)[0]

encrypted_session_key += 'rune'
cipher_aes = AES.new(encrypted_session_key)

plaintext = ''.join(random.choice(string.lowercase+string.uppercase+string.digits) for i in range(32))
padding = 16 - (len(plaintext) % 16)
to_encrypt = plaintext + ('g'*padding)

ciphertext = cipher_aes.encrypt(to_encrypt)

def get_answer():
	answer = raw_input()

	if answer == plaintext:
		print 'Your flag will be here 0_o'
	else:
		print "No cigar!"
		sys.exit()


print "\033[01;34m==================="
print "\033[01;36mBegin Secure Comms"
print "\033[01;34m==================="
time.sleep(1)
print "\033[01;32m[+] Transmitting public key\033[00m"
time.sleep(1)
print  public.exportKey() + "\n"
time.sleep(1)
print "\033[01;32m[+] Transmitting base64-encoded encrypted session key\033[00m"
time.sleep(1)
print "{}".format(encrypted_session_key.encode("base64"))
time.sleep(1)
print "\033[01;32m[+] Transmitting base64-encoded Secret Message\033[00m"
time.sleep(1)
print "{}".format(ciphertext.encode('base64'))
time.sleep(1)
print "\033[01;32m[+] Waiting 15 secs for confirmation\033[00m"

signal.signal(signal.SIGALRM,get_answer) 
signal.alarm(15)
try:
	get_answer()
except TypeError:
	print 'You have to be quicker'
```

- The plaintext of the message consists of 32 characters, randomly chosen from [a-zA-Z0-9]. 

- The plaintext is padded with the character 'g' until its length is divisible by 16. In this case since the length of the plaintext is always 32 (which is divisible by 16,) the implementation just pads it with 16 extra 'g's.

- The padded plaintext is then encrypted to `ciphertext` using AES (symmetric encryption) and the key is `encrypted_session_key`.

- We receive BASE64(`encrypted_session_key`) and BASE64(`ciphertext`). That's all we need to decrypt the cipher text and get the plaintext.

## Exploitation

We can write a script that given a key and a ciphertext can give us the plaintext and also strip it down from the extra padding.

```
from Crypto.Cipher import AES
import sys
import re

BLOCK_SIZE = 16

key = sys.argv[1].decode('base64')
ciphertext = sys.argv[2].decode('base64')

cipher_aes = AES.new(key)

padded_plaintext = cipher_aes.decrypt(ciphertext)
plaintext = re.sub('g{0,%d}$' % BLOCK_SIZE, '', padded_plaintext)

print 'Plaintext: %s' % plaintext
```

We can pass the key and the ciphertext as arguments to our script
```
$ python weakeyness.py QIcCua2/KWj2S/azcnVuZQ== tkQNmvsFOAIkZctJNMorEGAhn1XFec8hjgBaewl19wy9+ROr1aY0nzFI9sbyLTDO
Plaintext: LvjXdPXWZhcXs2T0RvcuP458jc0SZ8kE
```

We have a 15 seconds window to do our copy-paste (x3) back and forth, which is a fairly sufficient time.  

If let's say we only had 2 seconds, then we'd have to extend our script to parse the output from the remote service and then redirect our script's output to the service's input.

Full run:
```
$ telnet challenges.runesec.com 19853
Trying 35.176.251.226...
Connected to challenges.runesec.com.
Escape character is '^]'.
===================
Begin Secure Comms
===================
[+] Transmitting public key
-----BEGIN PUBLIC KEY-----
MCUwDQYJKoZIhvcNAQEBBQADFAAwEQIMQids/a6IYKWTm+lBAgEl
-----END PUBLIC KEY-----

[+] Transmitting base64-encoded encrypted session key
Nb13mxQVaKlP3JN3cnVuZQ==

[+] Transmitting base64-encoded Secret Message
2BJF/Y/frubV5Ucz7E5i0lrM3jXqTbc3i7t1kS8UhZu3BucI7vRsWZ+abEMGP3Id

[+] Waiting 15 secs for confirmation
DsB70Qt7uMm1LdfIVwUylajuonoZlO76

flag{4470afda881a37caacd7081bef3eedd7}
```
