# X) Read & Summarize

## x.1) Schneier 2025. [Applied Cryptography: Chapter 1 - foundations](<https://learning.oreilly.com/library/view/applied-cryptography-protocols/9781119096726/08_chap01.html>)

### 1.1 Terminology
- **Plaintext**  : Self explanatory
  - Denoted by _M_ for message, or _P_ for plaintext
  - The data in itself can take many forms, but as far as the computer is concerned, it's just binary data
- **Ciphertext** : Encrypted plaintext
  - Denoted by _C_
  - Also binary data. Sometimes bigger than _M_, sometimes smaller
- **Encryption** : Disguising a message in order to hide its substance
  - E(M) = C
- **Decryption** : Turning ciphertext back into plaintext
  - D(C) = M
- **Cryptographic algorithm** A.K.A a **Cipher** : Mathematical function used for encryption and decryption
  - If the way an algorithm works is kept a secret, it's a **restricted** algorithm.
    - (Consider legacy. Doesn't scale well. No quality control or standardization)
  - Modern cryptography solves this problem with a **key**. Denoted _K_
  - The _range of possible values_ for the aforementioned is called the **keyspace**
  - The function now becomes:
  - Ek(M) = C & Dk(C) = M
  - Some algorithms use different keys for _encryption and decryption_ operations (_k1_ & _k2_)
  - The security now depends on the keys. The algorithm can be public and analyzed. It doesn't matter if someone know the algorithm, as long as they don't have the key(s) they can't read the message

Expanding on the previous topic. There are two general types of key-based algorithms:

**A: Symmetric algorithms**
- The encryption key can be calculated from the decryption key and vice versa
- Most of the time _k1 and k2_ are the same!
- This requires that both parties agree on a key before secure communications can take place
- The key must remain a secret!
- Two categories within this one:
  - **Stream ciphers**
    - _Operate on the plaintext a single bit or byte at a time_
  - **Block ciphers**
    - _Operate on the plaintext in groups of bits_
    - Good to know: Modern computer algorithms have a block size of 64 bits

**B: Public-key / Asymmetric algorithms**
- Uses k1 and k2
- The catch is that the _decryption key_ a.k.a _private key_ **cannot be calculated** from the _encryption key_ a.k.a _public key_
  - (at least in a reasonable amount of time)
- Which means that the encryption key can be made public, hence the name!
- Which again means that anyone can encrypt their message using someones public key, but only the one with the corresponding private key can decrypt it!
- Encryption looks like: Ek(M) = C
- A quick note on **digital signatures**:
  - When signing digitally, sometimes messages will be encrypted with the private key and decrypted with the public key.


In addition to providing **confidentiality**, cryptography is often also asked to do:
- **Authentication**
  - It should be possible for the receiver to ascertain the origin of a given message
- **Integrity**
  - It should be possible to verify a message has not been tampered with during transit
- **Nonrepudiation**
  - A sender should not be able to falsely deny that he sent a message


**Cryptanalysis**
- The science of recovering the plaintext of a message without access to the key
- An attempted cryptanalysis = an attack

**4 General types of cryptanalytic attacks**:
  1. **Ciphertext-only attack**
     - The attacker has several ciphertexts encrypted using the same key
     - The job here is to recover the plaintext of as many messages as possible, or better yet aquire the key
  2. **Known-plaintext attack**
     - Same as before but additionally has access to the plaintexts as well
     - Job is to deduce the key(s)
  3. **Chosen-plaintext attack**
     - Same as before, but the attacker will choose which plaintext messages to encrypt
     - Why? Because the attacker might choose to decrypt blocks that might yield more results than other in cracking the key(s)
  4. **Adaptive-chosen-plaintext attack**
     - Same as previous, but in addition, the attacker is able to iteratively modify his choises based on previous results


The complexity of an attack can be measured like the following:
**1. Data complexity**
   - Amount of data needed as input for the attack
**2. Processing complexity**
   - Time needed to perform the attack a.k.a the work factor
**3. Storage requirements**
   - Amount of memory needed to perform the attack


**Final note**
- Only a **one-time pad** is unbreakable given infinite resources
  - The core idea is that the _encryption key_ is truly random & the same length as the message
  - The key is than `XOR`ed against the message to produce the ciphertext
  - **4 things to keep in mind for a truly unbreakable one-time pad encryption:**
    - The key has to be truly random
    - Key length has to equal the message length
    - The key is used only once
    - And lastly the obvious: The attacker doesn't have access to the one-time pad
- All other cryptosystems can be broken one way or another
  - That said, keep in mind that even if it's theoretically possible, it doesn't mean it's feasible. Especially for people like you and me with our limited resources. So don't go throwing your computer aways just because it was said that all cryptosystems can theoretically be cracked 😂




### 1.4 Simple XOR
I already touched on this subject in my `h3 - section: B` assigment, but to recap:
- Two bit streams are put side by side, and each bit is `XOR`'d, meaning the output will be a `1`, **ONLY** if **EITHER** value is a `1`, otherwise it will equal to `0`

When used in encryption, this means that `XOR`ing the same value twice restores the original
Meaning:
- plaintext `XOR` key = ciphertext
- ciphertext `XOR` key = plaintext

Here's how you break it:
1. Counting coincidences
   - You discover the length of the key by XORing the ciphertext against itself shifted various number of bytes, and count the bytes that are equal
   - If the offset is a multiple of the key length, then ~ 6% of the bytes will be equal
   - If not, then less than 0.4% will be equal (assuming ASCII text being encrypted using a random key)
   - The smallest offset that indicates a multiple of the key length = is the length of the key.

2. Discovering the plainttext
   - Shift the ciphertext by the discovered length and XOR it with itself.


### 1.7 Large Numbers
In order to give some perspective:
- The odds of being killed by a lightning (per day) = 1 in 9 billion (2^33)
- Age of our planet (estimate) = Around 2^30 years
- Number of atoms in the galaxy = 2^223


## x.2) [Python basics for hackers](<https://terokarvinen.com/python-for-hackers/>) - Karvinen 2024

**Feedback is the Breakfast of Champions**
- Try the smallest thing possible, then iteratively add on top of it
- Fast feedback loop makes it faster & more pleasant to code
- Working in small increments is helped by:
  - REPL: Read-Eval-Print
    - For python: `python3` & `ipython3`
    - ipython is the CLI version of jupyter. You can move up and down in your history, use tab completion and get help quickly from the CLI
  - tries/
    - Having a tries/ directory where you try every new library of reatures separate from the main program
  - F5 compile
    - Press F5 in your editor to compile and show the result

**Assert your beliefs**
- It's a good idea to stop execution of your hopes and dreames don't hold up
- Python is a loosely typed language, so it's convenient to use `assert` here

For example:
```py
name = bob
assert type(name) == int

# results in:
assert type(name) == int
       ^^^^^^^^^^^^^^^^^
AssertionError
```

**Debugging**
- By adding `breakpoint()` into your code, the debugger will stop execution where you tell it to!
- Python has the debugger built it. For the fancy iPython features, you need to set it up
```bash
$ sudo apt install python3-ipdb ipython3
$ export PYTHONBREAKPOINT=ipdb.set_trace
```

**Useful libraries to consider**
- requests (download web pages)
- binascii (convert hex text)
- base64   (binary to ASCII armor)



-------


# Cryptopals
**Objective**
- Solve [CryptoPals set 1](<https://cryptopals.com/sets/1>) challenges
- Use any programming language and text editor/IDE you like. But no AI!
- Solve tasks 1-4
- 5-8 are optional


> [!NOTE]
> I will be using `Python 3.14.3`


# A) Convert hex to base64
**Objective**
- The string:     `49276d206b696c6c696e6720796f757220627261696e206c696b65206120706f69736f6e6f7573206d757368726f6f6d`
- Should produce: `SSdtIGtpbGxpbmcgeW91ciBicmFpbiBsaWtlIGEgcG9pc29ub3VzIG11c2hyb29t`

### My solution
```py
import base64


def convertHexToBase(s):

    raw = bytes.fromhex(s)
    return base64.b64encode(raw).decode("ascii")


if __name__ == "__main__":
    s = input("Hex array to convert to base64 >> ")
    print("Computing...")
    print()
    print(f"Here you go:\n------------\n >> {convertHexToBase(s)}")
```
<img width="1861" height="250" alt="2026-03-06-21:01:53" src="https://github.com/user-attachments/assets/336b149d-6bb0-4544-9cce-8fb189f7ed25" />

**Explanation:**
- We decode the hex string to bytes using the `bytes.fromhex()` function
- Then in turn we `base64` encode the bytes using the `base64.b64encode()` function and further decode the string using ASCII for pretty printing
- This one was relatively simple (so far...)




-------




# B) Fixed XOR
**Objective**
- Write a function that takes two equal-length buffers and produces their XOR combination
- If the function works:
  - You feed it the string:                `1c0111001f010100061a024b53535009181c`
  - After hex decoding and XOR'd against:  `686974207468652062756c6c277320657965`
  - Should produce:                        `746865206b696420646f6e277420706c6179`


### My solution
```py
def xor(s1, s2):

    s1b = bytes.fromhex(s1)
    s2b = bytes.fromhex(s2)

    result = ""

    if len(s1) == len(s2):
        for c1, c2 in zip(s1b, s2b):
            result += chr(c1 ^ c2)

    return bytes(result, 'utf-8').hex()


if __name__ == "__main__":

    s1 = "1c0111001f010100061a024b53535009181c"
    s2 = "686974207468652062756c6c277320657965"
    print(f"XORing:\n-------\n >> {s1}\n and\n >> {s2}")
    print()
    print(f"The result:\n-----------\n >> {xor(s1, s2)}")
```

<img width="897" height="321" alt="2026-03-06-21:11:05" src="https://github.com/user-attachments/assets/356fd5c0-7074-464c-848e-7d65e5770b69" />

I'm not going to lie to you and tell you that I didn't struggle a little bit with this one... I've never used python to operate on binary before, so it took me a little while to fully understand what was really going on, especially with all the type conversions etc. I must've spent around an hour in the python3 shell just trying out different stuff and seeing what `foo` looks like after I poke around `bar` and decode `baz`, and how `o` behaves after i've done `x`, `y` and `z`. 

After many iterations and finally understanding what's what, I came up with a solution.


**Explanation**
- We convert the hex string to bytes same as before
- Then initialize an empty variable to store our result
- Quickly check that the buffers are equal in length
  - Quick note: This is unnecessary here as we know they are, but it's easy to iterate on this code and expand it to handle cases with different buffer lengths later on
- We then use the `zip()` function to combine our two strings into a tuple (which makes it easy to loop through)
- All that's left to do is simply:
  - `xor` the values
  - Convert the result back into a character using `chr()`
  - Append it to our result
- Once we have the XOR'd string, we turn it back to bytes, and then further back to hex
  - Note: We specify `utf-8` so that the machine knows how to accurately represent the string as bytes 
- Are there better ways to do this? 99% sure there is, but this is what I came up with for the time being




-----



# C) Single-byte XOR cipher

**Background**
- The hex encoded string `1b37373331363f78151b7f2b783431333d78397828372d363c78373e783a393b3736` has been XOR'd against a single character


**Objective**
- Find the key, decrypt the message

> [!NOTE]
> You can do this by hand. But don't: write code to do it for you.
>
> How? Devise some method for "scoring" a piece of English plaintext. Character frequency is a good metric. Evaluate each output and choose the one with the best score. 


### My solution
```py

from collections import Counter


def decodeXor(s):

    bs = bytes.fromhex(s)

    rList = []  # <- Result List
    cList = []  # <- Character List

    for i in range(255):
        for c in bs:
            cList.append(chr(c ^ i))  # XOR against each ascii char 1 by 1

        res = "".join(cList)

        if res.isprintable() and res.isascii():  # Sanity check
            rList.append({f"XOR char == {chr(i)}": res})

        cList.clear()
        i += 1

    return rList


def sortDecoded(v):

    cmon = Counter("etaoinsrhdlu")  # Most frequent letters
    rank = []

    for e in v:
        for k, v in e.items():
            top = Counter(dict(Counter(v.lower()).most_common(12)))
            rank.append((k, v, len(top & cmon)))  # Cmp most common against top 12 & count overlap

    rank.sort(key=lambda x: x[2], reverse=True)

    return rank


if __name__ == "__main__":

    s = "1b37373331363f78151b7f2b783431333d78397828372d363c78373e783a393b3736"
    print(f"\nDecrypting >> {s}")
    print()

    viable = decodeXor(s)
    ranked = sortDecoded(viable)

    print("Ranking most likely entries from best to worst:")
    print("-----------------------------------------------")

    for v in ranked:
        print(v)

```
<img width="1420" height="720" alt="2026-03-08-02:06:38" src="https://github.com/user-attachments/assets/faa636cb-3c29-48c7-978d-2847f4b7e2bd" />


### Walkthrough
First off, I just want to say, that this was in no way shape or form and easy task for me 😂
It took me hours upon hours to get the results I wanted. The most amusing part about it all is that after the decrypting and initial sorting using the `isprintable() and isascii()`, I could already see the answer, but I still needed a way to sort and score them (which probably took me about 2 hours)

But don't get me wrong, I enjoyed it thorougly and this was an extremely teaching exercise! 

I want to give a huge shoutout to [GeeksForGeeks](<https://www.geeksforgeeks.org/>), [this guy](<https://note.nkmk.me/en/>), [PythonGeeks](<https://pythongeeks.org/>) and Tero's _python for hackers_ article. Without these I wouldn't have been able to complete the objective, as the python docs are somewhat a tough read, at least for me... Plus you couldn't use AI. Which I normally use to explain certain concepts for me and only afterwards go and legit check. Shoot me.

### TL;DR: Task was hard and had to put in that workkk 🔨

But anyhow, let's get down to business.

Allthough I quickly want to preface this by saying if I had some more time, I could've probably solved it in a smarter and more efficient way, but on this run I just wanted to reach the finish line tbh.

We'll iterate over this at later time when we want to enhance our python skills. So don't be to harsh on my code innit 😂

### Explanation

**Part1 - `decodeXor()`**
- We naturally start by converting hex to bytes
- We "quickly" do a little brute force by going through the whole ASCII table char by char, XOR'ng as we go
- Before appending the result to our `rList`, we do a quick sanity check and remove all entries that don't confine to `isprintable() and isascii()`
  - This way we already _cut down_ on all the possibilities
- If the string passes the check we create a dictionary like so: 
  - **Key** = _Character that was used to XOR with_
  - **Value** = _Decoded string_
- Lastly the function returns the list


**Part2 - `sortDecoded()`**
- In our main function we collect the list of viable entries and pass it along to the second function
- With the inner loop `for k, v in e.items()` we access the string through `v`
```py
for e in viable:
        for k, v in e.items():
            top = Counter(dict(Counter(v.lower()).most_common(12)))   #111
            rank.append((k, v, len(top & cmon))
```
**#111**
- We then create a `Counter` object for the current string in the loop, which gives us a frequency table we can utilize

Here's an example of how it works:

<img width="718" height="94" alt="2026-03-08-02:00:38" src="https://github.com/user-attachments/assets/e0874e0e-240d-4418-a4a2-d67bf97009d1" />

We also do the same for the **12 most frequent characters** in the english language 
```py 
cmon = Counter("etaoinsrhdlu")
```

**The magic**
- We can now access the 12 **most frequently appearing** characters in the **decoded string** by using the _counter method_ `most_common()`
- Store the result in a variable = `top`
- And compare it against `cmon` for any **overlapping** values using `top & cmon`
  - (It's called an `intersection union operation`, and the ampersand (&) works as the operator)
- We wrap the result in `dict()` and further in `Counter()` to create a new Counter object
- Now we can easily count all the overlaps with `len()` A.K.A `length_of(#matching_chars_object#)`
- We then create a tuple with the values
  - `XOR char`
  - `decoded string`
  - `amont of overlaps`
    - and append it to our list
- Read more about using Counter [here](<https://www.geeksforgeeks.org/python/counters-in-python-set-1/>)

Lastly we sort the the list using the _third value in the tuple_ `x[2]` (the overlap counter) and _reverse the order_ to get entries with the most amount of matches at the top of the list.
```py
rank.sort(key=lambda x: x[2], reverse=True)
```

We can also change the last block of code to
```py
print(ranked[0])
```
<img width="1438" height="193" alt="2026-03-08-04:06:50" src="https://github.com/user-attachments/assets/299c1474-c4bd-431e-bfbf-97e7326ef407" />



------



# Detect single-character XOR
Background
- One of the 60-character strings in [this file](<https://cryptopals.com/static/challenge-data/4.txt>) has been encrypted by single-character XOR.

Objective
- Find it
- (Your code from #3 should help.)


Download the file
```bash
$ wget https://cryptopals.com/static/challenge-data/4.txt
$ cat 4.txt
```
<img width="1573" height="498" alt="2026-03-08-04:14:05" src="https://github.com/user-attachments/assets/1d301738-1830-4a82-b330-ac4fee5a0509" />


### My solution
```py

```






