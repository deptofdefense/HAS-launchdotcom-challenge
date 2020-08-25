What goes into making a challenge or easter egg? Sometimes it's driven by a fun story, sometimes it's driven by a cool technical challenge.

For the [LaunchDotCom.com](http://launchdotcom.com/) egg hunt, it was driven a bit by both. I was inspired by the 90s vibe from the website to include some old-school style tech. My original plan during the initial design phase was to build a fake voicemail system with a modem or fax in it for people to decode. Sadly, two other CTFs ended up using similar ideas in challenges and so I went to look for something different but still an audio challenge.

The first of two ideas was inspired by an old technique of hiding data in an audio spectrogram. A teammate first designed a challenge like this back in 2012 for [Ghost in the Shellcode](http://ghostintheshellcode.com/) and used it to embed a flag in non-audible frequencies that could still be picked up by laptop and phone microphones and displayed with the right tools. I wasn't sure whether the limitations of the plain-old-telephone-system would work with that technique though!

Indeed, POTS lines only transmit audio from [300Hz to 3400Hz](https://en.wikipedia.org/wiki/Plain_old_telephone_service#Characteristics), specifically tuned to the range of human voices. So any frequency transmitted over a standard telephone line would have to be something audible. This limited the amount of data I could embed in a waveform. One way around this limitation was to encode a relatively small amount of data in the waveform and use that data as a key to another longer and more data-dense.

So how should we encode the encrypted file? I went with a fairly straight-forward mechanism, just convert the bytes of the file into DTMF tones. There are several different encodings we could use for this but the simplest is octal. Digits 0 through 7 are well within the range of DTMF tones (fun fact, the [full DTMF](https://en.wikipedia.org/wiki/Dual-tone_multi-frequency_signaling) includes a 16 character alphabet so we could have been more dense and used Hex values directly!), making it easy to just use them as-is. This isn't particularly compressed, but it's fairly straight-forward (assuming you don't make any mistakes--more on that later!) and should be a relatively easy step. So we'll have to make it a bit of a challenge to find that audio file in the first place.

Now that I had the technical aspects of the challenge down, how do we work it into the story? Thankfully, there was a tremendously funny backstory being created for the [LaunchDotCom.com](http://launchdotcom.com/) website complete with fake personalities and press-releases. It was easy enough to make a few small changes to some planned biographical information to make a bit of a reconnaissance challenge.

First, make it easy to find out the voicemail box numbers of employees in the voicemail system with a public directory. Next, anyone doing some button mashing (or just having used other voicemail systems) might try to use the `*` key to access the employee login. Combine that with a default pin code of `00000000` (and indeed, give hints about that pin code on the website itself) that should be very guessable. The obviously long pin should _hopefully_ discourage potential brute-force solutions as we do pay between 1-2 cents per call!

Next it was just a matter of populating those obviously guessable pins with enough information to point to the pin for the CEO being his birthday which was calculable from the hints provided in his biography. The CEO's voicemail included the GPG-encrypted file encoded as octal, and the "strange beep" (which was the waveform encoded password) lived down a different tree of the system.

Other fun cameos and secrets include:

An actual voicemail recorded by Jeff Moss to the fictional CTO informing her that she couldn't give a presentation at DEF CON by one of our engineers claiming the work as her own.

Several creditors and other people trying to get paid or find out about the fictional story behind LaunchDotCom.

If you used caller-id spoofing to spoof the same number as the voicemail system, it would give you a few extra hints and take you straight to the employee login page as an extra bonus for anyone that tried that! (sadly, no-one did)

Several hints in the website pointed toward the voicemail system and gave hints. A WS_FTP.LOG file (era-consistent!) pointed to several files and fake directory_indexes shows extra files including: [http://launchdotcom.com/site/img/hinthere.gif](http://launchdotcom.com/site/img/hinthere.gif) which pointed people toward a "suspicious beep" in the voicemail system.

The website was actually tested internally on old browsers from the purported time period. It was fun watching people on twitter discover that live!

The website itself was built using [https://kit.voximplant.com/](https://kit.voximplant.com/) a neat system for building custom Interactive-Voice-Response (IVR) systems. It allowed drag-and-drop programming to build some pretty complicated logic including simulating voicemails, pin entries, and it would even report progress as people got to different sections of the voicemail via notifications to a slack channel! It even makes a nice picture to see how it was designed:

![voximplant](./vox.png)


Unfortunately, several people hunting the egg were stuck at the very last steps and while the HackASat organizers even released a hint in the faq pointing people toward using eyes instead of ears for certain audio, no solution was found. Upon further investigation we realized that the encoding mechanism for the encrypted file was unintentionally dropping null bytes! Specifically the difference between:

```py
    flagbytes = open('flag2.gpg', 'rb').read()
    tones = ""
    for byte in flagbytes:
        for char in oct(byte)[2:]:
            tones += char
```

and

```py
    flagbytes = open('flag2.gpg', 'rb').read()
    tones = ""
    for byte in flagbytes:
        print('%03d' % int(oct(byte)[2:]))
        tones += ('%03d' % int(oct(byte)[2:]))
```

Yup, small values were being truncated, then appended together resulting in a file with randomly missing leading zeros for each octal word. :-( Once we realized this error, we updated the system and had a few solutions in a short amount of time, which earned those folks some sweet HackASat swag.

Sorry for those hunters who were bitten by this, hopefully you enjoyed the rest of the challenge and were able to solve it after the update!

Thanks to John Marx for the hilarious press releases and to Kim and [The Difference](https://thedifferenceconsulting.com/) for an awesome website and going the extra mile to make it not only look like it came from the 90s but matching the webstack from that era too!

- Jordan Wiens

[https://vector35.com/](https://vector35.com/)


