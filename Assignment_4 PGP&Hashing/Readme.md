# <p style="text-align: center;"> INSTRUCTIONS </p>
## <p style="text-align: center;"> Please read the instructions properly before starting the assignment! </p>

1. Submit to Canvas a PDF file containing your explanations and your code files before the due date. Late submissions incur penalties as described in the first lecture (first you use up grace hours, then you lose points). Note that only PDF submissions will be graded.

2. You must use at most one page for your explanation for each problem (code you write may be on additional pages). Start each problem on a new page.

3. Refer to these pages to set up CTF accounts:

<ul><ul>
<li> 

[CTF for Linux/Mac Users](https://canvas.instructure.com/courses/6004067/pages/connecting-to-the-ctf-server-for-linux-slash-mac)

<li>  

[CTF for Windows Users](https://canvas.instructure.com/courses/6004067/pages/connecting-to-the-ctf-server-for-linux-slash-mac)

</ul></ul>

4. To access the CTF problems, create the SSH proxy as per the guide on Canvas. CTF problems have different flags for different users; so, do not copy and paste someone’s flag as it would be reported as incorrect, and will be recorded in the server log. Any such attempt would lead to 0 points in the assignment (and shall be reported to the Academic Office for violation of the honor code).

5. For CTF problems, you must use the following format in your explanation:
<ul><ul>

<li>CTF Username.  

<li>Flag.  

<li> Explain the vulnerability in the program, and explain conceptually how that vulnerability can be exploited to get the flag.  

<li> How did you exploit the vulnerability? List the steps taken and the reasoning behind each step. We should be able to replicate the exploit following the steps. Feel free to make references to your code, and use external references for ideas (with proper citations)! Note that ”use XYZ online solver” is not sufficient - you must explain how the online solver derived the answer for full credit.

<li>Append your source code in the same writeup. Your source code should be readable from the writeup PDF itself. Note that this does not count towards the page count above.
Omitting any of the sections above would result in points being deducted.

</ul></ul>

6. Be neat and concise in your explanations.

7. Feel free to discuss the assignment in general terms with other people, but the answers must be your own. You are encouraged to shared re- sources (e.g., online resources you found helpful) and discuss homework assignment solutions.

8. Post any clarifications or questions regarding this homework in Canvas Discussions

9. References: List resources outside of class material that helped you solve a problem. This includes online video tutorials, other problems on other platforms, etc. Remember that source code available online (e.g. stackoverflow) also needs to be cited.

--- 

## 1. Using PGP (4 points)

In class we learnt about public key encryption and signing, time to use it! This problem will require you to create a public/private OpenPGP key pair, using, for instance the GNU Privacy Guard (gpg), or similar package and share it on a key-server. You must provide us the fingerprint for the key you upload to the keys.mailvelope.com server.

Our email address is (Not provided to public). The OpenPGP fingerprint is (Not provided to public), and the public key is available on https://keys.mailvelope.com/. GPG may require using the HTTP Keyserver protocol address for the mailvelope server: hkps://keys.mailvelope.com/.

<ul>

1. Based on this information, how do you verify that the public key you got from the key-server is valid, i.e., that no one has modified it?

2. Complete the PGPChatbot Challenge on the CTF Server. Include your key’s fingerprint in the write-up.
Important note: Make sure to use your IITGN email address while uploading your public key to keys.mailvelope.com for this problem.

These webpages contain useful resources:

* http://www.gnupg.org/ - the GNU privacy guard homepage

* https://www.devdungeon.com/content/gpg-tutorialLinks to an external site. for installing gpg on your system and all commands you’ll ever need (including this problem).

* https://www.base64encode.org/ - Might come in handy to base64 encode your signed/encrypted messages.

In addition, please note:

* We recommend using gpg, a PGP utility by GNU. gpg is installed by default on WSL(ubuntu), ubuntu and MacOS. You will have to install gpg yourself on native Windows (fairly simple). You are free to use any other utility, too.

* If you use gpg over SSH without a GUI, ”–pinentry-mode loopback” flag might come in handy

* For MacOS users, you might face issues while pasting large chunks of data to the CTF problem due to MacOS terminal restrictions. In such case, put your input in a file (one input per line), and pipe the content of the file to your netcat session:


```ps
 cat input.txt | nc 192.168.2.3 XXXX
```

</ul>

## 2. Hash extension (6 points)
SHA256 is a commonly-used algorithm for hashing, based on Merkle–Damgard construction. What happens if it is used as a message-authentication code? Can you modify a message and regenerate its hash without knowing the original authentication key?

Solve the Hash Extension problem on the CTF server.

Start with the file ```pure_python_sha256.py``` which is a simple Python 3 implementation of SHA256 which you can modify in order to implement the extension attack. This problem does NOT require any brute force, it can be solved just by connecting to the server twice (once to get the original cookie and its hash, and the second time to submit your modified cookie and re-calculated hash).

--- 
### Information on SHA256: 


<b>SHA256 Pseudocode:</b> The ”Pseudocode” section of https://en.wikipedia.org/wiki/SHA-2#Pseudocode is a good reference for the SHA256 algorithm.

<b>SHA256 Padding:</b> The SHA256 algorithm operates on 64-byte chunks of the message. It will add padding in the following form (even if the message is already a multiple of 64-bytes long, it will still add padding): First it will append a single byte 0x80. Then, it will append 0x00 until the length is 8 bytes less than a multiple of 64 bytes (note that if the original length were, say, 63, then there would be one byte of 0x80 and then 56 bytes of 0x00 added). Finally, it adds the original (pre-padding) length of the message (in BITS, not BYTES). As an example:

* Start with a 50-character string ”XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXABC”
* In hex this is 50-bytes long: 5858...5858414243
* Append a 0x80, making it 51 bytes long: 5858...585841424380
* Append 0x00, until it is 8 less than a multiple of 64 bytes long (56 bytes): 5858...5858414243800000000000
* Append the original length of the unpadded message (50 bytes = 400 bits = 0x190 in hex): 5858...58584142438000000000000000000000000190
* Unlike the padding oracle attack, this attack itself is not based on padding, but you will need to manually pad the message before length-extending it.

<b>SHA256 on Long Strings: </b> The SHA256 algorithm is meant to operate on 64-byte blocks using Merkle–Damgard construction. For longer strings, it loops over itself multiple times, maintaining a running hash value consisting of 8 32-bit integers (often referred to as h0, h1, h2, h3, h4, h5, h6, h7). At the start of the hash, these are initialized to a fixed set of values (defined by the SHA256 specification), and during each loop iteration these 8 values are updated. At the end of the hash, these 8 values are converted to hex, concatenated together, and outputted as the hash value.  