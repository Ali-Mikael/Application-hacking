# A) Hidden strings
**OBJECTIVE:**
- Download [ezbin-challenges.zip](<https://terokarvinen.com/loota/yctjx7/ezbin-challenges.zip>)
- Run `passtr`
- Find the password

**Download the target:**
```bash
$ wget https://terokarvinen.com/loota/yctjx7/ezbin-challenges.zip
```

**Decompress:**
```bash
$ unzip ezbin-challenges.zip
```

**Move into the challenge directory uncovered from the zip file:**
```bash
$ cd challenges
```

**We have two programs:**
- <img width="580" height="106" alt="Screenshot from 2026-01-26 14-34-31" src="https://github.com/user-attachments/assets/c1ce9b66-bd07-47ec-b5fd-39af99fc2b12" />

**We run the other one:**
- <img width="657" height="295" alt="Screenshot from 2026-01-26 14-36-02" src="https://github.com/user-attachments/assets/ddac4d9d-8346-4fb7-a58d-eceb89a219a3" />



We don't know the password, so we quit the program and inspect it using `strings`:
```zsh
$ strings passtr
```
**Q:** What is `strings`?      
**A:** A utility used to print the sequences of printable characters in files, particularly useful for binaries!              

And just like that we're able to retrieve the password!
- <img width="778" height="494" alt="Screenshot from 2026-01-26 14-38-43" src="https://github.com/user-attachments/assets/618fd177-22cf-4f0b-ae44-2914b7327d57" />

Now we can type it in the program while it's none the wiser:
- <img width="817" height="144" alt="Screenshot from 2026-01-26 15-18-06" src="https://github.com/user-attachments/assets/65eeaa0b-aee0-4965-991f-f0d80c649795" />



# B) Hiding in plain sight
**OBJECTIVE:**
- Make a new version of the `passtr.c` program where the password doesn't appear directly as-is in the binary.
- Demonstrate with a test that it works.     


The task was much harder than I anticipated (especially because i'm not familiar with the C language). It actually took me a while to get it.      
At first I tried to obfuscate a variable holding the password. The problem here is that you have to initialise a variable with the password, and it will show up when using `strings`.

So my instinct said: Don't write it down in plain text anywhere.      


**This is what I came up with:**
- First transform the string into bytes (represented as hex), then `XOR` it with a non printable character like `0x1b`
- Add the encoded array into the program manually
- Byte array is then **decoded at runtime**


**What is XOR?**
- **XOR = eXclusive OR**
- One of the foundational **logic gates** used in computing
- **Nice to know:**
  - Modern computers use _transistors_ to form what are known as `logic gates`
  - They take input and produce output, the operation which takes place is based upon boolean algebra
  - Fun fact: A modern CPU consists of billions upon billions of transistors
- A logic gate commonly takes 2 (allthough can take more) inputs, and produces an **output**
  - Except the `NOT gate` a.k.a _the inverter_ which only takes one input and produces one output
- **To better understand the XOR gate:**
  - It's derived from the `OR gate`
    - Which will _always output a `1`_ if either `input1` **OR** `input2` **OR** `both` are `1`, otherwise it will output a `0`
- The `XOR gate` works in the same fashion, but in contrast is `exclusive`, meaning that **only** _either one_ can be a `1` to get an output of `1`.
- **Simplified:**
  - If both values are the same, it will be **FALSE**, a.k.a the output is `0`
  - When exactly **a single input** is `1`, it will be **TRUE**, a.k.a the output `1`

**The XOR truth table:**
| input1 | input2 | Output |
| -- | -- | ------ |
| 0  |  0 |  0     |
| 1  |  0 |  1     |
| 0  |  1 |  1     | 
| 1  |  1 |  0     | 

If you'd like to know more about XOR and how it's applied, I found this [interesting article](<https://accu.org/journals/overload/20/109/lewin_1915/>).      
Here is more on [logic gates](<https://learnabout-electronics.org/Digital/dig21.php>).     

I found a [list of nonprintable characters](<https://condor.depaul.edu/sjost/lsp121/documents/ascii-npr.htm>) and chose the escape character (represented as `1b` in hex).     
At first I tried to write the encoder myself, but without any prior experience with C, this proved to be properly cumbersome. With python I could've **maybe** done it myself.     
As the next natural step, I asked chatGPT to create me simple `XOR program for C`.      
```c
#include <stdio.h>
#include <string.h>

#define KEY 0x1b

int main(void) {
    char input[256];

    printf("Enter string to encode: ");
    if (!fgets(input, sizeof input, stdin))
        return 1;

    // strip nline
    input[strcspn(input, "\n")] = 0;

    printf("\nEncoded bytes:\n\n");

    for (size_t i = 0; i < strlen(input); i++) {
        printf("0x%02x,", (unsigned char)(input[i] ^ KEY));
        if ((i + 1) % 8 == 0)
            printf("\n");
    }

    printf("0x00\n");
    return 0;
}
```
We then compile and run the program, and as a result get the byte sequence:
- <img width="653" height="206" alt="Screenshot from 2026-01-26 18-22-29" src="https://github.com/user-attachments/assets/308c63ad-8dd2-4e17-aad8-72ecc49457f6" />     

We then paste the secret into our main program which decodes the array at runtime using the same key (0x1b).
```c
// passtr - a simple static analysis warm up exercise
// Copyright 2024 Tero Karvinen https://TeroKarvinen.com
// Modified by Ali G 2026

#include <stdio.h>
#include <string.h>

int main() {
        char input[20];

        unsigned char pword[]= {
        0x68,0x6e,0x6b,0x7e,0x69,0x36,0x68,0x7e,
        0x78,0x69,0x7e,0x6f,0x00
        };

        // decode
        for (size_t i = 0; pword[i]; i++)
                pword[i] ^= 0x1b;

        printf("What's the password?\n");
        scanf("%19s", input);

        if ( strcmp(input, pword) == 0 ) {
                printf("Yes! That's the password. FLAG{Tero-d75ee66af0a68663f15539ec0f46e3b1}\n");
        } else {
                printf("Sorry, no bonus.\n");
        }
        return 0;
}
```

Compile the program and run it:
```zsh
$ gcc passtr.c -o passtr2
```
<img width="784" height="142" alt="Screenshot from 2026-01-26 18-24-17" src="https://github.com/user-attachments/assets/cdc0a2ac-2740-423b-a3d1-f2b87fbd615c" />     


**No traces of the password:**
- <img width="779" height="585" alt="Screenshot from 2026-01-26 18-22-51" src="https://github.com/user-attachments/assets/8987e3dc-9f97-44fa-979e-cb670dfd8d4b" />
- <img width="673" height="129" alt="Screenshot from 2026-01-26 18-23-29" src="https://github.com/user-attachments/assets/7a57b0af-ec17-4e6d-a3b8-4c3b764ce9d8" />     


**Passtr version 1 for reference:**
- <img width="677" height="87" alt="Screenshot from 2026-01-26 18-33-21" src="https://github.com/user-attachments/assets/2e78700b-92bc-453c-aee4-2a7f6e1a0e4d" />



# C) Packd
Objective is same as before, put this time a bit more challenging apparently.

<img width="664" height="446" alt="Screenshot from 2026-01-27 20-52-17" src="https://github.com/user-attachments/assets/9db5fe87-5d5c-4c41-bd67-257601dbf3bb" />


I started off with a simple
```
$ strings packd
```
<img width="449" height="712" alt="image" src="https://github.com/user-attachments/assets/7b8ad732-376d-4cbe-8b55-76c48849bec0" />


This time the password is not clearly visible, and deducing from the flag, the contents are properly scrambled.

I started by gathering stuff that look like it could take us somewhere:
```
$Info: This file is packed with the UPX executable packer http://upx.sf.net $
$Id: UPX 4.21 Copyright (C) 1996-2023 the UPX Team. All Rights Reserved. $

/proc/self/exe

GCC: (Debian 12.

GLOBAL_OFFSET_TABL

.gnu.prop
```
We can atleast deduce that the program was compressed by the [UPX packer](<https://upx.github.io/>)....

I then did some digging, and found a CLI tool for UPX. I inspected the man pages and `upx --help` for help.


I found a decompress option, so I decided to use that one:
```
$ upx -d packd
```
And wouldn't you know:

<img width="784" height="521" alt="Screenshot from 2026-01-27 21-32-52" src="https://github.com/user-attachments/assets/09a32072-cf56-4247-ac0c-6a18310c1a97" />

<img width="777" height="134" alt="Screenshot from 2026-01-27 21-33-23" src="https://github.com/user-attachments/assets/b0c8a5bd-6cc6-45f5-8998-f8c69379b3af" />


Basically we just had to uncompress the program in order to read it cleanly. 




# D) Optional Bonus: Cryptopals
The challenge set can be found at [here](<https://www.cryptopals.com/sets/1>)!


> [!NOTE]
>
> This is not my primary concern at the moment, so I will be doing this on the side during the upcoming weeks!


Let's start with:

### [1. Convert hex to base64](<https://www.cryptopals.com/sets/1/challenges/1>)
The string:
```
49276d206b696c6c696e6720796f757220627261696e206c696b65206120706f69736f6e6f7573206d757368726f6f6d
```

Should produce:
```
SSdtIGtpbGxpbmcgeW91ciBicmFpbiBsaWtlIGEgcG9pc29ub3VzIG11c2hyb29t
```
