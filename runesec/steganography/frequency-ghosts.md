**Name**: Frequency Ghosts  
**Points**: 100  
**Instructions**: People say there is a ghost in this file: [frequency-ghosts.wav](https://challenges.runesec.com/static/f3702ddd4ee5fa4f34de56f3bf055ff2/frequency-ghosts.wav)  
**Hint**: Follow the title and be a bit creative!

## Exploration
 - The easiest thing we can do with an audio file, is to listen to it. It'd be heavenly if we'd hear some warm voice reciting the flag's MD5 hash for us. But, no. It's just random, unsettling noise. 

 - We recall the infamous puzzle [11B-X-1371](https://en.wikipedia.org/wiki/11B-X-1371) that had some (disturbing) images hidden in the video's audio [spectrogram](https://en.wikipedia.org/wiki/Spectrogram).
 
 - We need to find a way to check this .wav file's spectrogram.  
   
 ## Exploitation
 [Audacity](http://www.audacityteam.org/) is an open-source audio editor. It should be able to help us out with this.  
 
 We open the .wav file in Audacity and on the left side of the track, we click on the drop down to switch form waveform to spectrogram and... lo and behold.
