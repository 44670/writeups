# h4ck1ng.google writeup: main challenges
Website: http://h4ck1ng.google/


# EP000
We don't need any special tools to solve this episode. Chrome's developer tools are enough.
## Challenge 1
This one is easy but fun. It's a chess game, and the server will give out the flag if you win. However, the AI is cheating and will always win in normal cases.

By examining the network requests, we can find a "load_board.php" call when we start the game. It accepts a filename as a parameter and will return the data in that file on the server, which is a typical LFI flaw. We can just change the filename to an arbitrary file and get the PHP source code.

After reviewing the code, we can find a SQL injection flaw in the admin panel to allow us to log in. In the admin panel, we can change the AI thinking time and turn off the cheat mode. After that, we can win the game and get the flag.

Another solution is to get the flag from the environment variable directly by requesting the '/proc/self/environ' file. We will need this trick in the upcoming challenge.
## Challenge 2
The log data being searched is not related to the flag. We can find a hint in the HTML and get the Perl source code of that tool. It turns out that the tool has LFI and RCE flaws. We don't have to exploit the RCE flaw, just guess the flag file path (it's "/flag") and use a keyword like "http" to get the flag.
# EP001
Starting from this episode, we may need binary analysis/patching tools for some challenges. There are some free options like Ghidra, IDA Free, etc.
## Challenge 1
This challenge is about a go binary. It's a decryption tool, but we need to find the key. After being analyzed (the main function may be a good start), we can find that this tool will print out an URL under a certain condition. We can just copy that string constant or patch the binary to bypass the check and print it. It turns out to be a key file server. By downloading all the key files and trying them one by one, eventually, we will find the correct one and get the flag.
## Challenge 2
This one is about a C binary. Its main function does nothing, but there's a "print" function, which reads the system time and outputs an URL generated with some algorithms (like a TOTP). Instead of reimplementing the algorithm, we can just patch the main function to jump to the "print" function directly. After that, we can have a one-time URL and get the flag.
## Challenge 3
The chess game again! But now it's harder. After dumping the PHP source code with the same trick, we can find that it won't print the flag even player wins. Dumping environment variables is the only way to solve it.

Additionally, the "load_board.php" now checks the file extension (fortunately, .php is still allowed), so we can't read "/proc/self/environ" directly. Although the "load_board.php" accepts other protocols like php:///, however, I have tried for a long time and found no luck in bypassing the extension check.

What we should do here is find another flaw. After we reviewed the code again, we found a suspicious behavior that "unserialize" the user-uploaded data directly. Like Java serialization, PHP serialization can also be very dangerous. It allows us to construct any object on the server side. A vulnerable class is defined in the source code, and it will execute the command and print the result when deserialized. So we can just copy the source code in our local php environment, set-up the object that executes "cat /proc/self/environ", serialize it and upload to server. After that, we can dump the environment variables and get the flag.
# EP002
Starting from this episode, we may need to write some code to solve the challenges. Python should be enough.
## Challenge 1
The secret message is hiding in the png file. By comparing with the original image from the website, we can find that they are mostly identical except for some pixels in the first two rows. These pixels are all 0s and 1s in the channels.

