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

**Q:** What is XOR?     
**A:**
- **XOR = eXclusive OR**
- One of the foundational **logic gates** used in computing
- Good to know:
  - Modern computers use _transistors_ to form what are known as `logic gates`
  - They take input and produce output
  - The operation is based upon boolean algebra
  - Fun fact: A modern CPU consists of billions upon billions of transistors
- A logic gate commonly takes 2 (allthough can take more) inputs, and produces an **output**
  - Except the `NOT gate` a.k.a the inverter which only takes one input and produces one output
- The `XOR gate` takes two values, 1 and/or 0
  - If both values are the same, the output is **FALSE**, a.k.a `0`
  - If the two values are different, the output is **TRUE**, a.k.a `1`

The XOR truth table:
| V1 | V2 | Output |
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


No traces of the password:
- <img width="779" height="585" alt="Screenshot from 2026-01-26 18-22-51" src="https://github.com/user-attachments/assets/8987e3dc-9f97-44fa-979e-cb670dfd8d4b" />
- <img width="673" height="129" alt="Screenshot from 2026-01-26 18-23-29" src="https://github.com/user-attachments/assets/7a57b0af-ec17-4e6d-a3b8-4c3b764ce9d8" />     


Passtr version 1 for reference:
- <img width="677" height="87" alt="Screenshot from 2026-01-26 18-33-21" src="https://github.com/user-attachments/assets/2e78700b-92bc-453c-aee4-2a7f6e1a0e4d" />





