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
- Two bit streams are but side by side, and each bit is `XOR`ed, meaning the output will be a `1`, **ONLY** if **EITHER** value is a `1`, otherwise it will equal to `0`

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

results in:
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

This one I did already, here's my solution:
```py
import base64


def convertHexToBase(s):

    raw = bytes.fromhex(s)
    b64 = base64.b64encode(raw).decode("ascii")
    return b64


if __name__ == "__main__":
    s = input("Give me the hex sequence to convert to base64: ")
    print("All done:")
    print(convertHexToBase(s))
```
<img width="1861" height="198" alt="2026-03-05-23:16:59" src="https://github.com/user-attachments/assets/0d2376c6-0b94-4e78-85a3-7c0e85750142" />


# B) Fixed XOR
**Objective**
- Write a function that takes two equal-length buffers and produces their XOR combination
- If the function works:
  - You feed it the string:                `1c0111001f010100061a024b53535009181c`
  - After hex decoding and XOR'd against:  `686974207468652062756c6c277320657965`
  - Should produce:                        `746865206b696420646f6e277420706c6179`



My solution:
```py

```