After combining the 0s and 1s from all the channels in each pixel (don't forget the alpha one), we can construct a binary string. Then we can squeeze it into bytes. It turns out to be a certificate file. Something must be hidden in it.

We don't have to know how a certificate file is structured. Just decode it with base64, and we can easily extract the flag from the result.
## Challenge 2
The intended way to solve this challenge is to install Timesketch and analyze the incident timeline. However, I just got the flag by shamelessly searching "http" in the csv file.
## Challenge 3
This one is really interesting and may have more than one solution. It's a limited bash shell (not a keyword-based filter but a "higher" level one). Running any command or function directly will be rejected.

Fortunately, some keywords like "function" are allowed, so at least we can define functions but can't call them. After reading the bash manual, I have found that overriding built-in functions are allowed. So I just redefine the "echo" function to print the "/flag" file, and I am lucky that the "echo" function is called somewhere in the server-side script.

My solution should be an unintended one. There should be another solution by abusing the "tab-completion" feature.
# EP003
## Challenge 1
The password can be found in the video (~15:09). While examining the user's home folder and bash_history, we can find an unfinished project for downloading a file with the Google Drive API. All you need is the access token. Although the credential file has been deleted by the user, it's still stored in gcloud_cli's ".config" folder. It is an RSA private key of a Google Cloud service account.

What we should do now is to get the access token in GDrive's scope by OAuth2 with the credential we have found. It could be done with gcloud_cli by setting various parameters, but let's try the hard way because it's fun!

After reading the documents, we can find that we should use the RS256 algorithm to sign the JWT token with the private key, fortunately, there's a python library for JWT tokens. By building the proper OAuth2 request, we can get the access token of its GDrive, and download the flag file.
## Challenge 2
The game is a little difficult to play, but it's not necessary to play it. Following the hint, we can use the famous "konami" code and get access to the python shell. Unfortunately, it's a sandboxed one with many limitations:

1. Most built-in functions are removed, and of course, we can't do the import.
2. Both input and output are length-limited.
3. The variables we defined are not preserved.
This is quite challenging and may have many solutions. Here is mine:

After carefully reading the python manual, I found a `__builtins__` keyword for all built-in variables/functions, and you can even add new ones! So limitation No.3 is solved. I can add a new variable by something like `__builtins__.a` = 1.

By searching "ctf python sandbox escape," I have found a common way by doing `().__class__.__bases__[0].__subclasses__()[xxx]` trick, which allows you to use all the built-in classes. Then I still need to find the correct index of the appropriate class. So I just stored the list to a variable with the previous trick.

After that, I can list that array, but the output is too long, so I have to print about 20 items each time. Fortunately, I have found a class that can be used for importing modules, then I just imported the "os" module and read the "flag" file.
## Challenge 3
I can't get the apk to work. My phone may be too outdated. So I just decompiled it with jadx. It turns out to be extracting the "docid" field in the QR code and signing the message with HMAC-SHA256 while the HMAC secret is stored in the apk.

After reimplementing the algorithm and building the proper request, we can get the flag from the server. No installation of the app is needed.
# EP004
## Challenge 1
The source code could be downloaded from challenge 3's git server. It is hard to solve without it.
The flaw is quite obvious after we reviewed the code. After some verifications, it just extracts the tar.gz and output the original file content if the file exists. It doesn't check the path traverse problem, so we can just craft a tarball with a file named "../../../../../../flag" and let the server process it.

By the way, you don't have to buy a commercial web debugging tool to upload the file. Making an HTML form with a file input is enough.
## Challenge 2
By reading the source code carefully, we can find a flaw/backdoor in the safeEqual function. It does the comparison in an insane way: instead of comparing the chars that it appears to be, it actually compares the index of digits in the string.

So we can construct a password with no digits in the base64-encoded hash and keep resetting tin's password until we can log in with that. Fortunately, about 1% of the passwords satisfy that condition.
## Challenge 3
The git server rejects all the push requests. Following the hints in the error message, we can make the server call the script by setting the push option in the git command. Then we can modify the script to print the flag and let the server execute it.
# EP005
## Challenge 1
It's a raw bitmap image. By experimenting with various strides, we can find that 96 pixels/1bpp (or 48 pixels/2bpp) is the correct one. Then we can decode it into some image clips and combine them to get the flag. (I have used PowerPoint to do that)
## Challenge 2
This challenge is both challenging and thrilling. I don't want to be too spoiling here, so I will give out the critical parts of the solution here.

The source code could be downloaded from the website. By reading the source code, we can find that the only way to solve it is to forge a valid signature of the negative score data by ourselves. It looks like an impossible mission at a glance, but it's not.

After reviewing the source code, we can find an API to get the RSA public key from the server. The public key is strong enough, but E is 3, which may allow Bleichenbacher's attack. Unfortunately, the server checks a lot in the padding, so we can't use the original attack method to solve that.
Let's analyze how Bleichenbacher's attack (variant 2 in some articles) works. We can find that it's a fast way to solve X, which makes pow(X,3) equal to:

`prefix || random_bytes || suffix`

`prefix` and `suffix` are predefined constants we can control. `Random_bytes` is uncontrollable and inevitable.

And since pow(X,3) is smaller than the modulus N, pow(X, 3, mod N) is identical to pow(X,3). In this way, X is a valid signature if the verifier is happy with the random_bytes in the middle.
Let's have a look at the verifier's code again. It checks the most padding rules but leaves the "sequence" array's length unchecked. So we can insert a DER object in that sequence and make its length equal to the uncontrollable random_bytes' length.

After bypassing the verifier, we can get the flag by submitting a negative score.
## Challenge 3
The clues are hidden in the introduction animation of each episode. It's morse-encoded in the audio and quite obvious if you are using a wired headphone. I have wasted a lot of time on my Bluetooth headphone. It compresses the audio and hides the high-frequency details.

One of the episode's clues is wrong (maybe by mistake or intended), but it could be easily recovered by rethinking the slogan "hacktosafety" of this series.
