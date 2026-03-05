# X) Read & Summarize

## x.1) Schneier 2025. [Applied Cryptography: Chapter 1 - foundations](<https://learning.oreilly.com/library/view/applied-cryptography-protocols/9781119096726/08_chap01.html>)

**1.1 Terminology**
- **Plaintext**  : Self explanatory
  - Denoted by _M_ for message, or _P_ for plaintext
  - The data in itself can take many forms, but as far as the computer is concerned, it's just binary data
- **Ciphertext** : Encrypted plaintext
  - Denoted by _C_
  - Also binary data. Sometimes bigger than _M_, sometimes larger
- **Encryption** : Disguising a message in order to hide its substance
  - E(M) = C
- **Decryption** : Turning ciphertext back into plaintext
  - D(C) = M
- **Cryptographic algorithm** A.K.A a **Cipher** : Mathematical function used for encryption and decryption
  - If the way an algorithm works is kept a secret, it's a **restricted** algorithm.
    - (Consider legacy. Doesn't scale well. No quality control or standardization)
  - Modern cryptography solves this problem with a **key**
  - The _range of possible values_ for the aforementioned is called the **keyspace**
  - The function now becomes:
  - Ek(M) = C & Dk(C) = M
  - Some algorithms use different keys for _encryption and decryption_ operations (k1 & k2)
  - The security now depends on the keys. The algorithm can be public and analyzed. It doesn't matter if someone know the algorithm, as long as they don't have the key(s) they can't read the message

Expanding on the previous topic. There are two general types of key-based algorithms:

**Symmetric algorithms**
- The encryption key can be calculated from the decryption key and vice versa
- Most of the time _k1 and k2_ are the same!
- This requires that both parties agree on a key before secure communications can take place
- The key must remain a secret!
- Two categories within this one:
  - **Stream ciphers**
    - _Operate on the plaintext a single bit or byte at a time_
  - **Block ciphers**
    - _Operate on the plaintext in groups of bits_
    - Modern computer algorithms have a block size of 64 bits

Public-key algorithms
 


In addition to providing **confidentiality**, cryptography is often also asked to do:
- **Authentication**
  - It should be possible for the receiver to ascertain the origin of a given message
- **Integrity**
  - It should be possible to verify a message has not been tampered with during transit
- **Nonrepudiation**
  - A sender should not be able to falsely deny that he sent a message






1.4 Simple XOR

1.7 Large Numbers




## x.2) Python basics for hackers - Karvinen 2024
