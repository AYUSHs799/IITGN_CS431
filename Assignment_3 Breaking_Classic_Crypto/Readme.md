# <p style="text-align: center;">INSTRUCTIONS</p>
## <p style="text-align: center;">Please read the instructions properly before starting the assignment! </p>

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

## 1. Breaking Vigenere (2 points)
Your first task is to break the Vigenere cipher (which was considered unbreakable for centuries). Log into the CS431 CTF Server. Do not use the web shell; copy/pasting is hard. Follow the directions provided on Canvas to SSH into the server.

To connect to a network service, you can use netcat (nc). For example, ```”nc 192.168.2.3 80”``` would connect to a service running on port 80.

Solve the Vigenere problem. You can use an online solver for this problem, but please explain how the answer was derived.
Or better, write the solver yourself! Submit a writeup containing your CTF username following the above guidelines.

## 2. One-time Pads (4 points)
One-time pad is unbreakable crypto, but what happens if you reuse the pad? Solve the key-reuse problem on the CS431 CTF Server. The flag is the decrypted 32 decimal character message stored on the server. Submit a writeup containing your CTF username following the above guidelines.

## 3. ECB (4 points)
As discussed in the lecture, it is bad to use ECB mode with block cryptography. Solve the ECB problem on the CS431 CTF Server. Submit a writeup containing your CTF username following the above guidelines.