# h4ck1ng.google writeup: bonus challenges

Website: http://h4ck1ng.google/


# EP000 Bonus

The number can be called for free with Skype. It's a maze-adventure game, and the right path can be found in the video. 

The flag will be read only once at the end. An audio recorder is recommended.

# EP001 Bonus

The contents in the spam mail are not helpful. There's an "X-AES-CBC-Slice" header in each mail. One is the AES key, one is the IV, and the others are ciphertext. The order is randomized, and we have to recover it by ourselves.

We know that in the CBC mode, the plain text is XORed with the previous cipher text before the encryption. So for one block, we can decrypt it with the key and previous cipher text we guessed instead of starting with the IV and all the previous blocks.

I have written a CUDA-based GPU solver to enumerate all the possible "previous cipher text" and keys, trying to find patterns of the plaintext. It turns out that the key is always the same slice, and the plaintext is a base64-encoded image. 

You don't have to write the CUDA code if you already have such knowledge. A CPU-based solver is enough after that.

After decrypting all the slices, we know the previous cipher text of each slice, so we can recover the original order by topological sorting. The flag is just written in that image.


# EP002 Bonus

After reviewing the data, we can find that it communicates to a grpc OAuth server. 

For a grpc server, we need to guess the service name (or package name in Golang) and method name: `oauth.OAuth/GetToken`. (It may be hard to imagine without hints.) By editing the original protobuf request, we can craft a refresh-token request and get the flag from the server.

# EP003 Bonus

It's a "real world" game! The geographical locations and hints could be found in the ch2's maze, and the flags are hidden worldwide. Some of the places have not been visited/solved yet at the time of writing:

```
Sydney, Australia
Prague, Czech Republic
Trento, Italy
```

There's a dedicated channel in the official discord server for sharing efforts about this challenge. 

# EP004 Bonus

The HTML file contains a VR object, an URL to the Twitter search page, and an outline of the Google VRP's card, which is sent to every bug hunter along with the gifts. 

By extracting the VR object, we can find a piece of the QR code, and decoding it will result in nonsense since most part of the QR code is missing. Following the hint, there's a QR code on the back of the card, and we are supposed to ask a bug hunter on Twitter for a photo of it. Fortunately, someone has already posted a photo on the internet. By overlaying the QR code, we can decode it and get the flag.

# EP005 Bonus

This one is hosted on a youtube stream. You need ten votes to run each line of code.

By reading the manual, we could find there's a "flag ROM" on the bus. We can't read it by basic code, but there's a DMA-like device that allows us to transfer one byte from one device to another. On the bus, there's an adder, a 1-byte register, and an IO expander that connects 0x40 bit to the meeting room's light. So we can extract one byte by the following trick:

1. DMA the byte in the flag rom to the register.
2. DMA the register to the IO expander, read the 0x40 bit from the meeting room's light.
3. DMA the register value to the adder's two inputs, and do the addition, DMA the result back to the register. Essentially it's an l-shift operation.
4. Repeat step 2-3 until we know all the bits.


By repeating this process, we can extract all the flag bits except the 0x80 bit, which won't be a problem since they are all printable ASCII characters.

