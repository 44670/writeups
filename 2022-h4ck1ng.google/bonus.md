# h4ck1ng.google writeup: bonus challenges

Website: http://h4ck1ng.google/

Official Discord: https://discord.gg/xKvEKkW4KZ

# EP000 Bonus

The number can be called for free with Skype. It's a maze-adventure game and the right path could be found in the video. 

The flag will be read for only once at the ending, so an audio recorder may be needed.

# EP001 Bonus

The contents of the spam mails are not useful, there's a "X-AES-CBC-Slice" header in each mail, one of them is the AES key, one of them is the IV, and the others are ciphertext. The order is randomized and we have to recovery it by ourselves.

We know that in the CBC mode, the plain text is XORed with the previous cipher text before the encryption. So for one block, we can just decrypt it with the key and previous cipher text we guessed, instead of starting with the IV and all the previous blocks.

I have written a CUDA-based GPU solver to enumerate all the possible "previous cipher text" and key, trying to find patterns of the plaintext. It turns out that the key is always the same slice, and the plaintext a base64-encoded image. 

You don't have to write the CUDA code if you already have such knowledge, a cpu-based solver is enough after that.

After decrypted all the slices, we have known the previous cipher text of each slice, so we can just recover the original order by topological sorting. The flag is just written in that image.


# EP002 Bonus

After reviewing the data, we can find that it communicates to a grpc oauth server. 

For a grpc server, we have to guess the service name (or package name in Golang) and method name: oauth.OAuth/GetToken. (It may be hard to guess without hints.) By editing the original protobuf request, we can craft a refresh-token request, and get the flag from the server.

# EP003 Bonus

It's a "real world" game! The geographical locations and hints could be found in the ch2's maze and the flags are hidden in those places around the world. Some of the places have not been visited/solved yet at the time of writing:

```
Sydney, Australia
Prague, Czech Republic
Trento, Italy
Tamins, Switzerland (Due to weather conditions and safety concerns, this one may not available until next summer.)
```

There's a dedicated channel in the official discord server for sharing efforts about this challenge. [Join it](https://discord.gg/xKvEKkW4KZ) if you live in one of the places and want to help!

# EP004 Bonus

The html file contains a VR object, an url to twitter search page, and an outline of the Google VRP's card which is sent to every bug hunters along with the gifts. 

By extracting the VR object, we can find a piece of the QR code, decoding it will result nonsense since most part of the QR code is missing. Following the hint, there's a QR code on the back of the card and we are supposed ask a bug hunter on the twitter for a photo of it. Fortunately, someone has already posted a photo on the internet, by overlaying the QR code, we can decode it and get the flag.

# EP005 Bonus

This one is hosted on a youtube stream, you need 10 votes for running each line of code, fortunately you can register multiple youtube accounts under one google account.

By reading the manual, we could found there's a "flag rom" on the bus. We can't read it by basic code, but there's a DMA-like device which allows us to transfer one byte from one device to another. On the bus there's an adder, an 1-byte register, and an IO expander which connects 0x40 bit to the meeting room's light. So we can extract one byte by following trick:

1. DMA the byte in the flag rom to the register.
2. DMA the register to the IO expander, read the 0x40 bit from the meeting room's light.
3. DMA the register value to adder's two inputs, and do the addition, DMA the result back to the register, essentially it's a l-shift operation.
4. Repeat the step 2-3, until we know all the bits.


By repeating this process, we can extract all the bits of the flag except the 0x80 bit, and that won't be a problem since they are all printable ASCII characters.