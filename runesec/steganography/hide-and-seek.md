**Name**: Hide And Seek  
**Points**: 80  
**Instructions**: When there is nothing left to hide, there is nothing left to seek: [hidenseek.jpg](https://challenges.runesec.com/static/6c6ebd9fa1c682d9089f2bd3f6de1dd5/hidenseek.jpg)  
**Hint**: Let's play a game!  

## Exploration

- We open the image. No signs of a flag.

- Maybe we can open it in photoshop and start playing with the colors (invert, contrast, etc,) maybe there's something's to be discovered.

## Exploitation

Our photoshop efforts were in vain.  

However, we've discovered this tool called [steghide](http://steghide.sourceforge.net/). It seems like the go-to steganography encode/decoder.

There's an online running steghide at https://futureboy.us/stegano/decinput.html. We upload the image and this tool kindly gives us the flag. 

